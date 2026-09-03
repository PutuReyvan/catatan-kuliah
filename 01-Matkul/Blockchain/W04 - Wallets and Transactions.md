---
matkul: Blockchain
minggu: 4
sks: 2
sumber: Wallets and Transactions.pptx
tags: [kuliah/blockchain, minggu/w04]
status: draft
diproses: 2026-09-03
---

# W04 — Wallets and Transactions

## Ringkasan
> - **Salah kaprah terbesar: wallet TIDAK menyimpan ether.** Wallet cuma nyimpen *kunci*, kayak gantungan kunci. Ether-nya ada di blockchain.
> - Dua jenis wallet: **nondeterministic (JBOK)** — tiap kunci acak sendiri-sendiri; dan **deterministic (seeded)** — semua kunci diturunkan dari satu master seed. Bentuk paling canggih: **HD wallet (BIP-32/BIP-44)**, struktur pohon.
> - **BIP-39** ngubah entropy jadi 12–24 kata mnemonic biar backup-nya gak salah ketik. Kata-kata itu lalu di-stretch pakai PBKDF2 jadi seed 512-bit.
> - **Transaksi adalah satu-satunya hal yang bisa mengubah state Ethereum.** Contract gak jalan sendiri; semuanya berawal dari transaksi EOA.
> - Field transaksi: **nonce, gasPrice, gasLimit, recipient, value, data, v/r/s**. Nonce mencegah dua hal: urutan kacau dan replay attack.
> - **`tx.origin` vs `msg.sender`**: origin itu EOA paling awal (tetap sepanjang call chain), sender itu pemanggil langsung (berubah tiap hop).

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Wallet | Aplikasi yang mengelola kunci dan alamat, melacak saldo, serta membuat dan menandatangani transaksi |
| Nondeterministic (JBOK) wallet | "Just a Bunch of Keys" — tiap kunci dibuat dari angka acak terpisah, gak saling berhubungan |
| Deterministic (seeded) wallet | Semua private key diturunkan dari satu master key alias **seed** |
| HD wallet | Hierarchical Deterministic, kunci diturunkan dalam struktur pohon (BIP-32/BIP-44) |
| BIP | Bitcoin Improvement Proposal |
| Mnemonic (BIP-39) | Deretan kata yang, kalau urutannya benar, bisa merekonstruksi private key |
| PBKDF2 | Key-stretching function yang mengubah mnemonic + salt jadi seed 512-bit |
| Salt | Input tambahan yang bikin lookup table brute-force jadi susah dibangun; di BIP-39 juga jadi slot passphrase |
| EOA | Externally Owned Account, akun yang dikendalikan private key manusia |
| Nonce | Nomor urut transaksi dari satu alamat; mencegah urutan kacau dan replay |
| Gas | "Bahan bakar" Ethereum; mata uang virtual terpisah dari ether, punya kurs sendiri |
| gasPrice | Berapa wei yang mau dibayar pengirim per unit gas |
| gasLimit | Maksimum gas yang mau dibeli pengirim untuk transaksi itu |
| v, r, s | Tiga komponen digital signature ECDSA dari EOA pengirim |
| `tx.origin` | Alamat EOA yang mengawali transaksi; konstan sepanjang call chain |
| `msg.sender` | Alamat pemanggil pada call saat ini; berubah tiap hop |

## Isi

### Wallet itu gantungan kunci, bukan dompet
Di level tinggi, wallet adalah aplikasi software yang jadi antarmuka utama user ke Ethereum. Wallet mengontrol akses ke uang user: mengelola kunci dan alamat, melacak saldo, serta membuat dan menandatangani transaksi.

Slide secara khusus menandai satu **common misconception**:

> Orang mengira wallet Ethereum berisi ether atau token. Padahal, secara ketat, **wallet cuma menyimpan kunci, seperti gantungan kunci**. Ether dan token-nya tercatat di blockchain Ethereum. User mengendalikan token di jaringan dengan cara **menandatangani transaksi** memakai kunci di wallet-nya.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi: wallet itu bukan dompet, tapi **kunci brankas di bank**. Uangnya gak pernah ada di kantongmu — uangnya di brankas (blockchain), dan yang kamu bawa cuma kuncinya. Konsekuensi praktisnya: HP-mu hilang tapi seed-nya masih kamu ingat → uangmu aman, tinggal bikin kunci baru dari seed. Sebaliknya, seed-mu bocor → uangmu hilang, walaupun HP-nya masih di tanganmu.

