---
matkul: Forensics
minggu: 9
sks: 2
sumber: 9 File adn Archives Analysis.pptx
tags: [kuliah/forensics, minggu/w09]
status: draft
diproses: 2026-09-03
---

# W09 — File and Archives Analysis

## Ringkasan
> - File analysis dipecah jadi **dua aktivitas yang berbeda tapi saling melengkapi**: **content identification** (file ini sebenarnya apa) dan **metadata extraction** (metadata apa yang tertanam di dalamnya).
> - Analogi slide yang bagus: **seorang dokter bukan dokter karena menulis "Dr." di depan namanya.** Sama, file teks tetap file teks apa pun namanya. **Extension cuma petunjuk untuk Windows shell — konvensi demi kenyamanan, tidak lebih.**
> - **Tiga jenis metadata gambar**: **EXIF** (info perangkat: merek/model kamera, waktu, **geolokasi**), **IPTC** (untuk pers/fotojurnalis), **XMP** (berbasis XML dari Adobe, 2001, **extensible dan bisa dipakai jenis file lain**).
> - **Container vs codec**: file video adalah **container** yang berisi satu atau lebih **stream**; **codec** adalah metode kompresi/encoding stream itu. Butuh codec yang tepat untuk memutarnya.
> - **FourCC codes** — 4 byte yang mengidentifikasi codec di AVI, **mirip magic number**.
> - **Archive** bisa mempertahankan **timestamp filesystem** dan bahkan **UID/GID** dari sistem asalnya — ini artifact yang berharga.
> - **RAR adalah format pilihan grup pembajakan**, dan **sering dipakai untuk eksfiltrasi data** saat intrusi komputer.
> - Saat buku sumbernya ditulis, **PDF berbahaya adalah salah satu vektor utama kompromi sistem desktop**.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Content identification | Proses menentukan atau memverifikasi sebuah file itu sebenarnya apa |
| Metadata extraction | Pengambilan metadata tertanam yang ada di dalam sebuah file |
| EXIF | Exchangeable Image File Format; metadata perangkat perekam gambar |
| IPTC | Information Interchange Model dari International Press Telecommunications Council |
| XMP | eXtensible Metadata Platform berbasis XML dari Adobe (2001) |
| Container | File yang menampung satu atau lebih stream |
| Stream | Aliran data (video/audio) di dalam container |
| Codec | Metode kompresi dan encoding data dalam stream |
| RIFF | Resource Interchange File Format; container induk WAV dan AVI |
| INFO chunk | Bagian RIFF yang berisi tag metadata |
| FourCC code | Urutan tetap 4 byte pengidentifikasi codec di AVI; **mirip magic number** |
| Lossy compression | Kompresi yang membuang data; MP3 mencapai rasio 1:10 |
| AAC / M4A | Penerus MP3; disimpan di container MPEG-4 |
| ASF | Advanced Systems Format, container Microsoft untuk streaming (WMA/WMV) |
| Matroska (MKV) | Container terbuka; bisa membawa audio, video, gambar, dan subtitle |
| Tarball | Arsip `tar` yang dikompresi GZIP atau BZIP2 |
| Document Information Directory | Metadata PDF berupa pasangan key/value |
| exiftool / hachoir-metadata / AtomicParsley / qtinfo | Tool ekstraksi metadata yang disebut slide |

## Isi

### Kenapa analisis file itu penting
Untuk melakukan pemeriksaan yang komprehensif, **kita harus memahami sifat dari file yang kita identifikasi dan ekstrak**. Dengan memahami file-file itu, kita bisa **lebih berhasil mengungkap dan memanfaatkan artifact forensik tingkat lebih tinggi** yang mungkin ada di dalamnya.

Slide memberi beberapa skenario di mana analisis file jadi kunci:
- **Dokumen berbahaya** bisa jadi titik masuk awal dalam investigasi kompromi sistem
- **Keabsahan sebuah dokumen penting** mungkin dipertanyakan
- Examiner mungkin perlu **menemukan dan mengidentifikasi gambar atau video terlarang**
- **Keberadaan file yang sama di dua mesin berbeda bisa mengikat mesin-mesin itu dan penggunanya**

