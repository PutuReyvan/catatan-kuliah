---
matkul: Compilation Techniques
minggu: 21
sks: 3
sumber: Session 21-22_Intermediate Representation.pptx
tags: [kuliah/compiler, minggu/w21]
status: draft
diproses: 2026-09-04
---

# W21 — Intermediate Code Generator

> [!note] Deck ini secara eksplisit menggabungkan **dua sesi (Session 21 dan 22)**: Intermediate Representation dan Three Address Code. Note ini tetap dinomori W21 mengikuti konvensi vault untuk deck gabungan multi-sesi.

## Ringkasan
> - **Intermediate code** adalah kode INDEPENDEN dari mesin, tapi MENDEKATI instruksi mesin — jembatan antara source language dan target language. Bisa berbentuk **syntax tree/DAG, postfix notation,** atau **three-address code (quadruple)**.
> - **DAG (Directed Acyclic Graph)** MIRIP syntax tree, tapi NODE bisa punya LEBIH DARI SATU parent kalau merepresentasikan SUB-EKSPRESI YANG SAMA — otomatis MENGHILANGKAN redundansi perhitungan.
> - **Three-Address Code** = representasi LINEAR dari syntax tree/DAG, setiap instruksi PALING BANYAK punya 3 operand. Ada 3 cara representasi: **Quadruple** (op, arg1, arg2, result), **Triple** (op, arg1, arg2 — hasil dirujuk lewat POSISI), **Indirect Triple** (triple + array pointer terpisah, mudah di-reorder untuk optimisasi).
> - **Translation Scheme** menerapkan semantic action untuk MENGHASILKAN three-address code dari CFG — termasuk kasus KHUSUS untuk **boolean expression** dan **control flow** (while, if-else) yang butuh LABEL dan kadang **backpatching**.
> - Perhitungan alamat **ARRAY** (1D, 2D, multi-dimensi) BISA DISEDERHANAKAN jadi rumus `i*width + c`, dengan `c` dihitung SEKALI SAJA di COMPILE TIME — trik penting untuk efisiensi kode yang dihasilkan.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Intermediate code | Kode independen-mesin tapi mendekati instruksi mesin |
| DAG (Directed Acyclic Graph) | Seperti syntax tree, tapi node bisa punya banyak parent untuk sub-ekspresi sama |
| Three-Address Code | Representasi linear, tiap instruksi maksimal 3 operand |
| Quadruple | Representasi 3-address code dengan 4 field: op, arg1, arg2, result |
| Triple | Representasi 3-address code dengan 3 field, hasil dirujuk lewat posisi |
| Backpatching | Teknik mengisi target jump YANG BELUM DIKETAHUI setelah kode dihasilkan |

## Isi

### Intermediate Code: Konsep Dasar
**Intermediate code** adalah kode INDEPENDEN-MESIN, tapi DEKAT dengan instruksi mesin. Program dalam source language DIKONVERSI jadi program EKUIVALEN dalam intermediate language oleh **intermediate code generator**.

**Intermediate language bisa BERAGAM,** desainer compiler yang memutuskan. Pilihan umum: **syntax tree, postfix notation,** atau **three-address code (quadruple)** — deck ini FOKUS ke quadruple. Quadruple DEKAT dengan instruksi mesin, tapi BUKAN instruksi mesin SESUNGGUHNYA. Beberapa bahasa punya intermediate language yang WELL-DEFINED: **Java → Java Virtual Machine (JVM)**, **Prolog → Warren Abstract Machine**. Ada byte-code EMULATOR untuk mengeksekusi instruksi di intermediate language ini.

### Directed Acyclic Graph (DAG)
Seperti syntax tree untuk sebuah ekspresi, DAG punya LEAVES yang berkorespondensi dengan operand ATOMIK dan node INTERIOR yang berkorespondensi dengan OPERATOR. **PERBEDAANNYA:** sebuah node N di DAG bisa punya LEBIH DARI SATU parent kalau N merepresentasikan SUB-EKSPRESI YANG SAMA (common subexpression).

**Contoh:** DAG untuk `a + a * (b-c) + (b-c) * d` — node `(b-c)` HANYA DIBUAT SEKALI, meski muncul DUA KALI di ekspresi — kedua kemunculan itu MERUJUK ke node yang SAMA.

