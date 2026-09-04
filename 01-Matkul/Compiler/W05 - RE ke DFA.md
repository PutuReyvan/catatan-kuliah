---
matkul: Compilation Techniques
minggu: 5
sks: 3
sumber: Session 05_RE to DFA.pptx
tags: [kuliah/compiler, minggu/w05]
status: draft
diproses: 2026-09-04
---

# W05 — RE ke DFA

## Ringkasan
> - Ada DUA cara mengubah **Regular Expression (RE)** jadi automaton yang bisa dieksekusi: **Thompson's Construction** (RE → NFA-ε, lalu dikonversi ke DFA seperti minggu lalu) ATAU **konversi LANGSUNG RE → DFA** (tanpa lewat NFA sama sekali) memakai **Firstpos, Lastpos, Followpos.**
> - **Thompson's Construction** membangun NFA-ε secara REKURSIF dari struktur RE — ada aturan dasar untuk `∅`, `ε`, `a`, dan aturan gabungan untuk `R+S` (union), `R.S` (concatenation), `R*` (closure).
> - Metode LANGSUNG memakai **augmented regular expression** (RE ditambah simbol akhir `#`), membangun **syntax tree**, lalu menghitung 3 fungsi per node: **nullable** (bisa hasilkan string kosong?), **firstpos** (posisi simbol PERTAMA), **lastpos** (posisi simbol TERAKHIR).
> - **Followpos(i)** = posisi yang BISA MENGIKUTI posisi i dalam string yang dihasilkan RE — dihitung dari DUA aturan (concatenation-node dan star-node), lewat SATU traversal depth-first syntax tree.
> - Algoritma lengkap **RE → DFA** memakai firstpos/followpos untuk MEMBANGUN state DFA LANGSUNG tanpa perlu subset construction dari NFA — lebih efisien untuk implementasi lexical analyzer generator (seperti Lex/Flex).

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Thompson's Construction | Metode membangun NFA-ε secara rekursif dari struktur regular expression |
| Augmented regular expression | RE yang ditambah simbol akhir `#` untuk menandai akhir string |
| Nullable(n) | True kalau sub-ekspresi di node n bisa hasilkan string kosong |
| Firstpos(n) | Himpunan posisi simbol PERTAMA dari string yang dihasilkan sub-ekspresi di n |
| Lastpos(n) | Himpunan posisi simbol TERAKHIR dari string yang dihasilkan sub-ekspresi di n |
| Followpos(i) | Himpunan posisi yang bisa MENGIKUTI posisi i dalam string yang dihasilkan RE |

## Isi

### Konversi RE ke NFA-ε: Thompson's Construction
**Teorema:** Setiap Regular Expression R bisa dibuat jadi mesin NFA-ε M, sehingga L(M) = L(R).

**Aturan dasar (basis):**
1. **R = ∅** — automaton tanpa transisi menerima.
2. **R = ε** — automaton dengan satu transisi ε.
3. **R = a** — automaton dengan satu transisi berlabel `a`.

**Aturan gabungan (induktif):**
4. **R = R+S (union)** — gabungkan automaton R dan S lewat state baru dengan transisi ε ke KEDUA automaton, dan transisi ε dari KEDUA automaton ke state akhir baru.
5. **R = R.S (concatenation)** — automaton R diikuti automaton S, state akhir R disambung ke state awal S.
6. **R = R\* (Kleene closure)** — automaton R dilingkupi transisi ε yang mengizinkan PENGULANGAN (loop kembali) atau LOMPAT LANGSUNG (melewatkan R sepenuhnya, untuk kasus nol pengulangan).

**Contoh: Membangun NFA-ε untuk `(0+1)*1(0+1)`** — dibangun bertahap dari komponen paling dasar (0, 1) digabung dengan aturan union, lalu closure, lalu concatenation, sesuai struktur RE-nya.

**Contoh Thompson's Construction untuk `(a|b)*a`:**
```
Langkah 1: bangun automaton dasar untuk 'a' dan 'b'
Langkah 2: gabungkan jadi (a|b) lewat aturan union
Langkah 3: bungkus jadi (a|b)* lewat aturan closure
Langkah 4: sambung dengan 'a' lewat aturan concatenation -> (a|b)*a
```

