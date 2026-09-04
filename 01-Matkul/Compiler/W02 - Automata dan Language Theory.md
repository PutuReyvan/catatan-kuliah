---
matkul: Compilation Techniques
minggu: 2
sks: 3
sumber: Session 02_Automata and Language Theory.pptx
tags: [kuliah/compiler, minggu/w02]
status: draft
diproses: 2026-09-04
---

# W02 — Automata dan Language Theory

## Ringkasan
> - **Automata theory** dipelajari karena jadi MODEL untuk banyak hal: desain sirkuit digital, **lexical analyzer** compiler (topik utama matkul ini!), scanning teks besar, dan verifikasi protokol komunikasi.
> - Dua notasi penting yang BUKAN automaton tapi berperan besar: **Grammar** (untuk struktur data rekursif) dan **Regular Expression** (untuk struktur teks/string).
> - Empat konsep sentral teori automata: **Symbol** (entitas abstrak bermakna) → **Alphabet** (kumpulan simbol) → **String** (daftar simbol dari alphabet) → **Language** (kumpulan string dari alphabet yang sama).
> - **Regular Definition** dipakai saat regular expression jadi terlalu kompleks untuk ditulis langsung — memberi NAMA ke sub-ekspresi, lalu memakai nama itu untuk membangun ekspresi yang lebih besar (contoh klasik: definisi identifier di Pascal).
> - Automata juga jadi dasar studi **decidability** (apa yang BISA dilakukan komputer) dan **intractability** (apa yang bisa dilakukan komputer secara EFISIEN).

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Symbol | Entitas abstrak yang punya makna |
| Alphabet | Kumpulan simbol |
| String | Daftar simbol dari sebuah alphabet |
| Language | Kumpulan string dari alphabet yang sama |
| Regular Expression (RE) | Notasi untuk mendeskripsikan pola string/bahasa |
| Regular Definition | Rangkaian definisi RE bernama, dipakai untuk RE yang kompleks |

## Isi

### Kenapa Mempelajari Automata Theory?
Automata bisa dipakai sebagai MODEL untuk:
- Software mendesain dan mengecek perilaku sirkuit digital.
- **"Lexical analyzer" compiler** — komponen compiler yang memecah teks input jadi UNIT LOGIS seperti identifier, keyword, dan tanda baca. *(Ini akan jadi topik utama tiga minggu berikutnya: [[W03 - Lexical Analysis]], [[W04 - DFA dan NFA]], [[W05 - RE ke DFA]].)*
- Software untuk memindai TEKS BESAR (misalnya kumpulan halaman web) untuk menemukan kemunculan kata, frasa, atau pola lain.
- Software untuk VERIFIKASI berbagai jenis sistem yang punya jumlah state TERBATAS, seperti protokol komunikasi atau protokol pertukaran informasi aman.

### Structural Representation
Ada dua notasi PENTING yang BUKAN automaton, tapi berperan besar dalam studi automata dan aplikasinya:
- **Grammar** — model berguna untuk mendesain software yang memproses data dengan struktur REKURSIF. *(Dibahas mendalam di [[W07 - Context-Free Grammar]].)*
- **Regular Expression** — juga mendenotasikan struktur data, terutama string TEKS.

### Automata dan Kompleksitas
Automata ESENSIAL untuk mempelajari BATAS kemampuan komputasi:
- **Decidability** — studi tentang apa yang bisa DILAKUKAN komputer sama sekali. Masalah yang bisa diselesaikan komputer disebut **decidable**.
- **Intractability** — studi tentang apa yang bisa dilakukan komputer secara EFISIEN. Masalah yang bisa diselesaikan komputer memakai waktu tidak lebih dari fungsi yang tumbuh LAMBAT terhadap ukuran input disebut **tractable**.

### Definisi Formal dari Language dan Grammar
Secara linguistik-matematis: bahasa FORMAL dan bahasa NATURAL harus punya transisi yang HALUS/berjalan baik — misalnya compiler. **Formal Language** = bahasa yang sudah termasuk ekspresi linguistik matematika (contoh: C++, Java, Delphi, dll). Bandingkan dengan bahasa natural, misalnya kata dalam Bahasa Indonesia.

