---
matkul: Forensics
minggu: 8
sks: 2
sumber: 8 File Recovery and Data Carving.pptx
tags: [kuliah/forensics, minggu/w08]
status: draft
diproses: 2026-09-03
---

# W08 — File Recovery and Data Carving

## Ringkasan
> - **File carving** mengambil data dari **unallocated space** memakai **karakteristik file itu sendiri** (struktur file, header, footer) — **bukan** metadata filesystem. Jadi carving tetap jalan walau extension diganti atau hilang.
> - Tiga tool yang dibahas: **foremost** (header/footer, CLI), **recoverjpeg** (khusus gambar), **bulk_extractor** (tidak terbatas jenis file — bisa menarik **nomor kartu kredit, email, URL, riwayat pencarian, profil media sosial**).
> - Separuh deck ini sebenarnya tentang **antiforensic**, yaitu lawan dari carving.
> - Teknik menyembunyikan data, dari yang sederhana ke yang bikin praktisi tidak bisa tidur: ganti nama/extension → kubur di direktori tak berhubungan → **file di dalam file** → **enkripsi**.
> - **Enkripsi**: plain text + algoritma + key → cipher text. **Key space** menentukan seberapa mungkin dipecahkan brute force.
> - **Steganography** dari bahasa Yunani *stegos* (tertutup) + *graphie* (tulisan) = **tulisan tersembunyi**.
> - **Drive wiping** menimpa data supaya tidak bisa dipulihkan; **defragment atau reformat sering dicoba tapi hasilnya terbatas**.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| File carving / data carving | Menyusun ulang file dari fragmen data mentah saat metadata filesystem tidak tersedia |
| Unallocated space | Area media yang ditandai OS atau file table sebagai kosong / tidak dialokasikan |
| Header / footer | Bagian awal/akhir file yang bisa mengidentifikasi jenis file |
| foremost | Tool CLI yang memulihkan file dengan membaca header dan footer |
| `audit.txt` | File hasil foremost yang berisi detail temuan |
| Scalpel | Tool carving alternatif |
| recoverjpeg | Tool pemulihan gambar sederhana |
| bulk_extractor | Tool ekstraksi yang **tidak terbatas jenis file tertentu** |
| Antiforensic | Tool dan teknik untuk melawan kemajuan forensik |
| Plain text | Pesan asli yang belum dienkripsi |
| Cipher text | Versi teracak dari plain text yang tidak bisa dipahami |
| Algorithm | Metode yang dipakai mengenkripsi pesan |
| Key | Data yang dipakai mengenkripsi dan mendekripsi; umumnya berupa password atau passphrase |
| Substitution / Transposition | Dua jenis enkripsi klasik |
| Key space | Panjang/ruang kunci; berdampak langsung pada kemampuan memecahkan enkripsi |
| Brute force attack | Mencoba setiap kombinasi kunci sampai yang benar ditemukan |
| Dictionary attack | Serangan password berbasis daftar kata |
| Cryptanalysis | Ilmu memecahkan password/sandi |
| Steganography | Menyembunyikan pesan rahasia di dalam pesan biasa |
| Drive wiping | Menimpa data di hard drive supaya tidak bisa dipulihkan |

## Isi

### Apa itu file carving
**File carving mengambil data dan file dari unallocated space memakai karakteristik spesifik**, seperti **struktur file dan file header**, **alih-alih metadata tradisional** yang dibuat oleh atau diasosiasikan dengan filesystem.

**Unallocated space** adalah area media penyimpanan yang **ditandai sistem operasi atau file table sebagai kosong atau tidak dialokasikan** ke file atau data mana pun. Meskipun lokasi dan informasi tentang file-nya sudah tidak ada dan kadang rusak, **masih ada karakteristik tentang file itu yang tersisa di header dan footer-nya** yang bisa mengidentifikasi file tersebut, **atau bahkan fragmen dari file itu**.

Poin kunci yang menyambung langsung ke [[W02 - Key Technical Concepts]]:

> **Bahkan kalau file extension sudah diubah atau hilang sama sekali, file header berisi informasi yang bisa mengidentifikasi jenis file**, dan carving bisa dicoba dengan menganalisis informasi header dan footer.

Catatan praktis dari slide: **data carving adalah proses yang cukup panjang dan sebaiknya dilakukan dengan tool otomatis untuk menghemat waktu**. Akan membantu juga kalau investigator **sudah punya gambaran jenis file apa yang dicari**, supaya fokusnya lebih baik dan hemat waktu. Meski begitu — **ini forensik, dan kita tahu waktu serta kesabaran adalah kuncinya**.

