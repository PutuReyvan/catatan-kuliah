---
matkul: Forensics
sks: 2
sumber: rangkuman dari W01-W13
tags: [kuliah/forensics, ujian]
status: draft
diproses: 2026-09-03
---

# Computer Forensics — Review dan Glosari

Rangkuman lintas pertemuan W01–W13. Dibaca sebelum ujian, bukan pengganti note pertemuan.

---

## Bagian 1 — Review Cepat

### Yang paling mungkin keluar di ujian

Pembacaan atas apa yang **diulang-ulang** dan apa yang **berbentuk daftar terstruktur** — dua hal yang paling gampang dijadikan soal.

**1. File System Abstraction Model.** Kalau cuma sempat hafal satu hal dari matkul ini, hafal ini. Dia menjelaskan hampir semua materi lain.

```
Disk → Volume → File System → Data Unit → Metadata → File Name
fisik  sector    tata letak    isi data    inode      nama
```

**2. Order of Volatility** (7 tingkat, dari yang paling cepat hilang):

```
1. CPU, cache, register
2. Routing table, ARP cache, process table, kernel statistics
3. Memory (RAM)
4. Temporary filesystem / swap space
5. Data di hard disk
6. Data ter-log secara remote
7. Data di media arsip
```

**3. Empat kategori deleted data**, dari yang paling bisa dipulihkan:

| Kategori | Yang sudah putus | Cara pemulihan |
| --- | --- | --- |
| **Deleted** | Belum ada; semua lapisan masih terhubung | Catat nama + metadata, ekstrak data unit |
| **Orphaned** | Link nama↔metadata | Data dan metadata masih bisa, **tapi tidak ada korelasi ke nama file** |
| **Unallocated** | Nama dan metadata sudah dipakai ulang | **Carving** dari unallocated space |
| **Overwritten** | Data unit sudah dialokasikan ke file lain | **Pemulihan penuh mustahil**; sebagian mungkin |

**4. Tiga fase proses forensik:** **Acquisition → Analysis → Presentation**. Dan **delapan tahap** dari definisi di W01 (documenting the scene, search & seizure, evidence preservation, data acquisition, data analysis, case analysis, reporting, testifying).

**5. Artifact vs Evidence.** *Artifact* = jejak yang tertinggal akibat aktivitas, bisa tidak berbahaya. *Evidence* = sesuatu yang dipakai dalam proses hukum. **Memakai istilah "evidence" serampangan bisa membuat examiner kena masalah.**

**6. MAC times (Linux) vs timestamp Windows.** Ini jebakan klasik karena namanya mirip tapi tidak sama:

| Linux (Ext, di inode) | Windows (file system metadata) |
| --- | --- |
| **M**odified — isi ditulis | **Created** — kapan dibuat di media **ini** |
| **A**ccessed — isi dibaca | **Modified** — isi diubah lalu disimpan |
| **C**hanged — **inode**-nya berubah (permission, ownership) | **Accessed** — diakses filesystem, **bukan berarti "dibuka"** |
| **D**eleted — hanya saat file dihapus | — |

> Jebakan: **Changed ≠ Modified.** *Modified* = isinya berubah. *Changed* = kartu katalognya berubah.

**7. Tiga mode tidur Windows** dan nilai forensiknya: **Sleep** (RAM saja, volatile), **Hibernation** (semua RAM **ditulis ke disk**, nilai tinggi), **Hybrid sleep** (keduanya).

**8. Tiga file autentikasi Linux:** `/etc/passwd` (7 field), `/etc/group`, `/etc/shadow` (8 field).

**9. Empat tahap incident response:** Preparation → Detection and Analysis → Containment, Eradication and Recovery → Post Incident Activity.

**10. Tiga jenis metadata gambar:** **EXIF** (perangkat: kamera, waktu, **geolokasi**), **IPTC** (pers/fotojurnalis), **XMP** (Adobe 2001, XML, extensible).

**11. Tiga protokol email:** **SMTP** (kirim), **POP** (terima), **IMAP** (dua arah di server).

**12. MBR vs GPT:** MBR = **4 partisi primer, maks 2 TB**. GPT = **128 partisi primer, maks 8 ZB**.

**13. Angka-angka yang mungkin ditanya:**

```
sector historis        : 512 byte
data unit modern       : 4096 byte (4K) atau lebih
superblock Ext         : 1024 byte dari awal filesystem
volume header HFS+     : 1024 byte dari awal (+ cadangan 1024 byte sebelum akhir)
catalog record HFS+    : 8K
extents overflow HFS+  : dipakai kalau fork > 8 extent
MD5                    : 128 bit
SHA1                   : 160 bit
ASCII                  : 128 karakter (94 printable)
rotasi log Linux       : 28-30 hari
RAM minimum tool forensik : 4 GB
```