> [!info] Analogi
> Thompson's Construction itu seperti merakit LEGO dari potongan PALING DASAR. Kamu punya potongan dasar untuk huruf tunggal ('a', 'b'), lalu ada TIGA "instruksi resmi" cara menyambung potongan: **union** (rekatkan dua potongan jadi pilihan "salah satu boleh"), **concatenation** (sambung dua potongan berurutan), **closure** (buat loop yang bisa diulang berapa kali saja, termasuk NOL kali). Karena SETIAP regular expression, betapapun rumit, pada akhirnya tersusun dari kombinasi tiga operasi ini, kamu SELALU bisa merakit NFA-nya secara sistematis, potongan demi potongan, mengikuti struktur RE itu persis.

### Konversi RE ke DFA Secara Langsung
Kita BISA mengonversi regular expression LANGSUNG jadi DFA (tanpa membuat NFA lebih dulu) — metode ini memakai **augmented regular expression** dan fungsi **firstpos, lastpos, followpos**.

### Syntax Tree dan Augmented Regular Expression
Contoh syntax tree untuk `(a|b)*a`: setiap SIMBOL diberi NOMOR (posisi), setiap simbol ada di sebuah LEAF (daun), inner node adalah OPERATOR. **Augmented regular expression** = RE asli DITAMBAH simbol akhir `#`, jadi `(a|b)*a#` — simbol `#` ini menandai AKHIR string dan posisinya SELALU jadi bagian dari accepting state di DFA hasil.

### Firstpos, Lastpos, Nullable
Untuk menghitung followpos, dibutuhkan TIGA fungsi tambahan yang didefinisikan untuk SEMUA node syntax tree (bukan cuma leaf):
- **firstpos(n)** — himpunan posisi SIMBOL PERTAMA dari string yang dihasilkan sub-ekspresi berakar di n.
- **lastpos(n)** — himpunan posisi SIMBOL TERAKHIR dari string yang dihasilkan sub-ekspresi berakar di n.
- **nullable(n)** — TRUE kalau string KOSONG termasuk anggota string yang dihasilkan sub-ekspresi di n, FALSE kalau tidak.

**Tabel aturan perhitungan:**
| Node n | nullable(n) | firstpos(n) | lastpos(n) |
| --- | --- | --- | --- |
| leaf berlabel ε | true | ∅ | ∅ |
| leaf berlabel posisi i | false | {i} | {i} |
| n = c1 \| c2 (union) | nullable(c1) OR nullable(c2) | firstpos(c1) ∪ firstpos(c2) | lastpos(c1) ∪ lastpos(c2) |
| n = c1 c2 (concatenation) | nullable(c1) AND nullable(c2) | kalau nullable(c1): firstpos(c1)∪firstpos(c2), else firstpos(c1) | kalau nullable(c2): lastpos(c1)∪lastpos(c2), else lastpos(c2) |
| n = c1\* (star) | true | firstpos(c1) | lastpos(c1) |

### Followpos
**followpos(i)** = himpunan posisi yang BISA MENGIKUTI posisi i dalam string yang dihasilkan augmented regular expression.

**Dua aturan mendefinisikan followpos:**
1. Kalau n adalah CONCATENATION-node dengan child kiri c1 dan child kanan c2, dan i adalah posisi di lastpos(c1), maka SEMUA posisi di firstpos(c2) masuk ke followpos(i).
2. Kalau n adalah STAR-node, dan i adalah posisi di lastpos(n), maka SEMUA posisi di firstpos(n) masuk ke followpos(i).

Kalau firstpos dan lastpos SUDAH dihitung untuk setiap node, followpos setiap posisi bisa dihitung lewat SATU traversal depth-first syntax tree.

