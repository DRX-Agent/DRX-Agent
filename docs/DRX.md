# NerveAI — Product Requirements Document

Versi 0.1 · 10 November 2026 · Status: Draft untuk review · Fase 1 (tanpa agent spesialis)

## 1. Ringkasan

NerveAI adalah AI coding harness berbasis terminal (TUI) yang berjalan penuh dengan API gratis dan model lokal. Pengalaman intinya mirip OpenCode: pengguna mengobrol dengan AI di terminal, dan AI dapat membaca, mengedit file, serta menjalankan perintah di proyek mereka. Pembedanya adalah tampilan yang lebih rapi dan onboarding yang jauh lebih mudah, terutama untuk pengguna yang bergantung pada tier gratis dengan batas pemakaian ketat.

Fase 1 hanya memiliki satu agent umum. Agent spesialis (coding, design, marketing, planner/PRD, dan lainnya) baru dibangun di Fase 2, setelah harness stabil. Karena itu Fase 1 wajib menyiapkan titik ekstensi (extension point) agar agent bisa ditambahkan tanpa merombak arsitektur.

## 2. Latar belakang dan masalah

Harness open source seperti OpenCode sudah kuat, tetapi bagi pengguna yang ingin biaya nol ada beberapa hambatan:

- **Setup provider manual.** Pengguna harus mencari sendiri provider gratis, membuat API key, lalu menulis konfigurasi JSON.
- **Kuota gratis habis tanpa peringatan.** Satu tugas coding bisa memakai belasan sampai puluhan request, sementara beberapa tier gratis hanya mengizinkan puluhan request per hari. Pengguna baru tahu ketika error muncul.
- **Error rate limit membingungkan.** Pesan 429 mentah tidak memberi tahu apa yang harus dilakukan.
- **Model gratis tidak seragam.** Kualitas tool calling berbeda-beda, ID model gratis bisa kedaluwarsa, dan batas konteks ada yang kecil.
- **Risiko privasi tidak terlihat.** Banyak tier gratis dapat memakai prompt untuk meningkatkan produk penyedia, dan pengguna jarang menyadarinya.
- **Tampilan fungsional tetapi belum ramah pemula.**

## 3. Tujuan dan bukan-tujuan

### Tujuan (Fase 1)

| ID | Tujuan | Ukuran keberhasilan |
| --- | --- | --- |
| G1 | Pengguna baru bisa mendapat jawaban pertama dari AI dalam waktu singkat tanpa mengedit file konfigurasi | ≤ 3 menit dari instalasi ke respons pertama |
| G2 | Biaya operasional pengguna tetap $0 | Semua jalur default memakai tier gratis atau model lokal |
| G3 | Pengguna jarang terhenti oleh rate limit | Fallback otomatis antar provider berhasil pada sebagian besar kasus 429 |
| G4 | Tampilan lebih baik dan lebih mudah dipakai dibanding OpenCode bawaan | Survei beta: mayoritas penguji menilai lebih jelas dan nyaman |
| G5 | Arsitektur siap untuk agent spesialis di Fase 2 | Ada agent registry dan interface agent yang terdokumentasi |

### Bukan-tujuan (Fase 1)

- Agent spesialis (coding, design, marketing, planner, reviewer, dll.).
- Aplikasi desktop atau web penuh. Fase 1 fokus pada TUI.
- Layanan cloud, akun pengguna, atau sinkronisasi antar perangkat.
- Membeli atau menjual kuota API, dan memutar banyak akun gratis untuk menghindari batas. Hal ini umumnya melanggar ketentuan penyedia dan tidak akan dilakukan.
- Menyaingi model berbayar kelas atas dari sisi kualitas.

## 4. Pengguna target

| Persona | Deskripsi | Kebutuhan utama |
| --- | --- | --- |
| Pelajar / pemula | Belajar coding, tanpa anggaran untuk API | Setup mudah, penjelasan error yang ramah, biaya nol |
| Developer hemat | Sudah mahir, ingin alternatif gratis untuk proyek pribadi | Kontrol penuh, fallback pintar, indikator kuota |
| Maker / indie hacker | Membangun prototipe cepat | Kecepatan, tampilan jelas, hasil yang bisa langsung dipakai |

## 5. Keputusan pendekatan: fork atau bangun dari nol

| Opsi | Kelebihan | Kekurangan |
| --- | --- | --- |
| **A. Fork OpenCode (direkomendasikan)** | Fitur inti sudah ada (sesi, tool, izin, MCP, dukungan banyak provider). Waktu ke prototipe jauh lebih cepat. | Harus mengikuti arsitektur dan stack bawaan. Risiko perbedaan dengan upstream seiring waktu. |
| B. Bangun dari nol | Kontrol penuh atas desain dan stack | Memakan waktu berbulan-bulan hanya untuk menyamai fitur dasar |
| C. Plugin / konfigurasi di atas OpenCode tanpa fork | Paling ringan, mudah ikut pembaruan upstream | Perubahan UI mendalam dan onboarding sulit dilakukan |

