---
matkul: Secure Programming
sks: 3
dosen: Ika Dyah A.R., S.Kom., M.Kom.
jadwal: belum dikonfirmasi
tags: [kuliah/secure-programming, moc]
status: draft
diproses: 2026-09-04
---

# Secure Programming

Index matkul. Tiga belas deck PPT dari dosen, diproses jadi tiga belas note pertemuan.

**Format note matkul ini beda dari Blockchain/Forensics** atas permintaan eksplisit: bahasa di bagian Isi ditulis santai/gampang dimengerti, banyak analogi, dan tiap note ditutup dengan section **Istilah Khusus** (glosari lokal) dan **Quiz Pemahaman** (soal Bloom taksonomi level C4 ke atas — Analisis, Evaluasi, Cipta — bukan soal hafalan).

**Link referensi penting yang dicatat dosen:** [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/index.html) — kumpulan panduan praktis per topik keamanan (XSS, SQL Injection, Access Control, dll), dirujuk di hampir semua deck W03–W13.

## Daftar Pertemuan

| # | Note | Isinya |
| --- | --- | --- |
| W01 | [[W01 - Introduction to Web Application]] | HTTP/HTTPS, statelessness, session cookie, aturan emas PHP ("jangan percaya input"), open redirect, attack surface publishing |
| W02 | [[W02 - Session, Cookies & File Upload]] | Session vs cookie (analogi penitipan mantel), flag HttpOnly/Secure/SameSite, session hijacking (Firesheep), array superglobal, dasar file upload |
| W03 | [[W03 - Web Form Processing and Security]] | Trust boundary form, allowlist vs blocklist, 5 pertanyaan validasi, SQL injection dasar, validate/sanitize/encode, encoding per konteks |
| W04 | [[W04 - Web Form File Upload]] | Threat model upload (execute/store/trigger/exhaust/overwrite), 6 lapis validasi, kasus Equifax/Struts CVE-2017-5638 |
| W05 | [[W05 - Web Database I]] | Koneksi PDO aman, SELECT tidak aman vs prepared statement, INSERT + password_hash, kasus TalkTalk 2015 |
| W06 | [[W06 - Web Database II]] | CRUD dan risiko masing-masing, UPDATE dengan ownership check, DELETE + soft delete, CSRF pada aksi destruktif |
| W07 | [[W07 - Review]] | Konsolidasi W01–W06 lewat cerita K-Clinic, 6 anti-pattern audit, alur kerja aman, kasus MOVEit CVE-2023-34362 |
| W08 | [[W08 - Code Auditing OWASP I]] | A01 Broken Access Control (IDOR, forced browsing) + A05 Security Misconfiguration, cerita StudyHub |
| W09 | [[W09 - Code Auditing OWASP II]] | A02 Cryptographic Failures (hashing/encryption/salting) + A06 Vulnerable Components, kasus Log4Shell & RockYou |
| W10 | [[W10 - Code Auditing OWASP III]] | A03 Injection (SQLi + XSS, worm Samy) + A04 Insecure Design (use-case vs misuse-case) |
| W11 | [[W11 - Code Auditing OWASP IV]] | A07 Authentication Failures (MFA) + A08 Software/Data Integrity, kasus Colonial Pipeline & SolarWinds |
| W12 | [[W12 - Code Auditing OWASP V]] | A09 Logging & Alerting Failures + Mishandling Exceptional Conditions, log injection, fail closed |
| W13 | [[W13 - Review II]] | Konsolidasi seluruh OWASP Top 10 lewat cerita BrewNote, 4 kantong risiko, rantai serangan, jembatan Database↔Web Security |

## Rangkuman Ujian
- [[SecureProgramming - Review dan Glosari]] — review lintas pertemuan + glosari lengkap

## Peta Materi

Matkul ini punya struktur naratif yang sangat rapi — tiap deck pakai satu tokoh fiksi (Maya, Raka, Rafa) dan satu app fiktif buat ngebungkus materinya. Tiga blok besar:

**Blok 1 — Membangun aplikasi web dengan aman (W01–W06): "gimana caranya nulis fitur yang nggak gampang jebol"**
W01 kasih fondasi (HTTP/HTTPS/PHP/publishing). W02–W04 ngurusin identitas dan input user (session, cookie, form, file upload). W05–W06 ngurusin database (SELECT/INSERT lalu UPDATE/DELETE). Blok ini defensif dari sisi DEVELOPER — "gimana caranya gue nulis kode yang bener dari awal."

**Blok 2 — Review (W07): titik tengah**
Satu sesi penuh yang nyambungin ulang W01–W06 jadi satu alur kerja tunggal, plus nambahin kasus baru (MOVEit). Ini jembatan sebelum ganti sudut pandang di blok berikutnya.

**Blok 3 — Code Auditing berbasis OWASP Top 10 (W08–W13): "gimana caranya nemuin yang udah kepalang jebol"**
Sudut pandangnya berubah dari DEVELOPER ke AUDITOR — bukan lagi "gimana nulis kode aman", tapi "gimana meriksa kode ORANG LAIN dan nemuin cacatnya". W08–W12 masing-masing ngambil dua kategori OWASP (A01+A05, A02+A06, A03+A04, A07+A08, A09+exception handling). W13 nutup dengan sepuluh-duanya sekaligus lewat satu cerita BrewNote.

