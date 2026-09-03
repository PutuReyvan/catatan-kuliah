---
matkul: Blockchain
sks: 2
sumber: rangkuman dari W01-W10
tags: [kuliah/blockchain, ujian]
status: draft
diproses: 2026-09-03
---

# Blockchain Fundamental — Review dan Glosari

Rangkuman lintas pertemuan W01–W10. Dibaca sebelum ujian, bukan pengganti note pertemuan.

---

## Bagian 1 — Review Cepat

### Yang paling mungkin keluar di ujian

Ini bukan bocoran, cuma pembacaan atas apa yang **diulang-ulang** dosen di seluruh deck.

**1. Lima risiko keamanan smart contract + mitigasinya.** Ini pasangan yang paling terstruktur di seluruh matkul. Kalau cuma sempat hafal satu tabel, hafal yang ini:

| Risiko | Inti masalahnya | Mitigasi |
| --- | --- | --- |
| **Reentrancy** | Contract kirim ether ke alamat tak dikenal → fallback function jahat manggil balik sebelum state ter-update | (1) pakai `transfer()` yang cuma kasih 2300 gas, (2) **ubah state dulu, kirim ether belakangan**, (3) pakai mutex |
| **Arithmetic Overflow/Underflow** | Tipe data ukuran tetap; angka di luar rentang **berputar** (`uint8` 0−1 → 255) | Pakai library matematika seperti **SafeMath** OpenZeppelin. Pembagian dikecualikan (gak bisa over/underflow, dan EVM revert saat bagi 0) |
| **Insecure Randomness** | EVM deterministik supaya bisa diverifikasi → keacakan sejati mustahil. `block.number`/blockhash bisa diprediksi user dan ditahan miner | Pakai layanan pihak ketiga terverifikasi: **ChainLink VRF**, **VeeDo VDF** (hitung off-chain, bukti on-chain) |
| **Incorrect Access Control** | Fungsi kritis lupa dikasih modifier, salah nama modifier, atau peran terlalu berkuasa | `onlyOwner`/`hasRole` dari OpenZeppelin, prinsip **least privilege**, buat fungsi `internal` bila perlu |
| **Unchecked External Calls** | `call`/`callcode`/`delegatecall`/`send` gak melempar exception, cuma return `false` → eksekusi lanjut seolah sukses | (1) `transfer()` bukan `send()`, (2) cek return value, (3) **withdrawal pattern**, (4) event logging, (5) perhatikan gas limit |

**2. `msg.sender` vs `tx.origin`.** Paling sering ketuker, paling gampang dijadikan soal.

| | `tx.origin` | `msg.sender` |
| --- | --- | --- |
| Menunjuk ke | EOA yang **mengawali** transaksi | Pemanggil **call saat ini** |
| Sepanjang call chain | **Tetap** | **Berubah** tiap hop |
| Kalau A → B, di dalam B isinya | EOA aslinya | Contract A |

**3. Lima pilar defensive programming.** Minimalism/Simplicity, Code Reuse, Code Quality, Readability/Auditability, Test Coverage. Kutipan yang paling mungkin diminta: *"complexity is the enemy of security"*.

**4. Severity = Likelihood × Impact**, dan **tiga skala severity yang berbeda**:

| Firma | Skalanya |
| --- | --- |
| **Trail of Bits** | Informational → Undetermined → Low → Medium → High |
| **ConsenSys** | Minor → Medium → Major → Critical |
| **Code4rena** | QA → 2 Med → 3 High |

> Jebakan: di skala **Likelihood** Trail of Bits, **"Low" berarti kesulitan eksploitasinya rendah** — jadi justru gampang diserang dan lebih berbahaya. Bukan "kecil kemungkinannya".

**5. Empat standar ERC:**

| Standar | Jenis | Ciri |
| --- | --- | --- |
| **ERC-20** | Fungible | 6 fungsi wajib: `totalSupply`, `balanceOf`, `transfer`, `transferFrom`, `approve`, `allowance` |
| **ERC-721** | Non-fungible (NFT) | Tiap token unik, punya identifier unik + metadata |
| **ERC-1155** | Multi token | Fungible + non-fungible dalam **satu contract**, batch transfer, bisa semi-fungible |
| **ERC-4626** | Tokenized vault | API standar untuk yield-bearing vault berbasis satu ERC-20 |

**6. Tujuh komponen laporan audit:** Executive Summary, Severity Criteria, Scope of Audit, Vulnerabilities Identified, Recommendations, Gas Optimization, Conclusion and Summary.

