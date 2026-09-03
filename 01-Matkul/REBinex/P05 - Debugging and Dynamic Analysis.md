---
matkul: Reverse Engineering dan Binary Exploitation
minggu: 5
sks: 3
sumber: REBinex - P5.pptx
tags: [kuliah/rebinex, minggu/p05]
status: draft
diproses: 2026-09-04
---

# P05 — Debugging and Dynamic Analysis

> [!warning] Learning outcome di slide 2 nggak nyambung sama isi (masih tentang GDB). Lihat catatan di [[P02 - Assembly and Disassembly]].

## Ringkasan
> - **Dynamic analysis** (analisis dengan MENJALANKAN program) ngasih insight yang nggak bisa didapat dari **static analysis** (baca kode/biner doang). Contoh: adanya string "action" di biner nggak berarti aksinya BENERAN dieksekusi — kamu harus jalankan programnya buat tau.
> - **GDB (GNU Debugger)** = debugger buat C/C++ dkk. Ngebolehin kamu ngintip apa yang lagi dilakukan program di titik tertentu selama eksekusi.
> - Compile dengan flag **`-g`** biar GDB bisa baca info debugging-nya.
> - Perintah inti GDB: **`run`** (jalankan program), **`break`** (pasang breakpoint di baris/fungsi tertentu), **`continue`** (lanjut ke breakpoint berikutnya), **`step`** (masuk ke dalam sub-rutin), **`next`** (lompati sub-rutin, treat sebagai 1 instruksi), **`print`** (lihat nilai variabel), **`watch`** (pause tiap kali sebuah variabel berubah nilai).
> - **Breakpoint** berhenti di LOKASI tertentu (baris/fungsi). **Watchpoint** berhenti kapan pun sebuah VARIABEL berubah, di mana pun itu terjadi.
> - `backtrace`/`where` nunjukin jejak (stack trace) fungsi-fungsi yang mengarah ke sebuah crash — mirip stack trace exception di Java.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Static analysis | Analisis program TANPA menjalankannya |
| Dynamic analysis | Analisis program DENGAN menjalankannya, mengamati perilaku nyatanya |
| GDB | GNU Debugger, debugger command-line untuk C/C++ dan bahasa lain |
| Breakpoint | Titik henti di lokasi (baris/fungsi) tertentu |
| Watchpoint | Titik henti yang aktif tiap kali nilai sebuah variabel berubah |
| `step` / `next` | Eksekusi satu baris; `step` masuk ke sub-rutin, `next` melompatinya |
| Segmentation fault | Error saat program mengakses memori yang tidak sah |

## Isi

### The Basics: kenapa perlu dynamic analysis
Berbeda dari static analysis, **dynamic analysis membiarkanmu mengamati fungsionalitas SEBENARNYA dari program.** Contohnya, keberadaan sebuah string aksi di dalam biner nggak berarti aksi itu bakal beneran dieksekusi. Dalam konteks analisis malware, kalau malware-nya adalah keylogger, **dynamic analysis bisa membiarkanmu melacak lokasi file log si keylogger di sistem, mengungkap jenis rekaman yang disimpannya, memecahkan ke mana dia mengirim informasinya, dan seterusnya.** Insight semacam ini akan lebih sulit didapat kalau cuma pakai teknik static analysis dasar.

### Introducing GDB
**GDB (GNU Debugger)** adalah debugger untuk beberapa bahasa, termasuk C dan C++. GDB memungkinkanmu memeriksa apa yang sedang dilakukan program di titik tertentu selama eksekusinya. **Error seperti segmentation fault mungkin lebih gampang ditemukan dengan bantuan GDB.**

Waktu melakukan reverse engineering sebuah program, tool ini juga bisa dipakai untuk meninjau kode Assembly hasil kompilasi dalam varian **AT&T** atau **Intel**, untuk melihat langkah-per-langkah apa yang sedang terjadi. **Breakpoint** ditambahkan untuk menghentikan program di tengah jalan dan meninjau data di register memori untuk mengidentifikasi bagaimana data itu sedang dimanipulasi.

### Getting Started dengan GDB
Normalnya, kamu meng-compile program seperti ini:
```bash
gcc [flags] <source files> -o <output file>
# contoh:
gcc -Wall -Werror -ansi -pedantic-errors prog1.c -o prog1.x
```
Sekarang tambahkan opsi **`-g`** untuk mengaktifkan dukungan debugging bawaan (yang dibutuhkan gdb):
```bash
gcc -g <source files> -o <output file>
# contoh:
gcc -Wall -Werror -ansi -pedantic-errors -g prog1.c -o prog1.x
```

