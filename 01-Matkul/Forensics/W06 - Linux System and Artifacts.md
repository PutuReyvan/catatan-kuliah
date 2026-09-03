---
matkul: Forensics
minggu: 6
sks: 2
sumber: 6 Linux System and Artifacts.pptx
tags: [kuliah/forensics, minggu/w06]
status: draft
diproses: 2026-09-03
---

# W06 — Linux System and Artifacts

## Ringkasan
> - Filesystem Ext punya dua komponen di lapisan filesystem: **superblock** (1024 byte dari awal, sama seperti volume header HFS+) dan **group descriptor tables**.
> - **Inode** menyimpan metadata: ukuran, block yang dialokasikan, ownership (**UID/GID**), permission, link count, dan **empat timestamp MAC(D)**: **M**odified, **A**ccessed, **C**hanged, **D**eleted.
> - **Hard link** = beberapa nama file menunjuk ke inode yang sama; tiap hard link menaikkan link count inode satu.
> - Boot process: **bootloader → kernel (`/boot`) → initrd → inisialisasi hardware → `/sbin/init`**, lalu bercabang jadi gaya **System V** (runlevel, `/etc/inittab`) atau **BSD** (`/etc/rc`).
> - Tiga file autentikasi: **`/etc/passwd`** (7 field), **`/etc/group`**, **`/etc/shadow`** (8 field, berisi hash password).
> - Log aktivitas user ada di tiga file: **`/var/run/utmp`** (login aktif), **`/var/log/wtmp`** (jangka panjang), **`/var/log/lastlog`**. Dibaca dengan `last -f`.
> - **`at`** untuk tugas sekali jalan, **`cron`** untuk tugas berulang — dan **cron adalah cara favorit penyerang mempertahankan persistence**.
> - Praktik Kali: **`fdisk -l`** untuk mengenali device, **`md5sum`** untuk hash, **`dc3dd`** untuk imaging.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Ext2 / Ext3 / Ext4 | Keluarga filesystem Linux; Ext3 = Ext2 + journaling; Ext4 = pengganti modern |
| Superblock | Struktur data 1024 byte dari awal filesystem Ext; berisi layout, alokasi block dan inode |
| Group descriptor table | Komponen kedua lapisan filesystem Ext |
| Directory entry | Cara nama file disimpan di Ext: nama + alamat inode + flag direktori/file |
| Hard link | Directory entry tambahan yang menunjuk ke inode yang sama |
| Link count | Jumlah nama file yang merujuk ke satu inode |
| Inode | Struktur metadata Ext |
| UID / GID | User Identifier / Group Identifier |
| MAC times | Modified, Accessed, Changed — plus **Deleted** di Ext |
| Block | Data unit Ext; ukuran 1K, 2K, atau 4K sesuai superblock |
| initrd | Initial ramdisk; berisi driver dan modul yang dibutuhkan saat boot |
| Runlevel | Deskripsi numerik untuk sekumpulan script yang dijalankan pada suatu state |
| FHS | Filesystem Hierarchy Standard |
| GECOS | Field komentar di `/etc/passwd`, biasanya nama lengkap user |
| utmp / wtmp / lastlog | Tiga file catatan aktivitas login user |
| Syslog | Sistem logging; model client/server, bisa kirim ke server terpisah |
| `at` / `cron` | Penjadwalan tugas sekali jalan / berulang |
| `fdisk -l` | Perintah untuk mendaftar device sebelum akuisisi |

## Isi

### Filesystem Ext
Slide mencatat bahwa **sebagian besar sistem Linux saat ini memakai Ext3**. Ext3 adalah penerus **Ext2**, yang **menambahkan journaling tapi mempertahankan struktur dasar Ext2**. Faktanya, **volume Ext3 akan mount dengan senang hati sebagai Ext2** kalau user menjalankan perintah mount dengan tepat.

