---
matkul: Reverse Engineering dan Binary Exploitation
minggu: 8
sks: 3
sumber: REBinex - P8.pptx
tags: [kuliah/rebinex, minggu/p08]
status: draft
diproses: 2026-09-04
---

# P08 — Deep Dive into Buffer Overflow

> [!warning] Learning outcome di slide 2 nggak nyambung sama isi (masih tentang buffer overflow). Lihat catatan di [[P02 - Assembly and Disassembly]].

## Ringkasan
> - Register kunci buat ngerti stack: **EBP** (nunjuk ke DASAR/base stack fungsi yang aktif), **ESP** (nunjuk ke PUNCAK/top stack saat ini), **EIP** (alamat instruksi berikutnya yang dieksekusi).
> - Stack diisi dari memori TINGGI ke memori RENDAH. **Semua variabel diakses RELATIF ke EBP** — parameter fungsi ada DI ATAS EBP, variabel lokal ada DI BAWAH EBP.
> - Urutan tumpukan stack (dari atas ke bawah): parameter fungsi → **Old EIP** (alamat kembali) → **Old EBP** (EBP fungsi pemanggil) → variabel lokal.
> - Sepuluh **fungsi C berbahaya** yang sering jadi sumber buffer overflow: `strcpy`, `strncpy`, `strcat`, `printf`, `sprintf` (rentan format string), `scanf`, `fgets`, `gets`, `getws`, `memcpy`, `memmove`.
> - Kalau kamu bisa overflow variabel lokal SAMPAI nimpa **Old EIP**, dan ganti nilainya jadi alamat memori tempat kode jahat (**shellcode**) berada — begitu fungsinya `return`, program bakal LOMPAT ke kode jahat itu, bukan balik ke pemanggil aslinya.
> - Pola klasik shellcode injection: buffer diisi **data random (padding) → shellcode → alamat return yang menunjuk balik ke shellcode itu sendiri**.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| EBP | Stack pointer yang menunjuk ke BASE/dasar stack frame fungsi aktif |
| ESP | Stack pointer yang menunjuk ke TOP/puncak stack saat ini |
| Old EIP | Alamat return; instruksi yang dituju setelah fungsi selesai (`return`) |
| Old EBP | Nilai EBP milik fungsi pemanggil, disimpan untuk dipulihkan setelah fungsi selesai |
| Dangerous C functions | Fungsi C yang tidak memeriksa batas ukuran buffer secara otomatis |
| Return address overwrite | Teknik menimpa Old EIP dengan alamat pilihan penyerang |

## Isi

### Division of Memory in a Running Process
Memori proses yang sedang berjalan terbagi jadi beberapa section. Yang paling penting buat dibahas:
- **Stack** — struktur data LIFO yang banyak dipakai komputer dalam manajemen memori, dll.
- **EBP** — stack pointer yang menunjuk ke BASE/dasar stack.
- **ESP** — stack pointer yang menunjuk ke TOP/puncak stack.
- **EIP** — berisi alamat instruksi berikutnya yang mau dieksekusi.

### Stack Layout
Stack diisi dari memori TINGGI ke memori RENDAH. **Di dalam stack, semua variabel diakses relatif terhadap EBP.** Dalam sebuah program, setiap fungsi punya stack-nya sendiri. Semuanya direferensikan dari register EBP.

**Ingat, di dalam stack, semua variabel diakses relatif terhadap EBP. Di ATAS EBP, parameter fungsi disimpan.**

Variabel `a`, `b`, dan `c`, sebagai parameter fungsi, disimpan **DI ATAS EBP**. Semua variabel LOKAL sebuah fungsi disimpan **DI BAWAH EBP**. **"Old %ebp"** adalah nilai EBP dari fungsi sebelumnya. Karena setelah sebuah fungsi dieksekusi, dia harus kembali ke fungsi yang lebih lama; oleh karena itu, kita perlu menyimpan nilai baik Old EBP maupun Old EIP. Register **ESP** menyimpan alamat dasar (bottom) stack.

Variabel `x`, `y`, `z`, sebagai variabel lokal fungsi, disimpan **DI BAWAH EBP**.

> [!info] Analogi
> Bayangin sebuah fungsi itu **satu meja kerja dalam sebuah kantor bertingkat**. EBP itu "posisi meja kerjamu SENDIRI di lantai ini" — semua barang di mejamu (variabel lokal, DI BAWAH) dan barang yang dikirim padamu dari klien (parameter, DI ATAS) diukur relatif ke posisi meja itu. **Old EBP** itu "catatan posisi meja atasanmu di lantai sebelumnya" yang kamu simpen di lacimu — begitu kerjaanmu selesai, kamu balik ke posisi meja itu. **Old EIP** itu "instruksi apa yang harus atasanmu lanjutkan" begitu kamu selesai lapor.