---

### Ringkasan satu paragraf per pertemuan

**[[W01 - Digital Forensic Fundamental]]** — Digital forensic = metode ilmiah teruji untuk mendokumentasikan TKP elektronik sampai bersaksi sebagai ahli. Tujuannya "menemukan fakta, dan lewat fakta merekonstruksi kebenaran peristiwa". Examiner itu **digital archaeologist** yang menggali **artifact**. Artifact ≠ evidence. Proses inti: Acquisition (buat working copy + catat semua tindakan), Analysis (identification, analysis, interpretation), Presentation (laporan + mempertahankannya saat digugat). Ancaman: media sosial, hotspot gratis, VPN, **TRIM di SSD**, dan organisasi tanpa DFIR.

**[[W02 - Key Technical Concepts]]** — Tanpa paham cara kerja komputer, pemeriksaan tidak bisa dilakukan. Bit/byte/hex (1 huruf = 1 byte; hex base 16 dengan prefix `0x`). ASCII 128 karakter vs Unicode semua bahasa. **File extension paling tidak andal — pakai magic number.** Tiga cara data ditulis: elektromagnetik, flash (transistor, tahan tanpa listrik), optical (bump vs land). **RAM volatile, hard drive non-volatile** — ini pembeda terpenting secara forensik. Empat computing environment: stand-alone, networked, mainframe, cloud (urutan kesulitan akuisisi). Sector = 512 byte.

**[[W03 - Disk and File System Analysis]]** — **File System Abstraction Model** enam lapisan. MBR vs GPT. RAID 0 (striping, cepat, rapuh), RAID 1 (mirror, andal, boros), RAID 5 (striping + parity, min 3 disk). Hashing (MD5 128-bit, SHA1 160-bit) untuk **verifikasi integritas** — ubah 1 bit, hash berubah total. Carving = cari header/magic value, tebak titik akhir, simpan. **foremost**. Forensic imaging seperti "menyalin TKP-nya sendiri". Empat kategori deleted data. **File slack**: file 1 byte makan 1 block penuh, sisanya masih berisi data pemilik lama. `dd` → `dcfldd` (fork, +hash/log/split) → `dc3dd` (patch, ikut mainline).

**[[W04 - Mac OS X System and Artifacts]]** — HFS+ (varian HFSJ journaling, HFSX case-sensitive). Volume header 1024 byte dari awal + cadangan di akhir. Data unit = allocation block (4K), dialamati lewat **extent**. **Lima special file**: allocation (bitmap), **catalog** (padanan MFT, record 8K, berisi CNID), extents overflow (>8 extent = fragmentasi parah), startup, attributes. Artifact banyak berbentuk **.plist** (XML atau binary). **`launchd`** pengganti init, baca dari 4 direktori LaunchDaemons/LaunchAgents ← tempat cek persistence. Home directory = pusat artifact user, terutama `Library`. `.Trash` menyimpan path asli di `.DS_Store` **sejak 10.6**. **`sleepimage` = RAM dump permanen di disk.**

**[[W05 - Data Acquisition]]** — "Smoking gun tidak akan sampai ke juri kalau tidak dikumpulkan dengan benar." Dokumentasi TKP oleh first responder (ruangan, kondisi perangkat, isi layar, **buku dan catatan**, kabel). Ponsel: **Faraday bag** atau kaleng cat, jangan dimatikan kalau bisa. **Order of Volatility** 7 tingkat. **"If you don't write it down, it didn't happen."** **Chain of Custody** = formulir yang menjamin integritas bukti antar tangan. Bukti ditandai dengan permanent marker. **Forensic clone = bit-per-bit**; copy-paste tidak dapat unallocated space, file terhapus, dan data filesystem. **Write blocker** (hardware/software) menjaga admissibility. RAM berisi **password plaintext**, data tak terenkripsi, proses, IM, IP, Trojan. Standar: **SWGDE 2018**.

**[[W06 - Linux System and Artifacts]]** — Ext3 = Ext2 + journaling; Ext4 metadata sama tapi data unit berubah. **Superblock** 1024 byte dari awal + group descriptor tables. Directory entry = nama + alamat inode + flag. **Hard link** menaikkan link count. **Inode** simpan ukuran, block, UID/GID, permission, dan **MAC(D) times**. Block 1/2/4K. Boot: bootloader → kernel (`/boot`) → initrd → `/sbin/init` → **System V** (runlevel, `/etc/inittab`) atau **BSD** (`/etc/rc`). **FHS** tidak dipaksakan tapi diikuti. `/etc/passwd` (7 field), `/etc/group` (**user asing di root/wheel = curiga**), `/etc/shadow` (8 field). `.bash_history` **tanpa timestamp**. **`.ssh/known_hosts`** = catatan ke mana user pernah pergi. Log clear text, **roll over 28–30 hari**, gampang dihapus penyerang. utmp (aktif) / wtmp (jangka panjang) / lastlog, dibaca `last -f`. **`at`** sekali, **`cron`** berulang — **cron = cara favorit persistence**. Praktik: `fdisk -l`, `md5sum`, `dc3dd`.

