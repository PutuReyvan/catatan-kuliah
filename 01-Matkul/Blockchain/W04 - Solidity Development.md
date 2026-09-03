---
matkul: Blockchain
minggu: 4
sks: 2
sumber: Smart Contract and Solidity.pptx
tags: [kuliah/blockchain, minggu/w04]
status: draft
diproses: 2026-09-03
---

# W04 — Solidity Development

> [!note] Slide 2 nyatakan deck ini dibawakan dalam **dua sesi terpisah (S05 dan S07)**, dan dosen mengatur sendiri proporsinya. Jadi materi di note ini kemungkinan dibagi jadi dua pertemuan di kelas.

## Ringkasan
> - Siklus hidup contract ada tiga: **compile** (Solidity → bytecode EVM), **execute** (cuma jalan kalau dipanggil transaksi dari EOA), **terminate** (`SELFDESTRUCT`, dan itu pun harus sengaja diprogram).
> - Contract itu **single-threaded**: dieksekusi berurutan, bukan paralel. Dan **dormant** — diam sampai ada transaksi yang membangunkannya.
> - Solidity punya tiga tipe objek: **contract**, **interface** (cuma deklarasi), **library** (deploy sekali, dipakai contract lain lewat `delegatecall`).
> - Function punya empat **visibility** (public/external/internal/private) dan tiga **behaviour** (view/pure/payable). Ditambah **modifier** untuk access control seperti `onlyOwner`.
> - Error handling: `assert` untuk kondisi internal, `require` untuk validasi input. Kalau error, **semua perubahan state di-revert** — transaksinya atomik.
> - **Memanggil contract lain itu berguna tapi berbahaya.** Paling aman kalau contract-nya kamu yang bikin sendiri; paling berbahaya pakai raw call.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| Bytecode | Kode level rendah hasil kompilasi Solidity, yang dimengerti EVM |
| Contract creation transaction | Transaksi khusus ke alamat `0x0` untuk men-deploy contract |
| EOA | Externally Owned Account; satu-satunya yang bisa memicu eksekusi contract |
| `SELFDESTRUCT` | Opcode EVM untuk menghapus kode dan state internal contract |
| Interface | Seperti contract, tapi fungsinya cuma dideklarasikan, gak didefinisikan |
| Library | Contract yang dideploy sekali dan dipakai contract lain lewat `delegatecall` |
| Constructor | Fungsi yang jalan sekali saat contract dibuat, untuk inisialisasi state |
| Function modifier | Kondisi yang bisa ditempel ke banyak fungsi sekaligus; basis access control |
| `view` / `constant` | Fungsi yang berjanji gak mengubah state |
| `pure` | Fungsi yang gak baca **dan** gak tulis variabel di storage |
| `payable` | Fungsi yang bisa menerima pembayaran masuk |
| `require` | Cek input; hentikan eksekusi dengan error kalau gagal |
| `assert` | Cek kondisi internal yang seharusnya selalu benar |
| Event / `emit` | Mekanisme log di transaction receipt yang bisa "ditonton" DApp |
| `indexed` | Keyword yang bikin nilai event masuk hash table yang bisa dicari/difilter |
| Raw call | `call`, `callcode`, `delegatecall`, `send` — paling fleksibel, paling berbahaya |

## Isi

### Siklus hidup smart contract

**1. Compilation.** Smart contract ditulis dalam bahasa tingkat tinggi seperti Solidity, lalu **harus dikompilasi jadi bytecode level rendah** supaya bisa jalan di EVM. Setelah dikompilasi, contract dideploy ke platform Ethereum lewat **contract creation transaction** khusus — caranya dengan mengirim contract itu ke alamat khusus pembuatan contract, yaitu **`0x0`**.

Tiap contract mendapat alamat Ethereum yang **diturunkan dari transaksi pembuatannya, akun asalnya, dan nonce**. Alamat itu bisa dipakai untuk mengirim dana ke contract atau memanggil fungsinya.

**2. Execution.** Contract **cuma jalan kalau dipanggil oleh transaksi dari EOA**. Sebuah contract boleh memanggil contract lain, tapi **pemicu awalnya harus tetap transaksi dari EOA**. Contract tetap dorman sampai diaktifkan transaksi, entah langsung atau lewat rantai pemanggilan.

