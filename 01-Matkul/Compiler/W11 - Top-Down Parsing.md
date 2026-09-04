---
matkul: Compilation Techniques
minggu: 11
sks: 3
sumber: Session 11-12-13_Top-Down Parsing.pptx
tags: [kuliah/compiler, minggu/w11]
status: draft
diproses: 2026-09-04
---

# W11 — Top-Down Parsing

> [!note] Deck ini secara eksplisit menggabungkan **tiga sesi (Session 11, 12, dan 13)** dalam satu file: Recursive Descent Parsing, Predictive Parsing (LL), dan Construction of Parsing Table. Note ini tetap dinomori W11 mengikuti konvensi vault untuk deck gabungan multi-sesi.

## Ringkasan
> - Dua jenis Top-Down Parser: **Recursive-Descent Parsing** (BUTUH backtracking, tidak efisien, jarang dipakai) dan **Predictive Parsing** (TANPA backtracking, efisien, butuh grammar LL(1)).
> - **Predictive Parser** memilih production rule secara UNIK cukup dengan melihat SATU token saat ini — tapi grammar harus SUDAH bebas left recursion dan sudah di-left-factor DULU (TIDAK ADA JAMINAN 100% jadi LL(1) meski sudah ditransformasi).
> - **LL(1) Parser** = "L" pertama (scan kiri-ke-kanan), "L" kedua (hasilkan leftmost derivation), "1" (pakai SATU token lookahead). Dibangun dari **FIRST SET** dan **FOLLOW SET**, dipakai untuk MEMBANGUN parsing table M[A,a].
> - Grammar LL(1) VALID kalau memenuhi SYARAT KETAT: dua production untuk non-terminal yang sama TIDAK BOLEH punya FIRST yang overlap, dan KALAU salah satu bisa turunkan ε, yang lain TIDAK BOLEH punya FIRST yang overlap dengan FOLLOW. Grammar left-recursive dan ambigu OTOMATIS BUKAN LL(1).
> - Ada 4 teknik error recovery: **Panic-Mode** (pakai FOLLOW SET sebagai synchronizing token), **Phrase-Level** (routine error khusus per entri kosong), **Error Productions**, **Global Correction** (TIDAK PRAKTIS).

## Konsep Kunci
| Istilah | Penjelasan Singkat |
| --- | --- |
| Recursive-Descent Parsing | Top-down parsing dengan backtracking, satu prosedur per non-terminal |
| Predictive Parsing | Top-down parsing TANPA backtracking, pilih production dari 1 lookahead token |
| LL(1) Parser | Parser table-driven: Left-to-right scan, Leftmost derivation, 1 lookahead |
| FIRST(α) | Himpunan terminal yang bisa jadi simbol PERTAMA dari string turunan α |
| FOLLOW(A) | Himpunan terminal yang bisa MENGIKUTI non-terminal A dalam sentential form |
| Parsing Table M[A,a] | Tabel 2D yang memetakan (non-terminal, terminal lookahead) ke production rule |

## Isi

### Dua Jenis Top-Down Parser
1. **Recursive-Descent Parsing** — BUTUH backtracking (kalau pilihan production rule TIDAK BEKERJA, kita backtrack coba alternatif lain). Teknik parsing UMUM, tapi TIDAK BANYAK dipakai. TIDAK EFISIEN.
2. **Predictive Parsing** — TANPA backtracking, EFISIEN, butuh bentuk grammar KHUSUS (grammar LL(1)). **Recursive Predictive Parsing** adalah bentuk khusus Recursive Descent TANPA backtracking. **Non-Recursive (Table Driven) Predictive Parser** juga dikenal sebagai **LL(1) parser**.

### Recursive-Descent Parsing (dengan Backtracking)
Mencoba menemukan LEFTMOST DERIVATION. Contoh: grammar `S → aBc`, `B → bc | b`, input `abc` — GAGAL di satu pilihan, lalu BACKTRACK mencoba alternatif lain sampai berhasil.

### Predictive Parser
Saat menulis ulang non-terminal dalam satu langkah derivasi, predictive parser bisa MEMILIH SECARA UNIK sebuah production rule HANYA dengan melihat SIMBOL SAAT INI di input string.

Untuk grammar `A → α1 | ... | αn`, dengan token saat ini `a`: grammar harus **DIELIMINASI LEFT RECURSION**-nya dan **DI-LEFT-FACTOR** dulu supaya cocok untuk predictive parsing (grammar LL(1)) — TAPI **TIDAK ADA JAMINAN 100%** transformasi ini akan MENGHASILKAN grammar LL(1) yang valid.

