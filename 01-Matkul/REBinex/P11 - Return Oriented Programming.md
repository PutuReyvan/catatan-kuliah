---
matkul: Reverse Engineering dan Binary Exploitation
minggu: 11
sks: 3
sumber: REBinex - P11.pptx
tags: [kuliah/rebinex, minggu/p11]
status: draft
diproses: 2026-09-04
---

# P11 — Return Oriented Programming

> [!warning] Learning outcome di slide 2 nggak nyambung sama isi (masih tentang ROP). Lihat catatan di [[P02 - Assembly and Disassembly]].

## Ringkasan
> - **ROP (Return Oriented Programming)** = ide menyambungkan potongan-potongan kecil assembly (**gadgets**) yang SUDAH ADA di binary/library, pakai kontrol atas stack, buat bikin program ngelakuin hal yang lebih kompleks. Kebanyakan program NGGAK PUNYA fungsi `give_shell` yang nyaman kayak di [[P10 - Ret2win Exploitation]] — jadi kita harus manggil `system` atau fungsi exec lainnya SECARA MANUAL.
> - **32-bit**: argumen fungsi lewat STACK. Kalau kamu punya kontrol stack, kamu punya kontrol ARGUMEN. Trik: susun stack persis seperti kalau `system()` dipanggil normal — alamat PLT `system` → fake return address → alamat argumen (misal string global).
> - **64-bit**: argumen lewat REGISTER (calling convention), BUKAN stack. Buat manggil `system("/bin/sh")`, kamu harus bisa NGONTROL register **RDI** (argumen pertama).
> - Karena kamu nggak bisa langsung "mov ke RDI" dari stack, kamu pakai **gadget** — potongan kode kecil di binary kayak `pop rdi; ret` — yang ngambil nilai dari puncak stack, taruh ke RDI, terus `ret` (yang otomatis lompat ke gadget/fungsi BERIKUTNYA di stack-mu). Ini bikin kamu bisa **RANTAI** beberapa gadget buat ngontrol banyak register berurutan.
> - Tool buat cari gadget: **rp++** atau **ROPgadget**.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| ROP (Return Oriented Programming) | Merangkai potongan kecil assembly (gadget) yang sudah ada di binary, lewat kontrol stack |
| Gadget | Potongan kecil instruksi assembly yang diakhiri `ret`, dipakai sebagai "blok bangunan" ROP |
| Calling convention | Aturan bagaimana argumen fungsi diteruskan (via stack di 32-bit, via register di 64-bit) |
| RDI, RSI | Register 64-bit yang menyimpan argumen pertama dan kedua fungsi (calling convention Linux x86-64) |
| PLT (Procedure Linkage Table) | Struktur ELF yang menghubungkan pemanggilan fungsi ke lokasi aslinya di library |
| Fake call stack | Susunan data di stack yang dibuat penyerang meniru seolah-olah fungsi normal dipanggil |

## Isi

### Konsep Dasar
**Return Oriented Programming (atau ROP)** adalah ide merangkai potongan-potongan kecil assembly bersama-sama dengan kontrol stack untuk membuat program melakukan hal-hal yang lebih kompleks. Seperti yang kita lihat di buffer overflow, memiliki kontrol stack bisa sangat berpengaruh karena itu membolehkan kita menimpa saved instruction pointer, memberi kita kontrol atas apa yang program lakukan berikutnya. **Kebanyakan program TIDAK punya fungsi `give_shell` yang nyaman sama sekali. Jadi kita perlu mencari cara untuk secara manual memanggil `system` atau fungsi exec lainnya untuk mendapatkan shell kita.**

Ada perbedaan cara kita mendekati ROP di binary 32-bit dan 64-bit.

### ROP 32-bit
Sebuah kode sumber punya buffer overflow di variabel `echo`, yang bisa memberi kita kontrol EIP saat `main` return. Tapi kita tidak punya fungsi `give_shell`. Jadi, apa yang bisa kita lakukan?