### Konsep Sentral Automata Theory
Empat definisi istilah PALING PENTING yang meresap ke seluruh teori automata:
1. **Symbol** — entitas abstrak yang punya makna.
2. **Alphabet** — kumpulan simbol.
3. **String** — daftar simbol dari sebuah alphabet.
4. **Language** — kumpulan string dari alphabet yang SAMA.

> [!info] Analogi
> Bayangkan empat konsep ini seperti hierarki dari HURUF ke KALIMAT. **Symbol** itu seperti satu HURUF ('a', 'b', '1'). **Alphabet** itu seperti SELURUH huruf yang tersedia di keyboard-mu (misalnya {a, b, c, ..., z}). **String** itu seperti SATU KATA yang tersusun dari huruf-huruf itu (misalnya "cab"). **Language** itu seperti KAMUS yang berisi SEKUMPULAN kata yang valid dari alphabet itu (misalnya semua kata yang diawali huruf 'a'). Compiler harus memahami keempat lapisan ini untuk mengenali apakah sebuah program "berbicara dalam bahasa yang benar".

> [!info] Konteks tambahan (bukan dari slide)
> Slide 13-15 ("Language Operation") dan slide 16-19 (Closure, RE, Regular Definition karakteristiknya) tidak berhasil diekstrak sebagai teks/gambar dari file sumber — kemungkinan diagram/tabel yang tidak berupa gambar raster. Berikut adalah kerangka STANDAR yang lazim dipakai dalam teori automata untuk operasi bahasa (perlu diverifikasi ulang lewat gambar slide asli untuk memastikan notasi PERSIS yang dipakai dosen):
> - **Union (r|s atau r+s)** — semua string yang ada di bahasa r ATAU bahasa s.
> - **Concatenation (rs)** — semua string yang terbentuk dari menggabungkan satu string dari r diikuti satu string dari s.
> - **Kleene Closure (r\*)** — nol atau lebih pengulangan string dari r (termasuk string kosong ε).
> - **Positive Closure (r+)** — SATU atau lebih pengulangan string dari r (TIDAK termasuk string kosong, beda dari Kleene Closure).

### Regular Expression (RE)
Slide secara eksplisit mencatat perbedaan NOTASI antar textbook: **`+` dalam `(00+11)*` punya makna SAMA dengan operator `|` (OR)** — `+` adalah notasi yang dipakai buku Hopcroft, sementara buku Aho memakai `|` sebagai notasi OR. Ini poin penting karena bisa membingungkan kalau membaca dua sumber berbeda.

Untuk BAHASA yang EKUIVALEN, cara menulis RE-nya **TIDAK UNIK** — ada banyak cara berbeda menulis RE yang menghasilkan bahasa yang sama persis.

### Regular Definition
Menulis regular expression untuk beberapa bahasa bisa SULIT, karena RE-nya bisa jadi sangat KOMPLEKS. Dalam kasus itu, kita bisa memakai **regular definition** — memberi NAMA ke regular expression, dan memakai nama itu sebagai simbol untuk mendefinisikan regular expression LAIN.

**Format regular definition** — rangkaian definisi bentuk:
```
d1 → r1     dimana di adalah nama yang berbeda
d2 → r2     dan ri adalah regular expression di atas simbol
...          dalam {d1, d2, ..., di-1}
dn → rn      (simbol dasar atau nama yang sudah didefinisikan sebelumnya)
```

**Contoh: Identifier di Pascal**
```
id      → letter (letter | digit)*
letter  → A | B | ... | Z | a | b | ... | z
digit   → 0 | 1 | ... | 9
```
Kalau kita coba menulis regular expression untuk identifier TANPA memakai regular definition, RE-nya akan jadi KOMPLEKS:
```
(A|...|Z|a|...|z) ( (A|...|Z|a|...|z) | (0|...|9) )*
```

**Contoh: Unsigned Number di Pascal/Delphi**
```
unsigned-num  → digits opt-fraction opt-exponent
digit         → 0 | 1 | ... | 9
digits        → digit+
opt-fraction  → ( . digits )?
opt-exponent  → ( E (+|-)? digits )?
```

