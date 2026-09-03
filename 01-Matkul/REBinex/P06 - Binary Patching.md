---
matkul: Reverse Engineering dan Binary Exploitation
minggu: 6
sks: 3
sumber: REBinex - P6.pptx
tags: [kuliah/rebinex, minggu/p06]
status: draft
diproses: 2026-09-04
---

# P06 — Binary Patching

> [!warning] Learning outcome di slide 2 nggak nyambung sama isi (masih tentang binary patching). Lihat catatan di [[P02 - Assembly and Disassembly]].

## Ringkasan
> - **Binary patching** = ngubah kode/data di dalam program yang UDAH DI-COMPILE, tanpa perlu source code atau compiler-nya. Dipakai buat benerin vulnerability/kompatibilitas kalau source code-nya hilang, sistemnya udah gak didukung lagi — atau (di sisi jahat) buat bikin malware.
> - Patch pendek bisa langsung ditimpa ke kode yang ada (inline). Patch yang lebih panjang butuh **code cave** — ruang KOSONG yang gak kepake di file program, biasanya muncul karena kebutuhan byte alignment sebuah section, atau ruang kosong antar fungsi.
> - Contoh praktik: **`shellcode2.exe`** dari tantangan MalwareTech. Programnya nge-hash sebuah flag rahasia pake MD5 dan cuma nampilin HASH-nya di message box. **Tujuan patch-nya: bikin dia nampilin flag ASLI-nya, sebelum di-hash.**
> - Caranya: ganti instruksi **MOV** (yang ngambil alamat HASH-nya) jadi **LEA** (Load Effective Address, yang ngambil alamat data ASLI-nya sebelum di-hash) — dieksekusi lewat fitur **Patch Instruction** di Ghidra.
> - Setelah dipatch, program di-export ulang jadi format **PE**, lalu dijalanin — hasilnya message box nampilin flag aslinya, bukan hash-nya.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Binary patching | Mengubah kode/data di program yang sudah di-compile, tanpa source code |
| Code cave | Ruang kosong yang tidak dipakai di file executable, tempat kode patch bisa disisipkan |
| LEA | Load Effective Address; instruksi yang mengambil ALAMAT sebuah data (bukan isinya) |
| MOV | Instruksi yang memindahkan NILAI/isi dari satu lokasi ke lokasi lain |
| PE (Portable Executable) | Format file executable standar Windows |
| Two's complement | Cara komputer merepresentasikan bilangan negatif secara biner |

## Isi

### Why Binary Patching?
Ada beberapa keadaan di mana bisa berguna buat bikin modifikasi ke kode atau data di dalam program yang sudah di-compile. Kadang, ini diperlukan untuk memperbaiki sebuah kerentanan atau isu kompatibilitas TANPA source code atau compiler yang fungsional. Ini bisa terjadi ketika source code hilang, sistem sudah tidak didukung lagi, atau **sesederhana kamu ingin membuat sebuah malware yang menghancurkan.**

Langkah pertama untuk menyiapkan sebuah patch program adalah mengukur kompleksitas/panjang patch yang dibutuhkan dan mengidentifikasi kira-kira di mana patch itu perlu disisipkan. **Kalau patch-nya cukup pendek, mungkin bisa langsung mengganti kode yang ada secara inline.** Tapi, patch yang memperkenalkan fungsionalitas yang benar-benar baru umumnya TIDAK BISA ditulis secara inline dan akan butuh strategi berbeda.

### Code Caves
Dalam skenario ini, kita harus mencari **byte-byte yang tidak terpakai** yang dimuat dari file program ke ruang memori executable, disebut **code caves**. Code cave ini umumnya muncul ketika sebuah section executable membutuhkan byte alignment tertentu. **Patch yang lebih panjang bisa ditulis ke dalam code cave** bersamaan dengan instruksi percabangan (branching) yang sesuai untuk menyisipkan kode patch itu ke jalur kode yang tepat.

**Cave** adalah wilayah ruang yang tidak terpakai di binary target. Biasanya, kamu akan menemukan sedikit ruang ekstra di akhir sebuah section. Kamu mungkin beruntung dan menemukan ruang di antara fungsi-fungsi.

> [!info] Analogi
> **Code cave itu kayak ruang kosong di antara bab-bab sebuah buku** — mungkin karena bab sebelumnya berakhir di tengah halaman dan bab baru selalu mulai di halaman baru (karena "aturan format", mirip "byte alignment"), jadi ada sisa ruang putih yang gak kepake. Kamu bisa nulis catatan tambahan di ruang putih itu ("kode patch"), asal kamu juga naruh CATATAN PENGARAH di tempat yang relevan di buku ("ke halaman sekian buat baca catatan tambahan" — instruksi percabangan) biar pembaca (CPU) tau kapan harus mampir baca catatan itu.

