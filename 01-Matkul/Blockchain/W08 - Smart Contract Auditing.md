---
matkul: Blockchain
minggu: 8
sks: 2
sumber: Smart Contract Auditing.pptx
tags: [kuliah/blockchain, minggu/w08]
status: draft
diproses: 2026-09-03
---

# W08 — Smart Contract Auditing

> [!note] Slide 2 nyatakan deck ini dibawakan dalam **dua sesi terpisah (S11 dan S12)**.

## Ringkasan
> - **Audit** = penilaian keamanan eksternal atas basis kode proyek, diminta dan dibayar tim proyek. Outputnya laporan berisi isu, severity, skenario eksploitasi, dan rekomendasi perbaikan.
> - Biayanya nyata: **mulai dari USD $10K per minggu**, tergantung kompleksitas proyek dan reputasi firma auditor.
> - Temuan diklasifikasikan per kategori (versi **Trail of Bits**: Access Controls, Data Validation, Cryptography, dst — 13 kategori).
> - **Severity = Likelihood × Impact.** Trail of Bits pakai skala Informational→High; ConsenSys pakai Minor→Critical.
> - Dua teknik utama: **static analysis** (Slither di level Solidity, Mythril di level bytecode) dan **fuzzing** (Echidna, Harvey).
> - Puncaknya: **CTF showdown** di Damn Vulnerable DeFi.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Audit | Penilaian keamanan eksternal atas basis kode proyek, dibayar tim proyek |
| Attack surface | Luas permukaan yang bisa diserang; tujuan audit adalah menguranginya |
| Likelihood | Seberapa mudah/sulit kerentanan ditemukan dan dieksploitasi penyerang |
| Impact | Perkiraan besarnya dampak teknis dan bisnis kalau kerentanan dieksploitasi |
| Severity | Hasil kombinasi Likelihood dan Impact |
| Trail of Bits | Salah satu firma auditing keamanan blockchain papan atas |
| ConsenSys | Firma dengan skala severity Minor/Medium/Major/Critical |
| Static analysis | Menganalisis properti program **tanpa menjalankannya** |
| Slither | Static analyzer level **Solidity**, buatan Trail of Bits |
| Mythril | Security analyzer level **EVM bytecode** |
| Symbolic execution | Mengeksekusi kode dengan input simbolik, bukan data nyata, untuk menjelajah banyak jalur eksekusi |
| SMT solver | Mesin penyelesai batasan logika yang dipakai Mythril |
| Taint analysis | Analisis pelacakan aliran data yang "terkontaminasi" input |
| Fuzzing | Memberi input tak valid/tak terduga/acak secara otomatis, lalu memantau anomali |
| Echidna | Fuzzer smart contract berbasis Haskell, property-based |
| Harvey | Tool fuzzing smart contract lainnya |
| SlithIR | Intermediate representation milik Slither |

## Isi

### Apa itu audit, dan berapa harganya
**Audit adalah penilaian keamanan eksternal atas basis kode sebuah proyek**, yang biasanya diminta dan dibayar oleh tim proyek itu sendiri. Audit mendeteksi dan mendeskripsikan (dalam sebuah laporan) isu-isu keamanan beserta **kerentanan yang mendasarinya, severity/difficulty, skenario eksploitasi potensial, dan rekomendasi perbaikan**. Audit juga memberi **wawasan subjektif** soal kualitas kode, dokumentasi, dan testing.

Scope, kedalaman, dan format laporan audit berbeda-beda antar tim auditor, tapi umumnya mencakup aspek-aspek yang serupa.

**Scope dan tujuan untuk proyek Ethereum.** Untuk proyek smart contract berbasis Ethereum, scope-nya biasanya adalah **kode smart contract on-chain**, dan kadang mencakup **komponen off-chain** yang berinteraksi dengan smart contract tersebut.

Tujuan audit adalah menilai kode proyek (beserta spesifikasi dan dokumentasi terkait) dan **memberi peringatan ke tim proyek, biasanya sebelum peluncuran**, soal isu-isu keamanan potensial yang perlu ditangani untuk memperbaiki postur keamanannya. Ini memungkinkan penulis contract **mengurangi attack surface dan memitigasi risiko**.

**Biayanya:** tergantung tipe/scope audit, tapi umumnya **mulai dari USD $10.000 per minggu**, tergantung kompleksitas proyek, permintaan/penawaran pasar untuk audit, dan kekuatan/reputasi firma auditornya.

