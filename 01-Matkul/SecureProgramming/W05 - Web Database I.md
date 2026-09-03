---
matkul: Secure Programming
minggu: 5
sks: 3
sumber: S5-Web_Database_I.pptx
tags: [kuliah/secure-programming, minggu/w05]
status: draft
diproses: 2026-09-04
---

# W05 — Web Database (I): Connect & SELECT & INSERT

## Ringkasan
> - Cerita: mahasiswa mau daftar workshop cybersecurity lewat **Campus Event Portal**. Form-nya nyimpen nama, email, ID mahasiswa, event pilihan. PHP ngobrol ke MySQL lewat **PDO**. **Tiap operasi database bisa melindungi data atau malah mengekspos-nya.**
> - "Constructing a web application" itu bukan cuma "bikin dia jalan" — itu **bikin dia gagal dengan aman**.
> - Sambungan PHP↔database itu **jembatan kepercayaan**: kredensial jangan hardcode di kode, akun DB harus **least privilege** (cuma hak yang dibutuhin), dan error mentah jangan pernah ditampilin ke user.
> - **SELECT nggak aman**: nyambungin ID dari `$_GET` langsung ke query — penyerang bisa ganti `id=3` jadi `id=3 OR 1=1`, bikin query balik SEMUA baris.
> - **Prepared statement** motong query itu: struktur SQL dipisah dari data. Placeholder (`:id`) diisiin nilai sebagai DATA, bukan sebagai bagian perintah — jadi penyerang nggak bisa ngubah logika query-nya.
> - Password **JANGAN PERNAH** disimpen plaintext — pakai `password_hash()` waktu nyimpen, `password_verify()` waktu login.
> - Kasus nyata: **TalkTalk 2015**, SQL injection bocorin data ~157 ribu pelanggan, denda £400.000 dari regulator UK.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| PDO | Lapisan akses database PHP yang konsisten, mendukung prepared statement |
| DSN | String koneksi yang nentuin host, nama database, dan charset |
| Least privilege | Prinsip: akun cuma dikasih hak akses yang beneran dia butuhin |
| Prepared statement | Query yang strukturnya (SQL) dipisah dari datanya (nilai) |
| Placeholder | Tanda pengganti (misal `:id`) di query yang nanti diisi nilai sebagai data |
| SQL Injection | Serangan yang bikin input user ngubah LOGIKA query, bukan cuma NILAI-nya |
| `password_hash()` / `password_verify()` | Fungsi PHP buat nge-hash password waktu simpen, dan ngecek password waktu login |

## Isi

### Cerita pembuka: Campus Event Portal
Mahasiswa mau daftar workshop cybersecurity. Web app-nya harus nyimpen dan nampilin pendaftaran dengan aman. Form-nya ngumpulin nama, email, ID mahasiswa, dan event yang dipilih. PHP nerima data form dan ngobrol ke MySQL lewat **PDO**. **Tiap operasi database bisa melindungi data atau malah mengekspos-nya.**

Kenapa ini penting: web app umumnya nyimpen identitas, transaksi, log, dan konten dari user. **Satu query nggak aman bisa nge-bypass autentikasi, bocorin record, atau ngerusak data.** Secure programming artinya bikin fitur SAMBIL ngontrol jalur kegagalannya. **Membangun web app bukan cuma "bikin dia jalan" — itu bikin dia gagal dengan aman.**

### Arsitektur dasarnya
Alur yang ada di hampir tiap fitur web database: **Browser** ngirim input lewat GET atau POST → **PHP** nentuin query apa yang harus dijalanin → **PDO** nyediain lapisan akses database yang konsisten → **Database** ngebalikin baris atau nge-konfirmasi perubahan.

**Sebelum nulis kode, petain trust boundary-nya dulu:** input user itu nggak dipercaya, meskipun datengnya dari form yang "normal". Lapisan PHP bisa aja nyampur data sama perintah SQL tanpa sengaja. Akun database mungkin punya hak akses lebih dari yang dibutuhin fiturnya. Error bisa nyingkap nama tabel, path, atau detail koneksi.

### Connect to Database: pola PDO
```php
$dsn = 'mysql:host=localhost;dbname=campus_event;charset=utf8mb4';
$user = getenv('DB_USER');
$pass = getenv('DB_PASS');

$options = [
  PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
  PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
  PDO::ATTR_EMULATE_PREPARES => false,
];

$pdo = new PDO($dsn, $user, $pass, $options);
```
Ide kuncinya: aplikasi buka jalur terkontrol ke database; kredensial **jangan** di-hardcode. `ERRMODE_EXCEPTION` bikin error di-throw sebagai exception; `FETCH_ASSOC` bikin hasil query jadi array asosiatif; `EMULATE_PREPARES => false` matiin emulasi biar prepared statement beneran dijalanin native oleh driver-nya.

