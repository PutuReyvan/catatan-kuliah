---
matkul: Forensics
minggu: 4
sks: 2
sumber: 4 Mac OS X systems and artifacts.pptx
tags: [kuliah/forensics, minggu/w04]
status: draft
diproses: 2026-09-03
---

# W04 — Mac OS X System and Artifacts

## Ringkasan
> - Filesystem OS X adalah **HFS+ (Mac OS Extended)**, penerus HFS. Dua varian: **HFSJ** (journaling) dan **HFSX** (nama file case-sensitive).
> - **Volume header** selalu ada **1024 byte dari awal volume**, dengan salinan cadangan **1024 byte sebelum akhir volume**. Data unit-nya disebut **allocation block** (umumnya 4K), dan data file dialamati lewat **extent**.
> - HFS+ punya **lima special file** tersembunyi: **allocation, catalog, extents overflow, startup, attributes**. Catalog file itu padanan **MFT di NTFS**.
> - Artifact OS X banyak yang berbentuk **property list (.plist)** — ada versi **plain-text XML** dan **binary** (binary harus dikonversi dulu).
> - **`launchd`** menggantikan `init`; dia membaca tugas dari **4 direktori** LaunchDaemons/LaunchAgents.
> - Artifact user paling banyak ada di **home directory**, terutama folder **Library**. Yang penting: **`.Trash`** (sejak OS X 10.6 menyimpan path asli di `.DS_Store`) dan **`.bash_history`**.
> - **Swap dan hibernation** ada di `/private/var/vm` — `sleepimage` seukuran RAM, isinya **salinan memori** saat terakhir sleep.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| HFS+ / Mac OS Extended | Filesystem OS X, penerus HFS |
| HFSJ / HFSX | Varian HFS+ untuk journaling / nama file case-sensitive |
| Volume header | Struktur inti HFS+; berisi ukuran allocation block, timestamp pembuatan volume, dan lokasi special file |
| Allocation block | Data unit HFS+; ukurannya didefinisikan di volume header, umumnya 4K |
| Clump | Kumpulan allocation block, mirip block allocation group di filesystem Ext |
| Extent | Pointer 4-byte ke allocation block awal + nilai 4-byte panjang extent |
| Allocation file | Bitmap yang melacak status alokasi tiap block volume |
| Catalog file | Record untuk tiap file dan direktori; padanan **MFT** di NTFS |
| CNID | Catalog Node ID, identitas file/folder di catalog record |
| Extents overflow file | Record untuk fork dengan **lebih dari 8 extent**; menandakan fragmentasi parah |
| Startup file | Informasi untuk boot dari sistem yang tidak mengenal HFS+ |
| Attributes file | Menyimpan extended attribute; dipakai kompresi per-file di OS X 10.6 |
| Property list (.plist) | Format penyimpanan artifact OS X; plain-text XML atau binary |
| `launchd` | Pengganti `init`; menjalankan tugas startup dari plist |
| Kext | Kernel extension, di `/System/Library/Extensions` |
| Dotfile | File/direktori berawalan titik, disembunyikan dari user secara default |
| `sleepimage` | File hibernasi seukuran RAM, berisi salinan memori saat terakhir sleep |
| `.DS_Store` | File tersembunyi; di `.Trash` menyimpan **path asli** file yang dihapus |

## Isi

### Filesystem OS X
Filesystem yang dipakai OS X disebut **HFS Plus** atau **Mac OS Extended**. HFS+ adalah penerus **Hierarchical File System (HFS)** yang dipakai sistem operasi Mac sebelum OS X.

Saat ini ada **dua varian format HFS+**:
- **HFSJ** — mendukung **journaling**
- **HFSX** — mendukung **nama file case-sensitive**

Slide mencatat bahwa di luar kemampuan tambahan itu, **varian-varian ini tidak mengubah fungsi atau artifact yang tersedia bagi examiner**.

### Struktur HFS+
**Volume header** adalah salah satu struktur inti volume HFS+. Dia menyimpan data tentang filesystem, termasuk:
- **Ukuran allocation block**
- **Timestamp pembuatan volume**
- **Lokasi special file** yang dibutuhkan untuk operasi HFS+

Letaknya presisi dan patut dihafal: **volume header selalu berada 1024 byte dari awal volume**, dengan **salinan cadangan 1024 byte sebelum akhir volume**.

