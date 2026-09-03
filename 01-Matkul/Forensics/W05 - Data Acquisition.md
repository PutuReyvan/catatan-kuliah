---
matkul: Forensics
minggu: 5
sks: 2
sumber: 5 Data Acquistion.pptx
tags: [kuliah/forensics, minggu/w05]
status: draft
diproses: 2026-09-03
---

# W05 — Data Acquisition

## Ringkasan
> - Kalimat pembuka yang merangkum seluruh sesi: **"smoking gun" yang kamu temukan tidak akan pernah sampai ke juri kalau tidak dikumpulkan dan dipertanggungjawabkan dengan benar sejak dari TKP.** Tiga jam kerja dokumentasi yang membosankan itulah yang membawa buktimu ke pengadilan.
> - **Order of Volatility** — 7 tingkat, dari CPU/cache/register sampai archival media. Ambil yang paling volatile duluan.
> - **Chain of Custody (CoC)** = formulir yang secara hukum menjamin integritas bukti saat berpindah tangan, lengkap dengan identitas tiap pihak.
> - **Live vs dead system**: boot, reboot, atau shutdown **bisa menulis ke hard drive** dan menimpa file terhapus di unallocated space.
> - **Forensic clone** = salinan bit-per-bit. Copy-paste biasa **tidak dapat**: unallocated space, file terhapus/tertimpa sebagian, dan data filesystem.
> - **Write blocker** (hardware maupun software) mencegah penulisan ke media bukti — ini satu-satunya penjaga admissibility saat cloning.
> - RAM bisa berisi **password plaintext, data tak terenkripsi, proses berjalan, IM, alamat IP, dan Trojan**.
> - Standar acuan: **SWGDE Best Practices for Digital Evidence Collection v1.0 (Juli 2018)**.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| First responder | Pihak pertama di TKP; melakukan dokumentasi awal |
| Order of volatility | Urutan prioritas pengambilan bukti, dari yang paling mudah hilang |
| Chain of Custody (CoC) | Formulir yang menjamin integritas bukti selama berpindah tangan |
| Faraday bag | Wadah yang melindungi ponsel dari sinyal wireless; alternatifnya kaleng cat kosong |
| Live system | Perangkat dalam keadaan menyala |
| Dead system | Perangkat dalam keadaan mati |
| Forensic clone | Salinan hard drive bit-per-bit; disebut juga **bit stream image** |
| Forensically clean media | Media tujuan yang sudah dipastikan bersih untuk menampung clone |
| Write blocker | Perangkat/software yang mencegah penulisan ke media bukti |
| Physical image | Salinan tiap bit data persis seperti aslinya |
| Guymager / dc3dd | Tool imaging di Kali Linux |
| CAINE / Helix | Tool akuisisi live untuk RAM dan paging file |
| Airplane mode | Mode untuk mencegah koneksi lanjutan pada perangkat mobile |
| SWGDE | Scientific Working Group on Digital Evidence; penerbit best practice |

## Isi

### Kenapa prosedur pengumpulan itu segalanya
Slide membuka dengan kalimat yang layak dikutip utuh:

> **"Smoking gun" yang kamu temukan tidak akan pernah sampai ke juri kecuali dia dikumpulkan dan dipertanggungjawabkan dengan benar, dimulai dari TKP.** Sepenting apa pun itu, kamu tidak akan pernah melihatnya dilakukan dengan benar di acara polisi di TV. **Tidak ada yang membunuh keseruan lebih cepat dari tiga jam penuh mengurus dokumen.** Di dunia nyata, **tiga jam dokumen itulah yang membawa buktimu ke pengadilan.**

Semuanya dimulai di TKP. Dan **sekadar menemukan buktinya saja sudah bisa sulit** — apalagi dengan memory card seukuran perangko (atau lebih kecil). Benda seperti itu bisa disembunyikan di tempat yang jumlahnya nyaris tak terbatas.