### Dangerous C Functions
Fungsi-fungsi C yang **berbahaya** karena rentan overflow:
- **`strcpy`** (tidak menentukan panjang maksimum saat menyalin)
- `strncpy`
- `strcat`
- `printf`
- `sprintf` (kerentanan **format string** — lihat [[P13 - Format String Exploitation]])
- `scanf`
- `fgets`
- `gets`
- `getws`
- `memcpy`
- `memmove`

### Deep Dive into Stack Buffer Overflow
Diberikan sebuah fungsi `foo` dengan parameter `a`, `b`, `c` dan variabel lokal `x`, `y`, `z`. Karena `a`, `b`, `c` adalah parameter yang diteruskan ke fungsi, mereka disimpan DI ATAS EBP. Juga karena stack diisi dari memori tinggi ke rendah dan **parameter dibaca dari kanan ke kiri**, maka `c` ditulis PERTAMA di memori, diikuti `b` lalu `a`. `x`, `y`, `z` sebagai variabel lokal disimpan DI BAWAH EBP. **Juga diperlukan menyimpan Old EIP dan Old EBP dari fungsi `main`** untuk tahu ke mana harus kembali setelah fungsi ini selesai dieksekusi.

**Apa yang terjadi kalau kita input 11 huruf 'A' ke variabel `c`?** — perbandingannya ditunjukkan langsung dengan **apa yang terjadi kalau kita input 150 huruf 'A' ke variabel `c`**. Perbedaannya keliatan jelas: dengan 11 karakter, overflow-nya masih terbatas dan mungkin cuma merusak variabel tetangga. Dengan 150 karakter, overflow-nya sudah cukup jauh buat menembus ke wilayah **Old EIP**.

**Bayangkan sebuah situasi di mana kita bisa mengoverflow variabel `x`, `y`, dan `z` sedemikian rupa sehingga Old EIP dimodifikasi dan menyimpan ALAMAT MEMORI di mana kode jahat ditempatkan. Apa yang akan terjadi?**

Ketika fungsinya kembali (return), CPU akan mengambil nilai yang tersimpan sebagai Old EIP dan melompat ke sana — dan karena Old EIP sudah kita timpa dengan alamat kode jahat kita, **kode jahat itu yang akan dieksekusi**, bukan kode yang seharusnya dijalankan si program.

### Pola Injeksi Shellcode
Bayangkan sebuah buffer dengan panjang 500, didefinisikan di dalam sebuah fungsi. Sekarang buffer itu di-overflow sedemikian rupa sehingga isinya adalah: **data random**, diikuti **shellcode (kode jahat)**, dan kemudian **return address** yang menunjuk ke shellcode itu. Jadi, setelah fungsinya dieksekusi (dan kembali/return), instruksi yang ditunjuk oleh return address itu yang dieksekusi — **dan begitulah cara shellcode kita dieksekusi.**

> [!info] Konteks tambahan (bukan dari slide)
> Ini persis alasan kenapa **NOP sled** yang dibahas di [[P02 - Assembly and Disassembly]] berguna banget di sini: kamu jarang bisa nebak alamat PERSIS di mana shellcode-mu bakal berada di memori (apalagi kalau ASLR aktif — lihat [[P07 - Stack Security and Exploitation]]). Jadi trik praktisnya: taruh deretan panjang instruksi `nop` SEBELUM shellcode-nya, dan arahkan return address ke MANA SAJA di dalam deretan NOP itu. Selama kamu "mendarat" di suatu titik dalam deretan NOP-nya, eksekusi bakal "meluncur" (sled) sampai ke shellcode aslinya di ujung, walau tebakan alamatmu nggak persis.

## Diagram & Visual
- **Slide 3 — pembagian memori proses yang sedang berjalan**
  ![[99-Assets/REBinex/P08-slide03.png]]
- **Slide 5, 6 — EBP dan ESP; register EIP**
  ![[99-Assets/REBinex/P08-slide05.png]]
  ![[99-Assets/REBinex/P08-slide06.png]]
- **Slide 7, 8, 9, 10 — layout stack, parameter fungsi di atas EBP, variabel lokal di bawah EBP**
  ![[99-Assets/REBinex/P08-slide07.png]]
  ![[99-Assets/REBinex/P08-slide08.png]]
  ![[99-Assets/REBinex/P08-slide09.png]]
  ![[99-Assets/REBinex/P08-slide10.png]]
