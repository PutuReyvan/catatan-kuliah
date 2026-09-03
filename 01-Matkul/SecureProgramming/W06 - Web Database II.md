---
matkul: Secure Programming
minggu: 6
sks: 3
sumber: S6_Web_Database_II_Secure_Programming.pptx
tags: [kuliah/secure-programming, minggu/w06]
status: draft
diproses: 2026-09-04
---

# W06 — Web Database (II): UPDATE & DELETE

## Ringkasan
> - Lanjutan cerita minggu lalu: **portal internship kampus** go-live. Mahasiswa bisa update profil dan hapus draft submission. **Satu query yang ceroboh bisa ngubah banyak baris, ngehapus record yang salah, atau bocorin data sensitif.**
> - **CRUD** (Create, Read, Update, Delete) — SELECT itu risiko **kerahasiaan**, INSERT risiko **integritas & spam**, UPDATE risiko **integritas & privilege**, DELETE risiko **ketersediaan & recovery**. Fokus minggu ini: **U dan D — operasi favorit penyerang kalau otorisasi lemah**.
> - `UPDATE` tanpa `WHERE` yang bener = satu request bisa ngubah BANYAK record. `DELETE` tanpa `WHERE` = **semua baris jadi target**.
> - **UPDATE yang aman jawab tiga pertanyaan: SIAPA, APA, dan KENAPA.** Kepemilikan baris (ownership) harus dicek di `WHERE`, bukan cuma dipercaya dari hidden field form.
> - **Soft delete** (tandain `deleted_at`, jangan beneran dihapus) lebih disukai buat kebanyakan sistem bisnis — bisa dipulihkan, bisa diaudit.
> - Delete itu **keputusan risiko, bukan cuma perintah database**. Jangan pernah delete lewat link GET biasa — pakai POST + CSRF protection + konfirmasi.
> - Setiap operasi TULIS butuh empat hal: **Validasi, Otorisasi, Parameterisasi, Logging.**

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| CRUD | Create, Read, Update, Delete — empat operasi dasar terhadap data |
| Ownership check | Verifikasi bahwa baris data yang diubah/dihapus beneran milik user yang minta |
| Soft delete | Menandai baris sebagai "dihapus" (misal `deleted_at`) tanpa beneran menghapusnya |
| Hard delete | Menghapus baris secara fisik dari tabel |
| Audit trail | Catatan siapa mengubah apa dan kapan, buat investigasi/akuntabilitas |
| Transaction (rollback) | Kelompok perubahan database yang dijalankan sebagai satu unit; kalau gagal, semuanya dibatalkan |
| CSRF | Serangan yang nipu user yang lagi login buat ngirim aksi yang nggak dia maksud |
| `rowCount()` | Fungsi PDO buat ngecek berapa baris yang kena dampak query |

## Isi

### Cerita: portal magang kampus go-live
Tim urusan kemahasiswaan ngeluncurin portal web sederhana buat submission magang. Mahasiswa bisa update data profil dan ngehapus draft submission lama. Tim database bilang tabelnya udah siap — tapi kode web-nya ditulis buru-buru. **Satu query yang ceroboh bisa ngubah banyak baris, ngehapus record yang salah, atau bocorin data sensitif.** Fokus minggu ini: dari "gimana cara nge-query" ke "gimana cara melindungi perubahan database".

### Review jembatan dari kelas database
Yang udah kamu tau: tabel nyimpen data terstruktur — user, submission, log, role. Baris = record; kolom = atribut. Primary key ngenalin satu baris; foreign key nyambungin baris antar tabel. **Klausa `WHERE` nentuin baris mana yang kena dampak.** Di web app, input user sering jadi bagian dari operasi database.

**Model mentalnya:** Browser (form data/route) → PHP Controller (baca input) → PDO Layer (siapin query) → DBMS (eksekusi dengan aman). Aplikasinya nerima input HTTP, terus mutusin mau nge-query database atau enggak. PDO itu batas antara kode PHP dan eksekusi SQL. Prepared statement misahin struktur SQL dari nilai yang disupply user.

### Sebelum nulis UPDATE atau DELETE: pertanyaan keamanan koneksi
Akun database mana yang dipake aplikasi? Apa akun itu cuma punya izin yang dia butuhin? Bisa nggak error-nya bocorin nama database, nama tabel, atau stack trace? Kredensial disimpen di kode, config, atau environment variable? Koneksinya nerapin charset dan exception handling yang bener? **Prinsip least privilege — web app nggak boleh nyambung sebagai database root.**

