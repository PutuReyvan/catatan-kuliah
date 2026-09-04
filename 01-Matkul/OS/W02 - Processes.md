---
matkul: Operating Systems
minggu: 2
sks: 3
sumber: Processes.pptx
tags: [kuliah/os, minggu/w02]
status: draft
diproses: 2026-09-04
---

# W02 — Processes

## Ringkasan
> - **Process** = program yang SEDANG dieksekusi — bukan cuma file program di disk, tapi entitas aktif dengan state saat ini dan sumber daya sistem yang terkait.
> - Proses punya siklus hidup: **dibuat** (4 cara: inisialisasi sistem, system call dari proses lain, request user, batch job), lalu **dijalankan**, lalu **diterminasi** (normal exit, error exit, fatal error, atau dibunuh proses lain).
> - Model state proses berkembang dari **2-state** (running/not running) → **5-state** (New, Ready, Running, Blocked/Waiting, Exit) → dengan tambahan **Suspended state** untuk menangani swapping ke disk.
> - **Process Control Block (PCB)** / process table = struktur data TERPENTING di OS, menyimpan SEMUA informasi tentang sebuah proses.
> - **fork()** dan **exec()** family adalah system call UNIX/Linux fundamental: `fork()` membuat salinan identik proses (child), sementara `exec()` MENGGANTI isi proses yang sedang berjalan dengan program baru.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Process | Program yang sedang dieksekusi, dengan state dan sumber daya sistem terkait |
| Process Control Block (PCB) | Struktur data yang menyimpan semua informasi tentang sebuah proses |
| Swapping | Memindahkan proses dari main memory ke disk (dan sebaliknya) |
| fork() | System call yang membuat salinan identik dari proses yang memanggilnya |
| exec() family | System call yang mengganti isi proses saat ini dengan program baru |
| User Mode / Kernel Mode | Dua level privilese eksekusi CPU — user mode terbatas, kernel mode penuh akses |

## Isi

### Apa Itu Process
**Process** didefinisikan dengan beberapa cara yang saling melengkapi: **program yang sedang dieksekusi**, **instance dari sebuah program yang berjalan di komputer**, **entitas yang bisa ditugaskan ke dan dieksekusi di sebuah processor**, atau **unit aktivitas yang dicirikan oleh eksekusi urutan instruksi, state saat ini, dan sekumpulan sumber daya sistem terkait.**

> [!info] Analogi
> Bedakan "program" dan "process" seperti bedakan RESEP MASAKAN dan KEGIATAN MEMASAK. Resep (program) itu cuma teks statis di kertas — tidak berubah, tidak "melakukan" apa-apa. Begitu kamu mulai MEMASAK mengikuti resep itu (menjalankan program), itu jadi process — ada state saat ini (baru sampai langkah ke berapa), ada sumber daya yang dipakai (kompor, panci, bahan), dan bisa diberhentikan/dilanjutkan.

### Process Creation
Empat alasan/cara sebuah proses dibuat:
1. **System initialization** — proses dibuat saat OS baru menyala.
2. **Execution of a process-creation system call** oleh proses yang sedang berjalan.
3. **User request** untuk membuat proses baru.
4. **Initiation of a batch job**.

### Process Termination
Ada mekanisme yang harus ada agar proses bisa menandakan penyelesaiannya — job batch harus punya instruksi HALT atau system call eksplisit untuk terminasi; untuk aplikasi interaktif, aksi user (log off, keluar aplikasi) menandakan proses selesai. **Empat jenis terminasi:**
1. **Normal exit (voluntary)** — proses selesai dengan wajar.
2. **Error exit (voluntary)** — proses keluar sendiri karena error.
3. **Fatal error (involuntary)** — proses dipaksa berhenti karena error fatal.
4. **Killed by another process (involuntary)** — proses dibunuh oleh proses lain.

