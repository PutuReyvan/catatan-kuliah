---
matkul: Operating Systems
minggu: 3
sks: 3
sumber: Threads.pptx
tags: [kuliah/os, minggu/w03]
status: draft
diproses: 2026-09-04
---

# W03 — Threads

## Ringkasan
> - **Thread** = "lightweight process" — unit eksekusi DI DALAM sebuah proses, memungkinkan beberapa tugas dikerjakan secara simultan dalam satu proses yang sama (contoh nyata: browser bisa render halaman SEKALIGUS download file).
> - Tiga cara implementasi thread: **User Level Thread (ULT)** — dikelola sepenuhnya oleh aplikasi, kernel tidak tahu apa-apa; **Kernel Level Thread (KLT)** — dikelola langsung oleh kernel; dan **Hybrid** — kombinasi keduanya (contoh: Solaris).
> - Kelemahan besar ULT: kalau SATU thread melakukan blocking system call, SEMUA thread dalam proses itu ikut ter-block, dan tidak bisa memanfaatkan multiprocessing.
> - Setiap OS (Windows, Solaris, Linux, Android) punya model dan terminologi thread/proses sendiri-sendiri, meski konsep dasarnya sama.
> - Programming thread di POSIX (pthread) memakai fungsi `pthread_create()`, `pthread_self()`, dan `pthread_join()` untuk membuat, mengidentifikasi, dan menyinkronkan thread.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Thread | Unit eksekusi di dalam sebuah proses, disebut juga "lightweight process" |
| User Level Thread (ULT) | Thread yang dikelola sepenuhnya oleh aplikasi, kernel tidak menyadarinya |
| Kernel Level Thread (KLT) | Thread yang dikelola langsung oleh kernel OS |
| Event-driven server | Server berbasis finite-state machine yang merespons event, tanpa banyak thread |
| pthread | POSIX Thread — standar API pemrograman thread di UNIX/Linux |

## Isi

### Apa Itu Thread
**Thread** adalah unit eksekusi DI DALAM sebuah proses — sebuah "sequence of streams" dalam satu proses, juga dikenal sebagai **"Lightweight Process"**. Thread memungkinkan beberapa tugas dikerjakan SECARA SIMULTAN dalam satu proses yang sama.

> [!info] Analogi
> Kalau process itu seperti sebuah RESTORAN (punya dapur sendiri, kasir sendiri, seluruh sumber daya sendiri), thread itu seperti KOKI-KOKI yang bekerja di DAPUR YANG SAMA. Mereka berbagi peralatan dan bahan (memori dan sumber daya proses yang sama), tapi masing-masing bisa mengerjakan tugas berbeda secara bersamaan — satu koki motong sayur, satu lagi masak kuah, sementara "restoran" (proses)-nya tetap SATU. Bandingkan dengan membuka restoran BARU (fork process baru) yang berarti dapur, peralatan, semuanya dibangun dari nol dan terpisah total.

### Kegunaan Thread
Contoh nyata pemakaian thread: di **Browser** (satu thread merender halaman, thread lain mengunduh gambar/file secara bersamaan) dan di **Word-processor** (satu thread menangani pengetikan/tampilan, thread lain melakukan spell-check di latar belakang).

### Single-Threaded vs Multi-Threaded
Diagram membandingkan model **Single Threaded** (satu proses = satu alur eksekusi) dengan **Multi-threaded** (satu proses = banyak alur eksekusi yang berbagi ruang alamat yang sama).

### Tiga Cara Implementasi Thread

**1. User Level Thread (ULT)**
Seluruh manajemen thread dilakukan oleh APLIKASI — kernel TIDAK MENYADARI keberadaan thread-thread ini sama sekali.

*Kekurangan ULT:*
- Di sebagian besar OS, banyak system call bersifat blocking. Akibatnya, saat SATU ULT mengeksekusi system call yang blocking, TIDAK HANYA thread itu yang ter-block — SEMUA thread dalam proses itu ikut ter-block juga.
- Dalam strategi ULT murni, aplikasi multithreaded TIDAK BISA memanfaatkan multiprocessing — kernel menugaskan satu proses ke satu processor pada satu waktu, jadi hanya SATU thread dalam proses yang bisa dieksekusi pada satu waktu.

**2. Kernel Level Thread (KLT)**
Manajemen thread dilakukan LANGSUNG oleh kernel — tidak ada kode manajemen thread di level aplikasi, cukup sebuah API ke fasilitas kernel thread. **Windows** adalah contoh pendekatan ini.

*Kelebihan KLT:*
- Kernel bisa MENJADWALKAN SIMULTAN beberapa thread dari proses yang sama di beberapa processor sekaligus.
- Kalau satu thread dalam sebuah proses ter-block, kernel bisa menjadwalkan thread LAIN dari proses yang sama.
- Rutin kernel itu sendiri bisa bersifat multithreaded.

*Kekurangan KLT:*
- Transfer kontrol dari satu thread ke thread lain DALAM proses yang sama membutuhkan mode switch ke kernel — ini lebih MAHAL (secara latency) dibanding switching di level ULT.