> [!info] Konteks tambahan (bukan dari slide)
> Angka $10K/minggu itu bukan detail sepele — dia menjelaskan **kenapa profesi ini ada**. Kalau contract-mu memegang $50 juta dan bug satu baris bisa menguapkannya permanen tanpa bisa dipulihkan (lihat [[W07 - Smart Contract Pitfalls]]), audit $40 ribu itu **murah**. Ekonomi inilah yang bikin smart contract auditing jadi jalur karier tersendiri, dan itu yang lagi kamu pelajari di dua sesi ini.

### Klasifikasi temuan audit
Kerentanan yang ditemukan saat audit biasanya diklasifikasikan ke dalam kategori berbeda, yang membantu memahami **sifat kerentanannya, dampak/severity potensialnya, komponen/fungsionalitas proyek yang terdampak, dan skenario eksploitasinya**.

**Trail of Bits**, salah satu firma keamanan blockchain papan atas, memakai klasifikasi berikut:

| Kategori | Berkaitan dengan |
| --- | --- |
| **Access Controls** | Otorisasi user dan penilaian hak akses |
| **Auditing and Logging** | Pencatatan aksi atau pencatatan masalah |
| **Authentication** | Identifikasi user |
| **Configuration** | Konfigurasi keamanan server, perangkat, atau software |
| **Cryptography** | Perlindungan privasi atau integritas data |
| **Data Exposure** | Terbukanya informasi sensitif secara tidak sengaja |
| **Data Validation** | Ketergantungan yang tidak semestinya pada struktur atau nilai data |
| **Denial of Service** | Menyebabkan kegagalan sistem |
| **Error Reporting** | Pelaporan kondisi error secara aman |
| **Patching** | Menjaga software tetap mutakhir |
| **Session Management** | Identifikasi user yang sudah terautentikasi |
| **Timing** | Race condition, locking, atau urutan operasi |
| **Undefined Behavior** | Perilaku tak terdefinisi yang terpicu oleh program |

### Likelihood, Impact, dan Severity
**Likelihood** didefinisikan OWASP sebagai ukuran kasar **seberapa mungkin atau sulit** suatu kerentanan ditemukan dan dieksploitasi penyerang, dibagi tiga tingkat: Low, Medium, High.

Trail of Bits mengklasifikasikan tiap temuan ke **empat tingkat kesulitan** likelihood:

| Tingkat | Artinya |
| --- | --- |
| **Undetermined** | Tingkat kesulitan eksploitasinya tidak ditentukan selama engagement ini |
| **Low** | Umum dieksploitasi; sudah ada tool publik, atau bisa di-script |
| **Medium** | Penyerang harus menulis exploit sendiri, atau butuh pengetahuan mendalam soal sistem yang kompleks |
| **High** | Penyerang harus punya **akses insider** yang istimewa, mungkin perlu tahu detail teknis yang sangat rumit, atau harus menemukan kelemahan lain dulu untuk bisa mengeksploitasi isu ini |

**Impact** didefinisikan sebagai perkiraan **besarnya dampak teknis dan bisnis** pada sistem kalau kerentanan itu dieksploitasi, juga dibagi Low, Medium, High.

**Severity** dihitung dengan **menggabungkan estimasi Likelihood dan estimasi Impact** — tentukan Likelihood-nya (Low/Medium/High), lalu lakukan hal yang sama untuk Impact.

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan arah skala Likelihood versi Trail of Bits, karena **kebalikan dari intuisi**. "Low" **bukan** berarti kecil kemungkinan terjadi — "Low" artinya **kesulitannya rendah**, jadi justru **gampang** dieksploitasi dan **lebih berbahaya**. Ini jebakan soal ujian yang klasik. Bacanya: Low difficulty = high risk.

**Skala severity Trail of Bits:**

| Level | Artinya |
| --- | --- |
| **Informational** | Isunya tidak menimbulkan risiko langsung, tapi relevan dengan best practice keamanan atau Defence in Depth |
| **Undetermined** | Besarnya risiko tidak ditentukan selama engagement ini |
| **Low** | Risikonya relatif kecil, atau bukan risiko yang dinyatakan penting oleh klien |
| **Medium** | Informasi user individual berisiko; eksploitasi akan buruk bagi reputasi klien; dampak finansial sedang; kemungkinan ada implikasi hukum bagi klien |
| **High** | Menyangkut user dalam jumlah besar, sangat buruk bagi reputasi klien, atau punya implikasi hukum/finansial yang serius |

