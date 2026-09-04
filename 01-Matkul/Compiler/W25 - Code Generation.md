---
matkul: Compilation Techniques
minggu: 25
sks: 3
sumber: Session 25_Code Generation.pptx
tags: [kuliah/compiler, minggu/w25]
status: draft
diproses: 2026-09-04
---

# W25 — Code Generation

## Ringkasan
> - **Code generator** adalah tahap TERAKHIR compiler — input-nya intermediate representation, output-nya PROGRAM TARGET yang EKUIVALEN. Secara MATEMATIS, menghasilkan kode OPTIMAL adalah masalah **UNDECIDABLE** — dalam praktik, dipakai teknik HEURISTIK yang bagus tapi TIDAK PASTI optimal.
> - **6 isu utama** desain code generator: Input, Target Program, Memory Management, Instruction Selection, Register Allocation, Order of Evaluation.
> - Target program bisa berupa **Absolute Machine Language** (langsung eksekusi, tapi tidak fleksibel), **Relocatable Machine Language** (bisa di-link terpisah, LEBIH fleksibel), atau **Assembly Language** (paling mudah dihasilkan, tapi butuh tahap assembly TAMBAHAN).
> - **Register Allocation** dan **Order of Evaluation** SAMA-SAMA masalah **NP-Complete** — mencari alokasi register OPTIMAL, bahkan dengan SATU register saja, secara matematis SULIT dipecahkan secara efisien.
> - **Basic block bisa direpresentasikan sebagai DAG** (mirip [[W21 - Intermediate Code Generator]]) untuk mengidentifikasi common subexpression dan menentukan urutan generasi kode yang lebih baik.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Code generator | Tahap TERAKHIR compiler, menerjemahkan intermediate code jadi target program |
| Absolute Machine Language | Target berupa kode mesin siap eksekusi langsung di alamat TETAP |
| Relocatable Machine Language | Target berupa object module yang bisa di-LINK terpisah |
| Register Allocation | Memilih variabel mana yang DITEMPATKAN di register |
| Register Assignment | Menentukan REGISTER SPESIFIK mana untuk variabel yang sudah dipilih |
| Activation Record | Struktur memori untuk menyimpan data satu pemanggilan fungsi/prosedur |

## Isi

### Code Generation: Tahap Terakhir Compiler
Tahap TERAKHIR compiler adalah **code generator**. Input-nya adalah intermediate representation source program, dan output-nya adalah program TARGET yang EKUIVALEN.

### Syarat Code Generator
Kode output HARUS BENAR (correct) dan BERKUALITAS TINGGI — harus memakai sumber daya mesin target secara EFEKTIF, harus EFISIEN. **Secara matematis, masalah menghasilkan kode OPTIMAL adalah UNDECIDABLE.** Dalam PRAKTIK, dipakai TEKNIK HEURISTIK yang menghasilkan kode BAGUS, tapi TIDAK NECESSARILY optimal.

### Enam Isu Desain Code Generator
Meski SANGAT bergantung pada mesin target dan sistem operasi, isu PENTING dalam code generation:
1. **Input untuk code generator.**
2. **Target program.**
3. **Memory Management.**
4. **Instruction Selection.**
5. **Register Allocation.**
6. **Order Selection Evaluation.**
7. **Approach Code Generator (pendekatan desain).**

### Input untuk Code Generator
Input code generator terdiri dari intermediate representation source program yang dihasilkan front-end, PLUS informasi di symbol table yang dipakai untuk menentukan alamat RUN-TIME dari data object yang ditandai nama-nama di intermediate representation. **DIASUMSIKAN sebelum code generation:** front-end SUDAH melakukan scan, parse, dan translate source program jadi intermediate representation DETAIL. SELAIN itu, diasumsikan type checking SUDAH dilakukan — operator konversi tipe SUDAH disisipkan (kalau perlu), dan error semantik SUDAH terdeteksi.

### Target Program
Output code generator bisa berbentuk: **Absolute Machine Language, Relocatable Machine Language,** atau **Assembly Language**.

- **Absolute Machine Language** — punya keuntungan bisa DITEMPATKAN di lokasi TETAP di memori dan bisa DIEKSEKUSI LANGSUNG. Program KECIL bisa dikompilasi dan dijalankan CEPAT. Contoh: compiler "student-jobs" seperti WATFIV dan PL/C.
- **Relocatable Machine Language (Object Module)** — sub-program bisa dikompilasi TERPISAH. Sekumpulan relocatable object module bisa DIGABUNG (link) dan DIMUAT (load) oleh linking loader untuk dieksekusi. Meski butuh USAHA TAMBAHAN untuk linking & loading, kita punya FLEKSIBILITAS LEBIH karena bisa kompilasi subroutine SECARA TERPISAH.
- **Assembly Language** sebagai program target punya keuntungan KEMUDAHAN proses code generation — kita bisa hasilkan instruksi SIMBOLIK dan memakai fasilitas MACRO assembler untuk menghasilkan kode. Biayanya: butuh TAHAP TAMBAHAN assembly setelah code generation.