Poin yang gampang kelewat: smart contract dieksekusi **secara berurutan, bukan paralel**, seperti mesin single-threaded.

**3. Termination.** Kode contract **gak bisa diubah**, tapi bisa dihapus memakai opcode `SELFDESTRUCT`. Detailnya:
- Menghapus contract akan menghapus kode dan state internalnya, menyisakan akun kosong.
- Transaksi yang dikirim ke alamat contract yang sudah dihapus **gak menghasilkan eksekusi kode**.
- `SELFDESTRUCT` memberikan **gas refund**, sebagai insentif untuk melepas sumber daya jaringan.
- **Menghapus contract gak menghapus riwayat transaksinya**, karena blockchain itu immutable.
- Kemampuan `SELFDESTRUCT` **harus sengaja diprogram** oleh pembuat contract. Kalau contract-nya gak punya opcode ini atau opcode-nya gak bisa dijangkau, **contract itu gak bisa dihapus selamanya**.

> [!info] Konteks tambahan (bukan dari slide)
> Analogi siklus hidup: deploy contract itu kayak **masang vending machine di ruang publik**. Setelah tertanam, isinya gak bisa kamu ubah — mau ganti harga pun gak bisa. Dia diam aja sampai ada orang masukin koin (transaksi). Dan kalau kamu gak pasang lubang kunci di belakangnya waktu bikin (`SELFDESTRUCT`), mesin itu bakal berdiri di situ selamanya, bahkan setelah kamu sendiri lupa pernah masang.

### Tipe data Solidity
| Tipe | Keterangan |
| --- | --- |
| `bool` | Nilai boolean, true/false, dengan operator logika |
| `int` / `uint` | Integer bertanda (`int`) dan tak bertanda (`uint`), dideklarasikan dengan kelipatan 8 bit dari `int8` sampai `uint256` |
| `fixed` / `ufixed` | Bilangan fixed-point |
| `address` | Alamat Ethereum 20 byte. Punya member function penting: **`balance`** (mengembalikan saldo akun) dan **`transfer`** (mengirim ether ke akun) |
| `bytes` (fixed) | Array byte berukuran tetap |
| `bytes` / `string` (dynamic) | Array byte berukuran variabel |
| `enum` | Tipe buatan user untuk mengenumerasi nilai diskret: `enum NAMA {LABEL1, LABEL2, ...}` |
| Arrays | Array dari tipe apa pun, fixed maupun dynamic |

### Global variable bawaan
Variabel yang selalu tersedia di dalam contract:

| Variable | Type | Deskripsi |
| --- | --- | --- |
| `msg.sender` | address | Alamat akun yang mengirim transaksi saat ini |
| `msg.value` | uint | Jumlah Ether yang dikirim bersama transaksi saat ini |
| `block.coinbase` | address | Alamat miner yang menambang block saat ini |
| `block.difficulty` | uint | Difficulty block saat ini |
| `block.gaslimit` | uint | Gas maksimum yang boleh dipakai di block saat ini |
| `block.number` | uint | Nomor block saat ini |
| `block.timestamp` | uint | Timestamp block saat ini |
| `now` | uint | Alias untuk `block.timestamp` |
| `tx.origin` | address | Alamat akun yang awalnya membuat transaksi (pengirim transaksi pertama di call chain) |
| `tx.gasprice` | uint | Gas price (dalam Wei) untuk transaksi saat ini |

> [!info] Konteks tambahan (bukan dari slide)
> Ingat-ingat: `block.difficulty`, `block.timestamp`, dan `block.number` di tabel ini **kelihatan seperti sumber keacakan yang bagus**. Bukan. Ketiganya bisa dilihat dan sebagian bisa dipengaruhi miner. Itu persis jebakan **insecure randomness** yang dibahas di [[W07 - Smart Contract Pitfalls]].

### Contract, interface, dan library
Tipe data utama Solidity adalah **`contract`**. Mirip objek di bahasa berorientasi objek, contract adalah kontainer yang berisi data dan method. Selain `contract`, Solidity punya dua tipe objek lain:

- **Interface** — strukturnya persis seperti contract, kecuali **tidak ada satu pun fungsi yang didefinisikan; semuanya cuma dideklarasikan**. Saat diwariskan, tiap fungsi yang dideklarasikan interface **wajib didefinisikan oleh child**-nya.
- **Library** — contract yang dimaksudkan untuk **dideploy sekali saja** dan dipakai contract lain, memakai method **`delegatecall`**.

### Mendefinisikan function
Sintaks deklarasi fungsi di Solidity:

```solidity
function FunctionName([parameters]) {public|private|internal|external}
    [pure|view|payable] [modifiers] [returns (return types)]
```

**Nama fungsi** dipakai untuk memanggil fungsi itu lewat transaksi (dari EOA), dari contract lain, atau dari dalam contract itu sendiri. Satu fungsi di tiap contract boleh didefinisikan sebagai **fallback function** (keyword `fallback`) atau **receive ether function** (keyword `receive`). Setelah nama, ditulis **parameter** beserta nama dan tipenya.

**Visibility — siapa yang boleh memanggil:**

| Visibility | Siapa yang boleh memanggil |
| --- | --- |
| **Public** | Default. Bisa dipanggil contract lain, transaksi EOA, atau dari dalam contract sendiri |
| **External** | Seperti public, tapi **gak bisa dipanggil dari dalam contract** kecuali diawali `this` |
| **Internal** | Cuma bisa diakses dari dalam contract. Bisa dipanggil contract turunan (yang mewarisi) |
| **Private** | Seperti internal, tapi **gak bisa dipanggil contract turunan** |

**Behaviour — apa yang boleh dilakukan fungsi:**

| Keyword | Artinya |
| --- | --- |
| `constant` / `view` | Berjanji **tidak mengubah state** apa pun |
| `pure` | **Gak baca dan gak tulis** variabel di storage. Cuma boleh beroperasi pada argumen dan return data |
| `payable` | Bisa **menerima pembayaran masuk** |

> [!info] Konteks tambahan (bukan dari slide)
> Cara ngingetnya: `pure` < `view` < biasa, dari yang paling gak boleh nyentuh apa-apa sampai yang bebas. `pure` itu kayak kalkulator — cuma olah angka yang kamu kasih. `view` kayak orang yang boleh **baca** buku besar tapi gak boleh nulis. Fungsi biasa boleh nulis. Dan `payable` itu urusan lain lagi: tanpa `payable`, fungsi bakal **menolak** ether yang dikirim ke dia.

### Constructor
Saat sebuah contract dibuat, dia juga menjalankan **fungsi constructor** kalau ada, untuk menginisialisasi state contract. Constructor dijalankan **dalam transaksi yang sama dengan pembuatan contract**, tapi cuma dipakai **sekali**. Constructor itu opsional — gak semua contract perlu punya.

### `selfdestruct` dan function modifier
Contract dihancurkan lewat opcode EVM khusus bernama **`SELFDESTRUCT`**. Dulu namanya `SUICIDE`, tapi nama itu di-deprecate karena asosiasi negatifnya. Di Solidity, opcode ini diekspos sebagai built-in function `selfdestruct`, yang menerima **satu argumen: alamat penerima sisa saldo ether** di akun contract itu.

Yang penting soal proteksinya: kalau `destroy` dipanggil dari alamat selain `owner`, panggilan itu akan gagal. Tapi kalau alamat yang sama dengan yang disimpan di `owner` oleh constructor yang memanggilnya, contract akan self-destruct dan mengirim sisa saldo ke alamat owner.

> [!warning] Slide 18 mencatat: **per Solidity 0.8.18, `SELFDESTRUCT` sudah deprecated.**

Proteksi tadi dibuat pakai **function modifier**. Modifier paling sering dipakai untuk membuat kondisi yang berlaku ke banyak fungsi dalam satu contract. Contoh dari slide, modifier bernama `onlyOwner`: dia memasang kondisi pada fungsi mana pun yang dia modifikasi, mensyaratkan bahwa alamat yang tersimpan sebagai owner contract **sama dengan alamat `msg.sender` transaksi itu**.

Slide bilang, baca modifier ini sebagai kalimat: **"HANYA OWNER YANG BISA MENGHANCURKAN CONTRACT INI."** Ini adalah **design pattern dasar untuk access control**.

