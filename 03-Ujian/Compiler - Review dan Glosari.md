---
matkul: Compilation Techniques
tags: [kuliah/compiler, ujian]
status: draft
diproses: 2026-09-04
---

# Compilation Techniques — Review dan Glosari

## Cara Pakai
Baca ini SEBELUM buka slide asli — tujuannya kasih peta cepat semua materi satu matkul biar inget alur besarnya dulu, baru kalau perlu detail buka note per minggu ([[_Compiler]]).

## Ringkasan Cepat per Minggu

**[[W01 - Introduction to Compiler]]** — Compiler menerjemahkan source language ke target language lewat 2 fase (Analysis-Synthesis) dan 6 tahap detail (Lexical→Syntax→Semantic→Intermediate Code→Optimizer→Code Generator). Language Processing System lengkap: Preprocessor→Compiler→Assembler→Linker/Loader.

**[[W02 - Automata dan Language Theory]]** — Symbol→Alphabet→String→Language adalah hierarki konsep dasar. Regular Definition memberi NAMA ke RE supaya lebih mudah dibaca (contoh: identifier Pascal).

**[[W03 - Lexical Analysis]]** — Lexical analyzer mengelompokkan karakter jadi lexeme, hasilkan token `<name, attribute>`. Parser MEMINTA token satu-satu (pull model). Input buffering (buffer pairs, lexemeBegin+forward) menangani lookahead.

**[[W04 - DFA dan NFA]]** — NFA boleh banyak transisi + ε-transition; DFA tepat SATU transisi per state-simbol. Setiap NFA punya DFA ekuivalen via subset construction + ε-closure.

**[[W05 - RE ke DFA]]** — Dua jalur: Thompson's Construction (RE→NFA-ε→subset construction→DFA) ATAU langsung (syntax tree→firstpos/lastpos/followpos→DFA) tanpa lewat NFA sama sekali.

**[[W06 - DFA Minimization]]** — Table-filling (tandai pasangan distinguishable bertahap) dan Partition Method (pecah grup accepting/non-accepting sampai stabil) — dua cara mencapai DFA dengan state PALING SEDIKIT.

**[[W07 - Context-Free Grammar]]** — Chomsky hierarchy: Type-0 (Turing Machine) sampai Type-3 (Regular, Finite Automaton); CFG = Type-2, dikenali PDA. Grammar ambigu = 1 string, >1 parse tree — masalah SERIUS untuk compiler.

**[[W08 - Syntax Analysis - Parsing Fundamentals]]** — Parser cek token sesuai grammar, bangun parse tree/AST. AST lebih ringkas dari parse tree (buang detail grammar, simpan makna). Left Factoring dan Eliminasi Left Recursion WAJIB sebelum top-down parsing.

**[[W09 - Syntax Analysis - Parsing Strategies]]** — LMD (kiri dulu) vs RMD (kanan dulu). Top-Down (root→leaves, prediksi, LL) vs Bottom-Up (leaves→root, reduksi, LR) — Bottom-up bisa handle left-recursive, Top-down tidak. 4 teknik error recovery.

**[[W10 - Pushdown Automata]]** — PDA = FA + stack, padanan automaton untuk CFG. Terima bahasa via Empty Stack atau Final State. Instantaneous Description (ID) melacak state-input-stack langkah demi langkah.

**[[W11 - Top-Down Parsing]]** — Recursive-Descent (backtracking, lambat) vs Predictive (tanpa backtracking, LL(1)). FIRST/FOLLOW SET dipakai bangun Parsing Table M[A,a]. Grammar left-recursive/ambigu/belum-di-left-factor OTOMATIS bukan LL(1).

**[[W14 - Review I]]** — Rangkuman Session 1-13, tidak ada materi baru.

**[[W15 - Bottom-Up Parsing]]** — Handle = substring yang bisa direduksi. SLR dibangun dari LR(0) item + closure + goto → DFA viable prefix, pakai FOLLOW untuk reduce. LR(1) lebih kuat dari SLR (lookahead spesifik per konteks, item `[A→α·β,a]`).

**[[W18 - Syntax Directed Translation]]** — Synthesized attribute (child→parent) vs Inherited attribute (parent/sibling→child). S-Attributed (synthesized saja) paling mudah; L-Attributed (inherited terbatas) bisa dievaluasi 1 depth-first traversal. Eliminasi left recursion JUGA mengubah semantic action.

**[[W20 - Semantic Analyzer]]** — Type expression: basic type, type name, type constructor (array/pointer/product/function). Structural equivalence (bandingkan struktur) vs Name equivalence (bandingkan nama) — beda hasil untuk kasus SAMA. Siklus type expression HARUS diperlakukan sebagai basic type.

