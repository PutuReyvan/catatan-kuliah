---
matkul: Operating Systems
minggu: 8
sks: 3
sumber: File Systems.pptx
tags: [kuliah/os, minggu/w08]
status: draft
diproses: 2026-09-04
---

# W08 — File Systems

## Ringkasan
> - **File** = kumpulan data yang dibuat user. **File system** menyediakan cara menyimpan data sebagai file PLUS kumpulan fungsi untuk memanipulasinya (create, delete, open, close, read, write).
> - **Directory** ada dua model: **single-level** (semua file di satu level, sederhana tapi terbatas) dan **hierarchical** (bertingkat, seperti folder di dalam folder — model yang dipakai OS modern).
> - **File organization** ada 5 jenis: **Pile** (paling sederhana, cuma numpuk data), **Sequential** (paling umum, cocok tape/disk), **Indexed Sequential** (tambah index untuk random access), **Indexed** (akses HANYA lewat index), **Direct/Hashed** (akses langsung lewat hashing key).
> - **File allocation** di disk ada 3 cara: **Contiguous** (blok berurutan, cepat tapi rawan fragmentasi), **Chained/Linked List** (blok tersambung pointer, fleksibel tapi lambat untuk random access), **Indexed** (tabel index terpisah, gabungan kelebihan keduanya).
> - **inode** (UNIX/Linux) = struktur kontrol yang menyimpan SEMUA informasi kunci sebuah file — satu inode per file, tapi BISA punya banyak nama (hard link).

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| File system | Sistem yang menyediakan penyimpanan data sebagai file plus fungsi manipulasinya |
| Directory | Struktur yang mengorganisasi file, bisa single-level atau hierarchical |
| File organization | Struktur logis record dalam file, ditentukan cara aksesnya |
| File allocation | Cara blok-blok disk dialokasikan untuk menyimpan sebuah file |
| inode | Struktur kontrol UNIX yang menyimpan informasi kunci tentang sebuah file |
| Free space management | Cara OS melacak blok disk mana yang masih kosong |

## Isi

### File dan File System
**File** adalah kumpulan data yang dibuat oleh user. **File System** adalah salah satu bagian TERPENTING dari OS bagi seorang user — ini bagian OS yang paling langsung dirasakan dan berinteraksi dengan user sehari-hari.

**File System** menyediakan cara menyimpan data terorganisasi sebagai file, PLUS kumpulan fungsi yang bisa dilakukan terhadap file itu. Menjaga sekumpulan ATRIBUT yang terasosiasi dengan file. **Operasi tipikal:** Create, Delete, Open, Close, Read, Write.

### Directory: Single-Level vs Hierarchical
- **Single-level directory** — SEMUA file berada dalam SATU level directory yang sama (tidak ada subfolder). Sederhana tapi terbatas — masalah muncul kalau banyak user/file dengan nama yang mungkin bentrok.
- **Hierarchical directory** — directory bisa berisi SUBDIRECTORY, membentuk struktur bertingkat seperti pohon. Ini model yang dipakai OS modern (folder di dalam folder).

### Tujuan File Management System
Tujuan sistem manajemen file mencakup: memenuhi kebutuhan manajemen data user, MENJAMIN data dalam file VALID, mengoptimalkan performa, menyediakan dukungan I/O untuk BERBAGAI jenis storage device, MEMINIMALKAN potensi data hilang/rusak, menyediakan set rutin interface I/O yang TERSTANDARISASI ke proses user, dan menyediakan dukungan I/O untuk BANYAK user (di sistem multi-user).

### Arsitektur Software File System
File system software tersusun berlapis, dari yang paling DEKAT hardware sampai paling DEKAT user:

1. **Basic I/O Supervisor** — bertanggung jawab atas SEMUA inisiasi dan terminasi I/O file. Di level ini, struktur kontrol dijaga untuk menangani device I/O, scheduling, dan status file. Memilih DEVICE mana yang dipakai untuk I/O. Berkaitan dengan penjadwalan akses disk/tape untuk optimasi performa. Buffer I/O dialokasikan dan secondary memory dialokasikan di level ini. **Bagian dari operating system.**

