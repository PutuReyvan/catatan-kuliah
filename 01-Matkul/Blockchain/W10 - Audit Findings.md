---
matkul: Blockchain
minggu: 10
sks: 2
sumber: Audit Findings Examination.pptx
tags: [kuliah/blockchain, minggu/w10]
status: draft
diproses: 2026-09-03
---

# W10 — Audit Findings

## Ringkasan
> - Alasan audit itu wajib, dalam satu kalimat: **begitu smart contract di-deploy dia jadi immutable**, jadi error dan kerentanan mustahil diperbaiki. Laporan audit memastikan isu-isu itu ketemu dan dibereskan **sebelum** deploy.
> - Laporan audit punya **7 komponen baku**: Executive Summary, Severity Criteria, Scope of Audit, Vulnerabilities Identified, Recommendations, Gas Optimization, Conclusion and Summary.
> - Skala severity ketiga (setelah Trail of Bits dan ConsenSys di [[W09 - Smart Contract Auditing]]): **Code4rena** — QA, Med (2), High (3).
> - Seluruh contoh laporan di deck ini merujuk ke satu laporan nyata: **Notional audit 2021 di Code4rena**.
> - Dua hands-on: audit `SimpleAuction` (minimal 4 isu) dan `InSecureumDAO` (dipandu 4 pertanyaan).

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Executive Summary | Ikhtisar tingkat tinggi soal keamanan dan fungsionalitas contract |
| Severity Criteria | Cara baku mengkategorikan dan memprioritaskan kerentanan yang ditemukan |
| Scope of Audit | Batas audit: bagian mana yang diperiksa dan apa yang dikecualikan |
| Gas Optimization | Rekomendasi mengoptimalkan konsumsi gas |
| Code4rena | Platform keamanan smart contract dengan skala severity sendiri |
| QA (Quality Assurance) | Kategori Code4rena untuk risiko rendah dan risiko governance/sentralisasi |
| Governance/Centralization risk | Risiko dari hak istimewa admin |
| `createVote()` | Fungsi InSecureumDAO yang diperiksa: apakah non-member bisa membuat voting |
| `castVote()` | Fungsi InSecureumDAO yang diperiksa: apakah non-member bisa memberi suara |
| `removeAllMembers()` | Fungsi InSecureumDAO yang diperiksa: apakah ada fungsi kritis yang hilang |

## Isi

### Kenapa laporan audit itu penting
Auditing itu kritis karena **begitu smart contract di-deploy di blockchain, dia menjadi immutable**, sehingga **mustahil memperbaiki error atau kerentanan**. Laporan audit smart contract memastikan isu-isu itu **diidentifikasi dan diselesaikan sebelum deployment**, melindungi baik developer maupun user.

### Tujuh komponen laporan audit
Sebuah laporan biasanya terdiri dari komponen berikut:

| Komponen | Isinya |
| --- | --- |
| **Executive Summary** | Ikhtisar tingkat tinggi soal keamanan dan fungsionalitas contract; merangkum temuan kunci dan rekomendasi secara ringkas. Biasanya jadi **pembuka** laporan |
| **Severity Criteria** | Informasi soal cara baku yang dipakai untuk **mengkategorikan dan memprioritaskan** berbagai tipe kerentanan dan isu yang teridentifikasi selama audit |
| **Scope of Audit** | Mendefinisikan **batas** audit: bagian mana dari smart contract yang diperiksa dan apa saja yang dikecualikan. Bagian ini memastikan semua pihak paham **keterbatasan** audit tersebut |
| **Vulnerabilities Identified & Recommendations** | Salah satu bagian **paling kritis**: daftar kerentanan yang ditemukan selama audit — misalnya reentrancy, overflow, dan logic error. Tiap kerentanan dirinci, dijelaskan dampak potensialnya, dan diberi rekomendasi mitigasinya |
| **Gas Optimization** | Penggunaan gas yang efisien itu krusial karena langsung memengaruhi biaya transaksi. Laporan bisa memuat rekomendasi untuk mengoptimalkan konsumsi gas di kode contract |
| **Conclusion and Summary** | Merangkum temuan keseluruhan dan menegaskan kembali pentingnya menangani isu yang teridentifikasi. Memberi penilaian akhir atas keamanan dan fungsionalitas contract. Di sini juga bisa dinyatakan **disclosure** proses auditnya, **disclaimer** soal tidak adanya niat jahat, atau penegasan ulang proses audit yang sudah dilakukan |

