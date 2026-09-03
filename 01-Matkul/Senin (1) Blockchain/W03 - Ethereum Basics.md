---
matkul: Blockchain
minggu: 3
sks: 2
sumber: Ethereum Basics.pptx
tags: [kuliah/blockchain, minggu/w03]
status: draft
diproses: 2026-09-03
---

# W03 — Ethereum Basics

## Ringkasan
> - **Ethereum itu sistemnya, ether itu mata uangnya.** Satuan terkecil namanya **wei**; 1 ether = 10^18 wei. Nilai selalu disimpan internal sebagai unsigned integer dalam wei.
> - Wallet = gerbangmu ke Ethereum. Karena aplikasi wallet **harus punya akses ke private key-mu**, cuma pakai wallet dari sumber yang kamu percaya. Contoh: MetaMask, Jaxx, MyEtherWallet, Emerald.
> - **Kehilangan private key = kehilangan dana selamanya.** Gak ada yang bisa nolong. Jangan simpan private key dalam bentuk polos, dan backup mnemonic pakai **pena dan kertas**.
> - Dua jenis akun: **EOA** (punya private key, dikendalikan manusia) dan **contract account** (punya kode, gak punya private key, dikendalikan logika kodenya sendiri).
> - Smart contract pertama: **Faucet.sol** di Remix IDE — deklarasi contract, terima ether masuk, fungsi `withdraw` public, transfer ke `msg.sender`.
> - **`receive()` dan `fallback()`**: dipanggil kalau signature fungsi yang dipanggil gak cocok dengan fungsi mana pun. `receive()` kalau calldata **kosong**, `fallback()` kalau calldata **tidak kosong**. Modifier **`payable`** wajib supaya fungsi bisa menerima ether.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| ether / ETH / Ξ / ♦ | Mata uang Ethereum |
| wei | Satuan terkecil ether; 1 ether = 10^18 wei |
| Wallet | Aplikasi pengelola akun Ethereum; gerbang ke sistem, menyimpan kunci dan menyiarkan transaksi |
| MetaMask | Wallet ekstensi browser, mudah dipakai untuk testing, bisa konek ke berbagai node dan test blockchain |
| Jaxx | Wallet multiplatform dan multicurrency, dirancang untuk kesederhanaan |
| MyEtherWallet (MEW) | Wallet berbasis web yang jalan di browser mana pun |
| Emerald Wallet | Wallet desktop open source untuk Ethereum Classic dan chain berbasis Ethereum |
| Air-gapped device | Perangkat yang terputus total dari jaringan; tingkat keamanan tertinggi |
| Faucet | Layanan/contract yang membagikan test ether ke alamat mana pun yang meminta |
| EOA | Externally Owned Account; **punya private key** |
| Contract account | Akun berisi kode smart contract; **tidak punya private key**, dikendalikan logika kodenya |
| EVM | Ethereum Virtual Machine; global singleton, tiap node menjalankan salinan lokalnya |
| Remix | IDE berbasis browser untuk menulis, compile, dan deploy Solidity |
| Etherscan | Block explorer untuk melihat riwayat transaksi sebuah alamat |
| `receive()` | Dipanggil saat tidak ada signature fungsi yang cocok **dan calldata kosong** |
| `fallback()` | Dipanggil saat tidak ada signature fungsi yang cocok **dan calldata tidak kosong** |
| `payable` | Modifier yang mengizinkan fungsi menerima ether; tanpa ini transaksi ditolak |
| calldata / `msg.data` | Data yang menyertai pemanggilan fungsi |

## Isi

### Satuan mata uang
Satuan mata uang Ethereum disebut **ether**, diidentifikasi juga sebagai **"ETH"** atau dengan simbol **Ξ** atau **♦**.

Ether dibagi jadi satuan yang lebih kecil, sampai satuan terkecil yang mungkin, yang dinamai **wei**. Satu ether adalah **1 kuintiliun wei** (10^18, atau 1.000.000.000.000.000.000).

Slide menegaskan satu hal yang gampang ketuker:

> **Ethereum itu sistemnya, ether itu mata uangnya.**

Nilai ether **selalu direpresentasikan secara internal di Ethereum sebagai nilai unsigned integer dalam satuan wei**. Waktu kamu bertransaksi 1 ether, transaksinya meng-encode `1000000000000000000` wei sebagai nilainya.

Tiap denominasi ether punya **nama ilmiah** (memakai Sistem Satuan Internasional / SI) dan **nama sehari-hari** yang memberi penghormatan ke tokoh-tokoh besar komputasi dan kriptografi.