2. **Logical I/O** — level yang menangani abstraksi logis operasi file.

3. **Access Method** — level file system yang PALING DEKAT dengan user. Menyediakan interface STANDAR antara aplikasi dan file system/device penyimpan data. Access method BERBEDA merefleksikan struktur file dan cara akses/pemrosesan data yang BERBEDA.

### File Organization
**File organization** adalah struktur LOGIS record, ditentukan oleh CARA record itu diakses. Dalam memilih file organization, beberapa kriteria penting: **waktu akses SINGKAT, kemudahan UPDATE, EKONOMI penyimpanan, kemudahan PEMELIHARAAN, dan RELIABILITAS.** Prioritas kriteria ini tergantung APLIKASI yang akan memakai file tersebut.

**Lima jenis file organization:**

1. **The Pile** — bentuk PALING TIDAK KOMPLEKS. Data dikumpulkan dalam urutan KEDATANGANNYA. Setiap record terdiri dari satu "burst" data. Tujuannya SEKADAR mengakumulasi data dan menyimpannya. Akses record lewat PENCARIAN MENYELURUH (exhaustive search) — lambat.

2. **The Sequential File** — bentuk PALING UMUM. Format TETAP dipakai untuk record. Ada KEY FIELD yang secara unik mengidentifikasi record. Biasanya dipakai di aplikasi BATCH. Satu-satunya organisasi yang mudah disimpan di TAPE maupun DISK.

3. **Indexed Sequential File** — menambahkan INDEX ke file untuk mendukung random access. Menambahkan OVERFLOW FILE. Sangat mengurangi waktu akses satu record. Bisa memakai BANYAK LEVEL indexing untuk efisiensi lebih tinggi.

4. **Indexed File** — record diakses HANYA lewat indexnya. Bisa memakai record PANJANG VARIABEL. **Exhaustive index** — satu entri untuk SETIAP record di file utama. **Partial index** — entri hanya untuk record yang field-nya ADA. Dipakai terutama di aplikasi yang KETEPATAN WAKTU informasinya KRITIS — contoh: sistem reservasi maskapai dan sistem kontrol inventori.

5. **Direct/Hashed File** — mengakses LANGSUNG blok mana pun dengan alamat yang DIKETAHUI. Memanfaatkan HASHING pada nilai key. Sering dipakai saat: akses SANGAT CEPAT dibutuhkan, record PANJANG TETAP dipakai, record selalu diakses SATU PER SATU.

> [!info] Analogi
> Bayangkan lima jenis file organization ini seperti lima cara menyimpan resep masakan. **Pile** itu seperti melempar SEMUA resep ke satu kardus tanpa urutan — cari satu resep berarti bongkar semua kardus (exhaustive search). **Sequential** itu seperti menyusun resep berurutan berdasarkan abjad di rak buku — mudah dibaca dari awal ke akhir, tapi cari resep tertentu tetap harus geser-geser dari awal. **Indexed Sequential** itu seperti rak resep tadi TAPI ditambah kartu indeks di depan yang menunjuk langsung ke halaman tertentu — jauh lebih cepat. **Indexed** itu seperti katalog perpustakaan yang HANYA bisa dicari lewat kartu katalog, tidak bisa langsung buka rak. **Direct/Hashed** itu seperti sistem loker bernomor — kamu masukkan kode (hash key), langsung tahu persis loker mana yang harus dibuka, tanpa perlu cari-cari sama sekali.

### File Allocation (Alokasi Blok Disk)
Tiga cara utama mengalokasikan ruang disk untuk sebuah file:

1. **Contiguous Allocation** — file disimpan sebagai BLOK-BLOK YANG BERURUTAN di disk. Cepat untuk akses sekuensial, tapi rawan FRAGMENTASI — kalau file dihapus (misalnya file D dan F dihapus), muncul "lubang" kosong yang mungkin tidak cukup besar untuk file baru.