Dan satu pengamatan yang tajam:

> Fakta bahwa file-file ini **dirancang untuk berdiri sendiri (self-contained) dan dibagikan antar sistem** adalah salah satu karakteristik kunci yang membuatnya jadi **sumber artifact yang menarik**.

**Dua aktivitas** dalam file analysis:

| Aktivitas | Isinya |
| --- | --- |
| **Content identification** | Proses **menentukan atau memverifikasi sebuah file itu apa** |
| **Metadata extraction** | Pengambilan **metadata tertanam** yang mungkin ada di dalam file tersebut |

### Content identification: nama bukan identitas
Sebagian besar pengguna komputer familiar dengan konsep file extension di Windows untuk mengidentifikasi jenis file. **Tapi nama sebuah file bukanlah yang membuat file itu jadi file tersebut.**

Analogi dari slide, dan ini bagus:

> **Seorang dokter bukanlah dokter karena dia menulis "Dr." di depan namanya** — dia dokter karena bertahun-tahun sekolah dan pelatihan medis. **Bahkan tanpa gelarnya, dia tetap dokter.** Serupa dengan itu, **file teks sederhana tetaplah file teks**, apakah namanya "MyFile.txt" atau "ThisIsAnEmptyFolder". Extension memberi **petunjuk** kepada Windows shell soal isi file — **itu konvensi demi kenyamanan, tidak lebih dari itu**.

Dua alasan kenapa examiner **tidak bisa begitu saja menerima petunjuk itu sebagai kebenaran**:

1. **User bisa mengubah file association dan aplikasi default** yang dipakai membuka file tertentu, dalam upaya **menyembunyikan sifat file-file itu**. Contoh dari slide: user bisa mengubah semua file video AVI supaya berekstensi **`.BIN`**, lalu mengasosiasikan extension itu dengan pemutar video.
2. **Selama pemeriksaan, tidak jarang ditemukan file tanpa extension sama sekali** di lokasi yang menarik, atau dengan timestamp yang menempatkan file itu di periode waktu yang menarik. Umumnya ini **file temporary atau cache** yang tidak dimaksudkan untuk dikonsumsi end-user — dan **bisa berisi data investigasi yang krusial**.

> [!info] Konteks tambahan (bukan dari slide)
> Ini pengulangan ketiga dari tema yang sama di matkul ini: [[W02 - Key Technical Concepts]] memperkenalkan **magic number**, [[W08 - File Recovery and Data Carving]] memakainya untuk **carving**, dan di sini dipakai untuk **verifikasi**. Kalau ada satu prinsip yang jelas diulang-ulang dosen sepanjang semester, ini dia: **jangan percaya nama file, periksa isinya.**

### Gambar dan metadata-nya
Gambar adalah file yang berisi data untuk dirender sebagai grafik. Selain itu, **sebagian besar jenis file gambar mampu membawa berbagai metadata**, mulai dari komentar teks sederhana sampai **garis lintang dan bujur tempat gambar itu dibuat**.

Tergantung investigasinya, examiner mungkin tertarik pada:
- **Isi gambarnya** (misalnya foto orang tertentu), atau
- **Metadata-nya** (informasi yang menandakan gambar mungkin **sudah diubah dengan software editing**)

**Tiga jenis metadata gambar** yang dipakai hari ini:

| Jenis | Kepanjangan | Dikembangkan untuk | Isinya |
| --- | --- | --- | --- |
| **EXIF** | Exchangeable Image File Format | Menanamkan informasi tentang **perangkat yang menangkap gambar** (biasanya kamera) ke dalam gambarnya sendiri | Serangkaian **Tag dan Value**: **merek dan model kamera**, **tanggal dan waktu** gambar diambil, dan **informasi geolokasi** perangkat |
| **IPTC** | Information Interchange Model, dari International Press Telecommunications Council | Awalnya untuk menanamkan informasi tentang gambar yang dipakai **koran dan kantor berita**. Kadang disebut **"IPTC Headers"** | Dipakai terutama oleh **fotojurnalis** dan industri yang memproduksi gambar digital untuk cetak |
| **XMP** | eXtensible Metadata Platform | Dikembangkan **Adobe tahun 2001**; berbasis **XML** | **Sebagian besar menggantikan skema metadata sebelumnya**; bersifat **terbuka dan extensible**, sehingga **bisa dipakai untuk jenis file lain juga** |

