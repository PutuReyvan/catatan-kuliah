---
matkul: Compilation Techniques
minggu: 15
sks: 3
sumber: Session 15-16-17_Bottom-up Parsing.pptx
tags: [kuliah/compiler, minggu/w15]
status: draft
diproses: 2026-09-04
---

# W15 — Bottom-Up Parsing

> [!note] Deck ini secara eksplisit menggabungkan **tiga sesi (Session 15, 16, dan 17)** dalam satu file: Shift-Reduce Parsing, SLR (Simple LR) Parsing, dan LR(1) Parsing. Note ini tetap dinomori W15 mengikuti konvensi vault untuk deck gabungan multi-sesi.

## Ringkasan
> - **Bottom-Up Parser** (juga disebut **shift-reduce parser**) membangun parse tree dari LEAVES ke ROOT — MEREDUKSI string w jadi start symbol S, KEBALIKAN dari rightmost derivation.
> - **Handle** = substring yang cocok dengan RHS sebuah production, yang REDUKSINYA jadi non-terminal LHS adalah SATU LANGKAH sah dalam kebalikan rightmost derivation. Grammar UNAMBIGUOUS punya TEPAT SATU handle per right sentential form.
> - **SLR (Simple LR) Parser** dibangun dari **LR(0) item** (production dengan TITIK di posisi tertentu di RHS), lewat operasi **closure** dan **goto** membentuk DFA yang mengenali **viable prefix** — lalu FOLLOW SET dipakai menentukan aksi REDUCE.
> - **Konflik Shift-Reduce** dan **Reduce-Reduce** bisa terjadi — SLR "tidak cukup kuat mengingat konteks kiri" untuk beberapa grammar (grammar itu disebut TIDAK SLR meski TIDAK ambigu).
> - **LR(1) Parser** memperbaiki kelemahan SLR dengan menambahkan **lookahead symbol** LANGSUNG ke item (`[A → α·β, a]`) — lebih KUAT dari SLR tapi tabelnya jauh LEBIH BESAR.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Shift-Reduce Parser | Bottom-up parser yang membangun tree dari leaves ke root lewat shift dan reduce |
| Handle | Substring RHS yang bisa direduksi jadi LHS, satu langkah kebalikan rightmost derivation |
| LR(0) item | Production dengan titik penanda posisi di RHS |
| Viable prefix | Prefix sentential form yang tidak melewati handle paling kanan, selalu di puncak stack |
| Closure(I) / Goto(I,X) | Dua operasi membangun DFA item untuk SLR/LR parsing |
| Shift-Reduce Conflict | Kondisi parser tidak bisa putuskan shift atau reduce |

## Isi

### Pengantar Bottom-Up Parser
**Bottom-Up Parser** juga disebut **shift-reduce parser**. Membangun parse tree untuk input string dimulai dari LEAVES, bekerja NAIK ke ROOT. **Mereduksi string w jadi start symbol S.** Di SETIAP langkah reduksi, sebuah substring TERTENTU yang cocok dengan RHS production DIGANTIKAN oleh simbol di LHS.

**Contoh:** grammar `S → aABe`, `A → Abc | b`, `B → d`, proses `w = abbcde`:
```
Langkah reduksi:  abbcde -> aAbcde -> aAde -> aABe -> S
Rightmost derivation (kebalikannya): S -> aABe -> aAde -> aAbcde -> abbcde
```

### Handle Sebuah String
**Handle** sebuah string adalah SUBSTRING yang cocok dengan sisi KANAN sebuah production, dan REDUKSINYA jadi non-terminal di LHS mewakili SATU LANGKAH sepanjang KEBALIKAN dari rightmost derivation.

**Definisi formal:** handle dari right sentential form γ adalah sebuah production `A → β` dan sebuah POSISI di γ di mana string β BISA DITEMUKAN dan DIGANTI oleh A untuk menghasilkan right sentential form SEBELUMNYA dalam rightmost derivation dari γ.

**Contoh:** dari contoh sebelumnya, `abbcde` adalah right sentential form dengan handle `A → b`, dan `aAbcde` punya handle `A → Abc`, dst. **Kalau grammar UNAMBIGUOUS, HANYA ADA SATU handle untuk setiap right sentential form.**

