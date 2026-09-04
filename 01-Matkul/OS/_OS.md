---
matkul: Operating Systems
tags: [kuliah/os, moc]
status: draft
diproses: 2026-09-04
---

# Operating Systems — Index

## Daftar Pertemuan
| Minggu | Judul | Ringkasan Singkat |
| --- | --- | --- |
| [[W01 - Introduction to Operating Systems]] | Introduction to Operating Systems | Instruction cycle, interrupt, memory hierarchy, evolusi OS |
| [[W02 - Processes]] | Processes | Process lifecycle, model state, PCB, fork()/exec() |
| [[W03 - Threads]] | Threads | ULT vs KLT vs Hybrid, event-driven server, pthread programming |
| [[W04 - Multiple Processor Systems]] | Multiple Processor Systems | Granularity parallelism, UMA vs NUMA, SMP, distributed system |
| [[W05 - Process Scheduling]] | Process Scheduling | FCFS, SJF, SRT, Round Robin, HRRN, Multilevel Queue, Fair Share |
| [[W06 - Synchronization dan Inter-process Communication]] | Synchronization & IPC | Race condition, mutual exclusion, semaphore, monitor, message passing |
| [[W07 - Deadlock]] | Deadlock | 4 kondisi deadlock, prevention/avoidance/detection, Banker's Algorithm |
| [[W08 - File Systems]] | File Systems | File organization, file allocation, inode, NTFS |
| [[W09 - IO Management]] | I/O Management | Programmed/interrupt-driven/DMA, disk scheduling, RAID |
| [[W10 - Memory Management]] | Memory Management | Partitioning, paging, page table, segmentation |
| [[W11 - Virtual Memory]] | Virtual Memory | Page fault, fetch/replacement policy, LRU/FIFO/Clock, thrashing |
| [[W12 - Security]] | Security | Ancaman keamanan, buffer overflow, access control, OS hardening |
| [[W13 - Virtualization]] | Virtualization | Hypervisor Type 1/2, container, JVM, Linux VServer |

## Rangkuman Ujian
- [[OS - Review dan Glosari]]

## Peta Materi
Matkul ini mengikuti alur klasik textbook OS (Tanenbaum & Bos, *Modern Operating Systems*), membangun dari konsep DASAR sampai topik LANJUTAN:

1. **Fondasi** ([[W01 - Introduction to Operating Systems]]) — memahami hardware dasar (instruction cycle, interrupt, memory hierarchy) sebelum masuk ke konsep OS itu sendiri.
2. **Unit Eksekusi** ([[W02 - Processes]] → [[W03 - Threads]] → [[W04 - Multiple Processor Systems]]) — dari proses (unit dasar eksekusi program) ke thread (unit eksekusi lebih ringan di dalam proses) ke sistem multiprosesor (bagaimana thread/proses berjalan di banyak CPU sekaligus).
3. **Koordinasi Antar Proses** ([[W05 - Process Scheduling]] → [[W06 - Synchronization dan Inter-process Communication]] → [[W07 - Deadlock]]) — bagaimana OS memutuskan proses mana yang jalan (scheduling), bagaimana proses saling berkoordinasi aman (sinkronisasi), dan apa yang terjadi kalau koordinasi itu GAGAL (deadlock).
4. **Manajemen Sumber Daya Fisik** ([[W08 - File Systems]] → [[W09 - IO Management]] → [[W10 - Memory Management]] → [[W11 - Virtual Memory]]) — bagaimana OS mengelola PENYIMPANAN (file system), PERANGKAT I/O, dan MEMORI (dari fisik sampai virtual).
5. **Topik Lanjutan** ([[W12 - Security]] → [[W13 - Virtualization]]) — bagaimana OS melindungi diri dari ancaman, dan bagaimana satu mesin fisik bisa "berpura-pura" jadi banyak mesin sekaligus (virtualisasi).

