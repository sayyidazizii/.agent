# TASK SPECIFICATION

## 1. SCOPE & WHITELIST (STRICT)
- **Target Files (Modifiable)**:
  - `src/controllers/upload.controller.ts`
  - `src/services/storage.service.ts`
- **Read-Only Context Files**:
  - `src/types/storage.ts`
- **PROHIBITION**: JANGAN modifikasi file di luar Target Files di atas tanpa izin tertulis!

## 2. GOALS & REQUIREMENTS
- [Tulis objektif tugas di sini]
- Contoh: Implementasikan endpoint upload file yang memvalidasi magic bytes untuk format PDF/PNG.
- Contoh: Simpan file dengan nama UUIDv4 di luar web root dan catat metadatanya ke database.

## 3. SECURITY & PERFORMANCE DIRECTIVES
- Wajib patuhi seluruh batasan di: `.agent/rules/security-guardrails.md`
- Jalankan seluruh protokol audit di: `.agent/skills/security-audit.md`
- Jika butuh library baru, jelaskan alasannya dan minta izin terlebih dahulu sebelum menginstal.