| Nilai (dalam wei) | Eksponen | Nama umum | Nama SI |
| --- | --- | --- | --- |
| 1 | 10^0 | **wei** | Wei |
| 1.000 | 10^3 | **Babbage** | Kilowei atau femtoether |
| 1.000.000 | 10^6 | **Lovelace** | Megawei atau picoether |
| 1.000.000.000 | 10^9 | **Shannon** | Gigawei atau nanoether |
| 1.000.000.000.000 | 10^12 | **Szabo** | Microether atau micro |
| 1.000.000.000.000.000 | 10^15 | **Finney** | Milliether atau milli |
| 1.000.000.000.000.000.000 | 10^18 | **Ether** | Ether |
| 1.000.000.000.000.000.000.000 | 10^21 | **Grand** | Kiloether |
| 1.000.000.000.000.000.000.000.000 | 10^24 | — | Megaether |

> [!warning] Eksponen di slide aslinya ditulis dengan superscript (10³, 10⁶, dst) yang hilang saat diekstrak, jadi di PPT terlihat seperti "103" dan "106". Angka di tabel ini sudah dikembalikan ke bentuk yang benar berdasarkan kolom nilainya. Baris pertama di slide tertulis eksponen "1", yang seharusnya 10^0. Baris Megaether tidak punya nama umum di slide.

> [!info] Konteks tambahan (bukan dari slide)
> Yang paling sering kepakai sehari-hari cuma tiga: **wei** (satuan internal), **gwei** alias Shannon (satuan gas price — lihat [[W04 - Wallets and Transactions]]), dan **ether** itu sendiri. Sisanya (Babbage, Lovelace, Szabo, Finney) hampir gak pernah dipakai di praktik, tapi bisa keluar di soal hafalan karena nama-namanya khas: Babbage, Lovelace, Shannon, Szabo, Finney — semuanya tokoh komputasi dan kriptografi.

### Dasar-dasar wallet
Wallet dalam teknologi blockchain berarti **aplikasi software yang membantu kamu mengelola akun Ethereum-mu**. Singkatnya, **wallet Ethereum adalah gerbangmu ke sistem Ethereum**. Dia menyimpan kunci-kuncimu, dan bisa membuat serta menyiarkan transaksi atas namamu.

Slide menandai ini sebagai **penting**:

> Supaya sebuah aplikasi wallet bisa bekerja, dia **harus punya akses ke private key-mu**. Karena itu sangat vital kamu hanya mengunduh dan memakai aplikasi wallet **dari sumber yang kamu percaya**.

### Empat contoh wallet
| Wallet | Bentuk | Karakteristiknya |
| --- | --- | --- |
| **MetaMask** | Ekstensi browser (Chrome, Firefox, Opera, Brave) + aplikasi mobile iOS dan Android | Mudah dipakai dan praktis untuk testing, karena bisa terhubung ke berbagai node Ethereum dan test blockchain |
| **Jaxx** | Multiplatform (Android, iOS, Windows, macOS, Linux), mobile atau desktop tergantung di mana diinstal | Multicurrency; sering jadi pilihan bagus untuk user baru karena dirancang sederhana dan mudah dipakai |
| **MyEtherWallet (MEW)** | Terutama berbasis web, jalan di browser mana pun; tersedia juga di Android dan iOS | Punya banyak fitur canggih yang dipakai di berbagai contoh |
| **Emerald Wallet** | Aplikasi desktop open source (Windows, macOS, Linux) | Dirancang untuk blockchain **Ethereum Classic**, tapi kompatibel dengan blockchain berbasis Ethereum lain. Bisa menjalankan full node atau terhubung ke public remote node dalam mode "light". Punya companion tool untuk semua operasi lewat command line |

### Kendali dan tanggung jawab atas wallet
Blockchain terbuka seperti Ethereum itu penting karena beroperasi sebagai **sistem terdesentralisasi**. Itu berarti banyak hal, tapi satu aspek yang krusial: **tiap user Ethereum bisa — dan seharusnya — mengendalikan private key-nya sendiri**, karena kunci itulah yang mengendalikan akses ke dana dan smart contract.

Konsekuensinya ditulis tanpa basa-basi di slide:

> **Kalau kamu kehilangan private key-mu, kamu kehilangan akses ke dana dan contract-mu. Tidak ada seorang pun yang bisa membantumu memulihkan akses — danamu terkunci selamanya.**

