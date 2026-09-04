---
matkul: Operating Systems
minggu: 11
sks: 3
sumber: Week11-Pert11-VirtualMemory.pptx
tags: [kuliah/os, minggu/w11]
status: draft
diproses: 2026-09-04
---

# W11 — Virtual Memory

## Ringkasan
> - **Virtual address** (dibuat program, ada di ruang privat proses) berbeda dari **real address** (lokasi FISIK sesungguhnya) — MMU menerjemahkan dari virtual ke real. Virtual memory berevolusi dari teknik lama **overlay** (program dipecah manual jadi potongan oleh programmer) jadi otomatis lewat **paging**.
> - **Page fault** = CPU merujuk halaman yang page table entry-nya bilang TIDAK ADA di main memory — OS turun tangan mengambilnya dari disk.
> - OS harus buat keputusan di 4 area: **Fetch Policy** (kapan halaman dibawa masuk — Demand Paging vs Pre-paging), **Placement Policy** (di mana ditempatkan), **Replacement Policy** (halaman mana yang DIKELUARKAN kalau memori penuh), **Resident Set Management** (berapa banyak halaman per proses, Global vs Local).
> - Algoritma Page Replacement: **Optimal** (teoretis sempurna, TIDAK BISA diimplementasikan), **LRU** (ganti halaman paling lama TIDAK dipakai, akurat tapi mahal), **FIFO** (sederhana tapi kurang akurat), **Clock** (aproksimasi LRU yang praktis dan murah).
> - **Thrashing** = kondisi sistem SIBUK terus-menerus swap halaman masuk-keluar sampai hampir tidak ada waktu tersisa untuk kerja SUNGGUHAN — biasanya karena terlalu banyak proses berebut memori terbatas.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Virtual address | Alamat memori yang dihasilkan program, ada di ruang privat proses |
| Real/physical address | Lokasi fisik sesungguhnya di RAM |
| Page fault | Event ketika halaman yang diakses ternyata tidak ada di main memory |
| Demand Paging | Halaman dibawa ke memori HANYA saat benar-benar dirujuk |
| Page Replacement | Kebijakan memilih halaman mana yang dikeluarkan saat perlu ruang untuk halaman baru |
| Thrashing | Sistem terus-menerus swap halaman sampai hampir tidak ada kerja produktif |
| Working Set | Kumpulan halaman yang dipakai oleh k referensi memori paling baru |

## Isi

### Real Address vs Virtual Address
- **Real Address** — terletak di physical memory sesungguhnya.
- **Virtual Address** — alamat memori yang DIHASILKAN oleh program, ada di ruang PRIVAT proses.
- **MMU (Memory Management Unit)** menerjemahkan dari virtual address ke real address.

Slide menunjukkan operasi internal MMU dengan 16 halaman 4-KB — hubungan antara alamat virtual dan alamat physical memory diberikan lewat **page table**. Setiap halaman dimulai pada kelipatan 4096 dan berakhir 4095 alamat lebih tinggi (jadi "4K–8K" sebenarnya berarti 4096–8191, dan "8K–12K" berarti 8192–12287).

### Dari Overlay ke Virtual Memory
**Masalah program besar** — kalau program lebih besar dari memori yang tersedia. Solusi tahun 1960-an: memecah program jadi potongan kecil bernama **overlay**. Banyak overlay diizinkan ada di memori sekaligus, dimuat bergantian sesuai kebutuhan — TAPI ini pekerjaan MANUAL yang harus dilakukan programmer.

Konsep overlay kemudian berkembang jadi **virtual memory system**: program dipecah jadi beberapa PAGE, semua halaman berukuran SAMA. Setiap page adalah rentang alamat yang BERSEBELAHAN (contiguous). MMU memetakan alamat virtual ke alamat fisik SECARA OTOMATIS — tidak lagi manual seperti overlay.

