---
matkul: Secure Programming
minggu: 9
sks: 3
sumber: S9-OWASP_code_auditing_II.pptx
tags: [kuliah/secure-programming, minggu/w09]
status: draft
diproses: 2026-09-04
---

# W09 — Code Auditing based on OWASP Top 10 (II): A02 & A06

## Ringkasan
> - Maya, auditor baru, ngaudit source code **BrewNote** (app loyalty kafe) sebelum rilis. Dia tanya dua pertanyaan auditor buat tiap baris kode: **(1) Di mana kode ini PERCAYA sesuatu yang seharusnya nggak dia percaya? (2) Di mana data sensitif dibiarin TERBUKA?**
> - Dua benang: **A06 Supply Chain** (kode yang nggak Maya tulis sendiri — library dan dependency) dan **A02 Cryptographic Failures** (gimana BrewNote nyimpen dan ngirim password dan data pribadi).
> - **Supply chain failure** = komponen yang kamu andelin itu **rentan** (punya CVE), **usang/nggak dimaintain**, atau **nggak dipercaya** (dari sumber nggak resmi). Makin dalem dependency-nya (transitive), makin susah dilacak.
> - **Cryptographic failure** bukan soal enkripsi kuat yang dijebol — kebanyakan breach karena kripto yang **HILANG, LEMAH, atau SALAH PAKAI**.
> - Tiga kata yang jangan ketuker: **Hashing** (satu arah, buat password — bcrypt/Argon2id), **Encryption** (dua arah, bisa dibalikin dengan key — AES-GCM), **Salting** (nilai acak per-user sebelum di-hash).
> - Kasus: **Log4Shell (2021)** — bug logging library skor CVSS **10.0**, RCE tanpa autentikasi. **RockYou (2009→2021)** — password plaintext bocor jadi wordlist cracking yang masih dipake sampe sekarang.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Supply chain failure | Risiko dari komponen pihak ketiga (library/dependency) yang rentan, usang, atau nggak dipercaya |
| Transitive dependency | Dependency dari dependency-mu — yang nggak kamu pilih langsung tapi ikut kebawa |
| SBOM | Software Bill of Materials, daftar lengkap komponen software yang dipake |
| SCA | Software Composition Analysis, tool buat scan dependency cari kerentanan |
| Hashing | Fungsi satu arah yang nggak bisa dibalikin; buat password |
| Encryption | Perlindungan dua arah yang bisa dibalikin pakai key; buat data yang harus dibaca lagi |
| Salting | Nilai acak unik per-user yang ditambahin sebelum hashing |
| CVE | Common Vulnerabilities and Exposures, ID standar kerentanan software yang udah dikenal publik |

## Isi

### Cerita: Maya, auditor kode baru
**BrewNote**, aplikasi loyalti web buat kafe. Maya baru gabung startup kecil. Tugas pertamanya: audit source code BrewNote sebelum diluncurin ke ribuan pecinta kopi. Dia jago database — tabel, query, data tersimpan — tapi web app itu hal baru buatnya. Jadi dia nanya dua pertanyaan auditor ke tiap baris kode:

**Q1** — Di mana kode ini **PERCAYA** sesuatu yang seharusnya nggak dia percaya?
**Q2** — Di mana data sensitif dibiarin **TERBUKA**?

Dua benang yang ditelusuri hari ini: **A06 Supply Chain** (kode yang nggak Maya tulis: library dan dependency pihak ketiga yang dia warisi) dan **A02 Cryptographic Failures** (gimana BrewNote nyimpen dan ngirim password serta data pribadi). Pro PHP Security nge-frame ini persis: **"distrust by default."**

### Konsep: supply chain failures — "kode yang bukan kamu yang nulis"
App modern itu **dirakit, bukan ditulis dari nol**. BrewNote narik puluhan library pihak ketiga buat logging, image handling, PDF receipt, payment…

**Supply chain failure kejadian pas komponen yang kamu andelin itu:**
- **Vulnerable** — punya kelemahan keamanan yang dikenal (sebuah CVE)
- **Outdated/unsupported** — udah nggak dapet patch lagi
- **Untrusted** — diambil dari sumber nggak resmi atau yang udah diutak-atik

*CWE-1104 Unmaintained 3rd-party component · CWE-937 Known-vulnerable component*

**Kedalaman tersembunyi: nested dependencies.**
```
L0  Kodemu               1 package yang kamu pilih
L1  Direct dependencies  ~20 library yang kamu tambahin
L2  Transitive deps      Ratusan yang tertarik otomatis
L3  Yang nggak diketahui siapa pun    <- risiko sembunyi di sini
```