**7. Lima sifat hash function:** Avalanche effect, Deterministic, Fast, Unique, Irreversible.

**8. Tujuh field transaksi Ethereum:** nonce, gas price, gas limit, recipient, value, data, dan v/r/s.

**9. `receive()` vs `fallback()`.** Dua-duanya dipanggil kalau **tidak ada signature fungsi yang cocok**; pembedanya calldata:

| Fungsi | Kapan dipanggil |
| --- | --- |
| **`receive()`** | calldata **kosong** |
| **`fallback()`** | calldata **tidak kosong** |

Plus: modifier **`payable`** wajib supaya fungsi bisa menerima ether — tanpa itu transaksinya ditolak. Dan ingat sambungannya: fungsi inilah yang jadi **pintu masuk reentrancy** di W08.

**10. EOA vs contract account.** EOA **punya private key** dan bisa memulai transaksi. Contract account **punya kode tapi tidak punya private key**, dikendalikan logika kodenya sendiri, dan **cuma bisa bereaksi** — itu sebabnya "contract gak jalan sendiri".

---

### Ringkasan satu paragraf per pertemuan

**[[W01 - Introduction to Blockchain Technology]]** — Blockchain adalah distributed ledger yang mencatat transaksi di banyak komputer dan gak bisa diubah retroaktif. Isinya node, block, chain. Yang menjaganya: hash function SHA-256 (dengan lima sifatnya) dan consensus mechanism (PoW = pecahkan teka-teki; PoS = stake koin sebagai jaminan). Mining = brute-force nonce sampai hash jatuh di bawah target. Trilemma: security, scalability, decentralization — gak bisa maksimal bertiga. Turunannya DeFi.

**[[W02 - The Fundamentals of Ethereum]]** — Ethereum = "world computer": state machine deterministik dengan singleton state, dijalankan EVM. Beda dari Bitcoin: Bitcoin jaringan pembayaran dengan script terbatas, Ethereum blockchain general-purpose yang bisa diprogram; ether cuma utility currency untuk membayar komputasi. Empat tahap: Frontier, Homestead, Metropolis, Serenity, plus hard fork di antaranya (paling penting: **DAO fork** yang memecah Ethereum dan Ethereum Classic). Aplikasinya berbentuk DApp (smart contract + frontend) dan DAO (aturan di kode, bukan di orang).

**[[W03 - Ethereum Basics]]** — Ethereum itu sistemnya, ether mata uangnya; satuan terkecil **wei**, 1 ether = 10^18 wei, dan nilai selalu disimpan internal sebagai unsigned integer dalam wei. Denominasi punya nama tokoh: Babbage, Lovelace, Shannon, Szabo, Finney, Grand. Wallet = gerbang ke Ethereum, dan karena aplikasinya **harus punya akses ke private key**, cuma pakai dari sumber tepercaya (MetaMask, Jaxx, MyEtherWallet, Emerald). Hilang private key = hilang dana selamanya; jangan improvisasi keamanan, backup mnemonic pakai pena dan kertas. Dua jenis akun: **EOA** (punya private key) dan **contract account** (punya kode, gak punya private key, dikendalikan logikanya sendiri). Contract pertama: `Faucet.sol` di Remix. **`receive()`** dipanggil kalau signature gak cocok dan calldata kosong; **`fallback()`** kalau calldata tidak kosong; **`payable`** wajib supaya fungsi bisa terima ether.

**[[W04 - Wallets and Transactions]]** — Wallet gak nyimpen ether, cuma nyimpen kunci. Dua jenis: nondeterministic (JBOK) dan deterministic (satu seed menurunkan semua). Bentuk tercanggih: HD wallet BIP-32, struktur pohon. BIP-39 mengubah entropy jadi 12–24 kata mnemonic (2.048 kata, 11 bit per kata), lalu PBKDF2 + salt jadi seed 512-bit. Transaksi adalah **satu-satunya** pemicu perubahan state — contract gak jalan sendiri. Nonce mencegah urutan kacau dan replay. Gas ≠ ether, dia mata uang virtual terpisah supaya biaya komputasi gak ikut volatilitas ether. Field `to` **tidak divalidasi** — salah alamat = ether hangus.

