---
matkul: Forensics
minggu: 12
sks: 2
sumber: 12 Automating Analysis and Timeline Analysis.pptx
tags: [kuliah/forensics, minggu/w12]
status: draft
diproses: 2026-09-03
---

# W12 — Scripting and Automation for Forensic Tasks

> [!note] Judul di slide 1 adalah **"Scripting and Automation for Forensic Tasks"**, sementara nama file PPT-nya "Automating Analysis and Timeline Analysis". Judul note ini mengikuti slide, sesuai konvensi vault.

## Ringkasan
> - Tujuan otomasi dinyatakan jelas: **mengurangi jumlah persiapan yang dibutuhkan sebelum pekerjaan analisis yang sesungguhnya bisa dimulai.** Identifikasi partisi, filesystem, akses isi, cari file — semua itu tidak menarik dan tidak langsung relevan dengan tujuan pemeriksaan.
> - **Autopsy** = front end grafis untuk **The Sleuth Kit**, dibuat **Brian Carrier**. Sudah **built-in di Kali Linux**.
> - **PyFLAG** = Python-based Forensics and Log Analysis GUI; berbasis **web + database**, jadi user cukup butuh browser.
> - **Timeline dulu hampir seluruhnya hanya berisi MAC times filesystem**; sekarang nilai penambahan sumber data lain sudah disadari dan makin dimanfaatkan.
> - **Tiga relative time**: **before, after, during**. Titik awal kanonik semua timestamp filesystem adalah **waktu pembuatan filesystem itu sendiri** — timestamp sebelum itu **pasti palsu**.
> - **Inferred time** dipakai kalau timestamp granular tidak tersedia (umumnya untuk data terhapus). **Embedded time** datang dari isi file — misalnya **versi software pembuat PDF** menentukan tanggal paling awal file itu bisa ada.
> - **Periodicity** = jarak waktu antar peristiwa berulang. Berguna untuk **mendeteksi beacon backdoor** — tapi hati-hati, **program auto-update juga punya pola yang sama**.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Graphical investigation environment | Platform pemeriksaan berbasis GUI |
| Autopsy | Browser forensik grafis, front end visual untuk The Sleuth Kit |
| The Sleuth Kit | Kumpulan tool forensik command-line |
| Brian Carrier | Pembuat Autopsy (dan sumber pembedaan partition/volume di W03) |
| PyFLAG | Python-based Forensics and Log Analysis GUI |
| Relative time | Hubungan satu peristiwa dengan peristiwa lain: before, after, during |
| Window of compromise | Durasi antara masuknya penyerang sampai remediasi berhasil |
| Inferred time | Waktu yang disimpulkan ketika timestamp langsung tidak tersedia |
| Embedded time | Informasi waktu yang berasal dari isi file itu sendiri |
| Periodicity | Jarak waktu yang berlalu antar peristiwa yang berulang |
| Beacon traffic | Trafik berkala dari backdoor ke pengendalinya |
| NSRL | NIST National Software Reference Library, basis data hash file dikenal |
| Events sequencer | Fitur Autopsy yang menampilkan peristiwa terurut tanggal dan waktu |

## Isi

### Kenapa perlu lingkungan grafis
Banyak tool yang dibahas di sesi-sesi sebelumnya berbasis **command line atau konsol**, yang bisa jadi keunggulan besar dan memberi banyak fleksibilitas. Tapi **sebagian alur kerja lebih cocok di lingkungan grafis**.

Keunggulan memakai platform pemeriksaan grafis:
- **Manajemen kasus yang terintegrasi**
- **Pencarian kata kunci bawaan**
- **Kontinuitas yang lebih baik** saat memeriksa antar berbagai lapisan filesystem

Contoh utama lingkungan forensik grafis open source adalah **Autopsy browser**, dibuat **Brian Carrier** sebagai **front end visual untuk tool The Sleuth Kit**.

