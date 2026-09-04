---
matkul: Compilation Techniques
minggu: 9
sks: 3
sumber: Session 09_Syntax Analysis Parsing Strategies.pptx
tags: [kuliah/compiler, minggu/w09]
status: draft
diproses: 2026-09-04
---

# W09 — Syntax Analysis: Parsing Strategies

## Ringkasan
> - **Leftmost Derivation (LMD)** selalu mengembangkan non-terminal PALING KIRI duluan; **Rightmost Derivation (RMD)** selalu mengembangkan non-terminal PALING KANAN duluan — keduanya menghasilkan parse tree yang SAMA untuk grammar yang sama, hanya urutan pengembangannya berbeda.
> - **Top-Down Parsing** membangun tree dari ROOT ke LEAVES (memprediksi produksi mana yang dipakai) — memakai **Leftmost Derivation**. **Bottom-Up Parsing** membangun dari LEAVES ke ROOT (mereduksi substring jadi non-terminal) — memakai KEBALIKAN dari **Rightmost Derivation**.
> - Top-Down TIDAK BISA menangani grammar LEFT-RECURSIVE (grammar harus ditransformasi dulu, ingat [[W08 - Syntax Analysis - Parsing Fundamentals]]); Bottom-Up bisa menangani kelas grammar yang JAUH LEBIH LUAS termasuk left-recursive.
> - **4 teknik penanganan syntax error**: Panic-Mode Recovery (sederhana, buang input sampai token sinkronisasi), Phrase-Level Recovery (koreksi lokal), Error Productions (grammar diperluas untuk mengenali kesalahan umum), Global Correction (secara teori optimal tapi MAHAL secara komputasi, TIDAK praktis).

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Leftmost Derivation (LMD) | Selalu mengembangkan non-terminal PALING KIRI duluan |
| Rightmost Derivation (RMD) | Selalu mengembangkan non-terminal PALING KANAN duluan |
| Top-Down Parsing | Membangun parse tree dari root ke leaves, memakai LMD |
| Bottom-Up Parsing | Membangun parse tree dari leaves ke root, kebalikan RMD |
| Syntax error | Error saat input melanggar aturan grammar |
| Panic-Mode Recovery | Strategi error recovery paling sederhana, buang input sampai token sinkronisasi |

## Isi

### Review: Apa yang Dilakukan Parser?
Parser menerima token, mengecek grammar, membangun parse tree, dan melaporkan syntax error — *(lihat detail lengkap di [[W08 - Syntax Analysis - Parsing Fundamentals]]).*

### Dari Grammar ke Parsing
**Grammar** memberitahu kita APA itu kalimat yang VALID. **Parser** memberitahu kita BAGAIMANA mengenalinya. Ada **dua strategi parsing utama**: **Top-Down Parsing** dan **Bottom-Up Parsing**.

### Leftmost Derivation (LMD)
**Selalu mengembangkan non-terminal PALING KIRI duluan.**

**Contoh:** grammar `S → AB`, `A → a`, `B → b`:
```
S ⇒ AB    (expand S)
  ⇒ aB    (expand leftmost A)
  ⇒ ab    (expand B)
```
A dikembangkan SEBELUM B.

### Rightmost Derivation (RMD)
**Selalu mengembangkan non-terminal PALING KANAN duluan.**

**Contoh:** grammar sama `S → AB`, `A → a`, `B → b`:
```
S ⇒ AB    (expand S)
  ⇒ Ab    (expand rightmost B)
  ⇒ ab    (expand A)
```
B dikembangkan SEBELUM A.

### LMD vs RMD
| Aspek | Leftmost Derivation | Rightmost Derivation |
| --- | --- | --- |
| Definisi | Selalu ekspansi non-terminal PALING KIRI | Selalu ekspansi non-terminal PALING KANAN |
| Urutan ekspansi | Kiri → Kanan | Kanan → Kiri |
| Sentential form | Menghasilkan left-sentential form | Menghasilkan right-sentential form (canonical derivation) |
| Dipakai oleh | Top-Down Parsing (Recursive Descent, LL Parser) | Bottom-Up Parsing (LR Parser membangun KEBALIKAN dari RMD) |
| Output akhir | Kalimat yang SAMA | Kalimat yang SAMA |
| Parse tree | Menghasilkan parse tree yang SAMA dengan RMD | Menghasilkan parse tree yang SAMA dengan LMD |

> [!info] Konteks tambahan (bukan dari slide)
> Poin PENTING: meski proses PENURUNANNYA berbeda (urutan mana yang diekspansi duluan), **hasil AKHIR parse tree-nya SAMA PERSIS** untuk grammar yang sama — LMD dan RMD cuma dua cara BERBEDA untuk "membaca" pohon derivasi yang sama, bukan dua pohon yang berbeda. Bedanya baru terasa PENTING saat menentukan STRATEGI PARSING: Top-Down secara alami cocok dengan LMD (karena membangun dari root, mengembangkan simbol demi simbol dari kiri), sementara Bottom-Up secara alami cocok dengan (kebalikan) RMD.

