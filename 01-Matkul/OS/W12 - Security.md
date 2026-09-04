---
matkul: Operating Systems
minggu: 12
sks: 3
sumber: Week12-Pert12-Security.pptx
tags: [kuliah/os, minggu/w12]
status: draft
diproses: 2026-09-04
---

# W12 — Security

## Ringkasan
> - Sistem AMAN kalau resource dipakai dan diakses SESUAI MAKSUDNYA dalam SEMUA kondisi — tapi ini SECARA TEORI TIDAK BISA DICAPAI sepenuhnya. **Threat** = potensi pelanggaran keamanan; **attack** = usaha nyata melanggarnya (bisa disengaja atau tidak sengaja).
> - Lima kategori pelanggaran keamanan: **breach of confidentiality, breach of integrity, breach of availability, theft of service, denial of service.**
> - Keamanan harus terjadi di **4 level**: Physical, Application, Operating System, Network — dan keamanan SEKUAT mata rantai TERLEMAHnya (termasuk manusia, lewat phishing/social engineering).
> - **Buffer overflow** adalah salah satu celah paling klasik — attacker memasukkan lebih banyak data dari kapasitas buffer, MENIMPA memori di sekitarnya, berpotensi mengambil alih kontrol eksekusi program. Pertahanan ada di level **compile-time** (safe library, stack protection, bahasa pemrograman aman) dan **runtime** (guard pages, no-execute bit, address space randomization).
> - **Access control** diatur lewat kebijakan: **DAC** (berdasar identitas), **MAC** (berdasar label keamanan), **RBAC** (berdasar peran), **ABAC** (berdasar atribut) — dan proses **OS hardening** mencakup instalasi minimal, konfigurasi user/grup, kontrol resource, tools keamanan tambahan, dan testing berkala.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Threat | Potensi pelanggaran keamanan |
| Attack | Usaha nyata melanggar keamanan, bisa accidental atau malicious |
| Buffer overflow | Data yang dimasukkan melebihi kapasitas buffer, menimpa memori sekitarnya |
| DAC / MAC / RBAC / ABAC | Empat model kebijakan access control berbeda |
| Firewall | Pembatas akses jaringan antara host terpercaya dan tidak terpercaya |
| OS Hardening | Proses memperkuat keamanan OS lewat instalasi minimal dan konfigurasi ketat |

## Isi

### Masalah Keamanan (The Security Problem)
Sistem dikatakan AMAN kalau resource dipakai dan diakses SESUAI MAKSUDNYA dalam SEMUA kondisi — tapi ini **TIDAK BISA DICAPAI** secara mutlak. **Intruders (crackers)** mencoba menembus keamanan. **Threat** adalah potensi pelanggaran keamanan; **attack** adalah USAHA NYATA menembus keamanan. Attack bisa **accidental** (tidak disengaja) atau **malicious** (disengaja) — LEBIH MUDAH melindungi diri dari penyalahgunaan yang tidak disengaja dibanding yang disengaja.

### Kategori Pelanggaran Keamanan
1. **Breach of confidentiality** — pembacaan data TANPA otorisasi.
2. **Breach of integrity** — modifikasi data TANPA otorisasi.
3. **Breach of availability** — penghancuran data TANPA otorisasi.
4. **Theft of service** — pemakaian resource TANPA otorisasi.
5. **Denial of Service (DoS)** — mencegah PENGGUNAAN LEGITIMATE (menghalangi user sah memakai layanan).

### Metode Pelanggaran Keamanan
- **Masquerading** — melanggar autentikasi, PURA-PURA jadi user sah untuk eskalasi privilese.
- **Replay attack** — mengirim ulang pesan (apa adanya atau dimodifikasi).
- **Man-in-the-middle attack** — penyusup duduk di TENGAH aliran data, menyamar jadi pengirim ke penerima dan sebaliknya.
- **Session hijacking** — mengintersepsi sesi yang SUDAH TERBENTUK untuk melewati autentikasi.
- **Privilege escalation** — jenis serangan UMUM, mendapat akses MELEBIHI yang seharusnya dimiliki user/resource.

