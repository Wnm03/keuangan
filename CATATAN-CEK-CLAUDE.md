# Catatan Cek & Perbaikan — untuk Claude di sesi berikutnya

> **Cara pakai file ini (WAJIB dibaca dulu):**
> 1. Ini daftar kerja: apa yang **sudah** diverifikasi beres, dan apa yang **belum**.
> 2. Kalau kamu mengerjakan salah satu item "BELUM DIKERJAKAN" dan sudah
>    kamu **verifikasi sendiri** (jalan di browser / `npm run check` hijau),
>    pindahkan ke bagian "SUDAH SELESAI" dengan tanggal, atau — kalau memang
>    tidak ada tindak lanjut lain yang perlu dicatat — **hapus saja barisnya**
>    dari file ini supaya file ini tidak menggembung jadi riwayat basi.
> 3. Jangan tandai "selesai" hanya berdasarkan asumsi/baca kode — pengujian
>    nyata dulu (browser via Playwright + Chrome di `/home/claude/.cache/puppeteer/chrome/...`,
>    atau `npm run check`).

## SUDAH SELESAI (terverifikasi)

- ✅ **[2026-07-11] Housekeeping dokumentasi + `FILE-MAP.md` otomatis.**
  3 hal: (1) 2 item ✅ yang kesasar di "BELUM DIKERJAKAN" dipindah ke
  "SUDAH SELESAI" (2 entri di bawah ini); (2) `PEMISAHAN-FILE-ROADMAP.md`
  yang sudah basi (nyebut file yang sudah tidak ada) dipindah ke
  `archive/PEMISAHAN-FILE-ROADMAP.md.OBSOLETE-2026-07-11.md` dgn header
  peringatan; (3) script baru `scripts/generate-file-map.js` yang
  generate `FILE-MAP.md` (peta file+ringkasan & index fungsi global→file)
  OTOMATIS dari source, dipanggil tiap `node build.js` sukses — supaya
  TIDAK PERNAH basi seperti roadmap lama. **Mulai sekarang: cek
  `FILE-MAP.md` dulu kalau mau tahu "fungsi X ada di file mana" atau
  "file Y isinya apa", sebelum grep manual.** Detail lengkap: lihat
  `CLAUDE.md`, catatan kerja 2026-07-11 bagian ke-12.
  `npm test` → 187/187 pass. `node build.js` → sukses, versi naik ke 155,
  `FILE-MAP.md` ke-generate otomatis. Smoke-test browser tetap ✅ OK, 0
  error (perubahan sesi ini murni tooling, tidak menyentuh kode runtime).

- ✅ **[2026-07-11] Sinkronisasi BBM ↔ Transaksi ↔ Car Notes** — sudah diuji
  otomatis & 1 bug nyata ditemukan+diperbaiki (field dasar catatan BBM basi
  kalau checkbox "Sinkron ke Catatan Mobil" mati saat edit transaksi).
  Detail lengkap: lihat `CLAUDE.md`, catatan kerja 2026-07-11 bagian ke-3.

- ✅ **[2026-07-11] Logic Torsi Sparepart** (katalog 60+ spesifikasi torsi
  Honda Vario 125, kalibrasi kunci torsi fisik MOLLAR MLR-B11950) — sudah
  ditambah 22 test murni-logika utk `Torsi.calcExt` (kalkulator ekstensi/
  sambungan kunci), konversi satuan, & mode checklist servis. TIDAK ada bug
  ditemukan di sesi ini — murni menambah cakupan test yg sebelumnya nol utk
  area ini. Detail lengkap: lihat `CLAUDE.md`, catatan kerja 2026-07-11
  bagian ke-4.

- ✅ **[2026-07-11] Split `transaksi.js` → `tx-list-cashflow.js`** (area
  terakhir dari roadmap split, atas permintaan eksplisit user). Pindah 9
  fungsi + 1 var state verbatim: `txHTML`, `delTx`, `changeMonth`,
  `txListPeriode`, `setTxListPeriode`, `getTxListRange`, `setPeriode`,
  `getRange`, `computeCashflowForecast`, `setKeuanganTab`. `transaksi.js`
  864 → 729 baris. Terdaftar di `GROUP_B` (`build.js`) tepat sebelum
  `transaksi.js`. Build ke versi 154.
  **Diverifikasi lewat browser (Playwright + Chrome headless):**
  - Semua fungsi ke-expose ke `window`, tidak ada yang nyangkut di scope
    modul.
  - `changeMonth`, `getTxListRange`, `getRange`, `computeCashflowForecast`
    dites langsung, hasil masuk akal & tanpa error.
  - `txHTML(t)` dgn data contoh → markup benar, `data-action`
    editTx/delTx ter-escape rapi.
  - `setKeuanganTab` gonta-ganti tab Kelola↔Laporan tanpa error.
  - `delTx()` end-to-end: tambah dummy tx → hapus → array balik bersih.
  - Smoke-test internal: `✅ OK — 992 referensi getElementById() & 55
    data-action semuanya valid`, 0 `pageerror`.
  - `npm test` → 187/187 pass, 0 fail (tidak ada test yang perlu diubah).
  Detail lengkap: lihat `CLAUDE.md`, catatan kerja 2026-07-11 bagian
  ke-11. **Dengan ini seluruh roadmap split `transaksi.js` (bagian ke-5
  s/d ke-11) tuntas** — `transaksi.js` sekarang murni form Tambah/Edit
  Transaksi + beberapa fungsi kecil lintas-domain yang sengaja dibiarkan
  gabung (skala kecil, tidak layak jadi file sendiri).