**Supply chain, dijelasin lewat database:**

| Yang udah kamu lakuin sama database | Sekarang buat library aplikasi |
| --- | --- |
| Nggak nulis DB engine-nya sendiri, kamu percaya MySQL/PostgreSQL | Nggak nulis library logging/PDF/crypto sendiri, kamu percaya mereka |
| Nerapin patch DB & upgrade versi kalau ada CVE | Library juga dapet CVE; butuh di-patch dengan ritme yang sama |
| Nggak bakal jalanin database berumur tahunan yang belum di-patch di production | Library yang nggak di-maintain = database yang nggak di-patch: sasaran empuk |
| Ngecek siapa yang nyuplai stored procedure sebelum dipercaya | Cuma tarik package dari sumber resmi — kayak nge-vetting supplier |

### Konsep: cryptographic failures — pas proteksi hilang, lemah, atau disalahgunakan
Kebanyakan breach **BUKAN dari nge-jebol enkripsi kuat**. Mereka datang dari kripto yang hilang, lemah, atau dipake salah.

**Tiga bentuk kegagalan yang diburu Maya:**
- **Missing** — data sensitif dikirim/disimpen dalam teks polos (HTTP, password plaintext)
- **Weak** — algoritma yang udah dijebol: MD5, SHA-1, DES, RC4, mode ECB
- **Misused** — key hardcode, IV yang dipake ulang, key ke-commit ke source control

**Pertanyaan pertama auditor: apa yang perlu dilindungi, dan di mana?**
- **Data in transit** — lagi jalan lewat jaringan → **harus TLS (HTTPS), jangan pernah HTTP polos**
- **Data at rest** — lagi diem di database/file → **enkripsi; hash password pakai KDF yang kuat**

**Tiga kata yang auditor jangan pernah ketuker:**

| | Hashing | Encryption | Salting |
| --- | --- | --- | --- |
| Sifat | Satu arah, sidik jari. Nggak bisa dibalikin. Buat password | Dua arah: dilindungi sekarang, bisa dipulihkan pakai key. Buat data yang harus dibaca lagi | Nilai acak per-user, ditambahin sebelum hashing biar password sama tetep beda hasil |
| PAKAI | bcrypt, Argon2id, scrypt | AES-GCM, ChaCha20-Poly1305 | salt acak unik tiap user |
| HINDARI | MD5, SHA-1 (kecepetan / udah dijebol) | DES, RC4, mode ECB | nggak pakai salt / satu salt dipake bareng-bareng |

**Aturan praktis audit: password di-hash + di-salt (JANGAN PERNAH dienkripsi, jangan pernah plain). Rahasia yang harus dibaca lagi dienkripsi dengan cipher terautentikasi modern.**

### Kenali ancamannya: bendera merah dependency rentan
Terpaku di versi lama — `composer.json`/`package.json` nunjukin rilis yang udah ketinggalan tahunan dari versi sekarang. Nggak ada inventori sama sekali — nggak ada yang bisa jawab "versi apa aja yang kita jalanin?" — termasuk transitive dependency. Diambil dari sumber aneh — package ditarik lewat HTTP atau mirror nggak resmi, nggak ada pengecekan signature. Proyek nggak di-maintain — commit terakhir udah bertahun-tahun lalu, nggak ada patch keamanan buat rilis lama. CVE yang dikenal, belum di-patch — dependency-nya muncul di advisory CVE/NVD tapi app-nya tetep ngirim itu. Nggak ada scanning di pipeline — nggak ada langkah Dependency-Check/SCA — kerentanan lolos ke production diam-diam.

**Maya baca manifest dependency BrewNote:**
```json
{
  "require": {
    "monolog/monolog": "1.12.0",
    "guzzlehttp/guzzle": "6.2.0",
    "league/flysystem": "1.0.20",
    "phpmailer/phpmailer": "5.2.14"
  }
}
```
- **`phpmailer` 5.2.14** — Rilis lama 5.x, secara historis kena CVE RCE & header-injection. Severity tinggi.
- **`monolog` 1.12.0** — Library logging, ketinggalan bertahun-tahun — inget Log4Shell: logger itu attack surface beneran.
- **`guzzle` 6.2.0** — HTTP client jauh ketinggalan versi sekarang; cek advisory buat fix request smuggling.

Tindakannya: pin ke versi yang udah di-patch, jalanin ulang SCA, dan dokumentasiin tiap keputusan upgrade.

