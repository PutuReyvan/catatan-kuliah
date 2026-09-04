---
matkul: Compilation Techniques
minggu: 7
sks: 3
sumber: Session 07_Context-Free Grammar.pptx
tags: [kuliah/compiler, minggu/w07]
status: draft
diproses: 2026-09-04
---

# W07 — Context-Free Grammar

## Ringkasan
> - **Chomsky Hierarchy** membagi grammar jadi 4 tipe (Type-0 sampai Type-3), masing-masing berkorespondensi ke jenis AUTOMATON dan BATASAN production rule berbeda — CFG adalah **Type-2**, dikenali oleh **Pushdown Automaton (PDA)**.
> - **CFG (Context-Free Grammar)** didefinisikan formal sebagai `G = (V, T, P, S)` — himpunan Variabel, Terminal, Production, dan Start symbol.
> - **Derivation** = proses menerapkan production rule berulang kali dari start symbol sampai jadi kalimat lengkap — bisa **Leftmost Derivation (LMD)** atau **Rightmost Derivation (RMD)**.
> - **Grammar Ambigu** = kalau ADA string yang punya LEBIH DARI SATU parse tree — masalah SERIUS untuk bahasa pemrograman, karena artinya kode bisa punya lebih dari satu "arti" yang valid. Beberapa grammar ambigu bisa DIUBAH jadi tidak ambigu, tapi TIDAK ADA algoritma umum untuk melakukannya, dan beberapa bahasa **inherently ambiguous** (SEMUA grammar-nya pasti ambigu).
> - **Semua Regular Language BISA didefinisikan CFG, tapi TIDAK sebaliknya** — CFG lebih EKSPRESIF dari regular expression, dan ada aturan konversi langsung RE → CFG.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Context-Free Grammar (CFG) | Grammar Type-2 Chomsky, dikenali Pushdown Automaton |
| Variable (Non-terminal) | Simbol yang masih bisa DIURAIKAN lebih lanjut lewat production |
| Terminal | Simbol AKHIR yang tidak bisa diuraikan lagi, bagian dari string final |
| Production | Aturan penggantian bentuk A → α |
| Derivation | Proses penerapan production berulang dari start symbol sampai kalimat lengkap |
| Ambiguous grammar | Grammar di mana ada string dengan lebih dari satu parse tree |

## Isi

### Chomsky Classification (Hierarki Chomsky)
| Grammar | Bahasa | Automaton | Aturan Produksi (batasan) |
| --- | --- | --- | --- |
| **Type-0** | Unrestricted Grammar | Turing Machine | α → β (tanpa batasan) |
| **Type-1** | Context-Sensitive Grammar | Linear-Bounded Automaton | α → β, dengan \|α\| ≤ \|β\| |
| **Type-2** | Context-Free Grammar | Pushdown Automaton | A → β, A ∈ V, β ∈ (V∪T)* |
| **Type-3** | Regular Grammar | Finite State Automaton | A → β, A ∈ V, β ∈ VT\|T atau β ∈ TV\|T |

> [!info] Konteks tambahan (bukan dari slide)
> Hierarki Chomsky ini tersusun BERTINGKAT dari yang PALING TERBATAS (Type-3, regular — cuma bisa dikenali finite automaton) sampai yang PALING BEBAS (Type-0, unrestricted — butuh Turing Machine penuh). Semakin TINGGI angka Type-nya (Type-3 ke Type-0), semakin LONGGAR batasannya, dan semakin KUAT/EKSPRESIF bahasanya, tapi juga semakin MAHAL/RUMIT automaton yang dibutuhkan untuk mengenalinya. Compiler bahasa pemrograman umumnya berhenti di **Type-2 (CFG)** untuk struktur SINTAKS-nya — cukup ekspresif untuk struktur bersarang (nested) seperti tanda kurung, tapi masih cukup TERBATAS supaya bisa di-parse EFISIEN.

### Kenapa Context-Free Grammar Penting?
CFG dipakai untuk: mendefinisikan bahasa pemrograman, mendeskripsikan FORMAT DOKUMEN (seperti Document Type Definition/DTD di XML), formalitas konsep PARSING, dan mendefinisikan ekspresi MATEMATIKA. Ada automaton bernama **Pushdown Automaton (PDA)** yang mendeskripsikan SEMUA dan HANYA CFG (dibahas lebih lanjut di [[W10 - Pushdown Automata]]).

