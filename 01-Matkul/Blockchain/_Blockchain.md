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

Index matkul. Sembilan deck PPT dari dosen, diproses jadi sembilan note pertemuan.

## Daftar Pertemuan

| # | Note | Isinya |
| --- | --- | --- |
| W01 | [[W01 - Introduction to Blockchain Technology]] | Apa itu blockchain, struktur node/block/chain, PoW vs PoS, hash function, mining, trilemma, DeFi |
| W02 | [[W02 - The Fundamentals of Ethereum]] | Ethereum sebagai "world computer", bedanya dari Bitcoin, empat tahap pengembangan + hard fork, komponen, DApp dan DAO |
| W03 | [[W03 - Wallets and Transactions]] | Wallet cuma nyimpen kunci, JBOK vs deterministic vs HD wallet, BIP-32/39/44, struktur transaksi, nonce, gas, `tx.origin` vs `msg.sender` |
| W04 | [[W04 - Solidity Development]] | Siklus hidup contract, tipe data, global variable, visibility, modifier, inheritance, `assert`/`require`, events, memanggil contract lain |
| W05 | [[W05 - Smart Contract Standard]] | ERC-20, ERC-721, ERC-1155, ERC-4626, dan kenapa deviasi dari standar itu jadi audit finding |
| W06 | [[W06 - The Hardhat Framework]] | Setup Hardhat, struktur proyek, test dengan chai, compile, deploy, verify di Etherscan, rekonstruksi Parity hack |
| W07 | [[W07 - Smart Contract Pitfalls]] | Defensive programming, lima risiko utama (reentrancy, over/underflow, insecure randomness, access control, unchecked external calls) + mitigasinya |
| W08 | [[W08 - Smart Contract Auditing]] | Apa itu audit, klasifikasi temuan, likelihood/impact/severity, static analysis (Slither, Mythril), fuzzing (Echidna) |
| W09 | [[W09 - Audit Findings]] | Tujuh komponen laporan audit, skala severity Code4rena, dua tugas audit (`SimpleAuction`, `InSecureumDAO`) |

## Rangkuman Ujian
- [[Blockchain - Review dan Glosari]] — review lintas pertemuan + glosari lengkap

## Peta Materi

Matkul ini punya alur yang rapi dan menanjak. Tiga blok besar:

**Blok 1 — Fondasi (W01–W03): "apa yang terjadi di bawah sana"**
W01 kasih mesinnya secara umum (blockchain apa pun). W02 mempersempit ke Ethereum spesifik dan kenapa dia beda: bukan sekadar uang, tapi komputer. W03 menjelaskan bagaimana kamu, manusia, benar-benar menyentuh mesin itu — lewat kunci dan transaksi. **Kalau W03 gak paham, W04 gak akan nyambung**, karena `msg.sender`/`tx.origin` dan gas dipakai terus-menerus setelahnya.

**Blok 2 — Membangun (W04–W06): "cara nulis dan naruh kodenya"**
W04 bahasanya (Solidity). W05 aturan mainnya (standar ERC yang harus diikuti biar dikenali ekosistem). W06 alat kerjanya (Hardhat, dari compile sampai verify). Blok ini yang paling praktis dan paling banyak hands-on.

**Blok 3 — Membobol dan memeriksa (W07–W09): "cara nyari yang salah"**
W07 kasih daftar bug-nya. W08 kasih metode dan alat mencarinya, plus cara mengukur seberapa parah. W09 kasih cara menuliskan hasilnya jadi laporan. Blok ini yang jadi tujuan akhir matkul.

**Benang merah yang menembus semua blok:**

- **`msg.sender` vs `tx.origin`** — diperkenalkan di W03, muncul lagi sebagai global variable di W04, jadi dasar modifier `onlyOwner` di W04, dan jadi kategori kerentanan access control di W07.
- **Access control** — pola bug yang sama muncul tiga kali dengan nama berbeda: `initWallet` Parity (W06), `burn()` HospoWise (W07), dan `createVote()`/`removeAllMembers()` InSecureumDAO (W09). Semuanya **fungsi kritis yang lupa dikasih modifier**.
- **Skala severity** — Trail of Bits dan ConsenSys di W08, Code4rena di W09. Tiga skala berbeda, tanpa standar tunggal.
- **Return value ERC-20 yang gak dicek** — muncul sebagai temuan audit Notional di W05, lalu dijelaskan mekanismenya sebagai *unchecked external calls* di W07, lalu laporan Notional yang sama dibedah komponennya di W09.
- **Immutability** — dijelaskan sebagai fitur di W01, jadi kendala di W04 (`SELFDESTRUCT` harus diprogram sejak awal), lalu jadi **alasan seluruh profesi auditing ada** di W08 dan W09.

## Konsep Utama
Belum ada note atomik di `02-Konsep/`. Kandidat terkuat kalau nanti mau dibuat (konsep yang muncul di lebih dari satu pertemuan):

- **Hash function** — W01, dipakai implisit di seluruh matkul
- **msg.sender vs tx.origin** — W03, W04, W07
- **Gas** — W03, W06, W09
- **Access control (`onlyOwner`)** — W04, W06, W07, W09
- **Reentrancy** — W07, relevan ke W08 dan W09
- **Severity: Likelihood × Impact** — W08, W09

## Catatan Pemrosesan

**Penomoran minggu itu inferensi, bukan dari slide.** Yang dinyatakan eksplisit di PPT cuma ini:

| Deck | Sesi menurut slide |
| --- | --- |
| Smart Contract and Solidity (W04) | **S05 dan S07** |
| Smart Contract Pitfalls (W07) | **S09 dan S10** |
| Smart Contract Auditing (W08) | **S11 dan S12** |

Sisanya (W01, W02, W03, W05, W06, W09) diurutkan berdasarkan **alur materi**, bukan keterangan dosen. Kalau digabung, urutan itu pas mengisi 13 sesi — tapi ini tetap dugaan. **Cocokkan dengan silabus asli sebelum dipakai buat belajar UTS/UAS.**

**Duplikat.** Dua file di inbox identik byte-per-byte dengan pasangannya dan tidak diproses ulang: `Smart Contract Pitfalls (1).pptx` dan `Smart Contract and Solidity (1).pptx`.

**Lubang materi yang konsisten.** Tiga hal disebut berkali-kali tapi tidak pernah dijelaskan di deck mana pun:
1. **Fallback / receive function** — W04 menyebutnya dengan catatan "see previous session", padahal tidak ada deck sebelumnya yang membahasnya. Ini jantung mekanisme reentrancy di W07.
2. **`delegatecall`** — muncul tiga kali (library dan raw call di W04, arsitektur proxy Parity di W06, metode call level rendah di W07), tidak pernah dijelaskan.
3. **Flash loan** — jadi dasar ketiga tantangan CTF di W08, tidak pernah disinggung sama sekali.

**Materi yang sudah usang di slide.** Perlu dikonfirmasi ke dosen mana yang diujikan:
- Ethereum masih digambarkan PoW dengan Casper sebagai rencana (W02) — The Merge sudah terjadi September 2022.
- Testnet Rinkeby dan Ropsten (W06) sudah dimatikan.
- SafeMath sebagai mitigasi over/underflow (W07) — sejak Solidity 0.8.0 sudah otomatis revert.
- `SELFDESTRUCT` deprecated sejak 0.8.18, diakui slide sendiri (W04).
