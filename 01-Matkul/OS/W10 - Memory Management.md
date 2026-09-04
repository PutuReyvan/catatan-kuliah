---
matkul: Operating Systems
minggu: 10
sks: 3
sumber: Week10-Pert10-MemoryManagement.pptx
tags: [kuliah/os, minggu/w10]
status: draft
diproses: 2026-09-04
---

# W10 — Memory Management

## Ringkasan
> - Memori berevolusi dari **NO abstraction** (cuma satu program di memori sekaligus, komputer mini/PC awal) ke **address space** — abstraksi di mana tiap proses punya "alamat sendiri" yang independen dari proses lain.
> - **Dynamic relocation** memakai **base register** dan **limit register** untuk memberi tiap proses ruang alamat terpisah TANPA harus tahu di mana proses lain berada di memori fisik.
> - **Partitioning** (fixed dan dynamic) adalah cara LAMA mengalokasikan memori TANPA virtual memory — fixed partitioning boros karena INTERNAL FRAGMENTATION, dynamic partitioning lebih efisien tapi butuh PLACEMENT ALGORITHM.
> - **Paging** membagi memori jadi FRAME berukuran tetap, dipetakan lewat **page table** — masalahnya page table bisa JADI SANGAT BESAR, diatasi dengan **hierarchical page table**, **inverted page table**, dan di-cache lewat **TLB (Translation Lookaside Buffer)**.
> - **Segmentation** membagi memori jadi SEGMEN berdasarkan STRUKTUR LOGIS program (bukan ukuran tetap) — lebih natural untuk programmer, bisa DIGABUNG dengan paging (paged segmentation) untuk dapat kelebihan keduanya.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Address space | Kumpulan alamat yang bisa dipakai sebuah proses untuk mengakses memori |
| Base dan Limit register | Register yang mendefinisikan awal dan batas ruang alamat sebuah proses |
| Internal fragmentation | Ruang terbuang karena blok data lebih kecil dari partisi yang dialokasikan |
| Page table | Struktur yang memetakan halaman virtual ke frame memori fisik |
| TLB (Translation Lookaside Buffer) | Cache kecepatan tinggi untuk entri page table yang baru dipakai |
| Segmentation | Membagi memori jadi segmen berdasarkan struktur logis program |

## Isi

### NO Abstraction: Memori Tanpa Abstraksi
Bentuk memori PALING SEDERHANA sama sekali TIDAK PUNYA abstraksi (komputer mini awal, PC awal) — HANYA SATU program di memori pada satu waktu. Slide menunjukkan tiga cara sederhana mengorganisasi memori dengan OS dan satu user process (kemungkinan lain juga ada, tergantung apakah OS di bagian bawah, tengah, atau atas ROM).

**Menjalankan banyak program TANPA abstraksi:** simpan SELURUH isi memori ke file di penyimpanan non-volatile, lalu bawa masuk dan jalankan program berikutnya. Selama HANYA SATU program pada satu waktu di memori, TIDAK ADA konflik.

### Memory Abstraction: Address Space
Abstraksi kunci: **memory address space** — kumpulan alamat yang bisa dipakai SEBUAH PROSES untuk mengakses memori. Setiap proses punya address space-nya SENDIRI, INDEPENDEN dari proses lain (kecuali dalam keadaan khusus di mana proses SENGAJA ingin berbagi address space-nya).

### Dynamic Relocation
Memakai **base register** dan **limit register** untuk memberi tiap proses ruang alamat TERPISAH. Ilustrasi masalah relokasi: bayangkan program 16-KB dan program 16-KB LAIN yang dimuat BERURUTAN ke memori — tanpa mekanisme relokasi, alamat yang di-hardcode di dalam program kedua bisa BENTROK dengan lokasi memori program pertama. Base register memberi TITIK AWAL alamat fisik untuk sebuah proses; limit register membatasi SEBERAPA JAUH proses itu boleh mengakses memori — mencegah proses mengakses memori proses LAIN.

