---
matkul: Forensics
minggu: 2
sks: 2
sumber: 2 Key Technical Concepts.pptx
tags: [kuliah/forensics, minggu/w02]
status: draft
diproses: 2026-09-03
---

# W02 — Key Technical Concepts

## Ringkasan
> - Premisnya: **tanpa paham cara kerja komputer di dalam, kamu gak bisa melakukan pemeriksaan forensik.** Dan tidak semua proses serta hardware punya nilai forensik yang sama.
> - **Bit → byte → karakter.** Satu huruf/angka/spasi = satu byte. ASCII 128 karakter (94 printable), Unicode untuk semua bahasa dunia. Hexadecimal (base 16, prefix `0x`) itu cara ringkas menulis binary.
> - **File extension itu cara paling tidak andal** untuk mengenali file — gampang banget diganti. Yang andal: **file signature / magic number**.
> - Tiga cara data ditulis: **elektromagnetik** (HDD), **transistor mikroskopis** (flash), **pantulan cahaya** (optical).
> - **Volatilitas** itu pembeda paling penting secara forensik: **RAM volatile** (hilang begitu listrik putus), **hard drive non-volatile**.
> - Empat jenis computing environment: **stand-alone, networked, mainframe, cloud** — dan pilihan ini menentukan cara pengumpulan, tempat mencari data, tool, dan tingkat kerumitannya.
> - **Sector = 512 byte**, wadah terkecil yang dipakai komputer untuk menyimpan informasi.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Bit | Satu digit biner, 1 atau 0; binary = sistem bilangan **base 2** |
| Byte | Satuan yang mewakili satu huruf, angka, spasi, atau karakter khusus |
| Hexadecimal | Sistem **base 16**, ditulis dengan 0–9 dan A–F, sering diawali prefix `0x` |
| ASCII | American Standard Code for Information Interchange; **128 karakter**, 94 di antaranya printable |
| Unicode | Skema encoding untuk **semua bahasa dunia**, terdiri dari ribuan karakter |
| File extension | Cara paling umum mengenali file, tapi **paling tidak andal** karena mudah diubah |
| Magic number | Tanda pengenal di dalam file itu sendiri (file signature) |
| Magnetic disk | Partikel dimagnetisasi = 1, tidak dimagnetisasi = 0 |
| Flash memory | Tersusun dari transistor; bermuatan = 1, tidak = 0. **Menyimpan data tanpa listrik** |
| Optical storage | Membaca/menulis dengan laser, memanfaatkan beda pantulan **bump** dan **land** |
| Volatile memory | Data hanya ada selama ada aliran listrik — contohnya RAM |
| Non-volatile | Data tetap ada setelah komputer dimatikan — contohnya hard drive |
| SRAM / DRAM | Dua jenis RAM yang perlu diketahui |
| Unallocated space | Ruang kosong: belum terpakai, atau file penghuninya sudah dihapus |
| FAT | File Allocation Table, filesystem umum tertua |
| NTFS | New Technology File System, dipakai Windows modern |
| Sector | Wadah terkecil penyimpanan komputer, menampung sampai **512 byte** |
| IaaS / PaaS / SaaS | Tiga model cloud computing |

## Isi

### Kenapa harus paham teknis
Slide membuka dengan pembenaran yang tegas:

> **Pengetahuan mendalam tentang cara kerja internal komputer itu kritis bagi praktisi digital forensic.** Pengetahuan inilah yang memungkinkan kita melakukan pemeriksaan menyeluruh atas barang bukti dan memberikan pendapat yang akurat. Sederhananya, **kita tidak bisa melakukan pekerjaan kita tanpa itu**.

Dan satu kalimat yang gampang terlewat tapi penting: **tidak semua proses dan hardware punya nilai forensik yang sama.** Memory dan storage memainkan peran besar di hampir semua pemeriksaan.

### Bit, byte, dan sistem bilangan
Bagi komputer, semuanya hitam putih — semuanya soal **1 dan 0**. Komputer memakai bahasa bernama **binary**, yang hanya punya dua kemungkinan hasil: 1 atau 0. Tiap 1 atau 0 disebut **bit**. Secara matematis, binary diklasifikasikan sebagai sistem bilangan **base 2**.

Bagaimana byte berhubungan dengan huruf dan angka? **Tiap huruf, angka, spasi, dan karakter khusus diwakili oleh satu byte.** Contoh dengan character set ASCII:

| Karakter | Binary | Hexadecimal |
| --- | --- | --- |
| `A` (huruf besar) | `01000001` | — |
| `a` (huruf kecil) | `01100001` | `61` |
| `M` (huruf besar) | — | `4D` |

