---
matkul: Forensics
sks: 2
sumber: versi gampang dari rangkuman W01-W13
tags: [kuliah/forensics, ujian, ringkasan-simpel]
status: draft
diproses: 2026-09-03
---

# Computer Forensics — Ringkasan Simpel (Bahasa Gampang)

Ini versi "dijelasin ke temen" dari [[Forensics - Review dan Glosari]]. Kalau ada istilah teknis, langsung dijelasin di dalam kurung pas pertama kali muncul. Baca ini duluan kalau bingung, baru buka versi lengkapnya kalau butuh detail.

---

## W01 — Apa itu Forensik Digital

Forensik digital itu ilmu buat **nyari bukti di HP/komputer secara resmi dan bisa dipertanggungjawabkan** — bukan asal buka-buka file terus nebak-nebak.

Ada 3 tahap besar:
1. **Acquisition** (ambil) — kumpulin datanya, bikin salinan
2. **Analysis** (analisa) — periksa datanya, cari yang mencurigakan
3. **Presentation** (sajikan) — bikin laporan, jelasin temuannya

Ada istilah penting: **artifact** vs **evidence**.
- **Artifact** = jejak yang ketinggalan gara-gara ada aktivitas (misal history browser, file cache). Belum tentu itu "bukti kejahatan", bisa juga cuma jejak biasa.
- **Evidence** (bukti) = artifact yang udah dipakai di pengadilan/proses hukum.

Jadi kalau kamu nemu sesuatu, jangan langsung bilang "ini bukti!" — bilang aja "ini artifact", biar aman secara istilah.

Ancaman yang bikin forensik makin susah sekarang: **VPN** (nyembunyiin alamat internet asli), hotspot gratis di mana-mana, dan **TRIM** (fitur di SSD/flashdisk modern yang bikin file yang dihapus **beneran hilang**, susah dibalikin — beda sama HDD lama).

---

## W02 — Dasar-Dasar Teknis Komputer

Sebelum bisa jadi detektif digital, kamu harus ngerti cara komputer nyimpen data.

- **Bit** = angka 1 atau 0 doang. **Byte** = 8 bit digabung, biasanya = 1 huruf/angka.
- **Hex** (hexadecimal) = cara nulis angka biar lebih ringkas, pake 0-9 dan A-F. Biasa ditulis pake awalan `0x`.
- **ASCII** dan **Unicode** = cara komputer nerjemahin angka jadi huruf yang bisa dibaca manusia.

Poin PENTING: **nama file / ekstensi file (.jpg, .exe, dll) itu GAK BISA DIPERCAYA 100%**. Orang bisa gampang banget ganti nama `virus.exe` jadi `foto.jpg`. Yang beneran bisa dipercaya adalah **magic number** (kode rahasia di dalam file itu sendiri yang nunjukkin jenis file aslinya — kayak KTP file, gak bisa dipalsuin segampang nama file).

Tiga cara data disimpen:
- **Magnetik** (HDD lama) — pake magnet
- **Flash** (flashdisk, SSD) — pake transistor kecil, tetep nyimpen data walau gak ada listrik
- **Optical** (CD/DVD) — pake laser

Yang paling penting dihafal: **RAM itu volatile** (gampang ilang — begitu listrik mati, RAM langsung kosong), sedangkan **hard drive itu non-volatile** (data tetep ada walau komputer dimatiin). Ini penentu urutan kerja pas ambil bukti nanti.

**Sector** = kotak penyimpanan terkecil di disk, ukurannya 512 byte.

---

## W03 — Bedah Disk dan Filesystem

**Filesystem** = sistem yang ngatur gimana file disimpen dan dicari di dalam disk (kayak katalog perpustakaan).

Ada 6 lapisan dari yang paling "fisik" ke yang paling "manusiawi", **WAJIB HAFAL**:

```
Disk → Volume → File System → Data Unit → Metadata → File Name
(fisiknya) (partisinya) (aturan mainnya) (isi datanya) (data ttg data) (nama yg kamu liat)
```

- **Disk** = hardware fisiknya
- **Volume** = partisi (bagian) dari disk itu
- **Data unit** = potongan terkecil penyimpanan data (disebut **block**)
- **Metadata** = "data tentang data" — kayak kapan file dibuat, siapa yang punya, dll (bukan isi filenya, tapi info tentang filenya)
- **File name** = nama yang kamu lihat di explorer