**PyFLAG** adalah **Python-based Forensics and Log Analysis GUI**, dibuat **Michael Cohen dan David Collett** untuk mendukung **pemeriksaan terpadu atas berbagai jenis data berbeda** yang sering ditemui dalam pemeriksaan forensik modern.

Karena PyFLAG adalah aplikasi **berbasis web dengan backend database**, **user umumnya cukup butuh web browser** untuk melakukan pemeriksaan. Menjadi aplikasi web/database memberi PyFLAG beberapa keunggulan dibanding utilitas forensik yang lebih tradisional.

> [!info] Konteks tambahan (bukan dari slide)
> Nama **Brian Carrier** muncul dua kali di matkul ini: di [[W03 - Disk and File System Analysis]] sebagai sumber pembedaan *partition* vs *volume*, dan di sini sebagai pembuat Autopsy. Bukan kebetulan — dia penulis *File System Forensic Analysis*, buku rujukan standar bidang ini, dan **Autopsy pada dasarnya adalah wujud kode dari model abstraksi filesystem** yang kamu pelajari di W03. Kalau kamu paham lapisan Disk→Volume→FS→Data Unit→Metadata→File Name, tampilan Autopsy langsung masuk akal.

### Tujuan otomasi ekstraksi artifact
Slide menyatakan masalahnya dengan jujur:

> Salah satu tantangan tetap dalam analisis forensik adalah **banyaknya persiapan yang dibutuhkan sebelum analisis yang sesungguhnya bisa dilakukan**. Kalau kamu cuma ingin mengidentifikasi dan mengekstrak satu jenis file tertentu, kamu **tetap harus** mengidentifikasi partisi di image, mengidentifikasi filesystem-nya, mengakses isinya, menemukan file yang dicari, dan seterusnya. **Sebagian besar pekerjaan awal ini tidak menarik dan tidak langsung relevan dengan tujuan pemeriksaannya.**
>
> **Tujuan otomasi ekstraksi artifact adalah mengurangi jumlah persiapan yang dibutuhkan sebelum pekerjaan yang sesungguhnya bisa dimulai.**

### Timeline
Timeline adalah **tool analisis yang sangat berguna**, dan **belakangan dipakai jauh lebih luas** dibanding 5 atau 10 tahun sebelumnya.

Perkembangan pentingnya:

> Di masa lalu, **sebagian besar timeline hampir seluruhnya hanya terdiri dari MAC times filesystem** (modified, accessed, creation). Belakangan, **nilai dari menambahkan sumber data lain sudah disadari**, dan analis makin memanfaatkannya.

> [!info] Konteks tambahan (bukan dari slide)
> Inilah tempat di mana semua materi sebelumnya bertemu. MAC times dari [[W06 - Linux System and Artifacts]], timestamp Windows dari [[W07 - Windows System Artifacts]], EXIF dari [[W09 - File and Archives Analysis]], riwayat browser dari [[W10 - Internet Artifacts]], log dari mana-mana — **timeline adalah wadah yang menggabungkan semuanya jadi satu urutan peristiwa**. Itu sebabnya "timeline reconstruction" muncul di LO 3 seluruh matkul: dia bukan satu teknik, melainkan **hasil akhir** dari semua teknik.

### Relative time: before, after, during
Slide memperkenalkan **relative time** untuk membicarakan hubungan satu peristiwa dengan peristiwa lain. Ada **tiga**: **before, after, dan during**.

> [!warning] Slide 8 menulis "**four different types of relative times**" lalu langsung menyebut "**There are three relative times to consider**". Angkanya tidak konsisten di kalimat yang sama, dan **cuma tiga yang dijelaskan**.

**Before dan after** adalah konsep yang sangat sederhana tapi bisa diterapkan dengan cara menarik. Titik diskrit mana pun di timeline bisa digambarkan terjadi sebelum atau sesudah titik atau peristiwa lain. Kebalikannya juga benar: **peristiwa yang bergantung pada peristiwa lain tidak mungkin terjadi sebelum peristiwa itu**.

