---
matkul: Compilation Techniques
minggu: 18
sks: 3
sumber: Session 18-19_Syntax Directed Definition.pptx
tags: [kuliah/compiler, minggu/w18]
status: draft
diproses: 2026-09-04
---

# W18 — Syntax Directed Translation

> [!note] Deck ini secara eksplisit menggabungkan **dua sesi (Session 18 dan 19)**: Syntax Directed Definition (SDD) dan Translation Schemes. Note ini tetap dinomori W18 mengikuti konvensi vault untuk deck gabungan multi-sesi.

## Ringkasan
> - **Syntax-Directed Translation** = simbol grammar diberi **atribut** untuk membawa informasi (nilai, tipe, lokasi memori, dll). Nilai atribut dihitung lewat **semantic rule** yang terkait tiap production — bisa menghasilkan kode intermediate, mengisi symbol table, cek tipe, atau laporkan error.
> - Dua NOTASI menuliskan aturan semantik: **Syntax-Directed Definitions (SDD)** — spesifikasi level tinggi, TIDAK bicara soal urutan eksekusi — dan **Translation Schemes** — MENUNJUKKAN urutan eksekusi lewat aksi semantik `{...}` di tengah RHS production.
> - Atribut dibagi 2: **Synthesized** (dihitung dari CHILD ke PARENT, bottom-up) dan **Inherited** (dihitung dari PARENT/SIBLING ke CHILD, top-down).
> - **S-Attributed Definition** (hanya synthesized) mudah diimplementasi bottom-up. **L-Attributed Definition** (synthesized + inherited TERBATAS — inherited attribute Xj cuma boleh bergantung ke X1...Xj-1 dan inherited attribute A) bisa dievaluasi lewat SATU depth-first traversal, cocok untuk top-down MAUPUN sebagian bottom-up.
> - Saat ELIMINASI LEFT RECURSION (dari [[W08 - Syntax Analysis - Parsing Fundamentals]]), **semantic action juga harus DIUBAH** — atribut yang tadinya synthesized (dari grammar left-recursive) DIUBAH jadi kombinasi INHERITED (masuk ke non-terminal baru) dan SYNTHESIZED (keluar dari non-terminal baru).

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Synthesized attribute | Atribut yang dihitung dari nilai CHILD node ke PARENT (bottom-up) |
| Inherited attribute | Atribut yang dihitung dari PARENT/SIBLING ke CHILD (top-down) |
| Syntax-Directed Definition (SDD) | Notasi level tinggi, TIDAK spesifikasikan urutan eksekusi semantic rule |
| Translation Scheme | Notasi yang MENUNJUKKAN urutan eksekusi lewat semantic action `{...}` |
| Annotated Parse Tree | Parse tree yang menunjukkan nilai atribut di setiap node |
| S-Attributed / L-Attributed | Dua sub-kelas SDD yang bisa dievaluasi efisien dalam satu pass |

## Isi

### Syntax-Directed Translation: Konsep Dasar
Simbol grammar diasosiasikan dengan ATRIBUT untuk mengaitkan informasi dengan konstruksi bahasa pemrograman yang mereka representasikan. Nilai atribut ini DIEVALUASI lewat **semantic rule** yang terkait dengan production rule.

**Evaluasi semantic rule bisa:** menghasilkan kode INTERMEDIATE, memasukkan informasi ke SYMBOL TABLE, melakukan TYPE CHECKING, mengeluarkan pesan ERROR, atau melakukan aktivitas LAIN — hampir APA SAJA. Sebuah atribut bisa menyimpan HAMPIR APA SAJA: string, angka, lokasi memori, record kompleks.

### Dua Notasi: SDD vs Translation Schemes
Saat mengaitkan semantic rule dengan production, ada DUA notasi:
- **Syntax-Directed Definitions (SDD)** — memberi SPESIFIKASI LEVEL TINGGI untuk translasi. MENYEMBUNYIKAN banyak detail implementasi seperti URUTAN evaluasi semantic action. Kita kaitkan production dengan sekumpulan semantic action, TAPI TIDAK bilang KAPAN akan dievaluasi.
- **Translation Schemes** — MENUNJUKKAN urutan evaluasi semantic action yang terkait production. Dengan kata lain, translation scheme memberi SEDIKIT informasi soal detail implementasi.