HFS+ memakai **allocation block** sebagai data unit. Ukuran satu allocation block didefinisikan di volume header, tapi **4K byte adalah nilai yang umum**. Allocation block bisa dikelompokkan lagi jadi **clumps**, yang agak mirip dengan **block allocation group** di filesystem Ext pada Linux.

Data sebuah file dialamati dalam bentuk **extents**. Satu extent HFS+ hanyalah **pointer 4-byte ke allocation block awal** ditambah **nilai 4-byte yang menunjukkan panjang extent** tersebut.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa ada salinan cadangan volume header di ujung volume: kalau awal disk rusak secara fisik atau ditimpa, filesystem masih bisa dipulihkan dari salinan itu. Buat examiner, ini artinya **kalau volume header utama hilang, jangan langsung menyerah** — cari 1024 byte sebelum akhir volume. Ini contoh konkret dari "pemulihan volume ketika struktur partisi rusak" yang disebut di [[W03 - Disk and File System Analysis]].

### Lima special file HFS+
Sebagian besar struktur yang dibutuhkan HFS+ untuk berfungsi disimpan di dalam volume sebagai **hidden file** — mirip **MFT dan file terkaitnya di volume NTFS**. Volume HFS+ punya **lima file semacam itu**, yang **tidak bisa diakses langsung** dengan utilitas filesystem standar:

| # | Special file | Fungsinya |
| --- | --- | --- |
| 1 | **Allocation file** | **Bitmap** yang melacak status alokasi tiap block di volume |
| 2 | **Catalog file** | Berisi **record untuk tiap file dan direktori** di volume. Menjalankan banyak fungsi yang sama dengan **Master File Table di NTFS**. Lokasi extent pertamanya **wajib disimpan di volume header**; lokasi semua file lain disimpan di catalog record. **Panjang record 8K**, berisi **CNID** file/folder, **parent CNID**, **metadata timestamp**, dan informasi tentang **data fork dan resource fork** |
| 3 | **Extents overflow file** | Record untuk fork yang punya **lebih dari delapan extent**. File ini seharusnya **cukup jarang terisi** — punya lebih dari 8 extent menandakan **fragmentasi yang cukup parah** |
| 4 | **Startup file** | Menyimpan informasi yang dipakai saat **boot dari sistem yang tidak mengenal HFS+** |
| 5 | **Attributes file** | Menyimpan **extended attribute** untuk file. Dipakai untuk **kompresi per-file** di OS X 10.6 |

> [!info] Konteks tambahan (bukan dari slide)
> Kalau kamu ingat **File System Abstraction Model** dari [[W03 - Disk and File System Analysis]], kelima file ini memetakan rapi ke sana: **allocation file** = lapisan data unit (block mana yang terpakai), **catalog file** = lapisan metadata **dan** file name sekaligus. Dan karena catalog file memegang keduanya, **merusak catalog file di HFS+ langsung memutus nama↔metadata** — persis definisi file *orphaned*.

### Artifact sistem OS X
Seperti sistem Linux, OS X menempatkan semua volume di bawah **satu namespace terpadu** di bawah root directory **`/`**. Persis di bawah root:

| Direktori | Isinya |
| --- | --- |
| **`Applications`** | Lokasi standar semua aplikasi OS X yang terinstal. Umumnya aplikasi yang dijalankan interaktif lewat GUI |
| **`Library`** | Data pendukung yang mungkin perlu diubah saat program berjalan: **preferences, recent items**, dan sejenisnya. **Library di root** = konfigurasi seluruh sistem; **Library di direktori user** = data spesifik user |
| **`Network`** | Item di Network domain; **umumnya kosong** |
| **`System`** | Data spesifik sistem operasi, agak analog dengan isi **`system32` di Windows** |
| **`Users`** | Direktori induk untuk home directory tiap user |
| **`Volumes`** | Direktori induk untuk volume yang di-mount; mirip **`/mnt` atau `/media` di Linux** |
| **`bin` dan `sbin`** | Utilitas command-line bawaan sistem OS X |
| **`private`** | Berisi (antara lain) versi OS X dari **`/var`, `/tmp`, dan `/etc`** |