2. **Chained/Linked List Allocation** — file disimpan sebagai LINKED LIST dari blok-blok disk, setiap blok menunjuk ke blok berikutnya lewat pointer. Fleksibel (tidak perlu ruang berurutan), tapi LAMBAT untuk random access (harus telusuri dari awal). Variasi: **Linked List Allocation memakai FAT (File Allocation Table)** — tabel linked list-nya disimpan di MAIN MEMORY, bukan tersebar di tiap blok disk, mempercepat traversal.

3. **Indexed Allocation** — memakai TABEL INDEX terpisah yang mencatat SEMUA blok milik sebuah file, menggabungkan kelebihan contiguous (akses cepat lewat index) dan linked list (tidak perlu ruang berurutan).

### Sharing Files dan Hard Link
Slide menunjukkan bagaimana sebuah file bisa DIBAGI (shared) antar user lewat LINK — situasi SEBELUM linking, SETELAH link dibuat, dan SETELAH pemilik asli MENGHAPUS file (file tetap ada selama masih ada link lain yang menunjuk ke sana).

### Flash-Based File System
Slide menunjukkan komponen di dalam **flash SSD** tipikal — berbeda secara fundamental dari disk magnetik, mempengaruhi cara file system dirancang untuk media ini (misalnya soal wear leveling, yang tidak dibahas detail di slide).

### Free Space Management
Dua metode utama melacak blok disk mana yang KOSONG:

1. **Bit Table (Bitmap)** — memakai VEKTOR berisi SATU BIT untuk setiap blok di disk. Entri **0** = blok KOSONG, entri **1** = blok TERPAKAI.
2. **Linked List** — bagian-bagian kosong DIRANTAI bersama memakai pointer dan nilai panjang di tiap bagian kosong. Overhead ruang NEGLIGIBLE karena tidak perlu tabel alokasi disk terpisah. Cocok untuk SEMUA metode file allocation.
3. **Indexing** — memperlakukan ruang kosong SEBAGAI FILE dan memakai tabel index seperti untuk file allocation biasa. Untuk efisiensi, index sebaiknya berbasis PORSI UKURAN VARIABEL, bukan per blok. Memberikan dukungan efisien untuk SEMUA metode file allocation.

### Disk Quota
Kuota dilacak berbasis PER-USER dalam sebuah tabel kuota — membatasi berapa banyak ruang disk yang boleh dipakai tiap user.

### File Consistency
Empat state konsistensi file system: **(a) Consistent** (normal, semua sesuai), **(b) Missing block** (blok hilang, tidak tercatat di mana pun), **(c) Duplicate block in free list** (blok yang sama tercatat DUA KALI di daftar kosong), **(d) Duplicate data block** (blok data yang sama dipakai oleh DUA file sekaligus — masalah serius).

### UNIX File Management: inode
Sistem file UNIX membedakan ENAM tipe file. Semua tipe file UNIX diadministrasi OS lewat **inode**:
- **inode (index node)** adalah struktur kontrol yang berisi informasi KUNCI yang dibutuhkan OS untuk sebuah file tertentu.
- BEBERAPA nama file BISA diasosiasikan dengan SATU inode yang sama (ini yang disebut **hard link**).
- inode yang AKTIF diasosiasikan dengan TEPAT SATU file.
- Setiap file dikontrol oleh TEPAT SATU inode.

> [!info] Analogi
> inode itu seperti KARTU IDENTITAS RESMI sebuah file (menyimpan ukuran, pemilik, izin akses, lokasi data di disk, dll.), sementara NAMA FILE yang kamu lihat di folder itu cuma "PANGGILAN" atau LABEL yang menunjuk ke kartu identitas itu. Satu orang (inode) BISA punya beberapa nama panggilan berbeda (hard link) — hapus SATU nama panggilan tidak menghapus ORANGNYA, selama masih ada panggilan lain yang menunjuk ke kartu identitas yang sama.

### Windows File System: NTFS
**NTFS (New Technology File System)** dirancang pengembang Windows NT untuk memenuhi kebutuhan HIGH-END workstation dan server. **Fitur kunci NTFS:**
- **Recoverability** — bisa dipulihkan setelah crash.
- **Security** — kontrol akses yang kuat.
- **Large disks and large files** — mendukung disk dan file BERUKURAN BESAR.
- **Multiple data streams** — satu file bisa punya lebih dari satu "stream" data.
- **Journaling** — mencatat perubahan sebelum benar-benar diterapkan, untuk pemulihan cepat setelah crash.
- **Compression and encryption** — dukungan kompresi dan enkripsi built-in.
- **Hard and symbolic links** — mendukung kedua jenis link.