**[[W07 - Windows System Artifacts]]** — **Menekan delete tidak melakukan apa pun terhadap datanya**; cuma menandai ruangnya tersedia. Tiga mode tidur; **hibernation menulis semua RAM ke disk** (`hiberfil.sys`). **Registry** = "database untuk file konfigurasi" / sistem saraf pusat. **Masalah atribusi**: kita tahu apa yang dicari di Google, tapi sulit membuktikan siapa yang mengetik — jembatannya **SID**, tapi SID mengikat ke **akun**, bukan orang. External drive artifact untuk kasus pencurian IP. Dua rasa metadata: application dan file system (Created/Modified/Accessed; **"Accessed" ≠ "opened"**). **MRU**. **Restore point** = snapshot; **shadow copy** = sumber datanya, bisa menunjukkan **perubahan file dari waktu ke waktu** dan menyimpan file terhapus. **FTK Imager** untuk akuisisi live, dianalisis dengan **Volatility**.

**[[W08 - File Recovery and Data Carving]]** — Carving memakai **karakteristik file (header/footer)**, bukan metadata filesystem — jadi tetap jalan walau extension diganti. **foremost** (header/footer, hasilkan `audit.txt`), **Scalpel**, **recoverjpeg**, **bulk_extractor** (tidak terbatas jenis file: **nomor kartu kredit, email, URL, pencarian, media sosial**). Separuh deck ini soal **antiforensic**. Menyembunyikan data: ganti nama → kubur → **file dalam file** → **enkripsi**. Enkripsi: plain text + algorithm + key → cipher text; klasik = substitution & transposition; **key space** menentukan kelayakan brute force. Menyerang password lebih mudah lewat manusianya (**password berbasis kata dan pola**). **Steganography** = *stegos* (tertutup) + *graphie* (tulisan). **Drive wiping** efektif; **defrag/reformat hasilnya terbatas**.

**[[W09 - File and Archives Analysis]]** — Dua aktivitas: **content identification** dan **metadata extraction**. **"Dokter bukan dokter karena menulis Dr. di depan namanya"** — extension cuma konvensi demi kenyamanan. Tiga metadata gambar: **EXIF** (perangkat + **geolokasi**), **IPTC** (pers), **XMP** (Adobe, XML, extensible, dipakai juga di PDF). **Container vs codec**; **FourCC** = 4 byte pengidentifikasi codec di AVI, **mirip magic number**. RIFF menurunkan WAV dan AVI. Archive bisa mempertahankan **timestamp filesystem dan UID/GID sistem asal**. **RAR = format pilihan grup pembajakan dan sering dipakai eksfiltrasi data saat intrusi.** PDF bisa memuat **JavaScript** — saat buku ditulis, **PDF berbahaya salah satu vektor utama kompromi desktop**. Metadata PDF: Document Information Directory + XMP.

**[[W10 - Internet Artifacts]]** — **Tidak ada yang menunjukkan *evidence dynamics* lebih baik dari internet artifact.** Artifact IE: cookies (teks polos per host), **INDEX.DAT** (biner, lacak URL + jumlah kunjungan, baca dengan FTK/EnCase; MSIE punya 3 direktori: History, Cookies, Temporary Internet Files), **NTUSER.DAT** (registry per profil, memuat browser history), cache. **"Profil, bukan user."** **Email salah satu sumber bukti terbaik** — orang blak-blakan, dan email **persisten di banyak lokasi** (mesin tersangka, penerima, server, backup, smartphone, provider). SMTP/POP/IMAP. **Shared account**: pesan ditaruh di **Drafts**, tidak pernah dikirim, jadi tidak ada yang bisa dicegat. **PST** (Outlook) dan **OST** (offline). SNS butuh **subpoena/warrant** dan **cepat**, karena provider membuang data. **Xplico** = NFAT open source, **tidak bisa membaca SSL**.