### Property list (.plist)
Banyak artifact menarik di sistem OS X disimpan sebagai **property list** atau file **`.plist`**. Ada **dua jenis**:

| Jenis | Cara menanganinya |
| --- | --- |
| **Plain text (XML)** | Bisa diperiksa langsung atau dilihat di program penampil XML mana pun |
| **Binary** | **Harus dikonversi ke plain text sebelum dianalisis** |

Karena lebih ringkas dari padanan plain text-nya, **binary plist makin sering dipakai**. Instalasi OS X standar akan punya **ribuan file plist**, jadi **mengetahui mana yang relevan untuk pemeriksaanmu adalah kuncinya**.

### Startup dan services
Saat sistem boot, bootloader mem-boot kernel OS X (**`/mach_kernel`**), yang kemudian menjalankan proses **`launchd`**. `launchd` berfungsi sebagai **pengganti `init` dan proses init script** yang ada di sistem Linux.

`launchd` mengambil tugasnya dari **empat direktori**:

| Jenis tugas | Direktori |
| --- | --- |
| **System task yang jalan di background** | `/System/Library/LaunchDaemons` dan `/Library/LaunchDaemons` |
| **Launch task yang interaktif dengan user** | `/System/Library/LaunchAgents` dan `/Library/LaunchAgents` |

`launchd` akan membaca dan memproses plist di direktori-direktori ini, lalu menjalankan aplikasi yang sesuai. Catatan penting untuk examiner: **semua file plist di direktori ini seharusnya dalam format plain XML**, jadi **tidak perlu dikonversi sebelum diperiksa**.

> [!info] Konteks tambahan (bukan dari slide)
> Empat direktori ini adalah **tempat pertama yang diperiksa saat mencari persistence malware di Mac** — sama seperti Run key di registry Windows atau cron di Linux. Kalau ada program yang mau jalan otomatis setiap boot, dia harus menaruh plist di salah satu dari empat lokasi ini. Bandingkan dengan artifact autostart di [[W07 - Windows System Artifacts]] dan [[W06 - Linux System and Artifacts]].

### Kernel extension (Kext)
OS X bisa memuat fungsionalitas tambahan ke dalam kernel lewat **kernel extension**. Kext adalah **bundle dengan extension `.kext`** dan ada di direktori **`/System/Library/Extensions`**.

Di direktori itu ada banyak kernel extension bawaan Apple, dan **mungkin ada extension untuk perangkat keras atau program pihak ketiga yang butuh akses level rendah** — misalnya **software enkripsi disk**.

### Konfigurasi jaringan
Sebagian besar informasi konfigurasi jaringan sistem lokal di OS X disimpan di berbagai file plist di bawah **`/Library/Preferences/SystemConfiguration`**.

File **`preferences.plist`** berisi:
- Pengaturan umum untuk **semua network interface** di sistem
- Informasi **profil jaringan spesifik lokasi**, kalau fitur itu dipakai
- **Hostname komputer** — yang bisa penting dalam pemeriksaan terkait jaringan

### Hidden directory
Selain tipe bundle directory khusus yang menyembunyikan isinya dari user secara default, OS X juga menghormati penyembunyian **"dotfile" gaya Unix tradisional**: file dan direktori yang namanya diawali titik **akan disembunyikan dari pandangan user secara default**.

Slide mencatat: **jumlah hidden file di OS X tidak sebanyak di instalasi Linux standar**, tapi tetap ada beberapa — termasuk beberapa contoh signifikan (`.Trash` dan `.bash_history` di bawah).

### Aplikasi terinstal
**`/Library/Receipts`** berisi informasi tentang aplikasi yang dipasang lewat sistem OS X Installer. Direktori ini berisi berbagai **bundle "pkg"**, yang memuat informasi tentang paket terinstal.

Yang berguna secara forensik: **waktu pembuatan (creation time) direktori-direktori ini seharusnya sesuai dengan tanggal software itu dipasang.**

### Swap dan hibernation
OS X menyimpan swap file dan data hibernasi di **`/private/var/vm`**.

- Tergantung seberapa berat penggunaan (dan seberapa kekurangan sumber daya) sistemnya, akan ada **1 sampai 10 item swapfile bernomor** di direktori ini. Isinya **potongan memori yang di-page out**, dan bisa **bertahan di disk untuk waktu yang cukup lama**.
- Kalau hibernasi diaktifkan, akan ada file **`sleepimage`**. File ini **berukuran sama dengan RAM** yang tersedia di sistem, dan berisi **salinan memori sebagaimana adanya saat terakhir kali sistem di-sleep**.