> [!info] Konteks tambahan (bukan dari slide)
> Ini pembalikan ekspektasi yang penting untuk diterima sejak awal: **pekerjaan forensik itu 20% teknis, 80% prosedural.** Kamu bisa menemukan bukti paling menentukan di dunia, tapi kalau tidak bisa membuktikan dari mana asalnya, siapa saja yang pernah memegangnya, dan bahwa kamu tidak mengubahnya — bukti itu **tidak bernilai apa-apa di pengadilan**. Semua materi di sesi ini pada dasarnya menjawab satu pertanyaan: *"bagaimana kamu membuktikan bahwa kamu tidak merusak buktinya?"*

### TKP dan pengumpulan bukti
Dari sudut pandang praktis, **tidak semua TKP yang melibatkan bukti digital diperlakukan sama**. Bukti digital jadi fokus proses **pidana, perdata, dan administratif**, dan ada perbedaan jelas dalam cara TKP dan buktinya ditangani serta didokumentasikan:

- Kasus seperti **pembunuhan** butuh dokumentasi yang **sangat teliti**
- Kasus seperti **sengketa perdata** butuh respons yang **agak kurang intens**

Meski begitu, **ada prinsip dan protokol inti yang tetap konsisten**.

Setelah dinyatakan aman, **tugas nomor satu di TKP digital — atau TKP mana pun — adalah mengamankan barang buktinya**. TKP dan buktinya harus dilindungi dari **kompromi yang tidak disengaja maupun yang disengaja**. Mengamankan TKP tradisional berarti membatasi akses fisik dari orang-orang yang tidak punya alasan sah berada di sana — **tetangga yang kepo, media, dan atasan polisi** adalah penyusup TKP yang khas. Caranya: memasang garis polisi, menempatkan penjaga, atau sekadar meminta orang pergi.

### Dokumentasi TKP
Dokumentasi TKP juga harus dilakukan **first responder**. Bentuknya: **foto, video, rekaman suara, dan dokumentasi manual** atas hal-hal berikut:

- **Ruangan** tempat perangkat berada (meja, langit-langit, pintu masuk/keluar, jendela, pencahayaan, stopkontak, dan data drop)
- **Kondisi perangkat** (menyala, mati, lampu power berkedip)
- **Isi layar** dan apakah perangkatnya menyala (sistem operasi, program yang berjalan, tanggal dan waktu, konektivitas jaringan kabel dan/atau nirkabel)
- **Buku, catatan, dan potongan kertas**
- **Kabel yang tersambung dan yang tidak tersambung**

Kalau first responder sudah dilatih dalam pengumpulan dan pengawetan bukti, dia juga bisa mulai mengakuisisi apa yang dianggap **bukti fisik**: **unit sistem komputer, laptop, tablet, media penyimpanan tetap dan lepas-pasang, serta kabel dan charger**.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa "buku, catatan, dan potongan kertas" masuk daftar padahal ini forensik digital: karena **password sering ditulis di kertas**. Sticky note di bawah keyboard adalah klise justru karena benar. Dan buku/manual, seperti disebut di bagian removable media di bawah, **memberi petunjuk tingkat keahlian target**.

### Removable media
Kalau diperbolehkan secara hukum (misalnya dengan surat perintah), kita ingin menggeledah **di mana pun yang bisa memuat media penyimpanan**. Mengingat memory card "seukuran perangko" zaman sekarang, bukti ini bisa disembunyikan hampir di mana saja: **di dalam buku, dompet, pita topi**, dan sebagainya.

Meski ukurannya kecil, memory card bisa memuat **sangat banyak bukti potensial** — misalnya child pornography atau nomor kartu kredit curian.

Yang termasuk removable storage media: **DVD, external hard drive, thumb drive, dan memory card**.

Dan satu poin yang gampang terlewat: **kita tidak hanya tertarik pada perangkat dan media penyimpanan di TKP; area dan benda di sekitarnya juga layak dilihat.** Contohnya, **buku dan manual bisa memberi penyidik petunjuk soal tingkat keahlian target dan teknologi macam apa yang mungkin mereka hadapi**.

### Ponsel
Mandat pertama untuk ponsel, sama seperti perangkat elektronik lain: **jangan membuat perubahan apa pun pada perangkat atau media penyimpanannya.** Karena itu, **interaksi dengan ponsel harus dihindari kecuali benar-benar perlu**.

