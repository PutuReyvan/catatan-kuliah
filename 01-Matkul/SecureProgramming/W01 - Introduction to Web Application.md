---
matkul: Secure Programming
minggu: 1
sks: 3
sumber: Introduction to Web Application.pptx
tags: [kuliah/secure-programming, minggu/w01]
status: draft
diproses: 2026-09-04
---

# W01 — Introduction to Web Application

## Ringkasan
> - Cerita pembuka: **Maya** bikin web app jam 9 pagi, jalan mulus di laptopnya. Jam 11 malam, orang asing udah login jadi admin — **tanpa pernah tahu passwordnya**. Sesi ini nyari tahu kenapa itu bisa kejadian.
> - Tiga babak: **HTTP & HTTPS** (percakapan browser↔server), **PHP** (server yang mulai "mikir" dan bikin keputusan), **Publish ke server** (dari laptop pribadi ke internet terbuka).
> - **HTTP itu "pelupa"**: tiap request diperlakukan berdiri sendiri, server nggak inget kamu abis nanya apa. Makanya butuh **session cookie** sebagai "gelang tangan" bukti identitas yang ditunjukin ulang tiap kali.
> - **HTTPS itu amplop tersegel**, HTTP itu kartu pos. HTTPS wajib, tapi **enkripsi doang nggak bikin logika aplikasimu aman**.
> - Aturan emas PHP: **jangan pernah percaya input dari client**. Semua yang masuk dari `$_GET`, `$_POST`, cookie, atau header itu dari orang asing yang bisa ngirim apa aja.
> - Publish ke server itu **memperluas medan serangan**, bukan cuma "selesai bikin". Localhost = kamu satu-satunya pengunjung; production = siapa aja, 24 jam, termasuk bot yang terus-terusan nyoba-nyoba.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| HTTP | Bahasa/aturan percakapan browser dan server; sifatnya "pelupa" (stateless) |
| HTTPS | HTTP yang dibungkus enkripsi (TLS); bukti server benar-benar itu servernya |
| Statelessness | Tiap request diperlakukan berdiri sendiri, server nggak nyimpen ingatan |
| Session cookie | "Gelang tangan" bukti identitas yang dikirim ulang tiap request |
| PHP | Bahasa pemrograman yang jalan di server, yang bikin keputusan sebelum hasil dikirim ke browser |
| Selection (if/else) | Percabangan kode; tempat semua keputusan keamanan (login, akses) diambil |
| Open redirect | Kerentanan waktu tujuan redirect diambil dari input user tanpa dicek |
| Localhost | Server yang cuma bisa diakses dari komputer sendiri |
| Production | Server yang beneran online dan bisa diakses siapa aja |
| Security misconfiguration | Bug bukan di kode, tapi di pengaturan yang lupa dikencengin |

## Isi

### Cerita pembuka: apa yang kejadian sama Maya?
Maya bikin web app kecil: orang bisa login dan ninggalin review. Jam 9 pagi semua lancar di laptopnya. Jam 11 malam, ada orang yang nggak pernah dia kenal, login sebagai **admin** — **tanpa pernah tau passwordnya**.

Buat ngerti gimana itu bisa kejadian, kamu harus paham dulu **obrolan antara browser dan server**: tiap request, tiap response, dan semua asumsi di baliknya.

### Babak 1 — HTTP & HTTPS: obrolan tersembunyi di balik tiap halaman

**Apa itu web app sebenernya?** Kamu udah tau aplikasi database — data masuk, diproses, data keluar. Web app itu ide yang sama, cuma **usernya adalah orang asing yang nyampe ke kamu lewat internet publik, pakai browser**.

Alurnya: **Browser** (minta halaman, jalanin HTML/JS) → **HTTP/HTTPS** (jalur transportasinya) → **Web Server + PHP** (nerima request, jalanin logika) → **Database** (nyimpen dan ngasih data).

**HTTP itu obrolan yang sopan tapi pelupa.** Tiap interaksi web itu sepasang: browser nanya (**request**), server jawab (**response**). Contoh request: `GET /reviews`, ada header `Cookie: session=abc123`. Contoh response: `HTTP/1.1 200 OK` plus isi halamannya.

- **Method**: `GET` (baca), `POST` (kirim), `PUT`, `DELETE`
- **Status code**: `2xx` sukses, `3xx` dialihkan, `4xx` salah kamu, `5xx` salah server