**Contoh:** `stmt → if... | while... | begin... | for...` — saat menulis non-terminal `stmt`, kalau token saat ini `if`, kita HARUS memilih production PERTAMA — parser bisa memutuskan SECARA UNIK cukup dengan melihat token saat ini.

### Recursive Predictive Parsing
Setiap non-terminal berkorespondensi dengan SATU PROSEDUR.

**Contoh sederhana** (`A → aBb`, hanya satu production):
```
proc A {
    match token saat ini dengan a, lalu maju ke token berikutnya;
    call 'B';
    match token saat ini dengan b, lalu maju ke token berikutnya;
}
```

**Contoh dengan pilihan** (`A → aBb | bAB`):
```
proc A {
    case token saat ini {
        'a': match 'a', maju; call 'B'; match 'b', maju;
        'b': match 'b', maju; call 'A'; call 'B';
    }
}
```

**Kapan menerapkan ε-production:** `A → aA | bB | ε` — kalau SEMUA production LAIN gagal, kita terapkan ε-production. Pilihan yang PALING BENAR: terapkan ε-production untuk non-terminal A saat token saat ini ADA di **FOLLOW SET** A.

**Contoh lengkap** (`A → aBe | cBd | C`, `B → bB | ε`, `C → f`):
```
proc A {
    case token saat ini {
        a: match 'a', maju; call B; match 'e', maju;
        c: match 'c', maju; call B; match 'd', maju;
        f: call C;
    }
}
proc B {
    case token saat ini {
        b: match 'b', maju; call B;
        e, d: do nothing;  // ε-production, karena e,d ada di FOLLOW(B)
    }
}
proc C {
    match token saat ini dengan f, maju ke token berikutnya;
}
```

### Non-Recursive Predictive Parsing — LL(1) Parser
Non-recursive predictive parsing adalah parser TABLE-DRIVEN. Ini adalah top-down parser, juga dikenal sebagai **LL(1) Parser**:
- **"L" pertama** — SCANNING input dari KIRI ke KANAN.
- **"L" kedua** — menghasilkan **LEFTMOST DERIVATION**.
- **"1"** — memakai SATU simbol input LOOKAHEAD di setiap langkah.

```
Input buffer → [Stack + Parsing Table] → [Non-Recursive Predictive Parser] → Output
```

**Komponen LL(1) Parser:**
- **Input buffer** — string yang akan di-parse, DIAKHIRI simbol khusus `$`.
- **Output** — production rule yang merepresentasikan SATU LANGKAH derivasi (leftmost derivation) string di input buffer.
- **Stack** — berisi simbol grammar. Di DASAR stack ada simbol penanda akhir `$`. Awalnya stack cuma berisi `$S` (S = start symbol). Saat stack KOSONG (cuma tersisa `$`), parsing SELESAI.
- **Parsing table** — array 2D M[A,a]. Baris = non-terminal, kolom = terminal (atau simbol `$`). Setiap entri berisi production rule.

### Membangun LL(1) Parsing Table: FIRST SET dan FOLLOW SET
**FIRST(α)** — himpunan simbol TERMINAL yang muncul sebagai simbol PERTAMA dalam string yang diturunkan dari α (α adalah string simbol grammar apa pun). Kalau α bisa turunkan ε, maka ε juga ada di FIRST(α).

**FOLLOW(A)** — himpunan terminal yang muncul TEPAT SETELAH (mengikuti) non-terminal A dalam string yang diturunkan dari start symbol. Terminal `a` ada di FOLLOW(A) kalau S ⇒* ...Aa.... `$` ada di FOLLOW(A) kalau S ⇒* ...A.

### Menghitung FIRST Set
Terapkan aturan berikut sampai TIDAK ADA lagi terminal/ε yang bisa ditambahkan:
1. Kalau X terminal, maka FIRST(X) = {X}.
2. Kalau X non-terminal dan X → Y1Y2...Yn adalah production (n≥1): kalau terminal a ada di FIRST(Yi) dan ε ada di SEMUA FIRST(Yj) untuk j=1,...,i-1, maka a ada di FIRST(X). Kalau ε ada di SEMUA FIRST(Yj) untuk j=1,...,n, maka ε ada di FIRST(X).
3. Kalau X → ε adalah production, tambahkan ε ke FIRST(X).

