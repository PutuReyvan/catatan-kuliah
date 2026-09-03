---
matkul: Blockchain
minggu: 1
sks: 2
sumber: The Fundamentals of Blockchain Technology.pptx
tags: [kuliah/blockchain, minggu/w01]
status: draft
diproses: 2026-09-03
---

# W01 — Introduction to Blockchain Technology

## Ringkasan
> - Blockchain = **distributed ledger** yang nyimpen transaksi di banyak komputer sekaligus, dan begitu tercatat gak bisa diubah lagi. Lahir dari Bitcoin-nya Satoshi Nakamoto (2008).
> - Strukturnya cuma 3 hal: **nodes** (komputer peserta), **blocks** (kumpulan transaksi + header), **chain** (rantai block dari genesis sampai sekarang).
> - Yang bikin gak bisa dicurangi: **hash function** (SHA-256) + **consensus mechanism** (PoW / PoS). Ubah satu block, semua block sesudahnya ikut rusak.
> - **Blockchain trilemma**: security, scalability, decentralization — naikin satu, biasanya turun yang lain.
> - Turunan paling nyata: **DeFi**, keuangan tanpa perantara, dibangun dari smart contract, dApps, dan DEX.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Distributed ledger | Buku besar transaksi yang salinannya ada di banyak komputer, bukan di satu server |
| Node | Komputer yang ikut jaringan blockchain; memvalidasi dan meneruskan transaksi |
| Block | Unit data berisi header (timestamp + referensi ke block sebelumnya) dan daftar transaksi |
| Genesis block | Block pertama dalam rantai |
| Chain | Rangkaian block yang tersambung berurutan lewat hash |
| Consensus mechanism | Aturan yang bikin semua node sepakat soal isi ledger |
| Proof of Work (PoW) | Miner berlomba memecahkan teka-teki matematis untuk memvalidasi transaksi dan bikin block baru |
| Proof of Stake (PoS) | Validator dipilih berdasarkan jumlah koin yang dia *stake* sebagai collateral |
| Nonce | Bagian data block yang diubah-ubah miner supaya hash-nya jatuh di bawah target |
| Avalanche effect | Perubahan kecil di input bikin output hash berubah total |
| Blockchain trilemma | Trade-off antara security, scalability, dan decentralization |
| DeFi | Ekosistem aplikasi keuangan di atas blockchain, tanpa perantara |
| dApps | Decentralized applications |
| DEX | Decentralized Exchange, bursa tanpa operator terpusat |

## Isi

### Apa itu blockchain, dan kenapa penting
Blockchain adalah **distributed ledger technology** — teknologi buku besar yang mencatat transaksi di banyak komputer sekaligus, dirancang supaya transaksi yang sudah terdaftar **tidak bisa diubah secara retroaktif**. Asalnya dari penemuan Bitcoin oleh Satoshi Nakamoto tahun 2008.

Kenapa penting, menurut slide: blockchain meningkatkan transparansi, mengurangi fraud, dan memungkinkan dibangunnya aplikasi terdesentralisasi.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi: bayangin buku kas RT. Model lama, buku kas dipegang bendahara — kalau bendaharanya nakal, dia bisa tipp-ex satu baris dan gak ada yang tau. Model blockchain, **tiap warga punya fotokopi buku kas yang sama**, dan tiap halaman baru dikasih stempel yang dihitung dari isi halaman sebelumnya. Kalau bendahara mau ngubah pengeluaran bulan lalu, dia harus ngubah halaman itu, *plus* semua stempel halaman sesudahnya, *plus* meyakinkan mayoritas warga buat ikut ganti fotokopi mereka. Praktis mustahil. Itu inti blockchain.

### Struktur: node, block, chain
Tiga komponen yang harus nempel di kepala:

- **Nodes** — komputer individual yang ikut dalam jaringan; tugasnya memvalidasi dan meneruskan (relay) transaksi.
- **Blocks** — unit data yang menyimpan sekumpulan transaksi. Tiap block punya **header** (metadata: timestamp, referensi ke block sebelumnya) dan **daftar transaksi**.
- **Chain** — penyambungan block secara berurutan, dari **genesis block** sampai block terkini, membentuk ledger yang bersambung.