> [!info] Analogi
> Query yang aman tetep bisa jadi berbahaya kalau **koneksinya punya hak akses kebanyakan**. Bayangin karyawan baru yang cuma butuh akses ke laci kasirnya sendiri, tapi malah dikasih kunci master ke semua brankas kantor "biar praktis". Kalau kartu akses karyawan itu dicuri (analog: query-nya kena SQL injection), penyerang otomatis dapet akses ke SEMUA brankas, bukan cuma satu laci.

### CRUD lagi: kenapa UPDATE dan DELETE beda rasa
- **C**reate (INSERT) — bikin data, risiko integritas dan spam
- **R**ead (SELECT) — baca data, risiko kerahasiaan
- **U**pdate — ubah data yang ada, risiko integritas dan privilege
- **D**elete — hapus data, risiko ketersediaan dan recovery

**Kesalahan penanganan input yang SAMA bisa ngasih dampak bisnis yang SANGAT beda.** Hari ini fokus ke **U dan D — operasi yang paling disukai penyerang kalau otorisasi-nya lemah.**

### UPDATE Query: konsep dulu
```sql
UPDATE table_name
SET column_1 = value_1, column_2 = value_2
WHERE condition;
```
`UPDATE` ngubah baris yang ada. `SET` nentuin nilai barunya. `WHERE` nentuin baris mana yang kena dampak. **Tanpa `WHERE` yang bener, satu request bisa ngubah banyak record.** Dalam secure programming, UPDATE harus dikontrol lewat validasi DAN otorisasi.

**Ide intinya: UPDATE bukan penggantian teks. Itu perubahan terkontrol atas data bisnis.**

### UPDATE di PHP PDO — pola aman
```php
$sql = "UPDATE students SET phone = :phone WHERE id = :id";
$stmt = $pdo->prepare($sql);
$stmt->execute([':phone' => $phone, ':id' => $studentId]);
```
Perintah SQL ditulis duluan. Nilai yang dikontrol user dikirim sebagai parameter. Placeholder `:phone` itu data, bukan kode SQL. Placeholder `:id` membatasi record mana yang berubah. **Cek `rowCount()` cuma buat logika bisnis, bukan sebagai satu-satunya kontrol keamanan.**

**Skenario: update profil mahasiswa.** Mahasiswa buka Edit Profile → Form ngirim phone + address → Server validasi format + kepemilikan → PDO eksekusi UPDATE prepared → Database ngubah satu baris.

Empat checkpoint keamanan: (1) Apa input-nya keliatan valid? (2) Apa user yang login diizinin ngubah baris ini? (3) Apa query-nya di-parameterize? (4) Apa perubahannya di-log kalau nyentuh data sensitif?

### Ancaman: UPDATE yang jelek — waktu input jadi SQL
```php
// Pola berbahaya: string concatenation — JANGAN DIPAKAI
$sql = "UPDATE students SET phone = '$phone' WHERE id = $id";
$pdo->query($sql);
```
Aplikasi nyampur kode SQL dan input user. Penyerang bisa ngubah logika query yang dimaksud. `id` yang dimanipulasi bisa ngubah baris user lain. Kondisi `WHERE` yang hilang atau rusak bisa ngubah banyak record. **Fix-nya bukan "escape di mana-mana"; pakai parameterisasi plus validasi.**

**Pertanyaan keamanannya: bisa nggak ada bagian dari request user yang ngubah STRUKTUR atau TARGET dari statement SQL ini?**

### Validasi sebelum UPDATE
Validasi tipe: integer ID, format tanggal, format email, nilai enum. Validasi rentang: skor 0–100, umur wajar, batas ukuran file kalau relevan. Validasi aturan bisnis: form yang udah disubmit nggak bisa diedit lagi setelah approved. **Tolak field yang nggak dikenal**, alih-alih ngupdate mentah-mentah semua kunci yang disubmit. Normalisasi nilai sebelum disimpen kalau perlu.

```php
$id = filter_input(INPUT_POST, 'id', FILTER_VALIDATE_INT);
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);

$allowedStatus = ['draft', 'submitted'];
if (!in_array($status, $allowedStatus, true)) {
  throw new InvalidArgumentException('Invalid status');
}
```

