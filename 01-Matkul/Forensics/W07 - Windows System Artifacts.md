---
matkul: Forensics
minggu: 7
sks: 2
sumber: 7 Windows System Artifacts.pptx
tags: [kuliah/forensics, minggu/w07]
status: draft
diproses: 2026-09-03
---

# W07 — Windows System Artifacts

## Ringkasan
> - **Menekan tombol delete tidak melakukan apa pun terhadap datanya.** "Menghapus" file cuma memberi tahu komputer bahwa ruang itu **tersedia kalau dibutuhkan**. Datanya bertahan sampai ada file lain yang menimpanya.
> - Tiga mode tidur: **Sleep** (RAM saja), **Hibernation** (semua RAM **ditulis ke hard drive**), **Hybrid sleep** (campuran, untuk desktop). Yang punya nilai investigatif adalah **hibernation** — data jadi persisten.
> - **Registry** = "database untuk file konfigurasi" (definisi Microsoft TechNet), atau **sistem saraf pusat komputer**. Melacak konfigurasi dan preferensi user maupun sistem.
> - **Masalah atribusi**: kita bisa tahu istilah apa yang dicari di Google, tapi **sulit membuktikan siapa yang mengetiknya**. Jembatannya adalah **SID (security identifier)**, nomor unik tiap akun.
> - Dua rasa metadata: **application** dan **file system**. File system metadata = **Created, Modified, Accessed**.
> - **Restore point** = snapshot pengaturan sistem; **shadow copy** = sumber datanya, dan bisa menunjukkan **bagaimana sebuah file berubah dari waktu ke waktu**, bahkan menyimpan **salinan file yang sudah dihapus**.
> - Akuisisi memori Windows: **FTK Imager** (gratis, dari AccessData), hasilnya dianalisis di Kali dengan **Volatility**.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Deleted data | Data yang ruangnya ditandai tersedia, tapi isinya masih ada sampai ditimpa |
| Sleep | Mode hemat energi; data tetap di RAM, tujuannya cepat kembali beroperasi |
| Hibernation | Mode hemat daya untuk laptop; **semua data RAM ditulis ke hard drive** |
| Hybrid sleep | Campuran keduanya, terutama untuk desktop; daya minimal ke RAM **plus** menulis ke disk |
| `hiberfil.sys` | File hibernasi Windows (disebut di learning outcome) |
| Registry | Database konfigurasi Windows; melacak konfigurasi dan preferensi user dan sistem |
| Attribution | Masalah menghubungkan sebuah artifact ke **orang** tertentu |
| SID | Security Identifier; nomor unik tiap akun, dipakai melacak aksi |
| External drive artifact | Jejak bahwa perangkat penyimpanan eksternal pernah tersambung |
| Recycle Bin | "Tempat sampah" Windows; sering diandalkan user untuk menghapus bukti — keliru |
| Application metadata | Metadata yang dibuat aplikasi |
| File system metadata | Tanggal dan waktu file/folder dibuat, diakses, atau dimodifikasi |
| MRU | Most Recently Used; link pintasan ke aplikasi atau file yang baru dipakai |
| Restore Point (RP) | Snapshot pengaturan dan konfigurasi sistem pada satu momen |
| Shadow Copy | Sumber data untuk restore point; bisa berisi salinan file yang sudah dihapus |
| FTK Imager | Tool gratis AccessData untuk akuisisi live memori, paging file, dan drive image |
| Volatility | Tool analisis memori (dipakai di Kali Linux) |

## Isi

### Kenapa Windows penting
Slide membuka dengan permainan kata:

> Banyak yang bilang mata adalah jendela jiwa, tapi bagi forensic examiner, **Windows bisa jadi "jiwa" dari komputernya**.

