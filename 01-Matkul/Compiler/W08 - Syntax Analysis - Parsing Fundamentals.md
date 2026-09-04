---
matkul: Compilation Techniques
minggu: 8
sks: 3
sumber: Session 08_Syntax Analysis Parsing Fundamentals.pptx
tags: [kuliah/compiler, minggu/w08]
status: draft
diproses: 2026-09-04
---

# W08 — Syntax Analysis: Parsing Fundamentals

## Ringkasan
> - **Parser (syntax analyzer)** menerima TOKEN dari lexical analyzer dan MENGECEK apakah urutannya sesuai grammar bahasa — hasil sukses biasanya berupa **parse tree**. Lexical analysis cuma cek "kata VALID"; syntax analysis cek "SUSUNAN kata itu sesuai aturan bahasa".
> - Dua kategori PARSER: **Top-Down** (parse tree dibangun dari ROOT ke leaves) dan **Bottom-Up** (dibangun dari LEAVES ke root) — **LL** untuk top-down, **LR** untuk bottom-up.
> - **Parse Tree vs AST (Abstract Syntax Tree)**: parse tree berisi SEMUA simbol grammar (besar, mengikuti production rule persis), AST cuma menyimpan konstruksi bahasa yang ESENSIAL (lebih kecil, merepresentasikan MAKNA program).
> - **Grammar ambigu** diselesaikan dengan menulis ulang grammar memakai **operator precedence** (tingkatan E→T→F).
> - Dua teknik transformasi grammar WAJIB sebelum parsing: **Left Factoring** (menyatukan prefix yang sama supaya parser bisa memutuskan dengan SATU lookahead) dan **Eliminasi Left Recursion** (karena recursive-descent parser akan LOOP TAK TERHINGGA kalau grammar-nya left-recursive).

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Parser | Komponen compiler yang cek urutan token sesuai grammar |
| Parse tree | Representasi pohon yang menunjukkan bagaimana input string diturunkan dari grammar |
| AST (Abstract Syntax Tree) | Versi ringkas parse tree yang cuma simpan konstruksi esensial, merepresentasikan MAKNA program |
| Left Factoring | Teknik menyatukan prefix produksi yang sama supaya parser bisa memutuskan dengan 1 lookahead |
| Left Recursion | Kondisi non-terminal muncul sebagai simbol PERTAMA di ruas kanan produksinya sendiri |

## Isi

### Posisi Syntax Analysis dalam Compiler
```
Source Program → [Lexical Analysis] → Tokens → [Syntax Analysis] → Parse Tree/AST → [Semantic Analysis]
```
**Parser mengecek apakah urutan token mengikuti grammar bahasa.**

### Kenapa Syntax Analysis Dibutuhkan?
**Syntax Analyzer** membuat STRUKTUR SINTAKTIK dari source program yang diberikan — struktur ini biasanya berupa **parse tree**. Syntax Analyzer juga dikenal sebagai **parser**. Sintaks bahasa pemrograman dideskripsikan lewat **CFG (Context-Free Grammar)** — akan dipakai notasi **BNF (Backus-Naur Form)** untuk mendeskripsikan CFG.

Parser MENGECEK apakah source program memenuhi aturan yang tersirat dari CFG. Kalau MEMENUHI, parser membuat parse tree program itu. Kalau TIDAK, parser memberi pesan ERROR.

**Peran CFG:** memberi SPESIFIKASI SINTAKTIK PRESISI untuk bahasa pemrograman; desain grammar adalah FASE AWAL desain compiler; grammar bisa LANGSUNG dikonversi jadi parser lewat tool tertentu (parser generator).

**Perbedaan Lexical vs Syntax Analysis:** lexical analysis HANYA mengecek apakah KATA valid; syntax analysis memverifikasi apakah kata-kata itu TERSUSUN sesuai grammar bahasa pemrograman. Contoh: `if ( > x 0 )` — SEMUA token (`if`, `(`, `x`, `>`, `0`, `)`) VALID secara leksikal, tapi SUSUNANNYA salah secara sintaks (harusnya `if ( x > 0 )`).

### Apa Itu Parser?
**Definisi:** parser adalah komponen compiler yang bertanggung jawab mengecek apakah urutan token mengikuti grammar bahasa pemrograman.

**Tanggung jawab parser:**
- Menerima token dari lexical analyzer.
- Mengecek kebenaran GRAMATIKAL.
- Membangun parse tree (atau AST).
- Melaporkan error sintaks.