### Process Model dan Multiprogramming
Model proses menggambarkan bagaimana beberapa program bisa TERLIHAT berjalan bersamaan padahal sebenarnya bergiliran. Dengan **multiprogramming**, empat program dimuat di memori, tapi konseptual masing-masing berjalan sebagai proses sekuensial independen — pada satu waktu, hanya SATU program yang benar-benar aktif di CPU.

### Model State Proses: Dari 2-State ke 5-State
**Two-state process model** — model paling sederhana: proses hanya punya dua state, **Running** atau **Not Running**.

**Five-state process model** — model yang lebih detail, memisahkan "Not Running" jadi beberapa state berbeda: **New** (baru dibuat), **Ready** (siap dieksekusi, menunggu giliran CPU), **Running** (sedang dieksekusi), **Blocked/Waiting** (menunggu event tertentu, misalnya I/O selesai), dan **Exit** (sudah selesai).

### Suspended Process dan Swapping
**Swapping** melibatkan pemindahan sebagian atau seluruh proses dari main memory ke disk. Ketika TIDAK ADA proses di main memory yang berada di state Ready, OS men-swap salah satu proses yang Blocked keluar ke disk, masuk ke **suspend queue** — antrean proses yang sementara "dikeluarkan" dari main memory. OS kemudian membawa masuk proses lain dari suspend queue atau memenuhi permintaan proses baru. Eksekusi lalu dilanjutkan dengan proses yang baru masuk.

Karena swapping adalah operasi I/O, secara teori berpotensi MEMPERBURUK masalah, bukan memperbaikinya — tapi karena disk I/O umumnya adalah I/O tercepat di sistem, swapping biasanya justru MENINGKATKAN performa keseluruhan.

**Karakteristik proses yang di-suspend:**
- Proses BISA ATAU TIDAK sedang menunggu sebuah event.
- Proses tidak bisa keluar dari state ini sampai agen (dirinya sendiri, parent process, atau OS) secara eksplisit memerintahkan untuk keluar.
- Proses TIDAK segera tersedia untuk dieksekusi.
- Proses ditaruh di state suspended oleh sebuah agen dengan tujuan mencegah eksekusinya.

Model state kemudian berkembang dengan menambahkan **satu atau dua Suspend State** ke diagram 5-state — misalnya "Ready-Suspend" dan "Blocked-Suspend" — untuk membedakan proses yang sedang di-swap tapi masih menunggu I/O vs yang sudah siap dijalankan begitu dibawa kembali ke memori.

> [!info] Analogi
> Suspended process itu seperti buku yang kamu baca lalu TARUH KEMBALI di rak karena kamu sedang tidak punya waktu, meski kamu belum selesai membacanya. Buku itu bukan "hilang" (bukan Exit), tapi juga bukan "sedang kamu baca" (bukan Running) — dia disimpan (di-swap ke "rak" alias disk) sampai kamu punya waktu untuk melanjutkannya lagi. Ada dua kondisi buku yang ditaruh di rak: yang HALAMANNYA masih ditandai siap dibaca kapan saja (Ready-Suspend), dan yang kamu tunggu sesuatu dulu sebelum lanjut baca, misal menunggu terjemahan istilah asing (Blocked-Suspend).

### Process Table / Process Control Block (PCB)
**Process table** atau **Process Control Block** adalah struktur untuk menyimpan informasi tentang sebuah proses. Setiap proses punya SATU entri di tabel ini, dikelola oleh OS.

PCB adalah **struktur data TERPENTING di OS** — berisi SEMUA informasi tentang sebuah proses yang dibutuhkan OS. PCB dibaca dan/atau dimodifikasi oleh HAMPIR SETIAP modul di OS, dan mendefinisikan state OS itu sendiri. Tantangan utamanya BUKAN akses, tapi PROTEKSI — sebuah bug di satu rutin bisa merusak PCB, yang bisa menghancurkan kemampuan sistem mengelola proses yang terpengaruh. Perubahan desain struktur/semantik PCB bisa memengaruhi BANYAK modul di OS.