**[[W05 - Solidity Development]]** — Siklus hidup: compile ke bytecode → deploy lewat contract creation transaction ke `0x0` → eksekusi cuma kalau dipanggil transaksi EOA → terminasi lewat `SELFDESTRUCT` (harus sengaja diprogram; deprecated sejak 0.8.18). Contract itu dorman dan single-threaded. Tiga tipe objek: contract, interface (deklarasi saja), library (deploy sekali, dipakai lewat `delegatecall`). Visibility: public/external/internal/private. Behaviour: view/pure/payable. Modifier untuk access control (`onlyOwner`). `require` untuk input, `assert` untuk kondisi internal; error → semua state di-revert (atomik). Events + `emit` + `indexed` untuk komunikasi ke DApp. Memanggil contract lain: paling aman kalau bikin sendiri, paling bahaya raw call.

**[[W06 - Smart Contract Standard]]** — ERC = Ethereum Request for Comment, 50+ beredar. Empat yang dibahas: ERC-20 (fungible, 6 fungsi wajib, alur `approve` → `transferFrom`), ERC-721 (NFT, tiap token unik), ERC-1155 (fungible + non-fungible dalam satu contract, batch transfer, semi-fungible), ERC-4626 (tokenized vault). Alasan harus tau standar: biar gak bingung fungsi warisan dari mana, dan karena **deviasi dari standar itu sendiri jadi audit finding** — contohnya temuan Notional 2021 soal return value ERC-20 yang gak dicek.

**[[W07 - The Hardhat Framework]]** — Hardhat = development environment Ethereum, dibangun di sekitar **tasks** dan **plugins**, unopinionated. Hardhat Network = node lokal, memakai implementasi EVM versi JavaScript. Struktur: `contracts/`, `deployments/`, `test/`, `hardhat.config.js`. Alur: install → tulis contract → test pakai chai → simpan private key di `.env` → `npx hardhat compile` (hasilkan ABI + bytecode) → deploy → verify di Etherscan (menaikkan trust dan security). Hands-on beratnya: rekonstruksi **Parity wallet hack** (150.000+ ETH) — `initWallet` terbuka untuk siapa saja — memakai mainnet forking, account impersonation, dan block pinning.

**[[W08 - Smart Contract Pitfalls]]** — Premisnya: contract mengeksekusi persis apa yang ditulis, bukan apa yang dimaksud; semuanya publik; kerugian hampir mustahil dipulihkan. Lima pilar defensive programming, lalu lima risiko besar beserta mitigasinya (lihat tabel di atas). Tiga contract latihan: `EtherStore` (reentrancy), `TimeLock` (overflow), `BadRandomContract` (insecure randomness). Studi kasus HospoWise: fungsi `burn()` yang publik.

**[[W09 - Smart Contract Auditing]]** — Audit = penilaian keamanan eksternal yang dibayar tim proyek, biayanya mulai $10K/minggu. Tujuannya mengurangi attack surface sebelum peluncuran. Trail of Bits mengklasifikasikan temuan ke 13 kategori (Access Controls, Data Validation, Cryptography, dst). Severity = Likelihood × Impact. Dua teknik: static analysis (**Slither** di level Solidity, **Mythril** di level bytecode dengan symbolic execution + SMT solving + taint analysis) dan **fuzzing** (**Echidna**, Harvey). Puncaknya CTF di Damn Vulnerable DeFi.

**[[W10 - Audit Findings]]** — Audit wajib karena contract yang sudah di-deploy itu immutable. Tujuh komponen laporan audit. Skala severity ketiga: Code4rena (QA / 2 Med / 3 High). Semua contoh merujuk ke laporan Notional 2021. Dua tugas: audit `SimpleAuction` (minimal 4 isu) dan `InSecureumDAO` (dipandu 4 pertanyaan yang sebenarnya memetakan ke kategori temuan: access control, data validation, dan fungsi kritis yang lupa diproteksi).

---

### Pola yang muncul berulang

Kalau ada satu hal yang mau diingat dari seluruh matkul ini, ini dia:

**Bug yang paling sering dan paling mahal itu bukan bug yang canggih.** Parity kehilangan 150.000 ETH karena satu fungsi lupa diproteksi. HospoWise jebol karena `burn()` yang publik. InSecureumDAO di soal latihan punya pola yang sama. Tiga kejadian, satu bug: **fungsi kritis tanpa access control.**

**Immutability itu fitur sekaligus kutukan.** W01 menjualnya sebagai keunggulan (data gak bisa diubah = terpercaya). W05 menunjukkan sisi lainnya (kodemu juga gak bisa diperbaiki). W09 dan W10 menjelaskan kenapa seluruh industri auditing bernilai puluhan ribu dolar per minggu ada gara-gara itu.