### Definisi Formal CFG
```
G = (V, T, P, S)

V : himpunan VARIABEL (non-terminal)
T : himpunan TERMINAL
    V ∩ T = ∅ (disjoint, tidak boleh overlap)
P : himpunan PRODUCTION: A -> α
    A ∈ V (harus variabel)
    α ∈ (V ∪ T)* (bisa campuran variabel dan terminal)
S : START SYMBOL
```

### Grammar Derivation
**Contoh production untuk kalimat sederhana:**
```
1. <sentence>  -> <subject> <predicate>
2. <subject>   -> <noun>
3. <predicate> -> <verb> <object>
4. <object>    -> <noun>
5. <noun>      -> dog | rice | people
6. <verb>      -> eat | hit
```
(`< >` menandai variabel, `|` menandai alternatif/pilihan)

Lewat penerapan berulang production ini ("derivation"), dihasilkan KALIMAT LENGKAP, contoh: **"People eat rice"**, **"Dog eat dog"**. Kalimat yang SALAH (tidak bisa diturunkan dari grammar ini): "Hit the dog", "Eating rice people".

**Derivation** = proses penurunan production dari ATAS ke BAWAH (dari head ke body). Simbol derivasi: `⇒`.
- **Leftmost Derivation (LMD)** — derivasi dilakukan pada VARIABEL PALING KIRI.
- **Rightmost Derivation (RMD)** — derivasi dilakukan pada variabel PALING KANAN.

**Contoh RMD untuk "dog eat dog":**
```
<sentence> ⇒ <subject> <predicate>
           ⇒ <subject> <verb> <object>
           ⇒ <subject> <verb> <noun>
           ⇒ <subject> <verb> dog
           ⇒ <subject> eat dog
           ⇒ <noun> eat dog
           ⇒ dog eat dog
```

### CFG untuk Ekspresi Aritmatika
```
E -> E + E
E -> E * E
E -> (E)
E -> a | b | c | d
```

### Notasi Simbol Standar
1. **A, B, C, D, E, S** — variabel.
2. **Huruf kecil dan digit** — terminal.
3. **X, Y, Z** — terminal ATAU variabel.
4. **u, v, w, x, y, z** (huruf kecil) — string variabel.
5. **α, β, γ** — sentential form (V∪T)*.
6. Notasi singkat: kalau A→α1, A→α2, ..., A→αn, ditulis `A → α1 | α2 | ... | αn`.

### Derivation Tree (Parse Tree)
**Derivation Tree** adalah penggambaran derivasi dalam bentuk POHON. Contoh: `A → XYZ` digambarkan sebagai pohon dengan A di ROOT, dan X, Y, Z sebagai children-nya. Contoh nyata: parse tree untuk `-(id + id)`.

### Language of a Grammar (Bahasa dari Sebuah Grammar)
**Definisi:** Bahasa untuk CFG G, `L(G) = { w | w ∈ T* dan S ⇒* w }`, di mana **w ∈ T\*** berarti w adalah string terminal, dan **S ⇒\* w** berarti w BISA DITURUNKAN dari S. L disebut **"Context-Free Language"** kalau ADA CFG G sedemikian rupa sehingga L = L(G).

**Contoh 1:**
```
G = (V, T, P, S), V={S}, T={a,b}, P={S -> aSb, S -> ab}
String terpendek: S -> ab
Derivasi: S -> aSb -> aaSbb -> ... -> a^n b^n
L(G) = {a^n b^n | n >= 1}
```

**Contoh 2:**
```
G = (V, T, P, S), V={S}, T={a,b}, P={S -> aAa, A -> aAa, A -> a}
String terpendek: S -> aAa -> aba
Derivasi: S -> aAa -> aaAaa -> aaaAaaa -> ... -> a^n b a^n
L(G) = {a^n b a^n | n >= 1}
```

### Konversi RE ke CFG
**Semua Regular Language BISA didefinisikan CFG, tapi TIDAK BERLAKU SEBALIKNYA** (ada CFG yang bukan regular). Aturan konversi:
```
RE = ∅       -> S -> ε   (production S tidak menghasilkan terminal string)
RE = ε       -> S -> ε
RE = a       -> S -> a
RE = a+b     -> S -> a | b
RE = a.b     -> S -> ab
RE = a*      -> S -> aS | ε
RE = a+      -> S -> aS | a
```

