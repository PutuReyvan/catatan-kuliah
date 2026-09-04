---
matkul: Compilation Techniques
minggu: 14
sks: 3
sumber: Session 14_Review I.pptx
tags: [kuliah/compiler, minggu/w14]
status: draft
diproses: 2026-09-04
---

# W14 — Review I

> [!note] Deck ini adalah SESI REVIEW yang MERANGKUM ULANG materi Session 1-13 (Introduction to Compiler sampai Top-Down Parsing) — TIDAK ADA materi baru. Isi note ini SENGAJA dibuat ringkas, berupa PETA topik yang di-review PLUS kumpulan pertanyaan LATIHAN GABUNGAN yang menguji integrasi antar topik — untuk detail lengkap tiap topik, buka note minggu aslinya lewat tautan di tabel bawah.

## Ringkasan
> - Review mencakup **7 topik besar**: Introduction to Compiler, Automata & Regular Expression, DFA & NFA, Lexical Analysis, CFG, Syntax Analysis, Top-Down Parsing, dan Pushdown Automata.
> - Learning outcome review ini secara eksplisit meminta mahasiswa bisa **MENGANALISIS dan MENYELESAIKAN SOAL** yang melibatkan Regular Expression, NFA, DFA, DFA minimization, CFG, PDA, dan Top-Down Parsing — bukan cuma mengingat definisi, tapi MENGERJAKAN latihan konkret.
> - Materi disusun mengikuti alur PIPELINE compiler yang sesungguhnya: Automata Theory → Lexical Analysis → CFG → Syntax Analysis (Top-Down) — persis alur [[W01 - Introduction to Compiler]] sampai [[W11 - Top-Down Parsing]].

## Peta Topik yang Direview
| Topik Review | Note Asli | Poin Kunci yang Diuji |
| --- | --- | --- |
| Struktur Compiler & Language Processing System | [[W01 - Introduction to Compiler]] | Model Analysis-Synthesis, 6 tahap kompilasi, cousins of compiler |
| Konsep sentral Automata (Symbol, Alphabet, String, Language) | [[W02 - Automata dan Language Theory]] | Regular definition, notasi RE |
| Lexical Analyzer, Token/Pattern/Lexeme | [[W03 - Lexical Analysis]] | Format token, input buffering |
| NFA dan DFA formal | [[W04 - DFA dan NFA]] | Definisi formal, subset construction, ε-closure |
| Thompson's Construction & RE ke DFA langsung | [[W05 - RE ke DFA]] | Firstpos, lastpos, followpos, algoritma RE→DFA |
| DFA Minimization | [[W06 - DFA Minimization]] | Table-filling, partition method |
| Context-Free Grammar | [[W07 - Context-Free Grammar]] | Chomsky hierarchy, derivation, ambiguous grammar |
| Syntax Analysis Fundamentals | [[W08 - Syntax Analysis - Parsing Fundamentals]] | Parse tree vs AST, left factoring, left recursion |
| Parsing Strategies | [[W09 - Syntax Analysis - Parsing Strategies]] | LMD/RMD, top-down vs bottom-up, error recovery |
| Pushdown Automata | [[W10 - Pushdown Automata]] | Definisi formal PDA, ID, empty stack vs final state |
| Top-Down Parsing | [[W11 - Top-Down Parsing]] | Recursive descent, LL(1), FIRST/FOLLOW, parsing table |

## Quiz Pemahaman Gabungan
Level Bloom C4 ke atas — soal-soal ini SENGAJA menghubungkan BEBERAPA topik sekaligus, meniru semangat "Review Session" yang menguji INTEGRASI, bukan topik individual.

1. **(C4 – Analisis)** Analisis ALUR LENGKAP dari sebuah Regular Expression sampai jadi Predictive Parsing Table: RE → (Thompson's Construction) → NFA-ε → (subset construction) → DFA → (dipakai lexical analyzer untuk hasilkan token) → CFG (dari [[W07 - Context-Free Grammar]]) → (FIRST/FOLLOW) → LL(1) Parsing Table. Jelaskan di TITIK MANA hasil dari satu tahap menjadi INPUT untuk tahap berikutnya, dan kenapa urutan ini TIDAK BISA dibalik.
2. **(C4 – Analisis)** Bandingkan PERAN "state" di DFA (dari [[W04 - DFA dan NFA]]) dengan PERAN "non-terminal" di CFG (dari [[W07 - Context-Free Grammar]]). Keduanya sama-sama merepresentasikan "kondisi/konteks saat ini" dalam proses pengenalan bahasa — analisis kenapa CFG BUTUH stack tambahan (jadi PDA) sementara DFA TIDAK, mengacu pada perbedaan Regular Language vs Context-Free Language.
3. **(C5 – Evaluasi)** Evaluasi kenapa Lexical Analysis (berbasis Regular Expression/DFA, dari [[W03 - Lexical Analysis]] sampai [[W06 - DFA Minimization]]) dan Syntax Analysis (berbasis CFG/PDA, dari [[W07 - Context-Free Grammar]] sampai [[W11 - Top-Down Parsing]]) DIPISAH jadi DUA FASE BERBEDA di compiler, alih-alih digabung jadi SATU fase yang langsung memproses karakter mentah jadi parse tree. Apa keuntungan PEMISAHAN TANGGUNG JAWAB ini (pertimbangkan kompleksitas automaton yang dibutuhkan masing-masing)?
4. **(C5 – Evaluasi)** Bandingkan DFA Minimization (dari [[W06 - DFA Minimization]]) dengan Left Factoring/Eliminasi Left Recursion (dari [[W08 - Syntax Analysis - Parsing Fundamentals]]). Evaluasi: KEDUANYA adalah teknik "TRANSFORMASI" sebelum implementasi final — tapi DFA Minimization bertujuan MENGURANGI JUMLAH STATE (efisiensi RUANG), sementara Left Factoring/Left Recursion Elimination bertujuan membuat grammar BISA DIPARSING top-down SAMA SEKALI (bukan cuma efisiensi). Jelaskan perbedaan URGENSI kedua transformasi ini — mana yang OPSIONAL (nice-to-have) dan mana yang WAJIB (tanpa itu, parsing GAGAL TOTAL)?
5. **(C6 – Cipta)** Rancang SATU rangkaian kerja lengkap (end-to-end) untuk bahasa pemrograman FIKTIF sederhana yang HANYA punya statement assignment (`id = expr;`) dengan expr berupa `id`, angka, atau `expr + expr`. Tunjukkan: (a) Regular Expression untuk token `id` dan `number`, (b) CFG untuk struktur `id = expr;`, (c) apakah CFG itu perlu di-left-factor atau eliminasi left recursion sebelum dipakai predictive parsing, dan (d) SATU baris kode fiktif yang valid dalam bahasa ini beserta leftmost derivation-nya.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W11 - Top-Down Parsing]]
- [[W15 - Bottom-Up Parsing]]
- [[Compiler - Review dan Glosari]]