Konsekuensinya: sekali data dicatat di block dan block itu masuk ke rantai, data itu **tidak bisa diubah tanpa mengubah semua block sesudahnya**, dan itu butuh consensus dari seluruh jaringan.

### Consensus mechanism
Consensus mechanism memastikan semua node sepakat soal state blockchain dan memvalidasi transaksi. Slide menyebut dua mekanisme yang saat ini diterima:

- **Proof of Work (PoW)** — miner memecahkan teka-teki matematis yang rumit untuk memvalidasi transaksi dan membuat block baru.
- **Proof of Stake (PoS)** — validator dipilih berdasarkan jumlah koin yang mereka pegang dan bersedia mereka *stake* sebagai collateral.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi bedanya: **PoW itu lomba nyari kunci di tumpukan pasir** — siapa yang paling banyak nyekop (punya komputer paling kuat) paling mungkin menang, dan biaya listriknya jadi jaminan dia gak main curang. **PoS itu sistem uang jaminan** — kamu titip deposit gede ke sistem; kalau ketahuan curang, deposit hangus. Jadi jaminan kejujurannya bukan kerja keras, tapi uang yang dipertaruhkan.

### Alur hidup satu transaksi
Slide memecah operasi jaringan jadi enam langkah berurutan:

1. **Transaction Initiation** — user memulai transaksi (misal ngirim cryptocurrency).
2. **Transaction Broadcasting** — transaksi disiarkan ke jaringan node.
3. **Transaction Validation** — node memvalidasi lewat consensus mechanism: cek identitas pengirim, pastikan saldonya cukup, cek transaksinya patuh aturan protokol.
4. **Block Formation** — transaksi yang lolos validasi dikelompokkan jadi satu block oleh miner/validator.
5. **Block Addition** — block baru ditambahkan ke rantai, disambungkan ke block sebelumnya lewat **cryptographic hash**.
6. **Network Consensus → Transaction Completion** — jaringan mencapai consensus atas validitas block; begitu tercapai, block dianggap confirmed dan immutable, dan ledger di semua node ter-update.

### Kriptografi: hash function dan digital signature
Blockchain memakai dua jenis algoritma kriptografi: **asymmetric-key algorithms** dan **hash functions**.

Hash function-lah yang bikin semua peserta punya satu pandangan yang sama atas blockchain — dia menghasilkan string unik yang mewakili data di dalam tiap block. Blockchain umumnya pakai **SHA-256**. Peran utamanya dua: menyambungkan block satu ke lainnya, dan menjaga integritas data di dalam tiap block. Perubahan apa pun pada data block bikin rantai jadi inkonsisten dan invalid — ini dijamin oleh sifat hash yang disebut **avalanche effect**.

Lima sifat hash function menurut slide:

| Sifat | Artinya |
| --- | --- |
| **Avalanche effect** | Perubahan sedikit di data menghasilkan output yang beda total |
| **Deterministic** | Input yang sama selalu menghasilkan output yang sama |
| **Fast** | Menghitung hash itu cepat, gak butuh komputasi berat |
| **Unique** | Tiap input menghasilkan output yang acak dan unik (gak ada dua input beda dengan output sama) |
| **Irreversible** | Dari output, input aslinya gak bisa didapat balik |

### Hashing di dalam Bitcoin mining
Di Bitcoin mining, hashing adalah proses yang membungkus mekanisme proof-of-work: miner berlomba menemukan hash yang **nilainya sama dengan atau di bawah target** yang ditetapkan jaringan.

Cara kerjanya brute-force trial-and-error. Miner mencari satu set data yang hash-nya, kalau dikonversi ke nilai numerik, lebih kecil dari target jaringan. Contoh dari slide: kalau target jaringan 21, hash bernilai 33 itu kegedean — gagal; hash bernilai 17 menang. Angka 21/33/17 itu didapat dari mengubah hash SHA-256 ke nilai numeriknya.

