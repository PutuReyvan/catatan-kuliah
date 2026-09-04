---
matkul: Compilation Techniques
minggu: 4
sks: 3
sumber: Session 04_DFA & NFA.pptx
tags: [kuliah/compiler, minggu/w04]
status: draft
diproses: 2026-09-04
---

# W04 — DFA dan NFA

## Ringkasan
> - **Finite Automaton (FA)** = model matematika untuk MENGENALI string input, bertindak sebagai "recognizer" (menerima/menolak). Dua jenis: **NFA** dan **DFA**.
> - **NFA** — untuk satu state dan satu simbol, transisinya bisa BANYAK (nol, satu, atau lebih), bahkan boleh transisi **ε** (tanpa mengonsumsi simbol apa pun). String DITERIMA kalau ADA SETIDAKNYA SATU path dari start ke accepting state.
> - **DFA** — untuk setiap state dan simbol, transisinya **TEPAT SATU**. String diterima kalau, setelah semua simbol diproses, berakhir di accepting state — HANYA ADA SATU path yang mungkin.
> - **Setiap NFA punya DFA EKUIVALEN** yang mengenali bahasa yang sama — dikonversi lewat **subset construction**, di mana tiap state DFA merepresentasikan SEKUMPULAN state NFA.
> - **ε-closure(q)** = kumpulan semua state yang bisa dicapai dari q HANYA lewat transisi ε — konsep kunci untuk mengonversi NFA-dengan-ε ke DFA.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Finite Automaton (FA) | Model matematika untuk mengenali (menerima/menolak) string input |
| NFA | FA dengan transisi state-simbol yang bisa banyak/nol, boleh transisi ε |
| DFA | FA dengan tepat satu transisi per pasangan state-simbol |
| ε-transition | Transisi antar state TANPA mengonsumsi simbol input |
| ε-closure(q) | Semua state yang bisa dicapai dari q hanya lewat transisi ε |
| Subset construction | Metode mengonversi NFA jadi DFA ekuivalen |

## Isi

### Finite Automata: Gambaran Umum
**Finite Automaton (FA)** adalah model matematika untuk MENGENALI string input — bertindak sebagai recognizer:
```
Input String → [Finite Automaton] → Accepted / Rejected
```
Dua jenis FA: **Nondeterministic Finite Automaton (NFA)** dan **Deterministic Finite Automaton (DFA)**.

### Definisi Formal NFA
NFA terdiri dari:
- Himpunan HINGGA state **S**.
- Himpunan simbol input **Σ** (alphabet input) — ε (string kosong) TIDAK PERNAH jadi anggota Σ.
- **Fungsi transisi** yang memberi, untuk setiap state dan setiap simbol di Σ ∪ {ε}, sebuah HIMPUNAN state berikutnya.
- Sebuah state **s₀** dari S sebagai state AWAL (start/initial state).
- Himpunan state **F** (subset dari S) sebagai accepting state (final state).

### Contoh NFA: (a|b)\*abb
Transition table untuk NFA yang mengenali regular expression `(a|b)*abb`:
| State | a | b | ε |
| --- | --- | --- | --- |
| 0 | {0,1} | {0} | ∅ |
| 1 | ∅ | {2} | ∅ |
| 2 | ∅ | {3} | ∅ |
| 3 | ∅ | ∅ | ∅ |

### Acceptance Rule untuk NFA
**Sebuah NFA menerima string x kalau ADA SETIDAKNYA SATU path dari start state ke accepting state yang label transisinya MENGEJA x.** Transisi ε diabaikan karena tidak mengonsumsi simbol input apa pun.

**Contoh 1:** RE `(a|b)*abb`, input `aabb` — DITERIMA karena ada path dari start ke accepting state.
- **Multiple paths mungkin terjadi** — string input yang sama bisa mengikuti path BERBEDA dalam sebuah NFA.
- **Aturan penerimaan:** sebuah string diterima NFA kalau ADA path berlabel string itu yang mencapai accepting state — path yang berakhir di state NON-accepting TIDAK mempengaruhi hasil penerimaan.

