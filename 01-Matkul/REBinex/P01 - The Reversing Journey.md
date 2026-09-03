---
matkul: Reverse Engineering dan Binary Exploitation
minggu: 1
sks: 3
sumber: REBinex - P1.pptx
tags: [kuliah/rebinex, minggu/p01]
status: draft
diproses: 2026-09-04
---

# P01 — The Reversing Journey

## Ringkasan
> - **Reverse engineering (RE)** = proses narik ilmu atau blueprint desain dari sesuatu yang udah jadi/dibuat orang — kamu punya HASIL jadinya, kamu coba nebak gimana cara bikinnya.
> - RE dipake di DUA sisi pagar yang berlawanan: **pembuat malware** pake buat nyari celah keamanan, **pembuat antivirus** pake buat ngebedah malware itu dan nyari obatnya. Dipake juga sama **cracker** buat ngalahin proteksi copy software, dan buat ngebongkar algoritma **kriptografi** serta **DRM**.
> - **Assembly language** itu bukan SATU bahasa — itu SEKELUARGA bahasa, beda-beda tergantung platform/CPU-nya. Dia cuma representasi teks yang gampang dibaca manusia dari **opcode** (kode operasi) yang beneran dijalanin CPU.
> - Compiler nerjemahin bahasa tingkat tinggi (C, C++) jadi **machine code** yang spesifik-platform, ATAU jadi **bytecode** yang platform-independent (kayak Java, butuh virtual machine buat jalan).
> - Tiga kategori tool reverser: **system monitoring tools** (ngintip komunikasi program dengan OS), **disassembler** (ngubah biner jadi teks assembly — IDA Pro, Ghidra, Radare2), **debugger** (jalanin program selangkah-selangkah, bisa liat/ubah state-nya — GDB, WinDbg).
> - **Legalitas RE itu abu-abu dan tergantung konteks.** Di Uni Eropa, dekompilasi diizinin secara hukum kalau tujuannya interoperabilitas — aturan ini bahkan ngalahin klausul lisensi software yang bilang sebaliknya.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Reverse Engineering | Proses narik ilmu/blueprint dari sesuatu yang udah jadi |
| Assembly language | Representasi teks yang gampang dibaca dari opcode CPU; beda-beda per platform |
| Opcode | Angka yang mewakili satu perintah assembly |
| Object code / Machine code | Urutan opcode dan angka lain yang beneran dibaca dan dijalanin CPU |
| Compiler | Program yang ngubah source code jadi machine code |
| Bytecode | Format platform-independent, dijalanin lewat virtual machine (bukan langsung CPU) |
| Disassembler | Tool yang ngubah biner executable jadi teks assembly |
| Debugger | Tool buat jalanin program selangkah-selangkah dan ngintip/ubah state-nya |

## Isi

### Apa itu Reverse Engineering?
**Proses narik ilmu atau blueprint desain dari sesuatu yang buatan manusia.** Fakta menarik: reversing juga banyak dipakai berkaitan dengan software jahat, **di DUA sisi pagar**. Dipakai sama developer malware, DAN sama developer antidotenya. Reversing juga populer di kalangan **cracker** yang makainya buat nganalisis dan akhirnya ngalahin berbagai skema proteksi copy.

> [!info] Analogi
> Bayangin kamu nemu **mesin kue** yang udah jadi, tapi resepnya ilang. Reverse engineering itu kayak kamu ngebongkar mesinnya, ngeliat tiap gir dan pipa di dalamnya, terus nyoba nebak: "oh, kalau adonannya lewat sini, ditekan segini, dipanggang sekian menit — pasti gitu urutannya." Kamu nggak punya resep aslinya, tapi kamu bisa REKONSTRUKSI logikanya dari hasil jadinya.

### Kegunaan RE (RE Use Cases)

**Malicious Software.** Reversing dipake ekstensif di DUA UJUNG rantai software jahat.
- **Developer malware** sering pake reversing buat nyari **kerentanan software** yang ngebolehin program jahat dapet akses ke informasi sensitif atau bahkan kendali penuh atas sistem.
- **Developer antivirus** membedah dan menganalisis TIAP program jahat buat: **melacak tiap langkah** yang diambil program itu, **menaksir kerusakan** yang bisa ditimbulkan, **perkiraan laju infeksi**, **gimana caranya dibersihin** dari sistem yang kena, dan **apa infeksinya bisa dihindarin sama sekali**.

**Cryptographic Algorithms.** Kriptografi berbasis pada kerahasiaan: Alice ngirim pesan ke Bob, dienkripsi pakai rahasia yang (semoga) cuma diketahui mereka berdua. Kriptografi ngandelin algoritma ciphering dan key-generation-nya biar tetep aman. **Kalau sebuah program ngelakuin operasi kriptografi buat ngelindungin datanya, ngebalik (reverse) operasi itu bisa ngasih petunjuk gimana caranya dekripsi data itu balik ke bentuk aslinya.**