> [!info] Analogi
> Regular Definition itu seperti membuat SUB-RESEP dalam sebuah buku masak besar. Alih-alih menulis ULANG cara membuat "kaldu dasar" setiap kali resep membutuhkannya, kamu tulis SEKALI di awal ("Kaldu Dasar = ..."), lalu di resep-resep lain tinggal tulis "gunakan Kaldu Dasar" tanpa mengulang detailnya. `id → letter (letter | digit)*` itu seperti resep yang MEMANGGIL "sub-resep" `letter` dan `digit` yang sudah didefinisikan sebelumnya — jauh lebih rapi dan mudah dibaca daripada menulis SEMUA huruf dan angka berulang-ulang dalam satu ekspresi raksasa.

## Diagram & Visual
> [!warning] Slide 10-19 (diagram Central Concepts, Language Operation, Closure, RE, dan Characteristic of RE) tidak berhasil diekstrak sebagai gambar/teks — kemungkinan berupa SmartArt atau shape yang bukan format gambar raster biasa. Buka file PPT asli untuk melihat notasi dan diagram PERSIS yang dipakai dosen, terutama untuk memverifikasi operasi Union/Concatenation/Closure di atas.

## Rumus / Sintaks
```
Format Regular Definition:
d1 -> r1
d2 -> r2
...
dn -> rn

Contoh - Identifier Pascal:
id      -> letter (letter | digit)*
letter  -> A | B | ... | Z | a | b | ... | z
digit   -> 0 | 1 | ... | 9

Contoh - Unsigned Number Pascal/Delphi:
unsigned-num  -> digits opt-fraction opt-exponent
digits        -> digit+
opt-fraction  -> ( . digits )?
opt-exponent  -> ( E (+|-)? digits )?
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Decidability** | Studi tentang apa yang bisa/tidak bisa diselesaikan komputer sama sekali |
| **Intractability** | Studi tentang apa yang bisa diselesaikan komputer secara EFISIEN |
| **Kleene Closure (\*)** | Operasi nol atau lebih pengulangan sebuah pola |
| **ε (epsilon)** | Simbol yang merepresentasikan string kosong |

## Pertanyaan Terbuka
- Slide 10-19 (Central Concepts lanjutan, Language Operation, Closure Language, RE, Characteristic of RE) gagal diekstrak — perlu dibuka manual untuk detail notasi PERSIS operasi bahasa yang dipakai dosen (union, concatenation, closure).

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa Lexical Analyzer compiler disebut sebagai salah satu APLIKASI PALING LANGSUNG dari automata theory. Hubungkan dengan konsep Symbol-Alphabet-String-Language — bagaimana source code sebuah program bisa dipandang sebagai "string" dari sebuah "language" tertentu?
2. **(C4 – Analisis)** Bandingkan definisi identifier Pascal DENGAN regular definition (`id -> letter (letter|digit)*`) dan TANPA regular definition (`(A|...|Z|a|...|z)((A|...|Z|a|...|z)|(0|...|9))*`). Analisis: selain lebih PENDEK, keuntungan APA lagi yang didapat dari memakai regular definition, terutama kalau definisi `letter` atau `digit` perlu DIUBAH di kemudian hari?
3. **(C5 – Evaluasi)** Evaluasi klaim slide "untuk bahasa yang ekuivalen, cara menulis RE tidak unik". Buktikan klaim ini dengan menulis DUA regular expression BERBEDA yang keduanya mendeskripsikan bahasa yang SAMA: "string biner yang panjangnya genap" — lalu jelaskan kenapa keduanya ekuivalen meski notasinya berbeda.
4. **(C5 – Evaluasi)** Bandingkan konsep Decidability dan Intractability. Evaluasi: sebuah masalah bisa saja DECIDABLE (bisa diselesaikan komputer) tapi TIDAK TRACTABLE (butuh waktu sangat lama). Berikan CONTOH skenario nyata (bisa dari luar compiler) di mana perbedaan ini penting dipahami oleh seorang engineer.
5. **(C6 – Cipta)** Kerjakan latihan dari slide 22: tulis regular expression untuk "himpunan string atas alphabet {a, b} yang DIAWALI dengan 'abb'". Lalu rancang SATU regular definition (mengikuti format `d1 -> r1`) untuk mendefinisikan pola "nomor telepon Indonesia sederhana" (format: diawali "08", diikuti 8-11 digit angka).

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W01 - Introduction to Compiler]]
- [[W03 - Lexical Analysis]]
- [[Compiler - Review dan Glosari]]
