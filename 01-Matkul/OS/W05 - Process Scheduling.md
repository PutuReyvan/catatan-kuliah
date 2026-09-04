---
matkul: Operating Systems
minggu: 5
sks: 3
sumber: Process Scheduling.pptx
tags: [kuliah/os, minggu/w05]
status: draft
diproses: 2026-09-04
---

# W05 — Process Scheduling

## Ringkasan
> - Proses bergantian antara **CPU burst** (mengerjakan sesuatu di processor) dan **I/O burst** (menunggu perangkat I/O) — proses digolongkan **CPU-bound** (banyak burst CPU panjang) atau **I/O-bound** (sering menunggu I/O).
> - Ada 3 level scheduling: **Long-term** (menentukan proses mana yang MASUK sistem), **Medium-term** (bagian dari swapping), **Short-term/dispatcher** (paling sering jalan, menentukan proses mana yang jalan DETIK INI).
> - 5 kriteria penilaian scheduling: **CPU utilization, throughput, turnaround time, waiting time, response time.**
> - Banyak algoritma scheduling dengan trade-off berbeda: **FCFS** (simpel tapi bisa lama), **SJF/SPN** (optimal secara teori tapi butuh estimasi), **SRT** (versi preemptive SJF), **Round Robin** (adil tapi overhead context switch), **HRRN** (mencegah starvation), **Multilevel Queue** (kelas prioritas berbeda), **Fair Share** (adil per USER, bukan per proses).
> - Ada rumus konkret untuk **waiting time** dan **turnaround time** yang dipakai membandingkan algoritma secara kuantitatif — dan CONTOH PERHITUNGAN LENGKAP untuk tiap algoritma ada di slide.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| CPU-bound process | Proses yang didominasi burst CPU panjang, jarang menunggu I/O |
| I/O-bound process | Proses yang sering menunggu I/O, burst CPU-nya pendek-pendek |
| Long-term scheduling | Menentukan proses mana yang diterima masuk ke sistem |
| Short-term scheduling (dispatcher) | Menentukan proses mana yang dijalankan CPU saat ini juga |
| Turnaround time | Total waktu dari proses masuk sampai selesai dieksekusi |
| Waiting time | Total waktu proses menunggu di ready queue |
| Preemptive vs Non-preemptive | Apakah proses yang sedang jalan BISA disela paksa atau tidak |
| Starvation | Kondisi proses tidak PERNAH dapat giliran CPU karena terus dikalahkan proses lain |

## Isi

### Process Behavior: CPU Burst vs I/O Burst
Proses bergantian antara **burst penggunaan CPU** dan **periode menunggu I/O**. Berdasarkan pola ini, proses digolongkan:
- **CPU-bound process** — didominasi burst CPU yang PANJANG.
- **I/O-bound process** — sering diselingi periode menunggu I/O, burst CPU-nya PENDEK-PENDEK.

### Tiga Level Scheduling
Tujuan scheduling adalah menugaskan proses untuk dieksekusi oleh processor dengan cara yang memenuhi tujuan sistem (response time, throughput, efisiensi processor). Dipecah jadi tiga fungsi terpisah:

1. **Long-Term Scheduling** — menentukan program MANA yang diterima masuk sistem untuk diproses. Mengontrol DERAJAT multiprogramming — semakin banyak proses yang dibuat, semakin kecil persentase waktu yang bisa didapat tiap proses. Bisa membatasi jumlah proses untuk menjaga kualitas layanan bagi proses yang sudah ada.

2. **Medium-Term Scheduling** — bagian dari fungsi swapping (ingat [[W02 - Processes]]). Keputusan swap-in didasarkan pada kebutuhan mengelola derajat multiprogramming, mempertimbangkan kebutuhan memori proses yang di-swap-out.

3. **Short-Term Scheduling (dispatcher)** — dikenal sebagai DISPATCHER, PALING SERING dieksekusi di antara ketiganya. Membuat keputusan HALUS (fine-grained) tentang proses mana yang dijalankan BERIKUTNYA. Dipanggil saat terjadi event yang bisa membuat proses saat ini ter-block, atau saat ada peluang untuk MENYELA (preempt) proses yang sedang berjalan demi proses lain.

