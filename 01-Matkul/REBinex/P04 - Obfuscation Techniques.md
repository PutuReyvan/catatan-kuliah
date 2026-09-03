---
matkul: Reverse Engineering dan Binary Exploitation
minggu: 4
sks: 3
sumber: REBinex - P4.pptx
tags: [kuliah/rebinex, minggu/p04]
status: draft
diproses: 2026-09-04
---

# P04 — Obfuscation Techniques

> [!warning] Learning outcome di slide 2 nggak nyambung sama isi (masih tentang obfuscation, bukan file carving). Lihat catatan di [[P02 - Assembly and Disassembly]].

## Ringkasan
> - **Anti-reversing** = usaha pembuat program buat NYUSAHIN reverser. Tiga pendekatan dasar: **hapus info simbolik** (nama variabel/fungsi), **obfuscate/enkripsi program**, dan **sisipin kode anti-debugger**.
> - **Symbolic information** = nama variabel dan fungsi. Program C/C++ release build biasanya udah nggak punya info ini secara default; tapi bahasa bytecode kayak Java WAJIB nyimpen nama internal buat cross-referencing — jadi program Java lebih gampang di-dekompilasi jadi kode yang mirip source aslinya.
> - **Stripping** (`strip` command) buang info debugging (nama fungsi, mapping baris kode) dari biner yang di-compile pakai flag `-g`. Biner **non-stripped** nunjukin nama fungsi asli di symbol tree; biner **stripped** enggak.
> - **Enkripsi program**: dienkripsi setelah compile, DIDEKRIPSI SENDIRI saat runtime. Masalahnya: kunci dekripsi dan logika dekripsinya WAJIB ada di dalam executable-nya sendiri — dan versi yang udah didekripsi HARUS ada di memori pas dijalankan, jadi bisa "dicuri langsung dari RAM" tanpa perlu mecahin enkripsinya.
> - **Packer/unpacker**: program yang otomatis ngedekripsi executable yang di-encode dengan skema tertentu — hasilnya executable baru tanpa enkripsi.
> - **Stackstring**: teknik nyembunyiin string dari analis dengan naruh string itu ke stack **1 byte per instruksi MOV**, bukan sekali langsung sebagai literal string — biar nggak muncul jelas di daftar string statis biner.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Anti-reversing | Teknik yang dipakai pembuat program buat menyulitkan proses reverse engineering |
| Symbolic information | Nama variabel dan fungsi dalam program |
| Stripping | Menghapus info debugging (nama fungsi, mapping baris) dari biner |
| Code encryption | Mengenkripsi kode program, didekripsi sendiri saat runtime |
| Packer / Unpacker | Program yang mengenkripsi/mendekripsi ulang sebuah executable |
| Stackstring | Teknik menyembunyikan string dengan menulisnya ke stack satu byte per instruksi |
| Ghidra | Tool disassembler/decompiler open-source yang bisa dipakai untuk membongkar stackstring |

## Isi

### Basic Approach to Antireversing
Ada beberapa pendekatan anti-reversing, masing-masing punya kelebihan dan kekurangan sendiri:
1. **Eliminating Symbolic Information** — menghilangkan info simbolik
2. **Obfuscating/Encrypting the Program** — mengaburkan/mengenkripsi program
3. **Embedding Antidebugger Code** (untuk Dynamic Analysis) — menyisipkan kode anti-debugger

### Eliminating Symbolic Information
**Symbolic information** atau simbol adalah nama variabel program dan nama fungsi. Sebagai bagian dari anti-reversing, penghilangan informasi menghapus symbolic information dari program. Dalam bahasa berbasis compiler kayak C dan C++, **release build biasanya udah nggak nyertain symbolic information secara default.** Tapi, kalau sebuah program punya banyak DLL, dan DLL-DLL itu meng-export banyak fungsi, nama-nama semua fungsi yang di-export itu bisa lumayan membantu buat reverser.

Isu symbolic information itu BEDA buat sebagian besar bahasa berbasis bytecode (kayak Java). Itu karena bahasa-bahasa ini sering pake NAMA buat cross-referencing internal alih-alih alamat, jadi **semua nama internal DIPERTAHANKAN** waktu program dikompilasi. Ini menciptakan situasi di mana banyak program bytecode bisa **didekompilasi balik jadi bentuk yang sangat mirip source code aslinya, yang gampang dibaca**. String-string ini nggak bisa cuma dihapus begitu aja — mereka harus DIGANTI dengan string lain, biar cross-reference internalnya nggak putus. Strategi tipikalnya: bikin sebuah program yang ngelewatin executable-nya setelah dibuat dan cuma nge-rename semua nama internal jadi string yang nggak berarti.

**Stripping di praktik.** Kalau kamu meng-compile sebuah executable dengan flag `gcc -g`, dia berisi informasi debugging. Artinya buat tiap instruksi ada info baris source code mana yang menghasilkannya, nama variabel di source code dipertahankan dan bisa diasosiasikan ke memori yang sesuai saat runtime. **`strip`** bisa menghapus info debugging ini dan data lain yang termasuk di dalam executable yang nggak perlu buat eksekusi, dalam rangka mengurangi ukuran executable-nya.

