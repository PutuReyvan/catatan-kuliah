---
matkul: Operating Systems
minggu: 6
sks: 3
sumber: Synchronization and Inter-process Communication.pptx
tags: [kuliah/os, minggu/w06]
status: draft
diproses: 2026-09-04
---

# W06 — Synchronization dan Inter-process Communication

## Ringkasan
> - **Concurrency** (proses berjalan "bersamaan") menimbulkan masalah karena kecepatan relatif eksekusi proses TIDAK BISA diprediksi — bergantung aktivitas proses lain, cara OS menangani interrupt, dan kebijakan scheduling.
> - **Race condition** terjadi ketika beberapa proses/thread membaca-menulis data yang sama, dan hasil akhirnya bergantung pada URUTAN eksekusi — proses yang update TERAKHIR yang "menang".
> - **Critical section** = bagian program yang mengakses shared memory. **Mutual exclusion** = aturan bahwa cuma SATU proses boleh berada di critical section-nya pada satu waktu, untuk mencegah race condition.
> - Solusi mutual exclusion berkembang dari yang PALING KASAR (disable interrupt — cuma jalan di uniprocessor) sampai yang PALING ELEGAN: **Peterson's Solution** (software, busy waiting), **TSL instruction** (hardware atomic), **Semaphore** (dengan queue, tanpa busy waiting), dan **Monitor** (construct bahasa pemrograman, lebih mudah dikontrol daripada semaphore).
> - **Semaphore** dipakai untuk menyelesaikan masalah IPC klasik: **Producer-Consumer Problem** dan **Readers-Writers Problem**.
> - Selain shared memory, ada **Message Passing** (send/receive primitives) dan **Signal** (notifikasi async ke proses, seperti SIGINT/SIGKILL) sebagai mekanisme komunikasi/sinkronisasi antar proses.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Concurrency | Beberapa proses berjalan "bersamaan" (interleaved atau overlapped) |
| Race condition | Hasil akhir program bergantung pada URUTAN eksekusi proses yang tidak bisa diprediksi |
| Critical section | Bagian program yang mengakses shared memory/resource |
| Mutual exclusion | Aturan hanya SATU proses boleh berada di critical section pada satu waktu |
| Semaphore | Variabel integer dengan operasi atomik semWait/semSignal untuk sinkronisasi |
| Monitor | Construct bahasa pemrograman untuk sinkronisasi, lebih mudah dikontrol dari semaphore |
| Message Passing | Komunikasi antar proses lewat primitif send()/receive() |
| Signal | Notifikasi asinkron yang dikirim ke sebuah proses (misal SIGINT, SIGKILL) |

## Isi

### Concurrency: Kenapa dan Kesulitannya
**Interleaving** (bergantian) dan **overlapping** (tumpang tindih) adalah dua bentuk pemrosesan konkuren. Keduanya punya masalah yang SAMA: di sistem uniprocessor, kecepatan RELATIF eksekusi proses tidak bisa diprediksi — bergantung pada aktivitas proses lain, cara OS menangani interrupt, dan kebijakan scheduling OS.

**Kesulitan dalam concurrency:**
- Berbagi sumber daya GLOBAL.
- Sulit bagi OS mengelola alokasi sumber daya secara OPTIMAL.
- Sulit MENEMUKAN bug pemrograman karena hasilnya TIDAK deterministik dan TIDAK bisa direproduksi dengan mudah.

> [!info] Analogi
> Race condition itu seperti dua orang MENULIS di satu buku catatan bersama secara bersamaan tanpa koordinasi. Kalau orang A menulis "Beli susu" dan orang B, hampir bersamaan, menulis "Beli telur" DI BARIS YANG SAMA — hasil akhirnya tergantung SIAPA yang menulis TERAKHIR, catatan orang pertama bisa TERTIMPA tanpa disadari. Bug seperti ini SUSAH dilacak karena kadang muncul, kadang tidak — tergantung siapa yang "menang" menulis duluan di momen tertentu, yang bisa berbeda-beda setiap kali dijalankan.

