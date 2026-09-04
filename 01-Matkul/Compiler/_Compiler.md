---
matkul: Compilation Techniques
tags: [kuliah/compiler, moc]
status: draft
diproses: 2026-09-04
---

# Compilation Techniques — Index

## Daftar Pertemuan
| Minggu | Judul | Ringkasan Singkat |
| --- | --- | --- |
| [[W01 - Introduction to Compiler]] | Introduction to Compiler | Model Analysis-Synthesis, 6 tahap kompilasi, cousins of compiler |
| [[W02 - Automata dan Language Theory]] | Automata and Language Theory | Symbol-Alphabet-String-Language, Regular Definition |
| [[W03 - Lexical Analysis]] | Lexical Analysis (Scanning) | Token/Pattern/Lexeme, input buffering, DFA/NFA pengantar |
| [[W04 - DFA dan NFA]] | DFA & NFA | Definisi formal, subset construction, ε-closure |
| [[W05 - RE ke DFA]] | RE to DFA | Thompson's Construction, firstpos/lastpos/followpos |
| [[W06 - DFA Minimization]] | DFA Minimization | Table-filling, partition method |
| [[W07 - Context-Free Grammar]] | Context-Free Grammar | Chomsky hierarchy, derivation, grammar ambigu |
| [[W08 - Syntax Analysis - Parsing Fundamentals]] | Syntax Analysis: Parsing Fundamental | Parse tree vs AST, left factoring, eliminasi left recursion |
| [[W09 - Syntax Analysis - Parsing Strategies]] | Syntax Analysis: Parsing Strategies | LMD/RMD, top-down vs bottom-up, error recovery |
| [[W10 - Pushdown Automata]] | Pushdown Automata | PDA formal, ID, empty stack vs final state acceptance |
| [[W11 - Top-Down Parsing]] | Top-Down Parsing (Sesi 11-13) | Recursive descent, LL(1), FIRST/FOLLOW, parsing table |
| [[W14 - Review I]] | Review I | Rangkuman Session 1-13 |
| [[W15 - Bottom-Up Parsing]] | Bottom-Up Parsing (Sesi 15-17) | Shift-reduce, SLR, LR(1) |
| [[W18 - Syntax Directed Translation]] | Syntax Directed Translation (Sesi 18-19) | SDD, translation scheme, S/L-attributed |
| [[W20 - Semantic Analyzer]] | Semantic Analyzer / Type Checking | Type expression, structural/name equivalence, overloading |
| [[W21 - Intermediate Code Generator]] | Intermediate Code Generator (Sesi 21-22) | DAG, three-address code, alamat array |
| [[W23 - Code Optimization]] | Code Optimization (Sesi 23-24) | Peephole, basic block level, code motion |
| [[W25 - Code Generation]] | Code Generation | Target program, register allocation, DAG basic block |
| [[W26 - Review II]] | Review II | Rangkuman Session 15-22 |

## Rangkuman Ujian
- [[Compiler - Review dan Glosari]]

## Peta Materi
Matkul ini mengikuti alur **pipeline compiler lengkap**, dari teori dasar automata sampai kode target siap eksekusi — persis mengikuti 6 tahap kompilasi yang diperkenalkan di [[W01 - Introduction to Compiler]]:

1. **Fondasi Teori Bahasa** ([[W01 - Introduction to Compiler]] → [[W02 - Automata dan Language Theory]]) — memahami APA itu compiler dan dasar matematis bahasa formal sebelum masuk ke implementasi.
2. **Lexical Analysis (Tahap 1)** ([[W03 - Lexical Analysis]] → [[W04 - DFA dan NFA]] → [[W05 - RE ke DFA]] → [[W06 - DFA Minimization]]) — dari konsep token sampai bagaimana Regular Expression diubah jadi automaton yang efisien.
3. **Syntax Analysis (Tahap 2)** ([[W07 - Context-Free Grammar]] → [[W08 - Syntax Analysis - Parsing Fundamentals]] → [[W09 - Syntax Analysis - Parsing Strategies]] → [[W10 - Pushdown Automata]] → [[W11 - Top-Down Parsing]] → [[W14 - Review I]] → [[W15 - Bottom-Up Parsing]]) — bagian TERBESAR matkul ini, dua pendekatan besar parsing (top-down LL dan bottom-up LR) beserta fondasi teorinya (CFG, PDA).
4. **Semantic Analysis (Tahap 3)** ([[W18 - Syntax Directed Translation]] → [[W20 - Semantic Analyzer]]) — mengaitkan MAKNA ke struktur sintaks lewat atribut, lalu memverifikasi konsistensi TIPE.
5. **Intermediate Code Generation (Tahap 4)** ([[W21 - Intermediate Code Generator]]) — menerjemahkan program jadi representasi perantara (DAG, three-address code) yang independen mesin.
6. **Code Optimization (Tahap 5)** ([[W23 - Code Optimization]]) — memperbaiki kode intermediate TANPA mengubah maknanya.
7. **Code Generation (Tahap 6)** ([[W25 - Code Generation]] → [[W26 - Review II]]) — menerjemahkan kode intermediate yang sudah dioptimasi jadi kode TARGET (mesin/assembly) siap eksekusi.

