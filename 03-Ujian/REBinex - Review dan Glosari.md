---
matkul: Reverse Engineering dan Binary Exploitation
sks: 3
sumber: rangkuman dari P01-P13
tags: [kuliah/rebinex, ujian]
status: draft
diproses: 2026-09-04
---

# Reverse Engineering dan Binary Exploitation — Review dan Glosari

Rangkuman lintas pertemuan P01–P13. Dibaca sebelum ujian, bukan pengganti note pertemuan — tiap note P01–P13 udah punya **Istilah Khusus** dan **Quiz Pemahaman** sendiri, jadi file ini fokus ke hal yang cuma keliatan kalau ngeliat SEMUA minggu sekaligus.

---

## Bagian 1 — Review Cepat

### Peta besar: empat teknik eksploitasi, satu prinsip yang sama

Semua teknik di P10–P13 pada akhirnya adalah cara BERBEDA buat mengontrol **eksekusi program** (EIP/RIP), lewat celah masuk yang beda-beda:

| Teknik | Cara masuk | Target akhir | Butuh shellcode? |
| --- | --- | --- | --- |
| **Ret2win** (P10) | Buffer overflow, overwrite Old EIP | Fungsi yang SUDAH ADA di binary (`win()`, `give_shell()`) | Tidak |
| **ROP** (P11) | Buffer overflow + rangkaian gadget | Fungsi/urutan instruksi buatan sendiri dari potongan kode yang ada | Tidak |
| **Ret2libc** (P12) | Buffer overflow, overwrite Old EIP/RIP | `system()` di C library, dengan argumen `/bin/sh` | Tidak |
| **Shellcode injection klasik** (P08) | Buffer overflow, taruh kode BARU di buffer | Kode jahat yang disuntikkan sendiri oleh penyerang | **Ya** |
| **Format string** (P13) | `printf()` yang menerima string dari user langsung | Baca/tulis memori SEMBARANGAN | Tidak (bug beda sama sekali) |

**Kenapa 4 dari 5 teknik "tidak butuh shellcode":** karena proteksi **NX** (P07) membuat data di stack TIDAK BISA dieksekusi sebagai kode. Ret2win, ROP, dan ret2libc semuanya adalah cara MENGAKALI NX dengan cuma memakai kode yang SUDAH ADA dan SUDAH dieksekusi secara sah (di binary atau di libc) — bukan menyuntik kode baru.

### Urutan proteksi keamanan (checksec) dan cara masing-masing "dikalahkan"

| Proteksi | Fungsinya | Diakali dengan |
| --- | --- | --- |
| **NX** | Data tidak bisa dieksekusi sebagai kode | Ret2win/ROP/ret2libc (pakai kode yang SUDAH ADA, bukan kode baru) |
| **Stack Canary** | Deteksi overflow sebelum Old EIP tertimpa | Perlu di-bypass/dibocorkan dulu (lewat format string, misalnya) sebelum overflow lanjut |
| **ASLR** | Acak alamat memori tiap run | Matikan dulu untuk belajar (`randomize_va_space=0`), atau bocorkan alamat lewat kerentanan lain (format string arbitrary read) |
| **PIE** | Binary tidak punya alamat basis tetap | pwntools `elf.address` otomatis menyesuaikan; atau ASLR dimatikan |
| **RELRO** | GOT (dan PLT kalau Full) jadi read-only | Tidak dibahas cara mengakalinya secara eksplisit di deck manapun — catat sebagai celah materi |

### Trio "mencari alamat manual" (dipakai di P10 dan P12)
```
ldd     -> base address library (libc, dsb.)
readelf -> offset FUNGSI dari base (contoh: system() dari base libc)
strings -> offset STRING dari base (contoh: "/bin/sh" dari base libc)
radare2 (r2 -d -A, afl) -> alamat FUNGSI di dalam binary itu sendiri (untuk ret2win)
rabin2  -> cek endianness binary (little vs big-endian)
```

