---
matkul: Secure Programming
sks: 3
sumber: rangkuman dari W01-W13
tags: [kuliah/secure-programming, ujian]
status: draft
diproses: 2026-09-04
---

# Secure Programming — Review dan Glosari

Rangkuman lintas pertemuan W01–W13. Dibaca sebelum ujian, bukan pengganti note pertemuan — tiap note W01–W13 udah punya **Istilah Khusus** dan **Quiz Pemahaman** sendiri, jadi file ini fokus ke hal yang cuma keliatan kalau kamu ngeliat SEMUA minggu sekaligus.

---

## Bagian 1 — Review Cepat

### Cheat sheet: OWASP Top 10 (2021) satu halaman

Ini yang paling mungkin keluar di ujian — tabel gabungan dari W08–W13.

| # | Nama | Contoh kasus di cerita | Kasus nyata | Defense utama |
| --- | --- | --- | --- | --- |
| **A01** | Broken Access Control | `?student_id=1042→1043` (StudyHub) | — | Deny by default, cek ownership di server |
| **A02** | Cryptographic Failures | MD5 password, API key hardcode (BrewNote) | RockYou 2009/2021 | Argon2id/bcrypt + salt, TLS, AES-GCM |
| **A03** | Injection (SQLi + XSS) | `' OR '1'='1`, notice board XSS (CampusConnect) | Sony 2011, TalkTalk 2015 | Prepared statement, context-aware encoding |
| **A04** | Insecure Design | Reset password pakai security question (CampusConnect) | — | Threat modeling, misuse-case |
| **A05** | Security Misconfiguration | `display_errors=On`, `phpinfo()` publik (StudyHub) | — | Hardened baseline, matiin debug di prod |
| **A06** | Vulnerable & Outdated Components | `phpmailer 5.2.14`, library upload 3 tahun (BrewNote) | Log4Shell 2021, Equifax 2017 | SBOM, SCA scanning, patch terjadwal |
| **A07** | Identification & Auth Failures | Login tanpa throttle, plaintext compare (BiteClub) | Colonial Pipeline 2021 | MFA, rate-limit, `password_verify()` |
| **A08** | Software & Data Integrity Failures | Update HTTP tanpa signature, `unserialize()` cookie (BiteClub) | SolarWinds 2020 | Verifikasi signature, HMAC, CI/CD aman |
| **A09** | Logging & Alerting Failures | Login gagal nggak ke-log, catch block kosong (FinPay) | Rencana kesehatan anak (OWASP scenario), maskapai | Structured logging, alert, fail closed |
| **A10** | Server-Side Request Forgery | (disinggung W13 doang, nggak ada deck sendiri) | — | Validasi/allow-list URL keluar |

> Catatan: **A10 SSRF nggak punya deck sendiri** di matkul ini — cuma disinggung sekilas di W13 sebagai bagian dari sepuluh risiko. Kalau ujian nanya detail A10, materinya belum diajarin secara dalam — tanyakan ke dosen.

### Empat kantong OWASP (cara ngafal yang dianjurkan W13)
Jangan hafalin sepuluh nomornya satu-satu — hafalin EMPAT PERTANYAAN ini:

| Pertanyaan | Risiko |
| --- | --- |
| **Siapa boleh ngapain?** | A01, A07 |
| **Bisa dipercaya nggak input & datanya?** | A03, A02, A10 |
| **Dibangun & disetup dengan aman?** | A04, A05, A06 |
| **Bisa diverifikasi & dideteksi?** | A08, A09 |

### Pola "Concept at fault → What enabled it → The control"
Ini template analisis kasus yang dipake KONSISTEN dari W01 sampai W13. Kalau soal ujian nanya "analisis kasus X", jawab pake tiga kotak ini:
1. **Concept at fault** — konsep/fitur mana yang jadi titik lemah?
2. **What enabled it** — apa yang bikin titik lemah itu BISA dieksploitasi?
3. **The control** — perbaikan konkret apa yang nutup celahnya?

### Rantai serangan BrewNote (W13) — contoh "kenapa satu bug jarang cukup"
```
A05 (misconfig bocorin schema)
  → A03 (injection nguras data pake schema yang udah diketahui)
    → A02 (kripto lemah/md5 buka password hasil dump)
      → A07 (credential stuffing pake password bocor)
        → A01 (akses admin tercapai, data staf & customer lain terekspos)
```
**Pelajaran:** breach beneran biasanya RANTAI beberapa kelemahan kecil, bukan satu bug "critical" tunggal.

