---
matkul: Blockchain
minggu: 2
sks: 2
sumber: The Fundamentals of Ethereum.pptx
tags: [kuliah/blockchain, minggu/w02]
status: draft
diproses: 2026-09-03
---

# W02 — The Fundamentals of Ethereum

## Ringkasan
> - Ethereum sering disebut **"the world computer"**: state machine deterministik yang state-nya global dan tunggal (singleton), dijalankan oleh **EVM**.
> - Beda utama dari Bitcoin: Bitcoin itu jaringan pembayaran dengan bahasa script terbatas; Ethereum itu **blockchain general-purpose yang bisa diprogram** dengan kompleksitas tak terbatas. Ether bukan tujuan, tapi **utility currency** buat bayar komputasi.
> - Pengembangannya direncanakan dalam 4 tahap: **Frontier, Homestead, Metropolis, Serenity**, dengan banyak hard fork di antaranya (Ice Age, DAO, Tangerine Whistle, dst).
> - Komponennya: P2P network (ÐΞVp2p, port 30303), consensus rules (Yellow Paper), transactions, EVM, Merkle Patricia Tree, Nakamoto Consensus, dan client (Geth/Parity).
> - Dua bentuk aplikasi di atasnya: **DApp** (smart contract + frontend web) dan **DAO** (organisasi yang aturannya ditulis di kode, bukan di orang).

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| World computer | Julukan Ethereum: infrastruktur komputasi terdesentralisasi global |
| Singleton state | Satu state global yang sama untuk semua peserta jaringan |
| EVM | Ethereum Virtual Machine, VM berbasis stack yang mengeksekusi bytecode |
| Ether | Utility currency untuk mengukur dan membatasi biaya eksekusi |
| Hard fork | Perubahan fungsionalitas yang tidak backward-compatible |
| ÐΞVp2p | Protokol P2P Ethereum, jalan di TCP port 30303 |
| Yellow Paper | Dokumen yang mendefinisikan consensus rules Ethereum |
| Merkle Patricia Tree | Struktur data hash berseri tempat state Ethereum disimpan di tiap node |
| Nakamoto Consensus | Model consensus dari Bitcoin, pakai block bertanda tangan tunggal berurutan |
| Ethash | Algoritma PoW yang dipakai Ethereum (saat slide ditulis) |
| Casper | Rencana sistem voting berbobot berbasis PoS |
| Geth / Parity | Dua implementasi client Ethereum paling menonjol |
| DApp | Aplikasi web di atas infrastruktur P2P terdesentralisasi: smart contract + frontend |
| DAO | Decentralized Autonomous Organization, organisasi yang dijalankan oleh kode |

## Isi

### Ethereum dari dua sudut pandang
Slide sengaja mendefinisikan Ethereum dua kali, karena definisinya beda tergantung kamu ngeliat dari mana.

**Dari sudut pandang computer science**, Ethereum adalah *state machine yang deterministik tapi praktis tak terbatas*, terdiri dari satu **globally accessible singleton state** dan sebuah **virtual machine** yang menerapkan perubahan ke state itu.

**Dari sudut pandang praktis**, Ethereum adalah infrastruktur komputasi open source yang terdesentralisasi secara global, yang mengeksekusi program bernama **smart contract**. Dia pakai blockchain untuk menyinkronkan dan menyimpan perubahan state sistem, plus cryptocurrency bernama **ether** untuk mengukur (*meter*) dan membatasi (*constrain*) biaya sumber daya eksekusi.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi "world computer": bayangin satu komputer raksasa yang dipakai bareng-bareng seluruh dunia, tapi gak ada yang punya. Kamu gak bisa nyolokin flashdisk ke situ; cara satu-satunya nyuruh dia kerja adalah **kirim transaksi berbayar**. Dan karena semua orang harus bisa mengecek hasilnya, komputer ini **selalu deterministik** — program yang sama dengan input yang sama harus keluar hasil yang sama, di mana pun, kapan pun. Ether itu koin yang kamu masukin ke mesin biar dia mau jalan.

### Ethereum vs Bitcoin
Ethereum berbagi banyak elemen dengan blockchain terbuka lainnya:

- Jaringan **peer-to-peer** yang menghubungkan peserta.
- Algoritma consensus yang **Byzantine fault-tolerant** untuk sinkronisasi update state (saat itu berbasis proof-of-work).
- Primitif kriptografi seperti **digital signature** dan **hash**, plus mata uang digital (ether).