**Hexadecimal** (atau **hex**) adalah sistem **base 16** yang jadi cara praktis untuk mengekspresikan angka biner. Hex ditulis memakai angka **0–9** dan huruf **A–F**. Sering kali kamu akan melihat angka heksadesimal ditulis dengan prefix **`0x`**.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa hex, bukan desimal? Karena **1 digit hex = persis 4 bit**, jadi 1 byte = persis 2 digit hex. Konversinya rapi dan tanpa sisa, sementara desimal tidak. Itu sebabnya semua hex editor forensik menampilkan data dalam hex — kamu bisa membaca byte satu per satu tanpa berhitung. Ini kepakai langsung waktu membaca **magic number** dan waktu **carving** di [[W09 - File Recovery and Data Carving]].

### ASCII dan Unicode
Bagaimana 1 dan 0 berubah jadi huruf A dan B? Komputer memakai **encoding scheme** untuk mengubah biner jadi sesuatu yang bisa dibaca manusia. Ada dua yang perlu diperhatikan:

| | **ASCII** | **Unicode** |
| --- | --- | --- |
| Kepanjangan | American Standard Code for Information Interchange | — |
| Untuk | Bahasa Inggris | **Semua bahasa di dunia** |
| Jumlah karakter | **128 karakter**, hanya **94 yang printable** — sisanya control character untuk spacing dan processing | **Ribuan karakter** |

### File extension vs file signature
Secara fundamental, **file adalah string atau deretan bit dan byte**. Mengidentifikasi file bisa dilakukan dengan beberapa cara berbeda.

**File extension adalah cara yang paling umum.** Sebagai user, kita biasanya mengenali jenis file dari extension-nya — kalau sistemnya dikonfigurasi begitu. Sistem operasi bisa diatur supaya **file extension disembunyikan**.

Tapi untuk keperluan forensik, slide tegas:

> **File extension bukan cara yang paling andal untuk mengidentifikasi file.** File extension sangat mudah diubah, cuma butuh satu klik mouse dan beberapa ketukan keyboard. Di Windows, tinggal klik kanan nama file, rename, dan ganti extension-nya.

Jawabannya adalah **magic number** (file signature) — tanda pengenal yang ada **di dalam isi file itu sendiri**, bukan di namanya.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi: **file extension itu label di punggung map, magic number itu isi mapnya.** Siapa pun bisa nulis ulang label di punggung map dalam dua detik; yang jauh lebih repot adalah memalsukan isinya. Makanya penjahat sering menyamarkan file dengan mengganti extension (`rahasia.jpg` yang sebenarnya file ZIP), dan tool forensik selalu mengecek **byte pertama file**, bukan namanya. Konsep ini balik lagi jadi dasar file carving di [[W09 - File Recovery and Data Carving]].

### Tiga cara data ditulis
Data hari ini umumnya dibuat dengan tiga cara berbeda:

| Cara | Media | Cara membaca 1 dan 0 |
| --- | --- | --- |
| **Elektromagnetisme** | Magnetic disk (mayoritas drive saat ini) | Partikel **dimagnetisasi = 1**, tidak dimagnetisasi = 0 |
| **Transistor elektrik mikroskopis (flash)** | Thumb drive, memory card | Transistor **bermuatan = 1**, tanpa muatan = 0. **Tetap menyimpan data walau tanpa listrik** |
| **Pemantulan cahaya** | CD, DVD, Blu-ray | Laser memantul berbeda dari **bump** dan **land** (ruang di antaranya); perubahan pantulan inilah yang dibaca sebagai biner |

Lokasi penyimpanan di dalam komputer melayani tujuan berbeda: sebagian untuk **jangka pendek**, menampung sementara data yang sedang dipakai komputer saat itu; sisanya untuk penyimpanan **jangka panjang yang lebih permanen**.

### Data volatility
**Memory** dan **storage** adalah dua istilah yang agak sinonim untuk komputer — keduanya merujuk pada tempat internal penyimpanan data. Memory dipakai untuk penyimpanan jangka pendek, storage lebih permanen.

Apa pun sebutannya, ada **perbedaan signifikan** di antara keduanya, terutama **dari perspektif forensik**. Perbedaan itu terletak pada **volatilitas** datanya:

| | **RAM** | **Hard drive / flash drive** |
| --- | --- | --- |
| Sifat | **Volatile** | **Non-volatile** |
| Data bertahan | Hanya selama **listrik disuplai** | **Tetap ada** setelah komputer dimatikan |

Ada dua jenis RAM yang perlu diketahui: **Static RAM (SRAM)** dan **Dynamic RAM (DRAM)**.