### Cara melindungi wallet
Slide kasih enam aturan, dan dua di antaranya diulang dengan penekanan yang sama.

**Prinsip umum:**
- **Jangan berimprovisasi soal keamanan.** Pakai pendekatan standar yang sudah teruji.
- **Makin penting akunnya** (makin besar nilai dana yang dikendalikan, atau makin signifikan smart contract yang bisa diakses), **makin tinggi pula tindakan keamanan yang harus diambil**.
- Keamanan tertinggi didapat dari **air-gapped device**, tapi level ini **tidak dibutuhkan untuk setiap akun**.

**Aturan praktis:**
- **Jangan pernah menyimpan private key dalam bentuk polos**, apalagi secara digital. Untungnya, sebagian besar user interface sekarang bahkan gak mengizinkanmu melihat private key mentahnya.
- **Jangan simpan password apa pun di dokumen digital**, foto digital, screenshot, online drive, PDF terenkripsi, dan sejenisnya. Sekali lagi: **jangan berimprovisasi soal keamanan.** Pakai password manager, atau pena dan kertas.
- Saat kamu diminta mem-backup kunci sebagai **mnemonic word sequence**, pakai **pena dan kertas** untuk membuat backup fisik. **Jangan tunda tugas itu "buat nanti" — kamu pasti lupa.**

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan bahwa "jangan berimprovisasi" disebut **dua kali** di slide yang sama. Itu bukan pengulangan yang gak sengaja — itu kesalahan paling umum orang pintar: bikin skema penyimpanan kunci sendiri yang "lebih pintar" (dipotong jadi tiga, disimpan di tiga email, dienkripsi pakai sandi buatan sendiri). Skema begini gagal bukan karena diserang, tapi karena **kamu sendiri yang lupa cara ngerakitnya lagi**. Detail mnemonic dan seed-nya dibahas di [[W04 - Wallets and Transactions]].

### Ethereum sebagai The World Computer
Sampai titik ini, Ethereum diperlakukan sebagai cryptocurrency. **Padahal fungsi cryptocurrency itu justru cuma fungsi bawahan dari fungsi Ethereum yang sebenarnya.** Ether dimaksudkan untuk **membayar jalannya smart contract**, yaitu program komputer yang berjalan di komputer teremulasi bernama **EVM (Ethereum Virtual Machine)**.

**EVM adalah global singleton**, artinya dia beroperasi seolah-olah dia satu komputer global dengan instance tunggal, yang berjalan di mana-mana. **Tiap node di jaringan Ethereum menjalankan salinan lokal EVM** untuk memvalidasi eksekusi contract, sementara blockchain Ethereum mencatat perubahan state world computer ini saat dia memproses transaksi dan smart contract.

### EOA vs Contract Account
Ini pembedaan yang dipakai terus-menerus di seluruh matkul.

| | **Externally Owned Account (EOA)** | **Contract Account** |
| --- | --- | --- |
| Punya private key? | **Ya** | **Tidak** |
| Berisi apa? | — | **Kode smart contract** |
| Dikendalikan oleh | Pemegang private key | **Logika kode smart contract-nya sendiri** |
| Contoh | Akun yang kamu buat di MetaMask | Contract yang di-deploy ke blockchain |

Punya private key berarti punya kendali atas akses ke dana atau contract. Sementara contract **dimiliki dan dikendalikan oleh logika kodenya** — program yang tercatat di blockchain Ethereum saat contract account itu dibuat, dan dieksekusi oleh EVM.

> [!info] Konteks tambahan (bukan dari slide)
> Konsekuensi yang gak ditulis eksplisit di slide tapi menentukan seluruh materi setelah ini: karena contract account **gak punya private key**, dia **gak bisa memulai transaksi sendiri**. Dia cuma bereaksi. Itulah kenapa di [[W05 - Solidity Development]] disebut "contract gak jalan sendiri, semuanya dimulai dari transaksi EOA".

### Menulis smart contract pertama

**Lingkungannya.** Untuk menjalankan contract pertamamu, kamu butuh **bahasa pemrograman dan compiler**. Ethereum punya banyak bahasa tingkat tinggi yang semuanya bisa dipakai menulis contract dan menghasilkan **EVM bytecode**. Salah satu bahasa paling populer adalah **Solidity**, dan salah satu IDE paling populer adalah **Remix** (`https://remix-project.org/`).