Definisi formalnya: **data carving, dikenal juga sebagai file carving, adalah teknik forensik untuk menyusun ulang file dari fragmen data mentah ketika tidak ada metadata filesystem yang tersedia.** Ini prosedur umum saat melakukan pemulihan data setelah kegagalan perangkat penyimpanan, dan **bisa juga dilakukan pada core memory dump** sebagai bagian dari prosedur debugging.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa carving disebut "pilihan terakhir" (ingat kutipan *"when all else fails, we carve"* di [[W03 - Disk and File System Analysis]]): carving **membuang seluruh konteks**. Kamu dapat isi file-nya, tapi **tidak dapat nama aslinya, tidak dapat timestamp, tidak dapat lokasi folder, tidak dapat siapa pemiliknya** — karena semua itu ada di metadata, dan metadata-nya sudah hilang. Jadi carving memberimu **apa**-nya tanpa **kapan, di mana, dan siapa**-nya. Itu sebabnya file hasil carving lebih lemah sebagai bukti daripada file yang dipulihkan lewat filesystem.

### Tool carving

**foremost** — tool **command line interface (CLI)** yang sederhana dan efektif, memulihkan file dengan **membaca header dan footer**-nya.

```bash
foremost -i (forensic image) -o (output folder) --options
```

Folder hasilnya akan berisi file **`audit.txt`** yang memuat **detail temuan**. Slide juga menyebut **Scalpel** sebagai tool carving lain.

**recoverjpeg** — tool pemulihan gambar sederhana. Setelah tahu drive mana yang ingin dipulihkan gambarnya (misalnya `sda1`):

```bash
recoverjpeg /dev/sda
```

**bulk_extractor** — tool ketiga. Slide menjelaskan alasan keberadaannya:

> foremost dan Scalpel, seperti yang sudah kita lihat, cukup mengesankan dalam pemulihan dan carving file, **tapi terbatas pada jenis file tertentu**. Untuk ekstraksi data lebih lanjut, kita bisa memakai bulk_extractor.

Data lain yang bisa di-*carve* dan diekstrak bulk_extractor:
- **Nomor kartu kredit**
- **Alamat email**
- **URL**
- **Pencarian online**
- **Informasi website**
- **Profil dan informasi media sosial**

> [!info] Konteks tambahan (bukan dari slide)
> Perbedaan pendekatannya penting dipahami: **foremost mencari file**, **bulk_extractor mencari pola**. foremost bertanya "di mana ada byte yang dimulai dengan header JPEG?"; bulk_extractor bertanya "di mana ada 16 digit yang lolos validasi nomor kartu kredit?". Karena itu bulk_extractor bisa menemukan bukti yang **tidak pernah berupa file** — misalnya nomor kartu kredit yang cuma sempat lewat di memori lalu tersimpan di swap.

### Antiforensic
Pemeriksaan komputer dan bukti hasilnya rutin muncul di catatan kepolisian. **Untuk melawan kemajuan forensik yang relatif baru ini, tool dan teknik antiforensic bermunculan dalam jumlah signifikan.** Slide mencatat mereka dipakai oleh **kriminal, teroris, dan eksekutif korporat sekaligus**.

Pada Februari 2011, **Valerie Caproni**, General Counsel FBI, berbicara di House Subcommittee on Crime, Terrorism, and Homeland Security tentang enkripsi dan ancaman yang diwakilinya:

> "Seiring melebarnya jurang antara **kewenangan** dan **kemampuan**, pemerintah makin tidak mampu mengumpulkan bukti berharga dalam kasus mulai dari eksploitasi dan pornografi anak, kejahatan terorganisasi dan perdagangan narkoba, sampai terorisme dan spionase — **bukti yang sudah diizinkan pengadilan untuk dikumpulkan pemerintah**. Jurang ini menimbulkan ancaman yang makin besar bagi keselamatan publik."
>
> *(Caproni, 2011)*

### Menyembunyikan data
Teknik penyembunyian berkisar **dari yang sederhana sampai yang sangat kompleks**:

1. **Mengubah nama file dan extension**
2. **Mengubur file jauh di dalam direktori yang tampak tidak berhubungan**
3. **Menyembunyikan file di dalam file** ← mulai serius
4. **Enkripsi** ← paling serius

Slide bilang: **dua teknik terakhir itulah yang bisa membuat praktisi digital forensics tidak bisa tidur di malam hari.**