**[[W11 - Timeline Analysis and Correlation of Artifacts]]** — **Judul tidak cocok isi**: tidak ada timeline sama sekali. Isinya tool: **p0f** (identifikasi OS, **pasif**), **Nmap** (`-v -O -sV -Pn`, **aktif**; port open/filtered/closed), **Linux Explorer** (live forensic: proses, login, port, file mencurigakan, **rootkit**), **credential dumping** (ambil login/password untuk **lateral movement**), **memory dump**, **mimipenguin** (versi Linux dari **mimikatz**, ambil **password plaintext dari memori**).

**[[W12 - Scripting and Automation for Forensic Tasks]]** — Tujuan otomasi: **mengurangi persiapan sebelum analisis sesungguhnya**. **Autopsy** = front end grafis The Sleuth Kit, buatan **Brian Carrier**, built-in di Kali. **PyFLAG** berbasis web+database. **Timeline** dulu cuma MAC times, sekarang ditambah sumber lain. **Relative time**: before, after, during (**window of compromise**). **Titik awal kanonik = waktu filesystem dibuat**; timestamp lebih tua = palsu **atau** sisa ekstraksi arsip. **Inferred time** (untuk data terhapus) dan **embedded time** (versi software pembuat PDF = tanggal paling awal file bisa ada; merek printer = kerangka waktu pencetakan). **Periodicity** = jarak antar peristiwa berulang; **beacon backdoor sangat teratur — tapi auto-update juga**. Fitur Autopsy termasuk **hash database vs NSRL** dan **"topeng virtual"** yang tidak mengubah source data.

**[[W13 - Network Analysis]]** — Bersiap untuk **"kapan", bukan "kalau"** — tapi pertahanan perimeter tetap wajib kokoh. Kasus **FIS 2011, $13 juta dalam sehari**. **Protocol** = bahasa bersama; **TCP/IP** dipakai internet. **P2P** dominan untuk file sharing (termasuk konten ilegal). LAN/WAN/MAN/PAN/CAN/GAN. **Firewall** memfilter trafik dua arah; **IDS/NIDS** mendeteksi pola serangan dan aktivitas tidak biasa — **Snort** beroperasi sebagai sniffer real-time. Empat serangan: **DDoS, IP Spoofing, MitM, Social Engineering**. **Inside threat melewati banyak pengamanan.** Empat tahap incident response. Tantangan: **spoofing, rantai server perantara lintas negara, dan log yang memang tidak pernah dibuat**. **Wireshark** untuk capture; **PCAP** menangkap **OSI Layer 2–7**; **PcapXray** memvisualkan (Malicious, Tor, HTTPS, HTTP, DNS, ICMP).

---

### Pola yang muncul berulang

**Prosedur mengalahkan teknik.** Ini pesan tunggal paling keras di matkul ini, diulang dengan kata berbeda: W01 ("metode yang diturunkan secara ilmiah"), W05 ("tiga jam dokumen itulah yang membawa buktimu ke pengadilan", *"if you don't write it down, it didn't happen"*), W12 ("topeng virtual tanpa mengubah source data"). Kamu bisa menemukan bukti paling menentukan di dunia; kalau tidak bisa membuktikan kamu tidak merusaknya, **bukti itu tidak bernilai**.

**"Menghapus" hampir tidak pernah berarti menghapus.** W03 (empat kategori deleted, file slack), W07 ("menekan delete tidak melakukan apa pun terhadap datanya"), W08 (defrag dan reformat hasilnya terbatas). Yang dihapus adalah **link antar lapisan**, bukan datanya.

**Jangan percaya nama, periksa isinya.** W02 (magic number), W08 (carving berbasis header), W09 ("dokter bukan dokter karena menulis Dr."). Tiga kali, tiga konteks berbeda, satu prinsip.

**Volatilitas menentukan urutan kerja.** W02 memperkenalkan konsepnya, W05 mengubahnya jadi daftar prioritas, W04 dan W07 menunjukkan jalan memutarnya (`sleepimage`, `hiberfil.sys` = RAM yang jadi permanen), W11 menunjukkan apa yang bisa dipanen dari memori (mimipenguin).

**Batas kejujuran disiplin ini: atribusi.** W07 dan W10 dua-duanya berhenti di titik yang sama — kita bisa membuktikan **apa yang terjadi di akun mana**, tapi **tidak bisa membuktikan jari siapa yang di keyboard**. Setiap kesimpulan forensik harus menghormati batas ini.

---

## Bagian 2 — Glosari

### A–C