**Checklist keamanan koneksi:** kredensial disimpen di luar web root dan di luar repo source code; akun database **least privilege** (cuma izin SELECT/INSERT yang dibutuhin); error mentah database nggak ditampilin ke user; pakai `utf8mb4` biar nggak ada kejutan encoding; sentralisasi logika koneksi biar setting keamanannya konsisten di mana-mana.

### Story Step 2: "Show Event Details" — SELECT nggak aman
Panitia mau halaman `/event.php?id=3`. User klik link event, browser ngirim ID-nya. PHP pakai ID itu buat nge-query tabel `events`. Pertanyaan keamanan: **bisa nggak ID itu ngubah logika query-nya?**

**Bug-nya bukan sintaks SQL, tapi cara SQL-nya dirakit.** User ngirim ID lewat browser, misal `id=3`. PHP baca nilai itu dan **nyambungin langsung** ke string SQL. Masalahnya bukan sintaks, tapi **cara query-nya dirakit** — karena input ditaro langsung di dalem perintah, penyerang bisa ganti nilai ID normal jadi sesuatu yang jahat, kayak `3 OR 1=1`.

Pas itu kejadian, query akhirnya nggak lagi minta satu event spesifik. Logikanya berubah, dan bisa ngebalikin banyak baris dari database. **Pelajaran kuncinya: input user nggak boleh pernah jadi bagian dari STRUKTUR perintah SQL.** Bahkan SELECT sederhana bisa berbahaya kalau input yang dikontrol penyerang di-concat langsung ke SQL.

### SELECT Query: versi aman pakai prepared statement
Cara aman: user ngirim parameter GET, misal `id=3`, lewat browser. Sebelum dipake, aplikasi validasi dulu buat mastiin itu integer yang bener. Kalau invalid, aplikasi langsung berhenti dan balikin error, misal HTTP 400.

Setelah validasi, aplikasi nyiapin SQL statement **terpisah dari data user**. Struktur query-nya di-fix dulu pakai placeholder kayak `:id`. Terus, input user di-bind ke placeholder itu dan dijalanin sebagai **data, bukan bagian perintah SQL**.

Karena pemisahan ini, **user nggak bisa ngubah logika query-nya**. Database cuma nerima nilai parameter yang aman, dan ngebalikin satu baris yang cocok, atau nggak ada baris kalau record-nya nggak ada.

**Pelajaran kuncinya: prepared statement bikin struktur SQL tetep fix, sementara validasi input mastiin format datanya bisa diterima. Bareng-bareng, mereka nyegah SQL injection dan bikin aplikasinya lebih aman.**

> [!info] Analogi
> Bayangin prepared statement kayak **formulir isian bank yang cetakannya udah baku**. Kolom "Jumlah Transfer" udah dicetak sebagai kolom terpisah — kamu nggak bisa nulis kalimat perintah di situ ("transfer semua saldo ke rekening lain"), yang bisa kamu tulis cuma ANGKA. Beda sama kalau kasirnya nulis ulang instruksimu di secarik kertas kosong — di situ kamu bisa nyelipin kalimat apa aja yang kedengeran kayak perintah resmi.

**Gimana prepared statement mengubah permainan.** Di versi nggak aman, input user langsung digabung sama perintah SQL. Artinya penyerang bisa nyelipin sesuatu kayak `3 OR 1=1`, dan database mungkin memperlakukan itu sebagai bagian dari logika SQL.

Dengan prepared statement, prosesnya beda. Struktur SQL-nya disiapin duluan pakai placeholder, kayak `:id`. Setelah itu, nilai user dikirim terpisah dan di-bind sebagai data.

**Jadi walaupun user masukin `3 OR 1=1`, database nggak memperlakukan itu sebagai sintaks SQL yang bisa dieksekusi. Dia perlakuin seluruh nilai itu sebagai data biasa.** Pelajaran kuncinya sederhana: prepared statement melindungi struktur query. Mereka nyegah input yang dikontrol penyerang ngubah makna perintah SQL-nya.

### SELECT: risiko di luar SQL injection
Risiko query SELECT nggak cuma soal SQL injection. Bahkan pas query udah dilindungi prepared statement, kita tetep perlu ngecek isu keamanan lain:

1. **Otorisasi** — bisa nggak user akses record user lain cuma dengan ganti ID?
2. **Data minimization** — apa query-nya cuma ngebalikin field yang dibutuhin halamannya?
3. **Pagination dan limit** — bisa nggak satu request narik terlalu banyak baris dari database?
4. **Error handling** — apa aplikasinya bocorin apakah sebuah record privat itu ada atau enggak?

**Pelajaran kuncinya: SELECT query yang aman harus ngelindungin DUA-DUANYA — struktur query DAN logika akses datanya.**

### Story Step 3: "Register a Participant" — INSERT
Sekarang aplikasinya harus nulis data. Pas peserta submit form pendaftaran, aplikasi nerima beberapa input, kayak nama, email, ID mahasiswa, dan event pilihan. Sebelum nyimpen data, PHP harus validasi field wajib dan cek aturan bisnis, misal apakah event-nya masih ada kursi.

Setelah input dianggap valid, aplikasi nginsert baris pendaftaran baru ke database. **Ide keamanan kuncinya: INSERT ngubah input user jadi data yang tersimpen.** Kalau input yang cacat nggak divalidasi dengan bener, dia bisa ngerusak database atau jadi sumber serangan di masa depan.

**INSERT nggak aman** — concatenation bikin masalah injection DAN masalah kualitas data. Aplikasi baca raw POST input dari form pendaftaran dan langsung nyambunginnya ke SQL statement. Ini berbahaya karena input user jadi bagian dari perintah SQL. Kalau inputnya ada tanda kutip, query bisa error. Kalau ada fragmen SQL, perintahnya bisa diubah penyerang. **Bahkan tanpa injection**, data invalid kayak format email salah atau ID event yang nggak valid tetep bisa kesimpen.

**Pelajaran kuncinya: INSERT nggak boleh ngandelin concatenation langsung. Input user harus divalidasi dulu, baru diinsert pakai prepared statement.**

### INSERT: pola PDO yang aman
Validate first, then insert using placeholders. Pertama, user submit data lewat form pendaftaran, kayak nama, email, dan ID event. Sebelum ngeinsert apa pun ke database, aplikasi PHP validasi input-nya: nama di-trim, format email dicek, ID event harus integer valid.

Kalau inputnya invalid, aplikasi berhenti dan balikin error. Ini nyegah data buruk atau nggak valid kesimpen.

Setelah validasi, aplikasi nyiapin SQL statement pakai placeholder kayak `:name`, `:email`, `:event_id`. Nilai input beneran lalu di-bind terpisah dan dijalanin sebagai data, bukan bagian perintah SQL.

**Pelajaran kuncinya sederhana: validasi dulu, pakai placeholder, bind nilainya, dan cuma simpen data yang bersih ke database.**

### INSERT dengan password: JANGAN PERNAH simpen plaintext
Sebuah aturan penting kalau pendaftaran bikin akun user: **jangan pernah simpen password dalam bentuk plaintext.**

Pas user submit password, aplikasi harus ngecek dulu kebutuhan dasar, kayak panjang minimum. Setelah itu, password-nya harus diproses pakai `password_hash()` sebelum disimpen ke database.

Database cuma boleh nyimpen **hash**-nya, bukan password aslinya. Artinya, walaupun databasenya bocor, password asli nggak langsung keliatan.

Nanti, pas login, aplikasi pakai `password_verify()` buat bandingin password user sama hash yang tersimpen.

**Pelajaran kuncinya sederhana: simpen hash password, jangan pernah plaintext.**

### Kontrol integritas data buat INSERT
Constraint database itu bagian dari secure programming, karena mereka **melindungi data bahkan pas aplikasinya bikin kesalahan**. Buat operasi INSERT, aplikasi harus validasi input dulu, tapi database juga harus nerapin aturan penting: field wajib pakai `NOT NULL`, record duplikat bisa dicegah pakai `UNIQUE`, dan `FOREIGN KEY` mastiin nilai kayak `event_id` beneran ngerujuk ke record yang ada.

Aturan validasi juga bisa membatasi nilai ke format atau rentang yang diharapkan. Pas beberapa tabel terkait di-update bareng, **transaction** bantu jaga data tetap konsisten.

**Poin utamanya sederhana: kode yang aman dan constraint database seharusnya melindungi aturan yang sama, dua kali.**

### Kasus nyata: TalkTalk SQL Injection Breach
Tahun 2015, penyerang eksploitasi kelemahan SQL injection di halaman web lawas TalkTalk. Regulator UK (**ICO**) melaporkan langkah pengamanan datanya nggak memadai. Data pribadi sekitar **156.959 pelanggan** diakses, dan ICO ngasih denda **£400.000**.

