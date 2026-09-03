---
matkul: Forensics
minggu: 10
sks: 2
sumber: 10 Internet Artifacts.pptx
tags: [kuliah/forensics, minggu/w10]
status: draft
diproses: 2026-09-03
---

# W10 — Internet Artifacts

## Ringkasan
> - **Tidak ada yang menunjukkan konsep *evidence dynamics* lebih baik daripada internet artifact.** Tiap klik link, tiap bookmark, tiap query pencarian meninggalkan jejak.
> - Artifact browser Internet Explorer: **Cookies** (file teks polos terpisah per host — bisa diperiksa langsung), **INDEX.DAT** (biner, melacak URL yang dikunjungi dan jumlah kunjungan), **Cache** (Temporary Internet Files), **NTUSER.DAT** (file registry per profil user, memuat browser history).
> - Peringatan yang diulang: **kita bicara soal "profil", bukan "orang"** — punya profil di mesin bukan berarti jarinya ada di keyboard.
> - **Email adalah salah satu sumber bukti digital terbaik**, karena orang menulis dengan asumsi tidak akan dibaca siapa pun selain penerimanya, dan karena **email persisten dan berada di banyak lokasi**.
> - Tiga protokol email: **SMTP** (kirim), **POP** (terima), **IMAP** (dua arah, akses di server).
> - **Shared email account** — pesan ditaruh di folder **Drafts** tanpa pernah dikirim, supaya tidak ada yang bisa dicegat. Praktik populer di kalangan teroris.
> - **PST** = format penyimpanan mail Outlook; **OST** = versi offline-nya.
> - **Xplico** = NFAT open source berbasis GUI untuk mengekstrak artifact dari capture jaringan. **Tidak bisa membaca trafik terenkripsi SSL.**

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Evidence dynamics | Konsep bahwa bukti terus berubah/terbentuk seiring aktivitas |
| URL | Uniform Resource Locator |
| Cookie | File yang disimpan browser; di IE berupa file teks polos terpisah per host |
| INDEX.DAT | File biner container milik Internet Explorer; melacak URL, jumlah kunjungan, dan isi tiap direktori MSIE |
| NTUSER.DAT | File registry per profil user; berisi preferensi dan **browser history** |
| Cache | File yang tersimpan lokal akibat aktivitas browsing |
| SMTP | Simple Mail Transfer Protocol — **mengirim** |
| POP | Post Office Protocol — **menerima** |
| IMAP | Internet Message Access Protocol — **dua arah**, akses email di server |
| Shared e-mail account | Teknik komunikasi lewat folder Drafts tanpa pernah mengirim pesan |
| PST | Personal Storage Table, format penyimpanan mail Outlook |
| OST | Offline Storage Table, penyimpanan email offline Outlook |
| SNS | Social Networking Sites |
| Subpoena / search warrant | Instrumen hukum untuk mendapat data dari penyedia layanan |
| Xplico | Network Forensics Analysis Tool (NFAT) open source berbasis GUI |
| NFAT | Network Forensics Analysis Tool |
| RTP / VoIP | Real-time Transport Protocol / Voice over IP |

## Isi

### Internet artifact dan evidence dynamics
Slide membuka dengan klaim yang kuat:

> Bisa dibilang **tidak ada yang menunjukkan konsep *evidence dynamics* lebih baik daripada internet artifact**. Di sistem komputer pengguna akhir modern, **sebagian besar interaksi user dengan sistem kemungkinan besar terkait komunikasi internet** dalam bentuk apa pun. **Tiap klik pada sebuah link, tiap bookmark, dan tiap query pencarian bisa meninggalkan jejak yang menceritakan sesuatu** di sistem user.

Semuanya dimulai ketika seseorang **memasukkan alamat web atau URL** ke address bar browser.

> [!info] Konteks tambahan (bukan dari slide)
> *Evidence dynamics* adalah istilah forensik untuk kenyataan bahwa **bukti tidak diam** — dia terus dibuat, diubah, dan dihancurkan oleh aktivitas normal. Internet artifact contoh ekstremnya: cache diisi ulang, history dibatasi jumlahnya, cookie kedaluwarsa. Konsekuensi praktisnya menyambung langsung ke [[W05 - Data Acquisition]]: **makin lama kamu menunggu, makin banyak yang hilang** — dan itu berlaku bahkan tanpa ada yang sengaja menghapus.

### Artifact browser
Kalau sebagian besar waktu pengguna komputer dihabiskan di internet, maka kemungkinan **hampir semua waktu itu dihabiskan berinteraksi dengan web browser**.