| Istilah | Arti | Muncul di |
| --- | --- | --- |
| **Acquisition** | Pengumpulan media digital yang akan diperiksa | W01, W05 |
| **Air-gapped device** | Perangkat terputus total dari jaringan; keamanan tertinggi | W05 |
| **Allocation block** | Data unit HFS+, umumnya 4K | W04 |
| **Allocation file** | Special file HFS+; bitmap status alokasi tiap block | W04 |
| **Analysis** | Identifikasi, analisis, dan interpretasi item di dalam media | W01 |
| **Artifact** | Jejak yang tertinggal akibat aktivitas; **bukan** sinonim evidence | W01, W11 |
| **ASCII** | 128 karakter (94 printable), encoding untuk bahasa Inggris | W02 |
| **`at`** | Menjadwalkan tugas **sekali** di masa depan | W06 |
| **Attribution** | Masalah menghubungkan artifact ke **orang** tertentu | W07, W10 |
| **Autopsy** | Front end grafis The Sleuth Kit, buatan Brian Carrier; built-in di Kali | W12 |
| **`.bash_history`** | Riwayat shell; **tanpa timestamp** | W04, W06 |
| **Beacon traffic** | Trafik berkala dari backdoor ke pengendalinya | W12 |
| **Binary** | Sistem bilangan **base 2** | W02 |
| **Bit / Byte** | Satu digit biner / satuan yang mewakili satu karakter | W02 |
| **Block** | Data unit di filesystem turunan Unix; Ext = 1K/2K/4K | W03, W06 |
| **Brute force attack** | Mencoba setiap kombinasi kunci sampai benar | W08 |
| **bulk_extractor** | Tool ekstraksi **tidak terbatas jenis file**: kartu kredit, email, URL, pencarian | W08 |
| **Cache (browser)** | File yang disimpan lokal akibat aktivitas browsing | W10 |
| **Catalog file** | Special file HFS+; **padanan MFT**, record 8K, berisi CNID | W04 |
| **Carving** | Mengekstrak file dari data tak terstruktur lewat header dan magic value | W03, W08 |
| **Chain of Custody (CoC)** | Formulir yang menjamin integritas bukti saat berpindah tangan | W05 |
| **Cipher text** | Versi teracak dari plain text | W08 |
| **Clump** | Kumpulan allocation block di HFS+ | W04 |
| **CNID** | Catalog Node ID; identitas file/folder di catalog record HFS+ | W04 |
| **Codec** | Metode kompresi dan encoding stream | W09 |
| **Container** | File yang menampung satu atau lebih stream | W09 |
| **Content identification** | Menentukan atau memverifikasi sebuah file itu apa | W09 |
| **Cookie** | Di IE berupa file teks polos terpisah per host | W10 |
| **Core dump** | Lihat memory dump | W11 |
| **Credential dumping** | Mengambil login dan password dari OS dan software | W11 |
| **`cron`** | Menjadwalkan tugas **berulang**; **favorit penyerang untuk persistence** | W06 |

### D–H

| Istilah | Arti | Muncul di |
| --- | --- | --- |
| **Data unit** | Unit penyimpanan terkecil yang berdiri sendiri | W03 |
| **`dc3dd`** | Varian dd, patch atas GNU dd, untuk DoD Cyber Crime Center | W03, W06 |
| **`dcfldd`** | Varian dd, fork GNU dd; +hashing, logging, splitting | W03 |
| **`dd`** | Tool imaging open source paling dasar | W03 |
| **Deleted (kategori)** | Semua lapisan masih terhubung; **paling bisa dipulihkan** | W03 |
| **DFIR** | Digital Forensics and Incident Response | W01 |
| **Digital archaeologist** | Metafora untuk examiner yang menggali sisa peristiwa | W01 |
| **Directory entry** | Cara nama file disimpan di Ext: nama + alamat inode + flag | W06 |
| **Dotfile** | File berawalan titik, tersembunyi secara default | W04 |
| **Drive wiping** | Menimpa data agar tidak bisa dipulihkan | W08 |
| **`.DS_Store`** | Di `.Trash` menyimpan **path asli** file terhapus (sejak OS X 10.6) | W04 |
| **Embedded time** | Informasi waktu dari **isi** file (versi software, merek printer) | W12 |
| **Evidence** | Sesuatu yang dipakai dalam proses hukum | W01 |
| **Evidence dynamics** | Bukti terus terbentuk dan hilang seiring aktivitas | W10 |
| **EXIF** | Metadata **perangkat** perekam gambar: kamera, waktu, **geolokasi** | W09 |
| **Ext2 / Ext3 / Ext4** | Keluarga filesystem Linux; Ext3 = Ext2 + journaling | W06 |
| **Extent** | Pointer 4-byte ke allocation block awal + panjangnya (HFS+) | W04 |
| **Extents overflow file** | Special file HFS+ untuk fork dengan **>8 extent** | W04 |
| **Faraday bag** | Wadah pemerisai sinyal untuk ponsel; alternatifnya kaleng cat | W05 |
| **FAT** | File Allocation Table; filesystem umum tertua | W02 |
| **FHS** | Filesystem Hierarchy Standard; tidak dipaksakan tapi diikuti | W06 |
| **File extension** | Cara paling umum **tapi paling tidak andal** mengenali file | W02, W09 |
| **File slack** | Sisa ruang dalam block yang masih berisi data alokasi sebelumnya | W03 |
| **Firewall** | Program di gateway server yang memfilter trafik masuk dan keluar | W13 |
| **Flash memory** | Transistor; bermuatan = 1. **Menyimpan data tanpa listrik** | W02 |
| **foremost** | Tool carving berbasis header/footer; hasilkan `audit.txt` | W03, W08 |
| **Forensic clone** | Salinan bit-per-bit; disebut juga **bit stream image** | W05 |
| **FourCC code** | 4 byte pengidentifikasi codec di AVI; **mirip magic number** | W09 |
| **FTK Imager** | Tool gratis AccessData untuk akuisisi live memori dan drive | W07 |
| **GECOS** | Field komentar di `/etc/passwd`, umumnya nama lengkap user | W06 |
| **GPT** | GUID Partition Table; **128 partisi primer, maks 8 ZB** | W03 |
| **Group descriptor table** | Komponen kedua lapisan filesystem Ext | W06 |
| **Hard link** | Directory entry tambahan yang menunjuk ke inode yang sama | W06 |
| **Hashing** | Fungsi kriptografis untuk **verifikasi integritas bukti** | W03, W05, W06 |
| **Hexadecimal** | Base 16, ditulis 0–9 dan A–F, prefix `0x` | W02 |
| **HFS+** | Filesystem OS X (Mac OS Extended); varian HFSJ dan HFSX | W04 |
| **Hibernation** | Mode tidur yang **menulis semua RAM ke hard drive** (`hiberfil.sys`) | W07 |
| **Hybrid sleep** | Campuran sleep dan hibernation; untuk desktop | W07 |