**MBR** vs **GPT** = dua cara beda buat ngatur partisi di disk. MBR lebih lama (max 4 partisi utama, max 2 TB). GPT lebih baru (bisa sampe 128 partisi, size jauh lebih gede).

**RAID** = gabungin beberapa hard drive jadi kayak satu drive gede, biar lebih cepet atau lebih aman dari kerusakan.

**Hashing** = bikin "sidik jari digital" dari sebuah file (contoh: MD5, SHA1). Kalau isinya berubah SEDIKIT AJA, sidik jarinya beda total. Ini dipake buat **buktiin bukti digital gak diutak-atik** — hash sebelum dan sesudah harus sama persis.

**Carving** = teknik "menggali" file dari ruang kosong di disk berdasarkan pola/tanda khas di awal-akhir filenya (bukan dari catatan filesystem yang udah ilang).

4 tingkat "file yang dihapus", dari yang paling gampang balikin sampe paling susah:

| Tingkat | Gampangnya balikin |
|---|---|
| **Deleted** (baru dihapus) | Paling gampang, semua jejaknya masih nyambung |
| **Orphaned** (yatim piatu) | Masih bisa, tapi udah gak nyambung ke nama filenya |
| **Unallocated** (udah dipake ulang) | Susah, harus di-carving |
| **Overwritten** (udah ketimpa) | HAMPIR MUSTAHIL balikin lagi |

**File slack** = ruang sisa yang kebuang percuma. Misal file cuma 1 byte tapi minimal disimpen di ruang 4096 byte, sisa ruangnya (yang gak kepake) masih nyimpen JEJAK data lama dari file sebelumnya yang pernah ada di situ. Ini tempat sering nemu bukti tersembunyi!

---

## W04 — Sistem Mac (macOS)

Mac pake filesystem namanya **HFS+**.

5 "file spesial" yang nyimpen semua struktur pentingnya HFS+ (gak keliatan di Finder biasa):
1. **Allocation file** — daftar ruang mana yang kepake
2. **Catalog file** — daftar semua file & folder (kayak Master File Table di Windows)
3. **Extents overflow file** — buat file yang kepotong-potong (fragmentasi) parah
4. **Startup file** — buat proses nyalain komputer
5. **Attributes file** — nyimpen info tambahan file

**Property list / .plist** = file setting-an Mac, format-nya XML (bisa dibaca teks biasa) atau binary (harus di-convert dulu).

**launchd** = program yang ngatur apa aja yang otomatis jalan pas Mac nyala. Ini tempat pertama dicek kalau nyari malware yang "nempel" otomatis nyala terus.

**`.Trash`** = tempat sampah Mac. Sejak macOS 10.6, ada file tersembunyi `.DS_Store` yang nyimpen **lokasi asli** file sebelum dibuang — jadi walau filenya udah gak ada, kita masih bisa tau file itu tadinya ada di folder mana.

**sleepimage** = file gede (seukuran RAM komputer) yang isinya SALINAN PERSIS ISI RAM pas terakhir kali laptop di-hibernate/tidur. Ini penting banget karena RAM biasanya ilang pas mati, tapi kalau ada sleepimage, isi RAM itu "diselamatkan" ke disk — jadi masih bisa diperiksa walau laptopnya udah mati total.

---

## W05 — Cara Ambil Bukti (Data Acquisition)

Ini sesi paling penting soal **prosedur**. Intinya: **kalau caranya salah, buktinya gak bakal diterima di pengadilan**, semahal apapun buktinya.

Langkah di TKP:
1. Amankan lokasi
2. Dokumentasi (foto, video, catat kondisi alat — nyala/mati, kabel apa aja yang nyambung)
3. Kumpulin barang buktinya

Buat HP: dimasukin ke **Faraday bag** (kantong khusus yang blok sinyal wifi/seluler), biar gak ada yang bisa hapus data dari jarak jauh.

**Order of Volatility** (urutan mana yang paling cepet ilang, ambil duluan yang paling gampang ilang):

```
1. Isi CPU/register (paling cepet ilang)
2. Tabel routing, cache jaringan
3. RAM
4. File sementara / swap
5. Data di hard disk
6. Data yang ke-log di server lain
7. Data di arsip/backup (paling awet)
```

