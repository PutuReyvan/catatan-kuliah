---
matkul: Compilation Techniques
minggu: 10
sks: 3
sumber: Session 10_Pushdown Automata.pptx
tags: [kuliah/compiler, minggu/w10]
status: draft
diproses: 2026-09-04
---

# W10 — Pushdown Automata

## Ringkasan
> - **PDA (Pushdown Automaton)** adalah "COUNTERPART" (padanan) Finite Automaton untuk CFG — mengenali bahasa YANG PERSIS SAMA dengan Context-Free Grammar. Terdiri dari **finite control, input tape,** dan **STACK** (yang tidak dimiliki FA biasa).
> - Stack punya dua operasi dasar: **PUSH** (masukkan simbol) dan **POP** (keluarkan simbol). Setiap langkah PDA mengecek TIGA hal sekaligus: state saat ini, simbol input, DAN simbol di puncak stack.
> - PDA bisa MENERIMA bahasa dengan DUA cara: **Empty Stack** (stack jadi kosong) atau **Final State** (berakhir di state akhir).
> - **Instantaneous Description (ID)** = notasi triple `(q, w, γ)` yang merepresentasikan STATE saat ini, SISA input yang belum dibaca, dan ISI STACK saat ini — dipakai untuk melacak jalannya eksekusi PDA langkah demi langkah.
> - Contoh KLASIK: PDA untuk bahasa `L = {wcw^R | w ∈ (0+1)*}` (string PALINDROME dengan penanda tengah `c`) — butuh STACK untuk "mengingat" bagian PERTAMA string (w) supaya bisa dicocokkan dengan bagian KEDUA (w reversed) secara terbalik.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| PDA (Pushdown Automaton) | FA + stack, padanan automaton untuk CFG |
| Stack | Struktur data LIFO tempat PDA menyimpan/mengambil simbol |
| Push / Pop | Operasi memasukkan / mengeluarkan simbol dari puncak stack |
| Empty Stack Acceptance | Bahasa diterima kalau stack jadi KOSONG di akhir eksekusi |
| Final State Acceptance | Bahasa diterima kalau eksekusi berakhir di FINAL STATE |
| Instantaneous Description (ID) | Triple (state, sisa input, isi stack) yang menggambarkan status PDA |

## Isi

### Apa Itu Pushdown Automata (PDA)
**PDA adalah "counterpart" Finite Automaton UNTUK CFG** — PDA berperan bagi CFG PERSIS SEPERTI Finite Automaton berperan bagi Regular Grammar (ingat [[W07 - Context-Free Grammar]] — CFG = Type-2 Chomsky, dikenali PDA).

Finite automaton yang terdiri dari:
1. **Finite control** — sama seperti FA biasa.
2. **Input tape** — tempat string input dibaca.
3. **Stack** — KOMPONEN TAMBAHAN yang TIDAK dimiliki FA biasa, inilah yang memberi PDA "daya ingat" tambahan.

**Operasi stack:** **PUSH** (memasukkan simbol) dan **POP** (mengeluarkan simbol). Setiap langkah PDA mengecek: **STATE saat ini, SIMBOL INPUT, dan SIMBOL DI PUNCAK STACK** — ketiganya SEKALIGUS menentukan langkah apa yang diambil.

> [!info] Analogi
> Kalau Finite Automaton (dari [[W04 - DFA dan NFA]]) itu seperti orang yang HANYA bisa mengingat "posisi saat ini" (state) tanpa bisa mencatat apa pun dari masa lalu, PDA itu seperti orang yang SAMA tapi diberi TUMPUKAN KERTAS CATATAN (stack) yang bisa dia tulis (push) atau sobek dari atas (pop). Kemampuan "mencatat dan membaca ulang catatan PALING ATAS" inilah yang membuat PDA bisa mengenali bahasa yang JAUH LEBIH RUMIT daripada FA biasa — misalnya bahasa yang butuh MENCOCOKKAN jumlah pembuka dan penutup (seperti tanda kurung `(())` yang harus seimbang), yang MUSTAHIL dikenali FA biasa tanpa "memori" tambahan ini.

### Contoh Motivasi: Bahasa Palindrome dengan Penanda
PDA bisa mengenali bahasa sebagai CFL: `L = {wcw^R | w ∈ (0+1)*}` — string BINER (w), diikuti penanda `c`, diikuti KEBALIKAN (reverse) dari w.

**CFG yang sesuai untuk bahasa ini:**
```
G = ({S}, {0,1}, P, S), dengan production P:
S -> 0S0 | 1S1 | c
```

**Konstruksi dan mekanisme kerja PDA** yang menerima `L = {wcw^R | w ∈ (0+1)*}`:
- **Finite Control (FC)** punya 2 state: **q1** (untuk membaca bagian input di w), **q2** (untuk membaca bagian input di w^R).
- **Simbol Stack:** Blue Plate (B), Green Plate (G), Red Plate (R).
- **Simbol Input:** 0, c, 1.