**Contract-nya.** Yang ditulis adalah contract yang mengendalikan sebuah **faucet**. Kamu sudah pernah memakai faucet untuk mendapat test ether di jaringan test Ropsten. Faucet itu hal yang relatif sederhana: **dia membagikan ether ke alamat mana pun yang meminta, dan bisa diisi ulang secara berkala**. Faucet bisa diimplementasikan sebagai wallet yang dikendalikan manusia, atau sebagai web server.

Empat hal yang dilakukan `Faucet.sol` (slide 16):
1. **Mendeklarasikan objek contract**
2. **Membuat contract bisa menerima jumlah berapa pun yang masuk**
3. **Mendeklarasikan fungsi bernama `withdraw` sebagai public**, artinya bisa dipanggil contract lain
4. **Mentransfer ether dari contract saat ini ke alamat pengirim**

**Compile dan deploy** (slide 17), langkah demi langkah di Remix:
1. Buka Remix IDE
2. Buat file baru dan paste `Faucet.sol` yang sudah dibuat
3. Klik tab **"Solidity Compiler"** di sidebar kiri
4. Pilih versi compiler yang sesuai (samakan dengan versi Solidity-mu, misalnya **0.6.4**)
5. Klik tombol **"Compile Faucet.sol"**
6. Klik tab **"Run"** di sidebar kiri, lalu klik **"Deploy"**

**Eksekusinya.** Begitu smart contract dibuat di blockchain, dia punya **alamat Ethereum, sama seperti wallet**. Kapan pun seseorang mengirim transaksi ke alamat contract, itu **menyebabkan contract berjalan di EVM**, dengan transaksi tersebut sebagai input-nya.

Transaksi yang dikirim ke alamat contract boleh membawa **ether**, **data**, atau **keduanya**:
- Kalau berisi **ether**, ether itu **"didepositkan"** ke saldo contract.
- Kalau berisi **data**, data itu bisa **menyebut nama fungsi** di dalam contract dan **memanggilnya**, sambil melewatkan argumen ke fungsi itu.

> [!info] Konteks tambahan (bukan dari slide)
> Empat kombinasi value/data yang dibahas lebih rinci di [[W04 - Wallets and Transactions]] itu penjelasan lanjutan dari paragraf ini. Di sini kamu lihat efeknya dari sisi contract; di W04 kamu lihat bentuknya dari sisi struktur transaksi.

### `receive()`, `fallback()`, dan modifier `payable`
Ini bagian yang paling penting dari deck ini untuk materi keamanan nanti.

**Fungsi `receive()` dan `fallback()` dipanggil ketika signature fungsi yang dipanggil tidak cocok dengan fungsi mana pun di dalam contract.** Bedanya ada di calldata:

| Fungsi | Kapan dieksekusi |
| --- | --- |
| **`receive()`** | Tidak ada signature fungsi yang cocok **dan calldata kosong** |
| **`fallback()`** | Tidak ada signature fungsi yang cocok **dan ether dikirim langsung ke contract, tapi `msg.data` (calldata) tidak kosong** |

Kedua fungsi ini memastikan contract menangani transfer ether dan pemanggilan fungsi **secara anggun (gracefully)**.

**Modifier `payable`** dipakai untuk mengizinkan sebuah fungsi menerima ether. **Tanpa modifier ini, kalau kamu mencoba mengirim ether ke sebuah fungsi, transaksinya akan ditolak dan gagal.**

> [!info] Konteks tambahan (bukan dari slide)
> Ini yang dirujuk [[W05 - Solidity Development]] waktu dia bilang "see previous session" soal fallback dan receive — jawabannya ada di sini.
>
> Dan ini juga **jantung dari reentrancy attack** di [[W08 - Smart Contract Pitfalls]]. Rantai logikanya: contract korban mengirim ether ke alamat penyerang → karena itu cuma kiriman ether biasa tanpa nama fungsi, yang terpanggil adalah **`receive()`/`fallback()` milik penyerang** → dan penyerang menaruh kode jahat persis di situ, yang langsung memanggil balik contract korban sebelum saldo sempat dikurangi. Jadi fungsi yang tujuannya "menangani transfer ether secara anggun" itulah yang jadi pintu masuknya. Pahami bagian ini dulu sebelum masuk W08.

## Diagram & Visual
- **Slide 10 — langkah-langkah membuat wallet di MetaMask**
  ![[99-Assets/Blockchain/W03-slide10.png]]
- **Slide 11 — tampilan riwayat transaksi sebuah alamat di Etherscan**
  ![[99-Assets/Blockchain/W03-slide11.png]]