Banyak filesystem lain tersedia lewat kernel Linux, termasuk **ReiserFS, XFS, dan JFS**. Dan ini pengamatan forensik yang bagus:

> Karena filesystem ini **umumnya tidak dipakai di instalasi Linux default**, **keberadaannya bisa menandakan sistem yang dibangun untuk tujuan khusus** (bukan sistem desktop penggunaan umum).

**Ext4** adalah pengganti modern Ext3 yang mulai muncul sebagai opsi instalasi default di banyak distribusi. **Struktur metadata-nya tetap konsisten** dengan yang ada di Ext2/Ext3, tapi **lapisan data unit-nya berubah cukup drastis**.

> [!info] Konteks tambahan (bukan dari slide)
> Pengamatan "filesystem tidak biasa = sistem tujuan khusus" itu contoh bagus dari **penalaran forensik**, bukan sekadar hafalan teknis. Kamu belum melihat satu file pun, tapi keberadaan XFS saja sudah memberi tahu sesuatu tentang siapa yang membangun mesin itu dan untuk apa. Bandingkan dengan penalaran serupa soal `.bash_history` di [[W04 - Mac OS X System and Artifacts]].

### Empat lapisan filesystem Ext
Ini penerapan langsung **File System Abstraction Model** dari [[W03 - Disk and File System Analysis]].

**Lapisan File System.** Ext punya dua komponen utama:
- **Superblock** — struktur data yang ditemukan **1024 byte dari awal filesystem Ext**. Berisi informasi tentang **layout filesystem**, mencakup informasi **alokasi block dan inode**, serta **metadata kapan terakhir filesystem di-mount atau dibaca**.
- **Group descriptor tables**

**Lapisan File Name.** Nama file di Ext disimpan sebagai **directory entry**. Entri-entri ini disimpan di dalam **direktori**, yang sebenarnya cuma **block yang diisi directory entry**. Tiap directory entry berisi:
- **Nama file**
- **Alamat inode** yang terkait dengan file itu
- **Flag** yang menandakan apakah nama itu merujuk ke direktori atau file biasa

Ext mengizinkan **beberapa nama file menunjuk ke file yang sama** — nama tambahan ini disebut **hard link**. Hard link adalah **directory entry tambahan yang menunjuk ke inode yang sama**, dan **tiap hard link menaikkan link count inode itu satu**.

**Lapisan Metadata.** Metadata file di Ext disimpan di **inode**. Item yang menarik secara forensik di dalam inode Ext:
- **Ukuran file dan block yang dialokasikan**
- **Informasi ownership dan permission**
- **Timestamp** yang terkait dengan file
- **Flag** apakah dia milik direktori atau file biasa
- **Link count** — jumlah nama file yang merujuk ke inode ini

Informasi ownership mencakup nilai **UID (User Identifier)** dan **GID (Group Identifier)**, yang bisa penting di banyak pemeriksaan berbeda.

**Lapisan Data Unit.** Data unit di Ext disebut **block**. Ukurannya **1K, 2K, atau 4K** sesuai yang ditulis di superblock. Tiap block punya **alamat** dan merupakan bagian dari **block allocation group** seperti dijelaskan di block descriptor table. Alamat dan grup block **dimulai dari 0** di awal filesystem, lalu bertambah.

### MAC times
Inode Ext menyimpan **empat timestamp**, umumnya disebut **MAC times**:

| Timestamp | Diperbarui ketika |
| --- | --- |
| **(M)odified** | **Isi** file atau direktori **ditulis**. Jadi kalau file diedit, atau ada entri ditambah/dihapus dari direktori, timestamp ini berubah |
| **(A)ccessed** | **Isi** file atau direktori **dibaca**. Aktivitas apa pun yang membuka file untuk dibaca atau melihat isi direktori akan memperbarui timestamp ini |
| **(C)hanged** | **Inode-nya dimodifikasi**. Perubahan permission apa pun, atau perubahan yang menyebabkan Modified berubah, **juga** akan mengubah timestamp ini |
| **(D)eleted** | **Hanya** diperbarui ketika file dihapus |

