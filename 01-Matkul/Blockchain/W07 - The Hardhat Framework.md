---
matkul: Blockchain
minggu: 7
sks: 2
sumber: Hardhat Framework for Smart Contract.pptx
tags: [kuliah/blockchain, minggu/w07]
status: draft
diproses: 2026-09-03
---

# W07 — The Hardhat Framework

## Ringkasan
> - **Hardhat** = development environment untuk Ethereum: edit, compile, debug, deploy smart contract dan dApp dalam satu tempat.
> - Dirancang di sekitar dua konsep: **tasks** (fungsi async JavaScript bermetadata) dan **plugins** (tulang punggung Hardhat; kamu bebas pilih sendiri).
> - **Hardhat Network** = node Ethereum lokal buat development, jalan pakai implementasi EVM versi JavaScript di mesinmu sendiri.
> - Struktur proyek: `contracts/`, `deployments/`, `test/`, `hardhat.config.js`.
> - Alur lengkap satu proyek: install → tulis contract → test dengan chai → simpan private key di `.env` → `npx hardhat compile` → deploy ke testnet → **verify di Etherscan**.
> - Hands-on-nya berat: **merekonstruksi Parity wallet hack** (150.000+ ETH dicuri) memakai fitur *mainnet forking*, *account impersonation*, dan *block pinning* milik Hardhat.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Hardhat | Development environment untuk software Ethereum |
| Task | Fungsi async JavaScript dengan metadata; semua yang bisa dilakukan Hardhat itu task |
| Plugin | Tulang punggung Hardhat, dibangun dengan config DSL yang sama dengan file konfigurasi |
| Hardhat Network | Node jaringan Ethereum lokal bawaan, untuk development |
| GETH | Implementasi client Ethereum paling banyak dipakai, ditulis dalam Go |
| Ethers | Library untuk berinteraksi dengan Ethereum |
| Waffle | Framework untuk testing smart contract |
| chai | Library assertion yang dipakai untuk unit testing |
| ABI | Application Binary Interface; cara berinteraksi dengan contract, berisi spesifikasi fungsi |
| Infura / Alchemy | Layanan penyedia akses node Ethereum (Alchemy menyediakan archival node) |
| Contract verification | Mempublikasikan source code contract di Etherscan supaya bisa diperiksa publik |
| Mainnet forking | Menyalin state mainnet ke jaringan lokal untuk eksperimen |
| Account impersonation | Bertindak seolah-olah kamu pemilik suatu akun (khusus di jaringan lokal) |
| Block pinning | Menyetel state chain ke nomor block tertentu (butuh archival node) |
| Multi-sig wallet | Wallet dengan beberapa owner dan ambang batas minimum tanda tangan |
| Threshold | Jumlah minimum tanda tangan yang dibutuhkan untuk mengeksekusi transaksi |

## Isi

### Apa itu Hardhat
**Hardhat adalah development environment untuk software Ethereum.** Dia terdiri dari komponen-komponen berbeda untuk **editing, compiling, debugging, dan deploying** smart contract dan dApp, yang semuanya bekerja bersama membentuk satu development environment yang lengkap.

Hardhat dirancang di sekitar konsep **tasks** dan **plugins**:

- **Plugins** — tulang punggung Hardhat. Sebagian besar fungsionalitas Hardhat datang dari plugin, dan sebagai developer kamu bebas memilih yang mau kamu pakai. Plugin dibangun memakai config DSL yang sama dengan yang kamu pakai di konfigurasi Hardhat.
- **Tasks** — sebuah task adalah **fungsi async JavaScript dengan metadata terkait**. Metadata itu dipakai Hardhat untuk mengotomatiskan beberapa hal: parsing argumen, validasi, dan pesan bantuan sudah ditangani. **Semua yang bisa kamu lakukan di Hardhat didefinisikan sebagai task.**