### Dua jenis wallet
Wallet dibedakan berdasarkan **apakah kunci-kunci di dalamnya saling berhubungan atau tidak**.

**Nondeterministic wallet** — tiap kunci di-generate secara independen dari angka acak yang berbeda. Kunci-kuncinya gak saling berhubungan. Jenis ini juga dikenal sebagai **JBOK wallet**, dari frasa *"Just a Bunch of Keys"*.

**Deterministic wallet** — semua kunci diturunkan dari satu master key yang disebut **seed**. Semua kunci saling berhubungan, dan semuanya bisa di-generate ulang kalau kamu punya seed aslinya.

Detail deterministic wallet menurut slide:
- Seed adalah angka acak yang dikombinasikan dengan data lain (misal nomor indeks atau *chain code*) untuk membuat private key.
- **Seed-nya sendiri cukup untuk memulihkan semua kunci turunan.**
- Satu kali backup saat wallet dibuat sudah mengamankan semua dana dan smart contract.
- Karena itu, **keamanan seed itu krusial** — dia memberi akses ke seluruh wallet.

> [!info] Konteks tambahan (bukan dari slide)
> Bedanya kayak **setumpuk kunci lepas vs satu kunci induk**. JBOK: tiap kunci harus di-backup satu-satu; lupa satu, hilang satu brankas. Deterministic: cukup backup satu "resep" (seed), dan dari resep itu semua kunci bisa dicetak ulang kapan pun.

### HD wallet (BIP-32/BIP-44)
Bentuk paling canggih dari deterministic wallet adalah **hierarchical deterministic (HD) wallet**, didefinisikan oleh standar Bitcoin **BIP-32**. (BIP = Bitcoin Improvement Proposal.)

HD wallet menurunkan kunci dalam **struktur pohon**: satu parent key bisa menurunkan deretan child key, tiap child key bisa menurunkan deretan grandchild key, dan seterusnya.

Keunggulannya dibanding deterministic wallet biasa:
- **Pemisahan tanggung jawab** — cabang khusus untuk menerima pembayaran masuk, cabang berbeda untuk menerima kembalian dari pembayaran keluar.
- **Konfigurasi korporat** — cabang bisa dialokasikan per departemen, anak perusahaan, fungsi tertentu, atau kategori akuntansi.
- **Public key tanpa private key** — user bisa membuat deretan public key **tanpa akses ke private key pasangannya**. Ini memungkinkan HD wallet dipakai di server yang tidak aman, atau dalam mode *watch-only* / *receive-only*, tanpa membawa private key yang bisa membelanjakan dana.

### BIP-39: seed dan mnemonic code
Salah satu cara meng-encode private key untuk backup dan pemulihan yang aman adalah memakai **deretan kata** yang, kalau digabung dalam urutan yang benar, bisa merekonstruksi private key secara unik. Ini disebut **mnemonic**, dan pendekatannya distandarkan oleh **BIP-39**.

Contoh mnemonic 12 kata dari slide:

```
wolf juice proud gown wool unfair wall cliff insect more detail hub
```

Manfaatnya: user bisa menyimpan backup kunci **tanpa rawan salah ketik**.

Proses generate mnemonic menurut BIP-39, enam langkah:

1. Buat sequence acak kriptografis **S** sepanjang 128–256 bit.
2. Buat **checksum** dari S: ambil `panjang-S ÷ 32` bit pertama dari hash SHA-256 milik S.
3. Tambahkan checksum itu di **ujung** sequence acak S.
4. Bagi gabungan sequence-and-checksum jadi potongan **11 bit**.
5. Petakan tiap nilai 11-bit ke satu kata dari kamus **2.048 kata** yang sudah ditentukan.
6. Susun mnemonic code dari deretan kata itu, **urutannya dipertahankan**.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa 11 bit dan 2.048 kata: karena 2^11 = 2.048. Jadi tiap kata persis mewakili 11 bit. 12 kata × 11 bit = 132 bit = 128 bit entropy + 4 bit checksum. Checksum inilah yang bikin wallet bisa bilang "seed phrase kamu salah" begitu kamu salah ngetik satu kata — bukan karena dia tau seed-mu, tapi karena angkanya gak lolos cek.

