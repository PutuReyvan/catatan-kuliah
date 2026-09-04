---
matkul: Compilation Techniques
minggu: 6
sks: 3
sumber: Session 06_DFA Minimization.pptx
tags: [kuliah/compiler, minggu/w06]
status: draft
diproses: 2026-09-04
---

# W06 — DFA Minimization

## Ringkasan
> - Untuk BAHASA yang SAMA, bisa ada BANYAK DFA dengan jumlah/pasangan state BERBEDA. DFA punya jumlah state MINIMAL kalau SEMUA state-nya bisa DIBEDAKAN (distinguishable) satu sama lain.
> - Dua state p dan q disebut **distinguishable** kalau ADA string x di mana HANYA SATU dari δ(p,x) atau δ(q,x) yang berada di F (accepting state) — kalau TIDAK ada string seperti itu, p dan q bisa DIGABUNG jadi satu state.
> - **Metode Table-Filling** — tandai pasangan state secara BERTAHAP: mulai dari pasangan (accepting, non-accepting) yang OTOMATIS distinguishable, lalu SECARA REKURSIF tandai pasangan lain yang transisinya mengarah ke pasangan yang SUDAH ditandai.
> - **Metode Partition** — mulai dari DUA kelompok besar (accepting vs non-accepting), lalu PECAH tiap kelompok jadi subgrup lebih kecil selama ADA state dalam kelompok yang transisinya mengarah ke KELOMPOK BERBEDA untuk simbol yang sama — berhenti kalau tidak bisa dipecah lagi.
> - Kedua metode menghasilkan hasil yang SAMA: DFA dengan jumlah state PALING SEDIKIT yang tetap mengenali bahasa yang IDENTIK dengan DFA aslinya.

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Distinguishable states | Dua state yang bisa dibedakan oleh keberadaan string yang membedakan hasil (accept/reject) |
| Equivalent states | Dua state yang TIDAK distinguishable — bisa digabung jadi satu state |
| Table-filling method | Metode minimisasi dengan menandai pasangan state distinguishable secara bertahap |
| Partition method | Metode minimisasi dengan memecah kelompok state berdasarkan kesamaan tujuan transisi |

## Isi

### Kenapa DFA Perlu Diminimalkan?
Untuk BAHASA yang SAMA, bisa ada DFA dengan JUMLAH state yang BERBEDA-BEDA. Contoh: DFA untuk "string biner yang berakhir dengan 1" — bisa dibangun dengan **M1** (4 state), **M2** (3 state), atau **M3** (2 state), SEMUANYA mengenali bahasa yang SAMA PERSIS. DFA minimisasi mencari versi dengan JUMLAH STATE PALING SEDIKIT.

### Distinguishable States
**DFA punya jumlah state MINIMAL kalau SEMUA state-nya bisa DIBEDAKAN (distinguishable).** State p dan q disebut **"distinguishable"** kalau ADA string x, di mana δ(p,x) ada di F (accepting) TAPI δ(q,x) TIDAK ada di F (atau sebaliknya).

**Aturan turunan:** kalau δ(p,a) = p' dan δ(q,a) = q', dan p' & q' DISTINGUISHABLE, maka p dan q JUGA distinguishable (secara transitif lewat satu langkah transisi).

> [!info] Analogi
> Dua state distinguishable itu seperti dua orang KEMBAR yang secara fisik terlihat sama, tapi bisa DIBEDAKAN kalau kamu tanya PERTANYAAN yang tepat — misalnya "siapa nama ibumu?" — kalau jawaban mereka BEDA, mereka distinguishable (bisa dibedakan). Kalau TIDAK ADA pertanyaan apa pun yang bisa membedakan jawaban mereka (SEMUA jawaban mereka selalu SAMA persis, untuk pertanyaan apa pun), mereka EQUIVALENT — dan untuk keperluan praktis, kamu bisa memperlakukan mereka sebagai "SATU ORANG YANG SAMA" (digabung jadi satu state).

