---
matkul: Compilation Techniques
minggu: 3
sks: 3
sumber: Session 03_Lexical Analysis.pptx
tags: [kuliah/compiler, minggu/w03]
status: draft
diproses: 2026-09-04
---

# W03 — Lexical Analysis (Scanning)

## Ringkasan
> - **Lexical analyzer (scanner)** membaca ALIRAN karakter source program dan mengelompokkannya jadi urutan bermakna disebut **lexeme**. Untuk tiap lexeme, dihasilkan **token** berbentuk `<token-name, attribute-value>`.
> - Tiga istilah kunci yang SERING tertukar: **Pattern** (deskripsi bentuk lexeme sebuah token), **Token** (pasangan nama-token + atribut opsional), **Lexeme** (urutan karakter NYATA di source code yang cocok dengan pattern).
> - Lexical analyzer BEKERJA lewat pull-model: TIDAK mengembalikan semua token sekaligus, tapi mengembalikan SATU token setiap kali PARSER memintanya ("get next token").
> - **Input buffering** dibutuhkan karena kita tidak bisa yakin sudah melihat AKHIR sebuah identifier sampai bertemu karakter yang BUKAN huruf/digit — dipecahkan lewat skema **buffer pairs** dengan dua pointer (`lexemeBegin` dan `forward`).
> - **DFA (Deterministic Finite Automaton)** punya TEPAT SATU transisi per state-simbol; **NFA (Non-deterministic FA)** boleh punya BANYAK transisi (bahkan transisi ε tanpa mengonsumsi simbol) — keduanya jadi dasar cara scanner "mengenali" token.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Lexeme | Urutan karakter nyata di source code yang cocok dengan pattern token |
| Token | Pasangan nama-token dan atribut opsional, `<token-name, attribute-value>` |
| Pattern | Deskripsi bentuk yang boleh diambil lexeme dari sebuah token |
| Symbol table | Tabel yang menyimpan atribut lengkap identifier, dirujuk lewat pointer dari token |
| DFA | Automaton dengan tepat satu transisi per pasangan state-simbol |
| NFA | Automaton yang boleh punya banyak transisi, termasuk transisi ε |

## Isi

### Lexical Analyzer: Fungsi Dasar
**Lexical analyzer** membaca ALIRAN karakter yang membentuk source program dan mengelompokkan karakter-karakter itu jadi urutan bermakna disebut **lexeme**. Untuk setiap lexeme, lexical analyzer menghasilkan output berupa **token** berbentuk:
```
<token-name, attribute-value>
```

**Contoh:**
```
Source Program:    position = initial + rate * 60
Sequence of tokens: <id,1> <=> <id,2> <+> <id,3> <*> <60>
```

### Token, Pattern, Lexeme
Tiga istilah yang sering tertukar tapi PUNYA MAKNA BERBEDA:
- **Pattern** — deskripsi BENTUK yang boleh diambil lexeme dari sebuah token.
- **Token** — PASANGAN yang terdiri dari nama token dan nilai atribut OPSIONAL.
- **Lexeme** — urutan KARAKTER di source program yang cocok dengan pattern sebuah token dan diidentifikasi lexical analyzer sebagai INSTANCE dari token itu.

> [!info] Konteks tambahan (bukan dari slide)
> Nama token adalah simbol ABSTRAK yang merepresentasikan JENIS unit leksikal. Nama token adalah simbol input yang diproses PARSER. Token itu KATEGORI (seperti keyword atau identifier) yang merepresentasikan unit makna. Pattern mendefinisikan STRUKTUR yang cocok dengan sebuah token (misalnya regular expression untuk identifier). Lexeme adalah urutan karakter SESUNGGUHNYA di source code yang cocok dengan pattern token itu.

