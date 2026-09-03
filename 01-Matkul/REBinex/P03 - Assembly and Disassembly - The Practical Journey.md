---
matkul: Reverse Engineering dan Binary Exploitation
minggu: 3
sks: 3
sumber: REBinex - P3.pptx
tags: [kuliah/rebinex, minggu/p03]
status: draft
diproses: 2026-09-04
---

# P03 — Assembly and Disassembly: The Practical Journey

> [!warning] Learning outcome di slide 2 ("Explain File Carving") nggak nyambung sama isi deck ini (masih soal assembly). Lihat catatan di [[P02 - Assembly and Disassembly]] — kesalahan copy-paste yang sama berulang di semua deck P02-P13.

## Ringkasan
> - **Stack** = struktur data **LIFO** (Last In, First Out) buat nyimpen memori fungsi, variabel lokal, dan kontrol alur. Instruksinya: `push`, `pop`, `call`, `leave`, `enter`, `ret`. Stack dialokasikan dari atas ke bawah — makin banyak yang di-push, makin KECIL alamatnya.
> - Tiap kali `call` dijalanin, dibikin **stack frame** baru.
> - Parameter fungsi `main()` standar C: `int main(int argc, char** argv)` — `argc` = jumlah argumen, `argv` = pointer ke array string argumen. Buat akses `argv[1]`, kompiler nambahin **offset 4 byte** (ukuran satu alamat di sistem 32-bit) ke alamat awal array-nya.
> - **Global variable** direferensikan lewat **alamat memori tetap** (misal `dword_40CF60`). **Local variable** direferensikan lewat **alamat relatif ke EBP** di stack (misal `[ebp-4]`). Ini pembeda paling gampang buat nebak jenis variabel dari kode assembly.
> - Compiler kadang ganti `inc`/`dec` (increment/decrement) jadi `add`/`sub` — assembly hasil compile SERING nggak "intuitif" walau secara matematis sama aja.
> - Percabangan `if/else` di source code diterjemahin jadi **conditional jump** (`jnz`, dll) yang didahului **compare** (`cmp`). `jmp` biasa dipakai buat lompatin bagian `else` yang nggak perlu dijalanin.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Stack frame | Ruang di stack yang dibikin tiap kali sebuah fungsi dipanggil (`call`) |
| LIFO | Last In, First Out; yang terakhir masuk, itu yang pertama keluar |
| `argc` / `argv` | Parameter standar `main()` C: jumlah argumen / array pointer ke string argumen |
| Global variable | Variabel yang bisa diakses fungsi mana pun; di assembly, direferensikan lewat alamat memori tetap |
| Local variable | Variabel yang cuma bisa diakses di fungsi tempatnya didefinisikan; di assembly, direferensikan lewat offset relatif EBP |
| `cmp` | Instruksi assembly buat bandingin dua nilai |
| `jnz` / `jmp` | Conditional jump (lompat kalau kondisi terpenuhi) / unconditional jump (lompat selalu) |

## Isi

### The Stack
Memori buat fungsi, variabel lokal, dan kontrol alur disimpen di **stack**, struktur data yang dicirikan oleh push dan pop. Kamu **push** item ke stack, terus **pop** item itu dari stack. **Stack adalah struktur LIFO (last in, first out).**

Instruksi stack meliputi `push`, `pop`, `call`, `leave`, `enter`, dan `ret`. Stack dialokasikan dalam format **top-down** di memori, dan alamat tertinggi dialokasikan dan dipakai duluan. **Makin banyak nilai yang di-push ke stack, makin kecil alamat yang dipakai.**

**Tiap kali `call` dijalanin, sebuah stack frame baru dibuat.**

> [!info] Analogi
> Stack itu kayak **tumpukan piring di dapur restoran** — piring paling akhir dicuci, itu yang paling atas tumpukannya, dan itu juga yang pertama diambil kalau ada yang mau pinjam piring (LIFO). Tiap kali ada order baru masuk (`call`), koki nambahin satu "nampan kerja" baru di atas tumpukan — itulah **stack frame**-nya, tempat semua bahan (variabel lokal) buat order itu ditaruh, terpisah dari nampan order lain.

### C Main Method & Offsets
Program C standar punya dua argumen buat method `main`, biasanya dalam bentuk: `int main(int argc, char** argv)`.

- **`argc`** — integer berisi jumlah argumen di command line, TERMASUK nama programnya
- **`argv`** — pointer ke array string yang berisi argumen command-line

**Gimana `argv[1]` diakses (dari analisis kode assembly-nya):** pertama lokasi awal array-nya dimuat ke EAX. Lalu **4 (offset-nya)** ditambahin ke EAX buat dapetin `argv[1]`. Angka 4 dipakai karena tiap entri di array `argv` itu ALAMAT ke sebuah string, dan tiap alamat itu **4 byte** ukurannya di sistem 32-bit.

Dari contoh kode di slide: `argc` dibandingin ke `'3'`, `argv[1]` dibandingin ke `'-r'` — kalau `-r` disediain sebagai argumen, kode di titik tertentu bakal dieksekusi.