### Syntax-Directed Definitions (SDD)
SDD mengaitkan: dengan setiap simbol grammar, sekumpulan ATRIBUT. Himpunan atribut ini dipartisi jadi DUA subset: **synthesized** dan **inherited attribute**. Setiap production rule diasosiasikan dengan sekumpulan **semantic rule**. Semantic rule membentuk KETERGANTUNGAN antar atribut, direpresentasikan lewat **dependency graph** — graph ini menentukan URUTAN EVALUASI semantic rule tersebut. Evaluasi semantic rule mendefinisikan NILAI atribut, tapi juga bisa punya SIDE EFFECT seperti mencetak nilai.

### Annotated Parse Tree
**Annotated parse tree** = parse tree yang menunjukkan NILAI atribut di setiap node. Proses menghitung nilai atribut di node disebut **annotating (atau decorating)** parse tree. URUTAN perhitungan ini bergantung pada DEPENDENCY GRAPH yang dibentuk semantic rule.

### Bentuk Formal SDD
Untuk production `A → α`, semantic rule berbentuk `b = f(c1, c2, ..., cn)`, di mana f adalah fungsi, dan b bisa:
- **b adalah synthesized attribute** dari A, dan c1,...,cn adalah atribut simbol grammar di production `A→α`.
- **ATAU** b adalah **inherited attribute** salah satu simbol grammar di α (sisi KANAN production), dan c1,...,cn adalah atribut simbol grammar di production itu.

### Attribute Grammar
Semantic rule `b = f(c1,...,cn)` menunjukkan bahwa atribut b BERGANTUNG pada atribut c1,...,cn. Semantic rule bisa cuma MENGHITUNG nilai atribut, atau punya SIDE EFFECT seperti mencetak nilai. **Attribute grammar** adalah SDD di mana fungsi di semantic rule TIDAK BOLEH punya side effect (HANYA boleh menghitung nilai atribut).

### Contoh SDD: Kalkulator Sederhana (S-Attributed)
```
Production        Semantic Rules
L -> E return      print(E.val)
E -> E1 + T        E.val = E1.val + T.val
E -> T             E.val = T.val
T -> T1 * F        T.val = T1.val * F.val
T -> F             T.val = F.val
F -> ( E )         F.val = E.val
F -> digit         F.val = digit.lexval
```
Simbol E, T, F diasosiasikan dengan **synthesized attribute `val`**. Token `digit` punya synthesized attribute `lexval` (diasumsikan dihitung lexical analyzer).

**Contoh Kedua (menghasilkan kode intermediate):**
```
Production        Semantic Rules
E -> E1 + T        E.loc=newtemp(), E.code = E1.code||T.code||add E1.loc,T.loc,E.loc
E -> T             E.loc = T.loc, E.code = T.code
T -> T1 * F        T.loc=newtemp(), T.code = T1.code||F.code||mult T1.loc,F.loc,T.loc
T -> F             T.loc = F.loc, T.code = F.code
F -> ( E )         F.loc = E.loc, F.code = E.code
F -> id            F.loc = id.name, F.code = ""
```
E, T, F diasosiasikan dengan synthesized attribute `loc` dan `code`. Token `id` punya synthesized attribute `name`. `||` adalah operator KONKATENASI string.

### Inherited Attributes
**Contoh** (deklarasi variabel dengan tipe):
```
Production      Semantic Rules
D -> T L         L.in = T.type
T -> int         T.type = integer
T -> real        T.type = real
L -> L1 id       L1.inh = L.inh, addtype(id.entry, L.inh)
L -> id          addtype(id.entry, L.inh)
```
Simbol T punya synthesized attribute `type`. Simbol L punya **inherited attribute `in`** — dihitung dari PARENT/SIBLING (dari T.type), bukan dari CHILD-nya sendiri.

> [!info] Analogi
> Bedakan Synthesized dan Inherited Attribute seperti bedakan arah aliran INFORMASI di pohon keluarga. **Synthesized attribute** itu seperti "SIFAT ANAK YANG DIWARISKAN KE ORANG TUA sebagai RANGKUMAN" — misalnya nilai total belanja sebuah keluarga dihitung dari MENJUMLAHKAN belanja tiap anggota (dari BAWAH/child ke ATAS/parent). **Inherited attribute** itu KEBALIKANNYA — seperti "ATURAN RUMAH yang diturunkan dari ORANG TUA ke ANAK", misalnya "jenis mata uang yang dipakai" ditentukan orang tua dan diturunkan ke SEMUA anak (dari ATAS/parent-sibling ke BAWAH/child).

### S-Attributed vs L-Attributed Definitions
Membuat translator untuk SDD SEMBARANG bisa SULIT. Kita ingin mengevaluasi semantic rule SELAMA parsing (single-pass — parsing DAN evaluasi semantic rule dilakukan BERSAMAAN). Dua sub-kelas SDD:

1. **S-Attributed Definitions** — HANYA memakai synthesized attribute. Implementasinya LEBIH MUDAH (bisa dievaluasi dalam single pass selama parsing, terutama cocok untuk BOTTOM-UP parsing).
2. **L-Attributed Definitions** — SELAIN synthesized attribute, juga bisa memakai inherited attribute dengan cara TERBATAS. Bisa SELALU dievaluasi lewat kunjungan **depth-first** parse tree — artinya BISA JUGA dievaluasi selama parsing.

**Definisi formal L-Attributed:** sebuah SDD disebut L-attributed kalau SETIAP inherited attribute Xj (1≤j≤n) di sisi kanan `A → X1X2...Xn` HANYA bergantung pada:
- Atribut simbol X1,...,Xj-1 (di SEBELAH KIRI Xj dalam production), DAN
- Inherited attribute A itu sendiri.

**Setiap S-Attributed Definition JUGA L-Attributed** — pembatasan HANYA berlaku untuk inherited attribute, TIDAK untuk synthesized.

**Contoh SDD yang BUKAN L-attributed:**
```
A -> L M     L.in=l(A.i), M.in=m(L.s), A.s=f(M.s)
A -> Q R     R.in=r(A.in), Q.in=q(R.s), A.s=f(Q.s)
```
Ini BUKAN L-attributed karena `Q.in = q(R.s)` MELANGGAR batasan — `Q.in` harus dihitung SEBELUM masuk ke Q (karena inherited), TAPI nilai `Q.in` bergantung pada `R.s` yang baru TERSEDIA SETELAH kita KEMBALI dari R (R berada di SEBELAH KANAN Q, melanggar aturan "hanya boleh bergantung pada simbol di SEBELAH KIRI").

### Translation Schemes
Dalam SDD, kita TIDAK bicara soal WAKTU evaluasi semantic rule. **Translation scheme** adalah CFG di mana: atribut diasosiasikan ke simbol grammar, dan **semantic action** (diapit kurung kurawal `{...}`) disisipkan di DALAM sisi kanan production.

**Contoh format:** `A → {...} X {...} Y {...}`

**Restriksi desain:** untuk memastikan nilai atribut TERSEDIA saat sebuah semantic action MERUJUKNYA. Restriksi ini (dimotivasi oleh L-attributed definition) memastikan semantic action TIDAK MERUJUK atribut yang BELUM DIHITUNG. **Posisi semantic action di sisi kanan MENENTUKAN KAPAN aksi itu dieksekusi.**

### Translation Scheme untuk S-Attributed Definitions
Kalau SDD kita adalah S-attributed, konstruksi translation scheme SEDERHANA. Setiap semantic rule S-attributed disisipkan sebagai semantic action di **AKHIR** sisi kanan production terkait:
```
SDD:              E -> E1 + T    E.val = E1.val + T.val
Translation Scheme: E -> E1 + T  { E.val = E1.val + T.val }
```

### Contoh Translation Scheme: Infix ke Postfix
```
E -> T R
R -> + T { print("+") } R1
R -> ε
T -> id { print(id.name) }

Contoh: infix "a+b+c" -> postfix "ab+c+"
```
Penelusuran DEPTH-FIRST parse tree (mengeksekusi semantic action sesuai urutan itu) akan menghasilkan representasi POSTFIX dari ekspresi infix.

### Aturan Inherited Attribute dalam Translation Schemes
Kalau translation scheme mengandung atribut synthesized DAN inherited, HARUS diikuti aturan:
1. **Inherited attribute** sebuah simbol di sisi kanan HARUS dihitung dalam semantic action SEBELUM simbol itu.
2. Semantic action TIDAK BOLEH merujuk **synthesized attribute** simbol yang ada di SEBELAH KANAN semantic action itu.
3. Synthesized attribute non-terminal di sisi KIRI hanya bisa dihitung SETELAH SEMUA atribut yang dirujuknya sudah dihitung (biasanya semantic action ini ditaruh di AKHIR sisi kanan production).

**Dengan SDD L-attributed, SELALU MUNGKIN membangun translation scheme yang memenuhi KETIGA syarat ini** (belum tentu mungkin untuk syntax-directed translation UMUM).

### Top-Down Translation
Kita akan melihat implementasi L-attributed definition SELAMA predictive parsing. Alih-alih syntax-directed translation, kita bekerja dengan translation scheme — bagaimana MENGEVALUASI inherited attribute (di L-attributed definition) selama recursive predictive parsing.