### Level-Level Keamanan
Tidak mungkin punya keamanan ABSOLUT, tapi bisa membuat BIAYA bagi pelaku cukup TINGGI untuk mencegah sebagian besar penyusup. Keamanan harus terjadi di **EMPAT level** agar efektif:
1. **Physical** — data center, server, terminal yang terhubung.
2. **Application** — aplikasi jinak atau jahat bisa menyebabkan masalah keamanan.
3. **Operating System** — mekanisme proteksi, debugging.
4. **Network** — komunikasi yang diintersepsi, interupsi, DoS.

**Keamanan SEKUAT mata rantai yang PALING LEMAH dalam rantainya.** MANUSIA juga adalah risiko lewat phishing dan serangan social-engineering. Slide juga mengangkat pertanyaan reflektif: **apakah keamanan TERLALU BANYAK juga bisa jadi masalah?** (misalnya kalau prosedur keamanan terlalu ribet, user malah mencari jalan pintas yang lebih tidak aman).

### Ancaman Akses Sistem: Intruder
- **Masquerader** — individu yang TIDAK terotorisasi memakai komputer dan menembus kontrol akses sistem untuk mengeksploitasi akun user LEGITIMATE.
- **Misfeasor** — user LEGITIMATE yang mengakses data/program/resource TANPA otorisasi untuk itu, atau yang punya otorisasi tapi MENYALAHGUNAKAN privilesenya.
- **Clandestine user** — individu yang MEREBUT kontrol supervisor sistem dan memakainya untuk MENGHINDARI auditing/access control atau MENEKAN pengumpulan audit.

### Malicious Software
Dua karakteristik: **Parasitic** — BUTUH host program, berupa fragmen program yang TIDAK BISA berdiri sendiri tanpa aplikasi/utility/program sistem nyata (contoh: virus, logic bomb, backdoor). **Non-parasitic** — TIDAK BUTUH host program, program mandiri yang bisa dijadwalkan dan dijalankan sendiri oleh OS (contoh: worm, bot program).

### Program Threats
- **Malware** — software dirancang untuk mengeksploitasi, mematikan, atau merusak komputer.
- **Trojan Horse** — program yang bertindak secara TERSEMBUNYI (clandestine), mengeksploitasi mekanisme yang mengizinkan program tulisan user dijalankan user lain. Bisa membawa spyware, pop-up browser, covert channel — sampai **80% spam** dikirim lewat sistem yang terinfeksi spyware.
- **Spyware** — sering diinstall BERSAMA software legitimate, untuk menampilkan iklan atau menangkap data user.
- **Ransomware** — MENGUNCI data lewat enkripsi, meminta pembayaran untuk membukanya.
- **Trap Door** — identifier/password khusus yang MELEWATI prosedur keamanan normal — bisa disisipkan bahkan di dalam COMPILER.
- **Backdoor/Trapdoor** — pintu masuk RAHASIA, berguna untuk debugging programmer, tapi memungkinkan programmer TIDAK BERETIKA mendapat akses tanpa otorisasi.
- **Logic Bomb** — "meledak" saat kondisi TERTENTU terpenuhi (misalnya keberadaan/ketidakberadaan file tertentu, hari tertentu dalam seminggu, user tertentu yang menjalankan aplikasi).

Semua jenis ancaman ini mencoba MELANGGAR **Principle of Least Privilege** (prinsip memberi hak akses SEMINIMAL yang dibutuhkan).

### Code Injection dan Buffer Overflow
**Code injection** terjadi ketika kode sistem BUKAN malicious tapi punya BUG yang mengizinkan kode executable DITAMBAHKAN atau DIMODIFIKASI. Hasil dari paradigma pemrograman yang BURUK atau TIDAK AMAN, umum di bahasa low-level seperti C/C++ yang mengizinkan akses memori LANGSUNG lewat pointer. Tujuannya BUFFER OVERFLOW, di mana kode ditaruh di sebuah buffer dan eksekusi disebabkan oleh serangan itu. Bisa dijalankan oleh **script kiddie** — memakai tool yang ditulis orang lain untuk mengeksploitasi kerentanan yang teridentifikasi.