Parser bekerja pada STREAM TOKEN — item terkecilnya adalah TOKEN (bukan karakter). Sama seperti minggu-minggu sebelumnya: parser MEMINTA token lewat "get next token" ke lexical analyzer.

### Dua Kategori Parser
1. **Top-Down Parser** — parse tree dibangun dari ATAS ke BAWAH, dimulai dari ROOT.
2. **Bottom-Up Parser** — parse tree dibangun dari BAWAH ke ATAS, dimulai dari LEAVES.

Keduanya SAMA-SAMA memindai input dari KIRI ke KANAN (satu simbol setiap kali). Parser top-down dan bottom-up yang EFISIEN hanya bisa diimplementasikan untuk SUB-KELAS context-free grammar tertentu: **LL untuk top-down parsing, LR untuk bottom-up parsing** *(dibahas mendalam di [[W11 - Top-Down Parsing]] dan [[W15 - Bottom-Up Parsing]]).*

### Review Context-Free Grammar
Dalam CFG, kita punya: himpunan HINGGA terminal (dalam konteks ini, himpunan TOKEN), himpunan HINGGA non-terminal (syntactic-variable), himpunan HINGGA production rule berbentuk `A → α` di mana A non-terminal dan α string terminal/non-terminal (BOLEH string kosong), dan sebuah START SYMBOL.

**Contoh grammar ekspresi:**
```
E -> E + E | E - E | E * E | E / E | - E
E -> ( E )
E -> id
```

### Parse Tree
**Parse tree** adalah representasi pohon yang menunjukkan bagaimana sebuah input string DITURUNKAN dari grammar.

**Contoh:** grammar `Expr → Expr + Term`, `Expr → Term`, `Term → id`, untuk input `id + id`:
```
        Expr
       /  |  \
    Expr  +  Term
     |          |
   Term         id
     |
    id
```

### Abstract Syntax Tree (AST)
Meski parser SERING menghasilkan parse tree, fase compiler SELANJUTNYA biasanya memakai **Abstract Syntax Tree (AST)**. Berbeda dari parse tree, AST MENGHILANGKAN detail grammar sambil TETAP MEMPERTAHANKAN makna program.

**Contoh:** `a + b * c` sebagai AST — struktur pohon HANYA menunjukkan operator (`+`, `*`) dan operand (a, b, c), TANPA menyimpan simbol grammar perantara seperti `Expr`, `Term`, `Factor`.

**Perbandingan Parse Tree vs AST:**
| Aspek | Parse Tree | AST |
| --- | --- | --- |
| Isi | Berisi SEMUA simbol grammar | Berisi HANYA konstruksi bahasa yang esensial |
| Ukuran | LEBIH BESAR | LEBIH KECIL |
| Mengikuti | Production rule | Merepresentasikan SEMANTIK program |
| Dipakai di | Selama proses PARSING | Semantic analysis dan code generation |

**Parse tree mendeskripsikan BAGAIMANA program di-parse. AST mendeskripsikan APA MAKNA program itu.**

> [!info] Analogi
> Bedakan Parse Tree dan AST seperti bedakan CATATAN LENGKAP proses memasak vs RESEP RINGKAS. Parse tree itu seperti mencatat SETIAP GERAKAN memasak — "ambil pisau, potong 3 kali, taruh di piring, ambil sendok, aduk 2 kali" — detail LENGKAP sesuai "aturan resep" persis. AST itu seperti resep RINGKAS yang cuma bilang "iris bawang, tumis" — MENGHILANGKAN detail teknis tapi tetap mempertahankan MAKSUD/HASIL AKHIRNYA. Fase compiler SELANJUTNYA (semantic analysis, code generation) tidak peduli soal "bagaimana persis parser membaca grammar" — mereka cuma butuh tahu APA yang dimaksud kode itu, jadi AST lebih PRAKTIS dipakai.

### Ambiguitas Grammar
**Grammar disebut ambigu kalau SATU input string bisa menghasilkan LEBIH DARI SATU parse tree.**

**Contoh:** grammar `E → E+E | E*E | id`, input `id + id * id` — menghasilkan **DUA interpretasi berbeda** (Interpretasi 1 mengelompokkan `+` duluan, Interpretasi 2 mengelompokkan `*` duluan). **Compiler HARUS punya HANYA SATU makna** — ambiguitas ini TIDAK BOLEH dibiarkan.

**Menyelesaikan Ambiguitas:** cara UMUM — tulis ulang grammar memakai **operator precedence**.
```
Ambigu:                    Tidak Ambigu:
E -> E + E                 E -> E + T | T
E -> E * E                 T -> T * F | F
E -> (E)                   F -> (E) | id
E -> id
```