**Cookies (Internet Explorer).** Lokasinya berubah antar versi Windows:

| Versi Windows | Lokasi cookie IE |
| --- | --- |
| **Windows XP** | `Documents and Settings\%username%\Cookies` |
| **Vista dan Windows 7** | `Users\%username%\AppData\Roaming\Microsoft\Windows\Cookies` |
| **Windows 10 sebelum 1709** (Fall Creators Update) | `C:\Users\<Nama User>\AppData\Local\Microsoft\Windows\INetCookies` |
| **Windows 10 versi terbaru** | **Tidak ada file cookie lagi** |

Yang menguntungkan examiner: **Internet Explorer menyimpan cookie user sebagai file teks polos yang terpisah per host penerbit**, sehingga **bisa diperiksa langsung**.

**INDEX.DAT.** File **biner** yang mirip container, dipakai Microsoft Internet Explorer (MSIE). Slide bilang file ini **punya nilai yang cukup besar bagi forensic examiner**.

- **Ada banyak file INDEX.DAT** di satu sistem
- INDEX.DAT **melacak beberapa informasi mengenai URL yang dikunjungi, jumlah kunjungan**, dan sebagainya
- File ini **disembunyikan dari user** dan **harus dilihat memakai tool tertentu** — **FTK dan EnCase** bisa menguraikannya
- MSIE punya **tiga direktori**: **History, Cookies, dan Temporary Internet Files**. **File INDEX.DAT dipakai untuk melacak informasi dan isi tiap direktori itu** (Casey, 2009)

**NTUSER.DAT.** Berisi **pengaturan preferensi dan informasi individual untuk tiap profil user** — dan **browser history adalah bagian dari informasi itu**. **Ada satu NTUSER.DAT untuk tiap profil user** di sistem. Meskipun secara teknis ini file registry, **NTUSER.DAT terletak di folder user**.

Lalu slide mengulang peringatan atribusi dari [[W07 - Windows System Artifacts]]:

> Perhatikan bahwa kita bicara soal **"profil" user, bukan "user"**. **Menempatkan orang tertentu di depan keyboard adalah penentuan yang sangat sulit, kalau bukan mustahil.** Cuma karena seseorang punya profil di mesin itu, **tidak berarti jarinya ada di keyboard pada momen tertentu**.

**Cache.** Berisi file yang disimpan (cached) secara lokal di sistem sebagai hasil aktivitas browsing user:

| Versi Windows | Lokasi cache IE |
| --- | --- |
| **XP** | `Documents and Settings\%username%\Local Settings\Temporary Internet Files\Content.IE5` |
| **Vista dan Windows 7** | `Users\%username%\AppData\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5` |
| **Windows 10** | `Users\%username%\AppData\Local\Microsoft\Windows\INetCache` |

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa cache berharga melebihi history: **history memberitahumu bahwa user mengunjungi sebuah halaman; cache memberitahumu apa yang dia lihat di halaman itu.** Kalau halamannya sudah dihapus dari internet, atau isinya berubah, salinan cache di mesin tersangka bisa jadi satu-satunya bukti seperti apa halaman itu **pada saat dia melihatnya**.

### Latihan dari slide
Deck ini memberi tiga pertanyaan latihan yang **jawabannya tidak ada di slide** — semuanya soal browser non-Microsoft:

> **Slide 10:** Di mana lokasi cookie Mozilla dan Chrome?
>
> **Slide 12:** Di mana lokasi cache Mozilla dan Chrome? Bagaimana cara melihat password tersimpan di Mozilla dan Chrome?

### Email sebagai bukti
Untuk pengguna rumahan, penyimpanan email lokal mungkin mulai ditinggalkan demi webmail, **tapi masih banyak bisnis yang memakai mail tersimpan lokal**.

Slide menjelaskan kenapa email begitu berharga:

> Dari semua sumber potensial bukti digital, **email adalah salah satu yang terbaik**. Orang sering menyusun dan mengirim email **dengan asumsi tidak akan pernah dibaca siapa pun selain penerima yang dituju**. Karena itu, **pertukaran yang blak-blakan ini bisa (dan sudah pernah) berbalik menghantui pihak-pihak yang terlibat**. Email juga **persisten, berada di banyak lokasi sekaligus**, sehingga **lebih sulit dihilangkan**.

**Tiga protokol email:**

| Protokol | Kepanjangan | Fungsinya |
| --- | --- | --- |
| **SMTP** | Simple Mail Transfer Protocol | Dipakai klien email untuk **mengirim**, dan oleh server untuk **mengirim sekaligus menerima** |
| **POP** | Post Office Protocol | Dipakai klien email untuk **menerima** pesan |
| **IMAP** | Internet Message Access Protocol | Protokol komunikasi **dua arah**, dipakai klien untuk **mengakses email di server** |

