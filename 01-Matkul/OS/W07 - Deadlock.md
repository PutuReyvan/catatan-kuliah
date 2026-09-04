---
matkul: Operating Systems
minggu: 7
sks: 3
sumber: Deadlock.pptx
tags: [kuliah/os, minggu/w07]
status: draft
diproses: 2026-09-04
---

# W07 — Deadlock

## Ringkasan
> - **Deadlock** = pemblokiran PERMANEN sekumpulan proses yang bersaing memperebutkan resource atau saling berkomunikasi — masing-masing menunggu event yang HANYA BISA dipicu oleh proses lain yang JUGA sedang terblokir. Sifatnya permanen karena TIDAK ADA event yang pernah terpicu.
> - Deadlock terjadi kalau **4 kondisi** terpenuhi SEKALIGUS: **Mutual Exclusion, Hold-and-Wait, No-Preemption, Circular Wait.** Cegah SATU dari empat ini, deadlock tidak bisa terjadi.
> - Tiga strategi besar menangani deadlock: **Prevention** (rancang sistem supaya deadlock TIDAK MUNGKIN terjadi), **Avoidance** (izinkan 4 kondisi ada, tapi buat keputusan CERDAS supaya titik deadlock tidak pernah tercapai — pakai **Banker's Algorithm**), dan **Detection & Recovery** (biarkan deadlock terjadi, lalu DETEKSI dan PULIHKAN).
> - **Dining Philosophers Problem** adalah studi kasus klasik deadlock: filsuf bergiliran makan dengan garpu bersama, ilustrasi sempurna keempat kondisi deadlock sekaligus.
> - Tiap OS punya mekanisme sendiri untuk concurrency: Linux pakai **spinlock**, Windows pakai **synchronization object**, Android pakai **Binder** (RPC lightweight antar proses).

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Deadlock | Pemblokiran permanen sekumpulan proses yang saling menunggu event yang tak pernah terpicu |
| Mutual Exclusion (kondisi deadlock) | Setiap resource cuma bisa dipegang SATU proses atau tersedia |
| Hold-and-Wait | Proses yang sudah pegang resource bisa MINTA resource baru lagi |
| No-Preemption | Resource yang sudah diberikan TIDAK BISA direbut paksa |
| Circular Wait | Ada rantai melingkar proses yang saling menunggu resource satu sama lain |
| Banker's Algorithm | Algoritma deadlock avoidance yang menjaga sistem selalu di "safe state" |
| Safe State | State di mana ADA setidaknya satu urutan alokasi resource yang tidak berujung deadlock |

## Isi

### Apa Itu Deadlock
**Deadlock** adalah pemblokiran PERMANEN sekumpulan proses yang bersaing memperebutkan sumber daya sistem atau saling berkomunikasi. Sekumpulan proses dikatakan deadlock kalau SETIAP proses dalam set itu terblokir menunggu event yang HANYA BISA dipicu oleh proses lain yang JUGA terblokir dalam set yang sama. Sifatnya PERMANEN karena TIDAK ADA satu pun event yang pernah terpicu. **Tidak ada solusi efisien untuk kasus umum (general case).**

> [!info] Analogi
> Deadlock itu seperti dua mobil di jalan sempit satu arah yang saling berhadapan, masing-masing menunggu mobil LAIN mundur duluan supaya bisa lewat. Kalau kedua sopir sama-sama keras kepala menunggu yang lain mengalah, mereka akan terjebak DI SITU SELAMANYA — tidak ada satu pun yang akan mundur sendiri karena masing-masing menunggu tindakan pihak lain lebih dulu.

### Contoh Deadlock: Memory Request
Dengan ruang tersedia 200KB untuk dialokasikan, urutan kejadian tertentu (dua proses masing-masing meminta memori secara bertahap) bisa menghasilkan **deadlock kalau kedua proses maju ke permintaan KEDUA mereka** — masing-masing sudah memegang sebagian memori dan menunggu proses lain melepas memorinya, sementara tidak ada yang cukup untuk memenuhi permintaan kedua siapa pun.

### Contoh Deadlock: Consumable Resources
Bayangkan sepasang proses, di mana MASING-MASING mencoba MENERIMA pesan dari proses lain terlebih dahulu, BARU MENGIRIM pesan ke proses lain. **Deadlock terjadi kalau operasi Receive bersifat BLOCKING** — kedua proses sama-sama menunggu menerima pesan dari yang lain, tapi tidak ada yang pernah MENGIRIM karena masing-masing terjebak di langkah "menerima" duluan.

### Empat Kondisi Deadlock (Coffman Conditions)
Empat kondisi ini HARUS terpenuhi SEKALIGUS agar deadlock terjadi:

1. **Mutual Exclusion Condition** — setiap resource pada satu waktu HANYA dipegang oleh SATU proses, atau tersedia (available).
2. **Hold-and-Wait Condition** — proses yang SUDAH memegang resource (dari permintaan sebelumnya) bisa meminta resource BARU lagi.
3. **No-Preemption Condition** — resource yang sudah diberikan TIDAK BISA direbut paksa dari sebuah proses. Harus DILEPAS SECARA EKSPLISIT oleh proses yang memegangnya.
4. **Circular Wait Condition** — harus ada DAFTAR MELINGKAR dari dua atau lebih proses, di mana masing-masing menunggu resource yang dipegang anggota BERIKUTNYA dalam rantai itu.

### Resource Allocation Graph (RAG)
**Resource Allocation Graph** adalah cara visual memodelkan deadlock — menampilkan tiga situasi: **(a) Holding a resource** (proses memegang resource), **(b) Requesting a resource** (proses meminta resource), dan **(c) Deadlock** (siklus di graf yang menunjukkan circular wait). RAG juga dipakai untuk **deteksi deadlock** — kalau ada SIKLUS di grafnya, itu indikasi kuat deadlock.

### Tiga Strategi Utama Menangani Deadlock

**1. Deadlock Prevention**
Merancang sistem supaya kemungkinan deadlock DIHILANGKAN SAMA SEKALI. Dua metode utama:
- **Indirect** — mencegah terjadinya SALAH SATU dari 3 kondisi pertama (Mutual Exclusion, Hold-and-Wait, No-Preemption).
- **Direct** — mencegah terjadinya CIRCULAR WAIT secara langsung.

**Strategi per kondisi:**
- **Mutual exclusion** — kalau akses ke resource BUTUH mutual exclusion, itu harus didukung OS. Beberapa resource (misal file) mengizinkan akses MULTIPEL untuk baca, tapi hanya akses EKSKLUSIF untuk tulis. Deadlock TETAP BISA terjadi kalau lebih dari satu proses butuh izin TULIS.
- **Hold and wait** — bisa dicegah dengan MEWAJIBKAN proses meminta SEMUA resource yang dibutuhkan SEKALIGUS di awal, memblokir proses sampai SEMUA permintaan bisa dipenuhi bersamaan.
- **No Preemption** — kalau proses yang memegang resource tertentu DITOLAK permintaan lebih lanjut, proses itu harus MELEPAS resource aslinya dan meminta lagi dari awal. Alternatifnya, OS bisa PREEMPT (merebut paksa) proses kedua dan memaksanya melepas resource-nya.
- **Circular Wait** — dicegah dengan mendefinisikan URUTAN LINEAR (linear ordering) untuk tipe-tipe resource — proses HANYA BOLEH meminta resource sesuai urutan itu, sehingga siklus melingkar tidak mungkin terbentuk.

**2. Deadlock Avoidance**
Mengizinkan KETIGA kondisi (Mutual Exclusion, Hold-and-Wait, No-Preemption) TETAP ADA, tapi membuat keputusan CERDAS untuk memastikan titik deadlock TIDAK PERNAH tercapai. Keputusan dibuat SECARA DINAMIS: apakah permintaan alokasi resource saat ini, KALAU DIKABULKAN, berpotensi mengarah ke deadlock. Butuh PENGETAHUAN tentang permintaan proses di MASA DEPAN.

**Banker's Algorithm** — pendekatan "resource allocation denial":
- **State** sistem merefleksikan alokasi resource SAAT INI ke proses-proses.
- **Safe state** — state di mana ADA setidaknya SATU urutan alokasi resource ke proses yang TIDAK berujung deadlock.
- **Unsafe state** — state yang BUKAN safe state.

Contoh dengan 10 resource jenis sama: slide menunjukkan demonstrasi konkret state yang SAFE (ada jalan keluar aman) dan state yang UNSAFE (tidak ada jalan keluar aman) — algoritma bekerja dengan mensimulasikan SETIAP kemungkinan urutan alokasi untuk memastikan sistem SELALU berada di safe state sebelum benar-benar mengabulkan sebuah permintaan.

> [!info] Analogi
> Banker's Algorithm itu persis seperti namanya — cara kerja BANK memberi pinjaman. Bank tidak akan memberi pinjaman kalau itu membuat SEMUA uang tunai bank habis TANPA jaminan bisa membayar kembali nasabah lain yang butuh uang. Sebelum menyetujui SETIAP pinjaman (permintaan resource), bank mengecek: "kalau saya kasih pinjaman ini, apakah saya MASIH PUNYA cara untuk memenuhi semua kewajiban lain nanti?" Kalau jawabannya ya (safe state), pinjaman disetujui. Kalau tidak (unsafe state), pinjaman DITOLAK dulu, meski uangnya SECARA TEKNIS tersedia sekarang.

**Kelebihan Deadlock Avoidance:**
- Tidak perlu PREEMPT dan ROLLBACK proses, seperti di deadlock detection.
- LEBIH TIDAK RESTRIKTIF dibanding deadlock prevention.

**3. Deadlock Detection dan Recovery**
Alih-alih mencegah, sistem MEMBIARKAN deadlock bisa terjadi, lalu mendeteksinya dan pulih. Pengecekan deadlock bisa dilakukan SESERING SETIAP permintaan resource, atau LEBIH JARANG, tergantung seberapa mungkin deadlock terjadi.

**Strategi Recovery (pemulihan):**
- Abort SEMUA proses yang deadlock.
- Backup setiap proses deadlock ke checkpoint yang sudah ditentukan sebelumnya dan restart SEMUA proses.
- Abort proses deadlock SATU PER SATU sampai deadlock tidak ada lagi.
- Preempt resource SATU PER SATU sampai deadlock tidak ada lagi.

### Integrated Deadlock Strategy
Alih-alih merancang fasilitas OS yang HANYA memakai SATU strategi, lebih efisien memakai STRATEGI BERBEDA untuk situasi berbeda. Resource dikelompokkan jadi beberapa KELAS, dengan strategi linear ordering dipakai untuk mencegah deadlock ANTAR kelas resource, sementara DI DALAM tiap kelas dipakai algoritma yang paling sesuai untuk kelas itu.

**Empat kelas resource dan strategi yang cocok:**
| Kelas Resource | Contoh | Strategi Paling Sesuai |
| --- | --- | --- |
| **Swappable space** | Blok memori di secondary storage untuk swapping | Prevention (alokasikan SEMUA kebutuhan di awal) — masuk akal kalau kebutuhan storage maksimal SUDAH DIKETAHUI |
| **Process resources** | Perangkat yang bisa ditugaskan (tape drive), file | Avoidance sering efektif — wajar mengharapkan proses MENGUMUMKAN kebutuhan resource-nya di muka; Prevention lewat resource ordering juga bisa |
| **Main memory** | Dialokasikan ke proses dalam page/segmen | Prevention lewat PREEMPTION — proses yang di-preempt cukup di-swap ke secondary memory, membebaskan ruang untuk resolve deadlock |
| **Internal resources** | Misalnya I/O channel | Prevention lewat resource ordering |

### Dining Philosophers Problem
Studi kasus klasik: sekelompok filsuf duduk mengelilingi meja bundar, masing-masing perlu DUA GARPU (yang dibagi dengan tetangga kiri-kanan) untuk makan. **Dua syarat yang harus dipenuhi:**
1. TIDAK ADA dua filsuf yang boleh memakai garpu yang SAMA secara bersamaan (mutual exclusion).
2. TIDAK ADA filsuf yang boleh mati kelaparan (hindari deadlock DAN starvation).

> [!info] Konteks tambahan (bukan dari slide)
> Dining Philosophers ini ilustrasi SEMPURNA keempat kondisi deadlock: garpu itu **mutual exclusion** (satu garpu, satu pemegang), filsuf yang sudah pegang garpu kiri MENUNGGU garpu kanan itu **hold-and-wait**, garpu tidak bisa direbut paksa dari filsuf lain itu **no-preemption**, dan kalau SEMUA filsuf mengambil garpu KIRI mereka secara bersamaan lalu menunggu garpu KANAN, itu **circular wait** — deadlock sempurna, semua filsuf kelaparan selamanya menunggu garpu yang tidak akan pernah dilepas.

### Mekanisme Concurrency per OS
- **Linux — Spinlocks.** Teknik PALING UMUM untuk melindungi critical section. Hanya bisa diperoleh SATU thread pada satu waktu. Thread lain akan TERUS MENCOBA (spinning) sampai bisa memperoleh lock. Dibangun di atas lokasi integer di memori yang dicek tiap thread sebelum masuk critical section-nya. Efektif untuk situasi di mana waktu tunggu lock diperkirakan SANGAT SINGKAT. Kekurangan: thread yang terkunci TERUS BEREKSEKUSI dalam mode busy-waiting (boros CPU kalau waktu tunggunya ternyata lama).

- **Windows** — menyediakan sinkronisasi antar thread sebagai bagian dari arsitektur OBJECT (synchronization object).

- **Android — Binder.** Kernel Android menambahkan kapabilitas baru bernama **Binder** — menyediakan kapabilitas **RPC (Remote Procedure Call) ringan** yang efisien dari sisi memori dan pemrosesan. Juga dipakai untuk memediasi SEMUA interaksi antara dua proses. Mekanisme RPC ini bekerja antara dua proses di sistem yang SAMA tapi berjalan di VIRTUAL MACHINE berbeda. Metode komunikasi dengan Binder memakai system call `ioctl` — system call serba-guna untuk operasi I/O spesifik-device.

### Contoh Deadlock dengan pthread_mutex
Slide menunjukkan contoh kode nyata deadlock memakai `pthread_mutex_lock` dan `pthread_mutex_unlock` — pola klasik: dua thread masing-masing mengunci mutex dalam URUTAN YANG BERBEDA (thread A kunci mutex1 lalu coba kunci mutex2; thread B kunci mutex2 lalu coba kunci mutex1) menghasilkan deadlock karena masing-masing menunggu mutex yang dipegang yang lain.

> [!info] Konteks tambahan (bukan dari slide)
> Ini contoh nyata kenapa "linear ordering" (strategi mencegah Circular Wait) penting dalam praktik pemrograman sehari-hari: kalau SEMUA thread SEPAKAT selalu mengunci mutex1 SEBELUM mutex2 (urutan konsisten, bukan urutan sembarangan), circular wait seperti ini TIDAK AKAN PERNAH terjadi.

## Diagram & Visual
- **Slide 5 — Ilustrasi Situasi Deadlock**
  ![[99-Assets/OS/W07-slide05.png]]
- **Slide 8 — Contoh Deadlock: Memory Request**
  ![[99-Assets/OS/W07-slide08.png]]
- **Slide 9 — Contoh Deadlock: Consumable Resources**
  ![[99-Assets/OS/W07-slide09.png]]
- **Slide 11 — Pendekatan-pendekatan Deadlock**
  ![[99-Assets/OS/W07-slide11.png]]
- **Slide 12 — Resource Allocation Graph**
  ![[99-Assets/OS/W07-slide12.jpg]]
- **Slide 13 — Deteksi Deadlock memakai RAG**
  ![[99-Assets/OS/W07-slide13.png]]
- **Slide 14 — Cara Menghindari Deadlock**
  ![[99-Assets/OS/W07-slide14.png]]
  ![[99-Assets/OS/W07-slide14a.png]]
- **Slide 19 — Dua Pendekatan Deadlock Avoidance**
  ![[99-Assets/OS/W07-slide19.png]]
- **Slide 21-22 — Demonstrasi Safe State dan Unsafe State**
  ![[99-Assets/OS/W07-slide21.jpg]]
  ![[99-Assets/OS/W07-slide22.jpg]]
- **Slide 23-27 — Banker's Algorithm langkah demi langkah**
  ![[99-Assets/OS/W07-slide23.png]]
  ![[99-Assets/OS/W07-slide24.png]]
  ![[99-Assets/OS/W07-slide25.png]]
  ![[99-Assets/OS/W07-slide26.png]]
  ![[99-Assets/OS/W07-slide27.png]]
- **Slide 35 — Ilustrasi Dining Philosophers Problem**
  ![[99-Assets/OS/W07-slide35.png]]
- **Slide 39 — Windows Synchronization Objects**
  ![[99-Assets/OS/W07-slide39.png]]
- **Slide 41 — Binder Operation (Android)**
  ![[99-Assets/OS/W07-slide41.png]]
- **Slide 42 — Contoh Kode Deadlock dengan pthread_mutex**
  ![[99-Assets/OS/W07-slide42.png]]
  ![[99-Assets/OS/W07-slide42a.png]]

> [!warning] Slide 7 (contoh deadlock) gagal diekstrak — gambar rusak/tidak lengkap di file PPT sumbernya (error: `required <p:blipFill> child element not present`). Buka file PPT asli untuk isinya.

## Rumus / Sintaks
```
4 Kondisi Deadlock (semua harus terpenuhi sekaligus):
1. Mutual Exclusion   <- resource dipegang eksklusif SATU proses
2. Hold-and-Wait       <- proses pegang resource, minta resource baru lagi
3. No-Preemption       <- resource tidak bisa direbut paksa
4. Circular Wait       <- ada rantai melingkar proses saling menunggu

3 Strategi Penanganan:
- Prevention  -> cegah SALAH SATU dari 4 kondisi di atas
- Avoidance   -> izinkan 4 kondisi ada, tapi jaga sistem selalu di Safe State (Banker's Algorithm)
- Detection & Recovery -> biarkan terjadi, deteksi lewat RAG, lalu abort/preempt/rollback
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Rollback** | Mengembalikan proses ke checkpoint sebelumnya untuk pemulihan dari deadlock |
| **Linear ordering** | Urutan tetap untuk meminta tipe resource, mencegah circular wait |
| **Spinlock** | Lock di Linux di mana thread yang gagal terus mencoba (spinning) memperolehnya |
| **RPC (Remote Procedure Call)** | Mekanisme memanggil prosedur di proses/mesin lain seolah lokal |
| **ioctl** | System call serba-guna untuk operasi I/O spesifik-device di UNIX/Linux |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis Dining Philosophers Problem: identifikasi MASING-MASING dari 4 kondisi deadlock (Mutual Exclusion, Hold-and-Wait, No-Preemption, Circular Wait) dan tunjukkan bagaimana masing-masing muncul secara konkret dalam skenario filsuf dan garpu ini.
2. **(C4 – Analisis)** Bandingkan Deadlock Prevention dan Deadlock Avoidance dari sisi FLEKSIBILITAS sistem. Analisis: kenapa slide menyebut Avoidance "less restrictive" daripada Prevention — apa yang DIKORBANKAN Prevention demi menjamin deadlock TIDAK MUNGKIN terjadi sama sekali?
3. **(C5 – Evaluasi)** Sebuah sistem OS memilih strategi "Detection and Recovery" untuk resource yang JARANG sekali menyebabkan deadlock, tapi kalau terjadi, dampaknya BESAR (misalnya kehilangan banyak data proses yang di-abort). Evaluasi: apakah pilihan strategi ini tepat, atau apakah "Prevention" akan lebih masuk akal meski lebih restriktif? Pertimbangkan trade-off antara probabilitas rendah tapi dampak besar.
4. **(C5 – Evaluasi)** Bandingkan contoh deadlock pthread_mutex (dua thread mengunci mutex1/mutex2 dalam urutan BERBEDA) dengan solusi "linear ordering" untuk mencegah Circular Wait. Evaluasi: seberapa PRAKTIS solusi linear ordering ini diterapkan di proyek software BESAR dengan RATUSAN mutex dan BANYAK programmer berbeda — apa tantangan koordinasinya?
5. **(C6 – Cipta)** Rancang solusi pseudocode untuk Dining Philosophers Problem yang MENCEGAH deadlock memakai strategi "resource ordering" (salah satu strategi Direct Prevention untuk Circular Wait dari materi ini) — misalnya dengan aturan "filsuf harus SELALU mengambil garpu bernomor LEBIH KECIL dulu sebelum garpu bernomor lebih besar". Jelaskan kenapa aturan ini mencegah circular wait terbentuk.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W06 - Synchronization dan Inter-process Communication]]
- [[W08 - File Systems]]
- [[OS - Review dan Glosari]]