**Buffer overflow attack (buffer overrun)** — didefinisikan NIST sebagai: *"Kondisi di sebuah interface di mana LEBIH BANYAK input bisa dimasukkan ke sebuah buffer/data-holding area DARIPADA kapasitas yang dialokasikan, MENIMPA informasi lain. Attacker mengeksploitasi kondisi ini untuk MELUMPUHKAN sistem atau menyisipkan kode khusus untuk mengambil kontrol sistem."*

**Untuk mengeksploitasi buffer overflow, attacker butuh:**
1. Mengidentifikasi kerentanan buffer overflow di sebuah program yang bisa DIPICU pakai data dari sumber eksternal yang DIKONTROL attacker.
2. Memahami bagaimana buffer itu disimpan di memori proses, dan potensi MENGKORUPSI lokasi memori BERDEKATAN, mengubah alur eksekusi program.

> [!info] Analogi
> Buffer overflow itu seperti menuangkan air ke GELAS TAKARAN yang lebih dari kapasitasnya. Kalau kamu tuangkan lebih dari batas gelas, air TUMPAH ke MEJA di sekitarnya (memori yang berdekatan). Kalau meja itu kebetulan berisi "instruksi penting" (alamat kembali fungsi, misalnya), attacker bisa dengan SENGAJA "menumpahkan" data yang dirancang khusus supaya "tumpahan" itu MENGUBAH instruksi yang seharusnya dijalankan program — mengarahkan eksekusi ke kode jahat yang mereka sisipkan.

### Countermeasures untuk Buffer Overflow
Diklasifikasikan jadi dua kategori:

**1. Compile-Time Defenses** (memperkuat program agar TAHAN serangan):
- **Language extensions dan safe libraries** — misalnya **Libsafe**, mengimplementasikan semantik standar TAPI menambahkan cek tambahan agar operasi copy TIDAK melampaui ruang variabel lokal di stack frame.
- **Stack protection mechanisms** — instrumen kode masuk/keluar fungsi untuk menyiapkan lalu MENGECEK stack frame-nya untuk bukti korupsi. **StackGuard** adalah salah satu yang paling terkenal, ekstensi compiler GCC yang menyisipkan kode tambahan masuk/keluar fungsi.
- **Pilihan bahasa pemrograman** — memakai bahasa level-tinggi modern dengan notasi TIPE yang KUAT dan definisi jelas operasi yang diizinkan — TAPI ini punya biaya di penggunaan sumber daya (compile-time DAN runtime).
- **Safe coding techniques** — programmer meninjau kode dan menulis ulang konstruksi yang TIDAK AMAN. Contoh: proyek **OpenBSD** yang melakukan AUDIT menyeluruh terhadap codebase (OS, standard library, common utilities).

**2. Runtime Defenses** (mendeteksi dan menghentikan serangan saat program BERJALAN):
- **Guard pages** — celah (gap) ditempatkan di antara rentang alamat komponen address space, ditandai ILEGAL di MMU — usaha mengakses guard page menyebabkan proses DIABORT. Bisa juga ditempatkan di antara stack frame atau alokasi heap yang berbeda.
- **Executable address space protection** — memblokir EKSEKUSI kode di STACK, karena secara asumsi kode executable HARUSNYA hanya ada di tempat lain di address space proses. Ekstensi tersedia untuk Linux, BSD, sistem UNIX-style lain lewat **no-execute bit**.
- **Address space randomization** — memanipulasi LOKASI struktur data kunci di address space proses saat runtime. Memindah region memori stack sejauh sekitar 1 megabyte punya dampak MINIMAL ke sebagian besar program tapi membuat MENEBAK alamat buffer yang ditarget HAMPIR MUSTAHIL. Teknik lain: mengacak URUTAN pemuatan library standar dan lokasi alamat virtualnya.

