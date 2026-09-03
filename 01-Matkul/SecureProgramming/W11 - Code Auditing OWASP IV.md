---
matkul: Secure Programming
minggu: 11
sks: 3
sumber: S11- Code Auditing Based on OWASP Top10 IV.pptx
tags: [kuliah/secure-programming, minggu/w11]
status: draft
diproses: 2026-09-04
---

# W11 — Code Auditing based on OWASP Top 10 (IV): A07 & A08

## Ringkasan
> - **Rafa**, developer on-call di **BiteClub** (app food-delivery). Akun, login, reset password, dan **plugin "deals" yang auto-update** semuanya jalan di app ini.
> - **Act 1**: login yang ngebolehin semua orang masuk — penyerang lewat pintu depan pakai password curian dan alur reset yang lemah.
> - **Act 2**: update yang nggak pernah diverifikasi — plugin terpercaya ditukar jadi versi jahat. Kodenya "ditandatangani"... atau enggak?
> - **A07 & A08 dua-duanya soal PERCAYA:** A07 = percaya ke ORANG yang salah (password lemah, nggak ada MFA, session rusak). A08 = percaya ke KODE/DATA yang salah (update, dependency, data serial diterima tanpa verifikasi integritas).
> - **Tiga kata yang sering ketuker**: Identification (KAMU SIAPA — klaim username), Authentication (BUKTIKAN — password/OTP/key), Session management (TETEP TERBUKTI — session ID gantiin bukti tiap request).
> - **MFA** = kombinasi faktor dari kategori berbeda: **something you KNOW** (password), **something you HAVE** (authenticator app), **something you ARE** (biometrik). Satu password curian aja nggak cukup.
> - Kasus: **Colonial Pipeline (2021)** — satu akun VPN kompromi TANPA MFA = pipa bahan bakar terbesar AS berhenti berhari-hari. **SolarWinds (2020)** — update yang DITANDATANGANI tapi build pipeline-nya yang dikompromikan, nyebar ke 18.000+ organisasi.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Identification | Klaim identitas — nunjukin username/email/user-id |
| Authentication | Buktiin identitas — nyupply secret yang cuma dipunya user itu |
| Session management | Menjaga bukti identitas tetap "aktif" lewat session ID di tiap request setelah login |
| Credential stuffing | Nyoba pasangan email+password dari breach dump lain secara massal |
| MFA | Multi-Factor Authentication, kombinasi ≥2 faktor dari kategori berbeda |
| Software integrity | Verifikasi bahwa update/kode/paket beneran dari sumber terpercaya dan nggak diutak-atik |
| Insecure deserialization | Ngubah data serial yang bisa dilihat/diubah penyerang balik jadi objek hidup |
| CI/CD pipeline | Rangkaian otomatis buat build, test, dan deploy kode |
| HMAC | Kode verifikasi yang membuktikan data nggak diubah sejak ditandatangani |

## Isi

### Cerita: Rafa, developer on-call
Rafa maintain **BiteClub**, app web PHP buat startup food-delivery. Akun, login, reset password, dan plugin "deals" yang auto-update semuanya jalan di app ini.

**Act 1: Login yang ngebolehin semua orang masuk.** Penyerang lewat pintu depan pakai password curian dan alur reset yang lemah.

**Act 2: Update yang nggak pernah diverifikasi.** Update plugin yang terpercaya ditukar jadi yang jahat. Kodenya ditandatangani… atau enggak?

Hari ini kita audit kode Rafa di dua-duanya, terus dibenerin.

### Framing: di mana dua kategori ini duduk
OWASP Top 10 (2021) itu daftar konsensus risiko web app paling kritis. Hari ini kita audit dua darinya — **dua-duanya kegagalan soal kepercayaan.**

- **A07 — Identification & Authentication Failures.** Percaya ke ORANG yang salah. Password lemah, nggak ada MFA, session handling yang rusak ngebolehin penyerang nyamar jadi user.
- **A08 — Software & Data Integrity Failures.** Percaya ke KODE/DATA yang salah. Kategori baru 2021: update, dependency & data serial yang diterima tanpa verifikasi integritas.

**Jembatan database:** A07 itu akses di layer login; A08 itu constraint integritas dari supply chain-mu. Kamu udah percaya **CHECKSUM** pas restore backup — A08 minta hal yang sama buat kodemu.

### A07 — Authentication Failures: Act 1