- **Slide 12 — kode fungsi `foo` dan visualisasi stack-nya**
  ![[99-Assets/REBinex/P08-slide12.png]]
- **Slide 14 — susunan stack setelah `a`, `b`, `c` (parameter) dan `x`, `y`, `z` (lokal) dialokasikan**
  ![[99-Assets/REBinex/P08-slide14.png]]
- **Slide 17 — visualisasi Old EIP yang berhasil ditimpa alamat kode jahat**
  ![[99-Assets/REBinex/P08-slide17.png]]

> [!warning] Slide 15, 16, dan 18 (perbandingan input 11 huruf A vs 150 huruf A, dan diagram akhir injeksi shellocde 500-byte) nggak berhasil diekstrak sebagai gambar. Ini bagian PALING VISUAL dari deck ini — perbandingan langsung "sebelum vs sesudah overflow" yang penting buat intuisi. **Buka PPT aslinya di slide 15-16 dan 18.**

## Rumus / Sintaks
```
Urutan stack (dari alamat TINGGI ke RENDAH):
  [parameter fungsi: c, b, a]   <- di ATAS EBP, ditulis kanan-ke-kiri
  [Old EIP]                     <- alamat kembali setelah fungsi selesai
  [Old EBP]                     <- EBP fungsi pemanggil
  --- EBP menunjuk ke sini ---
  [variabel lokal: x, y, z]     <- di BAWAH EBP
  --- ESP menunjuk ke sini (puncak stack) ---

Pola injeksi shellcode klasik:
  [padding/random data][shellcode][return address -> menunjuk ke shellcode]
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **`strcpy` / `gets`** | Fungsi C klasik yang menyalin/membaca string TANPA mengecek batas ukuran buffer tujuan |
| **Return address overwrite** | Teknik menimpa Old EIP di stack dengan alamat pilihan penyerang |
| **NOP sled** | Deretan instruksi `nop` yang digunakan untuk memperbesar peluang "mendarat" tepat sebelum shellcode |
| **LIFO** | Last In, First Out; sifat dasar struktur data stack |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan overflow "11 huruf A" dan "150 huruf A" ke variabel `c` (slide 14-15). Analisis: berdasarkan urutan stack yang dijelaskan (parameter di atas EBP, Old EIP, Old EBP, lalu variabel lokal di bawah EBP), kenapa 150 karakter BISA menembus sampai Old EIP sementara 11 karakter belum tentu bisa? Apa yang menentukan "jarak" dari `c` ke Old EIP?
2. **(C4 – Analisis)** Sepuluh "dangerous C functions" di slide 11 semuanya berbahaya, tapi karena ALASAN yang sedikit berbeda-beda (misalnya `gets` vs `sprintf`). Pilih DUA fungsi dari daftar itu dan analisis: apa PERBEDAAN cara mereka bisa menyebabkan buffer overflow?
3. **(C5 – Evaluasi)** Sebuah program C menggunakan `strncpy` (dianggap "lebih aman" dari `strcpy` karena menerima parameter panjang maksimum) untuk semua penyalinan string-nya. Evaluasi: apakah ini otomatis membuat program tersebut AMAN dari buffer overflow? Sebutkan skenario di mana `strncpy` yang dipakai dengan CEROBOH (misalnya, parameter panjangnya salah dihitung) tetap bisa menyebabkan overflow.
4. **(C5 – Evaluasi)** Bandingkan pola injeksi shellcode klasik (padding → shellcode → return address menunjuk ke shellcode) dengan proteksi Stack Canary yang dibahas di [[P07 - Stack Security and Exploitation]]. Evaluasi: DI MANA TEPATNYA di urutan stack canary harus ditempatkan supaya pola serangan ini bisa dideteksi SEBELUM Old EIP berhasil ditimpa?
5. **(C6 – Cipta)** Rancang (secara konseptual, dengan diagram teks) susunan stack untuk fungsi `void proses(char nama[20], int umur)` yang punya SATU variabel lokal `char buffer[10]`. Gambarkan urutan dari alamat tertinggi ke terendah (parameter, Old EIP, Old EBP, variabel lokal), lalu jelaskan: kalau `buffer` di-overflow dengan 30 karakter, elemen APA di stack yang PERTAMA KALI berisiko tertimpa, dan elemen apa yang akan tertimpa SELANJUTNYA kalau overflow-nya berlanjut?

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_REBinex]]
- [[P07 - Stack Security and Exploitation]]
- [[P02 - Assembly and Disassembly]]
- [[P09 - Exploiting with Pwntools]]
- [[REBinex - Review dan Glosari]]