### Metode 1: Table-Filling Algorithm
```
Begin
  for p in F and q in Q-F do mark(p, q);   // tandai SEMUA pasangan accepting-nonaccepting
  for setiap pasangan (p,q) di F×F atau (Q-F)×(Q-F) do
    if untuk input a, ((p,a), (q,a)) sudah ditandai then
      mark(p, q)
      // tandai secara REKURSIF semua pasangan terkait
  for semua simbol input a do
    tempatkan semua (p,q) di daftar untuk ((p,a),(q,a)), kecuali kalau (p,a)=(q,a)
end
```

**Basis:** kalau p adalah accepting state dan q NON-accepting, maka pasangan {p,q} DISTINGUISHABLE.
**Induksi:** misal p dan q adalah state di mana untuk suatu simbol input a, r = δ(p,a) dan s = δ(q,a) adalah pasangan state yang SUDAH DIKETAHUI distinguishable. Maka {p,q} JUGA pasangan distinguishable.

**Contoh proses table-filling** (untuk DFA 8 state A-H): dimulai dengan menandai (X) semua pasangan accepting-nonaccepting, lalu SECARA BERTAHAP menandai pasangan lain berdasarkan aturan induksi, sampai TIDAK ADA lagi pasangan baru yang bisa ditandai — pasangan yang TERSISA TIDAK ditandai (ditandai O atau kosong) berarti EQUIVALENT dan bisa DIGABUNG.

**Hasil contoh:** state {A,E}, {B,H}, {C}, {D,F}, {G} — masing-masing kelompok ini digabung jadi SATU state di DFA minimal.

### Metode 2: Partition Method
1. Partisi himpunan state jadi DUA kelompok:
   - **G1** = himpunan accepting state.
   - **G2** = himpunan non-accepting state.
2. Untuk SETIAP kelompok G baru: partisi G jadi SUBGRUP sedemikian rupa sehingga state s1 dan s2 berada di kelompok yang SAMA **JIKA DAN HANYA JIKA**, untuk SEMUA simbol input a, state s1 dan s2 punya transisi ke state yang berada di kelompok yang SAMA.
3. **Start state** DFA minimal = kelompok yang mengandung start state DFA asli.
4. **Accepting state** DFA minimal = kelompok-kelompok yang mengandung accepting state DFA asli.

**Contoh Partition Method (DFA 3 state):**
```
G1 = {2}       (accepting)
G2 = {1,3}     (non-accepting)

Cek apakah G2 bisa dipartisi lagi:
move(1,a) = 2    move(1,b) = 3
move(3,a) = 2    move(3,b) = 3
-> state 1 dan 3 punya TUJUAN TRANSISI SAMA (ke grup yang sama) untuk a MAUPUN b
-> G2 TIDAK BISA dipartisi lagi

Hasil: DFA minimal punya 2 state: {1,3} dan {2}
```

**Contoh Partition Method Kedua (4-state DFA):**
```
0-equivalence (partisi awal accepting/non-accepting): {1,2,3} {4}
1-equivalence (partisi lebih lanjut): {1,2} {3} {4}

Tidak ada partisi lebih lanjut yang mungkin
Hasil DFA minimal: 3 state -> {1,2}, {3}, {4}
```

> [!info] Analogi
> Partition Method itu seperti mengelompokkan siswa di sekolah berdasarkan JADWAL PELAJARAN mereka. Mulai dari dua kelompok besar: "lulus" (accepting) dan "belum lulus" (non-accepting). Lalu dalam SETIAP kelompok, kamu pecah lagi berdasarkan: "kalau ditanya soal MATA PELAJARAN A, semua siswa di subgrup ini akan LANJUT ke kelompok yang SAMA?" Kalau JAWABANNYA IYA untuk SEMUA mata pelajaran, subgrup itu TIDAK BISA dipecah lagi — mereka semua "berperilaku identik" dan bisa dianggap SATU KELOMPOK SAJA (satu state).

## Diagram & Visual
- **Slide 4 — Tiga DFA berbeda jumlah state (M1, M2, M3) untuk bahasa yang sama**
  (diagram transisi dijelaskan dalam teks — perlu dibuka manual untuk visual lengkap)
- **Slide 7 — DFA 8-state (A-H) sebelum minimisasi**
  (diagram transisi kompleks — perlu dibuka manual)
- **Slide 9-10 — Tabel table-filling untuk DFA 8-state**
  (tabel dengan tanda X untuk pasangan distinguishable — perlu dibuka manual)
