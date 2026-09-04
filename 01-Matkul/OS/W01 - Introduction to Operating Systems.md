---
matkul: Operating Systems
minggu: 1
sks: 3
sumber: Introduction to Operating Systems.pptx
tags: [kuliah/os, minggu/w01]
status: draft
diproses: 2026-09-04
---

# W01 — Introduction to Operating Systems

## Ringkasan
> - **Instruction cycle** = siklus dasar cara CPU bekerja: ambil instruksi (fetch), lalu eksekusi (execute), berulang terus. Ini fondasi paling dasar sebelum memahami OS.
> - **Interrupt** = sinyal dari hardware ATAU software yang memaksa CPU BERHENTI sejenak dari kerjaan sekarang untuk menangani kejadian penting (misal ada tombol ditekan, timer habis, atau error hardware).
> - **Memory hierarchy** — dari yang PALING CEPAT tapi PALING MAHAL/KECIL (cache di dalam CPU) sampai yang PALING LAMBAT tapi PALING MURAH/BESAR (disk) — OS harus pintar mengatur data mengalir di hierarki ini.
> - OS berevolusi dari **tidak ada OS sama sekali** (programmer langsung pegang hardware) → **batch system** (job dikumpulkan dulu baru dijalankan) → **multiprogramming** (beberapa program "seolah" jalan bersamaan) → **time-sharing** (banyak user berbagi satu komputer secara real-time).
> - **System call** adalah "pintu resmi" yang dipakai program user untuk minta layanan ke OS — semacam API antara aplikasi dan kernel.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Instruction cycle | Siklus dasar fetch-execute yang dijalankan CPU berulang-ulang |
| Interrupt | Sinyal yang memaksa CPU berhenti sejenak untuk menangani kejadian penting |
| Memory hierarchy | Susunan bertingkat memori dari yang tercepat/termahal ke yang terlambat/termurah |
| Multiprogramming | Beberapa program dimuat di memori sekaligus, CPU berpindah-pindah mengerjakannya |
| Time-sharing | Banyak user mengakses satu komputer secara "bersamaan" lewat pembagian waktu CPU |
| System call | Cara resmi program user meminta layanan dari operating system |

## Isi

### Elemen Dasar Komputer
Sebelum masuk ke OS, penting memahami komponen dasar sebuah sistem komputer: **CPU** (Central Processing Unit) yang berisi PC (Program Counter), IR (Instruction Register), MAR/MBR (Memory Address/Buffer Register), **Main Memory**, dan **I/O Module**. Ketiganya terhubung lewat **System Bus** yang jadi jalur komunikasi data.

### Instruction Cycle
**Instruction cycle** adalah siklus dasar operasi CPU: CPU **mengambil (fetch)** instruksi dari memori, lalu **mengeksekusi (execute)** instruksi tersebut, dan berulang terus-menerus. Ini adalah fondasi cara kerja CPU yang paling mendasar — semua yang dilakukan komputer, pada akhirnya, adalah pengulangan siklus ini jutaan kali per detik.

> [!info] Analogi
> Instruction cycle itu seperti seseorang membaca resep masak satu baris demi satu baris: BACA baris instruksi ("potong bawang"), lalu KERJAKAN instruksi itu, lalu BACA baris berikutnya, lalu KERJAKAN lagi — terus berulang sampai resepnya selesai. CPU melakukan hal yang sama, tapi dalam skala miliaran kali per detik.

### Interrupt
**Interrupt** adalah kemunculan sebuah event yang berasal dari HARDWARE atau SOFTWARE, memberi sinyal ke CPU lewat bus. Software memicu interrupt dengan mengeksekusi sebuah system call. Interrupt memindahkan kontrol ke sebuah rutin penanganan interrupt (interrupt handling routine), yang terdaftar di sebuah array bernama **interrupt vector**.

**Empat jenis interrupt:**
1. **Program Interrupt** — dipicu oleh kondisi tertentu dalam eksekusi program (misalnya error).
2. **Timer Interrupt** — dipicu oleh timer internal CPU (penting untuk time-sharing).
3. **I/O Interrupt** — dipicu oleh perangkat I/O (misal disk selesai membaca data).
4. **Hardware Failure Interrupt** — dipicu oleh kegagalan hardware.

