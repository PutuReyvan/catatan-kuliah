---
matkul: Operating Systems
minggu: 13
sks: 3
sumber: Week14-Pert13-Virtualization.pptx
tags: [kuliah/os, minggu/w13]
status: draft
diproses: 2026-09-04
---

# W13 — Virtualization

> [!note] Nama file sumber deck ini tertulis "Week14-Pert13" — ada mismatch antara "Week14" dan "Pert13". Isi internal slide 1 secara eksplisit menulis **"Session 13"**, dan ini konsisten sebagai kelanjutan langsung dari [[W12 - Security]] (Session 12) — jadi note ini tetap dinomori **W13** mengikuti label "Session" internal dan urutan kurikulum, bukan angka "Week14" di nama file.

## Ringkasan
> - **Virtualisasi** memungkinkan SATU PC/server menjalankan BANYAK OS atau banyak sesi OS yang sama secara SIMULTAN. Komponen kuncinya: **VMM (Virtual Machine Monitor)** atau **hypervisor** — software yang duduk di ANTARA hardware dan VM, bertindak sebagai "resource broker".
> - Ada **Type 1 hypervisor** (bare-metal, langsung di atas hardware) dan **Type 2 hypervisor** (berjalan DI ATAS OS host yang sudah ada).
> - **Paravirtualization** = teknik virtualisasi berbantuan SOFTWARE, guest OS punya driver khusus supaya bekerja lebih efisien dengan hypervisor. **Hardware-assisted virtualization** = prosesor (AMD-V, Intel VT-x) punya instruksi KHUSUS untuk mempercepat virtualisasi.
> - **Container virtualization** (Docker, dll.) BEDA dari VM — container berbagi KERNEL OS host yang sama, jauh lebih RINGAN dari VM penuh yang masing-masing punya OS lengkap sendiri.
> - **JVM (Java Virtual Machine)** adalah contoh virtualisasi di level APLIKASI — "Write Once, Run Anywhere" — sementara **Linux VServer** adalah pendekatan virtualisasi ringan yang HANYA memakai SATU salinan kernel Linux untuk banyak "virtual server".

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Hypervisor (VMM) | Software yang mengelola dan menjembatani hardware dengan Virtual Machine |
| Type 1 Hypervisor | Hypervisor bare-metal, berjalan langsung di atas hardware |
| Type 2 Hypervisor | Hypervisor yang berjalan di atas OS host yang sudah ada |
| Paravirtualization | Virtualisasi dengan bantuan driver khusus di guest OS untuk efisiensi |
| Container | Virtualisasi level OS yang berbagi kernel host, lebih ringan dari VM |
| JVM (Java Virtual Machine) | Mesin virtual level aplikasi untuk menjalankan kode Java lintas platform |
| Ballooning | Teknik hypervisor memaksa guest OS melepas memori yang tidak dipakai |

## Isi

### Virtual Machines (VM)
Teknologi virtualisasi memungkinkan SATU PC atau server menjalankan BANYAK operating system atau BANYAK sesi dari SATU OS yang sama SECARA SIMULTAN. Mesin dengan software virtualisasi bisa meng-HOST banyak aplikasi, termasuk yang berjalan di OS BERBEDA, di satu platform. Host OS bisa mendukung sejumlah VM, masing-masing punya karakteristik OS tertentu dan (di beberapa versi virtualisasi) karakteristik platform HARDWARE tertentu.

Solusi yang memungkinkan virtualisasi adalah **Virtual Machine Monitor (VMM)**, atau **hypervisor**. Software ini duduk DI ANTARA hardware dan VM, bertindak sebagai **resource broker** (perantara sumber daya).

> [!info] Analogi
> Hypervisor itu seperti seorang RESEPSIONIS APARTEMEN yang mengelola satu bangunan besar (hardware fisik) untuk banyak PENYEWA (VM) yang masing-masing punya "rumah" (OS) sendiri-sendiri. Resepsionis itu memastikan setiap penyewa dapat AKSES ADIL ke fasilitas bersama (listrik, air, CPU, memori), TANPA satu penyewa bisa masuk ke unit penyewa lain — masing-masing "merasa" seolah memiliki bangunan itu sendirian, padahal sebenarnya berbagi infrastruktur fisik yang SAMA.