### Practical Example: `shellcode2.exe`
Diberikan tantangan **`shellcode2`** dari MalwareTech. Binary-nya adalah sebuah executable yang, ketika dijalankan, akan menampilkan message box berisi **MD5 sum dari sebuah string flag rahasia**. **Tugasnya: bikin sebuah patch untuk mengungkapkan nilai flag itu SETELAH di-decode oleh shellcode dan SEBELUM di-hash.**

Melihat struktur `shellcode2.exe`: di dalamnya terlihat `local_bc` diinisialisasi sebagai objek MD5, diikuti dengan pembentukan sebuah **stack string** (lihat [[P04 - Obfuscation Techniques]] untuk penjelasan lengkap teknik ini). Ketika melihat bagian akhir dari fungsi entry-nya, terlihat di mana flag itu di-hash dan message box-nya dibuat.

**Cara kerja alur di kode aslinya:** objek MD5 di `local_bc` direferensikan untuk memanggil method `MD5::digestString()` dengan alamat `local_2c` sebagai input. Sebuah referensi ke hash hasilnya disimpan di `local_c0`. Instruksi dari `4023a2` sampai `4023b2` mengoper nilai ini ke pemanggilan API `MessageBoxA` dengan judul dan gaya window tertentu.

### Now Let's Patch: `shellcode2.exe`

**Pertanyaannya: gimana caranya kita mencetak flag-nya SEBELUM dihash?** Kita bisa mengubah argumen ke `MessageBoxA` supaya dia mencetak nilai dari `local_2c` (flag ASLI) alih-alih nilai yang direferensikan oleh `local_c0` (HASH-nya). Alamat hash-nya dimuat ke **EAX** dengan instruksi **MOV** di `4023a9` lalu di-push ke stack sebagai argumen untuk `MessageBoxA`. **Ini yang perlu dipatch, supaya alamat `local_2c` yang di-push sebagai gantinya.** Instruksi **LEA (Load Effective Address)** memungkinkan kita melakukan persis itu.

**Langkah-langkah patching:**
1. Mulai dengan **klik-kanan instruksi MOV** dan pilih **Patch Instruction**.
2. Instruksinya akan berubah jadi field yang bisa diedit dengan autocompletion. **Patch ini jadi `LEA` dengan operand `EAX, [EBP + -0x28]`** supaya EAX menerima alamat `local_2c`.
3. Perhatikan bahwa penggunaan `-0x28` (bukan `-0x2c`) sebagai offset ke EBP itu untuk memperhitungkan **EBP asli yang di-push ke stack sebelum EBP dimuat dengan stack pointer baru**. Offset yang dihasilkan dikonversi ke bentuk **two's complement**-nya.
4. Program sekarang bisa diekspor lewat menu **File → Export Program** sebagai format **PE**. Menjalankan file `.exe`-nya menghasilkan `MessageBoxA` baru kita — **flag aslinya, bukan hash-nya, yang tampil.**

> [!info] Konteks tambahan (bukan dari slide)
> Pergeseran offset dari `-0x2c` ke `-0x28` itu contoh bagus kenapa reverse engineering butuh KETELITIAN, bukan cuma pemahaman konsep. `local_2c` namanya "2c" karena itu memang OFFSET-nya relatif ke EBP saat program pertama kali menyimpan struktur stack framenya — tapi begitu kamu tau ada "extra push" (EBP lama yang disimpan di awal fungsi), kamu harus GESER perhitungannya 4 byte. Salah satu byte offset ini dan patch-mu bisa nunjuk ke data yang SALAH SAMA SEKALI, atau bahkan bikin program crash.

## Diagram & Visual
- **Slide 6 — diagram/ilustrasi code cave**
  ![[99-Assets/REBinex/P06-slide06.png]]
- **Slide 8 — struktur `shellcode2.exe`: inisialisasi objek MD5 dan stack string**
  ![[99-Assets/REBinex/P06-slide08.png]]
- **Slide 9 — bagian akhir fungsi entry: hashing flag dan pembuatan message box**
  ![[99-Assets/REBinex/P06-slide09.png]]
- **Slide 12 — proses klik-kanan "Patch Instruction" di Ghidra**
  ![[99-Assets/REBinex/P06-slide12.png]]
- **Slide 13 — field editable dengan autocompletion untuk memasukkan instruksi LEA**
  ![[99-Assets/REBinex/P06-slide13.png]]
  ![[99-Assets/REBinex/P06-slide13b.png]]