### Ancaman Sistem dan Jaringan
Beberapa sistem "TERBUKA" (open) daripada aman secara default — mengurangi attack surface membuat sistem LEBIH SULIT dipakai, butuh lebih banyak pengetahuan untuk administrasi. Ancaman jaringan LEBIH SULIT dideteksi dan dicegah — sistem proteksi LEBIH LEMAH, lebih SULIT punya shared secret sebagai dasar akses, TIDAK ADA batasan fisik begitu sistem terhubung ke internet, bahkan menentukan LOKASI sistem yang terhubung SULIT — alamat IP adalah SATU-SATUNYA pengetahuan yang biasanya tersedia.

**Worm** — memakai mekanisme SPAWN, program MANDIRI. Contoh historis: **Internet Worm** — mengeksploitasi fitur networking UNIX (remote access) dan BUG di program `finger` dan `sendmail`. Mengeksploitasi mekanisme trust-relationship yang dipakai `rsh` untuk mengakses sistem "teman" TANPA password. Program **grappling hook** meng-upload program worm UTAMA (hanya 99 baris kode C). Sistem yang "terkait" itu kemudian meng-upload kode utama, mencoba menyerang sistem yang terhubung. Juga mencoba MEMBOBOL akun user lain di sistem lokal lewat PASSWORD GUESSING. Kalau sistem target SUDAH terinfeksi, ABORT — KECUALI setiap kelipatan ke-7 (mekanisme yang aneh dan cerdik untuk tetap "coba lagi" sesekali).

**Port Scanning** — usaha OTOMATIS menyambung ke RENTANG port di satu atau rentang alamat IP. Untuk mendeteksi protokol layanan yang MERESPON, dan mendeteksi OS serta versi yang berjalan di sistem. **nmap** memindai SEMUA port di rentang IP tertentu untuk respons. **nessus** punya database protokol dan bug (dan eksploit) untuk diterapkan ke sistem. Sering diluncurkan dari SISTEM ZOMBIE untuk mengurangi keterlacakan (trace-ability).

**Denial of Service (DoS)** — MEMBEBANI komputer target, mencegahnya melakukan pekerjaan berguna. **DDoS (Distributed DoS)** datang dari BANYAK situs sekaligus. Contoh mekanisme: membanjiri awal SYN handshake koneksi IP — berapa banyak koneksi yang baru dimulai bisa ditangani OS? Untuk trafik ke sebuah website: bagaimana membedakan menjadi TARGET dan MEMANG SEDANG POPULER? Bisa bersifat **accidental** (misalnya mahasiswa CS menulis kode `fork()` yang buruk, tanpa sengaja menghasilkan fork bomb) atau **purposeful** (pemerasan, hukuman).

> [!info] Konteks tambahan (bukan dari slide)
> "Kenapa Windows jadi target sebagian besar serangan?" — slide mengajukan pertanyaan ini dengan beberapa opsi jawaban yang disinggung: paling UMUM dipakai (target-nya lebih menguntungkan secara jumlah), **"Everyone is an administrator"** (banyak user Windows lama berjalan dengan hak admin penuh, memperbesar dampak serangan), dan **monoculture considered harmful** — kalau SEMUA sistem seragam (satu jenis OS mendominasi), satu eksploit bisa menyerang JUTAAN sistem sekaligus, berbeda dengan ekosistem yang lebih beragam.

### Autentikasi (Authentication)
Dalam sebagian besar konteks keamanan komputer, **user authentication** adalah building block FUNDAMENTAL dan lini pertahanan UTAMA. RFC 4949 mendefinisikan user authentication sebagai proses MEMVERIFIKASI identitas yang diklaim oleh atau untuk sebuah entitas sistem.

