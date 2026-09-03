---
matkul: Reverse Engineering dan Binary Exploitation
minggu: 2
sks: 3
sumber: REBinex - P2.pptx
tags: [kuliah/rebinex, minggu/p02]
status: draft
diproses: 2026-09-04
---

# P02 — Assembly & Disassembly

> [!warning] Learning outcome di slide 2 ("Explain File Carving", "Using file, binwalk, foremost command") **nggak nyambung sama sekali** ke isi deck ini, yang seluruhnya bahas assembly x86. Ini kesalahan copy-paste yang konsisten di SEMUA deck P02–P13 (LO-nya sama persis di semua slide 2, kemungkinan template yang lupa diupdate). Isi note ini ngikutin konten SLIDE-nya (assembly), bukan LO yang salah ketik itu.

## Ringkasan
> - Malware analysis (dan reversing secara umum) kerja di **tiga level abstraksi**: bahasa tingkat tinggi (yang ditulis programmer/malware author) → **machine code** (hasil compiler, dijalanin CPU) → **assembly** (yang dibaca reverser lewat disassembler).
> - **x86** itu arsitektur CPU paling populer buat PC. Ngikutin arsitektur **Von Neumann**: CPU (eksekusi), RAM (nyimpen data+kode), I/O (ngobrol sama perangkat luar).
> - Memori program dibagi 4 bagian: **Data** (nilai statis/global), **Code** (instruksi program), **Heap** (memori dinamis, alokasi/bebasin manual), **Stack** (variabel lokal, parameter fungsi, kontrol alur).
> - Instruksi assembly = **mnemonic + operand**. Tiga jenis operand: **immediate** (nilai tetap, misal `0x42`), **register** (misal `ecx`), **memory address** (dikurung siku, misal `[eax]`).
> - **Register** itu penyimpanan kecil di CPU yang aksesnya jauh lebih cepet dari RAM. Empat kategori: **general, segment, status flags, instruction pointer**.
> - **EIP (instruction pointer)** = alamat instruksi BERIKUTNYA yang bakal dieksekusi. **Kalau kamu ngontrol EIP, kamu ngontrol apa yang CPU jalanin** — ini konsep paling penting buat exploitasi kayak buffer overflow nanti.
> - `mov` = instruksi paling dasar, pindahin data. `[ebx]` artinya "data DI alamat memori EBX", bukan "nilai EBX itu sendiri" — kurung siku = "ambil isi di alamat ini".
> - `nop` (no operation) nggak ngapa-ngapain — dipake buat **"NOP sled"** di serangan buffer overflow, biar shellcode nggak "meleset" mulai eksekusi di tengah-tengah.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Von Neumann architecture | Arsitektur komputer dengan CPU, RAM, dan I/O sebagai komponen inti |
| Register | Penyimpanan kecil di CPU, akses lebih cepet dari RAM |
| Mnemonic | Nama instruksi assembly yang gampang dibaca (misal `mov`, `add`) |
| Operand | Data yang dipakai sebuah instruksi (immediate/register/memory address) |
| EIP | Instruction pointer; alamat instruksi berikutnya yang bakal dieksekusi |
| Stack | Area memori LIFO buat variabel lokal, parameter, dan kontrol alur |
| Heap | Area memori buat alokasi dinamis selama program jalan |
| NOP | Instruksi "no operation", nggak ngapa-ngapain, cuma lanjut ke instruksi berikutnya |

## Isi

### Tiga Level Abstraksi
Dalam arsitektur komputer tradisional, sebuah sistem komputer bisa direpresentasikan sebagai beberapa **level abstraksi** yang bikin cara buat nyembunyiin detail implementasinya. Contohnya, kamu bisa jalanin Windows OS di banyak jenis hardware berbeda, karena hardware di bawahnya diabstraksi dari OS-nya.

**Tiga level coding yang terlibat dalam analisis malware:**
- **Malware author** bikin program di level bahasa tingkat tinggi
- Terus pake **compiler** buat generate machine code yang dijalanin CPU
- **Malware analyst dan reverse engineer** kerja di level bahasa tingkat rendah — kita pake **disassembler** buat generate kode assembly yang bisa kita baca dan analisis buat ngerti gimana program itu beroperasi

> [!info] Analogi
> Bayangin tiga level ini kayak **jenjang penerjemahan surat**. Level tinggi itu surat aslinya dalam bahasa yang kamu ngerti (source code). Compiler itu penerjemah yang ngubah surat itu jadi kode Morse (machine code) yang cuma dimengerti si penerima (CPU). Reverser itu orang yang NEMU kode Morse-nya doang, terus pake alat khusus (disassembler) buat nerjemahin balik Morse itu jadi teks yang bisa dibaca (assembly) — tapi **teks Morse-yang-diterjemahin-balik nggak akan sepersis surat aslinya**, karena banyak konteks (nama variabel, komentar, struktur) yang udah ilang di proses compile.

