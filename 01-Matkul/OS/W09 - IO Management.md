---
matkul: Operating Systems
minggu: 9
sks: 3
sumber: Week9-Pert9-IOManagement.pptx
tags: [kuliah/os, minggu/w09]
status: draft
diproses: 2026-09-04
---

# W09 — I/O Management

## Ringkasan
> - I/O adalah salah satu titik BOTTLENECK terbesar performa sistem — perangkat I/O JAUH lebih lambat dibanding main memory dan processor.
> - Tiga teknik menjalankan I/O: **Programmed I/O** (CPU busy-wait sampai I/O selesai, boros CPU), **Interrupt-driven I/O** (CPU lanjut kerja lain, diberitahu lewat interrupt saat I/O selesai), dan **DMA (Direct Memory Access)** (modul DMA mengelola transfer data LANGSUNG antara memori dan I/O, TANPA melibatkan CPU sama sekali per byte-nya).
> - **Disk access time** = **seek time** (waktu head bergerak ke track) + **rotational delay** (waktu sektor berputar sampai di bawah head).
> - Banyak algoritma **disk scheduling** untuk meminimalkan seek time: **FIFO** (adil tapi lambat), **SSTF** (pilih request terdekat, tapi bisa starvation), **SCAN/elevator** (bergerak satu arah lalu balik), **C-SCAN**, **N-Step SCAN**, **F-SCAN**.
> - **RAID (Redundant Array of Independent Disks)** = strategi memakai BANYAK disk sekaligus untuk meningkatkan performa I/O DAN menyediakan redundansi (lewat parity) untuk pemulihan data kalau satu disk rusak.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Programmed I/O | CPU busy-wait menunggu operasi I/O selesai sebelum lanjut |
| Interrupt-driven I/O | CPU lanjut kerja lain, diberitahu lewat interrupt saat I/O selesai |
| DMA (Direct Memory Access) | Modul khusus yang mengelola transfer data memori-I/O tanpa membebani CPU per byte |
| Seek time | Waktu head disk bergerak ke track yang dituju |
| Rotational delay | Waktu sektor yang dituju berputar sampai berada di bawah head |
| Disk scheduling | Algoritma menentukan urutan pemrosesan request I/O disk untuk minimalkan seek time |
| RAID | Redundant Array of Independent Disks — kombinasi banyak disk untuk performa dan redundansi |

## Isi

### Kategori Perangkat I/O
Perangkat eksternal yang berinteraksi I/O dengan sistem komputer dikelompokkan jadi TIGA kategori (human-readable seperti monitor/keyboard, machine-readable seperti sensor/disk, dan communication seperti modem/network card). Perangkat berbeda dalam banyak hal: **data rate, aplikasi, kompleksitas kontrol, unit transfer, representasi data,** dan **kondisi error.**

### Device Controller
Sebuah I/O device terdiri dari **2 komponen**: bagian **mekanis** (hardware fisik) dan bagian **elektrik** — **device controller/adapter**. Tugas controller: mengonversi ALIRAN BIT serial jadi BLOK BYTE, dan melakukan koreksi error yang diperlukan.

### Memory-Mapped I/O
Setiap controller punya beberapa REGISTER untuk berkomunikasi dengan CPU. Salah satu implementasi: **memetakan control register ke RUANG MEMORI** (memory-mapped I/O). Ada tiga varian: **(a) Separate I/O and memory space** (ruang I/O dan memori terpisah total), **(b) Memory-mapped I/O** (register I/O "hidup" di ruang alamat memori yang sama), **(c) Hybrid** (kombinasi keduanya).

### Prinsip I/O Software
1. **Efficiency** — usaha UTAMA dalam desain I/O. Penting karena operasi I/O sering jadi BOTTLENECK — sebagian besar perangkat I/O SANGAT LAMBAT dibanding main memory dan processor. Area yang paling banyak diperhatikan: **disk I/O.**
2. **Generality** — idealnya menangani SEMUA perangkat secara SERAGAM. Berlaku untuk cara proses "melihat" perangkat I/O dan cara OS mengelola perangkat/operasi I/O. Keberagaman perangkat membuat generalitas SEJATI sulit dicapai — solusinya memakai pendekatan HIERARKIS dan MODULAR untuk desain fungsi I/O.

### Tiga Teknik Menjalankan I/O
1. **Programmed I/O** — processor mengeluarkan perintah I/O atas nama sebuah proses ke modul I/O; proses itu kemudian BUSY WAIT sampai operasi selesai sebelum lanjut. **Boros CPU** karena CPU terus "menunggu aktif".