**Contoh translation scheme dengan inherited attribute:**
```
D -> T id { addtype(id.entry, T.type), L.in = T.type } L
T -> int  { T.type = integer }
T -> real { T.type = real }
L -> id   { addtype(id.entry, L.in), L1.in = L.in } L1
L -> ε
```

### Eliminasi Left Recursion dari Translation Scheme
Grammar left-recursive (`E → E1+T {E.val=E1.val+T.val}`, dst.) TIDAK COCOK untuk top-down parsing. Saat kita ELIMINASI left recursion (mengikuti teknik [[W08 - Syntax Analysis - Parsing Fundamentals]]), **semantic action JUGA HARUS DIUBAH**.

**Pola umum:** grammar left-recursive dengan synthesized attribute:
```
A -> A1 Y   { A.a = g(A1.a, Y.y) }
A -> X      { A.a = f(X.x) }
```
Setelah eliminasi left recursion, atribut yang tadinya SEMUA synthesized berubah jadi KOMBINASI inherited (masuk non-terminal baru R) dan synthesized (keluar dari R):
```
A -> X { R.in=f(X.x) } R { A.a=R.syn }
R -> Y { R1.in=g(R.in,Y.y) } R1 { R.syn = R1.syn }
R -> ε { R.syn = R.in }
```

**Contoh konkret ekspresi aritmatika (setelah eliminasi left recursion):**
```
E -> T { A.in=T.val } A { E.val=A.syn }
A -> + T { A1.in=A.in+T.val } A1 { A.syn = A1.syn }
A -> - T { A1.in=A.in-T.val } A1 { A.syn = A1.syn }
A -> ε   { A.syn = A.in }
T -> F { B.in=F.val } B { T.val=B.syn }
B -> * F { B1.in=B.in*F.val } B1 { B.syn = B1.syn }
B -> ε   { B.syn = B.in }
F -> ( E ) { F.val = E.val }
F -> digit { F.val = digit.lexval }
```

**Contoh untuk generasi intermediate code:**
```
E -> T { A.in=T.loc } A { E.loc=A.loc }
A -> + T { A1.in=newtemp(); emit(add,A.in,T.loc,A1.in) } A1 { A.loc = A1.loc }
A -> ε   { A.loc = A.in }
T -> F { B.in=F.loc } B { T.loc=B.loc }
B -> * F { B1.in=newtemp(); emit(mult,B.in,F.loc,B1.in) } B1 { B.loc = B1.loc }
B -> ε   { B.loc = B.in }
F -> ( E ) { F.loc = E.loc }
F -> id    { F.loc = id.name }
```

> [!info] Analogi
> Eliminasi left recursion pada translation scheme itu seperti mengubah cara mencatat SALDO berjalan (running total) dari "menghitung total di AKHIR (setelah semua transaksi selesai dijumlahkan secara rekursif dari terakhir ke pertama)" menjadi "membawa saldo SEMENTARA maju ke transaksi berikutnya" (inherited attribute yang MENGALIR MAJU, bukan mundur). Angka `A.in` yang MASUK ke non-terminal baru itu seperti "SALDO SEMENTARA yang kamu bawa" saat memproses transaksi berikutnya, dan `A.syn` yang KELUAR itu "SALDO AKHIR" setelah SEMUA transaksi (tersisa) diproses.

### Bottom-up Evaluation of Inherited Attributes
Memakai **top-down translation scheme**, kita bisa implementasikan L-attributed definition APA PUN yang berbasis grammar LL(1). Memakai **bottom-up translation scheme**, kita JUGA bisa implementasikan L-attributed definition berbasis LL(1) (karena SETIAP grammar LL(1) JUGA grammar LR(1)). SELAIN itu, kita bisa implementasikan SEBAGIAN (TIDAK SEMUA) L-attributed definition berbasis grammar LR(1) memakai bottom-up translation scheme.

## Diagram & Visual
- **Slide 11 — Parsing Tree untuk input "5+3*4"**
  ![[99-Assets/Compiler/W18-slide11.png]]
- **Slide 12 — Dependency Graph untuk input "5+3*4"**
  ![[99-Assets/Compiler/W18-slide12.png]]
- **Slide 13 — Annotated Parse Tree untuk "5+3*4"**
  ![[99-Assets/Compiler/W18-slide13.png]]
  ![[99-Assets/Compiler/W18-slide13a.png]]
- **Slide 16 — Dependency Graph Inherited Attribute untuk "real id1, id2, id3"**
  ![[99-Assets/Compiler/W18-slide16.png]]