**Contoh 2:** RE `(aa*|bb*)`, input `aaa` — diterima lewat path tertentu. **Peran ε-transition:** tidak mengonsumsi simbol input apa pun, tidak muncul di string yang terbentuk.

> [!info] Analogi
> NFA itu seperti mencari jalan keluar dari LABIRIN dengan BANYAK jalur bercabang sekaligus — kamu boleh "mencoba" semua cabang secara paralel (secara konseptual). Kamu berhasil KELUAR (string diterima) kalau ADA SETIDAKNYA SATU dari SEMUA cabang yang kamu jelajahi itu berhasil sampai ke pintu keluar (accepting state) — jalur BUNTU lainnya tidak masalah, tidak menggagalkan keberhasilanmu. Ini beda dari DFA, di mana kamu HANYA punya SATU jalur tunggal yang harus diikuti — tidak ada percabangan sama sekali.

### Definisi Formal DFA
DFA terdiri dari:
- Himpunan HINGGA state **Q**.
- Himpunan simbol input **Σ** (input alphabet) — SETIAP simbol input adalah anggota Σ.
- **Fungsi transisi** yang memberi, untuk setiap state dan setiap simbol di Σ, **TEPAT SATU** state berikutnya.
- Sebuah state **q₀** dari Q sebagai start state.
- Himpunan state **F** (subset Q) sebagai accepting state.

**Contoh transition table DFA** yang mengenali `(a|b)*abb`:
| State | a | b |
| --- | --- | --- |
| 0 | 1 | 0 |
| 1 | 1 | 2 |
| 2 | 1 | 3 |
| 3 | 1 | 0 |

**Acceptance untuk DFA:** DFA menerima string x kalau, setelah memproses SEMUA simbol input, berakhir di accepting state. Karena DFA bersifat deterministic, **hanya ada SATU path yang mungkin** untuk sembarang string input.

### Perbandingan DFA vs NFA
| Aspek | DFA | NFA |
| --- | --- | --- |
| Definisi | Setiap state punya TEPAT SATU transisi per simbol input | State BOLEH punya banyak transisi untuk simbol yang sama |
| Fungsi transisi | δ : Q × Σ → Q | δ : Q × Σ → 2^Q (himpunan kuasa) |
| Next state | Tepat SATU next state per pasangan state-input | Nol, satu, atau banyak next state |
| Computation path | Hanya SATU path untuk string tertentu | BISA banyak path sekaligus |
| Kondisi diterima | Diterima kalau SATU-SATUNYA path berakhir di final state | Diterima kalau SETIDAKNYA SATU path mencapai final state |
| Transition diagram | Tidak ada state dengan DUA outgoing edge berlabel simbol sama | State BOLEH punya beberapa outgoing edge berlabel simbol sama |
| Implementasi | Lebih MUDAH diimplementasikan langsung di software/hardware | Biasanya DIKONVERSI ke DFA dulu sebelum diimplementasikan |
| Jumlah state | Bisa butuh LEBIH BANYAK state | Sering butuh LEBIH SEDIKIT state |
| Eksekusi | Deterministic, TIDAK ADA "tebak-tebakan" | Secara konseptual mengeksplorasi BANYAK kemungkinan sekaligus |

> [!info] Konteks tambahan (bukan dari slide)
> Meski NFA lebih mudah DIRANCANG (construct), DFA lebih mudah dan CEPAT dieksekusi. Ini alasan kenapa alur praktis pembuatan lexical analyzer biasanya: **Regular Expression → NFA (mudah dirancang) → DFA (mudah dieksekusi)**, dilanjutkan minggu depan di [[W05 - RE ke DFA]].

### Konversi NFA ke DFA
**Kenapa konversi diperlukan?** DFA lebih mudah DISIMULASIKAN dibanding NFA. NFA bisa punya: banyak transisi untuk simbol yang sama, ε-transisi, beberapa computation path sekaligus. **Setiap NFA punya DFA EKUIVALEN yang mengenali bahasa yang SAMA.**