**Contoh:** grammar `E→TE'`, `E'→+TE'|ε`, `T→FT'`, `T'→*FT'|ε`, `F→(E)|id`:
| Non-terminal | FIRST Set |
| --- | --- |
| E | {(, id} |
| E' | {+, ε} |
| T | {(, id} |
| T' | {*, ε} |
| F | {(, id} |

### Menghitung FOLLOW Set
Terapkan aturan berikut sampai TIDAK ADA yang bisa ditambahkan:
1. Taruh `$` di FOLLOW(S), di mana S = start symbol dan `$` = right endmarker.
2. Kalau ada production `A→αBβ`, maka SEMUA yang ada di FIRST(β) KECUALI ε ditaruh di FOLLOW(B).
3. Kalau ada production `A→αB` atau `A→αBβ` di mana FIRST(β) mengandung ε, maka SEMUA yang ada di FOLLOW(A) ditaruh di FOLLOW(B).

**Contoh (grammar sama seperti di atas):**
| Non-terminal | FOLLOW Set |
| --- | --- |
| E | {$, )} |
| E' | {$, )} |
| T | {+, ), $} |
| T' | {+, ), $} |
| F | {+, *, ), $} |

### Membangun Predictive Parsing Table
Untuk setiap production rule `A → α`:
1. Untuk setiap terminal a di FIRST(α): tambahkan `A→α` ke M[A,a].
2. Kalau ε ada di FIRST(α): untuk setiap terminal b di FOLLOW(A), tambahkan `A→α` ke M[A,b].
3. Kalau ε ada di FIRST(α) DAN `$` ada di FOLLOW(A): tambahkan `A→α` ke M[A,$].
4. SEMUA entri LAIN yang tidak terdefinisi adalah entri ERROR.

**Contoh hasil parsing table lengkap:**
| | id | + | * | ( | ) | $ |
| --- | --- | --- | --- | --- | --- | --- |
| E | E→TE' | | | E→TE' | | |
| E' | | E'→+TE' | | | E'→ε | E'→ε |
| T | T→FT' | | | T→FT' | | |
| T' | | T'→ε | T'→*FT' | | T'→ε | T'→ε |
| F | F→id | | | F→(E) | | |

### LL(1) Parser: Aksi Parser
Simbol di puncak stack (X) dan simbol saat ini di input (a) menentukan aksi parser. **Empat kemungkinan aksi:**
1. Kalau X DAN a keduanya `$` → parser BERHENTI (SUKSES selesai).
2. Kalau X DAN a adalah TERMINAL yang SAMA (bukan `$`) → parser POP X dari stack, MAJU ke simbol berikutnya di input buffer.
3. Kalau X adalah NON-TERMINAL → parser lihat entri M[X,a]. Kalau M[X,a] berisi production `X→Y1Y2...Yk`, POP X dari stack, PUSH Yk,...,Y1 ke stack (urutan TERBALIK). Parser juga MENGELUARKAN output production rule ini sebagai satu langkah derivasi.
4. Kalau tidak ada di atas → ERROR (semua entri KOSONG di parsing table adalah error; X terminal yang BEDA dari a juga error).

**Contoh trace lengkap** untuk input `id+id$`:
| Stack | Input | Action |
| --- | --- | --- |
| E$ | id+id$ | E→TE' |
| TE'$ | id+id$ | T→FT' |
| FT'E'$ | id+id$ | F→id |
| idT'E'$ | id+id$ | POP |
| T'E'$ | +id$ | T'→ε |
| E'$ | +id$ | E'→+TE' |
| +TE'$ | +id$ | POP |
| TE'$ | id$ | T→FT' |
| FT'E'$ | id$ | F→id |
| idT'E'$ | id$ | POP |
| T'E'$ | $ | T'→ε |
| E'$ | $ | E'→ε |
| $ | $ | Halt/Accepted |

**Contoh grammar sederhana `S→aBa`, `B→bB|ε`, input `abba`** — trace lengkap menghasilkan derivasi leftmost `S ⇒ aBa ⇒ abBa ⇒ abbBa ⇒ abba`, DITERIMA (accepted).

### Grammar yang BUKAN LL(1)
**Contoh grammar ambigu klasik (dangling else):** `S → iCtSE | a`, `E → eS | ε`, `C → b`. Grammar ini menghasilkan **DUA entri berbeda di M[E,e]** — MASALAH AMBIGUITAS (parser TIDAK BISA memutuskan secara unik saat token saat ini `e`).

**Apa yang harus dilakukan kalau parsing table punya entri GANDA?**
1. Kalau BELUM eliminasi left recursion, eliminasi DULU.
2. Kalau grammar belum di-left-factor, LAKUKAN dulu.
3. Kalau, SETELAH ditransformasi, parsing table MASIH punya entri ganda, grammar itu **AMBIGU** atau **secara INHEREN bukan grammar LL(1)**.

