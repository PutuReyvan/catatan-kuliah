---
matkul: Forensics
minggu: 13
sks: 2
sumber: 13 Network Analysis.pptx
tags: [kuliah/forensics, minggu/w13]
status: draft
diproses: 2026-09-03
---

# W13 — Network Analysis

## Ringkasan
> - Sikap dasar yang dianjurkan: bersiaplah dalam kerangka **"kapan" terjadi intrusi, bukan "kalau"**. Berasumsi kamu bisa menahan setiap hacker yang bertekad itu **tidak realistis** — tapi itu **bukan** berarti pertahanan perimeter boleh seadanya.
> - **Protocol** = bahasa bersama, seperangkat aturan komunikasi jaringan. **TCP/IP** adalah yang dipakai internet.
> - Jenis jaringan: **LAN, WAN**, plus **MAN, PAN, CAN, GAN**. Konfigurasi lain: **P2P**, yang mayoritas dipakai untuk file sharing — termasuk konten bajakan dan child pornography.
> - Tool pertahanan: **Firewall** (filter trafik masuk dan keluar) dan **NIDS** (contohnya **Snort** — sniffer yang mengawasi jaringan real-time dan memicu alert).
> - Empat serangan yang disebut: **DDoS, IP Spoofing, Man-in-the-Middle, Social Engineering**.
> - **Ancaman dari dalam punya keunggulan signifikan** karena bisa melewati banyak pengamanan yang sudah dipasang.
> - **Empat tahap incident response**: Preparation → Detection and Analysis → Containment, Eradication and Recovery → Post Incident Activity.
> - **Wireshark** untuk capture, **PcapXray** untuk memvisualkan trafik. **PCAP menangkap data dari OSI Layer 2–7.**

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Social engineering | Membujuk user berwenang agar membocorkan informasi sensitif |
| Protocol | Bahasa bersama / seperangkat aturan komunikasi jaringan |
| TCP/IP | Protokol jaringan yang dipakai internet |
| P2P | Peer-to-peer; semua mesin berfungsi sebagai klien sekaligus server |
| LAN / WAN | Local Area Network / Wide Area Network |
| MAN / PAN / CAN / GAN | Metropolitan / Personal / Campus / Global Area Network |
| IPv4 / IPv6 | Dua format alamat IP |
| Firewall | Program di network gateway server yang memfilter trafik masuk dan keluar |
| IDS / NIDS | Intrusion Detection System / Network IDS |
| Snort | NIDS open source terkenal; beroperasi sebagai sniffer real-time |
| DDoS | Distributed Denial of Service |
| IP Spoofing | Memalsukan alamat IP yang valid untuk mendapat akses |
| Man-in-the-Middle | Penyerang menyisipkan diri di antara dua pihak yang berkomunikasi |
| Inside threat | Ancaman yang berasal dari dalam organisasi |
| Incident response | Kemampuan organisasi merespons saat terjadi pelanggaran |
| Wireshark | Tool capture dan analisis paket |
| PCAP / libpcap | API yang menangkap data paket jaringan live dari OSI Layer 2–7 |
| PcapXray | Tool untuk memvisualisasikan trafik dari file PCAP |

## Isi

### Skala masalahnya
Slide membuka dengan kasus nyata:

> **Fidelity National Information Services Inc. (FIS)**, pemroses kartu kredit prabayar di Jacksonville, melaporkan bahwa **sebuah sindikat kriminal internasional mencuri $13 juta dalam satu hari** pada 2011. Pencurian itu diungkap dalam laporan pendapatan kuartal pertama yang dirilis **3 Mei 2011**. Para hacker menjalankan **operasi yang sangat terencana dan terkoordinasi baik**, melibatkan **ATM dari seluruh dunia** beserta **kartu kredit prabayar curian**. *(Krebs)*

### Social engineering
Dalam serangan social engineering, **user yang berwenang dibujuk oleh individu yang tidak berwenang agar membocorkan informasi sensitif**. Serangan yang umum termasuk **hacker yang menyamar sebagai karyawan, pelanggan, atau konsultan keamanan**.

Berbagai serangan ini **juga bisa dilakukan secara kombinasi**, memanfaatkan kerentanan **baik dari teknologinya maupun dari orang yang mengendalikannya**.

### Dasar-dasar jaringan
Menghubungkan komputer punya keunggulan yang jelas — **berbagi sumber daya** dan **kolaborasi** dua di antaranya. Sebuah jaringan punya kebutuhan dasar terlepas dari ukuran atau tujuannya:

1. **Sejenis koneksi** antar komputer atau perangkat — bisa **fisik** (misalnya kabel Ethernet) atau **nirkabel**.
2. **Cara berkomunikasi yang sudah disepakati.** Bahasa bersama atau seperangkat aturan ini dikenal sebagai **protocol**.