> [!info] Analogi
> Overlay itu seperti KAMU HARUS SENDIRI merencanakan pindahan rumah — memutuskan barang mana yang dibawa duluan, mana yang ditaruh di gudang dulu, dan kapan menukarnya, semua secara MANUAL dan direncanakan sebelumnya. Virtual memory (paging) itu seperti punya JASA PINDAHAN OTOMATIS yang PINTAR — kamu tinggal minta barang apa saja kapan saja, dan sistem itu YANG MENGATUR sendiri kapan barang dari gudang perlu dibawa masuk, tanpa kamu perlu merencanakan detail teknisnya.

### Eksekusi Proses dengan Virtual Memory
Ketika bagian proses yang berisi alamat logis DIBUTUHKAN, ia dibawa ke main memory. OS mengeluarkan permintaan **disk I/O Read**. Proses LAIN dijadwalkan berjalan SEMENTARA disk I/O berlangsung. Sebuah interrupt dikeluarkan saat disk I/O SELESAI, yang membuat OS menaruh proses yang terpengaruh ke state Ready.

### Implikasi Virtual Memory
1. **Lebih banyak proses bisa dijaga di main memory** — karena HANYA sebagian potongan proses tertentu yang dimuat, ada RUANG untuk lebih banyak proses. Ini membuat utilisasi processor LEBIH EFISIEN karena lebih mungkin setidaknya SATU dari proses-proses yang lebih banyak itu berada di state Ready kapan pun.
2. **Sebuah proses BISA lebih besar dari SELURUH main memory** — kalau program yang ditulis TERLALU BESAR, dulu programmer harus merancang cara menstruktur program jadi potongan yang bisa dimuat terpisah lewat strategi overlay. Dengan virtual memory berbasis paging/segmentation, tugas ITU diserahkan ke OS dan hardware — OS SECARA OTOMATIS memuat potongan proses ke main memory sesuai kebutuhan.

### Kebijakan OS untuk Virtual Memory
OS harus mengambil keputusan di LIMA area kebijakan:
1. **Fetch Policy** — menentukan KAPAN sebuah halaman harus dibawa ke memori. Dua pendekatan: **Demand Paging** dan **Pre-paging**.
2. **Placement Policy** — menentukan DI MANA di real memory sebuah potongan proses akan ditempatkan.
3. **Replacement Policy** — algoritma DASAR: Optimal, LRU, FIFO, Clock.
4. **Resident Set Management** — ukuran resident set (Fixed/Variable), cakupan resident set (Global/Local).
5. **Cleaning Policy** — kapan halaman yang dimodifikasi ditulis balik ke disk: Demand atau Pre-Cleaning.
6. **Load Control** — mengontrol derajat multiprogramming untuk mencegah thrashing.

### Fetch Policy
Menentukan KAPAN sebuah halaman harus dibawa ke memori:
- **Demand Paging** — HANYA membawa halaman ke main memory saat ada REFERENSI ke lokasi di halaman itu. Banyak PAGE FAULT terjadi saat proses PERTAMA KALI dimulai. Prinsip locality menunjukkan bahwa seiring lebih banyak halaman dibawa masuk, sebagian besar referensi MASA DEPAN akan merujuk ke halaman yang BARU SAJA dibawa masuk, dan page fault seharusnya TURUN ke level yang sangat rendah.
- **Pre-paging** — halaman LAIN (selain yang diminta page fault) DIBAWA MASUK sekaligus. Memanfaatkan karakteristik sebagian besar perangkat secondary memory. Kalau halaman-halaman proses disimpan SECARA BERSEBELAHAN di secondary memory, lebih EFISIEN membawa masuk BEBERAPA halaman sekaligus. TIDAK EFEKTIF kalau halaman ekstra itu ternyata TIDAK DIRUJUK. Tidak boleh disamakan dengan "swapping".

### Placement Policy
Menentukan DI MANA di real memory sebuah potongan proses akan berada. Isu desain PENTING dalam sistem SEGMENTATION. Untuk paging atau paging-dikombinasikan-segmentation, placement TIDAK RELEVAN karena hardware melakukan fungsinya dengan efisiensi yang SAMA di mana pun ditempatkan. Untuk sistem **NUMA** (ingat [[W04 - Multiple Processor Systems]]), strategi placement OTOMATIS diinginkan (karena akses memori lokal vs remote punya latency berbeda).