### Kenapa Virtualisasi?
Beberapa alasan utama memakai virtualisasi:
1. **Legacy hardware** — aplikasi yang dibangun untuk hardware LAMA tetap bisa dijalankan lewat virtualisasi hardware lama itu, memungkinkan hardware fisik LAMA pensiun.
2. **Rapid deployment** — VM baru bisa di-deploy dalam hitungan MENIT.
3. **Versatility** — pemakaian hardware bisa dioptimalkan dengan MEMAKSIMALKAN jenis aplikasi yang bisa ditangani satu komputer.
4. **Consolidation** — resource berkapasitas besar/kecepatan tinggi bisa dipakai LEBIH EFISIEN dengan dibagi antar banyak aplikasi SEKALIGUS.
5. **Aggregating** — virtualisasi memudahkan MENGGABUNGKAN banyak resource jadi SATU resource virtual (contoh: storage virtualization).
6. **Dynamics** — resource hardware bisa dialokasikan secara DINAMIS, meningkatkan load balancing dan fault tolerance.
7. **Ease of management** — VM memudahkan DEPLOYMENT dan TESTING software.
8. **Increased availability** — host VM di-CLUSTER bersama membentuk POOL sumber daya komputasi.

### Hypervisor: Fungsi dan Jenis
**Fungsi utama hypervisor:**
- Manajemen EKSEKUSI VM.
- EMULASI device dan kontrol akses.
- Eksekusi operasi PRIVILEGED oleh hypervisor atas nama guest VM.
- Manajemen VM (disebut juga **VM lifecycle management**).
- Administrasi platform dan software hypervisor itu sendiri.

**Type 1 dan Type 2 Hypervisor:**
- **Type 1 (bare-metal)** — hypervisor berjalan LANGSUNG di atas hardware, TANPA OS host di bawahnya.
- **Type 2 (hosted)** — hypervisor berjalan DI ATAS OS host yang sudah ada, seperti aplikasi biasa.

### Paravirtualization
Teknik virtualisasi berbantuan SOFTWARE yang memakai API khusus untuk menghubungkan VM dengan hypervisor untuk MENGOPTIMALKAN performa. Guest OS (Linux atau Microsoft Windows) punya dukungan paravirtualization KHUSUS sebagai bagian dari kernel-nya, plus driver paravirtualization spesifik yang mengizinkan OS dan hypervisor bekerja SAMA-SAMA lebih efisien dengan overhead translasi hypervisor. Dukungan ini sudah ditawarkan sebagai bagian dari banyak distribusi Linux umum sejak **2008**.

### Hardware-Assisted Virtualization
Pembuat prosesor **AMD** dan **Intel** menambahkan fungsionalitas ke prosesor mereka untuk meningkatkan performa dengan hypervisor. **AMD-V** dan **Intel VT-x** adalah ekstensi hardware-assisted virtualization yang bisa dimanfaatkan hypervisor selama pemrosesan. Prosesor Intel menawarkan instruction set tambahan bernama **Virtual Machine Extensions (VMX)**. Dengan sebagian instruksi ini menjadi bagian PROSESOR, hypervisor TIDAK PERLU LAGI menjaga fungsi-fungsi ini sebagai bagian dari software-nya sendiri — meningkatkan performa secara signifikan.

### Virtual Appliance
**Virtual appliance** adalah software MANDIRI yang bisa didistribusikan sebagai image VM. Terdiri dari sekumpulan aplikasi TERPAKET dan guest OS. INDEPENDEN dari arsitektur hypervisor atau prosesor, bisa berjalan di Type 1 MAUPUN Type 2 hypervisor. Virtual appliance jadi cara DE-FACTO distribusi software, menciptakan kebutuhan "vendor virtual appliance".

**Security Virtual Appliance (SVA)** — perkembangan penting: alat keamanan yang memonitor dan MELINDUNGI VM lain, berjalan DI LUAR VM-VM itu di dalam sebuah VM khusus yang di-hardening keamanannya. SVA mendapat visibilitas ke state VM serta trafik jaringan ANTAR VM (dan antara VM dan hypervisor) lewat **virtual machine introspection API** milik hypervisor.

**Keuntungan SVA:**
- TIDAK rentan terhadap cacat di Guest OS.
- INDEPENDEN dari konfigurasi jaringan virtual — tidak perlu dikonfigurasi ulang setiap kali konfigurasi jaringan virtual berubah karena migrasi VM atau perubahan konektivitas antar VM di host hypervisor.

### Container Virtualization
Container adalah pendekatan virtualisasi yang BERBEDA dari VM tradisional — sementara VM masing-masing punya OS LENGKAP sendiri, container BERBAGI kernel OS HOST yang sama, membuatnya jauh lebih RINGAN dan CEPAT dijalankan.