### Konsep stack yang WAJIB hafal (dari P02, P03, P08)
```
Urutan stack, dari alamat TINGGI ke RENDAH:
  [parameter fungsi]     <- ditulis kanan-ke-kiri, DI ATAS EBP
  [Old EIP]               <- alamat kembali (RETURN ADDRESS -- target utama exploit)
  [Old EBP]               <- EBP fungsi pemanggil
  --- EBP menunjuk di sini ---
  [variabel lokal]         <- DI BAWAH EBP
  --- ESP menunjuk di sini (puncak stack) ---
```
**EIP = alamat instruksi BERIKUTNYA yang dieksekusi. Kontrol EIP = kontrol CPU.** Ini kalimat tunggal paling penting di seluruh matkul (P02), dan menjelaskan HAMPIR SEMUA teknik eksploitasi minggu-minggu berikutnya.

### Trio penyembunyian info (P04, disambungkan ke seluruh matkul)
```
Stripping        : hapus NAMA fungsi/variabel dari binary (info debugging)
Code encryption   : enkripsi kode, didekripsi sendiri saat runtime (bisa "dicuri" dari memori)
Stackstring       : sembunyikan string dengan menulisnya 1 byte per instruksi MOV ke stack
```

### Gadget dan ROP chaining (P11), inti calling convention
```
32-bit : argumen fungsi lewat STACK      -- kontrol stack = kontrol argumen (LANGSUNG)
64-bit : argumen fungsi lewat REGISTER   -- butuh GADGET (misal "pop rdi; ret") untuk
                                            memindahkan nilai dari stack KE register
```

### Format string, pola serangan (P13)
```
%x, %p         : bocorkan NILAI dari stack (yang kebetulan ada di sana)
%N$x, %N$p     : akses LANGSUNG parameter ke-N (efisien, tidak perlu %x berulang)
%s             : baca STRING di ALAMAT yang ditunjuk sebuah nilai -- kalau alamatnya
                  bisa kamu KONTROL (kamu tulis sendiri di awal input), ini jadi
                  ARBITRARY READ (baca memori di alamat manapun yang kamu mau)
%n (disinggung, tidak dijelaskan detail) : tulis jumlah karakter tercetak ke sebuah
                  alamat -- dasar ARBITRARY WRITE
```
**Perangkap null byte**: `printf`/`%s` berhenti di byte bernilai 0. File ELF dimulai dengan `\x7fELF` (byte pertama `\x7f`, BUKAN null — tapi struktur data lain seperti alamat 32-bit sering punya byte null di tengahnya kalau alamatnya kecil), jadi urutan penulisan format specifier vs alamat targetnya harus diperhatikan (format specifier duluan, baru alamat).

---

### Ringkasan satu paragraf per pertemuan

**[[P01 - The Reversing Journey]]** — RE = narik ilmu/blueprint dari sesuatu yang sudah jadi. Dipakai di dua sisi pagar sama-sama: pembuat malware (cari celah) dan pembuat antivirus (bedah malware). Juga dipakai buat mengalahkan proteksi copy, membongkar kripto, dan DRM. Assembly = sekeluarga bahasa (bukan satu bahasa), tergantung platform. Compiler bisa hasilkan machine code (platform-specific) atau bytecode (platform-independent, butuh virtual machine). Tiga kategori tool: system monitoring, disassembler (IDA Pro, Ghidra, Radare2), debugger (GDB, WinDbg). Legalitas RE abu-abu di AS, tapi diizinkan di Uni Eropa untuk tujuan interoperabilitas.

**[[P02 - Assembly and Disassembly]]** — Tiga level abstraksi: high-level → compiler → machine code, dan reverser kerja lewat disassembler → assembly. x86 = arsitektur PC paling populer, ikut Von Neumann (CPU, RAM, I/O). Memori: Data (statis/global), Code, Heap (dinamis), Stack (lokal + kontrol alur). Instruksi = mnemonic + operand (immediate/register/memory address). Register: general (EAX-EDX), pointer (EIP/ESP/EBP), segment, status flags. **EIP = alamat instruksi berikutnya — kontrol EIP = kontrol CPU**, dasar SEMUA eksploitasi buffer overflow. `mov` = instruksi paling dasar. NOP dipakai di "NOP sled" untuk buffer overflow.

