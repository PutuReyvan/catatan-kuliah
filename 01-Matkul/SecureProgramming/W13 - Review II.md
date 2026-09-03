---
matkul: Secure Programming
minggu: 13
sks: 3
sumber: Review II.pptx
tags: [kuliah/secure-programming, minggu/w13]
status: draft
diproses: 2026-09-04
---

# W13 — Review II: OWASP Top 10 Web Application Security Risks

## Ringkasan
> - Review akhir semester, ngerangkum SELURUH OWASP Top 10 (2021) lewat satu cerita: **BrewNote**, web app kafe. Satu malam, seorang penyerang "main-main" sama BrewNote — dan kita ikutin tiap langkahnya.
> - **OWASP Top 10** = daftar konsensus, bukan checklist lengkap-segalanya. Peringkatnya dihitung dari tiga faktor: **Prevalence** (seberapa sering muncul), **Exploitability** (segampang apa dieksploitasi), **Impact** (separah apa kalau berhasil).
> - **10 risiko, 4 "kantong" — hafalin kantongnya, bukan daftarnya:** *Siapa boleh ngapain?* (A01, A07) · *Bisa dipercaya nggak input & datanya?* (A03, A02, A10) · *Dibangun & disetup dengan aman?* (A04, A05, A06) · *Bisa diverifikasi & dideteksi?* (A08, A09)
> - **Breach beneran jarang cuma satu bug.** Rantai serangan BrewNote: **A05** (misconfig bocorin schema) → **A03** (injection nguras data) → **A02** (kripto lemah buka password) → **A07** (auth failure ngasih akses) → **A01** (broken access nyampe admin). **Nggak ada satu bug pun yang "critical" sendirian — dirantai, mereka jadi kompromi penuh.**
> - **Lima kebiasaan yang ngalahin sebagian besar Top 10** — bahkan risiko yang belum pernah kamu liat: **jangan percaya input, deny by default, defense in depth, assume breach, verify jangan cuma percaya.**
> - **Jembatan besar dari database ke web security:** prepared statement = defense SQLi (A03). `WHERE owner_id = …` = access control (A01). `GRANT`/least privilege = deny-by-default (A01/A05). Enkripsi at-rest & TLS = A02. Transaction & audit log = A09.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| OWASP | Open Worldwide Application Security Project, organisasi nonprofit penerbit panduan keamanan web gratis |
| Prevalence | Seberapa sering sebuah kelemahan beneran muncul di aplikasi yang dites |
| Exploitability | Segampang apa penyerang bisa memicu sebuah kelemahan |
| Impact | Seberapa parah kerusakan kalau sebuah kelemahan berhasil dieksploitasi |
| Attack chain | Rangkaian beberapa kelemahan kecil yang digabung jadi satu kompromi besar |
| Cheat Sheet Series | Kumpulan panduan praktis OWASP per topik keamanan spesifik |

## Isi

### Setup: BrewNote — satu app, banyak pintu
**BrewNote** — web app kafe di lingkungan sekitar. Pelanggan login, nyimpen poin loyalti, order duluan, dan nyimpen kartu. Staf pakai halaman admin buat ngurus menu, refund, dan akun. Di baliknya ada app PHP yang ngobrol ke database MySQL — wilayah yang udah familiar. **Dibangun cepet. Diamankan belakangan. Kedengeran familiar?**

Tiap fitur itu pintu yang bisa diketok penyerang: **Login & session** (kamu siapa, dan apa kamu masih kamu?). **Data loyalti/akun** (bisa nggak gue liat poin orang lain?). **Form order & pencarian** (input-nya nyampe database dengan aman nggak?). **Halaman admin** (bisa nggak customer nyampe tool khusus staf?). **Detail kartu tersimpen** (data sensitifnya beneran dilindungi nggak?).

### Apa itu OWASP Top 10?
**Kosakata bersama buat risiko web.** OWASP (Open Worldwide Application Security Project) adalah nonprofit yang nerbitin panduan keamanan gratis. Top 10 adalah **dokumen kesadaran standar** — konsensus luas soal risiko paling kritis buat aplikasi web. Ini titik AWAL, bukan checklist segala-galanya — bahasa bersama biar tim, auditor, dan tool semuanya ngartiin hal yang sama.