> [!info] Konteks tambahan (bukan dari slide)
> Perbedaan mendasar VM vs Container: **VM** memvirtualisasikan HARDWARE — setiap VM punya kernel OS-nya SENDIRI, jadi kalau kamu jalankan 5 VM Linux, ada 5 SALINAN kernel Linux berjalan. **Container** memvirtualisasikan di level OS — SEMUA container di satu host BERBAGI SATU kernel yang sama (milik host), hanya proses/file system/network-nya yang diisolasi. Ini yang membuat container jauh lebih RINGAN startup-nya (detik, bukan menit) dan lebih HEMAT resource dibanding VM penuh.

### Microservices
NIST SP 800-180 mendefinisikan **microservice** sebagai: *"elemen dasar yang dihasilkan dari dekomposisi arsitektural komponen sebuah aplikasi jadi pola-pola yang LONGGAR TERIKAT (loosely coupled), terdiri dari layanan MANDIRI yang berkomunikasi satu sama lain memakai protokol komunikasi STANDAR dan sekumpulan API yang TERDEFINISI JELAS, INDEPENDEN dari vendor, produk, atau teknologi apa pun."*

### Docker
Menyediakan cara yang LEBIH SEDERHANA dan TERSTANDARISASI untuk menjalankan container. Container Docker juga berjalan di Linux. Salah satu alasan Docker LEBIH POPULER dibanding container pesaing: kemampuannya MEMUAT image container di host OS dengan cara SEDERHANA dan CEPAT. Container Docker disimpan di CLOUD sebagai image dan dipanggil untuk eksekusi user saat dibutuhkan dengan cara yang sederhana.

**Komponen utama Docker:** Docker image, Docker client, Docker host, Docker engine, Docker machine, Docker registry, Docker hub.

### Processor Issues dalam Virtualisasi
Dua strategi utama menyediakan resource prosesor di lingkungan virtual:
1. **Emulate a chip sebagai software** dan menyediakan akses ke resource itu. Contoh: **QEMU** dan Android emulator di Android SDK.
2. **Menyediakan segmen waktu pemrosesan** di prosesor fisik (pCPU) host virtualisasi ke virtual processor VM yang di-host di server fisik. Ini cara SEBAGIAN BESAR hypervisor virtualisasi menawarkan resource prosesor ke guest-nya.

### Memory Management dalam Virtualisasi
**Page sharing** — karena hypervisor mengelola page sharing, OS di VM TIDAK MENYADARI apa yang sebenarnya terjadi di sistem fisik.

**Ballooning** — hypervisor mengaktifkan **balloon driver** yang secara (virtual) "mengembang" dan MENEKAN guest OS untuk MENGALIRKAN (flush) halaman ke disk. Setelah halaman itu dikosongkan, balloon driver "mengempis" dan hypervisor bisa memakai memori fisik itu untuk VM LAIN.

**Memory overcommit** — kemampuan mengalokasikan LEBIH BANYAK memori daripada yang secara FISIK ada di host.

> [!info] Analogi
> Ballooning itu seperti mengempiskan BALON di dalam koper yang penuh untuk memaksa isi koper (memori guest OS) MENYUSUT tanpa harus membuka koper itu satu per satu. Hypervisor "meniup" balon virtual di dalam VM, memaksa VM itu MEMBUANG data yang kurang penting ke disk (bukan hypervisor yang langsung merampas memorinya paksa) — begitu VM sudah cukup "mengalah" dan melepas ruang, balon dikempiskan lagi, dan ruang yang terbebas itu dipakai hypervisor untuk VM lain yang lebih butuh.

### I/O dalam Lingkungan Virtual
Keuntungan virtualisasi jalur I/O beban kerja: memungkinkan INDEPENDENSI hardware dengan mengabstraksikan driver spesifik-vendor jadi versi yang LEBIH UMUM yang berjalan di hypervisor. Abstraksi ini memungkinkan:
- **Live migration** — salah satu KEKUATAN AVAILABILITY terbesar virtualisasi (memindahkan VM yang SEDANG BERJALAN ke host lain tanpa downtime).
- Berbagi resource AGREGAT, seperti jalur jaringan.

Kemampuan **memory overcommit** juga jadi manfaat lain dari virtualisasi I/O VM. Trade-off-nya: hypervisor mengelola SEMUA trafik dan butuh OVERHEAD prosesor. Ini pernah jadi masalah di masa AWAL virtualisasi, tapi sekarang prosesor multicore yang lebih cepat dan hypervisor yang lebih canggih sudah mengatasi kekhawatiran ini.

### Contoh Produk Virtualisasi
Slide menyebut beberapa produk hypervisor/virtualisasi populer: **VMware ESXi** (dengan fitur-fitur spesifiknya), **Xen**, **Hyper-V** — masing-masing dengan pendekatan arsitektur virtualisasi yang berbeda (detail teknis per-produk ada di gambar slide, bukan teks).

