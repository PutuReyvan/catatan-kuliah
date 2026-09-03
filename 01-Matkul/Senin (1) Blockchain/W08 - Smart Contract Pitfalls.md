---
matkul: Blockchain
minggu: 8
sks: 2
sumber: Smart Contract Pitfalls.pptx
tags: [kuliah/blockchain, minggu/w08]
status: draft
diproses: 2026-09-03
---

# W08 — Smart Contract Pitfalls

> [!note] Slide 2 nyatakan deck ini dibawakan dalam **dua sesi terpisah (S09 dan S10)**. Ini deck terpanjang di matkul ini (36 slide) dan isinya paling padat untuk ujian.

## Ringkasan
> - Premis dasarnya satu kalimat: **smart contract mengeksekusi persis apa yang ditulis, yang belum tentu sama dengan yang dimaksud programmer.** Semua contract itu publik, siapa pun bisa berinteraksi, dan **kerugian hampir selalu mustahil dipulihkan**.
> - **Defensive programming** punya 5 pilar: Minimalism/Simplicity, Code Reuse, Code Quality, Readability/Auditability, Test Coverage.
> - **Lima risiko keamanan utama** yang dibahas: Reentrancy, Arithmetic Overflow/Underflow, Insecure Randomness, Incorrect Access Control, Unchecked External Calls.
> - Tiap risiko punya pasangan mitigasinya, dan pasangan risiko↔mitigasi inilah yang paling mungkin keluar di ujian.
> - Ada 3 hands-on eksploitasi: **EtherStore** (reentrancy), **TimeLock** (overflow), **BadRandomContract** (insecure randomness).

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Defensive programming | Gaya pengembangan yang mengasumsikan segalanya bisa salah dan disalahgunakan |
| DRY | Don't Repeat Yourself — kalau sepotong kode berulang, jadikan fungsi atau library |
| Not Invented Here syndrome | Godaan membangun ulang dari nol komponen yang sudah ada dan teruji |
| Reentrancy | Contract eksternal memanggil balik fungsi contract korban sebelum eksekusi pertama selesai |
| Fallback function | Fungsi yang otomatis terpanggil saat contract menerima ether/call tak dikenal |
| Mutex | Variabel state yang mengunci contract selama eksekusi, mencegah panggilan reentrant |
| Overflow | Menambah angka melebihi batas atas tipe data, nilainya berputar balik ke bawah |
| Underflow | Mengurangi angka di bawah batas bawah tipe data, nilainya berputar ke atas |
| SafeMath | Library OpenZeppelin pengganti operator matematika standar, aman dari over/underflow |
| PRNG | Pseudo-random number generator, acak semu yang deterministik dari seed |
| VRF | Verifiable Random Function (ChainLink) — acak off-chain dengan bukti on-chain |
| VDF | Verifiable Delay Function (VeeDo) |
| `onlyOwner` / `hasRole` | Modifier pengecek hak akses dari library OpenZeppelin dan sejenisnya |
| Least privilege | Prinsip memberi hak seminimal mungkin |
| Withdrawal pattern | Pola di mana penerima menarik dananya sendiri lewat fungsi terpisah |

## Isi

### Kenapa smart contract security itu beda kelas
Slide membuka dengan empat kalimat yang jadi fondasi seluruh materi ini:

1. Sama seperti program lain, **smart contract mengeksekusi persis apa yang ditulis, yang tidak selalu sama dengan yang dimaksud programmer**.
2. Semua smart contract itu **publik**, dan user mana pun bisa berinteraksi dengannya cukup dengan membuat transaksi.
3. **Kerentanan apa pun bisa dieksploitasi**, dan kerugiannya **hampir selalu mustahil dipulihkan**.
4. Karena itu, mengikuti best practice dan memakai design pattern yang teruji itu **kritis**.