**Jawabannya: kita bisa memanggil `system` dengan argumen yang kita kontrol.** Karena argumen diteruskan lewat STACK di program Linux 32-bit, kalau kita punya kontrol stack, kita punya kontrol ARGUMEN. Ketika `main` return, kita ingin stack kita terlihat seperti seolah-olah sesuatu sudah memanggil fungsi `system` secara normal.

**Apa yang ada di dalam stack setelah sebuah fungsi dipanggil?** Kita tahu bahwa `main()` sendiri adalah sebuah fungsi, dia punya stack frame-nya sendiri. Sekarang, apa yang perlu ada di dalam stack supaya setelah kembali dari `main()`, sebuah fungsi lain bernama `system()` bisa dipanggil?

**Jawabannya**: stack frame `main()` perlu terlihat seperti ini: **alamat entri PLT dari `system`, diikuti fake return address (untuk dituju `system` setelah dia selesai), diikuti alamat argumen yang mau diteruskan ke `system`**. Lalu ketika `main()` return, dia akan lompat ke entri PLT `system()` dan stack-nya akan terlihat persis seperti seolah-olah `system` sudah dipanggil normal untuk pertama kalinya. **Kita tidak peduli ke mana `system` akan return, karena kita akan sudah mendapatkan shell kita saat itu.**

**Meletakkan Argumen.** Ini permulaan yang bagus, tapi kita perlu meneruskan argumen ke `system` agar sesuatu terjadi. Mengetahui tentang ASLR, stack dan dynamic library "berpindah-pindah" tiap kali program dijalankan, artinya kita tidak bisa dengan mudah memakai data di stack atau string di libc untuk argumen kita. Dalam kasus ini, kita punya global `name` yang sangat berguna yang akan berada di lokasi yang DIKENAL di dalam binary (di segmen BSS). Untuk menyederhanakan, kita perlu mencari alamat yang berisi nilai-nilai yang ingin kita gunakan sebagai argumen ke fungsi `system()` kita; seperti `/bin/sh`, dll.

**Ringkasan eksploitnya, langkah demi langkah:**
1. Masukkan "sh" atau perintah lain untuk dijalankan sebagai `name`
2. Isi stack dengan:
   - Garbage sampai saved EIP
   - Alamat entri PLT `system`
   - Sebuah fake return address untuk dituju `system` setelah selesai
   - Alamat global `name` untuk bertindak sebagai argumen PERTAMA ke `system`

> [!info] Analogi
> ROP 32-bit itu kayak **kamu nyuruh orang lain nge-print dokumen, padahal kamu nggak boleh masuk ke ruangan printer-nya**. Yang kamu BISA lakukan cuma nulis catatan di secarik kertas dan nyelipin lewat celah pintu ("kontrol stack"). Kalau catatanmu ditulis persis kayak formulir permintaan cetak resmi (formatnya sama kayak `system()` dipanggil normal) — nama fungsi yang mau dipanggil, alamat kembali, dan argumen ("cetak dokumen INI") — si penerima (CPU) bakal ngejalanin permintaanmu PERSIS kayak itu permintaan yang sah, walau kamu nggak pernah masuk ruangannya.

### ROP 64-bit
Di binary 64-bit kita harus bekerja sedikit lebih keras untuk meneruskan argumen ke fungsi. Ide dasar menimpa saved RIP tetap sama, tapi seperti dibahas di calling convention, **argumen diteruskan LEWAT REGISTER di program 64-bit.** Dalam kasus menjalankan `system`, ini artinya kita perlu mencari cara untuk mengontrol register **RDI**.

**Register 64-bit yang relevan:**
- `rax` — register a extended
- `rbx` — register b extended
- `rcx` — register c extended
- `rdx` — register d extended
- `rbp` — register base pointer (awal stack)
- `rsp` — register stack pointer (posisi sekarang di stack, tumbuh ke bawah)
- `rsi` — register source index (sumber untuk penyalinan data)
- `rdi` — register destination index (tujuan untuk penyalinan data) — **argumen pertama fungsi**

