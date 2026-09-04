---
matkul: Compilation Techniques
minggu: 20
sks: 3
sumber: Session 20_Semantic Analyzer.pptx
tags: [kuliah/compiler, minggu/w20]
status: draft
diproses: 2026-09-04
---

# W20 — Semantic Analyzer / Type Checking

> [!warning] Slide judul deck ini secara internal menulis **"Session 18-19"**, TAPI nama filenya "Session 20_Semantic Analyzer.pptx" — bentrok dengan deck [[W18 - Syntax Directed Translation]] yang JUGA berjudul "Session 18-19" (topiknya berbeda: Syntax Directed Definition/Translation Schemes). Ini kemungkinan besar kesalahan copy-paste dosen di template judul. Note ini tetap dinomori **W20** mengikuti urutan LOGIS kurikulum (setelah Syntax Directed Translation, sebelum Intermediate Code Generator) dan nama filenya, BUKAN label internal slide yang salah.

## Ringkasan
> - Compiler harus melakukan **semantic check** SELAIN syntactic check — bisa **static** (saat kompilasi) atau **dynamic** (saat run-time). **Type checking** adalah salah satu static check.
> - **Type expression** merepresentasikan tipe konstruksi bahasa: **basic type** (int, real, char, boolean...), **type name**, atau **type constructor** (array, product/cartesian, pointer, function `D→R`).
> - **Structural equivalence** membandingkan tipe berdasarkan STRUKTURnya (rekursif membandingkan komponen), berbeda dari **name equivalence** yang membandingkan berdasarkan NAMA tipe saja — keduanya bisa memberi hasil BERBEDA untuk konstruksi tipe yang "terlihat sama".
> - Type checking menghasilkan aturan untuk EKSPRESI (operasi aritmatika, indexing array, dereference pointer), STATEMENT (assignment, if, while), dan FUNGSI (pemanggilan fungsi, argument matching).
> - **Operator/Function Overloading** (SATU syntax, BERBEDA operasi tergantung tipe operand) dan **Polymorphic Function** (SATU kode, bisa dieksekusi dengan argumen TIPE BERBEDA) — dua konsep lanjutan terkait sistem tipe.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Type system | Kumpulan aturan untuk menetapkan type expression ke bagian-bagian program |
| Type expression | Representasi sintaktik tipe sebuah konstruksi bahasa |
| Type constructor | Operator yang membangun type expression baru (array, pointer, function, dll.) |
| Structural equivalence | Dua tipe dianggap sama kalau strukturnya SAMA (rekursif) |
| Name equivalence | Dua tipe dianggap sama kalau NAMA tipenya SAMA |
| Operator overloading | Satu syntax dengan operasi BERBEDA tergantung tipe operand |
| Polymorphic function | Fungsi yang bisa dieksekusi dengan argumen dari tipe BERBEDA |

## Isi

### Type Checking: Konsep Dasar
Compiler harus melakukan SEMANTIC CHECK selain syntactic check. **Semantic check** ada dua jenis:
- **Static** — dilakukan SELAMA kompilasi.
- **Dynamic** — dilakukan SAAT run-time.

**Type checking** adalah salah satu operasi static checking — tapi TIDAK SEMUA type checking bisa dilakukan saat compile-time; beberapa sistem JUGA memakai dynamic type checking. **Type system** = kumpulan ATURAN untuk menetapkan type expression ke bagian-bagian program.

**Type checker mengimplementasikan type system.** **Sound type system** MENGHILANGKAN kebutuhan run-time type checking untuk error tipe. Bahasa pemrograman disebut **strongly-typed** kalau SETIAP program yang diterima compiler-nya akan JALAN TANPA error tipe. Dalam PRAKTIK, beberapa operasi type checking dilakukan saat run-time (jadi sebagian besar bahasa pemrograman TIDAK strongly-typed).

**Contoh:** `int x[100]; ... x[i]` — sebagian besar compiler TIDAK BISA MENJAMIN `i` akan berada di antara 0 dan 99 (butuh CEK RUN-TIME, bukan cuma compile-time).