**Contoh perhitungan untuk `(a|b)*a#`** (posisi 1=a di dalam star, 2=b di dalam star, 3=a setelah star, 4=#):
```
firstpos(root) = {1,2,3}
followpos(1) = {1, 2, 3}
followpos(2) = {1, 2, 3}
followpos(3) = {4}
followpos(4) = {}
```

> [!info] Analogi
> Firstpos, lastpos, dan followpos itu seperti membaca PETA JALAN sebuah cerita bercabang. **Firstpos** itu "di halaman mana cerita ini BISA DIMULAI?" — kalau ada percabangan (union), bisa mulai dari BEBERAPA halaman sekaligus. **Lastpos** itu "di halaman mana cerita ini BISA BERAKHIR?". **Followpos(i)** itu "kalau aku baru saja selesai baca halaman i, halaman APA SAJA yang MUNGKIN aku baca SELANJUTNYA?" — informasi followpos inilah yang langsung dipakai untuk membangun TRANSISI di DFA, karena pada dasarnya followpos MENGGANTIKAN peran "mengikuti banyak jalur NFA sekaligus" tanpa perlu benar-benar membangun NFA-nya dulu.

### Algoritma Lengkap: RE → DFA Langsung
1. Buat SYNTAX TREE dari `(r)#`.
2. Hitung fungsi: **followpos, firstpos, lastpos, nullable.**
3. Masukkan **firstpos(root)** ke dalam state DFA sebagai state BELUM DITANDAI (unmarked).
4. **Selama** ada state S yang belum ditandai di DFA:
   - Tandai S.
   - Untuk SETIAP simbol input a:
     - Misal s1,...,sn adalah posisi-posisi di S yang simbolnya adalah a.
     - S' ← followpos(s1) ∪ ... ∪ followpos(sn).
     - move(S, a) ← S'.
     - Kalau S' TIDAK KOSONG dan BELUM ADA di state DFA, masukkan S' sebagai state BELUM DITANDAI.
5. Start state DFA = **firstpos(root)**.
6. Accepting state DFA = SEMUA state yang mengandung POSISI SIMBOL `#`.

### Contoh Lengkap: RE (a|b)\*abb → DFA Langsung
**Augmented RE:** `(a|b)*abb#` dengan posisi `1 2 3 4 5 6` (posisi 1=a dalam star, 2=b dalam star, 3=a, 4=b, 5=b, 6=#).

**Firstpos root** = {1, 2, 3}

**Tabel followpos:**
| Node | Followpos |
| --- | --- |
| 1 | {1, 2, 3} |
| 2 | {1, 2, 3} |
| 3 | {4} |
| 4 | {5} |
| 5 | {6} |
| 6 | — |

**Konstruksi DFA:**
| State | a | b |
| --- | --- | --- |
| {1,2,3} | followpos{1,3} = {1,2,3,4} | followpos{2} = {1,2,3} |
| {1,2,3,4} | followpos{1,3} = {1,2,3,4} | followpos{2,4} = {1,2,3,5} |
| {1,2,3,5} | followpos{1,3} = {1,2,3,4} | followpos{2,5} = {1,2,3,6} |
| *{1,2,3,6} | followpos{1,3} = {1,2,3,4} | followpos{2} = {1,2,3} |

State `{1,2,3,6}` ditandai bintang (*) karena mengandung posisi 6 (posisi `#`) — jadi ACCEPTING state.

### Contoh Kedua: (a|ε)bc\*#
```
(a|ε)  b  c*  #
  1        2   3   4

S1 = firstpos(root) = {1,2}
  a: followpos(1) = {2} = S2      move(S1,a) = S2
  b: followpos(2) = {3,4} = S3    move(S1,b) = S3
  b: followpos(2) = {3,4} = S3    move(S2,b) = S3
  c: followpos(3) = {3,4} = S3    move(S3,c) = S3

Start state: S1
Accepting states: {S3}
```

## Diagram & Visual
- **Slide 5 — Aturan Thompson's Construction untuk R+S, R.S, R\***
  ![[99-Assets/Compiler/W05-slide05.png]]
  ![[99-Assets/Compiler/W05-slide05a.png]]
  ![[99-Assets/Compiler/W05-slide05b.png]]
- **Slide 6 — NFA-ε untuk RE (0+1)\*1(0+1)**
  ![[99-Assets/Compiler/W05-slide06.png]]
- **Slide 16 — Syntax Tree Augmented RE (a|b)\*abb#**
  ![[99-Assets/Compiler/W05-slide16.png]]
- **Slide 17 — Tabel nullable, firstpos, lastpos untuk RE (a|b)\*abb**
  ![[99-Assets/Compiler/W05-slide17.png]]
- **Slide 18 — Hasil akhir DFA dari RE (a|b)\*abb**
  ![[99-Assets/Compiler/W05-slide18.png]]

## Rumus / Sintaks
```
Algoritma RE -> DFA Langsung:
1. Buat syntax tree dari (r)#
2. Hitung nullable, firstpos, lastpos, followpos
3. State awal DFA = firstpos(root), tandai "belum diproses"
4. Selama ada state S belum diproses:
     tandai S sudah diproses
     untuk setiap simbol input a:
       S' = union followpos(i) untuk semua posisi i di S berlabel a
       move(S, a) = S'
       kalau S' baru, tambahkan sebagai state belum diproses
5. Accepting state = semua state yang mengandung posisi '#'

Aturan Followpos:
- Concatenation c1.c2: i di lastpos(c1) -> firstpos(c2) masuk followpos(i)
- Star n*: i di lastpos(n) -> firstpos(n) masuk followpos(i)
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Syntax tree (RE)** | Pohon yang merepresentasikan struktur regular expression, leaf = simbol, internal node = operator |
| **Unmarked/Marked state** | Penanda status pemrosesan sebuah state selama algoritma dijalankan |
| **Depth-first traversal** | Penelusuran pohon yang menjelajah SATU cabang sampai habis sebelum ke cabang lain |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa metode RE→DFA langsung (firstpos/lastpos/followpos) dianggap LEBIH EFISIEN dibanding metode dua-langkah (RE→NFA via Thompson's, lalu NFA→DFA via subset construction dari [[W04 - DFA dan NFA]]). Apa langkah PERANTARA yang bisa DILEWATI dengan metode langsung ini?
2. **(C4 – Analisis)** Bandingkan aturan followpos untuk CONCATENATION-node dan STAR-node. Analisis: kenapa star-node butuh aturan KHUSUS TAMBAHAN (i di lastpos(n) → firstpos(n) masuk followpos(i)) yang TIDAK dimiliki concatenation biasa — apa yang secara STRUKTURAL berbeda dari operasi "closure" (bisa berulang) dibanding "sambung sekali"?
3. **(C5 – Evaluasi)** Evaluasi kenapa SIMBOL `#` (augmented RE) WAJIB ditambahkan sebelum membangun syntax tree, bukan opsional. Apa yang akan RUSAK dari algoritma "accepting state = state yang mengandung posisi #" kalau simbol `#` ini DIHILANGKAN?
4. **(C5 – Evaluasi)** Bandingkan hasil akhir DFA dari contoh `(a|b)*abb` (4 state: {1,2,3}, {1,2,3,4}, {1,2,3,5}, {1,2,3,6}) dengan hasil konversi NFA→DFA memakai subset construction untuk RE yang SAMA di [[W04 - DFA dan NFA]] (kalau kamu kerjakan). Evaluasi: apakah JUMLAH state akhirnya SAMA, dan kenapa secara TEORI kedua metode (langsung vs via NFA) HARUS menghasilkan DFA yang setidaknya EKUIVALEN (meski representasi state-nya mungkin berbeda notasi)?
5. **(C6 – Cipta)** Kerjakan latihan dari slide 20: untuk RE `(a|b)* (ab|bb) a*`, (a) buat syntax tree dengan penomoran posisi, (b) hitung firstpos, lastpos, dan followpos untuk setiap posisi, (c) bangun DFA hasilnya dan gambarkan diagram transisinya (boleh dalam bentuk tabel state-transisi kalau diagram sulit digambar dalam teks).

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W04 - DFA dan NFA]]
- [[W06 - DFA Minimization]]
- [[Compiler - Review dan Glosari]]
