---
matkul: Secure Programming
minggu: 12
sks: 3
sumber: S12-Code Auditing based on OWASP Top 10 (V).pptx
tags: [kuliah/secure-programming, minggu/w12]
status: draft
diproses: 2026-09-04
---

# W12 — Code Auditing based on OWASP Top 10 (V): A09 & Exception Handling

## Ringkasan
> - **FinPay**, app pembayaran kecil. Suatu pagi, peneliti luar ngirim email: "Data pelanggan kalian dijual online." Tim investigasi, dan nemuin sesuatu yang lebih parah dari breach-nya sendiri: **HAMPIR NGGAK ADA LOG.** Nggak ada catatan siapa login, query mana yang gagal, atau kapan penyerang pertama masuk. **Intrusinya mungkin udah aktif bertahun-tahun, sepenuhnya nggak keliatan.**
> - Ini beneran kejadian: contoh OWASP — situs asuransi kesehatan anak nggak bisa deteksi breach karena nggak ada logging. **3,5 juta+ record anak terekspos**, kemungkinan sejak 2013 — **7+ tahun nggak ketahuan.**
> - **Rantai satu jalur:** kondisi luar biasa terjadi → kamu LOG dia → log-nya TRIGGER alert → seseorang RESPON. **Putus satu mata rantai aja, penyerang tetep nggak keliatan.**
> - Rata-rata industri buat DETEKSI breach: **~200 hari.** Kasus gaya FinPay bisa nggak ketahuan **7+ tahun.**
> - **Dua cara salah nangenin error:** Mode A — bilang TERLALU BANYAK (stack trace bocor ke user, ngasih peta gratis ke penyerang). Mode B — bilang NGGAK BILANG APA-APA (catch block kosong, sistem ngira semua baik-baik aja padahal enggak).
> - **Log injection (CWE-117)**: kalau kamu nulis input mentah ke log, penyerang bisa nyelipin newline dan MEMALSUKAN baris log — nyembunyiin jejak atau nge-frame orang lain.
> - Prinsip: **jangan pernah log rahasia** (password, API key, PII lengkap); **selalu log siapa-apa-kapan-di mana**; **tangkep tiap error (`Throwable`), rollback, log, fail closed** — jangan pernah nampilin stack trace mentah ke user.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Logging | Mencatat event yang relevan secara keamanan dengan konteks yang cukup buat direkonstruksi nanti |
| Alerting | Ngubah log event penting jadi sinyal yang beneran DIRESPON manusia |
| Exceptional condition | Kejadian abnormal di luar alur normal (koneksi DB gagal, input salah, exception) |
| Log injection | Serangan yang nyelipin newline/karakter khusus ke input yang di-log buat memalsukan baris log |
| Fail closed | Prinsip: kalau ada error, tolak/batalkan aksinya secara default, jangan lanjut setengah-setengah |
| Mean-time-to-detect | Rata-rata waktu yang dibutuhin organisasi buat SADAR ada breach |
| `Throwable` | Tipe dasar PHP yang menangkap SEMUA jenis error dan exception |

## Isi

### Cerita: breach yang nggak ketauan tujuh tahun
**FinPay** adalah app pembayaran PHP kecil buatan tim Maya. Suatu pagi, peneliti luar ngirim email: **"Data pelanggan kalian dijual online."**

Tim-nya buru-buru investigasi — dan nemuin sesuatu yang **lebih parah dari breach-nya sendiri**: **hampir nggak ada log.** Nggak ada catatan siapa yang login, query mana yang gagal, atau kapan penyerangnya pertama kali masuk. **Intrusinya mungkin udah aktif selama bertahun-tahun, sepenuhnya nggak keliatan.**

**Ini beneran kejadian.** Contoh dari OWASP sendiri: situs asuransi kesehatan anak nggak bisa deteksi breach karena nggak punya logging atau monitoring. **Record lebih dari 3,5 juta anak terekspos** — dan intrusinya mungkin udah jalan sejak 2013, **lebih dari 7 tahun nggak ketahuan.**