Peluangnya tinggi bahwa examiner akan **lebih sering menemui Windows** daripada tidak saat melakukan investigasi. Kabar baiknya: **kita bisa memakai Windows itu sendiri sebagai tool** untuk memulihkan data dan melacak jejak yang ditinggalkan user. Karena itu, **examiner wajib punya pemahaman ekstensif tentang Windows dan semua fungsinya**.

### Deleted data
Bagian ini menjelaskan salah kaprah paling umum di dunia komputer:

> Bagi user rata-rata, menekan tombol delete memberi rasa aman yang memuaskan. Dengan satu klik mouse, kita mengira data kita **musnah selamanya**, tidak akan pernah lagi melihat cahaya matahari. **Pikir lagi.**
>
> **Menekan tombol delete tidak melakukan apa pun terhadap datanya.** File-nya **tidak pergi ke mana-mana**. "Menghapus" file **hanya memberi tahu komputer bahwa ruang yang ditempati file itu tersedia kalau komputer membutuhkannya**. Data yang dihapus **akan tetap ada sampai file lain ditulis menimpanya** — dan itu bisa memakan waktu cukup lama, **kalau memang terjadi sama sekali**.

> [!info] Konteks tambahan (bukan dari slide)
> Ini penjelasan level-user dari apa yang di [[W03 - Disk and File System Analysis]] dijelaskan secara struktural: yang dihapus adalah **link antar lapisan**, bukan data unit-nya. Empat kategori (deleted, orphaned, unallocated, overwritten) itu sebenarnya menjawab pertanyaan "**seberapa jauh proses penimpaan itu sudah berjalan?**".

### Hibernation file
Komputer kadang butuh istirahat dan bisa tidur siang seperti kita. Lewat proses "cybernap" ini, **lebih banyak bukti potensial bisa dihasilkan** — tergantung seberapa "dalam" PC-nya tertidur. Mode "deep sleep" seperti **hibernation** dan **hybrid sleep** **menyimpan data ke hard drive**, bukan sekadar menahannya di RAM (seperti "sleep").

Dan seperti kita tahu, **data yang ditulis ke drive lebih persisten dan bisa dipulihkan**. Sangat mungkin **file yang sudah dihapus tersangka masih bisa ditemukan di sini**.

Skenario yang dicontohkan slide:

> Katakanlah tersangka sedang mengerjakan dokumen yang memberatkan pada hari Senin. Dia harus pergi sebentar untuk menelepon. Dia menaruh laptopnya dalam mode hibernasi, yang **membuat komputer menyimpan semua yang sedang dia kerjakan ke hard drive**. Saat dia kembali 45 menit kemudian dan menghidupkan laptopnya lagi, semuanya persis seperti dia tinggalkan — **termasuk dokumen yang memberatkan itu**.

**Tiga mode tidur:**

| Mode | Ditujukan untuk | Yang terjadi pada data | Nilai forensik |
| --- | --- | --- | --- |
| **Sleep** | Menghemat energi **dan** membuat komputer kembali beroperasi secepat mungkin | Tetap di RAM | Rendah (volatile) |
| **Hibernation** | Mode hemat daya, **untuk laptop** bukan desktop | **Semua data di RAM ditulis ke hard drive** | **Tinggi** — di sinilah manfaat investigatif mulai terlihat, karena data di disk "jauh lebih sulit dihilangkan" |
| **Hybrid sleep** | Campuran kedua mode, **terutama untuk desktop** | Menjaga daya minimal ke RAM (mempertahankan data dan aplikasi) **sekaligus menulis data ke disk** | Tinggi |

> [!info] Konteks tambahan (bukan dari slide)
> Ini padanan Windows dari file **`sleepimage`** di macOS ([[W04 - Mac OS X System and Artifacts]]) — namanya **`hiberfil.sys`**, disebut di learning outcome tapi tidak pernah dibahas isinya di deck ini. Prinsipnya identik dan penting: **hibernation mengubah bukti volatile jadi non-volatile.** Kamu dapat isi RAM dari mesin yang sudah mati. Ini salah satu jalan memutar paling berharga terhadap masalah volatilitas di [[W05 - Data Acquisition]].