### x86 Architecture
Assembly language sebenernya sekeluarga bahasa. Tiap dialek assembly biasanya dipake buat program satu keluarga mikroprosesor, kayak **x86, x64, SPARC, PowerPC, MIPS, dan ARM**. **x86 itu arsitektur paling populer buat PC.** Sebagian besar komputer pribadi 32-bit itu x86, dikenal juga sebagai Intel IA-32, dan semua versi 32-bit modern Microsoft Windows dirancang buat jalan di arsitektur x86. Tambahan lagi, sebagian besar arsitektur AMD64 atau Intel 64 yang jalanin Windows mendukung biner x86 32-bit.

Internal sebagian besar arsitektur komputer modern (termasuk x86) ngikutin **arsitektur Von Neumann** dengan tiga komponen hardware:
- **CPU (central processing unit)** — eksekusi kode
- **RAM (main memory)** — nyimpen semua data dan kode
- **I/O (input/output system)** — antarmuka ke perangkat kayak hard drive, keyboard, monitor

**Control unit** ngambil instruksi buat dieksekusi dari RAM pake register (**instruction pointer**), yang nyimpen alamat instruksi yang mau dieksekusi. **Register** itu unit penyimpanan data dasar CPU, sering dipake biar CPU nggak perlu akses RAM (yang lebih lambat). **ALU (arithmetic logic unit)** ngejalanin instruksi yang diambil dari RAM dan naruh hasilnya di register atau memori. Proses ambil-dan-jalanin instruksi demi instruksi ini berulang selama program jalan.

### Main Memory
Layout memori dasar buat sebuah program:

- **Data** — nilai yang ditaruh pas program pertama kali dimuat. Kadang disebut **nilai statis** karena bisa aja nggak berubah selama program jalan. Bisa juga disebut **nilai global** karena tersedia buat semua bagian program.
- **Code** — instruksi yang diambil CPU buat ngejalanin tugas program. Code ngontrol apa yang dilakuin program dan gimana tugas-tugasnya diatur.
- **Heap** — dipake buat memori dinamis selama eksekusi program, buat bikin (alokasi) nilai baru dan ngilangin (bebasin) nilai yang program udah nggak butuhin. Bisa juga disebut nilai global karena tersedia buat semua bagian program.
- **Stack** — dipake buat variabel lokal dan parameter fungsi, dan buat bantu ngontrol alur program.

> [!info] Analogi
> **Data itu papan pengumuman kantor** — isinya bisa dibaca semua orang, jarang berubah. **Heap itu gudang barang** — kamu minta ruang ("alokasi") kapan pun butuh, dan harus balikin ("bebasin") kalau udah selesai, kalau lupa balikin, gudangnya penuh sampah (**memory leak**). **Stack itu tumpukan nampan di kantin** — nampan terakhir yang ditaruh, itu yang pertama diambil (LIFO), dan tiap orang (fungsi) yang lagi makan cuma pegang nampannya sendiri (variabel lokal).

### Instructions, Opcodes, Operands
**Instruksi** itu building block dari program assembly. Di x86 assembly, sebuah instruksi terdiri dari **mnemonic** dan nol atau lebih **operand**.

Tiap instruksi berkorespondensi ke **opcode** (operation code) yang ngasih tau CPU operasi apa yang mau dijalanin program. **Disassembler nerjemahin opcode jadi instruksi yang bisa dibaca manusia.** Contoh dari slide: nilai `0xB9` berkorespondensi ke `mov ecx`, dan `0x42000000` berkorespondensi ke nilai `0x42`.

**Operand** dipake buat ngenalin data yang dipakai sebuah instruksi. Tiga jenis operand:
- **Immediate operand** — nilai tetap (contoh: `0x42`)
- **Register operand** — ngerujuk ke register (contoh: `ecx`)
- **Memory address operand** — ngerujuk ke alamat memori yang berisi nilai yang dituju, biasanya ditandain lewat nilai, register, atau persamaan di dalam kurung siku, kayak `[eax]`

### Registers
**Register** itu sejumlah kecil penyimpanan data yang tersedia buat CPU, isinya bisa diakses lebih cepet dibanding penyimpanan yang tersedia di tempat lain. Prosesor x86 punya kumpulan register buat dipake sebagai penyimpanan sementara atau workspace.