### User Mode vs Kernel Mode
- **User Mode** — mode dengan hak akses LEBIH RENDAH (less-privileged); program user umumnya berjalan di sini.
- **System Mode / Kernel Mode** — mode dengan hak akses LEBIH TINGGI (more-privileged), juga disebut control mode; ini kernel dari operating system.

### System Interrupts, Time Slice, dan Trap
- **Interrupt** — dipicu event EKSTERNAL dan independen dari proses yang sedang berjalan (contoh: clock interrupt, I/O interrupt, memory fault).
- **Time slice** — jumlah waktu MAKSIMAL yang boleh dipakai sebuah proses sebelum diinterupsi (relevan untuk time-sharing, lihat [[W01 - Introduction to Operating Systems]]).
- **Trap** — kondisi error/exception yang dihasilkan DARI DALAM proses yang sedang berjalan. OS menentukan apakah kondisi ini fatal — kalau fatal, proses dipindah ke state Exit dan terjadi process switch.

### Implementasi di OS Modern: UNIX SVR4
UNIX SVR4 memakai model di mana SEBAGIAN BESAR OS dieksekusi di dalam lingkungan proses user. **System process** berjalan di kernel mode (menjalankan fungsi administratif/housekeeping). **User process** beroperasi di user mode untuk menjalankan program/utility user, tapi bisa berpindah ke kernel mode untuk mengeksekusi instruksi milik kernel — masuk ke kernel mode lewat system call, saat exception terjadi, atau saat interrupt terjadi.

### Process Management System Calls (UNIX/Linux)
**fork()** — membuat proses baru (child):
- Parent dan child dieksekusi SECARA BERSAMAAN (concurrently).
- Setiap proses bisa fork proses lain, menciptakan hierarki proses.
- Proses bisa memilih menunggu (wait) child-nya terminasi.
- Saat `fork()` dieksekusi, itu membuat DUA salinan identik dari address space — kedua proses mulai eksekusi setelahnya, parent dan child berjalan INDEPENDEN.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    pid_t retval;
    retval = fork();
    if (retval == 0) {
        // kode child, print Process ID
        printf("\n Child Process ID: %d", getpid());
        exit;
    } else {
        // kode parent
        if (retval == -1) {
            printf("\n Unable to fork child");  // fork gagal
        } else {
            printf("\n Parent Process ID: %d", getpid());
        }
    }
    return 0;
}
```

**EXEC() family** — mengganti isi proses saat ini dengan program baru. `execl()`/`execlp()` bagian dari `<unistd.h>`:
```c
int execl(const char *path, const char *arg0, ..., const char *argn, char* /*NULL*/);
```
- `execl()` butuh path LENGKAP program.
- `execlp()` cukup butuh NAMA program, akan mencarinya di direktori yang terdaftar di environment variable `PATH`.
- `execvp()` menerima `argv[]` (array pointer karakter yang diakhiri NULL) dan bisa menjalankan file executable biner atau shell script.
- `execvpe()`/`execle()` menambahkan parameter `envp[]` untuk menspesifikasikan environment program yang dieksekusi; `execvpe()` mencari program di `PATH`, sementara `execle()` butuh path lengkap.

```cpp
#include <iostream>
#include <stdlib.h>
using namespace std;
int main() {
    execl("/usr/bin/ls", "ls", "-la", 0);
    cout << "done";
    return 0;
}
// Kode di atas akan mengeksekusi program ls lalu terminasi
```

> [!info] Analogi
> `fork()` dan `exec()` itu dua operasi yang SERING dipakai berpasangan tapi maknanya BEDA JAUH. `fork()` itu seperti FOTOKOPI dirimu sendiri — hasilnya dua orang identik yang lanjut hidup masing-masing secara independen. `exec()` itu seperti seseorang yang TIBA-TIBA berganti identitas total — badannya (proses) sama, tapi "siapa dia" dan apa yang dia kerjakan (program yang dijalankan) benar-benar diganti dari nol. Pola umum di shell UNIX: `fork()` dulu untuk bikin child process, LALU child itu memanggil `exec()` untuk menjalankan program yang berbeda — makanya sering disebut pola "fork-exec".

## Diagram & Visual
- **Slide 9 — Alasan terminasi proses**
  ![[99-Assets/OS/W02-slide09.png]]
- **Slide 10 — Process Model (multiprogramming 4 program)**
  ![[99-Assets/OS/W02-slide10.jpg]]
- **Slide 12 — Modelling Multiprogramming**
  ![[99-Assets/OS/W02-slide12.png]]
- **Slide 13 — Two-State Process Model**
  ![[99-Assets/OS/W02-slide13.png]]
- **Slide 14 — Five-State Process Model**
  ![[99-Assets/OS/W02-slide14.png]]
- **Slide 19 — Process State Diagram dengan Satu Suspend State**
  ![[99-Assets/OS/W02-slide19.png]]
- **Slide 21 — Process State Diagram dengan DUA Suspend State**
  ![[99-Assets/OS/W02-slide21.png]]
- **Slide 23 — Contoh Entri Process Table**
  ![[99-Assets/OS/W02-slide23.jpg]]
- **Slide 30 — UNIX Process State Transition**
  ![[99-Assets/OS/W02-slide30.png]]
- **Slide 33 — Diagram fork()**
  ![[99-Assets/OS/W02-slide33.jpg]]

## Rumus / Sintaks
```c
// fork() - membuat proses child
pid_t retval = fork();
if (retval == 0) { /* kode child */ }
else if (retval == -1) { /* fork gagal */ }
else { /* kode parent */ }