**Fakta penting:**
- **Grammar left-recursive TIDAK BISA jadi grammar LL(1).**
- **Grammar yang belum di-left-factor TIDAK BISA jadi grammar LL(1).**
- **Grammar ambigu TIDAK BISA jadi grammar LL(1).**

### Properti Grammar LL(1)
Grammar G adalah LL(1) JIKA DAN HANYA JIKA, untuk dua production BERBEDA `A → α` dan `A → β`, KETIGA kondisi ini berlaku:
1. α dan β TIDAK BOLEH SAMA-SAMA menurunkan string yang dimulai TERMINAL yang SAMA.
2. PALING BANYAK SATU dari α atau β boleh menurunkan ε.
3. Kalau β bisa menurunkan ε, maka α TIDAK BOLEH menurunkan string apa pun yang dimulai terminal di FOLLOW(A).

> [!info] Analogi
> Aturan LL(1) itu seperti aturan memilih JALAN di persimpangan hanya dengan melihat RAMBU LALU LINTAS satu kali (satu lookahead), tanpa boleh berhenti untuk mengecek dua kali. Kalau DUA jalan berbeda punya rambu AWAL yang SAMA PERSIS (FIRST overlap), kamu TIDAK BISA memutuskan mana yang harus diambil hanya dengan sekali lihat — kamu HARUS "mundur" (backtrack) atau tersesat. Aturan LL(1) memastikan SETIAP persimpangan punya rambu awal yang UNIK, sehingga keputusan SELALU bisa diambil dengan sekali lihat saja.

### Error Recovery pada Predictive Parsing
Error bisa terjadi di predictive parsing (LL(1) parsing) kalau: simbol terminal di PUNCAK stack TIDAK COCOK dengan simbol input SAAT INI, ATAU puncak stack adalah non-terminal A, simbol input saat ini adalah a, dan entri parsing table M[A,a] KOSONG.

**Apa yang harus dilakukan parser saat error?** Parser harus bisa memberi PESAN ERROR (sebermakna mungkin), harus bisa PULIH dari kondisi error itu, dan harus bisa LANJUT parsing dengan sisa input.

**Teknik Error Recovery:**
1. **Panic-Mode Error Recovery** — SKIP simbol input sampai bertemu SYNCHRONIZING TOKEN. **SEMUA terminal di FOLLOW SET sebuah non-terminal bisa dipakai sebagai synchronizing token set** untuk non-terminal itu. Implementasi: semua entri KOSONG ditandai "synch" — parser skip input sampai bertemu simbol di FOLLOW(A) untuk non-terminal A di puncak stack, lalu POP A dari stack dan lanjut. Untuk simbol terminal yang TIDAK COCOK, parser POP terminal itu dari stack dan LAPORKAN error "terminal tersebut disisipkan (inserted)".
2. **Phrase-Level Error Recovery** — setiap entri KOSONG diisi POINTER ke rutin error KHUSUS. Rutin ini bisa: MENGUBAH/MENYISIPKAN/MENGHAPUS simbol input, mengeluarkan pesan error, POP item dari stack. HARUS HATI-HATI mendesain rutin ini karena bisa membuat parser masuk INFINITE LOOP.
3. **Error-Productions** — kalau kita punya gambaran BAIK soal error umum yang mungkin terjadi, grammar bisa DIPERLUAS dengan production yang menghasilkan konstruksi yang ERROR. Saat production error ini dipakai parser, kita bisa hasilkan diagnostik error yang SESUAI. Karena HAMPIR MUSTAHIL tahu SEMUA error yang mungkin dibuat programmer, metode ini TIDAK PRAKTIS.
4. **Global-Correction** — idealnya, compiler ingin membuat PERUBAHAN SESEDIKIT MUNGKIN dalam memproses input yang salah — harus MENGANALISIS SECARA GLOBAL input untuk menemukan error. Metode ini MAHAL dan TIDAK PRAKTIS dipakai.

**Contoh panic-mode error recovery** untuk grammar `S→AbS|e|ε`, `A→a|cAd`, dengan FOLLOW(S)={$}, FOLLOW(A)={b,d} — trace menunjukkan kasus "missing b, inserted" dan "unexpected e (illegal A)" dengan proses pemulihan skip token sampai synchronizing token ditemukan.

## Diagram & Visual
> [!warning] Deck ini sebagian besar berisi TABEL dan pseudo-code prosedur (bukan diagram gambar raster) — SEMUA sudah direkonstruksi ke format tabel/kode di atas. Tidak ada gambar (PICTURE shape) yang terekstrak dari deck ini sama sekali — SEMUA visual berupa text-box/shape manual PowerPoint.