**Rekomendasi:** mulai dengan Opsi A, tetapi disiplin menjaga perubahan tetap terisolasi.

Syarat pelaksanaan fork:

1. **Verifikasi lisensi** repositori OpenCode saat ini sebelum fork, dan penuhi kewajiban atribusinya (pemberitahuan lisensi dipertahankan).
2. **Tempatkan kode NerveAI di modul/paket terpisah** (provider router, quota tracker, onboarding, tema) dan minimalkan perubahan pada inti. Ini memudahkan merge dari upstream.
3. **Pelajari struktur repo terbaru** (TUI, server/core, lapisan provider) pada Milestone 0 sebelum memutuskan titik integrasi. Struktur dan stack upstream bisa berubah, jadi jangan berasumsi dari versi lama.
4. **Rebrand** nama, logo, perintah CLI (`nerve`), dan direktori konfigurasi.
5. Jika pada Milestone 0 ditemukan bahwa lapisan UI terlalu terikat, evaluasi Opsi C sebagai jalan tengah sebelum berkomitmen penuh pada fork.

## 6. Kebutuhan fungsional

Prioritas: **P0** wajib untuk rilis beta, **P1** penting, **P2** bila waktu ada.

### 6.1 Fitur inti (diwarisi dari OpenCode, harus tetap berfungsi)

| ID | Kebutuhan | Prioritas |
| --- | --- | --- |
| C1 | Sesi percakapan dengan riwayat tersimpan dan dapat dilanjutkan | P0 |
| C2 | Tool file: baca, tulis, edit, cari (glob/grep) | P0 |
| C3 | Tool eksekusi perintah shell dengan konfirmasi izin | P0 |
| C4 | Sistem izin per tool (tanya / izinkan / tolak) | P0 |
| C5 | File konteks proyek (misalnya AGENTS.md) dimuat otomatis | P0 |
| C6 | Tampilan diff dan kemampuan membatalkan perubahan | P1 |
| C7 | Dukungan MCP server | P1 |
| C8 | Slash command bawaan dan kustom | P1 |

### 6.2 Fitur khas NerveAI

| ID | Kebutuhan | Prioritas |
| --- | --- | --- |
| N1 | **Onboarding wizard** saat run pertama: pilih provider gratis, buka tautan pembuatan API key, tempel key, tes koneksi, pilih model default. Key disimpan dengan izin file ketat (atau keychain OS bila tersedia), tidak pernah ditampilkan penuh di layar atau log. | P0 |
| N2 | **Provider presets** untuk Google Gemini (AI Studio), Groq, OpenRouter (model :free), Mistral, Cerebras, Cloudflare Workers AI, GitHub Models, dan Ollama (lokal). Karena sebagian besar mendukung format OpenAI Chat Completions, gunakan satu adapter generik dengan base\_url dan model per preset. | P0 |
| N3 | **Fallback chain otomatis.** Pengguna mengurutkan provider. Saat menerima 429, kuota habis, atau timeout, NerveAI berpindah ke provider berikutnya, menampilkan notifikasi singkat, lalu menerapkan cooldown pada provider yang gagal. | P0 |
| N4 | **Quota meter.** Hitung request dan token per provider per hari secara lokal, tampilkan di status bar, beri peringatan saat mendekati batas yang dikonfigurasi. Karena batas resmi bisa berubah, nilai batas disimpan di registry yang dapat diperbarui (N5), dan pengguna dapat menimpanya. | P0 |
| N5 | **Model registry yang dapat diperbarui.** Daftar model gratis, batas, panjang konteks, dan dukungan tool calling disimpan dalam berkas JSON terpisah yang bisa diperbarui tanpa rilis baru. Sediakan perintah refresh dan penanda model yang sudah tidak tersedia. | P1 |
| N6 | **Lapisan kompatibilitas tool calling.** Validasi argumen tool dari model, coba ulang otomatis dengan pesan perbaikan bila JSON rusak, dan sediakan mode prompted tools untuk model yang tidak mendukung function calling secara native. | P0 |
| N7 | **Manajemen konteks.** Pantau pemakaian konteks, tampilkan di UI, lakukan kompaksi/ringkasan otomatis sebelum mendekati batas model (penting untuk model berkonteks kecil). | P1 |
| N8 | **Privacy guard.** Peringatan sekali di awal bahwa tier gratis tertentu dapat memakai prompt untuk perbaikan produk. Blokir atau minta konfirmasi sebelum mengirim file sensitif (.env, kunci privat, kredensial). Sediakan mode Local only yang hanya memakai Ollama. | P1 |
| N9 | **Pesan error ramah.** Ubah error mentah (401, 429, timeout, model tidak ditemukan) menjadi penjelasan dan saran tindakan konkret. | P0 |
| N10 | **Konfigurasi sederhana.** Berkas global dan per-proyek (misalnya nerve.json) dengan nilai bawaan masuk akal, serta pengimpor konfigurasi dari OpenCode. | P1 |
| N11 | **Titik ekstensi agent** (lihat Bagian 10): agent registry dan interface agent, dengan satu agent bawaan bernama General. | P0 |
| N12 | **Mode hemat kuota.** Opsi untuk menggunakan model kecil/cepat untuk tugas ringan (judul sesi, ringkasan) dan menyimpan kuota model utama untuk tugas berat. | P2 |

