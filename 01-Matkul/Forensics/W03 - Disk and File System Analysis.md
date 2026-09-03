---
matkul: Forensics
minggu: 3
sks: 2
sumber: 3 Disk and File System Analysis.pptx
tags: [kuliah/forensics, minggu/w03]
status: draft
diproses: 2026-09-03
---

# W03 — Disk and File System Analysis

## Ringkasan
> - **File System Abstraction Model**, dari level rendah ke tinggi: **Disk → Volume → File System → Data Unit → Metadata → File Name.** Hafalkan urutan ini, semua materi disk nyantol ke sini.
> - Media analysis punya tiga langkah: **Identification** (file apa yang aktif dan terhapus), **Extraction** (ambil data + metadata), **Analysis** (beri makna).
> - Dua skema partisi: **MBR** (maks 4 partisi primer, 2 TB) dan **GPT** (maks 128 partisi primer, 8 ZB).
> - **Hashing** (MD5 128-bit, SHA1 160-bit) dipakai untuk **verifikasi integritas barang bukti** — ubah 1 bit, hash berubah total.
> - Empat kategori "deleted data" dengan tingkat pemulihan berbeda: **Deleted → Orphaned → Unallocated → Overwritten**, dari yang paling bisa dipulihkan ke yang paling tidak.
> - **File slack**: file 1 byte tetap makan 1 block penuh, dan sisa ruangnya **masih berisi data pemilik lama**. Di situlah bukti sering ketemu.
> - Tool imaging: **`dd`** (dasar), **`dcfldd`** (fork, tambah hashing/logging/split), **`dc3dd`** (patch atas GNU dd, ikut update mainline lebih cepat).

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Media analysis | Identifikasi, ekstraksi, dan analisis file beserta filesystem tempat file itu berada |
| Disk | Perangkat penyimpanan fisik (SATA, SCSI, SD card) |
| Volume | Sekumpulan sector pada satu atau beberapa disk; "partition" terbatas pada satu disk fisik |
| Data unit | Unit penyimpanan terkecil yang berdiri sendiri; di Unix disebut **block** |
| Metadata | Data tentang data unit; di Unix disebut **inode** |
| Superblock | Contoh metadata khusus filesystem (Ext2) |
| MBR | Master Boot Record; maks **4 partisi primer**, disk sampai **2 TB** |
| GPT | GUID Partition Table; maks **128 partisi primer**, disk sampai **8 ZB** |
| RAID | Redundant Array of Inexpensive Disks; beberapa disk fisik dialamati sebagai satu unit logis |
| Hashing | Fungsi kriptografis yang mengubah data sembarang panjang jadi string berukuran tetap |
| MD5 / SHA1 | Menghasilkan hash **128-bit** / **160-bit** |
| Carving | Mengekstrak file dari aliran data tak terstruktur berdasarkan header dan magic value |
| foremost | Program file carving; memakai header, footer, dan pengetahuan struktur internal file |
| Forensic imaging | Membuat representasi seakurat mungkin dari media sumber |
| File slack | Sisa ruang dalam block yang masih berisi data dari alokasi sebelumnya |
| `dd` / `dcfldd` / `dc3dd` | Tiga tool imaging berbasis dd |

## Isi

### Konsep media analysis
Pada tingkat paling dasar, analisis forensik berurusan dengan **file di dalam media** — file terhapus, file di dalam folder, file di dalam file lain, semuanya tersimpan di atau di dalam suatu wadah. Tujuan media analysis adalah **mengidentifikasi, mengekstrak, dan menganalisis file-file itu beserta filesystem tempat mereka berada**.

Tiga langkahnya:

| Langkah | Isinya |
| --- | --- |
| **Identification** | Menentukan file aktif dan file terhapus mana yang tersedia di dalam suatu volume |
| **Extraction** | Pengambilan **data file dan metadata** yang relevan |
| **Analysis** | Proses menerapkan kecerdasan kita pada himpunan data itu, idealnya menghasilkan hasil yang bermakna |