### Enkripsi
**Enkripsi adalah konversi data ke suatu bentuk, yang disebut cipher text, yang tidak mudah dipahami oleh orang yang tidak berwenang.**

Empat istilah yang harus dibedakan:

| Istilah | Definisi |
| --- | --- |
| **Plain text** | Pesan asli yang belum dienkripsi. **Terbuka dan bisa dibaca siapa saja** |
| **Cipher text** | Versi **teracak** dari plain text, yang tidak bisa dipahami |
| **Algorithm** | **Metode** yang dipakai untuk mengenkripsi pesan |
| **Key** | **Data** yang dipakai untuk mengenkripsi dan mendekripsi informasi. **Password atau passphrase umumnya dipakai sebagai key** |

Alurnya: **plain text → (algoritma + key) → cipher text**.

**Enkripsi klasik** ada dua jenis: **Substitution** dan **Transposition**.

**Key space** adalah metrik yang sering dibahas saat bicara kekuatan suatu skema enkripsi. **Key space atau key length punya dampak langsung pada kemampuan kita memecahkan enkripsinya**, terutama dengan **brute force attack** — serangan yang mencoba memecahkan password dengan **mencoba setiap kemungkinan kombinasi kunci sampai yang benar ditemukan**.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa key space menentukan segalanya: menambah **satu bit** pada panjang kunci **menggandakan** jumlah kombinasi yang harus dicoba. Karena itu kunci 128-bit bukan "dua kali lebih kuat" dari 64-bit — dia sekitar **18 miliar miliar kali** lebih kuat. Inilah alasan slide berikutnya langsung beralih dari menyerang **kuncinya** ke menyerang **manusianya**: kuncinya tidak bisa dilawan, tapi orang yang memilihnya bisa.

### Memecahkan password
**Memecahkan password, atau cryptanalysis, bisa menakutkan atau praktis mustahil.** Untuk memberi peluang terbaik, kita perlu **memakai keuntungan apa pun yang bisa didapat**. Ada berbagai cara: **sebagian teknis, sebagian tidak** — dan slide menambahkan, **kadang sesederhana bertanya**.

Pilihannya mencakup **brute force attack, dictionary attack, dan reset password**. Ketiganya bisa membuahkan hasil positif.

Slide menegaskan prinsip penting: **menghindari enkripsi selalu lebih baik daripada harus menyerang passwordnya.**

Dan yang bekerja untuk keuntungan kita adalah **kerentanan yang dibawa manusia**:

> String panjang acak berisi huruf, angka, dan karakter itu **password yang sangat bagus**. Sayangnya, itu juga **sulit diingat orang**. Karena itu, **sebagian besar password didasarkan pada kata sungguhan, pola yang bisa dikenali, atau keduanya.**

### Steganography
**Steganography**, atau **stego** singkatnya, adalah cara lain yang **sangat efektif** untuk menyembunyikan data.

Asal katanya dari bahasa Yunani: **"Stegos"** yang berarti **tertutup** dan **"Graphie"** yang berarti **tulisan** — akarnya persis berarti **covered writing** (tulisan yang tertutup).

SearchSecurity.com mendefinisikan steganography sebagai **"penyembunyian sebuah pesan rahasia di dalam pesan biasa, dan pengambilannya kembali di tempat tujuan"** *(TechTarget, 2000)*.

> [!info] Konteks tambahan (bukan dari slide)
> Beda mendasar dari enkripsi, dan ini yang bikin stego berbahaya bagi examiner: **enkripsi menyembunyikan isi pesan, steganography menyembunyikan keberadaan pesan.** File terenkripsi kelihatan mencurigakan — kamu tahu ada sesuatu di situ, tinggal tidak bisa membacanya. File hasil stego kelihatan seperti **foto liburan biasa**, dan kamu bahkan tidak tahu harus curiga. Itu sebabnya stego jauh lebih sulit dilawan: masalahnya bukan memecahkannya, tapi **menyadari kamu perlu memecahkan sesuatu**.

### Penghancuran data
**Data destruction bisa dilakukan atau dicoba dengan beberapa cara, dan sebagiannya lebih baik dari yang lain.**

- **Software drive wiping** tersedia komersial dan **bisa efektif menghancurkan bukti potensial**. Efektivitasnya sangat bergantung pada **kualitas software-nya, cara pemakaiannya, dan jumlah "wipe" yang dilakukan**.
- **Defragmenting atau reformatting drive sering dicoba, tapi umumnya memberi hasil yang terbatas.**