### Type Expression
Tipe sebuah konstruksi bahasa didenotasikan lewat **type expression**. Type expression bisa berupa:
1. **Basic type** — tipe data primitif seperti integer, real, char, boolean; **type-error** untuk menandakan error tipe; **void** untuk "tanpa tipe".
2. **Type name** — nama yang dipakai untuk mendenotasikan sebuah type expression.
3. **Type constructor** — diterapkan ke type expression LAIN:
   - **arrays:** kalau T adalah type expression, `array(I,T)` adalah type expression, di mana I menandakan rentang index. Contoh: `array(0..99, int)`.
   - **products:** kalau T1 dan T2 type expression, cartesian product-nya `T1 x T2` adalah type expression. Contoh: `int x int`.
   - **pointers:** kalau T type expression, `pointer(T)` adalah type expression. Contoh: `pointer(int)`.
   - **functions:** fungsi bisa dipandang sebagai pemetaan dari tipe DOMAIN D ke tipe RANGE R — tipe fungsi didenotasikan `D → R`. Contoh: `int → int` mewakili fungsi yang menerima parameter int, kembaliannya juga int.

### Contoh Sistem Type Checking Sederhana
**Deklarasi tipe:**
```
P -> D;E
D -> D;D
D -> id:T    { addtype(id.entry, T.type) }
T -> char    { T.type = char }
T -> int     { T.type = int }
T -> real    { T.type = real }
T -> ↑T1     { T.type = pointer(T1.type) }
T -> array[intnum] of T1  { T.type = array(1...intnum.val, T1.type) }
```

**Type checking ekspresi:**
```
E -> id           { E.type = lookup(id.entry) }
E -> charliteral  { E.type = char }
E -> intliteral   { E.type = int }
E -> realliteral  { E.type = real }
E -> E1 + E2      { if (E1.type=int and E2.type=int) then E.type=int
                     else if (E1.type=int and E2.type=real) then E.type=real
                     else if (E1.type=real and E2.type=int) then E.type=real
                     else if (E1.type=real and E2.type=real) then E.type=real
                     else E.type = type-error }
E -> E1[E2]       { if (E2.type=int and E1.type=array(s,t)) then E.type=t
                     else E.type = type-error }
E -> E1↑          { if (E1.type=pointer(t)) then E.type=t
                     else E.type = type-error }
```

**Type checking statement:**
```
S -> id = E         { if (id.type=E.type) then S.type=void else S.type=type-error }
S -> if E then S1   { if (E.type=boolean) then S.type=S1.type else S.type=type-error }
S -> while E do S1  { if (E.type=boolean) then S.type=S1.type else S.type=type-error }
```

**Type checking fungsi:**
```
E -> E1(E2)  { if (E2.type=s and E1.type=s->t) then E.type=t else E.type=type-error }

Contoh: int f(double x, char y) { ... }
        f: double x char -> int    (argument types -> return type)
```

### Structural Equivalence
**Bagaimana kita tahu dua type expression SAMA?** Selama type expression dibangun dari basic type (TANPA type name), kita bisa pakai **structural equivalence** (perbandingan STRUKTUR) antara dua type expression.

**Algoritma structural equivalence (`sequiv`):**
```
sequiv(s, t):
  if s dan t basic type yang sama         -> return true
  else if s=array(s1,s2), t=array(t1,t2)  -> return sequiv(s1,t1) and sequiv(s2,t2)
  else if s=s1 x s2, t=t1 x t2            -> return sequiv(s1,t1) and sequiv(s2,t2)
  else if s=pointer(s1), t=pointer(t1)    -> return sequiv(s1,t1)
  else if s=s1->s2, t=t1->t2              -> return sequiv(s1,t1) and sequiv(s2,t2)
  else return false
```

### Nama untuk Type Expression, dan Masalah Siklus
Beberapa bahasa memberi NAMA ke type expression, lalu memakai nama itu sebagai type expression selanjutnya. Contoh (Pascal-like):
```
type link = ↑cell;
var p, q : link;
var r, s : ↑cell
// Apakah p,q,r,s punya tipe yang SAMA?
```
**Dua cara memperlakukan type name:** (1) dapatkan type expression EKUIVALEN untuk nama itu (lalu pakai structural equivalence), atau (2) perlakukan type name SEBAGAI basic type.