**Bau kripto yang bisa langsung ketauan Maya:**
```php
// simpen password baru
$hash = md5($password);
$db->query("INSERT INTO users (pw) VALUES ('$hash')");

// API key, langsung nempel di source
$key = "sk_live_8F3kZ9...";

// halaman login dilayanin lewat HTTP
$url = "http://brewnote.app/login";
```
- **MD5 buat password** — Dijebol & terlalu cepat — bisa di-crack dalam hitungan detik. **CWE-327.**
- **API key hardcode** — Rahasia di source = rahasia di tiap clone & backup. **CWE-259.**
- **Login lewat HTTP polos** — Kredensial lewat dalam teks polos. **CWE-319.**

### Analisis ancaman: dari dependency lemah ke breach
1. **Find** — penyerang scan cari app yang pakai library dengan CVE publik, tool otomatis lakuin ini di skala internet. →
2. **Exploit** — mereka kirim exploit yang udah dikenal. Cacat di komponen jalan dengan hak akses penuh app-nya. →
3. **Pivot** — eksekusi kode di server → nyampe database, secrets, jaringan internal. →
4. **Persist** — karena ini dependency, cacat yang sama nempel di banyak vendor, blast radius-nya besar.

**Kenapa auditor nge-rate ini TINGGI: komponen jalan dengan hak akses appnya sendiri, jadi satu library cacat bisa berarti kompromi server penuh — dan kamu ngewarisin risiko dari kode yang nggak pernah kamu tulis.**

### Dari hash lemah ke mass account takeover
1. Database bocor (misalnya lewat SQL injection atau pencurian backup)
2. Password disimpen sebagai hash MD5 cepat — **tanpa salt**
3. Penyerang jalanin cracking offline: miliaran tebakan per detik
4. Kebanyakan password ketemu dalam hitungan menit; dipake ulang di situs lain
5. **Mass account takeover** — user-nya, terus reputasi BrewNote

**Lensa risiko auditor:** **Likelihood** — Tinggi, database bocor itu umum; nge-crack hash lemah itu sepele. **Impact** — Parah, kredensial, PII, paparan regulasi (GDPR/PCI DSS). **Detectability** — Sering senyap, aplikasinya tetep jalan; nggak ada yang notice sampe kebocoran ketahuan.

### Kasus nyata 1: Log4Shell — waktu logging library bikin internet rusak
**Apache Log4j · CVE-2021-44228 · Desember 2021.** Cacat di library logging Java yang sangat populer, ngebolehin penyerang jalanin kode di server cuma dengan bikin string yang dirancang khusus itu ke-log. Kena di Log4j 2.0-beta9 sampe 2.14.1, di-patch di 2.15.0+ (perbaikan lanjutan sampe 2.17.1). Bisa ditrigger lewat input biasa (pesan chat, header) yang cuma di-log. Nyampe ke berbagai produk, seringnya sebagai transitive dependency yang dalem — tim nggak sadar mereka punya itu.

**10.0** skor CVSS, maksimum yang mungkin. **RCE** eksekusi kode jarak jauh tanpa autentikasi. **Bertahun-tahun** — tertanam sedalam itu, bakal nempel sedekade.

**Pelajaran audit: kamu nggak bisa nge-patch apa yang nggak kamu liat. Inventarisasi transitive dependency.**

### Kasus nyata 2: RockYou — kejadian pas password nggak dilindungi
**RockYou (2009) → kompilasi "RockYou2021".** Tahun 2009, sebuah breach ngeksposkan ~32 juta password yang disimpen **teks polos — nggak di-hash sama sekali**. Daftar itu jadi fondasi wordlist password-cracking yang masih dipake sampe sekarang. Tahun 2021, kompilasi bernama "RockYou2021" ngumpulin ~8,4 miliar password dari banyak kebocoran. **Akar masalah di semua kasus: password disimpen nggak di-hash, atau di-hash lemah/tanpa salt yang gampang dijebol.**

**Perbandingan auditor:**
- **Yang kejadian:** Teks polos/hash lemah → tiap password langsung bisa dipake → credential stuffing di seluruh internet.
- **Yang dikasih penyimpanan kuat:** Argon2id/bcrypt yang di-salt → database yang bocor jadi hampir nggak berguna → cracking jadi nggak feasible dalam skala besar.