### Registry
Windows Registry memainkan peran krusial dalam operasi sebuah PC. **Microsoft TechNet mendefinisikan registry sebagai "sekadar sebuah database untuk file konfigurasi"**. Slide menawarkan deskripsi lain: **sistem saraf pusat komputer**.

Registry **melacak konfigurasi dan preferensi user maupun sistem** — bukan tugas sederhana. Dari sudut pandang forensik, dia bisa **menyediakan banyak sekali bukti potensial**.

### Masalah atribusi
Ini salah satu bagian paling penting secara konseptual di seluruh matkul, dan slide menyampaikannya dengan enak dibaca:

> Digital forensics bisa dipakai menjawab banyak pertanyaan, misalnya: **istilah apa yang dicari lewat Google?** Itu bisa kita temukan. **Apakah Bob yang mengetik istilah itu?** Houston, kita punya masalah.
>
> Sayangnya, **kita jarang bisa menempelkan jari seseorang ke keyboard** pada saat sebuah artifact dibuat. Kita mungkin perlu **mengungkap bukti lain untuk menyambungkan titik-titiknya**.

Contohnya: sebuah PC keluarga bisa punya akun terpisah untuk ibu, ayah, dan tiap anak, dan tiap akun bisa dilindungi password. **Tiap akun di mesin itu diberi nomor unik yang disebut security identifier (SID)**. Banyak aksi di komputer **diasosiasikan dengan, dan dilacak oleh, SID tertentu**. **Lewat SID inilah kita bisa mengikat sebuah akun ke aksi atau peristiwa tertentu.**

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan batas yang jujur di sini: SID mengikat aksi ke **akun**, bukan ke **orang**. Kalau password akun itu ditulis di sticky note (ingat "buku, catatan, dan potongan kertas" di [[W05 - Data Acquisition]]), atau kalau akunnya tidak berpassword, jembatan dari akun ke orang **putus**. Itu sebabnya pembelaan "bukan saya yang pakai komputernya" sering bisa dipakai — dan kenapa examiner butuh bukti korelatif (CCTV, log akses gedung, aktivitas ponsel) di luar mesin itu sendiri.

### External drive
Informasi punya nilai, kadang nilainya besar. Slide memberi contoh: **resep Coca-Cola tidak disimpan di balik kunci cuma untuk main-main.** Pencurian kekayaan intelektual adalah kekhawatiran besar.

Salah satu cara calon pencuri bisa dengan mudah menyelundupkan data keluar organisasi adalah lewat **perangkat penyimpanan eksternal seperti thumb drive**. Akibatnya, **examiner sering diminta menentukan apakah ada perangkat semacam itu yang pernah tersambung ke sebuah komputer**.

### Recycle Bin
"Tempat sampah" sudah jadi kehadiran yang akrab di desktop kita sejak sistem Macintosh awal. Idenya bagus, terutama dari perspektif user kasual: **user mungkin tidak paham sector dan byte, tapi hampir semua orang "paham" tempat sampah**.

Tapi slide menutup dengan kalimat yang bagus:

> Kadang, tempat sampah itu justru yang "menangkap" mereka. **Ini terutama benar ketika mereka mengandalkan tempat sampah untuk menghapus bukti mereka.**

### Metadata
Metadata paling sering didefinisikan sebagai **data tentang data**. Ada **dua rasa metadata**:

| Jenis | Isinya |
| --- | --- |
| **Application metadata** | Metadata yang dibuat aplikasi |
| **File system metadata** | **Tanggal dan waktu** file atau folder **dibuat, diakses, atau dimodifikasi** |

Kalau kamu klik kanan sebuah file dan pilih **"Properties"**, kamu bisa melihat timestamp ini.

Tiga timestamp file system menurut slide:

| Timestamp | Kapan diperbarui |
| --- | --- |
| **Created** | Sering menandakan **kapan file atau folder dibuat di media tertentu**, misalnya hard drive (Casey, 2009). **Bagaimana file itu sampai ke sana membuat perbedaan** — sebuah file bisa di-*save*, *copy*, *cut and paste*, atau *drag and drop* |
| **Modified** | Diset ketika file **diubah dengan cara apa pun lalu disimpan** (Casey, 2009) |
| **Accessed** | Diperbarui setiap kali file **diakses oleh file system**. Slide menekankan: **"Accessed" tidak berarti sama dengan "opened"** |

> [!info] Konteks tambahan (bukan dari slide)
> Dua peringatan di tabel itu adalah jebakan klasik ujian dan juga jebakan nyata di persidangan:
> - **"Bagaimana file itu sampai ke sana membuat perbedaan"** — file yang di-*copy* bisa punya *Created* yang **lebih baru** dari *Modified*-nya, yang kelihatannya mustahil sampai kamu sadar file itu disalin dari tempat lain.
> - **"Accessed ≠ opened"** — antivirus yang memindai, backup yang berjalan, bahkan mengarahkan kursor ke sebuah file bisa memperbarui *Accessed*. Menyimpulkan "tersangka membuka file ini" cuma dari timestamp *Accessed* adalah kesalahan yang bisa mematahkan seluruh kesaksianmu.
>
> Bandingkan dengan MAC times Linux di [[W06 - Linux System and Artifacts]] — Linux punya **Changed** yang tidak ada di daftar Windows ini.

### Most Recently Used (MRU)
Windows berusaha membuat hidup kita — setidaknya di komputer — senyaman mungkin. **MRU (Most Recently Used) list** adalah salah satu contohnya: **link yang berfungsi sebagai pintasan ke aplikasi atau file yang baru saja dipakai**. Bisa dilihat dengan mengklik tombol Start Windows, atau lewat menu file di banyak aplikasi.

### Restore Point dan Shadow Copy
Kadang lebih mudah (atau perlu) bagi komputer untuk kembali ke titik waktu sebelumnya saat semuanya masih berjalan baik. Di Windows, ini disebut **restore point (RP)** — slide menyebutnya **"mesin waktu untuk komputer kita"**.

**Restore point** adalah **snapshot dari pengaturan dan konfigurasi sistem kunci pada momen tertentu** (Microsoft Corporation). Snapshot ini bisa dipakai untuk mengembalikan sistem ke kondisi berfungsi. Restore point dibuat dengan cara berbeda-beda:
- **Otomatis oleh sistem** sebelum peristiwa sistem besar, seperti instalasi software
- **Terjadwal pada interval reguler**, misalnya mingguan

**Shadow copy menyediakan data sumber untuk restore point.** Seperti restore point, shadow file adalah artifact lain yang **sangat layak dilihat**. Kegunaannya:
- **Menunjukkan bagaimana sebuah file berubah dari waktu ke waktu**
- **Menyimpan salinan file yang sudah dihapus** (Larson, 2010)

> [!info] Konteks tambahan (bukan dari slide)
> Kemampuan "menunjukkan bagaimana file berubah dari waktu ke waktu" itu luar biasa berharga dan sering diremehkan. Kebanyakan artifact cuma memberimu **satu snapshot**: isi file **sekarang**. Shadow copy bisa memberimu **beberapa versi dari file yang sama pada waktu berbeda** — misalnya membuktikan bahwa sebuah dokumen kontrak diubah setelah ditandatangani. Ini bahan mentah paling kaya untuk [[W11 - Timeline Analysis and Correlation]].

### Akuisisi memori Windows
Ada beberapa tool untuk sistem Windows yang bisa dipakai untuk **menangkap memori dan paging file**. Forensic image hasilnya lalu **bisa dibuka di mesin Kali Linux untuk dianalisis dengan Volatility**.