## 7. Desain pengalaman pengguna

### Prinsip

1. **Jelas sekilas.** Pengguna selalu tahu: model apa yang aktif, sisa kuota, dan apa yang sedang dilakukan AI.
2. **Tidak ada jalan buntu.** Setiap error disertai langkah berikutnya.
3. **Cepat dipelajari.** Command palette dan petunjuk keybinding terlihat, tidak perlu menghafal.
4. **Tetap terminal-native.** Cepat, ringan, bisa dipakai lewat SSH.

### Peningkatan dibanding OpenCode bawaan

- Tema siap pakai (gelap, terang, kontras tinggi) dengan satu palet aksen khas NerveAI.
- Status bar tetap berisi: provider dan model aktif, sisa kuota harian, pemakaian konteks, mode izin.
- Command palette (misalnya Ctrl+P) untuk ganti model, provider, tema, dan sesi.
- Tampilan tool call yang dapat dilipat (collapsed) agar percakapan tidak penuh log.
- Diff berwarna dengan penanda baris yang rapi.
- Layar bantuan kontekstual dan tur singkat pada penggunaan pertama.

### Sketsa tata letak (konseptual)

```
┌ NerveAI ─────────────────────────── gemini-flash ● groq ○ ┐
│                                                           │
│  kamu   Tambahkan validasi email di form signup           │
│                                                           │
│  nerve  Saya baca src/signup.ts dulu...                   │
│         ▸ read  src/signup.ts            (lihat)          │
│         ▸ edit  src/signup.ts   +12 −3   (lihat diff)     │
│                                                           │
├───────────────────────────────────────────────────────────┤
│ > ketik pesan, atau tekan Ctrl+P untuk perintah           │
├───────────────────────────────────────────────────────────┤
│ kuota gemini 41% · groq 12% │ konteks 38% │ izin: tanya   │
└───────────────────────────────────────────────────────────┘
```

### Alur onboarding

1. Jalankan `nerve` pertama kali.
2. Pilih jalur: Cepat (satu provider gratis), Lengkap (beberapa provider + fallback), atau Lokal (Ollama).
3. Wizard menampilkan tautan pembuatan key, pengguna menempel key, koneksi diuji.
4. Pilih model default, lalu tampil pesan sambutan dan contoh perintah pertama.

## 8. Arsitektur tingkat tinggi

```
 TUI (NerveAI)  ──►  Core / Server  ──►  Tool executor (file, shell, MCP)
                          │
                          ▼
                    Agent runtime  ◄── Agent registry (Fase 2)
                          │
                          ▼
   Provider layer:  Router + Fallback  ─►  Quota tracker
                          │                  │
                          ▼                  ▼
              Adapter OpenAI-compatible   Model registry (JSON)
              (Gemini, Groq, OpenRouter, Mistral, Cerebras,
               Cloudflare, GitHub Models, Ollama)

 Penyimpanan lokal: sesi, konfigurasi, penghitung kuota
```

Komponen baru yang ditambahkan NerveAI pada fork: **Router + Fallback**, **Quota tracker**, **Model registry**, **Tool-call compatibility layer**, **Privacy guard**, **Onboarding wizard**, dan **tema/UI**. Detail implementasi mengikuti stack upstream hasil riset Milestone 0.

## 9. Kebutuhan non-fungsional

- **Biaya:** jalur default $0. Tidak ada layanan berbayar yang dibutuhkan.
- **Platform:** Linux, macOS, Windows (disarankan via WSL pada awalnya).
- **Performa:** TUI responsif. Router menambah overhead yang tidak terasa dibanding latensi jaringan ke LLM.
- **Koneksi:** toleran terhadap jaringan lambat atau putus, dengan retry berbatas dan timeout jelas. Mode Local only berfungsi offline.
- **Keamanan:** API key tidak masuk log atau riwayat sesi. Perintah shell butuh izin eksplisit secara default. Telemetri nonaktif secara default (opt-in bila ada).
- **Keandalan:** kegagalan satu provider tidak boleh merusak sesi. Sesi tetap tersimpan dan dapat dilanjutkan.
- **Lisensi:** atribusi upstream dipertahankan, dan lisensi NerveAI ditentukan sebelum rilis publik.