**Apa yang bisa didapat dari email sebagai bukti:**
- **Komunikasi yang relevan dengan kasus**
- **Alamat email**
- **Alamat IP**
- **Tanggal dan waktu**

Dan yang penting untuk strategi investigasi — **email bisa ditemukan di banyak tempat**: mesin tersangka, **mesin penerima mana pun**, server perusahaan atau media backup, smartphone, penyedia layanan, dan **server mana pun yang mungkin dilewati pesan itu** dalam perjalanan ke tujuan akhirnya.

> [!info] Konteks tambahan (bukan dari slide)
> Daftar "banyak tempat" itu sebenarnya **keunggulan taktis**: tersangka bisa menghapus email dari mesinnya sendiri, tapi dia **tidak bisa menghapusnya dari mesin penerima, dari server perusahaan, atau dari backup**. Ini contoh bagus bahwa dalam forensik, **redundansi adalah temanmu** — dan mengapa "sudah saya hapus" hampir tidak pernah berarti "sudah hilang".

### Shared e-mail account
Teknik ini layak dipahami karena melawan asumsi dasar penyadapan:

> **Email bisa dipakai berkomunikasi bahkan tanpa pernah dikirim.** Caranya dengan membuat akun anonim — Yahoo!, misalnya — lalu **membagikan informasi login-nya**. User cukup **membuat pesan dan menaruhnya di folder "Drafts"** untuk dibaca orang lain. Begitu pesannya dibaca, bisa dihapus. **Akun-akun ini bisa untuk sekali pakai, sehingga nyaris mustahil dilacak atau dipantau.** Ini praktik yang populer di kalangan teroris.

**Richard Clarke**, mantan counterterrorism czar AS: *"Akun anonim sekali pakai sangat sulit dipantau."*

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa ini efektif: penyadapan jaringan mengintai **pesan yang bergerak**. Kalau pesannya tidak pernah dikirim, **tidak ada yang bergerak untuk dicegat** — datanya cuma duduk di server Yahoo. Ini juga menjelaskan kenapa artifact browser (bagian sebelumnya) jadi penting untuk kasus semacam ini: yang tersisa bukan email, melainkan **jejak bahwa seseorang login ke akun itu**.

### Social Networking Sites
Bukti media sosial bisa ditemukan di beberapa tempat: **komputer tersangka, smartphone, dan jaringan penyedia layanan**.

Mendapat bukti dari penyedia layanan butuh **tindakan yang relatif cepat** plus **subpoena atau surat perintah penggeledahan**. Slide mengingatkan:

> **Penyedia layanan hanya menyimpan informasi ini untuk jangka waktu tertentu.** Pada satu titik, data yang kamu butuhkan **akan dibuang tanpa intervensi hukum**. Dengan mempertimbangkan semuanya, **mengumpulkan bukti dari penyedia layanan mungkin memberi hasil terbaik**.

### Personal Storage Table (PST)
**PST adalah format penyimpanan mail yang dipakai klien email Microsoft Outlook.**

File PST user **bukan hanya untuk menyimpan email dari server MS Exchange**, tapi juga bisa menyimpan email dari akun **POP3, IMAP, dan bahkan HTTP** (seperti Windows Live Hotmail). File PST menyediakan format penyimpanan data untuk email di sistem komputer user.

Pengguna Outlook juga bisa punya file **OST**, yaitu untuk **penyimpanan email offline**. File ini memungkinkan user **terus membaca email yang sudah ada meskipun sedang offline** dan tidak bisa terhubung ke server MS Exchange.

### Packet analysis
Packet capture yang bisa diselidiki mencakup protokol: **TCP, UDP, HTTP, FTP, TFTP, SIP, POP, IMAP, dan SMTP**.

Data yang terkandung dalam capture paket jaringan dan internet — **bahkan akuisisi live** — bisa berisi artifact seperti:
- **Trafik HTTP**, misalnya situs web yang diramban
- **Email**
- **Chat Facebook**
- **RTP (Real-time Transport Protocol) dan VoIP**
- **File yang dicetak**

**Xplico** adalah **Network Forensics Analysis Tool (NFAT)** yang **open source dan berbasis GUI**, berfokus pada **mengekstrak artifact dari capture jaringan dan internet**.

Batasan pentingnya:

> **Trafik yang dienkripsi memakai SSL saat ini tidak bisa dilihat dengan Xplico.**