### Left Factoring
**Left factoring** MENGEKSTRAK PREFIX yang SAMA supaya parser bisa memutuskan dengan SATU token lookahead.

**Masalah umum:** `A → αβ1 | αβ2` di mana α TIDAK KOSONG dan simbol pertama β1 dan β2 (kalau ada) BERBEDA. Saat memproses α, parser TIDAK BISA tahu apakah harus mengembangkan A ke αβ1 atau A ke αβ2.

**Solusi:** tulis ulang jadi:
```
A -> αA'
A' -> β1 | β2
```
Sekarang parser bisa LANGSUNG mengembangkan A ke αA' begitu bertemu α, dan keputusan β1/β2 DITUNDA sampai lookahead berikutnya.

**Aturan umum:** untuk non-terminal A dengan DUA atau lebih alternatif berbagi PREFIX SAMA:
```
A -> αβ1 | ... | αβn | γ1 | ... | γm
```
Diubah jadi:
```
A -> αA' | γ1 | ... | γm
A' -> β1 | ... | βn
```

**Contoh 1:** `A → abB | aB | cdg | cdeB | cdfB | h`
```
Langkah 1: A -> aA' | cdg | cdeB | cdfB | h
           A' -> bB | B
Langkah 2: A -> aA' | cdA'' | h
           A' -> bB | B
           A'' -> g | eB | fB
```

### Left Recursion
**Left recursion:** sebuah non-terminal muncul sebagai simbol PERTAMA di ruas KANAN produksinya. Contoh: `Expr → Expr + Term`. Kalau grammar left-recursive, **recursive-descent parser** (dibahas [[W11 - Top-Down Parsing]]) akan memanggil `Expr()` yang memanggil `Expr()` lagi tanpa batas → **infinite loop** (`Expr() → Expr() → Expr() → ...`) — masalah SERIUS yang WAJIB dihilangkan sebelum parsing top-down.

### Eliminasi Immediate Left Recursion
**Pola umum:** `A → Aα | β` (β tidak dimulai dengan A) — dieliminasi jadi:
```
A -> βA'
A' -> αA' | ε
```

**Bentuk umum:** `A → Aα1 | ... | Aαm | β1 | ... | βn` (β1...βn tidak dimulai A):
```
A -> β1A' | ... | βnA'
A' -> α1A' | ... | αmA' | ε
```

**Contoh 1:** grammar ekspresi klasik
```
E -> E+T | T           =>  E -> TE'
T -> T*F | F               E' -> +TE' | ε
F -> id | (E)               T -> FT'
                             T' -> *FT' | ε
                             F -> id | (E)
```

### Masalah Left Recursion Tidak Langsung (Indirect)
**Grammar bisa TIDAK immediate-left-recursive, tapi TETAP left-recursive** secara tidak langsung. Contoh:
```
S -> Aa | b
A -> Sc | d
```
Grammar ini TIDAK langsung left-recursive (S tidak diawali S, A tidak diawali A) — tapi:
```
S -> Aa -> Sca   (S akhirnya "kembali" ke S sendiri secara tidak langsung)
A -> Sc -> Aac   (A akhirnya "kembali" ke A sendiri secara tidak langsung)
```
Jadi TETAP terjadi left recursion — HANYA menghilangkan immediate left recursion SAJA tidak cukup, kita harus menghilangkan SEMUA left recursion (langsung dan tidak langsung).

### Algoritma Umum Eliminasi Left Recursion
```
Urutkan non-terminal: A1, ..., An
for i dari 1 sampai n:
  for j dari 1 sampai i-1:
    ganti setiap produksi Ai -> Aj γ dengan Ai -> δ1γ | ... | δkγ
    (di mana Aj -> δ1 | ... | δk)
  eliminasi immediate left-recursion di antara produksi Ai
```

**Contoh:** `S → Aa | b`, `A → Ac | Sd | f`, urutan (S, A):
```
untuk S: tidak ada left recursion langsung
untuk A: ganti A -> Sd dengan A -> Aad | bd
         jadi A -> Ac | Aad | bd | f
         eliminasi immediate left recursion:
         A -> bdA' | fA'
         A' -> cA' | adA' | ε

Hasil akhir (tidak left-recursive):
S -> Aa | b
A -> bdA' | fA'
A' -> cA' | adA' | ε
```