**3. Hybrid (Kombinasi)**
Pembuatan thread dilakukan SEPENUHNYA di user space, begitu juga sebagian besar penjadwalan dan sinkronisasi thread dalam sebuah aplikasi — tapi tetap memanfaatkan kernel thread di bawahnya lewat multiplexing. **Solaris** adalah contoh yang baik dari pendekatan ini.

> [!info] Analogi
> Bayangkan ULT seperti sebuah TIM KECIL yang mengatur jadwal kerja mereka sendiri TANPA sepengetahuan manajer perusahaan (kernel) — cepat dan fleksibel, tapi kalau SATU anggota tim terjebak macet di jalan (blocking call), manajer perusahaan menganggap SELURUH TIM sedang tidak bekerja (karena manajer tidak tahu detail internal tim). KLT sebaliknya: manajer perusahaan TAHU PERSIS setiap anggota tim dan bisa mengatur mereka langsung ke ruang kerja berbeda (multiprocessor) — tapi setiap kali mau koordinasi ganti tugas, harus lapor dulu ke manajer (mode switch ke kernel), yang makan waktu. Hybrid itu tim yang punya OTONOMI mengatur diri sendiri sehari-hari, tapi manajer TETAP TAHU keberadaan mereka secara garis besar lewat kernel thread di baliknya.

### Event-Driven Servers
Tergantung kasusnya, thread bisa dianggap "mahal", terutama saat throughput tinggi dibutuhkan. Desain event-driven mengikuti **finite-state machine**, di mana sekumpulan event yang terjadi bisa mengubah state. Desain ini dipakai dalam paradigma pemrograman event-driven: server diimplementasikan sebagai finite-state machine yang merespons event, berinteraksi dengan OS memakai system call NON-BLOCKING (asynchronous) — sehingga tidak perlu banyak thread untuk menangani banyak koneksi sekaligus.

### Windows Process and Thread Management
Sebuah aplikasi Windows terdiri dari satu atau lebih PROSES. Setiap proses menyediakan sumber daya yang dibutuhkan untuk menjalankan sebuah program. **Thread** adalah entitas dalam proses yang bisa dijadwalkan untuk eksekusi. Konsep terkait lainnya:
- **Job object** — memungkinkan sekelompok proses dikelola sebagai satu unit.
- **Thread pool** — kumpulan worker thread yang secara efisien mengeksekusi callback asynchronous atas nama aplikasi.
- **Fiber** — unit eksekusi yang harus dijadwalkan SECARA MANUAL oleh aplikasi.
- **User-mode scheduling (UMS)** — mekanisme ringan yang memungkinkan aplikasi menjadwalkan thread-nya sendiri.

### Solaris Process
Solaris memakai EMPAT konsep terkait thread, termasuk **Lightweight Process (LWP)**. Struktur data LWP mencakup: identifier LWP, prioritas LWP (dan kernel thread yang mendukungnya), signal mask, nilai register user-level yang disimpan, kernel stack untuk LWP tersebut, data penggunaan sumber daya/profiling, pointer ke kernel thread terkait, dan pointer ke struktur proses.

### LINUX Tasks
LINUX memakai istilah **task** untuk merepresentasikan proses maupun thread dalam struktur data yang sama, dengan model process-thread tertentu (lihat diagram).

### Android Process and Thread Management
Aplikasi Android terdiri dari satu atau lebih instance dari EMPAT jenis komponen aplikasi, masing-masing bisa diaktifkan secara independen bahkan oleh aplikasi lain:
1. **Activities**
2. **Services**
3. **Content providers**
4. **Broadcast receivers**

### Thread Programming (POSIX pthread)
```c
// Membuat thread
int pthread_create(pthread_t *obj, pthread_attr_t *attr, void *(*func)(void*), void *arg);
```
- `obj` — pointer ke struktur `pthread_t`, menyimpan ID thread yang dibuat.
- `attr` — pointer ke `pthread_attr_t`, dipakai untuk mengatur properti khusus thread (scheduling, priority). Kalau NULL, thread dibuat dengan properti default.
- `func` — pointer ke fungsi yang akan dijalankan thread (hanya boleh menerima parameter `void*`).
- `arg` — argumen `void*` yang diteruskan ke `func`.
- Return 0 kalau sukses.

```c
pthread_self();  // mendapatkan ID thread saat ini
pthread_join();  // menunggu (join/rejoin) alur eksekusi thread lain
                  // thread pemanggil di-suspend sampai target thread selesai
                  // melepaskan resource, mencegah "zombie thread"
```

**Contoh program lengkap:**
```cpp
#include <iostream>
#include <pthread.h>
using namespace std;

void* task1(void* x) {
    int* temp = (int*) x;
    for (int count = 0; count < *temp; count++) {
        cout << "Thread A" << endl;
    }
    cout << "Thread A Complete" << endl;
    return NULL;
}

int main(int argc, char** argv) {
    pthread_t thread_a, thread_b;
    int N;
    if (argc != 2) {
        cout << "Error" << endl;
        return 0;
    }
    N = atoi(argv[1]);
    pthread_create(&thread_a, NULL, task1, &N);
    pthread_create(&thread_b, NULL, task2, &N);
    cout << "Waiting to join" << endl;
    pthread_join(thread_a, NULL);
    pthread_join(thread_b, NULL);
    return 0;
}
```