**Teorema:** Kalau L adalah L(M) untuk sebuah NFA, maka L diterima juga oleh sebuah DFA. Untuk setiap NFA M = (Q, Σ, δ, q₀, F), bisa ditemukan DFA EKUIVALEN M₁ = (Q₁, Σ₁, δ₁, q₁₀, F₁) yang mengenali bahasa yang SAMA, di mana:
- **Q₁** = himpunan SUBSET dari Q.
- **q₁₀** = q₀.
- **δ₁([q1,q2,...,qi], a) = [p1,p2,...,pj]** jika dan hanya jika δ({q1,...,qi}, a) = {p1,...,pj}.
- **F₁** = dibentuk dari SEMUA state di Q₁ yang berisi SETIDAKNYA SATU state di F.

**Contoh Konversi 1:**
NFA:
| State | 0 | 1 |
| --- | --- | --- |
| q0 | {q0,q1} | q2 |
| q1 | q0 | q1 |
| *q2 | q1 | {q0,q1} |

Hasil konversi ke DFA (subset construction): state DFA merepresentasikan HIMPUNAN state NFA — {q0}, {q0,q1}, {q2}, {q1}, {q1,q2}, {q0,q1} dst, dengan transisi yang dihitung dari GABUNGAN transisi seluruh anggota himpunan itu.

**Contoh Konversi 2:**
NFA M = ({q0, q1}, {0, 1}, δ, q0, {q1}), dengan fungsi transisi:
```
δ(q0, 0) = {q0, q1}     δ(q0, 1) = {q1}
δ(q1, 0) = ∅             δ(q1, 1) = {q0, q1}
```

Transition table DFA hasil konversi:
| State | Input 0 | Input 1 |
| --- | --- | --- |
| {q0} | {q0,q1} | {q1} |
| {q0,q1} | {q0,q1} | {q0,q1} |
| {q1} | ∅ | {q0,q1} |

### NFA dengan ε-Moves
Transisi **ε** diizinkan di NFA — kita bisa pindah dari satu state ke state lain TANPA mengonsumsi simbol apa pun. NFA menerima string x jika dan hanya jika ada path dari starting state ke salah satu accepting state yang label edge-nya MENGEJA x.

**Formal definisi NFA dengan ε-transition:**
```
M = (Q, (Σ ∪ {ε}), δ, q0, F)
Q, Σ, q0, F : sama seperti FA biasa
Fungsi transisi: δ : Q × (Σ ∪ {ε}) → Q
```

### ε-Closure
**ε-closure(q)** = himpunan SEMUA state yang bisa dicapai dari q HANYA lewat transisi ε.

**Definisi rekursif:**
- **Basis:** state q ada di ECLOSE(q). Jadi `Eclose(q) = {q}` minimal.
- **Induksi:** kalau state p ada di ECLOSE(q), dan ada transisi dari p ke r berlabel ε, maka state r JUGA ada di ECLOSE(q).

**Contoh:**
```
Eclose(Q0) = {Q0, Q1}
Eclose(Q1) = {Q1}
Eclose(Q2) = {Q2}
Eclose(Q3) = {Q0, Q1, Q3, Q4, Q5}
Eclose(Q4) = {Q0, Q1, Q4, Q5}
Eclose(Q5) = {Q0, Q1, Q5}
```

> [!info] Analogi
> ε-closure itu seperti mencari SEMUA RUANGAN yang bisa kamu capai dari satu ruangan HANYA lewat PINTU YANG SELALU TERBUKA (tanpa perlu kunci/tiket masuk — analog "tanpa mengonsumsi simbol"). Kalau dari ruangan A ada pintu terbuka ke B, dan dari B ada pintu terbuka lagi ke C, maka ε-closure(A) mencakup A, B, DAN C — meski kamu tidak "melakukan aksi apa pun" (tidak baca simbol input) untuk berpindah ke sana, cuma jalan lewat pintu yang memang selalu terbuka.

