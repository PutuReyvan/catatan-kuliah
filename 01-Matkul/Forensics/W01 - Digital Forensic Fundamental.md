---
matkul: Forensics
minggu: 1
sks: 2
sumber: Digital Forensic Fundamental.pptx
tags: [kuliah/forensics, minggu/w01]
status: draft
diproses: 2026-09-03
---

# W01 — Digital Forensic Fundamental

## Ringkasan
> - Digital forensic = **metode ilmiah yang teruji** untuk mendokumentasikan TKP elektronik, mencari dan menyita, mengawetkan barang bukti, mengakuisisi dan menganalisis data, sampai bersaksi sebagai ahli di pengadilan.
> - Tujuannya satu kalimat: **"menemukan fakta, dan lewat fakta itu merekonstruksi kebenaran suatu peristiwa."**
> - **Artifact ≠ evidence.** Artifact = jejak yang ditinggalkan aktivitas (bisa berbahaya, bisa juga tidak). Evidence = sesuatu yang dipakai dalam proses hukum. Salah pakai istilah bisa bikin examiner kena masalah.
> - Proses intinya tiga tahap: **Acquisition → Analysis → Presentation**.
> - Ancaman terbesar buat forensik sekarang: media sosial dan kanal bawah tanah, hotspot gratis di mana-mana, VPN, **TRIM di SSD**, dan organisasi tanpa personel/rencana DFIR sehingga bukti volatile di RAM hilang begitu sistem di-restart.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Digital forensic | Penerapan metode ilmiah yang teruji terhadap sumber digital, untuk merekonstruksi peristiwa kriminal atau mengantisipasi tindakan tidak sah |
| Artifact | Jejak yang tertinggal akibat aktivitas dan peristiwa; bisa tidak berbahaya, bisa juga sebaliknya |
| Evidence | Sesuatu yang dipakai dalam proses hukum — istilah yang harus dipakai hati-hati |
| Digital archaeologist | Metafora untuk examiner: menggali sisa-sisa peristiwa yang tertinggal di sistem |
| Acquisition | Pengumpulan media digital yang akan diperiksa |
| Working copy | Duplikat media asli yang dipakai untuk pemeriksaan |
| Analysis | Identifikasi, analisis, dan interpretasi item di dalam media |
| Presentation | Menyampaikan hasil analisis ke pihak berkepentingan, termasuk mempertahankannya saat digugat |
| Write blocker | Alat untuk menjaga data tidak berubah saat diperiksa |
| Chain of custody | Rantai penguasaan barang bukti (disebut sebagai LO, lihat Pertanyaan Terbuka) |
| DFIR | Digital Forensics and Incident Response |
| TRIM | Teknologi SSD yang menghapus data jauh lebih efisien dari disk magnetik lama |
| Volatile evidence | Bukti yang hilang saat sistem dimatikan, misalnya isi RAM, paging, dan swap file |

## Isi

### Definisi digital forensic
Slide memakai satu definisi panjang yang perlu dibaca pelan-pelan, karena hampir semua materi satu semester ini adalah pecahan dari kalimat ini:

> Penggunaan **metode yang diturunkan secara ilmiah dan sudah terbukti** untuk **mendokumentasikan TKP elektronik**, **pencarian dan penyitaan**, **pengawetan barang bukti**, **akuisisi data**, **analisis data**, **analisis kasus**, **pelaporan**, dan **bersaksi sebagai saksi ahli** — yang diturunkan dari sumber-sumber digital, dengan tujuan memfasilitasi atau memajukan rekonstruksi peristiwa yang ditemukan bersifat kriminal, atau membantu mengantisipasi tindakan tidak sah yang terbukti mengganggu operasi yang direncanakan.

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan dua tujuan yang disebut di ujung definisi: **merekonstruksi kejahatan yang sudah terjadi** (reaktif, ranah penegakan hukum) dan **mengantisipasi tindakan tidak sah yang mengganggu operasi** (proaktif, ranah korporat/DFIR). Dua-duanya disebut digital forensic, tapi konteks kerjanya beda jauh — yang pertama berujung di pengadilan, yang kedua berujung di laporan internal. Ini menjelaskan kenapa nanti ada materi yang legalistik (chain of custody) dan ada yang teknis murni (carving, timeline).

### Delapan tahap dalam definisi
Slide memecah definisi tadi jadi delapan tahap berurutan:

| # | Tahap | Isinya |
| --- | --- | --- |
| 1 | **Documenting the Electronic Crime Scene** | Merekam kondisi TKP digital, termasuk perangkat apa saja yang ditemukan dan posisinya |
| 2 | **Search and Seizure** | Mencari dan menyita perangkat digital **secara legal**, biasanya dengan surat perintah (warrant) |
| 3 | **Evidence Preservation** | Menjaga data tetap tidak berubah — misalnya dengan **write blocker** atau imaging |
| 4 | **Data Acquisition** | Mengambil salinan data **bit-by-bit** dari perangkat target supaya bisa dianalisis **tanpa menyentuh data aslinya** |
| 5 | **Data Analysis** | Menganalisis data untuk menemukan bukti: email, log aktivitas, file terhapus, dan lain-lain |
| 6 | **Case Analysis** | Menghubungkan hasil analisis data dengan **kronologi kasus** atau dugaan kejahatannya |
| 7 | **Reporting** | Menyusun laporan forensik berisi temuan, proses kerja, dan barang bukti yang diperoleh |
| 8 | **Testifying as an Expert Witness** | Kalau dibutuhkan di pengadilan, ahli forensik menjelaskan temuannya secara profesional sebagai saksi ahli |

### Tujuan analisis forensik
Slide mengutipnya sebagai satu kalimat:

> **"Tujuan dari setiap pemeriksaan forensik adalah menemukan fakta, dan lewat fakta-fakta itu merekonstruksi kebenaran suatu peristiwa."**

Examiner mengungkap kebenaran suatu peristiwa dengan **menemukan dan memaparkan sisa-sisa peristiwa** yang tertinggal di sistem. Sejalan dengan metafora **digital archaeologist**, sisa-sisa ini disebut **artifacts** — dan kadang disebut juga sebagai *evidence*.

> [!info] Konteks tambahan (bukan dari slide)
> Metafora arkeolognya pas: arkeolog gak nyari "kebenaran" langsung, dia nyari **pecahan periuk**. Dari pecahan itu baru direkonstruksi siapa yang tinggal di situ dan apa yang mereka lakukan. Forensik digital sama — kamu gak nemu file bernama `bukti_kejahatan.txt`, kamu nemu entri registry, timestamp yang janggal, dan potongan file di unallocated space, lalu **kamu yang menyusun ceritanya**. Bagian "menyusun cerita" itulah yang bikin ini keahlian, bukan sekadar menjalankan tool.

### Artifact vs Evidence
Ini pembedaan yang ditandai khusus oleh slide, dan sering jadi soal definisi.

| | **Evidence** | **Artifact** |
| --- | --- | --- |
| Definisi | Sesuatu yang **dipakai dalam proses hukum** | **Jejak yang tertinggal** akibat aktivitas dan peristiwa |
| Sifat | Punya status hukum | Bisa tidak berbahaya (*innocuous*), bisa juga tidak |
| Peringatan slide | **Memakai istilah ini serampangan bisa membuat examiner kena masalah** | — |

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa peringatannya keras: begitu kamu menyebut sesuatu "evidence", kamu secara implisit mengklaim benda itu relevan dan sah untuk dipakai di pengadilan — padahal yang menentukan itu hakim, bukan kamu. Sebagai examiner, yang kamu temukan itu **artifact**. Statusnya jadi *evidence* setelah melewati proses hukum. Di laporan, kebiasaan menulis "artifact" alih-alih "evidence" itu melindungi dirimu sendiri.

### Ruang lingkup investigasi
Investigasi digital forensic mencakup, tapi tidak terbatas pada:

- **Data Recovery**
- **Identity theft**
- **Malware dan ransomware investigation**
- **Network dan Internet Investigation**
- **Email Investigation**
- **Corporate Espionage**
- **Child Pornography Investigation**

### Tiga tahap proses forensik
Kalau delapan tahap tadi adalah rincian legal-proseduralnya, tiga tahap ini adalah **kerangka teknisnya**, dan inilah yang dipakai berulang-ulang di sisa semester.

**1. Acquisition.** Merujuk pada pengumpulan media digital yang akan diperiksa. Tergantung jenis pemeriksaannya, ini bisa berupa **hard drive fisik, optical media, storage card dari kamera digital, ponsel, chip dari embedded device, bahkan satu file dokumen saja**. Apa pun bentuknya, media yang diperiksa **harus diperlakukan dengan hati-hati**. Minimal, proses akuisisi harus terdiri dari:
- Membuat **duplikat dari media asli** (disebut **working copy**), dan
- Menjaga **catatan yang baik atas semua tindakan** yang dilakukan terhadap media asli mana pun.

**2. Analysis.** Merujuk pada pemeriksaan media yang sesungguhnya — bagian "**identification, analysis, and interpretation**" dari definisi DFRWS 2001.
- **Identification** terdiri dari menemukan item yang ada di dalam media, lalu **mempersempit himpunan itu** menjadi item atau artifact yang menarik.
- Item-item itu kemudian dikenai **analisis yang sesuai**: file system analysis, file content examination, log analysis, statistical analysis, atau jenis review lain.
- Akhirnya, examiner **menginterpretasikan** hasil analisis itu berdasarkan **pelatihan, keahlian, eksperimen, dan pengalamannya**.

