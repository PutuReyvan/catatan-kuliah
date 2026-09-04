---
matkul: Compilation Techniques
minggu: 26
sks: 3
sumber: Session 26_Review II.pptx
tags: [kuliah/compiler, minggu/w26]
status: draft
diproses: 2026-09-04
---

# W26 — Review II

> [!note] Deck ini adalah SESI REVIEW yang MERANGKUM ULANG materi Session 15-22 (Bottom-Up Parsing sampai Intermediate Code Generator) — TIDAK ADA materi baru. Sama seperti [[W14 - Review I]], isi note ini SENGAJA dibuat ringkas berupa PETA topik plus kumpulan pertanyaan LATIHAN GABUNGAN — untuk detail lengkap tiap topik, buka note minggu aslinya lewat tautan di tabel bawah.

## Ringkasan
> - Review mencakup **4 topik besar**: Bottom-Up Parsing, Syntax-Directed Translation, Intermediate Code Generator, dan Semantic Analyzer (meski topik terakhir DISEBUT di outline tapi TIDAK PUNYA slide detail terpisah di deck ini — kemungkinan sudah cukup terwakili lewat contoh type checking di materi SDT).
> - Learning outcome review ini meminta mahasiswa bisa **MENGANALISIS dan MENYELESAIKAN SOAL** yang melibatkan SLR/LR parsing, syntax-directed translation, type checking, representasi intermediate code, teknik optimisasi kode, dan generasi target code.
> - Materi disusun mengikuti KELANJUTAN pipeline compiler dari Review I: Bottom-Up Parsing → Syntax-Directed Translation → Intermediate Code Generation — persis alur [[W15 - Bottom-Up Parsing]] sampai [[W21 - Intermediate Code Generator]].

## Peta Topik yang Direview
| Topik Review | Note Asli | Poin Kunci yang Diuji |
| --- | --- | --- |
| Handle string & Bottom-Up Parser | [[W15 - Bottom-Up Parsing]] | Shift-reduce, handle, grammar ambigu |
| SLR Parser Construction | [[W15 - Bottom-Up Parsing]] | LR(0) item, closure, goto, DFA, SLR parsing table |
| Eksekusi Parser dengan Parsing Table | [[W15 - Bottom-Up Parsing]] | Trace stack-input-action lengkap |
| Annotated Parse Tree & SDD | [[W18 - Syntax Directed Translation]] | Synthesized attribute, dependency graph |
| Intermediate Code & DAG | [[W21 - Intermediate Code Generator]] | DAG untuk common subexpression |
| Three-Address Code (Quadruple, Triple) | [[W21 - Intermediate Code Generator]] | Representasi op-arg1-arg2-result |
| Three-Address Statements lengkap | [[W21 - Intermediate Code Generator]] | Binary/unary/move/jump/procedure call |
| Alamat Array (1D, 2D, Multi-D) | [[W21 - Intermediate Code Generator]] | Formula i*width+c, row-major |
| Translation Scheme untuk Array | [[W21 - Intermediate Code Generator]] | Grammar synthesized-only untuk array |

## Quiz Pemahaman Gabungan
Level Bloom C4 ke atas — soal-soal ini SENGAJA menghubungkan BEBERAPA topik sekaligus, meniru semangat "Review Session" yang menguji INTEGRASI, bukan topik individual.

1. **(C4 – Analisis)** Analisis ALUR LENGKAP dari SLR Parsing sampai jadi Three-Address Code: input token → (SLR parsing table, dari [[W15 - Bottom-Up Parsing]]) → reduce actions membangun parse tree → (Syntax-Directed Translation, dari [[W18 - Syntax Directed Translation]]) → annotated parse tree dengan atribut `place` dan `code` → (dari [[W21 - Intermediate Code Generator]]) → Three-Address Code final. Jelaskan di TITIK MANA proses REDUCE di bottom-up parsing menjadi PEMICU untuk mengeksekusi SEMANTIC ACTION yang menghasilkan kode.
2. **(C4 – Analisis)** Bandingkan PERAN "reduce" di Bottom-Up Parsing (dari [[W15 - Bottom-Up Parsing]]) dengan PERAN "synthesized attribute evaluation" di Syntax-Directed Translation (dari [[W18 - Syntax Directed Translation]]). Analisis: kenapa KEDUANYA sama-sama bergerak dari BAWAH (leaves/child) ke ATAS (root/parent) — apa HUBUNGAN STRUKTURAL antara "kapan sebuah reduce terjadi" dan "kapan sebuah synthesized attribute bisa dihitung"?
3. **(C5 – Evaluasi)** Evaluasi kenapa DAG (dari [[W21 - Intermediate Code Generator]]) dan Common Subexpression Elimination (dari [[W23 - Code Optimization]], meski belum direview di deck ini) pada dasarnya menyelesaikan MASALAH YANG SAMA — hindari menghitung ULANG sub-ekspresi yang IDENTIK. Bandingkan KAPAN masing-masing teknik diterapkan dalam pipeline compiler (DAG saat INTERMEDIATE CODE GENERATION vs CSE saat OPTIMIZATION) — apakah menerapkan DAG lebih awal membuat CSE JADI TIDAK PERLU LAGI, atau keduanya tetap saling melengkapi?
4. **(C5 – Evaluasi)** Bandingkan representasi Quadruple dan Triple (dari [[W21 - Intermediate Code Generator]]) dengan kebutuhan translation scheme untuk ARRAY (grammar synthesized-only, `L→id|Elist]`). Evaluasi: kenapa desainer compiler SENGAJA memilih grammar array yang HANYA butuh synthesized attribute (menghindari inherited) — hubungkan dengan kemudahan implementasi translation scheme yang sudah dipelajari di [[W18 - Syntax Directed Translation]] (S-Attributed lebih mudah dari L-Attributed).
5. **(C6 – Cipta)** Rancang SATU rangkaian kerja lengkap (end-to-end) untuk statement assignment array `A[i] := B[j] + 1` (A dan B array integer 1-dimensi, width=4, low=0). Tunjukkan: (a) bagaimana SLR parser akan mem-parse statement ini (urutan shift/reduce secara garis besar, tidak perlu tabel lengkap), (b) translation scheme yang menghasilkan three-address code untuk alamat `A[i]` dan `B[j]` (mengikuti pola grammar array dari [[W21 - Intermediate Code Generator]]), (c) three-address code LENGKAP hasil akhirnya.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W25 - Code Generation]]
- [[W15 - Bottom-Up Parsing]]
- [[W18 - Syntax Directed Translation]]
- [[W21 - Intermediate Code Generator]]
- [[Compiler - Review dan Glosari]]