Ada beberapa pilihan:

| Pilihan | Pertimbangannya |
| --- | --- |
| **Matikan ponselnya** | Kekhawatirannya sama seperti PC: ponsel mungkin **dilindungi password**. Begitu dimatikan, **kodenya mungkin dibutuhkan untuk mengakses ponsel** |
| **Isolasi dalam Faraday bag atau kaleng, biarkan tetap menyala** | Kalau memungkinkan, **ini mungkin yang terbaik**. Lalu diangkut ke lab untuk diperiksa di ruangan berperisai |
| **Tempatkan di wadah khusus yang memerisai sinyal wireless** | **Kaleng cat kosong** dan **Faraday bag** adalah dua pilihan paling umum. Keduanya efektif melindungi ponsel dari sinyal seluler |

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa memerisai sinyal itu mendesak, bukan sekadar kehati-hatian: ponsel yang masih terhubung jaringan bisa **menerima perintah remote wipe**. Tersangka (atau temannya) bisa menghapus seluruh isi ponsel dari jarak jauh sementara ponselnya ada di tanganmu. Faraday bag mencegah itu. Ini juga menjelaskan saran "airplane mode" untuk perangkat mati di bagian berikutnya.

### Survei TKP
Setelah mengamankan bukti, **survei TKP** akan memberi penyidik gambaran akurat tentang apa yang menanti. Ada beberapa pertanyaan yang perlu dijawab:

- **Jenis perangkat apa saja yang ada?**
- **Berapa banyak perangkat yang kita hadapi?**
- **Apakah ada perangkat yang sedang menyala?**
- **Tool apa yang akan dibutuhkan?**
- **Apakah keahlian yang diperlukan tersedia di tempat?**

### Order of Volatility
Ini salah satu daftar terpenting di seluruh matkul. Bukti sebaiknya diprioritaskan — dan **umumnya kita mulai dari bukti yang paling volatile duluan**.

| # | Bukti |
| --- | --- |
| 1 | **CPU, cache, dan isi register** |
| 2 | **Routing table, ARP cache, process table, kernel statistics** |
| 3 | **Memory (RAM)** |
| 4 | **Temporary file system / swap space** |
| 5 | **Data di hard disk** |
| 6 | **Data yang ter-log secara remote** |
| 7 | **Data yang ada di media arsip** |

*(Henry, 2009)*

> [!info] Konteks tambahan (bukan dari slide)
> Logikanya sederhana: **ambil dulu yang paling cepat hilang.** Isi register CPU berubah miliaran kali per detik; data di tape arsip akan tetap sama tahun depan. Kalau kamu meng-image hard disk lebih dulu (berjam-jam) sementara mesinnya menyala, isi RAM sudah berubah total saat kamu selesai. Perhatikan juga bahwa daftar ini **hanya relevan untuk live system** — kalau mesinnya sudah mati, poin 1–4 sudah lenyap sejak awal, dan itulah tepatnya kenapa keputusan "cabut colokan atau tidak" jadi krusial.

### Mendokumentasikan TKP
Ada pepatah lama di penegakan hukum:

> **"If you don't write it down, it didn't happen."** *(Kalau kamu tidak menuliskannya, itu tidak pernah terjadi.)*

Apa pun situasinya, **setiap kali bukti dikumpulkan, dokumentasi adalah bagian yang sangat vital dari prosesnya**. Proses dokumentasi ini **dimulai saat penyidik tiba di TKP**: biasanya diawali dengan mencatat **tanggal dan waktu kedatangan** beserta **semua orang yang ada di TKP**. Baik juga mencatat **kondisi barangnya**, terutama kalau ada kerusakan yang terlihat.

### Chain of Custody
Sebelum sepotong bukti sampai di hadapan juri, dia harus memenuhi serangkaian **persyaratan hukum yang ketat**. Salah satunya adalah **chain of custody yang terdokumentasi dengan baik**.

