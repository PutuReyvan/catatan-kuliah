---
matkul: Venture Creation
minggu: 3
sks: 3
sumber: Prototype Development (Low Fidelity).pptx
tags: [kuliah/vc, minggu/w03]
status: draft
diproses: 2026-09-04
---

# W03 — Prototype Development (Low Fidelity)

## Ringkasan
> - **Robust design** = pendekatan pengembangan produk yang memastikan performa TETAP KONSISTEN meski ada variasi di material, cara produksi, atau kondisi pemakaian. Beda dari "strong" (tahan banting) — robust itu soal ADAPTIF terhadap variasi.
> - Kerangka **Taguchi Method** membagi faktor jadi 4 kategori: **Control Factors** (yang bisa kamu atur), **Noise Factors** (yang TIDAK bisa kamu kontrol), **Signal Factors** (input dari user), dan **Output Responses** (hasil yang ingin dijaga tetap stabil).
> - Kerangka ini dipakai untuk produk fisik (speaker Bluetooth), layanan digital (aplikasi delivery makanan), MAUPUN layanan non-digital (jasa laundry) — fleksibel dipakai lintas jenis bisnis.
> - Setelah prototype dan robust design factor-nya dipresentasikan, dosen memberi feedback dan mahasiswa lanjut ke tahap validasi nyata: [[W04 - Prototype Testing (Customer Discovery)]].

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Robust design | Pendekatan desain yang menjaga performa tetap konsisten meski ada variasi kondisi |
| Taguchi Method | Pendekatan statistik untuk menemukan konfigurasi desain paling stabil |
| Control Factor | Variabel yang bisa dikendalikan/diatur oleh desainer atau bisnis |
| Noise Factor | Variabel yang TIDAK bisa dikontrol, sumber variasi performa |
| Signal Factor | Input/aksi dari user yang menentukan cara sistem dipakai |
| Output Response | Hasil/ukuran performa yang ingin dijaga tetap stabil |

## Isi

### Konsep Robust Design
**Robust design** adalah pendekatan pengembangan produk yang memastikan performa konsisten meski menghadapi variabilitas material, proses produksi, atau kondisi penggunaan. Penting untuk memahami perbedaan istilah **strong vs robust**: **strong berarti tahan banting (resistant)**, sedangkan **robust berarti adaptif (adaptable)** — mampu tetap berfungsi baik meski kondisinya berubah-ubah, bukan sekadar "kuat" secara fisik.

**Taguchi Method** adalah pendekatan statistik untuk menemukan konfigurasi desain yang paling STABIL. Robust design penting karena secara langsung meningkatkan kualitas, keandalan (reliability), dan kepuasan pelanggan.

> [!info] Analogi
> Bayangin dua jenis payung: Payung A terbuat dari besi tebal yang SANGAT KUAT (strong) tapi kaku dan berat — kalau tertiup angin kencang dari arah yang salah, dia bisa patah karena tidak bisa "mengalah". Payung B terbuat dari material yang lebih fleksibel dan dirancang untuk MELIPAT sedikit saat tertiup angin kencang, lalu kembali normal (robust) — dia bertahan justru karena BERADAPTASI, bukan cuma melawan. Robust design itu filosofi payung B: produk yang dirancang untuk tetap berfungsi baik di tengah kondisi yang TIDAK BISA KAMU KONTROL sepenuhnya.

### Empat Kategori Faktor (Kerangka Taguchi)
Kerangka ini membagi variabel yang memengaruhi performa produk/layanan jadi 4 kategori:

1. **Control Factors (C)** — variabel yang BISA dikendalikan/diatur oleh desainer, engineer, atau bisnis selama desain atau produksi untuk mencapai hasil yang diinginkan. *Contoh: pemilihan material, dimensi komponen, desain algoritma software, setting tekanan/torsi/suhu perakitan.*

2. **Noise Factors (N)** — variabel yang TIDAK BISA atau SULIT dikontrol, penyebab variasi performa. Bisa berasal dari lingkungan, perilaku user, atau inkonsistensi manufaktur. *Contoh: suhu/kelembapan ambient, cara user memakai/menyalahgunakan produk, variasi material dari supplier, keausan mesin, fluktuasi daya listrik, konektivitas jaringan (untuk layanan digital).*

3. **Signal Factors (S)** — input atau aksi yang berasal dari USER atau kondisi operasional. Mendefinisikan bagaimana sistem dipakai dan memengaruhi output. *Contoh: setting kecepatan blender, tekanan pada pedal rem, jumlah data yang diproses aplikasi, level volume speaker.*

4. **Output Responses (Y)** — hasil atau ukuran performa sistem — apa yang ingin dijaga tetap stabil dan konsisten meski ada variasi. *Contoh: retensi suhu di botol minum, kekuatan/ketahanan komponen, waktu loading aplikasi, daya tahan baterai, kepuasan pelanggan.*

### Contoh Penerapan
Slide memberikan tiga contoh konkret penerapan kerangka ini di jenis bisnis berbeda:

**Produk Fisik — Speaker Bluetooth Portabel:**
| Faktor | Contoh | Dampak |
| --- | --- | --- |
| Control | Material dan desain casing speaker | Memengaruhi kontrol getaran dan kualitas suara |
| Noise | Setting volume user | Pemakaian volume tinggi berlebihan menyebabkan distorsi/overheat |
| Signal | Level volume yang dipilih user | Menentukan seberapa keras speaker akan bermain |
| Output | Kejernihan suara dan daya tahan baterai | Hasil yang harus tetap stabil meski ada variasi |

