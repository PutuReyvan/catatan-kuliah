---
matkul: Reverse Engineering dan Binary Exploitation
sks: 3
dosen: belum dikonfirmasi
jadwal: belum dikonfirmasi
tags: [kuliah/rebinex, moc]
status: draft
diproses: 2026-09-04
---

# Reverse Engineering dan Binary Exploitation (COMP6843001)

Index matkul. Tiga belas deck PPT dari dosen (P1–P13), diproses jadi tiga belas note pertemuan.

**Format note matkul ini SAMA seperti Secure Programming**: bahasa di bagian Isi ditulis santai/gampang dimengerti, banyak analogi, dan tiap note ditutup dengan section **Istilah Khusus** (glosari lokal) dan **Quiz Pemahaman** (Bloom taksonomi C4 ke atas — Analisis, Evaluasi, Cipta).

## Daftar Pertemuan

| # | Note | Isinya |
| --- | --- | --- |
| P01 | [[P01 - The Reversing Journey]] | Apa itu RE, use case (malware/kripto/DRM), assembly vs machine code vs bytecode, kategori tool (monitor/disassembler/debugger), legalitas RE |
| P02 | [[P02 - Assembly and Disassembly]] | Von Neumann architecture, memory layout (data/code/heap/stack), instruksi/opcode/operand, register, EIP, `mov`, aritmetika, NOP |
| P03 | [[P03 - Assembly and Disassembly - The Practical Journey]] | Stack frame, `argc`/`argv`, global vs local variable di assembly, disassembly aritmetika, pengenalan if/else |
| P04 | [[P04 - Obfuscation Techniques]] | Anti-reversing (hapus simbol, enkripsi program, anti-debugger), stripping, packer/unpacker, stackstring |
| P05 | [[P05 - Debugging and Dynamic Analysis]] | Static vs dynamic analysis, GDB (breakpoint, watchpoint, step/next, backtrace) |
| P06 | [[P06 - Binary Patching]] | Kenapa patch binary, code cave, praktik patch `shellcode2.exe` pakai Ghidra (MOV→LEA) |
| P07 | [[P07 - Stack Security and Exploitation]] | Checksec, NX, PIE, ASLR, RELRO (Partial/Full), Stack Canary, Fortify Source, pengantar buffer overflow |
| P08 | [[P08 - Deep Dive into Buffer Overflow]] | EBP/ESP/EIP, layout stack detail, dangerous C functions, overwrite Old EIP, pola injeksi shellcode |
| P09 | [[P09 - Exploiting with Pwntools]] | Pwntools I/O (recv/send), context, kelas ELF (search, address), packing (p64/u64) |
| P10 | [[P10 - Ret2win Exploitation]] | Overwrite saved EIP ke fungsi target, cari padding (trial-error & De Bruijn), radare2, endianness |
| P11 | [[P11 - Return Oriented Programming]] | ROP 32-bit (argumen via stack) vs 64-bit (argumen via register + gadget chaining) |
| P12 | [[P12 - Ret2libc Exploitation]] | Serangan tanpa shellcode lewat `system()`+`/bin/sh` di libc, `ldd`/`readelf`/`strings`, otomatisasi pwntools |
| P13 | [[P13 - Format String Exploitation]] | Bahaya `printf` tanpa argumen cukup, `%N$x`, arbitrary read via `%s`, null byte, tantangan arbitrary write |

## Rangkuman Ujian
- [[REBinex - Review dan Glosari]] — review lintas pertemuan + glosari lengkap

## Peta Materi

Matkul ini adalah salah satu yang paling **linear dan bertumpuk** dari semua matkul di vault — hampir tiap minggu SECARA EKSPLISIT membangun di atas minggu sebelumnya, dan urutan ini WAJIB dipelajari berurutan. Tiga blok besar:

**Blok 1 — Fondasi Reversing (P01–P06): "cara membaca dan mengubah binary yang sudah jadi"**
P01 kasih gambaran besar (apa itu RE, buat apa, legal atau nggak). P02–P03 masuk ke bahasa assembly x86 secara mendalam — ini FONDASI yang dipakai LITERAL di setiap deck berikutnya. P04 soal cara pembuat program MENYULITKAN reverser (obfuscation), P05 soal tool buat MENGAMATI program berjalan (dynamic analysis/GDB), P06 soal cara MENGUBAH binary yang sudah jadi (patching). Blok ini defensif/observasional — belum ada "serangan" beneran.

**Blok 2 — Dasar Eksploitasi (P07–P09): "kenapa dan bagaimana program bisa dieksploitasi"**
P07 kasih peta proteksi keamanan modern (checksec: NX/PIE/ASLR/RELRO/Canary) — kamu HARUS tau ini exist sebelum belajar cara "menembusnya". P08 bedah MEKANISME buffer overflow sampai level register. P09 kenalan sama pwntools, tool yang dipakai TERUS-MENERUS di empat deck berikutnya.

**Blok 3 — Teknik Eksploitasi Konkret (P10–P13): "empat cara berbeda mengambil alih eksekusi program"**
P10 (ret2win) adalah versi PALING SEDERHANA — overwrite return address ke fungsi yang SUDAH ADA. P11 (ROP) menaikkan levelnya — merangkai POTONGAN KECIL kode buat kasus yang lebih rumit. P12 (ret2libc) itu APLIKASI KHUSUS dari konsep P10/P11, ditarget ke fungsi `system()` di library. P13 (format string) itu KELUARGA BUG YANG BEDA SAMA SEKALI dari buffer overflow — bukan soal "kelebihan input", tapi soal `printf` yang dikasih format string dari sumber yang salah.