Bedanya justru di tujuannya. Tujuan utama Ethereum **bukan** jadi jaringan pembayaran mata uang digital. Ether dimaksudkan sebagai **utility currency** untuk membayar penggunaan platform Ethereum sebagai *world computer*.

Beda kedua: Bitcoin punya bahasa scripting yang sangat terbatas, sementara Ethereum dirancang sebagai **blockchain general-purpose yang bisa diprogram**, menjalankan virtual machine yang sanggup mengeksekusi kode dengan kompleksitas apa pun dan tanpa batas.

### Empat tahap pengembangan dan hard fork-nya
Pengembangan Ethereum direncanakan dalam empat tahap berbeda, dengan perubahan besar di tiap tahap. Sebuah tahap bisa berisi sub-rilis yang disebut **hard fork** — perubahan fungsionalitas yang **tidak backward-compatible**.

Empat tahap utamanya: **Frontier, Homestead, Metropolis, Serenity**.

| Block | Nama | Apa yang terjadi |
| --- | --- | --- |
| #0 | **Frontier** | Tahap awal Ethereum, 30 Juli 2015 – Maret 2016 |
| #200.000 | **Ice Age** | Hard fork yang menaikkan difficulty secara eksponensial, untuk memotivasi transisi ke PoS |
| #1.150.000 | **Homestead** | Tahap kedua, diluncurkan Maret 2016 |
| #1.192.000 | **DAO** | Hard fork untuk mengganti rugi korban peretasan kontrak DAO; memecah jaringan jadi Ethereum dan Ethereum Classic |
| #2.463.000 | **Tangerine Whistle** | Mengubah perhitungan gas untuk operasi I/O-berat, dan membersihkan state sisa serangan DoS |
| #2.675.000 | **Spurious Dragon** | Menutup lebih banyak vektor serangan DoS, state clearing lagi, plus mekanisme proteksi replay attack |
| #4.370.000 | **Metropolis Byzantium** | Tahap ketiga, Oktober 2017; menambah fungsionalitas low-level, menyesuaikan block reward dan difficulty |
| #7.280.000 | **Constantinople / St. Petersburg** | Rencananya bagian kedua Metropolis; beberapa jam sebelum aktivasi ditemukan bug kritis, jadi ditunda dan diganti nama |
| #9.069.000 | **Istanbul** | Hard fork lanjutan dengan pendekatan dan konvensi penamaan yang sama |
| #9.200.000 | **Muir Glacier** | Satu-satunya tujuannya menyesuaikan difficulty lagi, karena kenaikan eksponensial dari Ice Age |

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa "Ice Age" perlu di-undo sama "Muir Glacier": Ice Age sengaja bikin mining makin lama makin susah — semacam **bom waktu** biar developer terpaksa pindah ke PoS. Tapi PoS-nya belum siap-siap juga, sementara bomnya udah keburu bikin jaringan melambat. Jadi Muir Glacier itu cuma **nunda bomnya**. Pola "pasang bom waktu, lalu tunda berkali-kali" ini kepake beberapa kali di sejarah Ethereum.

### Komponen Ethereum
| Komponen | Isinya |
| --- | --- |
| **P2P Network** | Jalan di Ethereum main network, TCP port **30303**, protokol **ÐΞVp2p** |
| **Consensus Rules** | Didefinisikan di **Yellow Paper** |
| **Transactions** | Pesan jaringan berisi (antara lain) sender, recipient, value, dan data payload |
| **State Machine** | Transisi state diproses oleh **EVM**, VM berbasis stack yang mengeksekusi bytecode. Program EVM = smart contract, ditulis di bahasa tingkat tinggi (Solidity) lalu dikompilasi ke bytecode |
| **Data structures** | State disimpan lokal di tiap node sebagai database (biasanya LevelDB milik Google), dalam struktur hash berseri bernama **Merkle Patricia Tree** |
| **Consensus Algorithm** | Pakai **Nakamoto Consensus** dari Bitcoin (block bertanda tangan tunggal, berurutan); ada rencana pindah ke sistem voting berbobot PoS bernama **Casper** |
| **Economic Security** | Saat itu pakai algoritma PoW bernama **Ethash**, yang akan dilepas saat pindah ke PoS |
| **Clients** | Beberapa implementasi client yang saling interoperable; paling menonjol **Go-Ethereum (Geth)** dan **Parity** |

### Dari blockchain general-purpose ke DApp
Ethereum awalnya dimaksudkan sebagai blockchain general-purpose yang bisa diprogram untuk macam-macam kegunaan. Tapi visinya cepat meluas jadi **platform untuk memprogram DApp**. DApp itu perspektif yang lebih luas daripada sekadar smart contract.