**[[P03 - Assembly and Disassembly - The Practical Journey]]** — Stack = LIFO, instruksinya push/pop/call/leave/enter/ret. `main(argc, argv)` — argv[1] diakses dengan +4 byte offset (32-bit). Global variable = alamat memori TETAP (`dword_XXXXXX`); local variable = offset RELATIF ke EBP (`[ebp-4]`) — ini pembeda paling gampang di assembly. Compiler bisa pilih `sub`/`add` alih-alih `inc`/`dec` meski secara logis lebih rumit — assembly hasil compile sering tidak intuitif. Pola if/else di assembly: `cmp` → conditional jump (`jnz`) → kode → `jmp` (lompati else) → kode else.

**[[P04 - Obfuscation Techniques]]** — Tiga pendekatan anti-reversing: hapus symbolic information, obfuscate/encrypt program, sisipkan kode anti-debugger. C/C++ release build otomatis tanpa simbol; bytecode (Java) WAJIB simpan nama internal untuk cross-referencing, jadi lebih mudah didekompilasi. Stripping (`strip`) hapus info debugging. Enkripsi program: kunci dan logika dekripsi WAJIB ada di dalam executable-nya sendiri, dan versi terdekripsi HARUS ada di memori saat runtime — bisa "dicuri langsung dari RAM". Unpacker otomatis dekripsi program yang di-pack. Stackstring: sembunyikan string dengan menulisnya 1 byte per instruksi MOV — dibongkar dengan retype ke `char[n]` di Ghidra.

**[[P05 - Debugging and Dynamic Analysis]]** — Dynamic analysis (jalankan program) beda dari static analysis (baca doang) — bisa mengungkap perilaku SEBENARNYA. GDB: compile dengan `-g`, lalu `run`, `break file:line` / `break func`, `continue`, `step` (masuk sub-rutin) vs `next` (lompati), `print`/`print/x`, `watch` (pause tiap variabel berubah — beda dari breakpoint yang pause di LOKASI), `backtrace`/`where` (stack trace crash).

**[[P06 - Binary Patching]]** — Patch pendek bisa inline; patch panjang butuh **code cave** (ruang kosong di binary, sering dari byte alignment). Contoh praktik: `shellcode2.exe` — ganti instruksi `MOV` (ambil alamat HASH) jadi `LEA` (ambil alamat FLAG ASLI) lewat fitur "Patch Instruction" di Ghidra, lalu export ulang sebagai PE. Offset harus disesuaikan (`-0x2c` → `-0x28`) karena ada extra push EBP lama di awal fungsi.

**[[P07 - Stack Security and Exploitation]]** — Checksec = "pemeriksaan kesehatan" binary: NX (data tidak bisa dieksekusi), PIE (binary tanpa alamat tetap, syarat ASLR), ASLR (acak alamat tiap run), RELRO (GOT read-only setelah linking — Partial: GOT saja, Full: GOT+PLT), Stack Canary (nilai acak sebelum Old EIP, deteksi overflow SEBELUM data penting tertimpa), Fortify Source (bounds-checking otomatis compiler). Buffer overflow = input melebihi kapasitas buffer, menimpa memori sekitarnya.

**[[P08 - Deep Dive into Buffer Overflow]]** — EBP (base stack fungsi aktif), ESP (puncak stack), EIP (instruksi berikutnya). Stack diisi tinggi ke rendah, semua variabel relatif ke EBP. Urutan: parameter (di atas EBP) → Old EIP → Old EBP → variabel lokal (di bawah EBP). Sepuluh dangerous C functions: `strcpy`, `gets`, `sprintf`, dll. Overflow yang menembus sampai Old EIP dan menimpanya dengan alamat kode jahat = kontrol eksekusi program. Pola injeksi klasik: padding → shellcode → return address menunjuk balik ke shellcode (sering dibantu NOP sled).