> [!info] Konteks tambahan (bukan dari slide)
> Jebakan klasik: **"Changed" bukan sinonim "Modified".** *Modified* = **isinya** berubah. *Changed* = **kartu katalognya** yang berubah (permission, ownership, nama). Ubah isi file → M dan C dua-duanya berubah. Ubah permission-nya saja → **cuma C** yang berubah. Itu sebabnya C berguna untuk mendeteksi manipulasi: penyerang yang mengubah timestamp file (*timestomping*) sering lupa bahwa tindakan itu sendiri memperbarui C. Konsep MAC times ini jadi tulang punggung [[W11 - Timeline Analysis and Correlation]].

### Proses boot Linux
Memahami proses boot itu penting saat menyelidiki sistem Linux. Pengetahuan tentang file yang dipakai saat startup bisa membantu examiner menentukan **versi sistem operasi yang berjalan dan kapan diinstal**. Selain itu, karena sifatnya yang terbuka, **user dengan hak istimewa yang cukup bisa mengubah banyak aspek proses boot** — jadi kamu perlu tahu **di mana mencari modifikasi jahat**.

Urutannya:

1. **Boot loader** dijalankan — dia menemukan dan memuat **kernel**.
2. **Kernel** adalah inti sistem operasi, umumnya ada di direktori **`/boot`**.
3. **Initial ramdisk (initrd)** dimuat. File initrd berisi **device driver, modul filesystem, modul logical volume**, dan item lain yang dibutuhkan untuk boot tapi **tidak dibangun langsung ke dalam kernel**.
4. Kernel **menginisialisasi hardware sistem**.
5. Kernel mulai menjalankan yang kita kenal sebagai sistem operasi, dimulai dengan proses **`/sbin/init`**.

Setelah `init` mulai, ada **dua metode utama** untuk membangkitkan sistem operasi Linux: **gaya System V** dan **gaya BSD**. Distribusi Linux umumnya mengikuti contoh System V untuk sebagian besar hal, termasuk tugas init dan pemrosesan runlevel.

| | **System V** | **BSD** |
| --- | --- | --- |
| Popularitas | **Gaya init paling umum** di distribusi Linux | Lebih sederhana |
| File yang dibaca | **`/etc/inittab`** untuk menentukan **runlevel** default | **`/etc/rc`** untuk menentukan service; konfigurasi dari **`/etc/rc.conf`**; service tambahan dari **`/etc/rc.local`**; kadang juga script dari **`/etc/rc.d/`** |
| Konsep khas | **Runlevel** — deskripsi numerik untuk sekumpulan script yang dijalankan pada suatu state. Contoh: **runlevel 3** = lingkungan konsol multiuser penuh; **runlevel 5** = lingkungan grafis | — |
| Dipakai oleh | Kebanyakan distribusi | **Slackware, Arch Linux**, dan lainnya |

### Filesystem Hierarchy Standard
Struktur direktori standar yang seharusnya diikuti sistem Linux didefinisikan di **Filesystem Hierarchy Standard (FHS)**. Standar ini menggambarkan organisasi dan penggunaan yang tepat atas berbagai direktori di sistem Linux.

Catatan penting: **FHS tidak dipaksakan**, tapi **sebagian besar distribusi Linux mengikutinya sebagai best practice**.

### User account: tiga file autentikasi
**`/etc/passwd`** adalah tempat pertama untuk mulai mencari informasi terkait user account. Berisi daftar user dan **path lengkap home directory** mereka.

Contoh entri dari slide, beserta tujuh field-nya:

```
forensics:x:500:500::/home/forensics:/bin/bash
```