### Java Virtual Machine (JVM)
Tujuan **JVM** adalah menyediakan RUANG runtime untuk sekumpulan kode Java agar bisa berjalan di OS APA PUN yang di-stage di platform hardware APA PUN, TANPA perlu mengubah kode untuk mengakomodasi OS/hardware yang berbeda. JVM bisa mendukung BANYAK thread. Menjanjikan **"Write Once, Run Anywhere."**

JVM dideskripsikan sebagai MESIN KOMPUTASI ABSTRAK yang terdiri dari:
- **Instruction set.**
- **Program counter register.**
- **Stack** untuk menyimpan variabel dan hasil.
- **Heap** untuk data runtime dan garbage collection.
- **Method area** untuk kode dan konstanta.

> [!info] Konteks tambahan (bukan dari slide)
> JVM adalah contoh **virtualisasi di level APLIKASI**, berbeda dari hypervisor yang memvirtualisasikan HARDWARE/OS secara penuh. Program Java tidak dikompilasi langsung jadi kode mesin spesifik-CPU — dikompilasi jadi **bytecode** yang bisa dijalankan JVM APA PUN, di OS/hardware APA PUN, selama ada JVM yang sesuai terinstall. Ini yang memungkinkan slogan "Write Once, Run Anywhere".

### Linux VServer
**Linux VServer** adalah pendekatan open-source, cepat, dan RINGAN untuk mengimplementasikan virtual machine di server Linux. HANYA SATU salinan kernel Linux yang terlibat (berbeda dari virtualisasi penuh yang punya kernel terpisah per VM). VServer terdiri dari modifikasi RELATIF MODEST ke kernel plus sekumpulan kecil tool userland OS. Kernel VServer mendukung sejumlah virtual server TERPISAH. Kernel mengelola SEMUA resource dan tugas sistem, termasuk process scheduling, memori, ruang disk, dan waktu prosesor.

**Arsitektur isolasi (4 elemen):**
1. **chroot** — command UNIX/Linux untuk membuat root directory (/) jadi sesuatu yang BUKAN default-nya, sepanjang umur proses saat ini. Command ini menyediakan **file system isolation.**
2. **chcontext** — utility Linux yang mengalokasikan security context BARU dan mengeksekusi command dalam context itu. Setiap virtual server punya execution context-nya SENDIRI, menyediakan **process isolation.**
3. **chbind** — mengeksekusi command dan MENGUNCI proses yang dihasilkan (dan child-nya) untuk memakai alamat IP SPESIFIK. System call ini menyediakan **network isolation.**
4. **capabilities** — merujuk pada PARTISI privilese yang tersedia untuk root user. Setiap virtual server bisa diberi SUBSET TERBATAS dari privilese root user, menyediakan **root isolation.**

> [!info] Analogi
> Linux VServer itu seperti sebuah RUMAH SUSUN dengan SATU fondasi bangunan (kernel Linux) yang sama untuk SEMUA unit apartemen (virtual server) di dalamnya. Empat mekanisme isolasi itu seperti empat lapis privasi tiap unit: **chroot** = setiap unit punya "pintu depan" sendiri yang membuat penghuninya merasa itu satu-satunya rumah yang ada (isolasi file system). **chcontext** = setiap unit punya kunci dan alarm sendiri, tidak bisa mengintip aktivitas unit lain (isolasi proses). **chbind** = setiap unit punya alamat pos SENDIRI (isolasi network). **capabilities** = meski satu gedung punya "pemilik gedung" (root), tiap penghuni cuma diberi wewenang TERBATAS sesuai unit mereka, tidak bisa mengklaim wewenang penuh atas seluruh gedung (isolasi root).

## Diagram & Visual
- **Slide 5 — Konsep Virtual Machine**
  ![[99-Assets/OS/W13-slide05.png]]
- **Slide 11 — Type 1 dan Type 2 Hypervisor**
  ![[99-Assets/OS/W13-slide11.png]]
- **Slide 13 — Diagram Paravirtualization**
  ![[99-Assets/OS/W13-slide13.png]]
- **Slide 17 — Virtual Machine vs Container**
  ![[99-Assets/OS/W13-slide17.png]]
- **Slide 18 — Alur Operasi I/O**
  ![[99-Assets/OS/W13-slide18.png]]
- **Slide 19 — OpenVZ File Scheme**
  ![[99-Assets/OS/W13-slide19.png]]