## Rumus / Sintaks
```
FIRST(X):
1. X terminal          -> FIRST(X) = {X}
2. X -> Y1Y2...Yn       -> a di FIRST(Yi), ε di FIRST(Y1..Yi-1) => a di FIRST(X)
                        -> ε di semua FIRST(Yj) => ε di FIRST(X)
3. X -> ε               -> ε di FIRST(X)

FOLLOW(A):
1. $ di FOLLOW(S)  (S = start symbol)
2. A -> alpha B beta    -> FIRST(beta)-{ε} masuk FOLLOW(B)
3. A -> alpha B  atau  A -> alpha B beta dengan ε di FIRST(beta)
                        -> FOLLOW(A) masuk FOLLOW(B)

Parsing Table M[A,a]:
untuk production A -> alpha:
  a di FIRST(alpha)                    -> M[A,a] = A->alpha
  ε di FIRST(alpha), b di FOLLOW(A)    -> M[A,b] = A->alpha
  ε di FIRST(alpha), $ di FOLLOW(A)    -> M[A,$] = A->alpha

LL(1) Parser Actions:
X=a=$           -> Halt (accept)
X=a (terminal)  -> pop X, maju input
X non-terminal  -> lookup M[X,a], pop X, push RHS terbalik
lainnya         -> Error
```

## Istilah Khusus
| Istilah | Penjelasan |
| --- | --- |
| **Backtracking** | Mundur dan mencoba alternatif lain saat pilihan production gagal |
| **Endmarker ($)** | Simbol khusus penanda akhir input dan dasar stack di LL(1) parser |
| **Dangling else problem** | Ambiguitas klasik saat grammar if-else tidak jelas pasangan else-nya |
| **Synchronizing token** | Token dari FOLLOW SET yang dipakai panic-mode recovery untuk melanjutkan parsing |

## Quiz Pemahaman
Level Bloom C4 ke atas.

1. **(C4 – Analisis)** Analisis kenapa "Recursive-Descent Parsing dengan backtracking" dianggap TIDAK EFISIEN dibanding Predictive Parsing. Hubungkan dengan kompleksitas waktu — apa yang terjadi pada JUMLAH percobaan parsing kalau grammar punya BANYAK alternatif production yang harus dicoba satu-satu sebelum menemukan yang BERHASIL?
2. **(C4 – Analisis)** Bandingkan FIRST SET dan FOLLOW SET dari sisi FUNGSI masing-masing dalam membangun parsing table. Analisis: kenapa FOLLOW SET HANYA dibutuhkan untuk non-terminal yang PRODUCTION-nya bisa turunkan ε — apa yang terjadi kalau SEMUA production sebuah non-terminal TIDAK PERNAH turunkan ε (tidak butuh FOLLOW sama sekali)?
3. **(C5 – Evaluasi)** Evaluasi grammar dangling-else (`S→iCtSE|a`, `E→eS|ε`) yang BUKAN LL(1) karena M[E,e] punya DUA entri. Jelaskan AKAR MASALAH ambiguitas ini — kenapa parser TIDAK BISA memutuskan "apakah else ini pasangan if TERDEKAT atau harus mengembalikan E→ε dan biarkan else itu jadi bagian struktur LUAR" hanya dengan SATU token lookahead?
4. **(C5 – Evaluasi)** Bandingkan Panic-Mode Error Recovery (praktis, dipakai luas) dengan Global-Correction (teoretis optimal, tidak praktis). Evaluasi: berdasarkan definisi FOLLOW SET sebagai "synchronizing token", jelaskan KENAPA memakai FOLLOW SET itu MASUK AKAL sebagai titik "aman" untuk melanjutkan parsing setelah error — apa MAKNA dari "token itu BISA MENGIKUTI non-terminal ini" dalam konteks pemulihan error?
5. **(C6 – Cipta)** Kerjakan latihan dari slide 41: untuk grammar `S→ABC`, `A→aA|C`, `B→b`, `C→c` — (a) hitung FIRST dan FOLLOW untuk SEMUA non-terminal (S, A, B, C), (b) bangun predictive parsing table lengkap, (c) trace eksekusi parsing untuk input `aabc$` langkah demi langkah (stack, input, action) sampai diterima atau ditolak.

## Versi Gue
<!-- JANGAN DISENTUH. Section ini diisi manual oleh pemilik vault. -->

## Terkait
- [[_Compiler]]
- [[W10 - Pushdown Automata]]
- [[W14 - Review I]]
- [[W15 - Bottom-Up Parsing]]
- [[Compiler - Review dan Glosari]]