> [!info] Konteks tambahan (bukan dari slide)
> Bandingin sama web app biasa: nemu bug SQL injection di hari Senin, kamu patch hari Selasa, selesai. Di smart contract, kode yang sudah ter-deploy **gak bisa di-patch**, source-nya bisa dibaca semua orang, dan yang jebol itu **uang, langsung, permanen**. Gak ada tombol undo, gak ada bank yang bisa dimintai reversal. Itu sebabnya seluruh matkul ini condong ke arah audit, bukan sekadar "belajar coding".

### Defensive programming: lima pilar
Untuk membuat program yang aman, program itu harus dikembangkan dengan cara yang aman. Defensive programming sangat cocok untuk smart contract, dan menekankan lima best practice.

**1. Minimalism / Simplicity.** **Kompleksitas adalah musuh keamanan.** Makin sederhana kodenya dan makin sedikit yang dia lakukan, makin kecil peluang munculnya bug atau efek yang tak terduga. Developer pemula sering tergoda menulis banyak kode; padahal yang seharusnya dilakukan adalah **mencari cara untuk melakukan lebih sedikit** — lebih sedikit baris, lebih sedikit kompleksitas, lebih sedikit "fitur". Slide bahkan bilang: kalau ada yang cerita proyeknya menghasilkan "ribuan baris kode" untuk smart contract-nya, **kamu justru harus mempertanyakan keamanan proyek itu**.