### I–P

| Istilah | Arti | Muncul di |
| --- | --- | --- |
| **IDS / NIDS** | Intrusion Detection System / versi jaringannya (contoh: **Snort**) | W13 |
| **IMAP** | Protokol email **dua arah**, akses email di server | W10 |
| **INDEX.DAT** | File biner IE; lacak URL yang dikunjungi dan jumlah kunjungan | W10 |
| **Inferred time** | Waktu yang disimpulkan saat timestamp langsung tidak ada | W12 |
| **initrd** | Initial ramdisk; driver dan modul yang dibutuhkan saat boot | W06 |
| **Inode** | Struktur metadata di filesystem Ext | W06 |
| **IoC** | Indicators of Compromise. **Jadi LO di W11 dan W13 tapi tidak pernah didefinisikan** | W11, W13 |
| **IPTC** | Metadata gambar untuk pers dan fotojurnalis | W09 |
| **Kext** | Kernel extension macOS, di `/System/Library/Extensions` | W04 |
| **Key / Key space** | Data untuk enkripsi-dekripsi / panjang ruang kunci | W08 |
| **LAN / WAN** | Local / Wide Area Network | W13 |
| **`last -f`** | Perintah membaca utmp dan wtmp | W06 |
| **Lateral movement** | Berpindah dari satu sistem terkompromi ke sistem lain | W11 |
| **`launchd`** | Pengganti `init` di macOS; baca dari 4 direktori LaunchDaemons/Agents | W04 |
| **Linux Explorer** | Tool live forensic Linux; termasuk deteksi rootkit | W11 |
| **Live vs dead system** | Perangkat menyala vs mati saat diselidiki | W05 |
| **MAC times** | Modified, Accessed, Changed — plus **Deleted** di Ext | W06 |
| **Magic number** | Tanda pengenal **di dalam** file; file signature | W02, W08 |
| **Magnetic disk** | Partikel dimagnetisasi = 1 | W02 |
| **MBR** | Master Boot Record; **4 partisi primer, maks 2 TB** | W03 |
| **`md5sum`** | Perintah Kali untuk membuat hash MD5 bukti | W06 |
| **Memory dump** | Snapshot data memori pada satu momen | W11 |
| **Metadata** | Data tentang data; dua rasa: **application** dan **file system** | W03, W07 |
| **Metadata extraction** | Pengambilan metadata tertanam di sebuah file | W09 |
| **mimikatz / mimipenguin** | Tool password dumping Windows / padanan Linux-nya | W11 |
| **MRU** | Most Recently Used; pintasan ke aplikasi dan file yang baru dipakai | W07 |
| **Nmap** | Network Mapper; scanning **aktif**, OS fingerprinting | W11 |
| **NSRL** | NIST National Software Reference Library; basis data hash file dikenal | W12 |
| **NTFS** | New Technology File System; dipakai Windows modern | W02 |
| **NTUSER.DAT** | File registry per profil user; **memuat browser history** | W10 |
| **Optical storage** | Laser membaca beda pantulan **bump** dan **land** | W02 |
| **Order of Volatility** | Urutan prioritas pengambilan bukti, 7 tingkat | W05 |
| **Orphaned** | Link nama↔metadata sudah tidak akurat | W03 |
| **OST** | Offline Storage Table; penyimpanan email offline Outlook | W10 |
| **Overwritten** | Data unit sudah dialokasikan ke file lain; **pemulihan penuh mustahil** | W03 |
| **p0f** | Tool identifikasi OS; bersifat **pasif** | W11 |
| **P2P** | Peer-to-peer; semua mesin jadi klien sekaligus server | W13 |
| **PCAP / libpcap** | API yang menangkap paket dari **OSI Layer 2–7** | W13 |
| **PcapXray** | Visualisasi trafik: Malicious, Tor, HTTPS, HTTP, DNS, ICMP | W13 |
| **Periodicity** | **Jarak waktu** antar peristiwa berulang (bukan "berapa kali sehari") | W12 |
| **Plain text** | Pesan asli sebelum dienkripsi | W08 |
| **Property list (.plist)** | Format artifact macOS; plain-text XML atau binary | W04 |
| **POP** | Post Office Protocol; klien **menerima** email | W10 |
| **Presentation** | Menyampaikan hasil analisis, termasuk mempertahankannya saat digugat | W01 |
| **PST** | Personal Storage Table; format mail Outlook | W10 |
| **PyFLAG** | Python-based Forensics and Log Analysis GUI; berbasis web + database | W12 |