## 10. Persiapan Fase 2: agent spesialis

Fase 1 hanya mengirim satu agent bawaan (General), tetapi harus mendefinisikan kontrak agent:

```
Agent {
  id, nama, deskripsi
  systemPrompt
  tools yang diizinkan
  izin (file, shell, jaringan)
  model / fallback chain khusus agent
  mode: utama (dipilih pengguna) atau subagent (dipanggil agent lain)
}
```

Calon agent Fase 2: Planner (PRD generator), Coding, Architect, UI/UX Designer, Marketing/Copywriter, Reviewer/QA, Debugger, Researcher, Docs writer, dan Orchestrator untuk mendelegasikan tugas. Pemilihan model per agent memungkinkan pembagian kuota gratis yang cerdas (misalnya model berkonteks besar untuk Planner dan Researcher, model cepat untuk Coding).

## 11. Metrik keberhasilan

| Metrik | Target beta |
| --- | --- |
| Waktu dari instalasi ke respons pertama | ≤ 3 menit |
| Sesi yang selesai tanpa terhenti oleh error kuota | Mayoritas sesi (target awal 80%) |
| Keberhasilan fallback saat provider utama kena 429 | ≥ 90% |
| Tool call yang berhasil setelah validasi/retry | Dibandingkan baseline tanpa lapisan kompatibilitas, harus meningkat |
| Kepuasan penguji beta atas tampilan dan kemudahan | Mayoritas menilai lebih baik dari OpenCode bawaan |

## 12. Roadmap

Estimasi untuk satu developer paruh waktu. Sesuaikan dengan waktu yang tersedia.

| Milestone | Isi | Perkiraan |
| --- | --- | --- |
| M0 — Riset dan fork | Verifikasi lisensi, pelajari struktur repo, build lokal, tentukan titik integrasi, putuskan A atau C | Minggu 1 |
| M1 — Rebrand dan presets | Nama, logo, perintah `nerve`, adapter generik, preset provider, konfigurasi dasar | Minggu 2 |
| M2 — Router dan kuota | Fallback chain, quota tracker, status bar, pesan error ramah | Minggu 3–4 |
| M3 — Onboarding dan UX | Wizard, tema, command palette, tampilan tool call dan diff | Minggu 5–6 |
| M4 — Keandalan model gratis | Lapisan kompatibilitas tool calling, manajemen konteks, privacy guard, model registry | Minggu 7–8 |
| M5 — Beta | Dokumentasi, paket distribusi, uji dengan 5–10 pengguna, perbaikan | Minggu 9–10 |
| Fase 2 | Agent registry penuh dan agent spesialis | Setelah beta stabil |

## 13. Risiko dan mitigasi

| Risiko | Dampak | Mitigasi |
| --- | --- | --- |
| Tier gratis berubah atau dihapus tanpa pemberitahuan | Fitur inti terganggu | Registry yang dapat diperbarui, banyak provider, dukungan lokal (Ollama) |
| Model gratis lemah dalam tool calling | Pengalaman buruk | Lapisan kompatibilitas, rekomendasi model teruji, mode prompted tools |
| Fork menyimpang dari upstream | Beban merge tinggi | Isolasi kode di modul sendiri, merge berkala, evaluasi Opsi C |
| Masalah lisensi atau atribusi | Hambatan hukum | Verifikasi lisensi di M0, pertahankan notice |
| Pelanggaran ketentuan penyedia | Akun diblokir | Tidak menyediakan rotasi multi-akun untuk menghindari batas |
| Kebocoran data lewat tier gratis | Privasi pengguna | Privacy guard, peringatan jelas, mode Local only |
| Eksekusi perintah shell berbahaya oleh AI | Kerusakan sistem | Izin eksplisit default, tampilan perintah sebelum dijalankan |
| Perangkat pengguna lemah untuk model lokal | Mode lokal tidak praktis | Jadikan opsional, sarankan model kecil, jangan jadi jalur default |

## 14. Pertanyaan terbuka

1. Bahasa antarmuka: Inggris saja, Indonesia saja, atau keduanya?
2. Lisensi NerveAI sendiri (selaras dengan lisensi upstream)?
3. Distribusi: npm, binary tunggal, atau keduanya?
4. Provider mana yang menjadi default di jalur Cepat? (Putuskan setelah uji kualitas tool calling di M4, tetapi butuh hipotesis awal di M1.)
5. Apakah telemetri anonim akan ada sama sekali, atau tidak sama sekali?
6. Sejauh mana dukungan Windows native diperlukan pada rilis beta?

## 15. Catatan tentang API gratis