### Resource Competition vs Cooperation
Proses konkuren bisa berhubungan dengan dua cara:
- **Resource competition** — proses konkuren berkonflik saat BERSAING memakai sumber daya yang sama (misal perangkat I/O, memori, waktu processor, clock).
- **Cooperation by sharing** — proses saling bekerja sama lewat berbagi sumber daya.
- **Cooperation by communication** — berbagai proses berpartisipasi dalam satu usaha bersama yang menghubungkan semua proses. Komunikasi menyediakan cara untuk sinkronisasi/koordinasi aktivitas. Biasanya lewat pesan (messages). Mutual exclusion BUKAN kebutuhan kontrol untuk jenis kerja sama ini — tapi masalah DEADLOCK dan STARVATION tetap ada.

### Critical Section dan Syarat Mutual Exclusion
**Critical section** adalah bagian program di mana shared memory diakses atau hal kritis lain dilakukan yang bisa memicu race. **Mutual exclusion harus ditegakkan**: hanya SATU proses pada satu waktu boleh masuk critical section-nya, di antara semua proses yang punya critical section untuk resource/objek yang sama.

**Empat syarat dasar (Tanenbaum):**
1. Tidak boleh ada DUA proses secara BERSAMAAN berada di dalam critical region masing-masing.
2. Tidak boleh ada ASUMSI soal kecepatan atau jumlah CPU.
3. Proses yang berjalan DI LUAR critical region-nya tidak boleh MEMBLOKIR proses lain.
4. Tidak boleh ada proses yang harus menunggu SELAMANYA untuk masuk critical region-nya.

**Enam syarat lengkap (Stallings):** ditegakkannya mutual exclusion; proses yang berhenti harus berhenti TANPA mengganggu proses lain; proses yang butuh akses critical section TIDAK BOLEH ditunda TANPA BATAS (tidak boleh deadlock/starvation); saat TIDAK ADA proses di critical section, proses yang minta masuk harus DIIZINKAN masuk TANPA delay; tidak ada asumsi soal kecepatan relatif proses/jumlah proses; proses hanya berada di critical section-nya untuk waktu yang TERBATAS.

### Solusi Mutual Exclusion: Disabling Interrupts
Di sistem uniprocessor, proses konkuren TIDAK BISA overlap eksekusinya — cuma bisa interleaved. Sebuah proses akan terus berjalan sampai memanggil layanan OS atau di-interrupt. Karena itu, untuk menjamin mutual exclusion, CUKUP mencegah proses di-interrupt — lewat primitif kernel untuk disable/enable interrupt.

**Kekurangan:**
- Efisiensi eksekusi bisa MENURUN NYATA karena processor terbatas kemampuannya menyelingi (interleave) proses.
- Pendekatan ini TIDAK BEKERJA di arsitektur MULTIPROCESSOR (karena disable interrupt di satu CPU tidak mencegah CPU lain mengakses resource yang sama).

### Solusi Software: Strict Alternation dan Peterson's Solution
Dijkstra melaporkan algoritma mutual exclusion untuk dua proses, dirancang matematikawan Belanda **Dekker**. Pendekatan software berkembang bertahap:
- **Strict Alternation** — dua proses bergantian STRIK memakai giliran, meski salah satu proses tidak butuh critical section, dia tetap harus menunggu gilirannya (tidak efisien).
- **Peterson's Solution** — solusi software klasik yang menjamin mutual exclusion untuk dua proses TANPA butuh dukungan hardware khusus, memakai kombinasi flag dan variabel "turn".

Kedua pendekatan ini memakai **busy waiting** — proses terus-menerus mengecek kondisi dalam sebuah loop (CPU tetap sibuk) sambil menunggu giliran masuk critical section.

### Solusi Hardware: TSL Instruction dan Compare & Swap
- **TSL (Test-and-Set-Lock) Instruction** — instruksi hardware ATOMIK yang membaca dan mengubah nilai memori dalam SATU operasi tak terputus, dipakai untuk implementasi lock.
- **Compare & Swap Instruction** — juga disebut "compare and exchange instruction". Perbandingan dilakukan antara nilai memori dan nilai test; kalau SAMA, terjadi swap. Dilakukan secara ATOMIK (tidak bisa disela/diinterupsi).