Sebuah komputer yang disita sebagai barang bukti melewati **banyak perhentian** dalam perjalanannya ke persidangan: dikumpulkan, dicatat masuk di lab, disimpan, dikeluarkan untuk analisis, dikembalikan untuk disimpan, dan seterusnya.

**CoC adalah formulir yang secara hukum menjamin integritas bukti saat dipertukarkan antar individu**, sehingga juga memberi **tingkat akuntabilitas** — karena identifikasi personal dibutuhkan saat mengisi formulirnya. Formulir ini memberi **log dan catatan persis** atas pengangkutan dan pertukaran antar pihak, **dari pengumpulan di TKP sampai presentasi di pengadilan**.

**Field khas pada formulir CoC:**
- Nomor kasus / tindak pidana
- Nama korban dan tersangka
- Tanggal dan waktu penyitaan, yang mencakup:
  - **Lokasi penyitaan**
  - **Nomor item**
  - **Deskripsi item**
  - **Tanda tangan dan identitas** individu yang **menyerahkan** dan yang **menerima** barang
- **Otorisasi untuk pemusnahan**
- **Saksi pemusnahan barang bukti**
- **Penyerahan ke pemilik yang sah**

### Marking evidence
**"Mata rantai" pertama dalam chain of custody di kasus mana pun adalah orang yang mengumpulkan buktinya.** Kasus perdata mungkin sedikit berbeda: **staf IT atau pihak lain** bisa jadi yang memegang kehormatan sebagai mata rantai pertama.

Bukti **ditandai saat dikumpulkan**. Biasanya item bukti ditandai dengan **inisial, tanggal, dan mungkin nomor kasus**. **Permanent marker adalah yang terbaik** untuk memastikan tandanya tidak luntur atau hilang sama sekali.

### Cloning
**Forensic clone adalah salinan hard drive yang persis, bit demi bit.** Dikenal juga sebagai **bit stream image**. Dengan kata lain, **setiap bit (1 atau 0) diduplikasi** ke media terpisah yang **forensically clean**, misalnya hard drive lain.

Kenapa repot-repot? Kenapa tidak copy-paste saja file-nya? Slide memberi tiga alasan yang signifikan:

1. **Copy-paste hanya mendapat data aktif** — yaitu data yang bisa diakses user; file dan folder yang biasa user pakai, seperti dokumen Microsoft Word.
2. **Copy-paste TIDAK mendapat data di unallocated space**, termasuk **file terhapus dan file yang tertimpa sebagian**.
3. **Copy-paste tidak menangkap data filesystem.**

Semua itu akan menghasilkan **pemeriksaan forensik yang tidak efektif dan tidak lengkap**.

**Risiko terbesar selama proses cloning** adalah **menulis ke drive sumber atau drive bukti**. Penulisan apa pun ke bukti akan **mengompromikan integritasnya dan membahayakan admissibility-nya**. Memasang perangkat atau software **write-blocking** yang berfungsi akan mencegah hal ini terjadi.

### Live system vs dead system
Saat menyelidiki perangkat yang **menyala (live system)** dan yang **mati (dead system)**, **pertimbangan khusus harus diberikan pada volatilitas data**.

Poin kuncinya: **boot, reboot, atau shutdown sebuah perangkat bisa menyebabkan data ditulis ke hard drive**, yang mengakibatkan **data (file terhapus) di unallocated space tertimpa**.

**Untuk perangkat yang menyala (powered-on):**
- **Gerakkan mouse atau usap touchpad** kalau kamu curiga perangkatnya dalam keadaan sleep. **Jangan klik tombolnya**, karena itu bisa membuka atau menutup program dan proses.
- **Foto dan rekam layarnya** beserta semua program yang terlihat, data, waktu, dan item desktop.
- **Cabut kabel power** pada desktop, dan **lepas baterai** kalau memungkinkan pada perangkat portabel.

Slide menegaskan: **sangat penting bahwa data yang tersimpan di RAM dan paging file dikumpulkan dengan modifikasi sesedikit mungkin.** Tool imaging yang disebut: **Guymager** dan **dc3dd** di Kali Linux; tool akuisisi live lain seperti **CAINE (Computer Aided INvestigative Environment)** dan **Helix** juga bisa dipakai untuk mengakuisisi RAM dan paging file.

