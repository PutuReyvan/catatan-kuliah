---
matkul: Operating Systems
minggu: 4
sks: 3
sumber: Multiple Processor Systems.pptx
tags: [kuliah/os, minggu/w04]
status: draft
diproses: 2026-09-04
---

# W04 — Multiple Processor Systems

## Ringkasan
> - **Multiprocessor system** = interkoneksi dua atau lebih CPU yang berbagi memori dan I/O. Beda dari "banyak komputer terpisah", di sini CPU-CPU-nya SALING TERHUBUNG dalam satu sistem.
> - **Granularity** parallelism ada 4 tingkat: **Independent** (proses sama sekali tidak sinkron, cocok time-sharing) → **Coarse-grained** (sinkron di level sangat kasar) → **Medium-grained** (thread dalam satu aplikasi, butuh koordinasi tinggi) → **Fine-grained** (paralelisme sangat detail dan kompleks).
> - Tiga model arsitektur multiprosesor: **Shared-memory** (semua CPU akses memori yang SAMA), **Message-passing multicomputer** (CPU terpisah, komunikasi lewat pesan), dan **Wide area distributed system** (node tersebar geografis).
> - **UMA (Uniform Memory Access)** vs **NUMA (Non-Uniform Memory Access)**: di UMA semua CPU mengakses memori dengan kecepatan SAMA; di NUMA, akses ke memori LOKAL lebih cepat dari memori JAUH (remote).
> - Tiga model OS untuk multiprosesor: **Each CPU has its own OS** (paling sederhana tapi kurang efisien), **Leader-Follower** (satu CPU jadi "master"), dan **SMP (Symmetric Multiprocessing)** — model paling umum di sistem modern, semua CPU setara.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Multiprocessor system | Interkoneksi dua atau lebih CPU yang berbagi memori dan I/O |
| Granularity | Tingkat kehalusan/kekasaran sinkronisasi antar proses paralel |
| UMA (Uniform Memory Access) | Semua CPU mengakses memori dengan kecepatan/latency yang SAMA |
| NUMA (Non-Uniform Memory Access) | Akses memori lokal lebih cepat dari akses memori remote |
| SMP (Symmetric Multiprocessing) | Model OS multiprosesor di mana semua CPU setara/simetris |
| Distributed system | Sistem dengan node-node yang punya memori privat masing-masing, tanpa shared memory fisik |
| Middleware | Lapisan software di atas OS untuk menyeragamkan hardware/OS berbeda di sistem terdistribusi |

## Isi

### Definisi Multiprocessor
**Multiprocessor system** adalah interkoneksi dua atau lebih CPU dengan memori dan perangkat input-output. Ini beda dari sekadar "banyak komputer" — CPU-CPU dalam sistem ini SALING TERHUBUNG dan (tergantung arsitektur) bisa berbagi sumber daya.

### Granularity Parallelism
Slide mengklasifikasikan tingkat sinkronisasi antar proses paralel dari yang PALING LONGGAR sampai PALING KETAT:

1. **Independent Parallelism** — TIDAK ADA sinkronisasi eksplisit antar proses. Setiap proses merepresentasikan aplikasi/job yang terpisah dan independen. Penggunaan tipikal: sistem time-sharing.

2. **Coarse and Very Coarse-Grained Parallelism** — ada sinkronisasi antar proses, tapi di level yang SANGAT KASAR. Mudah ditangani sebagai sekumpulan proses konkuren yang berjalan di uniprocessor multi-programmed. Bisa didukung di multiprocessor dengan sedikit atau tanpa perubahan software user.

3. **Medium-Grained Parallelism** — satu aplikasi bisa diimplementasikan efektif sebagai kumpulan THREAD dalam satu proses (ingat [[W03 - Threads]]). Programmer harus SECARA EKSPLISIT menspesifikasikan potensi paralelisme aplikasi. Butuh tingkat koordinasi dan interaksi TINGGI antar thread aplikasi — keputusan penjadwalan untuk SATU thread bisa memengaruhi performa SELURUH aplikasi.

4. **Fine-Grained Parallelism** — merepresentasikan penggunaan paralelisme yang JAUH LEBIH KOMPLEKS daripada penggunaan thread biasa. Area khusus dan terfragmentasi dengan banyak pendekatan berbeda.

> [!info] Analogi
> Bayangkan empat level ini seperti tingkat kerja sama dalam proyek kelompok. **Independent** itu seperti empat mahasiswa mengerjakan tugas INDIVIDU yang sama sekali berbeda topik — tidak perlu koordinasi sama sekali. **Coarse-grained** itu seperti kelompok yang membagi tugas jadi bab-bab besar, cek progres sekali di akhir minggu. **Medium-grained** itu seperti tim yang menulis SATU bab yang sama secara bersamaan — harus terus koordinasi kalimat per kalimat supaya nyambung. **Fine-grained** itu seperti menulis SATU KALIMAT bersama-sama kata demi kata secara real-time — butuh koordinasi paling ketat dan rumit dari semuanya.