### Top-Down Parsing
Mulai dari START SYMBOL dan mencoba MENURUNKAN input string. Parser MEMPREDIKSI produksi mana yang dipakai SAMBIL membaca input.

```
Direction:  Root (Start Symbol)
                 ↓
            Leaves (Input String)
```
**Top-down parsing membangun tree dari ROOT ke LEAVES.**

### Bottom-Up Parsing
Mulai dari INPUT STRING dan MEREDUKSINYA sampai mencapai START SYMBOL. Parser MEREDUKSI "handle" memakai production rule.

```
Direction:  Leaves (Input String)
                 ↑
            Root (Start Symbol)
```
**Bottom-up parsing membangun tree dari LEAVES ke ROOT.**

### Top-Down vs Bottom-Up: Perbandingan Lengkap
| Aspek | Top-Down Parsing | Bottom-Up Parsing |
| --- | --- | --- |
| Arah parsing | Root → Leaves | Leaves → Root |
| Titik mulai | Start symbol | Input string (token) |
| Konstruksi tree | Dari root ke bawah | Dari leaves ke atas |
| Derivasi dipakai | Leftmost derivation | KEBALIKAN dari rightmost derivation |
| Ide dasar | MEMPREDIKSI produksi mana yang dipakai | MEREDUKSI substring jadi non-terminal sampai dapat start symbol |
| Aksi parsing | Ekspansi non-terminal | Operasi SHIFT dan REDUCE |
| Keluarga parser | LL Parser, Recursive Descent Parser | LR, SLR, LALR, Canonical LR Parser |
| Kebutuhan grammar | TIDAK BISA menangani grammar left-recursive; grammar sering butuh TRANSFORMASI | Bisa menangani kelas grammar JAUH LEBIH LUAS, termasuk left-recursive |
| Kompleksitas implementasi | LEBIH SEDERHANA | LEBIH KOMPLEKS |
| Deteksi error | Bisa mendeteksi error LEBIH AWAL saat prediksi | Biasanya mendeteksi error SETELAH cukup banyak input dibaca |

> [!info] Analogi
> Top-Down Parsing itu seperti membangun rumah dengan CETAK BIRU DULU — kamu mulai dari "ini akan jadi RUMAH" (start symbol), lalu MEMPREDIKSI: "rumah ini punya atap, dinding, fondasi" (ekspansi non-terminal), terus turun ke detail lebih kecil sampai ke BATU BATA individual (leaves/token). Bottom-Up Parsing itu KEBALIKANNYA — kamu mulai dari TUMPUKAN batu bata yang SUDAH ADA (leaves/token), lalu MENGENALI pola "oh, batu bata-batu bata ini membentuk DINDING" (reduce), terus naik sampai akhirnya kamu bisa bilang "ini semua sekarang jadi satu RUMAH UTUH" (start symbol).

### Syntax Error
**Syntax error** terjadi kalau input MELANGGAR aturan grammar.

**Contoh BENAR:**
```
If ( x > 0 )
     y = 1;
```
**Contoh SALAH** (kurang tanda kurung buka):
```
If  x > 0 )
     y = 1;
```
Parser MENDETEKSI error ini dan MELAPORKANNYA.

### Empat Teknik Penanganan Syntax Error

**1. Panic-Mode Recovery**
- **Deskripsi:** buang simbol input SATU PER SATU sampai bertemu TOKEN SINKRONISASI (misalnya `;`, `}`), lalu lanjutkan parsing.
- **Kelebihan:** sederhana, mudah diimplementasikan, DIJAMIN tidak masuk infinite loop, banyak dipakai di praktik.
- **Kekurangan:** bisa MELEWATKAN bagian besar input, menyebabkan beberapa error TERLEWAT dilaporkan.

**2. Phrase-Level Recovery**
- **Deskripsi:** melakukan koreksi LOKAL terhadap sisa input, seperti menyisipkan, menghapus, atau mengganti token supaya parsing bisa LANJUT.
- **Kelebihan:** bisa pulih dari error sintaks KECIL tanpa melewatkan banyak input.
- **Kekurangan:** memilih perbaikan lokal yang BENAR itu SULIT, bisa menyebabkan pemulihan yang SALAH atau infinite loop kalau desainnya buruk.

**3. Error Productions**
- **Deskripsi:** memperluas grammar dengan production yang MENGENALI kesalahan pemrograman UMUM, memungkinkan parser memberi diagnostik yang lebih BERMAKNA.
- **Kelebihan:** menghasilkan pesan error yang INFORMATIF untuk error yang sudah DIANTISIPASI.
- **Kekurangan:** butuh MENGANTISIPASI kesalahan umum dan menjaga aturan grammar TAMBAHAN.

**4. Global Correction**
- **Deskripsi:** menghitung JUMLAH MINIMUM penyisipan, penghapusan, dan substitusi yang dibutuhkan untuk mengubah input SALAH jadi program VALID.
- **Kelebihan:** secara TEORI menemukan koreksi OPTIMAL.
- **Kekurangan:** SANGAT MAHAL secara komputasi dan umumnya TIDAK PRAKTIS untuk compiler nyata.