> [!info] Analogi
> Interrupt itu seperti telepon berdering saat kamu sedang mengerjakan sesuatu. Kamu BERHENTI sejenak dari pekerjaan (eksekusi program berjalan), MENANGANI telepon itu (interrupt handling routine), lalu KEMBALI ke pekerjaan semula persis di titik kamu berhenti. Kalau ada BANYAK telepon masuk sekaligus (multiple interrupts), kamu perlu strategi: entah menahan semua telepon lain sampai yang pertama selesai (disable interrupt), atau memprioritaskan yang paling penting duluan (priority scheme).

### Multiple Interrupts
Ketika sebuah interrupt terjadi SAAT interrupt lain sedang diproses, ada dua pendekatan:
1. **Disable interrupt** sementara interrupt lain sedang diproses.
2. **Priority scheme** — interrupt dengan prioritas lebih tinggi bisa menyela interrupt berprioritas lebih rendah.

### Pipelining
Slide membahas **3-stage pipeline vs Superscalar CPU** — pipelining membantu meningkatkan performa dengan cara memproses beberapa instruksi secara bertahap (tumpang tindih), bukan menunggu satu instruksi selesai total sebelum memulai instruksi berikutnya.

### Memory Hierarchy dan Cache
**Memory hierarchy** menyusun jenis memori dari yang PALING CEPAT (tapi PALING KECIL/MAHAL) sampai yang PALING LAMBAT (tapi PALING BESAR/MURAH). Main memory dibagi jadi cache line — cache line yang paling sering dipakai disimpan di **cache berkecepatan tinggi** yang letaknya di dalam atau sangat dekat dengan CPU. Ada beberapa tipe cache: **L1, L2, dan L3**, berbeda dari sisi ukuran (L1 paling kecil tapi tercepat, L3 paling besar tapi agak lebih lambat).

> [!info] Konteks tambahan (bukan dari slide)
> Analogi umum untuk memory hierarchy: bayangkan meja kerja (cache — sangat cepat diakses, tapi muat sedikit barang), lemari di ruangan yang sama (main memory — agak lebih lambat, muat lebih banyak), dan gudang di lantai lain (disk — jauh lebih lambat, tapi muat sangat banyak). OS dan CPU terus berusaha menaruh data yang PALING SERING dipakai di "meja kerja" supaya tidak perlu bolak-balik ke gudang.

### Operating System sebagai Interface dan Resource Manager
OS didefinisikan sebagai **program yang mengontrol eksekusi program aplikasi** dan **interface antara aplikasi dan hardware**. OS juga berperan sebagai **resource manager** — mengatur alokasi CPU, memori, dan perangkat I/O ke berbagai proses yang berjalan.

**Layanan utama OS:**
- Program development
- Program execution
- Akses ke perangkat I/O
- Akses terkontrol ke file
- Akses sistem
- Deteksi dan respons error
- Accounting

### Evolusi Operating System
Sejarah OS berkembang melalui beberapa tahap:

1. **Komputer awal (tanpa OS)** — programmer berinteraksi LANGSUNG dengan hardware. Komputer dijalankan dari konsol dengan lampu display, saklar toggle, dan printer. User mengakses komputer secara BERGILIRAN ("in series"). Masalah: penjadwalan manual lewat lembar tanda tangan kertas (sering boros waktu) dan waktu setup program yang lama.

2. **Monitor (batch system awal)** — komputer awal sangat mahal, jadi penting memaksimalkan utilisasi processor. User TIDAK LAGI punya akses langsung ke processor — job diserahkan ke operator komputer yang mengumpulkannya (batch) dan menaruhnya di input device. Program kembali ke monitor saat selesai.

3. **Job Control Language (JCL)** — bahasa untuk mengontrol batch job.

4. **Multi-programmed batch system** — beberapa job dimuat sekaligus di memori, CPU berpindah mengerjakan job yang siap, meningkatkan utilisasi CPU.