### Tiga Model Multiprosesor
Slide menampilkan tiga model arsitektur (Figure 8.1):
- **(a) Shared-memory multiprocessor** — semua CPU berbagi SATU memori yang sama.
- **(b) Message-passing multicomputer** — CPU-CPU terpisah, berkomunikasi lewat pertukaran pesan.
- **(c) Wide area distributed system** — sistem tersebar secara geografis luas.

### Multiprocessor Hardware: UMA
**UMA (Uniform Memory Access)** — semua CPU mengakses memori dengan kecepatan/latency yang SAMA. Ada beberapa implementasi:
- **UMA dengan Arsitektur Berbasis Bus** — semua CPU terhubung ke memori lewat satu bus bersama.
- **UMA memakai Crossbar Switch** — misalnya crossbar switch 8x8, dengan crosspoint yang bisa terbuka (open) atau tertutup (closed) untuk mengarahkan koneksi CPU ke memori.
- **UMA memakai Multistage Switching Networks** — misalnya switch 2x2 dengan dua input line (A, B) dan dua output line (X, Y), memakai format pesan tertentu untuk routing.

### Multiprocessor Hardware: NUMA
**NUMA (Non-Uniform Memory Access)** — punya tiga karakteristik kunci:
1. Ada SATU address space tunggal yang terlihat oleh SEMUA CPU.
2. Akses ke memori REMOTE dilakukan lewat instruksi LOAD dan STORE biasa.
3. Akses ke memori REMOTE LEBIH LAMBAT dibanding akses ke memori LOKAL.

Contoh: sebuah sistem directory-based multiprocessor dengan 256 node, di mana alamat memori 32-bit dibagi jadi beberapa field, dan setiap node punya "directory" yang melacak status data di node lain.

> [!info] Analogi
> UMA itu seperti perpustakaan dengan SATU rak buku di TENGAH ruangan — semua pembaca (CPU), di mana pun mereka duduk, butuh waktu SAMA untuk berjalan ke rak dan kembali. NUMA itu seperti perpustakaan dengan rak buku KECIL di dekat setiap meja baca (memori lokal) PLUS satu rak besar di gudang jauh (memori remote) — kalau buku yang kamu cari ada di rak dekat mejamu, cepat sekali; tapi kalau harus ambil dari gudang jauh, jauh lebih lambat. NUMA jadi masuk akal untuk sistem BESAR karena membangun "satu rak di tengah yang sama cepat untuk semua orang" jadi tidak praktis kalau jumlah CPU-nya sangat banyak.

### Tipe OS untuk Multiprocessor
1. **Each CPU Has Its Own Operating System** — memori multiprosesor dipartisi di antara (misal) 4 CPU, tapi berbagi SATU salinan kode OS yang sama; setiap CPU punya data privat OS-nya sendiri. Pendekatan paling sederhana, tapi kurang efisien untuk koordinasi sumber daya lintas CPU.

2. **Leader-Follower Multiprocessors** — satu CPU berperan sebagai "leader" (mengatur/mengoordinasikan), CPU lain sebagai "follower" yang menjalankan tugas atas arahan leader.

3. **SMP (Symmetric Multiprocessing) Multiprocessors** — SEMUA CPU setara/simetris, bisa menjalankan salinan OS yang sama dan mengakses sumber daya bersama tanpa satu CPU "berkuasa" atas yang lain. Ini model yang paling UMUM dipakai di sistem multiprosesor modern.

### Distributed System
**Distributed system** — setiap node punya memori PRIVAT-nya sendiri, TIDAK ADA shared physical memory di seluruh sistem. Distributed system bahkan LEBIH LONGGAR terikat (loosely coupled) dibanding multicomputer (message-passing).

**Middleware** — salah satu cara sistem terdistribusi mencapai keseragaman meski hardware dan OS di bawahnya berbeda-beda adalah dengan menambahkan LAPISAN software DI ATAS operating system. Lapisan ini disebut **middleware** — menyembunyikan perbedaan hardware/OS dari aplikasi yang berjalan di atasnya.

### Diskusi: Ke Mana Multicore Diarahkan?
Slide menutup dengan pertanyaan diskusi terbuka: CPU multicore sudah umum di desktop dan laptop, dan desktop dengan puluhan-ratusan core bukan hal jauh lagi. Ada dua kemungkinan cara memanfaatkan kekuatan ini:
1. **Paralelisasi aplikasi desktop standar** (word processor, web browser).
2. **Paralelisasi layanan yang ditawarkan OS itu sendiri** (misalnya pemrosesan TCP, atau library service umum seperti fungsi library HTTP aman).

