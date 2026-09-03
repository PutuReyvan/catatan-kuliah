---
matkul: Forensics
sks: 2
dosen: Ika Dyah A.R.
jadwal: Senin (2)
tags: [kuliah/forensics, moc]
status: draft
diproses: 2026-09-03
---

# Senin (2) — Computer Forensics

Index matkul. Tiga belas deck PPT dari dosen, diproses jadi tiga belas note pertemuan.

**Penomoran minggu di matkul ini pasti**, bukan dugaan — tiap deck menulis **"SESSION N"** di slide judulnya, dan 13 deck itu pas mengisi 13 pertemuan.

## Daftar Pertemuan

| # | Note | Isinya |
| --- | --- | --- |
| W01 | [[W01 - Digital Forensic Fundamental]] | Definisi digital forensic, 8 tahap, artifact vs evidence, proses Acquisition→Analysis→Presentation, ancaman terhadap forensik |
| W02 | [[W02 - Key Technical Concepts]] | Bit/byte/hex, ASCII vs Unicode, file extension vs magic number, magnetik/flash/optical, volatilitas, filesystem, sector 512 byte |
| W03 | [[W03 - Disk and File System Analysis]] | File System Abstraction Model, MBR vs GPT, RAID, hashing, carving, forensic imaging, 4 kategori deleted data, file slack, dd/dcfldd/dc3dd |
| W04 | [[W04 - Mac OS X System and Artifacts]] | HFS+, volume header, 5 special file, plist, launchd, kext, home directory, `.Trash`, `sleepimage` |
| W05 | [[W05 - Data Acquisition]] | Dokumentasi TKP, Order of Volatility, Chain of Custody, Faraday bag, cloning, write blocking, live vs dead system, bukti di RAM |
| W06 | [[W06 - Linux System and Artifacts]] | Ext2/3/4, superblock, inode, MAC(D) times, hard link, boot process, `/etc/passwd`-`group`-`shadow`, utmp/wtmp, cron, fdisk, dc3dd |
| W07 | [[W07 - Windows System Artifacts]] | Deleted data, sleep vs hibernation vs hybrid, Registry, masalah atribusi dan SID, metadata, MRU, restore point dan shadow copy, FTK Imager |
| W08 | [[W08 - File Recovery and Data Carving]] | Konsep carving, foremost/recoverjpeg/bulk_extractor, antiforensic, enkripsi, key space, steganography, drive wiping |
| W09 | [[W09 - File and Archives Analysis]] | Content identification vs metadata extraction, EXIF/IPTC/XMP, container vs codec, format audio-video, ZIP/RAR/7z/tar, RTF dan PDF |
| W10 | [[W10 - Internet Artifacts]] | Artifact browser IE (cookies, INDEX.DAT, NTUSER.DAT, cache), email sebagai bukti, SMTP/POP/IMAP, shared account, PST/OST, Xplico |
| W11 | [[W11 - Timeline Analysis and Correlation of Artifacts]] | p0f, Nmap, Linux Explorer, credential dumping, mimipenguin *(isi deck tidak sesuai judulnya)* |
| W12 | [[W12 - Scripting and Automation for Forensic Tasks]] | Autopsy, PyFLAG, tujuan otomasi, relative time (before/after/during), inferred dan embedded time, periodicity |
| W13 | [[W13 - Network Analysis]] | Social engineering, dasar jaringan, firewall dan NIDS, 4 serangan, inside threat, 4 tahap incident response, Wireshark, PCAP, PcapXray |

## Rangkuman Ujian
- [[Forensics - Review dan Glosari]] — review lintas pertemuan + glosari lengkap

## Peta Materi

Matkul ini punya alur yang jelas dalam empat blok:

**Blok 1 — Fondasi (W01–W03): "apa itu forensik, dan apa yang ada di dalam disk"**
W01 memberi kerangka proseduralnya (delapan tahap, tiga fase, artifact vs evidence). W02 memberi pengetahuan teknis dasar yang jadi prasyarat semuanya (bit, byte, magic number, volatilitas). W03 menyatukan keduanya lewat **File System Abstraction Model** — model enam lapisan yang jadi tulang punggung seluruh semester.