**2. Code Reuse.** Jangan bikin ulang roda. Kalau sudah ada library atau contract yang melakukan sebagian besar dari yang kamu butuhkan, **pakai ulang**. Di dalam kodemu sendiri, ikuti prinsip **DRY (Don't Repeat Yourself)**: kalau ada potongan kode yang berulang lebih dari sekali, tanya apakah itu bisa ditulis sebagai fungsi atau library. **Kode yang sudah banyak dipakai dan diuji kemungkinan besar lebih aman daripada kode baru mana pun yang kamu tulis.** Waspadai **"Not Invented Here" syndrome** — godaan untuk "memperbaiki" fitur atau komponen dengan membangunnya dari nol. Risiko keamanannya sering kali lebih besar daripada nilai perbaikannya.

**3. Code Quality.** **Kode smart contract itu gak kenal ampun.** Tiap bug bisa berujung kerugian uang. Kamu **tidak boleh** memperlakukan pemrograman smart contract sama seperti pemrograman umum. Terapkan metodologi rekayasa dan pengembangan software yang ketat, seperti yang kamu lakukan di **aerospace engineering** atau disiplin lain yang sama-sama tak kenal ampun. Sekali kamu "meluncurkan" kodemu, sedikit sekali yang bisa kamu lakukan untuk memperbaiki masalah.

**4. Readability / Auditability.** Kodemu harus jelas dan mudah dipahami. Makin mudah dibaca, makin mudah diaudit. Smart contract itu publik — semua orang bisa membaca bytecode-nya dan siapa pun bisa me-reverse-engineer-nya. Karena itu justru menguntungkan mengembangkan pekerjaanmu **secara terbuka**, dengan metodologi kolaboratif dan open source, supaya bisa memanfaatkan kebijaksanaan kolektif komunitas developer. Tulis kode yang terdokumentasi baik dan mudah dibaca, mengikuti gaya dan konvensi penamaan komunitas Ethereum.

**5. Test Coverage.** Uji semua yang bisa kamu uji. Smart contract berjalan di lingkungan eksekusi publik, di mana **siapa pun bisa mengeksekusinya dengan input apa pun yang mereka mau**. Kamu **tidak boleh pernah berasumsi** bahwa input, misalnya argumen fungsi, itu well-formed, berada dalam batas yang wajar, atau punya niat yang baik. Uji semua argumen untuk memastikan mereka berada dalam rentang yang diharapkan dan formatnya benar **sebelum** mengizinkan kodemu lanjut jalan.

---

### Risiko 1: Reentrancy Attack

**Cara kerjanya.** Salah satu fitur smart contract Ethereum adalah kemampuannya **memanggil dan memakai kode dari contract eksternal lain**. Contract juga biasanya menangani ether, dan karenanya sering mengirim ether ke berbagai alamat user eksternal. Operasi-operasi ini mengharuskan contract mengirim **external call**. External call inilah yang **bisa dibajak penyerang**, yang lalu bisa memaksa contract mengeksekusi kode lebih lanjut (lewat **fallback function**), termasuk memanggil balik ke dalam dirinya sendiri.

Lebih detail: serangan ini bisa terjadi saat sebuah contract mengirim ether ke alamat yang tidak dikenal. Penyerang bisa menyusun dengan hati-hati sebuah contract di alamat eksternal yang **memuat kode jahat di fallback function**-nya. Jadi saat contract korban mengirim ether ke alamat itu, ether itu **memicu kode jahat tersebut**. Biasanya kode jahat itu mengeksekusi fungsi di contract yang rentan, melakukan operasi yang tidak diharapkan developer. Istilah "reentrancy" datang dari fakta bahwa contract jahat eksternal **memanggil kembali (re-enter)** fungsi di contract yang rentan.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi reentrancy: bayangin ATM yang **ngeluarin uang duluan, baru ngurangi saldo**. Kamu tarik Rp1 juta; mesin ngeluarin uangnya, dan sebelum sempat nulis "saldo berkurang", kamu pencet tarik lagi. Saldonya masih tercatat utuh, jadi mesin ngeluarin lagi. Dan lagi. Urutan yang salah — **bayar dulu, catat belakangan** — itulah seluruh bug-nya. Makanya mitigasi nomor duanya persis kebalikan: **catat dulu, baru bayar**.

**Hands-on (slide 13–14).** Ada contract bernama **`EtherStore`** yang berfungsi sebagai brankas Ethereum yang mengizinkan depositor menarik **maksimal 1 ether per minggu**. Contract ini punya dua fungsi publik: `depositFunds` (cuma menambah saldo pengirim) dan `withdrawFunds` (mengizinkan pengirim menentukan jumlah wei yang ditarik). `withdrawFunds` **dimaksudkan** hanya berhasil kalau jumlah yang diminta kurang dari 1 ether **dan** belum ada penarikan dalam seminggu terakhir.

**Tugas:** buat smart contract jahat yang mengeksploitasi contract EtherStore.

**Tiga strategi mitigasi:**

| Strategi | Kenapa berhasil |
| --- | --- |
| **1. Pakai `transfer()` bawaan** saat mengirim ether ke contract eksternal | `transfer` cuma mengirim **2300 gas** bersama external call-nya, yang **tidak cukup** bagi alamat/contract tujuan untuk memanggil contract lain (yaitu, me-reenter contract pengirim) |
| **2. Semua logika yang mengubah state variable dijalankan SEBELUM ether dikirim keluar** (atau sebelum external call apa pun) | Praktik yang baik: kode yang melakukan external call ke alamat tak dikenal sebaiknya jadi **operasi terakhir** di dalam suatu fungsi atau blok eksekusi |
| **3. Pakai mutex** | Menambahkan state variable yang **mengunci contract** selama eksekusi kode, mencegah panggilan reentrant |

---

### Risiko 2: Arithmetic Overflow / Underflow

**Cara kerjanya.** EVM menetapkan **tipe data berukuran tetap** untuk integer. Artinya sebuah variabel integer cuma bisa merepresentasikan rentang angka tertentu. Contohnya, `uint8` cuma bisa menyimpan angka di rentang **[0, 255]**. Mencoba menyimpan 256 ke dalam `uint8` akan menghasilkan **0**.

Kalau tidak hati-hati, variabel di Solidity bisa dieksploitasi ketika input user tidak dicek dan perhitungan dilakukan sehingga menghasilkan angka **di luar rentang tipe data** yang menyimpannya.

**Underflow.** Bayangin mengurangi 1 dari variabel `uint8` (unsigned = tidak negatif) yang nilainya 0. Aksi ini menghasilkan **255**, karena kamu memberikan angka di bawah rentang `uint8`, sehingga hasilnya **berputar (wraps around)** dan memberikan angka terbesar yang bisa disimpan `uint8`. Serupa, menambahkan 256 ke `uint8` akan **membiarkan variabelnya tidak berubah**, karena kamu berputar sepanjang keseluruhan rentang uint.

Slide kasih analogi sendiri: **odometer mobil**, yang mengukur jarak tempuh — dia reset ke 000000 setelah melewati angka terbesarnya, 999999.

**Overflow.** Menambahkan angka yang lebih besar dari rentang tipe data disebut overflow. Contoh: menambahkan **257** ke `uint8` yang nilainya 0 akan menghasilkan angka **1**.

Cara berpikir yang membantu: anggap variabel berukuran tetap itu **siklis** — kita mulai lagi dari nol kalau menambahkan angka di atas angka terbesar yang bisa disimpan, dan mulai menghitung mundur dari angka terbesar kalau mengurangi dari nol. Untuk tipe `int` bertanda (yang bisa merepresentasikan angka negatif), kita mulai lagi setelah mencapai nilai negatif terbesar. Contoh: mengurangi 1 dari `int8` yang nilainya **-128** akan menghasilkan **127**.

**Hands-on (slide 19–20).** Ada contract bernama **`TimeLock`**, dirancang berfungsi sebagai brankas berjangka: user bisa mendeposit ether dan ether itu akan terkunci **minimal seminggu**. User boleh memperpanjang waktu tunggu jadi lebih dari seminggu kalau mau, tapi begitu didepositkan, user bisa yakin ether-nya terkunci aman minimal seminggu — begitu maksud contract ini. Skenario penggunaannya: kalau user dipaksa menyerahkan private key-nya, contract ini bisa memastikan ether-nya tidak bisa diambil untuk sementara waktu.

**Tugas:** lakukan arithmetic overflow untuk menerima ether-nya, **tanpa peduli berapa pun sisa LockTime**-nya.

**Mitigasi.** Teknik konvensional saat ini adalah **memakai atau membangun library matematika** yang menggantikan operator standar penjumlahan, pengurangan, dan perkalian. Perlu dicatat: **operasi pembagian dikecualikan**, karena dia tidak menyebabkan over/underflow, dan EVM sudah otomatis revert saat pembagian dengan 0. Library **SafeMath dari OpenZeppelin** bisa dipakai untuk menghindari kerentanan under/overflow.

---

### Risiko 3: Insecure Randomness

**Akar masalahnya.** EVM adalah runtime environment untuk mengeksekusi smart contract, dan dia memakai **model deterministik**. Artinya untuk input tertentu, mengeksekusi smart contract **akan selalu menghasilkan output yang sama**. Sifat ini esensial untuk menjamin konsistensi dan keandalan smart contract di blockchain.

Di EVM, transaksi **diputar ulang oleh banyak node** untuk memverifikasinya. Kalau sebuah smart contract mengimplementasikan fungsi Random Number Generator (RNG), fungsi itu harus **secara konsisten menghasilkan angka acak yang sama** selama eksekusi, supaya node lain bisa memverifikasi perilaku contract-nya. Padahal ini **bertentangan dengan prinsip fundamental keacakan** — menurut definisinya, angka acak seharusnya selalu unik dan tidak bisa diprediksi.

Salah satu cara mengatasinya adalah memakai **pseudo-random number generator (PRNG)**, yang menghasilkan deretan byte yang **terlihat acak secara deterministik**, berdasarkan nilai seed privat awal dan state internal.

> [!info] Konteks tambahan (bukan dari slide)
> Ini paradoks yang layak dipahami sekali dan diingat selamanya: **blockchain harus deterministik supaya bisa diverifikasi, tapi keacakan harus tidak dapat diprediksi.** Dua sifat ini saling meniadakan. Jadi keacakan sejati **tidak bisa lahir dari dalam blockchain** — dia harus didatangkan dari luar (oracle) dengan bukti bahwa hasilnya tidak dicurangi. Itulah kenapa mitigasinya semua berupa layanan eksternal.

**Kenapa contoh yang "kelihatan aman" itu tetap bobol.** Slide menampilkan `BadRandomContract.sol` lalu membedah kenapa dia rentan. Mungkin **tergoda berpikir implementasi itu aman karena seed keacakannya tidak dikendalikan penyerang**. Tapi kenyataannya **seed itu diketahui penyerang**.

Dua masalah konkretnya:
1. Seorang **miner bisa menahan block** yang sudah ditemukannya, kalau angka acak yang diturunkan dari block hash itu merugikan dia. Dengan menahan block itu, tentu si miner kehilangan block reward.
2. Yang lebih mengkhawatirkan: karena **`block.number` adalah variabel yang tersedia di blockchain**, dia **bisa dipakai sebagai parameter input oleh user mana pun**. Dalam kasus contract perjudian, seorang user bisa memakai `uint(blockhash(block.number - 1))` sebagai input taruhannya dan **selalu menang**.

**Hands-on (slide 26).** Berdasarkan prinsip di atas, eksploitasi `BadRandomContract` dengan **memprediksi angka acaknya** untuk dikirim sebagai jawaban ke contract. Caranya: buat fungsi `attack()` yang, saat dipanggil, **memakai logika yang sama persis** dengan yang dipakai contract aslinya untuk menghasilkan angka.

**Mitigasi.** Strategi paling andal adalah **memakai library pihak ketiga yang terverifikasi**:
- **ChainLink's Verifiable Random Functions (VRF)**
- **VeeDo's Verifiable Delay Function (VDF)**

Layanan ini **tidak gratis**, tapi bekerja dengan model pembayaran yang berbeda-beda. Meskipun memakai komponen **off-chain** untuk menghitung angka acaknya, mereka **mempublikasikan bukti validitas** perhitungan itu **on-chain**, sehingga tetap bisa diverifikasi.

---

### Risiko 4: Incorrect Access Control

**Konteksnya.** Mayoritas smart contract memakai **library Ownership dari OpenZeppelin**. `Ownable` milik OpenZeppelin mendefinisikan modifier seperti **`onlyOwner`** yang mengecek apakah user yang memanggil fungsi itu adalah owner contract-nya. Library lain juga mengecek apakah user punya hak untuk memanggil fungsi, memakai modifier/fungsi seperti **`hasRole`**.

**Tiga contoh kerentanannya:**

| Kerentanan | Penjelasan |
| --- | --- |
| **Missed Modifier Validations** | Validasi yang hilang, entah di dalam modifier atau di dalam `require`/pernyataan kondisional, kemungkinan besar berujung pada kompromi contract atau hilangnya dana |
| **Incorrect Modifier Names** | Karena kesalahan developer atau salah eja, nama modifier atau fungsi bisa berbeda dari yang dimaksud. Aktor jahat bisa memanfaatkan ini untuk memanggil fungsi kritis **tanpa modifier**, yang bisa berujung hilangnya dana atau berpindahnya kepemilikan, tergantung logika fungsinya |
| **Overpowered Roles** | Membiarkan user punya peran yang terlalu berkuasa bisa menimbulkan kerentanan. **Praktik least privilege harus selalu diikuti** dalam memberikan hak akses |

**Studi kasus: HospoWise.** HospoWise, proyek yang membawa industri perhotelan ke blockchain, **diretas karena fungsi `burn()` yang bersifat publik**.

Yang dilakukan hacker, berurutan:
1. Membuat contract yang berinteraksi dengan contract Hospo
2. Menukar ETH dengan token
3. **Membakar (burn) token dari pair**
4. Melakukan sync pada pair untuk **menaikkan harga token secara artifisial**
5. Menjual token yang harganya sudah kemahalan itu, mendapat keuntungan

Ini bisa dicegah kalau fungsi `burn()` punya access control seperti **`onlyOwner`**, atau kalau fungsinya dibuat **`internal`** dengan logika access control yang benar.

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan pola yang muncul dua kali di matkul ini: **Parity `initWallet`** ([[W07 - The Hardhat Framework]]) dan **HospoWise `burn()`**. Dua-duanya bug yang sama persis bentuknya — **fungsi kritis yang lupa dikasih modifier**. Bukan bug matematika yang rumit, bukan eksploitasi kriptografi. Cuma satu kata yang lupa ditulis. Ini yang bikin access control jadi kategori temuan audit yang paling sering muncul.

---

### Risiko 5: Unchecked External Calls

**Cara kerjanya.** Di Solidity kamu bisa memakai beberapa metode call level rendah yang bekerja pada alamat mentah: **`call`, `callcode`, `delegatecall`, dan `send`**. Metode level rendah ini **tidak pernah melempar exception**, tapi **mengembalikan `false`** kalau call-nya menemui exception.

Ide utama kerentanan jenis ini: **nilai kembalian dari message call tidak dicek**. Akibatnya, eksekusi akan **tetap lanjut** meskipun contract yang dipanggil melempar exception. Kalau call-nya gagal secara tidak sengaja, atau penyerang **memaksa call itu gagal**, ini bisa menyebabkan perilaku tak terduga pada logika program berikutnya.

**Contoh: contract `Lotto`.** Di contract itu ada bug: `send` dipakai **tanpa mengecek responsnya**. Kalau transaksi ke pemenang gagal, `payedOut` **tetap di-set menjadi `true`** — terlepas dari apakah ether-nya benar-benar terkirim atau tidak. Dalam kasus ini, **publik bisa menarik hadiah si pemenang** lewat fungsi `withdrawLeftOver`.

**Lima strategi mitigasi:**

| Strategi | Isinya |
| --- | --- |
| **Use Transfer Over Send** | Sebisa mungkin pakai `transfer()` alih-alih `send()`, karena `transfer()` **me-revert transaksi** kalau external call-nya gagal |
| **Check Return Values** | Selalu validasi nilai kembalian fungsi `send()` atau `call()` supaya bisa mengambil tindakan yang tepat kalau mereka mengembalikan `false` |
| **Adopt a Withdrawal Pattern** | Dengan withdrawal pattern, **end-user yang memanggil fungsi terpisah** untuk menyelesaikan transaksinya, sehingga contract bisa menangani kegagalan external call dengan lebih anggun |
| **Implement Event Logging** | Dengan event, contract bisa mencatat aktivitas tertentu. Kalau fungsi seperti `send()` atau `call()` gagal, event ini bisa terekam — memberikan transparansi dan keterlacakan |
| **Gas Limit Considerations** | Pastikan selalu ada gas yang memadai untuk operasinya. Meskipun `send()` dan `transfer()` otomatis memakai stipend 2300 gas, operasi lain mungkin butuh lebih |

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan ketegangan yang gak dibahas slide: mitigasi **reentrancy** menyarankan `transfer()` **karena** dia cuma ngasih 2300 gas. Mitigasi **unchecked external call** juga menyarankan `transfer()`, tapi lalu ngingetin soal keterbatasan 2300 gas itu. Dua-duanya benar dan dua-duanya soal `transfer()`, tapi dari sudut yang berlawanan. Kalau ditanya di ujian, jawab sesuai konteks risikonya.

## Diagram & Visual
- **Slide 14 — kode `EtherStore.sol` (target hands-on reentrancy)**
  ![[99-Assets/Blockchain/W08-slide14.png]]
- **Slide 20 — kode `TimeLock.sol` (target hands-on arithmetic overflow)**
  ![[99-Assets/Blockchain/W08-slide20.png]]
- **Slide 24 — kode `BadRandomContract.sol` (target hands-on insecure randomness)**
  ![[99-Assets/Blockchain/W08-slide24.png]]
- **Slide 32 — kode contract `Lotto` (contoh unchecked external call)**
  ![[99-Assets/Blockchain/W08-slide32.png]]

> [!warning] **Keempat contract yang jadi bahan latihan ada dalam bentuk gambar, bukan teks.** `EtherStore.sol`, `TimeLock.sol`, `BadRandomContract.sol`, dan `Lotto` gak bisa di-copy dari note ini — harus diketik ulang dari gambar di atas atau dari PPT aslinya. Kode contract HospoWise di slide 30 juga tidak terekstrak.

## Rumus / Sintaks

Batas tipe data yang jadi sumber over/underflow:
```
uint8   : [0, 255]        256 -> 0      -1 -> 255      +257 dari 0 -> 1
int8    : [-128, 127]     -128 - 1 -> 127
```

Stipend gas pada pengiriman ether:
```
transfer() dan send() -> 2300 gas
```

Pola prediksi keacakan yang bocor:
```solidity
uint(blockhash(block.number - 1))   // bisa dihitung sendiri oleh penyerang
```

## Hands-on
1. **Reentrancy** (slide 13–14) — buat contract jahat yang menguras `EtherStore`.
2. **Arithmetic overflow** (slide 19–20) — bobol `TimeLock` supaya ether bisa diambil berapa pun sisa LockTime-nya.
3. **Insecure randomness** (slide 26) — buat fungsi `attack()` yang memprediksi angka acak `BadRandomContract` dengan meniru logika generatornya.

## Pertanyaan Terbuka
- **Fallback function** adalah inti mekanisme reentrancy. Penjelasannya ada di [[W03 - Ethereum Basics]] slide 20–21 (`receive()` vs `fallback()`, plus modifier `payable`) — baca itu dulu kalau mekanisme serangan ini terasa gak nyambung. Deck W08 sendiri gak mengulang penjelasannya.
- Sejak **Solidity 0.8.0**, over/underflow **otomatis revert** tanpa perlu SafeMath. Slide masih mengajarkan SafeMath sebagai mitigasi utama. Perlu ditanya apakah yang diujikan versi pra-0.8 atau kondisi terkini — sementara [[W07 - The Hardhat Framework]] sendiri memakai compiler 0.8.8.
- Ada **ketidakcocokan angka di slide 17**: "menambahkan 256 ke `uint8` akan membiarkan variabelnya tidak berubah" itu benar (256 mod 256 = 0), tapi kalimat di slide 18 "menambahkan 257 ke `uint8` bernilai 0 menghasilkan 1" juga benar. Keduanya konsisten, tapi penyajiannya membingungkan kalau dibaca cepat. Pastikan paham aritmetika modulonya, bukan hafal angkanya.
- `delegatecall` disebut lagi di sini sebagai metode call level rendah, dan **lagi-lagi gak dijelaskan**. Ini kali ketiga dia muncul tanpa penjelasan (setelah library dan raw call di [[W05 - Solidity Development]], dan proxy Parity di [[W07 - The Hardhat Framework]]).
- Slide menyebut 5 risiko, sementara OWASP Smart Contract Top 10 (yang dijadikan referensi di slide 36) punya 10. Perlu ditanya apakah yang di luar 5 ini masuk ujian.
- Studi kasus HospoWise gak dikasih tanggal atau angka kerugiannya.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Blockchain]]
- [[W07 - The Hardhat Framework]]
- [[W09 - Smart Contract Auditing]]
- [[W05 - Solidity Development]]
- [[W04 - Wallets and Transactions]]
- [[Blockchain - Review dan Glosari]]
