# 🤖 .agent — AI Coding Governance & Execution Telemetry Framework

Framework terstruktur untuk mengontrol, mengaudit, dan memantau agen pengodean AI (seperti **Antigravity CLI (`agy`)**, Claude Code, Cursor, Codex, dll.) dengan prinsip **Zero-Trust Security**, batasan whitelist file yang ketat, dan telemetri eksekusi real-time.

---

## 📌 Daftar Isi
- [Latar Belakang & Tujuan](#-latar-belakang--tujuan)
- [Struktur Direktori](#-struktur-direktori)
- [Penjelasan Komponen & Modul](#-penjelasan-komponen--modul)
  - [1. task.md](#1-taskmd)
  - [2. rules/security-guardrails.md](#2-rulessecurity-guardrailsmd)
  - [3. skills/security-audit.md](#3-skillssecurity-auditmd)
  - [4. prompt.example.txt](#4-promptexampletxt)
  - [5. logs/execution.log & logs/logs.html](#5-logsexecutionlog--logslogshtml)
- [Contoh Data Log Telemetri](#-contoh-data-log-telemetri)
- [Panduan Penggunaan (Workflow)](#-panduan-penggunaan-workflow)
  - [Langkah 1: Integrasi ke Proyek Target](#langkah-1-integrasi-ke-proyek-target)
  - [Langkah 2: Isi Spesifikasi Task](#langkah-2-isi-spesifikasi-task)
  - [Langkah 3: Jalankan Perintah CLI Agent](#langkah-3-jalankan-perintah-cli-agent)
  - [Langkah 4: Pantau via Dashboard](#langkah-4-pantau-via-dashboard)
- [Dashboard Visual (HTML Telemetry)](#-dashboard-visual-html-telemetry)

---

## 🎯 Latar Belakang & Tujuan

Ketika memberikan kendali kepada agen AI untuk mengubah kode pada proyek perangkat lunak, terdapat risiko nyata berupa:
1. **Scope Creep / Unintended Edits**: AI mengubah file di luar area yang diminta.
2. **Security Vulnerabilities**: Kode yang dihasilkan rentan terhadap Path Traversal, Insecure Deserialization, SSRF, SQL Injection, atau kebocoran kredensial.
3. **Loss of Visibility**: Tidak ada catatan audit mengenai apa yang dilakukan agen, berapa lama proses berlangsung, dan apakah kode lulus verifikasi keamanan.

Framework `.agent` mengatasi hal tersebut dengan pendekatan **3 Lapis Pertahanan**:
* **Lapisan 1 (Kontrol Cakupan)**: Whitelist file yang boleh diubah via `task.md`.
* **Lapisan 2 (Batasan Keamanan & Performa)**: Larangan keras (negative constraints) via `rules/security-guardrails.md`.
* **Lapisan 3 (Audit & Telemetri)**: Protokol pengujian menyeluruh via `skills/security-audit.md` yang otomatis tercatat ke `logs/execution.log` dan dapat dipantau di `logs/logs.html`.

---

## 📂 Struktur Direktori

```plaintext
.agent/
├── logs/
│   ├── execution.log            # Catatan riwayat eksekusi task (format Markdown Table)
│   └── logs.html                # Dashboard visual HTML interaktif (auto-parser log)
├── rules/
│   └── security-guardrails.md   # Pedoman keamanan ketat & batasan nol-toleransi (Zero-Trust)
├── skills/
│   └── security-audit.md        # Standar operasional audit diff, batas cakupan, & Big-O
├── prompt.example.txt           # Template perintah CLI untuk mode Dry-Run, Eksekusi, & Review
├── task.md                      # Lembar kerja spesifikasi task & target whitelist file
└── README.md                    # Dokumentasi lengkap framework
```

---

## 🧩 Penjelasan Komponen & Modul

### 1. `task.md`
Dokumen kerja utama sebelum instruksi diberikan kepada AI. Berfungsi sebagai **kontrak pengerjaan**:
* **Scope & Whitelist (Strict)**: Menentukan secara presisi file mana yang **boleh dimodifikasi** (`Target Files`) dan file yang hanya boleh **dibaca sebagai konteks** (`Read-Only Context Files`). File di luar daftar ini dilarang keras diubah.
* **Goals & Requirements**: Rincian tujuan teknis dan fungsionalitas yang ingin dicapai.
* **Security & Performance Directives**: Referensi ke aturan keamanan yang wajib dipatuhi.

### 2. `rules/security-guardrails.md`
Kumpulan aturan negatif (prohibitions) dengan prinsip Zero-Trust:
* **Secrets & Credentials**: Dilarang hardcode kredensial, API key, token JWT, atau password.
* **File Handling**: Wajib validasi file upload via *magic bytes*, penyimpanan di luar *webroot*, penamaan acak (UUIDv4), proteksi *Zip Slip*, serta limit ukuran file.
* **Strict Input Validation**: Menolak *blacklist-based filtering*; wajib menggunakan *allowlist schema* (Zod, Joi, Pydantic, DTO).
* **Arbitrary Behavior Prevention**: Menolak eksekusi kode dinamis (`eval`, `child_process.exec`), dynamic imports berbasis user input, dan SSRF (blokir IP privat/RFC 1918 dan metadata cloud).
* **Database & Injection**: Menolak *query concatenation*; wajib parameterized queries.
* **Efficiency & Regression**: Dilarang algoritma sub-optimal $O(N^2)$ jika dapat diselesaikan dalam $O(N)$ atau $O(\log N)$, dilarang memory leak, dan dilarang mengubah fungsionalitas di luar scope.

### 3. `skills/security-audit.md`
Prosedur verifikasi wajib yang harus dijalankan AI setelah menulis/mengubah kode:
1. **Scope Boundary Check**: Memastikan modifikasi hanya terjadi pada file whitelist.
2. **Threat Modeling & Vulnerability Audit**: Checklist pengujian kerentanan (Magic bytes, Path Traversal, SSRF, Deserialization, Parameterized DB).
3. **Performance & Complexity Review**: Verifikasi Big-O, pembersihan stream/koneksi, dan timeout.
4. **Regression & Edge-Case Audit**: Penanganan payload kosong/null dan eksekusi test/linter proyek.
5. **Append Execution Log**: Kewajiban menulis 1 baris hasil eksekusi ke `logs/execution.log`.
6. **Mandatory Terminal Summary**: Menampilkan tabel rangkuman audit sebelum merespons user.

### 4. `prompt.example.txt`
Contoh format instruksi untuk memanggil CLI agent (**Antigravity CLI / agy**):
* **Mode 1: Perencanaan Aman (Dry-Run)**: AI menyusun rencana eksekusi dan threat model tanpa mengubah file apapun sebelum dikonfirmasi.
* **Mode 2: Eksekusi Penuh (Execute & Self-Audit)**: AI mengimplementasikan kode, menerapkan guardrails, melakukan self-audit, dan mencatat log.
* **Mode 3: Audit Perubahan Manual (Diff Review)**: AI mengevaluasi perubahan `git diff` terkini yang ditulis secara manual oleh pengembang.

### 5. `logs/execution.log` & `logs/logs.html`
* **`execution.log`**: File log berbasis Markdown Table yang mencatat riwayat run setiap task secara persisten.
* **`logs.html`**: Antarmuka berbasis browser mandiri (standalone HTML/CSS/JS) dengan tema gelap modern, metrik statistik (Total Runs, Success Rate, Failed Runs, Security Passes), fitur pencarian instan, dan filter status.

---

## 📊 Contoh Data Log Telemetri

Data pada `logs/execution.log` menggunakan format Markdown Table standar:

```markdown
| Timestamp | Task Name | Status | Duration | Security Audit | Summary / Root Cause |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 2026-10-06 09:30 | Setup File Upload Service | SUCCESS | 45s | PASS | Implementasi endpoint upload file dengan validasi magic bytes (PNG/PDF) dan penyimpanan UUID di luar webroot |
| 2026-10-06 10:05 | User Authentication & JWT Refresh | SUCCESS | 1m 12s | PASS | Implementasi schema validasi Zod, query SQL parameterized, dan penghapusan password hash dari response |
| 2026-10-06 10:40 | Archive Decompression Utility | FAILED | 35s | FAIL | Ditolak: Terdeteksi potensi Zip Slip vulnerability pada path ekstraksi arsip tanpa verifikasi canonical path |
| 2026-10-06 11:00 | Fix Archive Decompression (Zip Slip) | SUCCESS | 28s | PASS | Memperbaiki validasi canonical path menggunakan path.resolve boundary check dan isolasi target directory |
| 2026-10-06 11:15 | External Webhook Dispatcher | SUCCESS | 50s | PASS | Enforce SSRF protection: memblokir private IP (RFC 1918, 127.0.0.1, 169.254.169.254) dan membatasi protokol ke HTTPS |
```

### Keterangan Kolom:
| Kolom | Tipe Nilai | Penjelasan |
| :--- | :--- | :--- |
| **Timestamp** | `YYYY-MM-DD HH:mm` | Waktu saat task selesai dieksekusi |
| **Task Name** | String | Nama/judul tugas sesuai yang tercantum di `task.md` |
| **Status** | `SUCCESS` / `FAILED` | Status keberhasilan eksekusi kode |
| **Duration** | String (cth: `45s`, `1m 12s`) | Estimasi atau durasi pengerjaan task oleh agen |
| **Security Audit**| `PASS` / `FAIL` | Hasil pemeriksaan terhadap checklist di `security-guardrails.md` |
| **Summary / Root Cause** | String | Ringkasan perubahan yang dilakukan, atau alasan kegagalan/pelanggaran |

---

## 🚀 Panduan Penggunaan (Workflow)

### Langkah 1: Integrasi ke Proyek Target
Salin folder `.agent` ke dalam *root directory* proyek Anda:

```bash
# Clone atau salin folder .agent ke proyek Anda
cp -r /path/to/.agent /path/to/your-project/
```

Jika tidak ingin folder `.agent` ikut ter-commit ke repositori utama proyek, tambahkan ke exclusion git:
```powershell
# Di PowerShell:
Set-Content -Path .git\info\exclude -Value ".agent/" -Encoding ASCII
```
*(Atau tambahkan `.agent/` ke dalam `.gitignore` proyek).*

---

### Langkah 2: Isi Spesifikasi Task
Buka `.agent/task.md` dan tentukan target pengerjaan:

```markdown
## 1. SCOPE & WHITELIST (STRICT)
- **Target Files (Modifiable)**:
  - `src/controllers/upload.controller.ts`
  - `src/services/storage.service.ts`
- **Read-Only Context Files**:
  - `src/types/storage.ts`

## 2. GOALS & REQUIREMENTS
- Implementasikan upload handler yang memvalidasi header magic bytes file gambar.
- Simpan file dengan nama UUID di storage lokal di luar webroot.
```

---

### Langkah 3: Jalankan Perintah CLI Agent

Jalankan perintah menggunakan CLI pilihan Anda (contoh menggunakan Antigravity CLI `agy`):

```bash
# Eksekusi penuh dengan audit mandiri
agy "Eksekusi seluruh instruksi di .agent/task.md. Patuhi aturan di .agent/rules/security-guardrails.md dan tampilkan audit summary dari .agent/skills/security-audit.md sebelum selesai."
```

Atau jalankan mode **Dry-Run** terlebih dahulu untuk mereview rencana tindakan:
```bash
agy "Baca spesifikasi di .agent/task.md. Buat execution plan dan threat model sesuai .agent/skills/security-audit.md. JANGAN ubah file apa pun sebelum saya konfirmasi."
```

---

### Langkah 4: Pantau via Dashboard
Buka file `.agent/logs/logs.html` di browser Anda untuk melihat riwayat eksekusi, tingkat keberhasilan, dan hasil audit keamanan secara langsung.

---

## 🖥️ Dashboard Visual (HTML Telemetry)

Fitur pada [`logs/logs.html`](file:///D:/PROJECT%20TESTING/.agent/logs/logs.html):
* **Cards Ringkasan**: Menampilkan **Total Runs**, **Success Rate (%)**, **Failed Runs**, dan **Security Checks Passed**.
* **Filter Interaktif**: Filter log berdasarkan status (`Semua`, `SUCCESS`, `FAILED`).
* **Instant Search**: Pencarian teks cepat berdasarkan nama task atau ringkasan tindakan.
* **Auto-Fetch / Manual Select**: Otomatis membaca `execution.log` jika dibuka via local web server, atau mendukung pemilihan file manual jika dibuka langsung melalui `file:///` di browser yang memiliki batasan CORS.

---

## 📄 Lisensi
Framework ini bersifat terbuka di bawah lisensi [MIT](LICENSE). Bebas digunakan, dimodifikasi, dan didistribusikan untuk proyek pribadi maupun enterprise.