Seluruh contoh untuk tiap komponen di atas diambil dari laporan nyata **Notional audit 2021** di Code4rena: `https://code4rena.com/reports/2021-08-notional#summary`. Contoh gas optimization-nya spesifik menunjuk ke `https://github.com/code-423n4/2021-08-notional-findings/issues/52`.

> [!info] Konteks tambahan (bukan dari slide)
> Perhatikan urutan komponennya — itu bukan urutan acak, tapi **urutan yang dibaca dua audiens berbeda**. Executive Summary dan Conclusion ditulis untuk **manajemen dan investor** yang gak akan baca kodenya. Vulnerabilities dan Gas Optimization ditulis untuk **developer** yang harus memperbaikinya. Severity Criteria dan Scope ada di antara keduanya, dan itu yang **melindungi auditornya sendiri**: kalau nanti ada yang jebol di bagian yang gak masuk scope, dokumen itu yang jadi buktinya.

### Skala severity Code4rena
Selain OWASP dan Trail of Bits (dibahas di [[W09 - Smart Contract Auditing]]), **Code4rena**, sebuah platform keamanan smart contract, menerapkan tingkat severity berikut untuk program mereka:

| Level | Definisi |
| --- | --- |
| **QA (Quality Assurance)** | Mencakup **Low risk** (misal: aset tidak berisiko — state handling, fungsi tidak sesuai spesifikasi, masalah pada komentar) dan **Governance/Centralization risk** (termasuk hak istimewa admin). **Tidak mencakup** Gas optimization, yang diajukan dan dinilai terpisah. Isu **non-critical** (gaya penulisan kode, kejelasan, sintaks, versioning, monitoring off-chain seperti events) **tidak dianjurkan** diajukan |
| **2 — Med** | Aset tidak berisiko langsung, tapi **fungsi protokol atau ketersediaannya bisa terdampak**, atau bisa membocorkan nilai lewat jalur serangan hipotetis dengan asumsi yang dinyatakan, tapi butuh persyaratan eksternal |
| **3 — High** | Aset **bisa dicuri/hilang/dikompromikan secara langsung** — atau tidak langsung, kalau ada jalur serangan valid yang **tidak** mengandalkan asumsi yang mengada-ada (*hand-wavy hypotheticals*) |

> [!info] Konteks tambahan (bukan dari slide)
> Sekarang kamu sudah punya **tiga skala severity berbeda** dari tiga organisasi: Trail of Bits (Informational→High), ConsenSys (Minor→Critical), dan Code4rena (QA/Med/High). Ini bukan pengulangan yang sia-sia — poinnya justru bahwa **tidak ada standar tunggal**. Kalau ditanya di ujian "sebutkan tingkat severity", jawaban yang benar bergantung pada **firma mana yang ditanyakan**. Perhatikan juga garis besar yang sama di ketiganya: pertanyaan pemisah utamanya selalu **"apakah aset bisa hilang secara langsung?"**

## Diagram & Visual
- **Slide 3 — Executive Summary pada laporan Notional (Code4rena)**
  ![[99-Assets/Blockchain/W10-slide03.png]]
- **Slide 4 — Severity Criteria pada laporan Notional**
  ![[99-Assets/Blockchain/W10-slide04.png]]
- **Slide 6 — Scope of Audit pada laporan Notional**
  ![[99-Assets/Blockchain/W10-slide06.png]]
- **Slide 7 — Vulnerabilities Identified & Recommendations pada laporan Notional**
  ![[99-Assets/Blockchain/W10-slide07.png]]