> [!info] Konteks tambahan (bukan dari slide)
> "Atomik" di sini artinya operasi itu DIJAMIN selesai UTUH tanpa bisa disela di TENGAH-TENGAH oleh proses lain — seperti menekan tombol yang, sekali ditekan, TIDAK BISA "setengah tertekan". Ini kunci kenapa instruksi hardware seperti TSL bisa dipakai untuk membangun lock yang aman, sementara operasi biasa (baca lalu tulis terpisah) BISA disela di tengah, menciptakan race condition.

### Semaphore
**Semaphore** adalah variabel yang bisa diinisialisasi dengan nilai integer NON-NEGATIF, dengan dua operasi:
- **semWait** — MENGURANGI nilai semaphore.
- **semSignal** — MENAMBAH nilai semaphore.

Ada varian **strong** dan **weak** semaphore — sebuah QUEUE dipakai untuk menampung proses yang MENUNGGU pada semaphore tersebut (berbeda dari busy waiting — proses yang menunggu semaphore di-BLOCK, bukan terus mengecek dalam loop).

**Implementasi semaphore** harus mengimplementasikan `semWait` dan `semSignal` sebagai primitif ATOMIK — bisa diimplementasikan di HARDWARE/FIRMWARE, atau lewat skema software seperti algoritma Dekker/Peterson, atau lewat skema hardware-supported lain untuk mutual exclusion.

> [!info] Analogi
> Semaphore itu seperti sistem TIKET ANTREAN di sebuah kantor layanan dengan JUMLAH KURSI TERBATAS. Nilai semaphore = jumlah kursi kosong. Setiap orang yang mau masuk (semWait) mengurangi jumlah kursi kosong satu; kalau kursi habis (nilai 0), orang berikutnya harus MENUNGGU DI ANTREAN (di-block, tidak berdiri terus-menerus mengecek pintu setiap detik — beda dari busy waiting). Begitu ada orang keluar (semSignal), jumlah kursi bertambah satu, dan orang PALING DEPAN di antrean langsung dipanggil masuk.

### Producer-Consumer Problem
Masalah IPC klasik: satu atau lebih proses PRODUCER menghasilkan item dan menaruhnya di buffer bersama, satu atau lebih proses CONSUMER mengambil item dari buffer itu. Masalah muncul kalau buffer PENUH (producer harus menunggu) atau buffer KOSONG (consumer harus menunggu). Solusinya memakai KOMBINASI semaphore: satu semaphore untuk menghitung slot KOSONG, satu untuk slot TERISI, dan satu semaphore MUTEX untuk melindungi akses ke buffer itu sendiri (mutual exclusion).

### Readers-Writers Problem
Sebuah area data dibagi (shared) di antara BANYAK proses. Sebagian proses HANYA MEMBACA data (readers), sebagian HANYA MENULIS (writers). **Syarat yang harus dipenuhi:**
1. SEBERAPA PUN JUMLAH readers boleh MEMBACA file secara SIMULTAN.
2. HANYA SATU writer pada satu waktu boleh MENULIS ke file.
3. Kalau ada writer yang sedang MENULIS, TIDAK ADA reader yang boleh membaca.

Solusinya juga memakai kombinasi semaphore untuk mengatur akses bersama readers (banyak boleh sekaligus) dan eksklusif writers (harus sendirian).

> [!info] Analogi
> Readers-Writers Problem itu seperti aturan perpustakaan untuk buku referensi khusus. BANYAK ORANG boleh MEMBACA buku itu di tempat SECARA BERSAMAAN (readers). Tapi kalau petugas perpustakaan sedang MEREVISI/MENULIS ulang isi buku itu (writer), TIDAK ADA yang boleh membacanya sampai revisi selesai — dan hanya SATU petugas yang boleh merevisi pada satu waktu, supaya tidak ada dua revisi bentrok menimpa satu sama lain.

### Monitor
**Monitor** adalah CONSTRUCT bahasa pemrograman yang menyediakan fungsionalitas setara semaphore, tapi LEBIH MUDAH DIKONTROL. Diimplementasikan di berbagai bahasa pemrograman: Concurrent Pascal, Pascal-Plus, Modula-2, Modula-3, Java. Juga bisa diimplementasikan sebagai library program. Berupa modul software yang berisi satu atau lebih prosedur, urutan inisialisasi, dan data lokal.