| # | Field | Keterangan |
| --- | --- | --- |
| 1 | **username** | `forensics` |
| 2 | **hashed password field** | **Sudah deprecated** (isinya `x`, password sebenarnya di `/etc/shadow`) |
| 3 | **user ID** | `500` |
| 4 | **primary group ID** | `500` |
| 5 | **GECOS comment field** | Umumnya nama lengkap user, atau nama yang lebih deskriptif untuk service account |
| 6 | **path home directory** | `/home/forensics` |
| 7 | **program saat login awal** | Normalnya shell default user — `/bin/bash` |

**`/etc/group`** formatnya mirip `/etc/passwd` tapi **field-nya lebih sedikit**:

```
root:x:0:root
bin:x:1:root,bin,daemon
daemon:x:2:root,bin,daemon
wheel:x:10:root
```

Field-nya: **nama grup**, **hash password grup** (grup berpassword jarang dipakai), **group ID**, dan **daftar anggota grup yang dipisah koma**.

Slide menandai satu hal yang langsung berguna dalam investigasi:

> **User tidak sah tambahan di grup `root` atau `wheel` bisa mencurigakan dan layak diselidiki lebih lanjut.**

**`/etc/shadow`** adalah item ketiga yang dibutuhkan untuk autentikasi dasar Linux. Berisi **hash password user** dan informasi terkait password. Delapan field-nya:

| # | Field |
| --- | --- |
| 1 | **Username** |
| 2 | **Encrypted password** |
| 3 | **Jumlah hari sejak Unix epoch (1 Jan 1970)** saat password terakhir diubah |
| 4 | **Minimum hari** antar perubahan password |
| 5 | **Waktu maksimum** password berlaku |
| 6 | **Jumlah hari sebelum kedaluwarsa** untuk memperingatkan user |
| 7 | **Tanggal kedaluwarsa absolut** |
| 8 | **Dicadangkan** untuk penggunaan masa depan |

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan contoh `/etc/shadow` di slide: `bin:*:...` dan `gdm:!!:...`. Tanda `*` dan `!!` di field password itu **bukan hash** — itu penanda bahwa akun tersebut **tidak bisa dipakai login**. Jadi kalau kamu melihat akun sistem yang tiba-tiba punya hash asli, itu tanda bahaya: seseorang **mengaktifkan akun service supaya bisa dipakai login**. Ini teknik persistence yang klasik.

### Home directory
Di Linux, home directory user melayani tujuan yang kurang lebih sama seperti di sistem operasi lain: **memberi user lokasi untuk menyimpan data spesifik mereka**. Proses dan service yang berperilaku baik juga akan menyimpan data yang dibuat otomatis di subdirektori.

| Direktori | Isinya |
| --- | --- |
| **Desktop** | File di sini seharusnya **terlihat di desktop user** dalam sesi grafis interaktif |
| **Documents** | Direktori default untuk file dokumen kantoran — teks, spreadsheet, presentasi |
| **Downloads** | Default untuk file yang **diunduh dari host remote**; browser yang GNOME-aware, klien file-sharing, dan sejenisnya menaruh datanya di sini |
| **Music** | Lokasi default file musik |
| **Pictures** | Lokasi default gambar. **Catatan: gambar hasil scan atau dari perangkat imaging yang terpasang (webcam, kamera) kemungkinan berakhir di sini** kecuali diarahkan lain |
| **Public** | File yang dibagikan ke orang lain |
| **Templates** | Menyimpan template dokumen. **Direktori ini kosong secara default, jadi tambahan apa pun bisa menandakan jenis file yang sering dipakai** |
| **Videos** | Lokasi default video |

### Shell history
Shell default di sebagian besar distribusi Linux adalah **BASH**. Perintah yang diketik di sesi shell mana pun biasanya disimpan di file **`.bash_history`** di home directory user. Sesi shell mencakup **virtual terminal langsung, jendela aplikasi terminal GUI, atau login remote lewat SSH**.