**Untuk melakukan ini, kita akan memakai potongan kecil assembly di dalam binary, disebut "gadget".** Gadget-gadget ini biasanya melakukan pop satu atau lebih register dari stack, lalu memanggil `ret`, yang membolehkan kita merangkai mereka bersama dengan membuat sebuah fake call stack yang besar. Misalnya, kalau kita perlu kontrol RDI dan RSI, kita bisa mencari dua gadget di programnya yang terlihat seperti `pop rdi; ret` dan `pop rsi; ret`.

Kita bisa memakai tool seperti **rp++** atau **ROPgadget**. Kalau kita bisa menemukan gadget yang tepat, kita bisa menyusun fake call stack dengan gadget-gadget ini untuk mengeksekusinya secara berurutan, mem-pop nilai yang kita kontrol ke dalam register, dan lalu berakhir dengan sebuah lompatan ke `system()`.

> [!info] Analogi
> **Gadget itu kayak domino** — tiap gadget itu satu potong kecil "pop nilai dari stack ke register X, lalu ret" yang, begitu dijalanin, OTOMATIS "jatuh" ke domino berikutnya yang alamatnya kamu taruh persis setelahnya di stack (karena instruksi `ret` mengambil alamat berikutnya dari stack dan lompat ke sana). Kamu susun urutan domino ini (gadget demi gadget) sedemikian rupa sehingga tiap "jatuh" ngisi satu register yang kamu butuhin, sampai akhirnya domino terakhir manggil `system()` dengan semua register udah keisi persis yang kamu mau.

### Contoh ROP 64-bit langkah demi langkah
Bayangkan sebuah stack frame yang disusun sedemikian rupa untuk mengontrol RDI dan RSI. Susunannya (dari yang paling dekat ke posisi kembali `main`):
- **Alamat gadget `pop rdi; ret`**
- **Nilai yang mau di-pop ke RDI**
- **Alamat yang menjadi tujuan `ret` gadget RDI (yaitu gadget `pop rsi` berikutnya)**
- **Nilai yang mau di-pop ke RSI**
- **Nilai untuk R15 (kemungkinan garbage, karena beberapa gadget mem-pop dua register sekaligus)**
- **Alamat tujuan `ret` gadget RSI — di sinilah eksekusi lompat setelah RDI dan RSI terkontrol**

Melangkah satu instruksi pada satu waktu: `main` return, lompat ke gadget `pop rdi`. Instruksi `pop rdi` dijalankan, mem-pop puncak stack ke RDI. Gadget RDI kemudian `ret` ke gadget `pop rsi`. RSI dan R15 di-pop. Dan akhirnya gadget RSI `ret`, lompat ke fungsi apa pun yang kita mau, tapi sekarang dengan RDI dan RSI di-set ke nilai yang kita kontrol.

## Diagram & Visual
- **Slide 4 — kode sumber contoh dengan buffer overflow pada variabel `echo`**
  ![[99-Assets/REBinex/P11-slide04.png]]
- **Slide 6, 7 — visualisasi stack setelah pemanggilan fungsi normal (untuk ROP 32-bit)**
  ![[99-Assets/REBinex/P11-slide06.png]]
  ![[99-Assets/REBinex/P11-slide07.png]]
- **Slide 12, 13 — visualisasi gadget `pop rdi`/`pop rsi` (ROP 64-bit)**
  ![[99-Assets/REBinex/P11-slide12.png]]
  ![[99-Assets/REBinex/P11-slide13.png]]
- **Slide 14, 15 — susunan fake call stack ROP 64-bit dan langkah `main` return ke gadget pertama**
  ![[99-Assets/REBinex/P11-slide14.png]]
  ![[99-Assets/REBinex/P11-slide15.png]]
- **Slide 17, 18, 19 — langkah-langkah eksekusi gadget berurutan (pop rdi → ret ke gadget rsi → pop rsi/r15 → ret ke fungsi tujuan)**
  ![[99-Assets/REBinex/P11-slide17.png]]
  ![[99-Assets/REBinex/P11-slide18.png]]
  ![[99-Assets/REBinex/P11-slide19.png]]