### File System Abstraction Model
Ini kerangka paling penting di deck ini. Progresi logis dari file system mana pun, **dari level rendah ke tinggi**:

```
Disk  →  Volume  →  File System  →  Data Unit  →  Metadata  →  File Name
```

**Disk.** Merujuk pada perangkat penyimpanan fisik — hard drive SCSI atau SATA, atau Secure Digital Card dari kamera digital. Analisis di level ini **biasanya di luar kemampuan sebagian besar examiner**: analisis media fisik pada hard drive konvensional butuh pelatihan dan pengetahuan khusus yang ekstensif, akses ke **clean room**, dan peralatan **electron microscopy** yang mahal. Tapi dengan naiknya flash media dan SSD, analisis di level ini **mungkin jadi terjangkau bagi lebih banyak examiner**.

**Volume.** Dibuat memakai seluruh atau sebagian dari satu atau beberapa disk. Satu disk bisa berisi beberapa volume, atau satu volume bisa membentang di beberapa disk, tergantung konfigurasi. Istilah **"partition"** sering dipakai bergantian dengan volume; **Carrier membuat pembedaan**: sebuah *partition* **terbatas pada satu disk fisik**, sedangkan *volume* adalah **kumpulan dari satu atau lebih partition**. Sederhananya, volume menggambarkan sejumlah sector pada disk dalam suatu sistem.

**File System.** Diletakkan di atas sebuah volume, dan menggambarkan tata letak file beserta metadata terkaitnya. Item di lapisan file system mencakup metadata yang khusus dan hanya dipakai untuk operasi file system itu sendiri — **Ext2 superblock** contoh yang bagus.

**Data Unit.** Unit penyimpanan data terkecil yang berdiri sendiri yang tersedia di suatu filesystem. Di filesystem turunan Unix disebut **blocks**. Umumnya berupa kelipatan pangkat 2 dari ukuran sector fisik disk. **Secara historis ukuran sector tiap disk adalah 512 byte** — sebagian besar filesystem modern memakai **4096 byte (4K) atau lebih** sebagai data unit terkecil yang bisa dialamati.

Informasi yang tersedia di lapisan data unit itu sederhana: **isi dari data unit itu**. Kalau data unit dialokasikan ke gambar JPEG, isinya sebagian data JPEG. Kalau dialokasikan ke file teks, isinya teks.

**Metadata.** Merujuk pada **data tentang data**. Kalau lapisan data unit menyimpan data, maka lapisan metadata berisi **data tentang data unit tersebut**. Di filesystem turunan Unix, unit metadata ini disebut **inodes**. Isi persisnya tergantung filesystem, tapi umumnya lapisan ini setidaknya berisi:
- **File time stamps**
- **File ownership information**
- **Data unit mana saja** yang dialokasikan ke unit metadata ini

**File Name.** Lapisan tempat **manusia beroperasi**. Terdiri dari nama file dan folder/direktori. Artifact yang tersedia di lapisan ini bervariasi tergantung filesystem, tapi minimal, **nama file punya pointer ke struktur metadata yang berkorespondensi dengannya**.

> [!info] Konteks tambahan (bukan dari slide)
> Model ini bukan sekadar hafalan — dia **menjelaskan kenapa file bisa "hilang" dengan cara berbeda-beda**. Bayangkan perpustakaan:
> - **Data unit** = buku fisik di rak
> - **Metadata** = kartu katalog (siapa penulisnya, kapan masuk, di rak mana)
> - **File name** = judul di kartu katalog yang kamu cari
>
> Menghapus file biasanya **cuma membuang kartu katalognya**, bukan bukunya. Bukunya masih di rak sampai ada buku lain yang menimpanya. Empat kategori "deleted data" di bawah nanti persis menggambarkan **lapisan mana yang sudah putus**.

### Partisi dan tata letak disk
Dua skema partisi utama yang dipakai hari ini:

| | **MBR** (Master Boot Record) | **GPT** (GUID Partition Table) |
| --- | --- | --- |
| Partisi primer | Awalnya hanya **4** | **128** |
| Ukuran disk maks | **2 Terabyte** | **8 Zettabyte** |
| Status | Skema lama | Dikembangkan sebagai **pengganti MBR** |

Slide mencatat sesuatu yang penting untuk mengatur ekspektasi: **partition table kemungkinan besar tidak berisi informasi yang relevan untuk sebagian besar investigasi.** Analisis forensik atas partition table biasanya **terbatas pada pemulihan volume ketika struktur partisinya hilang atau rusak**.

### RAID
**RAID (Redundant Array of Inexpensive Disks)** dirancang sebagai cara untuk mengambil beberapa disk fisik dan mengalamatinya sebagai **satu unit logis**.

| Level | Minimum disk | Cara kerja | Konsekuensi |
| --- | --- | --- | --- |
| **RAID 0** | 2 | Di-**stripe** di level block: block A ke disk 0, block B ke disk 1, dan seterusnya | **Menaikkan kecepatan tulis**, tidak mengorbankan ruang penyimpanan, tapi **menaikkan kerapuhan data** — kehilangan satu drive berarti kehilangan separuh block |
| **RAID 1** | 2 | Kebalikan RAID 0 — block **di-mirror** antar pasangan drive | **Menaikkan kecepatan baca dan keandalan**, tapi **memotong ruang tersedia jadi separuh** dari ruang fisik |
| **RAID 5** | 3 | Striping di beberapa disk **plus membuat parity block**, yang juga di-stripe antar disk | Parity block dipakai **membuat ulang data** kalau ada drive yang hilang |

Selain itu ada setup **"nested" atau "hybrid"** yang menggabungkan dua level RAID secara berurutan. Contoh: **RAID 50** (atau 5+0) adalah sepasang set RAID 5 yang kemudian di-stripe.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa RAID penting untuk forensik padahal slide tidak menjelaskannya: kalau kamu menyita server ber-RAID 0 lalu meng-image satu disk saja, **yang kamu dapat adalah separuh block acak dari tiap file** — tidak ada file yang utuh. Untuk RAID, kamu harus meng-image **semua disk** dan merekonstruksi array-nya. Ini kelemahan materi yang layak ditanyakan (lihat Pertanyaan Terbuka).

### Hashing
Salah satu aktivitas kunci yang dilakukan di **banyak titik berbeda sepanjang pemeriksaan** adalah menghasilkan **cryptographic hash**.

Fungsi hash kriptografis mengambil **data dalam jumlah sembarang** sebagai input dan mengembalikan **string berukuran tetap** sebagai output. Nilai hasilnya disebut **hash of data**.

| Algoritma | Panjang hash |
| --- | --- |
| **MD5** | **128 bit** |
| **SHA1** | **160 bit** |
| SHA256, SHA512 | Sesuai angka di namanya |

Sifat kunci untuk forensik: **modifikasi satu bit saja pada data input akan menghasilkan nilai hash yang sama sekali berbeda.** Dari sifat ini, kegunaan inti hashing dalam analisis forensik jadi mudah ditentukan: **verifikasi integritas barang bukti digital**.

> [!info] Konteks tambahan (bukan dari slide)
> Cara pakainya di praktik: kamu hash disk asli **sebelum** imaging, lalu hash image hasilnya. **Kalau dua hash itu identik, kamu punya bukti matematis bahwa salinanmu persis sama dengan aslinya** — dan bahwa kamu tidak mengubah apa pun. Itulah cara membuktikan integritas di pengadilan tanpa harus meminta hakim percaya begitu saja. Sifat "satu bit berubah, hash berubah total" ini sama dengan **avalanche effect** yang dipelajari di matkul Blockchain.

