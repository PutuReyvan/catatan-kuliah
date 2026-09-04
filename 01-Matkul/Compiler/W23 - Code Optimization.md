---
matkul: Compilation Techniques
minggu: 23
sks: 3
sumber: Session 23-24_Control Flow & Basic Blocks + Code Optimization.pptx
tags: [kuliah/compiler, minggu/w23]
status: draft
diproses: 2026-09-04
---

# W23 — Code Optimization

> [!note] Deck ini secara eksplisit menggabungkan **dua sesi (Session 23 dan 24)**: Control Flow & Basic Blocks, dan Code Optimization. Note ini tetap dinomori W23 mengikuti konvensi vault untuk deck gabungan multi-sesi.

## Ringkasan
> - **Code optimization** harus memenuhi 3 kriteria: MAKNA program TETAP SAMA (correctness), harus ada PERCEPATAN rata-rata, dan HASILNYA harus SEBANDING dengan usaha optimisasinya. Peluang optimisasi ada di 3 level: programmer, intermediate code, target code.
> - **Peephole Optimization** — teknik LOKAL sederhana: periksa POTONGAN PENDEK instruksi target, ganti dengan versi lebih PENDEK/CEPAT. Mencakup: Constant Folding, Unreachable Code elimination, Flow-of-control optimization, Algebraic Simplification, Dead Code elimination, Reduction in Strength.
> - Di level **Basic Block**: Common Subexpression Elimination, Constant Propagation, Copy Propagation, Dead Code Elimination, Code Motion.
> - Optimisasi di **CFG (Control Flow Graph)** HARUS mempertimbangkan ALUR KONTROL antar basic block (bukan cuma dalam SATU block) — dan MENERAPKAN satu optimisasi bisa MEMBUKA PELUANG untuk optimisasi lain.
> - **Loop Optimization: Code Motion** — memindahkan hitungan yang NILAINYA TIDAK BERUBAH (loop invariant) KELUAR dari loop, supaya TIDAK dihitung ULANG di SETIAP iterasi.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Peephole Optimization | Optimisasi lokal dengan memeriksa potongan pendek instruksi target |
| Constant Folding | Menghitung ekspresi konstan di compile-time, bukan run-time |
| Common Subexpression Elimination | Menghindari menghitung ULANG sub-ekspresi yang sama |
| Dead Code Elimination | Menghapus kode yang HASILNYA tidak pernah dipakai |
| Reduction in Strength | Mengganti operasi MAHAL dengan operasi setara yang LEBIH MURAH |
| Code Motion | Memindahkan perhitungan loop-invariant KELUAR dari loop |

## Isi

### Kriteria Transformasi Peningkatan Kode
**Criteria for Code-Improving Transformation:**
1. **Meaning must be preserved (correctness)** — MAKNA program TIDAK BOLEH berubah.
2. **Speedup must occur ON AVERAGE** — harus ada percepatan RATA-RATA (tidak harus SETIAP kasus, tapi secara UMUM).
3. **Work done must be worth the effort** — usaha OPTIMISASI harus SEBANDING hasilnya (jangan optimisasi RUMIT untuk penghematan yang TIDAK SIGNIFIKAN).

**Peluang optimisasi ada di 3 tingkat:**
1. **Programmer** — lewat pemilihan ALGORITMA yang lebih baik, atau DIRECTIVE khusus.
2. **Intermediate code.**
3. **Target code.**

### Peephole Optimization
Teknik SEDERHANA tapi EFEKTIF untuk meningkatkan target code SECARA LOKAL. Metode untuk MENCOBA meningkatkan performa program target dengan MEMERIKSA potongan PENDEK instruksi target dan MENGGANTI instruksi ini dengan urutan yang lebih PENDEK atau LEBIH CEPAT, KAPAN PUN memungkinkan.

**Karakteristik Peephole Optimization:**
1. Redundant instruction elimination.
2. Flow of control information.
3. Algebraic Simplification.
4. Use of machine idioms.

**1. Constant Folding** — menghitung ekspresi KONSTAN di compile-time:
```
x := 32          menjadi          x := 64
x := x + 32
```