### Memory Management
Pemetaan NAMA di source program ke ALAMAT data object di run-time memory dilakukan BERSAMA oleh front-end DAN code generator. Diasumsikan NAMA di three-address statement MERUJUK entri symbol-table untuk nama itu. TIPE di deklarasi menentukan WIDTH (jumlah storage) yang dibutuhkan nama tersebut.

**Konsep terkait:** register **R0, R1, ..., Rn-1**; format instruksi `op source, destination`; contoh instruksi: **MOV** (move source to destination), **ADD** (tambah source ke destination), **SUB** (kurangi source dari destination); **address mode; instruction cost.**

### Instruction Selection
**Keseragaman dan kelengkapan instruction set** adalah faktor PENTING. Faktor penting LAIN: kecepatan dan IDIOM instruksi mesin.

**Contoh:** three-address statement `x := y + z` (x,y,z dialokasikan STATIS) bisa diterjemahkan jadi:
```
MOV y, R0    // load y ke register R0
ADD z, R0    // tambahkan z ke R0
MOV R0, x    // simpan R0 ke x
```

**Masalah code generation statement-per-statement:** SERING menghasilkan kode BURUK. Contoh: statement `a:=b+c`, `d:=a+e` diterjemahkan jadi:
```
MOV b, R0
ADD c, R0
MOV R0, a
MOV a, R0    <- REDUNDAN (nilai a masih di R0 dari instruksi sebelumnya!)
ADD e, R0
MOV R0, d
```
**Statement KETIGA dan KEEMPAT REDUNDAN** — begitu juga statement ketiga kalau `a` TIDAK dipakai lagi setelahnya. Instruksi `MOV R0, a` diikuti langsung `MOV a, R0` itu KONYOL — nilainya SUDAH ada di R0!

> [!info] Analogi
> Masalah code generation statement-per-statement itu seperti menulis surat SATU KALIMAT PER WAKTU tanpa melihat konteks kalimat sebelumnya — kamu bisa saja menulis "aku menaruh kunci di saku" lalu SEGERA setelahnya menulis "aku ambil kunci dari sakuku" hanya karena kamu LUPA kamu BARU SAJA menaruhnya di sana. Melihat KONTEKS beberapa kalimat SEKALIGUS (bukan satu per satu) memungkinkan kamu menyadari "oh, aku TIDAK PERLU mengambilnya lagi, kuncinya SUDAH di tanganku" — persis seperti bagaimana code generator yang BAIK harus mempertimbangkan BEBERAPA statement sekaligus, bukan menerjemahkan satu-satu secara buta.

### Register Allocation
Instruksi yang melibatkan operand REGISTER biasanya LEBIH PENDEK dan LEBIH CEPAT daripada yang melibatkan operand di MEMORI. Karena itu, pemakaian register yang EFISIEN SANGAT PENTING untuk menghasilkan kode BAGUS.

**Pemakaian register dibagi 2 sub-masalah:**
1. **Register Allocation** — memilih VARIABEL MANA yang akan ditempatkan di register pada TITIK tertentu dalam program.
2. **Register Assignment** — menentukan REGISTER SPESIFIK MANA yang dipakai variabel tersebut.

**Menemukan alokasi register OPTIMAL itu SULIT, bahkan dengan HANYA SATU register. Secara matematis, masalah ini NP-Complete.**

### Order Selection Evaluation
URUTAN melakukan komputasi bisa memengaruhi EFISIENSI kode target. Beberapa urutan komputasi butuh LEBIH SEDIKIT register untuk menyimpan hasil INTERMEDIATE dibanding urutan lain. **Mengambil urutan TERBAIK adalah masalah NP-Complete.**

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa Register Allocation dan Order Evaluation SAMA-SAMA NP-Complete? Karena KEDUANYA pada dasarnya adalah masalah OPTIMASI KOMBINATORIAL — mencari SUSUNAN terbaik dari SEKIAN BANYAK kemungkinan (kombinasi variabel-ke-register, atau urutan eksekusi instruksi) yang MEMINIMALKAN sumber daya yang dipakai. Inilah kenapa compiler PRAKTIS TIDAK mencoba mencari solusi SEMPURNA — mereka memakai HEURISTIK (seperti "graph coloring" untuk register allocation) yang menghasilkan solusi BAGUS dalam waktu WAJAR, meski TIDAK dijamin optimal secara matematis.

### Approach Code Generator
Kriteria PALING PENTING untuk code generator: MENGHASILKAN KODE BAGUS. Salah satu tujuan desain PENTING Code Generator:
- MUDAH diimplementasikan.
- MUDAH ditest.
- MUDAH dipelihara (maintain).

### Basic Block sebagai DAG
Seperti ekspresi (ingat [[W21 - Intermediate Code Generator]]), basic block bisa direpresentasikan sebagai DAG:
1. Buat NODE untuk nilai AWAL setiap variabel yang muncul di block.
2. Untuk setiap statement s, buat node N dengan CHILDREN adalah node yang mendefinisikan OPERAND s.
3. Setiap node N diberi LABEL operasi yang dipakai.
4. Setiap node N punya DAFTAR variabel yang DIDEFINISIKANNYA.
5. Node N ditandai sebagai OUTPUT NODE kalau nilainya DIPAKAI di LUAR basic block itu.