**Dua langkah proses autentikasi:**
1. **Identification step** — mempresentasikan sebuah IDENTIFIER ke sistem keamanan.
2. **Verification step** — mempresentasikan atau menghasilkan informasi autentikasi yang MENGONFIRMASI ikatan antara entitas dan identifier itu.

**Empat cara autentikasi (means of authentication):**
1. **Something the individual IS** (biometrik statis) — sidik jari, retina, wajah.
2. **Something the individual DOES** (biometrik dinamis) — pola suara, karakteristik tulisan tangan, ritme mengetik.
3. **Something the individual KNOWS** — password, PIN, jawaban pertanyaan yang sudah disepakati.
4. **Something the individual POSSESSES** (disebut TOKEN) — kartu elektronik, smart card, kunci fisik.

### Pertahanan Keamanan
**Defense in depth** adalah teori keamanan PALING UMUM — beberapa LAPISAN keamanan sekaligus. **Security policy** mendeskripsikan apa yang sedang DIAMANKAN. **Vulnerability assessment** membandingkan state SESUNGGUHNYA sistem/jaringan dengan security policy. **Intrusion detection** berusaha mendeteksi percobaan atau keberhasilan intrusi:
- **Signature-based detection** — mendeteksi pola BURUK yang sudah dikenal.
- **Anomaly detection** — mendeteksi PERBEDAAN dari perilaku normal, bisa mendeteksi serangan ZERO-DAY, tapi rawan **false-positive** dan **false-negative**.

**Virus protection** — mencari pola virus yang dikenal di semua program atau saat eksekusi, atau menjalankan di SANDBOX supaya tidak bisa merusak sistem. **Auditing, accounting, dan logging** semua/sebagian aktivitas sistem atau jaringan. **Praktik komputasi aman** — hindari sumber infeksi, download hanya dari situs "baik", dll.

### Firewall
**Network firewall** ditempatkan di antara host TERPERCAYA dan TIDAK TERPERCAYA. Membatasi akses jaringan antara kedua domain keamanan itu. Bisa DI-TUNNEL atau DI-SPOOF — **tunneling** mengizinkan protokol yang DILARANG "melakukan perjalanan" DI DALAM protokol yang DIIZINKAN (contoh: telnet di dalam HTTP). Aturan firewall biasanya berbasis HOSTNAME atau alamat IP, yang bisa DI-SPOOF.

- **Personal firewall** — lapisan software di SEBUAH host, bisa memonitor/membatasi trafik ke dan dari host itu.
- **Application proxy firewall** — memahami protokol APLIKASI dan bisa mengontrolnya (contoh: SMTP).
- **System-call firewall** — memonitor SEMUA system call penting dan menerapkan aturan padanya (contoh: "program ini boleh mengeksekusi system call itu").

### Access Control
Mengimplementasikan security policy yang menspesifikasikan SIAPA atau APA boleh mengakses resource sistem SPESIFIK, dan jenis akses yang DIIZINKAN di tiap kasus. Memediasi antara user dan resource sistem (aplikasi, OS, firewall, router, file, database). **Security administrator** menjaga sebuah AUTHORIZATION DATABASE yang menspesifikasikan jenis akses ke resource mana yang diizinkan untuk user tertentu. Fungsi access control MENGONSULTASI database ini untuk menentukan APAKAH memberikan akses. **Fungsi auditing** memonitor dan mencatat akses user ke resource sistem.

**Hak akses (Access Rights):**
| Hak Akses | Deskripsi |
| --- | --- |
| **None** | User TIDAK boleh membaca directory yang berisi file itu |
| **Knowledge** | User bisa tahu file itu ADA dan siapa pemiliknya, bisa meminta akses tambahan ke pemilik |
| **Execution** | User bisa memuat dan mengeksekusi program tapi TIDAK bisa menyalinnya |
| **Reading** | User bisa membaca file untuk tujuan apa pun, termasuk menyalin dan eksekusi |
| **Appending** | User bisa MENAMBAHKAN data tapi TIDAK bisa memodifikasi/menghapus konten yang ada |
| **Updating** | User bisa MEMODIFIKASI, MENGHAPUS, dan MENAMBAHKAN data file |
| **Changing protection** | User bisa MENGUBAH hak akses yang diberikan ke user lain |
| **Deletion** | User bisa MENGHAPUS file dari file system |