**Determinisme itu syarat sekaligus batas.** Blockchain harus deterministik supaya semua node bisa memverifikasi hasil yang sama. Konsekuensinya, keacakan sejati mustahil lahir dari dalam sistem, dan `pure`/`view` bisa dijamin, dan transaksi bisa diputar ulang untuk diverifikasi. Satu sifat, konsekuensi ke mana-mana.

---

## Bagian 2 — Glosari

### A–C

| Istilah | Arti | Muncul di |
| --- | --- | --- |
| **ABI** | Application Binary Interface; spesifikasi fungsi contract (argumen, state mutability, nama) — cara aplikasi berinteraksi dengan contract | W07 |
| **Access control** | Pembatasan siapa yang boleh memanggil fungsi tertentu | W05, W07, W08, W10 |
| **Alchemy** | Penyedia node Ethereum yang menyediakan archival node (dibutuhkan untuk block pinning) | W07 |
| **`allowance`** | Fungsi wajib ERC-20 yang berkaitan dengan jatah token spender | W06 |
| **`approve`** | Fungsi wajib ERC-20; mengizinkan spender menarik token atas nama owner | W06 |
| **Archival node** | Node yang menyimpan seluruh state historis chain | W07 |
| **`assert`** | Fungsi error handling untuk menguji **kondisi internal** yang seharusnya selalu true | W05 |
| **Air-gapped device** | Perangkat yang terputus total dari jaringan; tingkat keamanan tertinggi, tapi tidak dibutuhkan untuk setiap akun | W03 |
| **Attack surface** | Luas permukaan yang bisa diserang; audit bertujuan menguranginya | W09 |
| **Avalanche effect** | Perubahan kecil di input menghasilkan output hash yang beda total | W01 |
| **Babbage** | Nama umum untuk 10^3 wei (kilowei / femtoether) | W03 |
| **Batch transfer** | Mentransfer banyak token dalam satu transaksi (fitur ERC-1155) | W06 |
| **`balanceOf`** | Fungsi wajib ERC-20; menunjukkan saldo pemilik token | W06 |
| **BIP** | Bitcoin Improvement Proposal | W04 |
| **BIP-32** | Standar hierarchical deterministic (HD) wallet | W04 |
| **BIP-39** | Standar mnemonic code (seed phrase) | W04 |
| **`block.number`** | Global variable berisi nomor block saat ini; **bisa diprediksi**, jangan dipakai untuk keacakan | W05, W08 |
| **`block.timestamp`** | Global variable berisi timestamp block saat ini (alias `now`) | W05 |
| **Block** | Unit data berisi header (timestamp + referensi block sebelumnya) dan daftar transaksi | W01 |
| **Block pinning** | Menyetel state chain ke nomor block tertentu (fitur Hardhat, butuh archival node) | W07 |
| **Bytecode** | Kode level rendah yang dimengerti EVM; hasil kompilasi Solidity | W02, W05, W07 |
| **`call`** | Metode call level rendah pada alamat mentah; **return `false`**, tidak melempar exception | W05, W08 |
| **calldata** | Data yang menyertai pemanggilan fungsi (`msg.data`); kosong atau tidaknya menentukan `receive()` vs `fallback()` | W03 |
| **Casper** | Rencana sistem voting berbobot berbasis PoS untuk Ethereum | W02 |
| **`castVote()`** | Fungsi InSecureumDAO yang diperiksa dalam latihan audit | W10 |
| **chai** | Library assertion untuk unit testing di ekosistem JavaScript | W07 |
| **Chain** | Rangkaian block yang tersambung berurutan lewat hash | W01 |
| **ChainLink VRF** | Verifiable Random Function; solusi keacakan yang bisa diverifikasi | W08 |
| **Code4rena** | Platform keamanan smart contract; skala severity QA / Med / High | W06, W10 |
| **Contract account** | Akun berisi kode smart contract; **tidak punya private key**, dikendalikan logika kodenya sendiri | W03 |
| **Consensus mechanism** | Aturan yang bikin semua node sepakat soal state blockchain | W01 |
| **ConsenSys** | Firma dengan skala severity Minor / Medium / Major / Critical | W09 |
| **Constructor** | Fungsi yang jalan sekali saat contract dibuat, untuk inisialisasi state | W05 |
| **Contract creation transaction** | Transaksi khusus ke alamat `0x0` untuk men-deploy contract | W05 |
| **`createVote()`** | Fungsi InSecureumDAO yang diperiksa dalam latihan audit | W10 |