### Otorisasi sebelum UPDATE: siapa punya baris ini?
```php
$sql = "UPDATE submissions
        SET title = :title
        WHERE id = :id AND student_id = :student_id";

$stmt = $pdo->prepare($sql);
$stmt->execute([
  ':title' => $title,
  ':id' => $submissionId,
  ':student_id' => $_SESSION['student_id']
]);
```
Jangan percaya hidden field form sebagai bukti kepemilikan. Pakai identitas **session yang udah terautentikasi** di sisi server. Taruh constraint kepemilikan di klausa `WHERE`. Pisahin pengecekan berbasis role dari pengecekan kepemilikan per-baris. **UPDATE yang aman jawab: siapa, apa, dan kenapa.**

### Transaction dan audit trail
```php
$pdo->beginTransaction();
try {
  // 1) Update status aplikasi
  // 2) Insert audit log
  $pdo->commit();
} catch (Throwable $e) {
  $pdo->rollBack();
  throw $e;
}
```
Pakai transaction kalau beberapa perubahan database harus berhasil bareng-bareng. Jaga audit trail buat update yang sensitif: siapa mengubah apa dan kapan. Hindari nyimpen nilai sensitif langsung di log. Jangan tampilin detail exception mentah ke user. **Nilai keamanannya: investigasi, pemulihan, dan akuntabilitas yang lebih baik.**

### DELETE Query: konsep dulu
```sql
DELETE FROM table_name WHERE condition;
```
`DELETE` ngehapus baris dari tabel. `WHERE` nentuin baris mana yang dihapus. **Tanpa `WHERE`, perintahnya bisa ngehapus SEMUA baris.** Aksi delete sering ngefek ke availability dan kelangsungan bisnis. Banyak sistem lebih milih **soft delete** buat kemudahan recovery.

**Pergeseran mindset: DELETE itu keputusan risiko, bukan cuma perintah database.**

### DELETE yang aman dengan PDO
```php
$sql = "DELETE FROM submissions
        WHERE id = :id
          AND student_id = :student_id
          AND status = 'draft'";

$stmt = $pdo->prepare($sql);
$stmt->execute([':id' => $submissionId, ':student_id' => $_SESSION['student_id']]);
```
Parameterize nilai identifier-nya. Batasi berdasarkan pemilik yang terautentikasi. Batasi berdasarkan state bisnis: **cuma draft yang boleh dihapus**. Konfirmasi niat user di UI. Log operasi delete yang berdampak tinggi.

### Soft Delete vs Hard Delete
| Hard Delete | Soft Delete |
| --- | --- |
| Baris beneran dihapus | Baris ditandai "dihapus" |
| Lebih susah dipulihin | Bisa dipulihkan atau diaudit |
| State tabel lebih simpel | Butuh filter tambahan di SELECT |

```sql
-- CONTOH SOFT DELETE
UPDATE submissions
SET deleted_at = NOW(), deleted_by = :user_id
WHERE id = :id AND student_id = :user_id;
```

### Ancaman: DELETE yang jelek — bencana `WHERE` yang hilang
```php
// Berbahaya
$sql = "DELETE FROM submissions";
$pdo->exec($sql);

// Juga berbahaya: id yang nggak dipercaya digabung langsung
$sql = "DELETE FROM submissions WHERE id = $id";
```
Nggak ada `WHERE` berarti **tiap baris jadi target**. ID yang nggak dipercaya bisa ngubah target penghapusan. Link delete langsung lewat GET bisa ketriger nggak sengaja. Endpoint delete tanpa proteksi CSRF bisa disalahgunakan dari situs lain. Fitur delete tanpa logging bikin investigasi susah.

**Diskusi kelas: apa dampak bisnisnya kalau semua submission magang tiba-tiba raib?**

### Keamanan UI dan HTTP method buat Delete
Jangan lakuin aksi delete pakai link GET biasa. Pakai semantik POST/DELETE dan proteksi CSRF sisi server. Minta konfirmasi buat aksi yang destruktif. Tunjukin item yang PERSIS bakal dihapus sebelum disubmit. Balikin pesan aman ke user; simpen error teknis di log server.

```html
<form method="POST" action="/submission/delete.php">
  <input type="hidden" name="csrf" value="...">
  <input type="hidden" name="id" value="123">
  <button type="submit">Delete Draft</button>
</form>
```

### Threat model buat UPDATE dan DELETE
Manipulasi input: ngubah `id`, role, status, atau hidden field. SQL injection: ngubah intent query pas kode nyambungin input. **Broken access control**: ngupdate atau ngehapus record user lain. **CSRF**: nipu user yang lagi login buat submit perubahan. Privilege DB berlebihan: aplikasinya bisa ngubah lebih banyak tabel dari yang dibutuhin.