> [!info] Konteks tambahan (bukan dari slide)
> Ini konsep tunggal paling penting di seluruh deck ini untuk praktik forensik. Konsekuensinya langsung: **kalau kamu menemukan komputer menyala di TKP, mematikannya berarti menghancurkan barang bukti** — password yang belum tersimpan, kunci enkripsi, proses malware yang cuma hidup di memori, koneksi jaringan aktif, semuanya lenyap. Ini nyambung ke ancaman terakhir di [[W01 - Digital Forensic Fundamental]] dan jadi dasar prinsip "**ambil yang paling volatile duluan**" di [[W06 - Data Acquisition]].

### Jenis-jenis RAM
Slide 16 mendaftar jenis RAM berdasarkan umurnya:

| Jenis | Keterangan |
| --- | --- |
| **EDO RAM** (Extended Data Output) | Salah satu jenis DRAM paling awal |
| **SDRAM** (Synchronous Dynamic RAM) | Mulai menyinkronkan diri dengan clock speed CPU. Maks 166 MT/s, transfer 1,3 GB/s. Dilabeli PC100, PC133, PC166 |
| **DDR-SDRAM / DDR1** | Efektif **menggandakan** transfer rate SDRAM. Maks 400 MT/s, 3,2 GB/s |
| **DDR2** | Maks 800 MT/s, 6,4 GB/s |
| **DDR3** | Konsumsi daya sampai **sepertiga lebih rendah** dari DDR2. Maks 1.600 MT/s, 14,9 GB/s |
| **DDR4** | Maks 3.200 MT/s, 21 GB/s |
| **GDDR SDRAM** | Dipakai di kartu grafis untuk rendering video |

*(MT/s = millions of transfers per second.)*

### Computing environment
Tidak semua "lingkungan" komputasi diciptakan sama, dan perbedaannya substansial. Kita bisa menemui komputer individual, jaringan berbagai ukuran, atau sistem yang lebih kompleks.

Perbedaan ini punya **dampak signifikan** pada:
- Proses pengumpulanmu
- Di mana kamu mencari data
- Tool yang akan kamu pakai
- Tingkat kompleksitas yang dibutuhkan

Karena itu, **klarifikasi lingkungan yang akurat berguna dimiliki sejak awal investigasi — bahkan sebelum kamu tiba di TKP.**

Empat kategorinya: **stand-alone, networked, mainframe, dan cloud.**

**Cloud computing.** Kamu mungkin tidak familiar dengan istilahnya, tapi kalau kamu memakai Gmail, Facebook, atau Twitter, kamu sudah memakainya. Model komputasi "baru" ini **sangat mirip dalam banyak hal dengan sistem mainframe zaman dulu**: seperti mainframe, sumber daya komputasi dipindahkan dari mesin lokal ke suatu tempat terpusat lain. Tiga modelnya: **IaaS, PaaS, SaaS**.

> [!info] Konteks tambahan (bukan dari slide)
> Mengapa kategori lingkungan ini penting secara forensik: di **stand-alone**, kamu tinggal sita komputernya. Di **networked**, buktinya tersebar di beberapa mesin dan log server. Di **cloud**, **komputer fisiknya bahkan tidak ada di negaramu** — kamu tidak bisa menyita apa pun, dan harus lewat proses hukum ke penyedia layanan, mungkin lintas yurisdiksi. Urutan empat kategori itu sebenarnya urutan **tingkat kesulitan akuisisi**, dari termudah ke tersulit.

### Filesystem
Dengan jutaan atau miliaran file berkeliaran di dalam komputer kita, harus ada cara untuk menjaga semuanya tetap rapi. Fungsi yang tak tergantikan ini adalah tanggung jawab **file system**.

File system **melacak ruang kosong drive serta lokasi tiap file**. Ruang kosong itu, yang juga dikenal sebagai **unallocated space**, statusnya bisa **kosong** atau **file yang sebelumnya menempati lokasi itu sudah dihapus**.

Dua filesystem yang disebut:
- **FAT (File Allocation Table)** — yang tertua dari filesystem umum
- **NTFS (New Technology File System)** — dipakai Windows 7, Vista, XP, dan Windows Server. **Jauh lebih kuat dari FAT** dan mampu melakukan lebih banyak fungsi; contohnya, *"NTFS bisa otomatis memulihkan beberapa error terkait disk, yang tidak bisa dilakukan FAT32."*

Analogi yang dipakai slide sendiri: bayangkan **ruang penyimpanan di dalam lemari arsip dengan banyak kompartemen**. Ada yang khusus menyimpan berkas berurutan alfabet, ada yang kronologis, ada kompartemen untuk alat tulis, lain-lain, bahkan barang acak. Meskipun semuanya dipakai menyimpan hal berbeda, semuanya bisa dilabeli dan mudah dikenali, serta diatur sedemikian rupa sehingga isi tiap kompartemen mudah diakses atau diambil.

