---
matkul: Secure Programming
minggu: 2
sks: 3
sumber: S2-Session Cookies FileUpload.pptx
tags: [kuliah/secure-programming, minggu/w02]
status: draft
diproses: 2026-09-04
---

# W02 — Session, Cookies & File Upload

## Ringkasan
> - Maya login ke website bank-nya, pindah tab, transfer duit, balik lagi — situsnya masih inget itu dia, padahal dia nggak pernah ngetik password lagi. Padahal **web itu nggak punya ingatan**. Jadi gimana caranya bank inget Maya?
> - **Session** = catatan kecil di SERVER (kayak nitip mantel ke tukang jaga). **Cookie** = tiket kecil yang dipegang BROWSER, isinya cuma nomor rak (session ID).
> - Jangan ketuker: session isinya data beneran dan aman dipercaya; cookie cuma bawa ID-nya doang dan **bisa diintip, disalin, dicuri** siapa aja yang megang browser itu (termasuk penyerang).
> - Flag cookie penting: **HttpOnly** (JS nggak bisa baca), **Secure** (cuma lewat HTTPS), **SameSite** (nggak dikirim ke situs lain).
> - `$_SESSION` itu **bisa dipercaya** (server yang isi). `$_GET`, `$_POST`, `$_COOKIE`, `$_FILES` itu **nggak bisa dipercaya** (user yang isi) — **jangan pernah** dipakai buat keputusan akses.
> - **Session hijacking** = penyerang nyolong session ID yang valid, jadi bisa nyamar jadi korban **tanpa perlu tau passwordnya**. Empat cara: sniffing, XSS, fixation, ID lemah/bocor.
> - **File upload** itu input dari user juga, tapi lebih berbahaya — bisa jadi script yang nyamar jadi gambar.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Session | Catatan data di server, dicari lewat kunci (session ID) |
| Cookie | Teks kecil `key=value` yang disimpan browser, dikirim otomatis tiap request |
| Session ID | Nomor unik yang menghubungkan cookie ke data session di server |
| HttpOnly | Flag cookie yang bikin JavaScript nggak bisa baca isinya |
| Secure (flag) | Flag cookie yang bikin dia cuma dikirim lewat HTTPS |
| SameSite | Flag cookie yang batasin pengiriman ke request dari situs lain |
| Session hijacking | Penyerang nyolong session ID valid buat nyamar jadi korban |
| Session fixation | Penyerang masang ID yang udah dia tau duluan, nunggu korban login pakai ID itu |
| `$_SESSION` | Array superglobal PHP, isinya dikontrol server — aman dipercaya |
| `$_COOKIE` / `$_FILES` | Array superglobal yang isinya dari user — jangan pernah dipercaya buat keputusan akses |

## Isi

### Cerita pembuka: Maya cuma mau tetep login
Maya buka website bank, login, cek saldo, buka tab baru, transfer duit, balik lagi — situsnya masih inget itu dia. Dia nggak pernah ngetik password lagi. Kedengeran simpel, kan? Tapi ini teka-tekinya: **web itu nggak punya ingatan.** Tiap request ke server diperlakukan kayak dari orang asing total. Jadi gimana caranya bank inget Maya sepanjang klik, tab, dan menit?

Kalau kamu udah belajar database, kamu udah punya modal: kamu tau gimana sistem nyimpen dan nyari record pakai kunci. **Sebuah session itu pada dasarnya catatan kecil yang disimpan di server, dicari lewat kunci.**

### Web itu pikun: HTTP nggak inget kamu
Tiap request itu berdiri sendiri. Server jawab, terus langsung lupa kamu pernah nanya.

```
Kamu: "Tunjukin profil gue"      Server: "Kamu siapa?"
Kamu: "Tunjukin keranjang gue"   Server: "Kamu siapa?"
Kamu: "Checkout"                 Server: "Kamu siapa?"
```

Solusinya: kasih user sesuatu buat dibawa yang membuktikan identitasnya. State harus ditambahin di atas HTTP. Dua mekanisme kerja bareng: **session** (ingatan yang disimpan di server) dan **cookie** (tiket kecil yang dipegang browser).

