---
matkul: Secure Programming
minggu: 4
sks: 3
sumber: S4-Web_Form_File_Upload.pptx
tags: [kuliah/secure-programming, minggu/w04]
status: draft
diproses: 2026-09-04
---

# W04 — Web Form File Upload

## Ringkasan
> - Cerita: **Raka** bikin form upload buat portal kampus dalam 5 menit — mahasiswa upload bukti tugas. Versi pertama cuma ngecek "ada file atau enggak". Terus ada "mahasiswa" yang upload: **`shell.php.jpg`**. Pertanyaan buat kelas: itu gambar, script PHP, atau dua-duanya?
> - File itu input user juga, tapi **lebih besar, lebih kompleks, kadang bisa dieksekusi**. Jangan pernah percaya nama file, ekstensi, MIME type, atau pengecekan client-side doang.
> - Lima cara serangan lewat upload: **Execute** (jalanin web shell), **Store** (nyimpen malware/konten ilegal), **Trigger** (eksploitasi bug library parser/gambar), **Exhaust** (habisin disk/memory), **Overwrite** (nimpa file yang udah ada).
> - Enam lapis validasi: **Request → Size → Type (ekstensi + MIME + magic bytes) → Name (rename acak) → Storage (di luar webroot) → Access (lewat otorisasi aplikasi)**.
> - Prinsip penyimpanan: **file yang diupload harusnya jadi DATA, bukan kode aplikasi yang bisa dieksekusi.**
> - Kasus nyata: **Equifax 2017** — bug parser upload multipart di Apache Struts (CVE-2017-5638) bocorin data pribadi **147 juta orang**.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| `$_FILES` | Array superglobal PHP yang nyimpen metadata file yang diupload |
| `move_uploaded_file()` | Fungsi PHP buat mindahin file dari lokasi sementara ke lokasi permanen |
| Web shell | Script (biasanya PHP) yang, kalau berhasil diupload dan dijalankan server, ngasih penyerang kendali command line lewat browser |
| Double extension | Trik nama file kayak `avatar.php.jpg` buat ngelabui pengecekan ekstensi |
| Polyglot file | File yang valid di lebih dari satu format sekaligus |
| Allowlist ekstensi | Daftar ekstensi yang diizinkan; selain itu ditolak |
| Magic bytes / MIME sniffing | Ngecek isi file beneran (bukan cuma nama/klaim browser) buat nentuin jenisnya |
| finfo | Ekstensi PHP buat baca MIME type asli dari isi file |
| Webroot | Folder yang bisa diakses langsung lewat URL |

## Isi

### Cerita pembuka: FitCampus Upload Portal
Tim kampus bikin web app kecil buat mahasiswa upload bukti tugas. Developer bernama **Raka** bikin form upload PHP simpel. Jalan dalam 5 menit. Mahasiswa upload laporan PDF dan screenshot. Admin download file-nya sebelum ngasih nilai.

Versi pertamanya cuma ngecek: **apakah filenya ada atau enggak.**

Terus ada "mahasiswa" yang upload: **`shell.php.jpg`**.

Pertanyaan buat kelas: **ini gambar, script PHP, atau dua-duanya?**

Sesi ini ngikutin app-nya Raka dari "jalan" sampai "bisa dipertanggungjawabkan (defensible)".

### Kenapa file upload berisiko
File itu input user, tapi **lebih gede, lebih kompleks, dan kadang bisa dieksekusi.**

| Input biasa | Input file |
| --- | --- |
| `name = "Raka"` | `filename = "report.pdf"` |
| `email = "raka@mail.com"` | `content` = binary / script / gambar |
| — | `size` = bisa gede banget |

Mindset keamanan: **jangan pernah percaya nama file, ekstensi, MIME type, atau pengecekan client-side sendirian.**

### Alur besar upload HTTP
Yang kejadian pas browser ngirim file ke server: **Browser** (form HTML milih file) → **HTTP POST** (request `multipart/form-data`) → **PHP** (file sementara terupload) → **Application** (validasi dan pindahin) → **Storage** (simpan aman atau tolak).

**Keputusan keamanannya harus kejadian SEBELUM file itu bisa dijangkau atau dieksekusi.**

### Form HTML sederhana
```html
<form method="POST" action="upload.php" enctype="multipart/form-data">
  <label>Upload report</label>
  <input type="file" name="report">
  <button type="submit">Upload</button>
</form>
```
- `method="POST"` — ngirim data ke server
- `enctype="multipart/form-data"` — **wajib** buat data file
- `name="report"` — jadi kunci di `$_FILES`
- **Input client-side bukan validasi keamanan**

