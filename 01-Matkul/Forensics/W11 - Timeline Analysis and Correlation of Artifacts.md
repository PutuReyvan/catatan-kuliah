---
matkul: Forensics
minggu: 11
sks: 2
sumber: 11 Timeline Analysis & correlation.pptx
tags: [kuliah/forensics, minggu/w11]
status: draft
diproses: 2026-09-03
---

# W11 — Timeline Analysis and Correlation of Artifacts

> [!warning] **Judul deck tidak cocok dengan isinya.** Judulnya "Timeline Analysis & Correlation of Artifacts", tapi **tidak ada satu slide pun yang membahas timeline maupun korelasi**. Isi sebenarnya adalah **pengenalan tool artifact**: p0f, Nmap, Linux Explorer, dan credential dumping. Deck ini juga **paling pendek di seluruh matkul (12 slide)**. Lihat Pertanyaan Terbuka.

## Ringkasan
> - Definisi artifact versi deck ini: **jejak yang tertinggal**, bisa diibaratkan **tapak kaki end-user atau hacker** — dan **end-user sering tidak sadar artifact itu ada**.
> - **p0f** untuk mengidentifikasi perangkat dan sistem operasi.
> - **Nmap** untuk information gathering dan fingerprinting: menemukan port yang **open, filtered, atau closed**, sekaligus **fingerprint OS**-nya.
> - **Linux Explorer** untuk live forensic Linux: proses dan PID, username dan login, port dan service, file mencurigakan, **deteksi rootkit**.
> - **Credential dumping** = mengambil informasi login dan password (hash maupun clear text) dari OS dan software, untuk **lateral movement** dan akses informasi terbatas.
> - **mimipenguin** = versi Linux dari **mimikatz**; mengambil password **plaintext yang tidak terenkripsi** dari proses di memori.
> - **Memory dump** (core dump / system dump) = snapshot data memori pada satu momen; bisa berisi data forensik berharga tentang kondisi sistem **sebelum** crash atau kompromi.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Artifact | Jejak yang tertinggal; tapak kaki end-user atau hacker |
| p0f | Tool untuk mengidentifikasi perangkat dan sistem operasi |
| Nmap | Network Mapper; information gathering dan OS fingerprinting |
| Fingerprinting | Menentukan sistem operasi dan service dari respons jaringan |
| Open / filtered / closed port | Port terbuka / dipantau-difirewall / tertutup |
| Linux Explorer | Tool live forensic Linux |
| Rootkit | Malware yang menyembunyikan keberadaannya di level sistem |
| Credential dumping | Mengambil informasi login dan password dari OS dan software |
| Lateral movement | Berpindah dari satu sistem terkompromi ke sistem lain di jaringan yang sama |
| Memory dump | Snapshot data memori komputer pada satu momen; disebut juga core dump / system dump |
| mimikatz | Tool password-cracking populer (Windows) |
| mimipenguin | Padanan mimikatz untuk Linux |
| swap_digger | Tool yang disebut sebagai pembanding mimipenguin |

## Isi

### Apa itu artifact
Slide mengulang dan mempertajam definisi dari [[W01 - Digital Forensic Fundamental]]:

> Jadi, apa itu artifact dalam cyber security? **Artifact adalah jejak yang tertinggal.** Kamu bisa mengasosiasikannya dengan **tapak kaki end-user atau hacker**. Tapi, **end-user sering tidak sadar bahwa artifact itu ada**.

Ada beberapa cara berbeda untuk mengungkap berbagai artifact yang berguna untuk investigasi forensik. Slide mengelompokkan tool yang dibahas jadi dua fokus:

| Fokus | Tool |
| --- | --- |
| **Memori dan swap** | Sebagian besar tool di bab ini |
| **Jaringan dan perangkat** | **Nmap** dan **p0f** |

### p0f — identifikasi perangkat dan OS
Salah satu tool untuk mengidentifikasi perangkat dan sistem operasi adalah **p0f**. Ada beberapa fungsi sintaks di p0f untuk membantu investigasi dengan mengidentifikasi perangkat dan sistem operasi — daftarnya bisa dilihat dengan:

```bash
p0f -h
```

