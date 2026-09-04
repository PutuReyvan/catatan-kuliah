---
matkul: Operating Systems
tags: [kuliah/os, ujian]
status: draft
diproses: 2026-09-04
---

# Operating Systems — Review dan Glosari

## Cara Pakai
Baca ini SEBELUM buka slide asli — tujuannya kasih peta cepat semua materi satu matkul biar inget alur besarnya dulu, baru kalau perlu detail buka note per minggu ([[_OS]]).

## Ringkasan Cepat per Minggu

**[[W01 - Introduction to Operating Systems]]** — Instruction cycle (fetch-execute) dan interrupt adalah fondasi paling dasar. Memory hierarchy (cache L1/L2/L3 → main memory → disk) mengatur trade-off kecepatan vs kapasitas. OS berevolusi: tanpa abstraksi → batch → multiprogramming → time-sharing (CTSS 1961).

**[[W02 - Processes]]** — Process = program yang sedang dieksekusi. Siklus hidup: create (4 cara) → run → terminate (4 jenis). Model state berkembang dari 2-state ke 5-state, ditambah Suspended state untuk swapping. PCB adalah struktur data terpenting di OS. `fork()` membuat salinan proses, `exec()` mengganti isinya dengan program baru.

**[[W03 - Threads]]** — Thread = "lightweight process", unit eksekusi di dalam proses. Tiga implementasi: ULT (kernel tidak tahu, satu thread blocking = semua blocking), KLT (kernel tahu, lebih fleksibel tapi overhead mode switch), Hybrid (gabungan, contoh Solaris). pthread API: create, self, join.

**[[W04 - Multiple Processor Systems]]** — Granularity parallelism dari Independent (tanpa sinkronisasi) sampai Fine-Grained (paling kompleks). UMA (akses memori seragam) vs NUMA (akses lokal lebih cepat dari remote). SMP adalah model OS multiprosesor paling umum. Distributed system tidak punya shared memory fisik sama sekali.

**[[W05 - Process Scheduling]]** — 3 level: Long-term (siapa masuk sistem), Medium-term (bagian swapping), Short-term/dispatcher (paling sering jalan). 5 kriteria: CPU utilization, throughput, turnaround, waiting, response time. Algoritma: FCFS, SJF/SPN, SRT, Round Robin, HRRN, Multilevel Queue, Fair Share — masing-masing trade-off berbeda antara kesederhanaan, keadilan, dan risiko starvation.

**[[W06 - Synchronization dan Inter-process Communication]]** — Race condition terjadi kalau proses berebut data bersama tanpa koordinasi. Critical section harus dilindungi mutual exclusion. Solusi berkembang dari disable interrupt (kasar) → Peterson's Solution (software, busy waiting) → TSL instruction (hardware atomik) → Semaphore (dengan queue) → Monitor (construct bahasa, lebih mudah dikontrol). Semaphore dipakai selesaikan Producer-Consumer dan Readers-Writers Problem.