### Contoh Aplikasi: NFA-ε untuk Bilangan Desimal
Contoh NFA-ε yang menerima bilangan desimal (tanda +/-, digit, titik desimal, digit — salah satu kelompok digit boleh kosong). **Bahasa yang diterima:** L(M) = {w | δ(q0,w) ∩ F ≠ ∅}.

**Menghitung string "5.6":**
```
δ(q0, ε) = ECLOSE(q0) = {q0,q1}
δ(q0, 5) = δ(q0,5) ∪ δ(q1,5) = {q1,q4} = ECLOSE(q1) ∪ ECLOSE(q4) = {q1,q4}
δ(q0, 5.) = δ(q1,.) ∪ δ(q4,.) = {q2,q3} = ECLOSE(q2) ∪ ECLOSE(q3) = {q2,q3,q5}
δ(q0, 5.6) = δ(q2,6) ∪ δ(q3,6) ∪ δ(q5,6) = {q3} = ECLOSE(q3) = {q3,q5}  → DITERIMA
```

**Menghitung string "+.12"** → DITERIMA (jejak perhitungan serupa, berakhir di state accepting).
**Menghitung string "-31"** → DITOLAK (jejak perhitungan berakhir di state {q1}, BUKAN accepting state).

### Teorema Ekuivalensi
**Kalau L diterima sebagai NFA dengan ε-transition, maka L JUGA diterima oleh NFA TANPA ε-transition.** Setiap NFA-ε punya NFA ekuivalen (tanpa ε), dan setiap NFA-ε punya DFA ekuivalen.

### Algoritma Konversi ε-NFA ke DFA
Diberikan ε-NFA **E = (QE, Σ, δE, q0, FE)**, DFA ekuivalen **A = (QD, Σ, δD, {qD}, FD)** didefinisikan:
- **QD** adalah himpunan SUBSET dari QE.
- **qD** = ECLOSE(q0).
- **FD** adalah himpunan state di QD yang mengandung SETIDAKNYA SATU accepting state dari E.
- **δD(S, a)** dihitung untuk setiap a di Σ dan himpunan S di QD:
  1. Misal S = {p1, p2, ..., pk}.
  2. Hitung ∪ᵏᵢ₌₁ δE(pi, a), sebut himpunan ini {r1, r2, ..., rm}.
  3. Maka δD(S, a) = ∪ᵐⱼ₌₁ ECLOSE(rj).

**Contoh lengkap konversi** (dari ε-NFA untuk bilangan desimal): dimulai dengan start state D = ECLOSE(q0) = {q0,q1}, lalu dihitung transisi untuk setiap simbol (+, -, ., digit), membentuk state DFA baru {q1}, {q2}, {q1,q4}, {q2,q3,q5}, {q3,q5} — setiap state DFA dicek apakah mengandung accepting state NFA asli, sampai SEMUA state baru sudah ditelusuri dan tidak ada state baru lagi yang muncul.

**Contoh kedua (angka biner dengan a/b, notasi S0-S2):**
```
S0 = ε-closure({0}) = {0,1,2,4,7}
ε-closure(move(S0,a)) = ε-closure({3,8}) = {1,2,3,4,6,7,8} = S1
ε-closure(move(S0,b)) = ε-closure({5}) = {1,2,4,5,6,7} = S2
transfunc[S0,a] = S1     transfunc[S0,b] = S2
... (proses berulang untuk S1 dan S2 sampai tidak ada state baru)
```
Hasil: S0 = start state DFA (karena 0 ∈ S0), **S1 = accepting state DFA (karena 8 ∈ S1)**.

## Diagram & Visual
- **Slide 7 — NFA Diagram dan Transition Table untuk (a|b)\*abb**
  ![[99-Assets/Compiler/W04-slide07.png]]
- **Slide 8 — Accepting Path untuk input "aabb"**
  ![[99-Assets/Compiler/W04-slide08.png]]
  ![[99-Assets/Compiler/W04-slide08a.png]]
