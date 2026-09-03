---
matkul: Secure Programming
minggu: 7
sks: 3
sumber: S7_Review.pptx
tags: [kuliah/secure-programming, minggu/w07]
status: draft
diproses: 2026-09-04
---

# W07 — Review: Menyambung Semua Topik Jadi Satu Alur Aman

## Ringkasan
> - Ini review tengah semester, nyambungin W01–W06 lewat cerita baru: **K-Clinic**, portal janji temu pasien yang mau rilis. Tim database bilang "tabel dan query udah siap." Tim keamanan nanya: **"Bisa nggak input user jadi jalur serangan?"**
> - Misi hari ini: nyambungin ulang topik-topik sebelumnya jadi satu alur kerja aman: **PHP request → session/cookie/array → validasi → file upload → manipulasi database → output aman.**
> - Query SQL yang VALID bukan otomatis operasi web yang AMAN — konteks web nambahin identitas, otorisasi, state, dan input yang bermusuhan.
> - Enam pola **anti-pattern** yang harus dicari duluan sebelum nyari serangan canggih: SQL concatenation, ownership check yang hilang, mass assignment, percaya hidden field mentah-mentah, error leakage verbose, nggak ada transaction boundary.
> - Alur kerja aman: **Validate → Authenticate → Authorize → Parameterize → Store safely → Encode output.** Keamanan itu bukan satu langkah — itu alur dari input, ke proses, ke penyimpanan, sampai output.
> - Kasus baru minggu ini: **MOVEit Transfer 2023** (CVE-2023-34362) — SQL injection di aplikasi transfer file, dieksploitasi grup CL0P, nyambungin file handling + input web + akses database + batas autentikasi + patching + incident response — semua topik jadi satu.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Trust boundary | Titik di mana data berpindah dari "dikontrol sistem" ke "dikontrol user" atau sebaliknya |
| Anti-pattern | Pola kode yang KELIHATAN jalan tapi nyimpen risiko keamanan yang dikenal |
| Mass assignment | Nyimpen semua field dari request langsung ke database tanpa filter |
| Secure workflow | Urutan langkah baku: validate → authenticate → authorize → parameterize → store → encode |
| Blast radius | Seberapa luas dampak kalau satu kerentanan berhasil dieksploitasi |

## Isi

### Cerita: The App Before Release
**K-Clinic** adalah portal janji temu kecil yang dibangun tim junior. Pasien daftar, login, upload dokumen identitas, dan booking janji temu. Staf bisa lihat, update, dan hapus record janji temu.

Tim database bilang: **"Tabel dan query udah siap."** Tim keamanan nanya: **"Bisa nggak input user jadi jalur serangan?"**

**Lensa cybersecurity:** tiap fitur PHP yang nerima, nyimpen, atau nampilin data yang dikontrol user harus di-review sebagai **batas keamanan (security boundary)**.

**Misi hari ini:** nyambungin ulang topik-topik sebelumnya jadi satu alur kerja aman: **PHP request → session/cookie/array → validasi → file upload → manipulasi database → output aman.**

### Jalur penuh data user
**Browser → PHP Request → Session/Cookie → Validation → File/DB → Response.** Satu langkah yang nggak aman bisa ngerusak langkah berikutnya.

**Pertanyaan review keamanan di tiap langkah:** apakah datanya dipercaya? Siapa yang ngontrol dia? Apa yang kejadian kalau dia rusak, kegedean, jahat, atau diputar ulang (replayed)?

**Mindset database:** query SQL yang valid **bukan otomatis** operasi web yang aman. Konteks web nambahin identitas, otorisasi, state, dan input yang bermusuhan.

### Recall konsep: PHP, superglobal, dan output

**PHP jalan di server.** Browser ngirim HTTP request. PHP bikin response-nya. User bisa mengubah data request sebelum nyampe ke PHP.

**Superglobal adalah pintu input.** `$_GET`, `$_POST`, `$_COOKIE`, `$_SESSION`, `$_FILES`, dan `$_SERVER` itu praktis, tapi **nggak otomatis aman**.

**Output juga bikin risiko.** Data yang ditampilin balik ke HTML, JavaScript, SQL, atau path file harus ditangani sesuai konteksnya.

```php
$email = $_POST['email'] ?? '';
// Keamanan dimulai SEBELUM dipakai
// Validate → Authorize → Process → Encode
```