Hardhat bersifat **unopinionated** soal tools yang akhirnya kamu pakai, tapi datang dengan beberapa default bawaan yang bisa di-override.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi: Hardhat itu **bengkel**, bukan mobil. Dia gak maksa kamu pakai merek obeng tertentu — dia cuma nyediain meja kerja, lift, dan rak, lalu kamu yang milih sendiri mau pasang alat (plugin) yang mana. Bandingannya, framework yang *opinionated* itu kayak paket bengkel resmi: semua alat sudah ditentukan, gampang tapi kaku.

### Hardhat Network
Hardhat sudah **built-in dengan Hardhat Network**, yaitu node jaringan Ethereum lokal yang dirancang untuk development.

Penjelasan slide soal kenapa ini mungkin: Ethereum pada intinya adalah **sekumpulan spesifikasi** yang harus dipatuhi semua client. Ada berbagai implementasi dari protokol Ethereum (yaitu, client), yang paling banyak dipakai adalah **GETH** (ditulis dalam Go).

Di balik layar, **Hardhat memakai implementasi EVM versi JavaScript** untuk menjalankan file-filemu. Artinya kamu menjalankan **Ethereum JS di mesinmu sendiri**. Itulah cara Hardhat tau apa yang harus dilakukan saat kamu mengirim transaksi, melakukan testing, dan men-deploy contract secara internal.

### Struktur proyek
| Folder / File | Isinya |
| --- | --- |
| `contracts/` | Semua contract dan contract turunanmu — termasuk semua contract, interface, library, dan abstract contract yang kamu buat. Pengecualiannya cuma kalau kamu mengimpor contract lain lewat package npm |
| `deployments/` | Script untuk men-deploy contract ke sebuah jaringan |
| `test/` | Semua test case. Praktik yang baik: pisahkan test per file contract |
| `hardhat.config.js` | File konfigurasi Hardhat |

### Setup environment
Instalasi awal:

```bash
mkdir hardhat-tutorial
cd hardhat-tutorial
mkdir project1
cd project1
npm init -y
npm install --save-dev hardhat
```

Verifikasi instalasi dengan menjalankan `npx hardhat`. Dari menu yang muncul, pilih opsi **Create an empty hardhat.config.js**, yang akan memberi file konfigurasi Hardhat yang kosong.

Lalu install plugin-plugin yang paling umum dipakai:

```bash
npm install --save-dev @nomiclabs/hardhat-ethers ethers @nomiclabs/hardhat-waffle ethereum-waffle chai
```

**Ethers** adalah library untuk berinteraksi dengan Ethereum, dan **Waffle** adalah framework untuk testing smart contract.

Terakhir, buka `hardhat.config.js` dan tambahkan kode yang di-require: `hardhat-ethers` dan `hardhat-waffle`, plus memberi tahu Hardhat bahwa kita mau memakai compiler Solidity versi **"0.8.8"**.

### Membuat dan menguji contract
Pertama, buat direktori `contracts` sesuai struktur proyek di atas. Lalu buat file bernama **`Token.sol`** (kodenya diambil dari `https://pastebin.com/BZN2bdS0`).

Slide mengingatkan: **praktik yang baik adalah menamai file sama dengan nama contract-nya.** Contract ini contract Token yang sangat sederhana (**bukan ERC-20 compliant**) di mana seluruh suplai awal diberikan ke owner.

Untuk testing, buat folder `test`, lalu di dalamnya buat file **`token.js`** (dari `https://pastebin.com/Um0rm0Kf`). Testing memakai library **chai** untuk unit testing. Jalankan dengan:

```bash
npx hardhat test
```

Kalau semua benar, semua test case akan lolos.

### Deploy contract
**1. Amankan private key.** Install dependency `dotenv` dan buat file `.env`:

```bash
npm install dotenv
nano .env
```

File ini dipakai untuk **menjaga private key smart contract kita tetap aman** saat push kode ke Github atau tempat lain. Slide men-deploy ke **jaringan Rinkeby** lewat layanan API **Infura** (`https://app.infura.io/login`) — buat akun di sana lalu update file `.env`.

**2. Tentukan jaringan.** Di `hardhat.config.js`, tambahkan keyword **`networks`** untuk memberi tahu Hardhat di jaringan mana kita mau deploy contract, misalnya `"rinkeby"`, `"ropsten"`, atau `"mainnet"`.