5. **Uni-programming vs Multiprogramming** — uni-programming = satu program berjalan sampai selesai baru program berikutnya dimulai (banyak waktu CPU terbuang menunggu I/O); multiprogramming = beberapa program dimuat sekaligus, CPU beralih ke program lain saat satu program menunggu I/O.

6. **Time-Sharing System** — bisa menangani banyak job interaktif sekaligus; waktu processor DIBAGI di antara banyak user. Banyak user mengakses sistem secara simultan lewat terminal, dengan OS menyisipkan (interleaving) eksekusi tiap program user dalam burst singkat (quantum).

7. **CTSS (Compatible Time-Sharing System)** — salah satu OS time-sharing PERTAMA, dikembangkan di MIT oleh Project MAC, pertama dijalankan di IBM 709 tahun 1961. Berjalan dengan memori utama 32.000 kata 36-bit (5000 di antaranya dipakai monitor). Memakai teknik **time slicing**: interrupt clock sistem terjadi kira-kira setiap 0,2 detik, di setiap interrupt OS mengambil alih kontrol dan bisa menugaskan processor ke user lain. Status program user lama disimpan ke disk sebelum program user baru dimuat, lalu dipulihkan saat gilirannya tiba lagi.

> [!info] Analogi
> Evolusi ini seperti evolusi antrian di loket bank: awalnya (tanpa OS) setiap orang bebas masuk ke ruang teller kapan saja (kacau). Lalu ada SATU ANTRIAN dengan nomor urut (batch system) — tapi kalau satu orang lama, semua nunggu. Lalu bank buka BEBERAPA LOKET sekaligus (multiprogramming) — kalau satu loket sedang menghitung uang lama (I/O), teller lain tetap bisa layani nasabah lain. Akhirnya, bank memberi tiap nasabah waktu SANGAT SINGKAT bergiliran cepat (time-sharing) sehingga SEMUA orang merasa dilayani "hampir bersamaan", meski sebenarnya bergiliran sangat cepat.

### System Calls
Interface antara program user dan OS pada dasarnya berurusan dengan ABSTRAKSI. System call yang tersedia bervariasi antar OS. Sebuah system call dipanggil dari program C dengan memanggil sebuah library procedure yang namanya sama dengan system call itu.

### Struktur OS Modern
Slide menunjukkan struktur berbagai OS modern lewat diagram: **Windows, UNIX tradisional, UNIX modern, LINUX,** dan **Android** — masing-masing punya lapisan arsitektur berbeda antara kernel, layanan sistem, dan aplikasi user.

## Diagram & Visual
- **Slide 6 — Komponen PC sederhana**
  ![[99-Assets/OS/W01-slide06.jpg]]
- **Slide 7 — Top-Level View of Computer Components**
  ![[99-Assets/OS/W01-slide07.png]]
- **Slide 8 — Basic Instruction Cycle**
  ![[99-Assets/OS/W01-slide08.png]]
- **Slide 9 — Contoh Eksekusi Program**
  ![[99-Assets/OS/W01-slide09.png]]
- **Slide 12 — Instruction Cycle dengan Interrupt**
  ![[99-Assets/OS/W01-slide12.png]]
- **Slide 13 — 3-Stage Pipeline vs Superscalar CPU**
  ![[99-Assets/OS/W01-slide13.jpg]]
- **Slide 15 — Contoh I/O Interrupt**
  ![[99-Assets/OS/W01-slide15.jpg]]
- **Slide 16 — Memory Hierarchy**
  ![[99-Assets/OS/W01-slide16.png]]
- **Slide 18 — Diagram Cache**
  ![[99-Assets/OS/W01-slide18.png]]
- **Slide 19 — Operating System Interface**
  ![[99-Assets/OS/W01-slide19.png]]
- **Slide 20 — Where is the Operating System?**
  ![[99-Assets/OS/W01-slide20.jpg]]
- **Slide 22 — Hardware and Software Structure**
  ![[99-Assets/OS/W01-slide22.png]]