**Masalah SIKLUS:**
```
type link = ↑cell;
type cell = record
    x : int,
    next : link
    end;
```
Kita TIDAK BISA memakai structural equivalence kalau ADA SIKLUS di type expression (definisi `link` merujuk `cell`, dan `cell` merujuk balik ke `link`). Kita HARUS memperlakukan type name SEBAGAI basic type — tapi ini berarti type expression `link` DIANGGAP BERBEDA dari type expression `cell`, meski secara struktural terhubung.

> [!info] Analogi
> Masalah siklus dalam structural equivalence itu seperti mencoba membandingkan DUA CERMIN yang saling berhadapan — kalau kamu coba "mengintip strukturnya secara mendalam" (rekursif), kamu akan terjebak dalam PANTULAN TAK TERHINGGA (link → cell → link → cell → ...), tidak pernah selesai. Solusinya: berhenti mencoba "mengintip sampai dasar", dan cukup bandingkan NAMA CERMINNYA saja ("apakah ini cermin bernama link, atau cermin bernama cell?") — inilah kenapa type name yang terlibat siklus HARUS diperlakukan sebagai basic type, bukan dibongkar strukturnya berulang-ulang.

### Type Conversion (Coercion)
```
x + y   // apa tipe ekspresi ini (int atau double)?
```
Kalau tipe x adalah `double` dan tipe y adalah `int`, kode yang harus dihasilkan:
```
inttoreal   y, , t1     // konversi y (int) jadi real, simpan di t1
real+       t1, x, t2   // tambahkan t1 (real) dengan x (real), simpan di t2
```

### Static Checking: Tiga Jenis
1. **Type checks** — operator dan operand harus punya tipe yang KOMPATIBEL.
2. **Flow-of-control checks** — statement kontrol transfer harus punya TARGET yang SAH (misalnya `break`/`continue`).
3. **Uniqueness checks** — bahasa bisa mensyaratkan kemunculan UNIK dalam situasi tertentu, misalnya deklarasi variabel, label `case` di statement `switch`.

Cek-cek ini SERING bisa DIINTEGRASIKAN dengan parsing.

### Data Type dan Type Checking
**Data type** = himpunan NILAI beserta himpunan OPERASI yang bisa dilakukan padanya. Type checking bertujuan MEMVERIFIKASI operasi dalam kode program memang DIIZINKAN pada nilai operand-nya.

**Type Constructor dan Type Expression (formal):**
- Base type adalah type expression (boolean, char, int, float).
- Type name adalah type expression.
- Type constructor diterapkan ke type expression adalah type expression: **arrays** — `array(T)`; **records** — `record(f1:T1,...,fn:Tn)` dengan f1...fn identifier UNIK; **pointers** — `ptr(T)`; **functions** — `(T1,...,Tn) → T`.

### Kenapa Memakai Type Expression?
Contoh kasus program C dengan tipe function pointer kompleks (`f`, `f()`, `*f()`, `(*f())[2]`) menunjukkan bagaimana type expression BERUBAH langkah demi langkah tergantung OPERATOR yang diterapkan — misalnya:
```
Kode Program     | Type Expression        | Aturan
f                | ()->ptr(ptr(char))     | symbol table lookup
f()              | ptr(ptr(char))         | jika e:T1->T2 dan e1:T1, maka e(e1):T2
*f()             | ptr(char)              | jika e:ptr(T), maka *e:T
(*f())[2]        | char                   | jika e1:array(T), e2:int, maka e1[e2]:T
```
Slide menunjukkan contoh kode C real yang KOMPLEKS dengan `qsort()` dan function pointer cast, mempertanyakan apakah `(*f())[2]` LEGAL — menunjukkan betapa RUMITNYA type checking untuk bahasa dengan sistem tipe fleksibel seperti C.