**Kasus grammar ambigu:** grammar `E→E+E|E*E|(E)|id` punya DUA rightmost derivation berbeda untuk string `id+id*id` — artinya beberapa right sentential form punya **LEBIH DARI SATU handle** (contoh: `E→id` dan `E→E+E` sama-sama jadi handle valid dari `E+E*id`).

> [!info] Analogi
> Handle itu seperti "LANGKAH BENAR berikutnya" saat membongkar susunan LEGO yang sudah jadi, kembali ke potongan-potongan dasarnya. Kalau instruksi rakit LEGO-nya JELAS (grammar unambiguous), SELALU ada SATU cara benar membongkar langkah demi langkah kembali ke awal. Tapi kalau instruksinya AMBIGU (bisa dirakit dengan DUA urutan berbeda menghasilkan bentuk yang sama), saat kamu MEMBONGKARNYA, ada LEBIH DARI SATU "langkah pertama yang valid" — kamu bingung mau mulai bongkar dari bagian MANA.

### LR(k) Parser: Konsep Dasar
**LR(k)** — scan dari KIRI ke KANAN, dengan **rightmost derivation**, memakai **k simbol lookahead**.

### Membangun SLR (Simple LR) Parser: LR(0) Item

**Viable prefix** — prefix dari sebuah right sentential form yang TIDAK MELEWATI handle PALING KANAN dari sentential form itu. SELALU muncul di PUNCAK stack shift-reduce parser.

**LR(0) item dari grammar G** — sebuah production G dengan TITIK di suatu posisi di RHS.

**Contoh:** production `A → XYZ` menghasilkan EMPAT item:
```
A -> ·XYZ
A -> X·YZ
A -> XY·Z
A -> XYZ·
```

### Tiga Operasi Konstruksi SLR
1. **Augmenting a grammar** — tambahkan `S' → S` untuk menandai parser KAPAN harus berhenti dan menerima.
2. **Closure operation** — misal I adalah himpunan item G, `closure(I)` adalah himpunan item yang dibentuk dari I lewat DUA aturan:
   - Setiap item di I ditambahkan ke closure(I).
   - Kalau `A → α·Bβ ∈ closure(I)` dan `B → γ` adalah production, tambahkan item `B → ·γ` ke closure(I) (kalau BELUM ADA). Terapkan aturan ini SAMPAI tidak ada item baru yang bisa ditambahkan.
3. **GOTO operation** — misal I = himpunan item dan X = simbol grammar, maka `goto(I,X)` = closure dari HIMPUNAN SEMUA item `[A → αX·β]` sedemikian rupa sehingga `[A → α·Xβ] ∈ I`.

**Contoh closure:** grammar `E'→E`, `E→E+T|T`, `T→T*F|F`, `F→(E)|id` — mulai dengan `I = {E'→·E}`:
```
closure(I):
E' -> ·E
E  -> ·E + T
E  -> ·T
T  -> ·T * F
T  -> ·F
F  -> ·(E)
F  -> ·id
```
**Kernel items** (titik TIDAK di ujung kiri) vs **non-kernel items** (titik di ujung kiri).

**Contoh goto:** `I = {E'→E·, E→E·+T}`, `goto(I, +)`:
```
E -> E + ·T
T -> ·T * F
T -> ·F
F -> ·(E)
F -> ·id
```

### Membangun SLR Parsing Table
1. Bangun DFA dari grammar yang diberikan.
2. Cari FOLLOW(A) untuk SEMUA non-terminal.
3. Tentukan aksi parsing untuk setiap Ii:
   - Kalau `[A → α·aβ] ∈ Ii` dan `goto(Ii, a) = Ij`, set `action[i,a] = "shift j"` (Sj). (a harus terminal.)
   - Kalau `[A → α·] ∈ Ii`, set `action[i,a] = "reduce A→α"` untuk SEMUA a di FOLLOW(A), KECUALI A = S'.
   - Kalau `[S' → S·] ∈ Ii`, set `action[i,$] = accept`.