**[[P09 - Exploiting with Pwntools]]** — Pwntools = library Python CTF dari Gallopsled, biar exploit developer tidak nulis ulang tool yang sama. I/O: `recv`/`recvline`/`recvuntil`, `send`/`sendline`. `context` = variabel global, otomatis dipakai fungsi lain (arch, endianness). Kelas ELF: `elf.search()` (cari string), `elf.address` (alamat basis, otomatis sesuaikan PIE/ASLR). `p64()`/`u64()` pack/unpack integer sesuai `context`.

**[[P10 - Ret2win Exploitation]]** — Overwrite saved EIP ke alamat fungsi yang SUDAH ADA tapi tidak pernah dipanggil (`win()`/`give_shell()`). Butuh: padding (trial-error manual, atau **De Bruijn sequence** — tidak ada pola n-karakter berulang, jadi offset langsung ketemu dari posisi match) dan alamat tujuan (radare2: `r2 -d -A`, lalu `afl`). Endianness krusial: little-endian (byte tidak signifikan duluan) vs big-endian, dicek pakai `rabin2`.

**[[P11 - Return Oriented Programming]]** — ROP = rangkai potongan kecil assembly (**gadget**) lewat kontrol stack, untuk hal lebih kompleks dari ret2win. 32-bit: argumen lewat stack langsung (susun seolah `system()` dipanggil normal: alamat PLT → fake return → argumen). 64-bit: argumen lewat register (RDI, RSI, dst — calling convention), butuh gadget seperti `pop rdi; ret` yang dirangkai (chained) lewat `ret` yang otomatis "jatuh" ke gadget berikutnya di stack. Tool: rp++, ROPgadget.

**[[P12 - Ret2libc Exploitation]]** — Serangan TANPA shellcode, cocok untuk binary dengan NX aktif. Manfaatkan `system()` (sudah ada di libc, bisa jalankan APA PUN) + string `/bin/sh` (juga sudah ada di libc). Cari base libc (`ldd`), offset `system()` (`readelf -s`), offset `/bin/sh` (`strings -a -t x`). 32-bit: argumen langsung di stack setelah return address. 64-bit: butuh gadget `pop rdi; ret` (dari P11). Pwntools otomatisasi penuh: `libc.sym['system']`, `libc.search(b'/bin/sh')`.

**[[P13 - Format String Exploitation]]** — Bug beda sama sekali dari buffer overflow: `printf()` mengharap sebanyak parameter sesuai format specifier; kalau kurang, dia ambil nilai BERIKUTNYA dari stack — bocorkan data. Kalau string yang dikontrol USER langsung jadi format string, penyerang bisa masukkan `%x`/`%s` sendiri. `%N$x` = akses langsung parameter ke-N (efisien). `%s` = baca STRING di alamat yang ditunjuk nilai stack — kalau alamatnya bisa dikontrol penyerang, jadi **arbitrary read**. Perangkap: `printf`/`%s` berhenti di null byte, jadi urutan format specifier vs alamat target penting.

---

### Pola yang muncul berulang

**"Kontrol EIP = kontrol CPU"** (P02) adalah kalimat tunggal yang menjelaskan hampir seluruh sisa semester — P08 (overwrite Old EIP), P10 (ret2win), P11 (ROP), P12 (ret2libc) semuanya adalah CARA BERBEDA mencapai hal yang sama: bikin EIP/RIP menunjuk ke alamat pilihan penyerang.

**Stack-relative addressing** (EBP sebagai "titik nol" lokal) muncul di P02 (konsep), P03 (praktik C-ke-assembly), P06 (perhitungan offset patch), P08 (urutan lengkap parameter/Old EIP/Old EBP/lokal), dan P10 (padding). Kalau kamu paham urutan stack sekali dengan benar, semua materi eksploitasi setelahnya jadi soal "di mana persisnya X berada relatif ke Y".