Setelah mnemonic terbentuk, prosesnya belum selesai. Mnemonic mewakili entropy sepanjang 128–256 bit. Entropy itu lalu dipakai untuk menurunkan seed yang lebih panjang (**512-bit**) lewat key-stretching function **PBKDF2**. Seed hasilnya itulah yang dipakai membangun deterministic wallet dan menurunkan kunci-kuncinya.

PBKDF2 menerima dua parameter: **mnemonic** dan **salt**. Fungsi salt di key-stretching pada umumnya adalah **mempersulit pembuatan lookup table** yang bisa dipakai brute-force. Tapi di standar BIP-39, salt punya fungsi tambahan: dia **memungkinkan penambahan passphrase** yang berfungsi sebagai faktor keamanan ekstra untuk melindungi seed.

### Transaksi: satu-satunya pemicu perubahan state
Transaksi adalah **pesan bertanda tangan** yang berasal dari **externally owned account (EOA)**, dikirim lewat jaringan Ethereum, dan dicatat di blockchain.

Cara lain melihatnya: transaksi adalah **satu-satunya hal yang bisa memicu perubahan state, atau menyebabkan contract dieksekusi di EVM**. Ethereum adalah state machine singleton global, dan transaksilah yang bikin state machine itu berpindah state.

Slide menegaskan konsekuensinya dengan kalimat yang layak dihafal:

> **Contract gak jalan sendiri. Ethereum gak berjalan otonom. Semuanya dimulai dari sebuah transaksi.**

### Struktur transaksi
Transaksi adalah pesan biner terserialisasi. Perlu dicatat: tiap client dan aplikasi menyimpan transaksi di memori dengan struktur data internalnya sendiri, mungkin ditambahi metadata yang sebenarnya gak ada di transaksi jaringan. **Bentuk standar satu-satunya adalah network-serialization-nya.**

Isinya:

| Field | Isi |
| --- | --- |
| **Nonce** | Nomor urut yang diterbitkan EOA pengirim, dipakai mencegah replay pesan |
| **Gas price** | Jumlah ether (dalam wei) yang bersedia dibayar pengirim per unit gas |
| **Gas limit** | Maksimum gas yang bersedia dibeli pengirim untuk transaksi ini |
| **Recipient** | Alamat Ethereum tujuan |
| **Value** | Jumlah ether (dalam wei) yang dikirim ke tujuan |
| **Data** | Payload data biner dengan panjang variabel |
| **v, r, s** | Tiga komponen digital signature ECDSA dari EOA pengirim |

### Nonce: kenapa dia ada
Nonce terikat pada alamat pengirim dan menandakan **jumlah transaksi dari alamat itu**. Nonce **tidak disimpan eksplisit di blockchain**, tapi dihitung secara dinamis dengan menghitung transaksi yang sudah terkonfirmasi.

Nonce krusial untuk dua hal, dan slide kasih contoh untuk keduanya.

**1. Menetapkan urutan pemrosesan.** Bayangin kamu melakukan dua transaksi: 6 ether (yang penting) dan 8 ether, sementara saldomu 10 ether. Ngirim transaksi 6 ether duluan seharusnya bikin transaksi itu yang diproses duluan.
- *Tanpa nonce*: urutan pemrosesan acak — bisa jadi transaksi 8 ether yang diproses duluan, dan yang 6 ether gagal karena saldo keburu habis.
- *Dengan nonce*: transaksi diproses sesuai urutan nilai nonce-nya.

**2. Mencegah duplikasi (replay attack).** Bayangin saldomu 100 ether, dan kamu mengirim 2 ether untuk beli sesuatu.
- *Tanpa nonce*: transaksi itu bisa diputar ulang terus-menerus dan menguras akun. 49 transaksi lagi, saldomu jadi 0.
- *Dengan nonce*: tiap transaksi jadi unik. Begitu satu transaksi terverifikasi, transaksi dengan nonce yang sama **gak akan diproses lagi**.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi nonce: kayak **nomor urut antrean bank yang dicetak di slip setoranmu**. Teller memproses slip sesuai nomor urut, jadi urutannya gak bisa kebolak. Dan karena tiap nomor cuma dilayani sekali, slip yang difotokopi orang jahat bakal ditolak — "nomor 47 sudah dipanggil".