> [!info] Analogi
> DAG itu seperti CATATAN BELANJA yang PINTAR mengenali barang DUPLIKAT. Kalau syntax tree biasa itu seperti menulis "beli tomat" DUA KALI di daftar belanja (satu untuk resep A, satu untuk resep B) — meski sama-sama "tomat", ditulis TERPISAH. DAG itu seperti menyadari "oh, tomat untuk resep A dan tomat untuk resep B itu SAMA, cukup beli SEKALI, tapi PAKAI untuk KEDUA resep" — menghindari kerja GANDA yang tidak perlu.

**Contoh syntax-directed definition membangun DAG** untuk `a + a * (b-c) + (b-c) * d`:
```
P1 = Leaf(id, entry-a)
P2 = Leaf(id, entry-a) = P1     <- deteksi node SAMA, pakai ulang P1
P3 = Leaf(id, entry-b)
P4 = Leaf(id, entry-c)
P5 = Node('-', P3, P4)
P6 = Node('*', P1, P5)
P7 = Node('+', P1, P6)
P8 = Leaf(id, entry-b) = P3     <- pakai ulang P3
P9 = Leaf(id, entry-c) = P4     <- pakai ulang P4
P10 = Node('-', P3, P4) = P5    <- pakai ulang P5 (sub-ekspresi sama!)
P11 = Leaf(id, entry-d)
P12 = Node('*', P5, P11)
P13 = Node('+', P7, P12)
```

### Three-Address Code
**Three-address code** adalah representasi LINEAR dari syntax tree atau DAG, di mana NAMA berkorespondensi dengan node INTERIOR graph.

**Contoh** (DAG untuk `a+a*(b-c)+(b-c)*d`, dengan `t1..t5` sebagai temporary variable):
```
t1 = b - a
t2 = a * t1
t3 = a + t2
t4 = t1 * d
t5 = t3 + t4
```

### Tiga Representasi Three-Address Code

**1. Quadruple** — punya EMPAT field: **op, arg1, arg2, result**. Contoh: `x = y+z` direpresentasikan dengan `+` di op, `y` di arg1, `z` di arg2, `x` di result.

**Pengecualian:**
- Instruksi UNARY seperti `x = minus y` TIDAK PAKAI arg2.
- Statement copy seperti `x = y`: op adalah `=`.
- Operator seperti `param` TIDAK PAKAI arg2 maupun result.
- Jump kondisional/unconditional menaruh target LABEL di field result.

**Contoh tabel quadruple:**
| # | Op | Arg1 | Arg2 | Result |
| --- | --- | --- | --- | --- |
| 0 | Minus | c | | t1 |
| 1 | * | b | t1 | t2 |
| 2 | Minus | c | | t3 |
| 3 | * | b | t3 | t4 |
| 4 | + | t2 | t4 | t5 |
| 5 | = | t5 | | a |

**2. Triple** — HANYA TIGA field: **op, arg1, arg2**. Field result TIDAK ADA (dipakai terutama untuk nama sementara). Memakai triple, kita MERUJUK hasil operasi `x op y` lewat POSISINYA, bukan nama temporary eksplisit.

**Contoh tabel triple** (menunjukkan `(0)`, `(2)`, dst. sebagai REFERENSI ke baris sebelumnya):
| # | Op | Arg1 | Arg2 |
| --- | --- | --- | --- |
| 0 | Minus | c | |
| 1 | * | b | (0) |
| 2 | Minus | c | |
| 3 | * | b | (2) |
| 4 | + | t2 | (3) |
| 5 | = | a | (4) |

> [!info] Konteks tambahan (bukan dari slide)
> Kelebihan Triple dibanding Quadruple: LEBIH HEMAT memori karena tidak perlu nama temporary eksplisit. Kekurangannya: kalau instruksi PERLU dipindah posisinya (misalnya untuk OPTIMISASI kode di [[W23 - Code Optimization]]), SEMUA referensi posisi `(n)` yang menunjuk ke instruksi itu HARUS DIUPDATE juga — ini yang mendorong munculnya varian **Indirect Triple** (triple + array pointer TERPISAH yang menunjuk urutan eksekusi) supaya reorder instruksi jadi LEBIH MUDAH (cukup ubah array pointer, tanpa mengubah triple aslinya).