**2. Unreachable Code** — kode yang TIDAK PERNAH BISA dijalankan dihapus:
```
goto L2
x := x + 1    // TIDAK PERLU — tidak pernah dijalankan (setelah goto tanpa target L1 ke sini)
```

**3. Flow of control optimizations**:
```
goto L1          menjadi           goto L2
...
L1: goto L2       // "goto L1 lalu L1 goto L2" disederhanakan LANGSUNG jadi "goto L2"
                  // TIDAK DIBUTUHKAN kalau tidak ada branch lain ke L1
```

**4. Algebraic Simplification:**
```
x := x + 0    // TIDAK DIBUTUHKAN — penjumlahan dengan 0 tidak mengubah nilai
```

**5. Dead Code** — kode yang hasilnya TIDAK PERNAH dipakai lagi:
```
x := 32                di mana x TIDAK dipakai setelah statement ini
y := x + y      menjadi        y := y + 32
```

**6. Reduction in Strength** — mengganti operasi MAHAL dengan operasi SETARA yang lebih MURAH:
```
x := x * 2      menjadi        x := x + x
                atau           x := x << 2
```

> [!info] Analogi
> Peephole Optimization itu seperti mengedit ESAI dengan membaca lewat "LUBANG KUNCI" kecil — kamu HANYA melihat BEBERAPA KATA sekaligus (bukan seluruh esai), tapi bisa langsung mengenali dan memperbaiki pola BOROS yang UMUM: kalimat berulang yang tidak perlu (redundant instruction), kalimat yang TIDAK PERNAH DIBACA siapa pun karena letaknya setelah kesimpulan (unreachable code), atau frasa "menambahkan nol" secara harfiah yang bisa dihapus (algebraic simplification). Kamu tidak perlu paham SELURUH esai untuk menemukan perbaikan LOKAL macam ini.

### Optimisasi di Level Basic Block
1. **Common Subexpression Elimination** — hindari menghitung ULANG sub-ekspresi yang SAMA.
2. **Constant Propagation** — GANTI variabel dengan nilai KONSTANnya kalau nilainya sudah DIKETAHUI compile-time.
3. **Copy Propagation** — GANTI penggunaan variabel HASIL COPY dengan variabel ASLINYA.
4. **Dead Code Elimination** — hapus kode yang HASILNYA tidak pernah dipakai.
5. **Code Motion** — pindahkan perhitungan yang bisa DILAKUKAN LEBIH AWAL (di luar loop) keluar dari tempat berulang.

**Contoh Common Subexpression Elimination:** `a[i+1] = b[i+1]`
```
Sebelum:              Sesudah:
t1 = i+1              t1 = i+1
t2 = b[t1]            t2 = b[t1]
t3 = i+1     <- SAMA seperti t1!   (dihapus, t3 tidak dipakai lagi)
a[t3] = t2            a[t1] = t2   <- pakai t1 langsung
```

**Contoh Constant Propagation** (setelah diketahui `i` adalah konstan `4`):
```
i = 4
t1 = i+1    -> t1 = 5           (i diganti nilai konstannya)
t2 = b[t1]  -> t2 = b[5]        (t1 diganti nilai konstannya)
a[t1] = t2  -> a[5] = t2
```
**Kode Final** (setelah SEMUA optimisasi diterapkan):
```
i = 4
t2 = b[5]
a[5] = t2
```

### Optimisasi pada Control Flow Graph (CFG)
Harus MEMPERHITUNGKAN alur kontrol (bukan hanya dalam satu basic block). Jenis optimisasi yang relevan: **Common Sub-expression Elimination, Constant Propagation, Dead Code Elimination, Partial Redundancy Elimination**, dll. **Menerapkan SATU optimisasi bisa MEMBUKA PELUANG untuk optimisasi LAIN** — optimisasi seringkali dilakukan ITERATIF, berulang sampai TIDAK ADA lagi perubahan yang bisa dibuat.

### Simple Loop Optimization: Code Motion
Pindahkan **invariant** (nilai yang TIDAK BERUBAH selama loop berjalan) KELUAR dari loop.