**FTK Imager** dari **AccessData** adalah **tool gratis** untuk **akuisisi live memori, paging file, dan drive image**. Akuisisi live bisa dilakukan dengan FTK Imager untuk mengambil **RAM dan paging file**.

## Diagram & Visual
- **Slide 7 — ilustrasi terkait hibernation file**
  ![[99-Assets/Forensics/W07-slide07.png]]

> [!warning] **Slide 14 ("Metadata") isinya cuma judul + gambar** — kemungkinan besar screenshot dialog Properties Windows yang menunjukkan ketiga timestamp. Gambarnya tidak lolos ekstraksi. **Buka PPT aslinya di slide 14.**
>
> Slide 16 juga awalnya punya gambar dalam format WMF yang tidak didukung Obsidian, jadi tidak diikutkan.

## Rumus / Sintaks

Tiga mode tidur dan implikasi forensiknya:
```
Sleep         -> data di RAM saja            -> volatile, nilai rendah
Hibernation   -> SEMUA RAM ditulis ke disk   -> persisten, nilai TINGGI (hiberfil.sys)
Hybrid sleep  -> RAM + ditulis ke disk       -> persisten, nilai tinggi
```

Timestamp file system Windows:
```
Created   : kapan file dibuat di media INI (cara file sampai ke sana berpengaruh)
Modified  : kapan isi file diubah lalu disimpan
Accessed  : kapan file diakses filesystem  -- BUKAN berarti "dibuka"
```

## Pertanyaan Terbuka
- Slide 3 mencantumkan **sepuluh learning outcome**, tapi **enam di antaranya tidak dibahas sama sekali**: **Registry Analysis** (registry cuma didefinisikan, tidak ada satu pun key yang ditunjukkan), **`hiberfil.sys` Analysis**, **Prefetch Files Analysis**, **Event Logs Analysis**, **User Activity Artefacts (Jump Lists, RecentDocs, Timeline)**, dan **Link Files (.lnk) Analysis**. Ini **lubang materi terbesar di seluruh matkul** — enam topik yang semuanya inti forensik Windows. **Prioritas nomor satu untuk ditanyakan ke dosen.**
- **Registry dijelaskan pentingnya tapi tidak dijelaskan strukturnya.** Tidak ada hive (SAM, SYSTEM, SOFTWARE, NTUSER.DAT), tidak ada key, tidak ada contoh. Padahal "Registry Analysis" adalah learning outcome tersendiri.
- **External drive artifact** dijelaskan *kenapa* penting, tapi **tidak dijelaskan di mana mencarinya** (yang jawabannya ada di registry — key USBSTOR).
- **Recycle Bin** dibahas satu slide penuh secara retoris, tapi **tidak ada informasi teknis sama sekali**: di mana lokasinya, apa itu `$I`/`$R` file, bagaimana memulihkan isinya.
- Windows tidak punya timestamp **Changed** seperti Linux, tapi NTFS sebenarnya menyimpan **empat** timestamp (MACE / MACB, termasuk *Entry Modified*). Slide cuma menyebut tiga. Perlu dikonfirmasi mana yang diujikan.
- **Volatility** disebut sebagai tool analisis memori dengan catatan "as we'll delve into in a later chapter", tapi **tidak ada deck di matkul ini yang membahasnya**. Cek [[W13 - Automating Analysis and Timeline Analysis]].
- Slide ini **satu-satunya deck yang tidak mencantumkan nama dosen di slide judul** (sementara slide 1 deck lain menulis "Ika Dyah A.R."). Tidak berpengaruh ke materi, cuma catatan.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Forensics]]
- [[W03 - Disk and File System Analysis]]
- [[W05 - Data Acquisition]]
- [[W06 - Linux System and Artifacts]]
- [[W08 - File Recovery and Data Carving]]
- [[W11 - Timeline Analysis and Correlation]]
- [[Forensics - Review dan Glosari]]