### D–G

| Istilah | Arti | Muncul di |
| --- | --- | --- |
| **DAO** | Decentralized Autonomous Organization; organisasi yang aturannya di kode, keputusannya lewat proposal dan voting on-chain | W02 |
| **DAO fork** | Hard fork block #1.192.000 yang mengganti rugi korban hack DAO dan memecah Ethereum / Ethereum Classic | W02 |
| **dApp / DApp** | Aplikasi web di atas infrastruktur P2P terdesentralisasi: smart contract + frontend | W01, W02 |
| **Damn Vulnerable DeFi** | Platform CTF smart contract yang dipakai untuk tugas W09 | W09 |
| **Defensive programming** | Gaya pengembangan dengan lima pilar: Minimalism, Code Reuse, Code Quality, Readability, Test Coverage | W08 |
| **DeFi** | Decentralized Finance; ekosistem aplikasi keuangan tanpa perantara | W01 |
| **`delegatecall`** | Metode call level rendah; dipakai library dan arsitektur proxy. **Tidak pernah dijelaskan di slide mana pun** | W05, W07, W08 |
| **Deterministic wallet** | Wallet yang semua kuncinya diturunkan dari satu seed | W04 |
| **DEX** | Decentralized Exchange | W01 |
| **DRY** | Don't Repeat Yourself | W08 |
| **Echidna** | Fuzzer smart contract berbasis Haskell, property-based, memakai ABI contract | W09 |
| **EOA** | Externally Owned Account; akun yang **punya private key**. Satu-satunya yang bisa memicu eksekusi contract | W03, W04, W05 |
| **`emit`** | Keyword untuk memicu data event masuk ke transaction log | W05 |
| **ERC** | Ethereum Request for Comment; standar teknis Ethereum, 50+ beredar | W06 |
| **ERC-20 / 721 / 1155 / 4626** | Standar fungible / NFT / multi token / tokenized vault | W06 |
| **Emerald Wallet** | Wallet desktop open source untuk Ethereum Classic dan chain berbasis Ethereum | W03 |
| **Ether** | Cryptocurrency Ethereum; **utility currency** untuk membayar eksekusi | W02 |
| **Ethash** | Algoritma PoW yang dipakai Ethereum (saat slide ditulis) | W02 |
| **Etherscan** | Block explorer Ethereum; melihat riwayat transaksi alamat dan memverifikasi source code contract | W03, W07 |
| **Ethers** | Library JavaScript untuk berinteraksi dengan Ethereum | W07 |
| **EVM** | Ethereum Virtual Machine; VM berbasis stack yang mengeksekusi bytecode | W02, W05 |
| **Event** | Log entry di transaction receipt yang bisa "ditonton" DApp | W05 |
| **`fallback()`** | Dipanggil saat tidak ada signature fungsi yang cocok **dan calldata tidak kosong**. **Jantung mekanisme reentrancy** | W03, W05, W08 |
| **Faucet** | Layanan/contract yang membagikan test ether ke alamat mana pun yang meminta | W03 |
| **Finney** | Nama umum untuk 10^15 wei (milliether) | W03 |
| **Frontier / Homestead / Metropolis / Serenity** | Empat tahap pengembangan Ethereum | W02 |
| **Fungible** | Tiap unit identik dan bisa saling ditukar | W06 |
| **Fuzzing** | Memberi input tak valid/acak secara otomatis lalu memantau anomali | W09 |
| **Gas** | "Bahan bakar" Ethereum; **mata uang virtual terpisah dari ether** dengan kurs sendiri | W04 |
| **gasLimit** | Maksimum gas yang bersedia dibeli pengirim untuk sebuah transaksi | W04 |
| **gasPrice** | Wei yang bersedia dibayar pengirim per unit gas | W04 |
| **Genesis block** | Block pertama dalam rantai | W01 |
| **GETH** | Go-Ethereum; implementasi client Ethereum paling banyak dipakai | W02, W07 |
| **gwei** | Satuan gas price; 1 gwei = 1 miliar wei | W04 |

### H–P