**Manual dulu, baru otomatisasi.** Pola pedagogis matkul ini konsisten: tiap teknik (cari padding di P10, cari gadget di P11, cari alamat libc di P12) diajarkan CARA MANUALNYA dulu (GDB, radare2, `ldd`/`readelf`/`strings`), BARU ditunjukkan cara pwntools mengotomatisasinya. Ini penting untuk ujian — kalau ditanya "gimana cara kerja `libc.sym['system']`", jawabannya harus merujuk ke proses manual `ldd`+`readelf` yang diwakilinya.

**NX sebagai poros perubahan strategi.** Sebelum P07 (NX diperkenalkan), pendekatan "wajar" untuk exploit adalah menyuntik shellcode baru (P08). Setelah P07, TIGA teknik berikutnya (P10, P11, P12) semuanya dirancang khusus untuk BEKERJA MESKIPUN NX aktif, dengan cara TIDAK PERNAH mengeksekusi data baru — cuma memakai ulang kode yang sudah ada dan sudah "halal" dieksekusi.

---

## Bagian 2 — Glosari Gabungan

Istilah lengkap ada di section **Istilah Khusus** tiap note P01–P13. Ini cuma istilah yang PALING SERING nongol lintas minggu.

| Istilah | Arti singkat | Muncul di |
| --- | --- | --- |
| **EIP / RIP** | Instruction pointer; alamat instruksi berikutnya yang dieksekusi (32-bit / 64-bit) | P02, P08, P10, P11, P12 |
| **EBP / ESP** | Base pointer (dasar stack frame) / stack pointer (puncak stack saat ini) | P02, P03, P08 |
| **Old EIP / Old EBP** | Return address dan EBP fungsi pemanggil, disimpan di stack frame | P03, P06, P08 |
| **Stack frame** | Ruang di stack yang dibuat tiap kali sebuah fungsi dipanggil | P02, P03, P08 |
| **Opcode / Operand** | Kode operasi CPU / data yang dipakai sebuah instruksi | P02 |
| **`mov` / `lea`** | Pindahkan NILAI dari satu lokasi ke lokasi lain / ambil ALAMAT sebuah data | P02, P06 |
| **NOP / NOP sled** | Instruksi "tidak melakukan apa-apa" / deretan NOP untuk memperbesar peluang shellcode "mendarat" | P02, P08 |
| **Stripping** | Menghapus info debugging (nama fungsi/variabel) dari binary | P04, P07 |
| **Packer / Unpacker** | Program yang mengenkripsi/mendekripsi ulang sebuah executable | P04 |
| **Stackstring** | Menyembunyikan string dengan menulisnya 1 byte per instruksi MOV ke stack | P04 |
| **GDB** | GNU Debugger; breakpoint (lokasi), watchpoint (variabel), step/next | P05 |
| **Code cave** | Ruang kosong di binary tempat kode patch bisa disisipkan | P06 |
| **Checksec** | Tool untuk memeriksa proteksi aktif di sebuah binary | P07 |
| **NX** | Memisahkan memori kode dan data; data tidak bisa dieksekusi | P07 |
| **PIE / ASLR** | Binary tanpa alamat tetap / pengacakan alamat memori tiap eksekusi | P07, P09, P10, P12 |
| **RELRO (Partial/Full)** | GOT (dan PLT kalau Full) dibuat read-only setelah linking | P07 |
| **Stack Canary** | Nilai acak untuk mendeteksi buffer overflow sebelum data penting tertimpa | P07, P08 |
| **Buffer overflow** | Input melebihi kapasitas buffer, menimpa memori di sekitarnya | P07, P08 |
| **Dangerous C functions** | `strcpy`, `gets`, `sprintf`, dll yang tidak mengecek batas buffer | P08 |
| **Pwntools** | Library Python untuk pengembangan exploit CTF | P09, P10, P11, P12 |
| **`context`** | Variabel global pwntools, otomatis dipakai fungsi lain | P09 |
| **ELF class (`elf.address`, `elf.search`)** | Fitur pwntools untuk lookup alamat/simbol secara dinamis | P09, P12 |
| **`p32()`/`p64()` / `u32()`/`u64()`** | Pack/unpack integer jadi/dari bentuk byte, sesuai `context` | P09, P12 |
| **Ret2win** | Overwrite return address ke fungsi "kemenangan" yang sudah ada | P10 |
| **De Bruijn sequence** | String tanpa pola n-karakter berulang, untuk mencari offset dengan cepat | P10 |
| **radare2 (`r2`, `afl`, `rabin2`)** | Tool reverse engineering: analisis fungsi dan endianness binary | P10 |
| **Endianness** | Urutan byte suatu nilai multi-byte (little vs big-endian) | P09, P10 |
| **Gadget** | Potongan kecil instruksi diakhiri `ret`, blok bangunan ROP | P11, P12 |
| **`pop rdi; ret`** | Gadget umum untuk mengontrol register argumen pertama di 64-bit | P11, P12 |
| **Ret2libc** | Serangan tanpa shellcode, manfaatkan `system()`+`/bin/sh` di libc | P12 |
| **`ldd` / `readelf` / `strings`** | Trio perintah untuk mencari base address / offset fungsi / offset string secara manual | P12 |
| **Format specifier (`%x`, `%s`, `%N$x`)** | Placeholder di `printf` yang diganti nilai argumen; jadi celah kalau string-nya dikontrol user | P13 |
| **Arbitrary read/write** | Kemampuan membaca/menulis memori di alamat pilihan penyerang | P13 |

