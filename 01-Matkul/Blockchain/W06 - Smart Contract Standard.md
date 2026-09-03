---
matkul: Blockchain
minggu: 6
sks: 2
sumber: Smart Contract Standard.pptx
tags: [kuliah/blockchain, minggu/w06]
status: draft
diproses: 2026-09-03
---

# W06 — Smart Contract Standard

## Ringkasan
> - Standar di Ethereum namanya **ERC** (Ethereum Request for Comment). Sekarang ada lebih dari 50 ERC beredar.
> - **ERC-20** = token **fungible** (tiap token identik dan bisa ditukar). Wajib punya 6 fungsi: `totalSupply`, `balanceOf`, `transfer`, `transferFrom`, `approve`, `allowance`.
> - **ERC-721** = token **non-fungible** (NFT), tiap token unik dan gak bisa ditukar. Buat collectible, item game, seni digital, real estate.
> - **ERC-1155** = **multi token**, bisa bikin token fungible dan non-fungible dalam satu contract, plus **batch transfer** yang hemat gas. Bisa bikin token **semi-fungible**.
> - **ERC-4626** = standar **tokenized vault** untuk vault yang menghasilkan yield.
> - Kenapa harus tau standar: (1) biar gak bingung fungsi warisan dari mana, (2) **deviasi dari standar itu sendiri bisa jadi audit finding**.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| ERC | Ethereum Request for Comment, wadah memperkenalkan informasi teknis ke developer dan user |
| Fungible | Tiap unit identik nilainya dan bisa saling ditukar (seperti uang) |
| Non-fungible | Tiap unit unik dan gak bisa saling ditukar (seperti tiket bernomor kursi) |
| Semi-fungible | Punya sebagian sifat fungible dan sebagian non-fungible |
| ERC-20 | Standar token fungible |
| ERC-721 | Standar token non-fungible (NFT) |
| ERC-1155 | Standar multi token, fungible + non-fungible dalam satu contract |
| ERC-4626 | Standar tokenized vault untuk vault yang menghasilkan yield |
| Batch transfer | Mentransfer banyak token dalam satu transaksi |
| Overridden function | Fungsi standar yang implementasinya ditimpa; wajib dicurigai saat audit |

## Isi

### Apa itu ERC
Kalau bicara standar untuk Ethereum, yang dibicarakan adalah **ERC**. ERC singkatan dari **Ethereum Request for Comment**, dan berfungsi sebagai wadah untuk memperkenalkan informasi teknis kepada developer dan user.

Standar-standar ini memberi panduan untuk pengembangan smart contract dan dApps yang bisa dipakai untuk **membuat, mengelola, dan menukar token** di jaringan Ethereum. Saat ini ada **lebih dari 50 ERC** beredar.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi: ERC itu kayak **standar colokan listrik**. Gak ada polisi yang maksa pabrik bikin colokan tipe C, tapi kalau kamu bikin colokan bentuk sendiri, gak ada stopkontak di dunia yang mau nerima kamu. Sama: contract token yang gak ikut ERC-20 gak akan dikenali MetaMask, Uniswap, atau exchange mana pun. Standar itu bukan aturan hukum, tapi **tiket masuk ke ekosistem**.

### ERC-20: fungible token standard
**ERC-20** adalah standar teknis untuk pembuatan token di blockchain Ethereum. Token ERC-20 bersifat **fungible**, artinya tiap token **identik nilainya** dengan token lain dari jenis yang sama dan bisa dipakai secara bergantian.

Untuk disebut ERC-20 compliant, sebuah token wajib punya atribut ini yang di-hardcode di dalam kodenya:

| Fungsi | Kegunaannya |
| --- | --- |
| `totalSupply` | Memberikan informasi total suplai token |
| `balanceOf` | Menunjukkan saldo akun pemilik token |
| `transfer` | Mengeksekusi transfer token ke alamat tertentu |
| `transferFrom` | Mengizinkan transfer **dari** alamat tertentu |
| `approve` | Mengizinkan *spender* menarik token |
| `allowance` | Mengembalikan token dari spender ke owner |

**Contoh alur dari slide** — misalnya kita membuat token ERC-20 baru bernama "ABC Token" dengan total suplai 1.000.000 token, tiap token bisa dibagi sampai 18 angka desimal:

1. Alice punya 500.000 ABC Token.
2. Bob mau beli 100 ABC Token dari Alice.
3. **Alice `approve` Bob** untuk membelanjakan sampai 100 ABC Token atas namanya.
4. Bob memanggil fungsi **`transferFrom()`** dengan alamat Alice, alamatnya sendiri, dan nilai 100 token.
5. Fungsi `transferFrom()` **mengurangi 100 token dari saldo Alice dan menambahkannya ke saldo Bob**.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa harus ada dua langkah (`approve` lalu `transferFrom`), padahal ada `transfer`? Karena `transfer` itu **kamu yang dorong**, sementara `approve`+`transferFrom` itu **kamu kasih izin, orang lain yang tarik**. Analoginya: `transfer` = kamu transfer manual ke tukang sayur. `approve` = kamu daftarkan autodebit dengan plafon Rp100 ribu, lalu tukang sayurnya yang menarik sendiri. Pola kedua itulah yang bikin smart contract lain (misal DEX) bisa memindahkan tokenmu tanpa kamu harus hadir di tiap langkah.

### ERC-721: non-fungible token standard
**ERC-721** adalah standar token Ethereum yang mendefinisikan contract **non-fungible token (NFT)**. Tidak seperti token fungible macam Ether atau token ERC-20, **tiap token ERC-721 itu unik dan gak bisa saling ditukar**.

Ini bikin dia cocok untuk merepresentasikan aset seperti **collectible, item game, seni digital, dan real estate**.

Standar ERC-721 mendefinisikan sekumpulan fungsi yang memungkinkan pembuatan, kepemilikan, dan transfer NFT. Tiap NFT diwakili oleh **identifier unik**, dan bisa punya **metadata** terkait seperti nama, deskripsi, dan gambar. Token ERC-721 bisa diperdagangkan di berbagai marketplace, dan kepemilikannya disimpan di blockchain Ethereum.

> [!info] Konteks tambahan (bukan dari slide)
> Fungible vs non-fungible paling gampang dibedain lewat pertanyaan: **"kalau ditukar dengan yang lain, kamu rugi gak?"** Uang Rp50.000-mu ditukar sama Rp50.000 orang lain — gak masalah, itu **fungible**. Tiket konser kursi A1 ditukar sama kursi Z99 — jelas masalah, itu **non-fungible**.

### ERC-1155: multi token standard
**ERC-1155** adalah standar token yang diusulkan **Enjin tahun 2018**. Keunikannya: dia memungkinkan pembuatan **token fungible dan non-fungible sekaligus dalam satu contract yang sama**, yang memberi keuntungan besar dari sisi efisiensi dan fleksibilitas.

Keunggulan yang disebut slide:

**1. Mengurangi jumlah transaksi.** Secara tradisional, bikin dApp yang mengelola beberapa jenis token butuh **contract terpisah untuk tiap jenis**, yang merepotkan dan gak efisien. Dengan ERC-1155, developer bisa bikin **satu smart contract** yang mengelola token fungible dan non-fungible sekaligus.

**2. Batch transfers.** ERC-1155 mendukung transfer banyak token dalam **satu transaksi**. Ini bisa mengurangi biaya gas dan memperbaiki efisiensi keseluruhan saat mentransfer token dalam jumlah besar.

**3. Token semi-fungible.** Token yang punya sebagian sifat fungible dan sebagian non-fungible. Contoh dari slide: sebuah item game mungkin punya sekumpulan properti yang unik untuk item itu (bikin dia **non-fungible**), tapi juga punya sekumpulan properti yang dibagi bersama banyak item lain (bikin dia **fungible**).

> [!info] Konteks tambahan (bukan dari slide)
> Contoh semi-fungible yang gampang: **potion di game**. Semua "Health Potion" itu sama persis dan bisa ditukar — fungible. Tapi "Pedang Legendaris #3 milik Reyvan, level 40, sudah dipakai bunuh 200 monster" itu unik — non-fungible. Sebelum ERC-1155, satu game harus deploy dua contract terpisah buat dua hal ini. Sekarang cukup satu.

### ERC-4626: tokenized vault standard
**ERC-4626** adalah standar yang dirancang untuk merapikan dan menstandarkan parameter teknis dari **vault yang menghasilkan yield** (*yield-bearing vaults*). Dia memperkenalkan **API standar** untuk tokenized yield-bearing vault yang merepresentasikan *share* dari satu token ERC-20 yang mendasarinya.

ERC-4626 juga menyertakan **ekstensi opsional** untuk tokenized vault yang memakai ERC-20, yang menyediakan fungsionalitas dasar untuk **deposit, withdraw token, dan cek saldo**.

Manfaatnya: developer mendapat pola implementasi yang lebih konsisten dan robust, mengurangi usaha integrasi, dan membuka akses ke yield di berbagai aplikasi dengan sedikit usaha khusus.