| Istilah | Arti | Muncul di |
| --- | --- | --- |
| **Grand** | Nama umum untuk 10^21 wei (kiloether) | W03 |
| **Hard fork** | Perubahan fungsionalitas yang **tidak backward-compatible** | W02 |
| **Hardhat** | Development environment Ethereum; dibangun di sekitar tasks dan plugins | W07 |
| **Hardhat Network** | Node Ethereum lokal bawaan Hardhat, memakai implementasi EVM versi JavaScript | W07 |
| **Harvey** | Tool fuzzing smart contract. Disebut tapi tidak dijelaskan | W09 |
| **`hasRole`** | Modifier pengecek peran, alternatif `onlyOwner` | W08 |
| **Hash function** | Fungsi yang menghasilkan string unik dari data; menyambungkan block dan menjaga integritas | W01 |
| **HD wallet** | Hierarchical Deterministic wallet; kunci diturunkan dalam struktur pohon | W04 |
| **HospoWise** | Studi kasus peretasan akibat fungsi `burn()` yang publik | W08 |
| **Impact** | Perkiraan besarnya dampak teknis dan bisnis kalau kerentanan dieksploitasi | W09 |
| **Immutability** | Sekali tercatat, tidak bisa diubah. Fitur di W01, kendala di W05, alasan auditing ada di W09–W10 | W01, W05, W10 |
| **Impersonate account** | Bertindak seolah pemilik suatu akun (fitur Hardhat di jaringan lokal) | W07 |
| **`indexed`** | Keyword yang bikin nilai event bisa dicari/difilter aplikasi | W05 |
| **Infura** | Layanan API penyedia akses node Ethereum | W07 |
| **`initWallet`** | Fungsi Parity yang terbuka untuk siapa saja; penyebab hilangnya 150.000+ ETH | W07 |
| **Interface** | Seperti contract tapi fungsinya cuma dideklarasikan, wajib didefinisikan child-nya | W05 |
| **Jaxx** | Wallet multiplatform dan multicurrency, dirancang untuk kesederhanaan | W03 |
| **JBOK wallet** | "Just a Bunch of Keys"; wallet nondeterministic | W04 |
| **Least privilege** | Prinsip memberi hak seminimal mungkin | W08 |
| **Lovelace** | Nama umum untuk 10^6 wei (megawei / picoether) | W03 |
| **LevelDB** | Database tempat node menyimpan state Ethereum secara lokal | W02 |
| **Library** | Contract yang dideploy sekali dan dipakai contract lain lewat `delegatecall` | W05 |
| **Likelihood** | Seberapa mudah/sulit kerentanan ditemukan dan dieksploitasi. **"Low" = mudah dieksploitasi** | W09 |
| **Mainnet forking** | Menyalin state mainnet ke jaringan lokal untuk eksperimen | W07 |
| **MetaMask** | Wallet ekstensi browser + aplikasi mobile; mudah dipakai untuk testing, bisa konek ke berbagai node dan test blockchain | W03 |
| **Merkle Patricia Tree** | Struktur data hash berseri tempat state Ethereum disimpan | W02 |
| **MyEtherWallet (MEW)** | Wallet berbasis web yang jalan di browser mana pun, tersedia juga di Android dan iOS | W03 |
| **Mnemonic** | Deretan kata (BIP-39) yang bisa merekonstruksi private key | W04 |
| **Modifier** | Kondisi yang bisa ditempel ke banyak fungsi; basis access control | W05 |
| **`msg.sender`** | Alamat pemanggil call **saat ini**; **berubah** tiap hop | W04, W05 |
| **`msg.value`** | Jumlah Ether yang dikirim bersama transaksi saat ini | W05 |
| **Multi-sig wallet** | Wallet dengan beberapa owner dan ambang minimum tanda tangan (threshold) | W07 |
| **Mutex** | State variable yang mengunci contract selama eksekusi; mitigasi reentrancy | W08 |
| **Mythril** | Security analyzer level **EVM bytecode**; symbolic execution + SMT solving + taint analysis | W09 |
| **Nakamoto Consensus** | Model consensus dari Bitcoin yang dipakai Ethereum | W02 |
| **NFT** | Non-fungible token; standar ERC-721 | W06 |
| **Node** | Komputer peserta jaringan yang memvalidasi dan meneruskan transaksi | W01 |
| **Nonce** (mining) | Bagian data block yang diubah-ubah miner supaya hash jatuh di bawah target | W01 |
| **Nonce** (transaksi) | Nomor urut transaksi dari satu alamat; mencegah urutan kacau dan replay attack | W04 |
| **Non-fungible** | Tiap unit unik dan tidak bisa saling ditukar | W06 |
| **Not Invented Here syndrome** | Godaan membangun ulang dari nol komponen yang sudah teruji | W08 |
| **`onlyOwner`** | Modifier access control paling umum; dari library Ownable OpenZeppelin | W05, W08 |
| **OpenZeppelin** | Penyedia library standar Solidity (SafeMath, Ownable, dst) | W08 |
| **Overflow** | Menambah angka melebihi batas atas tipe data; nilainya berputar ke bawah | W08 |
| **Parity hack** | Peretasan multi-sig wallet, 150.000+ ETH dicuri lewat `initWallet` yang terbuka | W07 |
| **`payable`** | Modifier yang mengizinkan fungsi menerima ether; tanpa ini transaksi yang membawa ether **ditolak** | W03, W05 |
| **PBKDF2** | Key-stretching function yang mengubah mnemonic + salt jadi seed 512-bit | W04 |
| **Plugin** (Hardhat) | Tulang punggung Hardhat; sumber sebagian besar fungsionalitasnya | W07 |
| **PoS** | Proof of Stake; validator dipilih berdasarkan koin yang di-stake | W01 |
| **PoW** | Proof of Work; miner memecahkan teka-teki matematis | W01 |
| **PRNG** | Pseudo-random number generator; acak semu yang deterministik dari seed | W08 |
| **`pure`** | Fungsi yang tidak baca dan tidak tulis storage | W05 |