2. **Interrupt-Driven I/O** — processor mengeluarkan perintah I/O atas nama proses. Kalau NON-BLOCKING, processor LANJUT mengeksekusi instruksi dari proses yang mengeluarkan perintah I/O tadi. Kalau BLOCKING, instruksi berikutnya yang dieksekusi processor berasal dari OS, yang akan menaruh proses saat ini ke state blocked dan menjadwalkan proses lain.

3. **Direct Memory Access (DMA)** — modul DMA mengontrol PERTUKARAN data antara main memory dan modul I/O, TANPA melibatkan CPU untuk setiap byte data yang ditransfer — CPU cukup memberi instruksi awal (misal alamat, jumlah data), lalu DMA yang mengerjakan transfer secara independen, memberi tahu CPU lewat interrupt setelah SELESAI.

> [!info] Analogi
> Bandingkan tiga teknik ini dengan cara mengantar paket. **Programmed I/O** itu seperti kamu BERDIRI DI DEPAN PINTU menunggu kurir datang, tidak melakukan apa pun lain sampai paket diterima — boros waktu. **Interrupt-driven I/O** itu seperti kamu lanjut kerja di dalam rumah, dan BEL PINTU akan berbunyi (interrupt) saat kurir tiba — kamu tidak perlu menunggu di depan pintu terus-menerus. **DMA** itu seperti menyewa ASISTEN KHUSUS yang menangani SELURUH proses terima-taruh paket ke gudang TANPA kamu terlibat sama sekali — kamu cuma diberi tahu SETELAH semuanya beres, kamu bahkan tidak perlu buka pintu sendiri.

### I/O Software Layers dan Device Independence
I/O software disusun berlapis (Layers of the I/O system), masing-masing lapisan punya fungsi utamanya sendiri — dari **device driver** (paling dekat hardware) sampai lapisan yang dilihat langsung oleh **user process** (paling jauh dari hardware). Komunikasi antara driver dan device controller sebenarnya SELALU lewat BUS, meski secara logis digambarkan berlapis.

**Device Independent I/O Software** menyediakan **Uniform Interface** — TANPA standar driver interface, tiap aplikasi harus tahu detail SPESIFIK tiap device; DENGAN standar driver interface, aplikasi cukup memakai interface SERAGAM tanpa peduli detail hardware di baliknya.

**Buffering** — empat pendekatan berbeda menyimpan data sementara selama transfer I/O: **(a) Unbuffered input** (tanpa buffer sama sekali), **(b) Buffering di user space**, **(c) Buffering di kernel diikuti copy ke user space**, **(d) Double buffering di kernel** (dua buffer bergantian, satu diisi sementara satu lagi diproses — meningkatkan throughput).

### Disk Geometry dan Struktur
Disk fisik punya **geometri** tertentu (bisa punya beberapa "zona" dengan geometri fisik berbeda, dipetakan ke "geometri virtual" yang seragam untuk kemudahan OS). Konsep terkait: **sector** (unit penyimpanan terkecil di disk), **cylinder** (kumpulan track sejajar di semua platter), dan **cylinder skew** (offset sektor antar track untuk mengoptimalkan waktu akses saat head pindah track). **Interleaving** — cara menyusun urutan nomor sektor fisik (no interleaving, single interleaving, double interleaving) untuk mengoptimalkan kecepatan baca berurutan.

### Parameter Performa Disk
Detail aktual operasi disk I/O bergantung pada sistem komputer, OS, dan sifat I/O channel serta hardware disk controller.

Saat disk drive beroperasi, disk berputar dengan kecepatan KONSTAN. Untuk membaca/menulis, head harus diposisikan di TRACK yang diinginkan dan di AWAL sektor yang diinginkan di track itu.
- **Seek time** — waktu yang dibutuhkan MEMINDAHKAN lengan disk ke track yang dibutuhkan. Terdiri dari dua komponen: **initial startup time** (waktu mulai bergerak) dan waktu MELINTASI track-track yang harus dilewati setelah lengan mencapai kecepatan penuh. Ada juga **settling time** — waktu setelah memposisikan head di atas track target sampai identifikasi track TERKONFIRMASI. Perbaikan besar datang dari komponen disk yang lebih KECIL dan RINGAN. Rata-rata seek time disk modern: DI BAWAH 10ms.
- **Rotational delay** — waktu yang dibutuhkan AREA yang dialamati untuk berputar sampai bisa diakses head baca/tulis. Disk berputar dari kecepatan 3.600 rpm (perangkat genggam seperti kamera digital) sampai 15.000 rpm.
- **Access time = seek time + rotational delay.**