### Kenapa harus tau standar
Slide kasih dua alasan, dan yang kedua ini yang penting buat matkul ini.

**1. Memperluas wawasan.** Saat mengimplementasikan sebuah contract, developer akan merujuk ke standar contract dan mengimplementasikan fungsi-fungsinya sesuai yang didefinisikan standar. **Kalau kamu gak familiar dengan fungsi dan event standar yang diwarisi contract-contract ini, kamu bakal habis berjam-jam cuma buat nebak fungsi itu ngapain dan datangnya dari mana.**

**2. Deviasi dari standar bisa berarti audit finding.**
- Beri perhatian khusus pada fungsi apa pun yang **di-override** (ditimpa) di dalam contract.
- Verifikasi bahwa fungsi yang di-override memenuhi kebutuhan keamanan dan **tidak memperkenalkan kerentanan baru**.
- Sebagian implementasi mungkin **tidak sesuai standar ERC**, dan karenanya bisa menghasilkan perilaku yang tidak terduga — atau bahkan bisa dieksploitasi.

### Studi kasus: Notional token audit (2021)
Slide menampilkan temuan audit nyata yang berhubungan dengan standar:

> **`CompoundToNotionalV2.notionalCallback` — nilai kembalian (return values) ERC20 tidak dicek.**

Sumbernya: `https://code4rena.com/reports/2021-08-notional#h-03-compoundtonotionalv2notionalcallback-erc20-return-values-not-checked`

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa ini jadi temuan **High**: fungsi `transfer` di ERC-20 mengembalikan `bool`. Sebagian token mengembalikan `false` ketika transfer gagal, alih-alih membatalkan transaksi. Kalau kodemu gak mengecek nilai kembaliannya, kodemu akan lanjut jalan **seolah-olah transfer berhasil**, padahal uangnya gak pindah. Ini sambung langsung ke **unchecked external calls** di [[W08 - Smart Contract Pitfalls]].

### Testimoni
Slide 13 menampilkan testimoni dari **Richard Tan, angkatan B25, jurusan cybersecurity**, yang terlibat di beberapa proyek blockchain auditing sejak 2023, dari masa-masa awalnya.

## Diagram & Visual
- **Slide 4 — ilustrasi cara kerja token ERC-20**
  ![[99-Assets/Blockchain/W06-slide04.jpg]]
- **Slide 10 — tabel rangkuman perbandingan standar token Ethereum**
  ![[99-Assets/Blockchain/W06-slide10.jpg]]
- **Slide 12 — cuplikan laporan audit Notional (code4rena) terkait return value ERC-20**
  ![[99-Assets/Blockchain/W06-slide12.png]]
- **Slide 13 — testimoni Richard Tan soal pengalaman auditing**
  ![[99-Assets/Blockchain/W06-slide13.png]]

> [!warning] Slide 4, 10, 12, dan 13 di PPT aslinya cuma judul + gambar. Terutama **slide 10 (tabel rangkuman perbandingan standar)** — itu kemungkinan besar tabel yang paling berguna buat belajar dan isinya gak bisa diekstrak jadi teks. Buka gambarnya.

## Hands-on
Slide 14 — implementasikan token yang ERC-20 compliant, panduan di:
`https://www.quicknode.com/guides/ethereum-development/smart-contracts/how-to-create-and-deploy-an-erc20-token`

## Pertanyaan Terbuka
- Deskripsi `allowance` di slide agak janggal: ditulis "returns tokens from a spender to the owner", padahal fungsi `allowance` sebenarnya **mengembalikan sisa jatah** yang masih boleh dibelanjakan spender, bukan mengembalikan token. Perlu dikonfirmasi apakah ini salah ketik di slide atau memang definisi yang dipakai dosen.
- Dari 50+ ERC yang disebut ada, cuma 4 yang dibahas. Perlu ditanya apakah ada standar lain yang masuk ujian (ERC-165, ERC-2612 permit, dll).
- ERC-721 dan ERC-1155 **gak dikasih daftar fungsi wajibnya** seperti ERC-20. Kalau soal ujian minta menyebutkan fungsi ERC-721, materinya gak ada di slide.
- Slide 10 (tabel rangkuman) berupa gambar — isi tabelnya gak bisa diverifikasi dari teks ekstraksi.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Blockchain]]
- [[W05 - Solidity Development]]
- [[W07 - The Hardhat Framework]]
- [[W08 - Smart Contract Pitfalls]]
- [[Blockchain - Review dan Glosari]]