Kelemahannya sama seperti di macOS: **bash mencatat history sebagai daftar perintah sederhana, tanpa timestamp** atau indikasi kapan perintah dimasukkan. **Korelasi entri history dengan informasi waktu dari filesystem atau logfile akan penting** kalau waktu eksekusi suatu perintah relevan untuk investigasimu.

### SSH
Direktori **`.ssh`** berisi file terkait penggunaan klien **Secure Shell (ssh)**. SSH sering dipakai di sistem Linux dan mirip-Unix untuk terhubung ke sistem remote lewat konsol teks. SSH juga menawarkan **transfer file, connection tunneling, dan kemampuan proxy**. Mungkin ada **file konfigurasi klien** yang bisa menandakan kasus penggunaan SSH tertentu.

Yang paling berguna: saat user terhubung ke host remote memakai program ssh, **hostname atau alamat IP host remote beserta public key host itu dicatat di file `.ssh/known_hosts`**. Entri di file ini **bisa dikorelasikan dengan log server untuk mengaitkan aktivitas tersangka ke mesin tertentu**.

> [!info] Konteks tambahan (bukan dari slide)
> `known_hosts` itu artifact yang kuat karena dia **catatan ke mana user pernah pergi**, dan sebagian besar user tidak sadar file ini ada. Kalau mesin tersangka punya entri untuk server korban, dan log server korban menunjukkan koneksi dari IP tersangka pada waktu yang cocok — kamu punya korelasi dua arah. Ini persis pola kerja yang jadi materi [[W11 - Timeline Analysis and Correlation]].

### Log
Analisis log di Linux **umumnya cukup lugas**. Sebagian besar log disimpan dalam **clear text, satu baris per event**. Mengidentifikasi log mana yang berisi data yang kamu cari bisa jadi menantang, tapi **memprosesnya setelah ketemu biasanya lebih ringan daripada di sistem Windows**.

Tapi slide memperingatkan bahwa kemudahan ini bermata dua:

> Log di sistem Linux cenderung **"roll over" setelah 28–30 hari** secara default, dan **menghapus atau memodifikasi log adalah salah satu tugas paling dasar yang mungkin dilakukan penyerang**.

**Log aktivitas user.** Catatan langsung aktivitas user di sistem Linux disimpan di **tiga file utama**:

| File | Isinya |
| --- | --- |
| **`/var/run/utmp`** | **Hanya menyimpan informasi logon yang sedang aktif** |
| **`/var/log/wtmp`** | Menyimpan informasi logon **jangka panjang** (sesuai periode rotasi log sistem) |
| **`/var/log/lastlog`** | — |

`utmp` dan `wtmp` mencatat **logon dan logoff user dalam format biner**. Keduanya **bisa diakses lewat perintah `last` dengan flag `-f`**.

**Syslog.** Sebagian besar log sistem di Linux disimpan di bawah direktori **`/var/log`** — entah langsung di root direktori itu atau di berbagai subdirektori spesifik aplikasi yang menghasilkan log. **Syslog beroperasi dengan model client/server**, yang memungkinkan **event dicatat ke syslog server remote yang khusus**.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa remote syslog server itu penting secara forensik: kalau log cuma ada di mesin yang diretas, **penyerang bisa menghapusnya** — dan slide sendiri bilang itu tugas paling dasar seorang penyerang. Kalau log dikirim real-time ke server terpisah, penyerang harus meretas **dua** mesin untuk menutupi jejaknya. Ini alasan utama organisasi memakai log server terpusat.

### Penjadwalan tugas
Ada **dua mekanisme utama** untuk menjadwalkan pekerjaan di Linux:

| Mekanisme | Untuk | Lokasi |
| --- | --- | --- |
| **`at`** | Menjalankan tugas **sekali**, pada titik waktu tertentu di masa depan | Job `at` ada di **`/var/spool/cron`** |
| **`cron`** | Menjadwalkan **tugas berulang** — proses yang dijalankan tiap malam, seminggu sekali, dua minggu sekali, dan seterusnya | **Dua lokasi**, lihat di bawah |