**Contoh:**
```
Sebelum:                        Sesudah:
while (i <= limit - 2)          t := limit - 2
    ...                         while (i <= t)
                                     ...
```
`limit - 2` DIHITUNG ULANG di SETIAP iterasi loop di versi "sebelum" — padahal nilainya TIDAK PERNAH BERUBAH (kecuali `limit` diubah di dalam loop). Dengan memindahkannya keluar loop (dihitung SEKALI SAJA sebagai `t`), kita menghemat perhitungan berulang yang TIDAK PERLU.

> [!info] Analogi
> Code Motion itu seperti tidak perlu menimbang ULANG berat badan setiap kali kamu mau tahu "apakah aku masih di bawah 70kg?" saat berjalan 100 langkah — kamu TIMBANG SEKALI di awal (`t := limit - 2`, dihitung SEKALI di luar loop), lalu CUKUP BANDINGKAN hasil timbangan itu tiap langkah (`while (i <= t)`), bukan menimbang ULANG dari NOL setiap kali.

### Studi Kasus Lengkap: Optimisasi Quick Sort
Slide menampilkan STUDI KASUS lengkap: three-address code untuk algoritma **Quick Sort** (partition step, 30 baris instruksi), lalu:
1. **Menemukan Basic Block** — membagi kode jadi blok-blok berdasarkan TITIK MASUK/KELUAR (leader statement).
2. **Membangun Flow Graph** — menghubungkan basic block berdasarkan alur EKSEKUSI (termasuk jump dan branch kondisional).
3. **Menerapkan Common Subexpression Elimination** — SECARA BERTAHAP (banyak langkah/iterasi) di sepanjang flow graph, menghilangkan perhitungan `4*i`, `4*j`, `4*n` yang BERULANG.
4. **Dead Code Elimination** — menghapus assignment yang hasilnya TIDAK PERNAH dipakai lagi.
5. **Reduction in Strength** — mengganti perkalian `4*i` (dan sejenisnya) dengan operasi SHIFT yang lebih murah, atau teknik lain untuk menghindari perkalian berulang.

> [!warning] Studi kasus Quick Sort ini (slide 13-31) SANGAT BERGANTUNG pada diagram VISUAL langkah-demi-langkah (flow graph dan urutan transformasi optimisasi) yang HANYA tersedia sebagai GAMBAR di slide, bukan teks. Diagram-diagram ini PENTING untuk memahami PROSES optimisasi secara konkret pada kode nyata — SANGAT DISARANKAN membuka gambar-gambar di bawah SECARA BERURUTAN untuk mengikuti alur transformasinya.

## Diagram & Visual
> [!warning] SEMUA diagram berikut (slide 15-31) awalnya format WMF, sudah dikonversi ke PNG — ini adalah RANGKAIAN VISUAL LENGKAP studi kasus optimisasi Quick Sort, disarankan dibuka BERURUTAN untuk mengikuti proses optimisasi step-by-step.
- **Slide 15 — Flow Graph dari three-address code Quick Sort**
  ![[99-Assets/Compiler/W23-slide15.png]]
- **Slide 16-27 — Rangkaian langkah Common Subexpression Elimination (12 langkah)**
  ![[99-Assets/Compiler/W23-slide16.png]]
  ![[99-Assets/Compiler/W23-slide17.png]]
  ![[99-Assets/Compiler/W23-slide18.png]]
  ![[99-Assets/Compiler/W23-slide19.png]]
  ![[99-Assets/Compiler/W23-slide20.png]]
  ![[99-Assets/Compiler/W23-slide21.png]]
  ![[99-Assets/Compiler/W23-slide22.png]]
  ![[99-Assets/Compiler/W23-slide23.png]]
  ![[99-Assets/Compiler/W23-slide24.png]]
  ![[99-Assets/Compiler/W23-slide25.png]]
  ![[99-Assets/Compiler/W23-slide26.png]]
  ![[99-Assets/Compiler/W23-slide27.png]]
- **Slide 28-29 — Dead Code Elimination**
  ![[99-Assets/Compiler/W23-slide28.png]]
  ![[99-Assets/Compiler/W23-slide29.png]]
- **Slide 30-31 — Reduction in Strength**
  ![[99-Assets/Compiler/W23-slide30.png]]
  ![[99-Assets/Compiler/W23-slide31.png]]