4. Untuk semua non-terminal A: kalau `goto(Ii, A) = Ij`, set `goto[i,A] = j`.
5. Semua entri LAIN diisi "error".
6. State AWAL parser adalah state yang dibangun dari himpunan item yang berisi `[S' → S·]`.

**Contoh SLR Parsing Table lengkap** (grammar `E→E+T|T`, `T→T*F|F`, `F→(E)|id`; FOLLOW(E)={+,$,)}, FOLLOW(T)={*,+,$,)}, FOLLOW(F)={*,+,$,)}):
| State | id | + | * | ( | ) | $ | Goto E | Goto T | Goto F |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | S5 | | | S4 | | | 1 | 2 | 3 |
| 1 | | S6 | | | | Accept | | | |
| 2 | | r2 | S7 | | r2 | r2 | | | |
| 3 | | r4 | r4 | | r4 | r4 | | | |
| 4 | S5 | | | S4 | | | 8 | 2 | 3 |
| 5 | | r6 | r6 | | r6 | r6 | | | |
| 6 | S5 | | | S4 | | | | 9 | 3 |
| 7 | S5 | | | S4 | | | | | 10 |
| 8 | | S6 | | | S11 | | | | |
| 9 | | r1 | S7 | | r1 | r1 | | | |
| 10 | | r3 | r3 | | r3 | r3 | | | |
| 11 | | r5 | r5 | | r5 | r5 | | | |

### Eksekusi Parser dengan Parsing Table
**Konfigurasi:** `(S0X0S1X1...XmSm, aiai+1...an$)` = (isi stack, sisa input belum dibaca).

**Hasil konfigurasi setelah `action[Sm, ai]`:**
- **= Sj (shift, goto state j):** `(S0X0S1X1...XmSmaiS, ai+1...an$)`.
- **= rp (reduce A→β):** `(S0X0S1X1...Xm-rSm-rAS, aiai+1...an$)` di mana `S = goto[Sm-r, A]` dan `r` = panjang β.
- **accept:** parsing SELESAI.
- **error:** butuh error recovery.

**Contoh trace lengkap** untuk input `id * id + id`:
| Step | Stack | Input | Action |
| --- | --- | --- | --- |
| 1 | 0$ | id*id+id$ | shift |
| 2 | 5id0$ | *id+id$ | reduce F→id |
| 3 | 3F0$ | *id+id$ | reduce T→F |
| 4 | 2T0$ | *id+id$ | shift |
| 5 | 7*2T0$ | id+id$ | shift |
| 6 | 5id7*2T0$ | +id$ | reduce F→id |
| 7 | 10F7*2T0$ | +id$ | reduce T→T*F |
| 8 | 2T0$ | +id$ | reduce E→T |
| 9 | 1E0$ | +id$ | shift |
| 10 | 6+1E0$ | id$ | shift |
| 11 | 5id6+1E0$ | $ | reduce F→id |
| 12 | 3F6+1E0$ | $ | reduce T→F |
| 13 | 9T6+1E0$ | $ | reduce E→E+T |
| 14 | 1E0$ | $ | accept |

### Stack Implementation of Shift-Reduce Parsing
- **Shift** — simbol input BERIKUTNYA digeser ke PUNCAK stack.
- **Reduce** — handle di stack DIGANTIKAN dengan non-terminal terkait (handle SELALU muncul di PUNCAK stack).
- **Accept** — mengumumkan penyelesaian SUKSES.

### Konflik saat Shift-Reduce Parsing
1. **Shift/Reduce conflict** — parser TIDAK BISA memutuskan apakah harus SHIFT atau REDUCE.
2. **Reduce/Reduce conflict** — parser TIDAK BISA memutuskan production MANA yang dipakai untuk reduce.

**Contoh klasik shift-reduce conflict (dangling else):**
```
stmt -> if expr then stmt
      | if expr then stmt else stmt
      | other
```
Ketika stack punya handle "if expr then stmt", parser BINGUNG: SHIFT (menunggu kemungkinan `else` berikutnya) atau REDUCE (anggap statement ini sudah selesai)?