### Q–Z

| Istilah | Arti | Muncul di |
| --- | --- | --- |
| **QA (Code4rena)** | Kategori severity untuk low risk dan governance/centralization risk | W10 |
| **Raw call** | Pemanggilan contract-ke-contract secara manual; paling fleksibel, **paling berbahaya** | W05 |
| **Reentrancy** | Contract eksternal memanggil balik fungsi contract korban sebelum eksekusi pertama selesai | W08 |
| **`receive()`** | Dipanggil saat tidak ada signature fungsi yang cocok **dan calldata kosong** | W03, W05 |
| **Remix IDE** | IDE berbasis browser untuk Solidity; dipakai di hands-on W03 dan W04 | W03, W04 |
| **`removeAllMembers()`** | Fungsi InSecureumDAO yang diperiksa dalam latihan audit | W10 |
| **`require`** | Fungsi error handling untuk menguji **input** | W05 |
| **`revert`** | Salah satu dari empat fungsi error handling. Disebut tapi tidak dijelaskan | W05 |
| **Rinkeby / Ropsten** | Testnet Ethereum yang dipakai di slide. **Sudah dimatikan** | W03, W07 |
| **SafeMath** | Library OpenZeppelin pengganti operator matematika, aman dari over/underflow | W08 |
| **Salt** | Input tambahan yang mempersulit lookup table brute-force; di BIP-39 juga slot passphrase | W04 |
| **Shannon** | Nama umum untuk 10^9 wei (gigawei / nanoether); ini yang sehari-hari disebut **gwei** | W03 |
| **Seed** | Master key yang menurunkan semua private key di deterministic wallet | W04 |
| **`selfdestruct` / SELFDESTRUCT** | Opcode penghapus contract; harus sengaja diprogram, **deprecated sejak Solidity 0.8.18** | W05 |
| **Semi-fungible** | Punya sebagian sifat fungible dan sebagian non-fungible (ERC-1155) | W06 |
| **`send`** | Metode call level rendah; **return `false`**, tidak melempar exception. Hindari, pakai `transfer` | W08 |
| **Severity** | Hasil kombinasi Likelihood dan Impact | W09 |
| **SHA-256** | Algoritma hash yang umum dipakai blockchain | W01 |
| **Singleton state** | Satu state global yang sama untuk seluruh jaringan | W02 |
| **Slither** | Static analyzer level **Solidity**, buatan Trail of Bits; <1 detik per contract | W09 |
| **SlithIR** | Intermediate representation milik Slither | W09 |
| **SMT solver** | Mesin penyelesai batasan logika yang dipakai Mythril | W09 |
| **Solidity** | Bahasa tingkat tinggi untuk menulis smart contract | W02, W05 |
| **Static analysis** | Menganalisis properti program **tanpa menjalankannya** | W09 |
| **Symbolic execution** | Mengeksekusi kode dengan input simbolik untuk menjelajah banyak jalur eksekusi | W09 |
| **Szabo** | Nama umum untuk 10^12 wei (microether) | W03 |
| **Taint analysis** | Analisis pelacakan aliran data yang terkontaminasi input | W09 |
| **Task** (Hardhat) | Fungsi async JavaScript bermetadata; semua yang bisa dilakukan Hardhat adalah task | W07 |
| **Threshold** | Jumlah minimum tanda tangan untuk mengeksekusi transaksi di multi-sig wallet | W07 |
| **`totalSupply`** | Fungsi wajib ERC-20; total suplai token | W06 |
| **Trail of Bits** | Firma auditing keamanan blockchain papan atas; pembuat Slither | W09 |
| **`transfer`** (address) | Member function `address` untuk mengirim ether; **cuma kirim 2300 gas**, dan revert kalau gagal | W05, W08 |
| **`transfer`** (ERC-20) | Fungsi wajib ERC-20; transfer token ke alamat tertentu | W06 |
| **`transferFrom`** | Fungsi wajib ERC-20; transfer dari alamat tertentu setelah `approve` | W06 |
| **Trilemma** | Trade-off security, scalability, decentralization | W01 |
| **`tx.origin`** | Alamat EOA yang **mengawali** transaksi; **tetap** sepanjang call chain | W04, W05 |
| **`tx.gasprice`** | Gas price transaksi saat ini | W05 |
| **Underflow** | Mengurangi angka di bawah batas bawah tipe data; nilainya berputar ke atas | W08 |
| **`uint8`** | Integer tak bertanda 8 bit, rentang [0, 255] | W05, W08 |
| **Unchecked external call** | Nilai kembalian message call tidak dicek, eksekusi lanjut seolah sukses | W08 |
| **VDF** | Verifiable Delay Function (VeeDo) | W08 |
| **v, r, s** | Tiga komponen digital signature ECDSA pada transaksi | W04 |
| **`view` / `constant`** | Fungsi yang berjanji tidak mengubah state | W05 |
| **VRF** | Verifiable Random Function (ChainLink) | W08 |
| **Waffle** | Framework testing smart contract | W07 |
| **Wallet** | Aplikasi pengelola akun Ethereum; gerbang ke sistem. **Menyimpan kunci, bukan ether** | W03, W04 |
| **wei** | Satuan terkecil ether; 1 ether = 10^18 wei. Nilai selalu disimpan internal sebagai unsigned integer dalam wei | W03, W04 |
| **Withdrawal pattern** | Pola di mana penerima menarik dananya sendiri lewat fungsi terpisah | W08 |
| **World computer** | Julukan Ethereum | W02 |
| **Yellow Paper** | Dokumen yang mendefinisikan consensus rules Ethereum | W02 |
| **ÐΞVp2p** | Protokol P2P Ethereum, TCP port 30303 | W02 |