> [!info] Konteks tambahan (bukan dari slide)
> **EXIF adalah artifact paling produktif dari ketiganya**, karena satu alasan: **geolokasi**. Sebuah foto bisa memberitahumu **di mana** dan **kapan** ia diambil, dan **dengan kamera atau ponsel mana**. Kalau kamu menemukan foto yang sama di dua perangkat, EXIF-nya bisa memberitahu perangkat mana yang **memotretnya** dan mana yang **cuma menerimanya**. Perhatikan juga sisi lain: karena metadata bisa **diedit**, keberadaan EXIF yang **hilang atau ganjil** juga bermakna — itu tanda gambar sudah lewat software editing, yang relevan untuk **file tampering detection**.

### Audio
File audio berisi data yang **menghasilkan suara ketika di-decode dengan benar** — bisa musik, pesan voicemail, atau materi rekaman lain. Kalau kasusmu berputar pada identifikasi dan ekstraksi data audio, **kemungkinan isi audionya yang jadi kepentingan utama**. Tapi **format audio bisa membawa metadata yang kaya**.

| Format | Keterangan |
| --- | --- |
| **WAV** | Waveform Audio File Format, standar penyimpanan bitstream audio yang dikembangkan **Microsoft dan IBM** untuk PC desktop. Disimpan sebagai serangkaian **tagged chunk** di dalam container **RIFF**. RIFF mendukung **INFO chunk** yang berisi berbagai tag metadata; selain itu file RIFF juga **bisa memuat metadata XMP** |
| **MP3** | Format paling populer untuk musik digital. Diterbitkan **MPEG tahun 1993**, lalu jadi format pilihan untuk jaringan file sharing awal seperti **Napster dan Gnutella**. Memakai **skema kompresi lossy berbasis studi pendengaran manusia**, memungkinkan **rasio kompresi 1:10** — satu lagu tipikal jadi sekitar **5–6 MB** |
| **MP4 / AAC / M4A** | **Advanced Audio Coding (AAC)** adalah **penerus modern MP3**. File AAC bisa disebut **"MP4 Audio"** atau **"M4A"**. Audio AAC biasanya disimpan di container **MPEG-4**. Karena container MP4 bisa menyimpan audio, video, atau keduanya, **penamaan M4A dipakai sebagai petunjuk bahwa container MP4 ini hanya berisi audio** |
| **ASF / WMA** | **Advanced Systems Format (ASF)** adalah container Microsoft yang dirancang untuk **streaming media**. Dipakai menyimpan **Windows Media Audio (WMA)** dan **WMV**. Container ASF dan codec WMA **keduanya proprietary milik Microsoft**, tapi sudah di-**reverse engineer** sampai bisa diputar atau dikonversi memakai software berbasis **libavcodec** seperti **ffmpeg** atau **VLC**. Metadata-nya bisa diekstrak dengan **exiftool** dan **hachoir-metadata** |

### Video: container dan codec
File video berisi (paling tidak) data yang **di-decode jadi urutan gambar bergerak**. Mayoritas besar file video **juga berisi komponen audio yang disinkronkan** dengan komponen videonya.

Karena ada dua komponen berbeda ini, **file video umumnya dibuat dalam bentuk container file yang berisi satu atau lebih stream**. Metode yang dipakai untuk **mengompresi dan meng-encode data di stream itu disebut codec**. **Untuk memutar kontennya dengan sukses, dibutuhkan codec yang tepat.**