### Session itu penitipan mantel di teater

> [!info] Analogi
> Kamu nyerahin mantel ke petugas penitipan. Mereka simpan di rak bernomor (**server nyimpen datamu**). Mereka kasih kamu tiket kecil bernomor (**session ID**). Kamu cuma pegang tiketnya — bukan mantelnya. Buat ambil mantel balik, kamu tunjukin tiket. Mereka cari nomornya.

| Analoginya | Padanan di web |
| --- | --- |
| Mantel / barangmu | Data session di server |
| Rak bernomor | Tempat penyimpanan session (file, DB, memory) |
| Nomor tiket | Session ID |
| Nunjukin tiket | Browser ngirim ID di tiap request |
| Petugas nyari nomornya | Server mulihin state kamu |

**Alur hidup session:** Login (user submit kredensial valid, sekali) → Create (server bikin session ID unik + record kosong) → Hand off (ID dikirim ke browser, biasanya lewat cookie) → Reuse (browser ngirim balik ID di tiap request berikutnya) → Expire (idle timeout atau logout ngancurin session-nya).

Di PHP, `session_start()` bikin record-nya dan otomatis nyambungin ID-nya — tapi Pro PHP Security ngingetin: **setting default ini harus dikencengin dulu, jangan dipercaya apa adanya.**

### Cookie itu secarik catatan yang dibawa browser
Cookie adalah string kecil `key=value` yang server minta browser buat nyimpen. Browser ngirim balik otomatis tiap request ke situs itu. **Itu caranya session ID berpindah antara client dan server.** "Tetep login"-nya Maya itu cuma cookie yang bawa session ID-nya.

**Jangan ketuker session dan cookie:**

| Session | Cookie |
| --- | --- |
| Data hidup di SERVER | Data hidup di BROWSER |
| Isinya konten beneran (user, keranjang, role) | Biasanya cuma nyimpen session ID |
| User nggak pernah lihat isinya | User (dan penyerang) bisa ngintip isinya |

Karena cookie ada di sisi client, dia **bisa dibaca, disalin, atau dicuri**.

**Anatomi cookie dan flag yang melindunginya:**
```
Set-Cookie: PHPSESSID=8f4a...e1; HttpOnly; Secure; SameSite=Strict; Path=/
```
- **HttpOnly** — JavaScript nggak bisa baca cookie-nya — nge-blok pencurian lewat script XSS
- **Secure** — cookie cuma dikirim lewat HTTPS — nyetop penyadapan jaringan
- **SameSite** — nggak dikirim di request lintas-situs — batesin penyalahgunaan gaya CSRF
- **Expiry/Path** — ngatur seberapa lama dan di mana cookie itu berlaku

### Array: di mana sebenernya data session tinggal?
Kamu udah tau tabel database — baris dan kolom, dicari lewat kunci. Array PHP itu "sepupu" di memori dari ide itu, dan begitulah cara web nyimpen data session.

| Jenis Array | Cara nyari nilainya | Contoh |
| --- | --- | --- |
| Indexed | Berdasarkan posisi: 0, 1, 2… | `$cart[0]` |
| Associative | Berdasarkan kunci bernama — kayak baris yang dicari lewat kolom | `$user['name']` |
| Superglobal | Array khusus yang otomatis diisi PHP tiap request | `$_SESSION`, `$_COOKIE` |

```php
session_start();
// setelah cek password
$_SESSION['user_id'] = 42;
$_SESSION['role']    = 'customer';

// di halaman berikutnya, otomatis:
echo $_SESSION['user_id'];  // 42
```

**Nggak semua array bisa dipercaya.** Pertanyaannya: siapa yang ngisi array ini — server, atau user?

- **Dikontrol server** (`$_SESSION`) — cuma kodemu yang nulis ke sana. **Aman dipercaya** buat identitas dan role.
- **Dikontrol user** (`$_GET`, `$_POST`, `$_COOKIE`) — diisi dari request — penyerang bisa masukin apa aja. **JANGAN PERNAH** dipercaya buat keputusan akses.