> [!info] Analogi
> Left recursion itu seperti instruksi "untuk mengerjakan langkah 1, kerjakan dulu langkah 1" — sebuah lingkaran setan yang tidak pernah bisa DIMULAI secara nyata kalau diikuti PERSIS APA ADANYA (recursive-descent parser akan terus memanggil dirinya sendiri tanpa pernah "maju"). Eliminasi left recursion itu seperti menulis ulang instruksi jadi "kerjakan bagian DASAR dulu (β), BARU ulangi bagian tambahan (α) sebanyak yang dibutuhkan" — mengubah rekursi yang MENGARAH KE KIRI (macet) jadi rekursi yang MENGARAH KE KANAN (bisa terus maju, dan berhenti kalau ε dipilih).

## Diagram & Visual
> [!warning] Deck ini didominasi diagram parse tree dan derivasi yang dibuat manual dengan text-box PowerPoint (bukan gambar raster) — semua sudah direkonstruksi ke format teks/tabel di atas, tapi tampilan VISUAL asli (pohon dengan garis penghubung) perlu dibuka manual di file PPT untuk referensi visual yang lebih jelas, terutama untuk grammar ambiguity (slide 15) dan AST (slide 13).

## Rumus / Sintaks
```
Left Factoring:
A -> alpha*beta1 | ... | alpha*betan | gamma1 | ... | gammam
=>
A  -> alpha*A' | gamma1 | ... | gammam
A' -> beta1 | ... | betan

Eliminasi Immediate Left Recursion:
A -> A*alpha1 | ... | A*alpham | beta1 | ... | betan
=>
A  -> beta1*A' | ... | betan*A'
A' -> alpha1*A' | ... | alpham*A' | epsilon

Algoritma Umum Eliminasi Left Recursion (indirect):
urutkan non-terminal A1..An
for i = 1 to n:
  for j = 1 to i-1:
    substitusi Ai -> Aj*gamma dengan produksi Aj
  eliminasi immediate left recursion pada Ai
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **BNF (Backus-Naur Form)** | Notasi standar untuk menuliskan context-free grammar |
| **Lookahead** | Token yang "diintip" parser sebelum memutuskan aturan produksi mana yang dipakai |
| **Recursive-descent parser** | Parser top-down yang diimplementasikan lewat pemanggilan fungsi rekursif per non-terminal |
| **Immediate left recursion** | Left recursion di mana non-terminal LANGSUNG memanggil dirinya sendiri di posisi pertama |
| **Indirect left recursion** | Left recursion yang terjadi lewat RANTAI non-terminal lain, bukan langsung |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa AST "menghilangkan detail grammar sambil tetap mempertahankan makna program". Ambil contoh `a + b * c` — bagian STRUKTUR grammar APA (misalnya node `Expr`, `Term`, `Factor` di parse tree) yang HILANG di AST, dan kenapa fase SEMANTIC ANALYSIS tidak membutuhkan detail itu?
2. **(C4 – Analisis)** Bandingkan Left Factoring dan Eliminasi Left Recursion. Analisis: KEDUANYA sama-sama teknik transformasi grammar SEBELUM parsing, tapi MASALAH APA yang masing-masing selesaikan — Left Factoring menyelesaikan masalah APA (petunjuk: soal lookahead), sementara Eliminasi Left Recursion menyelesaikan masalah APA (petunjuk: soal infinite loop)?
3. **(C5 – Evaluasi)** Evaluasi grammar `S → Aa | b`, `A → Sc | d` yang disebut "tidak immediate left-recursive tapi TETAP left-recursive". Buktikan klaim ini dengan menunjukkan RANTAI SUBSTITUSI konkret yang membuktikan S (atau A) akhirnya "kembali ke dirinya sendiri" di posisi pertama.
4. **(C5 – Evaluasi)** Bandingkan grammar ambigu `E→E+E|E*E|id` dengan versi tidak ambigu `E→E+T|T, T→T*F|F, F→(E)|id`. Evaluasi: SELAIN menyelesaikan ambiguitas, apa efek SAMPINGAN dari transformasi ini terhadap grammar — apakah grammar hasil transformasi ini MASIH left-recursive? Kalau iya, langkah APA lagi yang perlu dilakukan sebelum grammar ini siap dipakai recursive-descent parser?
5. **(C6 – Cipta)** Kerjakan latihan #1 dari slide 30: eliminasi left recursion untuk grammar berikut, tunjukkan LANGKAH DEMI LANGKAH:
   ```
   Q -> QED | q
   E -> ε
   D -> NFA | d
   N -> DFA | n
   F -> f
   A -> a
   ```

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W07 - Context-Free Grammar]]
- [[W09 - Syntax Analysis - Parsing Strategies]]
- [[Compiler - Review dan Glosari]]