- **Slide 9 — Accepting Path untuk input "aaa"**
  ![[99-Assets/Compiler/W04-slide09.png]]
  ![[99-Assets/Compiler/W04-slide09a.png]]
- **Slide 12 — DFA Diagram dan Transition Table untuk (a|b)\*abb**
  ![[99-Assets/Compiler/W04-slide12.png]]

## Rumus / Sintaks
```
NFA formal: M = (S, Σ, δ, s0, F)
δ : S x (Σ ∪ {ε}) -> 2^S     (bisa banyak next state, ε diizinkan)

DFA formal: M = (Q, Σ, δ, q0, F)
δ : Q x Σ -> Q                (tepat satu next state, tanpa ε)

Subset Construction (NFA -> DFA):
QD = himpunan subset QE
qD = ECLOSE(q0)
FD = subset QD yang mengandung >= 1 accepting state NFA
δD(S,a) = ECLOSE( gabungan δE(p,a) untuk semua p di S )

ε-closure(q):
Basis:     q ada di ECLOSE(q)
Induksi:   kalau p ada di ECLOSE(q) dan ada transisi ε dari p ke r,
           maka r JUGA ada di ECLOSE(q)
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Recognizer** | Sistem yang menentukan apakah sebuah input diterima atau ditolak |
| **Himpunan kuasa (2^Q)** | Himpunan dari semua subset himpunan Q, dipakai dalam fungsi transisi NFA |
| **Accepting/Final state** | State yang menandakan input diterima kalau eksekusi berakhir di sana |
| **Unmarked/Marked state** | Penanda dalam algoritma subset construction untuk state DFA yang belum/sudah diproses |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa fungsi transisi DFA dinotasikan δ: Q × Σ → Q (menghasilkan SATU state), sementara fungsi transisi NFA dinotasikan δ: Q × Σ → 2^Q (menghasilkan HIMPUNAN state). Apa konsekuensi PRAKTIS dari perbedaan notasi ini terhadap cara komputer MENGEKSEKUSI kedua jenis automaton ini?
2. **(C4 – Analisis)** Bandingkan proses menghitung ε-closure(q0) untuk automaton dengan BANYAK rantai transisi ε bertingkat (misalnya q0→q1 via ε, lalu q1→q2 via ε) dengan automaton yang HANYA punya satu transisi ε langsung. Analisis: kenapa definisi ε-closure harus REKURSIF (basis + induksi), bukan cukup melihat SATU langkah transisi ε saja?
3. **(C5 – Evaluasi)** Evaluasi trade-off "NFA sering butuh LEBIH SEDIKIT state, tapi DFA lebih cepat dieksekusi". Untuk compiler PRODUKSI (dipakai jutaan kali kompilasi program setiap hari), argumentasikan kenapa DFA (meski kadang butuh lebih banyak state) tetap jadi pilihan akhir yang lebih masuk akal dibanding NFA.
4. **(C5 – Evaluasi)** Bandingkan hasil perhitungan string "+.12" (DITERIMA) dan "-31" (DITOLAK) pada contoh ε-NFA bilangan desimal di materi. Evaluasi: di LANGKAH mana persis proses "-31" gagal mencapai accepting state, dan apa yang secara STRUKTURAL berbeda dari proses "5.6" yang berhasil diterima?
5. **(C6 – Cipta)** Kerjakan latihan #1 dari slide 36: konversikan NFA berikut ke DFA memakai subset construction, tunjukkan LANGKAH DEMI LANGKAH (state DFA baru yang terbentuk, transisi dihitung dari gabungan transisi NFA):
   ```
   State | 0     | 1
   p     | {p,q} | {p}
   q     | {r}   | {r}
   r     | {s}   | -
   *s    | {s}   | {s}
   ```
   (p = start state, s = accepting state)

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W03 - Lexical Analysis]]
- [[W05 - RE ke DFA]]
- [[Compiler - Review dan Glosari]]