### Q–Z

| Istilah | Arti | Muncul di |
| --- | --- | --- |
| **RAID 0 / 1 / 5** | Striping / mirroring / striping + parity (min 3 disk) | W03 |
| **RAM (volatile)** | Data hanya ada selama listrik disuplai | W02, W05 |
| **Recycle Bin** | "Tempat sampah" Windows; sering diandalkan menghapus bukti — keliru | W07 |
| **Registry** | Database konfigurasi Windows; "sistem saraf pusat komputer" | W07 |
| **Relative time** | Hubungan antar peristiwa: **before, after, during** | W12 |
| **Restore Point** | Snapshot pengaturan dan konfigurasi sistem | W07 |
| **RIFF** | Resource Interchange File Format; container induk WAV dan AVI | W09 |
| **Rootkit** | Malware yang menyembunyikan keberadaannya di level sistem | W11 |
| **Runlevel** | Deskripsi numerik untuk set script pada suatu state (System V) | W06 |
| **Sector** | Wadah terkecil penyimpanan; **512 byte** | W02 |
| **SHA1** | Menghasilkan hash **160 bit** (MD5: 128 bit) | W03 |
| **SID** | Security Identifier; nomor unik tiap akun Windows | W07 |
| **`sleepimage`** | File hibernasi macOS seukuran RAM; **RAM dump di disk** | W04 |
| **Snort** | NIDS open source; sniffer real-time yang memicu alert | W13 |
| **Social engineering** | Membujuk user berwenang agar membocorkan informasi sensitif | W13 |
| **Steganography** | *Stegos* (tertutup) + *graphie* (tulisan); **menyembunyikan keberadaan pesan** | W08 |
| **Superblock** | Struktur 1024 byte dari awal filesystem Ext | W03, W06 |
| **SWGDE** | Penerbit *Best Practices for Digital Evidence Collection* (2018) | W05 |
| **Syslog** | Sistem logging Linux; model client/server, bisa ke server remote | W06 |
| **Tarball** | Arsip `tar` yang dikompresi GZIP atau BZIP2 | W09 |
| **TCP/IP** | Protokol jaringan yang dipakai internet | W13 |
| **The Sleuth Kit** | Kumpulan tool forensik command-line; Autopsy front end-nya | W12 |
| **Timeline** | Rekonstruksi urutan peristiwa; dulu hanya MAC times | W12 |
| **`.Trash`** | Recycle bin macOS; simpan path asli di `.DS_Store` sejak 10.6 | W04 |
| **TRIM** | Teknologi SSD yang menghapus data jauh lebih efisien | W01 |
| **UID / GID** | User / Group Identifier | W06 |
| **Unallocated space** | Ruang yang ditandai kosong; belum terpakai atau file lamanya dihapus | W02, W03, W08 |
| **Unicode** | Encoding untuk semua bahasa dunia, ribuan karakter | W02 |
| **utmp / wtmp / lastlog** | Login aktif / login jangka panjang / login terakhir | W06 |
| **Volatility** | Tool analisis memori (dipakai di Kali) | W07 |
| **Volume** | Sekumpulan sector pada satu atau beberapa disk | W03 |
| **Volume header** | Struktur inti HFS+, 1024 byte dari awal + cadangan di akhir | W04 |
| **Window of compromise** | Durasi antara masuknya penyerang sampai remediasi | W12 |
| **Wireshark** | Tool capture dan analisis paket; pre-installed di Kali | W13 |
| **Write blocker** | Perangkat/software yang mencegah penulisan ke media bukti | W05 |
| **XMP** | eXtensible Metadata Platform (Adobe, 2001, XML); dipakai juga di PDF | W09 |
| **Xplico** | NFAT open source berbasis GUI; **tidak bisa membaca SSL** | W10 |