### Tugas dari slide
> **Slide 18 — Assignment:** Buat program sederhana memakai bahasa apa pun yang kamu suka (**Python disarankan**) untuk menentukan apakah dua file input itu **file yang sama atau berbeda, berdasarkan hash**.

### Carving
Slide mengutip: *"seorang forensic examiner bijak pernah berkata, **'kalau semua cara lain gagal, kita carve.'**"*

Mengekstrak isi file yang bermakna dari **aliran data yang tidak terstruktur** adalah ilmu sekaligus seni tersendiri. Disiplin ini jadi fokus banyak presentasi di **Digital Forensics Research Workshop** selama bertahun-tahun, dan kemajuannya terus berlanjut sampai hari ini.

Pada intinya, proses carving melibatkan:
1. **Mencari file header dan magic value** di dalam aliran data
2. **Menentukan (atau menebak) titik akhir file**-nya
3. **Menyimpan sub-aliran itu** keluar sebagai file hasil carving

**foremost** adalah program file carving yang awalnya ditulis **Jesse Kornblum dan Kris Kendall** di Air Force Office of Special Investigations, lalu diperbarui **Nick Mikus** dari Naval Postgraduate School. Dia memakai **header, footer, dan pengetahuan tentang struktur internal** jenis file yang didukung untuk membantu carving. Daftar lengkap jenis file yang didukung natif ada di man page-nya, tapi mencakup yang biasa: **gambar JPEG, dokumen office, file arsip**, dan lainnya.

### Forensic imaging
Dalam membuat forensic image, kita berusaha menangkap **representasi seakurat mungkin dari media sumber**. Slide memakai analogi yang bagus:

> Ini tidak berbeda dengan **garis polisi** yang dipasang di TKP fisik. Garis itu dipasang untuk **meminimalkan perubahan** yang terjadi di TKP, yang pada gilirannya memberi penyidik data seakurat mungkin.
>
> Sekarang bayangkan kalau penyidik bisa **membuat salinan dari TKP-nya sendiri**. Di dunia nyata itu kegilaan — tapi **itulah yang kita lakukan dengan forensic image**.

### Empat kategori deleted data
Alasan lain examiner memakai forensic imaging adalah **kelengkapan**. Sekadar memeriksa filesystem aktif seperti yang ditampilkan sistem operasi **tidak cukup menyeluruh** untuk pemeriksaan forensik. Sebagian besar volume berisi **banyak sekali data yang berpotensi menarik di luar file yang terlihat dan teralokasi** di filesystem yang ter-mount.

Empat kategorinya, diurutkan dari yang paling bisa dipulihkan:

| Kategori | Kondisinya | Bisa dipulihkan? |
| --- | --- | --- |
| **Deleted** | File "di-unlink" — nama file tidak lagi tampil saat user melihat direktori, dan nama file, struktur metadata, serta data unit ditandai "free". **Tapi koneksi antar lapisan itu masih utuh** saat teknik forensik diterapkan | **Paling bisa dipulihkan.** Cukup catat nama file dan struktur metadata, lalu ekstrak data unit-nya |
| **Orphaned** | Mirip deleted, kecuali **link antara nama file dan struktur metadata sudah tidak akurat** | Pemulihan data (dan struktur metadata) **masih mungkin**, tapi **tidak ada korelasi langsung** dari nama file ke data yang dipulihkan |
| **Unallocated** | Entri nama file dan struktur metadata terkaitnya sudah **ter-unlink dan/atau dipakai ulang** | Satu-satunya cara pemulihan adalah **carving** data unit yang belum dipakai ulang dari unallocated space |
| **Overwritten** | Satu atau lebih data unit-nya sudah **dialokasikan ulang ke file lain** | **Pemulihan penuh tidak mungkin lagi.** Pemulihan sebagian mungkin, tergantung seberapa jauh penimpaannya |