**Mekanisme aksi:**
1. Kondisi awal: isi stack = R (Red Plate), start state = q1.
2. Untuk input di w & state saat ini = q1: kalau input = 0 → state tetap q1, stack PUSH B. Kalau input = 1 → state tetap q1, stack PUSH G.
3. Input = c & state = q1 → pindah ke state q2, stack TIDAK BEROPERASI.
4. Untuk input di w^R & state = q2: kalau input=0 DAN puncak stack=B → state tetap q2, stack POP B. Kalau input=1 DAN puncak stack=G → state tetap q2, stack POP G.
5. Setelah input w^R SELESAI, input = ε, puncak stack = R, state = q2 → state tetap q2, stack POP R — stack jadi KOSONG.
6. Di LUAR kondisi di atas, PDA TIDAK BERGERAK (transisi undefined = tolak).

> [!info] Analogi
> Bayangkan PDA ini seperti seseorang yang MEMBACA setengah bagian pertama sebuah kalimat (w) sambil MENUMPUK BATU BERWARNA di tanah sesuai huruf yang dibaca (0→batu biru, 1→batu hijau) — analogi PUSH. Begitu ketemu penanda tengah (c), dia BERHENTI menumpuk dan mulai MENGAMBIL batu dari tumpukan PALING ATAS satu per satu (POP), sambil mencocokkan warnanya dengan huruf yang dia baca di bagian KEDUA kalimat (w reversed). Kalau SEMUA warna cocok DAN tumpukan batu habis TEPAT saat kalimat selesai dibaca, berarti bagian kedua BENAR-BENAR kebalikan (reverse) dari bagian pertama — string diterima!

**Tabel transisi lengkap:**
| Stack Symbol | State | Input 0 | Input 1 | Input c |
| --- | --- | --- | --- | --- |
| BLUE (B) | q1 | Push B, tetap q1 | Push G, tetap q1 | Masuk ke q2 |
| BLUE (B) | q2 | Pop puncak, tetap q1 (tulisan asli: fix q1) | — | — |
| GREEN (G) | q1 | Push B, tetap q1 | Push G, tetap q1 | Masuk ke q2 |
| GREEN (G) | q2 | — | Pop puncak, tetap q2 | — |
| RED (R) | q1 | Push B, tetap q1 | Push G, tetap q1 | Masuk ke q2 |
| RED (R) | q2 | Pop elemen puncak dari stack | | |

### Dua Cara PDA Menerima Bahasa
1. **Accepted by Empty Stack** — stack menjadi KOSONG di akhir eksekusi.
2. **Accepted by Final State** — Finite Automaton berakhir di FINAL STATE.

### Definisi Formal PDA
```
M = (Q, Σ, Γ, δ, q0, Z0, F)

Q  : himpunan STATE
Σ  : alphabet INPUT
Γ  : alphabet STACK
q0 ∈ Q : INITIAL state
Z0 ∈ Γ : simbol AWAL di stack
F  ⊆ Q : himpunan FINAL state
δ  : fungsi transisi
     Q x (Σ ∪ {ε}) x Γ -> subset dari (Q x Γ*)
```

### Fungsi Transisi (Move)
Fungsi transisi (move) untuk PDA deterministik didefinisikan:
1. **δ(q, a, z) = (p, γ)** — di mana q, p adalah state, a ∈ Σ, z adalah simbol stack, γ ∈ Γ*.
2. **δ(q, ε, z) = (p, γ)** — transisi TANPA membaca simbol input (analog ε-transition di NFA).

### Contoh Lengkap: PDA untuk L = {wcw^R}
```
M = ({q1, q2}, {0, 1, c}, {R, B, G}, δ, q1, R, ∅)   (diterima lewat empty stack)

δ didefinisikan:
1. δ(q1, 0, R) = (q1, BR)
2. δ(q1, 0, B) = (q1, BB)
3. δ(q1, 0, G) = (q1, BG)
4. δ(q1, c, R) = (q2, R)
5. δ(q1, c, B) = (q2, B)
6. δ(q1, c, G) = (q2, G)
7. δ(q2, 0, B) = (q2, ε)
8. δ(q2, ε, R) = (q2, ε)
9. δ(q1, 1, R) = (q1, GR)
10. δ(q1, 1, B) = (q1, GB)
11. δ(q1, 1, G) = (q1, GG)
12. δ(q2, 1, G) = (q2, ε)
```

### Instantaneous Description (ID)
**ID** adalah triple **(q, w, γ)**, di mana q = state, w = sisa input yang BELUM dibaca, γ = isi stack.

**Notasi langkah:** `(q, aw, z) ⊢ (p, w, β)` JIKA δ(q, a, z) = (p, β). (Catatan: 'a' bisa juga sama dengan ε.)