**Tiga kata yang orang sering ketuker:**
- **Identification** — SIAPA yang kamu klaim? Nunjukin username/email/user-id. *Jembatan DB: kayak primary key yang ngenalin baris.*
- **Authentication** — BUKTIIN itu. Nyupply secret (password, OTP, key) yang cuma dipunya user itu. *Jembatan DB: kayak kolom password yang verifikasi kunci.*
- **Session mgmt** — TETEP TERBUKTI. Setelah login, session ID gantiin bukti itu di tiap request berikutnya. *Jembatan DB: kayak baris di tabel sessions, dicari lewat token.*

**Tes bau: tanda-tanda aplikasi punya auth failure.** Credential stuffing dibolehin — nggak ada throttle, nggak ada MFA — penyerang muter ulang password dari breach dump. Password lemah/default diterima — nerima 'Password1', 'admin/admin', secret pendek. Brute force memungkinkan — percobaan login nggak dibatasi, nggak ada lockout atau delay. Pemulihan password nggak aman — 'security question'/reset token yang gampang ditebak. Plaintext/hashing lemah — password disimpen nggak di-hash atau pakai MD5/SHA-1. Session handling rusak — ID gampang ditebak, ID di URL, nggak ada rotasi setelah login.

**Ngaudit `login()`-nya Rafa:**
```php
// BiteClub — login.php (versi rilis)
$user = $db->query(
  "SELECT id, pass FROM users
   WHERE email = '" . $_POST['email'] . "'"   // (1)
)->fetch();

if ($user && $_POST['pass'] === $user['pass']) {    // (2)
    $_SESSION['uid'] = $user['id'];                 // (3)
    // nggak ada attempt limit, nggak ada MFA, nggak ada rotasi
}
```
1. Perbandingan tersimpen plaintext + string SQL dirakit lewat concatenation — password bisa dibaca DAN bisa di-injeksi.
2. `===` bandingin password mentah ke nilai yang tersimpen. Nggak ada hashing, nggak ada `password_verify()`.
3. Session dibuat tanpa attempt limit, tanpa MFA, dan ID-nya nggak pernah di-regenerate setelah login.

**Gimana kelemahan itu jadi breach:**
1. **Acquire** — ambil breach dump publik: jutaan pasangan email+password
2. **Stuff** — script login ke BiteClub — nggak ada lockout, jadi coba semua
3. **Land** — password yang dipake ulang berhasil. Penyerang sekarang jadi user valid yang login
4. **Persist** — session ID nggak pernah di-rotate → fixate/replay buat akses yang bertahan

**Dampaknya:** account takeover dalam skala besar, paparan data pelanggan dan detail pembayaran, dan pijakan buat eskalasi privilege — **semua tanpa satu pun "hacking"**, cuma password yang dipake ulang ketemu app yang nggak pernah bilang "tidak".

### Kasus: waktu kredensial lemah buka gerbang
**Colonial Pipeline (2021).** Satu akun VPN yang dikompromikan — **tanpa multi-factor authentication** — ngebolehin penyerang masuk. Password-nya belakangan ketemu di set data breach, konsisten sama pemakaian ulang. **Hasil: salah satu pipa bahan bakar terbesar AS mati berhari-hari; disrupsi supply meluas — dari satu kontrol yang hilang.** Pelajaran A07: **identitas adalah perimeternya.**

**Yang dipetik auditor:** faktor kedua kemungkinan besar bisa nge-blok masuknya langsung. Akun yang nggak dipakai/legacy harus dimatiin — makin sedikit pintu. Screening password yang udah pernah bocor nangkep kredensial yang dipake ulang. Event auth harus di-log dan di-alert (nyambung ke A09).

### A08 — Software & Data Integrity Failures: Act 2

**Integrity = "apa ini nyampe TANPA DIUBAH, dari yang gue harapin?"** Kategori baru 2021. Bukan soal kode-nya buggy atau enggak — ini soal apa kamu **MEMVERIFIKASI** bahwa kode, update, dan data itu nggak diutak-atik sebelum kamu percaya.

- **Untrusted dependencies** — plugin, library, modul yang ditarik dari sumber, repo, atau CDN yang belum diverifikasi
- **Insecure CI/CD pipeline** — sistem build/deploy yang nerima kode atau artifact tanpa ngecek signature
- **Auto-updates tanpa signing** — update didownload & diterapkan tanpa verifikasi integritas sumbernya
- **Insecure deserialization** — objek serial yang bisa dilihat & diubah penyerang, diubah balik jadi objek hidup

**Sinyal dan insting database yang udah kamu punya:**
- **CHECKSUM** buktiin backup yang di-restore nggak korup
- **FOREIGN KEY** nolak baris yang ngerusak referential integrity
- **Transaction LOG** bikin kamu bisa percaya state-nya kecapai secara sah

**A08 minta kamu perluas insting itu, dari data ke KODE dan UPDATE.**