**Tiap operasi TULIS butuh:** 1) Validasi, 2) Otorisasi, 3) Parameterisasi, 4) Logging.

### Kasus: TalkTalk SQL Injection (lagi, dari sudut UPDATE/DELETE)
Tahun 2015, TalkTalk kena insiden siber besar lewat SQL injection di halaman web lawas. Regulator UK (ICO) bilang SQL injection itu serangan yang **udah lama dipahami dengan pertahanan yang udah dikenal**. Serangan SQL injection sebelumnya udah pernah kejadian, tapi nggak ditindaklanjuti dengan bener. **Pelajaran buat topik ini: penanganan query yang nggak aman bukan "masalah kecil" — itu bisa jadi insiden bisnis.** Dari satu halaman rentan, ke paparan data pelanggan, ke dampak regulasi. **Prinsip pertahanan yang sama berlaku buat SELECT, UPDATE, dan DELETE: jangan pernah biarin input jadi struktur SQL.**

## Diagram & Visual
- **Slide 5, 7, 13 — bridge dari database class, mental model request-ke-query, dan story scene update profil.**
  ![[99-Assets/SecureProgramming/W06-slide05.png]]
  ![[99-Assets/SecureProgramming/W06-slide07.png]]

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **`beginTransaction()` / `commit()` / `rollBack()`** | Trio fungsi PDO buat ngejalanin beberapa query sebagai satu unit yang bisa dibatalkan kalau ada yang gagal |
| **`Throwable`** | Tipe dasar di PHP yang menangkap semua jenis error dan exception |
| **DELETE/PUT semantics** | Prinsip HTTP: aksi yang mengubah/menghapus data seharusnya dikirim lewat method yang bukan GET |
| **CSRF token** | Nilai rahasia unik per sesi/form yang membuktikan request beneran datang dari form aplikasi, bukan dari situs lain |
| **`deleted_at` / `deleted_by`** | Pola kolom umum buat implementasi soft delete: kapan dan siapa yang "menghapus" |
| **Referential integrity** | Prinsip database bahwa data yang saling terhubung (lewat foreign key) harus tetap konsisten |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan dua kode delete di slide 21: `DELETE FROM submissions` (tanpa WHERE) dan `DELETE FROM submissions WHERE id = $id` (dengan concatenation). Analisis: mana yang risikonya LEBIH BESAR buat sebuah portal kampus dengan ribuan submission, dan kenapa — pertimbangkan skala dampak vs. kemudahan dieksploitasi.
2. **(C4 – Analisis)** Pola "UPDATE yang aman jawab siapa, apa, dan kenapa" (slide 16) dipraktikkan lewat `WHERE id = :id AND student_id = :student_id`. Analisis: kalau baris `AND student_id = :student_id` DIHAPUS dari query itu, tapi prepared statement-nya tetep dipakai (jadi nggak ada SQL injection) — kerentanan APA yang tetep ada, dan termasuk kategori threat model yang mana?
3. **(C5 – Evaluasi)** Sebuah tim developer bilang: "Kita udah pake prepared statement buat semua UPDATE dan DELETE, jadi kita udah aman dari OWASP-level risk." Evaluasi klaim ini pakai empat pilar dari slide 23 (Validasi, Otorisasi, Parameterisasi, Logging) — pilar mana yang TIDAK terpenuhi cuma dengan pakai prepared statement, dan apa akibat konkretnya kalau diabaikan?
4. **(C5 – Evaluasi)** Nilai trade-off antara soft delete dan hard delete buat KONTEKS submission magang mahasiswa (bukan konteks umum). Faktor apa dari cerita minggu ini (audit trail, business impact, recovery) yang bikin soft delete lebih masuk akal di sini, dan adakah skenario di portal ini di mana hard delete tetap lebih tepat?
5. **(C6 – Cipta)** Rancang alur DELETE buat fitur baru: staf admin (bukan mahasiswa) bisa ngehapus PERMANEN submission yang statusnya udah "rejected" lebih dari 1 tahun (data cleanup). Jelasin bagaimana keempat pilar (Validasi, Otorisasi, Parameterisasi, Logging) diterapkan berbeda di sini dibanding alur delete draft mahasiswa biasa — sebutkan minimal satu perbedaan konkret di tiap pilar.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W05 - Web Database I]]
- [[W07 - Review]]
- [[SecureProgramming - Review dan Glosari]]