- **Slide 8 — bagian Gas Optimization dan contoh temuannya (issue #52)**
  ![[99-Assets/Blockchain/W10-slide08.png]]
  ![[99-Assets/Blockchain/W10-slide08b.png]]
- **Slide 9 — Conclusion and Summary pada laporan Notional**
  ![[99-Assets/Blockchain/W10-slide09.png]]

> [!warning] Deck ini **hampir seluruhnya screenshot**. Tiap komponen laporan dijelaskan satu paragraf lalu ditunjukkan wujud aslinya lewat tangkapan layar laporan Notional. Artinya untuk melihat **bentuk nyata** tiap bagian laporan, gambar-gambar di atas itu isinya — teksnya gak bisa diekstrak. Kalau tugasnya nulis laporan audit, buka gambar-gambar ini sebagai contoh format.

## Hands-on
Deck ini isinya lebih banyak latihan daripada teori. Ada dua tugas audit.

**1. Audit `SimpleAuction` (slide 10).**
- Analisis smart contract bernama `SimpleAuction.sol`
- **Ada minimal 4 isu** di dalamnya
- Boleh pakai tool apa pun yang kamu punya (lihat Slither/Mythril/Echidna di [[W09 - Smart Contract Auditing]])
- Setelah kerentanan diidentifikasi, **susun laporan audit yang detail**
- Kode: `https://pastebin.com/qsEhatXH`
- Referensi dari speaker notes dosen: `https://medium.com/@bugbountydegen/how-to-use-slither-to-audit-smart-contracts-ff78ee959dd9`

**2. Audit `InSecureumDAO` (slide 11–15).**
- Analisis smart contract bernama `InSecureumDAO.sol`
- Identifikasi kelemahan dan kerentanan, pakai pertanyaan pemandu di bawah
- Setelah itu, susun laporan audit yang detail
- Kode: `https://pastebin.com/4u5M7QVx`
- Referensi dari speaker notes dosen: `https://github.com/x676f64/secureum-mind_map/blob/master/quizzes/7.%20Audit%20Fndings%20101.md`

**Empat pertanyaan pemandu untuk InSecureumDAO:**

| # | Pertanyaan |
| --- | --- |
| 1 | Apakah fungsi **`createVote()`** mencegah non-member membuat voting? |
| 2 | Apakah fungsi **`castVote()`** mencegah non-member memberikan suara? |
| 3 | Bisakah seseorang memasukkan **`_voteId` yang duplikat** ke fungsi `createVote()`? |
| 4 | Apakah fungsi **`removeAllMembers()`** kehilangan sebuah fungsi kritis? Kalau iya, apa yang hilang? |

> [!info] Konteks tambahan (bukan dari slide)
> Keempat pertanyaan itu bukan pertanyaan acak — mereka **peta kategori temuan** dari materi sebelumnya. Nomor 1 dan 2 itu **access control** ([[W08 - Smart Contract Pitfalls]], kategori Trail of Bits: Access Controls). Nomor 3 itu **data validation**. Nomor 4 itu **missed modifier / fungsi kritis yang lupa diproteksi** — pola yang sama persis dengan Parity `initWallet` dan HospoWise `burn()`. Kalau kamu bisa menamai kategori tiap temuan, laporan auditmu langsung naik kelas.

## Pertanyaan Terbuka
- Deck ini **gak punya bagian teori "cara menulis"** yang konkret — dia menjelaskan *apa* isi tiap komponen laporan, bukan *bagaimana* menulisnya (panjangnya, nada bahasanya, tingkat detail per temuan). Contoh formatnya cuma berupa screenshot Notional.
- Format penomoran severity Code4rena aneh: ada "2 — Med" dan "3 — High", tapi **gak ada "1"**. Kemungkinan besar QA yang menempati posisi 1, tapi slide gak menyatakannya.
- `SimpleAuction` disebut punya "**setidaknya** 4 isu" — jumlah pastinya gak diberikan, jadi gak ada kunci jawaban untuk mengecek pekerjaan sendiri.
- **Gak ada template laporan audit yang diberikan.** Kalau tugas akhirnya menyusun laporan, strukturnya harus dibangun sendiri dari tujuh komponen di atas.
- Materi berhenti di sini (9 deck). Kalau kelasnya 13 pertemuan, kemungkinan ada sesi review/UAS atau presentasi tugas yang PPT-nya gak ada.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Blockchain]]
- [[W09 - Smart Contract Auditing]]
- [[W08 - Smart Contract Pitfalls]]
- [[W06 - Smart Contract Standard]]
- [[Blockchain - Review dan Glosari]]