**HTTP lupa kamu begitu dia selesai jawab.** Ini yang disebut **statelessness** — tiap request itu berdiri sendiri, server nggak inget request sebelumnya. Terus gimana caranya dia tau kamu masih login?

Jawabannya: ada **token identitas** yang dikirim bareng tiap request — namanya **session cookie**.

> [!info] Analogi
> Session cookie itu kayak **gelang tangan di festival musik**. Sekali kamu beli tiket dan ditempelin gelang, kamu nggak perlu nunjukin tiket lagi tiap mau masuk panggung — cukup tunjukin gelangnya. Masalahnya: **siapa pun yang bisa nyopet gelang itu, bisa masuk pura-pura jadi kamu**. Itu inti dari tiga ancaman berikut:
> - **Eavesdropping** — nyolong dengan baca cookie dari koneksi nggak terenkripsi (nguping obrolan di kafe)
> - **Session hijacking** — muter ulang cookie curian buat nyamar jadi korban (pake gelang curian buat masuk)
> - **Session fixation** — nipu korban supaya makai token yang emang udah diketahui penyerang dari awal (nyelipin gelang palsu duluan)

**HTTPS: nyegel amplopnya.** HTTP ngirim semuanya sebagai teks yang bisa dibaca siapa aja — wifi kafe, router, ISP, semuanya bisa baca. HTTPS membungkus obrolan yang sama dengan enkripsi (TLS).

> [!info] Analogi
> **HTTP itu kartu pos** — siapa pun yang megang bisa baca isinya. **HTTPS itu amplop tersegel** — isinya terenkripsi, dan ada sertifikat yang membuktikan amplop itu beneran dari alamat yang benar. Tapi ingat prinsip dari Myer & Southwell: **enkripsi melindungi data selagi dia berjalan, tapi TIDAK bikin logika aplikasimu aman**. HTTPS itu perlu, tapi nggak cukup.

**Studi kasus: waktu "cuma HTTP" nyerahin akun ke orang asing.** Bertahun-tahun, banyak situs besar cuma ngenkripsi halaman login, terus balik ke HTTP polos setelahnya — cookie session-nya jalan telanjang. Tahun 2010, ada tool browser yang bikin ini tinggal klik: duduk di wifi kafe bareng, tangkep cookie yang lewat, login jadi siapa aja di deket situ — **tanpa butuh password**. Solusinya yang diadopsi seluruh industri: **HTTPS di mana-mana**, bukan cuma di halaman login.

### Babak 2 — Introduction to PHP: di sinilah server berhenti sekadar nerusin pesan, dan mulai bikin keputusan

**PHP jalan di server — dan itu mengubah model ancamannya.** Browser cuma nampilin hasil. **PHP yang menentukan hasil itu apa** — sebelum pernah keluar dari server.

```php
<?php
  // user adalah orang asing
  $name = $_GET['user'];
  echo "Welcome, " . $name;
?>
```

**Aturan emasnya: jangan pernah percaya input dari client.** Tiap nilai dari `$_GET`, `$_POST`, cookie, atau header itu dari orang asing yang bisa ngirim apa aja. Baris kode di atas nge-echo langsung — nah, dari situlah **XSS** (Cross-Site Scripting) mulai. Myer & Southwell bilang: **anggap semua input bersalah sampai terbukti valid**.

**If/else: tiap pengecekan keamanan itu keputusan.** *Selection* (percabangan) bikin server milih jalur. Autentikasi, otorisasi, validasi — semuanya cuma if-statement yang **nggak boleh kelewat**.

```php
if ($user->isLoggedIn()) {
    showDashboard();
} elseif ($attempts > 3) {
    lockAccount();
} else {
    redirect('/login');
}
```

Tiap cabang itu **gerbang**. Keputusan otorisasi hidup di sini — siapa yang boleh masuk, siapa yang nggak. **Pengecekan yang kelewat = pintu yang kebuka.** Kondisi yang hilang atau salah diam-diam ngasih akses. Ancamannya: **Broken Access Control** ada di puncak daftar OWASP — biasanya gara-gara `if` yang cacat.

**Loops: kuat buat kamu, kuat juga buat penyerang.** *Repetition* (`for`, `while`, `foreach`) memproses list — baris database, item di keranjang belanja. Kekuatan yang sama buat iterasi data bisa dibalik buat nyerang kamu:

```php
foreach ($reviews as $r) {
    echo htmlspecialchars($r['comment']);
    // escape tiap baris, tiap kali
}
```

Pola-nya: **loop sekali, render banyak** — jadi kesalahan escaping berulang di tiap item. Penyerang juga loop: **brute force** nyoba tebak password ribuan kali per menit. **Denial of service** — loop tanpa batas atas input user bisa nguras resource server.

**Redirect: ngirim browser ke tempat lain.** Redirect bilang ke browser "pergi ke sini aja". Di PHP cuma satu baris — dan baris itu nyentuh HTTP header, jadi sensitif secara keamanan.

```php
header('Location: /dashboard');
exit; // selalu berhenti sesudahnya
```

**Ancaman: open redirect.**

```php
header('Location: ' . $_GET['next']);
```

Tujuannya berasal dari user. Penyerang bikin link ke situs terpercayamu yang diam-diam ngarahin korban ke halaman phishing.

> [!info] Analogi
> Bayangin kamu nunjukin arah dengan bilang "ikutin plang ini", tapi kamu nggak pernah ngecek plangnya nunjuk ke mana — jadi siapa pun bisa ganti plang itu buat ngarahin orang ke jurang. Kontrolnya: **redirect cuma ke daftar tujuan internal yang udah fix (allow-list)**.

**Studi kasus: link terpercaya yang ternyata bukan.** Korban dapet link yang beneran diawali brand yang dia percaya: `trusted-bank.com/go?next=evil-site.com`. Lolos dari "apa domainnya kelihatan bener?" sekilas. Servernya lalu ngalihin browser ke klon buatan penyerang, yang nyolong login. Kepercayaannya **dipinjam** dari redirect situs asli.

### Babak 3 — Publishing to a Web Server: momen aplikasimu berhenti privat, dan seluruh internet bisa ngetok pintu

**Dari localhost ke internet publik.** Di laptopmu, kamu satu-satunya pengunjung. Publish berarti server web sungguhan (Apache/Nginx) sekarang **jawab request dari siapa aja, di mana aja, kapan aja**.

| | Localhost | Production |
| --- | --- | --- |
| Pengunjung | Satu, kamu doang | Siapa aja, 24/7 |
| Error tampil di layar | Nggak masalah | **Harus di-log, bukan ditampilin** |
| Penyerang beneran | Nggak ada | Bot nyoba-nyoba terus menerus |
| Kesalahan | Tetap privat | **Jadi celah/exposure** |

Pergeseran cara pikir: **"jalan di komputer gue" itu AWAL dari masalah keamanan, bukan akhir dari proyek.** Publish memperluas medan serangan (attack surface) secara besar-besaran.

**Lima langkah dan risikonya:**
1. **Prepare** — pisahin file, dependensi, config dari rahasia (secrets)
2. **Transfer** — upload lewat SFTP/SSH, **jangan pernah** FTP polos
3. **Configure** — setting web server + PHP, izin file
4. **Connect** — sambung ke database pakai akun **hak akses terbatas**
5. **Secure** — aktifin HTTPS, sembunyiin error, siapin logging

**Keamanan bukan langkah ke-5 — dia menjalar di sepanjang lima langkah itu.** Upload teks polos atau akun DB yang kelewat berkuasa bisa ngerusak semua langkah lainnya.

**Apa yang keekspos kalau kamu nggak hati-hati:**
- **Error verbose** — stack trace nunjukin path file, query, struktur DB ke siapa aja
- **Rahasia keekspos** — file config, `.env`, atau backup ketinggal di web root dan bisa didownload
- **Kredensial default** — panel admin dan database masih pakai password bawaan pabrik
- **Layanan terbuka** — port database atau SSH bisa dijangkau dari seluruh internet
- **Software usang** — server, PHP, atau library belum di-patch dengan exploit yang udah dikenal
- **Directory listing** — server dengan senang hati nunjukin semua file kalau nggak ada halaman index

**Studi kasus: breach yang mulai dari sebuah SETTING, bukan exploit canggih.** Tahun demi tahun, laporan keamanan nemuin penyebab utama yang sama untuk breach cloud: **bukan hacking yang pinter, tapi server, bucket, atau database yang ketinggalan bisa diakses publik secara default.** Flag debug yang lupa dimatiin, panel admin tanpa password, database yang nempel ke IP publik — **scanner otomatis nemuin ini dalam hitungan menit** setelah online. Itu sebabnya **Security Misconfiguration** ada di OWASP Top 10.