Batas dan syarat tier gratis berubah sering. Contohnya, beberapa penyedia memindahkan model unggulan ke berbayar pada 2026, dan sebagian menyaratkan kartu pembayaran untuk kredit gratis. Karena itu semua angka batas di registry harus diperlakukan sebagai nilai awal yang diverifikasi ulang sebelum rilis, bukan sebagai jaminan. Tier gratis cocok untuk penggunaan pribadi dan prototipe, bukan untuk layanan yang bergantung padanya secara produksi.

## 16. Organisasi tim dan pembagian kerja (2 orang)

Bagian ini melengkapi bagian 1–15. Roadmap di bagian 12 tetap berlaku sebagai urutan milestone, sedangkan bagian ini menjelaskan siapa mengerjakan apa. Pengembangan dilakukan oleh dua orang dengan pendekatan vibe coding (AI menulis sebagian besar kode, manusia mengarahkan dan mereview).

### 16.1 Peran

Pembagian dilakukan **berdasarkan lapisan**, bukan fitur acak, supaya kedua orang jarang mengedit file yang sama.

|  | **Orang A — Core dan Provider** | **Orang B — UX dan Produk** |
| --- | --- | --- |
| Fokus | Mesin di belakang layar | Semua yang dilihat dan disentuh pengguna |
| Fitur PRD | N2 preset provider, N3 fallback chain, N4 quota tracker, N5 model registry, N6 kompatibilitas tool calling, N7 manajemen konteks, N8 privacy guard, N11 agent registry | N1 onboarding wizard, N9 pesan error ramah, N10 konfigurasi, N12 mode hemat kuota (tampilan), tema, command palette, status bar, tampilan tool call dan diff, rebrand |
| Pengujian | Test provider palsu (mock 429, timeout, JSON rusak), test kuota | Test alur onboarding, snapshot tampilan, uji pengguna beta |
| Tanggung jawab tambahan | Keamanan penyimpanan API key | Dokumentasi, distribusi (npm / binary), situs atau README |

Peran bisa ditukar sesuai keahlian masing-masing. Yang penting, satu orang memegang satu lapisan sepanjang Fase 1.

### 16.2 Kepemilikan kode

Nama folder final ditetapkan setelah Milestone 0 (menyesuaikan struktur repo upstream). Prinsipnya:

| Area (usulan) | Pemilik | Aturan |
| --- | --- | --- |
| Modul provider: adapter, router, quota, registry, tool-call compat, privacy guard | Orang A | Orang B tidak mengubah langsung, cukup mengajukan permintaan atau PR |
| Modul UI: tema, komponen TUI, onboarding, palette, status bar | Orang B | Orang A tidak mengubah langsung |
| Kontrak antarmuka (lihat 16.3) | Bersama | Perubahan butuh persetujuan keduanya |
| Inti upstream OpenCode | Bersama | Diubah seminimal mungkin, setiap perubahan dicatat dan dijelaskan di PR |
| Konfigurasi, CI, dokumen proyek | Orang B (CI dibantu A) |  |

### 16.3 Kontrak antarmuka (disepakati di M0)

Ini langkah terpenting agar kerja bisa paralel. Orang B dapat membangun status bar, pesan error, dan onboarding memakai data palsu (mock) tanpa menunggu Orang A selesai. Bentuk di bawah adalah pseudo-code, sesuaikan dengan bahasa dan stack upstream.

```
ProviderConfig   { id, nama, baseUrl, apiKeyRef, modelDefault, urutanFallback }
ModelInfo        { id, provider, konteksMaks, dukungToolCalling, gratis, statusTersedia }
QuotaSnapshot    { provider, requestHariIni, tokenHariIni, batasRequest, batasToken, persen }
ProviderStatus   { aktif: ProviderConfig, cooldown: Provider[], kuota: QuotaSnapshot[] }
FallbackEvent    { dari, ke, alasan (429 | kuota | timeout | error), waktu }
NerveError       { kode, pesanRamah, saranTindakan, detailTeknis }

Fungsi yang disediakan Orang A:
  testConnection(providerConfig) -> { ok, galat? }
  listModels(provider)           -> ModelInfo[]
  getProviderStatus()            -> ProviderStatus
  subscribe(FallbackEvent | QuotaSnapshot)  // untuk status bar dan notifikasi
```

Aturan perubahan kontrak: ubah lewat PR kecil yang diberi label contract, dan kabari pasangan sebelum merge.

### 16.4 Roadmap per orang

Urutan dan perkiraan waktu mengikuti bagian 12.