Secara umum, **DApp adalah aplikasi web yang dibangun di atas layanan infrastruktur P2P yang terbuka dan terdesentralisasi**. Minimal, sebuah DApp terdiri dari:

1. **Smart contract** di blockchain
2. **Frontend user interface** berbasis web

### DAO: organisasi yang dijalankan kode
**DAO (Decentralized Autonomous Organization)** adalah organisasi yang dimiliki bersama dan bekerja menuju misi bersama. DAO memungkinkan seseorang bekerja dengan orang-orang sepemikiran di seluruh dunia **tanpa harus percaya pada seorang pemimpin baik hati** yang mengelola dana atau operasional.

Konkretnya, di DAO:
- Gak ada CEO yang bisa membelanjakan dana seenaknya, gak ada CFO yang bisa memanipulasi pembukuan.
- Aturan berbasis blockchain yang **ditanam di dalam kode** mendefinisikan cara organisasi bekerja dan cara dana dibelanjakan.
- Ada **treasury bawaan** yang gak bisa diakses siapa pun tanpa persetujuan kelompok.
- Keputusan diatur lewat **proposal dan voting**, semuanya terjadi transparan on-chain.

Kenapa perlu DAO? Karena memulai organisasi yang melibatkan dana dan uang butuh kepercayaan besar pada orang-orang yang kamu ajak kerja. Padahal susah percaya sama orang yang cuma pernah kamu temui lewat internet. Dengan DAO, **kamu gak perlu percaya siapa pun di kelompok itu — cukup percaya pada kode DAO-nya**, yang 100% transparan dan bisa diverifikasi siapa saja.

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan ironi yang belum dibahas di slide ini: hard fork **DAO** di tabel di atas itu justru terjadi karena sebuah DAO **kena hack**. Jadi premis "cukup percaya pada kodenya" itu benar hanya sejauh kodenya benar. Ini yang jadi bahan [[W08 - Smart Contract Pitfalls]] dan [[W09 - Smart Contract Auditing]] nanti.

### Kenapa belajar Ethereum
Blockchain punya kurva belajar yang curam, karena menggabungkan banyak disiplin sekaligus: programming, information security, kriptografi, ekonomi, sistem terdistribusi, jaringan peer-to-peer, dan lainnya. Ethereum bikin kurva itu jauh lebih landai, jadi bisa mulai cepat — tapi tepat di bawah permukaan lingkungan yang kelihatan sederhana itu ada banyak lapisan kompleksitas.

Ethereum juga membangun komunitas developer yang besar lebih cepat dari platform blockchain mana pun, dan developer yang terbiasa dengan aplikasi JavaScript bisa langsung masuk dan menghasilkan kode yang jalan dengan cepat.

Tapi slide menutup dengan peringatan yang penting banget:

> Ini pedang bermata dua. **Gampang nulis kode, tapi susah banget nulis kode yang bagus dan aman.**

## Diagram & Visual
- **Slide 12 — ilustrasi konsep DAO (organisasi tanpa pemimpin terpusat)**
  ![[99-Assets/Blockchain/W02-slide12.png]]

> [!warning] Deck ini paling miskin gambar dari seluruh matkul: cuma 1 gambar non-dekoratif dari 14 slide. Sisanya teks penuh, jadi note ini udah mewakili hampir seluruh isi PPT-nya.

## Pertanyaan Terbuka
- Slide ditulis saat Ethereum masih PoW dan Casper masih "rencana". **The Merge sudah terjadi September 2022** dan Ethereum sekarang full PoS — perlu dikonfirmasi ke dosen apakah yang diujikan versi slide (PoW/Ethash) atau kondisi terkini.
- Tahap **Serenity** disebut sebagai tahap keempat tapi gak pernah dijelaskan isinya. Kemungkinan besar itu yang sekarang dikenal sebagai Eth2/PoS.
- Merkle Patricia Tree cuma disebut nama, gak dijelaskan struktur atau kenapa dipilih. Kalau keluar di ujian, kemungkinan cuma level definisi.
- Rinkeby, Ropsten, dan testnet lain baru muncul di [[W07 - The Hardhat Framework]]; di sini belum dibahas ada berapa jaringan Ethereum.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Blockchain]]
- [[W01 - Introduction to Blockchain Technology]]
- [[W04 - Wallets and Transactions]]
- [[Blockchain - Review dan Glosari]]