### Bentuk-Bentuk Three-Address Statement
**Binary Operator:** `op y,z,result` atau `result := y op z`
```
add  a,b,c    // c = a+b
gt   a,b,c    // c = (a>b)
addr a,b,c    // penjumlahan real
addi a,b,c    // penjumlahan integer
```

**Unary Operator:** `op y,,result` atau `result := op y`
```
uminus    a,,c    // c = -a
not       a,,c    // c = !a
inttoreal a,,c    // konversi tipe
```

**Move Operator:** `mov y,,result` atau `result := y`
```
mov   a,,c
movi  a,,c   // move integer
movr  a,,c   // move real
```

**Unconditional Jump:** `jmp ,,L` atau `goto L`
```
jmp ,,L1    // lompat ke label L1
jmp ,,7     // lompat ke statement nomor 7
```

**Conditional Jump:** `jmprelop y,z,L` atau `if y relop z goto L`
```
jmpgt   y,z,L1   // lompat ke L1 kalau y>z
jmpgte  y,z,L1   // lompat kalau y>=z
jmpe    y,z,L1   // lompat kalau y==z
jmpne   y,z,L1   // lompat kalau y!=z
jmpnz   y,,L1    // lompat kalau y bukan nol
jmpz    y,,L1    // lompat kalau y nol
jmpt    y,,L1    // lompat kalau y true
jmpf    y,,L1    // lompat kalau y false
```

**Procedure Call:** `param x,,` dan `call p,n,`
```
f(x+1, y):
    add   x,1,t1
    param t1,,
    param y,,
    call  f,2,
```

**Indexed Assignment:**
```
move y[i],,x   // x := y[i]
move x,,y[i]   // y[i] := x
```

**Address dan Pointer Assignment:**
```
moveaddr y,,x   // x := &y
movecont y,,x   // x := *y
```

### Syntax-Directed Translation ke Three-Address Code
```
S -> id := E   { S.code = E.code || gen('mov' E.place ',,' id.place) }
E -> E1 + E2   { E.place=newtemp(); E.code = E1.code||E2.code||gen('add' E1.place','E2.place','E.place) }
E -> E1 * E2   { E.place=newtemp(); E.code = E1.code||E2.code||gen('mult' E1.place','E2.place','E.place) }
E -> - E1      { E.place=newtemp(); E.code = E1.code||gen('uminus' E1.place',,'E.place) }
E -> ( E1 )    { E.place = E1.place; E.code = E1.code }
E -> id        { E.place = id.place; E.code = null }
```

**Kontrol alur (dengan LABEL):**
```
S -> while E do S1
   { S.begin=newlabel(); S.after=newlabel();
     S.code = gen(S.begin":") || E.code || gen('jmpf' E.place',,'S.after)
              || S1.code || gen('jmp'',,'S.begin) || gen(S.after":") }

S -> if E then S1 else S2
   { S.else=newlabel(); S.after=newlabel();
     S.code = E.code || gen('jmpf' E.place',,'S.else) || S1.code
              || gen('jmp'',,'S.after) || gen(S.else":") || S2.code || gen(S.after":") }
```

### Translation Scheme (dengan lookup dan error handling)
```
S -> id := E   { p=lookup(id.name); if (p!=nil) emit('mov' E.place',,'p) else error("undefined-variable") }
E -> E1 + E2   { E.place=newtemp(); emit('add' E1.place','E2.place','E.place) }
E -> - E1      { E.place=newtemp(); emit('uminus' E1.place',,'E.place) }
E -> id        { p=lookup(id.name); if (p!=nil) E.place=id.place else error("undefined-variable") }
```

### Translation Scheme dengan Backpatching (Boolean Expression & Control Flow)
Untuk `while` dan `if-else`, target JUMP ke `else`/`after` seringkali BELUM DIKETAHUI saat kode SEDANG dihasilkan (target itu ada di kode yang BELUM diproses). Solusinya: **backpatching** — TARUH placeholder `'NOTKNOWN'` dulu, lalu ISI KEMUDIAN saat lokasi sesungguhnya sudah diketahui.