> [!info] Analogi
> Bayangkan Token, Pattern, dan Lexeme seperti KATEGORI PRODUK di toko. **Pattern** itu seperti "aturan" yang menentukan produk apa masuk kategori "Minuman Kaleng" (kemasan kaleng, isi cairan). **Token** itu NAMA KATEGORI itu sendiri, misalnya `minuman-kaleng`, kadang dengan info tambahan (atribut) seperti "merek: Cola". **Lexeme** itu BARANG SESUNGGUHNYA yang kamu ambil dari rak — sekaleng Coca-Cola tertentu yang benar-benar ada di tanganmu, yang COCOK dengan pattern "Minuman Kaleng" dan diberi label token `minuman-kaleng`.

### Kelas-Kelas Token
| Token | Deskripsi Informal | Contoh Lexeme |
| --- | --- | --- |
| `if` | karakter i, f | if |
| `else` | karakter e, l, s, e | else |
| `comparison` | `<` atau `>` atau `<=` atau `>=` atau `==` atau `!=` | `<=`, `!=` |
| `id` | huruf diikuti huruf dan digit | pi, score, D2 |
| `number` | konstanta numerik apa pun | 3.14159, 0, 6.02e23 |
| `literal` | apa pun kecuali `"`, dikelilingi `"` | "core dumped" |

### Token dan Atribut
**Token merepresentasikan SEKUMPULAN string yang dideskripsikan sebuah pattern.** Contoh: Identifier merepresentasikan sekumpulan string yang DIMULAI huruf, DILANJUTKAN huruf dan digit. String AKTUAL (misalnya `newval`) disebut LEXEME.

Karena satu token bisa merepresentasikan LEBIH DARI SATU lexeme, informasi TAMBAHAN perlu disimpan untuk lexeme spesifik itu — informasi tambahan ini disebut **ATRIBUT** token. Untuk kesederhanaan, sebuah token bisa punya SATU atribut yang menyimpan informasi yang dibutuhkan. Untuk identifier, atribut ini adalah POINTER ke **symbol table**, dan symbol table menyimpan atribut sesungguhnya untuk token itu.

**Contoh atribut:**
```
<id, attr>     -> attr adalah pointer ke symbol table
<assgop, _>    -> tidak butuh atribut (kalau cuma ada satu operator assignment)
<num, val>     -> val adalah nilai aktual dari angka itu
```
**Tipe token dan atributnya SECARA UNIK mengidentifikasi sebuah lexeme.** Regular expression banyak dipakai untuk menspesifikasikan pattern.

### Lexical Errors
Cukup SULIT bagi lexical analyzer mengetahui bahwa ada error KODE SUMBER. Contoh: `fi ( a == f(x) ) …` — "fi" bisa jadi typo dari "if", tapi lexical analyzer TIDAK PUNYA CARA mengetahui itu (secara leksikal, "fi" tetap terlihat seperti identifier yang VALID).

**Strategi recovery paling sederhana: "panic mode" recovery.** Aksi error-recovery lain yang mungkin:
- Menghapus SATU karakter dari sisa input.
- Menyisipkan karakter yang HILANG dari sisa input.
- Mengganti sebuah karakter dengan karakter LAIN.
- Menukar (transpose) DUA karakter yang bersebelahan.

### Input Buffering
Kita TIDAK BISA yakin sudah melihat AKHIR sebuah identifier sampai bertemu karakter yang BUKAN huruf atau digit. Di bahasa C, operator satu-karakter seperti `-`, `=`, atau `<` bisa juga jadi AWAL dari operator dua-karakter seperti `->`, `==`, atau `<=`.

Solusinya:
1. Memakai skema **two-buffer** yang menangani LOOKAHEAD besar.
2. Mempertimbangkan memakai **"sentinels"**.

**Buffer Pairs** — dua POINTER ke input dijaga:
- Pointer **lexemeBegin** — menandai AWAL lexeme yang sedang diproses.
- Pointer **forward** — bergerak MAJU mencari akhir lexeme, sampai ketemu karakter yang menandakan lexeme SUDAH berakhir.