### Running the Program
Untuk menjalankan programnya, buka gdb dari command line, terus:
```
(gdb) run
```
Ini menjalankan programnya. Kalau nggak ada masalah serius (misalnya programnya nggak segfault), programnya seharusnya jalan baik-baik saja di sini juga. Kalau programnya BERMASALAH, kamu (seharusnya) dapet informasi berguna seperti **nomor baris di mana dia crash**, dan parameter untuk fungsi yang menyebabkan error itu.

### Help
Kalau kamu pernah bingung soal sebuah command atau cuma pengen informasi lebih, pakai command **"help"**, dengan atau tanpa argumen: `(gdb) help [command]`. Kamu seharusnya dapet deskripsi yang bagus dan mungkin beberapa info berguna lainnya.

### Setting Breakpoints
**Breakpoint** bisa dipakai untuk menghentikan jalannya program di tengah, di titik yang ditentukan. Cara paling simpel adalah command **"break"**:
```
(gdb) break file1.c:6
```
Ini masang breakpoint di baris 6, di `file1.c`. Sekarang, kalau programnya pernah mencapai lokasi itu waktu berjalan, program akan berhenti dan meminta command lain darimu.

**Tapi bagaimana kalau kita nggak punya source code-nya?**

> [!info] Tip dari slide
> Kamu bisa memasang breakpoint sebanyak yang kamu mau, dan program seharusnya berhenti eksekusi kalau mencapai salah satu dari mereka.

Kamu juga bisa memberi tahu gdb untuk berhenti di sebuah FUNGSI tertentu. Misalnya kamu punya fungsi bernama `my_func`: `int my_func(int a, char *b);` — kamu bisa berhenti setiap kali fungsi ini dipanggil: `(gdb) break my_func`.

Setelah memasang breakpoint, kamu bisa coba pakai command `run` lagi. Kali ini, seharusnya berhenti di mana kamu suruh (kecuali ada error fatal sebelum mencapai titik itu). Kamu bisa lanjut ke breakpoint berikutnya dengan mengetik **"continue"** (mengetik `run` lagi akan me-restart program dari awal, yang nggak terlalu berguna): `(gdb) continue`.

Kamu bisa **single-step** (mengeksekusi cuma baris kode berikutnya) dengan mengetik `step`. Ini ngasih kamu kontrol yang sangat detail atas gimana program berjalan: `(gdb) step`. Mirip dengan `step`, command **"next"** juga single-step, kecuali yang ini nggak mengeksekusi tiap baris di sebuah sub-rutin, dia cuma memperlakukannya sebagai SATU instruksi: `(gdb) next`.

> [!info] Tip dari slide
> Mengetik `step` atau `next` berkali-kali bisa melelahkan. Kalau kamu cuma menekan ENTER, gdb akan mengulangi command yang barusan kamu berikan. Kamu bisa melakukan ini berkali-kali.

> [!info] Analogi
> **`step` itu kayak nonton film dengan main-menu dibuka tiap kali ada adegan flashback** — kamu masuk dan nonton flashback-nya dari awal sampai akhir sebelum lanjut ke cerita utama. **`next` itu nonton film yang sama tapi flashback-nya di-skip langsung ke ujungnya** — kamu tau flashback-nya terjadi, tapi nggak diajak masuk ke detailnya. Kalau kamu pengen tau APA yang terjadi DI DALAM sebuah fungsi, pakai `step`. Kalau kamu cuma pengen tau HASIL fungsi itu tanpa peduli detail internalnya, pakai `next`.

### Print Variable Values
GDB juga bisa dipakai untuk melihat isi program, seperti nilai variabel, dsb. Command **`print`** mencetak nilai variabel yang ditentukan, dan **`print/x`** mencetak nilainya dalam heksadesimal:
```
(gdb) print my_var
(gdb) print/x my_var
```

### Setting Watchpoints
Kalau breakpoint menginterupsi program di sebuah baris atau fungsi tertentu, **watchpoint bekerja pada VARIABEL**. Mereka mem-pause program setiap kali nilai variabel yang diawasi dimodifikasi. Contoh command watch:
```
(gdb) watch my_var
```
Sekarang, setiap kali nilai `my_var` dimodifikasi, program akan berhenti dan mencetak nilai lama dan baru.