**3. Buat script deploy.**

```bash
mkdir deployments
touch deployments/deployToken.js
```

Kodenya dari `https://pastebin.com/2kDyTDm3`. Isi script deploy itu, baris per baris:

| Baris | Artinya |
| --- | --- |
| `const initialSupply = ethers.utils.parseEther("100000");` | Membuat variabel `initialSupply` bernilai 100.000 × 10^18 |
| `const [deployer] = await ethers.getSigners();` | Deployer contract-nya — alamat dari private key yang disediakan di file `.env` |
| `const tokenFactory = await ethers.getContractFactory("Token");` | Abstraksi contract versi Ethers, supaya bisa di-deploy |
| `const contract = await tokenFactory.deploy(initialSupply);` | Baris yang men-deploy contract dengan `initialSupply` sebagai argumen constructor |

**4. Compile.** Slide menegaskan alasannya: **EVM gak tau apa itu Solidity, dia gak ngerti. Dia cuma ngerti bytecode**, kode yang bisa dibaca mesin. Jadi contract harus dikompilasi jadi bytecode Ethereum:

```bash
npx hardhat compile
```

Semua informasi relevan seperti **ABI (application binary interface)** dan bytecode akan berada di `artifacts/contracts/Token.sol/Token.json`. **ABI pada dasarnya adalah cara kita berinteraksi dengan contract** — dia berisi semua spesifikasi fungsi seperti argumen, state mutability, dan nama. Di bagian bawah file itu juga ada bytecode contract-nya.

**5. Deploy.**

```bash
npx hardhat run deployments/deployToken.js --network rinkeby
```

Kalau berhasil, alamat contract yang ter-deploy akan muncul.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa ABI penting: bytecode di blockchain itu cuma angka. ABI adalah **daftar menu**-nya — tanpa ABI, kamu tau ada restoran di alamat itu tapi gak tau apa yang bisa dipesan dan gimana cara mesennya. Itu sebabnya frontend dApp selalu butuh ABI, bukan cuma alamat contract.

### Verifikasi contract di Etherscan
Memverifikasi contract di Etherscan itu penting **supaya orang bisa melihat source code-nya**. Slide kasih dua alasan:
- **Meningkatkan kepercayaan**, karena orang bisa melihat sumber dari protokol yang mereka pakai.
- **Meningkatkan keamanan**, karena akan ada lebih banyak mata yang memeriksa, sehingga lebih banyak user yang ikut memverifikasi keamanannya.

Caranya: buka `https://rinkeby.etherscan.io/`, masukkan alamat yang barusan di-deploy, klik tab "contract". Kalau yang terlihat cuma bytecode, artinya **contract-nya belum terverifikasi**.

Hardhat memfasilitasi proses verifikasi dengan plugin **hardhat-etherscan**. Dapatkan Etherscan API key dari `https://etherscan.io/apis`, tambahkan ke file `.env`, lalu sesuaikan `hardhat.config.js`. Verifikasi dijalankan dengan:

```bash
npx hardhat verify --network rinkeby CONTRACT_ADDRESS "100000000000000000000000"
```

Kalau berhasil, akan muncul pesan **"Successfully verified contract Token on Etherscan"**, dan source code-nya jadi terlihat.

### Hands-on: merekonstruksi Parity wallet hack
Ini bagian paling berat sekaligus paling menarik dari deck ini.

**Latar belakang.** Parity hack adalah peretasan yang sangat besar dan penting di Ethereum. **Penyerang mencuri lebih dari 150.000 ETH.**

**Arsitektur yang diserang.** Multi-sig wallet itu bagus untuk menyimpan dana dalam jumlah besar dan/atau mengurangi risiko satu pihak, karena punya sekumpulan owner dan sebuah **threshold** — jumlah minimum tanda tangan yang dibutuhkan untuk mengeksekusi suatu transaksi.

Tanpa masuk terlalu dalam: ada sebuah **implementation contract** atau *"singleton"* yang memuat seluruh fungsionalitas wallet, dan sebuah **proxy factory** yang men-deploy proxy contract yang mendelegasikan semua pemanggilan ke implementation contract. Jadi saat kamu membuat wallet baru, wallet itu punya alamat dan storage yang unik, tapi **mendelegasikan semua call ke implementation contract**.