- **Slide 26 — Page Sharing**
  ![[99-Assets/OS/W13-slide26.png]]
- **Slide 28 — I/O di Lingkungan Virtual**
  ![[99-Assets/OS/W13-slide28.png]]
- **Slide 31-35 — Diagram VMware ESXi, Xen, Hyper-V**
  ![[99-Assets/OS/W13-slide31.png]]
  ![[99-Assets/OS/W13-slide32.png]]
  ![[99-Assets/OS/W13-slide34.png]]
  ![[99-Assets/OS/W13-slide35.png]]
- **Slide 39 — Arsitektur Linux VServer**
  ![[99-Assets/OS/W13-slide39.png]]
- **Slide 40 — Linux VServer Token Bucket Scheme**
  ![[99-Assets/OS/W13-slide40.png]]

## Rumus / Sintaks
```
Fungsi Hypervisor:
- Execution management VM
- Device emulation & access control
- Eksekusi privileged operation untuk guest
- VM lifecycle management
- Administrasi platform hypervisor

Linux VServer - 4 Elemen Isolasi:
chroot       -> File system isolation
chcontext    -> Process isolation
chbind       -> Network isolation
capabilities -> Root isolation
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Bare-metal hypervisor** | Istilah lain Type 1 Hypervisor, berjalan langsung di atas hardware |
| **Live migration** | Memindahkan VM yang sedang berjalan ke host lain tanpa downtime |
| **Bytecode (Java)** | Kode hasil kompilasi Java yang bisa dijalankan JVM apa pun |
| **Loosely coupled** | Komponen sistem yang saling bergantung minimal, istilah kunci di definisi microservice |
| **Resource broker** | Peran hypervisor menjembatani dan mengalokasikan sumber daya ke VM |

## Pertanyaan Terbuka
- Slide 8-9 (Hypervisors), 21 (Processor Allocation), 25 (Ring 0), 30 (Performance Technology), dan 33 (VMware ESXi Features) hanya berupa judul TANPA detail teks yang bisa diekstrak — perlu dibuka manual kalau butuh detail konsep-konsep ini untuk ujian, terutama soal "Ring 0" yang relevan untuk memahami privilege level virtualisasi hardware.

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa Container Virtualization dianggap "lebih ringan" daripada Virtual Machine tradisional. Hubungkan dengan konsep isolasi Linux VServer (chroot, chcontext, chbind, capabilities) — bagaimana pendekatan "berbagi SATU kernel" ini secara fundamental berbeda dari VM yang masing-masing punya kernel sendiri?
2. **(C4 – Analisis)** Bandingkan Paravirtualization dan Hardware-Assisted Virtualization sebagai DUA pendekatan berbeda untuk mengoptimalkan performa virtualisasi. Analisis: yang satu membutuhkan MODIFIKASI di sisi guest OS (paravirtualization), yang lain membutuhkan dukungan di sisi PROSESOR (hardware-assisted) — apa trade-off dari masing-masing pendekatan?
3. **(C5 – Evaluasi)** Sebuah perusahaan mempertimbangkan memakai Security Virtual Appliance (SVA) untuk memonitor semua VM mereka. Evaluasi: kenapa SVA yang berjalan DI LUAR VM yang dimonitor (bukan sebagai agen DI DALAM setiap VM) dianggap lebih aman — kaitkan dengan keuntungan "tidak rentan terhadap cacat di Guest OS" yang disebutkan di materi.
4. **(C5 – Evaluasi)** Bandingkan strategi "Memory Overcommit" (mengalokasikan lebih banyak memori dari yang tersedia secara fisik) dengan risiko yang mungkin muncul kalau SEMUA VM di host itu TIBA-TIBA butuh memori penuh secara bersamaan. Evaluasi: kenapa teknik "Ballooning" penting sebagai MEKANISME PENGAMAN untuk skenario ini, dan apa yang terjadi kalau ballooning GAGAL memenuhi kebutuhan memori tepat waktu?
5. **(C6 – Cipta)** Rancang skenario keputusan (fiktif, untuk latihan) di mana sebuah startup teknologi harus memilih antara memakai VIRTUAL MACHINE penuh (misalnya lewat VMware ESXi) atau CONTAINER (misalnya Docker) untuk men-deploy aplikasi web mereka yang terdiri dari banyak microservice kecil. Usulkan pilihan yang PALING MASUK AKAL dan jelaskan alasannya berdasarkan karakteristik container (ringan, cepat, berbagi kernel) vs VM (isolasi penuh, overhead lebih besar) yang sudah dipelajari di materi ini.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W12 - Security]]
- [[OS - Review dan Glosari]]