> [!info] Konteks tambahan (bukan dari slide)
> Yang membedakan p0f dan pantas dicatat: **p0f bersifat pasif.** Dia **mendengarkan** trafik yang lewat dan menebak sistem operasi dari karakteristik paketnya, **tanpa mengirim apa pun**. Nmap sebaliknya — dia **aktif**, mengirim paket dan menunggu respons. Bedanya penting secara forensik dan hukum: **scanning aktif meninggalkan jejak di sistem target dan bisa dianggap tindakan intrusif**, sementara pengamatan pasif tidak.

### Nmap — information gathering dan fingerprinting
Tool **Nmap (Network Mapper)** dipakai untuk **mengumpulkan informasi tentang sumber daya dan perangkat di jaringan**, menemukan port yang **open, filtered (dipantau atau difirewall), atau closed**, sekaligus **melakukan fingerprint sistem operasinya**.

Perintah yang dicontohkan slide:

```bash
nmap -v -O -sV 172.16.0.0/24 -Pn
```

| Flag | Artinya |
| --- | --- |
| **`-v`** | Verbose output |
| **`-O`** | Mengaktifkan **deteksi sistem operasi** |
| **`-sV`** | **Menyelidiki port terbuka** untuk menentukan **service dan versi**-nya |
| **`-Pn`** | **Memperlakukan semua host sebagai online** (melewati tahap discovery) |

### Linux Explorer — live forensic
Saat melakukan **live forensic** pada mesin Linux, **Linux Explorer** bisa dipakai untuk mengumpulkan informasi dan artifact. Yang bisa ditemukan:

- **Proses dan process ID**
- **Username dan login**
- **Port dan informasi service**
- **File yang mencurigakan**
- **Deteksi rootkit**

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan kata "**live**" — ini berarti tool dijalankan **pada sistem yang masih menyala**, dan langsung bertabrakan dengan prinsip di [[W05 - Data Acquisition]]: **setiap tindakanmu di sistem live mengubah sistem itu**. Menjalankan Linux Explorer akan membuat proses baru, mengubah timestamp akses, dan mungkin menulis ke disk. Itu bisa diterima **asalkan kamu mendokumentasikan apa yang kamu jalankan dan kapan** — kembali lagi ke *"if you don't write it down, it didn't happen"*.

### Credential dumping
**Credential dumping adalah teknik yang sangat populer**, di mana penyerang **menyisir komputer yang sudah dikompromikan untuk mencari kredensial**, supaya bisa **bergerak lateral (lateral movement) dan/atau melancarkan serangan lebih lanjut**.

Definisinya: **proses mendapatkan informasi login akun dan password**, biasanya dalam bentuk **hash atau password clear text**, **dari sistem operasi dan software**. Kredensial itu kemudian bisa dipakai untuk **melakukan Lateral Movement dan mengakses informasi terbatas**.

**Memory dump** (dikenal juga sebagai **core dump** atau **system dump**) adalah **snapshot dari data memori komputer pada satu momen tertentu**. Sebuah memory dump bisa **berisi data forensik berharga tentang kondisi sistem sebelum suatu insiden** seperti crash atau kompromi keamanan.

### Password dumping dengan mimipenguin
Salah satu dari banyak cara melakukan password dumping adalah dengan tool. Slide mencontohkan **mimipenguin**:

- **Mimipenguin didasarkan pada mimikatz**, tool password-cracking yang sangat populer.
- Sama seperti **swap_digger**, mimipenguin bisa **mengambil artifact yang berjalan di memori** dengan **men-dump proses memori yang mungkin berisi password tidak terenkripsi dalam bentuk plaintext**.
- Dengan menjalankan mimipenguin, **password di perangkat itu akan ter-dump**.

> [!info] Konteks tambahan (bukan dari slide)
> Ini penutup dari benang merah yang dimulai jauh di [[W02 - Key Technical Concepts]]. Ingat daftar isi RAM dari [[W05 - Data Acquisition]]: **"password dalam bentuk clear text"**. Di sana itu cuma klaim; **di sini kamu diberi tool untuk membuktikannya**. Alasan teknisnya: aplikasi harus memegang passwordmu dalam bentuk asli untuk sesaat sebelum bisa memakainya, dan **potongan memori itu sering tidak dibersihkan** setelahnya.
>
> Perhatikan juga bahwa tool ini **dual-use**: mimikatz/mimipenguin dipakai penyerang untuk mencuri kredensial **dan** dipakai examiner untuk memulihkan bukti. Bedanya cuma **otorisasi**, bukan teknologinya.