**Empat kategori register:**
- **General registers** — dipake CPU selama eksekusi
- **Segment registers** — dipake buat nge-track bagian memori
- **Status flags** — dipake buat bikin keputusan
- **Instruction pointers** — dipake buat nge-track instruksi berikutnya yang mau dieksekusi

General registers terbagi jadi 3 grup:
- **Data registers** — empat register data 32-bit dipake buat operasi aritmetika, logika, dan lainnya (**EAX, EBX, ECX, EDX**)
- **Pointer registers** — **EIP, ESP, EBP**
- **Index registers** — dipake buat pengalamatan terindeks, kadang dipake juga dalam penjumlahan/pengurangan (**SI, DI**)

### EIP, the Instruction Pointer
Di arsitektur x86, **EIP** (dikenal juga sebagai instruction pointer atau program counter) adalah register yang berisi **alamat memori dari instruksi berikutnya yang mau dieksekusi** buat sebuah program. **Satu-satunya tujuan EIP adalah ngasih tau prosesor apa yang harus dilakuin berikutnya.**

**Kalau EIP dirusak** (artinya, dia nunjuk ke alamat memori yang nggak berisi kode program yang sah), CPU nggak akan bisa eksekusi, jadi program yang lagi jalan kemungkinan besar crash.

**Ketika kamu mengontrol EIP, kamu bisa mengontrol apa yang dieksekusi CPU. Penyerang harus punya kode serangan di memori dan kemudian mengubah EIP supaya nunjuk ke kode itu, buat mengeksploitasi sistem, seperti lewat Buffer Overflow.**

> [!info] Konteks tambahan (bukan dari slide)
> Kalimat terakhir itu adalah **inti dari SELURUH matkul ini** — semua materi tentang buffer overflow, ROP, dan format string exploitation ([[P07 - Stack Security and Exploitation]] sampai [[P13 - Format String Exploitation]]) pada akhirnya adalah berbagai CARA buat mengontrol EIP (atau RIP di 64-bit) supaya CPU ngejalanin instruksi pilihan penyerang, bukan instruksi yang dimaksud programmer. Hafalkan konsep ini sekarang; semuanya akan kembali ke sini.

### Simple Instructions: `mov`
Instruksi paling sederhana dan paling umum adalah **`mov`**, dipakai buat mindahin data dari satu lokasi ke lokasi lain. Dengan kata lain, itu instruksi buat baca dan tulis ke memori.

Dalam sebuah instruksi assembly, operand yang dikurung siku menandakan **referensi memori ke data**. Contoh: `[ebx]` mereferensikan data DI alamat memori EBX. Melakukan kalkulasi persamaan di dalam sebuah instruksi cuma mungkin kalau lagi menghitung alamat memori. Misalnya, `mov eax, ebx+esi*4` (tanpa kurung siku) itu instruksi yang TIDAK VALID. Instruksi yang valid seharusnya `mov eax, [ebx+esi*4]`.

### Arithmetic
x86 assembly punya banyak instruksi buat aritmetika, dari penjumlahan dan pengurangan dasar sampai operator logika. Penjumlahan atau pengurangan menambah/mengurangi nilai dari operand tujuan. Format instruksi penjumlahan: `add destination, value`. Format instruksi pengurangan: `sub destination, value`.

**Instruksi `sub` mengubah dua flag penting: zero flag (ZF) dan carry flag (CF).** ZF di-set kalau hasilnya nol, dan CF di-set kalau tujuannya lebih kecil dari nilai yang dikurangkan.

### NOP
**`nop`, tidak melakukan apa-apa.** Ketika dieksekusi, eksekusi cuma lanjut ke instruksi berikutnya. Instruksi `nop` sebenarnya adalah samaran untuk `xchg eax, eax`, tapi karena menukar EAX dengan dirinya sendiri nggak ngapa-ngapain, dia populer disebut NOP (no operation).

**Ini umum dipakai dalam "NOP sled" untuk serangan buffer overflow**, ketika penyerang belum punya kontrol sempurna atas eksploitasi mereka. NOP menyediakan padding eksekusi, yang mengurangi risiko shellcode jahat mulai dieksekusi di TENGAH-TENGAH, dan karenanya malfungsi.

> [!info] Analogi
> Bayangin **NOP sled** kayak **karpet merah yang panjang menuju panggung**. Kamu nggak yakin persis di mana tamu VIP (eksekusi program) bakal mendarat kalau dia lompat dari pesawat (buffer overflow), tapi asal dia mendarat DI MANA SAJA di karpet merah itu, dia bakal jalan lurus ke panggung (shellcode) di ujungnya. Karpetnya (deretan `nop`) nggak ngapa-ngapain sendiri — fungsinya cuma "kalau eksekusi nyasar di sini, tetep aja lanjut jalan ke tujuan yang benar".