**Apa yang salah.** Untuk arsitektur seperti ini, kamu harus memastikan dua hal:
1. **Semua fungsi yang mengubah state harus terproteksi** — cuma kelompok tertentu yang boleh memanggilnya.
2. **Fungsi awal yang men-setup contract cuma boleh dipanggil sekali.**

Kalau kita lihat kodenya, fungsi **`initWallet` di Parity terbuka untuk dipanggil siapa saja**. Artinya kamu bisa memanggil fungsi itu langsung, **menambahkan alamatmu sendiri sebagai owner**, dan mengambil alih wallet-nya.

Setelah menemukan kerentanan itu, yang langsung dilakukan hacker adalah **mencari wallet dengan jumlah ETH terbesar**. Contoh yang dipakai slide: kembali ke **block 4043802** dan mengambil **82.189 ETH** dari sebuah wallet. Transaksi aslinya bisa dilihat di:
`https://etherscan.io/tx/0xeef10fc5170f669b86c4cd0444882a96087221325f8bf2f55d6188633aa7be7c`

> [!info] Konteks tambahan (bukan dari slide)
> Analogi bugnya: bayangin brankas apartemen yang tombol **"daftarkan pemilik baru"**-nya ada di luar, di lorong, tanpa kunci. Selama gak ada yang mencet, semua aman. Begitu satu orang sadar tombol itu ada dan bisa dipencet siapa saja — dia daftarkan namanya sebagai pemilik, lalu buka brankasnya secara sah. Kodenya jalan **persis seperti yang ditulis**; yang salah adalah yang ditulis. Ini persis peringatan di [[W08 - Smart Contract Pitfalls]]: *"a smart contract will execute exactly what is written, which is not always what the programmer intended."*

**Tugasnya:** pakai Hardhat untuk **merekonstruksi hack tersebut**. Caranya dengan **fork mainnet Ethereum** dan berinteraksi dengan wallet dan alamat di sana. Fitur Hardhat yang dipakai:

- **Impersonate accounts** — fitur ini memungkinkan kamu bertindak seolah-olah kamu pemilik akun tertentu. Untuk contoh ini, kita bertindak seolah-olah kita adalah hacker-nya.
- **Pinning a block** — Hardhat mengizinkanmu menentukan nomor block tertentu. Artinya state chain akan berperilaku seolah-olah kita sedang berada di block tersebut.

> [!warning] Slide 28 mencatat: untuk bisa **pin a block**, kamu butuh akses ke **archival node**. Alchemy menyediakan ini. Jadi Infura saja (yang dipakai untuk deploy di bagian sebelumnya) kemungkinan **tidak cukup** untuk hands-on ini.

## Diagram & Visual
- **Slide 2 — logo/identitas Hardhat**
  ![[99-Assets/Blockchain/W07-slide02.png]]
- **Slide 5 — diagram struktur direktori proyek Hardhat**
  ![[99-Assets/Blockchain/W07-slide05.png]]
- **Slide 8 — output terminal `npx hardhat` (menu pilihan setup)**
  ![[99-Assets/Blockchain/W07-slide08.jpg]]
- **Slide 10 — isi `hardhat.config.js` awal (require plugin + versi compiler 0.8.8)**
  ![[99-Assets/Blockchain/W07-slide10.jpg]]
- **Slide 11 — struktur direktori setelah `Token.sol` dibuat**
  ![[99-Assets/Blockchain/W07-slide11.jpg]]
- **Slide 12 — struktur direktori setelah folder `test` dan `token.js` dibuat**
  ![[99-Assets/Blockchain/W07-slide12.jpg]]
- **Slide 13 — output `npx hardhat test` dengan semua test case lolos**
  ![[99-Assets/Blockchain/W07-slide13.jpg]]
- **Slide 14 — isi file `.env`**
  ![[99-Assets/Blockchain/W07-slide14.png]]
