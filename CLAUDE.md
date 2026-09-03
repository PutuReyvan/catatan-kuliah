# CLAUDE.md — Vault Catatan Kuliah

## Konteks

Vault Obsidian ini adalah catatan kuliah Teknik Informatika. Sumber utamanya slide
PPT dari dosen.

Volume yang harus diantisipasi:
- 1 matkul 2 SKS ≈ 13 file PPT (per pertemuan)
- 1 matkul 4 SKS ≈ 26 file PPT
- Total per semester bisa 100–150 file PPT

**Tujuan vault ini adalah bahan belajar yang bisa dibaca cepat di HP, bukan arsip
slide.** Kalau sebuah note cuma nyalin ulang isi slide dalam bentuk markdown, note
itu gagal. Note yang bagus itu bikin orang paham tanpa buka PPT-nya lagi.

Vault ini disinkronkan ke Android lewat Git (private repo) + GitSync. Artinya:
setiap perubahan harus di-commit supaya kebaca di HP.

---

## Struktur Folder

```
/
├── CLAUDE.md               <- file ini
├── 00-Inbox/               <- PPT mentah dibuang ke sini dulu
│   └── <Matkul>/
├── 01-Matkul/              <- output note per pertemuan
│   └── <Matkul>/
│       ├── _<Matkul>.md    <- index / MOC matkul
│       └── W01 - <Judul>.md
├── 02-Konsep/              <- note atomik, konsep lintas matkul
├── 03-Ujian/               <- rangkuman buat UTS/UAS
├── 99-Assets/              <- gambar & diagram hasil extract
│   └── <Matkul>/
└── .progress.md            <- tracker file mana yang udah diproses
```

Nama matkul pakai bentuk pendek dan konsisten: `Jarkom`, `Basdat`, `Struktur-Data`,
`Kriptografi`. Sekali dipilih, jangan diganti-ganti.

---

## Konvensi Penamaan

- Note pertemuan: `W01 - Pengantar Jaringan.md` (dua digit, selalu, biar urut)
- Judul diambil dari slide judul PPT-nya, bukan dari nama file PPT (nama file dosen
  biasanya berantakan: `PERTEMUAN 1 fix REVISI(2).pptx`)
- Note konsep: nama konsepnya aja, `TCP Handshake.md`, `Normalisasi 3NF.md`
- Gambar: `99-Assets/<Matkul>/W01-slide07.png`
- Jangan pakai spasi ganda, karakter aneh, atau emoji di nama file

---

## Template Note Pertemuan

Setiap note pertemuan WAJIB pakai struktur ini persis:

```markdown
---
matkul: <Nama Matkul>
minggu: <angka>
sks: <angka>
sumber: <nama file PPT asli>
tags: [kuliah/<matkul-slug>, minggu/w<NN>]
status: draft
diproses: <YYYY-MM-DD>
---

# W<NN> — <Judul Pertemuan>

## Ringkasan
> 3–5 poin. Apa inti pertemuan ini. Ditulis supaya bisa dibaca 30 detik
> sebelum kelas mulai.

## Konsep Kunci
| Istilah | Penjelasan |
| --- | --- |
| ... | ... |

## Isi

### <Sub-topik 1>
<Penjelasan naratif, bukan bullet salin-tempel dari slide. Gabungkan slide-slide
yang membahas satu topik jadi satu bagian yang nyambung.>

### <Sub-topik 2>
...

## Diagram & Visual
- **Slide 7 — <deskripsi singkat apa yang digambarkan>**
  ![[99-Assets/<Matkul>/W01-slide07.png]]

## Rumus / Sintaks
<Kalau ada. Pakai code block atau LaTeX. Kalau gak ada, hapus section ini.>

## Pertanyaan Terbuka
- <Hal yang di slide disinggung tapi gak dijelasin, atau yang kelihatannya bakal
  keluar di ujian tapi slide-nya kurang detail>

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_<Matkul>]]
- [[W<NN-1> - <judul sebelumnya>]]
- [[<konsep terkait di 02-Konsep>]]
```

### Aturan mutlak soal template