### Run-time Storage: Static vs Stack Allocation
Dua strategi alokasi: **Static** dan **Stack**.

**Static Allocation** — call statement diterjemahkan jadi:
```
MOVE #here+20, callee.static_area
GOTO callee.code_area
```

**Contoh three-address code dan Activation Record:**
```
/* code for c */
action1
call p
action2
halt

/* code for p */
action3
return
```

**Activation Record** — struktur memori untuk MENYIMPAN data satu pemanggilan fungsi/prosedur. Contoh:
```
Activation Record untuk c (64 bytes):
0: return address
8: arr
...
56: i
60: j

Activation Record untuk p (88 bytes):
0: return address
4: buf
...
84: n
```

## Diagram & Visual
- **Slide 4 — Posisi Code Generator dalam Pipeline Compiler**
  ![[99-Assets/Compiler/W25-slide04.png]]
- **Slide 17 — Contoh Representasi DAG untuk Basic Block**
  ![[99-Assets/Compiler/W25-slide17.png]]

## Rumus / Sintaks
```
Format instruksi target sederhana: op source, destination
Contoh: x := y + z  =>
  MOV y, R0
  ADD z, R0
  MOV R0, x

Static Allocation Call Statement:
  MOVE #here+20, callee.static_area
  GOTO callee.code_area

3 Jenis Target Program:
  Absolute Machine Language      -> siap eksekusi langsung, tidak fleksibel
  Relocatable Machine Language   -> object module, bisa link terpisah (fleksibel)
  Assembly Language              -> mudah dihasilkan, butuh tahap assembly tambahan
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **NP-Complete** | Kelas masalah komputasi yang secara umum sulit dipecahkan secara efisien |
| **Machine idiom** | Pola instruksi khas sebuah mesin yang lebih efisien dari pola umum |
| **Address mode** | Cara instruksi mesin menentukan lokasi operand (langsung, tidak langsung, dll.) |
| **Linking loader** | Program yang menggabungkan (link) dan memuat (load) object module untuk eksekusi |
| **Static area / Code area** | Wilayah memori untuk data statis dan kode program pada activation record |

## Pertanyaan Terbuka
- Slide tidak menjelaskan detail ALGORITMA heuristik konkret untuk Register Allocation (misalnya graph coloring) — hanya menyebutkan masalahnya NP-Complete. Perlu dicek dari sumber tambahan kalau butuh algoritma detail untuk ujian.

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa code generation "statement-per-statement" (menerjemahkan satu instruksi three-address code jadi kode target TANPA melihat konteks statement lain) SERING menghasilkan kode BURUK. Ambil contoh `a:=b+c`, `d:=a+e` — tunjukkan instruksi PERSIS mana yang REDUNDAN dan JELASKAN kenapa compiler perlu melihat LEBIH DARI SATU statement sekaligus untuk menghindarinya.
2. **(C4 – Analisis)** Bandingkan Absolute Machine Language dan Relocatable Machine Language sebagai target program. Analisis: TRADE-OFF apa yang harus DITERIMA saat memilih Relocatable (fleksibilitas kompilasi terpisah) dibanding Absolute (kecepatan kompilasi-eksekusi langsung)?
3. **(C5 – Evaluasi)** Evaluasi klaim "menemukan alokasi register optimal itu NP-Complete, bahkan dengan HANYA SATU register". Kaitkan dengan Order Selection Evaluation yang JUGA NP-Complete — jelaskan kenapa compiler PRODUKSI (GCC, Clang, dll.) TIDAK mencoba mencari solusi SEMPURNA, dan strategi APA yang MASUK AKAL sebagai gantinya (petunjuk: heuristik).
4. **(C5 – Evaluasi)** Bandingkan Basic Block sebagai DAG (materi ini) dengan DAG untuk ekspresi (dari [[W21 - Intermediate Code Generator]]). Evaluasi: APA MANFAAT TAMBAHAN yang didapat dari merepresentasikan SELURUH basic block sebagai DAG (bukan cuma satu ekspresi) — bagaimana ini membantu code generator MENENTUKAN urutan generasi kode yang LEBIH BAIK?
5. **(C6 – Cipta)** Rancang instruksi target (format `op source, destination`, mengikuti contoh MOV/ADD/SUB di materi) untuk menerjemahkan three-address code berikut, dengan MENGHINDARI instruksi redundan seperti pada contoh `a:=b+c; d:=a+e`:
   ```
   t1 = x + y
   t2 = t1 * 2
   z = t2 - w
   ```
   Tunjukkan urutan instruksi target LENGKAP (pakai register R0, R1 secukupnya) dan JELASKAN keputusan register apa saja yang kamu ambil untuk MEMINIMALKAN jumlah instruksi MOV yang tidak perlu.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W23 - Code Optimization]]
- [[W26 - Review II]]
- [[Compiler - Review dan Glosari]]
