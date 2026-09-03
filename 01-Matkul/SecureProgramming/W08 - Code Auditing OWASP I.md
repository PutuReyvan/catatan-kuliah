---
matkul: Secure Programming
minggu: 8
sks: 3
sumber: S8_OWASP_Code_Auditing_I.pptx
tags: [kuliah/secure-programming, minggu/w08]
status: draft
diproses: 2026-09-04
---

# W08 — Code Auditing based on OWASP Top 10 (I): A01 & A05

## Ringkasan
> - Mulai seri **OWASP Top 10**: sesi ini fokus ke **A01 Broken Access Control** (peringkat #1) dan **A05 Security Misconfiguration** (peringkat #5).
> - **Code auditing** = review terstruktur dan manual atas source code dan konfigurasi buat nemuin kelemahan sebelum penyerang nemuin duluan. Melengkapi scanner otomatis — **manusia bisa nangkep kesalahan logika dan otorisasi yang alat nggak bisa**.
> - Cerita: **Maya** bangun portal **StudyHub**. Mahasiswa buka URL `studyhub.edu/grades?student_id=1042`, iseng ganti jadi `1043`... dan ngeliat nilai orang lain. **Nggak ada login yang di-bypass** — aplikasinya emang nggak pernah ngecek record itu punya siapa.
> - **Authentication** = kamu siapa (login). **Authorization** = kamu boleh ngapain (access control). Tiga prinsip: **server-side only, deny by default, least privilege.**
> - Enam wajah broken access control: **IDOR, forced browsing, privilege elevation, metadata tampering, missing API controls, CORS misconfiguration.**
> - Security misconfiguration itu **bukan bug di kode — celahnya ada di pengaturan, default, dan apa yang lupa dimatiin.**
> - Statistik: A01 muncul di **94%** aplikasi yang dites; A05 di **90%**.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Code auditing | Review manual dan terstruktur atas source code buat nemuin kelemahan keamanan |
| OWASP Top 10 | Daftar peringkat risiko keamanan web app paling kritis, berbasis data komunitas |
| Authentication | "Kamu siapa?" — proses login/verifikasi identitas |
| Authorization | "Kamu boleh ngapain?" — access control, penegakan kebijakan |
| IDOR | Insecure Direct Object Reference; ngubah ID di URL buat ngakses record orang lain |
| Forced browsing | Ngetik URL admin langsung sebagai user biasa/nggak login |
| Deny by default | Prinsip: tutup semua akses dulu, baru buka yang emang diizinin secara eksplisit |
| Security misconfiguration | Kerentanan yang bersumber dari pengaturan/default yang nggak dikencengin, bukan dari bug kode |

## Isi

### Konsep: apa itu code auditing?
**Code auditing** adalah review terstruktur dan manual atas source code dan konfigurasi buat nemuin kelemahan **sebelum penyerang nemuinnya duluan**. Ini **melengkapi scanner otomatis** — manusia bisa nangkep kesalahan logika dan otorisasi yang alat nggak bisa. Didorong oleh checklist pola risiko yang udah dikenal — di sinilah **OWASP Top 10** masuk. Pro PHP Security nyebut ini sebagai **berpikir kayak penyerang tentang kodemu sendiri**.

**Jembatan dari kelas database-mu:** review query buat nyari SQL injection itu udah bagian dari code audit. `GRANT`/`REVOKE` dan izin level-baris = access control, cuma di dalam DBMS. Database yang dibiarin `root`/tanpa password = misconfiguration. Hari ini kita naikin insting yang sama ke level aplikasi web.

### OWASP Top 10 (2021): peta audit kita
Sebuah peringkat risiko keamanan web app paling kritis yang dibangun komunitas, berbasis bukti data. Ini checklist yang mengarahkan audit kita. Hari ini fokus ke dua dari sepuluh: **A01 Broken Access Control** dan **A05 Security Misconfiguration**.

### Cerita: Maya & portal StudyHub
Maya, developer junior di kampus, baru rilis **StudyHub**, portal PHP tempat mahasiswa liat nilai, upload tugas, dan admin ngurus semua orang.
- **Act I** — seorang mahasiswa buka pintu yang seharusnya terkunci.
- **Act II** — pengaturan yang kelupaan ngasih penyerang peta gratis.
- **Act III** — Maya audit, benerin, dan ngencengin StudyHub.

### Konsep: apa itu access control?
**Authentication** — kamu siapa? (login). **Authorization** — kamu boleh ngapain? (access control). **Access control** menegakkan kebijakan biar user nggak bisa bertindak di luar izin yang dimaksudkan. Kegagalan berujung ke pengungkapan, modifikasi, atau penghancuran data yang nggak sah.

**Dua model penegakan:**
- **Server-side only** — pengecekan harus jalan di kode server terpercaya. Jangan pernah percaya browser, hidden field, atau parameter URL.
- **Deny by default** — mulai tertutup; buka akses cuma buat yang secara eksplisit diizinin — kayak database yang nggak ngasih hak apa pun sampai kamu `GRANT`.
- **Least privilege** — tiap role dapet minimum yang dia butuhin — nggak lebih.

### Kenali ancamannya: Act I — pintu yang nggak terkunci
Apa yang mahasiswa itu perhatiin: halaman nilai Maya kebuka dari URL kayak `studyhub.edu/grades?student_id=1042`. Mahasiswa yang penasaran ganti `1042` jadi `1043` … dan ngeliat nilai orang lain. **Nggak ada login yang di-bypass** — aplikasinya cuma nggak pernah ngecek record itu punya user yang login atau bukan.

**Tanda bahaya yang dicari auditor:** Object ID diambil langsung dari URL atau form, dipakai buat ngambil data. Nggak ada ownership check ("ini record punya gue?") sebelum data dibalikin. Halaman admin bisa dijangkau dengan langsung ngetik URL-nya (*force browsing*). Role disimpen di cookie atau hidden field yang bisa diedit user. Pengecekan yang sama di-copy-paste di mana-mana — gampang kelupaan di satu halaman.

### Analisis kode: dari rentan ke terverifikasi
**Rentan — percaya inputnya:**
```php
$id = $_GET['student_id'];
$row = $db->query("SELECT * FROM grades WHERE student_id = $id");
echo render($row);  // nilai siapa aja
```
Query-nya ngebalikin ID apa pun yang diminta — **ownership nggak pernah dicek**.

**Sudah diperbaiki — nerapin ownership di sisi server:**
```php
$id  = (int) $_GET['student_id'];
$me  = $_SESSION['student_id'];
if ($id !== $me && !isAdmin($me)) {
  http_response_code(403); exit;
}  // deny by default, baru lanjut
```
Bandingin sama session: pengecekan sisi server yang nggak bisa diedit user — kayak nerapin `GRANT` di dalam DBMS. **Aturan audit: tiap request yang baca atau nulis record harus verifikasi record itu punya si pemanggil — di kode server yang terpercaya.**

> [!info] Analogi
> Bayangin loket teller bank yang ngasih saldo rekening cuma berdasarkan **nomor yang KAMU sebutin**, tanpa pernah ngecek KTP-mu. Kamu bisa sebut nomor rekening siapa aja dan teller-nya bakal kasih tau saldonya. **Yang seharusnya kejadian**: teller cocokin nomor rekening yang kamu sebutin sama KTP yang kamu tunjukin — kalau nggak cocok dan kamu bukan manajer bank, ditolak.

### Enam wajah broken access control
1. **Insecure Direct Object Reference (IDOR)** — ngedit ID di URL buat nyampe record user lain (bug nilai Maya)
2. **Forced browsing** — ngetik URL admin langsung sebagai user biasa — atau bahkan yang belum login
3. **Privilege elevation** — bertindak sebagai admin sambil login sebagai user biasa, misalnya lewat ngubah role flag
4. **Metadata tampering** — muter ulang atau ngedit JWT, cookie, atau hidden field buat dapetin hak akses
5. **Missing API controls** — endpoint POST/PUT/DELETE yang nggak ada otorisasi di baliknya
6. **CORS misconfiguration** — ngizinin panggilan API dari origin yang nggak dipercaya buat baca data terproteksi

### Kasus: ini risiko #1 karena satu alasan
- **#1** peringkat di OWASP Top 10 (2021), naik dari #5
- **94%** aplikasi yang dites punya bentuk kelemahan access control
- **318 ribu+** kejadian di dataset kontribusi OWASP

**IDOR di dunia nyata:** breach account-takeover dan data-exposure berulang kali terlacak balik ke ID yang bisa diubah di URL dan API — persis pola bug nilai Maya, cuma dalam skala jutaan record. OWASP mapping ini ke **CWE-639** (user-controlled key) dan **CWE-200** (sensitive data exposure). **Forced browsing** dan cacat privilege — konsol admin yang bisa dijangkau tanpa hak admin, dan **CSRF (CWE-352)** yang nyalahgunain session yang lagi login — adalah kegagalan access control berulang di laporan insiden. **Pelajaran: dampaknya bisa kompromi satu akun penuh, atau seluruh database.**

### Konsep: apa itu security misconfiguration?
Pengaturan nggak aman, bukan kode nggak aman. Aplikasinya kehilangan hardening di suatu titik di seluruh stack — aplikasi, framework, web server, database, atau cloud. Kelemahannya ada di **konfigurasi, default, dan apa yang lupa dimatiin** — bukan di logika program. Terkait erat sama **A06** (komponen usang/rentan): software yang belum di-patch itu juga kegagalan konfigurasi.

**Di mana dia sembunyi (target audit):** akun dan password default yang belum diubah. Fitur, port, aplikasi contoh yang nggak diperlukan tapi masih aktif. Error verbose/stack trace yang ditampilin ke user. Directory listing nyala; file backup dan `.git` di web root. Header keamanan yang hilang; default framework yang nggak aman.

### Kenali ancamannya: Act II — pengaturan yang kelupaan
**Apa yang rusak di StudyHub:** crash nampilin stack trace lengkap — ngeksposkan path file dan nama database. Folder `/uploads` nge-list semua file karena directory listing nyala. Halaman `phpinfo()` yang ketinggalan ngeksposkan seluruh konfigurasi server. Tool admin masih pakai password default dari tutorial.

**`php.ini` — audit dimulai dari sini:**
```ini
; berbahaya di production
display_errors = On
expose_php     = On

; setting yang dikencengin
display_errors = Off
log_errors     = On
expose_php     = Off
```
**OWASP PHP Configuration Cheat Sheet** adalah checklist buat file ini.

### Analisis kebocorannya: gimana satu setting kecil jadi peta besar
1. **Recon** — penyerang triger error. Stack trace verbose ngeksposkan path, versi framework, dan detail DB. →
2. **Map** — directory listing dan halaman `phpinfo()` yang ketinggalan ngeksposkan struktur dan versi komponen persis. →
3. **Exploit** — CVE yang udah dikenal buat versi-versi itu + password admin default = kompromi penuh — **nggak butuh hacking canggih.**

**Kenapa ini berbahaya:** misconfiguration jarang nge-rusak apa pun sendirian — **dia ngilangin tebakan buat penyerang.** Tiap detail yang bocor mempersingkat jalur dari probing ke exploitasi. OWASP mapping ini ke **CWE-16** (Configuration) dan **CWE-756** (missing custom error page).

### Kasus: kesalahan konfigurasi dalam skala besar
- **#5** peringkat di OWASP Top 10 (2021), naik dari #6
- **90%** aplikasi yang dites punya bentuk misconfiguration
- **208 ribu+** kejadian di seluruh dataset OWASP

**Cloud storage terbuka:** bucket cloud yang bisa dibaca publik — misconfiguration sharing default — udah ngeksposkan volume besar data pribadi. **Nggak ada kode yang dieksploitasi; sebuah izin cuma dibiarin terbuka ke internet.** **Kredensial default dan aplikasi contoh:** konsol admin dengan password default yang nggak diubah dan aplikasi sample/demo yang ketinggalan itu skenario serangan OWASP yang terdokumentasi — penyerang login langsung dan ambil alih.

### Pencegahan: menghentikan broken access control
- **Deny by default** — blok semua kecuali resource yang eksplisit publik — mulai tertutup
- **Enforce server-side** — jalanin pengecekan di kode terpercaya; jangan pernah andelin browser, cookie, atau hidden field
- **Centralise the check** — implementasiin access control sekali, pakai ulang di mana-mana, jangan copy-paste
- **Enforce record ownership** — verifikasi baris itu punya si pemanggil — kayak izin level-baris di DB-mu
- **Protect tokens** — invalidate session pas logout; jaga JWT tetap short-lived dan tervalidasi
- **Log & test** — log kegagalan access control, rate limit, dan tulis unit test otorisasi

### Pencegahan: mengencengin konfigurasi
- **Repeatable hardening** — otomatiskan baseline yang terkunci; dev, QA, dan prod dikonfigurasi identik (kredensial beda)
- **Minimal platform** — hapus fitur, sample app, port, dan akun default yang nggak dipakai
- **Hide internals** — `display_errors` Off, `expose_php` Off, nggak ada directory listing, halaman error custom
- **Send security headers** — pasang header/directive (misal CSP, HSTS) buat ngarahin perilaku browser
- **Patch & review** — lacak update sebagai bagian patch management (nyambung ke A06); review izin cloud
- **Verify automatically** — jalanin pengecekan otomatis buat konfirmasi setting-nya bener di tiap environment

### Empat prinsip yang melindungi keduanya
1. **Default deny / default secure** — tertutup dan dikencengin sampai secara eksplisit dibuka — buat izin maupun setting
2. **Never trust the client** — otorisasi dan konfigurasi diputusin di kode server terpercaya doang
3. **Least privilege everywhere** — user, role, service, dan fitur masing-masing dapet minimum yang dibutuhin
4. **Automate & verify** — test, logging, dan proses yang berulang — asumsiin manusia bakal lupa

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — kontennya semua teks dan blok kode.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **JWT** | JSON Web Token, format token yang sering dipake buat autentikasi/otorisasi berbasis klaim |
| **CORS** | Cross-Origin Resource Sharing, mekanisme browser yang ngatur situs mana aja yang boleh manggil API-mu |
| **CSP (Content-Security-Policy)** | Header HTTP yang membatasi resource/script apa yang boleh dimuat browser |
| **HSTS** | Header HTTP yang maksa browser selalu pakai HTTPS buat situs tertentu |
| **`phpinfo()`** | Fungsi PHP yang nampilin seluruh detail konfigurasi server — berbahaya kalau kebuka publik |
| **`http_response_code()`** | Fungsi PHP buat ngeset status code HTTP response (misal 403 Forbidden) |
| **CWE** | Common Weakness Enumeration, katalog standar jenis-jenis kerentanan software |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan bug nilai Maya (`?student_id=1042` → `1043`) dengan kasus misconfiguration StudyHub (stack trace bocor). Analisis: kalau CUMA satu dari dua ini yang dibenerin duluan, mana yang lebih ngurangin risiko keseluruhan — dan gimana keduanya sebenernya SALING MEMPERKUAT kalau dibiarin bareng (petunjuk: recon → map → exploit)?
2. **(C4 – Analisis)** Dari enam wajah broken access control (IDOR, forced browsing, privilege elevation, metadata tampering, missing API controls, CORS misconfiguration), pilih DUA yang menurutmu paling gampang DITEMUKAN penyerang tanpa tool khusus, dan DUA yang paling gampang DILEWATKAN auditor pemula. Jelasin alasannya.
3. **(C5 – Evaluasi)** Sebuah tim bilang: "Kita udah aman dari access control issue karena kita nyembunyiin link ke halaman admin — nggak ada yang nge-link ke situ dari menu manapun." Evaluasi klaim "security through obscurity" ini pakai konsep forced browsing dan prinsip deny-by-default.
4. **(C5 – Evaluasi)** Bandingkan dampak IDOR (misalnya bug nilai Maya) dengan dampak security misconfiguration (misalnya `phpinfo()` kebuka publik) dari sisi SIAPA yang paling gampang mengeksploitasinya — apakah butuh skill teknis tinggi, atau bisa ditemukan otomatis lewat scanner? Apa implikasinya buat prioritas perbaikan?
5. **(C6 – Cipta)** StudyHub mau nambah fitur baru: dosen bisa liat rekap nilai SEMUA mahasiswa di kelas yang dia ajar (bukan cuma satu mahasiswa). Rancang pengecekan otorisasi (pseudocode, mirip pola di slide 9) buat endpoint ini — pastikan ngikutin ketiga prinsip (server-side only, deny by default, least privilege), dan jelasin kenapa endpoint ini butuh logika otorisasi yang BEDA dari cek `$id !== $me` yang dipake buat nilai individual.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W07 - Review]]
- [[W09 - Code Auditing OWASP II]]
- [[SecureProgramming - Review dan Glosari]]