```
S -> while {E.inloc=S.inloc} E do
     { emit(E.outloc 'jmpf' E.place',,'NOTKNOWN'); S1.inloc=E.outloc+1; } S1
     { emit(S1.outloc 'jmp'',,'S.inloc); S.outloc=S1.outloc+1;
       backpatch(E.outloc, S.outloc); }

S -> if {E.inloc=S.inloc} E then
     { emit(E.outloc 'jmpf' E.place',,'NOTKNOWN'); S1.inloc=E.outloc+1; } S1 else
     { emit(S1.outloc 'jmp'',,'NOTKNOWN'); S2.inloc=S1.outloc+1;
       backpatch(E.outloc, S2.inloc); } S2
     { S.outloc=S2.outloc; backpatch(S1.outloc, S.outloc); }
```

> [!info] Analogi
> Backpatching itu seperti menulis SURAT dengan meninggalkan RUANG KOSONG untuk "tanggal balasan" karena kamu BELUM TAHU kapan surat itu akan dibalas. Kamu tetap TULIS DAN KIRIM surat itu SEKARANG (menghasilkan kode `jmpf` dengan target 'NOTKNOWN'), tapi begitu KAMU TAHU informasi yang hilang itu (lokasi label `S.after` misalnya), kamu KEMBALI dan ISI ruang kosong tadi dengan info yang BENAR (backpatch). Ini perlu karena kode program dibaca SEKALI dari atas ke bawah, tapi target JUMP sering ada DI BAWAH (belum diproses) saat instruksi jump-nya sendiri dihasilkan.

### Contoh Lengkap: Three-Address Code untuk Program Nyata
```
x := 1;
y := x+10;
while (x<y) {
    x := x+1;
    if (x%2 == 1) then y := y+1;
    else y := y-2;
}
```
Hasil three-address code:
```
01: mov  1,, x
02: add  x,10, t1
03: mov  t1,, y
04: lt   x,y, t2
05: jmpf t2,,17
06: add  x,1, t3
07: mov  t3,, x
08: mod  x,2, t4
09: eq   t4,1, t5
10: jmpf t5,,14
11: add  y,1, t6
12: mov  t6,, y
13: jmp  ,,16
14: sub  y,2, t7
15: mov  t7,, y
16: jmp  ,,4
17:
```

### Alamat Array
**Array 1-Dimensi:** elemen array bisa diakses CEPAT kalau disimpan di BLOK LOKASI BERURUTAN. Lokasi `A[i] = baseA + (i-low)*width`, di mana **baseA** = alamat lokasi PERTAMA array A, **width** = lebar tiap elemen array, **low** = index elemen PERTAMA array.

**Optimasi rumus:** `baseA+(i-low)*width` bisa DITULIS ULANG jadi `i*width + (baseA-low*width)`. Jadi lokasi `A[i]` bisa dihitung saat RUN-TIME dengan formula `i*width + c`, di mana `c = baseA-low*width` **DIHITUNG SAAT COMPILE-TIME**. Intermediate code generator HANYA perlu menghasilkan kode untuk formula `i*width+c` (SATU perkalian, SATU penjumlahan).

**Array 2-Dimensi:** disimpan **row-major** (baris demi baris) atau **column-major** (kolom demi kolom) — SEBAGIAN BESAR bahasa pemrograman memakai **row-major**. Lokasi `A[i1,i2] = baseA + ((i1-low1)*n2+i2-low2)*width`, ditulis ulang jadi: `((i1*n2)+i2)*width + (baseA-((low1*n1)+low2)*width)`.

**Array Multi-Dimensi:** secara umum, lokasi `A[i1,i2,...,ik]` dihitung memakai RECURRENCE EQUATION:
```
e1 = i1
em = em-1 * nm + im
```
Lalu dikalikan `width` dan ditambah konstanta `c` (dihitung compile-time).

### Translation Scheme untuk Array
Grammar dengan **L→id|id[Elist]**, **Elist→Elist,E|E** butuh INHERITED ATTRIBUTE. Untuk MENGHINDARI kebutuhan inherited attribute (cukup SYNTHESIZED saja), dipakai grammar ALTERNATIF:
```
L -> id | Elist ]
Elist -> Elist , E | id [ E
```