**Layanan Digital — Aplikasi Delivery Makanan Online:**
| Faktor | Contoh | Dampak |
| --- | --- | --- |
| Control | Kapasitas server aplikasi dan efisiensi algoritma | Menentukan kecepatan dan waktu respons |
| Noise | Stabilitas koneksi internet | Memengaruhi performa aplikasi, terutama di daerah rural |
| Signal | Jumlah user aktif yang memesan | Input utama yang bervariasi sesuai permintaan pelanggan |
| Output | Waktu penyelesaian order & kepuasan user | Harus tetap stabil meski ada variasi beban |

**Layanan Non-Digital — Jasa Laundry:**
| Faktor | Contoh |
| --- | --- |
| Control | Jenis/dosis deterjen, suhu & durasi cuci, kecepatan spin dryer, kapasitas mesin, prosedur sortir |
| Noise | Cuaca/kelembapan, fluktuasi listrik/air, keausan mesin, lonjakan order mendadak, perilaku pelanggan yang kurang jelas |
| Signal | Jenis layanan (regular/express/dry clean), berat & jenis kain, instruksi khusus, jadwal pickup/delivery |
| Output | Kebersihan pakaian, waktu penyelesaian, tingkat ketepatan waktu, tingkat kesalahan, kepuasan pelanggan |

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan pola yang berulang di ketiga contoh: **Noise Factor selalu berisi "sesuatu yang bisa berubah tapi TIDAK kamu kontrol"** — cuaca, koneksi internet, perilaku pelanggan. Ini poin penting: robust design BUKAN tentang menghilangkan noise (karena memang tidak bisa), tapi tentang MERANCANG SISTEM supaya tetap berfungsi baik WALAUPUN noise itu ada.

### Aktivitas Mahasiswa
Mahasiswa diminta mengisi tabel kosong "What Is Your Robust Design Factors?" untuk produk/layanan mereka sendiri (mengisi Control, Noise, Signal, Output berdasarkan ide bisnis kelompok), lalu mempresentasikan bersama prototype (bisa berupa sketsa, video, gambar, dummy, atau MVP) untuk mendapat feedback dari dosen.

## Diagram & Visual
- **Slide 16 — Contoh Prototype Testing Report (template)**
  ![[99-Assets/VC/W03-slide16.png]]
  ![[99-Assets/VC/W03-slide16a.jpg]]

> [!warning] Slide 6 dan 7 (Robust Design Example) hanya berjudul "Robust Design Example" tanpa teks pendamping dan tanpa gambar yang berhasil diekstrak — kemungkinan contoh visual/diagram yang perlu dibuka manual di file PPT asli.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **MVP (Minimum Viable Product)** | Versi produk paling sederhana yang sudah cukup untuk diuji ke pasar |
| **Dummy (prototype)** | Model tiruan produk yang belum berfungsi penuh, dipakai untuk menunjukkan konsep |
| **Reliability** | Keandalan produk — konsistensi performa dari waktu ke waktu |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis contoh "Aplikasi Delivery Makanan Online": kenapa "stabilitas koneksi internet" diklasifikasikan sebagai Noise Factor, BUKAN Control Factor, padahal secara teknis penyedia aplikasi BISA berinvestasi memperluas infrastruktur jaringan? Di titik mana sebuah faktor berpindah dari "Noise" ke "Control" tergantung skala/sumber daya bisnisnya?
2. **(C4 – Analisis)** Bandingkan Signal Factor dan Noise Factor dari sisi SUMBERNYA. Analisis: keduanya sama-sama berasal dari "luar kendali penuh desainer", tapi kenapa Signal Factor dianggap input yang WAJAR/DIHARAPKAN, sementara Noise Factor dianggap sumber masalah yang harus DIANTISIPASI?
3. **(C5 – Evaluasi)** Sebuah kelompok merancang robust design factor untuk ide bisnis "jasa titip beli makanan kampus", tapi mereka HANYA mengisi Control Factor dan Output Response, mengosongkan Noise dan Signal Factor karena "belum kepikiran". Evaluasi: risiko apa yang muncul dari prototype yang dirancang tanpa analisis Noise Factor yang jelas, khususnya saat masuk ke tahap [[W04 - Prototype Testing (Customer Discovery)]]?
4. **(C5 – Evaluasi)** Bandingkan filosofi "strong" vs "robust" dalam konteks startup mahasiswa yang sumber dayanya terbatas. Evaluasi: kenapa mengejar "strong" (misalnya membangun fitur yang sangat lengkap dan tahan banting dari awal) justru BISA jadi strategi yang KURANG tepat dibanding mengejar "robust" (fitur minimal tapi adaptif) untuk tahap Low Fidelity prototype?
5. **(C6 – Cipta)** Rancang tabel Robust Design Identification Factors (Control, Noise, Signal, Output) LENGKAP untuk ide bisnis fiktif "aplikasi peminjaman buku antar mahasiswa di kampus". Isi minimal 2 contoh konkret di setiap kategori, mengikuti format tabel yang dipakai di tiga contoh slide (speaker, delivery app, laundry).

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_VC]]
- [[W02 - Opportunity Recognition dan Problem Framing]]
- [[W04 - Prototype Testing (Customer Discovery)]]
- [[VC - Review dan Glosari]]