Slide bertanya: pendekatan mana yang PALING MENJANJIKAN, dan kenapa?

> [!info] Konteks tambahan (bukan dari slide)
> Pertanyaan ini masih relevan sampai sekarang. Argumen untuk paralelisasi APLIKASI: manfaat langsung terasa user (misalnya browser terasa lebih responsif). Argumen untuk paralelisasi LAYANAN OS: manfaatnya otomatis dinikmati SEMUA aplikasi yang memakai layanan itu (misalnya semua aplikasi yang pakai networking dapat manfaat dari TCP processing yang dipararelkan), tanpa developer aplikasi perlu menulis ulang kode paralel sendiri-sendiri.

## Diagram & Visual
- **Slide 6 — Tabel Sinkronisasi Granularity dan Proses**
  ![[99-Assets/OS/W04-slide06.png]]
- **Slide 11 — Tiga Model Multiprosesor (shared-memory, message-passing, distributed)**
  ![[99-Assets/OS/W04-slide11.jpg]]
- **Slide 13 — Crossbar Switch 8x8**
  ![[99-Assets/OS/W04-slide13.jpg]]
- **Slide 14 — Multistage Switching Network**
  ![[99-Assets/OS/W04-slide14.jpg]]
- **Slide 16 — NUMA Directory-Based Multiprocessor (256 node)**
  ![[99-Assets/OS/W04-slide16.jpg]]
- **Slide 17 — Each CPU Has Its Own OS**
  ![[99-Assets/OS/W04-slide17.jpg]]
- **Slide 18 — Leader-Follower Multiprocessor Model**
  ![[99-Assets/OS/W04-slide18.jpg]]
- **Slide 19 — SMP Multiprocessor Model**
  ![[99-Assets/OS/W04-slide19.jpg]]
- **Slide 21 — Diagram Middleware**
  ![[99-Assets/OS/W04-slide21.jpg]]

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Crossbar switch** | Jaringan switch yang menghubungkan setiap input ke setiap output lewat matriks crosspoint |
| **Directory-based multiprocessor** | Arsitektur NUMA yang melacak status data lintas node lewat struktur "directory" |
| **Loosely coupled** | Sistem dengan keterikatan longgar antar komponennya, umum di distributed system |
| **Multicomputer** | Sistem dengan banyak komputer terpisah yang berkomunikasi lewat message-passing |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa Medium-Grained Parallelism (thread dalam satu aplikasi) butuh koordinasi lebih TINGGI dibanding Coarse-Grained Parallelism. Hubungkan dengan konsep thread yang berbagi address space yang sama (dari [[W03 - Threads]]) — kenapa berbagi memori yang sama justru meningkatkan KEBUTUHAN koordinasi, bukan menguranginya?
2. **(C4 – Analisis)** Bandingkan UMA dan NUMA dari sisi SKALABILITAS. Analisis: kenapa UMA (dengan bus/crossbar bersama) menjadi semakin SULIT diskalakan ke ratusan CPU, sementara NUMA (dengan memori lokal per node) lebih mudah diskalakan?
3. **(C5 – Evaluasi)** Sebuah perusahaan merancang sistem dengan 500 CPU dan memilih model OS "Each CPU Has Its Own Operating System" (satu salinan kode OS dibagi, tapi data privat terpisah per CPU). Evaluasi: apa risiko/kelemahan pendekatan ini dibanding SMP untuk skala 500 CPU, terutama soal koordinasi sumber daya bersama?
4. **(C5 – Evaluasi)** Bandingkan argumen "paralelisasi aplikasi desktop" vs "paralelisasi layanan OS" dari pertanyaan diskusi di slide. Evaluasi pendapatmu sendiri: mana yang lebih EFISIEN dari sisi usaha development (effort developer) untuk memanfaatkan CPU dengan ratusan core, dan kenapa?
5. **(C6 – Cipta)** Rancang skenario (fiktif, untuk latihan) sebuah aplikasi web server yang akan di-deploy di server dengan arsitektur NUMA (banyak node, memori lokal per node). Usulkan strategi penempatan data/proses (data placement) yang MEMINIMALKAN akses memori remote yang lambat — misalnya bagaimana kamu akan menempatkan cache/data yang sering diakses relatif terhadap CPU yang paling sering memprosesnya.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W03 - Threads]]
- [[W05 - Process Scheduling]]
- [[OS - Review dan Glosari]]