**Chain of Custody (CoC)** = formulir yang nyatet SIAPA AJA yang pernah pegang barang bukti itu, dari awal sampe ke pengadilan. Kalau ada yang bolong/gak jelas, buktinya bisa ditolak hakim.

**Forensic clone** = nyalin drive **bit-per-bit** (persis sama, sampai bagian kosongnya juga ikut disalin) — BEDA sama copy-paste biasa yang cuma nyalin file yang keliatan aja.

**Write blocker** = alat/software yang MENCEGAH data ditulis ke drive bukti pas lagi diperiksa. Wajib dipake biar bukti aslinya gak berubah sedikit pun.

---

## W06 — Sistem Linux

Filesystem Linux yang paling umum: **Ext4** (dulu Ext2 → Ext3 → Ext4).

**Inode** = "kartu identitas" tiap file di Linux, isinya: ukuran, siapa pemiliknya, dan 4 waktu penting yang disebut **MAC(D) times**:

- **M**odified = kapan ISI file terakhir diubah
- **A**ccessed = kapan file terakhir DIBUKA/dibaca
- **C**hanged = kapan "kartu identitasnya" berubah (misal permission diubah)
- **D**eleted = kapan file dihapus

Tiga file penting buat login:
- `/etc/passwd` = daftar user
- `/etc/group` = daftar grup
- `/etc/shadow` = password yang udah di-hash (diacak)

**Cron** = fitur buat jadwalin tugas otomatis yang berulang (misal "jalanin script ini tiap jam 2 pagi"). Ini **favorit banget dipake hacker** buat bikin malware-nya tetep jalan otomatis walau komputer di-restart.

Command penting: `fdisk -l` (lihat semua drive yang nyambung), `md5sum` (bikin hash buat cek keaslian bukti).

---

## W07 — Sistem Windows

Fakta penting: **nge-klik "Delete" itu SEBENERNYA GAK NGAPA-NGAPAIN ke datanya**. Yang keapus cuma "label" yang bilang "ruang ini boleh dipake lagi" — datanya sendiri masih di situ sampe ketimpa data baru.

3 mode tidur laptop:
- **Sleep** — data cuma disimpen di RAM (gampang ilang)
- **Hibernation** — SEMUA isi RAM ditulis ke hard disk (jadi lebih awet, bisa diperiksa walau laptop mati total) — filenya namanya `hiberfil.sys`
- **Hybrid sleep** — gabungan keduanya

**Registry** = database gede berisi semua setting-an Windows dan aplikasi.

Masalah **atribusi** (nentuin SIAPA yang make komputer): kita bisa tau APA yang dicari di Google, tapi susah buktiin SIAPA yang ngetiknya. Windows punya **SID** (kode unik per akun user) buat nge-link aktivitas ke akun tertentu — tapi ini nge-link ke AKUN, bukan ke ORANGNYA. Kalau passwordnya bocor atau akunnya kepake bareng-bareng, ya susah nentuin orangnya.

**Shadow copy** / **Restore Point** = "cadangan otomatis" Windows dari waktu ke waktu. Berguna banget buat liat GIMANA sebuah file berubah dari waktu ke waktu, atau nemu file yang udah dihapus.

---

## W08 — Balikin File yang Dihapus (Data Carving)

**Carving** = teknik nyari file dari ruang kosong disk pake ciri khas file itu (bukan dari catatan filesystem, karena catatannya udah ilang). Makanya carving TETEP BISA jalan walau nama filenya udah diganti atau dihapus.

Tool-tool yang dipake:
- **foremost** — cari file pake pola awal-akhir (header/footer)
- **bulk_extractor** — lebih canggih, bisa nemu nomor kartu kredit, email, URL, dll dari data mentah

**Antiforensic** = kebalikannya carving — cara-cara buat NGEHINDARIN forensik nemuin data. Tingkat kesulitannya:

```
1. Ganti nama file        <- gampang dilawan
2. Sembunyiin di folder aneh
3. Naruh file di dalam file lain  <- mulai susah
4. Enkripsi               <- paling susah
```

**Enkripsi** = ngubah data jadi "kode acak" (**cipher text**) yang cuma bisa dibaca pake **key** (kunci) yang bener. Makin panjang key-nya (**key space**), makin susah dijebol dengan cara coba-coba (**brute force**).