- **Slide 15 — `hardhat.config.js` setelah ditambah bagian `networks`**
  ![[99-Assets/Blockchain/W07-slide15.png]]
- **Slide 18 — proses `npx hardhat compile`**
  ![[99-Assets/Blockchain/W07-slide18.jpg]]
- **Slide 21 — tampilan Etherscan menunjukkan contract yang belum terverifikasi (cuma bytecode)**
  ![[99-Assets/Blockchain/W07-slide21.jpg]]
- **Slide 22 — `hardhat.config.js` dengan konfigurasi hardhat-etherscan**
  ![[99-Assets/Blockchain/W07-slide22.png]]
- **Slide 23 — tampilan Etherscan setelah source code terverifikasi**
  ![[99-Assets/Blockchain/W07-slide23.png]]

> [!warning] Deck ini **paling banyak gambarnya** (13 gambar non-dekoratif dari 29 slide), dan hampir semuanya screenshot terminal, isi file config, atau struktur direktori. Isi file `hardhat.config.js` dan `.env` **gak bisa di-copy dari note ini** — harus dibaca dari gambar. Slide 26 (kode `initWallet` yang tidak terproteksi) juga tidak terekstrak sebagai gambar terpisah.

## Rumus / Sintaks

Setup awal:
```bash
mkdir hardhat-tutorial && cd hardhat-tutorial
mkdir project1 && cd project1
npm init -y
npm install --save-dev hardhat
npx hardhat
npm install --save-dev @nomiclabs/hardhat-ethers ethers @nomiclabs/hardhat-waffle ethereum-waffle chai
npm install dotenv
```

Perintah harian:
```bash
npx hardhat test                                          # jalankan test
npx hardhat compile                                       # Solidity -> bytecode + ABI
npx hardhat run deployments/deployToken.js --network rinkeby
npx hardhat verify --network rinkeby CONTRACT_ADDRESS "100000000000000000000000"
```

Lokasi ABI dan bytecode hasil compile:
```
artifacts/contracts/Token.sol/Token.json
```

## Hands-on
1. **Proyek Hardhat pertama** — dari instalasi sampai contract terverifikasi di Etherscan (slide 7–23). Kode: `Token.sol` (`https://pastebin.com/BZN2bdS0`), `token.js` (`https://pastebin.com/Um0rm0Kf`), `deployToken.js` (`https://pastebin.com/2kDyTDm3`).
2. **Rekonstruksi Parity wallet hack** (slide 24–28) — fork mainnet, impersonate account hacker, pin ke block 4043802, ambil 82.189 ETH dari wallet target. Butuh archival node (Alchemy).

## Pertanyaan Terbuka
- **Rinkeby, Ropsten, dan Infura setup di slide ini sudah usang.** Rinkeby dan Ropsten sudah dimatikan sejak 2022–2023; testnet yang aktif sekarang Sepolia dan Holesky. Perlu ditanya ke dosen testnet mana yang dipakai di praktikum, karena semua perintah `--network rinkeby` di slide gak akan jalan.
- Compiler yang ditentukan slide adalah **0.8.8**, sementara [[W05 - Solidity Development]] menyebut `SELFDESTRUCT` deprecated sejak **0.8.18**. Perlu dipastikan versi mana yang dipakai di kelas.
- Slide 24 nyebut arsitektur proxy dan **`delegatecall`** sebagai inti Parity hack, tapi `delegatecall` gak pernah dijelaskan mekanismenya, baik di sini maupun di [[W05 - Solidity Development]]. Ini lubang materi yang cukup besar untuk memahami hack-nya.
- `deployments/` sebagai nama folder itu konvensi slide ini; dokumentasi Hardhat resmi biasanya pakai `scripts/`. Perlu dipastikan mana yang dinilai.
- Hands-on Parity butuh akun Alchemy berbayar/terdaftar. Perlu ditanya apakah disediakan kampus.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Blockchain]]
- [[W06 - Smart Contract Standard]]
- [[W08 - Smart Contract Pitfalls]]
- [[W05 - Solidity Development]]
- [[Blockchain - Review dan Glosari]]