> [!info] Analogi
> Base dan limit register itu seperti sistem HOTEL dengan banyak kamar. Base register itu seperti NOMOR KAMAR yang diberikan ke tamu (misalnya kamar 301) — tamu itu SELALU merujuk ke ruangannya sebagai "kamar saya", tapi resepsionis (hardware) yang TAHU PERSIS itu sebenarnya kamar 301 di lantai 3. Limit register itu seperti BATAS luas kamar itu — tamu tidak bisa "berjalan" melewati dinding kamarnya ke kamar tamu lain. Program tidak perlu tahu di MANA persis dia ditempatkan secara fisik — cukup pakai alamat RELATIF ("alamat 0" = awal kamarnya sendiri), dan hardware yang menerjemahkan ke alamat fisik SESUNGGUHNYA.

### Swapping
**Swapping** — alokasi memori berubah SEIRING proses masuk dan keluar dari memori (lihat kembali [[W02 - Processes]]). Area yang tidak terpakai (shaded region di diagram) terus BERUBAH bentuk dan ukuran seiring proses datang-pergi.

### Managing Free Memory
Dua cara melacak memori BEBAS (free): sebagian memori dengan lima proses dan tiga "hole" (celah kosong) bisa direpresentasikan sebagai:
- **Bitmap** — vektor bit, di mana region kosong ditandai 0.
- **Linked list** — informasi yang SAMA direpresentasikan sebagai daftar bertaut (mirip konsep Free Space Management di [[W08 - File Systems]], tapi di sini untuk memori bukan disk).

### Address Translation dan MMU
Karena segmen berukuran TIDAK SAMA, TIDAK ADA hubungan sederhana antara alamat LOGIS dan alamat FISIK. **MMU (Memory Management Unit)** — komponen hardware (biasanya SEKARANG jadi bagian dari chip CPU, dulu bisa jadi chip terpisah) yang bertanggung jawab menerjemahkan alamat LOGIS (yang dipakai program) ke alamat FISIK (lokasi sesungguhnya di RAM).

### Memory Partitioning: Tanpa Virtual Memory
**Memory management** membawa proses ke main memory untuk dieksekusi processor. Ada dua pendekatan besar:
- **Melibatkan virtual memory** — berbasis segmentation dan paging (dibahas selanjutnya).
- **Partitioning** — dipakai dalam beberapa variasi di OS yang SUDAH USANG (obsolete), TIDAK melibatkan virtual memory.

**Fixed Partitioning — Kekurangan:**
- Program mungkin terlalu BESAR untuk muat di satu partisi — perlu didesain memakai OVERLAY.
- Utilisasi main memory TIDAK EFISIEN — program APA PUN, berapa pun ukurannya, menempati SELURUH partisi.
- **Internal fragmentation** — ruang terbuang karena blok data yang dimuat LEBIH KECIL dari partisi.
- Jumlah partisi yang ditentukan saat SYSTEM GENERATION membatasi jumlah proses AKTIF di sistem.
- Job kecil TIDAK memanfaatkan ruang partisi secara efisien.

**Dynamic Partitioning:**
Partisi berukuran dan berjumlah VARIABEL. Proses dialokasikan TEPAT sebesar yang dibutuhkannya. Teknik ini dipakai OS mainframe IBM, **OS/MVT**.

**Placement Algorithm** menentukan LOKASI partisi baru dialokasikan (misalnya First-Fit, Best-Fit, Worst-Fit — istilah umum yang mendasari materi ini, meski nama spesifiknya tidak muncul di teks slide).

### Buddy System
Menggabungkan skema FIXED dan DYNAMIC partitioning. Ruang yang tersedia diperlakukan sebagai SATU BLOK. Blok memori tersedia dalam ukuran $2^K$ kata, dengan $L \leq K \leq U$, di mana:
- $2^L$ = ukuran blok TERKECIL yang dialokasikan.
- $2^U$ = ukuran blok TERBESAR yang dialokasikan; umumnya $2^U$ adalah ukuran SELURUH memori yang tersedia untuk dialokasikan.