- **Slide 24 — Diagram infix ke postfix translation scheme**
  ![[99-Assets/Compiler/W18-slide24.png]]
- **Slide 25 — Hasil depth-first traversal untuk translation scheme infix-postfix**
  ![[99-Assets/Compiler/W18-slide25.png]]
- **Slide 30-31 — Diagram eliminasi left recursion dengan inherited/synthesized attribute**
  ![[99-Assets/Compiler/W18-slide30.png]]
  ![[99-Assets/Compiler/W18-slide31.png]]

## Rumus / Sintaks
```
Bentuk umum semantic rule SDD:
b = f(c1, c2, ..., cn)
  b synthesized dari A         (bergantung pada child A->alpha)
  ATAU
  b inherited dari simbol di alpha   (bergantung pada context)

L-Attributed constraint:
inherited(Xj) hanya boleh bergantung pada:
  - atribut X1...Xj-1 (simbol di KIRI Xj)
  - inherited attribute A

Eliminasi Left Recursion + Translation Scheme:
A -> A1 Y { A.a = g(A1.a, Y.y) }
A -> X    { A.a = f(X.x) }
=>
A -> X { R.in=f(X.x) } R { A.a=R.syn }
R -> Y { R1.in=g(R.in,Y.y) } R1 { R.syn=R1.syn }
R -> ε { R.syn = R.in }
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Dependency graph** | Graph yang menunjukkan ketergantungan antar atribut, menentukan urutan evaluasi |
| **newtemp()** | Fungsi menghasilkan nama variabel sementara baru untuk kode intermediate |
| **emit()** | Fungsi mengeluarkan (menulis) satu baris instruksi kode intermediate |
| **Side effect (semantic rule)** | Efek sampingan semantic rule selain menghitung nilai atribut, misalnya mencetak |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis contoh SDD yang "BUKAN L-attributed" (`A→LM`, `A→QR` dengan `Q.in=q(R.s)`). Jelaskan SECARA PERSIS kenapa urutan evaluasi ini MUSTAHIL — apa yang HARUS terjadi DULU (masuk ke Q butuh Q.in) vs apa yang BARU TERSEDIA BELAKANGAN (R.s baru ada setelah keluar dari R)?
2. **(C4 – Analisis)** Bandingkan S-Attributed Definition dan L-Attributed Definition dari sisi KEMUDAHAN implementasi. Analisis: kenapa S-Attributed "sedikit LEBIH MUDAH" diimplementasikan dibanding L-Attributed, PADAHAL L-Attributed adalah SUPERSET dari S-Attributed (setiap S-attributed JUGA L-attributed)?
3. **(C5 – Evaluasi)** Evaluasi transformasi eliminasi left recursion pada translation scheme ekspresi aritmatika (dari synthesized attribute `E.val=E1.val+T.val` jadi kombinasi inherited/synthesized dengan non-terminal baru A). Buktikan bahwa hasil transformasi ini MENGHASILKAN NILAI YANG SAMA dengan cara menelusuri perhitungan `E.val` untuk input `"5+3"` di KEDUA versi grammar (sebelum dan sesudah eliminasi) — tunjukkan nilainya SAMA meski cara perhitungannya berbeda.
4. **(C5 – Evaluasi)** Bandingkan translation scheme untuk konversi infix-ke-postfix (`E→TR`, `R→+T{print("+")}R1|ε`) dengan translation scheme untuk generasi kode intermediate (`E→T{A.in=T.loc}A{E.loc=A.loc}`, dst.). Evaluasi: KEDUANYA sama-sama "menerjemahkan" ekspresi aritmatika, tapi APA PERBEDAAN MENDASAR tujuan akhir masing-masing (satu menghasilkan STRING postfix, satu menghasilkan INSTRUKSI kode) — dan kenapa yang kedua butuh fungsi `newtemp()` dan `emit()` sementara yang pertama tidak?
5. **(C6 – Cipta)** Kerjakan latihan dari slide 34: menggunakan grammar hasil eliminasi left recursion (`E→T{A.in=T.val}A{E.val=A.syn}`, dst. — lihat bagian "Eliminasi Left Recursion" di atas), buat **Annotated Parse Tree LENGKAP** untuk statement `(5 + 3) * 4`. Tunjukkan nilai SETIAP atribut (val, in, syn) di SETIAP node parse tree, dan pastikan hasil akhirnya `E.val = 32`.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W15 - Bottom-Up Parsing]]
- [[W20 - Semantic Analyzer]]
- [[Compiler - Review dan Glosari]]