**Blok 2 — Per sistem operasi (W04, W06, W07) + akuisisi (W05): "di mana artifact tinggal"**
Tiga deck OS yang paralel: macOS (W04), Linux (W06), Windows (W07). Ketiganya menjawab pertanyaan yang sama untuk sistem berbeda — di mana user account disimpan, di mana log, di mana artifact aktivitas user, bagaimana startup diatur. W05 menyela di antaranya karena akuisisi harus dilakukan **sebelum** kamu bisa memeriksa apa pun.

**Blok 3 — Per jenis data (W08–W10): "cara menggali isinya"**
W08 memulihkan yang sudah dihapus (carving) dan membahas lawannya (antiforensic). W09 membedah file yang utuh (metadata gambar, container media, arsip, dokumen). W10 mengurus jejak internet (browser, email, paket).

**Blok 4 — Menggabungkan (W11–W13): "menyusun jadi cerita"**
W11 (tool artifact), W12 (otomasi dan konsep timeline), W13 (jaringan). Blok ini seharusnya jadi puncak matkul, dan **justru blok yang paling banyak lubang materinya** — lihat catatan di bawah.

**Benang merah yang menembus semua blok:**

- **File System Abstraction Model** (W03) — dipakai untuk menjelaskan special file HFS+ (W04), lapisan Ext (W06), empat kategori deleted data (W03), dan tampilan Autopsy (W12).
- **"Jangan percaya nama file"** — magic number diperkenalkan di W02, dipakai untuk carving di W08, dipakai untuk verifikasi di W09. Diulang tiga kali dengan kata-kata berbeda.
- **Volatilitas** — dibedakan di W02, jadi Order of Volatility di W05, jadi alasan `sleepimage` berharga di W04 dan `hiberfil.sys` di W07, dan jadi tempat mimipenguin bekerja di W11.
- **Timestamp** — MAC(D) times Linux di W06, Created/Modified/Accessed Windows di W07, EXIF di W09, lalu semuanya jadi bahan mentah timeline di W12.
- **Masalah atribusi** — dinyatakan di W07 ("kita jarang bisa menempelkan jari seseorang ke keyboard"), diulang di W10 ("kita bicara profil, bukan user"). Ini batas jujur dari seluruh disiplin ini.
- **Hashing** — verifikasi integritas di W03, dipakai praktis dengan `md5sum` di W06, jadi tugas pemrograman di W03 slide 18, dan jadi fitur Autopsy plus pencocokan NSRL di W12.
- **Prosedur mengalahkan teknik** — W01 ("tanpa dikumpulkan dengan benar tidak akan sampai ke juri"), W05 ("tiga jam dokumen itulah yang membawa buktimu ke pengadilan", *"if you don't write it down, it didn't happen"*).

## Konsep Utama
Belum ada note atomik di `02-Konsep/`. Kandidat terkuat kalau nanti mau dibuat (konsep yang muncul di lebih dari satu pertemuan):

- **File System Abstraction Model** — W03, W04, W06, W12
- **Volatilitas dan Order of Volatility** — W02, W04, W05, W07, W11
- **Magic number / file signature** — W02, W08, W09
- **MAC times dan timestamp** — W06, W07, W09, W12
- **Hashing untuk integritas bukti** — W03, W05, W06, W12
- **Chain of Custody** — W05 (dan disebut sebagai LO di W01)
- **Masalah atribusi (SID vs orang)** — W07, W10

## Catatan Pemrosesan

**Ini matkul dengan kesenjangan learning outcome terbesar sejauh ini.** Tiap deck membuka dengan daftar LO di slide 3, dan **di hampir semua deck, sebagian besar LO itu tidak pernah dibahas** di slide mana pun. Rekapnya:

| Deck | LO yang tidak dibahas |
| --- | --- |
| W01 | Chain of custody, types of digital evidence & sources, digital forensic readiness |
| W02 | Metadata in filesystems, slack space (baru muncul di W03) |
| W03 | SSD vs HDD forensics, logical vs physical imaging, common disk errors |
| W04 | **APFS**, Unified Logging System, Time Machine, **FileVault** |
| W05 | **Handling damaged or encrypted media**, memory acquisition (cuma nama tool) |
| W06 | **systemd journal**, artefak malware & persistence, XFS |
| W07 | **Registry analysis, hiberfil.sys, Prefetch, Event Logs, Jump Lists/RecentDocs, .lnk files** ← enam sekaligus |
| W08 | Fragmented carving, validation of carved files, **TRIM/SSD**, PhotoRec |
| W09 | Audio/video forensic sebagai teknik, steganography detection, file tampering detection |
| W10 | **Modern browser artifacts**, **instant messaging (WhatsApp/Telegram/Signal)**, session reconstruction, credential tokens |
| W11 | **Timeline reconstruction** ← *judul deck-nya sendiri*, anti-forensic detection, volatile artifacts, IoC, reporting |
| W12 | **Python untuk forensik, dfVFS/plaso/pytsk3**, batch processing, automated timeline reports, automation pitfalls |
| W13 | Protocol analysis, session reconstruction, IoC extraction, encrypted traffic, **korelasi network dengan host** |

**Tiga topik yang jadi LO berulang tapi tidak pernah dibahas sama sekali:**
1. **Timeline reconstruction** — muncul di LO 3 seluruh matkul, jadi judul W11, tapi konsepnya cuma disinggung di W12 dan **tekniknya tidak pernah diajarkan** (tidak ada `mactime`, `log2timeline`/`plaso`, atau super timeline).
2. **IoC (Indicators of Compromise)** — LO di W11 dan W13, **tidak pernah didefinisikan**.
3. **Reporting** — bagian dari LO 3 dan tahap ke-7 di W01, tapi **tidak ada satu deck pun** yang mengajarkan cara menulis laporan forensik.

**Materi yang sudah usang di slide.** Deck ini jelas sudah lama tidak diperbarui (banyak referensi dari 2011). Perlu dikonfirmasi mana yang diujikan:
- W02: NTFS digambarkan untuk "Windows 7, Vista, XP"; macOS disebut HFS+ (padahal sudah APFS sejak 2017)
- W06: "sebagian besar Linux memakai Ext3" (sudah Ext4 sejak ~2010); System V init dan runlevel (sudah systemd)
- W10: **seluruh bagian browser hanya Internet Explorer**, yang sudah dipensiunkan 2022; Xplico tidak bisa membaca SSL, padahal hampir semua trafik sekarang HTTPS
- W12: Autopsy 2.4 (sudah versi 4.x); PyFLAG sudah tidak dikembangkan
- W13: contoh kasus dan referensi dari 2011

**Format gambar.** Enam gambar di deck W02, W03, dan W07 berformat **WMF** (vector clipart lama) yang **tidak bisa ditampilkan Obsidian**, terutama di Android, jadi tidak diikutkan ke vault. Yang paling merugikan: **W02 slide 10 berjudul "Magic Number" dan isinya cuma gambar** — kemungkinan besar tabel file signature.

**Slide yang perlu dibuka manual** (isinya tidak bisa diekstrak sama sekali):
- **W02 slide 10** — tabel Magic Number
- **W03 slide 8** (diagram disk vs volume) dan **slide 29** (ilustrasi file slack)
- **W04 slide 11, 17, 19** — Tabel 6.1, 6.2, 6.3 (terutama **6.2: daftar logfile penting macOS**)
- **W05 slide 12** (Faraday bag) dan **slide 20** (diagram cloning)
- **W06 slide 16** (tabel FHS) dan **slide 27** (daftar logfile Linux)
- **W07 slide 14** — screenshot Properties Windows
