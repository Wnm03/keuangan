# Instruksi untuk Claude Code — Repo Keluarga W

Repo ini adalah PWA client-side (tanpa backend) untuk manajemen keuangan,
zakat, bisnis, dan kendaraan keluarga. Source dipecah per fitur, lalu
digabung jadi `app-bundle-a.min.js` / `app-bundle-b.min.js` oleh `build.js`.

## Perintah penting
- `npm install` — sekali di awal (untuk eslint/esbuild).
- `npm run lint` — ESLint (`eslint.config.js`).
- `npm test` — `node --test tests/*.test.js`, unit test asli (bukan mock).
- `npm run build` — jalankan `build.js`, hasilkan bundle.
- `npm run check` — lint && test && build, jalankan semua sekaligus.

## Tugas default kalau diminta "perbaiki bug" / "self-test" / "fix sampai hijau"
1. Jalankan `npm run check`.
2. Kalau ada yang gagal:
   - Baca error paling atas dulu (biasanya akar masalah).
   - Untuk error test: baca pesan `_selfTestAssert` di `tests/*.test.js`
     (sudah deskriptif dalam Bahasa Indonesia), lalu cari fungsi terkait
     di **file sumber**, BUKAN di `app-bundle-a.min.js` / `app-bundle-b.min.js`
     — file itu hasil build otomatis dan akan tertimpa lagi tiap build.
   - Untuk error lint: ikuti aturan `eslint.config.js`.
   - Untuk error build: cek `build.js` dan urutan GROUP_A/GROUP_B di komentar
     paling atas tiap file `features-*.js` — banyak modul saling referensi
     jadi urutan load penting, jangan diubah sembarangan.
3. Buat perubahan sekecil mungkin yang menyelesaikan akar masalah.
4. Jalankan lagi `npm run check`, ulangi sampai semua pass, 0 fail, build sukses.
5. Kalau perbaikan yang "benar" butuh keputusan produk (bukan sekadar bug
   teknis, misal aturan pajak/zakat berubah) — STOP dan tanya dulu, jangan menebak.
6. Di akhir, ringkas: apa yang rusak, kenapa, dan file apa saja yang diubah.

## Yang tidak boleh disentuh langsung
- `app-bundle-a.min.js`, `app-bundle-b.min.js` — hasil build, edit di source lalu build ulang.
- Urutan file di `build.js` (GROUP_A/GROUP_B) — hanya diubah kalau memang ada alasan struktural yang jelas.