> [!info] Analogi
> Bayangin binary yang **non-stripped** itu kayak **naskah drama dengan catatan sutradara di pinggirnya** — tiap adegan dikasih label "Adegan 3: Konfrontasi di Dapur" dan tiap karakter punya nama jelas. Binary yang **stripped** itu naskah yang SAMA persis, tapi semua label dan nama karakter udah dihapus, tinggal dialognya doang — kamu masih bisa nonton pertunjukannya (program tetep JALAN), tapi kamu nggak tau lagi adegan mana yang lagi ditonton atau siapa yang lagi ngomong (fungsi mana yang lagi dieksekusi).

### Encrypting the Program
Enkripsi kode program adalah metode umum buat mencegah analisis statis. Caranya dengan mengenkripsi program di suatu titik SETELAH dia dikompilasi (sebelum dikirim ke customer) dan menyisipkan semacam kode dekripsi di dalam executable-nya. Sayangnya, pendekatan ini biasanya cuma menciptakan ketidaknyamanan buat reverser yang terampil karena **di sebagian besar kasus, segala sesuatu yang dibutuhkan untuk dekripsi program HARUS berada di dalam executable itu sendiri.** Ini mencakup logika dekripsinya, dan yang lebih penting, **kunci dekripsinya**.

**Program itu harus mendekripsi kodenya sendiri saat runtime SEBELUM kode itu dieksekusi.** Artinya salinan program yang sudah terdekripsi (atau bagian-bagiannya) HARUS berada di memori saat runtime (kalau enggak, program nggak akan bisa jalan). **Kalau tersimpan di memori, dia bisa diekstrak keluar dari memori itu.** Dengan cara ini, penyerang bisa dapetin versi program yang sudah terdekripsi secara langsung, tanpa harus memecahkan algoritma enkripsinya.

Meski begitu, enkripsi kode adalah teknik yang umum dipakai buat menghambat analisis statis karena secara signifikan mempersulit proses analisis program dan kadang bisa memaksa reverser melakukan analisis runtime pada program. **Sayangnya, di sebagian besar kasus, program terenkripsi bisa didekripsi secara programatik** memakai program **unpacker** khusus yang familiar dengan algoritma enkripsi spesifik yang diimplementasikan di programnya dan bisa otomatis menemukan kunci dan mendekripsi programnya. **Unpacker biasanya membuat executable baru yang berisi program aslinya minus enkripsinya.**

**Melawan unpacking otomatis.** Satu-satunya cara melawan unpacking otomatis dari executable (selain memakai hardware terpisah yang menyimpan kunci dekripsi atau benar-benar melakukan dekripsinya) adalah dengan MENCOBA menyembunyikan kunci itu di dalam program. **Satu taktik efektif adalah memakai kunci yang DIHITUNG SAAT RUNTIME, di dalam program.** Algoritma pembangkitan kunci semacam itu bisa dirancang dengan mudah agar membutuhkan unpacker yang sangat canggih. Ini bisa dicapai dengan mempertahankan banyak variabel global yang terus-menerus diakses dan dimodifikasi oleh berbagai bagian program.

### Stackstring Data Obfuscation
**Stackstring** adalah teknik yang dipakai untuk **'menyembunyikan' string dari analis malware** dengan menaruh string itu ke stack **satu karakter (byte) dalam satu waktu**. Ini biasanya dilakukan dengan mengalokasikan sebuah chunk memori dan lalu memakai (banyak) instruksi **MOV** untuk menaruh string itu ke dalam chunk memori yang dialokasikan.

**Cara membongkar stackstring pakai Ghidra.** Sebagian stack string bisa diidentifikasi atau dipecahkan dengan operasi sederhana di Ghidra's Decompiler view. Dalam kasus penyalinan satu-byte, mungkin bisa membaca nilai stack string dengan mengupdate deskripsi stack frame Ghidra sehingga byte-byte yang buram itu diinterpretasikan sebagai elemen di dalam array karakter.

**Contoh praktik: `strings2.exe` dari MalwareTech.** Bisa dilihat ada banyak operasi MOV, masing-masing memindahkan SATU nilai byte ke stack. Untuk mendekode stack string-nya, di Ghidra kamu bisa **retype elemen paling awal jadi `char[n]`**, di mana `n` adalah panjang string yang mau direkonstruksi. Dengan menghitung jumlah operasi MOV-nya, ternyata ada **37 karakter** — jadi variabelnya di-retype jadi `char[37]`. Hasilnya, data yang tadinya diobfuskasi jadi bisa dibaca dengan jelas.