### Lima prinsip yang ngalahin hafalan (W13)
1. **Never trust input** (A03, A10)
2. **Deny by default** (A01, A05)
3. **Defence in depth** (rantai secara keseluruhan)
4. **Assume breach** (A09)
5. **Verify, don't trust** (A02, A08)

### Jembatan Database → Web Security (tabel W13, paling sering dijadiin soal)
| Dari kelas DATABASE | Sama dengan di WEB SECURITY | Nyetop risiko |
| --- | --- | --- |
| Prepared statements | Pertahanan SQL Injection | A03 |
| `WHERE owner_id = …` | Otorisasi/ownership check sisi-server | A01 |
| `GRANT`/role & least privilege | Access control deny-by-default | A01 / A05 |
| Enkripsi at-rest & TLS | Perlindungan kriptografis data | A02 |
| Transaction & audit log | Security logging & monitoring | A09 |
| `CHECKSUM` pada backup | Verifikasi integritas kode/update | A08 |
| `ROLLBACK` transaksi | Exception handling / fail closed | A09 (exception mishandling) |

### Tiga kata yang paling sering ketuker (kumpulan dari beberapa minggu)

**W03 — Validate / Sanitize / Encode:**
- Validate = tolak yang nggak sesuai aturan
- Sanitize = hapus/ubah bagian yang nggak diinginkan
- Encode = bikin aman DI TUJUANNYA (HTML/JS/SQL/URL beda-beda caranya)

**W09 — Hashing / Encryption / Salting:**
- Hashing = satu arah, buat password (bcrypt/Argon2id)
- Encryption = dua arah, bisa dibalikin pakai key (AES-GCM)
- Salting = nilai acak per-user sebelum hashing

**W11 — Identification / Authentication / Session management:**
- Identification = klaim ("aku user X")
- Authentication = bukti (password/OTP/key)
- Session management = tetap terbukti (session ID tiap request)

### Cerita dan tokoh per minggu (buat nginget "minggu ini bahas apa")
| Minggu | App fiktif | Tokoh | Fokus |
| --- | --- | --- | --- |
| W01 | (web app Maya) | Maya | HTTP/HTTPS/PHP/publish |
| W02 | (bank Maya) | Maya | Session & cookie |
| W03 | Kafe (feedback form) | — | Form validation |
| W04 | FitCampus | Raka | File upload |
| W05 | Campus Event Portal | — | SELECT & INSERT |
| W06 | Portal magang kampus | — | UPDATE & DELETE |
| W07 | K-Clinic | — | Review W01–W06 + MOVEit |
| W08 | StudyHub | Maya | A01 & A05 |
| W09 | BrewNote | Maya | A02 & A06 |
| W10 | CampusConnect | Maya | A03 & A04 |
| W11 | BiteClub | Rafa | A07 & A08 |
| W12 | FinPay | Maya | A09 & exception handling |
| W13 | BrewNote (lagi) | — | Semua OWASP Top 10 |

### Angka-angka yang layak dihafal
```
A01 Broken Access Control  : #1 peringkat, 94% app kena, 318rb+ kejadian
A05 Misconfiguration       : #5 peringkat, 90% app kena, 208rb+ kejadian
A03 Injection               : 94% app kena, 274rb+ kejadian
Log4Shell CVSS score        : 10.0 (maksimum)
Mean-time-to-detect breach  : ~200 hari (industri)
FinPay-style breach         : bisa nggak ketauan 7+ tahun
Equifax 2017                : 147 juta orang, $700 juta+ settlement
TalkTalk 2015                : ~156.959 pelanggan, denda £400.000
SolarWinds 2020              : 18.000+ organisasi ke-distribusi update jahat
Colonial Pipeline 2021       : 1 akun VPN tanpa MFA
MOVEit 2023 (CVE-2023-34362) : SQLi tanpa autentikasi
Samy worm 2005 (XSS)         : 1 juta+ akun MySpace dalam ~20 jam
RockYou 2009 → 2021          : 32 juta → 8,4 miliar password bocor
```

---

## Bagian 2 — Glosari Gabungan