> [!info] Tip dari slide
> Kamu mungkin bertanya-tanya gimana gdb menentukan variabel `my_var` mana yang harus diawasi kalau ada lebih dari satu yang dideklarasikan di programmu. Jawabannya (mungkin agak menyebalkan) adalah dia mengandalkan **scope** variabel itu, relatif terhadap posisimu di program saat itu. Ini artinya kamu harus ingat nuansa rumit soal scope dan extent. :(

### Other Useful Commands
- **`backtrace`** — menghasilkan stack trace dari pemanggilan fungsi yang mengarah ke segfault (harusnya mengingatkanmu ke exception Java)
- **`where`** — sama seperti backtrace; kamu bisa menganggap versi ini bekerja bahkan saat kamu masih di tengah-tengah program
- **`finish`** — menjalankan sampai fungsi saat ini selesai
- **`delete`** — menghapus breakpoint yang ditentukan
- **`info breakpoints`** — menampilkan informasi tentang semua breakpoint yang dideklarasikan

## Diagram & Visual
> [!warning] **Slide 15 ("A / UPX Packing") kosong total** — bukan cuma gagal diekstrak, tapi memang **nggak ada gambar maupun teks lain di slide itu selain judulnya** (sudah dicek langsung dari file PPTX aslinya). Sepertinya ini section divider untuk demo praktik UPX packing yang isinya nggak pernah ditambahkan ke slide (kemungkinan didemokan langsung di kelas tanpa slide pendukung, atau lupa diisi). Nggak ada yang bisa "dibuka manual" di sini karena kontennya memang tidak pernah ada di file-nya.

## Rumus / Sintaks
```bash
# compile dengan info debugging
gcc -g prog1.c -o prog1.x

# buka GDB
gdb prog1.x
```
```
(gdb) run                    # jalankan program
(gdb) help [command]         # bantuan
(gdb) break file1.c:6        # breakpoint di baris 6 file1.c
(gdb) break my_func          # breakpoint di fungsi my_func
(gdb) continue               # lanjut ke breakpoint berikutnya
(gdb) step                   # eksekusi 1 baris, MASUK ke sub-rutin
(gdb) next                   # eksekusi 1 baris, LOMPATI sub-rutin
(gdb) print my_var           # lihat nilai variabel
(gdb) print/x my_var         # lihat nilai variabel dalam hex
(gdb) watch my_var           # pause tiap kali my_var berubah
(gdb) backtrace              # stack trace penyebab crash
(gdb) where                  # sama seperti backtrace
(gdb) finish                 # jalankan sampai fungsi saat ini selesai
(gdb) delete <n>             # hapus breakpoint nomor n
(gdb) info breakpoints       # daftar semua breakpoint
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Segmentation fault** | Error yang terjadi ketika program mencoba mengakses memori yang tidak diizinkan |
| **AT&T / Intel syntax** | Dua gaya penulisan assembly x86 yang berbeda; GDB bisa menampilkan salah satunya |
| **Scope (variabel)** | Jangkauan/wilayah kode di mana sebuah variabel bisa diakses dan dikenali |
| **UPX** | Ultimate Packer for eXecutables, tool packer populer yang bisa mengompresi/mengenkripsi biner |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan breakpoint dan watchpoint dari sisi "apa yang mereka pantau". Analisis: kalau kamu curiga sebuah variabel `secret_flag` diubah nilainya di suatu tempat yang TIDAK kamu ketahui lokasinya di kode, mana yang lebih efektif dipakai duluan — breakpoint atau watchpoint? Jelaskan.
2. **(C4 – Analisis)** Kasus keylogger di slide 3 dipakai sebagai contoh kenapa dynamic analysis penting. Analisis: sebutkan MINIMAL DUA informasi spesifik tentang keylogger itu yang HANYA bisa didapat lewat dynamic analysis (menjalankan programnya), dan jelaskan kenapa static analysis (baca kode/string doang) tidak cukup untuk masing-masing.
3. **(C5 – Evaluasi)** Slide 9 menanyakan: "tapi bagaimana kalau kita tidak punya source code-nya?" tapi jawabannya tidak dijelaskan secara eksplisit di slide manapun di deck ini. Evaluasi: berdasarkan pemahamanmu tentang `break file1.c:6` (yang butuh nama file dan nomor baris) vs `break my_func` (yang cuma butuh nama fungsi), teknik mana yang MASIH BISA dipakai kalau kamu cuma punya BINARY tanpa source code, dan kenapa?
4. **(C5 – Evaluasi)** Dalam konteks reverse engineering malware, evaluasi risiko melakukan dynamic analysis (menjalankan malware beneran) dibanding static analysis (baca binernya doang tanpa dijalankan). Apa trade-off keamanan yang harus dipertimbangkan seorang analis sebelum memilih pendekatan dynamic?
5. **(C6 – Cipta)** Rancang sesi debugging GDB (urutan command, bukan kode program) untuk menyelidiki sebuah program yang crash secara acak saat memproses input tertentu. Sebutkan MINIMAL 5 command GDB yang akan kamu pakai secara berurutan, dan jelaskan tujuan tiap langkahnya (misalnya: langkah 1 untuk menemukan LOKASI crash, langkah berikutnya untuk memeriksa NILAI variabel yang relevan, dst).

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_REBinex]]
- [[P04 - Obfuscation Techniques]]
- [[P06 - Binary Patching]]
- [[REBinex - Review dan Glosari]]