---

## Bagian 3 — Yang Perlu Dicek Sendiri

**Materi yang jadi learning outcome tapi tidak ada di deck mana pun** (lihat rekap lengkap per deck di [[_Forensics]]):

1. **Timeline reconstruction** — ada di LO 3 seluruh matkul, jadi judul W11, tapi **tekniknya tidak pernah diajarkan**. Tidak ada `mactime`, `log2timeline`/`plaso`, atau super timeline. Konsepnya cuma disinggung di W12.
2. **IoC (Indicators of Compromise)** — LO di W11 dan W13, **tidak pernah didefinisikan**.
3. **Reporting** — tahap ke-7 di W01 dan bagian dari LO 3, tapi **tidak ada deck yang mengajarkan cara menulis laporan forensik**.
4. **Chain of custody** — LO di W01, baru dibahas di W05. Pastikan mana yang jadi acuan.
5. **Registry analysis** — LO tersendiri di W07, tapi registry cuma **didefinisikan**, tanpa satu pun hive atau key.
6. **Python untuk forensik (dfVFS, plaso, pytsk3)** — LO di W12, judul deck-nya "Scripting and Automation", **tanpa satu baris kode pun**.
7. **Artifact browser modern (Chrome/Firefox) dan instant messaging (WhatsApp/Telegram/Signal)** — LO di W10; Chrome dan Firefox **cuma muncul sebagai soal latihan tanpa jawaban** (W10 slide 10 dan 12).

**Slide yang harus dibuka manual** (tidak bisa diekstrak sama sekali):

| Lokasi | Isinya |
| --- | --- |
| **W02 slide 10** | **Tabel Magic Number** — kemungkinan besar tabel file signature |
| **W03 slide 8, 29** | Diagram disk vs volume; **ilustrasi terbentuknya file slack** |
| **W04 slide 11, 17, 19** | Tabel 6.1, 6.2, 6.3 — terutama **6.2: daftar logfile penting macOS** |
| **W05 slide 12, 20** | Faraday bag; **diagram proses cloning** |
| **W06 slide 16, 27** | **Tabel FHS**; **daftar logfile Linux yang penting** |
| **W07 slide 14** | Screenshot Properties Windows (ketiga timestamp) |

**Materi yang sudah usang** — tanyakan mana yang diujikan:
- **W02**: NTFS "Windows 7/Vista/XP"; macOS = HFS+ (sudah **APFS** sejak 2017)
- **W06**: "sebagian besar Linux memakai Ext3" (sudah Ext4); System V init dan runlevel (sudah **systemd**)
- **W10**: **seluruh bagian browser hanya Internet Explorer** (dipensiunkan 2022); **Xplico tidak bisa baca SSL** padahal web sekarang hampir semua HTTPS
- **W12**: Autopsy 2.4 (sudah 4.x); PyFLAG sudah tidak dikembangkan
- **W13**: kasus dan referensi dari 2011

**Kesalahan atau ketidakkonsistenan di slide** yang perlu dikonfirmasi:
- **W02**: macOS "supported versions: up to version 1" — jelas salah ketik
- **W03 slide 27**: angka **4095** (skala block) dan **511** (skala sector) dipakai berdekatan; pahami aritmetikanya, jangan hafal salah satu
- **W06 slide 32**: `dd` disebut singkatan "Data Destroyer" — tidak akurat
- **W06 slide 28**: job `at` ditulis di `/var/spool/cron`; di banyak distro sebenarnya `/var/spool/at`
- **W09 slide 15**: "Gnutella (formerly Kazaa)" — keliru, dua jaringan berbeda
- **W12 slide 8**: menulis "four relative times" lalu "three relative times" di slide yang sama; **jawab tiga** (before, after, during)
- **W12 slide 4**: kalimat terpotong — *"The Autopsy browser does not provide."*

## Terkait
- [[_Forensics]]