Contoh kanoniknya, dan ini penting:

> **Pembuatan filesystem yang sedang diperiksa** adalah **"titik awal" kanonik untuk semua titik waktu filesystem**. **Informasi waktu apa pun yang menunjukkan tanggal sebelum ini adalah palsu** — entah dipalsukan dengan sengaja, atau sebagai konsekuensi dari **mempertahankan waktu asli saat ekstraksi arsip**.

**During** menggambarkan sekumpulan data waktu dengan **awal dan akhir yang terbatas** — yaitu **durasi peristiwa**. Contoh utamanya: **window of compromise** dalam sebuah intrusi.

- Diberikan **waktu masuknya penyerang** dan **waktu remediasi berhasil**, waktu di antaranya adalah **durasi kompromi**. Ini **sering jadi periode waktu terbesar yang langsung relevan**.
- **Di dalam window itu, durasi tambahan bisa ditemukan.** Contohnya, **jendela antara login dan logout akun yang dikompromikan** bisa memberi window akses tambahan untuk diselidiki lebih jauh.
- Manfaatnya: ini **membantu mengurangi jumlah informasi waktu relevan yang harus diproses examiner**.

> [!info] Konteks tambahan (bukan dari slide)
> Aturan "timestamp sebelum pembuatan filesystem itu palsu" adalah salah satu **pemeriksaan kewarasan** paling berguna yang bisa kamu lakukan, dan gratis. Tapi perhatikan penjelasan keduanya dari slide: bisa juga **bukan pemalsuan** — file yang diekstrak dari arsip ZIP/TAR **membawa timestamp aslinya**, yang bisa jauh lebih tua dari disk itu sendiri. Jadi temuan ini artinya **"selidiki lebih lanjut"**, bukan otomatis "ada yang berbohong". Ini menyambung ke [[W09 - File and Archives Analysis]]: *"banyak archive mempertahankan sebagian timestamp filesystem"*.

### Inferred time dan embedded time
**Inferred time.** Dalam banyak investigasi, **titik waktu yang benar-benar granular untuk data yang dicari mungkin tidak tersedia, atau salah, atau menyesatkan**. Kalau begitu keadaannya, **waktu yang disimpulkan mungkin jadi waktu paling akurat yang kamu punya**.

Umumnya ini berlaku untuk **data yang terhapus**: data yang sudah dihapus **tidak punya struktur metadata terkait secara langsung**, atau punya struktur metadata yang **hanya terhubung longgar** ke isi datanya. Dalam keadaan seperti ini, **informasi temporal tambahan masih mungkin disimpulkan**.

**Embedded time.** Isi file bisa jadi **sumber informasi waktu yang lain**. Dua contoh dari slide:

- **Banyak file PDF berisi nama dan versi software yang membuatnya.** **Tanggal rilis versi software itu adalah tanggal paling awal file itu mungkin ada.**
- Saat memeriksa **dokumen office yang keasliannya dipertanyakan**, keberadaan **merek dan model printer** bisa dipakai untuk **membuat kerangka waktu kapan dokumen itu pasti dicetak**.

> [!info] Konteks tambahan (bukan dari slide)
> Ini teknik yang elegan dan layak diingat karena **tidak bisa dilawan dengan mengubah timestamp**. Seseorang bisa mengatur ulang jam sistem dan memalsukan tanggal file, tapi kalau metadata di dalam dokumen menyebut **"dibuat dengan Word 2019"** sementara dokumennya mengaku ditandatangani tahun 2015, **dokumen itu berbohong** — dan penipunya harus tahu tentang metadata tertanam untuk bisa menutupinya. Kebanyakan tidak tahu.

### Periodicity
**Periodicity** merujuk pada **laju terulangnya suatu peristiwa atau aktivitas**. Slide membuat pembedaan yang halus tapi penting:

> Ini kadang disebut "frequency", tapi **frequency** bisa juga merujuk pada **seberapa sering** sesuatu terjadi ("empat kali sehari"), sementara **periodicity merujuk secara eksplisit pada waktu yang berlalu di antara peristiwa yang berulang**.