Aturan emas Pro PHP Security: **"Filter input, escape output."**

### Session Hijacking: kalau session ID itu tiket ke akun Maya, gimana kalau tiketnya dicuri?

**Session Hijacking**: penyerang dapetin session ID valid dan makainya buat nyamar jadi korban — **tanpa pernah tau passwordnya**.

**Empat cara tiketnya dicuri:**
1. **Sniffing** — di wifi terbuka tanpa HTTPS, ID-nya jalan teks polos. Penyerang tinggal baca dari kabel/udara.
2. **XSS theft** — JavaScript yang disuntikin baca `document.cookie` dan ngirim ID-nya ke penyerang — bisa dicegah `HttpOnly`.
3. **Fixation** — penyerang masang ID yang dia tau duluan, nipu korban login pakai ID itu, terus dia pake ulang ID-nya.
4. **Prediction/leakage** — ID yang lemah/gampang ditebak, atau bocor di URL dan log, bisa dipake ulang penyerang.

**Anatomi bajakan wifi kafe:** Maya login → cookie `PHPSESSID` tersimpan → penyerang nguping (trafik nggak dienkripsi) → ID-nya kecopet persis → penyerang paste ID itu di browser-nya sendiri → server liat ID valid, ngasih akun Maya. **Nggak ada password yang dibobol.** Semuanya cuma nunggangin satu string yang dicuri.

**Studi kasus: Firesheep (2010).** Ekstensi browser bernama Firesheep dirilis buat buktiin sebuah poin: di wifi terbuka, dia otomatis nangkep cookie session dari situs yang cuma HTTPS di login doang, terus balik ke HTTP. Siapa aja tinggal klik nama di sidebar, langsung masuk akun media sosial orang itu. Ini maksa situs-situs besar pindah ke HTTPS penuh. Yang dicuri: session cookie. Vektornya: sniffing di HTTP nggak terenkripsi. Perbaikannya: flag `Secure` + HTTPS di mana-mana.

**Menilai risiko kayak analis:**

| Ancaman | Kemungkinan | Dampak | Kontrol utama |
| --- | --- | --- | --- |
| Sniffing di wifi terbuka | Tinggi | Tinggi | HTTPS + flag Secure |
| XSS nyolong cookie | Sedang | Tinggi | HttpOnly + escape output |
| Session fixation | Sedang | Tinggi | Regenerate ID pas login |
| Session ID gampang ditebak | Rendah | Tinggi | ID acak panjang (default PHP) |

Pola: dampak hampir selalu Tinggi karena hadiahnya akun penuh. Makanya defender nyerang kolom **kemungkinan**-nya.

### Toolkit pertahanan
- **HTTPS di mana-mana** — enkripsi semua trafik, bukan cuma halaman login
- **Secure + HttpOnly** — nyetop pencurian lewat jaringan dan lewat JavaScript
- **Regenerate ID** — panggil `session_regenerate_id()` pas login buat ngalahin fixation
- **Timeout pendek** — expire session yang idle, jadi ID curian punya jendela pemakaian kecil
- **Bind ke konteks** — kaitin session ke sinyal user-agent/IP; nggak cocok = putuskan
- **Logout beneran** — hancurin record di server, bukan cuma cookie-nya

```php
// 1. paksa cookie secure + http-only
session_set_cookie_params([
  'secure'   => true,
  'httponly' => true,
  'samesite' => 'Strict'
]);
session_start();

// 2. cegah fixation pas login
session_regenerate_id(true);

// 3. simpan identitas cuma di server
$_SESSION['user_id'] = $id;
```

### Satu hal lagi yang dibawa: file upload
Cookie dan session itu data yang browser bawa MASUK. File upload itu data yang user dorong KELUAR ke server-mu — dan pantes dicurigai sama besarnya.