**Pelajaran: halaman web-database lawas tetap jadi bagian dari attack surface hari ini.**

## Diagram & Visual
- **Slide 6, 9–10, 13–14, 19–20 — diagram alur, screenshot kode, dan ilustrasi story flow untuk koneksi PDO, SELECT tidak aman/aman, dan alur INSERT.**
  ![[99-Assets/SecureProgramming/W05-slide06.png]]
  ![[99-Assets/SecureProgramming/W05-slide09.png]]
  ![[99-Assets/SecureProgramming/W05-slide10.png]]
  ![[99-Assets/SecureProgramming/W05-slide13.png]]
  ![[99-Assets/SecureProgramming/W05-slide14.png]]
  ![[99-Assets/SecureProgramming/W05-slide19.png]]
  ![[99-Assets/SecureProgramming/W05-slide20.png]]

> [!warning] Deck ini punya banyak gambar (15 total) yang kebanyakan versi visual dari kode dan alur yang udah dijelasin lengkap di teks note ini — nggak semuanya ditautkan satu-satu di atas biar nggak berlebihan. Kalau butuh liat tampilan visual aslinya, cek folder `99-Assets/SecureProgramming/W05-*`.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **PDO (PHP Data Objects)** | Lapisan abstraksi PHP buat akses database, mendukung banyak jenis DBMS dan prepared statement |
| **`utf8mb4`** | Charset MySQL yang mendukung karakter Unicode penuh (termasuk emoji), lebih aman dari kejutan encoding |
| **`PDO::ATTR_EMULATE_PREPARES`** | Setting PDO; kalau `false`, prepared statement dijalankan native oleh driver database, bukan disimulasikan PHP |
| **Trust boundary** | Garis pemisah antara data yang dikontrol sistem dan data yang datang dari user |
| **`FOREIGN KEY`** | Constraint database yang mastiin sebuah nilai kolom ngerujuk ke record valid di tabel lain |
| **Transaction (database)** | Kumpulan operasi database yang dijalankan sebagai satu unit — semua berhasil, atau semua dibatalkan |
| **ICO** | Information Commissioner's Office, regulator perlindungan data di Inggris |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan query SELECT nggak aman (`WHERE id = $id`) dengan versi prepared statement (`WHERE id = :id`). Kalau penyerang ngirim `id=3 OR 1=1` ke DUA-duanya, jelasin PERSIS di titik mana perbedaan hasilnya muncul — apa yang beda di level "bagaimana database membaca input itu"?
2. **(C4 – Analisis)** Slide 15 bilang "secure SELECT queries must protect both the query structure and the data access logic." Analisis skenario ini: sebuah halaman `/event.php?id=3` udah pakai prepared statement (aman dari SQLi), tapi TETEP bisa nampilin data event yang seharusnya cuma buat panitia. Kegagalan apa yang terjadi di sini, dan kenapa prepared statement doang nggak nutupnya?
3. **(C5 – Evaluasi)** Seorang developer nulis: `$hash = sha1($password); $db->query("INSERT ... ('$hash')");` — dia bilang ini udah "aman" karena password-nya udah di-hash (nggak plaintext) DAN dia inget buat nge-escape tanda kutipnya. Evaluasi: apa yang MASIH salah di pendekatan ini, dari sisi pilihan fungsi hash DAN dari sisi cara nulis query-nya?
4. **(C5 – Evaluasi)** Kasus TalkTalk 2015 kena denda karena "langkah pengamanan data nggak memadai", bukan cuma karena ada bug SQLi-nya. Menurutmu, dari materi minggu ini (koneksi least-privilege, prepared statement, hash password, constraint database), kontrol MANA yang paling mungkin, kalau diterapkan konsisten, bakal ngurangin DAMPAK breach — walaupun bug SQLi-nya tetep ada?
5. **(C6 – Cipta)** Rancang alur INSERT buat fitur baru: mahasiswa bisa ngasih rating 1–5 dan komentar buat sebuah event setelah acaranya selesai. Sebutkan minimal 4 langkah alurnya (validasi apa aja, prepared statement-nya kayak gimana, constraint database apa yang relevan) — pastikan alurnya nyegah minimal DUA jenis masalah yang dibahas minggu ini (misalnya: SQLi, dan satu masalah kualitas data/integrity lainnya).

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W03 - Web Form Processing and Security]]
- [[W06 - Web Database II]]
- [[W07 - Review]]
- [[SecureProgramming - Review dan Glosari]]