File yang nama dan/atau struktur metadatanya masih utuh tapi sebagian atau seluruh data unit-nya sudah ditimpa kadang disebut **Deleted/Overwritten** atau **Deleted/Reallocated**.

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan bahwa keempat kategori ini **persis mengikuti File System Abstraction Model** dari atas: *deleted* = ketiga lapisan masih terhubung; *orphaned* = link nama↔metadata putus; *unallocated* = metadata sudah dipakai ulang, tinggal data unit; *overwritten* = data unit-nya sendiri sudah hilang. Kalau kamu paham modelnya, keempat kategori ini tidak perlu dihafal — bisa diturunkan sendiri.

### File slack
Seperti disebut sebelumnya, ruang minimum yang bisa dialokasikan di sebuah volume adalah **satu block**. Dengan asumsi block size 4K, pada drive standar dengan sector 512 byte, ini berarti **file teks ASCII berisi satu byte — huruf 'a' — akan memakan delapan sector di disk**.

Kita cuma menyediakan 'a'. Dari mana 4095 byte lainnya yang tertulis ke disk?

Jawabannya, seperti biasa, **tergantung**. Filesystem dan sistem operasi berbeda menanganinya berbeda, tapi umumnya prosesnya:
- **Cluster yang akan dipakai ditandai "allocated"** dan ditugaskan ke struktur metadata file itu.
- **Huruf 'a' diikuti 511 null byte (hex 00) ditempatkan di sector pertama.**

Lalu bagaimana tujuh sector berikutnya ditulis ke disk? Slide menjawab: **itu bukan kelalaian — sector-sector itu memang tidak ditulis.** Mereka **mempertahankan data apa pun yang terakhir tersimpan di sana selama alokasi sebelumnya**. Inilah yang dikenal sebagai **file slack** atau **slack space**.

Ilustrasi yang dipakai slide (tiga tampilan berurutan atas delapan block yang sama):
1. Awalnya barisan itu terdiri dari block baru, kosong, unallocated.
2. **File A dibuat**, mendapat alokasi delapan block, dan kedelapan block itu diisi data.
3. **File A "dihapus"**, dan beberapa waktu kemudian **lima block pertama dialokasikan ulang dan ditimpa** dengan isi File B.
4. Hasilnya: **tiga block masih berisi data File A**, dalam status unallocated tapi **masih bisa dipulihkan**.

> [!info] Konteks tambahan (bukan dari slide)
> Ini salah satu tempat paling produktif untuk mencari bukti, justru karena **user tidak tahu keberadaannya dan tidak bisa membersihkannya lewat cara biasa**. Menghapus file, mengosongkan Recycle Bin, bahkan memformat quick — semuanya tidak menyentuh slack space. Perhatikan juga sambungannya ke [[W02 - Key Technical Concepts]]: konsep "sector 512 byte" yang di sana terasa cuma trivia teknis, di sini jadi **alasan bukti bisa bertahan**.

### Tool imaging: dd dan turunannya

| Tool | Asal | Yang membedakan |
| --- | --- | --- |
| **`dd`** | Tool open source paling dasar untuk membuat forensic image | Hampir selalu ada di sistem operasi mirip-Unix mana pun, dan jadi **basis beberapa utilitas imaging forensik lain**. Sederhananya, `dd` menyalin data dari satu tempat ke tempat lain; user bisa memberi berbagai argumen dan flag untuk memodifikasi perilaku sederhana itu |
| **`dcfldd`** | Dibuat untuk **Defense Computer Forensics Laboratory** oleh **Nick Harbour**; **fork dari GNU dd** | Kemampuan tambahannya berkisar pada **pembuatan dan validasi hash**, **logging aktivitas**, dan **memecah file output jadi potongan berukuran tetap** |
| **`dc3dd`** | Dibuat **Jesse Kornblum** untuk **Department of Defense Cyber Crime Center**; dikembangkan sebagai **patch atas GNU dd**, bukan fork | Karena berupa patch, dc3dd bisa **mengadopsi perubahan dari dd mainline lebih cepat** daripada dcfldd. Punya semua fitur tambahan dcfldd, **plus fitur inti dd yang saat ini belum ada di rilis dcfldd terbaru** |