### Kenapa ini penting
Kenapa satu sesi penuh buat "cuma logging"? **Logging itu satu-satunya bukti yang kamu punya SETELAH ada serangan.** Rata-rata industri buat deteksi breach itu sekitar **200 hari**. Error handling yang jelek bocorin rahasia SEKALIGUS nyembunyiin serangan.

**Kamu udah tau ini: dari kelas Database-mu.**
- Transaction log database nyatet tiap perubahan biar DB bisa recovery dan diaudit → **Log keamanan aplikasi lakuin hal yang sama buat aksi user & penyerang.**
- `ROLLBACK` pada transaction yang gagal bikin data tetap konsisten pas ada yang salah → **Exception handling itu ROLLBACK-nya level aplikasi: gagal dengan bersih, jangan ninggalin state setengah jadi.**
- `GRANT`/audit di tabel sensitif ngontrol dan nyatet siapa nyentuh apa → **Logging & alerting jawab pertanyaan yang sama — siapa-apa-kapan — di level aplikasi.**

### Tiga kata, didefinisikan presisi
- **Logging** — Mencatat event yang relevan secara keamanan — login, akses ditolak, kegagalan validasi — dengan konteks cukup buat direkonstruksi nanti. **"Black-box recorder."**
- **Alerting** — Ngubah event log penting jadi sinyal tepat waktu yang beneran DIRESPON manusia (atau tim SOC). Logging tanpa alerting = **detektor asap tanpa suara.**
- **Exceptional condition** — Kejadian abnormal apa pun yang nggak direncanain alur normal — koneksi DB gagal, input salah, exception yang dilempar. Cara kamu nanganinnya nentuin apa kamu tetep aman. **"Yang nggak terduga."**

**Mereka membentuk satu rantai: kondisi luar biasa terjadi → kamu LOG dia → log-nya TRIGGER alert → seseorang RESPON. Putus satu mata rantai aja, dan penyerang tetep nggak keliatan.**

### Di mana A09 duduk di OWASP Top 10
**Risiko sama, dua nama:** edisi 2021: **A09 – Security Logging and Monitoring Failures**. Edisi 2025: **A09 – Security Logging and Alerting Failures**. Perubahan namanya nambahin **alerting** — logging yang nggak ada yang nindaklanjuti itu nggak cukup. Teks 2025 juga secara eksplisit nyebut mishandling error & exceptional condition sebagai bagian kategori ini.

**Kenapa gampang kelewat:** susah dites — ini soal apa yang **NGGAK** kejadian (nggak ada log, nggak ada alert). Jarang muncul sebagai satu CVE tunggal, jadi scanner diem aja. Masuk daftar lewat survei komunitas, bukan data mentah.

**Mapped weakness (CWE):** **CWE-778** logging yang nggak cukup · **CWE-117** improper output neutralization for logs (log injection) · **CWE-223** penghilangan informasi relevan-keamanan · **CWE-532** penyisipan informasi sensitif ke file log.

### Security Logging & Alerting Failures: mengenali kegagalannya di kode
**Bau aplikasi yang buta:** Login, login gagal & akses ditolak nggak di-log. Kegagalan validasi lenyap — nggak ada jejak input yang lagi probing. Log ada tapi nggak ada yang baca; nggak ada alert yang nyala. Aksi bernilai tinggi (pembayaran, perubahan role) nggak ninggalin audit trail. Pen test & scan memicu nol alert. **Data sensitif (password, token) malah TERTULIS ke dalam log.**

**Bau audit — pengecekan login yang senyap:**
```php
if (!password_verify($pw, $hash)) {
    // login gagal... dan nggak ada
    // apa pun yang direkam sama sekali
    header('Location: /login');
    exit;
}
```
**Yang ditanya auditor: "Kalau penyerang nyoba 10.000 password di sini, di mana buktinya?" Jawabannya: nggak ada di mana-mana — itulah kegagalan A09 (CWE-778).**