- ✅ **[2026-07-11] Verifikasi browser split `tx-cobek.js` + `tx-target.js`**
  (kerjaan bagian ke-9 di sesi sebelumnya sudah lolos sintaks/unit-test,
  tapi verifikasi visual di browser belum sempat — sandbox saat itu tanpa
  Chrome/Playwright). Sesi ini punya akses Chrome
  (`/home/claude/.cache/puppeteer/chrome/...`) + Playwright global, jadi
  langsung dicek nyata (bukan cuma baca kode). **Tidak ditemukan bug** —
  murni verifikasi, tidak ada perubahan kode. Hasil:
  - `isCobekStockCatName` (di `tx-cobek.js`): dites pakai kategori Cobek
    ASLI di data (`Bisnis › Cobek`, id `sub_cb_cobek`) → `true`. Dites nama
    kategori/sub acak yang tidak nyambung → `false` (tidak asal-true).
    Dites skenario inti fitur ini (rename total nama kategori & sub jadi
    "Bisnis Kios Renovasi" / "Peralatan Rumah Tangga", TANPA ganti id) →
    tetap `true` lewat fallback id `sub_cb_cobek` — fallback rename-proof
    yang jadi alasan utama fungsi ini ada betul-betul jalan.
  - `openTargetModal`, `saveTarget`, `onTargetDanaDaruratToggle`,
    `showTargetAccountTx`, `addTarget`, `delTarget`, `onTargetAccChange`
    (semua di `tx-target.js`): semua ke-expose ke `window` (typeof
    `function`, tidak ada yang "hilang" nyangkut di scope modul). Alur
    nyata dicoba: buka modal → isi nama+nominal → `saveTarget()` → target
    baru nambah persis 1 di `D.targets` dengan field benar. Toggle "Dana
    Darurat" → hint rekomendasi muncul dengan teks & angka masuk akal.
    `delTarget()` pada target manual → tersplice bersih dari array.
  - Smoke-test internal: `✅ OK — 992 referensi getElementById() & 55
    data-action semuanya valid`, 0 `pageerror` di console selama semua
    skenario di atas.
  - `npm test` → 187/187 pass. `node build.js` → sukses, versi naik ke 153.
  **Kesimpulan: split ke-9 (Cobek + Target) bersih, tidak ada kekurangan
  atau sisa fungsi ganda/hilang.** Area terakhir yang masih menunggu split
  (belum dikerjakan) tetap "List Transaksi & Cashflow Forecast" — lihat
  item di bawah.