**Cron job sistem** ada di sekumpulan direktori yang didefinisikan di file **`/etc/crontab`**, dan biasanya di direktori bernama tepat: **`/etc/cron.hourly`, `/etc/cron.daily`, `/etc/cron.weekly`, dan `/etc/cron.monthly`**.

**Tugas terjadwal yang ditambahkan user** akan ada di **`/var/spool/cron`** — sama seperti job yang ditambahkan lewat perintah `at`.

Slide menutup dengan peringatan yang tegas:

> Seperti bisa kamu duga, **cron job adalah cara yang luar biasa bagi penyerang untuk mempertahankan persistence** di sistem yang sudah dikompromikan, jadi **memverifikasi job-job ini akan sangat kritis dalam investigasi intrusi**.

### Praktik di Kali Linux

**Identifikasi device dengan `fdisk`.** Untuk mendaftar device dan memastikan kamu mengetahuinya **sebelum melakukan operasi akuisisi apa pun**, perintah **`fdisk -l`** harus dijalankan **sebelum yang lain**. Mungkin perlu **`sudo fdisk -l`** kalau yang sebelumnya tidak berfungsi — `sudo` memungkinkan user menjalankan perintah sebagai root, mirip fitur **Run as Administrator di Windows**.

Contoh dari slide: ada satu hard disk terdaftar sebagai **`sda`**, dengan partisi primer **`sda1`**, serta partisi Extended dan Linux swap sebagai **`sda2`** dan **`sda5`**. Setelah memasang flash drive 2 GB untuk akuisisi, `fdisk -l` dijalankan lagi dan menunjukkan: **Disk `sdb`, ukuran 1,9 GB, sector size 512 byte, filesystem FAT32.**

**Menjaga integritas bukti.** Untuk membuktikan bukti tidak dirusak, **hash bukti harus disediakan sebelum dan selama, atau setelah, akuisisi**. Di Kali Linux, dipakai perintah **`md5sum`** diikuti path device:

```bash
md5sum /dev/sdx
```

Saat melakukan akuisisi atau forensic imaging memakai **`dc3dd`**, kita harus mendapat **hasil yang persis sama** saat menghitung hash file image yang dibuat — untuk memastikan bukti asli dan salinannya **benar-benar sama**, sehingga integritas bukti terjaga. Slide juga membuat **hash SHA-1** untuk keperluan perbandingan.

**Fitur `dc3dd`.** Slide menyebut dc3dd menawarkan yang terbaik dari DD dengan lebih banyak fitur:

- **Hashing on-the-fly** dengan lebih banyak pilihan algoritma (**MD5, SHA-1, SHA-256, SHA-512**)
- **Verifikasi hash**
- **Meter** untuk memantau progres dan waktu akuisisi
- **Menulis error ke file**
- **Memecah file output** (bisa campur output terpecah dan utuh)
- **Verifikasi file**
- **Wiping file output** (pattern wiping)

> **Tugas dari slide 32:** coba lakukan fitur-fitur di atas di Linux kamu.

## Diagram & Visual
- **Slide 16 — Tabel Filesystem Hierarchy Standard: direktori utama Linux dan isinya**
  ![[99-Assets/Forensics/W06-slide16.png]]
- **Slide 29 — output `fdisk -l` menunjukkan disk `sda` beserta partisinya**
  ![[99-Assets/Forensics/W06-slide29.png]]
- **Slide 30 — output `fdisk -l` setelah flash drive 2 GB dipasang (`sdb`)**
  ![[99-Assets/Forensics/W06-slide30.png]]
- **Slide 31 — output `md5sum` dan `sha1sum` untuk verifikasi integritas bukti**
  ![[99-Assets/Forensics/W06-slide31.png]]
  ![[99-Assets/Forensics/W06-slide31b.png]]