- **Slide 14 — konversi offset ke two's complement**
  ![[99-Assets/REBinex/P06-slide14.png]]

> [!warning] Ini deck **tutorial visual step-by-step** — teks slide-nya menjelaskan LOGIKA tiap langkah, tapi tampilan Ghidra yang sesungguhnya (di mana tepatnya klik, gimana bentuk field autocomplete-nya) cuma ada di gambar. Slide 15 (hasil export dan message box baru) tidak berhasil diekstrak sebagai gambar — kalau mau lihat hasil akhirnya, buka PPT aslinya. Referensi lengkap tantangan dan tutorialnya (dari speaker notes dosen): `malwaretech.com/challenges/windows-reversing` dan `tripwire.com/state-of-security/ghidra-101-binary-patching`.

## Rumus / Sintaks
```
; SEBELUM patch (mengambil alamat HASH):
MOV EAX, [alamat_local_c0]     ; ambil alamat hash
PUSH EAX                        ; jadi argumen MessageBoxA

; SESUDAH patch (mengambil alamat FLAG ASLI):
LEA EAX, [EBP + -0x28]          ; ambil alamat local_2c (flag asli, offset digeser dari -0x2c)
PUSH EAX                        ; jadi argumen MessageBoxA -- flag asli yang tampil
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Byte alignment** | Aturan penempatan data di memori pada kelipatan ukuran tertentu, demi efisiensi akses CPU |
| **MessageBoxA** | Fungsi Windows API untuk menampilkan kotak dialog pesan |
| **`MD5::digestString()`** | Method yang menghitung hash MD5 dari sebuah string |
| **Patch Instruction** | Fitur di Ghidra untuk mengedit instruksi assembly secara langsung di dalam binary |
| **PE (Portable Executable)** | Format file `.exe`/`.dll` standar di sistem Windows |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan instruksi `MOV EAX, [alamat]` dan `LEA EAX, [alamat]`. Analisis: kenapa mengganti `MOV` jadi `LEA` (bukan mengganti operand-nya doang, atau menambah instruksi baru) adalah cara paling MINIMAL untuk mencapai tujuan patch di kasus `shellcode2.exe`?
2. **(C4 – Analisis)** Slide 14 menjelaskan offset harus digeser dari `-0x2c` ke `-0x28` karena "EBP asli di-push ke stack sebelum EBP dimuat dengan stack pointer baru". Analisis: hubungkan penjelasan ini dengan konsep stack frame dari [[P03 - Assembly and Disassembly - The Practical Journey]] — instruksi APA (dari materi minggu sebelumnya) yang biasanya bertanggung jawab men-"push EBP lama" ini di awal sebuah fungsi?
3. **(C5 – Evaluasi)** Seorang mahasiswa ingin menambahkan patch yang JAUH LEBIH PANJANG dari sekadar mengganti satu instruksi MOV jadi LEA (misalnya, menambahkan seluruh fungsi validasi baru). Evaluasi: kenapa pendekatan "Patch Instruction" inline yang dipakai di contoh `shellcode2.exe` TIDAK CUKUP untuk kasus ini, dan solusi apa (dari konsep di slide 5-6) yang harus dipakai sebagai gantinya?
4. **(C5 – Evaluasi)** Bandingkan use case binary patching yang "baik" (memperbaiki vulnerability tanpa source code) dengan yang "jahat" (disebut eksplisit di slide 3: "membuat malware yang menghancurkan"). Dari sudut pandang TEKNIK yang dipakai, apakah ada perbedaan sama sekali antara dua tujuan ini? Apa yang sebenarnya membedakan keduanya?
5. **(C6 – Cipta)** Rancang skenario binary patching BARU (berbeda dari `shellcode2.exe`) di mana kamu perlu MENGUBAH LOGIKA PERCABANGAN sebuah program — misalnya, sebuah program yang mengecek `if (password_benar) { buka_pintu(); }` dan kamu ingin membuatnya SELALU membuka pintu terlepas dari passwordnya. Instruksi assembly APA (dari konsep `cmp`/`jnz`/`jmp` di [[P03 - Assembly and Disassembly - The Practical Journey]]) yang akan kamu targetkan untuk dipatch, dan jadi APA kamu akan mengubahnya?

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_REBinex]]
- [[P03 - Assembly and Disassembly - The Practical Journey]]
- [[P04 - Obfuscation Techniques]]
- [[P05 - Debugging and Dynamic Analysis]]
- [[P07 - Stack Security and Exploitation]]
- [[REBinex - Review dan Glosari]]