| Format | Keterangan |
| --- | --- |
| **MPEG-1 / MPEG-2** | Standar kompresi dan transmisi video-audio dari MPEG. **MPEG-1** (kadang disebut file MPG) dipakai di **Video CD (VCD)** tahun 1990-an — populer di Asia tapi tidak meluas di Amerika. **MPEG-2** adalah standar berikutnya dengan kualitas lebih tinggi, dipakai di **DVD serta transmisi kabel dan satelit digital**. **Keduanya tidak membawa metadata yang berarti** |
| **MPEG-4 / MP4** | Container MPEG-4 juga menampung konten video, umumnya berekstensi **MP4**. Sebagian besar file MP4 yang ditujukan untuk diputar di berbagai sistem punya **video stream dengan MPEG-4 Advanced Video Codec (MPEG-4 AVC), disebut juga H.264**, dan **audio stream AAC**. Karena container-nya sama, metadata bisa diekstrak memakai **AtomicParsley**, sama seperti MP4 audio |
| **AVI** | **Audio Video Interleave**, container yang diperkenalkan **Microsoft tahun 1992**. Seperti WAV, **AVI keturunan format RIFF** — menyimpan isi sebagai "chunk" dan bisa punya **INFO chunk** untuk metadata; **metadata XMP juga bisa ditanam**. Codec di dalam AVI diidentifikasi memakai **FourCC code**, urutan tetap **4 byte**. Slide menyebutnya **mirip magic number**: kode ini memberi tahu software pemutar codec apa yang dibutuhkan |
| **ASF / WMV** | **Windows Media Video (WMV)**, format proprietary Microsoft untuk kompresi video, disimpan di container **ASF** dan umumnya disertai **stream WMA yang tersinkronisasi** |
| **MOV** | **QuickTime File Format**, lebih dikenal lewat extension **`.MOV`** dari film Apple QuickTime. **Sebagian besar sudah digantikan MPEG-4**, tapi file QuickTime lama masih bisa ditemukan saat investigasi. Metadata-nya diekstrak dengan **`qtinfo`**, bagian dari paket `quicktimeutils` di Ubuntu |
| **MKV** | **Matroska Multimedia Container**, standar terbuka yang relatif baru; bisa membawa **audio, video, gambar, dan track subtitle**. Populer di kalangan pelaku **file sharing**, terutama untuk video **animasi Jepang**, sebagian karena kemampuannya **membawa subtitle di dalam file** |

> [!info] Konteks tambahan (bukan dari slide)
> Pembedaan **container vs codec** ini sering bikin bingung, tapi analoginya sederhana: **container itu kotak makan, codec itu cara makanannya dimasak.** Kotak yang sama (MP4) bisa berisi masakan berbeda (H.264, H.265, dan lain-lain). Itu sebabnya file MP4 kadang tidak bisa diputar meski aplikasimu "mendukung MP4" — kotaknya kamu kenal, isinya tidak. Dan **FourCC code** adalah label di kotak itu yang memberitahu isinya dimasak dengan cara apa.

### Archive forensic
**Archive file adalah container file yang dirancang untuk menampung file lain.** Container ini umumnya bisa **menerapkan berbagai algoritma kompresi** ke file yang dimuatnya, dan **mungkin mendukung enkripsi** isinya.

Yang paling berharga secara forensik:
- Archive file mungkin punya **sedikit metadata**, biasanya berupa catatan yang disediakan user
- **Banyak archive mempertahankan sebagian timestamp filesystem** saat menambahkan file ke container
- **Sebagian jenis archive mempertahankan informasi dari sistem asalnya**, termasuk **informasi UID dan GID dari sistem mirip-Unix**

