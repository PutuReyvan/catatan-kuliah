---
matkul: Compilation Techniques
minggu: 1
sks: 3
sumber: Session 01_Intro to Compiler.pptx
tags: [kuliah/compiler, minggu/w01]
status: draft
diproses: 2026-09-04
---

# W01 — Introduction to Compiler

## Ringkasan
> - **Compiler** = program yang MEMBACA program dalam satu bahasa (source language) dan MENERJEMAHKANNYA ke bahasa lain (target language) — kalau ada error, compiler mendeteksi dan melaporkannya.
> - Bahasa pemrograman berevolusi dari **machine language** (biner, sulit) → **assembly language** (1950-an, lebih manusiawi tapi masih hardware-dependent) → **high-level language** (abstrak, portable, mudah dibaca).
> - Proses kompilasi punya **2 bagian besar**: **Analysis** (memecah source program jadi bagian-bagian, bikin intermediate representation) dan **Synthesis** (membangun target program dari intermediate representation itu).
> - **Language Processing System** lengkap melibatkan lebih dari cuma compiler: **Preprocessor → Compiler → Assembler → Linker/Loader** — masing-masing "sepupu" (cousin) compiler dengan peran berbeda.
> - Compiler sesungguhnya terdiri dari **6 tahap** (fase) berurutan: Lexical Analyzer → Syntax Analyzer → Semantic Analyzer → Intermediate Code Generator → Code Optimizer → Code Generator — DIDAMPINGI oleh Symbol-Table Manager dan Error Handler yang aktif di SEMUA tahap.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Compiler | Program yang menerjemahkan source language jadi target language |
| Source language / Target language | Bahasa asal (ditulis programmer) dan bahasa tujuan (hasil terjemahan) |
| Analysis | Memecah source program jadi bagian, bikin intermediate representation |
| Synthesis | Membangun target program dari intermediate representation |
| Language Processing System | Rangkaian lengkap: preprocessor, compiler, assembler, linker/loader |
| Symbol-table manager | Komponen yang mencatat informasi identifier selama seluruh proses kompilasi |

## Isi

### Evolusi Bahasa Pemrograman
Komputer awal (1940-an) diprogram memakai **machine language** — instruksi biner (0 dan 1) yang LOW-LEVEL dan HARDWARE-DEPENDENT, sulit ditulis, dipahami, dan di-debug. Untuk meningkatkan produktivitas, **assembly language** diperkenalkan tahun 1950-an. Kemudian, **high-level programming language** dikembangkan supaya pemrograman lebih intuitif, mudah dipelihara, dan portable.

Perkembangan high-level language secara signifikan meningkatkan pemrograman lewat: memperkenalkan ABSTRAKSI dari detail hardware, mendukung pemrograman TERSTRUKTUR dan MODULAR, meningkatkan KETERBACAAN dan kemudahan PEMELIHARAAN. Seiring bahasa pemrograman berevolusi, desain compiler juga jadi lebih KOMPLEKS dan PENTING — **kemajuan bahasa pemrograman MENDORONG kemajuan desain compiler.**

> [!info] Analogi
> Bayangkan evolusi ini seperti evolusi cara manusia menerjemahkan bahasa asing. Machine language itu seperti berkomunikasi HANYA lewat kode Morse — teknis presisi tapi sangat sulit dan lambat untuk manusia biasa. Assembly language itu seperti kamus terjemahan kata-per-kata — sedikit lebih mudah, tapi kamu masih harus tahu struktur bahasa targetnya persis. High-level language itu seperti punya PENERJEMAH PROFESIONAL — kamu tinggal bicara natural dalam bahasamu sendiri (source language), dan penerjemah (compiler) yang mengurus semua detail teknis menerjemahkannya ke bahasa target dengan benar.

### Pemahaman Dasar: Apa Itu Compiler
**Compiler adalah program yang MEMBACA program yang ditulis dalam suatu bahasa (source language) dan MENERJEMAHKANNYA ke bahasa lain (target language).**

> [!info] Analogi
> Proses ini dianalogikan seperti proses terjemahan dalam kehidupan sehari-hari: kamu (Source Language) berbicara ke penerjemah (Translator) yang mengubahnya ke Bahasa Lain (Other Language) untuk didengar orang lain. Proses kompilasi di komputer bekerja PERSIS SAMA — Source Language diproses Compiler menjadi Target Language.