**Benang merah yang menembus semua blok:**

- **EIP/RIP sebagai target utama** — diperkenalkan di P02 ("kalau kamu kontrol EIP, kamu kontrol CPU"), jadi INTI dari P08 (overwrite Old EIP), P10 (ret2win), P11 (ROP), dan P12 (ret2libc). Satu kalimat di minggu ke-2 menjelaskan HAMPIR SELURUH sisa semester.
- **Stack frame (EBP-relative addressing)** — dijelaskan konsepnya di P02, dipraktikkan di P03 (C ke assembly), jadi DASAR PERHITUNGAN untuk P06 (offset patch `-0x28`), P08 (urutan parameter/Old EIP/Old EBP/variabel lokal), dan P10 (padding).
- **Endianness** — disinggung sekilas soal `mov` di P02, jadi eksplisit di P10 (little vs big-endian saat mengirim alamat), dan implisit di SEMUA payload P10-P13 (`p32`/`p64` di pwntools otomatis menangani ini).
- **ldd/readelf/strings sebagai trio "mencari alamat manual"** — diperkenalkan di P12, tapi konsepnya (mencari base address, offset fungsi, offset string) itu generalisasi dari "Finding the Address" di P10 lewat radare2.
- **Pwntools sebagai lapisan otomatisasi** — P09 kenalin konsepnya, lalu P10-P13 SEMUANYA pakai pwntools buat merakit payload final, sambil tetap mengajarkan cara manual di baliknya (biar kamu paham APA yang diotomatisasi pwntools, bukan cuma cara pakainya).

## Konsep Utama
Belum ada note atomik di `02-Konsep/`. Kandidat terkuat kalau nanti mau dibuat (konsep yang muncul di lebih dari satu pertemuan):

- **EIP/Instruction Pointer** — P02, P08, P10, P11, P12
- **Stack frame (EBP, Old EIP, Old EBP)** — P02, P03, P06, P08, P10
- **NX / ASLR / PIE / RELRO / Canary** — P07, disebut ulang di P09 (elf.address), P10 (endianness/base), P12 (disable ASLR)
- **Gadget dan ROP chaining** — P11, jadi dasar teknik di P12 (64-bit ret2libc)
- **Format specifier positional access (`%N$x`)** — P13, konsep tersendiri yang tidak muncul di deck lain

## Catatan Pemrosesan

**Kesalahan Learning Outcome yang KONSISTEN di 12 dari 13 deck.** Slide 2 ("Learning Outcomes") di **P02 sampai P13** SEMUANYA menulis persis: *"Explain File Carving"* dan *"Using file, binwalk, foremost command to Implement File Carving"* — sebuah LO yang sama sekali tidak berhubungan dengan file carving (itu topik forensik, bukan reverse engineering/binary exploitation). Ini jelas kesalahan copy-paste template yang tidak pernah diperbarui dosen dari slide ke slide. **Isi note-note di vault ini mengikuti KONTEN SLIDE yang sesungguhnya (assembly, debugging, exploitation, dst.), bukan LO yang salah ketik itu.** Cuma P01 yang punya LO yang benar-benar sesuai isinya ("Explain the flow of reverse engineering in binary programs").

**Ekstraksi gambar.** Deck-deck di matkul ini SANGAT bergantung pada screenshot kode, diagram stack, dan tangkapan layar tool (Ghidra, GDB, radare2, pwntools) — total **106 gambar** berhasil diekstrak dari 245 slide, jauh lebih banyak dibanding matkul-matkul sebelumnya. Ambang batas ukuran file gambar diturunkan khusus untuk matkul ini (dari 15KB ke 3KB) karena banyak screenshot kode berukuran kecil yang nyaris terlewat oleh filter default. Beberapa slide TETAP tidak berhasil diekstrak sebagai gambar meski sudah diturunkan ambang batasnya (terutama di P06 slide 15, P07 slide 17-19, dan P08 slide 15-16/18) — ditandai eksplisit di section Diagram & Visual masing-masing note dengan saran untuk membuka PPT aslinya.

**Slide kosong.** P05 slide 15 ("A / UPX Packing") **benar-benar kosong** — bukan gagal ekstrak, tapi memang tidak ada konten apa pun selain judul di file PPTX aslinya (sudah diverifikasi langsung dari struktur file). Kemungkinan section demo praktik yang didemokan langsung di kelas tanpa slide pendukung, atau lupa diisi.

**Duplikat.** Tidak ada file identik (dicek lewat md5) di antara 13 PPT yang diupload.

**Slide dengan speaker notes.** Berbeda dari Secure Programming (yang tidak ada speaker notes sama sekali), deck-deck di matkul ini punya cukup banyak speaker notes berisi LINK REFERENSI (terutama P04, P05, P06, P07, P09, P10 yang merujuk ke `ctf101.org`, `ir0nstone.gitbook.io`, `malwaretech.com`, `krekelbits.wordpress.com`, dan `tripwire.com`). Link-link ini sudah dicatat di badan tiap note yang relevan.

**SKS matkul belum dikonfirmasi** — ditulis `3` di frontmatter sebagai asumsi administratif (mengikuti pola Secure Programming), belum ada info resmi. **Cek dan koreksi manual kalau salah.** Nama dosen dan jadwal juga belum diketahui — tidak ada nama dosen tercantum di slide manapun di 13 deck ini (beda dari dua matkul sebelumnya yang selalu mencantumkan nama dosennya).