Istilah lengkap ada di section **Istilah Khusus** tiap note W01–W13. Ini cuma istilah yang PALING SERING nongol lintas minggu, buat referensi cepat.

| Istilah | Arti singkat | Muncul di |
| --- | --- | --- |
| **Trust boundary** | Garis pemisah data yang dikontrol sistem vs. dikontrol user | W03, W07, W10 |
| **Prepared statement / placeholder** | Query SQL yang strukturnya dipisah dari datanya | W03, W05, W06, W07, W10, W13 |
| **Allowlist vs Blocklist** | Daftar "yang BOLEH" (kuat) vs daftar "yang DILARANG" (gampang dielakkan) | W03, W04 |
| **Least privilege** | Akun/role cuma dikasih hak yang beneran dibutuhin | W02, W05, W06, W08, W13 |
| **Deny by default** | Tutup semua akses dulu, baru buka yang eksplisit diizinin | W08, W13 |
| **IDOR** | Insecure Direct Object Reference; ngubah ID di URL buat akses record orang lain | W08, W13 |
| **XSS (stored/reflected/DOM)** | Injection ke browser; tiga jenis beda sumber payload-nya | W02, W03, W07, W10, W13 |
| **SQL Injection** | Injection ke database lewat input yang nggak divalidasi/di-parameterize | W03, W05, W06, W07, W10, W13 |
| **`password_hash()` / `password_verify()`** | Fungsi PHP standar buat hash & verifikasi password dengan aman | W05, W09, W11 |
| **CSRF** | Nipu user yang lagi login buat ngirim aksi yang nggak dia maksud | W02, W06, W07 |
| **CVE / NVD** | ID dan database standar buat kerentanan software yang udah dikenal publik | W04, W07, W09, W11, W13 |
| **CWE** | Katalog kategori jenis kerentanan (misal CWE-89 = SQL Injection) | W08–W13 |
| **Fail closed** | Kalau ada error, TOLAK aksinya secara default | W12 |
| **MFA (Multi-Factor Authentication)** | Kombinasi ≥2 faktor beda kategori (know/have/are) | W11 |
| **SBOM / SCA** | Daftar komponen software / tool scan kerentanan dependency | W09, W11 |
| **Insecure design vs implementation bug** | Kontrol yang nggak pernah dirancang vs kontrol yang ada tapi salah kode | W10 |
| **Threat modeling / misuse-case** | Teknik nanya "gimana ini bisa disalahgunakan?" sebelum/selagi desain | W10 |
| **Log injection** | Nyelipin newline ke input yang di-log buat memalsukan baris log | W12 |
| **Soft delete** | Nandain baris "dihapus" (`deleted_at`) tanpa beneran ngehapus | W06 |
| **Anti-pattern (audit)** | Pola kode yang kelihatan jalan tapi nyimpen risiko dikenal | W07 |

---

## Bagian 3 — Yang Perlu Dicek Sendiri

1. **A10 Server-Side Request Forgery** cuma disinggung sekilas di W13, nggak punya deck/sesi sendiri kayak sembilan risiko lainnya. Kalau ujian nanya detail A10 (kayak yang lain dapet "Concept → Threat → Real case → Prevention"), materinya belum ada di slide manapun.
2. **W04 slide 11** ("Threat #1: Web Shell") nggak bisa diekstrak sama sekali dari PPT-nya (diblokir antivirus, kemungkinan besar karena isinya contoh literal kode web shell). Isi di note W04 itu **rekonstruksi dari konteks**, bukan kutipan persis. Buka PPT aslinya kalau butuh kode persisnya.
3. **Dua inkonsistensi penomoran sesi** di dalam slide dosen sendiri (bukan salah proses vault): file S2 dan S3 SAMA-SAMA nulis "SESSION 2" di judul; file S8 nulis "Session 10" (bentrok sama file S10 yang beneran Session 10). Urutan W01–W13 di vault ini dipakai berdasarkan NAMA FILE dan urutan topik, bukan label session di dalam slide.
4. **SKS matkul belum dikonfirmasi** — ditulis `3` di frontmatter sebagai asumsi administratif, cek dan koreksi manual.
5. **Jadwal matkul** belum diketahui — beda dari Blockchain ("Senin (1)") dan Forensics ("Senin (2)"), belum ada info matkul ini jam berapa/hari apa.

## Terkait
- [[_SecureProgramming]]