## Diagram & Visual
- **Slide 5 — output `p0f -h` (daftar opsi sintaks p0f)**
  ![[99-Assets/Forensics/W11-slide05.png]]
- **Slide 6 — tampilan Nmap untuk information gathering**
  ![[99-Assets/Forensics/W11-slide06.png]]
- **Slide 7 — output `nmap -v -O -sV 172.16.0.0/24 -Pn`**
  ![[99-Assets/Forensics/W11-slide07.png]]
- **Slide 10 — output mimipenguin yang menampilkan password hasil dump**
  ![[99-Assets/Forensics/W11-slide10.png]]

> [!warning] Semua isi teknis deck ini ada di **screenshot output**, bukan teks. Yang bisa dibaca dari note ini cuma perintah dan penjelasannya; **hasil sesungguhnya harus dilihat dari gambar di atas**.

## Rumus / Sintaks

```bash
p0f -h                                   # daftar opsi p0f (identifikasi OS, PASIF)

nmap -v -O -sV 172.16.0.0/24 -Pn         # scan jaringan (AKTIF)
#  -v   verbose
#  -O   deteksi sistem operasi
#  -sV  probe port terbuka -> service + versi
#  -Pn  anggap semua host online (lewati discovery)
```

Tiga status port menurut Nmap:
```
open      : port terbuka, ada service mendengarkan
filtered  : dipantau atau difirewall
closed    : tertutup
```

## Pertanyaan Terbuka
- **Ini deck dengan kesenjangan judul-isi terbesar di seluruh matkul.** Judulnya "Timeline Analysis & Correlation of Artifacts", tapi **tidak ada satu slide pun** tentang timeline maupun korelasi artifact. Dari sembilan learning outcome di slide 3, **hanya empat yang dibahas** (identifying devices, information gathering/fingerprinting, live Linux forensic, password dumping). Yang **tidak dibahas**:
  - **Timeline Reconstruction and Correlation of Artifacts** ← *judul deck-nya sendiri*
  - **Anti-Forensic Techniques Detection**
  - **Volatile Artifacts Analysis (RAM, Process, Network)**
  - **Malware Artifacts & Indicators of Compromise (IoCs)**
  - **Artefact Analysis Reporting Techniques**

  **Ini prioritas tertinggi untuk ditanyakan ke dosen.** Timeline analysis adalah salah satu keterampilan inti forensik dan muncul di **LO 3 seluruh matkul** ("timeline reconstruction"), tapi materinya tidak ada di deck mana pun.
- Deck ini **cuma 12 slide** — paling pendek di matkul ini, sementara topik yang dijanjikan judulnya justru salah satu yang paling besar. Kemungkinan besar sebagian besar materinya **disampaikan lisan atau lewat praktikum**.
- **MAC times** ([[W06 - Linux System and Artifacts]]) dan **timestamp Created/Modified/Accessed** ([[W07 - Windows System Artifacts]]) adalah **bahan mentah untuk timeline**, dan sudah diajarkan. Tapi **cara menyusunnya jadi timeline tidak pernah diajarkan** — tidak ada `mactime`, tidak ada `log2timeline`/`plaso`, tidak ada super timeline.
- **swap_digger** disebut sebagai pembanding mimipenguin tapi **tidak pernah dijelaskan**.
- **Linux Explorer** disebut tanpa satu pun perintah atau cara instalasi.
- Slide 4 menyebut "most of the tools used in this chapter focus specifically on **memory and swap analysis**", padahal dari empat tool yang benar-benar dibahas, **dua di antaranya (p0f, Nmap) justru soal jaringan**, dan cuma mimipenguin yang menyentuh memori. Kalimat ini kemungkinan tersisa dari bab buku sumbernya yang isinya lebih lengkap.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W01 - Digital Forensic Fundamental]]
- [[W05 - Data Acquisition]]
- [[W06 - Linux System and Artifacts]]
- [[W12 - Automating Analysis and Timeline Analysis]]
- [[W13 - Network Analysis]]
- [[Forensics - Review dan Glosari]]