### Frame Locking
Ketika sebuah FRAME di-lock, halaman yang saat ini disimpan di frame itu TIDAK BOLEH digantikan. Kernel OS dan struktur kontrol kunci disimpan di frame yang di-lock. Buffer I/O dan area time-critical bisa di-lock ke frame main memory. Locking dicapai dengan mengasosiasikan LOCK BIT ke setiap frame.

### Page Fault
**Page fault** adalah event di mana CPU merujuk sebuah halaman yang entri page table-nya menunjukkan halaman itu TIDAK ADA di main memory. Menyebabkan OS TURUN TANGAN dan membawa halaman itu dari secondary storage (disk) ke RAM.

### Replacement Policy dan Thrashing
**Replacement policy** menangani PEMILIHAN halaman di main memory yang akan DIGANTIKAN saat halaman baru harus dibawa masuk. Tujuannya: halaman yang DIHAPUS adalah halaman yang PALING KECIL kemungkinannya dirujuk lagi dalam waktu dekat. Semakin RUMIT kebijakan replacement, semakin BESAR overhead hardware dan software untuk mengimplementasikannya.

**Thrashing** — kondisi sistem terlalu SIBUK melakukan swap halaman masuk-keluar terus-menerus, sampai hampir TIDAK ADA waktu tersisa untuk kerja produktif sesungguhnya. Biasanya terjadi karena TERLALU BANYAK proses bersaing memperebutkan memori yang TERBATAS.

### Algoritma Basic Page Replacement

1. **Optimal** — memilih untuk digantikan halaman yang WAKTU sampai referensi berikutnya PALING PANJANG. Menghasilkan JUMLAH page fault PALING SEDIKIT. **MUSTAHIL diimplementasikan** (butuh tahu masa depan) — jadi dipakai sebagai TOLOK UKUR TEORETIS untuk membandingkan algoritma lain, bukan dipakai di dunia nyata.

2. **Least Recently Used (LRU)** — mengganti halaman yang TIDAK DIRUJUK untuk waktu PALING LAMA. Berdasarkan prinsip locality, ini SEHARUSNYA halaman yang PALING KECIL kemungkinannya dirujuk dalam waktu dekat. SULIT diimplementasikan — satu pendekatan adalah menandai setiap halaman dengan WAKTU referensi terakhir, yang membutuhkan overhead BESAR.

   *Simulasi LRU (Aging Algorithm)* — mensimulasikan LRU DI SOFTWARE tanpa overhead penuh melacak waktu exact. Contoh: 6 halaman selama 5 clock tick, direpresentasikan (a) sampai (e) — setiap tick, bit "referenced" digeser masuk ke sebuah counter/register per halaman, membentuk pendekatan LRU yang lebih murah.

3. **First-In First-Out (FIFO)** — memperlakukan frame halaman yang dialokasikan ke sebuah proses sebagai BUFFER SIRKULER. Halaman DIHAPUS dengan gaya round-robin. Kebijakan replacement PALING SEDERHANA diimplementasikan. Halaman yang sudah PALING LAMA di memori yang digantikan (TIDAK peduli seberapa sering dipakai — beda dari LRU).

4. **Clock Policy** — membutuhkan asosiasi BIT TAMBAHAN dengan setiap frame, disebut **use bit**. Saat halaman PERTAMA dimuat ke memori atau DIRUJUK, use bit di-set ke 1. Sekumpulan frame diperlakukan sebagai buffer SIRKULER (seperti jarum jam berputar). SETIAP frame dengan use bit 1 DILEWATI oleh algoritma (tapi bit-nya di-reset ke 0 saat dilewati — kesempatan kedua). Frame page divisualisasikan sebagai terlayout melingkar seperti jam.