**Sinyal audit:** Dependency dari URL random, bukan registry/lockfile terpercaya. Nggak ada pengecekan signature/hash pada update atau artifact yang didownload. Secret CI/CD kebuka; siapa pun bisa ngubah build-nya. `unserialize()`/pickle pada data yang ngelintas trust boundary. State sisi-client dikirim balik ke server dan dipercaya apa adanya.

**Ngaudit updater "deals plugin"-nya Rafa:**
```php
// BiteClub — update_deals.php
$pkg = file_get_contents(
   "http://deals.cdn.example/latest.phar"
);  // (1) nggak ada TLS, nggak ada signature

file_put_contents("plugins/deals.phar", $pkg);
require "plugins/deals.phar";  // (2) jalanin

// belakangan: restore keranjang user dari cookie
$cart = unserialize($_COOKIE['cart']); // (3)
```
1. Update diambil lewat HTTP polos tanpa digital signature — siapa pun di jalur bisa nukar filenya.
2. Package yang nggak terverifikasi langsung dieksekusi. Kode yang udah diutak-atik sekarang jalan dengan hak akses app-nya.
3. `unserialize()` pada data cookie yang dikontrol penyerang — klasik **insecure deserialization (CWE-502)**.

**Satu artifact yang diutak-atik, banyak korban:**
1. **Tamper** — penyerang modifikasi package atau kompromi CDN/langkah build
2. **Distribute** — tiap instalasi narik "update"-nya — kelihatan normal dan terpercaya
3. **Execute** — kode yang nggak terverifikasi jalan dengan hak akses app di tiap host
4. **Spread** — satu perubahan supply chain nyampe ribuan korban downstream

**Kenapa ini sangat berbahaya:** kode jahatnya dikirim lewat kanal yang **TERPERCAYA** dan sering "ditandatangani" secara benar sejauh yang korban tau. **Defender nggak sedang nyari — mereka udah percaya sumbernya.**

### Kasus: SolarWinds Orion, breach supply chain buku teks
**SolarWinds (2020).** Penyerang nyusup build pipeline dan nyisipin komponen jahat ke update software Orion. Update-nya ditandatangani dan didistribusikan normal — **ini bukan bug di kode, ini kerusakan di PROSES-nya.** Jangkauannya: update jahat itu didistribusikan ke lebih dari **18.000 organisasi**, dengan sekitar 100 dilaporkan terdampak lebih jauh — salah satu breach paling signifikan dari jenis ini.

**Bentuk yang sama dengan updater Rafa:** kepercayaan ditaruh di kanal update tanpa bukti integritas independen. Signature yang valid aja NGGAK cukup — **BUILD-nya sendiri yang dikompromikan.** Pertahanan = verifikasi integritas di tiap langkah, dari kode sampe production. Tooling: **OWASP Dependency-Check/CycloneDX** buat komponen & SBOM.

### Pencegahan: login Rafa, dibangun ulang dengan aman
```php
$stmt = $db->prepare(                 // (1) parameterized
  "SELECT id, pass_hash FROM users WHERE email = ?");
$stmt->execute([$_POST['email']]);
$u = $stmt->fetch();

if ($u && password_verify(             // (2) hash kuat
        $_POST['pass'], $u['pass_hash'])
    && rate_ok($ip) && mfa_ok($u)) {   // (3) throttle+MFA
  session_regenerate_id(true);         // (4) rotasi ID
  $_SESSION['uid'] = $u['id'];
}
```
1. Prepared statement — input nggak pernah bisa ngubah query (sekalian ngebunuh injection-nya juga).
2. `password_verify()` ngelawan hash satu-arah yang kuat (bcrypt/argon2). Nggak ada plaintext.
3. Gerbang rate-limit + MFA sebelum session diterbitkan — nyetop stuffing & brute force.
4. Regenerate session ID pas login ngalahin session fixation.

### MFA, kontrol bernilai tertinggi
Kombinasiin faktor dari kategori berbeda biar password curian doang nggak cukup:
- **Something you KNOW** — password, PIN, passphrase
- **Something you HAVE** — authenticator app (TOTP), security key, device
- **Something you ARE** — sidik jari, wajah, biometrik lain

**Catatan auditor:** utamain TOTP berbasis app atau hardware key daripada SMS (risiko SIM-swap). Wajibkan MFA di akun admin & remote access dulu. Padukan dengan screening password bocor dan lockout yang masuk akal — tapi hindari lockout yang terlalu agresif sampe jadi denial-of-service.