**Skala severity ConsenSys:**

| Level | Artinya |
| --- | --- |
| **Minor** | Bersifat subjektif. Biasanya berupa saran seputar best practice atau keterbacaan. Pemelihara kode boleh memakai penilaiannya sendiri apakah mau menanganinya |
| **Medium** | Bersifat objektif tapi **bukan kerentanan keamanan**. Harus ditangani kecuali ada alasan yang jelas untuk tidak |
| **Major** | Kerentanan keamanan yang mungkin tidak langsung bisa dieksploitasi, atau butuh kondisi tertentu untuk bisa dieksploitasi. **Semua isu major harus ditangani** |
| **Critical** | Kerentanan keamanan yang **langsung bisa dieksploitasi** dan perlu diperbaiki |

### Static analysis
**Static analysis** adalah teknik menganalisis properti program **tanpa benar-benar mengeksekusi program itu**. Ini kontras dengan software testing, di mana program benar-benar dijalankan dengan input yang berbeda-beda.

Untuk smart contract, static analysis bisa dilakukan pada **kode Solidity** atau pada **EVM bytecode**:
- **Slither** melakukan static analysis di level **Solidity**
- **Mythril** menganalisis **EVM bytecode**

Static analysis umumnya adalah kombinasi dari **control flow analysis** dan **data flow analysis**.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi static vs dynamic: **static analysis itu baca resep**, dynamic testing itu **masak lalu icip**. Baca resep bisa nemu "lho, garamnya dimasukin dua kali" tanpa nyalain kompor sama sekali — cepat, murah, dan bisa memeriksa semua langkah. Tapi ada hal yang cuma ketahuan pas dimasak. Makanya audit beneran pakai dua-duanya, plus fuzzing.

**Slither.** Slither adalah framework static analysis open source yang robust, dirancang khusus untuk memindai kerentanan smart contract, dibuat oleh **Trail of Bits**. Slither membantu developer smart contract mengidentifikasi kerentanan di kode mereka **dalam hitungan detik**. Selain deteksi kerentanan, Slither juga memfasilitasi optimasi dan review kode.

Fitur Slither:
- Mendeteksi kode Solidity yang rentan dengan **false positive yang rendah**
- Mengidentifikasi **di mana** kondisi error terjadi di source code
- Mudah diintegrasikan ke continuous integration dan build Truffle
- **Detector API** untuk menulis analisis kustom dengan Python
- Mampu menganalisis contract yang ditulis dengan Solidity >= 0.4
- **SlithIR** (intermediate representation) memungkinkan analisis yang sederhana dan presisi tinggi
- Mem-parsing dengan benar **99,9%** dari seluruh kode Solidity publik
- Waktu eksekusi rata-rata **kurang dari 1 detik per contract**

**Mythril.** Mythril adalah tool analisis keamanan untuk **EVM bytecode**. Dia mendeteksi kerentanan keamanan di smart contract yang dibangun untuk Ethereum, Hedera, Quorum, Vechain, Rootstock, Tron, dan blockchain EVM-compatible lainnya. Dia memakai **symbolic execution, SMT solving, dan taint analysis** untuk mendeteksi berbagai kerentanan.

Perbedaan Mythril dan Slither: **Mythril melakukan static analysis pada bahasa EVM bytecode**. Ini berarti menganalisis kode contract tanpa mengeksekusinya, memungkinkan deteksi kerentanan potensial **lebih awal di siklus pengembangan**.

Fitur kunci lain Mythril adalah kemampuannya melakukan **symbolic execution** — teknik mengeksekusi kode dengan **input simbolik alih-alih data nyata**, yang memungkinkan Mythril **menjelajahi banyak jalur eksekusi** dan menemukan kerentanan yang mungkin tidak terlihat lewat static analysis biasa.

### Fuzzing
**Fuzzing** atau fuzz testing adalah teknik pengujian software otomatis yang **memberikan data yang tidak valid, tidak terduga, atau acak sebagai input** ke sebuah program. Program itu lalu **dipantau** untuk melihat exception seperti crash, gagalnya assertion bawaan kode, atau kebocoran memori potensial.

Kenapa fuzzing sangat relevan untuk smart contract: karena **siapa pun bisa berinteraksi dengan contract di blockchain memakai input acak, tanpa harus punya alasan atau ekspektasi yang valid** — inilah yang disebut *arbitrary byzantine behaviour*.