---

## Bagian 3 — Yang Perlu Dicek Sendiri

Hal-hal yang **tidak ada di slide** tapi kemungkinan besar dibutuhkan:

1. **Severity Matrix** (W09 slide 8) — matriks Likelihood × Impact tidak berhasil diekstrak dari PPT. Buka manual.
2. **Tabel perbandingan standar token** (W06 slide 10) — berupa gambar, isinya belum diverifikasi.
3. ~~Fallback / receive function tidak ada di deck mana pun.~~ **Terjawab:** ada di [[W03 - Ethereum Basics]] slide 20–21. Yang masih perlu dicek: definisi `fallback()` di slide menyebut "ether dikirim langsung ke contract" sebagai bagian dari syarat, padahal pembeda sebenarnya cuma kosong/tidaknya calldata.
4. **`delegatecall`** — disebut tiga kali, tidak pernah dijelaskan.
5. **Flash loan** — dasar ketiga tantangan CTF di W09, tidak pernah disinggung.
6. **Semua kode contract** (`EtherStore`, `TimeLock`, `BadRandomContract`, `Lotto`, `FibonacciLib`, `Incrementor`, contoh Solidity di W05) berupa **gambar**, bukan teks. Harus diketik ulang dari gambar atau diambil dari pastebin yang disebut slide.
7. **Materi yang sudah usang**: Ethereum PoW/Casper (W02), testnet Ropsten (W03) dan Rinkeby (W07), SafeMath sebagai keharusan (W08), `SELFDESTRUCT` (W05). Tanyakan mana yang diujikan.
8. **Versi compiler gak konsisten antar deck**: W03 pakai 0.6.4, W07 pakai 0.8.8, W05 bicara soal perubahan di 0.8.18. Bedanya 0.6 ke 0.8 itu besar (over/underflow otomatis revert sejak 0.8.0). Pastikan versi mana yang dipakai di praktikum.

## Terkait
- [[_Blockchain]]