| Milestone | Orang A | Orang B | Bersama |
| --- | --- | --- | --- |
| M0 — Riset dan fork | Pelajari lapisan provider upstream | Pelajari TUI upstream dan sistem tema | Verifikasi lisensi, build lokal, tentukan titik integrasi, tulis kontrak (16.3), siapkan AGENTS.md dan CI |
| M1 — Rebrand dan presets | Adapter generik OpenAI-compatible, preset 8 provider, penyimpanan API key aman | Rebrand (nama, logo, perintah nerve), tema dasar, struktur konfigurasi | Review silang |
| M2 — Router dan kuota | Fallback chain, cooldown, quota tracker, test dengan provider palsu | Status bar, notifikasi fallback, pesan error ramah (pakai mock dulu) | Uji integrasi saat data nyata tersedia |
| M3 — Onboarding dan UX | Model registry dan perintah refresh, testConnection yang andal | Onboarding wizard, command palette, tampilan tool call lipat, diff | Latihan onboarding dari nol di mesin bersih |
| M4 — Keandalan model gratis | Kompatibilitas tool calling, manajemen konteks, privacy guard | Indikator konteks, peringatan privasi, layar bantuan, mode hemat kuota (tampilan) | Review keamanan (lihat 16.6) |
| M5 — Beta | Perbaikan bug provider, pemantauan kuota saat uji | Dokumentasi, paket distribusi, kumpulkan umpan balik 5–10 penguji | Uji end-to-end, triase bug, keputusan rilis |

Ketergantungan utama: onboarding (B, M3) membutuhkan testConnection (A, M1–M3). Karena itu fungsi ini dibuat lebih awal dalam versi sederhana, dan disempurnakan kemudian.

### 16.5 Alur kerja

- **Repositori:** satu repo hasil fork. Branch main dilindungi, tidak ada push langsung.
- **Branch:** satu fitur satu branch, dinamai dengan ID PRD, misalnya feat/N3-fallback-chain.
- **Pull request:** kecil dan fokus pada satu ID. Hasil AI yang besar sulit direview, jadi pecah menjadi beberapa PR.
- **Review silang:** setiap PR direview oleh orang lain. Kode boleh ditulis AI, tetapi harus ada manusia kedua yang membaca diff-nya.
- **CI:** typecheck, lint, dan test berjalan otomatis pada setiap PR. PR yang gagal CI tidak di-merge.
- **Merge dari upstream:** satu orang saja yang melakukannya, sebulan sekali atau ketika ada pembaruan penting, supaya konflik tidak berlipat. Usulan: Orang A, karena perubahan upstream paling sering menyentuh lapisan provider.
- **Pelacakan tugas:** papan sederhana (GitHub Projects atau Trello) dengan kolom Backlog, Dikerjakan, Review, Selesai. Setiap kartu merujuk ke satu ID PRD.
- **Catatan keputusan:** keputusan penting ditulis singkat di berkas docs/keputusan.md (tanggal, keputusan, alasan).

### 16.6 Aturan vibe coding

1. **Tulis AGENTS.md di repo:** struktur proyek, folder yang dimiliki masing-masing, aturan jangan ubah inti upstream sembarangan, cara menjalankan test, dan gaya kode. Dengan begitu AI di perangkat kedua orang bekerja dengan konteks yang sama.
2. **Satu perintah, satu ID, dengan kriteria penerimaan jelas.** Contoh: kerjakan N3 dengan kriteria di 16.7, jangan perintah samar seperti buat fallback yang bagus.
3. **Test lebih dulu atau bersamaan.** Pada vibe coding, test adalah pengaman utama. Router fallback sangat cocok diuji dengan provider palsu yang sengaja mengembalikan 429.
4. **Jangan menerima perubahan di luar area sendiri** tanpa sepengetahuan pemiliknya.
5. **Area berisiko tinggi wajib direview manual oleh kedua orang** (bukan hanya dipercayakan pada AI):

| Area | Risiko |
| --- | --- |
| Penyimpanan dan penampilan API key | Kebocoran ke log, layar, atau riwayat sesi |
| Izin eksekusi shell | AI menjalankan perintah berbahaya |
| Privacy guard | File sensitif terkirim ke penyedia gratis |
| Logika kuota dan fallback | Salah hitung membuat pengguna kehabisan kuota atau terkena blokir |

### 16.7 Kriteria penerimaan contoh

**N3 — Fallback chain**

- Saat provider utama mengembalikan 429, permintaan otomatis dicoba ke provider berikutnya tanpa pengguna mengetik ulang.
- Notifikasi singkat menampilkan perpindahan dari provider mana ke mana, dan alasannya.
- Provider yang gagal masuk cooldown dan tidak dicoba lagi sampai cooldown selesai.
- Jika semua provider gagal, pengguna menerima NerveError dengan saran tindakan, dan sesi tidak rusak.
- Ada test otomatis dengan provider palsu untuk kasus 429, timeout, dan semua gagal.

**N4 — Quota meter**