1. **Section `## Versi Gue` tidak boleh pernah ditimpa, diisi, atau dihapus.** Kalau
   note-nya diproses ulang, isi section itu harus dipertahankan apa adanya. Ini
   catatan tangan pemilik vault dan itu bagian paling penting dari note-nya.
2. `status: draft` diganti jadi `status: reviewed` hanya oleh manusia, bukan olehmu.

---

## Aturan Konten

**Jangan mengarang.** Kalau slide isinya cuma judul dan gambar tanpa teks yang bisa
diekstrak, tulis apa adanya dan kasih tanda:

```
> [!warning] Slide 12–14 cuma gambar tanpa teks. Perlu dibuka manual.
```

Jangan ditambahin penjelasan dari pengetahuan umum seolah-olah itu dari slide dosen.
Kalau kamu nambahin konteks dari luar slide karena penjelasannya perlu, tandai
eksplisit:

```
> [!info] Konteks tambahan (bukan dari slide)
```

Ini penting karena yang diujikan adalah versi dosennya, bukan versi textbook.

**Pertahankan istilah aslinya.** Slide kuliah di Indonesia sering campur: badan
kalimatnya Indonesia tapi istilah teknisnya Inggris. Jangan diterjemahin.
`three-way handshake` tetap `three-way handshake`, bukan "jabat tangan tiga arah".

**Baca speaker notes.** Kalau file PPT punya speaker notes, isinya sering lebih
berharga daripada slide-nya. Masukkan ke bagian yang relevan.

**Gabungkan, jangan petakan 1:1.** Satu slide bukan berarti satu section. 5 slide
yang membahas satu topik digabung jadi satu penjelasan yang mengalir.

---

## Pipeline Pemrosesan

### Ekstraksi
- Pakai `python-pptx` untuk `.pptx`
- File `.ppt` lama gak bisa dibaca `python-pptx` — konversi dulu pakai
  `libreoffice --headless --convert-to pptx`
- Extract juga gambar yang tertanam ke `99-Assets/<Matkul>/`
- Skip gambar yang jelas dekorasi (logo kampus, background, bullet icon).
  Ambil yang diagram, chart, screenshot, atau tabel.

### Batching
Volume-nya besar, jadi:
1. **Proses satu matkul per satu waktu.** Jangan sekali jalan semua matkul.
2. Setelah setiap matkul kelar, commit, lalu lapor ringkas: berapa note dibuat,
   file mana yang bermasalah, slide mana yang perlu dibuka manual.
3. Update `.progress.md` setiap file selesai, formatnya:
   `- [x] Jarkom/W01 — 2026-09-03`
4. Kalau sesi terputus, **baca `.progress.md` dulu** sebelum mulai. Jangan
   proses ulang yang sudah ada.
5. Kalau note tujuan sudah ada, jangan ditimpa diam-diam. Tanya dulu.

### Index Matkul
Setelah semua pertemuan satu matkul selesai, generate `_<Matkul>.md`:
- Daftar semua pertemuan dengan wikilink dan satu baris deskripsi
- Section "Konsep Utama" yang nge-link ke note di `02-Konsep/`
- Section "Peta Materi" — mana yang nyambung ke mana antar minggu

---

## Git

Repo ini private. Jangan pernah bikin public, jangan push ke remote lain.

`.gitignore` harus berisi minimal:
```
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/plugins/obsidian-git/data.json
conflict-files-obsidian-git.md
.trash/
00-Inbox/
```

`00-Inbox/` di-ignore karena isinya PPT mentah — berat dan gak perlu ikut ke HP.

**Commit per matkul, bukan per file.** 13 commit untuk satu matkul itu bikin
history berisik. Format pesan commit:
```
notes(jarkom): tambah W01-W13
```

Push otomatis setelah commit. Kalau push gagal, bilang — jangan diem, karena
artinya HP gak akan dapet update.

---

## Yang Tidak Perlu Ditanya

Kerjakan langsung tanpa konfirmasi: baca file di `00-Inbox/`, bikin note baru,
bikin folder, extract gambar, commit, push.

## Yang Harus Ditanya Dulu

- Nimpa note yang sudah ada
- Hapus file apa pun
- Ganti nama folder matkul yang sudah dipakai
- Ubah template di file ini