**Baseline keamanan PHP sebelum ngoding:** matiin tampilan error detail di production; log error dengan aman sebagai gantinya. Simpen rahasia di luar web root dan di luar source code publik. Batasi ukuran upload, ukuran request, memory, dan waktu eksekusi. Pakai HTTPS dan konfigurasi cookie yang aman. Jalanin akun database dengan **least privilege**, bukan sebagai root/admin. **Secure programming bukan cuma "kode yang bagus" — dia juga nyakup konfigurasi runtime yang aman.**

### Session, Cookie, dan Array: tiga penyimpanan berbeda
| **Array** | **Cookie** | **Session** |
| --- | --- | --- |
| Nilai terstruktur di PHP | Data kecil di browser | State di sisi server |
| Buat ngelompokin field request, metadata file upload, baris database | Berguna buat preferensi atau session ID | Berguna setelah login dan lintas halaman |
| Risiko: percaya struktur tanpa ngecek kunci dan tipe | Risiko: dikontrol user, bisa direplay, dicuri, atau diubah | Risiko: fixation, hijacking, kegagalan timeout |

**Aturan keamanan: lokasi penyimpanan mengubah tingkat kepercayaannya.**

**Cerita ancaman session: "Aku adalah user yang login."** Jalur serangan di K-Clinic: penyerang nyolong session ID dari cookie nggak aman atau komputer bersama → server nerima ID-nya dan muat session korban → penyerang ngubah janji temu atau ngeliat record pasien. Kontrol yang diterapkan: regenerate session ID setelah login dan perubahan privilege; pakai atribut cookie `HttpOnly`, `Secure`, dan `SameSite`; set idle dan absolute timeout; hancurin session pas logout.

### Validasi, sanitasi, encoding: gerbang pertama
**Validation nanya:** apa nilai ini bisa diterima buat field ini? Cek tipe, panjang, format, rentang, dan nilai yang diizinkan. **Sanitization nanya:** bisa nggak kita ubah atau buang karakter yang nggak aman? Pakai dengan hati-hati; jangan dianggap sebagai perbaikan universal. **Encoding nanya:** gimana caranya nilai ini ditampilin dengan aman di konteks HTML, attribute, JavaScript, URL, atau SQL?

```php
$email = $_POST['email'] ?? '';
if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    exit('Invalid email');
}
echo htmlspecialchars($email, ENT_QUOTES, 'UTF-8');
```
**Ide: validasi SEBELUM diproses, encode SEBELUM ditampilin.**

**Cerita mikro Stored XSS: field nama.** Pasien masukin ini sebagai display name: `<script>fetch('/steal?c='+document.cookie)</script>`. Kalau disimpen dan ditampilin tanpa encoding: tiap staf yang buka halaman itu ngejalanin kode penyerang. Session cookie dan data internal bisa keekspos. **Penyimpanan database bikin serangannya persisten.**

### File Upload: alur simpel, risiko besar
**Sudut pandang pemula:** "Upload artinya mindahin file dari storage sementara ke folder aplikasi." **Sudut pandang keamanan:** "Upload artinya nerima byte, nama, tipe, dan ukuran yang dikontrol penyerang ke servermu."

**Cerita ancaman: `avatar.php`.** Developer cuma ngecek: "namanya berakhiran `.jpg`". Penyerang upload `avatar.php.jpg` atau file dengan content type yang dimanipulasi. Filenya nyangkut di folder uploads publik. **Kalau servernya ngejalanin dia, upload jadi remote code execution.** Kalau cuma tersimpen, dia tetep bisa mungkinin hosting malware, XSS, atau kebocoran data.

**Yang bikin ini berbahaya:** metadata file bisa bohong. Nama file bisa berisi trik path. Isi file bisa raksasa, aktif, atau nggak terduga. Lokasi penyimpanan bisa ngubah file data jadi kode yang bisa dieksekusi. **Jangan pernah percaya ekstensi doang.**

```php
$allowed = ['image/jpeg' => 'jpg', 'image/png' => 'png'];
$finfo = new finfo(FILEINFO_MIME_TYPE);
$mime = $finfo->file($_FILES['photo']['tmp_name']);

if (!isset($allowed[$mime])) { exit('Invalid file'); }
$name = bin2hex(random_bytes(16)) . '.' . $allowed[$mime];
```

