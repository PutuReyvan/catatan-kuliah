---
matkul: Blockchain
sks: 2
dosen: Chrisando Ryan Pardomuan
jadwal: Senin (1)
tags: [kuliah/blockchain, moc]
status: draft
diproses: 2026-09-03
---

# Senin (1) — Blockchain Fundamental

Index matkul. Sepuluh deck PPT dari dosen, diproses jadi sepuluh note pertemuan.

## Daftar Pertemuan

| # | Note | Isinya |
| --- | --- | --- |
| W01 | [[W01 - Introduction to Blockchain Technology]] | Apa itu blockchain, struktur node/block/chain, PoW vs PoS, hash function, mining, trilemma, DeFi |
| W02 | [[W02 - The Fundamentals of Ethereum]] | Ethereum sebagai "world computer", bedanya dari Bitcoin, empat tahap pengembangan + hard fork, komponen, DApp dan DAO |
| W03 | [[W03 - Ethereum Basics]] | Satuan ether sampai wei, wallet dan cara melindunginya, EOA vs contract account, contract pertama (`Faucet.sol`) di Remix, `receive()` / `fallback()` / `payable` |
| W04 | [[W04 - Wallets and Transactions]] | Wallet cuma nyimpen kunci, JBOK vs deterministic vs HD wallet, BIP-32/39/44, struktur transaksi, nonce, gas, `tx.origin` vs `msg.sender` |
| W05 | [[W05 - Solidity Development]] | Siklus hidup contract, tipe data, global variable, visibility, modifier, inheritance, `assert`/`require`, events, memanggil contract lain |
| W06 | [[W06 - Smart Contract Standard]] | ERC-20, ERC-721, ERC-1155, ERC-4626, dan kenapa deviasi dari standar itu jadi audit finding |
| W07 | [[W07 - The Hardhat Framework]] | Setup Hardhat, struktur proyek, test dengan chai, compile, deploy, verify di Etherscan, rekonstruksi Parity hack |
| W08 | [[W08 - Smart Contract Pitfalls]] | Defensive programming, lima risiko utama (reentrancy, over/underflow, insecure randomness, access control, unchecked external calls) + mitigasinya |
| W09 | [[W09 - Smart Contract Auditing]] | Apa itu audit, klasifikasi temuan, likelihood/impact/severity, static analysis (Slither, Mythril), fuzzing (Echidna) |
| W10 | [[W10 - Audit Findings]] | Tujuh komponen laporan audit, skala severity Code4rena, dua tugas audit (`SimpleAuction`, `InSecureumDAO`) |

## Rangkuman Ujian
- [[Blockchain - Review dan Glosari]] — review lintas pertemuan + glosari lengkap

## Peta Materi

Matkul ini punya alur yang rapi dan menanjak. Tiga blok besar:

**Blok 1 — Fondasi (W01–W04): "apa yang terjadi di bawah sana"**
W01 kasih mesinnya secara umum (blockchain apa pun). W02 mempersempit ke Ethereum spesifik dan kenapa dia beda: bukan sekadar uang, tapi komputer. W03 turun ke tanah — satuan uangnya, wallet-nya, dua jenis akunnya, dan contract pertama yang benar-benar kamu deploy sendiri. W04 kembali naik ke mekanismenya: bagaimana kunci diturunkan dan bagaimana transaksi disusun. **Kalau W04 gak paham, W05 gak akan nyambung**, karena `msg.sender`/`tx.origin` dan gas dipakai terus-menerus setelahnya.

W03 dan W04 kelihatan tumpang tindih (dua-duanya bahas wallet), tapi sudut pandangnya beda: **W03 dari sisi pemakai** (install MetaMask, jangan hilangin seed, ini contract pertamamu), **W04 dari sisi mekanisme** (BIP-32/39, PBKDF2, field transaksi, nonce).

**Blok 2 — Membangun (W05–W07): "cara nulis dan naruh kodenya"**
W05 bahasanya (Solidity). W06 aturan mainnya (standar ERC yang harus diikuti biar dikenali ekosistem). W07 alat kerjanya (Hardhat, dari compile sampai verify). Blok ini yang paling praktis dan paling banyak hands-on.

**Blok 3 — Membobol dan memeriksa (W08–W10): "cara nyari yang salah"**
W08 kasih daftar bug-nya. W09 kasih metode dan alat mencarinya, plus cara mengukur seberapa parah. W10 kasih cara menuliskan hasilnya jadi laporan. Blok ini yang jadi tujuan akhir matkul.

**Benang merah yang menembus semua blok:**