**Steganography** = nyembunyiin pesan RAHASIA di dalam file yang keliatannya biasa aja (misal pesan rahasia disisipin di dalam foto liburan). Beda sama enkripsi: enkripsi bikin orang TAU ada sesuatu yang disembunyiin tapi gak bisa baca; steganography bikin orang GAK SADAR ada yang disembunyiin sama sekali.

---

## W09 — Analisa File, Gambar, Video, dan Arsip (ZIP dll)

Dua langkah analisa file:
1. **Content identification** = mastiin file itu SEBENERNYA jenis apa (jangan percaya cuma dari namanya)
2. **Metadata extraction** = ambil info tersembunyi yang nempel di file itu

3 jenis metadata gambar:
- **EXIF** = info dari kamera/HP: merek kamera, tanggal foto, bahkan **koordinat GPS lokasi foto diambil**!
- **IPTC** = biasa dipake wartawan foto
- **XMP** = format universal dari Adobe, bisa dipake di banyak jenis file

**Container** vs **Codec**: bayangin **container** itu kotak makan, **codec** itu cara masakannya dimasak. File MP4 (container) bisa isinya video dimasak pake berbagai codec beda-beda.

Soal file ZIP/RAR/7z (arsip):
- Arsip bisa nyimpen **tanggal asli file** dari sebelum di-zip
- **RAR** sering dipake buat nyuri data (eksfiltrasi) karena enkripsinya kuat dan bisa dipecah jadi banyak bagian kecil

**PDF** bisa nyimpen JavaScript (kode program!) di dalamnya, makanya PDF sering dipake buat nyebar virus/malware.

---

## W10 — Jejak di Internet (Browser, Email)

Browser (khususnya Internet Explorer yang dibahas di sini, walau udah gak dipake lagi sekarang) nyimpen beberapa jenis jejak:
- **Cookie** = file kecil yang nyimpen info dari website yang dikunjungi
- **Cache** = salinan lokal dari halaman web yang pernah dibuka (jadi bisa liat isi halaman itu WALAU halamannya udah dihapus dari internet)
- **History** = daftar situs yang pernah dikunjungi

Penting: kita ngomongin **"profil user"**, bukan **"orang"**. Punya profil di komputer itu **BUKAN BERARTI** orang itu yang make komputer pas kejadian tertentu.

**Email** itu salah satu bukti digital TERBAIK karena:
- Orang nulis email dengan asumsi cuma dibaca penerima doang (jadi lebih jujur/blak-blakan)
- Email itu **nyebar ke banyak tempat sekaligus** (HP tersangka, HP penerima, server, backup) — jadi susah dihapus semuanya

3 protokol email: **SMTP** (buat KIRIM), **POP** (buat TERIMA, biasa didownload ke device), **IMAP** (buat akses email yang tetep di server, dua arah).

Trik licik: **shared email account** — bikin 1 akun email bareng-bareng, terus pesannya DITARUH DI FOLDER DRAFT (gak pernah dikirim beneran), jadi gak ada yang bisa nyadap karena emailnya emang gak pernah "jalan" lewat internet.

---

## W11 — Tool-Tool Investigasi (Judulnya Salah, Isinya Beda!)

**PENTING:** Judul aslinya "Timeline Analysis" tapi isinya ternyata soal tool-tool lain, bukan soal timeline sama sekali. Aneh tapi ya udah, ini isi aslinya:

- **p0f** = tool buat nebak sistem operasi target secara DIAM-DIAM (pasif, gak ngirim apa-apa, cuma "nguping")
- **Nmap** = tool buat scan jaringan secara AKTIF (ngirim paket, liat port mana yang kebuka/ketutup)
- **Linux Explorer** = tool buat periksa Linux yang lagi NYALA (live forensic), bisa cari proses mencurigakan, deteksi rootkit (malware yang nyembunyiin diri di level sistem)
- **Credential dumping** = teknik ngambil password/login yang lagi "nyangkut" di memori komputer
- **mimipenguin** = tool buat "meres" password plaintext (belum dienkripsi) dari memori Linux — versi Linux-nya tool terkenal **mimikatz** (buat Windows)

---

## W12 — Otomasi dan Konsep Timeline