**Digital Rights Management (DRM).** Komputer modern udah ngubah sebagian besar materi berhak cipta jadi informasi digital. Penyedia konten media ngembangin atau ngakuisisi teknologi bernama DRM buat ngelindungin materinya. **Buat ngalahin teknologi DRM, kamu harus paham dulu cara kerjanya.** Dengan teknik reversing, cracker bisa belajar rahasia internal teknologinya dan nemuin modifikasi paling sederhana yang mungkin dibuat ke program biar proteksinya nonaktif.

### Assembly Language
CPU baca **machine code**, yang nggak lain cuma urutan bit berisi daftar instruksi buat CPU jalanin. **Assembly language cuma representasi teks dari bit-bit itu** — kita namain elemen di urutan kode ini biar bisa dibaca manusia. Jadi, assembly language itu SEKELUARGA bahasa, bukan SATU bahasa. **Beneran tergantung platform apa program itu di-compile.**

Tiap perintah assembly diwakili oleh angka, disebut **operation code, atau opcode**. **Object code**: urutan opcode dan angka lain yang dipake bareng opcode itu buat ngejalanin operasi. CPU terus-terusan baca object code dari memori, decode dia, dan bertindak berdasarkan instruksi yang tertanam di situ. Developer nulis kode di assembly language = developer pake program assembler buat nerjemahin kode assembly language teks jadi kode biner, yang bisa di-decode CPU.

### Gimana Program di-Compile
CPU cuma bisa jalanin machine code, jadi gimana caranya bahasa pemrograman diterjemahin jadi machine code? Jawabannya: pake **compiler**, program yang ngambil file source (kode kita) dan ngehasilin file machine code yang berkorespondensi. Machine code-nya bisa:
- **Object code spesifik-platform** yang di-decode LANGSUNG sama CPU (kayak Windows dan Linux baca object code yang beda)
- Format khusus **platform-independent bernama bytecode** (kayak Java yang bisa jalan di banyak platform)

**Platform-specific (C dan C++).** C dan C++ ngehasilin object code yang bisa dibaca mesin dari source code-nya. Object code ini, kalau di-disassemble, muncul sebagai program assembly language buatan mesin. Developer software ngegariskan tugas-tugas di bahasa tingkat tinggi; compiler yang ngurus detail proses eksekusinya dan masukin itu ke dalam object code hasilnya. **Hambatan terbesar dalam mecahin kode buatan compiler adalah optimisasi** yang diterapin sebagian besar compiler modern.

Compiler pake berbagai teknik buat minimalin ukuran kode dan ningkatin performa eksekusi. **Kode hasil optimisasi sering counter-intuitive dan susah dibaca.** Misalnya, compiler yang mengoptimalkan sering ngeganti instruksi yang lugas dengan operasi yang secara matematis setara tapi tujuannya jauh dari jelas sekilas pandang.

**Platform-independent (Java, dll).** Compiler buat bahasa tingkat tinggi kayak Java ngehasilin **bytecode** bukan object code. Bytecode mirip object code, cuma bedanya biasanya di-decode oleh sebuah PROGRAM, bukan CPU. Caranya: kita bikin compiler yang generate bytecode, terus pake program bernama **virtual machine** buat decode bytecode itu dan ngejalanin operasi yang dideskripsiin di situ. Virtual machine-nya sendiri yang ngubah bytecode jadi object code standar yang kompatibel sama CPU di bawahnya.

### Tools

**System Monitoring Tools.** Reversing level-sistem butuh berbagai tool buat sniffing, monitoring, menjelajahi, dan mengekspos program yang lagi di-reverse. Tool-tool ini ngumpulin info tentang aplikasi dan lingkungannya dari sistem operasi. **OS dimanfaatin buat narik info karena sebagian besar komunikasi antara program dan dunia luar lewat OS.** System-monitoring tools bisa nge-track aktivitas jaringan, akses file, akses registry, dan lainnya. Contoh: **Windows ProcMon, RegMon**.

**Disassemblers.** Program yang ngubah biner executable-nya sebuah program jadi file teks berisi kode assembly language. **Disassembly itu spesifik-prosesor**, tapi sebagian disassembler dukung banyak arsitektur CPU. **Disassembler berkualitas tinggi itu tool esensial buat reverser.** Sebagian reverser milih pake disassembler bawaan di dalam debugger low-level tertentu. Contoh: **IDA Pro, Ghidra, Radare2, Binary Ninja**.