### Menganalisis dampaknya
1. **Intrusion** — penyerang masuk lewat cacat lain
2. **Dwell** — nggak ada log → mereka bertahan berminggu-minggu, berbulan-bulan, bertahun-tahun
3. **Exfiltrate** — data keluar; nggak ada alert yang pernah dipicu
4. **Disclosure** — pihak ketiga yang ngasih tau kamu — bukan sistemmu sendiri

**~200 hari** rata-rata industri buat deteksi breach. **7+ tahun** breach gaya FinPay bisa tetep tersembunyi. **$$$ denda** paparan GDPR/regulasi setelah terungkap.

**Insight inti: kerusakan A09 bukan pembobolannya — itu WAKTU kamu jadi buta. Waktu deteksi itu keseluruhan permainan.**

### Jebakan halus: Log Injection
**Log juga bisa diserang (CWE-117).** Kalau kamu nulis input mentah user ke log, penyerang bisa nyelipin newline dan **memalsukan baris log palsu** — nyembunyiin jejak mereka atau nge-frame orang lain.

*Paralel database: ini sepupunya SQL injection. Data nggak dipercaya harus dinetralkan sebelum masuk ke log, sama persis kayak kamu parameterize sebuah query.*

**Username dikontrol penyerang → log dipalsukan:**
```
username = admin\n[OK] login admin from 10.0.0.9

// Yang sekarang ditampilin file log-nya:
[FAIL] login 'admin
[OK]  login admin from 10.0.0.9
      ↑ baris palsu, keliatan sah
```
**FIX: strip/encode CR & LF, terus log field yang terstruktur — jangan pernah concatenation mentah.**

### Studi kasus: benang merahnya sama
- **Rencana kesehatan anak** — nggak ada logging atau monitoring sama sekali. Pihak luar yang lapor breach-nya. 3,5 juta+ record terekspos; intrusi kemungkinan aktif sejak 2013 — 7+ tahun nggak ketahuan.
- **Maskapai besar (10 tahun data)** — data penumpang termasuk paspor & kartu dibobol di host pihak ketiga. Maskapainya baru tau setelah dikasih tau providernya — bukan dari monitoring sendiri.
- **Maskapai Eropa — pelanggaran GDPR** — kerentanan app pembayaran ngebolehin penyerang panen 400.000+ record pembayaran pelanggan; insiden yang harus dilaporkan ke GDPR nyusul.

**Benang merah bersama: di tiap kasus, korban dikasih tau oleh PIHAK LAIN. Sistem mereka sendiri tetep diam.**

### Mishandling of Exceptional Conditions

**Dua cara salah nangenin error:**

**Mode A — Bilang TERLALU BANYAK.** Error mentah nyampe ke user dan ngeksposkan internal.
```
Warning: mysqli_connect():
Access denied for user 'root'@
'10.0.2.5' (using password: YES)
in /var/www/app/db.php:14
```
**Bocor:** user DB, IP host, path file, tech stack — peta gratis buat penyerang.

**Mode B — Nggak Bilang Apa-apa.** Error-nya ditangkep dan diam-diam dibuang.
```php
try {
    chargeCard($order);
} catch (Exception $e) {
    // nggak ngapa-ngapain <- bahaya
}
```
**Nyembunyikan:** sistemnya ngira semua baik-baik aja. Nggak ada log, nggak ada alert, kemungkinan pembayaran setengah jadi.

### Kenapa dua ancaman ini bersaudara
**Error handling yang bagus itu SUMBER dari logging yang bagus.** Kalau kamu nggak nangkep error-nya, kamu nggak punya event buat di-log. Catch block kosong nggak cuma nyembunyiin bug — dia **ngilangin sinyal persis** yang dibutuhin A09 buat dicatat dan di-alert. **Dua kegagalan ini saling memperkuat satu sama lain.**