**3. Presentation.** Merujuk pada proses examiner membagikan hasil tahap analisis kepada pihak yang berkepentingan. Ini terdiri dari menghasilkan **laporan** berisi tindakan yang diambil examiner, artifact yang ditemukan, dan **makna dari artifact tersebut**. Tahap presentasi juga bisa mencakup **examiner mempertahankan temuannya saat digugat** (*under challenge*).

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan kata terakhir di tiap tahap: acquisition menekankan **catatan**, analysis menekankan **interpretasi berbasis pengalaman**, presentation menekankan **mempertahankan temuan**. Ketiganya menunjuk ke hal yang sama — dalam forensik, **hasil tanpa proses yang bisa dipertanggungjawabkan itu tidak bernilai**. Tool bisa menemukan file terhapus dalam 5 menit; yang butuh satu semester adalah belajar mempertanggungjawabkan bagaimana kamu menemukannya.

### Ancaman terhadap digital forensic
Lima hal yang disebut slide bikin pekerjaan forensik makin sulit:

1. **Media sosial dan kanal komunikasi bawah tanah** dengan cepat jadi bentuk komunikasi termudah antar hacker dan hacktivist sepemikiran.
2. **Hotspot wireless gratis** hampir di tiap jalan.
3. **VPN** menambah kompleksitas investigasi digital forensic.
4. **SSD memakai teknologi TRIM** yang menghapus data jauh lebih efisien daripada disk magnetik lama.
5. **Lingkungan tanpa personel forensik terlatih dan tanpa rencana, kebijakan, serta implementasi DFIR** — pelanggaran dan insiden bisa tidak terdeteksi berminggu-minggu atau berbulan-bulan, sehingga **bukti volatile penting yang tersimpan di memori (RAM) beserta paging dan swap file akan hilang begitu sistem di-restart**.

> [!info] Konteks tambahan (bukan dari slide)
> Poin 1–3 pada dasarnya soal **anonimitas** (susah menghubungkan aktivitas ke orang), sementara poin 4–5 soal **hilangnya data** (buktinya benar-benar lenyap). Yang kedua lebih gawat, karena anonimitas masih bisa dilawan dengan korelasi bukti, sedangkan data yang sudah hilang tidak bisa dikembalikan. Itu juga alasan kenapa nanti di materi akuisisi, **urutan pengambilan bukti berdasarkan tingkat volatilitas** jadi hal yang ditekankan.

## Referensi Utama Matkul
Slide 15 menyebut daftar rujukan yang dipakai sepanjang semester — dua yang paling sering muncul di deck-deck berikutnya:

- **Parasram, S.V.N. (2020).** *Digital Forensics with Kali Linux, Second Edition.* ISBN 9781838640804
- **Sammons, J. (2015).** *The Basics of Digital Forensics, Second Edition.* Syngress. ISBN 9780128016350
- Carvey, H. (2005). *Locard's Exchange Principle in the Digital World.*
- Saferstein, R. (2006). *Criminalistics: An Introduction to Forensic Science.*
- Widup, S. (2014). *Computer Forensics and Digital Investigation with EnCase Forensic v7.*
- Altheide, C. & Carvey, H. (2011). *Digital Forensics with Open Source Tools.*

## Diagram & Visual
> [!warning] Deck ini **tidak punya satu pun gambar non-dekoratif** yang bisa diekstrak dari 16 slide. Seluruh isinya teks, jadi note ini sudah mewakili keseluruhan PPT-nya.

## Pertanyaan Terbuka
- Slide 3 mencantumkan **"CHAIN OF CUSTODY IN DIGITAL FORENSIC"**, **"TYPES OF DIGITAL EVIDENCE & SOURCES"**, dan **"DIGITAL FORENSIC READINESS"** sebagai learning outcome, tapi **ketiganya tidak pernah dibahas** di slide mana pun dalam deck ini. Kemungkinan besar dijelaskan lisan di kelas. Ini tiga topik yang sangat mungkin keluar di ujian — **wajib ditanyakan**.
- **Locard's Exchange Principle** muncul di daftar referensi (Carvey 2005) tapi tidak pernah disebut di isi slide. Padahal itu prinsip dasar seluruh ilmu forensik ("setiap kontak meninggalkan jejak") dan langsung menjelaskan kenapa artifact bisa ada.
- **DFRWS 2001 definition** dirujuk di slide 12 sebagai sumber istilah "identification, analysis, and interpretation", tapi definisi lengkapnya tidak pernah ditampilkan.
- Ancaman TRIM pada SSD disebut sekilas tanpa dijelaskan mekanismenya. Ini penting karena artinya **file terhapus di SSD sering benar-benar tidak bisa dipulihkan**, berbeda dengan HDD — bandingkan dengan materi recovery di [[W09 - File Recovery and Data Carving]].

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W02 - Key Technical Concepts]]
- [[Forensics - Review dan Glosari]]