- Penghitung request dan token per provider tersimpan lokal dan bertahan setelah aplikasi ditutup.
- Status bar memperbarui persentase setelah setiap permintaan.
- Peringatan muncul saat pemakaian mencapai ambang yang dikonfigurasi (misalnya 80%).
- Penghitung direset sesuai jendela waktu provider, dan pengguna dapat menimpa batas secara manual.

**N1 — Onboarding wizard**

- Pengguna baru mencapai respons AI pertama dalam 3 menit tanpa mengedit file konfigurasi.
- Wizard menguji koneksi sebelum menyimpan key, dan menampilkan penjelasan jika gagal.
- API key tidak tampil penuh di layar maupun log.
- Wizard dapat dijalankan ulang dari command palette.

### 16.8 Definition of Done (per fitur)

- Kriteria penerimaan terpenuhi dan dites, termasuk test otomatis bila relevan.
- CI hijau dan PR sudah direview orang lain.
- Dokumentasi singkat diperbarui (README, bantuan di dalam aplikasi, atau AGENTS.md).
- Tidak ada perubahan tak terduga pada inti upstream.
- Dicoba manual di mesin bersih untuk fitur yang menyentuh onboarding atau konfigurasi.

### 16.9 Komunikasi

- Sinkronisasi singkat sekitar 15 menit, dua sampai tiga kali seminggu: apa yang selesai, apa yang menghambat, perubahan kontrak.
- Satu sesi retrospektif singkat di akhir setiap milestone.
- Perubahan kontrak atau pembagian area dibahas langsung, bukan hanya lewat komentar PR.

### 16.10 Risiko khusus tim dan mitigasi

| Risiko | Mitigasi |
| --- | --- |
| Konflik merge karena sama-sama mengedit file yang sama | Kepemilikan folder (16.2), PR kecil, merge ke main sering |
| Satu pihak menunggu pihak lain (blocking) | Kontrak dan mock di M0 sehingga B bisa maju lebih dulu, testConnection versi sederhana dirilis di M1 |
| Beban kerja tidak seimbang (A cenderung lebih berat di M2 dan M4, B di M3) | Tinjau di tiap retrospektif, saling bantu lewat review atau pair session |
| Kode AI tidak konsisten antara dua perangkat | AGENTS.md bersama, lint dan formatter otomatis, review silang |
| Kuota gratis habis karena dipakai dua orang untuk pengembangan dan pengujian | Satu API key per orang (satu akun per orang, bukan satu orang memakai banyak akun), serta Ollama lokal untuk pengujian rutin |
| Perbedaan pemahaman karena tidak ada catatan | docs/keputusan.md dan papan tugas yang selalu diperbarui |

### 16.11 Hal yang perlu disepakati berdua sebelum mulai

1. Siapa menjadi Orang A dan siapa Orang B. orang A = Raffxzr ( saya ),&#32;
2. Alat vibe coding yang dipakai (dan apakah AGENTS.md cukup, atau perlu aturan tambahan per alat).
3. Siapa yang memegang merge dari upstream (usulan: Orang A).
4. Jadwal sinkronisasi mingguan dan alat komunikasi.
5. Pembagian hak akses repositori dan aturan proteksi branch main.
6. Jawaban atas pertanyaan terbuka di bagian 14, terutama bahasa antarmuka dan cara distribusi.

## 17. Keputusan tim (hasil kesepakatan awal)

Dicatat 11 November 2026. Bila ada pertentangan dengan bagian 9, 12, atau 16, bagian ini yang berlaku. Jawaban yang ditulis langsung di bagian 14 dan 16.11 tetap dipertahankan sebagai catatan asli.

### 17.1 Tim

| Peran | Orang | Tanggung jawab |
| --- | --- | --- |
| Orang A | Raffxzr | Core dan Provider, pemegang merge dari upstream |
| Orang B | Dirgatama (username GitHub menyusul) | UX dan Produk |

### 17.2 Keputusan produk (menjawab bagian 14)