**Kenapa ini penting buatmu: ini ngubah "jadilah aman" jadi sepuluh masalah yang bisa dinamain, ditest, dan dibenerin.**

**Edisi saat ini: Top 10:2021.** **3** kategori baru vs 2017. **4** yang berganti nama/ruang lingkup. **~500 ribu** kejadian CWE yang dianalisis. Ada release candidate 2025; **2021 tetep jadi standar yang dirujuk buat matkul ini.**

### Gimana sebuah risiko dapet peringkatnya
OWASP nggabungin tiga faktor. Sebuah risiko bukan cuma soal "seberapa parah" — tapi juga "seberapa sering":
- **Prevalence** — seberapa sering kelemahan ini beneran muncul di app yang dites? (tingkat kejadian)
- **Exploitability** — segampang apa buat penyerang memicunya? Tooling, skill, dan akses yang dibutuhin.
- **Impact** — kalau dieksploitasi, seberapa besar kerusakannya? Kehilangan data, account takeover, kompromi penuh.

**Pergeseran kunci di 2021: peringkat diseimbangkan ulang ke arah prevalence.** Itu sebabnya **Broken Access Control** loncat dari #5 ke #1 — dia muncul hampir di mana-mana yang ada autentikasi.

### Sepuluh risiko, empat kantong
**Hafalin kantongnya, bukan daftarnya.** Tiap kantong jawab satu pertanyaan keamanan.

| Pertanyaan | Kantong | Risiko |
| --- | --- | --- |
| Siapa boleh ngapain? | **Access & Identity** | A01 Broken Access Control · A07 Identification & Auth Failures |
| Bisa dipercaya nggak input & datanya? | **Injection & Crypto** | A03 Injection · A02 Cryptographic Failures · A10 SSRF |
| Dibangun & disetup dengan aman? | **Design & Config** | A04 Insecure Design · A05 Misconfiguration · A06 Vulnerable Components |
| Bisa diverifikasi & dideteksi? | **Integrity & Visibility** | A08 Software/Data Integrity · A09 Logging & Monitoring |

### Pengintaian, lalu Analisis
Seorang penyerang ngabisin satu malam sama BrewNote. Kita ikutin tiap gerakannya — namain risiko OWASP-nya, terus nalar kenapa itu bisa berhasil.