### Updater Rafa, dibikin bisa dipercaya
```php
$pkg = https_get(                      // (1) TLS aja
  "https://deals.example/latest.phar");

if (!sig_verify($pkg, $pubkey,         // (2) signature
                $detached_sig)) {
    log_and_abort("bad signature");    //     fail closed
}
require "plugins/deals.phar";

$cart = json_decode($_COOKIE['cart'],  // (3) data, bukan
                    true);             //     objek
verify_hmac($_COOKIE['cart'], $key);   // (4) tag integritas
```
1. Ambil lewat TLS dari host terpercaya — nggak ada teks polos, nggak gampang ditukar di tengah jalan.
2. Verifikasi digital signature sebelum dipakai; **fail closed** kalau nggak cocok.
3. Deserialize ke DATA murni (`json_decode`), jangan pernah ke objek hidup dari byte yang nggak dipercaya.
4. Lampirkan & verifikasi **HMAC** biar state cookie yang diutak-atik ditolak.

### Di luar kode: mengamankan supply chain
- **Trusted sources only** — tarik dependency dari registry yang udah divetting dengan lockfile; pertimbangkan repo internal yang known-good
- **Sign & verify everything** — digital signature pada update, artifact, dan data serial; verifikasi sebelum dipercaya
- **Harden the CI/CD pipeline** — access control, manajemen secret, review perubahan kode & config sebelum merge
- **Use SCA/SBOM tooling** — OWASP Dependency-Check & CycloneDX buat nandain komponen yang udah dikenal rentan
- **Review changes** — review manusia atas perubahan dependency dan konfigurasi buat nangkep tambahan jahat
- **Integrity for serialized data** — HMAC/signature pada objek atau state apa pun yang ngelintas trust boundary

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — kontennya semua teks dan blok kode.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **TOTP** | Time-based One-Time Password, kode sekali pakai yang berubah tiap beberapa detik, dihasilkan authenticator app |
| **SIM-swap** | Serangan di mana nomor HP korban dipindahin ke SIM card penyerang, buat nyegat OTP lewat SMS |
| **`.phar`** | Format arsip PHP yang bisa berisi kode yang bisa langsung dieksekusi |
| **`json_decode()`** | Fungsi PHP buat ngubah string JSON jadi data PHP biasa (array/object stdClass), TANPA menjalankan kode apa pun |
| **`sig_verify()` / HMAC** | Mekanisme buat memverifikasi bahwa data/file nggak diubah sejak ditandatangani/di-hash oleh pihak terpercaya |
| **Fail closed** | Prinsip: kalau verifikasi gagal, TOLAK aksinya secara default (lawan dari "fail open" yang malah ngelanjutin) |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan kasus Colonial Pipeline (A07 — kredensial VPN tanpa MFA) dengan SolarWinds (A08 — build pipeline dikompromikan). Analisis: kenapa "satu kontrol yang hilang" di Colonial Pipeline punya dampak yang bisa langsung dikaitkan ke SATU akun, sementara di SolarWinds dampaknya "menyebar" ke 18.000+ organisasi meskipun akar masalahnya juga "satu titik lemah"?
2. **(C4 – Analisis)** Kode `unserialize($_COOKIE['cart'])` di updater Rafa itu masalah A08 (integrity), tapi konsepnya mirip sama masalah "percaya hidden field mentah" yang dibahas di W07 (anti-pattern). Analisis: apa persamaan DAN perbedaan mendasar antara "insecure deserialization" dan "trusting client-controlled state" secara umum?
3. **(C5 – Evaluasi)** Sebuah tim nerapin MFA cuma buat akun ADMIN, nggak buat akun user biasa, dengan alasan "user biasa nggak punya akses sensitif." Evaluasi keputusan ini pakai skenario credential stuffing (slide 10) — apa risiko yang MASIH ada buat akun user biasa meskipun mereka "nggak sensitif secara individual"?
4. **(C5 – Evaluasi)** Bandingkan efektivitas dua kontrol A08 ini buat kasus SolarWinds-style attack: (a) digital signature verification pada update, vs (b) code review manusia di CI/CD pipeline. Kalau signature-nya SENDIRI bisa "valid" (karena build process-nya yang dikompromikan), kontrol mana yang lebih mungkin nangkep serangan ini duluan?
5. **(C6 – Cipta)** BiteClub mau nambah fitur baru: driver bisa update status pengiriman lewat app mobile yang manggil API `PATCH /orders/{id}/status`. Rancang pengecekan keamanan (gabungan A07 + A08) buat endpoint ini — minimal sebutkan: (a) gimana driver dibuktikan identitasnya, (b) gimana session/token-nya dijaga, (c) gimana data status yang dikirim divalidasi/diverifikasi integritasnya sebelum disimpan ke database.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W10 - Code Auditing OWASP III]]
- [[W12 - Code Auditing OWASP V]]
- [[SecureProgramming - Review dan Glosari]]