### Struktur Umum Compiler
Compiler menerjemahkan source program jadi target program yang EKUIVALEN. Selama kompilasi:
- Source code DIANALISIS.
- Error DIDETEKSI dan DILAPORKAN (pesan error dihasilkan saat compiler mendeteksi error leksikal, sintaks, atau semantik).
- Target code DIHASILKAN kalau memungkinkan.

### Klasifikasi Compiler
Compiler bisa diklasifikasikan berdasarkan PERILAKUNYA:
- **Single-pass Compiler** — melewati bagian-bagian tiap unit kompilasi HANYA SEKALI, langsung menerjemahkan tiap bagian jadi kode mesin final.
- **Multi-pass Compiler** — memproses source code/abstract syntax tree BEBERAPA KALI.
- **Load-and-go Compiler** — bentuk INTERMEDIATE program umumnya disimpan di RAM, TIDAK disimpan ke file system (langsung dieksekusi setelah kompilasi).
- **Debugging Compiler** — menghasilkan informasi debugging saat kompilasi.
- **Optimizing Compiler** — mencoba MEMINIMALKAN atau MEMAKSIMALKAN atribut tertentu dari program executable yang dihasilkan.

### Cousins of the Compiler (Sepupu Compiler)
Compiler bukan satu-satunya komponen dalam menghasilkan program yang bisa dijalankan. **Model kompilasi** terdiri dari 2 bagian:
1. **Analysis (bagian 1)** — memecah source program jadi bagian-bagian konstituen, membuat **intermediate representation** dari source program.
2. **Synthesis (bagian 2)** — membangun target program yang diinginkan DARI intermediate representation itu.

**Language Processing System** yang lengkap mencakup:
- **Preprocessor** — menangani macro, penyertaan file (file inclusion).
- **Compiler** — menerjemahkan source code jadi target code (biasanya assembly).
- **Assembler** — mengonversi kode assembly jadi kode mesin.
- **Linker/Link-editor** — menggabungkan beberapa object file jadi satu.
- **Loader** — memuat program ke memori untuk dieksekusi.

Alurnya: **Skeletal Source Program → (Preprocessor) → Source Program → (Compiler) → Target Assembly Program → (Assembler) → Relocatable Machine Code → (Loader/Link-editor) → Absolute Machine Code.**

> [!info] Konteks tambahan (bukan dari slide)
> Disebut "cousins" (sepupu) karena mereka semua BERKELUARGA (sama-sama bagian dari proses mengubah kode sumber jadi program yang bisa dijalankan), tapi masing-masing punya TUGAS SPESIFIK yang berbeda: preprocessor menangani teks SEBELUM kompilasi, compiler menerjemahkan bahasa tingkat tinggi ke assembly, assembler menerjemahkan assembly ke biner, linker menggabungkan potongan-potongan kode jadi satu, dan loader menaruhnya siap dieksekusi di memori. Compiler HANYA satu bagian dari rantai proses yang lebih besar ini.

### 6 Tahap (Fase) Kompilasi
Proses kompilasi terdiri dari enam tahap berurutan, dengan **Symbol-Table Manager** dan **Error Handler** aktif di SEMUA tahap:

```
Source Program
     ↓
1. Lexical Analyzer
     ↓
2. Syntax Analyzer
     ↓
3. Semantic Analyzer
     ↓
4. Intermediate Code Generator
     ↓
5. Code Optimizer
     ↓
6. Code Generator
     ↓
Target Program (Object Code)
```

Symbol-table manager dan Error handler terhubung ke SEMUA 6 tahap secara paralel — mencatat informasi identifier dan menangani error di SETIAP tahap, bukan hanya di satu tahap tertentu.