### Notions of Type Equivalence
1. **Name equivalence** — di beberapa bahasa (misal Pascal), tipe bisa diberi NAMA. Name equivalence memandang nama tipe yang BERBEDA sebagai tipe yang BERBEDA — dua tipe name-equivalent HANYA JIKA punya nama tipe yang SAMA.
2. **Structural equivalence** — dua type expression structurally equivalent kalau punya STRUKTUR SAMA (menerapkan type constructor yang SAMA ke type expression yang structurally equivalent).

**Contoh Pascal:**
```
type p = ↑node;
     q = ↑node;
var x : p;
    y : q;
```
x dan y **structurally equivalent** (sama-sama pointer ke node), tapi **BUKAN name-equivalent** (nama tipenya berbeda: p vs q).

### Representasi Type Expression: Type Graph
**Type graph** = representasi type expression bergrafik: basic type diberi "internal value" PREDEFINED; named type direpresentasikan lewat pointer ke HASH TABLE; type expression komposit `f(T1,...,Tn)` direpresentasikan sebagai node yang mengidentifikasi constructor f, dengan pointer ke node-node T1,...,Tn. Contoh: `int x[10][20]`.

### Contoh Implementasi: Type Checking dengan Yacc
```c
Type result_type(Type t1, Type t2) {
    if (t1 == error || t2 == error) return error;
    if (t1 == t2) return t1;
    if (t1 == double || t2 == double) return double;
    if (t1 == float || t2 == float) return float;
    ...
}
```
| Production | Semantic Rule | Kode Yacc |
| --- | --- | --- |
| E → id | E.type = id.type | `{ $$ = symtab_lookup(id_name); }` |
| E → intcon | E.type = INTEGER | `{ $$ = INTEGER; }` |
| E → E1+E2 | E.type = result_type(E1.type,E2.type) | `{ $$ = result_type($1, $3); }` |

**Array:**
```c
E -> id[E1] {
    t1 = id.type;
    if (t1 == ARRAY && E1.type == INTEGER)
        E.type = id.element_type;
    else E.type = error;
}
```
**Function call:**
```c
E -> id '(' expr_list ')' {
    if (id.return_type == VOID) E.type = error;
    else if (chk_arg_types(id, expr_list))  // actual match formal jumlah&tipe
        E.type = id.return_type;
    else E.type = error;
}
```

### Type Checking Statement
Jenis statement BERBEDA punya kebutuhan tipe BERBEDA:
- `if`, `while` MEMBUTUHKAN kondisi boolean.
- LHS assignment harus **"l-value"** — sesuatu yang BISA DIBERI NILAI (assignable).
- LHS dan RHS assignment harus tipe yang **"kompatibel"** — kalau tipenya berbeda, KONVERSI dibutuhkan.

### Operator Overloading
**Overloading** merujuk pada pemakaian SYNTAX yang SAMA untuk MERUJUK operasi BERBEDA, tergantung tipe operand. Contoh: di Java, `+` bisa merujuk ke penjumlahan integer, penjumlahan floating point, atau KONKATENASI string.

Compiler memakai informasi TIPE operand untuk MENYELESAIKAN (resolve) overloading — mencari tahu operasi MANA yang sebenarnya dimaksud. Kalau informasi TIDAK CUKUP untuk resolve overloading, compiler bisa memberi ERROR.

### Polymorphic Functions
Sepotong kode (fungsi, operator) yang bisa DIEKSEKUSI dengan argumen dari TIPE BERBEDA. Contoh: operator built-in untuk indexing array, manipulasi pointer. **Kenapa dipakai:** memudahkan MANIPULASI struktur data TERLEPAS dari tipenya.

**Contoh ML:**
```
fun length(lptr) = if null(lptr) then 0
                    else length(tl(lptr)) + 1
```
Fungsi `length` bisa dipakai untuk LIST dari tipe APA PUN (list of int, list of string, dll.) — itulah polimorfisme.

## Diagram & Visual
- **Slide 12 — Diagram tipe fungsi (argument types → return type)**
  ![[99-Assets/Compiler/W20-slide12.png]]