**Drive wiping utility** dipakai untuk **menimpa data di hard drive sedemikian rupa sehingga tidak bisa dipulihkan**. Sebagian besar aplikasi ini **dipromosikan dan/atau ditujukan untuk menjaga informasi pribadi atau korporat tetap privat** — dua tujuan yang mulia. Sayangnya, **utility yang sama bisa dipakai untuk tujuan lain yang kurang terhormat**.

Contoh tool yang disebut slide: **Darik's Boot and Nuke, DiskWipe, CBL Data Shredder, Webroot Window Washer,** dan **Evidence Eliminator**.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa "defragment atau reformat memberi hasil terbatas" bagi yang berusaha menghapus: keduanya **cuma mengatur ulang atau membuat ulang metadata**, tidak menimpa data unit-nya. Setelah quick format, hampir semua data masih utuh di disk dan **bisa di-carve**. Ini kebalikan langsung dari materi di [[W03 - Disk and File System Analysis]]: kalau kamu paham bahwa yang dihapus itu **link antar lapisan**, kamu langsung paham kenapa format bukan penghapusan.
>
> Catat juga sisi lain yang tidak dibahas slide: **keberadaan tool wiping itu sendiri adalah artifact.** Menemukan "Evidence Eliminator" terinstal, atau jejaknya di registry, adalah temuan yang bermakna — bahkan (justru) ketika datanya sudah tidak bisa dipulihkan.

## Diagram & Visual
- **Slide 14 — ilustrasi terkait key space dan brute force**
  ![[99-Assets/Forensics/W08-slide14.png]]

> [!warning] Deck ini nyaris tidak punya gambar: cuma **satu** gambar non-dekoratif dari 22 slide. **Tidak ada satu pun screenshot output tool** — tidak ada contoh hasil `foremost`, isi `audit.txt`, maupun output `bulk_extractor`. Untuk praktikum, harus dicari sendiri.

## Rumus / Sintaks

Perintah carving dari slide:
```bash
foremost -i <forensic image> -o <output folder> --options
# hasilnya: folder output berisi audit.txt (detail temuan)

recoverjpeg /dev/sda
```

Alur enkripsi:
```
plain text --[ algorithm + key ]--> cipher text
                                    (key = umumnya password/passphrase)
```

Empat tingkat penyembunyian data, dari yang mudah dilawan ke yang sulit:
```
1. ganti nama / extension       -> dilawan dengan magic number (W02)
2. kubur di direktori tersembunyi -> dilawan dengan pencarian menyeluruh
3. file di dalam file            -> steganography, SULIT
4. enkripsi                      -> SULIT
```

## Pertanyaan Terbuka
- Slide 3 mencantumkan **empat learning outcome yang tidak dibahas**: **Fragmented File Carving**, **Validation of Carved Files**, **TRIM and Its Impact on Data Recovery (SSD)**, dan **PhotoRec**. **TRIM khususnya penting** — itu ancaman yang sudah disebut di [[W01 - Digital Forensic Fundamental]] dan artinya **carving di SSD sering gagal total**, tapi sampai deck ini pun tidak pernah dijelaskan mekanismenya. **Wajib ditanyakan.**
- **Fragmented file carving** adalah masalah terbesar carving di dunia nyata (file yang tidak tersimpan berurutan), dan sama sekali tidak disinggung. Deck ini seolah mengasumsikan semua file kontigu.
- **Validation of carved files** juga tidak dibahas — bagaimana kamu tahu file hasil carving itu utuh dan bukan sampah? Ini yang membedakan carving yang berguna dari yang menghasilkan ribuan file rusak.
- **Substitution dan Transposition** cuma disebut namanya di slide 13, **tanpa satu pun penjelasan atau contoh**.
- **Dictionary attack** disebut sebagai salah satu opsi tapi tidak pernah dijelaskan. Slide 15 menjanjikan "we'll dig into these attacks more in an upcoming section" — **section itu tidak pernah datang** di deck ini.
- **Steganography dijelaskan definisinya tapi tidak cara mendeteksinya.** Tidak ada tool steganalysis yang disebut.
- Perintah `recoverjpeg` di slide tidak konsisten: teksnya bilang "misalnya `sda1`" tapi perintahnya `recoverjpeg /dev/sda` (tanpa angka 1).
- Deck ini **tidak mencantumkan nama dosen** di slide judul, sama seperti [[W07 - Windows System Artifacts]].

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W02 - Key Technical Concepts]]
- [[W03 - Disk and File System Analysis]]
- [[W07 - Windows System Artifacts]]
- [[W09 - File and Archives Analysis]]
- [[Forensics - Review dan Glosari]]