### Global vs Local Variables
Global variable bisa diakses dan dipakai fungsi mana pun di dalam program. Local variable cuma bisa diakses oleh fungsi tempat dia didefinisikan.

**Global variable direferensikan lewat ALAMAT MEMORI, dan local variable direferensikan lewat ALAMAT STACK.**

- **Contoh global variable**: variabel global `x` ditandai sebagai `dword_40CF60`, sebuah lokasi memori di `0x40CF60`. `x` berubah di memori ketika EAX dipindahkan ke `dword_40CF60`.
- **Contoh local variable**: variabel lokal `x` berada di stack pada offset KONSTAN relatif ke `EBP`. Lokasi memori `[ebp-4]` dipakai secara konsisten sepanjang fungsi itu.

> [!info] Analogi
> **Global variable itu papan pengumuman di lobi kantor** — alamatnya (lokasi papan) TETAP, siapa pun dari lantai mana pun bisa datang baca/tulis di situ. **Local variable itu catatan di meja kerja masing-masing pegawai** — posisinya selalu "di atas meja SAYA" (relatif ke EBP-nya sendiri), dan begitu pegawai itu pulang (fungsinya selesai), catatan itu ilang bareng mejanya (stack frame dibongkar).

### Disassembling Arithmetic
Contoh dari slide: dua variabel lokal `a` dan `b` (di-label `var_4` dan `var_8` sama IDA Pro), dan berbagai operasi aritmetika. Prosesnya: `var_4` dan `var_8` diinisialisasi ke 0 dan 1. `a` dipindahin ke EAX, lalu `0x0b` ditambahin ke EAX (jadi `a` bertambah 11). `b` kemudian dikurangkan dari `a`.

**Fun fact dari slide:** di sini compiler MEMILIH pakai instruksi `sub` dan `add`, **bukan** instruksi `inc` (increment) dan `dec` (decrement) yang secara logika lebih langsung — meskipun secara matematika hasilnya sama.

> [!info] Konteks tambahan (bukan dari slide)
> Ini contoh nyata dari yang disinggung di [[P02 - Assembly and Disassembly]] soal compiler yang "counter-intuitive". Programmer pemula mungkin nulis `a++` yang secara logika = increment, tapi compiler bebas pilih instruksi APA PUN yang secara matematis setara dan lebih optimal buat arsitektur target-nya. Ini kenapa reverser nggak bisa nebak source code ASLI cuma dari pola instruksi assembly-nya — banyak source code berbeda bisa menghasilkan assembly yang sama, dan sebaliknya.

### Recognizing IF Statements
Dalam assembly, sebuah keputusan berkorespondensi ke **conditional jump** (`jnz`, ditunjukkan di contoh slide). Keputusan buat lompat dibuat berdasarkan perbandingan (`cmp`), yang ngecek apakah `var_4` sama dengan `var_8` (`var_4` dan `var_8` berkorespondensi ke `x` dan `y` di source code).

**Kalau nilainya nggak sama, lompatannya terjadi**, dan kodenya mencetak "x is not equal to y." **Kalau nggak**, kodenya lanjut jalan biasa dan mencetak "x equals y."

Perhatikan juga adanya `jmp` yang **melompati bagian else** dari kode. **Penting buat kamu sadar bahwa cuma SATU dari dua jalur kode ini yang bisa diambil.**

> [!info] Konteks tambahan (bukan dari slide)
> Pola "cmp → conditional jump → (kode if) → jmp (lompatin else) → (kode else)" ini adalah **sinyal paling universal buat ngenalin percabangan `if/else`** di kode assembly mana pun, terlepas dari bahasa aslinya. Kalau kamu lagi nge-reverse binari dan nemu urutan `cmp` diikuti `jXX` (jump kondisional apa pun — `jz`, `jnz`, `je`, `jne`, `jg`, `jl`, dll), itu HAMPIR SELALU tempat sebuah keputusan `if` diambil.

## Diagram & Visual
- **Slide 4 — diagram stack frame tiap kali `call` dijalankan**
  ![[99-Assets/REBinex/P03-slide04.png]]
- **Slide 6, 7 — kode C contoh dan hasil terjemahan assembly-nya (C Main Method & Offsets)**
  ![[99-Assets/REBinex/P03-slide06.png]]
  ![[99-Assets/REBinex/P03-slide07.png]]
- **Slide 8 — screenshot assembly beranotasi Point 1/2/3 (perbandingan argc, argv[1])**
  ![[99-Assets/REBinex/P03-slide08.png]]
- **Slide 10 — Code A dan Code B, contoh global vs local variable (C)**
  ![[99-Assets/REBinex/P03-slide10.png]]
  ![[99-Assets/REBinex/P03-slide10b.png]]
- **Slide 11, 12 — hasil assembly Code A (global) dan Code B (local)**
  ![[99-Assets/REBinex/P03-slide11.png]]
  ![[99-Assets/REBinex/P03-slide12.png]]