### Koneksi database: review jembatan kepercayaannya
PHP nggak "ngomong SQL" begitu aja; dia pakai database driver. Connection string milih host, nama database, charset, dan driver. Kredensial ngenalin apa yang aplikasinya diizinin lakuin. Opsi koneksi nentuin error handling dan perilaku fetch. **Akun database harus cuma punya izin yang dibutuhin aplikasinya.**

> Kesalahan keamanan: pakai akun database dengan hak akses tinggi berarti **satu SQL injection bisa jadi kompromi database penuh**.
> Mindset aman: koneksi PHP-DB itu jembatan kepercayaan. Lindungi jembatannya dengan least privilege, parameterisasi, dan error handling yang aman.

```php
$dsn = 'mysql:host=localhost;dbname=kclinic;charset=utf8mb4';
$pdo = new PDO($dsn, $dbUser, $dbPass, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,
]);
```

### SELECT, INSERT, UPDATE, DELETE — recap singkat lewat K-Clinic
**SELECT aman:**
```php
$stmt = $pdo->prepare('SELECT id, patient_name, schedule_date FROM appointments WHERE patient_id = :patient_id');
$stmt->execute(['patient_id' => $currentUserId]);
```
Parameterisasi misahin struktur SQL dari nilai datanya. Pakai identitas user dari **session terpercaya**, bukan hidden form field. Validasi filter: rentang tanggal, nilai status, kolom sort.

**INSERT tanpa nyimpen risiko:**
```php
$stmt = $pdo->prepare('INSERT INTO appointments (patient_id, note, schedule_date) VALUES (:patient_id, :note, :schedule_date)');
```
Sebelum INSERT: validasi aturan bisnis. Sesudah INSERT: encode data pas ditampilin nanti — data tersimpen bisa jadi **stored XSS** kalau output-nya ditangani sembarangan. **Safe write = validate + parameterize + encode later.**

**UPDATE dengan otorisasi baris:**
```php
$stmt = $pdo->prepare('UPDATE appointments SET note = :note WHERE id = :id AND patient_id = :patient_id');
```
Kesalahan umum: cuma ngecek `WHERE id = :id` bisa bikin celah otorisasi — user bisa ngubah record orang lain cuma dengan ganti ID.

**DELETE (lebih disukai soft delete):**
```php
$stmt = $pdo->prepare('UPDATE appointments SET deleted_at = NOW() WHERE id = :id AND patient_id = :patient_id');
```
**Delete bukan cuma SQL; itu risiko bisnis.**

### Anti-pattern manipulasi data yang harus dideteksi
| Anti-pattern | Contoh |
| --- | --- |
| SQL concatenation | `"SELECT * FROM users WHERE id = " . $_GET['id']` |
| Ownership check hilang | `UPDATE record WHERE id = :id` (tanpa cek pemilik) |
| Mass assignment | Nyimpen semua field request langsung ke database |
| Percaya hidden field mentah | Makai `role`, `price`, `owner_id`, atau `status` dari field sisi client |
| Error leakage verbose | Nampilin SQL error, stack trace, path file, atau kredensial |
| Nggak ada transaction boundary | Tulis parsial ninggalin state bisnis yang nggak konsisten |

**Kebiasaan review: cari pola-pola ini di kode sebelum nyari serangan canggih.**

### Kasus 1: TalkTalk 2015 (lagi)
Penyerang eksploitasi SQL injection di halaman web. ICO nyatet SQL injection itu serangan yang udah dipahami dengan baik dan pertahanan yang udah dikenal. Serangan SQL injection sebelumnya di Juli dan September 2015 nggak ditindaklanjuti karena monitoring yang nggak memadai. **Pelajaran secure programming: prepared statement ngurangin risiko injection. Halaman lawas harus di-review, bukan diabaikan. Monitoring harus mendeteksi percobaan serangan berulang. Utang keamanan jadi risiko bisnis.**

### Kasus 2 (baru): MOVEit Transfer 2023
**MOVEit Transfer** adalah aplikasi web managed file transfer yang dipakai buat data bisnis sensitif. **CVE-2023-34362** adalah kerentanan SQL injection di aplikasi web MOVEit Transfer. **CISA** melaporkan grup **CL0P** mengeksploitasi kerentanan zero-day itu, dimulai dari SQL injection. **NVD** nyatet penyerang yang nggak terautentikasi bisa akses database dan berpotensi ngubah atau ngehapus elemen database.