> [!info] Analogi
> Tiga level ini seperti tiga tingkat keputusan di sebuah rumah sakit UGD. **Long-term** itu seperti keputusan "berapa banyak pasien BARU yang boleh diterima rumah sakit hari ini" (kontrol jumlah total). **Medium-term** itu seperti keputusan "pasien mana yang dipindah dari ruang rawat penuh ke ruang tunggu sementara" (swapping). **Short-term/dispatcher** itu seperti perawat yang, DETIK INI JUGA, memutuskan pasien mana yang ditangani dokter SELANJUTNYA — keputusan ini dibuat PALING SERING, kadang setiap beberapa detik.

### Kriteria Penilaian Short-Term Scheduling
Lima kriteria utama untuk mengevaluasi kebijakan scheduling:
1. **CPU utilization** — menjaga CPU SESIBUK mungkin.
2. **Throughput** — jumlah proses yang SELESAI dieksekusi per unit waktu.
3. **Turnaround time** — jumlah waktu TOTAL untuk mengeksekusi satu proses tertentu (dari masuk sampai selesai).
4. **Waiting time** — jumlah waktu proses MENUNGGU di ready queue.
5. **Response time** — waktu dari saat request diajukan sampai RESPONS PERTAMA dihasilkan (bukan output selesai) — relevan untuk lingkungan time-sharing.

### Selection Function dan Decision Mode
**Selection function** menentukan proses MANA (di antara proses yang Ready) yang dipilih berikutnya untuk dieksekusi — bisa berdasarkan prioritas, kebutuhan sumber daya, atau karakteristik eksekusi proses. Kalau berdasarkan karakteristik eksekusi, tiga besaran pentingnya:
- **w** = waktu yang sudah dihabiskan menunggu di sistem sejauh ini.
- **e** = waktu yang sudah dihabiskan dalam eksekusi sejauh ini.
- **s** = total waktu servis yang dibutuhkan proses (termasuk e) — umumnya harus diestimasi atau disuplai user.

**Decision mode** menentukan KAPAN selection function dijalankan — dua kategori: **Non-preemptive** (proses yang sedang jalan TIDAK BISA disela sampai selesai/blocking sendiri) dan **Preemptive** (proses yang sedang jalan BISA disela paksa).

**Kategori kriteria scheduling berdasarkan jenis sistem:** Batch, Interactive, Real Time.

### Algoritma Scheduling

**1. First Come First Serve (FCFS / FIFO)**
Kebijakan scheduling PALING SEDERHANA — proses yang paling lama berada di ready queue dipilih untuk dijalankan. Performa jauh LEBIH BAIK untuk proses PANJANG daripada proses PENDEK, dan cenderung FAVORIT ke proses CPU-bound dibanding I/O-bound.

*Contoh perhitungan:* Proses P1(burst=24), P2(burst=3), P3(burst=3), tiba berurutan P1→P2→P3.
```
Gantt Chart:  | P1 (0-24) | P2 (24-27) | P3 (27-30) |
Waiting time: P1=0, P2=24, P3=27
Average waiting time = (0+24+27)/3 = 17
```

**2. Shortest Process Next / Shortest Job First (SPN/SJF)**
Kebijakan NON-PREEMPTIVE — proses dengan waktu eksekusi TERPENDEK yang diharapkan dipilih berikutnya. Proses pendek "melompat" ke depan antrean. Ada RISIKO STARVATION untuk proses panjang. Kesulitannya: harus TAHU atau ESTIMASI waktu proses tiap proses — kalau estimasi programmer jauh di BAWAH waktu aktual, sistem bisa membatalkan job tersebut.

*Contoh perhitungan (SJF non-preemptive):*
```
Process | Arrival | Burst
P1      | 0.0     | 7
P2      | 2.0     | 4
P3      | 4.0     | 1
P4      | 5.0     | 4

Gantt Chart: | P1 (0-7) | P3 (7-8) | P2 (8-12) | P4 (12-16) |
Average waiting time = (0+6+3+7)/4 = 4
```

**3. Shortest Remaining Time (SRT)**
Versi PREEMPTIVE dari SPN — scheduler SELALU memilih proses dengan sisa waktu eksekusi TERPENDEK. Ada risiko starvation proses panjang. Seharusnya memberikan performa turnaround time LEBIH BAIK daripada SPN, karena job pendek langsung diprioritaskan bahkan menyela job panjang yang sedang jalan.

*Contoh perhitungan (SJF preemptive, data sama seperti di atas):*
```
Average waiting time = (9+1+0+2)/4 = 3   <- lebih baik dari SJF non-preemptive (4)
```