**Benang merah lintas minggu yang berulang:**
- **Regular Definition** diperkenalkan di [[W02 - Automata dan Language Theory]], diulang persis sama di [[W03 - Lexical Analysis]] dalam konteks token — pola pengajaran "kenalkan konsep abstrak, lalu terapkan ke kasus konkret" yang berulang di matkul ini.
- **Left Recursion Elimination** (dari [[W08 - Syntax Analysis - Parsing Fundamentals]]) muncul LAGI di [[W18 - Syntax Directed Translation]] — TAPI kali ini semantic action-nya JUGA harus diubah, bukan cuma grammar-nya.
- **DAG** muncul di DUA konteks berbeda: sebagai representasi INTERMEDIATE CODE di [[W21 - Intermediate Code Generator]], dan sebagai representasi BASIC BLOCK untuk register allocation di [[W25 - Code Generation]] — konsep yang sama dipakai untuk tujuan berbeda.
- **Alamat array** (`i*width+c`) diperkenalkan di [[W21 - Intermediate Code Generator]] dan diulang PERSIS SAMA di [[W26 - Review II]] — menunjukkan topik ini dianggap CUKUP PENTING untuk direview dua kali.
- **FIRST/FOLLOW SET**, yang jadi kunci [[W11 - Top-Down Parsing]] (LL parsing table), muncul lagi dalam bentuk berbeda sebagai dasar [[W15 - Bottom-Up Parsing]] (FOLLOW dipakai SLR, lookahead lebih presisi dipakai LR(1)) — konsep yang sama, tapi CARA PAKAI-nya berbeda antara top-down dan bottom-up.

## Konsep Utama
Kandidat note atomik untuk `02-Konsep/` (belum dibuat, dicatat sebagai referensi):
- DFA dan NFA
- Context-Free Grammar
- FIRST Set dan FOLLOW Set
- LL(1) Parser
- SLR/LR Parser
- Three-Address Code
- Syntax-Directed Translation

## Catatan Pemrosesan
- Sumber ASLI **26 file** (Session 1-26), tapi ternyata sangat banyak DUPLIKAT: **4 file identik** untuk deck "Top-Down Parsing" (dilabeli Session 11, 12, 13, dan 13(1) — semuanya 1 file yang sama, membahas Session 11-13 sekaligus), **2 file identik** untuk "Bottom-up Parsing" (Session 15, 16 — membahas Session 15-17), **2 file identik** untuk "Syntax Directed Definition" (Session 18, 19), **2 file identik** untuk "Intermediate Representation" (Session 21, 22) — total 6 file duplikat via MD5 dihapus.
- `[!]` DITEMUKAN duplikat TAMBAHAN yang TIDAK terdeteksi lewat MD5 (karena metadata file berbeda meski isi SAMA PERSIS): file "Session 23_Control Flow & Basic Blocks.pptx" dan "Session 24_Code Optimization.pptx" — dicek MANUAL lewat diff hasil ekstraksi teks, ternyata KONTEN 100% IDENTIK. Salah satu dihapus.
- Setelah dedup MENYELURUH: **19 deck unik** dari 26 file asli, dipetakan ke **19 note** (W01-W11, W14, W15, W18, W20, W21, W23, W25, W26) — nomor W12, W13, W16, W17, W19, W22, W24 TIDAK ADA sebagai file terpisah karena FOLD ke deck gabungan (W11, W15, W18, W21, W23).
- `[!]` File sumber [[W20 - Semantic Analyzer]] secara internal salah label "Session 18-19" (BENTROK dengan judul internal [[W18 - Syntax Directed Translation]] yang topiknya BEDA) — kemungkinan kesalahan copy-paste template dosen. Note tetap dinomori W20 mengikuti nama file dan urutan logis kurikulum.
- `[!]` [[W14 - Review I]] dan [[W26 - Review II]] adalah SESI REVIEW murni tanpa materi baru — note-nya SENGAJA dibuat RINGKAS (peta topik + quiz gabungan lintas-minggu) untuk menghindari duplikasi konten yang sudah lengkap di note minggu aslinya.
- `[!]` Banyak diagram di deck ini format **WMF** — SEMUA 49 file WMF berhasil DIKONVERSI ke PNG memakai PIL/Pillow (via Windows GDI), konsisten dengan solusi yang dipakai di matkul [[_OS|Operating Systems]].
- Ambang batas ekstraksi gambar 3KB diterapkan dari awal — total **80 gambar** berhasil diekstrak ke `99-Assets/Compiler/`.
- `[!]` Beberapa slide di [[W01 - Introduction to Compiler]], [[W02 - Automata dan Language Theory]], dan [[W06 - DFA Minimization]] berisi DIAGRAM/tabel yang dibuat manual dengan shape/text-box PowerPoint (BUKAN gambar raster) — TIDAK BISA diekstrak sebagai gambar. Isi konseptualnya sudah direkonstruksi ke format teks/tabel di note masing-masing, tapi diagram VISUAL asli perlu dibuka manual.
- `[!]` [[W23 - Code Optimization]] slide 15-31 (studi kasus lengkap optimisasi Quick Sort) SANGAT bergantung pada rangkaian diagram visual (flow graph + 12 langkah Common Subexpression Elimination) — SEMUA gambar sudah diekstrak dan disertakan berurutan di note, tapi memang perlu dibuka satu-satu untuk mengikuti alurnya.
- Tidak ada masalah antivirus pada ekstraksi teks matkul ini.
- SKS ditulis 3 sebagai asumsi administratif, belum dikonfirmasi; nama dosen: **Maulin Nasari, S.T., M.Kom.**; kode matkul: **COMP6062001, COMP6062016, COMP6062049**; sumber utama: Aho/Sethi/Ullman "Compiler: Principles, Techniques, and Tools" (Dragon Book) dan Hopcroft/Motwani/Ullman "Introduction to Automata Theory, Languages, and Computation".
- Semua wikilink dan image embed sudah diverifikasi resolve (dicek dengan script Python).