### Digital forensic tools
Slide menyarankan **minimal 4 GB RAM** supaya tool forensik berjalan lancar, lalu menampilkan daftar tool forensik yang tersedia di **Kali Linux** (dalam bentuk gambar).

## Diagram & Visual
- **Slide 33 — daftar tool forensik yang tersedia di Kali Linux**
  ![[99-Assets/Forensics/W03-slide33.png]]

> [!warning] Dua gambar penting **tidak berhasil diikutkan**:
> - **Slide 8** — diagram pembedaan disk vs volume (format WMF, tidak didukung Obsidian)
> - **Slide 29** — ilustrasi tiga tahap terbentuknya **file slack** (gambar tidak lolos ekstraksi)
>
> Slide 29 khususnya sayang, karena itu visualisasi dari penjelasan file slack di atas. **Buka PPT aslinya di slide 8 dan 29.**

## Rumus / Sintaks

Model abstraksi filesystem (hafalkan urutannya):
```
Disk → Volume → File System → Data Unit → Metadata → File Name
 fisik  sector    tata letak    isi data   inode      nama
```

Angka-angka yang perlu diingat:
```
sector historis  : 512 byte
data unit modern : 4096 byte (4K) atau lebih
MBR              : 4 partisi primer,   maks 2 TB
GPT              : 128 partisi primer, maks 8 ZB
MD5              : 128 bit
SHA1             : 160 bit
```

Aritmetika file slack dari slide:
```
file 1 byte, block size 4K, sector 512 byte
-> memakan 8 sector (4096 / 512)
-> sector 1 : 'a' + 511 null byte (0x00)
-> sector 2-8 : TIDAK ditulis, isinya data pemilik lama  <- file slack
```

## Pertanyaan Terbuka
- Slide 3 mencantumkan **"SSD vs HDD Forensics"** dan **"Logical vs Physical Imaging"** serta **"Common Disk Errors and Their Impact on Forensic Analysis"** sebagai learning outcome, tapi **ketiganya tidak pernah dibahas** di deck ini. SSD vs HDD forensics khususnya penting, karena nyambung ke ancaman TRIM di [[W01 - Digital Forensic Fundamental]]. **Wajib ditanyakan.**
- **RAID dijelaskan mekanismenya tapi tidak implikasi forensiknya** — apa yang terjadi kalau kamu meng-image satu disk saja dari array RAID 0? Ini pertanyaan ujian yang sangat mungkin muncul.
- **MD5 dan SHA1 keduanya sudah dianggap tidak aman secara kriptografis** (ada collision attack praktis untuk keduanya). Slide tidak menyebut ini. Perlu ditanya apakah untuk verifikasi integritas forensik keduanya masih diterima, dan apakah SHA256 sudah jadi standar di praktik.
- Slide 27 menulis file 1 byte "akan memakan delapan sector"; itu benar untuk block 4K dengan sector 512 byte. Tapi kalimat berikutnya menanyakan "dari mana **4095 byte** lainnya" — angka 4095 mengacu ke sisa block 4K, sementara penjelasannya lalu bicara soal 511 null byte di sector pertama. **Dua angka ini di skala berbeda** (block vs sector); pastikan paham keduanya, jangan hafal salah satu.
- Tugas hashing di slide 18 tidak menyebut algoritma yang harus dipakai maupun format output yang diminta.
- Nama **Carrier** dirujuk untuk pembedaan partition vs volume, tapi tidak ada di daftar referensi slide. (Maksudnya kemungkinan besar Brian Carrier, penulis *File System Forensic Analysis*.)

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W02 - Key Technical Concepts]]
- [[W04 - Mac OS X Systems and Artifacts]]
- [[W06 - Data Acquisition]]
- [[W09 - File Recovery and Data Carving]]
- [[Forensics - Review dan Glosari]]