**Contoh soal:** buat CFG untuk RE `(ab + ba)* (abb)*` (dipecah jadi bagian A = `(ab+ba)*` dan B = `(abb)*`):
```
Production CFG:
S -> A B
A -> C A | ε
C -> ab | ba
B -> D B | ε
D -> abb

CFG G = {V,T,P,S}
V = {S,A,B,C,D}
T = {a,b}
```

> [!info] Analogi
> Konversi RE ke CFG itu seperti menerjemahkan "instruksi resep singkat" (RE) jadi "diagram alur lengkap" (CFG). RE seperti `a*` cukup bilang "ulangi a sebanyak yang kamu mau, atau tidak sama sekali" — sementara CFG-nya `S -> aS | ε` secara EKSPLISIT menuliskan DUA PILIHAN: "tambah satu 'a' lagi lalu proses ulang (S -> aS)" ATAU "berhenti di sini (S -> ε)". CFG lebih VERBOSE tapi juga lebih FLEKSIBEL — bisa mengekspresikan struktur BERSARANG (nested) yang RE TIDAK BISA, seperti tanda kurung yang harus SEIMBANG `(())`.

### Ambiguous Grammar
**Grammar ambigu:** ada string di T\* yang punya LEBIH DARI SATU LMD atau RMD (atau lebih dari satu parse tree). Kalau SETIAP string di L(G) punya PALING BANYAK SATU parse tree, G disebut **unambiguous**.

**Ada bahasa yang TIDAK BISA ditemukan grammar unambiguous-nya.** Grammar bahasa pemrograman KOMPUTER seharusnya menghasilkan yang UNAMBIGUOUS. **Masalah:** apakah sebuah grammar bisa DIPUTUSKAN ambigu atau tidak? Kalau tidak bisa dipastikan, TIDAK ADA algoritma yang bisa menjamin proses langkah demi langkah untuk menentukan masalah ini — ini adalah masalah **UNDECIDABLE** secara umum.

**Contoh ambiguitas:** parse tree untuk `a+a*a` dari grammar:
```
E -> E + E
E -> E * E
E -> (E)
E -> a | b | c | d
```
Menghasilkan **DUA parse tree berbeda (T1 dan T2)** — satu menganggap `+` dievaluasi lebih dulu (mengelompokkan `a*a` sebagai satu E, lalu `+a` sebagai E), satu lagi menganggap `*` dievaluasi lebih dulu — AMBIGU karena hasilnya bisa BEDA tergantung urutan operasi mana yang "dipercaya" parser.

**Beberapa grammar ambigu (TIDAK SEMUA) bisa DIUBAH jadi tidak ambigu.** Contoh: CFG ekspresi aritmatika di atas bisa ditransformasi jadi UNAMBIGUOUS dengan menambahkan LEVEL PRIORITAS operator:
```
E -> T | E + T
T -> F | T * F
F -> I | (E)
I -> a | b | c
```
**TIDAK ADA algoritma umum** untuk mengonversi grammar ambigu jadi unambiguous.

**Inherently Ambiguous Language** — sebuah Context-Free Language L disebut inherently ambiguous kalau **SEMUA** CFG untuk L pasti ambigu (tidak ada satu pun versi unambiguous-nya). Contoh:
```
S -> AB | C
A -> aAb | ab
B -> cBd | cd
C -> aCd | aDd
D -> bDc | bc
```
Bahasa ini punya DUA cara menurunkan string tertentu (misalnya `aabbccdd`) — satu lewat jalur `AB`, satu lewat jalur `C` — MEMBUKTIKAN bahasa ini secara INHEREN ambigu.

> [!info] Konteks tambahan (bukan dari slide)
> Ambiguitas grammar adalah masalah SERIUS bagi desainer bahasa pemrograman. Kalau grammar bahasa pemrograman ambigu, artinya COMPILER bisa MENAFSIRKAN kode sumber yang SAMA dengan CARA BERBEDA (misalnya urutan operasi berbeda), yang bisa menghasilkan BUG yang sangat sulit dilacak — kode terlihat "benar" tapi hasil eksekusinya tidak konsisten tergantung compiler mana yang dipakai. Inilah kenapa hampir SEMUA bahasa pemrograman modern dirancang dengan grammar yang secara EKSPLISIT menghindari ambiguitas, biasanya lewat aturan PRIORITAS OPERATOR (precedence) dan ASOSIATIVITAS yang jelas.