> [!info] Analogi
> Buffer pairs itu seperti dua jari yang kamu pakai membaca kalimat: jari PERTAMA (`lexemeBegin`) tetap DIAM di AWAL kata yang sedang kamu baca, sementara jari KEDUA (`forward`) terus BERGERAK MAJU karakter demi karakter sampai kamu YAKIN kata itu sudah SELESAI (misalnya ketemu spasi). Kamu perlu DUA jari karena kamu tidak tahu sebuah kata SUDAH SELESAI sampai kamu MELIHAT karakter setelahnya — kalau cuma pakai satu jari, kamu tidak bisa "mundur" untuk menandai di mana kata itu SEBENARNYA dimulai.

### Alur Kerja Lexical Analyzer
Lexical Analyzer membaca source program KARAKTER DEMI KARAKTER untuk menghasilkan token. Secara NORMAL, lexical analyzer TIDAK mengembalikan daftar token sekaligus — ia mengembalikan SATU token setiap kali PARSER MEMINTANYA.

```
Source Program → [Lexical Analyzer] --token--> [Parser]
                       ↑
                  "get next token"
                  (diminta oleh Parser)
```

### Regular Expressions dan Regular Definitions
**Regular expression (RE)** dipakai untuk mendeskripsikan token bahasa pemrograman. Sebuah RE dibangun dari RE yang LEBIH SEDERHANA (memakai aturan pendefinisian). Setiap regular expression MENDENOTASIKAN sebuah bahasa — bahasa yang didenotasikan RE disebut **regular set**.

*(Detail Regular Definition dan contoh Identifier Pascal / Unsigned Number sudah dibahas di [[W02 - Automata dan Language Theory]] — materi ini diulang persis sama di deck minggu ini sebagai penguatan, sekarang dikaitkan LANGSUNG ke konteks token compiler.)*

### Finite Automata: DFA dan NFA
**DFA (Deterministic Finite Automaton)** — model matematika dengan input-output DISKRET. Konfigurasi internalnya disebut **"state"**. Ada TRANSISI antar state berdasarkan simbol input. **Hanya ADA SATU transisi** dari satu state dengan sebuah simbol input SPESIFIK (deterministic — makanya namanya). Graph berarah yang mendeskripsikan FA disebut **"Transition Diagram"**.

**NFA (Non-deterministic Finite Automaton)** — model matematika terdiri dari:
- **S** — sekumpulan STATE.
- **Σ (sigma)** — sekumpulan simbol input (alphabet).
- **move** — fungsi transisi yang memetakan pasangan state-simbol ke SEKUMPULAN state (BISA LEBIH DARI SATU, beda dari DFA).
- **s0** — state AWAL (start/initial state).
- **F** — sekumpulan ACCEPTING state (final state).

Transisi **ε (epsilon)** DIIZINKAN di NFA — artinya kita bisa PINDAH dari satu state ke state lain TANPA MENGONSUMSI simbol apa pun. **Sebuah NFA menerima string x, JIKA DAN HANYA JIKA ada PATH dari starting state ke salah satu accepting state sedemikian rupa sehingga label edge di sepanjang path itu MENGEJA (spell out) string x.**

> [!info] Konteks tambahan (bukan dari slide)
> Perbedaan MENDASAR DFA vs NFA: di DFA, setiap "langkah" itu PASTI dan TUNGGAL — dari satu state, dengan satu simbol input, hasilnya SELALU satu state tertentu. Di NFA, dari satu state dengan satu simbol input, ada BEBERAPA kemungkinan state tujuan sekaligus (atau bahkan bisa "melompat" state lewat transisi ε tanpa baca simbol apa pun) — seolah automaton itu men-"CABANG" ke banyak kemungkinan sekaligus dan menerima string kalau SALAH SATU cabang itu berhasil sampai ke accepting state. NFA lebih mudah DIRANCANG dari regular expression, tapi lebih SULIT diimplementasikan langsung di komputer (yang butuh determinisme) — makanya butuh KONVERSI NFA ke DFA, topik yang dibahas mendalam minggu depan di [[W05 - RE ke DFA]].