> [!info] Analogi
> Empat teknik ini seperti empat cara guru mengoreksi ESAI dengan banyak kesalahan tata bahasa. **Panic-Mode** itu seperti guru yang, begitu nemu kalimat rusak, langsung LONCAT ke tanda baca berikutnya (titik) dan lanjut baca dari situ — cepat tapi mungkin melewatkan detail kesalahan lain di tengah. **Phrase-Level** itu seperti guru yang mencoba MEMPERBAIKI sendiri kalimat yang rusak di tempat (menambah/menghapus kata) supaya bisa lanjut membaca esainya secara utuh. **Error Productions** itu seperti guru yang SUDAH HAFAL pola kesalahan umum murid-muridnya (misalnya "sering lupa 's' di kata kerja") dan langsung kasih catatan SPESIFIK untuk pola itu. **Global Correction** itu seperti guru yang menghitung PERSIS berapa PERUBAHAN MINIMAL yang dibutuhkan supaya esai itu jadi SEMPURNA secara tata bahasa — TEORINYA paling akurat, tapi PRAKTIKNYA terlalu makan waktu untuk dilakukan tiap esai.

## Diagram & Visual
- **Slide 5 — Ilustrasi peran parser (menerima token, cek grammar, bangun tree, laporkan error)**
  ![[99-Assets/Compiler/W09-slide05.png]]

## Rumus / Sintaks
```
Leftmost Derivation:  selalu ekspansi non-terminal PALING KIRI
Rightmost Derivation: selalu ekspansi non-terminal PALING KANAN

Top-Down:  Start Symbol -> ... -> Input String (prediksi, pakai LMD)
Bottom-Up: Input String -> ... -> Start Symbol (reduksi, kebalikan RMD)

4 Teknik Error Recovery:
1. Panic-Mode Recovery    -> buang input sampai token sinkronisasi
2. Phrase-Level Recovery  -> koreksi lokal (insert/delete/replace)
3. Error Productions      -> grammar diperluas kenali kesalahan umum
4. Global Correction      -> minimal edit distance (teoretis optimal, mahal)
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Sentential form** | String hasil derivasi parsial (campuran terminal/non-terminal) |
| **Handle (bottom-up)** | Substring yang dikenali dan direduksi jadi non-terminal dalam bottom-up parsing |
| **Synchronizing token** | Token penanda (seperti `;` atau `}`) tempat panic-mode recovery melanjutkan parsing |
| **Shift dan Reduce** | Dua aksi dasar dalam bottom-up parsing: geser token / reduksi jadi non-terminal |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa Top-Down Parsing "TIDAK BISA menangani grammar left-recursive", sementara Bottom-Up BISA. Hubungkan dengan cara kerja Top-Down yang MEMPREDIKSI produksi (ekspansi dari non-terminal) — kenapa prediksi ini gagal total kalau non-terminal langsung memanggil dirinya sendiri di posisi pertama (ingat [[W08 - Syntax Analysis - Parsing Fundamentals]])?
2. **(C4 – Analisis)** Bandingkan Panic-Mode Recovery dan Global Correction dari sisi TRADE-OFF akurasi vs biaya komputasi. Analisis: kenapa HAMPIR SEMUA compiler produksi (GCC, Clang, dll.) memilih pendekatan yang LEBIH DEKAT ke Panic-Mode/Phrase-Level daripada Global Correction, meski Global Correction "secara teori optimal"?
3. **(C5 – Evaluasi)** Evaluasi klaim "Bottom-Up Parsing biasanya mendeteksi error SETELAH cukup banyak input dibaca" dibanding Top-Down yang "bisa mendeteksi error lebih AWAL". Untuk EDITOR KODE MODERN yang butuh feedback error SECEPAT MUNGKIN saat mengetik (real-time), evaluasi mana pendekatan yang lebih cocok, dan trade-off apa yang harus diterima.
4. **(C5 – Evaluasi)** Bandingkan LMD dan RMD dari sisi HUBUNGANNYA dengan Top-Down dan Bottom-Up parsing. Evaluasi: kenapa Bottom-Up Parsing memakai "KEBALIKAN dari RMD" (bukan RMD secara langsung) — apa yang terjadi kalau kamu MEMBACA proses reduksi bottom-up dari AWAL ke AKHIR dibanding dari AKHIR ke AWAL?
5. **(C6 – Cipta)** Kerjakan latihan dari slide 16: untuk grammar `E→E+T|T`, `T→T*F|F`, `F→(E)|id`, dan string `id + id * id` — (a) buat Leftmost Derivation lengkap, (b) buat Rightmost Derivation lengkap, (c) gambarkan (dalam bentuk teks berindentasi) parse tree-nya, (d) jelaskan apakah grammar ini SUDAH BENAR menegakkan prioritas perkalian (precedence) — buktikan dengan menunjukkan bagaimana struktur pohon MEMAKSA `id * id` dievaluasi sebagai satu kesatuan SEBELUM dijumlahkan dengan `id` pertama.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W08 - Syntax Analysis - Parsing Fundamentals]]
- [[W10 - Pushdown Automata]]
- [[Compiler - Review dan Glosari]]