- **Slide 22 — Type Graph untuk int x[10][20]**
  ![[99-Assets/Compiler/W20-slide22.png]]

## Rumus / Sintaks
```
Type Expression Building Blocks:
Basic type:      int, real, char, boolean, void, type-error
Array:           array(I, T)
Product:         T1 x T2
Pointer:         pointer(T)
Function:        D -> R

Structural Equivalence (sequiv):
sequiv(s,t):
  basic type sama         -> true
  array(s1,s2)/array(t1,t2) -> sequiv(s1,t1) and sequiv(s2,t2)
  pointer(s1)/pointer(t1)   -> sequiv(s1,t1)
  s1->s2 / t1->t2            -> sequiv(s1,t1) and sequiv(s2,t2)
  lainnya                   -> false

Type Checking Ekspresi Aritmatika (int/real):
int + int   -> int
int + real  -> real
real + real -> real
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Sound type system** | Sistem tipe yang menghilangkan kebutuhan cek tipe saat run-time |
| **Strongly-typed language** | Bahasa di mana SEMUA program yang diterima compiler jalan tanpa error tipe |
| **L-value** | Sesuatu yang BISA diberi nilai (assignable), misalnya variabel di sisi kiri `=` |
| **Coercion** | Konversi tipe implisit oleh compiler (misalnya int ke real) |
| **Hash table (type graph)** | Struktur data untuk menyimpan/mencari named type secara efisien |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa Structural Equivalence TIDAK BISA dipakai untuk type expression yang mengandung SIKLUS (contoh `link`/`cell` yang saling merujuk). Jelaskan APA yang terjadi kalau algoritma `sequiv` dijalankan PERSIS SEPERTI didefinisikan, tanpa penanganan khusus untuk siklus — di titik mana algoritma itu akan berjalan TANPA HENTI?
2. **(C4 – Analisis)** Bandingkan Name Equivalence dan Structural Equivalence memakai contoh Pascal `type p=↑node; q=↑node; var x:p; y:q;`. Analisis: kenapa DUA sistem penilaian kesetaraan tipe ini bisa memberi JAWABAN BERBEDA untuk PASANGAN VARIABEL YANG SAMA (x dan y) — apa IMPLIKASI PRAKTIS bagi programmer kalau bahasa yang dipakai memilih Name Equivalence (misalnya operasi `x := y` menjadi ERROR TIPE meski struktur keduanya identik)?
3. **(C5 – Evaluasi)** Evaluasi definisi "strongly-typed language" (SEMUA program yang diterima compiler akan jalan TANPA error tipe). Berdasarkan contoh `int x[100]; ... x[i]` (compiler TIDAK BISA menjamin `i` valid), evaluasi APAKAH bahasa seperti C bisa disebut strongly-typed — jelaskan jawabanmu dan kaitkan dengan perbedaan static vs dynamic type checking.
4. **(C5 – Evaluasi)** Bandingkan Operator Overloading (SATU syntax, operasi BERBEDA per tipe, diresolve saat COMPILE TIME) dengan Polymorphic Function (SATU kode, bisa jalan untuk BERBAGAI tipe TANPA perlu tahu tipe spesifiknya). Evaluasi: dari sudut pandang DESAIN bahasa pemrograman, kenapa polimorfisme dianggap lebih "FLEKSIBEL" — apa BATASAN dari overloading yang TIDAK dimiliki polimorfisme (pertimbangkan berapa banyak versi fungsi yang harus DITULIS ULANG untuk overloading vs polimorfisme)?
5. **(C6 – Cipta)** Rancang ATURAN semantic rule (mengikuti format `E → production { semantic rule }` dari materi) untuk operator MODULO (`%`) yang HANYA valid antara dua operand bertipe `int` (TIDAK ada implicit conversion dari `real`, beda dari operator `+` di materi yang boleh campur int-real). Tulis aturan lengkapnya, termasuk kasus ERROR kalau salah satu operand bertipe `real`.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W18 - Syntax Directed Translation]]
- [[W21 - Intermediate Code Generator]]
- [[Compiler - Review dan Glosari]]