**4. Round Robin**
Memakai preemption berbasis CLOCK — juga disebut TIME SLICING karena setiap proses diberi SEPOTONG waktu sebelum disela. Isu desain utamanya: PANJANG time quantum/slice yang dipakai. Sangat efektif untuk sistem time-sharing/transaction-processing umum. Kekurangan: perlakuan relatifnya terhadap proses CPU-bound vs I/O-bound (proses I/O-bound sering "kehilangan" sisa quantum-nya karena keburu menunggu I/O).

*Contoh: Round Robin dengan time slice=20, 4 proses dengan burst time dan arrival time berbeda-beda* — hasil Gantt Chart menunjukkan proses saling bergantian tiap 20 unit waktu (lihat gambar).

**5. Highest Response Ratio Next (HRRN)**
Memilih proses berikutnya dengan RASIO TERTINGGI. Menarik karena memperhitungkan USIA (age) proses — meski job pendek tetap diuntungkan, proses yang MENUA tanpa dilayani akan meningkatkan rasionya sehingga akhirnya bisa MELEWATI job pendek yang lebih baru datang. Ini mekanisme anti-starvation yang elegan.

**6. Multilevel Queue**
Proses bisa dibagi jadi KELAS berbeda dengan kebutuhan scheduling berbeda. Proses dibagi ke beberapa antrean dengan LEVEL PRIORITAS berbeda (tinggi ke rendah). Preemption diizinkan. Algoritma scheduling BERBEDA bisa dipakai untuk antrean berbeda. Dengan mekanisme FEEDBACK, proses yang sudah lama menunggu di level prioritas rendah bisa DINAIKKAN prioritasnya.

**7. Multilevel Queue dengan Feedback Mechanism**
Memakai banyak antrean dengan prioritas berbeda, memungkinkan proses BERPINDAH antrean berdasarkan penggunaan CPU-nya (feedback). Menguntungkan task I/O-bound, mencegah starvation lewat priority boost, dan menyeimbangkan responsiveness dengan throughput. Algoritma di antrean berbeda bisa berbeda-beda.

**8. Fair Share Scheduling**
Keputusan scheduling didasarkan pada KUMPULAN proses (per user), bukan per proses individual. Setiap USER diberi jatah (share) processor. Tujuannya memonitor pemakaian untuk memberi LEBIH SEDIKIT sumber daya ke user yang sudah memakai LEBIH dari jatah adilnya, dan LEBIH BANYAK ke yang memakai KURANG dari jatahnya.

*Contoh:* 4 user (A, B, C, D) masing-masing 1 proses → masing-masing dapat 25%. Kalau B menambah proses KEDUA, A, B, C, D TETAP dapat 25% masing-masing SEBAGAI USER — tapi kedua proses B berbagi 25% itu, jadi masing-masing proses B cuma dapat 12,5%, MENJAGA KEADILAN antar user (bukan antar proses).

> [!info] Analogi
> Fair Share Scheduling itu seperti membagi kue ulang tahun berdasarkan JUMLAH KELUARGA yang datang, BUKAN jumlah orang. Kalau keluarga A datang 1 orang dan keluarga B datang 3 orang, kue TETAP dibagi rata per KELUARGA (masing-masing keluarga dapat porsi kue yang SAMA BESAR) — tapi di dalam keluarga B, potongan kue keluarga itu dibagi lagi jadi 3 untuk ketiga anggotanya, jadi masing-masing orang di keluarga B dapat POTONGAN LEBIH KECIL daripada satu-satunya orang di keluarga A. Ini mencegah satu keluarga yang datang RAMAI-RAMAI "memonopoli" kue lebih dari keluarga lain.

### Thread Scheduling
Slide membandingkan scheduling **user-level thread** dan **kernel-level thread** (lihat kembali [[W03 - Threads]]) — dengan process quantum 50ms dan thread yang berjalan 5ms per CPU burst, pola penjadwalannya BERBEDA tergantung apakah kernel MENYADARI thread individual (kernel-level) atau tidak (user-level).