**Contoh: string input `001c100`**
```
(q1, 001c100, R) ⊢ (q1, 01c100, BR)
                 ⊢ (q1, 1c100, BBR)
                 ⊢ (q1, c100, GBBR)
                 ⊢ (q2, 100, GBBR)
                 ⊢ (q2, 00, BBR)
                 ⊢ (q2, 0, BR)
                 ⊢ (q2, ε, R)
                 ⊢ (q2, ε, ε)     -> DITERIMA (accepted by empty stack)
```

### Bahasa yang Diterima PDA
Untuk PDA M = (Q, Σ, Γ, δ, q0, Z0, F):

**L(M)** — bahasa yang diterima lewat FINAL STATE:
```
L(M) = { w | (q0, w, Z0) ⊢* (p, ε, γ), untuk suatu p ∈ F dan γ ∈ Γ* }
```

**N(M)** — bahasa yang diterima lewat EMPTY STACK (null stack):
```
N(M) = { w | (q0, w, Z0) ⊢* (p, ε, ε), untuk suatu p ∈ Q }
```

## Diagram & Visual
> [!warning] Diagram transisi PDA (slide 12) dibuat manual dengan text-box PowerPoint (bukan gambar raster) — sudah direkonstruksi ke tabel transisi di atas, tapi diagram VISUAL (state q1, q2 dengan panah berlabel `input, top-of-stack / stack-baru`) perlu dibuka manual di file PPT asli untuk referensi visual.

## Rumus / Sintaks
```
Definisi Formal PDA:
M = (Q, Σ, Γ, δ, q0, Z0, F)

Fungsi transisi:
δ(q, a, z) = (p, γ)     dengan input a dibaca
δ(q, ε, z) = (p, γ)     tanpa membaca input

Instantaneous Description (ID):
(q, aw, z) |- (p, w, β)   jika δ(q, a, z) = (p, β)

Bahasa diterima PDA:
L(M) = { w | (q0,w,Z0) |-* (p,ε,γ), p ∈ F }        (final state)
N(M) = { w | (q0,w,Z0) |-* (p,ε,ε), p ∈ Q }         (empty stack)
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Finite control** | Bagian PDA yang menyimpan state saat ini, sama seperti pada FA |
| **LIFO (Last In First Out)** | Prinsip kerja stack — elemen yang terakhir dimasukkan adalah yang pertama dikeluarkan |
| **w^R (w reversed)** | Notasi untuk string w yang dibalik urutannya |
| **Turnstile (⊢)** | Simbol yang menunjukkan SATU langkah transisi ID ke ID berikutnya |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa PDA BUTUH stack untuk mengenali `L = {wcw^R}`, sementara Finite Automaton (TANPA stack) TIDAK BISA mengenali bahasa ini. Hubungkan dengan konsep bahwa `w` bisa punya PANJANG SEMBARANG — kenapa FA dengan JUMLAH STATE TERBATAS tidak cukup untuk "mengingat" seluruh isi w sebelum mencocokkannya dengan w^R?
2. **(C4 – Analisis)** Bandingkan Empty Stack Acceptance dan Final State Acceptance. Analisis: pada contoh ID untuk string "001c100" di materi, tunjukkan langkah PERSIS di mana stack menjadi KOSONG — apakah momen itu BERSAMAAN dengan input yang habis dibaca? Kenapa keduanya harus terjadi BERSAMAAN untuk "accepted by empty stack"?
3. **(C5 – Evaluasi)** Evaluasi hubungan PDA dengan CFG (dari [[W07 - Context-Free Grammar]]). Slide menyebut PDA sebagai "FA counterpart dari CFG" — jelaskan MAKNA klaim ini: apakah SETIAP CFG punya PDA ekuivalen yang mengenali bahasa yang sama, dan sebaliknya? Apa implikasinya bagi desain PARSER (yang pada dasarnya adalah implementasi PDA) di compiler nyata?
4. **(C5 – Evaluasi)** Bandingkan mekanisme PUSH saat membaca w (state q1) dengan mekanisme POP saat membaca w^R (state q2) di contoh PDA `L={wcw^R}`. Evaluasi: apa yang terjadi kalau string input punya `w` dan `w^R` yang TIDAK BENAR-BENAR merupakan kebalikan satu sama lain (misalnya "01c10" bukan "01c10" yang benar)? Di langkah mana proses ID akan GAGAL (tidak ada transisi yang berlaku)?
5. **(C6 – Cipta)** Kerjakan latihan dari slide 15: diberikan PDA dengan definisi transisi lengkap (state q1, q2, simbol stack B, M, H), (a) konstruksikan PDA-nya (deskripsikan komponennya: Q, Σ, Γ, q0, Z0, F), (b) cari Instantaneous Description LENGKAP untuk string input `001100`, tunjukkan SETIAP langkah `⊢` dari awal sampai diterima atau ditolak.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W09 - Syntax Analysis - Parsing Strategies]]
- [[W11 - Top-Down Parsing]]
- [[Compiler - Review dan Glosari]]