### Algoritma Disk Scheduling
1. **First In First Out (FIFO)** — proses request secara BERURUTAN sesuai kedatangan. ADIL untuk semua proses. Mendekati performa penjadwalan ACAK kalau banyak proses bersaing memakai disk.

2. **Shortest Service Time First (SSTF)** — memilih request I/O disk yang butuh PERGERAKAN PALING SEDIKIT dari posisi lengan saat ini. Selalu memilih seek time MINIMUM. Risiko: request yang JAUH bisa terus "kalah" (starvation) kalau terus ada request baru yang lebih dekat.

3. **SCAN (Elevator Algorithm)** — lengan bergerak HANYA SATU ARAH, memenuhi SEMUA request tertunda sampai mencapai track TERAKHIR di arah itu, lalu arah DIBALIK. Menguntungkan job yang requestnya di track PALING DALAM dan PALING LUAR, serta job yang datang PALING BARU.

4. **C-SCAN (Circular Scan)** — membatasi scanning HANYA SATU ARAH. Setelah track terakhir dikunjungi di satu arah, lengan dikembalikan ke UJUNG LAWAN disk dan scan dimulai LAGI dari sana (tanpa memproses saat kembali) — memberikan waktu tunggu yang lebih SERAGAM dibanding SCAN biasa.

5. **N-Step SCAN** — membagi antrean request disk jadi SUB-ANTREAN dengan panjang N. Sub-antrean diproses SATU PER SATU memakai SCAN. Selama satu antrean diproses, request BARU harus ditambahkan ke antrean LAIN. Kalau request kurang dari N di akhir sebuah scan, semuanya diproses di scan berikutnya.

6. **F-SCAN** — memakai DUA sub-antrean. Saat scan dimulai, SEMUA request ada di SATU antrean, yang lain KOSONG. Selama scan berjalan, SEMUA request baru ditaruh di antrean LAIN. Servis request baru DITUNDA sampai SEMUA request lama selesai diproses.

> [!info] Analogi
> Kelima algoritma ini seperti strategi berbeda seorang tukang pos mengantar surat di jalan LURUS satu arah dengan banyak rumah. **FIFO** itu antar surat sesuai urutan DITERIMA di kantor pos (bisa bolak-balik jalan tidak efisien). **SSTF** itu selalu antar ke rumah TERDEKAT dari posisi sekarang (efisien tapi rumah jauh bisa TERLUPAKAN terus-menerus). **SCAN/Elevator** itu jalan TERUS SATU ARAH sampai ujung jalan, antar SEMUA surat yang searah, baru balik arah (seperti lift yang berhenti di semua lantai searah sebelum balik). **C-SCAN** itu SELALU antar searah SAJA, lalu balik ke ujung TANPA antar apa pun saat perjalanan pulang, baru mulai lagi dari situ — lebih ADIL waktu tunggunya.

### Handling Bad Sector
Slide menunjukkan tiga cara menangani SEKTOR RUSAK di disk: **(a)** track dengan sektor rusak, **(b)** substitusi sektor CADANGAN untuk menggantikan yang rusak, **(c)** MENGGESER semua sektor untuk melewati yang rusak.

### RAID (Redundant Array of Independent Disks)
**RAID** terdiri dari TUJUH level, dari 0 sampai 6. Istilah ini pertama dicetuskan dalam paper oleh sekelompok peneliti di **University of California, Berkeley**. Paper itu menjabarkan berbagai konfigurasi dan aplikasi, memperkenalkan definisi level-level RAID.

Strategi RAID memakai BANYAK disk drive dan mendistribusikan data sedemikian rupa untuk memungkinkan akses SIMULTAN ke data dari beberapa drive sekaligus. Meningkatkan performa I/O dan memudahkan peningkatan kapasitas SECARA BERTAHAP. Kontribusi UNIKNYA adalah mengatasi kebutuhan REDUNDANSI secara efektif — memanfaatkan informasi PARITY yang disimpan, memungkinkan PEMULIHAN data yang hilang akibat kegagalan disk.

> [!info] Konteks tambahan (bukan dari slide)
> Level RAID yang paling sering disebut dalam praktik: **RAID 0** (striping murni, cepat tapi TANPA redundansi — satu disk rusak = semua data hilang), **RAID 1** (mirroring, data disalin identik ke dua disk — redundan tapi kapasitas efektif cuma setengah), **RAID 5** (striping dengan parity terdistribusi — keseimbangan performa dan redundansi yang populer). Slide ini tidak menjelaskan detail per-level, jadi kalau butuh detail spesifik tiap level untuk ujian, sebaiknya dicek dari gambar slide 40 atau sumber tambahan.