### Grammar yang TIDAK Ambigu tapi TIDAK SLR(1)
Contoh grammar yang **TIDAK ambigu, tapi BUKAN SLR(1)**:
```
S -> L = R
S -> R
L -> * R
L -> id
R -> L
```
Karena `FOLLOW(R) = FOLLOW(S) = FOLLOW(L) = {=}`, muncul konflik di **`Action[2, =]` → Shift ATAU Reduce**. Ini terjadi karena **SLR TIDAK CUKUP KUAT mengingat konteks KIRI (left context)** yang cukup untuk memutuskan aksi berikutnya saat bertemu `=`.

### LR(1) Parser: Mengatasi Kelemahan SLR
**Ide sentral:** di SLR, reduksi `A → α` ditentukan dengan melihat apakah `a` muncul SETELAH α (pakai FOLLOW), sementara di **LR** dilihat apakah `βAa` DIIZINKAN (konteks LEBIH SPESIFIK).

**Item LR(1)** — DEFINISI ULANG item untuk MENYERTAKAN simbol terminal: `[A → α·β, a]`. **Simbol lookahead a TIDAK berpengaruh saat β ≠ ε** (baru relevan setelah β habis, mirip FOLLOW tapi lebih SPESIFIK per konteks).

**Closure(I) untuk LR(1):**
```
Ulangi:
  untuk setiap item [A -> alpha*B*beta, a] di I:
    untuk setiap production B -> gamma di G':
      untuk setiap terminal b di FIRST(beta*a):
        tambahkan [B -> ·gamma, b] ke himpunan I
sampai tidak ada item baru ditambahkan
```

**Goto[I,X] untuk LR(1):**
```
Inisialisasi J = himpunan kosong
untuk setiap item [A -> alpha*X*beta, a] di I:
  tambahkan item [A -> alphaX·beta, a] ke J
  hitung closure(J)
```

**Contoh:** grammar `S'→S`, `S→CC`, `C→cC|d` — dibangun LR(1) Finite State Diagram, lalu LR(1) Parsing Table:
| State | c | d | $ | Goto S | Goto C |
| --- | --- | --- | --- | --- | --- |
| 0 | S3 | S4 | | 1 | 2 |
| 1 | | | Accept | | |
| 2 | S6 | S7 | | | 5 |
| 3 | S3 | S4 | | | 8 |
| 4 | R3 | R3 | | | |
| 5 | | | R1 | | |
| 6 | S6 | S7 | | | 9 |
| 7 | | | R3 | | |
| 8 | R2 | R2 | | | |
| 9 | | | R2 | | |

> [!info] Konteks tambahan (bukan dari slide)
> LR(1) LEBIH KUAT dari SLR karena membawa informasi LOOKAHEAD SPESIFIK per KONTEKS (bukan sekadar FOLLOW SET GLOBAL non-terminal). Konsekuensinya: tabel LR(1) bisa jadi JAUH LEBIH BESAR daripada SLR, karena state yang SAMA secara LR(0) bisa "pecah" jadi BEBERAPA state LR(1) berbeda (tergantung lookahead-nya). Inilah kenapa dalam praktik compiler produksi, dipakai **LALR (Lookahead-LR)** — kompromi yang MENGGABUNGKAN state LR(1) yang punya "core" LR(0) sama, mendapatkan kekuatan MENDEKATI LR(1) tapi dengan ukuran tabel MENDEKATI SLR. LALR inilah yang dipakai tool populer seperti **yacc/bison**.

## Diagram & Visual
> [!warning] Sebagian besar diagram DFA/state di deck ini (slide 8, 13, 21, 24) berformat WMF yang sudah dikonversi ke PNG — tapi beberapa diagram lain (closure/goto tabel, parsing table) dibuat manual dengan text-box PowerPoint dan sudah direkonstruksi ke format tabel di atas.
- **Slide 8 — Konsep LR(k) parsing**
  ![[99-Assets/Compiler/W15-slide08.png]]
- **Slide 13 — DFA construction untuk grammar ekspresi augmented**
  ![[99-Assets/Compiler/W15-slide13.png]]
- **Slide 21 — Ilustrasi Shift/Reduce conflict pada grammar non-SLR**
  ![[99-Assets/Compiler/W15-slide21.png]]