---

## Bagian 3 — Yang Perlu Dicek Sendiri

1. **Semua deck P02-P13 punya Learning Outcome yang salah** (menyebut "File Carving" — topik forensik yang tidak berhubungan sama sekali). Ini kesalahan copy-paste dosen yang konsisten, bukan indikasi materi yang hilang — konten SLIDE-nya tetap sesuai judul masing-masing deck. Tapi ini berarti **tidak ada cara memverifikasi dari LO** apakah semua yang seharusnya diajarkan sudah tercakup — LO-nya sendiri tidak bisa dipakai sebagai checklist.
2. **RELRO (P07) tidak pernah dijelaskan cara "dikalahkan"/dieksploitasi** di deck manapun — beda dari NX, ASLR, dan Canary yang masing-masing punya teknik pengelakan yang eksplisit dibahas (ret2win/ROP/ret2libc untuk NX; disable/bocorkan alamat untuk ASLR; dst). Kalau ujian menanyakan cara mengeksploitasi RELRO, materinya belum ada.
3. **P13 menyinggung "arbitrary write" sebagai tantangan akhir** (menimpa variabel `auth` jadi 10) TAPI TIDAK PERNAH menjelaskan mekanisme `%n` (format specifier yang jadi dasar arbitrary write) secara eksplisit — cuma disebut sebagai latihan terbuka tanpa solusi. Materi arbitrary write secara teknis belum lengkap.
4. **P05 slide 15 ("UPX Packing") kosong total** — bukan gagal ekstrak, memang tidak ada isinya di file PPTX. Kalau demo UPX packing pernah didemokan di kelas, catatannya tidak ada di slide manapun.
5. **Format 64-bit calling convention** (P11) cuma menyebut RDI dan RSI (dua argumen pertama) — tidak menjelaskan register untuk argumen ketiga dan seterusnya (RDX, RCX, R8, R9 di calling convention System V AMD64 standar). Kalau exploit butuh lebih dari 2 argumen (seperti quiz P11 nomor 5 tentang `write()`), informasi register RDX perlu dicari dari sumber luar.
6. **SKS matkul belum dikonfirmasi** — ditulis `3` di frontmatter sebagai asumsi administratif, cek dan koreksi manual.
7. **Nama dosen dan jadwal matkul belum diketahui** — tidak dicantumkan di slide manapun (beda dari Blockchain dan Forensics/Secure Programming yang selalu mencantumkan nama dosennya).

## Terkait
- [[_REBinex]]