## Diagram & Visual
- **Slide 9 — Ilustrasi Token dan Atribut**
  ![[99-Assets/Compiler/W03-slide09.png]]
- **Slide 12 — Diagram Buffer Pairs**
  ![[99-Assets/Compiler/W03-slide12.png]]
  ![[99-Assets/Compiler/W03-slide12a.png]]

> [!warning] Slide 17-18 ("Finite automata") tidak berhasil diekstrak sebagai gambar/teks — kemungkinan diagram transisi yang tidak berupa gambar raster biasa. Buka file PPT asli untuk melihat contoh transition diagram DFA/NFA secara visual.

## Rumus / Sintaks
```
Format Token: <token-name, attribute-value>

Contoh:
position = initial + rate * 60
->  <id,1> <=> <id,2> <+> <id,3> <*> <60>

NFA formal: (S, Σ, move, s0, F)
S     = himpunan state
Σ     = alphabet (himpunan simbol input)
move  = fungsi transisi state x simbol -> himpunan state
s0    = state awal
F     = himpunan accepting/final state
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Panic mode recovery** | Strategi error-recovery paling sederhana di lexical analyzer |
| **Lookahead** | Karakter yang "diintip" scanner sebelum memutuskan akhir sebuah lexeme |
| **Sentinel** | Karakter penanda khusus di ujung buffer untuk mendeteksi batas input secara efisien |
| **Transition Diagram** | Graph berarah yang menggambarkan sebuah finite automaton |
| **Regular set** | Bahasa yang didenotasikan oleh sebuah regular expression |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa lexical analyzer "sulit mengetahui ada error source-code" memakai contoh `fi ( a == f(x) )`. Kaitkan dengan definisi Token/Pattern/Lexeme — kenapa "fi" secara LEKSIKAL tetap valid meski secara SEMANTIK/SINTAKS itu kemungkinan typo?
2. **(C4 – Analisis)** Bandingkan pull-model ("lexical analyzer mengembalikan token SAAT DIMINTA parser") dengan hipotetis push-model (lexical analyzer memproses SEMUA token sekaligus di awal, menyimpannya di list). Analisis: keuntungan APA dari pull-model dari sisi efisiensi memori, terutama untuk source program yang SANGAT BESAR?
3. **(C5 – Evaluasi)** Evaluasi kebutuhan skema "buffer pairs" (dua pointer lexemeBegin dan forward) untuk kasus operator dua-karakter di C (`->`, `==`, `<=`). Jelaskan APA yang akan terjadi kalau scanner HANYA memakai SATU pointer dan langsung memutuskan token begitu bertemu karakter `<` (tanpa mengintip karakter berikutnya) — token APA yang salah dikenali untuk input `<=`?
4. **(C5 – Evaluasi)** Bandingkan DFA dan NFA dari sisi KEMUDAHAN implementasi vs KEMUDAHAN perancangan. Evaluasi: kenapa NFA "lebih mudah dirancang dari regular expression" tapi "lebih sulit diimplementasikan langsung di komputer" — apa yang membuat determinisme (DFA) penting untuk EKSEKUSI nyata di mesin?
5. **(C6 – Cipta)** Rancang tabel token (mengikuti format tabel "Classes of Token" di atas) untuk BAHASA PEMROGRAMAN FIKTIF sederhana yang punya keyword `loop`, `stop`, operator perbandingan `eq`/`neq`, identifier, dan angka. Sertakan minimal 5 baris token (Token | Deskripsi Informal | Contoh Lexeme), lalu tulis urutan token untuk baris kode fiktif: `loop x neq 10 stop`.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W02 - Automata dan Language Theory]]
- [[W04 - DFA dan NFA]]
- [[Compiler - Review dan Glosari]]