**A01 — Broken Access Control (risiko #1).** Authentication = kamu siapa? Authorization = kamu boleh ngapain? Broken access control = aplikasinya gagal nerapin yang kedua. **Breach BrewNote:** penyerang liat halaman loyalti-nya sendiri di `/account?id=1043` … terus ngedit URL-nya jadi `id=1044` dan baca data orang asing. Nggak ada pengecekan yang ngonfirmasi kepemilikan. **94%** aplikasi yang dites punya bentuk broken access control — kelemahan paling umum di dataset OWASP 2021. Pola ini disebut **IDOR** — Insecure Direct Object Reference — percaya kunci yang disupply user (CWE-639) buat nunjuk ke record. Fix-nya (preview): deny by default, dan cek kepemilikan di server pada TIAP request — jangan pernah percaya ID di URL.

**A03 — Injection; Jembatan DB.** Injection kejadian pas input nggak dipercaya diperlakukan sebagai kode alih-alih data. SQL injection itu yang udah setengah kamu tau dari database. **Pencarian rentan BrewNote:** `$q = "SELECT * FROM orders WHERE note = '$input'";` — penyerang ngetik `' OR '1'='1` … dan klausa `WHERE`-nya jadi selalu true. Semua order bocor. **Jembatan DB:** kamu udah belajar pertahanan ini di kelas database TANPA label keamanannya — **parameterized query / prepared statement**. Injection lebih luas dari SQL: SQL/NoSQL (ke query database), Command (ke shell OS), Cross-Site Scripting/XSS (ke browser — sekarang masuk lipatan A03).

**A02 — Cryptographic Failures (risiko #2).** Dulunya "Sensitive Data Exposure". Dinamai ulang buat nunjuk ke akar masalahnya: data yang seharusnya dilindungi — in transit atau at rest — tapi enggak, atau dilindungi kripto yang lemah/usang. **Kegagalan umum:** password disimpen plaintext atau hash cepat/tanpa-salt. Trafik lewat HTTP, atau HTTPS-nya nggak dipaksa. Key hardcode; algoritma usang (MD5, SHA-1, DES). *Jembatan DB: ini encryption at-rest & TLS database-mu, diterapin ujung-ke-ujung.* **Breach BrewNote:** BrewNote nyimpen nomor kartu dan password sebagai hash `md5()` polos. Pas penyerang nge-dump tabel users lewat injection A03, password-nya efektif kebaca — dan dipake ulang di situs lain. **Perhatiin rantainya: A03 ngeluarin datanya → A02 bikin dia bisa dipake. Breach beneran jarang satu risiko — mereka kombinasi.**

**A07 — Identification & Authentication Failures.** Kelemahan dalam ngonfirmasi identitas dan ngelola session: password lemah dibolehin, nggak ada rate-limiting, token session gampang ditebak/nggak dilindungi, "lupa password" yang rusak. **Breach BrewNote:** BrewNote nggak punya login throttling. Penyerang ambil daftar email dari dump sebelumnya dan otomatisin ribuan percobaan login pakai password bocor dari situs lain. Karena pelanggan pake ulang password, beberapa akun — termasuk satu staf — kebuka. **Kenapa analisis penting:** form login-nya kelihatan baik-baik aja dan "jalan". Cacatnya nggak keliatan sampai kita nanya: apa yang nyetop orang nyoba sejuta kali? Mengenali kontrol yang HILANG itu skillnya.

**A05 & A06 — Risiko yang senyap.**
- **A05 · Security Misconfiguration** — app-nya baik-baik aja; setup-nya enggak. Password default, pesan error verbose, direktori terbuka, fitur nggak perlu yang masih nyala. **Breach BrewNote:** `display_errors` PHP nyala di production. Request yang sengaja dirusak nyetak stack trace lengkap — ngungkapin nama database, struktur tabel, dan path file yang dipake penyerang buat nyempurnain tiap serangan lain.
- **A06 · Vulnerable & Outdated Components** — kodemu mungkin bersih, tapi dependency-mu bukan punyamu. Library lama bawa CVE publik yang siapa aja bisa cek. **Breach BrewNote:** BrewNote pakai library image-upload berumur 3 tahun dengan CVE publik. Penyerang nggak butuh kreativitas — cuma exploit yang cocok dari pencarian. **Unpatched = udah pre-breached.**

**Empat lagi buat dikenali:**
- **A04 · Insecure Design** — cacat yang tertanam di RENCANA-nya, bukan kode-nya. BrewNote ngebolehin refund tanpa batas tanpa langkah approval — jalan sesuai desain, desainnya yang salah. *Fix: threat model sebelum bangun.*
- **A08 · Software & Data Integrity Failures** — percaya update, plugin, atau pipeline CI/CD tanpa verifikasi. Auto-update yang nggak ditandatangani bisa ngirim kode penyerang. *Fix: verifikasi signature & sumbernya.*
- **A09 · Logging & Monitoring Failures** — BrewNote nggak nge-log apa-apa. Seluruh serangan malam itu nggak keliatan sampai pelanggan komplain berminggu-minggu kemudian. **Kamu nggak bisa merespon apa yang nggak bisa kamu liat.**
- **A10 · Server-Side Request Forgery** — server ditipu buat ngambil URL pilihan penyerang — misalnya endpoint admin internal atau metadata cloud. Aplikasinya jadi proxy-nya penyerang.

### Analisis breach-nya: rantainya
```
A05  Misconfig bocorin schema      → stack trace ngungkapin struktur DB
A03  Injection nguras data         → ' OR '1'='1 ngosongin tabel-tabel
A02  Kripto lemah buka password    → hash md5 dijebol seketika
A07  Auth failure ngasih akses     → credential stuffing, nggak ada rate limit
A01  Broken access nyampe admin    → record orang asing & staf terekspos
```
**Nggak ada satu bug pun yang "critical" sendirian. Dirantai, mereka jadi kompromi penuh. Ini kenapa kamu analisis SISTEMNYA, bukan cuma satu baris kode.**

### Menerapkan Web Security Method
Tiap risiko yang kita kenali itu punya defense konkret bernama — dan OWASP Cheat Sheet gratis yang ngasih tau persis caranya.

| Risiko OWASP | Metode keamanan web utamanya |
| --- | --- |
| A01 Broken Access Control | Deny by default; tegakkan otorisasi di sisi server tiap request |
| A02 Cryptographic Failures | TLS di mana-mana; enkripsi at-rest; hash password pakai bcrypt/argon2 |
| A03 Injection | Parameterized query; validasi input; encode output (XSS) |
| A04 Insecure Design | Threat-model dari awal; pakai pola desain aman & arsitektur referensi |
| A05 Misconfiguration | Baseline yang dikencengin; matiin debug di prod; least functionality |
| A06 Vulnerable Components | Inventarisasi dependency; patch terjadwal; lacak CVE |
| A07 Auth Failures | MFA; rate-limit & lockout; manajemen session yang kuat |
| A08 Integrity Failures | Verifikasi signature & sumber; amankan CI/CD pipeline |
| A09 Logging & Monitoring | Log event keamanan; alert; test jalur respon |
| A10 SSRF | Validasi & allow-list URL keluar; segmentasi jaringan |

### Prinsip yang ngalahin hafalan
Lima kebiasaan ngalahin sebagian besar Top 10 — bahkan risiko yang belum pernah kamu liat.
1. **Never trust input** — perlakukan tiap byte dari user sebagai musuh sampai tervalidasi (A03, A10)
2. **Deny by default** — mulai dengan nol akses; kasih minimum (A01, A05)
3. **Defence in depth** — lapisin kontrol — nggak ada satu kegagalan pun yang fatal (rantai secara keseluruhan)
4. **Assume breach** — log, monitor, dan rencanain respon — kamu bakal ngelewatin sesuatu (A09)
5. **Verify, don't trust** — cek signature, sumber, dan kepemilikan sebelum bertindak (A02, A08)

### Kasus: Equifax (2017) — A06 Vulnerable Component
Cacat yang belum di-patch di framework Apache Struts ngebolehin penyerang jalanin kode di server Equifax. **Perbaikannya udah tersedia selama berbulan-bulan.** Komponennya — bukan kode Equifax sendiri — yang jadi pintu terbuka. **Kenapa BrewNote mirip kasus ini:** akar masalah yang sama kayak library upload lama BrewNote: kerentanan yang udah dikenal dan dipublikasikan, dibiarin nggak di-patch. Skalanya beda; pelajarannya identik.

**~147 juta** orang data pribadinya terekspos. **$700 juta+** dalam settlement & denda. **1** patch yang hilang di akarnya.

### Kasus: keluarga Injection
SQL injection udah nggerakin breach lebih dari dua dekade — dari pencurian kartu retail sampe database pemilih dan pelanggan yang bocor. **Dia bertahan karena fix-nya simpel tapi harus diterapin di MANA-MANA — satu query yang kelewat aja udah cukup.** *Jembatan DB, sekali lagi:* parameterized query itu prepared statement yang sama dari kelas database-mu — keamanan dan praktik DB yang baik itu kebiasaan yang sama. **Kenapa dia tetep di daftar:** masih **~94%** app yang dites buat injection di 2021 (OWASP). Tool otomatis nge-scan seluruh web buat ini terus-menerus. XSS — injection ke browser — sekarang bagian dari A03.

### Kamu udah tau lebih banyak dari yang kamu kira
Kelas database-mu udah ngajarin defense web-security dengan nama berbeda. Ide sama, label baru.

| Dari kelas DATABASE | Hal yang sama di WEB SECURITY | Nyetop |
| --- | --- | --- |
| Prepared statements | Pertahanan lawan SQL Injection | A03 |
| `WHERE owner_id = …` | Otorisasi/pengecekan kepemilikan sisi-server | A01 |
| `GRANT`/role & least privilege | Access control deny-by-default | A01 / A05 |
| Enkripsi at-rest & TLS | Perlindungan kriptografis data | A02 |
| Transaction & audit log | Logging & monitoring keamanan | A09 |

### Metode review yang bisa diulang
Terapin ini ke web app apa pun — ini inti prosedural dari LO3.
1. **List the doors** — petain tiap entry point: form, URL, API, upload, halaman admin
2. **Name the risk** — buat tiap pintu, tanya kantong OWASP mana yang berlaku — siapa/percaya/setup/verifikasi
3. **Analyse the path** — bisa nggak risikonya dirantai? Lacak apa yang dijangkau penyerang kalau satu pintu kebuka
4. **Apply the method** — cocokin tiap risiko ke defense-nya dan OWASP Cheat Sheet yang relevan
5. **Verify & monitor** — test fix-nya, terus log dan pantau biar percobaan berikutnya keliatan

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — kontennya semua teks, tabel, dan diagram alur.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **SSRF (Server-Side Request Forgery)** | Serangan yang nipu server buat ngambil URL yang dipilih penyerang, sering buat nyampe resource internal |
| **Cloud metadata endpoint** | URL internal khusus di server cloud yang nyimpen kredensial/konfigurasi sensitif — target umum serangan SSRF |
| **Reference architecture** | Contoh/pola arsitektur yang udah divetting dan aman, dipake sebagai acuan desain |
| **Least functionality** | Prinsip: matiin/hapus fitur yang nggak beneran dipakai, biar attack surface-nya lebih kecil |
| **cheatsheetseries.owasp.org** | Situs kumpulan panduan praktis OWASP per topik keamanan (link penting yang dicatat di awal sesi ini) |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Petain rantai breach BrewNote (A05→A03→A02→A07→A01) ke EMPAT kantong OWASP (Access & Identity / Injection & Crypto / Design & Config / Integrity & Visibility). Analisis: apa rantai ini nunjukin pola pergerakan tertentu ANTAR kantong (misalnya, selalu dari "Design & Config" ke "Injection & Crypto" ke "Access & Identity")? Kalau ada, jelasin urutan logisnya.
2. **(C4 – Analisis)** Bandingkan kasus Equifax (2017, A06) dengan rantai breach BrewNote. Equifax "cuma" satu risiko yang nggak di-patch, tapi dampaknya (147 juta orang) jauh lebih besar dari BrewNote (kafe kecil). Analisis: faktor APA (dari tiga faktor OWASP: prevalence, exploitability, impact) yang paling nentuin skala dampak Equifax, terlepas dari "cuma satu risiko"?
3. **(C5 – Evaluasi)** Sebuah tim developer baca lima prinsip (never trust input, deny by default, defense in depth, assume breach, verify don't trust) dan bilang: "Kalau kita udah nerapin kelimanya, kita nggak perlu belajar OWASP Top 10 secara detail lagi." Evaluasi klaim ini — apa yang MASIH hilang kalau tim cuma pegang lima prinsip umum tanpa tau detail spesifik tiap risiko (misalnya, gimana caranya tau kode kena A08 vs A04 kalau nggak tau definisi masing-masing)?
4. **(C5 – Evaluasi)** Dari sepuluh baris tabel "Risk → Method" (slide 18), pilih TIGA metode yang menurutmu PALING SUSAH diterapin konsisten oleh tim developer kecil (kayak tim Maya di BrewNote) dengan sumber daya terbatas, dan jelasin kenapa — pertimbangkan biaya, kompleksitas teknis, atau kebutuhan proses/organisasi (bukan cuma kode).
5. **(C6 – Cipta)** Pakai "metode review yang bisa diulang" (5 langkah di slide 23) buat SATU fitur baru BrewNote yang belum disebut di cerita: fitur "pelanggan bisa ngundang teman lewat link referral, dan dapet poin loyalti kalau temannya daftar." Jalanin kelima langkahnya secara singkat: (1) list pintunya, (2) namain risiko OWASP yang paling relevan per pintu, (3) analisis kemungkinan rantai serangan, (4) sebutkan metode pertahanan utamanya, (5) sebutkan apa yang perlu di-log/monitor.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W01 - Introduction to Web Application]]
- [[W07 - Review]]
- [[W08 - Code Auditing OWASP I]]
- [[W09 - Code Auditing OWASP II]]
- [[W10 - Code Auditing OWASP III]]
- [[W11 - Code Auditing OWASP IV]]
- [[W12 - Code Auditing OWASP V]]
- [[SecureProgramming - Review dan Glosari]]