**Empat kategori kebijakan access control:**
1. **Discretionary Access Control (DAC)** — mengontrol akses berdasarkan IDENTITAS requestor dan aturan akses yang menyatakan apa yang boleh dilakukan requestor.
2. **Mandatory Access Control (MAC)** — mengontrol akses dengan MEMBANDINGKAN label keamanan dengan clearance keamanan.
3. **Role-Based Access Control (RBAC)** — mengontrol akses berdasarkan PERAN (role) yang dimiliki user dalam sistem, dan aturan yang menyatakan akses yang diizinkan untuk peran tertentu.
4. **Attribute-Based Access Control (ABAC)** — mengontrol akses berdasarkan ATRIBUT user, resource yang diakses, dan kondisi lingkungan saat itu.

> [!info] Analogi
> Bayangkan empat model access control ini dengan analogi keamanan gedung kantor. **DAC** itu seperti pemilik ruangan yang BEBAS memutuskan sendiri siapa boleh masuk ruangannya — fleksibel tapi rawan inkonsistensi kebijakan antar ruangan. **MAC** itu seperti sistem clearance militer — kamu HANYA boleh masuk ruangan yang level keamanannya SAMA ATAU DI BAWAH clearance-mu, TIDAK PEDULI siapa pemilik ruangannya, aturan ditetapkan PUSAT. **RBAC** itu seperti kartu akses berdasarkan JABATAN — "Manager" otomatis bisa masuk ruang rapat manajer, terlepas siapa orangnya, karena akses melekat ke PERAN bukan individu. **ABAC** itu paling FLEKSIBEL — aturan bisa berbunyi "boleh masuk ruang server HANYA kalau jam kerja DAN dari departemen IT DAN pakai laptop perusahaan" — menggabungkan BANYAK atribut sekaligus.

### UNIX Access Control dan System Call
UNIX memakai model access control berbasis owner/group/other dengan permission read/write/execute (rwx).

```c
int chmod(const char* pathname, mode_t mode);   // mengatur permission file
int chown(const char *path, uid_t owner, gid_t group);  // mengatur owner/group
```

**Contoh kode chmod():**
```cpp
#include <iostream>
#include <sys/stat.h>
#include <sys/types.h>
using namespace std;

int main(int argc, char* argv[]) {
    if (argc != 2) {
        cout << "Incorrect usage" << endl;
        return 1;
    }
    int retval = chmod(argv[1], 0644);   // rw- r-- r--
    if (retval < 0) {
        cout << "Could not chmod the file" << endl;
        return 1;
    }
    return 0;
}
```

### Operating System Hardening
**Langkah dasar mengamankan sebuah OS:**
1. **Install dan patch OS.**
2. **Harden dan konfigurasi OS** untuk memenuhi kebutuhan keamanan yang teridentifikasi, dengan cara:
   - Menghapus service, aplikasi, protokol yang TIDAK PERLU.
   - Mengonfigurasi user, group, dan permission.
   - Mengonfigurasi resource control.
3. **Install dan konfigurasi kontrol keamanan tambahan** — antivirus, host-based firewall, IDS, bila diperlukan.
4. **Test keamanan** OS dasar untuk memastikan langkah-langkah di atas MEMADAI menangani kebutuhan keamanannya.