Yang diubah-ubah miner adalah bagian data block yang disebut **nonce**. Untuk tiap variasi (block + nonce + data), miner menghitung hash-nya dan mengecek apakah sudah di bawah target.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi mining: bayangin kamu disuruh nyari kombinasi gembok yang kalau diputar hasilnya nunjuk angka di bawah 21. Gemboknya gak bisa dibalik-hitung (sifat *irreversible*), jadi satu-satunya cara ya coba satu-satu: putar, lihat, gagal, putar lagi. Yang diputar itu **nonce**. Yang pertama nemu, dia yang berhak nulis halaman baru di buku besar dan dapet hadiah.

### Blockchain trilemma
Trilemma merujuk ke trade-off antara tiga aspek kritis:

- **Security** — pertahanan yang kokoh supaya pihak jahat gak bisa mengambil alih jaringan.
- **Scalability** — sanggup menampung banyak transaksi dan user tanpa bikin fee dan waktu transaksi naik drastis.
- **Decentralization** — kontrol atas jaringan terdistribusi merata ke semua peserta, bukan terpusat di satu entitas.

Ketiganya saling terkait sedemikian rupa sehingga **memperbaiki satu sering kali mengorbankan yang lain**.

### Manfaat dan turunannya: DeFi
Manfaat yang disebut slide: **decentralization** (gak ada single point of control, mengurangi risiko fraud dan sensor), **transparency** (semua transaksi tercatat di ledger publik), **security** (teknik kriptografi menjaga integritas dan kerahasiaan data), dan **immutability** (sekali tercatat, gak bisa diubah).

**DeFi (Decentralized Finance)** adalah ekosistem aplikasi keuangan yang dibangun di atas jaringan blockchain. DeFi menghapus perantara dan memungkinkan transaksi peer-to-peer. Komponennya: **smart contracts**, **dApps**, dan **DEXs**.

- Benefit: aksesibilitas lebih luas, transparansi, dan kontrol penuh atas keuangan pribadi.
- Risiko: kompleksitas teknis, ketidakpastian regulasi, dan potensi kerentanan keamanan.

## Diagram & Visual
- **Slide 2 — ilustrasi konsep blockchain sebagai rantai block**
  ![[99-Assets/Blockchain/W01-slide02.png]]
- **Slide 5 — visual pembanding Proof of Work vs Proof of Stake**
  ![[99-Assets/Blockchain/W01-slide05.jpg]]
- **Slide 10 — skema hashing dalam proses Bitcoin mining**
  ![[99-Assets/Blockchain/W01-slide10.png]]
- **Slide 12 — ilustrasi pencarian nonce terhadap target jaringan**
  ![[99-Assets/Blockchain/W01-slide12.jpg]]
- **Slide 13 — tampilan demo blockchain interaktif (andersbrownworth.com)**
  ![[99-Assets/Blockchain/W01-slide13.png]]

> [!warning] Slide 5 dan 12 di PPT aslinya cuma judul + gambar, gak ada teks yang bisa diekstrak. Deskripsi di atas dibaca dari konteks slide sebelum dan sesudahnya — buka PPT-nya kalau butuh detail persis.

## Hands-on
Slide 13 nyuruh main-main dengan demo blockchain interaktif untuk membayangkan cara kerja elemen kriptografisnya: `https://andersbrownworth.com/blockchain/`

## Pertanyaan Terbuka
- Slide nyebut "currently two acceptable mechanisms" (PoW dan PoS), padahal ada PoA, DPoS, dll. Apakah dosen membatasi scope ke dua ini aja buat ujian?
- Contoh target "21 / 33 / 17" itu disederhanakan banget. Gimana angka target sebenarnya ditentukan dan di-adjust jaringan (difficulty adjustment)? Slide gak bahas.
- Trilemma disebut tapi gak dikasih contoh konkret chain mana yang mengorbankan apa. Berpotensi jadi soal analisis.
- Asymmetric-key algorithm disebut sebagai salah satu dari dua jenis kriptografi, tapi cuma hash function yang dijelaskan detail. Digital signature baru kebahas di [[W04 - Wallets and Transactions]].

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Blockchain]]
- [[W02 - The Fundamentals of Ethereum]]
- [[Blockchain - Review dan Glosari]]