```
S -> L := E   { if (L.offset is null) emit('mov' E.place',,'L.place)
                else emit('mov' E.place',,'L.place'['L.offset']') }
E -> L        { if (L.offset is null) E.place=L.place
                else { E.place=newtemp(); emit('mov' L.place'['L.offset']'',,'E.place) } }
L -> id       { L.place=id.place; L.offset=null; }
L -> Elist ]  { L.place=newtemp(); L.offset=newtemp();
                emit('mov' c(Elist.array)',,'L.place);
                emit('mult' Elist.place','width(Elist.array)','L.offset) }
Elist -> Elist1 , E  { Elist.array=Elist1.array; Elist.place=newtemp(); Elist.ndim=Elist1.ndim+1;
                        emit('mult' Elist1.place','limit(Elist.array,Elist.ndim)','Elist.place);
                        emit('add' Elist.place','E.place','Elist.place); }
Elist -> id [ E      { Elist.array=id.place; Elist.place=E.place; Elist.ndim=1; }
```

**Contoh 1 (array 1D double, index 5..100):** `x := A[y]` menghasilkan:
```
mov  c,,t1     // c = baseA - 5*8
mult y,8,t2
mov  t1[t2],,t3
mov  t3,,x
```

**Contoh 2 (array 2D int, 1..10 x 1..20):** `x := A[y,z]` menghasilkan:
```
mult y,20,t1
add  t1,z,t1
mov  c,,t2     // c = baseA - (1*20+1)*4
mult t1,4,t3
mov  t2[t3],,t4
mov  t4,,x
```

**Contoh 3 (array 3D int, 0..9 x 0..19 x 0..29):** `x := A[w,y,z]` menghasilkan:
```
mult w,20,t1
add  t1,y,t1
mult t1,30,t2
add  t2,z,t2
mov  c,,t3     // c = baseA - ((0*20+0)*30+0)*4
mult t2,4,t4
mov  t3[t4],,t5
mov  t5,,x
```

### Deklarasi dan Symbol Table
```
P -> M D
M -> ε   { offset=0 }
D -> D ; D
D -> id : T  { enter(id.name, T.type, offset); offset=offset+T.width }
T -> int     { T.type=int; T.width=4 }
T -> real    { T.type=real; T.width=8 }
T -> array[num] of T1  { T.type=array(num.val,T1.type); T.width=num.val*T1.width }
T -> ↑T1     { T.type=pointer(T1.type); T.width=4 }
```
`enter` membuat entri symbol table dengan nilai yang diberikan.

### Nested Procedure Declarations
Untuk SETIAP prosedur, kita buat SATU symbol table. Fungsi bantu:
- **mktable(previous)** — buat symbol table BARU, `previous` = parent symbol table.
- **enter(symtable,name,type,offset)** — buat entri VARIABEL baru di symbol table.
- **enterproc(symtable,name,newsymbtable)** — buat entri PROSEDUR baru di symbol table PARENT-nya.
- **addwidth(symtable,width)** — taruh total WIDTH semua entri di HEADER symbol table itu.

Dua stack dibutuhkan: **tblptr** (pointer ke symbol table) dan **offset** (offset SAAT INI di symbol table di puncak tblptr).

```
P -> M D    { addwidth(top(tblptr),top(offset)); pop(tblptr); pop(offset) }
M -> ε      { t=mktable(nil); push(t,tblptr); push(0,offset) }
D -> proc id N D ; S  { t=top(tblptr); addwidth(t,top(offset)); pop(tblptr); pop(offset);
                         enterproc(top(tblptr),id.name,t) }
D -> id : T { enter(top(tblptr),id.name,T.type,top(offset)); top(offset)=top(offset)+T.width }
N -> ε      { t=mktable(top(tblptr)); push(t,tblptr); push(0,offset) }
```

## Diagram & Visual
- **Slide 4 — Diagram Intermediate Code Generation dalam pipeline compiler**
  ![[99-Assets/Compiler/W21-slide04.png]]
- **Slide 6 — Contoh DAG untuk a+a*(b-c)+(b-c)*d**
  ![[99-Assets/Compiler/W21-slide06.png]]
- **Slide 11 — Diagram Triple**
  ![[99-Assets/Compiler/W21-slide11.png]]