**Detail per langkah:**
- **Remove unnecessary services** — jangan pakai default instalasi, kustomisasi hanya paket yang DIPERLUKAN. Preferensi KUAT: JANGAN install software yang tidak diinginkan sama sekali, DARIPADA install lalu hapus/nonaktifkan nanti — banyak script uninstall GAGAL menghapus semua komponen paket sepenuhnya, dan software yang dinonaktifkan bisa DIAKTIFKAN LAGI attacker kalau berhasil masuk sebagian.
- **Configure users, groups, authentication** — batasi privilese elevated hanya ke user yang BUTUH. Amankan akun default yang termasuk saat instalasi — akun yang TIDAK diperlukan dihapus atau dinonaktifkan. Akun sistem yang mengelola service seharusnya diset TIDAK BISA dipakai untuk login interaktif. Ganti password default dengan nilai baru yang aman.
- **Configure resource controls** — setelah user dan grup didefinisikan, permission yang sesuai bisa diset di data dan resource sesuai policy — misalnya membatasi user mana yang bisa eksekusi program tertentu atau baca/tulis di direktori tertentu.
- **Install additional security controls** — antivirus, host-based firewall, IDS/IPS, application white-listing. Karena prevalensi malware yang luas, antivirus yang tepat adalah komponen keamanan KRITIS.
- **Test the system security** — langkah TERAKHIR: memastikan konfigurasi keamanan sebelumnya sudah diimplementasikan BENAR, mengidentifikasi kerentanan yang harus dikoreksi. Dilakukan setelah hardening awal DAN diulang SECARA PERIODIK sebagai bagian pemeliharaan keamanan.

### Security Maintenance: Logging dan Backup
**Logging** bisa menghasilkan volume informasi SIGNIFIKAN — penting mengalokasikan ruang yang cukup. Sebaiknya ada sistem ROTASI dan ARSIP log otomatis. Analisis OTOMATIS lebih disukai karena lebih mungkin mengidentifikasi aktivitas ABNORMAL — analisis MANUAL log itu melelahkan dan TIDAK ANDAL untuk mendeteksi kejadian buruk. Logging yang efektif membantu administrator sistem mengidentifikasi kejadian LEBIH CEPAT dan AKURAT setelah breach/kegagalan.

**Backup vs Archive:**
- **Backup** — proses membuat salinan data secara REGULER, memungkinkan pemulihan data yang hilang/korup dalam periode RELATIF SINGKAT (beberapa jam sampai beberapa minggu).
- **Archive** — proses menyimpan salinan data untuk periode LEBIH LAMA (bulan atau tahun), untuk memenuhi kebutuhan legal dan operasional mengakses data lampau.

### Windows Access Control Scheme
Saat user LOGIN ke sistem Windows, skema NAMA/PASSWORD dipakai untuk mengautentikasi user. Kalau logon DITERIMA, sebuah proses dibuat untuk user, dan **access token** diasosiasikan dengan objek proses itu. Access token mencakup **Security ID (SID)** — identifier yang dipakai sistem untuk mengenali user ini demi tujuan keamanan. Token juga berisi SID untuk security GROUP yang menjadi anggota user itu.

## Diagram & Visual
- **Slide 13 — Four Layered Model of Security**
  ![[99-Assets/OS/W12-slide13.jpg]]
- **Slide 15 — Diagram Trampoline Code Execution (Buffer Overflow)**
  ![[99-Assets/OS/W12-slide15.png]]
  ![[99-Assets/OS/W12-slide15a.png]]
- **Slide 21 — Standard Security Attacks**
  ![[99-Assets/OS/W12-slide21.jpg]]
- **Slide 26 — Diagram Firewall**
  ![[99-Assets/OS/W12-slide26.jpg]]
- **Slide 29 — Diagram Buffer Overflow Attack**
  ![[99-Assets/OS/W12-slide29.png]]
- **Slide 38-39 — Access Control Structure, Roles, dan Resources**
  ![[99-Assets/OS/W12-slide38.png]]
  ![[99-Assets/OS/W12-slide39.png]]
- **Slide 40 — UNIX Access Control**
  ![[99-Assets/OS/W12-slide40.png]]
- **Slide 54 — Windows Security Structure**
  ![[99-Assets/OS/W12-slide54.png]]