**Transaksi yang setengah jadi:** charge berhasil → insert order melempar error → ketangkep & diabaikan. Customer udah bayar tapi nggak punya order.

*Jembatan DB: kamu belajar `ROLLBACK` buat kejadian PERSIS kayak gini. Exception yang diabaikan itu transaksi yang lupa kamu rollback.*

**Yang diresepin OWASP 2025:** Tiap transaction yang melempar error harus di-**rollback** dan dicoba ulang. Selalu **"fail closed"** — tolak sebagai default kalau ada error. Kalau app atau user berperilaku mencurigakan, **naikin alert**.

**Error verbose sebagai reconnaissance:** stack trace dan error DB yang detail ditampilin ke user itu langkah pertama klasik di intrusi nyata: mereka ngasih path file, versi framework, dan struktur query ke penyerang. **Panduan OWASP tentang error handling ngasumsiin penyerang baca TIAP error yang kamu keluarin** — jadi production harus nggak ngungkapin apa pun soal internal.

**Bahaya spesifik PHP:** dengan `display_errors` **ON** di production, PHP nampilin warning langsung ke halaman — connection string, path, nama fungsi semuanya bocor. Myer & Southwell ngingetin buat nggak percaya perilaku default PHP: **konfigurasi aman itu pertahanan kelas satu, bukan pemikiran belakangan.**

### Mencegah logging & alerting failures

**YANG HARUS di-log (event relevan-keamanan):** login sukses & gagal, logout, lockout. Penolakan access-control & perubahan privilege. Kegagalan validasi input sisi-server. Aksi bernilai tinggi: pembayaran, pemberian role, penghapusan. **Siapa, apa, kapan, di mana** — dengan timestamp yang konsisten.

**JANGAN PERNAH di-log (mask/hash/buang):** Password, secret, API key, sertifikat. Session ID lengkap (hash kalau perlu). Nomor kartu/PII lengkap kecuali diizinin secara hukum. Source code atau detail stack internal di log yang menghadap user. Input mentah user yang belum disanitasi (→ log injection).

Contoh baris log yang bagus, terstruktur, aman dari injection:
```json
{"ts":"2026-06-30T09:14:02Z","event":"authn_login_fail","user":"maya","src_ip":"10.0.0.9","app":"finpay"}
```

**Dari log ke alert ke respon:** Collect (event terstruktur, disanitasi dari injection) → Centralise (kirim ke log store, misal ELK, dengan clock tersinkron) → Alert (aturan pada event kunci nge-page tim yang tepat, cepat) → Respond (playbook & rencana incident response jalan).

**Lindungi log-nya sendiri:** audit trail append-only/tamper-evident buat aksi bernilai tinggi. Batasin siapa yang bisa baca log (mereka nyimpen konteks sensitif). Sinkronkan waktu di semua server — forensik butuh satu jam. Jangan biarin kegagalan logging bikin app crash atau bocorin data. **Alert dengan bijak** (hindari fatigue): definisiin use case jelas soal apa yang layak alert; terlalu banyak false positive → alert beneran diabaikan; tiap alert butuh playbook biar beneran bisa ditindaklanjuti; **honeytoken** bikin intrusi memicu alarm lebih awal.

### Menangani kondisi luar biasa dengan aman: pola exception yang aman
**Sebelum — bocor & telen error:**
```php
try {
  $db->charge($order);
} catch (Exception $e) {
  echo $e->getMessage(); // bocor
}  // nggak ada log, nggak ada rollback
```
**Sesudah — fail closed & log:**
```php
try {
  $db->charge($order);
} catch (Throwable $e) {
  $db->rollBack();
  log_security($e); show_error();
}
```
**Tangkep secara luas** — pakai global handler/`Throwable` biar nggak ada yang lolos nggak tertangani. **Nggak ngungkapin apa-apa** — tampilin user pesan umum + reference ID; jangan pernah stack trace. **Rekam sepenuhnya** — log detail asli di sisi server buat kamu sendiri — inilah link ke A09. **Fail closed** — pas error, tolak dan rollback. Jangan pernah lanjut setengah jadi.