- **Slide 26-29 — Diagram perhitungan alamat array 1D dan 2D**
  ![[99-Assets/Compiler/W21-slide26.png]]
  ![[99-Assets/Compiler/W21-slide27.png]]
  ![[99-Assets/Compiler/W21-slide28.png]]
  ![[99-Assets/Compiler/W21-slide29.png]]

## Rumus / Sintaks
```
Quadruple: (op, arg1, arg2, result)
Triple:    (op, arg1, arg2)  -- hasil dirujuk lewat posisi (n)

Array 1D:  lokasi A[i] = i*width + c,  c = baseA - low*width  (dihitung compile-time)
Array 2D:  lokasi A[i1,i2] = ((i1*n2)+i2)*width + c
Array kD (recurrence):
  e1 = i1
  em = em-1 * nm + im
  lokasi = ek*width + c

Backpatching:
1. emit instruksi jump dengan target 'NOTKNOWN'
2. proses kode berikutnya, catat lokasi label sesungguhnya
3. backpatch(lokasi_instruksi_jump, lokasi_label_sesungguhnya)
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Common subexpression** | Sub-ekspresi yang muncul lebih dari sekali dalam satu ekspresi besar |
| **newtemp() / newlabel()** | Fungsi menghasilkan nama variabel/label sementara baru yang unik |
| **Row-major / Column-major** | Dua cara menyimpan array multi-dimensi secara linear di memori |
| **Symbol table nested** | Symbol table bertingkat untuk mendukung scope prosedur di dalam prosedur |

## Pertanyaan Terbuka
- Slide tidak menjelaskan detail **Indirect Triple** secara eksplisit (hanya disebutkan sebagai bagian outline "Triple indirect representation") — perlu dibuka manual untuk detail struktur array pointer tambahannya.

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa DAG bisa "otomatis menghilangkan redundansi perhitungan" dibanding syntax tree biasa. Ambil contoh `a+a*(b-c)+(b-c)*d` — tunjukkan berapa kali `(b-c)` DIHITUNG ULANG di syntax tree biasa vs di DAG, dan jelaskan DAMPAK ini terhadap JUMLAH instruksi three-address code yang dihasilkan.
2. **(C4 – Analisis)** Bandingkan Quadruple dan Triple dari sisi KEBUTUHAN MEMORI dan KEMUDAHAN REORDER instruksi (untuk optimisasi di [[W23 - Code Optimization]]). Analisis: kenapa Triple LEBIH SULIT di-reorder dibanding Quadruple, PADAHAL Triple lebih HEMAT memori?
3. **(C5 – Evaluasi)** Evaluasi optimasi rumus alamat array dari `baseA+(i-low)*width` menjadi `i*width+c`. Buktikan bahwa KEDUA rumus ini EKUIVALEN secara matematis, lalu jelaskan KEUNTUNGAN PRAKTIS dari bentuk kedua — kenapa menghitung `c` di COMPILE-TIME (bukan run-time) menghemat operasi yang harus dilakukan program SETIAP KALI array diakses saat program berjalan?
4. **(C5 – Evaluasi)** Bandingkan translation scheme untuk `while` (yang butuh backpatch SATU target — `S.after`) dengan `if-then-else` (yang butuh backpatch DUA target — `S.else` dan `S.after`). Evaluasi: kenapa `if-then-else` butuh backpatching LEBIH KOMPLEKS — hubungkan dengan fakta bahwa ada DUA CABANG (then dan else) yang masing-masing perlu tahu KE MANA melompat setelah cabangnya selesai dieksekusi.
5. **(C6 – Cipta)** Kerjakan dengan mengikuti pola Contoh 1-3 di materi (array 1D, 2D, 3D): rancang three-address code untuk mengakses `x := B[p,q]` dari array 2D `B` bertipe `char` dengan dimensi `0..49 x 0..99` (width=1 byte, low1=0, low2=0, n2=100). Tunjukkan LANGKAH DEMI LANGKAH perhitungan konstanta `c` dan kode three-address lengkapnya, mengikuti format Contoh 2 di materi.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W20 - Semantic Analyzer]]
- [[W23 - Code Optimization]]
- [[Compiler - Review dan Glosari]]