- **Slide 13, 14 — kode C dan assembly untuk contoh operasi aritmetika**
  ![[99-Assets/REBinex/P03-slide13.png]]
  ![[99-Assets/REBinex/P03-slide14.png]]
  ![[99-Assets/REBinex/P03-slide14b.png]]
- **Slide 16, 17 — kode C dan assembly untuk contoh percabangan if/else**
  ![[99-Assets/REBinex/P03-slide16.png]]
  ![[99-Assets/REBinex/P03-slide17.png]]

> [!warning] **Deck ini seluruhnya berbasis contoh kode dalam gambar** — teks slide-nya cuma penjelasan naratif ("examine the code below", "the assembly code would be like this"), sementara kode C dan assembly persisnya ADA di gambar-gambar di atas. Kalau mau latihan baca kode persisnya sendiri (bukan cuma penjelasannya), buka gambar-gambar tersebut atau PPT aslinya.

## Rumus / Sintaks
```
; pola umum akses argv[1] (32-bit):
mov eax, [argv_base]     ; alamat awal array argv dimuat ke eax
add eax, 4                ; +4 byte offset karena tiap alamat = 4 byte di 32-bit
                           ; sekarang eax menunjuk ke argv[1]

; pembeda global vs local variable:
mov dword_40CF60, eax     ; GLOBAL -- alamat memori TETAP
mov [ebp-4], eax          ; LOCAL  -- offset RELATIF ke EBP

; pola pengenalan if/else:
cmp var_4, var_8           ; bandingkan x dan y
jnz label_not_equal        ; kalau TIDAK sama, lompat (conditional jump)
; ... kode untuk "x equals y" ...
jmp label_end               ; lompati bagian else (unconditional jump)
label_not_equal:
; ... kode untuk "x is not equal to y" ...
label_end:
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **`push` / `pop`** | Instruksi buat naruh nilai ke puncak stack / ngambil nilai dari puncak stack |
| **`call` / `ret`** | Instruksi buat manggil fungsi (lompat + simpan alamat kembali) / kembali dari fungsi |
| **`leave` / `enter`** | Instruksi buat membongkar / menyiapkan stack frame sebuah fungsi |
| **IDA Pro** | Salah satu disassembler populer yang otomatis melabeli variabel lokal sebagai `var_N` |
| **`var_4`, `var_8`** | Notasi label IDA Pro untuk local variable, angkanya menunjukkan offset dari EBP |
| **`dword_XXXXXX`** | Notasi label untuk global variable, berupa alamat memori tetap |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Kamu lagi nge-reverse sebuah binari dan nemu instruksi `mov eax, [ebp-8]` di satu fungsi, dan `mov eax, dword_501020` di fungsi lain. Analisis: tanpa liat source code aslinya sama sekali, apa yang bisa kamu simpulkan tentang JENIS variabel yang diakses di masing-masing instruksi, dan seberapa yakin kamu dengan kesimpulan itu?
2. **(C4 – Analisis)** Bandingkan pola assembly untuk `if/else` (slide 18: `cmp` → `jnz` → kode → `jmp` → kode else) dengan konsep "hanya satu jalur yang bisa diambil". Analisis: kalau kamu nemu kode assembly dengan `cmp` dan `jnz` TAPI TANPA `jmp` di antara dua blok kode berikutnya, apa implikasinya terhadap struktur if/else aslinya (petunjuk: mungkin bukan if/else biasa, tapi apa?)?
3. **(C5 – Evaluasi)** Compiler di slide 15 memilih `sub`/`add` daripada `dec`/`inc` meski secara logika `inc`/`dec` lebih "pas". Evaluasi: apa ini SELALU berarti compiler-nya "lebih pintar" dari cara nulis manual, atau ada skenario di mana pemilihan compiler ini justru kurang optimal? (Petunjuk: pikirkan ukuran instruksi dan efek samping pada flag CPU.)
4. **(C5 – Evaluasi)** Seorang mahasiswa berpendapat: "Kalau saya bisa baca assembly dengan sempurna, saya bisa merekonstruksi source code C ASLI persis kata demi kata." Evaluasi klaim ini pakai konsep dari [[P02 - Assembly and Disassembly]] (proses compile itu satu arah, banyak informasi hilang) dan contoh `inc` vs `sub`/`add` di atas.
5. **(C6 – Cipta)** Rancang (dalam bentuk deskripsi, bukan gambar) susunan stack frame untuk pemanggilan fungsi `int hitung(int a, int b)` yang punya SATU variabel lokal `int hasil`, dipanggil dari `main()`. Sebutkan urutan dari alamat TERTINGGI ke TERENDAH: parameter, saved EIP, saved EBP, dan variabel lokal — jelasin kenapa urutannya seperti itu berdasarkan sifat "top-down" stack yang dijelasin di bagian The Stack.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_REBinex]]
- [[P02 - Assembly and Disassembly]]
- [[P04 - Obfuscation Techniques]]
- [[P08 - Deep Dive into Buffer Overflow]]
- [[REBinex - Review dan Glosari]]