### Mengencengin konfigurasi PHP
```ini
display_errors = Off
display_startup_errors = Off
log_errors = On
error_log = /var/log/php/error.log
error_reporting = E_ALL
expose_php = Off
html_errors = Off
zend.exception_ignore_args = On

# log_errors jaga detail buat KAMU
# display_errors nyembunyiin dari MEREKA
```
`display_errors = Off` — nyetop stack trace, path, dan string DB nyampe ke user. `log_errors = On` — jaga detail itu di sisi server — bukti A09-mu. `expose_php = Off` — buang banner versi PHP yang jadi fingerprint penyerang. `zend.exception_ignore_args = On` — jaga secret yang lewat sebagai argumen tetap di luar stack trace.

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — kontennya semua teks dan blok kode.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **SOC** | Security Operations Center, tim yang mantau dan merespon alert keamanan |
| **ELK** | Elasticsearch, Logstash, Kibana — tumpukan tool populer buat nyimpen dan nganalisis log terpusat |
| **Honeytoken** | Data umpan palsu (misal kredensial dummy) yang ditaruh sengaja buat mancing alarm kalau ada yang nyoba pakai |
| **Append-only log** | Log yang cuma bisa DITAMBAH, nggak bisa diubah atau dihapus entrinya — nyegah penyerang ngerapihin jejak |
| **`error_reporting`** | Setting PHP yang nentuin level error mana yang dilaporkan/dicatat |
| **Alert fatigue** | Kondisi tim jadi cuek sama alert karena kebanyakan false positive |
| **Tamper-evident** | Sifat data/catatan yang bikin ketauan kalau ada yang berusaha mengubahnya |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Bandingkan Mode A (error terlalu verbose) dan Mode B (catch block kosong). Analisis: dari sudut pandang PENYERANG, mode mana yang lebih MENGUNTUNGKAN buat serangan tahap awal (reconnaissance), dan mode mana yang lebih MENGUNTUNGKAN buat mempertahankan akses jangka panjang (persistence)?
2. **(C4 – Analisis)** Log injection (CWE-117) dan SQL injection punya "bentuk" yang mirip — data nggak dipercaya masuk ke suatu sistem tanpa dinetralkan. Analisis: apa yang jadi "trust boundary" di masing-masing kasus, dan kenapa fix buat SQLi (prepared statement) TIDAK otomatis nutup log injection?
3. **(C5 – Evaluasi)** Sebuah tim FinPay sekarang nge-log SEMUA hal, termasuk full request body (yang kadang berisi nomor kartu kredit) ke file log biasa yang bisa diakses semua developer. Evaluasi pendekatan ini: dari sisi mana ini SALAH ARAH meskipun niatnya "biar nggak ada yang kelewat"? Kaitkan ke daftar "JANGAN PERNAH di-log" dan konsep CWE-532.
4. **(C5 – Evaluasi)** Nilai efektivitas kontrol "honeytoken" dibanding kontrol "structured logging + alert rules" buat kasus FinPay-style breach (intrusi 7 tahun nggak ketahuan). Mana yang lebih mungkin memotong "dwell time" (waktu penyerang bertahan) dari bertahun-tahun jadi berhari-hari, dan kenapa?
5. **(C6 – Cipta)** Rancang SATU baris log terstruktur (format JSON kayak contoh di slide 18) buat event baru: "seorang user gagal login 5 kali berturut-turut dalam 1 menit dari IP yang sama, dan akunnya kena auto-lockout." Sebutkan field apa aja yang perlu ada, field mana yang HARUS di-mask/hash, dan jelasin gimana event ini seharusnya jadi TRIGGER buat sebuah alert (bukan cuma tercatat diam di file log).

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W11 - Code Auditing OWASP IV]]
- [[W13 - Review II]]
- [[SecureProgramming - Review dan Glosari]]