> [!info] Analogi
> Buddy System itu seperti membagi selembar kertas besar jadi dua secara TERUS-MENERUS sampai dapat ukuran yang PAS untuk kebutuhanmu. Kalau kamu butuh sepotong kecil kertas, kamu ambil selembar besar, lipat jadi dua (buddy pertama dan kedua), kalau masih terlalu besar, lipat lagi jadi dua-dua (4 bagian), terus sampai ukurannya PAS. Kalau kamu SELESAI pakai satu potongan kecil, dan potongan "SAUDARA" (buddy)-nya JUGA kosong, keduanya bisa DIGABUNG LAGI jadi potongan yang lebih besar — mengurangi fragmentasi dibanding kalau potongan-potongan kecil dibiarkan berserakan selamanya.

### Paging
Istilah **virtual memory** BIASANYA diasosiasikan dengan sistem yang memakai paging. Pemakaian paging untuk mencapai virtual memory PERTAMA kali dilaporkan di komputer **Atlas**. Setiap proses punya **page table**-nya SENDIRI. Setiap entri page table (**PTE — Page Table Entry**) berisi nomor FRAME dari halaman terkait di main memory.

**Masalah ukuran page table:** untuk alamat 32-bit (setengah untuk OS, setengah untuk user space), jumlah page di user process adalah $2^{32}/pagesize$. Untuk pagesize $= 2^{10}$, page table akan punya $2^{22}$ ENTRI — PER PROSES! Ini masalah besar untuk memori. **Solusi: "page" the page table itu sendiri** (paging bertingkat).

**Two-Level Hierarchical Page Table** — page table sendiri DIBAGI jadi halaman-halaman, dengan level tambahan (outer page table / page directory) yang menunjuk ke page table yang lebih detail — hanya bagian page table yang BENAR-BENAR dibutuhkan yang perlu ada di memori pada satu waktu.

**Inverted Page Table** — bagian NOMOR HALAMAN dari alamat virtual dipetakan ke sebuah HASH VALUE. Hash value ini menunjuk ke inverted page table. PROPORSI TETAP dari real memory dibutuhkan untuk tabel ini, TERLEPAS dari jumlah proses atau halaman virtual yang didukung. Disebut "inverted" karena mengindeks entri page table berdasarkan NOMOR FRAME, bukan nomor halaman virtual (kebalikan dari page table biasa).

> [!info] Konteks tambahan (bukan dari slide)
> Perbedaan mendasar: page table BIASA punya SATU entri per halaman VIRTUAL (bisa jutaan entri per proses, karena address space virtual bisa sangat besar). Inverted page table punya SATU entri per FRAME FISIK (jumlahnya TETAP, sebesar RAM fisik yang ada, TIDAK PEDULI berapa banyak proses atau berapa besar address space virtual masing-masing) — makanya ukurannya jauh lebih HEMAT untuk sistem dengan BANYAK proses.

### Translation Lookaside Buffer (TLB)
Setiap referensi memori VIRTUAL bisa menyebabkan DUA akses memori FISIK: satu untuk mengambil page table entry, satu lagi untuk mengambil DATA sesungguhnya. Untuk mengatasi efek "menggandakan" waktu akses memori ini, sebagian besar skema virtual memory memakai cache kecepatan tinggi khusus bernama **TLB (Translation Lookaside Buffer)**. Cache ini berfungsi SAMA seperti memory cache biasa dan berisi entri page table yang PALING BARU dipakai.

> [!info] Analogi
> TLB itu seperti CATATAN CONTEKAN kecil yang kamu simpan di saku untuk alamat-alamat yang PALING SERING kamu tuju. Tanpa contekan itu, setiap kali mau ke suatu tempat, kamu harus buka BUKU PETA TEBAL (page table lengkap) dulu untuk cari alamat detailnya — dua langkah (buka buku, lalu jalan). Dengan contekan (TLB), kalau alamat itu sudah pernah kamu cari sebelumnya dan MASIH ada di saku, kamu langsung tahu tanpa perlu buka buku peta lagi — jauh lebih cepat.