**Untuk perangkat yang mati (powered-off):**
- **Jangan pernah dinyalakan** kecuali oleh investigator forensik.
- Langkah khusus harus diambil untuk memastikan **data yang ada tidak terhapus dan data baru tidak tertulis**.
- Untuk perangkat portabel dan mobile yang sudah mati: **lepas baterainya** (kalau bisa) dan **taruh di kantong bukti**, supaya tidak ada cara untuk menyalakannya secara tidak sengaja setelah dicabut.
- Perangkat juga sebaiknya **dialihkan ke airplane mode** untuk menghindari koneksi dan komunikasi lebih lanjut.

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan kontradiksi yang tidak dibahas slide: untuk desktop yang menyala, sarannya **cabut kabel power** (biar tidak ada shutdown yang menulis ke disk). Tapi mencabut power **juga menghancurkan seluruh isi RAM** — poin 1–4 di Order of Volatility. Jadi urutan yang benar sebenarnya: **akuisisi RAM dulu selagi menyala, baru cabut colokan.** Slide berikutnya ("Principles of Live Collection") menjelaskan kapan langkah itu sepadan dilakukan.

### Prinsip live collection
Setelah menemukan komputer yang menyala di TKP, ada dua pertanyaan yang harus dijawab sejak awal.

**1. Apakah bukti potensial yang bisa dipulihkan benar-benar sepadan dengan waktu dan usahanya?** Dalam beberapa kasus, jawabannya bisa "**tidak**":
- Dalam kasus yang **melibatkan malware, RAM sangat vital**.
- Dalam kasus lain, misalnya **kepemilikan child pornography yang sudah jelas**, **RAM kemungkinan bernilai kecil**.

**2. Apakah sumber daya yang dibutuhkan tersedia?** Menangkap bukti di memori dengan sukses butuh **tool dan pelatihan khusus**. Tanpa dua bahan kunci itu, **mungkin lebih baik menyerah dan cabut saja colokannya** — risiko mengompromikan buktinya bisa jadi terlalu besar.

Slide menutup dengan nasihat yang bagus: **penting untuk bisa mengenali kapan kamu sudah kewalahan dan kapan kamu harus minta bantuan.**

### Bukti di dalam RAM
Volatile memory sebuah komputer (RAM) bisa berisi **bukti yang sangat berharga**, termasuk:

- **Proses yang sedang berjalan**
- **Perintah konsol yang dieksekusi**
- **Password dalam bentuk clear text**
- **Data yang tidak terenkripsi**
- **Instant message**
- **Alamat Internet Protocol**
- **Trojan horse**

*(Shipley & Reeve, 2006)*

### Write blocking
**Bekerja pada bukti asli bisa — dan biasanya akan — memodifikasi isi mediumnya.** Contohnya, mem-boot laptop sitaan ke OS aslinya akan **membuat data tertulis ke hard drive**, dan juga bisa **menghapus dan memodifikasi isi RAM dan paging file**.

Untuk mencegah ini, **write blocker harus dipakai**. Write blocker, sesuai namanya, **mencegah data ditulis ke media bukti**. Write blocker tersedia dalam **tipe hardware maupun software**.

### Data imaging dan hashing
**Imaging** merujuk pada **penyalinan data secara persis** — bisa berupa file, folder, partisi, atau seluruh media penyimpanan/drive.

Saat melakukan copy file dan folder biasa, **tidak semua file mungkin tersalin**, karena atributnya diset sebagai *system* atau bahkan *hidden*. Untuk mencegah ada file yang tertinggal, kita melakukan **jenis penyalinan khusus di mana setiap bit disalin atau di-image persis seperti adanya di medium saat itu** — seperti mengambil foto atau snapshot dari datanya.

Membuat salinan tiap bit data secara persis disebut membuat **physical image**. Melakukan **bit-stream copy memastikan integritas salinannya**. Untuk membuktikannya lebih jauh, **hash dari bukti asli dan hash dari physical image dihitung lalu dibandingkan**.