> [!warning] Deck ini SANGAT bergantung pada diagram visual untuk memahami rangkaian gadget ROP 64-bit langkah demi langkah — teks di slide 14-19 (nomor baris "Line 5/6/7...") merujuk ke diagram yang cuma bisa dipahami penuh dari gambarnya. **Sangat disarankan buka gambar-gambar di atas secara berurutan** sambil membaca penjelasan teks, karena urutan visual ini sulit direkonstruksi dari teks saja.

## Rumus / Sintaks
```
; ROP 32-bit -- susunan stack setelah main() return:
[alamat PLT system()]
[fake return address]     <- ke mana system() akan "return" (tidak penting)
[alamat argumen, misal &name]   <- argumen pertama system() lewat STACK

; ROP 64-bit -- fake call stack untuk kontrol RDI dan RSI:
[alamat gadget: pop rdi; ret]
[nilai untuk RDI]
[alamat gadget: pop rsi; ret]   <- tujuan ret dari gadget RDI
[nilai untuk RSI]
[nilai untuk R15 -- garbage]     <- gadget rsi kadang pop 2 register sekaligus
[alamat fungsi tujuan]           <- tujuan ret dari gadget RSI, RDI & RSI sudah terkontrol
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **BSS segment** | Bagian memori program untuk variabel global yang belum diinisialisasi; punya lokasi yang dikenal/tetap |
| **rp++ / ROPgadget** | Tool untuk mencari gadget (potongan instruksi diakhiri `ret`) di dalam sebuah binary |
| **Calling convention** | Aturan standar tentang bagaimana argumen diteruskan ke fungsi dan siapa yang membersihkan stack setelahnya |
| **PLT entry** | Alamat "pintu masuk" sebuah fungsi library di Procedure Linkage Table |
| **Fake call stack** | Susunan data buatan penyerang di stack yang meniru pola pemanggilan fungsi normal |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan cara meneruskan argumen di ROP 32-bit (lewat stack) dan ROP 64-bit (lewat register via gadget). Analisis: kenapa ROP 64-bit membutuhkan LANGKAH TAMBAHAN (mencari dan merangkai gadget) yang tidak diperlukan di 32-bit?
2. **(C4 – Analisis)** Slide 8 menyebut alasan kita tidak bisa "dengan mudah memakai data di stack atau string di libc" untuk argumen — karena ASLR membuat lokasinya "berpindah-pindah". Analisis: kenapa variabel global `name` di segmen BSS TIDAK terpengaruh masalah yang sama (kenapa lokasinya "dikenal" meski ASLR aktif)? Hubungkan dengan konsep PIE dari [[P07 - Stack Security and Exploitation]].
3. **(C5 – Evaluasi)** Seorang mahasiswa berhasil menemukan gadget `pop rdi; ret` tapi TIDAK BISA menemukan gadget `pop rsi; ret` di binary targetnya. Evaluasi: kalau fungsi yang ingin dipanggil (misalnya `system()`) cuma butuh SATU argumen (lewat RDI saja), apakah ketiadaan gadget RSI ini benar-benar menghalangi eksploitasinya? Jelaskan.
4. **(C5 – Evaluasi)** Bandingkan kompleksitas ROP dengan ret2win ([[P10 - Ret2win Exploitation]]). Evaluasi: dalam situasi CTF nyata, kapan seorang exploit developer akan MEMILIH ROP daripada ret2win biasa, meskipun ROP jelas lebih rumit untuk disusun?
5. **(C6 – Cipta)** Rancang (secara konseptual, diagram teks) fake call stack untuk memanggil fungsi `write(1, "/bin/sh", 7)` di sistem 64-bit — fungsi ini butuh TIGA argumen (`RDI`=1, `RSI`="/bin/sh" alamat, `RDX`=7). Dengan asumsi kamu punya gadget `pop rdi; ret`, `pop rsi; pop r15; ret` (2 pop sekaligus, seperti contoh di slide), dan `pop rdx; ret` yang terpisah — susun urutan gadget dan nilai yang perlu kamu taruh di stack, dari posisi PALING DEKAT ke saved return address, sampai akhirnya melompat ke alamat fungsi `write`.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_REBinex]]
- [[P10 - Ret2win Exploitation]]
- [[P12 - Ret2libc Exploitation]]
- [[REBinex - Review dan Glosari]]