### Gas
**Gas adalah bahan bakar Ethereum.** Poin pentingnya: **gas bukan ether** — dia mata uang virtual terpisah dengan kurs sendiri terhadap ether.

Ethereum memakai gas untuk mengontrol jumlah sumber daya yang boleh dipakai satu transaksi, karena transaksi itu akan diproses di ribuan komputer di seluruh dunia. Memisahkan gas dari ether **melindungi dari volatilitas nilai ether**. Gas mengatur biaya untuk sumber daya komputasi, memori, dan penyimpanan.

Field `gasPrice` di transaksi membuat pengirim bisa menetapkan harga gas-nya sendiri. Gas price diukur dalam **wei per unit gas** — misalnya 3 gwei = 3 miliar wei.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi gas vs ether: kayak **liter bensin vs rupiah**. Jarak Jakarta–Bandung selalu butuh sekitar jumlah liter yang sama (itu **gas**, ditentukan oleh berat kerjanya), tapi harga per liternya naik-turun (itu **gasPrice**, ditentukan oleh keramaian jaringan). Total ongkos = liter × harga per liter. `gasLimit` itu ukuran tangki: kalau kekecilan, kamu mogok di tengah jalan — dan di Ethereum, bensin yang terlanjur kepakai tetap hangus.

### Recipient: hati-hati, gak ada validasi
Penerima transaksi ditulis di field `to`, berisi alamat Ethereum 20 byte. Alamatnya bisa EOA atau contract address.

Yang berbahaya: **Ethereum gak melakukan validasi lebih lanjut atas field ini. Nilai 20 byte apa pun dianggap valid.** Kalau nilai 20 byte itu berkorespondensi dengan alamat yang gak punya private key, atau gak punya contract, transaksinya **tetap valid**. Ethereum gak punya cara untuk tau apakah suatu alamat diturunkan dengan benar dari sebuah public key.

Akibatnya: mengirim transaksi ke alamat yang salah kemungkinan besar akan **membakar ether yang dikirim**, membuatnya tidak bisa diakses selamanya — karena sebagian besar alamat gak punya private key yang diketahui, sehingga gak ada signature yang bisa dibuat untuk membelanjakannya.

### Value dan data: empat kombinasi
"Payload" utama transaksi ada di dua field: **value** dan **data**. Transaksi bisa punya keduanya, cuma value, cuma data, atau gak dua-duanya — **keempat kombinasinya valid**.

| Kombinasi | Artinya |
| --- | --- |
| Cuma **value** | Sebuah pembayaran (*payment*) |
| Cuma **data** | Sebuah pemanggilan contract (*invocation*) |
| **Value + data** | Pembayaran sekaligus pemanggilan |
| **Gak dua-duanya** | Kemungkinan besar cuma buang-buang gas — tapi tetap mungkin dilakukan |

### `tx.origin` vs `msg.sender`
Ini bagian yang paling sering keluar di soal, dan paling sering ketuker.

**`tx.origin`** mengidentifikasi **pengirim asli** sebuah transaksi. Dia menunjuk ke external account yang mengawali transaksi, dan **tetap konstan** sepanjang interaksi smart contract berikutnya (seluruh call chain). Kalau transaksi dimulai lewat wallet MetaMask, alamat wallet MetaMask user itulah yang tersimpan di `tx.origin`, dan alamat ini gak berubah walaupun transaksinya melewati banyak contract. Konsistensi ini penting untuk melacak pengirim awal.

**`msg.sender`** mengidentifikasi **pengirim call saat ini**. Variabel ini dinamis dan bisa berubah sepanjang proses transaksi. Kalau transaksi melewati beberapa smart contract, nilai `msg.sender` berubah jadi alamat contract terakhir di call chain. Contohnya: kalau Contract A memanggil Contract B, maka `msg.sender` di dalam Contract B adalah **Contract A**.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi: kamu titip surat ke satpam, satpam nyerahin ke resepsionis, resepsionis naruh di meja bos.
> - `tx.origin` = **kamu**, dari sudut pandang siapa pun di rantai itu. Gak berubah.
> - `msg.sender` dari sudut pandang bos = **resepsionis**, bukan kamu.
>
> Ini juga alasan kenapa pakai `tx.origin` buat access control itu berbahaya: kalau kamu ketipu ngirim transaksi ke contract jahat, contract jahat itu bisa memanggil contract-mu, dan `tx.origin` di sana **tetap alamatmu** — jadi cek "hanya pemilik" lolos padahal yang nyuruh bukan kamu. Cek yang benar pakai `msg.sender`. Konsep ini balik lagi di [[W08 - Smart Contract Pitfalls]].