**[[W07 - Deadlock]]** — 4 kondisi WAJIB terpenuhi sekaligus: Mutual Exclusion, Hold-and-Wait, No-Preemption, Circular Wait. 3 strategi: Prevention (cegah salah satu kondisi), Avoidance (Banker's Algorithm, jaga Safe State), Detection & Recovery (biarkan terjadi, deteksi via RAG, lalu abort/preempt). Dining Philosophers Problem = studi kasus klasik yang menunjukkan keempat kondisi sekaligus.

**[[W08 - File Systems]]** — File organization: Pile, Sequential, Indexed Sequential, Indexed, Direct/Hashed — masing-masing trade-off kecepatan akses vs kemudahan update. File allocation: Contiguous (cepat, rawan fragmentasi), Chained/Linked List (fleksibel, lambat random access), Indexed (gabungan kelebihan). inode = "kartu identitas" file di UNIX, satu file satu inode tapi bisa banyak nama (hard link).

**[[W09 - IO Management]]** — 3 teknik I/O: Programmed (busy-wait, boros), Interrupt-driven (CPU lanjut kerja lain), DMA (transfer langsung tanpa CPU per byte). Access time = seek time + rotational delay. Disk scheduling: FIFO, SSTF, SCAN/elevator, C-SCAN, N-Step SCAN, F-SCAN. RAID menggabungkan banyak disk untuk performa DAN redundansi lewat parity.

**[[W10 - Memory Management]]** — Address space = abstraksi kunci, tiap proses punya ruang alamat sendiri. Base/limit register untuk dynamic relocation. Partitioning (fixed/dynamic) adalah cara lama tanpa virtual memory. Paging membagi memori jadi frame tetap, page table bisa jadi sangat besar (diatasi hierarchical/inverted page table + TLB). Segmentation membagi memori berdasarkan struktur logis program.

**[[W11 - Virtual Memory]]** — Page fault = halaman yang dirujuk ternyata tidak ada di memori. OS punya 5 kebijakan: Fetch (Demand Paging vs Pre-paging), Placement, Replacement (Optimal/LRU/FIFO/Clock), Resident Set Management (Global/Local), Cleaning. Thrashing = sistem terlalu sibuk swap sampai tidak ada kerja produktif tersisa.

**[[W12 - Security]]** — Sistem aman = TIDAK BISA DICAPAI mutlak, tujuannya bikin biaya serangan cukup tinggi. 4 level keamanan: Physical, Application, OS, Network — sekuat mata rantai terlemah (termasuk manusia). Buffer overflow = celah klasik, dilawan dengan compile-time defense (safe library, stack protection) dan runtime defense (guard pages, ASLR). Access control: DAC, MAC, RBAC, ABAC.

**[[W13 - Virtualization]]** — Hypervisor/VMM menjembatani hardware dan VM. Type 1 (bare-metal) vs Type 2 (hosted). Paravirtualization (software-assisted) vs Hardware-assisted virtualization (AMD-V/Intel VT-x). Container BEDA dari VM — berbagi kernel host, jauh lebih ringan. JVM = virtualisasi level aplikasi ("Write Once, Run Anywhere"). Linux VServer = virtualisasi ringan dengan SATU kernel, isolasi lewat chroot/chcontext/chbind/capabilities.

## Glosari Lintas Minggu
| Istilah | Penjelasan | Muncul di |
| --- | --- | --- |
| Interrupt | Sinyal yang memaksa CPU berhenti sejenak menangani kejadian penting | W01, W02, W09 |
| Process Control Block (PCB) | Struktur data terpenting OS, simpan semua info tentang sebuah proses | W02, W05 |
| fork() / exec() | System call UNIX untuk membuat proses baru / mengganti isi proses | W02 |
| Lightweight Process (Thread) | Unit eksekusi di dalam proses, lebih ringan dari proses penuh | W03, W04 |
| UMA / NUMA | Akses memori seragam vs akses lokal lebih cepat dari remote | W04 |
| Turnaround time / Waiting time | Ukuran performa scheduling: total waktu vs waktu tunggu di antrean | W05 |
| Semaphore | Variabel dengan operasi atomik semWait/semSignal untuk sinkronisasi | W06, W07 |
| Critical section | Bagian program yang mengakses shared memory | W06 |
| Deadlock (4 kondisi) | Mutual Exclusion, Hold-and-Wait, No-Preemption, Circular Wait | W07 |
| Banker's Algorithm | Algoritma deadlock avoidance, jaga sistem selalu di Safe State | W07 |
| inode | Struktur kontrol UNIX yang menyimpan info kunci sebuah file | W08 |
| DMA (Direct Memory Access) | Transfer data memori-I/O tanpa membebani CPU per byte | W09 |
| Seek time / Rotational delay | Dua komponen access time disk | W09 |
| RAID | Kombinasi banyak disk untuk performa dan redundansi | W09 |
| Page table / Frame | Struktur pemetaan halaman virtual ke memori fisik | W10, W11 |
| TLB (Translation Lookaside Buffer) | Cache cepat untuk entri page table yang baru dipakai | W10, W11 |
| Page fault | Event ketika halaman yang diakses ternyata tidak ada di memori | W11 |
| Thrashing | Sistem terlalu sibuk swap sampai hampir tidak ada kerja produktif | W11 |
| LRU / FIFO / Clock | Tiga algoritma page replacement dengan trade-off berbeda | W11 |
| Buffer overflow | Data melebihi kapasitas buffer, menimpa memori sekitarnya | W12 |
| DAC / MAC / RBAC / ABAC | Empat model kebijakan access control | W12 |
| Hypervisor (VMM) | Software penjembatan hardware dan Virtual Machine | W13 |
| Container | Virtualisasi level OS, berbagi kernel host, lebih ringan dari VM | W13 |

## Yang Perlu Dicek Sendiri (Slide Kurang Detail)
- **[[W06 - Synchronization dan Inter-process Communication]]**: slide 19 dan 27 gagal diekstrak (gambar rusak di file sumber).
- **[[W07 - Deadlock]]**: slide 7 (contoh deadlock) gagal diekstrak dengan error yang sama.
- **[[W09 - IO Management]]**: detail LENGKAP tiap level RAID (0-6) hanya ada di gambar slide 40, tidak ada teks penjelasan tiap level — wajib dibuka manual untuk ujian. Windows async/sync I/O dan RAID configuration (slide 48-50) juga cuma judul tanpa detail.
- **[[W11 - Virtual Memory]]**: Linux Three-Level Page Table dan Windows Paging region states (slide 39-42) cuma judul tanpa detail teks lengkap.
- **[[W12 - Security]]**: Security Maintenance steps (slide 50) cuma judul tanpa daftar detail; jawaban pasti "kenapa Windows jadi target paling umum?" tidak diberikan definitif oleh slide.
- **[[W13 - Virtualization]]**: banyak slide (Hypervisors detail, Processor Allocation, **Ring 0**, Performance Technology, VMware ESXi Features) cuma judul tanpa detail teks — terutama "Ring 0" penting dipahami untuk privilege level virtualisasi hardware, wajib dibuka manual.
- **Mismatch penomoran**: file sumber [[W13 - Virtualization]] bernama "Week14-Pert13" (bukan "Week13-Pert13") — internal slide tetap konsisten menulis "Session 13".