## Diagram & Visual
- **Slide 4 — Diagram Chomsky Classification (hierarki 4 tipe grammar)**
  ![[99-Assets/Compiler/W07-slide04.png]]
- **Slide 25 — Ilustrasi bahasa inherently ambiguous**
  ![[99-Assets/Compiler/W07-slide25.png]]
- **Slide 26 — Parse tree untuk string 'aabbccdd'**
  ![[99-Assets/Compiler/W07-slide26.png]]

## Rumus / Sintaks
```
Definisi Formal CFG:
G = (V, T, P, S)

Bahasa dari CFG:
L(G) = { w | w ∈ T* dan S =>* w }

Konversi RE ke CFG:
RE = a     -> S -> a
RE = a+b   -> S -> a | b
RE = a.b   -> S -> ab
RE = a*    -> S -> aS | ε
RE = a+    -> S -> aS | a

Contoh CFG anbn:
S -> aSb | ab    =>  L(G) = {a^n b^n | n >= 1}
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Sentential form** | String hasil derivasi parsial, bisa berisi campuran variabel dan terminal |
| **Pushdown Automaton (PDA)** | Automaton dengan stack, mengenali persis bahasa Context-Free |
| **Undecidable problem** | Masalah yang tidak bisa diselesaikan dengan algoritma yang pasti berhenti |
| **Precedence (operator)** | Aturan urutan prioritas operasi, dipakai menghilangkan ambiguitas grammar aritmatika |

## Pertanyaan Terbuka
- Slide 14 tidak punya teks yang bisa diekstrak — kemungkinan diagram tambahan tentang notasi CFG.

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa CFG (Type-2) berada di ANTARA Context-Sensitive (Type-1) dan Regular (Type-3) dalam hierarki Chomsky. Hubungkan dengan batasan production rule masing-masing — bagian mana dari aturan CFG (`A -> β`, A harus VARIABEL TUNGGAL) yang membuatnya LEBIH TERBATAS dari Type-1 tapi LEBIH BEBAS dari Type-3?
2. **(C4 – Analisis)** Bandingkan grammar ambigu `E -> E+E | E*E | (E) | a|b|c|d` dengan versi UNAMBIGUOUS-nya (`E->T|E+T`, `T->F|T*F`, dst.). Analisis: bagaimana STRUKTUR BERTINGKAT (E, T, F, I) di versi kedua secara IMPLISIT menegakkan aturan PRIORITAS operator (`*` sebelum `+`) tanpa perlu aturan eksplisit terpisah?
3. **(C5 – Evaluasi)** Evaluasi klaim "Semua Regular Language bisa didefinisikan CFG, tapi tidak sebaliknya". Berikan CONTOH bahasa yang CFG tapi BUKAN Regular (petunjuk: pikirkan bahasa yang butuh MENGHITUNG/MENCOCOKKAN jumlah simbol, seperti a^n b^n) — jelaskan kenapa Finite Automaton (yang mengenali Regular Language) TIDAK BISA mengenali bahasa seperti ini, sementara Pushdown Automaton BISA.
4. **(C5 – Evaluasi)** Bandingkan konsep "ambiguous grammar" (bisa diperbaiki jadi unambiguous) dengan "inherently ambiguous language" (TIDAK BISA diperbaiki, karena SEMUA grammar untuk bahasa itu pasti ambigu). Evaluasi: kenapa perbedaan ini PENTING dipahami desainer bahasa pemrograman — apa yang harus dilakukan desainer bahasa kalau ternyata FITUR yang mereka inginkan membentuk bahasa yang inherently ambiguous?
5. **(C6 – Cipta)** Kerjakan latihan #1 dari slide 27: buat CFG untuk RE `a* b* (a|c)*`. Tunjukkan production lengkapnya (V, T, P, S) mengikuti aturan konversi RE→CFG yang sudah dipelajari, lalu berikan CONTOH satu string yang valid diturunkan dari grammar itu beserta langkah derivasinya.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W06 - DFA Minimization]]
- [[W08 - Syntax Analysis - Parsing Fundamentals]]
- [[Compiler - Review dan Glosari]]