### Anatomi upload di PHP: `$_FILES`
```php
$_FILES['report']['name']      // nama file asli
$_FILES['report']['type']      // MIME type yang diklaim browser
$_FILES['report']['tmp_name']  // path sementara di server
$_FILES['report']['error']     // kode status upload
$_FILES['report']['size']      // ukuran dalam byte
```
**Peringatan keamanan:** `name` bisa dikontrol penyerang. `type` bisa dipalsukan. `tmp_name` itu sementara dan nggak boleh diekspos. `error` wajib dicek sebelum diproses.

### Versi ngajar: `upload.php` minimal (sengaja nggak lengkap)
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $target = 'uploads/' . $_FILES['report']['name'];
    move_uploaded_file($_FILES['report']['tmp_name'], $target);
    echo 'Upload successful';
}
?>
```
**Diagnosa kelas:** Gimana kalau nama filenya `shell.php`? Gimana kalau filenya 500 MB? Gimana kalau file itu nimpa `index.php`? Gimana kalau `uploads/` bisa diakses lewat web?

### Threat model: apa yang bakal dicoba penyerang?
Keamanan file upload dimulai dengan nanya apa yang diinginkan penyerang:

- **Execute** — upload web shell atau script
- **Store** — nyembunyiin malware atau konten ilegal
- **Trigger** — eksploitasi bug parser/library gambar
- **Exhaust** — habisin disk atau memory pakai file gede
- **Overwrite** — nimpa file yang udah ada lewat tabrakan nama

Threat modeling ngubah "fitur upload" jadi daftar mode kegagalan yang spesifik.

### Threat #1: Web Shell (rekonstruksi dari konteks)
> [!warning] Slide ke-11 di deck asli nggak bisa diekstrak — antivirus nge-block file teks-nya karena isinya kemungkinan besar contoh kode **web shell** (script PHP satu-baris yang ngejalanin perintah OS lewat parameter web). Ini hal yang wajar buat materi tentang bahaya upload — AV mendeteksi pola itu sebagai malware signature, bukan salah software vault-nya. Bagian di bawah ini **direkonstruksi dari alur logis slide 10 → 12** (Threat Model → Threat #2: Dangerous Images), bukan kutipan persis dari slide.

Secara konsep, "Threat #1" ini ngejelasin skenario `shell.php.jpg` dari cerita pembuka: kalau server dikonfigurasi buat ngejalanin file berdasarkan ekstensi terakhir yang dikenali (`.php`), sebuah nama file dengan **double extension** kayak `shell.php.jpg` bisa tetep dieksekusi sebagai PHP oleh sebagian konfigurasi server (misalnya via `AddHandler` di Apache yang salah setting), meskipun kelihatan kayak file gambar. Begitu web shell itu berhasil ke-upload dan bisa diakses lewat URL publik, penyerang tinggal buka URL-nya lewat browser buat **ngejalanin perintah di server** — inilah yang OWASP sebut *Unrestricted File Upload* yang berujung *Remote Code Execution*.

> [!info] Analogi
> Bayangin loket penitipan barang di bandara yang cuma ngecek label koper ("ini koper baju"), tapi nggak pernah nge-scan isinya. Kalau ada yang nempelin label "baju" di koper isi barang berbahaya, dan petugasnya cuma percaya labelnya, barang itu lolos ke pesawat. **Web shell yang disamarkan jadi gambar itu persis kayak gitu** — labelnya (ekstensi/nama file) bilang "gambar", tapi isinya kode yang bisa dieksekusi.

### Threat #2: "Gambar" yang berbahaya
Nggak semua file yang keliatan kayak gambar itu konten aman.

**Contoh:** file SVG yang isinya JavaScript. Double extension: `avatar.php.jpg`. **Polyglot file** — valid di lebih dari satu format sekaligus. Metadata gambar yang dipakai bawa payload.

**Dampak keamanan:** Stored XSS pas ditampilin di browser. Eksploitasi parser di sisi server. Eksfiltrasi data atau pencurian session. Kerusakan reputasi dari konten yang di-hosting.

### Threat #3: Denial of Service dan Overwrite
Fitur upload bisa nyerang **availability** dan **integrity**.

**Kegagalan availability:** file gede banget ngabisin storage. Banyak file kecil ngabisin batas inode/direktori. Zip bomb yang ngembang tak terduga pas diproses.

**Kegagalan integrity:** penyerang pake ulang nama file: `invoice.pdf`. Aplikasi nimpa file yang udah ada. Trik path yang nyoba kabur dari folder upload.

**Validasi bukan cuma soal tipe. Dia juga ngelindungin kapasitas dan kepemilikan file.**

### Kasus nyata: Apache Struts Multipart Parser (Equifax, 2017)
Penyerang eksploitasi **CVE-2017-5638** di Apache Struts. Komponen yang rentan terkait pemrosesan request `multipart/file upload`. Breach-nya mengekspos data pribadi **147 juta orang**.

**Pelajaran:** keamanan upload nyakup kode aplikasi DAN framework di bawahnya. Bahkan kalau kode PHP-mu udah hati-hati, stack upload-mu mencakup web server, framework, library, folder sementara, dan izin storage.

### Prinsip keamanan dari Pro PHP Security
Pengembangan PHP yang aman memperlakukan input eksternal sebagai musuh sampai terbukti aman:
- Validasi input sebelum dipakai
- Batasi apa yang diterima aplikasi
- Hindari percaya data yang disediakan client
- Pakai least privilege buat file dan direktori
- Bangun defense in depth, bukan satu pengecekan doang

Buat file upload: **file itu sendiri, namanya, MIME type-nya, dan path storage-nya — semuanya input sensitif secara keamanan.**

### Checklist validasi berlapis
1. **Request** — method POST, error upload, konteks CSRF/session
2. **Size** — batas byte maksimum, kuota, tolak file kosong
3. **Type** — allowlist ekstensi + MIME + magic bytes/signature
4. **Name** — bikin nama file acak baru
5. **Storage** — di luar webroot atau direktori non-eksekusi
6. **Access** — otorisasi download lewat logika aplikasi

**Layer 1 — Request & error check:**
```php
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    exit('Invalid request');
}
if (!isset($_FILES['report']) || $_FILES['report']['error'] !== UPLOAD_ERR_OK) {
    exit('Upload failed');
}
```

**Layer 2 — Allowlist ekstensi** (allowlist ngalahin blocklist — blocklist ketinggalan ekstensi berbahaya baru, allowlist cocok sama kebutuhan bisnis, **tapi tetep belum cukup sendirian**, jangan pernah ngandelin asumsi double-extension):
```php
$allowedExt = ['pdf', 'png', 'jpg'];
$ext = strtolower(pathinfo($_FILES['report']['name'], PATHINFO_EXTENSION));
if (!in_array($ext, $allowedExt, true)) {
    exit('File type not allowed');
}
```

**Layer 3 — MIME type dan magic bytes** (jangan percaya `Content-Type` dari browser doang; ekstensi jawab "namanya ngaku apa?", cek MIME/signature jawab "isinya beneran keliatan kayak apa?"):
```php
$finfo = new finfo(FILEINFO_MIME_TYPE);
$mime = $finfo->file($_FILES['report']['tmp_name']);
$allowedMime = ['pdf' => 'application/pdf', 'png' => 'image/png', 'jpg' => 'image/jpeg'];
if (($allowedMime[$ext] ?? '') !== $mime) {
    exit('Invalid file content');
}
```

**Layer 4 — Size, filename, path:**
```php
$maxBytes = 2 * 1024 * 1024; // 2 MB
if ($_FILES['report']['size'] <= 0 || $_FILES['report']['size'] > $maxBytes) {
    exit('Invalid file size');
}
$newName = bin2hex(random_bytes(16)) . '.' . $ext;
$destination = __DIR__ . '/../storage/uploads/' . $newName;
```
Set batas ukuran berbasis kebutuhan bisnis, jangan pakai ulang nama asli, generate nama acak biar nggak tabrakan, dan bangun path-nya cuma di sisi server.

### Desain penyimpanan yang aman
| Nggak aman | Lebih aman |
| --- | --- |
| `/var/www/html/uploads/shell.php` | `/app/storage/uploads/93af...c2.pdf` |
| URL: `https://site.com/uploads/shell.php` | Dilayani cuma lewat `download.php` setelah otentikasi |