- **Slide 24 — LR(1) Finite State Diagram untuk grammar S'→S, S→CC, C→cC|d**
  ![[99-Assets/Compiler/W15-slide24.png]]

## Rumus / Sintaks
```
LR(0) item dari A -> XYZ:
A -> ·XYZ | A -> X·YZ | A -> XY·Z | A -> XYZ·

Closure(I) [SLR/LR(0)]:
tambahkan semua item I
selama A -> alpha*B*beta di closure(I) dan B->gamma production:
  tambahkan B -> ·gamma

Goto(I, X) = closure({ [A -> alphaX·beta] | [A -> alpha·Xbeta] di I })

SLR Parsing Table Construction:
[A -> alpha*a*beta] di Ii, goto(Ii,a)=Ij  -> action[i,a] = shift j
[A -> alpha·] di Ii, a di FOLLOW(A)        -> action[i,a] = reduce A->alpha
[S' -> S·] di Ii                            -> action[i,$] = accept

LR(1) item: [A -> alpha·beta, a]
Closure(I) [LR(1)] tambah lookahead: b di FIRST(beta*a)
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Augmented grammar** | Grammar asli ditambah production `S' → S` untuk menandai penerimaan |
| **Kernel item** | Item LR dengan titik TIDAK di posisi paling kiri |
| **Non-kernel item** | Item LR dengan titik di posisi PALING KIRI (hasil closure) |
| **SLR (Simple LR)** | Varian LR paling sederhana, pakai FOLLOW SET untuk keputusan reduce |
| **LALR (Lookahead-LR)** | Kompromi antara SLR dan LR(1), dipakai tool seperti yacc/bison |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa "Handle SELALU muncul di PUNCAK stack" dalam shift-reduce parsing. Hubungkan dengan definisi "viable prefix" — kenapa parser HANYA perlu melihat PUNCAK stack (bukan seluruh isi stack) untuk memutuskan kapan melakukan reduce?
2. **(C4 – Analisis)** Bandingkan operasi Closure dan Goto dalam konstruksi DFA untuk SLR. Analisis: Closure "MEMPERLUAS" satu set item dengan menambahkan kemungkinan turunan BARU (item non-kernel), sementara Goto "BERPINDAH" ke set item BARU berdasarkan simbol tertentu. Kenapa KEDUA operasi ini dibutuhkan BERSAMA-SAMA untuk membangun SELURUH DFA (tidak cukup salah satu saja)?
3. **(C5 – Evaluasi)** Evaluasi grammar `S→L=R|R`, `L→*R|id`, `R→L` yang TIDAK ambigu tapi BUKAN SLR(1). Jelaskan AKAR MASALAHNYA: kenapa FOLLOW(R)=FOLLOW(L)={=} membuat SLR tidak bisa membedakan situasi "harus shift =" vs "harus reduce R→L" — dan kenapa LR(1) (dengan lookahead SPESIFIK per konteks) bisa menyelesaikan masalah yang SLR (dengan FOLLOW SET global) tidak bisa?
4. **(C5 – Evaluasi)** Bandingkan Top-Down Parsing (LL, dari [[W11 - Top-Down Parsing]]) dan Bottom-Up Parsing (LR, dari materi ini) dari sisi KELAS GRAMMAR yang bisa ditangani. Evaluasi: kenapa Bottom-Up bisa menangani grammar left-recursive TANPA transformasi (sementara Top-Down TIDAK BISA sama sekali) — hubungkan dengan cara kerja REDUCE yang tidak butuh "memprediksi" production di awal seperti Top-Down.
5. **(C6 – Cipta)** Kerjakan latihan dari slide 26: untuk grammar `S → (L) | a`, `L → L,S | S` — (a) buat augmented grammar dan tentukan kernel/non-kernel item set-nya (mulai dari `S'→·S`), (b) buat transition diagram untuk operasi GOTO (boleh dalam bentuk daftar state dan transisinya kalau diagram visual sulit), (c) bangun tabel SLR lengkap (Action + Goto) mengikuti format tabel di materi ini.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W11 - Top-Down Parsing]]
- [[W14 - Review I]]
- [[W18 - Syntax Directed Translation]]
- [[Compiler - Review dan Glosari]]