Kegunaannya:

- **Periodicity adalah sinyal yang berguna saat menganalisis data waktu terkait trafik backdoor.** **Sebagian besar program backdoor punya periode yang sangat tetap untuk beacon traffic** kembali ke pengendalinya.
- **Sayangnya, banyak program auto-update yang tidak berbahaya juga punya karakteristik yang sama.**
- Maka tidak mengherankan kalau **periodicity bisa dipakai untuk mengklasifikasikan trafik otomatis versus trafik manusia**.

> [!info] Konteks tambahan (bukan dari slide)
> Alasan ini bekerja sederhana: **manusia tidak teratur, mesin teratur.** Orang mengecek email jam 9:03, lalu 9:47, lalu 11:12 — berantakan. Backdoor menghubungi pengendalinya **tepat setiap 300 detik**, selamanya. Keteraturan itu sendiri yang jadi tanda bahaya, terlepas dari isi trafiknya (yang mungkin terenkripsi dan tidak terbaca — lihat batasan SSL di [[W10 - Internet Artifacts]]). Peringatan soal auto-update itu jujur dan penting: **periodicity mempersempit kecurigaan, tidak membuktikan apa pun sendirian.**

### Autopsy
**Autopsy menawarkan akses GUI ke berbagai tool investigasi command-line dari The Sleuth Kit**, termasuk file analysis, image dan file hashing, pemulihan file terhapus, dan manajemen kasus.

Catatan praktis dari slide: **Autopsy bisa merepotkan saat instalasi, tapi untungnya sudah built-in di Kali Linux** dan sangat mudah disiapkan serta dipakai.

Slide juga mencatat bahwa **fitur Autopsy berbeda antara versi Windows dan versi Linux**. Fitur resmi The Sleuth Kit dan **Autopsy 2.4 di Kali Linux**:

| Fitur | Isinya |
| --- | --- |
| **Image analysis** | Menganalisis direktori dan file, termasuk **menyortir file, memulihkan file terhapus, dan mempratinjau file** |
| **File activity timelines** | **Membuat timeline berdasarkan timestamp file** — kapan ditulis, diakses, dan dibuat |
| **Image integrity** | Membuat **hash MD5** dari file image yang dipakai, maupun dari file individual |
| **Hash databases** | Mencocokkan hash file yang tidak dikenal (misalnya `.exe` yang dicurigai berbahaya) dengan yang ada di **NIST National Software Reference Library (NSRL)** |
| **Events sequencer** | Menampilkan **peristiwa terurut berdasarkan tanggal dan waktu** |
| **File analysis** | Menganalisis seluruh file image untuk menampilkan informasi dan isi direktori serta file |
| **Keyword search** | Pencarian memakai kata kunci dan **daftar ekspresi yang sudah ditentukan** |
| **Metadata analysis** | Melihat detail dan struktur metadata file yang **esensial untuk pemulihan data** |
| **Parsing data and indexing** | Menempatkan **"topeng virtual" di atas barang bukti sebenarnya**, sehingga investigator bisa menjalankan query **tanpa mengubah "source data"** atau buktinya |
| **Report generating** | Menyusun temuan jadi **laporan yang mudah dibaca** |

> [!info] Konteks tambahan (bukan dari slide)
> Dua fitur di tabel itu adalah wujud langsung dari prinsip yang diajarkan sepanjang semester:
> - **"Topeng virtual ... tanpa mengubah source data"** = versi software dari **write blocker** di [[W05 - Data Acquisition]].
> - **Hash database vs NSRL** = cara **membuang** file yang membosankan, bukan menemukan yang menarik. NSRL berisi hash jutaan file sistem dan aplikasi yang dikenal; kalau hash sebuah file cocok, itu file Windows biasa dan **bisa diabaikan**. Di image berisi 500.000 file, menyingkirkan 480.000 yang dikenal itu jauh lebih berharga daripada mencari satu per satu.