| Format | Keterangan |
| --- | --- |
| **ZIP** | Salah satu format kompresi dan arsip **tertua yang masih dipakai**. Didukung di platform mana pun. Bisa memakai banyak mekanisme kompresi dan **dua bentuk enkripsi**: skema berbasis password yang **lemah** dari spesifikasi ZIP asli, dan bentuk yang lebih modern memakai **AES**. Perintah **`unzip`** bisa dipakai **untuk mengambil informasi isi arsip tanpa benar-benar mengekstraknya** — bagus untuk memeriksa **tanggal modifikasi file yang tertanam di arsip** |
| **RAR** | **Roshal Archive**, format proprietary yang dikembangkan **Eugene Roshal**. Fitur kuncinya: **kompresi sangat bagus**, **kemampuan perbaikan dan pemulihan arsip**, **pemecahan arsip (splitting)**, dan **enkripsi kuat bawaan**. Karena semua itu, **RAR adalah format pilihan grup pembajakan** yang mendistribusikan konten, dan **sering dipakai untuk mengeksfiltrasi data saat intrusi komputer** |
| **7-Zip (7z)** | Format arsip dan kompresi **terbuka** yang jadi populer sebagai pengganti fungsi ZIP maupun RAR. Punya **kompresi yang sangat efisien**, mendukung **enkripsi AES yang kuat**, dan mendukung **file yang sangat besar**. Programnya **open source** dan bisa memproses banyak format arsip lain di Linux, Windows, dan OS X |
| **Tar / Gzip / Bzip2** | Di Linux, **"tarball"** adalah metode standar mengarsipkan dan mengompresi data. Sesuai semangat Unix, **langkah pengarsipan dan kompresi dipisah ke tool berbeda**: perintah **`tar`** menggabungkan file terpilih jadi satu arsip solid, lalu arsip itu **dikompresi dengan GZIP atau BZIP2**. Hasilnya itulah "tarball". Versi `tar` modern menerima flag yang menunjukkan kompresi apa yang dipakai saat arsip dibuat |

> [!info] Konteks tambahan (bukan dari slide)
> Kalimat "**RAR sering dipakai untuk mengeksfiltrasi data saat intrusi komputer**" itu bukan trivia — itu **indikator investigasi**. Menemukan `.rar` besar yang baru dibuat di lokasi ganjil (folder temp, root direktori web) adalah salah satu tanda paling dikenal dari **staging data sebelum dicuri**. Penyerang mengarsipkan dulu supaya transfernya satu file dan terkompresi. Alasannya juga jelas dari daftar fitur: kompresi bagus (transfer cepat), splitting (potongan kecil lolos filter), enkripsi kuat (isinya tidak terbaca kalau tertangkap).

### Document forensic
"Document" adalah istilah yang relatif umum, tapi dalam konteks analisis file forensik, **"document" adalah jenis file yang berisi teks, gambar, dan informasi rendering**. Berbagai jenis file Microsoft Office dan **PDF** milik Adobe adalah contohnya.

Slide mencatat bahwa **ini bagian paling luas dari babnya**, karena banyaknya jenis dokumen yang bisa menarik dan karena **kekayaan data forensik yang bisa diambil dari file-file ini**.

**Rich Text Format (RTF)** — format dokumen yang dikembangkan Microsoft untuk **memfasilitasi transfer dokumen antar platform berbeda**. Ini format asli dokumen yang dibuat di **Windows Wordpad** dan aplikasi **TextEdit di Mac OS X**. RTF sudah dipakai **sejak 1987** dan mendapat beberapa pembaruan signifikan, termasuk **penambahan elemen XML** di revisi terbaru.

**PDF** — pada dasarnya, PDF adalah **container file yang menampung serangkaian instruksi layout PostScript beserta font dan grafik yang tertanam**. Selain menampilkan data dokumen, PDF bisa berisi **field formulir interaktif** seperti input teks dan checkbox, sehingga bisa menggantikan formulir kertas tradisional.

Selama bertahun-tahun, jenis konten yang bisa disimpan di PDF terus bertambah dan **kini mencakup link ke konten eksternal, JavaScript, dan objek film Flash**. Slide mencatat:

> Pada periode saat buku ini ditulis, **PDF berbahaya adalah salah satu vektor utama untuk kompromi sistem desktop.**

**PDF bisa berisi dua jenis metadata:**

| Jenis | Isinya |
| --- | --- |
| **Document Information Directory** | Pasangan **key/value** berisi **informasi kepengarangan**, **judul dokumen**, dan **timestamp pembuatan/modifikasi** |
| **XMP** | PDF modern mendukung **Extensible Metadata Platform** — metode yang sama yang dipakai menyimpan metadata di beberapa format file grafis |

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan kenapa PDF jadi vektor serangan favorit, karena logikanya langsung terbaca dari deskripsi di atas: PDF **bisa memuat JavaScript**. Artinya "dokumen" itu sebenarnya bisa **menjalankan kode**, sementara semua orang memperlakukannya seperti selembar kertas digital yang aman dibuka. Ini menyambung langsung ke kalimat di awal deck: *"dokumen berbahaya bisa jadi titik masuk awal dalam investigasi kompromi sistem"*.