## Rumus / Sintaks
```c
// UNIX access control system calls
int chmod(const char* pathname, mode_t mode);
int chown(const char *path, uid_t owner, gid_t group);

// Contoh: set permission rw-r--r-- (0644)
chmod(argv[1], 0644);
```
```
4 Level Keamanan: Physical -> Application -> Operating System -> Network
4 Kategori Access Control: DAC | MAC | RBAC | ABAC
2 Kategori Countermeasure Buffer Overflow: Compile-time | Runtime
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Zero-day attack** | Serangan yang mengeksploitasi kerentanan yang belum diketahui/di-patch vendor |
| **Sandbox** | Lingkungan terisolasi untuk menjalankan program mencurigakan tanpa risiko ke sistem asli |
| **Script kiddie** | Penyerang yang memakai tool eksploit buatan orang lain tanpa memahami detail teknisnya |
| **Zombie system** | Sistem yang sudah dikompromikan dan dipakai attacker untuk menyerang sistem lain |
| **Security ID (SID)** | Identifier unik yang dipakai Windows untuk mengenali user demi keperluan keamanan |
| **Attack surface** | Total titik/celah yang bisa dieksploitasi attacker di sebuah sistem |

## Pertanyaan Terbuka
- Slide tidak memberikan jawaban DEFINITIF untuk pertanyaan reflektif "kenapa Windows jadi target paling umum?" — hanya menyebut beberapa faktor kandidat (popularitas, everyone-is-admin, monoculture) tanpa kesimpulan tunggal.
- Slide 50 (Security Maintenance steps) hanya berupa judul tanpa daftar detail lengkap yang terekstrak — perlu dibuka manual untuk detail langkahnya.

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa slide menyatakan "keamanan sekuat mata rantai paling lemah dalam rantainya" dengan menyebutkan MANUSIA sebagai risiko (phishing, social engineering). Bagaimana empat level keamanan (Physical, Application, OS, Network) BISA sempurna, tapi sistem TETAP bisa dibobol lewat faktor manusia?
2. **(C4 – Analisis)** Bandingkan Compile-Time Defenses dan Runtime Defenses untuk buffer overflow. Analisis: kenapa kombinasi KEDUANYA (bukan salah satu saja) dianggap pendekatan yang lebih kuat — apa yang bisa DILEWATKAN compile-time defense tapi TERTANGKAP runtime defense, atau sebaliknya?
3. **(C5 – Evaluasi)** Sebuah organisasi memilih MAC (Mandatory Access Control) untuk sistem yang menangani data sangat sensitif (misalnya data militer/pemerintahan). Evaluasi: kenapa MAC lebih cocok dibanding DAC untuk kasus ini, dan risiko APA yang muncul kalau organisasi ini salah memilih DAC (di mana PEMILIK data yang menentukan akses, bukan otoritas pusat)?
4. **(C5 – Evaluasi)** Bandingkan strategi "jangan install software yang tidak perlu SAMA SEKALI" dengan "install lalu nonaktifkan/hapus nanti" dalam konteks OS Hardening. Evaluasi: mengapa slide secara TEGAS menyatakan preferensi kuat untuk opsi PERTAMA — kaitkan dengan risiko software yang "dinonaktifkan" bisa DIAKTIFKAN KEMBALI attacker.
5. **(C6 – Cipta)** Rancang skenario Address Space Randomization (fiktif, untuk latihan konseptual) yang menjelaskan bagaimana teknik ini menggagalkan usaha attacker mengeksploitasi buffer overflow yang SUDAH teridentifikasi di sebuah program. Jelaskan kenapa attacker yang tadinya SUDAH tahu persis alamat buffer target (dari analisis sebelumnya) menjadi TIDAK BISA lagi mengandalkan pengetahuan itu setelah address space randomization diaktifkan.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_OS]]
- [[W11 - Virtual Memory]]
- [[W13 - Virtualization]]
- [[OS - Review dan Glosari]]