// exec() family - mengganti isi proses dengan program baru
int execl(const char *path, const char *arg0, ..., NULL);
int execlp(const char *file, const char *arg0, ..., NULL);   // cari di PATH
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[], char *const envp[]);
int execle(const char *path, const char *arg, ..., NULL, char *const envp[]);
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Address space** | Rentang alamat memori yang dialokasikan untuk sebuah proses |
| **Process hierarchy** | Struktur bertingkat proses parent-child yang terbentuk lewat fork() berulang |
| **PATH (environment variable)** | Daftar direktori yang dicari sistem saat mencari lokasi sebuah program |
| **Zombie thread/process** | Proses yang sudah selesai tapi resource-nya belum dilepas sepenuhnya oleh parent |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa model 5-state dianggap lebih AKURAT merepresentasikan siklus hidup proses dibanding model 2-state. Skenario APA yang tidak bisa direpresentasikan dengan baik oleh model 2-state (misalnya proses yang menunggu I/O)?
2. **(C4 – Analisis)** Bandingkan proses yang di-swap ke Blocked-Suspend dengan yang di-swap ke Ready-Suspend. Analisis: kenapa OS perlu membedakan KEDUA kondisi ini, bukan cukup satu Suspend state saja?
3. **(C5 – Evaluasi)** Sebuah sistem sering mengalami crash yang mencurigakan berasal dari korupsi Process Control Block. Evaluasi: kenapa slide menyebut "kesulitannya bukan di akses, tapi di proteksi" untuk PCB — apa risiko konkret kalau satu modul OS yang buggy bisa MEMODIFIKASI PCB proses lain secara sembarangan?
4. **(C5 – Evaluasi)** Bandingkan pola "fork() lalu langsung exec()" (umum di shell UNIX) dengan hipotetis "fork() TANPA exec()" (child menjalankan kode yang SAMA PERSIS dengan parent selamanya). Evaluasi: dalam skenario apa pola KEDUA (tanpa exec) tetap berguna, dan dalam skenario apa pola itu jadi tidak efisien?
5. **(C6 – Cipta)** Rancang program C sederhana (pseudo-code atau kode nyata) yang memakai `fork()` untuk membuat SATU child process, di mana child menjalankan `execl()` untuk menjalankan command `ls -la`, sementara parent MENUNGGU child selesai (pakai `wait()` atau logika serupa) sebelum mencetak "Child selesai" ke layar.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W01 - Introduction to Operating Systems]]
- [[W03 - Threads]]
- [[OS - Review dan Glosari]]