**TCP/IP (Transmission Control Protocol/Internet Protocol)** adalah protokol jaringan yang sangat umum dipakai, dan **itu juga yang dipakai di internet**.

**P2P (peer-to-peer)** adalah konfigurasi jaringan lain yang umum dipakai. Sesuai namanya, **semua mesin di jaringan bisa dan memang berfungsi sebagai klien sekaligus server**. Jaringan P2P **jarang dipakai di lingkungan komersial**; **file sharing adalah penggunaan dominannya** — musik, film, dan software adalah file yang paling umum dibagikan.

Slide menambahkan sisi gelapnya: **P2P juga jadi saluran utama tidak hanya untuk musik, video, dan software bajakan, tapi juga child pornography.** Ini masalah besar tidak hanya di Amerika tapi di seluruh dunia.

**Jenis jaringan:**

| Jenis | Cakupan |
| --- | --- |
| **LAN** (Local Area Network) | Umumnya dianggap **jaringan kantor terkecil**; terdiri dari komputer dan perangkat **di satu kantor atau gedung** |
| **WAN** (Wide Area Network) | **Lebih besar**, kadang jauh lebih besar. Terdiri dari **beberapa LAN di lokasi berbeda**, bisa terbentang sangat jauh |
| **MAN, PAN, CAN, GAN** | Metropolitan, Personal, Campus, dan Global Area Network |

Format alamat IP yang dicontohkan slide:

```
IPv4 : 198.122.55.16
IPv6 : 2008:0eb3:29a2:0000:0000:8c1d:0967:7256
```

### Network security tools
Slide menyatakan sikap dasar yang realistis:

> Soal keamanan, pendekatan terbaik (dan paling realistis) adalah **bersiap dalam kerangka "kapan" ada intrusi, bukan "kalau" ada intrusi**. Bekerja dengan asumsi bahwa kamu akan bisa menahan **setiap** hacker yang bertekad itu **tidak realistis**.

Tapi slide segera menutup celah salah tafsirnya:

> Apakah itu berarti organisasi hanya perlu mengambil langkah minimal untuk melindungi jaringannya, dan memfokuskan sumber daya lebih ke respons daripada pencegahan? **Sama sekali tidak.** **Pertahanan perimeter yang kokoh harus selalu dipakai**, yang cakupannya biasanya ditentukan oleh **anggaran dan personel** yang tersedia untuk menjalankannya.

**Firewall** adalah *"sekumpulan program terkait, yang berada di network gateway server, yang melindungi sumber daya jaringan privat dari user jaringan lain"* (TechTarget, 2000). Firewall bertindak sebagai **filter untuk trafik jaringan masuk maupun keluar**, dan **memutuskan apakah trafik boleh lewat setelah memeriksa paket jaringannya dengan cermat**.

**IDS (Intrusion Detection System)** bertujuan **mendeteksi serangan dari luar maupun dari dalam organisasi**. IDS biasanya **memantau jaringan mencari pola serangan jaringan yang dikenali**, serta **aksi dan aktivitas sistem maupun user yang tidak biasa**.

**Snort** adalah **NIDS (network intrusion detection system) open source yang terkenal**. Snort **beroperasi sebagai sniffer**, mengawasi jaringan **secara real time** dan **memicu alert** kalau ada potensi masalah teridentifikasi.

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan dua pendekatan deteksi yang tersirat dalam kalimat IDS: **"pola serangan yang dikenali"** (signature-based — cepat dan akurat, tapi buta terhadap serangan baru) dan **"aktivitas yang tidak biasa"** (anomaly-based — bisa menangkap serangan baru, tapi banyak false positive). Ini kembali ke masalah **periodicity** di [[W12 - Scripting and Automation for Forensic Tasks]]: membedakan "tidak biasa tapi jahat" dari "tidak biasa tapi wajar" adalah kesulitan intinya.

### Network attacks
Slide mencatat bahwa serangan ini **berubah dengan kecepatan luar biasa**, sehingga jadi tekanan konstan bagi industri keamanan. Empat yang disebut:

| Serangan | Cara kerjanya |
| --- | --- |
| **DDoS** (Distributed Denial of Service) | Memakai **komputer terkompromi dalam jumlah masif** untuk menyerang **satu sistem tunggal** |
| **Identity Spoofing (IP Spoofing)** | Penyerang **memalsukan alamat IP yang valid atau "dikenal"** untuk mendapat akses ke jaringan target |
| **Man-In-The-Middle** | Hacker **menyisipkan dirinya di antara kamu** dan orang atau entitas yang sedang kamu ajak berkomunikasi |
| **Social Engineering** | **Salah satu serangan paling efektif** yang tersedia bagi hacker |