Dua tool fuzzing smart contract yang populer: **Echidna** dan **Harvey**.

**Echidna** adalah program Haskell yang dirancang untuk fuzzing/property-based testing pada smart contract Ethereum. Dia memakai **kampanye fuzzing berbasis grammar yang canggih**, yang didasarkan pada **ABI contract**, untuk memfalsifikasi predikat yang didefinisikan user atau assertion Solidity.

> [!info] Konteks tambahan (bukan dari slide)
> Bedanya fuzzing dari unit test: unit test menguji **kasus yang kamu pikirkan**. Fuzzing menguji **kasus yang gak kamu pikirkan** — dan justru di situlah bug-nya hidup. Kamu gak nulis test "gimana kalau ada yang deposit 0 ether 10.000 kali berturut-turut", tapi fuzzer akan coba, karena dia gak punya asumsi soal apa yang "masuk akal".

## Diagram & Visual
- **Slide 12 — pengenalan Slither**
  ![[99-Assets/Blockchain/W08-slide12.png]]
- **Slide 14 — kode contract `FibonacciLib` dan `FibonacciBalance` untuk hands-on Slither**
  ![[99-Assets/Blockchain/W08-slide14.png]]
  ![[99-Assets/Blockchain/W08-slide14b.png]]
- **Slide 15 — pengenalan Mythril**
  ![[99-Assets/Blockchain/W08-slide15.png]]
- **Slide 18 — tampilan repositori Echidna di GitHub**
  ![[99-Assets/Blockchain/W08-slide18.png]]
- **Slide 19 — kode contract `Incrementor` untuk hands-on Echidna**
  ![[99-Assets/Blockchain/W08-slide19.png]]
- **Slide 20 — tampilan Damn Vulnerable DeFi (CTF showdown)**
  ![[99-Assets/Blockchain/W08-slide20.png]]

> [!warning] **Slide 8 (Severity Matrix) tidak berhasil diekstrak, baik sebagai teks maupun gambar.** Padahal itu matriks Likelihood × Impact = Severity, yang kemungkinan besar keluar di ujian. **Buka PPT aslinya di slide 8.** Kode `FibonacciLib`, `FibonacciBalance`, dan `Incrementor` juga berupa gambar, gak bisa di-copy dari note ini.

## Hands-on
1. **Slither** (slide 14) — install Slither, lalu jalankan untuk mengidentifikasi kerentanan dari contract `FibonacciLib` dan `FibonacciBalance`. Referensi dari speaker notes dosen: `https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#fib_balance_security`
2. **Echidna** (slide 19) — install Echidna, lalu jalankan untuk mengidentifikasi kerentanan dari contract `Incrementor`.
3. **CTF Showdown** (slide 20) — kerjakan tiga tantangan di `https://www.damnvulnerabledefi.xyz/`:
   - **Unstoppable**
   - **Naïve Receiver**
   - **Truster**

## Pertanyaan Terbuka
- **Severity Matrix di slide 8 gak terekstrak sama sekali.** Ini bagian paling mungkin jadi soal (kombinasi Likelihood × Impact), dan isinya gak ada di note ini. Prioritas untuk dibuka manual.
- Slide menjelaskan Slither dan Mythril secara detail, tapi **gak ada satu pun contoh perintah CLI**-nya. Hands-on-nya bilang "install and run" tanpa menunjukkan caranya. Perlu dicari sendiri atau ditanya di kelas.
- **Harvey disebut sebagai tool fuzzing populer tapi sama sekali gak dijelaskan** — cuma namanya yang muncul.
- Tiga tantangan Damn Vulnerable DeFi (Unstoppable, Naïve Receiver, Truster) semuanya berbasis **flash loan**, konsep yang **belum pernah dibahas** di matkul ini sama sekali. Perlu belajar mandiri sebelum mengerjakan.
- Speaker notes di slide 14, 19, dan 20 semuanya menunjuk ke **URL yang sama** (`#fib_balance_security`), padahal isinya beda-beda. Kemungkinan besar dosen lupa mengganti link untuk slide 19 dan 20.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Blockchain]]
- [[W07 - Smart Contract Pitfalls]]
- [[W09 - Audit Findings]]
- [[W05 - Smart Contract Standard]]
- [[Blockchain - Review dan Glosari]]