## Diagram & Visual
- **Slide 7 — Kegunaan Thread di Browser dan Word-processor**
  ![[99-Assets/OS/W03-slide07.png]]
  ![[99-Assets/OS/W03-slide07a.png]]
- **Slide 8-9 — Single vs Multi-Threaded**
  ![[99-Assets/OS/W03-slide08.png]]
  ![[99-Assets/OS/W03-slide09.png]]
- **Slide 10-12 — Thread Implementation & User Level Thread (ULT)**
  ![[99-Assets/OS/W03-slide10.png]]
  ![[99-Assets/OS/W03-slide11.png]]
  ![[99-Assets/OS/W03-slide11a.png]]
  ![[99-Assets/OS/W03-slide12.png]]
- **Slide 16 — Kernel Level Thread (KLT)**
  ![[99-Assets/OS/W03-slide16.png]]
- **Slide 18 — Tabel Latency Thread vs Process Operation**
  ![[99-Assets/OS/W03-slide18.png]]
- **Slide 19 — Hybrid: Multiplexing user-level threads onto kernel-level threads**
  ![[99-Assets/OS/W03-slide19.jpg]]
- **Slide 20 — Relasi Thread dan Process**
  ![[99-Assets/OS/W03-slide20.png]]
- **Slide 22 — Tiga Cara Membangun Server**
  ![[99-Assets/OS/W03-slide22.jpg]]
- **Slide 27 — Windows Thread States**
  ![[99-Assets/OS/W03-slide27.png]]
- **Slide 30 — Solaris Thread States**
  ![[99-Assets/OS/W03-slide30.png]]
- **Slide 32-33 — Linux Process-Thread Model**
  ![[99-Assets/OS/W03-slide32.png]]
  ![[99-Assets/OS/W03-slide33.png]]
- **Slide 35-36 — Android Process/Thread dan State Transition Diagram**
  ![[99-Assets/OS/W03-slide35.png]]
  ![[99-Assets/OS/W03-slide36.png]]
- **Slide 37-39 — Contoh Kode Thread Programming**
  ![[99-Assets/OS/W03-slide37.png]]
  ![[99-Assets/OS/W03-slide38.png]]
  ![[99-Assets/OS/W03-slide39.png]]

## Rumus / Sintaks
```c
// POSIX pthread API dasar
int pthread_create(pthread_t *obj, pthread_attr_t *attr, void *(*func)(void*), void *arg);
pthread_t pthread_self(void);
int pthread_join(pthread_t thread, void **retval);
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Lightweight Process (LWP)** | Istilah lain untuk thread, menekankan bahwa overhead-nya lebih ringan dari proses penuh |
| **Finite-state machine** | Model komputasi dengan sejumlah state terbatas, berubah berdasarkan event |
| **Fiber (Windows)** | Unit eksekusi yang dijadwalkan MANUAL oleh aplikasi, bukan oleh kernel |
| **Non-blocking / asynchronous system call** | System call yang tidak menghentikan eksekusi program menunggu hasil, dipakai di event-driven server |
| **Multiplexing (thread)** | Memetakan banyak user-level thread ke sejumlah lebih sedikit kernel-level thread |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa kelemahan ULT ("satu thread blocking, semua thread ikut blocking") TIDAK terjadi pada KLT. Hubungkan jawabanmu dengan fakta bahwa kernel "menyadari" keberadaan thread di KLT tapi TIDAK di ULT.
2. **(C4 – Analisis)** Bandingkan pendekatan Event-Driven Server (finite-state machine, non-blocking) dengan pendekatan multi-threaded server biasa. Analisis: dalam skenario throughput SANGAT TINGGI (ribuan koneksi bersamaan), kenapa event-driven server bisa lebih efisien daripada membuat ribuan thread?
3. **(C5 – Evaluasi)** Sebuah aplikasi butuh performa maksimal di sistem MULTIPROCESSOR (banyak core CPU). Evaluasi: dari tiga pendekatan (ULT, KLT, Hybrid), mana yang PALING TIDAK cocok untuk skenario ini, dan jelaskan alasannya berdasarkan poin "kekurangan ULT" di materi.
4. **(C5 – Evaluasi)** Bandingkan overhead `pthread_create()` (membuat thread baru) dengan `fork()` (membuat proses baru, dari [[W02 - Processes]]). Evaluasi: kenapa istilah "lightweight process" dipakai untuk thread — apa yang membuatnya secara struktural LEBIH RINGAN dibanding fork() sebuah proses penuh?
5. **(C6 – Cipta)** Rancang program pthread (kode nyata atau pseudo-code) yang membuat DUA thread: thread pertama menghitung jumlah angka genap dari 1 sampai N, thread kedua menghitung jumlah angka ganjil dari 1 sampai N, lalu program utama MENUNGGU kedua thread selesai (pakai `pthread_join()`) sebelum mencetak kedua hasilnya.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W02 - Processes]]
- [[W04 - Multiple Processor Systems]]
- [[OS - Review dan Glosari]]