**Kenapa kasus ini pas buat review:** kasus ini nyambungin **file handling, input aplikasi web, akses database, batas autentikasi, patching, dan incident response** — semua topik yang udah dibahas dari W02 sampai W06.

**Diskusi:** kalau K-Clinic ngurusin upload file pasien, kontrol MANA dari materi hari ini yang bisa ngurangin blast radius?

### Alur kerja aman: dari Request ke Database
Alur kerja aman buat nangenin request user: pertama, **validasi** input dengan ngecek tipe, panjang, format, rentang, dan aturan allowlist. Kedua, **autentikasi** user buat tau siapa yang bikin request. Ketiga, **otorisasi** aksinya buat mastiin user diizinin akses atau ngubah target record.

Setelah itu, **parameterize** semua nilai SQL pakai prepared statement. Lalu, **simpan data dengan aman**, termasuk file, metadata, secrets, dan log. Terakhir, **encode output**-nya dengan bener sebelum ditampilin di HTML, JavaScript, atau URL.

**Poin utamanya: keamanan bukan satu langkah. Itu alur dari input, ke proses, ke penyimpanan, sampai output.**

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — kontennya semua teks, diagram alur, dan blok kode.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **CISA** | Cybersecurity and Infrastructure Security Agency, badan keamanan siber pemerintah AS |
| **NVD** | National Vulnerability Database, database resmi kerentanan software AS |
| **Zero-day** | Kerentanan yang dieksploitasi penyerang SEBELUM vendor sempat bikin/rilis patch-nya |
| **Managed file transfer (MFT)** | Aplikasi enterprise buat transfer file bisnis yang sensitif secara aman |
| **Blast radius** | Seberapa jauh dampak menyebar kalau satu kerentanan berhasil dieksploitasi |
| **Idle timeout / absolute timeout** | Batas waktu session karena nggak ada aktivitas, vs batas waktu total session terlepas dari aktivitas |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Petain jalur data K-Clinic (Browser → PHP Request → Session/Cookie → Validation → File/DB → Response) ke MASING-MASING dari enam anti-pattern di slide 20. Untuk tiap anti-pattern, di titik mana di jalur itu dia paling mungkin muncul?
2. **(C4 – Analisis)** Kasus MOVEit (2023) dan TalkTalk (2015) sama-sama SQL injection, tapi MOVEit levelnya "unauthenticated attacker" (nggak perlu login) sementara materi W05-W06 selalu ngasumsiin ada `$_SESSION['patient_id']` yang udah login. Analisis: gimana skenario "attacker nggak terautentikasi" ini bisa kejadian meskipun aplikasinya PUNYA sistem login — konsep apa dari W01 (attack surface / trust boundary) yang paling relevan jelasin ini?
3. **(C5 – Evaluasi)** Sebuah kode di K-Clinic nge-pass semua review checklist SQLi (pakai prepared statement) DAN semua review checklist XSS (pakai `htmlspecialchars`). Evaluasi: apa review ini CUKUP buat nyatain "K-Clinic siap rilis"? Kaitkan jawabanmu ke keenam anti-pattern — sebutkan minimal 2 anti-pattern yang MASIH bisa lolos meskipun SQLi dan XSS udah ditutup.
4. **(C5 – Evaluasi)** Bandingkan tiga jenis "penyimpanan" (Array, Cookie, Session) dari sisi "siapa yang bisa saya percaya buat keputusan otorisasi". Kalau kamu cuma boleh pakai SATU dari ketiganya buat nyimpen role user ("staff" vs "patient"), pilih yang mana dan jelasin kenapa dua lainnya nggak cocok.
5. **(C6 – Cipta)** K-Clinic mau nambah fitur: staf bisa export daftar janji temu hari ini ke file CSV yang bisa didownload. Rancang alur kerja aman (Validate → Authenticate → Authorize → Parameterize → Store → Encode) buat fitur ini — jelasin di tiap langkah, ancaman spesifik apa yang lagi dicegah, dan sebutkan minimal SATU anti-pattern dari slide 20 yang paling relevan buat diwaspadai di fitur export ini.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W01 - Introduction to Web Application]]
- [[W02 - Session, Cookies & File Upload]]
- [[W03 - Web Form Processing and Security]]
- [[W04 - Web Form File Upload]]
- [[W05 - Web Database I]]
- [[W06 - Web Database II]]
- [[W08 - Code Auditing OWASP I]]
- [[SecureProgramming - Review dan Glosari]]