### Clock: Programmable Clock dan Simulasi Timer
**Programmable clock** adalah komponen hardware yang bisa diprogram untuk menghasilkan interrupt pada interval tertentu. OS bisa **mensimulasikan banyak timer** memakai SATU clock fisik saja — dengan menjadwalkan interrupt clock berikutnya berdasarkan timer VIRTUAL yang paling dekat waktu kedaluwarsanya.

### UNIX Buffer Cache
Pada dasarnya adalah DISK CACHE. Operasi I/O dengan disk ditangani lewat buffer cache. Transfer data antara buffer cache dan ruang proses user SELALU memakai DMA — TIDAK memakai siklus processor sama sekali, tapi TETAP memakai siklus bus.

**Tiga daftar yang dijaga:**
- **Free list** — daftar semua slot di cache yang TERSEDIA untuk dialokasikan.
- **Device list** — daftar semua buffer yang saat ini TERASOSIASI dengan tiap disk.
- **Driver I/O queue** — daftar buffer yang SEDANG mengalami atau MENUNGGU I/O di device tertentu.

### Linux I/O dan Disk Scheduler
Linux I/O SANGAT MIRIP implementasi UNIX lain. Mengasosiasikan sebuah SPECIAL FILE dengan tiap I/O device driver. Perangkat **block, character,** dan **network** dikenali. Default disk scheduler di Linux 2.4: **Linux Elevator.**

**The Elevator Scheduler** — menjaga SATU antrean untuk request baca dan tulis disk, melakukan fungsi SORTING dan MERGING pada antrean. Saat request baru ditambahkan, EMPAT operasi dipertimbangkan berurutan:
1. Kalau request menuju sektor disk yang SAMA atau BERDEKATAN dengan request tertunda di antrean, request lama dan baru DIGABUNG (merge) jadi satu request.
2. Kalau ada request di antrean yang CUKUP LAMA, request baru disisipkan di EKOR antrean.
3. Kalau ada lokasi yang SESUAI, request baru disisipkan secara TERURUT.
4. Kalau tidak ada lokasi yang sesuai, request baru ditaruh di EKOR antrean.

**Deadline Scheduler** — mengatasi dua masalah skema elevator: **(1)** request blok yang JAUH bisa tertunda SANGAT LAMA karena antrean terus diupdate secara dinamis; **(2)** aliran request TULIS bisa memblokir request BACA cukup lama, memblokir proses. Dikembangkan 2002, memakai DUA PASANG antrean — selain masuk antrean elevator terurut seperti biasa, request yang sama JUGA ditaruh di EKOR antrean FIFO khusus (FIFO baca atau FIFO tulis). Saat item di kepala salah satu antrean FIFO SUDAH MELEWATI batas waktu kedaluwarsanya, scheduler MEMPRIORITASKAN antrean FIFO itu — mengambil request kedaluwarsa PLUS beberapa request berikutnya dari antrean itu.

**Anticipatory I/O Scheduler** — elevator dan deadline scheduling bisa KONTRAPRODUKTIF kalau ada BANYAK request baca SINKRON. Di Linux, anticipatory scheduler diTUMPANGKAN di atas deadline scheduler. Saat request baca DIKIRIM, anticipatory scheduler membuat sistem SENGAJA MENUNDA — karena ada kemungkinan BESAR aplikasi yang mengirim request baca terakhir akan mengirim request baca LAIN ke region disk yang SAMA. Request itu akan DILAYANI SEGERA. Kalau tidak, scheduler kembali memakai algoritma deadline scheduling.

## Diagram & Visual
- **Slide 7 — Tipikal Data Rate Device, Network, dan Bus**
  ![[99-Assets/OS/W09-slide07.jpg]]
- **Slide 9 — Memory Mapped I/O**
  ![[99-Assets/OS/W09-slide09.jpg]]
- **Slide 13 — Operasi DMA Transfer**
  ![[99-Assets/OS/W09-slide13.jpg]]
- **Slide 14-16 — Interrupt Structure dan Contoh Programmed/Interrupt-driven I/O**
  ![[99-Assets/OS/W09-slide14.jpg]]
  ![[99-Assets/OS/W09-slide15.jpg]]
  ![[99-Assets/OS/W09-slide16.jpg]]
- **Slide 17-21 — I/O Software Layers dan Uniform Interface**
  ![[99-Assets/OS/W09-slide17.jpg]]
  ![[99-Assets/OS/W09-slide18.jpg]]
  ![[99-Assets/OS/W09-slide19.jpg]]
  ![[99-Assets/OS/W09-slide20.jpg]]
  ![[99-Assets/OS/W09-slide21.jpg]]