Untuk menginstal sistem operasi apa pun di hard drive atau media penyimpanan lepas-pasang, **perangkat itu harus diformat dulu** dan disiapkan untuk sistem operasinya dengan memilih filesystem yang sesuai.

**Filesystem per sistem operasi:**

| OS | Filesystem | Versi yang didukung |
| --- | --- | --- |
| **Microsoft Windows** | **NTFS** | Server 2019/2016/2012/2008, Windows 10, 8, 7, Vista, XP, 2000, NT |
| **Macintosh (macOS)** | **HFS+** (Hierarchical File System) | macOS sampai versi 1 *(lihat Pertanyaan Terbuka)* |
| **Linux** | **Ext4** (Fourth Extended File System) | Red Hat, Kali, Ubuntu, dan sebagainya |

Untuk Linux slide mencatat: beberapa filesystem tersedia, tapi **Ext4 yang direkomendasikan** kalau kamu tidak yakin harus pakai yang mana.

### Cara hard drive magnetik menyimpan data
Komputer menyimpan data di ruang-ruang terdefinisi yang disebut **sector**. Anggap sector sebagai **wadah terkecil yang bisa dipakai komputer** untuk menyimpan informasi. **Tiap sector menampung sampai 512 byte data.**

> [!info] Konteks tambahan (bukan dari slide)
> Angka 512 byte ini adalah fondasi dari konsep **file slack** yang dibahas di [[W03 - Disk and File System Analysis]]. Kalau file-mu cuma 1 byte tapi wadah terkecilnya 512 byte (atau lebih, kalau block size-nya 4K), maka ada **ruang sisa** yang isinya bukan datamu — melainkan data lama dari file yang dulu menempati tempat itu. Ruang sisa itulah tempat bukti sering ditemukan.

## Diagram & Visual
> [!warning] Deck ini punya beberapa gambar (slide 10 "Magic Number", slide 12, dan slide 22), tapi **semuanya dalam format WMF** (vector clipart lama) yang **tidak bisa ditampilkan Obsidian**, terutama di Android. Gambar-gambar itu tidak diikutkan ke vault.
>
> Yang paling merugikan: **slide 10 judulnya "Magic Number" dan isinya cuma gambar** — kemungkinan besar berisi **tabel file signature** (contoh magic number per jenis file). Itu materi yang sangat mungkin keluar di ujian. **Buka PPT aslinya di slide 10.**

## Rumus / Sintaks

Konversi karakter yang dicontohkan slide:
```
'A' (besar)  = 01000001 binary
'a' (kecil)  = 01100001 binary = 0x61
'M' (besar)  = 0x4D

1 sector = 512 byte
ASCII    = 128 karakter (94 printable)
```

## Pertanyaan Terbuka
- **Slide 10 ("Magic Number") isinya cuma gambar WMF yang tidak bisa diekstrak.** Ini materi inti — tabel file signature. Prioritas utama untuk dibuka manual.
- Slide 3 mencantumkan **"Metadata in Filesystems"** dan **"Slack Space and Unallocated Space"** sebagai learning outcome, tapi **slack space tidak pernah dijelaskan** di deck ini (unallocated space cuma disinggung satu kalimat). Materinya baru muncul di [[W03 - Disk and File System Analysis]]. Perlu dipastikan mana yang jadi acuan ujian.
- Tabel filesystem menulis macOS memakai **"HFS+, supported versions: macOS up to version 1"** — angka "1" ini jelas salah ketik. Kemungkinan maksudnya macOS 10.12 (Sierra) ke bawah, karena setelah itu Apple pindah ke **APFS**. Konfirmasi ke dosen; bandingkan juga dengan [[W05 - Mac OS X Systems and Artifacts]].
- Deskripsi NTFS masih menyebut **"Windows 7, Vista, XP"** sebagai yang terkini — deck ini jelas ditulis lama. Isi materinya tetap valid, tapi contoh versinya usang.
- **SRAM dan DRAM** disebut sebagai "dua jenis yang perlu diketahui", tapi **bedanya tidak pernah dijelaskan**.
- Tabel jenis RAM sangat detail (sampai angka MT/s dan GB/s), padahal **relevansi forensiknya tidak dijelaskan sama sekali**. Perlu ditanya apakah angka-angka ini perlu dihafal.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W01 - Digital Forensic Fundamental]]
- [[W03 - Disk and File System Analysis]]
- [[Forensics - Review dan Glosari]]