- **Slide 24 — OS sebagai Resource Manager**
  ![[99-Assets/OS/W01-slide24.png]]
- **Slide 25 — Struktur Operating System**
  ![[99-Assets/OS/W01-slide25.jpg]]
- **Slide 28 — Early Computer System**
  ![[99-Assets/OS/W01-slide28.jpg]]
  ![[99-Assets/OS/W01-slide28a.jpg]]
- **Slide 32 — Multi-programmed Batch System**
  ![[99-Assets/OS/W01-slide32.png]]
- **Slide 33 — Uni-programming vs Multiprogramming**
  ![[99-Assets/OS/W01-slide33.png]]
  ![[99-Assets/OS/W01-slide33a.png]]
- **Slide 37 — Contoh System Call**
  ![[99-Assets/OS/W01-slide37.png]]
- **Slide 38-42 — Struktur Windows, UNIX tradisional, UNIX modern, LINUX, Android**
  ![[99-Assets/OS/W01-slide38.png]]
  ![[99-Assets/OS/W01-slide39.png]]
  ![[99-Assets/OS/W01-slide40.png]]
  ![[99-Assets/OS/W01-slide41.png]]
  ![[99-Assets/OS/W01-slide42.png]]

> [!info] Catatan konversi gambar: sebagian gambar diagram di deck ini aslinya format **WMF** (Windows Metafile) yang tidak bisa ditampilkan Obsidian secara native — semua sudah dikonversi ke PNG saat ekstraksi supaya bisa ditampilkan di vault ini.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **PC (Program Counter)** | Register yang menyimpan alamat instruksi berikutnya yang akan dieksekusi |
| **IR (Instruction Register)** | Register yang menyimpan instruksi yang sedang dieksekusi |
| **Interrupt vector** | Array yang berisi daftar rutin penanganan interrupt |
| **Trap** | Interrupt yang dipicu oleh software (kondisi error/exception di dalam program) |
| **Kernel** | Inti dari operating system yang mengelola sumber daya sistem secara langsung |
| **Quantum (time-sharing)** | Potongan waktu singkat yang dialokasikan ke satu user/proses secara bergiliran |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa CTSS (1961) memilih interval time slicing sekitar 0,2 detik. Apa yang akan terjadi (dari sisi pengalaman user) kalau interval itu dibuat JAUH lebih besar (misalnya 5 detik) atau JAUH lebih kecil (misalnya 0,001 detik)?
2. **(C4 – Analisis)** Bandingkan pendekatan "Disable interrupt" dan "Priority scheme" untuk menangani multiple interrupts. Analisis: skenario APA yang membuat "Disable interrupt" berbahaya untuk dipakai (misalnya interrupt penting yang terlewat)?
3. **(C5 – Evaluasi)** Sebuah sistem lama beroperasi dengan uni-programming (satu program jalan sampai selesai, baru program berikutnya dimulai). Evaluasi: dalam kondisi seperti apa uni-programming JUSTRU lebih masuk akal dibanding multiprogramming, meski secara umum multiprogramming dianggap lebih efisien?
4. **(C5 – Evaluasi)** Bandingkan memory hierarchy (cache L1/L2/L3 → main memory → disk) dengan prinsip time-sharing (CPU dibagi dalam quantum singkat). Evaluasi: apa KESAMAAN filosofis di balik kedua konsep ini — keduanya sama-sama berusaha menyelesaikan masalah "sumber daya terbatas tapi kebutuhan banyak" dengan cara apa?
5. **(C6 – Cipta)** Rancang skenario (fiktif, untuk latihan) di mana sebuah sistem MODERN (bukan CTSS 1961) mengalami masalah performa karena terlalu banyak interrupt terjadi bersamaan (misalnya banyak perangkat I/O aktif sekaligus). Usulkan strategi kombinasi (disable interrupt + priority scheme) yang bisa dipakai untuk mengatasi skenario ini, dan jelaskan urutan prioritas yang kamu tetapkan.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W02 - Processes]]
- [[OS - Review dan Glosari]]