Analisis paket yang bisa dilakukan Xplico: **HTTP and Web Analysis**, **VoIP Analysis**, dan **Email analysis**.

> [!info] Konteks tambahan (bukan dari slide)
> Batasan SSL itu bukan detail kecil — itu **membatasi seluruh bab ini** untuk penggunaan modern. Ketika deck ini ditulis, sebagian besar web masih HTTP polos. Sekarang **hampir semua trafik dienkripsi HTTPS**, jadi daftar "trafik HTTP, email, chat Facebook" itu sebagian besar **tidak lagi bisa dibaca dari capture**. Yang tersisa dari capture modern umumnya cuma **metadata**: siapa bicara dengan siapa, kapan, dan berapa banyak. Ini pertanyaan bagus untuk diajukan ke dosen.

### Tugas dari slide
> **Slide 23 — Assignment:** Lakukan analisis memakai **XPLICO** untuk **HTTP and Web Analysis**, **VoIP Analysis**, dan **Email analysis**.
>
> *Catatan slide: panduan ada di E-book Chapter 10 (Parasram, Digital Forensics with Kali Linux).*

## Diagram & Visual
> [!warning] Deck ini **tidak punya satu pun gambar non-dekoratif** dari 25 slide. Tidak ada screenshot Xplico, tidak ada tampilan INDEX.DAT, tidak ada contoh header email. Seluruh isinya teks — jadi note ini sudah mewakili keseluruhan PPT-nya, tapi untuk tugas Xplico harus mengandalkan e-book.

## Rumus / Sintaks

Lokasi artifact Internet Explorer per versi Windows:
```
COOKIES
  XP        : Documents and Settings\%username%\Cookies
  Vista/7   : Users\%username%\AppData\Roaming\Microsoft\Windows\Cookies
  Win10<1709: Users\<user>\AppData\Local\Microsoft\Windows\INetCookies
  Win10 baru: sudah tidak ada file cookie

CACHE
  XP        : Documents and Settings\%username%\Local Settings\Temporary Internet Files\Content.IE5
  Vista/7   : Users\%username%\AppData\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5
  Win10     : Users\%username%\AppData\Local\Microsoft\Windows\INetCache

INDEX.DAT  : biner, banyak file, lacak URL + jumlah kunjungan (baca dengan FTK/EnCase)
NTUSER.DAT : registry per profil user, memuat browser history
```

Protokol email:
```
SMTP -> KIRIM         (klien kirim; server kirim & terima)
POP  -> TERIMA        (klien menerima pesan)
IMAP -> DUA ARAH      (klien mengakses email di server)
```

## Pertanyaan Terbuka
- Slide 3 mencantumkan **sembilan learning outcome**, dan setidaknya **empat tidak dibahas**: **Modern Browser Artifacts (Profiles, Cookies, Autofill, Sync Data)**, **Instant Messaging Artifacts (WhatsApp, Telegram, Signal, Skype)**, **Session Reconstruction from Packet Analysis**, dan **Credential Artefacts and Tokens**. Bagian **instant messaging** khususnya menonjol — itu tempat komunikasi sungguhan terjadi sekarang, dan **tidak dibahas sama sekali**. **Wajib ditanyakan.**
- **Seluruh bagian browser hanya membahas Internet Explorer**, browser yang sudah dipensiunkan Microsoft pada 2022. **Chrome dan Firefox cuma muncul sebagai soal latihan tanpa jawaban** (slide 10 dan 12). Padahal LO-nya menyebut "Modern Browser Artifacts". Ini kesenjangan terbesar deck ini.
- **Tiga soal latihan di slide 10 dan 12 tidak punya jawaban di deck mana pun.** Kalau ini keluar di ujian, jawabannya harus dicari sendiri.
- **"Extract Information from Email"** jadi LO, tapi **tidak ada satu pun teknik** yang ditunjukkan — tidak ada pembacaan header email, tidak ada penelusuran `Received:` untuk melacak jalur pesan, tidak ada tool untuk membuka PST.
- **Xplico tidak bisa membaca SSL**, dan hari ini hampir semua trafik SSL/TLS. Perlu ditanya apakah ada tool pengganti yang dipakai di praktikum, atau apakah materi capture ini memang diajarkan sebagai historis.
- **Evidence dynamics** dipakai sebagai konsep pembuka tapi **tidak pernah didefinisikan** di deck mana pun di matkul ini.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W07 - Windows System Artifacts]]
- [[W09 - File and Archives Analysis]]
- [[W11 - Timeline Analysis and Correlation of Artifacts]]
- [[W13 - Network Analysis]]
- [[Forensics - Review dan Glosari]]