> [!info] Konteks tambahan (bukan dari slide)
> Rangkaian tiga alat ini bekerja sebagai satu sistem, dan pahami hubungannya: **write blocker** mencegah kamu mengubah aslinya, **bit-stream copy** memastikan kamu mendapat *semuanya* (termasuk slack dan unallocated space dari [[W03 - Disk and File System Analysis]]), dan **hash** membuktikan secara matematis bahwa dua hal pertama berhasil. Hilangkan salah satunya, dan kamu kehilangan kemampuan membuktikan integritas. Itu sebabnya ketiganya selalu diajarkan bersamaan.

### Pedoman akuisisi
**SWGDE Best Practices for Digital Evidence Collection, Version 1.0**, terbit **Juli 2018**, menguraikan best practice untuk computer forensics di bidang:
- **Evidence collection and handling**
- **Documentation**

Dokumen lengkapnya bisa diunduh dari `https://www.swgde.org/`.

## Diagram & Visual
- **Slide 27 — ilustrasi write blocker**
  ![[99-Assets/Forensics/W05-slide27.png]]

> [!warning] Dua slide yang seluruhnya gambar **tidak berhasil diikutkan**:
> - **Slide 12 — "Faraday Bag"** (judul + gambar saja)
> - **Slide 20 — "Cloning"** (judul + gambar saja, kemungkinan diagram proses cloning)
>
> Slide 20 khususnya sayang karena kemungkinan besar itu diagram alur cloning sumber→tujuan lewat write blocker. **Buka PPT aslinya di slide 12 dan 20.**

## Rumus / Sintaks

Order of Volatility — hafalkan urutannya:
```
1. CPU, cache, register
2. Routing table, ARP cache, process table, kernel statistics
3. Memory (RAM)
4. Temporary filesystem / swap space
5. Data di hard disk
6. Data ter-log secara remote
7. Data di media arsip
```

Tiga hal yang TIDAK didapat dari copy-paste biasa:
```
1. data di unallocated space (termasuk file terhapus & tertimpa sebagian)
2. data filesystem
3. file dengan atribut system / hidden
```

## Pertanyaan Terbuka
- Slide 3 mencantumkan **"Handling Damaged or Encrypted Media"** sebagai learning outcome, tapi **tidak pernah dibahas sama sekali**. Padahal media terenkripsi (BitLocker, FileVault — lihat [[W04 - Mac OS X System and Artifacts]]) adalah hambatan paling nyata dalam akuisisi modern. **Wajib ditanyakan.**
- **"Memory (RAM) Acquisition"** juga jadi learning outcome, tapi deck ini cuma menyebut **nama tool**-nya (Guymager, dc3dd, CAINE, Helix) tanpa satu pun **langkah atau perintah**. Slide 23 menjanjikan "more on this will be covered in later chapters" — perlu dipastikan sesi mana yang dimaksud.
- Ada **ketegangan antara dua saran** yang tidak diselesaikan slide: "cabut kabel power" pada perangkat menyala vs "RAM sangat berharga". Urutan yang benar (akuisisi RAM dulu, baru cabut) tidak pernah dinyatakan eksplisit. Ini kandidat kuat soal esai.
- **Write blocker dibahas dua kali** (slide 21 dan 27) tapi **tidak pernah dijelaskan cara kerjanya** — bagaimana persisnya perangkat itu memblokir penulisan, dan apa bedanya versi hardware dan software dari sisi keandalan di pengadilan.
- Formulir **CoC hanya didaftar field-nya**, tidak ada contoh formulir nyatanya.
- Referensi **Henry (2009)** untuk Order of Volatility dan **Shipley & Reeve (2006)** untuk bukti di RAM **tidak ada di daftar referensi** di slide 30–31.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W01 - Digital Forensic Fundamental]]
- [[W03 - Disk and File System Analysis]]
- [[W04 - Mac OS X System and Artifacts]]
- [[W06 - Linux System and Artifacts]]
- [[Forensics - Review dan Glosari]]