- **Slide 11 — DFA hasil minimisasi (state digabung: {A,E}, {B,H}, {C}, {D,F}, {G})**
  (diagram transisi hasil akhir — perlu dibuka manual)

> [!warning] SEMUA diagram transisi (node dan panah berlabel) di deck ini terekstrak sebagai TEKS POSISI (nama state, label transisi) TANPA gambar visual pendampingnya (kemungkinan besar dibuat dengan SmartArt/shape manual di PowerPoint, bukan gambar raster). Struktur logisnya sudah direkonstruksi dari teks di atas, tapi diagram VISUAL lengkap perlu dibuka manual dari file PPT asli.

## Rumus / Sintaks
```
Table-Filling Algorithm:
1. Tandai semua pasangan (p,q) dengan p accepting, q non-accepting
2. Ulangi: untuk setiap pasangan (p,q) belum ditandai,
   kalau ada simbol a dengan (δ(p,a), δ(q,a)) SUDAH ditandai,
   maka tandai (p,q) juga
3. Ulangi langkah 2 sampai tidak ada perubahan
4. Pasangan yang TIDAK PERNAH ditandai = equivalent -> gabung jadi satu state

Partition Method:
1. G1 = accepting states, G2 = non-accepting states
2. Untuk setiap grup G: pecah G jadi subgrup, s1 & s2 sekelompok
   HANYA JIKA untuk semua simbol a, tujuan transisi mereka
   ada di grup yang SAMA
3. Ulangi sampai tidak ada grup yang bisa dipecah lagi
4. Start state baru = grup berisi start state lama
5. Accepting state baru = grup berisi accepting state lama
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **k-equivalence** | Dua state dianggap ekuivalen sampai kedalaman k langkah pengecekan partisi |
| **0-equivalence** | Partisi awal murni berdasarkan status accepting/non-accepting |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa BASIS dari Table-Filling algorithm dimulai dari pasangan (accepting, non-accepting) — bukan pasangan lain. Kaitkan dengan definisi distinguishable ("δ(p,x) di F, δ(q,x) TIDAK di F") — kenapa pasangan accepting-nonaccepting SECARA OTOMATIS memenuhi definisi ini bahkan untuk string KOSONG (x = ε)?
2. **(C4 – Analisis)** Bandingkan Table-Filling Method dan Partition Method. Analisis: meski KEDUANYA menghasilkan DFA minimal yang SAMA, jelaskan perbedaan cara berpikirnya — Table-Filling berangkat dari "mencari yang BERBEDA" (top-down dari pasangan), sementara Partition berangkat dari "mengelompokkan yang SAMA" (bottom-up dari grup besar) — mana yang menurutmu lebih INTUITIF dipahami manusia, dan kenapa?
3. **(C5 – Evaluasi)** Evaluasi hasil DFA minimal M3 (2 state) untuk bahasa "string biner berakhir 1" dibanding M1 (4 state). Kalau KETIGA DFA (M1, M2, M3) mengenali bahasa yang PERSIS SAMA, apa KEUNTUNGAN PRAKTIS memakai versi minimal (M3) dibanding versi lebih besar (M1) dalam konteks IMPLEMENTASI lexical analyzer sungguhan (pertimbangkan memori dan kecepatan)?
4. **(C5 – Evaluasi)** Bandingkan konsep "0-equivalence" dan "1-equivalence" dari contoh partition method kedua di materi. Evaluasi: kenapa proses partisi harus dilakukan BERTAHAP (0-equivalence dulu, baru 1-equivalence, dst.) — apa yang terjadi kalau kita LANGSUNG mencoba memecah state jadi grup FINAL tanpa proses bertahap ini?
5. **(C6 – Cipta)** Kerjakan latihan #2 dari slide 16 memakai Partition Method — untuk DFA berikut, tentukan DFA minimal ekuivalennya (tunjukkan langkah G1/G2 awal, lalu proses pemecahan grup sampai stabil):
   ```
   State | 0 | 1
   A     | B | C
   B     | A | C
   C     | D | E
   D     | B | F
   E     | G | C
   *F    | G | E
   G     | A | F
   ```

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W05 - RE ke DFA]]
- [[W07 - Context-Free Grammar]]
- [[Compiler - Review dan Glosari]]