### Jadi, apa yang kejadian sama Maya?
1. **Situsnya jalan di HTTP polos** — cookie session-nya lewat begitu aja di wifi bersama, kebaca siapa pun di dekatnya.
2. **Input dipercaya mentah-mentah** — redirect dan echo langsung makan input user, buka pintu buat phishing dan XSS.
3. **Error debug dibiarin nyala** — stack trace nunjukin struktur database ke pengunjung pertama yang mancing error.

### Tabel Konsep → Ancaman → Kontrol pertama
| Konsep | Ancaman yang dibuka | Kontrol pertama |
| --- | --- | --- |
| HTTP statelessness + cookie | Session hijacking | HTTPS di mana-mana; cookie `Secure` |
| Transport HTTP polos | Eavesdropping | Enkripsi TLS (HTTPS) |
| Percaya input client di PHP | XSS / injection | Validasi input, escape output |
| Redirect dikontrol user | Open-redirect phishing | Allow-list tujuan internal aja |
| Loop atas input user | Brute force / DoS | Rate limiting, batas cek |
| Publish dengan pengaturan default | Security misconfiguration | Kencengin config, sembunyiin error |

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — semua kontennya berupa teks dan blok kode. Nggak ada yang perlu dibuka manual dari PPT.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Client** | Browser di sisi user; yang minta halaman dan nampilin hasil |
| **Request / Response** | Sepasang pesan: browser nanya (request), server jawab (response) |
| **Cookie** | Potongan data kecil yang disimpan browser dan dikirim ulang tiap request ke situs yang sama |
| **Session** | "Ingatan" yang disimpan di server, biasanya ditautkan lewat cookie |
| **TLS** | Teknologi enkripsi di balik HTTPS |
| **Statelessness** | Sifat HTTP yang nggak nyimpen ingatan antar request |
| **Superglobal ($_GET, $_POST, dll)** | Array bawaan PHP yang otomatis keisi data dari request — semuanya berasal dari user, jadi nggak boleh dipercaya begitu aja |
| **XSS (Cross-Site Scripting)** | Serangan yang nyisipin script jahat lewat input yang di-echo tanpa di-escape |
| **Open redirect** | Kerentanan waktu tujuan pengalihan halaman diambil langsung dari input user |
| **Attack surface** | Total "pintu" yang bisa dicoba diserang — makin banyak fitur/akses publik, makin luas |
| **Security misconfiguration** | Kerentanan yang muncul karena pengaturan (bukan kode) yang nggak dikencengin |

## Quiz Pemahaman
Level Bloom C4 ke atas (Analisis, Evaluasi, Cipta) — nggak ada soal hafalan definisi doang.

1. **(C4 – Analisis)** Maya nambahin fitur baru: user bisa nge-follow user lain lewat link `follow.php?target_id=55`. Analisis: bagian mana dari cerita W01 (statelessness, aturan emas PHP, atau attack surface publishing) yang paling relevan buat ngejelasin kenapa fitur ini bisa disalahgunakan, dan kenapa?
2. **(C4 – Analisis)** Bandingkan skenario "session cookie dicuri lewat wifi kafe" (HTTP) dengan "session cookie dicuri lewat XSS yang nge-echo input mentah". Apa persamaan akar masalahnya, dan di titik mana masing-masing seharusnya dicegah (transport vs. output)?
3. **(C5 – Evaluasi)** Sebuah tim bilang: "Aplikasi kita udah aman karena udah pakai HTTPS penuh." Evaluasi klaim ini pakai prinsip dari Myer & Southwell yang disebut di slide 9. Apa yang masih bisa salah meskipun HTTPS udah aktif 100%?
4. **(C5 – Evaluasi)** Dari lima langkah publishing (Prepare, Transfer, Configure, Connect, Secure), mana yang menurutmu paling sering diremehkan tim developer pemula, dan kenapa kegagalan di langkah itu bisa "membatalkan" empat langkah lainnya?
5. **(C6 – Cipta)** Rancang sebuah checklist pre-launch singkat (5 poin) buat tim Maya, supaya kejadian di cerita pembuka nggak terulang lagi — checklist itu harus nyambungin minimal satu poin ke tiap babak (HTTP/HTTPS, PHP, Publishing).

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W02 - Session, Cookies & File Upload]]
- [[SecureProgramming - Review dan Glosari]]