**[[W21 - Intermediate Code Generator]]** — DAG hindari hitung ulang common subexpression. Three-Address Code: Quadruple (op,arg1,arg2,result) vs Triple (rujuk lewat posisi). Alamat array disederhanakan jadi `i*width+c` (c dihitung compile-time). Backpatching untuk target jump yang belum diketahui.

**[[W23 - Code Optimization]]** — 3 kriteria: correctness, speedup rata-rata, worth the effort. Peephole (lokal): constant folding, dead code, reduction in strength. Basic block level: CSE, constant/copy propagation. Code Motion pindahkan loop-invariant keluar loop.

**[[W25 - Code Generation]]** — Tahap TERAKHIR compiler, kode optimal itu UNDECIDABLE (pakai heuristik). Target: Absolute/Relocatable Machine Language/Assembly. Register Allocation & Order Evaluation = NP-Complete.

**[[W26 - Review II]]** — Rangkuman Session 15-22, tidak ada materi baru.

## Glosari Lintas Minggu
| Istilah | Penjelasan | Muncul di |
| --- | --- | --- |
| Token, Pattern, Lexeme | Kategori abstrak, deskripsi bentuk, string konkret di source code | W03 |
| DFA / NFA | Deterministic vs Non-deterministic Finite Automaton | W04, W05 |
| ε-closure | Semua state yang dicapai lewat transisi ε | W04, W05 |
| Firstpos/Lastpos/Followpos | Fungsi untuk konversi RE→DFA langsung | W05 |
| Distinguishable states | Dua state yang bisa dibedakan lewat suatu string | W06 |
| CFG (Context-Free Grammar) | G=(V,T,P,S), Type-2 Chomsky | W07 |
| Ambiguous grammar | String dengan >1 parse tree | W07, W09, W15 |
| Left Factoring / Left Recursion | Transformasi grammar wajib untuk top-down parsing | W08, W11, W18 |
| LMD / RMD | Leftmost / Rightmost Derivation | W09, W15 |
| PDA (Pushdown Automaton) | FA + stack, padanan automaton CFG | W10 |
| FIRST / FOLLOW Set | Himpunan terminal awal/pengikut, dasar parsing table | W11, W15 |
| LL(1) / SLR / LR(1) | Tiga varian parser dengan kekuatan berbeda | W11, W15 |
| Handle | Substring yang bisa direduksi jadi non-terminal | W15 |
| Synthesized / Inherited attribute | Dua arah aliran atribut di SDT | W18 |
| Structural / Name equivalence | Dua cara membandingkan kesetaraan tipe | W20 |
| DAG (Directed Acyclic Graph) | Representasi hindari common subexpression | W21, W25 |
| Three-Address Code | Quadruple/Triple, representasi linear kode intermediate | W21, W26 |
| Backpatching | Isi target jump belakangan setelah lokasi diketahui | W21 |
| Peephole Optimization | Optimisasi lokal potongan pendek instruksi | W23 |
| NP-Complete | Kelas masalah sulit dipecahkan efisien | W25 |

## Yang Perlu Dicek Sendiri (Slide Kurang Detail)
- **[[W01 - Introduction to Compiler]]**: slide "Cousins of the Compiler" dan "Compiler construction tools" tidak punya detail teks/gambar terekstrak.
- **[[W02 - Automata dan Language Theory]]**: slide 10-19 (Language Operation, Closure, RE Characteristic) gagal diekstrak — notasi Union/Concatenation/Closure PERSIS perlu dicek manual di slide asli.
- **[[W06 - DFA Minimization]]**: SEMUA diagram transisi (M1/M2/M3, DFA 8-state) berupa shape manual PowerPoint, tidak terekstrak sebagai gambar — perlu dibuka manual.
- **[[W20 - Semantic Analyzer]]**: file ini secara internal SALAH LABEL "Session 18-19" (bentrok dengan [[W18 - Syntax Directed Translation]]) — kemungkinan typo template dosen.
- **[[W21 - Intermediate Code Generator]]**: detail struktur "Indirect Triple" tidak dijelaskan lengkap di slide, hanya disebut di outline.
- **[[W23 - Code Optimization]]**: studi kasus Quick Sort (slide 15-31) SANGAT bergantung diagram visual berurutan — wajib dibuka manual/berurutan untuk memahami alur optimisasinya.
- **[[W25 - Code Generation]]**: algoritma heuristik KONKRET untuk Register Allocation (misalnya graph coloring) tidak dijelaskan — slide hanya menyebut masalahnya NP-Complete.
- **Duplikat file bermasalah**: total 26 file sumber, TUJUH di antaranya duplikat (6 terdeteksi via MD5, 1 lagi — "Session 23" vs "Session 24" — cuma terdeteksi lewat perbandingan MANUAL isi teks karena metadata file-nya berbeda meski konten identik). Kalau menemukan file serupa di folder sumber, WAJIB cek isi teksnya, bukan cuma nama file atau hash.