- **Slide 16 — kode `Faucet.sol` beserta anotasi keempat bagiannya**
  ![[99-Assets/Blockchain/W03-slide16.png]]
- **Slide 19 — langkah berinteraksi dengan contract yang sudah di-deploy**
  ![[99-Assets/Blockchain/W03-slide19.png]]
- **Slide 21 — contoh kode `receive()` dan `fallback()`**
  ![[99-Assets/Blockchain/W03-slide21.jpg]]

> [!warning] Slide 21 di PPT aslinya cuma judul + gambar kode, tanpa teks. Contoh kode `receive()`/`fallback()`-nya **gak bisa di-copy dari note ini**. Begitu juga `Faucet.sol` di slide 16 — kodenya berupa gambar, cuma anotasi empat barisnya yang berupa teks.

## Rumus / Sintaks

Konversi satuan:
```
1 ether = 10^18 wei = 1.000.000.000.000.000.000 wei
1 gwei  = 10^9 wei   (nama umum: Shannon)
```

Aturan pemilihan `receive()` vs `fallback()`:
```
tidak ada signature fungsi yang cocok
├── calldata KOSONG        -> receive()
└── calldata TIDAK kosong  -> fallback()

fungsi tanpa `payable` + dikirimi ether -> transaksi DITOLAK
```

## Hands-on
1. **Membuat wallet dengan MetaMask** (slide 10):
   - Download plugin MetaMask di Google Chrome (`https://chromewebstore.google.com/detail/metamask/nkbihfbeogaeaoehlefnkodbefgpgknn`)
   - Buat wallet baru di MetaMask
   - Berpindah network di MetaMask
   - Dapatkan test ether (pakai Faucet)
   - Referensi dari speaker notes dosen: `https://github.com/ethereumbook/ethereumbook/blob/develop/02intro.asciidoc`
2. **Menjelajahi riwayat transaksi** (slide 11) — lihat semua transaksi sebuah alamat memakai block explorer Etherscan (`https://etherscan.io/`)
3. **Menulis, compile, dan deploy `Faucet.sol`** (slide 14–17) di Remix IDE
4. **Berinteraksi dengan smart contract** (slide 19):
   - Lihat alamat contract di block explorer (etherscan.io)
   - Kirim ether ke contract memakai MetaMask
   - Tarik sebagian ether dari contract memakai Remix

## Pertanyaan Terbuka
- **Ropsten sudah dimatikan** (disebut di slide 15 sebagai test network tempat mengambil faucet), begitu juga Rinkeby yang dipakai di [[W07 - The Hardhat Framework]]. Testnet yang aktif sekarang Sepolia dan Holesky. Perlu ditanya testnet mana yang dipakai di praktikum.
- Versi compiler yang disarankan slide adalah **0.6.4**, sementara [[W07 - The Hardhat Framework]] memakai **0.8.8** dan [[W05 - Solidity Development]] menyebut perubahan di 0.8.18. Perlu dipastikan versi mana yang dipakai konsisten di kelas — perbedaan 0.6 ke 0.8 itu besar (misalnya over/underflow otomatis revert sejak 0.8.0).
- Definisi `fallback()` di slide agak rancu: ditulis "dijalankan saat tidak ada signature yang cocok **dan ether dikirim langsung** ke contract, tapi calldata tidak kosong". Pembeda sebenarnya cuma **kosong/tidaknya calldata**; keberadaan ether tidak menentukan mana yang terpanggil. Perlu dikonfirmasi ke dosen versi mana yang dipakai untuk menjawab soal.
- Deck ini menyebut wallet **Jaxx** dan **Emerald** yang sekarang sudah tidak aktif dikembangkan. Kalau soalnya "sebutkan contoh wallet", tetap jawab sesuai slide.
- `Faucet.sol` yang dijadikan contoh **tidak punya access control apa pun** pada fungsi `withdraw` — siapa pun bisa menarik. Di deck ini itu memang disengaja (namanya juga faucet), tapi bandingkan dengan pola bug yang dibahas di [[W08 - Smart Contract Pitfalls]] supaya gak kebawa jadi kebiasaan.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Blockchain]]
- [[W02 - The Fundamentals of Ethereum]]
- [[W04 - Wallets and Transactions]]
- [[W05 - Solidity Development]]
- [[W08 - Smart Contract Pitfalls]]
- [[Blockchain - Review dan Glosari]]