> [!info] Analogi
> Stackstring itu kayak **catatan rahasia yang ditulis satu huruf per selembar kertas kecil, terus kertas-kertasnya disebar ke banyak laci berbeda**, alih-alih ditulis sekaligus di satu lembar kertas utuh. Kalau kamu cuma nge-scan isi laci satu-satu dengan cepat (mencari string yang "keliatan jelas" di biner), kamu nggak bakal langsung nemu pesannya — kamu harus tau URUTAN membuka laci yang bener (urutan instruksi MOV) buat nyusun ulang pesannya jadi utuh. Ghidra bisa bantu kalau kamu kasih tau dia "ini ada 37 laci yang harus dibaca berurutan sebagai satu pesan" (`retype ke char[37]`).

## Diagram & Visual
- **Slide 7, 8 — contoh symbol tree non-stripped binary vs stripped binary**
  ![[99-Assets/REBinex/P04-slide07.png]]
  ![[99-Assets/REBinex/P04-slide08.png]]
- **Slide 12 — diagram Packer-Unpacker**
  ![[99-Assets/REBinex/P04-slide12.jpg]]
- **Slide 16-19 — screenshot langkah demi langkah membongkar stackstring di `strings2.exe` pakai Ghidra**
  ![[99-Assets/REBinex/P04-slide16.png]]
  ![[99-Assets/REBinex/P04-slide17.png]]
  ![[99-Assets/REBinex/P04-slide18.png]]
  ![[99-Assets/REBinex/P04-slide19.png]]

> [!warning] Bagian stackstring (slide 16-19) itu tutorial VISUAL langkah-demi-langkah di Ghidra — teks slide-nya menjelaskan KONSEP-nya, tapi tampilan tool dan hasil retype-nya cuma bisa dilihat dari gambar. Kalau mau praktik ngikutin persis, buka gambarnya atau referensi aslinya: `krekelbits.wordpress.com/2018/07/25/solving-malwaretechs-string-2` (dicatat di speaker notes slide asli).

## Rumus / Sintaks
```bash
# compile dengan info debugging
gcc -g program.c -o program

# hapus info debugging (stripping) dari binary yang sudah ada
strip program
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **DLL (Dynamic Link Library)** | File library Windows yang fungsi-fungsinya bisa di-export dan dipakai program lain |
| **Static analysis** | Menganalisis program TANPA menjalankannya (baca kode/biner-nya saja) |
| **Dynamic/Runtime analysis** | Menganalisis program SAAT dijalankan (observasi perilakunya secara langsung) |
| **Unpacker** | Program yang otomatis mendekripsi/mengembalikan executable yang di-pack/dienkripsi ke bentuk aslinya |
| **Ghidra Decompiler view** | Fitur di Ghidra yang menampilkan hasil dekompilasi (mendekati kode sumber) dari sebuah fungsi |
| **`retype`** | Aksi di Ghidra untuk mengubah tipe data sebuah variabel secara manual, dipakai untuk membongkar stackstring |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan "eliminating symbolic information" pada program C/C++ (release build otomatis nggak punya simbol) vs program bytecode seperti Java (WAJIB menyimpan nama internal). Analisis: kenapa perbedaan ini terjadi, dan apa konsekuensinya buat seberapa "readable" hasil dekompilasi masing-masing jenis program?
2. **(C4 – Analisis)** Slide 10-11 menjelaskan bahwa program terenkripsi HARUS mendekripsi dirinya sendiri di memori saat runtime, dan itu jadi celah bagi penyerang. Analisis: kenapa celah ini TIDAK BISA dihindari sepenuhnya hanya dengan mengenkripsi program lebih kuat (misalnya pakai algoritma enkripsi yang lebih canggih)? Apa akar masalah strukturalnya?
3. **(C5 – Evaluasi)** Sebuah developer malware memutuskan memakai kunci dekripsi yang dihitung secara dinamis saat runtime (bukan hardcode) sebagai pertahanan melawan unpacker otomatis (slide 13). Evaluasi: apakah teknik ini menghilangkan SEPENUHNYA risiko "dekripsi diambil dari memori" yang dijelaskan di slide 10? Jelaskan kenapa ya atau tidak.
4. **(C5 – Evaluasi)** Bandingkan efektivitas stripping (menghapus simbol) dengan stackstring (menyembunyikan string) sebagai teknik anti-reversing. Kalau seorang reverser TIDAK punya akses ke Ghidra atau tool decompiler canggih, teknik mana yang menurutmu lebih efektif menghambat mereka, dan kenapa?
5. **(C6 – Cipta)** Rancang strategi analisis (langkah-langkah, bukan kode) untuk membongkar sebuah binary yang MENGGABUNGKAN dua teknik sekaligus: sudah di-strip (tanpa nama fungsi) DAN memakai stackstring untuk menyembunyikan pesan pentingnya. Urutkan langkah mana yang kamu lakukan duluan dan kenapa.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_REBinex]]
- [[P03 - Assembly and Disassembly - The Practical Journey]]
- [[P05 - Debugging and Dynamic Analysis]]
- [[REBinex - Review dan Glosari]]