Monitor mendukung sinkronisasi lewat **condition variable** yang ada DI DALAM monitor dan hanya bisa diakses DI DALAM monitor. Dua fungsi operasi condition variable:
- **cwait(c)** — menangguhkan eksekusi proses pemanggil pada kondisi c.
- **csignal(c)** — melanjutkan eksekusi salah satu proses yang di-block setelah cwait pada kondisi yang sama.

### Message Passing
Ketika proses saling berinteraksi, ada dua kebutuhan fundamental yang harus dipenuhi (sinkronisasi dan komunikasi data). **Message passing** adalah satu pendekatan yang menyediakan KEDUA fungsi ini — bekerja di distributed system, shared memory multiprocessor, MAUPUN uniprocessor system.

Fungsi aktualnya biasanya disediakan lewat sepasang primitif:
```
send(destination, message)
receive(source, message)
```
Proses MENGIRIM informasi dalam bentuk pesan ke proses lain yang ditandai sebuah destination. Proses MENERIMA informasi dengan mengeksekusi primitif receive, menandakan source dan message.

### Shared Memory System Call
Slide menjelaskan urutan system call untuk shared memory: membuat objek shared memory, mengonfigurasi UKURAN objek tersebut, lalu memakai fungsi `mmap()` untuk membuat memory-mapped file yang berisi objek shared-memory tersebut — `mmap()` mengembalikan pointer ke memory-mapped file yang dipakai untuk mengakses objek shared-memory.

### Signal
**Signal** dikirim ke sebuah proses — secara default, sebagian besar signal menyebabkan proses TERMINASI. Signal terbagi ke beberapa grup: **Hardware, Software, Input/Output, Process Control, Resource Control**. Untuk memakai signal di UNIX, program harus meng-include `<signal.h>`.

**kill()** — utility untuk mengirim signal ke sebuah proses:
```c
int kill(pid_t pid, int sig);
```

**Signal umum:**
| Signal | Nomor | Deskripsi |
| --- | --- | --- |
| SIGHUP | 1 | Terdeteksi hang up di terminal pengontrol, atau kematian proses pengontrol |
| SIGINT | 2 | Dikirim saat user mengirim sinyal interrupt (Ctrl+C) |
| SIGQUIT | 3 | Dikirim saat user mengirim sinyal quit (Ctrl+D) |
| SIGFPE | 8 | Dikirim saat ada operasi matematika ILEGAL |
| SIGKILL | 9 | Proses harus LANGSUNG berhenti, TANPA operasi cleanup apa pun |
| SIGALRM | 14 | Sinyal alarm clock (dipakai untuk timer) |
| SIGTERM | 15 | Sinyal terminasi software (dikirim oleh `kill` secara default) |

**Signal handler** — fungsi biasa dengan bentuk `void name(int);` — tidak punya return type, punya satu argumen (nomor signal). Nama handler bisa apa saja.

```cpp
#include <iostream>
#include <signal.h>
#include <sys/types.h>
using namespace std;

void siginthandle(int signalx) {
    cout << "Recieved signal " << signalx << endl;
    signal(SIGINT, siginthandle);
}

int main() {
    string mystring;
    signal(SIGINT, siginthandle);
    for (;;) {
        // do something
    }
    return 0;
}
```

> [!warning] Slide 19 dan 27 ("Mutual exclusion using semaphore" dan sebuah slide di section Mutual Exclusion) gagal diekstrak — file gambarnya rusak/tidak lengkap di PPT sumbernya (error: `required <p:blipFill> child element not present`). Buka file PPT asli untuk melihat isi visual kedua slide ini.

## Diagram & Visual
- **Slide 9 — Ilustrasi Race Condition**
  ![[99-Assets/OS/W06-slide09.jpg]]
- **Slide 16 — Critical Sections / Mutual Exclusion**
  ![[99-Assets/OS/W06-slide16.jpg]]
- **Slide 20 — Strict Alternation**
  ![[99-Assets/OS/W06-slide20.png]]
- **Slide 21 — Peterson's Solution**
  ![[99-Assets/OS/W06-slide21.png]]
- **Slide 22 — TSL Instruction**
  ![[99-Assets/OS/W06-slide22.png]]
- **Slide 23 — Common Concurrency Mechanism**
  ![[99-Assets/OS/W06-slide23.png]]