Kenapa upload berbahaya: file itu input user, sama kayak `$_COOKIE`. File yang disamarkan (misal `shell.php` diganti nama jadi `shell.jpg`) bisa jalan di server. `$_FILES` itu array web lain yang **nggak boleh dipercaya mentah-mentah**.

Checklist upload aman: validasi tipe dan ukuran beneran (jangan percaya ekstensi), rename file, simpan di luar web root, jangan pernah biarin upload dieksekusi sebagai kode. Prinsipnya sama: **filter input, escape output**.

### Satu model mental yang ngiket semuanya
Semua materi hari ini itu satu pertanyaan yang diulang: **siapa yang bawa apa, dan bisa dipercaya nggak?**

- Server nyimpen kebenaran → data session ada di sini, terpercaya
- Browser nyimpen tiket → cookie cuma bawa session ID
- Array nyimpen field → `$_SESSION` terpercaya; `$_COOKIE`/`$_FILES` nggak
- Penyerang mau tiketnya → colong ID = jadi si user
- Defender ngelindungin tiketnya → HTTPS, flag, regenerate, expire

## Diagram & Visual
> [!warning] Deck ini nggak punya gambar non-dekoratif yang bisa diekstrak — kontennya semua teks, diagram alur, dan blok kode.

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Session store** | Tempat data session disimpan di server: file, database, atau memory |
| **`session_start()`** | Fungsi PHP yang bikin record session dan nyambungin session ID-nya |
| **`session_regenerate_id()`** | Fungsi PHP buat ganti session ID (dipanggil pas login, buat ngalahin fixation) |
| **Indexed array** | Array yang nilainya dicari lewat posisi angka (0, 1, 2, …) |
| **Associative array** | Array yang nilainya dicari lewat nama kunci, kayak kolom di tabel |
| **Superglobal** | Array bawaan PHP yang otomatis keisi tiap request (`$_SESSION`, `$_GET`, dll) |
| **Eavesdropping** | Menguping/membaca data yang lewat jaringan tanpa izin |
| **CSRF** | Serangan yang manfaatin session aktif korban buat ngirim aksi yang nggak dia maksud (disinggung lewat flag SameSite) |
| **Web root** | Folder utama yang bisa diakses langsung lewat URL server web |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Sebuah situs kampus nyimpen session ID di URL (`?sid=abc123`), pakai HTTP biasa, dan ID-nya nggak pernah berubah setelah login. Seorang mahasiswa share link jadwalnya di grup chat publik. Analisis: dari empat cara pencurian session (sniffing, XSS, fixation, prediction/leakage), mana yang PALING gampang dieksploitasi di skenario ini, dan kenapa?
2. **(C4 – Analisis)** Bandingkan flag `HttpOnly` dan `Secure` pada cookie. Kalau developer cuma sempet pasang SATU dari dua flag itu, ancaman mana yang tetep kebuka di tiap pilihan? Jelasin alasannya pakai konsep sniffing vs. XSS.
3. **(C5 – Evaluasi)** Seorang developer bilang: "$_SESSION dan $_COOKIE itu sama-sama nyimpen data user, jadi perlakuannya sama aja." Evaluasi pernyataan ini — di mana letak kesalahan pemahamannya, dan apa akibatnya kalau developer itu makai `$_COOKIE` buat nyimpen role user ("admin"/"customer")?
4. **(C5 – Evaluasi)** Nilai efektivitas kontrol "regenerate session ID pas login" dibanding kontrol "timeout pendek" buat ngelawan skenario Firesheep (sniffing wifi terbuka). Mana yang lebih relevan buat kasus itu spesifik, dan kenapa?
5. **(C6 – Cipta)** Rancang sebuah form upload avatar profil (deskripsikan alurnya, bukan kodenya) yang nerapin minimal 4 prinsip dari checklist upload aman di materi ini. Jelasin di tiap langkah, ancaman apa yang lagi kamu cegah.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_SecureProgramming]]
- [[W01 - Introduction to Web Application]]
- [[W03 - Web Form Processing and Security]]
- [[W04 - Web Form File Upload]]
- [[SecureProgramming - Review dan Glosari]]