Teknik apa pun untuk memproses aliran data tak terstruktur **berlaku untuk file-file ini** — termasuk **string extraction** dan **file carving**.

> [!info] Konteks tambahan (bukan dari slide)
> Ini penting sekali dan gampang terlewat: **`sleepimage` adalah RAM dump yang tersimpan permanen di disk.** Semua yang bikin RAM berharga secara forensik — password plaintext, kunci enkripsi, proses yang berjalan (lihat [[W05 - Data Acquisition]]) — bisa jadi ada di file ini, dan **file ini non-volatile**. Artinya kamu bisa mendapat isi memori dari mesin yang sudah mati. Ini salah satu jalan memutar paling berguna terhadap masalah volatilitas di [[W02 - Key Technical Concepts]].

### Log sistem
OS X berbagi banyak log dengan sistem mirip-Unix lain seperti Linux. Umumnya, aplikasi turunan BSD dan Linux untuk OS X akan menyimpan log di bawah **`/private/var/log`**.

OS X umumnya menjalankan **syslog daemon** dan menghasilkan banyak log yang sama (termasuk log turunan syslog) seperti sistem Linux standar.

### Artifact user
Tiap user di sistem punya **plist di bawah `/private/var/db/dslocal/nodes/Default/users/`** yang sesuai dengan **short username**-nya. Isinya informasi user dasar, mirip entri `/etc/passwd` di Linux, termasuk:
- **Path ke shell default** user
- **Nama panjang** yang ditampilkan
- **UID** user

Informasi grup disimpan di **`/private/var/db/dslocal/nodes/Default/groups/`** dengan format serupa. Yang **sangat penting** adalah file **`admin.plist`** di direktori groups: memeriksa file ini membantu menentukan **apakah seorang user punya hak administrator atau "root"** di sistem itu. Di cuplikan yang ditampilkan slide, "root" dan "user1" keduanya Administrator.

### Home directory
Sistem OS X umumnya **cukup rapi**. Karena itu, **home directory user adalah tempat sebagian besar artifact yang dihasilkan user — langsung maupun tidak langsung — akan ditemukan**.

Meskipun aktivitas user tertentu akan menghasilkan artifact di area sistem (misalnya login dan logout), **hampir semua aktivitas user pasca-autentikasi akan terbatas pada item dan data residual di home directory-nya sendiri**.

Dari semua direktori standar di home directory, **yang berisi artifact spesifik-OS X paling banyak adalah `Library`**.

### `.Trash`
Tiap direktori user seharusnya berisi direktori tersembunyi bernama **`.Trash`**. Lokasi ini dipakai sebagai **penyimpanan file sementara** saat file "dihapus" memakai aplikasi Finder.

Yang berubah antar versi, dan ini detail yang bagus untuk soal ujian:

| Versi | Yang bisa diperiksa |
| --- | --- |
| **OS X 10.5 dan sebelumnya** | **Hanya bisa melihat isinya.** Versi ini **tidak melacak lokasi asli** file, dan **tidak menyimpan metadata lain** apa pun tentang file itu |
| **OS X 10.6 dan sesudahnya** | **Menyimpan path asli** file yang dihapus di dalam file tersembunyi **`.DS_Store`** di dalam direktori `.Trash`. Path aslinya bisa ditentukan dengan mudah lewat **hex editor** |

### Shell history
OS X memakai **BASH (Bourne Again Shell)** secara default, jadi kalau user pernah memakai terminal, **shell history-nya akan ada di file `.bash_history`** di home directory.

Slide menambahkan pengamatan yang menarik secara investigatif:

> Karena **sebagian besar user OS X tidak pernah memakai Terminal**, **adanya entri di file ini bisa jadi tanda bahwa user itu "power user"** atau setidaknya cukup melek teknologi.