## Diagram & Visual
- **Slide 4 — diagram tiga level abstraksi (high-level → compiler → machine code, dan malware analyst di jalur disassembler → assembly)**
  ![[99-Assets/REBinex/P02-slide04.png]]
- **Slide 13 — tabel/diagram jenis-jenis operand (immediate/register/memory address)**
  ![[99-Assets/REBinex/P02-slide13.png]]
- **Slide 21 — contoh instruksi Multiplication and Division**
  ![[99-Assets/REBinex/P02-slide21.png]]
- **Slide 22 — contoh instruksi Shifting Arithmetic**
  ![[99-Assets/REBinex/P02-slide22.png]]

> [!warning] Deck ini punya total 10 gambar; sebagian besar (slide 6, 8-9, 10-11, 14-19, dll) adalah ilustrasi diagram pendukung yang isinya udah lengkap dijelaskan di teks note ini. Yang paling penting untuk dilihat langsung adalah slide 21-22 (contoh instruksi multiplication/division dan shifting) karena isinya kode yang nggak ada padanan teksnya sama sekali di slide.

## Rumus / Sintaks
```
mov ecx, 0x42          ; pindahin nilai immediate 0x42 ke register ecx
mov eax, [ebx]          ; pindahin data DI ALAMAT ebx ke eax (bukan nilai ebx itu sendiri)
mov eax, [ebx+esi*4]    ; hitung alamat, lalu ambil data di sana (VALID)
mov eax, ebx+esi*4      ; INVALID -- kalkulasi tanpa kurung siku cuma boleh untuk alamat

add destination, value  ; destination = destination + value
sub destination, value  ; destination = destination - value  (ubah ZF dan CF)

nop                     ; sama dengan xchg eax, eax -- literal tidak melakukan apa-apa
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **EAX, EBX, ECX, EDX** | Empat data register 32-bit x86 untuk operasi aritmetika/logika umum |
| **ESP** | Stack pointer, nunjuk ke "puncak" stack saat ini |
| **EBP** | Base pointer, nunjuk ke "dasar" stack frame fungsi yang sedang aktif |
| **Zero Flag (ZF) / Carry Flag (CF)** | Flag status yang otomatis di-set instruksi aritmetika tertentu, dipakai untuk pengambilan keputusan (percabangan) |
| **Disassembler** | Tool yang menerjemahkan opcode biner jadi teks assembly yang bisa dibaca |
| **Shellcode** | Kode mesin kecil (sering dalam bentuk biner mentah) yang disisipkan penyerang untuk dieksekusi lewat kerentanan |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan instruksi `mov eax, ebx` dan `mov eax, [ebx]`. Analisis: apa perbedaan HASIL dari dua instruksi ini, dan gimana kesalahan mengetik kurung siku (lupa atau salah taruh) bisa mengubah total perilaku sebuah program?
2. **(C4 – Analisis)** Slide 17 bilang "when you control EIP, you can control what is executed by the CPU." Analisis: kenapa EIP jadi TARGET UTAMA penyerang, bukan register lain seperti EAX atau EBX? Apa yang membuat EIP secara struktural berbeda dari register data biasa?
3. **(C5 – Evaluasi)** Sebuah mahasiswa bilang: "NOP sled itu nggak penting, penyerang cuma perlu tau alamat PERSIS shellcode-nya dan langsung lompat ke situ." Evaluasi klaim ini — dalam kondisi dunia nyata apa (terkait ASLR atau ketidakpastian alamat) NOP sled tetap berguna meskipun penyerang "seharusnya" tau alamat pastinya?
4. **(C5 – Evaluasi)** Tiga level abstraksi (high-level → machine code → assembly) diajarkan sebagai satu arah (compiler mengubah high-level jadi machine code). Evaluasi: kenapa reverser HARUS kerja di arah SEBALIKNYA (dari machine code balik ke assembly, bukan balik ke high-level code persis aslinya)? Apa yang HILANG secara permanen di proses compile yang bikin proses "dekompilasi sempurna" itu nggak mungkin?
5. **(C6 – Cipta)** Rancang sepotong kode assembly x86 sederhana (pakai instruksi yang udah dipelajari: `mov`, `add`, `sub`, `nop`) yang menghitung `hasil = (a + 5) - b`, dengan `a` disimpan di EAX dan `b` di EBX di awal. Tulis instruksinya baris per baris, dan jelasin kenapa kamu pilih urutan operasi tersebut.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_REBinex]]
- [[P01 - The Reversing Journey]]
- [[P03 - Assembly and Disassembly - The Practical Journey]]
- [[P08 - Deep Dive into Buffer Overflow]]
- [[REBinex - Review dan Glosari]]