- **`msg.sender` vs `tx.origin`** — diperkenalkan di W04, muncul lagi sebagai global variable di W05, jadi dasar modifier `onlyOwner` di W05, dan jadi kategori kerentanan access control di W08.
- **Access control** — pola bug yang sama muncul tiga kali dengan nama berbeda: `initWallet` Parity (W07), `burn()` HospoWise (W08), dan `createVote()`/`removeAllMembers()` InSecureumDAO (W10). Semuanya **fungsi kritis yang lupa dikasih modifier**.
- **Skala severity** — Trail of Bits dan ConsenSys di W09, Code4rena di W10. Tiga skala berbeda, tanpa standar tunggal.
- **Return value ERC-20 yang gak dicek** — muncul sebagai temuan audit Notional di W06, lalu dijelaskan mekanismenya sebagai *unchecked external calls* di W08, lalu laporan Notional yang sama dibedah komponennya di W10.
- **Immutability** — dijelaskan sebagai fitur di W01, jadi kendala di W05 (`SELFDESTRUCT` harus diprogram sejak awal), lalu jadi **alasan seluruh profesi auditing ada** di W09 dan W10.
- **`receive()` / `fallback()`** — dijelaskan di W03 sebagai "cara contract menangani transfer ether dengan anggun", disebut sambil lalu di W05, lalu ternyata **justru itu pintu masuk reentrancy** di W08. Satu fitur, dari fitur jadi kerentanan.
- **EOA vs contract account** — dibedakan di W03, jadi alasan "contract gak jalan sendiri" di W05, dan jadi dasar `tx.origin` vs `msg.sender` di W04.

## Konsep Utama
Belum ada note atomik di `02-Konsep/`. Kandidat terkuat kalau nanti mau dibuat (konsep yang muncul di lebih dari satu pertemuan):

- **Hash function** — W01, dipakai implisit di seluruh matkul
- **receive() dan fallback()** — W03, W05, W08 (kandidat terkuat: satu konsep, tiga peran berbeda)
- **EOA vs contract account** — W03, W04, W05
- **msg.sender vs tx.origin** — W04, W05, W08
- **Gas** — W04, W07, W10
- **Access control (`onlyOwner`)** — W05, W07, W08, W10
- **Reentrancy** — W08, relevan ke W09 dan W10
- **Severity: Likelihood × Impact** — W09, W10

## Catatan Pemrosesan

**Penomoran minggu itu inferensi, bukan dari slide.** Yang dinyatakan eksplisit di PPT cuma ini:

| Deck | Sesi menurut slide |
| --- | --- |
| Smart Contract and Solidity (W05) | **S05 dan S07** |
| Smart Contract Pitfalls (W08) | **S09 dan S10** |
| Smart Contract Auditing (W09) | **S11 dan S12** |

Sisanya (W01, W02, W04, W06, W07, W10) diurutkan berdasarkan **alur materi**, bukan keterangan dosen — kecuali **W03 (Ethereum Basics), yang posisinya dikonfirmasi langsung oleh pemilik vault**. Kalau digabung, urutannya pas mengisi 13 sesi:

`S01` W01 · `S02` W02 · `S03` W03 · `S04` W04 · `S05` W05 · `S06` W06 · `S07` W05 (lanjutan) · `S08` W07 · `S09–S10` W08 · `S11–S12` W09 · `S13` W10

Pas 13, tapi tetap sebagian dugaan. **Cocokkan dengan silabus asli sebelum dipakai buat belajar UTS/UAS.**

**Duplikat.** Dua file di inbox identik byte-per-byte dengan pasangannya dan tidak diproses ulang: `Smart Contract Pitfalls (1).pptx` dan `Smart Contract and Solidity (1).pptx`.

**Lubang materi yang konsisten.** Dua hal disebut berkali-kali tapi tidak pernah dijelaskan di deck mana pun:
1. **`delegatecall`** — muncul tiga kali (library dan raw call di W05, arsitektur proxy Parity di W07, metode call level rendah di W08), tidak pernah dijelaskan.
2. **Flash loan** — jadi dasar ketiga tantangan CTF di W09, tidak pernah disinggung sama sekali.

*(Sebelumnya **fallback / receive function** juga masuk daftar ini, karena W05 merujuknya sebagai "see previous session" tanpa ada deck yang membahasnya. Sekarang terjawab: penjelasannya ada di **W03 slide 20–21**.)*

**Materi yang sudah usang di slide.** Perlu dikonfirmasi ke dosen mana yang diujikan:
- Ethereum masih digambarkan PoW dengan Casper sebagai rencana (W02) — The Merge sudah terjadi September 2022.
- Testnet Rinkeby dan Ropsten (W07) sudah dimatikan.
- SafeMath sebagai mitigasi over/underflow (W08) — sejak Solidity 0.8.0 sudah otomatis revert.
- `SELFDESTRUCT` deprecated sejak 0.8.18, diakui slide sendiri (W05).