### File System Calls (UNIX/Linux, C/C++)
**`stat()`** — memberi kemampuan meng-query objek di file system. Butuh header `<sys/types.h>` dan `<sys/stat.h>`. Memungkinkan mengetahui: ukuran file, pemilik file, grup pemilik, string permission, waktu terakhir dimodifikasi.

```c
int stat(const char* path, struct stat* buf);

struct stat {
    dev_t     st_dev;      // ID device yang berisi file
    ino_t     st_ino;      // Nomor inode
    mode_t    st_mode;     // Tipe dan mode file
    nlink_t   st_nlink;    // Jumlah hard link
    uid_t     st_uid;      // User ID pemilik
    gid_t     st_gid;      // Group ID pemilik
    off_t     st_size;     // Total ukuran, dalam byte
    blksize_t st_blksize;  // Ukuran blok untuk I/O filesystem
    blkcnt_t  st_blocks;   // Jumlah blok 512B yang dialokasikan
    struct timespec st_atim;  // Waktu akses terakhir
    struct timespec st_mtim;  // Waktu modifikasi terakhir
    struct timespec st_ctim;  // Waktu perubahan status terakhir
};
```

**Contoh penggunaan `stat()`:**
```cpp
#include <iostream>
#include <sys/stat.h>
#include <sys/types.h>
using namespace std;

int main(int argc, char* argv[]) {
    struct stat details;
    int retval;
    if (argc != 2) {
        cout << "Insufficient arguments" << endl;
        return 0;
    }
    retval = stat(argv[1], &details);
    if (retval == 0) {
        cout << "Size of file is: " << details.st_size << endl;
        cout << "File is owned by uid: " << details.st_uid << endl;
    } else {
        cout << "Could not stat the file - does not exist" << endl;
    }
    return 0;
}
```

**System call lain yang relevan:**
```c
int remove(const char* path);
int unlink(const char* path);
int creat(const char *path, mode_t mode);
int mkdir(const char *path, mode_t mode);
int chdir(const char* path);
```

**Struktur directory entry (`dirent`):**
```c
struct dirent {
    ino_t d_ino;
    off_t d_off;
    unsigned short d_reclen;
    char d_name[1];
};
```
`d_name` berisi nama file dari `readdir()`. Setelah selesai memproses directory entry, tutup `DIR*` dengan `closedir()`.

## Diagram & Visual
- **Slide 7 — Contoh File Extension**
  ![[99-Assets/OS/W08-slide07.png]]
- **Slide 8 — Struktur File Executable dan Archive**
  ![[99-Assets/OS/W08-slide08.jpg]]
- **Slide 9 — Single-Level Directory**
  ![[99-Assets/OS/W08-slide09.jpg]]
- **Slide 10 — Hierarchical Directory**
  ![[99-Assets/OS/W08-slide10.jpg]]
- **Slide 13 — Arsitektur Software File System**
  ![[99-Assets/OS/W08-slide13.png]]
- **Slide 17 — Elemen Manajemen File**
  ![[99-Assets/OS/W08-slide17.png]]
- **Slide 20-24 — Diagram lima jenis File Organization (Pile, Sequential, Indexed Sequential, Indexed, Direct/Hashed)**
  ![[99-Assets/OS/W08-slide20.png]]
  ![[99-Assets/OS/W08-slide21.png]]
  ![[99-Assets/OS/W08-slide22.png]]
  ![[99-Assets/OS/W08-slide23.png]]
- **Slide 27-30 — Diagram File Allocation (Contiguous, Chained, FAT, Indexed)**
  ![[99-Assets/OS/W08-slide27.jpg]]
  ![[99-Assets/OS/W08-slide28.jpg]]
  ![[99-Assets/OS/W08-slide29.jpg]]
  ![[99-Assets/OS/W08-slide30.jpg]]