> [!warning] **Slide 27 ("Common Log file of Interest") isinya cuma judul** — tabel daftar logfile pentingnya tidak terekstrak, baik sebagai teks maupun gambar. Itu materi praktis yang langsung kepakai. **Buka PPT aslinya di slide 27.**
>
> Isi **Tabel FHS di slide 16** juga berupa gambar, jadi daftar direktori Linux beserta isinya tidak bisa dibaca dari teks note ini.

## Rumus / Sintaks

Lokasi penting Linux:
```
/boot                    kernel
/sbin/init               proses init
/etc/inittab             runlevel default (System V)
/etc/rc, rc.conf, rc.local, rc.d/    konfigurasi init (BSD)
/etc/passwd              daftar user (7 field)
/etc/group               daftar grup
/etc/shadow              hash password (8 field)
/var/run/utmp            login yang sedang aktif
/var/log/wtmp            login jangka panjang
/var/log/lastlog         login terakhir
/var/log/                log sistem (syslog)
/etc/crontab             definisi direktori cron sistem
/etc/cron.{hourly,daily,weekly,monthly}    cron job sistem
/var/spool/cron          cron job user + job `at`
~/.bash_history          riwayat shell (TANPA timestamp)
~/.ssh/known_hosts       host remote yang pernah dikunjungi
```

Perintah praktikum:
```bash
sudo fdisk -l            # daftar device, JALANKAN SEBELUM AKUISISI
md5sum /dev/sdx          # hash bukti
last -f /var/log/wtmp    # baca catatan login
```

Angka yang perlu diingat:
```
superblock Ext : 1024 byte dari awal filesystem
block Ext      : 1K, 2K, atau 4K
rotasi log     : 28-30 hari (default)
runlevel 3     : konsol multiuser penuh
runlevel 5     : lingkungan grafis
```

## Pertanyaan Terbuka
- Slide 3 mencantumkan **"Systemd Journal and Log Analysis"** dan **"Artefak Malware dan Persistence di Linux"** sebagai learning outcome, tapi **keduanya tidak dibahas**. **systemd** khususnya penting: distribusi Linux modern (Ubuntu, Fedora, Debian, RHEL) **sudah tidak memakai System V init** yang dijelaskan panjang lebar di deck ini — mereka memakai **systemd** dengan `journalctl`, bukan runlevel dan `/etc/inittab`. Materi boot process di deck ini **sudah usang untuk sistem modern**. **Wajib ditanyakan.**
- **XFS** disebut di learning outcome ("Linux File Systems (Ext4, XFS) and Forensic Considerations") tapi cuma muncul sekali sebagai nama, tanpa penjelasan sama sekali.
- Deck menyebut **"sebagian besar sistem Linux saat ini memakai Ext3"** — sudah usang, Ext4 sudah jadi default sejak sekitar 2010.
- **`/var/log/lastlog`** disebut sebagai salah satu dari tiga file utama, tapi **tidak pernah dijelaskan** apa isinya atau bedanya dengan utmp/wtmp.
- Slide 28 menulis job `at` ada di `/var/spool/cron`, padahal di banyak distribusi lokasinya `/var/spool/at`. Perlu diverifikasi mana yang jadi acuan jawaban.
- Isi **Tabel FHS (slide 16)** dan **daftar logfile (slide 27)** tidak bisa diekstrak. Keduanya materi hafalan yang praktis.
- Slide 32 menyebut dc3dd sebagai singkatan dari **"DD (Data Destroyer)"** — itu keliru, `dd` bukan singkatan resmi dari itu (sering dijelaskan sebagai *data duplicator* atau *convert and copy*). Kalau ditanya di ujian, jawab sesuai slide tapi ketahui bahwa ini tidak akurat.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W03 - Disk and File System Analysis]]
- [[W05 - Data Acquisition]]
- [[W07 - Windows System Artifacts]]
- [[W11 - Timeline Analysis and Correlation]]
- [[Forensics - Review dan Glosari]]