Seperti di Linux, **file ini tidak menyimpan nilai waktu** untuk tiap entri — tapi lewat pemeriksaan sistem yang cermat, **waktu perkiraan** untuk aktivitas yang terdaftar bisa disimpulkan.

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan bentuk penalarannya di sini: keberadaan `.bash_history` yang terisi **bukan bukti kejahatan** — dia bukti **tingkat keahlian**. Dan tingkat keahlian mengubah asumsimu untuk seluruh pemeriksaan: user yang melek teknis lebih mungkin memakai enkripsi, anti-forensik, dan penghapusan yang disengaja. Ini contoh artifact yang nilainya bukan pada isinya, tapi pada **apa yang disiratkan tentang orangnya**.

## Diagram & Visual
- **Slide 11 — Tabel 6.1: pembagian tugas startup `launchd` (menurut dokumentasi Apple)**
  ![[99-Assets/Forensics/W04-slide11.png]]
- **Slide 17 — Tabel 6.2: logfile penting untuk examiner di bawah `/private/var/log`**
  ![[99-Assets/Forensics/W04-slide17.png]]
- **Slide 19 — Tabel 6.3: direktori default di home directory user**
  ![[99-Assets/Forensics/W04-slide19.png]]

> [!warning] Ketiga tabel di atas (6.1, 6.2, 6.3) **berupa gambar, bukan teks** — isinya tidak bisa dibaca dari note ini. Tabel 6.2 khususnya penting: itu daftar logfile yang paling berguna untuk examiner, dan isinya tidak ada di mana pun selain gambar itu. Cuplikan `admin.plist` di slide 18 juga tidak ikut terekstrak.

## Rumus / Sintaks

Lokasi-lokasi penting OS X:
```
/mach_kernel                                    kernel
/System/Library/LaunchDaemons                   startup, background, sistem
/Library/LaunchDaemons                          startup, background
/System/Library/LaunchAgents                    startup, interaktif, sistem
/Library/LaunchAgents                           startup, interaktif
/System/Library/Extensions                      kernel extension (.kext)
/Library/Preferences/SystemConfiguration        konfigurasi jaringan (preferences.plist)
/Library/Receipts                               aplikasi terinstal (bundle pkg)
/private/var/vm                                 swapfile + sleepimage
/private/var/log                                log sistem
/private/var/db/dslocal/nodes/Default/users/    plist per user
/private/var/db/dslocal/nodes/Default/groups/   plist grup (admin.plist!)
~/.Trash                                        recycle bin (+ .DS_Store sejak 10.6)
~/.bash_history                                 riwayat shell
```

Angka yang perlu diingat:
```
volume header      : 1024 byte dari awal volume (+ cadangan 1024 byte sebelum akhir)
allocation block   : umumnya 4K
catalog record     : 8K
extents overflow   : dipakai kalau fork punya > 8 extent
sleepimage         : seukuran RAM sistem
```

## Pertanyaan Terbuka
- Slide 3 mencantumkan **empat learning outcome yang tidak pernah dibahas** di deck ini: **APFS**, **Mac OS X Unified Logging System**, **Time Machine and Snapshots**, dan **FileVault and Encrypted Volumes**. Ini bukan topik kecil — **APFS sudah menggantikan HFS+ sejak macOS High Sierra (2017)**, dan FileVault adalah alasan utama akuisisi Mac jadi sulit. Deck ini seluruhnya membahas HFS+ yang sudah usang. **Wajib ditanyakan mana yang diujikan.**
- **Data fork dan resource fork** disebut sebagai isi catalog record tapi tidak pernah dijelaskan. Konsep fork itu khas Mac dan sering jadi soal.
- Slide 17 merujuk **"Chapter 5"** untuk informasi detail soal syslog dan logging Linux — itu rujukan ke buku sumbernya, dan materinya ada di [[W06 - Linux System and Artifacts]].
- Slide 4 kalimatnya terpotong: *"Beyond these extended capabilities, because these variants don't alter the function or artifacts available to the examiner"* — tidak ada anak kalimatnya. Maksudnya kemungkinan besar "keduanya bisa diperlakukan sama saat pemeriksaan".
- Isi **Tabel 6.2 (logfile penting)** tidak bisa diekstrak. Itu daftar praktis yang paling langsung kepakai kalau ada praktikum Mac.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W03 - Disk and File System Analysis]]
- [[W05 - Data Acquisition]]
- [[W06 - Linux System and Artifacts]]
- [[W07 - Windows System Artifacts]]
- [[Forensics - Review dan Glosari]]