**Autopsy** = software GUI (ada tampilan visualnya, gak cuma teks) buat bantu analisa forensik, gratis dan udah include di Kali Linux.

Kenapa perlu OTOMASI: karena kalau manual, banyak banget kerjaan "persiapan" yang membosankan sebelum analisa beneran bisa dimulai (cari partisi, cari filesystem, dll).

**Timeline** = urutan waktu kejadian, disusun dari berbagai sumber waktu di komputer. 3 jenis hubungan waktu:
- **Before** (sebelum)
- **After** (sesudah)
- **During** (selama — ada durasi/rentang waktu, misal "berapa lama hacker ada di sistem")

Aturan penting: **waktu paling tua yang mungkin ada di sebuah file = waktu filesystem-nya dibuat**. Kalau ada timestamp yang LEBIH TUA dari itu, berarti PALSU (atau file itu dipindahin dari tempat lain yang bawa timestamp aslinya).

**Inferred time** = waktu yang "ditebak/disimpulkan" karena data aslinya udah gak ada (biasa buat data yang udah dihapus).

**Embedded time** = info waktu yang nempel DI DALAM ISI file. Contoh: kalau PDF dibuat pake software versi tertentu, berarti file itu GAK MUNGKIN dibuat sebelum software itu rilis.

**Periodicity** = jarak waktu antar kejadian yang berulang. Berguna buat nemuin **malware backdoor** yang biasanya "lapor" ke servernya secara teratur banget (misal tiap 5 menit persis) — tapi HATI-HATI, program update otomatis biasa juga punya pola kayak gitu.

---

## W13 — Analisa Jaringan (Network)

Prinsip dasar: siapin diri buat **"KAPAN" ada serangan, bukan "KALAU" ada serangan**. Realistis aja, gak ada sistem yang 100% kebal.

**Firewall** = "satpam" yang nyaring trafik internet masuk-keluar, nentuin mana yang boleh lewat.

**IDS/NIDS** (Intrusion Detection System) = sistem yang MENDETEKSI kalau ada yang mencurigakan (bukan mencegah, cuma kasih alarm). Contohnya **Snort**.

4 jenis serangan yang dibahas:
- **DDoS** = banyak komputer nyerang bareng-bareng satu target sampe down
- **IP Spoofing** = mekingin alamat IP asli
- **Man-in-the-Middle** = penyerang "nyempil" di tengah komunikasi dua orang, nyadap tanpa ketauan
- **Social Engineering** = nipu ORANGNYA langsung (bukan nyerang sistemnya) buat kasih info rahasia

**Ancaman dari DALAM organisasi** justru lebih bahaya karena bisa lewatin banyak pengamanan yang udah dipasang (soalnya dia emang udah punya akses resmi).

4 tahap kalau ada insiden keamanan:
```
1. Preparation (persiapan sebelum kejadian)
2. Detection and Analysis (deteksi & analisa pas kejadian)
3. Containment, Eradication and Recovery (kurung, bersihin, pulihkan)
4. Post Incident Activity (evaluasi setelah kejadian)
```

**Wireshark** = tool buat "nyadap" dan liat isi paket data yang lewat di jaringan.

**PCAP** = format file hasil rekaman paket jaringan (kayak rekaman CCTV tapi buat trafik internet).

---

## Kesimpulan Besar (Benang Merah Semua Materi)

Kalau capek baca semua di atas, ini inti dari SELURUH matkul dalam beberapa kalimat:

1. **Prosedur itu segalanya.** Bukti paling keren pun GAK BERGUNA kalau cara ngambilnya salah atau gak tercatat rapi.
2. **"Dihapus" gak selalu beneran hilang.** Data yang "dihapus" biasanya cuma label-nya doang yang ilang, isinya masih ada sampe ketimpa data baru.
3. **Jangan percaya nama file.** Selalu cek ISI-nya (magic number), bukan cuma namanya.
4. **Yang paling cepet ilang, ambil duluan.** RAM sebelum hard disk, hard disk sebelum arsip lama.
5. **Kita nemuin AKUN yang dipake, bukan ORANG yang pake.** Jangan gegabah nuduh orang cuma dari jejak digital doang.

---

## Terkait
- [[_Forensics]] — daftar lengkap semua pertemuan
- [[Forensics - Review dan Glosari]] — versi lengkap & detail (buat belajar lebih dalam)