### Ancaman dari dalam
Slide menandai satu hal yang sering diabaikan:

> Penting untuk menyadari bahwa ancaman datang **bukan hanya dari luar organisasi, tapi juga dari dalam**. Langkah pencegahan harus memperhitungkan kedua kemungkinan itu. **Ancaman dari dalam punya keunggulan signifikan karena bisa melewati banyak pengamanan yang sudah dipasang.**

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa inside threat sangat merepotkan secara forensik: hampir semua pertahanan yang dibahas di slide ini **menghadap ke luar**. Firewall memeriksa trafik yang melintasi perimeter; orang dalam **sudah ada di dalam perimeter**. Dan aktivitas mereka **terlihat sah**, karena memang mereka berwenang mengakses data itu — yang tidak sah adalah **niatnya**. Itu sebabnya kasus orang dalam lebih bergantung pada **artifact host** (thumb drive yang pernah tersambung di [[W07 - Windows System Artifacts]], arsip RAR untuk staging di [[W09 - File and Archives Analysis]]) daripada pada bukti jaringan.

### Incident response
Organisasi **harus mampu merespons ketika pelanggaran terjadi**. **Punya rencana beserta tool dan personel untuk merespons secara efektif bisa sangat membantu mengurangi kerusakan.**

**Empat tahap utama incident response:**

1. **Preparation**
2. **Detection and Analysis**
3. **Containment, Eradication and Recovery**
4. **Post Incident Activity**

> [!info] Konteks tambahan (bukan dari slide)
> Empat tahap itu adalah **siklus NIST**, bukan garis lurus: *Post Incident Activity* memberi umpan balik kembali ke *Preparation*. Perhatikan juga bahwa **tahap pertama terjadi sebelum ada insiden apa pun** — ini padanan langsung dari **digital forensic readiness** yang disebut sebagai learning outcome di [[W01 - Digital Forensic Fundamental]] (tapi juga tidak dibahas di sana).

### Bukti jaringan dan tantangannya
**Serangan hacker biasanya mengikuti sebuah jalur, baik menuju maupun melalui jaringan target.** Karena itu, **ada potensi untuk menemukan bukti di sepanjang rute itu**. **"Melacak" penyusup** karenanya jadi **langkah kritis** dalam proses menemukan dan mengidentifikasi mereka.

**Tool investigasi jaringan** yang disebut slide:
- **Wireshark** (`www.wireshark.org`)
- **NetIntercept** (Niksun)
- **Netwitness Investigator**
- **Snort** (`www.snort.org`)

**Tantangan investigasi jaringan.** Mengidentifikasi hacker yang bertanggung jawab **sama sekali bukan tugas sederhana**:

- Tersangka bisa **memalsukan (spoof) alamat IP aslinya**, berpotensi **mengirim penyidik mengejar bayangan**.
- Hacker bisa **menyalurkan serangannya melalui banyak server perantara yang tersebar di seluruh dunia**.
- **Log bisa jadi sumber bukti yang hebat, tapi hanya kalau log itu benar-benar ada untuk kita periksa.** **Kadang fungsi logging memang dimatikan sejak awal**, artinya **tidak ada log yang pernah dihasilkan**.

> [!info] Konteks tambahan (bukan dari slide)
> Masalah "rantai server perantara" itu bukan cuma soal teknis — **itu masalah yurisdiksi**. Tiap hop bisa berada di negara berbeda, dan mendapat log dari tiap hop butuh **kerja sama hukum lintas negara** yang bisa memakan waktu berbulan-bulan, sementara log-nya sendiri sudah **roll over dalam 28–30 hari** ([[W06 - Linux System and Artifacts]]). Waktu selalu bekerja melawan penyidik.

### Wireshark dan PCAP
**Wireshark** adalah tool yang **sangat populer dan terkenal** untuk analisis jaringan dan paket serta troubleshooting. Sudah **pre-installed di Kali Linux** dan **relatif mudah dipakai** begitu kamu punya gambaran soal **filter, protokol, dan kode warna**.

Cara memulai capture: **klik ikon sirip hiu biru**, atau lewat **Capture | Start**. Proses capture paket **otomatis dimulai** setelah tombol itu diklik.

Slide mengingatkan bahwa ini baru permulaan:

> Paket yang tertangkap ini **baru langkah awal** untuk melakukan forensik terhadap paket. **Setelah paketnya tertangkap, berikutnya adalah menganalisisnya.**

**PCAP.** **Packet Capture atau PCAP** (dikenal juga sebagai **libpcap**) adalah **application programming interface (API) yang menangkap data paket jaringan live dari OSI model Layer 2–7**. Network analyzer seperti Wireshark **membuat file `.pcap`** untuk mengumpulkan dan merekam data paket dari jaringan.