| Topik | Keputusan | Dampak pelaksanaan |
| --- | --- | --- |
| Bahasa antarmuka | Indonesia dan Inggris, default Indonesia | Sejak M1 semua teks UI lewat sistem i18n (berkas bahasa), tidak ada teks yang ditulis langsung di kode. Menambahkan belakangan jauh lebih mahal. Pemilik: Orang B. |
| Lisensi NerveAI | MIT | Verifikasi lisensi upstream dan dependensi di M0, pertahankan pemberitahuan lisensi upstream, dan sediakan berkas LICENSE serta daftar atribusi. |
| Distribusi | npm dan binary tunggal | Binary dibangun otomatis oleh CI untuk tiap OS. Cek di M0 apakah upstream sudah punya skrip build yang bisa dipakai ulang. |
| Provider default jalur Cepat | Ditentukan setelah pengujian M4. Hipotesis awal untuk M1: Gemini Flash sebagai utama dan Groq sebagai fallback cepat | Hipotesis harus dibuktikan dengan uji tool calling nyata, bukan hanya dari kecepatan. |
| Telemetri anonim | Opsional dan nonaktif secara default, tidak boleh mengirim data pribadi, API key, atau isi percakapan | Rekomendasi: tidak dibangun di Fase 1 (butuh backend, kebijakan privasi, dan biaya). Cukup audit di M0 bahwa fork tidak mengirim data apa pun ke luar tanpa izin. Umpan balik beta lewat GitHub Issues. |
| Windows native | Windows 10 dan 11 didukung sejak beta untuk fitur utama (instalasi, konfigurasi, penggunaan). Menggantikan catatan WSL di bagian 9. | CI diuji juga di Windows, uji manual di Windows bersih, dan perhatikan perbedaan path, shell (PowerShell/cmd), serta perilaku terminal pada tool shell dan TUI. |

Prinsip yang disepakati: mudah dipakai, fleksibel, ringan, tidak merepotkan pengguna, dengan keamanan, privasi, dan kenyamanan sebagai syarat.

### 17.3 Keputusan operasional (menjawab 16.11)

**Alat vibe coding.** AGENTS.md menjadi sumber aturan tunggal untuk semua alat. Masing-masing bebas memakai alat favoritnya. Jika suatu alat membaca berkas aturan lain (misalnya berkas khusus alat tersebut), isi berkas itu cukup satu baris yang menunjuk ke AGENTS.md, supaya aturan tidak tersebar dan tidak saling bertentangan. Setelah harness dasar bisa berjalan, gunakan NerveAI atau OpenCode sendiri dengan provider gratis untuk mengembangkan NerveAI (dogfooding), karena ini juga menjadi uji nyata. Untuk area berisiko tinggi di 16.6 (penyimpanan key, izin shell, privacy guard, logika kuota), pakai model terkuat yang tersedia dan tetap review manual.

**Repositori dan hak akses.**

- Buat satu **organisasi GitHub** untuk proyek (misalnya nerveai, cek ketersediaan nama), dan taruh repo utama di sana. Raffxzr sebagai owner, Dirgatama sebagai anggota dengan akses write. Dengan begitu proyek tidak terikat pada satu akun pribadi, mudah dialihkan, dan siap menerima kontributor.
- Repo bersifat **publik** (selaras dengan lisensi MIT). Pada repo publik, proteksi branch dan secret scanning tersedia gratis. Pada repo privat paket gratis, fitur proteksi sangat terbatas, jadi verifikasi bila memilih privat.
- Proteksi branch main: wajib lewat pull request, minimal 1 persetujuan dari orang lain, CI harus hijau, force push dilarang, merge dengan squash, dan branch dihapus setelah merge.
- Tidak ada API key di repo. .env masuk .gitignore dan secret scanning diaktifkan.
- Upstream OpenCode ditambahkan sebagai remote bernama upstream. Merge dari upstream dilakukan oleh Raffxzr di branch terpisah (misalnya sync/upstream-YYYY-MM), lalu masuk ke main lewat PR biasa.

**Jadwal dan komunikasi.**

- **Harian:** check-in teks sekitar 5 menit di grup chat (apa yang selesai, rencana hari ini, hambatan). Tidak perlu rapat.
- **Sabtu, sekitar 3 jam:**
  - 0:00–0:30 review PR dan tentukan prioritas minggu depan.
  - 0:30–2:15 kerja berdua (pair session atau integrasi antar lapisan).
  - 2:15–3:00 uji lintas lapisan dan lintas OS, merge dari upstream bila jadwalnya (sebulan sekali), retrospektif singkat.
- **Alat:** grup chat (WhatsApp atau Discord) dan GitHub Projects untuk papan tugas.

### 17.4 Dampak pada roadmap

Keputusan di 17.2 menambah pekerjaan pada roadmap bagian 12:

| Milestone | Tambahan |
| --- | --- |
| M0 | Audit upstream: lisensi, ada tidaknya pengiriman data keluar, dukungan Windows dan skrip build binary |
| M1 | Sistem i18n (Indonesia dan Inggris), CI di Linux, macOS, dan Windows |
| M3 | Pipeline build binary per OS |
| M5 | Uji di Windows bersih, binary Windows, dokumentasi dua bahasa |

Perkiraan 10 minggu pada bagian 12 berpotensi mundur sekitar 2 minggu. Jika jadwal mepet, urutan prioritas pemangkasan: fitur P2 lebih dulu, lalu binary (npm tetap jalan), sedangkan i18n dan Windows lewat npm dipertahankan karena murah bila dilakukan sejak awal dan mahal bila ditambah belakangan.