> [!info] Analogi
> Bandingkan keempat algoritma ini dengan cara membersihkan lemari baju yang penuh. **Optimal** itu seperti tahu PERSIS baju mana yang TIDAK akan kamu pakai lagi paling lama ke depan (mustahil tahu masa depan, tapi kalau BISA tahu, ini strategi terbaik). **LRU** itu buang baju yang PALING LAMA TIDAK PERNAH kamu pakai — masuk akal tapi kamu harus INGAT persis kapan terakhir kali pakai tiap baju (overhead besar). **FIFO** itu buang baju yang PALING LAMA ada di lemari — TIDAK PEDULI apakah kamu sering pakai baju itu atau tidak (bisa salah buang baju favorit yang kebetulan sudah lama dibeli). **Clock** itu seperti cek baju SATU PER SATU berputar, kasih "kesempatan kedua" ke baju yang baru-baru ini dipakai (tandai, lewati), baru buang yang benar-benar TIDAK ditandai — murah tapi cukup akurat.

### Working Set
**Working Set** adalah kumpulan HALAMAN yang dipakai oleh **k** referensi memori PALING BARU. Fungsi $w(k, t)$ adalah UKURAN working set pada waktu $t$. **Working Set Algorithm** memakai konsep ini untuk menentukan berapa BANYAK halaman yang harus dijaga di memori untuk sebuah proses — didefinisikan lewat WINDOW waktu tertentu.

### Global vs Local Replacement Policy
- **Local page replacement** — halaman yang digantikan HANYA dipilih dari halaman milik PROSES ITU SENDIRI.
- **Global page replacement** — halaman yang digantikan bisa dipilih dari halaman milik PROSES MANA PUN yang sedang di memori.

### UNIX Page Replacement
**Page frame data table** dipakai untuk page replacement. Pointer dipakai untuk membentuk LIST di dalam tabel. SEMUA frame yang tersedia dirangkai bersama dalam **list of free frames** untuk membawa masuk halaman. Ketika jumlah frame tersedia turun di bawah THRESHOLD tertentu, kernel akan MENCURI (steal) sejumlah frame untuk mengompensasi.

### Linux Virtual Memory dan Page Replacement
Linux memakai struktur **three-level page table**. Page replacement Linux BERBASIS Clock Algorithm, tapi **use bit DIGANTI dengan variabel AGE 8-bit**. Variabel age BERTAMBAH tiap kali halaman diakses. SECARA PERIODIK, bit age DIKURANGI. Halaman dengan age 0 adalah halaman "TUA" yang belum dirujuk untuk beberapa waktu — kandidat TERBAIK untuk replacement. Ini bentuk kebijakan **least frequently used**.

### Windows Paging
Saat proses dibuat, ia bisa memakai HAMPIR SELURUH user space sebesar 2GB. Ruang ini dibagi jadi halaman berukuran TETAP, dikelola dalam region kontigu yang dialokasikan pada batas 64 KB.

**Resident Set Management System** — Windows memakai **variable allocation, local scope**. Saat diaktifkan, sebuah proses diberi struktur data untuk mengelola working set-nya. Working set proses aktif DISESUAIKAN tergantung ketersediaan main memory.

### Android Memory Management
Android menambahkan beberapa EKSTENSI ke fasilitas manajemen memori Linux kernel standar:
- **ASHMem** — menyediakan anonymous shared memory, mengabstraksikan memori sebagai FILE DESCRIPTOR. File descriptor bisa diteruskan ke proses lain untuk berbagi memori.
- **Pmem** — mengalokasikan virtual memory sehingga secara FISIK BERSEBELAHAN (contiguous). Berguna untuk hardware yang TIDAK mendukung virtual memory.
- **Low Memory Killer** — memungkinkan sistem MEMBERI TAHU aplikasi bahwa mereka perlu membebaskan memori. Kalau aplikasi TIDAK KOOPERATIF, aplikasi itu DITERMINASI.

## Diagram & Visual
- **Slide 6 — Operasi Internal MMU dengan 16 Halaman 4-KB**
  ![[99-Assets/OS/W11-slide06.jpg]]
- **Slide 7 — Address Translation MMU**
  ![[99-Assets/OS/W11-slide07.jpg]]
- **Slide 14 — Operating System Software untuk Virtual Memory**
  ![[99-Assets/OS/W11-slide14.png]]
- **Slide 27 — Simulasi LRU (Aging Algorithm)**
  ![[99-Assets/OS/W11-slide27.jpg]]