**Tujuannya: file yang diupload harusnya jadi DATA, bukan kode aplikasi yang bisa dieksekusi.**

### Kerangka upload PHP yang lebih aman
```php
<?php
$allowed = ['pdf'=>'application/pdf', 'png'=>'image/png', 'jpg'=>'image/jpeg'];
$maxBytes = 2 * 1024 * 1024;

if ($_SERVER['REQUEST_METHOD'] !== 'POST') exit('Invalid request');
if (!isset($_FILES['report']) || $_FILES['report']['error'] !== UPLOAD_ERR_OK) exit('Upload failed');
if ($_FILES['report']['size'] <= 0 || $_FILES['report']['size'] > $maxBytes) exit('Invalid size');

$ext = strtolower(pathinfo($_FILES['report']['name'], PATHINFO_EXTENSION));
$mime = (new finfo(FILEINFO_MIME_TYPE))->file($_FILES['report']['tmp_name']);
if (!isset($allowed[$ext]) || $allowed[$ext] !== $mime) exit('Invalid type');

$newName = bin2hex(random_bytes(16)) . '.' . $ext;
move_uploaded_file($_FILES['report']['tmp_name'], __DIR__ . '/../storage/uploads/' . $newName);
```

### Pola desain upload yang aman
**Accept** (cuma form dan konteks terotentikasi yang diharapkan) → **Validate** (ukuran, ekstensi, MIME/signature, aturan bisnis) → **Rename** (nama file acak, nama asli disimpan cuma sebagai metadata) → **Store** (di luar webroot atau storage non-eksekusi) → **Serve** (download lewat logika otorisasi).

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — kontennya semua teks dan blok kode. Slide 11 (Threat #1: Web Shell) khususnya nggak bisa dibaca sama sekali karena kena block antivirus — lihat catatan di bagian Threat #1 di atas. **Buka PPT aslinya di slide 11** kalau butuh contoh kode persisnya (buka dengan hati-hati, kode di situ kemungkinan contoh web shell).

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Remote Code Execution (RCE)** | Kondisi di mana penyerang bisa ngejalanin perintah/kode di server dari jarak jauh |
| **Content-Type / MIME type** | Label yang menandai jenis isi sebuah file (misal `image/png`); bisa dipalsukan browser/user |
| **`pathinfo()`** | Fungsi PHP buat ngambil bagian-bagian nama file, termasuk ekstensinya |
| **`random_bytes()` / `bin2hex()`** | Kombinasi fungsi PHP buat bikin string acak yang aman secara kriptografis, dipake buat nama file baru |
| **Inode** | Struktur data filesystem yang nyimpen metadata file; kehabisan inode = nggak bisa bikin file baru walau disk masih ada ruang |
| **Zip bomb** | File arsip kecil yang, kalau diekstrak, ngembang jadi ukuran raksasa dan nguras resource |
| **CVE** | Common Vulnerabilities and Exposures, ID standar buat kerentanan software yang udah dikenal publik |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan kode `upload.php` minimal di cerita Raka (slide 9) dengan kerangka upload aman (slide 22). Identifikasi MINIMAL TIGA perbedaan konkret, dan untuk tiap perbedaan, jelasin ancaman spesifik apa yang ditutup.
2. **(C4 – Analisis)** Kasus `shell.php.jpg` bisa dicegah di beberapa lapis checklist (Type: ekstensi, Type: MIME/magic bytes, atau Storage). Analisis: kalau developer CUMA nerapin allowlist ekstensi (Layer 2) tapi lupa Layer 3 (MIME check) dan Layer 5 (storage di luar webroot), skenario serangan apa yang MASIH mungkin lolos?
3. **(C5 – Evaluasi)** Sebuah tim bilang: "Kita udah aman karena kita cuma nerima file `.jpg`, `.png`, `.pdf` — nggak ada `.php` di allowlist kita." Evaluasi klaim ini pakai konsep Threat #2 (Dangerous Images) dan Threat #3 (Overwrite) — sebutkan minimal 2 cara serangan yang TETEP mungkin meskipun allowlist-nya cuma tiga ekstensi itu.
4. **(C5 – Evaluasi)** Bandingkan kasus Equifax 2017 (bug di framework/parser upload) dengan skenario Raka (bug di kode aplikasi sendiri). Menurutmu, kenapa checklist validasi 6-lapis di materi ini TETEP nggak cukup buat mencegah kasus seperti Equifax? Apa yang perlu ditambahin di luar kode aplikasi?
5. **(C6 – Cipta)** FitCampus mau nambah fitur baru: dosen bisa upload rubrik penilaian dalam format `.xlsx` atau `.docx` (bukan cuma PDF/gambar). Rancang penyesuaian checklist 6-lapis buat fitur ini — sebutkan minimal 1 penyesuaian konkret di tiap lapis (Request, Size, Type, Name, Storage, Access), dan jelasin kenapa `.xlsx`/`.docx` butuh perhatian ekstra dibanding `.pdf`/`.jpg`.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W02 - Session, Cookies & File Upload]]
- [[W03 - Web Form Processing and Security]]
- [[W07 - Review]]
- [[SecureProgramming - Review dan Glosari]]