> [!info] Konteks tambahan (bukan dari slide)
> Enam tahap ini adalah kerangka STANDAR yang dipakai di hampir semua textbook compiler (termasuk "Dragon Book" karya Aho, Sethi, Ullman yang jadi salah satu referensi matkul ini) — dan akan jadi PETA BESAR seluruh matkul ini: [[W03 - Lexical Analysis]] sampai [[W06 - DFA Minimization]] membahas tahap 1 (Lexical Analyzer), [[W07 - Context-Free Grammar]] sampai [[W18 - Syntax Directed Translation]] membahas tahap 2 (Syntax Analyzer) dan tahap 3 (Semantic Analyzer, [[W20 - Semantic Analyzer]]), [[W21 - Intermediate Code Generator]] membahas tahap 4, [[W23 - Code Optimization]] membahas tahap 5, dan [[W25 - Code Generation]] membahas tahap 6 — SELURUH matkul ini pada dasarnya adalah pembahasan MENDALAM dari diagram 6 tahap ini, satu per satu.

## Diagram & Visual
- **Slide 16 — Contoh terjemahan (translation example) dari source code ke target code**
  ![[99-Assets/Compiler/W01-slide16.png]]

> [!warning] Beberapa slide (9, 10, 13, 15, 17, 18) berjudul "Basic understanding", "Cousins of the Compiler", "Preprocessor functions", "Stages compilation" (diagram kedua), dan "Compiler construction tools" — kemungkinan berisi DIAGRAM/SmartArt yang TIDAK terekstrak sebagai gambar biasa (bukan format raster PICTURE). Isi konseptualnya sudah direkonstruksi di atas dari speaker notes dan teks yang tersedia, tapi diagram visualnya perlu dibuka manual di file PPT asli.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Intermediate Representation (IR)** | Bentuk perantara program antara source code dan target code, dipakai internal compiler |
| **Object code** | Kode hasil kompilasi, biasanya dalam bentuk kode mesin/assembly |
| **Source-to-source translator** | Compiler yang menerjemahkan satu bahasa tingkat tinggi ke bahasa tingkat tinggi lain |
| **Relocatable machine code** | Kode mesin yang alamatnya belum final, masih bisa dipindah linker |
| **Absolute machine code** | Kode mesin dengan alamat FINAL, siap dieksekusi langsung |

## Pertanyaan Terbuka
- Slide "Cousins of the Compiler" dan "Compiler construction tools" (2 slide) tidak punya detail teks/gambar yang terekstrak — perlu dibuka manual untuk detail lengkap tool-tool konstruksi compiler yang disebutkan (misalnya scanner generator, parser generator).

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa "kemajuan bahasa pemrograman mendorong kemajuan desain compiler" (bukan sebaliknya). Berikan alasan konkret kenapa arah kausalitas ini masuk akal — fitur bahasa APA yang membutuhkan compiler jadi lebih kompleks untuk mendukungnya?
2. **(C4 – Analisis)** Bandingkan bagian Analysis dan Synthesis dalam model kompilasi. Analisis: kenapa intermediate representation jadi "jembatan" penting antara keduanya — apa yang terjadi kalau compiler LANGSUNG menerjemahkan source ke target TANPA intermediate representation?
3. **(C5 – Evaluasi)** Evaluasi pertanyaan dari slide latihan: "Apa keuntungan bagi language-processing system di mana compiler menghasilkan assembly language, bukan machine language langsung?" Jelaskan jawabanmu dengan mengaitkan peran Assembler sebagai salah satu "cousin" compiler.
4. **(C5 – Evaluasi)** Bandingkan Single-pass Compiler dan Multi-pass Compiler. Evaluasi: untuk bahasa pemrograman MODERN yang kompleks (misalnya dengan banyak fitur type inference dan optimisasi canggih), kenapa Multi-pass lebih umum dipakai dibanding Single-pass, meski Single-pass secara teori lebih CEPAT?
5. **(C6 – Cipta)** Rancang diagram alur (pseudo-diagram tekstual) yang menunjukkan bagaimana SATU baris kode source `x = a + b * 2;` akan diproses melalui KEENAM tahap kompilasi (Lexical → Syntax → Semantic → Intermediate Code → Optimizer → Code Generator), dengan menyebutkan SECARA UMUM apa yang terjadi ke kode itu di SETIAP tahap (detail lengkap akan dipelajari di minggu-minggu berikutnya, cukup gambaran besarnya saja).

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W02 - Automata dan Language Theory]]
- [[Compiler - Review dan Glosari]]