- **Slide 30 — Clock Page Replacement Algorithm**
  ![[99-Assets/OS/W11-slide30.jpg]]
- **Slide 31 — Rangkuman Algoritma Page Replacement**
  ![[99-Assets/OS/W11-slide31.png]]
- **Slide 32-34 — Working Set dan Working Set Algorithm**
  ![[99-Assets/OS/W11-slide32.jpg]]
  ![[99-Assets/OS/W11-slide33.jpg]]
  ![[99-Assets/OS/W11-slide34.png]]
- **Slide 35 — Local vs Global Page Replacement**
  ![[99-Assets/OS/W11-slide35.jpg]]
- **Slide 36 — Resident Set**
  ![[99-Assets/OS/W11-slide36.png]]
- **Slide 38 — UNIX Page Replacement**
  ![[99-Assets/OS/W11-slide38.png]]

## Rumus / Sintaks
```
5 Kebijakan OS untuk Virtual Memory:
1. Fetch Policy         -> Demand Paging vs Pre-paging
2. Placement Policy     -> di mana ditempatkan (penting untuk NUMA)
3. Replacement Policy   -> Optimal, LRU, FIFO, Clock
4. Resident Set Mgmt    -> Fixed/Variable size, Global/Local scope
5. Cleaning Policy      -> Demand vs Pre-Cleaning

Page Replacement Algorithms (dari teoretis ke praktis):
Optimal  -> terbaik tapi MUSTAHIL diimplementasikan (butuh tahu masa depan)
LRU      -> akurat tapi overhead besar (perlu tag waktu tiap halaman)
FIFO     -> paling sederhana, tapi kurang akurat
Clock    -> aproksimasi LRU yang murah dan praktis (use bit + circular buffer)
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Use bit** | Bit yang menandai halaman baru dimuat/dirujuk, dipakai algoritma Clock |
| **Resident set** | Kumpulan halaman sebuah proses yang saat ini ada di main memory |
| **File descriptor** | Referensi yang dipakai proses untuk mengakses file/resource I/O |
| **Contiguous (fisik)** | Alamat memori yang berurutan tanpa terputus secara fisik |

## Pertanyaan Terbuka
- Slide 39-42 (Linux Three-Level Page Table, Windows Paging region states) hanya berupa poin judul tanpa detail teks lengkap — perlu dibuka manual kalau butuh detail untuk ujian.

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa algoritma Optimal "mustahil diimplementasikan" di dunia nyata, tapi TETAP dipelajari dan dipakai sebagai referensi. Apa manfaat praktis dari mempelajari algoritma yang tidak bisa dipakai langsung ini?
2. **(C4 – Analisis)** Bandingkan LRU dan Clock Algorithm. Analisis: bagaimana Clock Algorithm mencapai APROKSIMASI LRU yang "cukup baik" dengan overhead JAUH lebih rendah — hubungkan dengan mekanisme use bit dan circular buffer.
3. **(C5 – Evaluasi)** Sebuah sistem mengalami THRASHING parah karena terlalu banyak proses berjalan bersamaan dengan memori terbatas. Evaluasi: kebijakan MANA (dari 5 kebijakan OS: Fetch, Placement, Replacement, Resident Set, Cleaning) yang PALING LANGSUNG relevan untuk mengatasi thrashing, dan bagaimana Load Control bisa membantu?
4. **(C5 – Evaluasi)** Bandingkan Global dan Local page replacement policy. Evaluasi: skenario APA yang membuat Global policy BERISIKO — misalnya satu proses yang "rakus" bisa mengambil halaman dari proses LAIN yang justru sedang butuh halamannya. Apa keuntungan Local policy dalam mencegah masalah ini?
5. **(C6 – Cipta)** Diberikan urutan referensi halaman: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5 dengan 3 frame yang tersedia. Simulasikan algoritma FIFO dan hitung jumlah page fault yang terjadi. Bandingkan dengan simulasi LRU untuk urutan yang sama, dan simpulkan mana yang menghasilkan LEBIH SEDIKIT page fault untuk kasus spesifik ini.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W10 - Memory Management]]
- [[W12 - Security]]
- [[OS - Review dan Glosari]]