- **Slide 22 — Empat Pendekatan Buffering**
  ![[99-Assets/OS/W09-slide22.jpg]]
- **Slide 23-26 — Disk Geometry, Sector, Cylinder, Interleaving**
  ![[99-Assets/OS/W09-slide23.jpg]]
  ![[99-Assets/OS/W09-slide24.jpg]]
  ![[99-Assets/OS/W09-slide25.jpg]]
  ![[99-Assets/OS/W09-slide26.jpg]]
- **Slide 37 — Handling Bad Sector**
  ![[99-Assets/OS/W09-slide37.jpg]]
- **Slide 40 — Level-level RAID**
  ![[99-Assets/OS/W09-slide40.png]]
  ![[99-Assets/OS/W09-slide40a.jpg]]
- **Slide 41-42 — Programmable Clock dan Simulasi Timer**
  ![[99-Assets/OS/W09-slide41.jpg]]
  ![[99-Assets/OS/W09-slide42.jpg]]

## Rumus / Sintaks
```
Access time = Seek time + Rotational delay

Disk Scheduling Algorithms:
FIFO       -> urut kedatangan, adil tapi bisa lambat
SSTF       -> pilih seek time minimum, cepat tapi rawan starvation
SCAN       -> satu arah sampai ujung, lalu balik (elevator)
C-SCAN     -> satu arah saja, balik tanpa proses, mulai lagi
N-Step SCAN -> proses per sub-antrean ukuran N pakai SCAN
F-SCAN     -> dua sub-antrean, request baru ditunda sampai lama selesai
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Bottleneck** | Titik yang membatasi performa keseluruhan sistem karena jadi hambatan utama |
| **Parity (RAID)** | Data tambahan yang dihitung dari data asli, dipakai memulihkan data yang hilang |
| **Striping (RAID)** | Membagi data ke beberapa disk sekaligus untuk meningkatkan throughput |
| **Cylinder skew** | Offset posisi sektor antar track untuk optimasi waktu akses saat pindah track |
| **Special file (Linux)** | File yang merepresentasikan device driver di UNIX/Linux |

## Pertanyaan Terbuka
- Slide 38-40 menyebut RAID punya 7 level (0-6) tapi tidak menjelaskan detail karakteristik SETIAP level secara tekstual (hanya lewat gambar/tabel di slide 40) — perlu dibuka manual untuk detail lengkap tiap level kalau dibutuhkan untuk ujian.
- Slide 48-50 (Windows asynchronous/synchronous I/O, I/O completion techniques, Windows RAID configuration) hanya berupa judul dan poin tanpa detail teks yang bisa diekstrak — perlu dibuka manual.

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa DMA dianggap LEBIH EFISIEN dibanding Interrupt-driven I/O untuk transfer data BERUKURAN BESAR (misalnya membaca file besar dari disk). Hubungkan dengan fakta bahwa Interrupt-driven I/O tetap butuh CPU untuk MEMINDAHKAN data byte demi byte, sementara DMA tidak.
2. **(C4 – Analisis)** Bandingkan SSTF dan SCAN dari sisi RISIKO STARVATION. Analisis: kenapa SCAN (elevator) TIDAK punya masalah starvation separah SSTF, meski keduanya sama-sama mengutamakan request yang "dekat" dengan posisi head saat ini?
3. **(C5 – Evaluasi)** Sebuah server database butuh LATENSI RENDAH untuk operasi BACA (read-heavy workload), tapi Linux Elevator Scheduler standar bisa membuat request baca tertunda lama kalau ada banyak request tulis. Evaluasi: kenapa Deadline Scheduler lebih cocok untuk kasus ini dibanding Elevator Scheduler biasa?
4. **(C5 – Evaluasi)** Bandingkan RAID 0 (striping tanpa redundansi) dan RAID 1 (mirroring) dari sisi TRADE-OFF kapasitas vs keamanan data. Evaluasi: untuk sistem yang menyimpan data KRITIS (misalnya data transaksi keuangan), mana yang lebih tepat, dan kenapa RAID 0 SANGAT BERISIKO untuk kasus ini meski performanya lebih cepat?
5. **(C6 – Cipta)** Rancang skenario (fiktif, untuk latihan) di mana sebuah aplikasi streaming video BUTUH baca sekuensial data BESAR dari disk secara terus-menerus. Usulkan kombinasi teknik I/O (Programmed/Interrupt-driven/DMA) dan algoritma disk scheduling yang PALING SESUAI untuk skenario ini, dengan alasan konkret berdasarkan karakteristik masing-masing teknik yang sudah dipelajari.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W08 - File Systems]]
- [[W10 - Memory Management]]
- [[OS - Review dan Glosari]]
