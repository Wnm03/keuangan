# Struktur Repo — Apa yang Masuk Git vs Di-exclude

File `.gitignore` di root sudah mengatur ini secara otomatis. Dokumen ini
cuma penjelasan singkat kenapa, biar jelas kalau nanti ada yang nanya lagi.

## ✅ MASUK repo (di-commit)

```
kw/
├── *.html                  # app_production.html, index.html
├── *.js                    # semua source fitur (transaksi.js, akun.js, dst)
├── *.css                   # styles.css
├── app-bundle-a.min.js     # hasil build produksi (dipakai app, bukan cuma artefak)
├── app-bundle-b.min.js     #   idem
├── manifest.json, sw.js    # PWA config
├── package.json
├── eslint.config.js
├── icon-192.svg, icon-512.svg
├── *.md                    # CLAUDE.md, FILE-MAP.md, PRE-MERGE-LINT-CHECK.md,
│                            #   CATATAN-CEK-CLAUDE.md — semua dokumentasi aktif
├── scripts/                # release.sh, generate-file-map.js, collect-app-globals.js
├── bump-version.sh
├── rollback.sh
└── tests/                  # semua *.test.js + tests/helpers/
```
**Kenapa:** semua ini source of truth atau dokumentasi/tooling aktif yang
dipakai proses `npm run check` / `npm run release`. Tanpa ini, orang lain
(atau Claude Code sesi berikutnya) tidak bisa build/test ulang.

## ❌ EXCLUDE dari repo (masuk `.gitignore`)

```
kw/backups/     # 8 file bundle timestamped, snapshot otomatis tiap build/merge
kw/archive/     # dokumen basi/OBSOLETE (mis. PEMISAHAN-FILE-ROADMAP...OBSOLETE)
node_modules/   # dependency, di-install ulang lewat npm install
*.log
.DS_Store
```
**Kenapa:**
- `kw/backups/` — bukan source of truth, akan terus numpuk tiap kali build
  kalau ikut di-commit (sudah 5MB dari 8 file, akan makin besar seiring waktu).
  Cara resmi bikin rilis sudah ada di `CLAUDE.md` (`npm run release`, pakai
  `git archive` dari commit), jadi snapshot manual ini tidak diperlukan di
  histori Git.
- `kw/archive/` — arsip dokumen yang sudah ditandai OBSOLETE, disimpan lokal
  untuk referensi tapi tidak perlu ikut nge-track di Git.

## Kalau folder ini sudah kadung ke-commit sebelumnya

`.gitignore` cuma mencegah file BARU ikut ter-track. File yang sudah pernah
di-commit tetap ada di histori sampai dihapus manual dari tracking:

```bash
git rm -r --cached kw/backups kw/archive
git commit -m "chore: stop tracking kw/backups & kw/archive (lihat .gitignore)"
```