### Inheritance
Objek contract di Solidity mendukung **inheritance**, yaitu mekanisme memperluas base contract dengan fungsionalitas tambahan. Dengan konstruksi ini, contract `Child` mewarisi semua method, fungsionalitas, dan variabel dari `Parent`.

Solidity juga mendukung **multiple inheritance**, yang ditulis dengan nama contract dipisah koma. Dengan begitu `Child` mewarisi semua dari `Parent1` **dan** `Parent2`.

### Error handling: `assert` dan `require`
Sebuah pemanggilan contract bisa berhenti dan mengembalikan error. Error handling di Solidity ditangani empat fungsi: **`assert`, `require`, `revert`, dan `throw`** (yang terakhir sudah deprecated).

Saat contract berhenti dengan error:
- **Semua perubahan state** (perubahan variabel, saldo, dll) **di-revert**.
- Reversion ini merambat ke atas sepanjang rantai pemanggilan contract kalau ada beberapa contract yang terpanggil.
- Ini memastikan transaksi bersifat **atomik**: selesai sepenuhnya, atau di-revert sepenuhnya.

Bedanya `assert` dan `require`: keduanya bekerja dengan cara yang sama — mengevaluasi kondisi dan menghentikan eksekusi dengan error kalau kondisinya false. Yang membedakan adalah **konvensi**:

- **`assert`** dipakai kalau hasilnya **diharapkan selalu true** — untuk menguji **kondisi internal**.
- **`require`** dipakai untuk menguji **input** (seperti argumen fungsi atau field transaksi), menetapkan ekspektasi kita atas kondisi tersebut.

```solidity
require(msg.sender == owner, "Only the contract owner can call this function");
assert(totalSupply == balances_from + balances_to);
```

> [!info] Konteks tambahan (bukan dari slide)
> Cara gampang mbedain: **`require` itu satpam di pintu depan** — dia curiga sama orang luar, tugasnya nolak input yang gak beres. **`assert` itu alarm kebakaran di dalam gedung** — kalau dia bunyi, artinya ada yang salah di logikamu sendiri, bukan salah tamunya. Kalau `assert` pernah gagal, yang perlu diperbaiki adalah kodemu, bukan input-nya.

### Events
Saat sebuah transaksi selesai (berhasil atau gagal), dia menghasilkan **transaction receipt**. Receipt ini berisi **log entries** yang memberi informasi tentang aksi-aksi yang terjadi selama eksekusi transaksi.

Light client dan layanan DApp bisa **"menonton" (watch)** event tertentu untuk:
- Melaporkannya ke user interface
- Mengubah state aplikasi supaya mencerminkan event di contract yang mendasarinya

Keyword **`indexed`** membuat nilainya jadi bagian dari tabel terindeks (hash table) yang bisa **dicari atau difilter** oleh aplikasi. Untuk memicu data event masuk ke transaction log, dipakai keyword **`emit`**.

> [!info] Konteks tambahan (bukan dari slide)
> Kenapa events penting: smart contract gak bisa "menelepon" frontend-mu. Jadi polanya dibalik — contract **berteriak ke log** (`emit`), dan frontend **menguping log** itu. Notifikasi "transaksi berhasil" di aplikasi crypto mana pun sebenarnya cuma frontend yang lagi nungguin event.

### Memanggil contract lain: berguna tapi berbahaya
Memanggil contract lain dari dalam contract-mu itu sangat berguna tapi **berpotensi berbahaya**. Risikonya muncul dari:
- **Ketidakpastian soal contract yang kamu panggil.**
- **Ketidakpastian soal contract yang memanggil contract-mu.**

Kamu mungkin berharap sebagian besar yang berinteraksi denganmu itu EOA. Tapi **gak ada yang menghalangi contract yang rumit dan mungkin jahat untuk memanggil kodemu**. Sebaliknya, contract-mu mungkin memanggil contract lain yang gak dikenal dan berpotensi berbahaya.

Tiga cara memanggil contract lain, diurutkan dari yang paling aman:

1. **Membuat instance contract baru** — cara paling aman adalah kalau contract itu **kamu sendiri yang buat**, karena dengan begitu kamu pasti tau interface dan perilakunya.
2. **Mengalamati instance yang sudah ada** — kamu meng-*cast* alamat sebuah instance contract yang sudah ada, menerapkan interface yang kamu ketahui ke instance itu. Karena itu **sangat krusial** kamu tau dengan pasti bahwa instance yang kamu alamati memang benar bertipe seperti yang kamu asumsikan.
3. **Raw call** — Solidity menyediakan fungsi level lebih rendah lagi untuk memanggil contract lain. Fungsi-fungsi ini berkorespondensi langsung dengan opcode EVM bernama sama, dan memungkinkan kita menyusun pemanggilan contract-ke-contract secara manual. Ini mekanisme **paling fleksibel dan paling berbahaya**.

## Diagram & Visual
- **Slide 12 — contoh deklarasi function di Solidity**
  ![[99-Assets/Blockchain/W04-slide12.png]]
- **Slide 16 — contoh kode constructor**
  ![[99-Assets/Blockchain/W04-slide16.png]]
- **Slide 17 — contoh kode `selfdestruct`**
  ![[99-Assets/Blockchain/W04-slide17.png]]
- **Slide 19 — contoh modifier `onlyOwner` dan pemakaiannya**
  ![[99-Assets/Blockchain/W04-slide19.png]]
  ![[99-Assets/Blockchain/W04-slide19b.png]]
- **Slide 20 — sintaks inheritance tunggal dan multiple inheritance**
  ![[99-Assets/Blockchain/W04-slide20.png]]
  ![[99-Assets/Blockchain/W04-slide20b.png]]
- **Slide 23 — contoh deklarasi event dengan keyword `indexed`**
  ![[99-Assets/Blockchain/W04-slide23.png]]
- **Slide 24 — contoh pemakaian `emit`**
  ![[99-Assets/Blockchain/W04-slide24.png]]
- **Slide 26 — kode membuat instance contract baru**
  ![[99-Assets/Blockchain/W04-slide26.png]]
- **Slide 27 — kode mengalamati instance contract yang sudah ada**
  ![[99-Assets/Blockchain/W04-slide27.png]]
- **Slide 28 — kode raw call**
  ![[99-Assets/Blockchain/W04-slide28.png]]

> [!warning] **Hampir semua contoh kode di deck ini berupa gambar, bukan teks.** Artinya kode di slide 12, 16, 17, 19, 20, 23, 24, 26, 27, dan 28 gak bisa di-copy dari note ini — harus dilihat dari gambar di atas atau dari PPT aslinya. Slide 3 juga kosong total (gak ada teks maupun gambar yang bisa diekstrak).

## Rumus / Sintaks

Deklarasi function:
```solidity
function FunctionName([parameters]) {public|private|internal|external}
    [pure|view|payable] [modifiers] [returns (return types)]
```

Error handling:
```solidity
require(msg.sender == owner, "Only the contract owner can call this function");
assert(totalSupply == balances_from + balances_to);
```

Alamat khusus deploy contract:
```
0x0
```

## Pertanyaan Terbuka
- Slide 13 nyebut fallback dan receive function "see previous session", tapi **gak ada deck di matkul ini yang membahasnya**. Kemungkinan ada materi sesi yang belum masuk, atau dijelaskan lisan di kelas. Perlu ditanyakan — fallback function itu jantung dari reentrancy attack di [[W07 - Smart Contract Pitfalls]].
- `revert` disebut sebagai salah satu dari empat fungsi error handling, tapi **cuma `assert` dan `require` yang dijelaskan**. Bedanya `revert` dengan `require` gak dibahas.
- `delegatecall` disebut dua kali (di library dan di raw call) tapi gak pernah dijelaskan bedanya dengan `call` biasa. Padahal ini yang dieksploitasi di Parity hack yang muncul di [[W06 - The Hardhat Framework]].
- Tipe `fixed`/`ufixed` disebut di tabel tipe data, padahal di Solidity versi sekarang tipe ini belum didukung penuh. Perlu diverifikasi versi Solidity yang dipakai di praktikum.
- `SELFDESTRUCT` sudah deprecated sejak 0.8.18 (diakui slide sendiri). Perlu dikonfirmasi apakah ini masih diujikan.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Blockchain]]
- [[W03 - Wallets and Transactions]]
- [[W05 - Smart Contract Standard]]
- [[W07 - Smart Contract Pitfalls]]
- [[Blockchain - Review dan Glosari]]