- **Slide 29-30 — Solusi Producer-Consumer Problem memakai Semaphore**
  ![[99-Assets/OS/W06-slide29.png]]
  ![[99-Assets/OS/W06-slide30.png]]
- **Slide 32-33 — Solusi Readers-Writers Problem memakai Semaphore**
  ![[99-Assets/OS/W06-slide32.png]]
  ![[99-Assets/OS/W06-slide33.png]]
- **Slide 40-42 — Shared Memory System Call dan Contohnya**
  ![[99-Assets/OS/W06-slide40.png]]
  ![[99-Assets/OS/W06-slide40a.png]]
  ![[99-Assets/OS/W06-slide40b.png]]
  ![[99-Assets/OS/W06-slide41.png]]
  ![[99-Assets/OS/W06-slide42.png]]

## Rumus / Sintaks
```c
// Message Passing
send(destination, message);
receive(source, message);

// Semaphore
semWait(S);    // S = S - 1, block kalau S < 0
semSignal(S);  // S = S + 1, bangunkan proses yang menunggu

// Signal (UNIX)
#include <signal.h>
int kill(pid_t pid, int sig);
void handler(int signum) { /* ... */ }
signal(SIGINT, handler);
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Busy waiting** | Proses terus mengecek kondisi dalam loop sambil menunggu, tetap memakai CPU |
| **Atomik (operasi)** | Operasi yang dijamin selesai utuh tanpa bisa disela di tengah |
| **Deadlock** | Kondisi dua/lebih proses saling menunggu selamanya, tidak ada yang bisa lanjut (dibahas detail di [[W07 - Deadlock]]) |
| **Mutex** | Semaphore biner (0/1) yang khusus dipakai untuk mutual exclusion |
| **mmap()** | System call untuk memetakan file/objek ke ruang alamat memori proses |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa "Disabling Interrupts" TIDAK BEKERJA di arsitektur multiprocessor, padahal bekerja di uniprocessor. Hubungkan dengan konsep multiprocessor dari [[W04 - Multiple Processor Systems]] — kenapa mendisable interrupt di SATU CPU tidak cukup untuk menjamin mutual exclusion kalau ada CPU LAIN yang bisa mengakses resource yang sama?
2. **(C4 – Analisis)** Bandingkan busy waiting (Peterson's Solution) dengan semaphore (yang memakai queue). Analisis: dari sisi EFISIENSI CPU, kenapa semaphore dianggap lebih baik untuk proses yang harus MENUNGGU LAMA, sementara busy waiting mungkin lebih baik untuk waktu tunggu yang SANGAT SINGKAT?
3. **(C5 – Evaluasi)** Sebuah tim programmer mengimplementasikan Producer-Consumer Problem tapi LUPA menambahkan semaphore mutex untuk melindungi akses ke buffer (hanya pakai semaphore "empty" dan "full"). Evaluasi: race condition APA yang bisa terjadi kalau dua producer menulis ke buffer secara BERSAMAAN tanpa mutex, meski semaphore "empty"/"full" sudah benar?
4. **(C5 – Evaluasi)** Bandingkan Readers-Writers Problem dengan Producer-Consumer Problem dari sisi JENIS kebutuhan sinkronisasinya. Evaluasi: kenapa Readers-Writers butuh aturan "banyak boleh sekaligus (readers) TAPI satu-satu untuk writer", sementara Producer-Consumer tidak punya perbedaan perlakuan seperti itu antara producer dan consumer?
5. **(C6 – Cipta)** Rancang pseudocode (memakai semaphore, format `semWait`/`semSignal`) untuk menyelesaikan masalah baru: "Barbershop Problem" (fiktif, untuk latihan) — satu tukang cukur, satu kursi cukur, dan ruang tunggu dengan N kursi. Kalau ruang tunggu penuh, pelanggan baru pulang. Kalau tidak ada pelanggan, tukang cukur tidur sampai ada pelanggan datang. Tentukan semaphore apa saja yang kamu butuhkan dan bagaimana pelanggan/tukang cukur berinteraksi lewat semaphore itu.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W05 - Process Scheduling]]
- [[W07 - Deadlock]]
- [[OS - Review dan Glosari]]