**Benang merah lintas minggu yang berulang:**
- **Swapping/Suspended state** ([[W02 - Processes]]) menjadi dasar konsep swapping memori di [[W10 - Memory Management]] dan [[W11 - Virtual Memory]] — proses "dikeluarkan sementara" dari memori adalah tema yang berulang di level proses MAUPUN level memori.
- **Locality of reference** muncul di [[W10 - Memory Management]] (menentukan page size optimal) dan [[W11 - Virtual Memory]] (mendasari kenapa Demand Paging dan LRU bekerja baik) — prinsip yang sama dipakai untuk menjelaskan dua fenomena berbeda.
- **Mutual exclusion** yang dipelajari di [[W06 - Synchronization dan Inter-process Communication]] jadi SYARAT PERTAMA dari 4 kondisi deadlock di [[W07 - Deadlock]] — solusi di satu minggu jadi bagian masalah di minggu berikutnya kalau diterapkan tanpa hati-hati.
- **Clock/interrupt** diperkenalkan di [[W01 - Introduction to Operating Systems]], jadi dasar mekanisme Round Robin scheduling di [[W05 - Process Scheduling]], dan muncul lagi sebagai "programmable clock" di [[W09 - IO Management]].
- **Isolasi proses** yang jadi tema besar [[W02 - Processes]] (address space terpisah) muncul lagi dalam wujud berbeda di [[W13 - Virtualization]] — Linux VServer "mengisolasi" seluruh virtual server, bukan cuma proses individual, memakai prinsip yang serupa (chroot untuk file system, chcontext untuk proses).

## Konsep Utama
Kandidat note atomik untuk `02-Konsep/` (belum dibuat, dicatat sebagai referensi):
- Process Control Block (PCB)
- Semaphore dan Mutex
- Deadlock (4 Kondisi)
- Page Table dan Virtual Memory
- RAID
- Buffer Overflow
- Hypervisor Type 1 vs Type 2

## Catatan Pemrosesan
- Sumber 13 file PPT, SEMUA unik (dicek md5, tidak ada duplikat) — beda dari beberapa matkul sebelumnya yang punya duplikat.
- Penomoran W01-W13 dikonfirmasi PASTI lewat teks "Session N" yang eksplisit tertulis di SETIAP slide pertama tiap deck — tidak ada dugaan/inferensi sama sekali.
- `[!]` File sumber [[W13 - Virtualization]] bernama "Week14-Pert13-Virtualization.pptx" — ada mismatch "Week14" vs "Pert13" di nama file, tapi internal slide tetap menulis "Session 13", konsisten dengan urutan kurikulum. Note tetap dinomori W13.
- `[!]` Banyak diagram di deck ini aslinya format **WMF (Windows Metafile)** yang TIDAK BISA ditampilkan Obsidian secara native. SEMUA 102 file WMF berhasil DIKONVERSI ke PNG memakai PIL/Pillow (via Windows GDI) sebelum dimasukkan ke vault — jauh lebih baik dibanding solusi di matkul Forensics sebelumnya yang terpaksa SKIP gambar WMF sepenuhnya.
- Ambang batas ekstraksi gambar 3KB diterapkan dari awal — total **212 gambar** berhasil diekstrak ke `99-Assets/OS/` (matkul dengan jumlah gambar terbanyak sejauh ini).
- `[!]` [[W06 - Synchronization dan Inter-process Communication]] slide 19 dan 27 gagal diekstrak karena file gambar rusak di sumbernya (`required <p:blipFill> child element not present`) — bukan masalah threshold ekstraksi.
- `[!]` [[W07 - Deadlock]] slide 7 juga gagal diekstrak dengan error yang sama.
- Beberapa deck (terutama [[W09 - IO Management]], [[W11 - Virtual Memory]], [[W12 - Security]], [[W13 - Virtualization]]) punya slide berisi HANYA judul tanpa detail teks yang terekstrak — dicatat sebagai `Pertanyaan Terbuka` di masing-masing note.
- SKS ditulis 3 sebagai asumsi administratif, belum dikonfirmasi; nama dosen: **Dr. Zulfany Erlisa Rasjid B.Sc., MMSI**; kode matkul: **COMP6697001**; sumber utama: Tanenbaum & Bos, *Modern Operating Systems 5th ed.* dan Stallings, *Operating Systems: Internals and Design Principles 9th ed.*
- Semua wikilink dan image embed sudah diverifikasi resolve (dicek dengan script Python).