### Pencegahan: playbook supply chain (dari OWASP)
- **Inventory everything** — pertahankan SBOM buat semua komponen client & server-side dan transitive dependency-nya
- **Scan continuously** — otomatiskan dengan tool SCA (OWASP Dependency-Check, retire.js) dan pantau feed CVE/NVD
- **Patch on a cadence** — update segera; hapus dependency yang nggak dipakai; buang yang nggak di-maintain
- **Trust the source** — tarik cuma dari sumber resmi lewat koneksi aman; utamain package yang bertanda tangan
- **Subscribe to alerts** — dapetin bulletin keamanan buat komponen yang kamu pakai biar tau lebih awal
- **Virtual patch kalau kejebak** — kalau nggak bisa upgrade segera, pasang virtual patch/WAF rule buat beli waktu

### Pencegahan: playbook kripto & perbaikan BrewNote
- Enkripsi in-transit: paksa TLS (HTTPS) + HSTS; jangan pernah HTTP polos
- Hash password pakai KDF yang lambat dan di-salt: Argon2id, bcrypt, atau scrypt
- Enkripsi data at-rest pakai cipher terautentikasi modern (AES-GCM)
- Jaga key di luar source: pakai secrets manager/environment, rotasi key
- Jangan simpen apa yang nggak kamu butuhin; klasifikasiin data sensitif dulu
- Pakai library yang udah teruji — jangan pernah bikin kripto sendiri

```php
// hashing yang kuat dan di-salt (KDF)
$hash = password_hash($password, PASSWORD_ARGON2ID);

// secret dari environment, bukan dari kode
$key = getenv("STRIPE_KEY");

// paksa HTTPS
$url = "https://brewnote.app/login";

// verifikasi pas login
password_verify($input, $hash);
```
**Catatan:** `password_hash()` bawaan PHP nge-salt otomatis — persis pelajaran "pakai kripto vetted-nya platform" dari Pro PHP Security.

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — kontennya semua teks dan blok kode.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **CycloneDX** | Format standar buat SBOM (Software Bill of Materials) |
| **PII** | Personally Identifiable Information, data yang bisa mengidentifikasi seseorang |
| **PCI DSS** | Payment Card Industry Data Security Standard, standar keamanan buat data kartu pembayaran |
| **KDF (Key Derivation Function)** | Fungsi yang ngubah password jadi key/hash dengan proses lambat yang disengaja, biar susah di-brute-force |
| **IV (Initialization Vector)** | Nilai acak yang dipakai sekali di awal proses enkripsi; kalau dipake ulang bisa ngelemahin keamanannya |
| **ECB mode** | Mode enkripsi block cipher yang lemah karena pola data yang sama menghasilkan output terenkripsi yang sama juga |
| **Retire.js** | Tool SCA khusus buat nyari library JavaScript yang punya kerentanan dikenal |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan Log4Shell (kerentanan di LIBRARY logging) dengan kasus MD5 di BrewNote (kesalahan di KODE APLIKASI sendiri). Dari sisi audit, kenapa Log4Shell jauh lebih susah "dicegah" cuma dengan code review manual dibanding kesalahan MD5 yang Maya temukan langsung di `login.php`?
2. **(C4 – Analisis)** Rantai "dari hash lemah ke mass account takeover" (slide 17) punya 5 langkah. Analisis: kalau organisasi CUMA bisa motong rantai itu di SATU titik, titik mana yang paling efektif dipotong (misalnya: nyegah SQLi di langkah 1, atau ganti ke Argon2id di langkah 2), dan kenapa?
3. **(C5 – Evaluasi)** Sebuah tim bilang: "Kita nggak perlu SBOM atau SCA scanning, karena kita review tiap baris kode kita sendiri dengan teliti." Evaluasi klaim ini pakai konsep transitive dependency dan kasus Log4Shell — kenapa "review kode sendiri" nggak cukup buat nutup risiko A06?
4. **(C5 – Evaluasi)** Bandingkan efektivitas dua kontrol ini buat mencegah dampak RockYou-style breach: (a) hash password pakai Argon2id dengan salt, vs (b) enforce password policy yang kuat (minimal 12 karakter, kombinasi kompleks). Mana yang lebih efektif ngelindungin data KALAU database-nya udah kebobolan, dan kenapa?
5. **(C6 – Cipta)** BrewNote mau nambah fitur baru: generate laporan PDF kwitansi pembelian yang dikirim ke email pelanggan. Rancang checklist audit A02 + A06 (minimal 4 poin, gabungan dari checklist slide 24) yang HARUS dicek khusus buat fitur ini, dengan alasan kenapa masing-masing poin relevan buat fitur PDF + email ini secara spesifik (bukan cuma nyalin checklist generik).

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W08 - Code Auditing OWASP I]]
- [[W10 - Code Auditing OWASP III]]
- [[SecureProgramming - Review dan Glosari]]