## Diagram & Visual
- **Slide 14 — ilustrasi struktur container RIFF / format WAV**
  ![[99-Assets/Forensics/W09-slide14.png]]

> [!warning] Deck ini hampir seluruhnya teks — cuma **satu gambar non-dekoratif dari 34 slide**. Tidak ada satu pun **contoh output tool** (`exiftool`, `AtomicParsley`, `unzip -l`), padahal seluruh materi berbicara soal mengekstrak metadata. Untuk praktikum harus dicari sendiri.

## Rumus / Sintaks

Tiga jenis metadata gambar:
```
EXIF  -> info PERANGKAT   : merek/model kamera, tanggal-waktu, GEOLOKASI
IPTC  -> info PERS        : untuk fotojurnalis dan kantor berita
XMP   -> Adobe, 2001, XML : extensible, dipakai juga di PDF dan format lain
```

Container vs codec:
```
container = kotaknya   (MP4, AVI, ASF, MKV, RIFF)
codec     = isinya     (H.264/MPEG-4 AVC, AAC, WMA, WMV)
FourCC    = label 4-byte pengidentifikasi codec di AVI (mirip magic number)
```

Tool ekstraksi metadata yang disebut slide:
```
exiftool            : umum, ASF dan banyak format lain
hachoir-metadata    : ASF
AtomicParsley       : MP4 (audio maupun video)
qtinfo              : MOV / QuickTime (paket quicktimeutils di Ubuntu)
unzip               : lihat isi ZIP tanpa mengekstrak
```

## Pertanyaan Terbuka
- Slide 3 mencantumkan **sebelas learning outcome**, dan setidaknya **lima tidak dibahas**: **Audio Forensic** dan **Video Forensic** sebagai disiplin (deck ini cuma mendaftar **format** file, bukan teknik analisisnya), **Hidden Data and Steganography Detection**, **File Tampering Detection**, **Handling Encrypted or Compressed Archives**, dan **Hashing and Integrity Checking of Media Files**. **Wajib ditanyakan.**
- Perbedaan antara **"mendaftar format file"** dan **"melakukan forensik media"** itu besar. Deck ini seluruhnya berisi yang pertama. Tidak ada satu pun teknik untuk **mendeteksi gambar yang diedit**, **memverifikasi keaslian rekaman audio**, atau **menemukan splice pada video** — padahal itu yang biasanya dimaksud "image/audio/video forensic".
- **Steganography detection** jadi learning outcome di sini **dan** steganography dibahas di [[W08 - File Recovery and Data Carving]], tapi **tidak ada satu pun tool atau metode deteksi** yang disebut di kedua deck.
- **Microsoft Office file types** disebut sebagai contoh utama "document" di slide 30, lalu **tidak pernah dibahas sama sekali** — cuma RTF dan PDF yang dapat slide sendiri. Padahal metadata Office (author, revisi, waktu edit total) adalah salah satu artifact dokumen paling produktif.
- Slide 15 menulis **"Gnutella (formerly Kazaa)"** — ini keliru, Gnutella dan Kazaa dua jaringan yang berbeda, dan Kazaa bukan nama lama Gnutella.
- Slide 22 (ASF/WMV) menulis "memproses file ini sama persis dengan memproses **ASF/WMV**" — kemungkinan besar maksudnya **ASF/WMA** (bagian audio sebelumnya). Salah ketik.
- Beberapa keterangan sudah usang: **MP3 sebagai "format paling populer"**, **Napster/Gnutella**, dan **objek Flash di PDF** (Flash sudah mati sejak 2020).

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W02 - Key Technical Concepts]]
- [[W08 - File Recovery and Data Carving]]
- [[W10 - Internet Artifacts]]
- [[Forensics - Review dan Glosari]]