## Rumus / Sintaks
```
3 Kriteria Code-Improving Transformation:
1. Correctness      -> makna program tidak berubah
2. Speedup           -> ada percepatan RATA-RATA
3. Worth the effort  -> hasil sebanding usaha

Teknik Peephole Optimization:
- Constant Folding           : x=32; x=x+32   ->  x=64
- Unreachable Code           : hapus kode setelah goto tanpa branch masuk
- Flow-of-control            : goto L1; L1: goto L2  ->  goto L2
- Algebraic Simplification   : x=x+0  ->  dihapus
- Dead Code                  : x=32 (tak dipakai); y=x+y  ->  y=y+32
- Reduction in Strength      : x=x*2  ->  x=x+x  atau  x=x<<1

Teknik Basic Block Level:
Common Subexpression Elimination, Constant Propagation,
Copy Propagation, Dead Code Elimination, Code Motion

Code Motion (loop invariant):
while (i <= limit-2) { ... }
=>
t := limit-2
while (i <= t) { ... }
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Basic block** | Urutan instruksi berurutan tanpa percabangan masuk/keluar di tengahnya |
| **Leader statement** | Statement pertama sebuah basic block, penanda batas blok |
| **Flow graph** | Graph yang menghubungkan basic block berdasarkan alur eksekusi program |
| **Loop invariant** | Nilai/ekspresi yang TIDAK BERUBAH selama loop berjalan |
| **Partial Redundancy Elimination** | Teknik menghilangkan komputasi yang redundan di SEBAGIAN (bukan semua) jalur eksekusi |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa "Work done must be worth the effort" jadi salah satu KRITERIA WAJIB code optimization, bukan cuma "Speedup must occur". Berikan CONTOH skenario di mana sebuah optimisasi BISA mempercepat program, tapi TETAP TIDAK LAYAK diterapkan karena kompleksitas implementasi optimizer-nya TIDAK SEBANDING dengan penghematan yang didapat.
2. **(C4 – Analisis)** Bandingkan Common Subexpression Elimination dan Dead Code Elimination. Analisis: KEDUANYA sama-sama "menghapus" sesuatu dari kode, tapi ALASAN penghapusannya BERBEDA — jelaskan perbedaan MENDASAR antara "menghapus karena SUDAH DIHITUNG sebelumnya" (CSE) vs "menghapus karena HASILNYA TIDAK PERNAH DIPAKAI" (Dead Code).
3. **(C5 – Evaluasi)** Evaluasi teknik Reduction in Strength (`x*2` menjadi `x+x` atau `x<<1`). Untuk PROSESOR MODERN (yang sering punya instruksi perkalian HARDWARE yang sudah SANGAT CEPAT), evaluasi apakah optimisasi ini MASIH RELEVAN — pertimbangkan bahwa optimisasi ini AWALNYA dirancang untuk era di mana operasi perkalian JAUH LEBIH LAMBAT dari penjumlahan/shift.
4. **(C5 – Evaluasi)** Bandingkan Code Motion (memindahkan loop invariant KELUAR loop) dengan Common Subexpression Elimination biasa (dalam SATU basic block). Evaluasi: kenapa Code Motion butuh ANALISIS LEBIH DALAM (harus tahu apakah sebuah ekspresi BENAR-BENAR tidak berubah SELAMA SELURUH iterasi loop, bukan cuma dalam satu blok) dibanding CSE biasa yang cukup melihat DALAM SATU basic block saja?
5. **(C6 – Cipta)** Rancang urutan optimisasi (Constant Folding → Common Subexpression Elimination → Dead Code Elimination → Reduction in Strength) untuk potongan kode berikut, tunjukkan HASIL SETELAH SETIAP TAHAP optimisasi diterapkan:
   ```
   a = 10
   b = a * 2
   c = a + 5
   d = a * 2
   e = c + 1
   f = b + d
   ```
   (petunjuk: `a` konstan, `b` dan `d` adalah ekspresi yang SAMA, dan cek apakah ada variabel yang hasilnya TIDAK PERNAH dipakai lagi setelah dihitung.)

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W21 - Intermediate Code Generator]]
- [[W25 - Code Generation]]
- [[Compiler - Review dan Glosari]]