## Diagram & Visual
> [!warning] Deck ini **tidak punya satu pun gambar non-dekoratif** dari 18 slide. **Tidak ada satu pun screenshot Autopsy** — padahal seluruh dua slide terakhir mendaftar fitur-fitur GUI-nya. Untuk praktikum, tampilan Autopsy harus dilihat langsung di Kali.

## Rumus / Sintaks

Tiga relative time:
```
before  : titik A terjadi sebelum titik B
after   : titik A terjadi sesudah titik B
during  : durasi peristiwa (awal + akhir), contoh: window of compromise
```

Aturan kewarasan timestamp:
```
titik awal kanonik = waktu FILESYSTEM DIBUAT
timestamp lebih tua dari itu  ->  palsu, ATAU sisa dari ekstraksi arsip
```

Sumber waktu selain timestamp filesystem:
```
inferred time : disimpulkan, dipakai saat metadata hilang (umumnya data terhapus)
embedded time : dari ISI file
   - versi software pembuat PDF -> tanggal PALING AWAL file bisa ada
   - merek/model printer di dokumen -> kerangka waktu pencetakan
```

Periodicity:
```
periodicity = JARAK WAKTU antar peristiwa berulang   (bukan "berapa kali sehari")
tetap & teratur -> mesin (beacon backdoor ATAU auto-update)
tidak teratur   -> manusia
```

## Pertanyaan Terbuka
- Slide 3 mencantumkan **sembilan learning outcome**, dan **lima tidak dibahas sama sekali**: **Introduction to Python for Forensic Automation**, **Common Python Libraries for Forensic (dfVFS, plaso, pytsk3)**, **Batch Processing for Large Dataset**, **Generating Automated Timeline Reports**, dan **Automation Pitfalls and Limitations**.

  Ini serius: **judul deck-nya "Scripting and Automation", tapi tidak ada satu baris kode pun di dalamnya.** Tidak ada Python, tidak ada `plaso`/`log2timeline` (padahal itu tool standar industri untuk membuat timeline otomatis), tidak ada `pytsk3`. **Wajib ditanyakan.**
- **"Advantages and Disadvantages of Automating Forensic"** juga jadi LO — slide membahas *advantages*-nya, tapi **disadvantages dan pitfalls tidak pernah disebut**. Padahal itu justru bagian yang penting: tool otomatis bisa salah, dan examiner tetap bertanggung jawab atas hasilnya.
- Slide 4 kalimatnya **terpotong di tengah**: *"The Autopsy browser does not provide."* — tidak ada lanjutannya. Kemungkinan ada informasi yang hilang tentang keterbatasan Autopsy.
- Slide 8 menyebut **"four different types of relative times"** lalu **"three relative times to consider"** dalam satu slide. Kalau ditanya di ujian, jawabannya kemungkinan besar **tiga** (before, after, during) karena cuma itu yang dijelaskan.
- **Autopsy 2.4** yang disebut slide sudah sangat lama; Autopsy modern sudah versi 4.x dengan antarmuka yang jauh berbeda. Perlu dipastikan versi mana yang dipakai di praktikum.
- **PyFLAG dibahas satu slide penuh** padahal proyeknya sudah lama tidak dikembangkan. Perlu ditanya apakah ini masih diujikan.
- **Kaitan antara materi timeline di deck ini dan judul [[W11 - Timeline Analysis and Correlation of Artifacts]]** tidak pernah dijelaskan. Sebenarnya **materi timeline yang dijanjikan W11 justru ada di sini** — kemungkinan urutan atau pembagian deck-nya tertukar. Ini layak dikonfirmasi.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W03 - Disk and File System Analysis]]
- [[W06 - Linux System and Artifacts]]
- [[W07 - Windows System Artifacts]]
- [[W11 - Timeline Analysis and Correlation of Artifacts]]
- [[W13 - Network Analysis]]
- [[Forensics - Review dan Glosari]]