**Format PCAP** hadir dalam beberapa bentuk: **Libpcap, WinPcap, dan PCAPng**.

**PcapXray** adalah tool yang membantu **memvisualisasikan trafik**. Jenis trafik yang bisa divisualkan:
- **Malicious**
- **Tor**
- **HTTPS**
- **HTTP**
- **DNS**
- **ICMP**

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan bahwa PcapXray bisa **mengidentifikasi trafik HTTPS dan Tor** meskipun **tidak bisa membacanya**. Ini poin penting yang menjawab batasan SSL di [[W10 - Internet Artifacts]]: bahkan ketika isinya terenkripsi, **metadata-nya tetap bicara** — siapa terhubung ke siapa, kapan, berapa lama, seberapa besar. Menemukan **trafik Tor** di jaringan korporat sudah bermakna dengan sendirinya, tanpa perlu tahu isinya apa.

## Diagram & Visual
- **Slide 20 — tampilan Wireshark setelah proses capture paket berjalan**
  ![[99-Assets/Forensics/W13-slide20.png]]

> [!warning] Cuma **satu gambar** dari 23 slide. **Tidak ada screenshot PcapXray**, tidak ada contoh filter Wireshark, tidak ada contoh isi file PCAP. Untuk praktikum, harus dieksplorasi langsung di Kali.

## Rumus / Sintaks

Format alamat IP:
```
IPv4 : 198.122.55.16
IPv6 : 2008:0eb3:29a2:0000:0000:8c1d:0967:7256
```

Empat tahap incident response:
```
1. Preparation
2. Detection and Analysis
3. Containment, Eradication and Recovery
4. Post Incident Activity
```

Empat serangan jaringan yang disebut:
```
DDoS               : banyak komputer terkompromi -> menyerang SATU sistem
IP Spoofing        : memalsukan alamat IP yang dikenal
Man-in-the-Middle  : penyerang menyisip di antara dua pihak
Social Engineering : menyerang MANUSIA, bukan mesin
```

PCAP:
```
menangkap dari OSI Layer 2-7
format: Libpcap, WinPcap, PCAPng
PcapXray memvisualkan: Malicious, Tor, HTTPS, HTTP, DNS, ICMP
```

## Pertanyaan Terbuka
- Slide 3 mencantumkan **sembilan learning outcome**, dan **lima tidak dibahas**: **Protocol Analysis (HTTP, DNS, TLS/SSL)**, **Session Reconstruction from PCAP**, **Indicators of Compromise (IoC) Extraction from Network Traffic**, **Handling and Analyzing Encrypted Traffic**, dan **Correlating Network Artifacts with Host-based Evidence**.

  Yang terakhir itu menonjol — **korelasi bukti jaringan dengan bukti host adalah keterampilan puncak dari seluruh matkul ini**, tempat semua sesi sebelumnya bertemu, dan **tidak dibahas sama sekali**. **Wajib ditanyakan.**
- **"PCAP Analysis" jadi LO tapi yang diajarkan cuma cara meng-capture**, bukan cara menganalisis. Slide 20 sendiri mengakui *"setelah paketnya tertangkap, berikutnya adalah menganalisisnya"* — lalu deck-nya berakhir.
- **Wireshark cuma dibahas dua slide** dan berhenti di "klik ikon sirip hiu". **Tidak ada satu pun display filter** (`http`, `ip.addr ==`, `tcp.port ==`), padahal slide sendiri bilang Wireshark mudah dipakai "begitu kamu punya gambaran soal filter". Gambaran itu tidak pernah diberikan.
- **IoC (Indicators of Compromise)** muncul sebagai LO di sini **dan** di [[W11 - Timeline Analysis and Correlation of Artifacts]], tapi **tidak pernah didefinisikan di deck mana pun** sepanjang matkul.
- Sebagian besar isi deck ini sebenarnya **materi keamanan jaringan umum** (firewall, IDS, jenis serangan, incident response), **bukan network forensic**. Porsi forensik sesungguhnya cuma tiga slide terakhir.
- **NetIntercept dan Netwitness Investigator** cuma disebut sebagai URL tanpa penjelasan; keduanya juga sudah lama tidak ada dalam bentuk itu.
- Contoh kasus **FIS 2011** dan sebagian besar referensi berasal dari 2011. Materi ini jelas sudah lama tidak diperbarui.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W10 - Internet Artifacts]]
- [[W11 - Timeline Analysis and Correlation of Artifacts]]
- [[W12 - Scripting and Automation for Forensic Tasks]]
- [[Forensics - Review dan Glosari]]