**Benang merah yang menembus semua blok:**

- **"Jangan pernah percaya input dari client"** — aturan emas dari W01, muncul lagi sebagai alasan validasi di W03, sebagai alasan prepared statement di W05, dan jadi tema besar A03 (Injection) di W10.
- **Trust boundary** — diperkenalkan di W03 (form), dipakai lagi persis di W10 buat ngejelasin SQLi *dan* XSS sebagai fenomena yang sama (interpreter beda).
- **Deny by default / least privilege** — muncul di koneksi database (W05, W06), lalu jadi salah satu dari tiga prinsip access control di W08 (A01).
- **Cerita sebagai metafora audit:** tiap "Case Study" ngajarin cara mikir yang sama — Concept at fault → What enabled it → The control (pertama kali dipakai eksplisit di W01, dipakai konsisten sampai W13).
- **`password_hash()`/`password_verify()`** — diajarin sebagai teknik konkret di W05, lalu jadi bagian checklist audit A02 di W09 dan A07 di W11.
- **Jembatan Database↔Web Security** yang eksplisit disebut berkali-kali: prepared statement = defense SQLi, `GRANT`/least privilege = deny-by-default, `CHECKSUM` = integrity check (A08), transaction log = security log (A09). W13 ngumpulin semua jembatan ini jadi satu tabel penutup.

## Konsep Utama
Belum ada note atomik di `02-Konsep/`. Kandidat terkuat kalau nanti mau dibuat (konsep yang muncul di lebih dari satu pertemuan):

- **Trust boundary** — W01, W03, W07, W10
- **Prepared statement / parameterized query** — W03, W05, W06, W07, W10, W13
- **Deny by default / least privilege** — W02, W05, W06, W08, W13
- **Validate → Authenticate → Authorize → Parameterize → Store → Encode** (secure workflow) — W03, W07, W13
- **Hashing vs Encryption vs Salting** — W05, W09
- **Concept at fault → What enabled it → The control** (pola analisis kasus) — W01, dipakai di semua deck berikutnya

## Catatan Pemrosesan

**Sumber PPT sangat rapi dan konsisten** — beda dari matkul Forensics sebelumnya, deck-deck ini nggak punya banyak lubang materi antara learning outcome dan isi slide. Tiap deck W03 dst. secara eksplisit ngutip **Myer & Southwell, *Pro PHP Security*** sebagai referensi utama, plus mulai W10 nambah **Mann, *Security Principles for PHP Applications***.

**Penomoran sesi.** Nama file dari dosen (S1–S13, di mana file "Introduction to Web Application" tanpa nomor diperlakukan sebagai S1) dipakai sebagai urutan utama W01–W13, karena urutan ini logis secara materi (fondasi → form/database → review → audit OWASP → review akhir). **Tapi ada tiga inkonsistensi penomoran DI DALAM slide sendiri** (bukan kesalahan pemrosesan vault):
- File **S2** ("Session Cookies FileUpload") dan file **S3** ("Web_Form_Processing_and_Security") **DUA-DUANYA** nulis "SESSION 2" di slide judulnya.
- File **S8** ("OWASP_Code_Auditing_I") nulis "Session 10" di slide judulnya — bentrok sama file **S10** yang juga nulis "Session 10" (dan itu emang benar S10).

Ini kemungkinan salah ketik dosen pas nyalin template slide. Nama file dan urutan topik tetap dipakai sebagai acuan W01–W13 karena lebih konsisten dan logis.

**Duplikat.** Nggak ada file identik (dicek lewat md5) di antara 13 PPT yang diupload.

**Slide yang nggak bisa diekstrak.** **W04 slide 11** ("Threat #1: Web Shell") gagal diekstrak sama sekali — file teks hasil ekstraksi diblokir antivirus (kemungkinan besar karena isinya contoh kode web shell PHP, yang emang materi topik minggu itu sendiri). Isi slide itu **direkonstruksi dari konteks** slide sebelum-sesudahnya di note W04, ditandai jelas sebagai rekonstruksi bukan kutipan. **Buka PPT aslinya di slide 11 kalau butuh kode persisnya** (hati-hati, isinya contoh web shell).

**Gambar.** Cuma tiga deck (W05, W06, W07) yang punya gambar non-dekoratif yang ke-ekstrak — 18 total, semuanya screenshot/diagram pendukung kode yang udah lengkap dijelasin di teks. Sepuluh deck lainnya (termasuk semua deck W08–W13 OWASP series) nggak punya gambar sama sekali — semua kontennya teks dan blok kode asli, jadi nggak ada risiko kehilangan materi dari gambar yang nggak ke-ekstrak.

**Kredit SKS matkul ini belum dikonfirmasi** — ditulis `3` di frontmatter sebagai asumsi (konten 13 sesi, kode-berat, cocok pola matkul praktikum), tapi ini administratif doang, bukan dari isi slide. **Cek dan koreksi manual kalau salah.**