## Diagram & Visual
- **Slide 6 — struktur pohon HD wallet (parent → child → grandchild key)**
  ![[99-Assets/Blockchain/W04-slide06.png]]
- **Slide 10 — alur BIP-39: entropy → checksum → potongan 11 bit → kata**
  ![[99-Assets/Blockchain/W04-slide10.png]]
- **Slide 12 — alur mnemonic + salt → PBKDF2 → seed 512-bit**
  ![[99-Assets/Blockchain/W04-slide12.png]]
- **Slide 13 — tampilan Mnemonic Code Converter (iancoleman.io/bip39)**
  ![[99-Assets/Blockchain/W04-slide13.png]]
- **Slide 15 — struktur transaksi Ethereum**
  ![[99-Assets/Blockchain/W04-slide15.png]]
- **Slide 22 — ilustrasi kombinasi value dan data pada transaksi**
  ![[99-Assets/Blockchain/W04-slide22.png]]
- **Slide 26 — kode EntryContract.sol dan UnderlyingContract.sol untuk demo tx.origin vs msg.sender**
  ![[99-Assets/Blockchain/W04-slide26.png]]
  ![[99-Assets/Blockchain/W04-slide26b.png]]

> [!warning] Slide 6, 10, dan 12 di PPT aslinya cuma judul + diagram tanpa teks. Penjelasan di note ini diambil dari slide teks di sekitarnya. Kode di slide 26 juga berupa gambar, bukan teks — gak bisa di-copy dari note ini, buka gambarnya.

## Rumus / Sintaks
Perbedaan inti yang harus dihafal:

```solidity
// EOA -> Contract A -> Contract B

// Di dalam Contract B:
tx.origin    // alamat EOA yang mengawali transaksi (TETAP)
msg.sender   // alamat Contract A (BERUBAH tiap hop)
```

Contoh mnemonic BIP-39 (12 kata):

```
wolf juice proud gown wool unfair wall cliff insect more detail hub
```

## Hands-on
1. **Mnemonic code converter** (slide 13) — generate mnemonic, seed, dan extended private key di `https://iancoleman.io/bip39/`
2. **Membedakan transaction origin** (slide 25–26):
   - Buat dua contract, `EntryContract` dan `UnderlyingContract`
   - Buat method bernama `printTxOriginAndMsgSender` untuk melihat masing-masing alamat
   - Panggil `UnderlyingContract` dari `EntryContract`
   - Deploy keduanya ke Remix IDE
   - Amati perbedaan nilai alamat yang keluar dari tiap contract
   - Referensi dari speaker notes dosen: `https://dev.to/fassko/understanding-txorigin-and-msgsender-in-solidity-l9o`

## Pertanyaan Terbuka
- Slide 8 punya kesalahan cetak: dia bilang "here's a seed in hexadecimal form" tapi yang ditampilkan **kata-kata mnemonic yang sama persis** dengan bentuk 12-mnemonic di bawahnya. Bentuk hex aslinya gak pernah ditampilkan.
- BIP-44 disebut di judul slide tapi **isinya gak pernah dijelaskan** — cuma BIP-32 dan BIP-39 yang dibahas. Padahal BIP-44 itu yang mendefinisikan struktur path `m/44'/60'/0'/0`.
- Digital signature (v, r, s / ECDSA) cuma disebut sebagai field, gak dijelaskan cara kerjanya. Ini "asymmetric-key algorithm" yang digantung dari [[W01 - Introduction to Blockchain Technology]] dan sampai sini belum kebayar juga.
- Konsekuensi keamanan `tx.origin` (phishing lewat contract perantara) belum dibahas di sini — kemungkinan besar muncul di [[W08 - Smart Contract Pitfalls]].

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Blockchain]]
- [[W02 - The Fundamentals of Ethereum]]
- [[W05 - Solidity Development]]
- [[Blockchain - Review dan Glosari]]