### Contoh Program: Menghitung Waiting Time dan Turnaround Time (FCFS)
```c
#include <stdio.h>

void findWaitingTime(int processes[], int n, int bt[], int wt[]) {
    wt[0] = 0;
    for (int i = 1; i < n; i++)
        wt[i] = bt[i-1] + wt[i-1];
}

void findTurnAroundTime(int processes[], int n, int bt[], int wt[], int tat[]) {
    for (int i = 0; i < n; i++)
        tat[i] = bt[i] + wt[i];
}

void findavgTime(int processes[], int n, int bt[]) {
    int wt[n], tat[n], total_wt = 0, total_tat = 0;
    findWaitingTime(processes, n, bt, wt);
    findTurnAroundTime(processes, n, bt, wt, tat);
    printf("Processes   Burst time   Waiting time   Turn around time\n");
    for (int i = 0; i < n; i++) {
        total_wt += wt[i];
        total_tat += tat[i];
        printf("   %d        %d        %d        %d\n", i+1, bt[i], wt[i], tat[i]);
    }
    printf("Average waiting time = %f\n", (float)total_wt / n);
    printf("Average turn around time = %f\n", (float)total_tat / n);
}

int main() {
    int processes[] = {1, 2, 3, 4};
    int n = sizeof processes / sizeof processes[0];
    int burst_time[] = {8, 5, 8, 2};
    findavgTime(processes, n, burst_time);
    return 0;
}
```

## Diagram & Visual
- **Slide 4 — CPU-bound vs I/O-bound process behavior**
  ![[99-Assets/OS/W05-slide04.jpg]]
- **Slide 5 — Jenis-jenis scheduling**
  ![[99-Assets/OS/W05-slide05.png]]
- **Slide 7 — Scheduling and Process State**
  ![[99-Assets/OS/W05-slide07.png]]
- **Slide 18 — Tujuan algoritma scheduling di berbagai kondisi**
  ![[99-Assets/OS/W05-slide18.jpg]]
- **Slide 31 — Thread Scheduling: User-level vs Kernel-level**
  ![[99-Assets/OS/W05-slide31.jpg]]

## Rumus / Sintaks
```
Waiting time (FCFS) = burst time proses sebelumnya + waiting time proses sebelumnya
Turnaround time = burst time + waiting time
Average waiting time = total semua waiting time / jumlah proses
Average turnaround time = total semua turnaround time / jumlah proses

Contoh FCFS: P1(24), P2(3), P3(3) -> avg waiting = (0+24+27)/3 = 17
Contoh SJF non-preemptive -> avg waiting = 4
Contoh SJF preemptive (SRT) -> avg waiting = 3  (lebih baik dari non-preemptive)
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Gantt Chart** | Diagram batang horizontal yang menunjukkan urutan dan durasi eksekusi tiap proses |
| **Aging (dalam scheduling)** | Teknik menaikkan prioritas proses yang sudah lama menunggu, untuk mencegah starvation |
| **Time quantum** | Potongan waktu tetap yang dialokasikan ke satu proses dalam Round Robin |
| **Response ratio (HRRN)** | Ukuran yang menggabungkan waktu tunggu dan waktu servis untuk menentukan prioritas |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa FCFS "cenderung favorit ke proses CPU-bound dibanding I/O-bound". Hubungkan dengan definisi CPU-bound/I/O-bound process di awal materi — apa yang terjadi pada proses I/O-bound yang harus MENUNGGU proses CPU-bound panjang selesai dulu di FCFS?
2. **(C4 – Analisis)** Bandingkan SJF (non-preemptive) dan SRT (preemptive) memakai contoh angka di slide (average waiting time 4 vs 3). Analisis: KAPAN preemption terjadi di contoh SRT itu — proses apa yang MENYELA proses apa, dan kenapa itu menurunkan average waiting time?
3. **(C5 – Evaluasi)** Sebuah sistem time-sharing memilih Round Robin dengan time quantum SANGAT KECIL (misalnya 1ms). Evaluasi: apa dampak negatif dari quantum yang terlalu kecil terhadap overhead sistem (pertimbangkan biaya context switch), meski secara teori response time-nya akan sangat cepat?
4. **(C5 – Evaluasi)** Bandingkan HRRN dan Multilevel Queue dengan Feedback sebagai DUA solusi berbeda untuk masalah STARVATION. Evaluasi: mekanisme MANA yang lebih SEDERHANA diimplementasikan, dan trade-off apa yang mungkin hilang dari kesederhanaan itu?
5. **(C6 – Cipta)** Diberikan 4 proses: P1(arrival=0, burst=6), P2(arrival=1, burst=3), P3(arrival=2, burst=8), P4(arrival=3, burst=2). Hitung Gantt Chart dan average waiting time untuk (a) FCFS dan (b) SJF non-preemptive. Bandingkan hasil kedua algoritma dan simpulkan mana yang lebih baik untuk kasus ini.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W04 - Multiple Processor Systems]]
- [[W06 - Synchronization dan Inter-process Communication]]
- [[OS - Review dan Glosari]]