**Debuggers.** Ngebolehin user nge-trace program, ngejalanin satu baris pada satu waktu, dan ngamatin atau ngubah state program itu. Debugger bantu developer identifikasi dan atasi masalah dengan masang **breakpoint** dan meriksa alur eksekusi program dari dekat. Debugger nampilin program dalam bentuk source-code, meskipun mereka kerjanya sama machine code — ngebolehin developer kerja pakai representasi kode yang familiar. **Reverser sangat ngandelin debugger dalam mode disassembly**, pake disassembler bawaan buat step-through dan analisis kode yang udah di-disassemble. Contoh: **GDB, WinDbg, lldb, Xcode Debugger**.

> [!info] Analogi
> **Disassembler itu kamus terjemahan sekali jalan** — dia ngambil buku berbahasa asing (biner) dan langsung ngasih kamu terjemahan lengkapnya (kode assembly), tapi kamu bacanya sendiri dari awal sampe akhir. **Debugger itu guru les privat yang nemenin kamu baca kalimat per kalimat**, berhenti tiap kali kamu minta ("breakpoint"), dan ngebolehin kamu nyoret-nyoret catatan di pinggir buku (ubah state) buat liat apa yang kejadian kalau kata tertentu diganti.

### Elephant in the Room: apa Reverse Engineering itu legal?
Pertanyaan hukum utama seputar klausul reverse-engineering di perjanjian lisensi adalah **apakah mereka bisa dipaksakan secara hukum (enforceable)**. Di AS, kelihatannya nggak ada satu jawaban otoritatif tunggal buat pertanyaan ini — semuanya tergantung situasi spesifik gimana reverse engineering itu dilakuin.

**Di Uni Eropa**, isu ini udah didefinisikan dengan jelas lewat **Directive on the Legal Protection of Computer Programs**. Direktif ini nentuin bahwa **dekompilasi program software itu diperbolehin dalam kasus interoperabilitas**. **Direktif ini ngalahin perjanjian lisensi shrink-wrap apa pun**, setidaknya soal hal ini.

## Diagram & Visual
- **Slide 11 — ilustrasi ringan yang menyertai candaan "who here wants to write in assembly? 😂"**
  ![[99-Assets/REBinex/P01-slide11.png]]

> [!warning] Cuma satu gambar di deck ini, dan sifatnya dekoratif/lucu-lucuan, bukan diagram teknis — nggak ada materi inti yang bergantung padanya.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Cracker** | Orang yang ngalahin skema proteksi software (misal copy protection) pakai teknik reversing |
| **DRM (Digital Rights Management)** | Teknologi buat ngelindungin konten berhak cipta secara digital |
| **x86, x64, ARM** | Contoh keluarga arsitektur CPU, masing-masing punya "dialek" assembly-nya sendiri |
| **IDA Pro / Ghidra / Radare2 / Binary Ninja** | Contoh tool disassembler yang populer buat reverse engineering |
| **GDB / WinDbg / lldb** | Contoh tool debugger yang populer |
| **Shrink-wrap license** | Perjanjian lisensi software yang dianggap disetujui user begitu segel kemasan software dibuka |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan peran reversing di sisi developer malware vs. sisi developer antivirus. Analisis: kenapa DUA-duanya bisa pakai teknik yang PERSIS SAMA (reversing) tapi menghasilkan tujuan yang berlawanan? Apa yang membedakan keduanya kalau bukan teknik yang dipakai?
2. **(C4 – Analisis)** Bandingkan disassembler dan debugger dari sisi "apa yang bisa kamu LIAT" vs "apa yang bisa kamu LAKUIN" ke program yang di-reverse. Kenapa reverser sering butuh DUA-duanya, bukan cuma salah satu?
3. **(C5 – Evaluasi)** Sebuah perusahaan software nulis di lisensinya: "Dilarang keras melakukan reverse engineering terhadap software ini dengan alasan apa pun." Evaluasi klausul ini pakai konteks Directive Uni Eropa yang dibahas di slide 18 — dalam kondisi apa klausul ini BISA jadi nggak berlaku di negara-negara Uni Eropa?
4. **(C5 – Evaluasi)** Compiler modern ngelakuin optimisasi yang "counter-intuitive dan susah dibaca" (slide 11). Evaluasi: apa optimisasi compiler ini sengaja dirancang buat mempersulit reverse engineering, atau itu cuma efek SAMPINGAN dari tujuan lain? Jelasin alasanmu.
5. **(C6 – Cipta)** Rancang skenario penggunaan reverse engineering yang LEGAL dan ETIS (bukan buat crack software bajakan) yang belum disebut di slide ini — jelasin: (a) apa yang mau kamu pahami dari program itu, (b) tool kategori mana (system monitoring/disassembler/debugger) yang paling cocok kamu pakai, dan (c) kenapa itu termasuk penggunaan yang sah secara hukum.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_REBinex]]
- [[P02 - Assembly and Disassembly]]
- [[REBinex - Review dan Glosari]]