### Page Size: Trade-off
- Semakin KECIL ukuran page, semakin KECIL internal fragmentation — TAPI semakin BANYAK page yang dibutuhkan per proses, artinya page table jadi LEBIH BESAR.
- Untuk program besar di lingkungan multiprogramming BERAT, sebagian page table proses AKTIF harus berada di virtual memory, bukan main memory.
- Karakteristik fisik sebagian besar perangkat secondary memory FAVORIT ke ukuran page LEBIH BESAR untuk transfer blok data yang lebih efisien.
- Teknik pemrograman modern di program BESAR cenderung MENGURANGI locality of reference dalam sebuah proses — mempersulit pemilihan ukuran page yang optimal secara universal.

### Segmentation
**Segmentation** memungkinkan programmer memandang memori sebagai TERDIRI DARI beberapa address space atau SEGMEN.

**Keuntungan:**
- Menyederhanakan penanganan struktur data yang TUMBUH.
- Memungkinkan program diubah dan dikompilasi ulang secara INDEPENDEN.
- Cocok untuk BERBAGI data antar proses.
- Cocok untuk PROTEKSI.

**Organisasi Segmentation:** setiap entri segment table berisi ALAMAT AWAL segmen terkait di main memory dan PANJANG segmen. Sebuah bit dibutuhkan untuk menentukan apakah segmen SUDAH ada di main memory. Bit lain untuk menentukan apakah segmen sudah DIMODIFIKASI sejak dimuat ke main memory.

> [!info] Konteks tambahan (bukan dari slide)
> Perbedaan mendasar paging vs segmentation: **paging** membagi memori jadi potongan berukuran SAMA dan TETAP (frame), TIDAK memedulikan struktur logis program — murni soal EFISIENSI penyimpanan. **Segmentation** membagi memori berdasarkan STRUKTUR LOGIS program (misalnya "segmen kode", "segmen data", "segmen stack") dengan ukuran BERVARIASI sesuai kebutuhan tiap bagian — lebih NATURAL bagi programmer karena mencerminkan cara program disusun secara konseptual.

### Combined Paging with Segmentation
Menggabungkan KEDUA pendekatan: memori dibagi jadi SEGMEN (sesuai struktur logis program), tapi SETIAP SEGMEN itu SENDIRI dibagi lagi jadi PAGE. Ini mendapatkan kelebihan KEDUANYA — fleksibilitas logis dari segmentation DAN efisiensi alokasi seragam dari paging.

## Diagram & Visual
- **Slide 5 — Tiga Cara Sederhana Organisasi Memori**
  ![[99-Assets/OS/W10-slide05.jpg]]
- **Slide 8-9 — Dynamic Relocation dan Masalah Relokasi**
  ![[99-Assets/OS/W10-slide08.jpg]]
  ![[99-Assets/OS/W10-slide09.jpg]]
- **Slide 11 — Swapping**
  ![[99-Assets/OS/W10-slide11.jpg]]
- **Slide 12 — Managing Free Memory (Bitmap vs Linked List)**
  ![[99-Assets/OS/W10-slide12.jpg]]
- **Slide 15 — Posisi dan Fungsi MMU**
  ![[99-Assets/OS/W10-slide15.jpg]]
- **Slide 16 — Logical-to-Physical Address Translation**
  ![[99-Assets/OS/W10-slide16.png]]
- **Slide 18 — Teknik Memory Partitioning**
  ![[99-Assets/OS/W10-slide18.png]]
- **Slide 20 — Assignment to Fixed Partitioning**
  ![[99-Assets/OS/W10-slide20.png]]
- **Slide 23 — Efek Dynamic Partitioning**
  ![[99-Assets/OS/W10-slide23.png]]
- **Slide 26 — Placement Algorithm**
  ![[99-Assets/OS/W10-slide26.png]]