- **Slide 31-32 — Sharing Files dan Link**
  ![[99-Assets/OS/W08-slide31.jpg]]
  ![[99-Assets/OS/W08-slide32.jpg]]
- **Slide 33 — Komponen Flash SSD**
  ![[99-Assets/OS/W08-slide33.jpg]]
- **Slide 34 — Free Space Management (Linked List dan Bitmap)**
  ![[99-Assets/OS/W08-slide34.jpg]]
- **Slide 36 — Disk Quota**
  ![[99-Assets/OS/W08-slide36.jpg]]
- **Slide 37 — File System States (Consistency)**
  ![[99-Assets/OS/W08-slide37.jpg]]
- **Slide 40 — Diagram inode**
  ![[99-Assets/OS/W08-slide40.png]]

## Rumus / Sintaks
```c
// File system calls (UNIX/Linux)
int stat(const char* path, struct stat* buf);
int remove(const char* path);
int unlink(const char* path);
int creat(const char *path, mode_t mode);
int mkdir(const char *path, mode_t mode);
int chdir(const char* path);

// Directory reading
struct dirent { ino_t d_ino; off_t d_off; unsigned short d_reclen; char d_name[1]; };
DIR* opendir(const char* path);
struct dirent* readdir(DIR* dirp);
int closedir(DIR* dirp);
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Fragmentasi** | Ruang disk kosong yang terpecah-pecah jadi potongan kecil tidak berguna, umum di Contiguous Allocation |
| **FAT (File Allocation Table)** | Tabel di memori yang mencatat rantai blok linked-list sebuah file |
| **Journaling** | Teknik mencatat perubahan sebelum diterapkan, untuk pemulihan cepat setelah crash |
| **Hard link** | Nama file tambahan yang menunjuk ke inode yang SAMA dengan file asli |
| **Symbolic link** | Referensi/pointer ke PATH file lain, berbeda dari hard link yang menunjuk langsung ke inode |
| **Wear leveling** | Teknik meratakan penulisan di SSD supaya tidak ada sel yang aus lebih cepat dari yang lain |

## Pertanyaan Terbuka
- Slide menyebut "flash-based file system" (SSD) tapi tidak membahas detail teknik seperti wear leveling atau garbage collection yang relevan untuk SSD — perlu dicek dari sumber lain kalau dibutuhkan untuk ujian.

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa Contiguous Allocation RAWAN fragmentasi setelah file dihapus (contoh: file D dan F dihapus), sementara Linked List Allocation TIDAK punya masalah fragmentasi eksternal yang sama. Hubungkan dengan cara masing-masing metode menyimpan blok file di disk.
2. **(C4 – Analisis)** Bandingkan Indexed Sequential File dan Indexed File. Analisis: kenapa Indexed Sequential MASIH bisa diakses secara sequential (selain random), sementara Indexed File HANYA bisa diakses lewat indexnya?
3. **(C5 – Evaluasi)** Sebuah sistem reservasi maskapai (butuh akses SANGAT CEPAT ke record penerbangan tertentu) sedang memilih antara Sequential File dan Direct/Hashed File. Evaluasi: mana yang lebih tepat, dan jelaskan risiko kalau memilih Sequential File untuk kasus penggunaan seperti ini.
4. **(C5 – Evaluasi)** Bandingkan Bit Table (bitmap) dan Linked List untuk Free Space Management. Evaluasi: untuk DISK YANG SANGAT BESAR (misalnya terabyte), overhead penyimpanan MANA yang lebih signifikan — bitmap yang menyimpan 1 bit per blok, atau linked list yang menyimpan pointer per bagian kosong? Pertimbangkan jumlah blok kosong yang tersebar vs jumlah blok total.
5. **(C6 – Cipta)** Rancang program C++ sederhana (memakai `stat()`) yang menerima path file dari argumen command-line, lalu mencetak: ukuran file, jumlah hard link (`st_nlink`), dan waktu modifikasi terakhir (`st_mtim`). Sertakan penanganan error kalau file tidak ditemukan.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W07 - Deadlock]]
- [[W09 - IO Management]]
- [[OS - Review dan Glosari]]