- ✅ **[2026-07-11] Bug: edit transaksi cicilan LAMA (histori) diam-diam menimpa jadwal
  cicilan aktif (termasuk kategori) yang dipakai buat semua pembayaran BERIKUTNYA.**
  Root cause: 1 bill cicilan (`D.bills`) dipakai bersama oleh SEMUA transaksi
  pembayarannya (semua transaksi punya `billLinkId` yang sama ke bill itu) — bill
  merepresentasikan jadwal/sisa cicilan yang LIVE, bukan snapshot 1 transaksi. Modal
  edit transaksi (`transaksi.js`) sebelumnya menyamakan "edit transaksi cicilan APAPUN
  yang tertaut" dengan "edit jadwal cicilan aktif": tiap kali user edit transaksi
  cicilan (termasuk yang sudah lama/histori, misal cuma mau betulin kategori bulan
  lalu), field jadwal (total harga/tenor/bunga/jatuh tempo/**kategori**) di bill ikut
  ditimpa ulang dari form — form itu sendiri di-prefill dari state bill yang SEKARANG,
  jadi kalau user cuma ganti kategori/catatan tanpa sadar, kategori BILL (dan semua
  cicilan berikutnya yang belum dibayar) ikut berubah diam-diam.
  **Fix:** field jadwal cicilan/langganan (total/tenor/bunga/jatuh tempo/kategori/akun
  bill) sekarang hanya disinkron ke `D.bills` kalau transaksi yang diedit adalah
  transaksi TERBARU yang tertaut ke bill itu (id transaksi terbesar di antara semua
  yang share `billLinkId` sama). Kalau bukan (transaksi lama/histori), hanya field
  transaksi itu sendiri (kategori/subkategori/akun/catatan/tanggal) yang berubah — bill
  & transaksi lain sama sekali tidak tersentuh. User dikasih toast info kalau editnya
  kena transaksi lama, mengarahkan ke 📋 Riwayat Pembayaran kalau memang mau ubah jadwal.
  Berlaku juga utk tagihan `langganan` (bukan cuma `cicilan`), karena pola sharing bill-
  nya sama persis. Build ke versi 140.
  **Diverifikasi lewat browser (Playwright + Chrome headless), skenario nyata:**
  - Buat cicilan 6x @ Rp400rb/bulan → bayar 2x lewat `markBillPaid` (real code path,
    bukan mock) → sisa tenor 3, bill masih aktif.
  - Edit transaksi PALING LAMA (transaksi ke-1, sudah lewat 2 pembayaran berikutnya):
    ganti kategori jadi "Cicilan Motor (Koreksi)" → simpan. Hasil: kategori BILL tetap
    "Cicilan Motor" (tidak berubah), transaksi lain (2 pembayaran berikutnya) sama
    sekali tidak berubah, cuma transaksi yang diedit yang kategorinya berubah. ✓
  - Edit transaksi TERBARU/aktif: ganti Total Harga jadi Rp3.000.000 (dari Rp2.400.000)
    → simpan. Hasil: `bill.totalHarga` & `bill.amount` (cicilan/bulan) ikut update
    sesuai perhitungan baru, seperti sebelumnya (perilaku yang benar tetap jalan). ✓
  - `npm test` → 103/103 pass, 0 fail. `node build.js` → sintaks bundle valid.
  - Smoke-test internal tetap bersih: `✅ OK — 992 referensi getElementById() & 55
    data-action semuanya valid`.

- ✅ **[2026-07-10] Bug: `FinCoach.dismiss` & `FinCoach.showAll` tidak ke-expose ke `window`.**
  Root cause: modul `FinCoach` (di `modules-calc.js`) tidak dimasukkan ke daftar
  `Object.assign(window,{...})` di `features-sheets-pwa-selftest.js` — beda dari
  ~40 modul lain yang sudah ada di daftar itu (kelihatan seperti human error/typo
  saat modul ini ditambahkan). Akibatnya tombol ✕ (sembunyikan insight) dan
  "Lihat semua →" di widget "🩺 Insight Cepat" Dashboard diam/toast error saat
  ditap.
  **Fix:** tambahkan `FinCoach` ke daftar `Object.assign(window,{...})`, lalu
  `node build.js` (versi naik ke 137). Diverifikasi: smoke-test internal jadi
  `✅ OK — 991 referensi getElementById() & 55 data-action semuanya valid`,
  dan klik langsung tombol dismiss di browser headless tidak lagi memicu error.

- ✅ **[2026-07-10] Saran UX: modal tidak bisa ditutup dengan tombol Escape.**
  Ditambahkan listener `keydown` global di `modal-navigasi.js` (setelah
  `openQS`/`closeQS`) yang menutup modal/overlay yang lagi terbuka saat
  Escape ditekan, dengan urutan prioritas: kalkulator popup → quick-switcher
  (`qsXxx`) → modal sistem (confirm/prompt/choice/info/pinPrompt — lewat
  resolver `_xxxAnswer` masing2 supaya `Promise` yang di-`await` tetap
  ke-resolve, bukan nge-hang) → modal fitur generik `.overlay.open` (kalau
  ada beberapa bertumpuk, pilih yang z-index tertinggi lebih dulu).
  Build ke versi 138.
  **Diverifikasi lewat browser (Playwright + Chrome headless):**
  - Modal generik (`globalSearchModal`) → Escape menutup, `open` jadi false. ✓
  - `askConfirm()` → Promise ter-resolve `false` lewat Escape, tidak hang. ✓
  - Modal bertumpuk (kalkulator di atas modal transaksi) → Escape menutup
    satu-per-satu (calc dulu, baru modal induk di Escape ke-2), bukan
    langsung menutup semua sekaligus. ✓
  - Smoke-test internal tetap bersih: `✅ OK — 992 getElementById() & 55
    data-action semuanya valid`.
  - `npm test` → 103/103 pass, 0 fail.

## BELUM DIKERJAKAN (butuh tindak lanjut di sesi berikutnya)

_(kosong — tidak ada item pending per 2026-07-11)_

## Cara jalanin pengecekan otomatis lagi (kalau perlu ulang dari nol)

Server statis lokal + Playwright pakai Chrome dari cache Puppeteer (karena
`npx playwright install` butuh internet & jaringan container ini biasanya
mati). Semua HARUS dalam satu pemanggilan bash (server mati kalau sesi bash
berakhir):

```bash
cd app-fixed && (python3 -m http.server 8877 > /tmp/server.log 2>&1 &)
sleep 1
node /path/ke/script-probe.js   # chromium.launch({ executablePath:
                                #   '/home/claude/.cache/puppeteer/chrome/linux-131.0.6778.204/chrome-linux64/chrome',
                                #   args:['--no-sandbox','--disable-setuid-sandbox'] })
```

Cek smoke-test bawaan app (paling cepat & paling dipercaya untuk nangkep
`data-action` yang putus / `getElementById` yang hilang): buka
`http://localhost:8877/index.html`, tunggu ~1 detik, baca console — harus
muncul `✅ [smoke-test] OK — ... semuanya valid`. Kalau muncul
`❌ [smoke-test] Ditemukan N masalah`, itu bug nyata yang harus diperbaiki
sebelum kirim ke user.