- **Slide 28-29 — Contoh Buddy System**
  ![[99-Assets/OS/W10-slide28.png]]
  ![[99-Assets/OS/W10-slide29.png]]
- **Slide 31 — Address Translation dalam Sistem Paging**
  ![[99-Assets/OS/W10-slide31.png]]
- **Slide 33-34 — Two-Level Hierarchical Page Table**
  ![[99-Assets/OS/W10-slide33.png]]
  ![[99-Assets/OS/W10-slide34.png]]
- **Slide 36 — Struktur Inverted Page Table**
  ![[99-Assets/OS/W10-slide36.png]]
- **Slide 39 — Pemakaian TLB**
  ![[99-Assets/OS/W10-slide39.png]]
- **Slide 44 — Address Translation Segmentation**
  ![[99-Assets/OS/W10-slide44.png]]
- **Slide 46 — Address Translation Combined Paging+Segmentation**
  ![[99-Assets/OS/W10-slide46.png]]

## Rumus / Sintaks
```
Buddy System: blok memori ukuran 2^K kata, L <= K <= U
  2^L = ukuran blok terkecil
  2^U = ukuran blok terbesar (= total memori tersedia)

Ukuran Page Table (alamat 32-bit, pagesize 2^10):
  Jumlah page per proses = 2^32 / 2^10 = 2^22 entri PER PROSES

Access time via TLB:
  TLB hit  -> 1x akses memori (langsung data)
  TLB miss -> 2x akses memori (page table entry, lalu data)
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Overlay** | Teknik lama memuat bagian program secara bergantian saat program lebih besar dari partisi memori |
| **Locality of reference** | Kecenderungan program mengakses alamat memori yang berdekatan dalam periode waktu singkat |
| **Frame** | Unit blok memori fisik berukuran tetap tempat sebuah page disimpan |
| **Page Directory** | Level atas dari two-level page table yang menunjuk ke page table detail |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa Fixed Partitioning menyebabkan Internal Fragmentation, sementara Dynamic Partitioning TIDAK punya masalah yang sama (tapi punya masalah lain — external fragmentation, meski tidak dibahas rinci di slide ini). Hubungkan dengan definisi masing-masing pendekatan.
2. **(C4 – Analisis)** Bandingkan Two-Level Hierarchical Page Table dan Inverted Page Table sebagai DUA solusi berbeda untuk masalah "page table terlalu besar". Analisis: kenapa Inverted Page Table punya ukuran yang TETAP terlepas dari jumlah proses, sementara Two-Level Page Table ukurannya TETAP BERTAMBAH seiring jumlah proses (meski lebih hemat dari single-level table)?
3. **(C5 – Evaluasi)** Sebuah sistem memilih ukuran PAGE SANGAT KECIL untuk meminimalkan internal fragmentation. Evaluasi: apa dampak NEGATIF dari keputusan ini terhadap ukuran page table dan efisiensi transfer data dari secondary storage, berdasarkan trade-off "Page Size" yang dibahas di materi?
4. **(C5 – Evaluasi)** Bandingkan Paging murni dengan Segmentation murni dari sudut pandang PROGRAMMER yang menulis kode. Evaluasi: kenapa Segmentation dianggap lebih "natural" untuk programmer (misalnya untuk "menyederhanakan penanganan struktur data yang tumbuh"), sementara Paging lebih fokus ke efisiensi SISTEM daripada kenyamanan programmer?
5. **(C6 – Cipta)** Rancang skenario (fiktif, untuk latihan) di mana sebuah sistem dengan RAM terbatas (misalnya 4GB) harus menjalankan BANYAK proses (misalnya 50 proses) sekaligus dengan address space virtual besar masing-masing. Usulkan kombinasi teknik memory management (dari materi ini: paging bertingkat, inverted page table, TLB, segmentation) yang PALING SESUAI untuk skenario ini, dan jelaskan alasan kombinasi pilihanmu.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W09 - IO Management]]
- [[W11 - Virtual Memory]]
- [[OS - Review dan Glosari]]
