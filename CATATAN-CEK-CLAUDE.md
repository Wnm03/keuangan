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

- ✅ **[2026-07-12] Integrasi Kasir/Order dengan `priceRekoWidgetList` (kw194-kasir-order-pricereko, build #194).**
  Lanjutan dari item "Ide lanjutan yang BELUM dikerjakan" di entri OngkirCalc di bawah: widget
  "🤖 Rekomendasi Harga Jual AI" (`PriceRekoWidget`) sebelumnya HANYA kelihatan di tab Etalase —
  kasir yang checkout langsung dari tab 🧠 Kasir atau form 🛒 Transaksi Manual (Order) tidak tahu
  kalau harga jual sebuah produk sudah menyimpang jauh dari estimasi, kecuali sengaja buka Etalase
  dulu.
  **Perubahan** (`cobek.js` + `kasir.js` + `styles.css`):
  1. `PriceRekoWidget.checkOne(p)` (baru, `cobek.js`) — versi per-produk dari `scan()`, balikin
     `{reko,diffPct}` kalau produk menyimpang ≥`THRESHOLD_PCT` dari estimasi, atau `null` kalau
     wajar/belum ada Harga Beli-Jual. `scan()` di-refactor supaya reuse fungsi ini (bukan
     reimplementasi rumus 2x) — SATU sumber kebenaran rumus reko dipakai Etalase, Kasir, & Order.
  2. `Kasir.renderGrid()` — tile produk yang flagged dapat badge ⬇️/⬆️ (elemen `<button>` terpisah
     pojok kiri-atas tile, class `.kasir-tile-pricewarn`, BUKAN nempel ke `data-action` tile itu
     sendiri supaya tap badge tidak ikut nge-trigger `addToCart`) — tap badge memanggil
     `Kasir.openPriceReko(pid)` yang delegasi ke `PriceRekoWidget.openDetail()` yang sudah ada
     (buka `productModal` produk itu, auto-expand panel Rekomendasi Harga Jual).
  3. `Kasir.renderCart()` — baris keranjang yang flagged dapat hint teks kecil "Reko Etalase: RpX"
     di bawah nama produk.
  4. `Order.renderItems()` (form 🛒 Transaksi Manual lama, `cobek.js`) — baris item yang flagged
     dapat hint sama ("Reko Etalase: RpX") + link "detail →" yang memanggil
     `openPriceRekoWidgetDetail(pid)` (wrapper `PriceRekoWidget.openDetail` yang sudah ada).
  5. CSS baru `.kasir-tile-pricewarn` di `styles.css` (badge bulat kecil, posisi absolute pojok
     kiri-atas, mirror `.kasir-tile-badge` yang sudah ada di pojok kanan-atas utk qty keranjang).
  Modul `Kasir` (POS) TIDAK diubah alur checkout/`recordShopSale`-nya sama sekali — murni tambahan
  visual+link ke alur yang sudah ada, konsisten dgn rumus & UX "🔍 Detail" yang sudah dipakai widget
  Etalase.
  **Verifikasi:** 2 test baru `PriceRekoWidget.checkOne()` + 2 test baru `Order.renderItems()`
  ditambahkan ke `tests/cobek.test.js`; file test baru `tests/kasir.test.js` (12 test, `kasir.js`
  sebelumnya nol test sama sekali) mencakup badge/hint reko baru DI ATAS jalur inti Kasir
  (`renderGrid`/`renderCart`/`addToCart`/`computeTotals`/`_checkoutInner` sukses & gagal). `npm
  test` → 1059/1059 pass (naik dari 1043). `node build.js` → sukses, versi naik ke build #194.
  Belum sempat smoke-test browser (Playwright) sesi ini — perlu dicoba visual (khususnya tap badge
  ⬆️/⬇️ di tile Kasir tidak ikut nge-trigger tambah ke keranjang) sebelum dianggap 100% final, tapi
  logic inti sudah diverifikasi lewat unit test. `npm run lint` juga belum bisa dijalankan (sandbox
  tanpa akses internet).

- ✅ **[2026-07-12] Preferensi jarak/ongkos per Produsen di OngkirCalc (kw192-ongkir-produsen-pref, build #193).**
  Lanjutan dari `PriceReko`/`OngkirCalc` (kw190/191): sebelumnya tiap buka panel "📍 Hitung dari
  Jarak & Ongkir" di `productModal`, field Etape 1 (Jarak km & Ongkos/km "Ambil ke Produsen") selalu
  kosong walau produknya dari produsen yang SAMA dengan sebelumnya — padahal jarak rute ke 1 produsen
  kan tetap, cuma jumlah pcs/etape 2 (ke rumah konsumen) yang beda-beda tiap order.
  **Perubahan** (`cobek.js` + `modals.js`):
  1. `D.produsen[].jarakKm`/`.biayaPerKm` (field baru, opsional) menyimpan rute Etape 1 per produsen.
  2. `OngkirCalc.prefillFromProdusen()`: dipanggil saat panel Ongkir dibuka (`toggle()`) & saat ganti
     Produsen (`Etalase.onProdusenChange()`) — isi otomatis field jarak/ongkos KALAU kosong (tidak
     menimpa input manual yang sudah ada), plus tampilkan hint di `#ongkirProdusenPrefHint` (baru,
     di atas field Etape 1 di `modals.js`).
  3. `OngkirCalc.saveProdusenPref()`: link baru "💾 Simpan sbg rute tetap Produsen ini" di bawah
     field Etape 1 — validasi Produsen & Jarak terisi, simpan ke `D.produsen`, toast konfirmasi.
  4. Ganti Produsen di dropdown `pProdusen` sekarang RESET dulu field Etape 1 lalu isi ulang dari
     preferensi produsen yang baru dipilih (bukan nyisa dari produsen sebelumnya).
  5. `Produsen.renderList()` (tab Bisnis Shop → Produsen) menampilkan rute tersimpan (📍 X km × Rp/km)
     di baris info produsen kalau sudah ada.
  Etape 2 (Pekalongan→Rumah Konsumen) SENGAJA TIDAK disimpan per produsen karena beda-beda tiap order.
  **Verifikasi:** 12 test baru ditambahkan di `tests/cobek.test.js` (prefill kosong/ada-rute/tidak-
  menimpa-input-manual, saveProdusenPref validasi & sukses, toggle() & onProdusenChange() memanggil
  prefill). `npm test` → 1043/1043 pass (naik dari 1033, +10 di cobek.test.js karena beberapa test
  digabung). `node build.js` → sukses, versi naik ke build #193. Belum sempat smoke-test browser
  (Playwright) sesi ini — perlu dicoba visual sebelum dianggap 100% final, tapi logic inti sudah
  diverifikasi lewat unit test.
  **Ide lanjutan yang BELUM dikerjakan (dari daftar user)**: buffer % susut/pecah di kalkulasi harga
  (PriceReko/OngkirCalc belum memperhitungkan barang pecah/rusak saat transport), dan integrasi
  Kasir/Order dengan `priceRekoWidgetList` (widget rekomendasi harga di Etalase belum terhubung ke
  alur POS Kasir).

- ✅ **[2026-07-11] 2 bug dari laporan screenshot user (build v188): renderDashboard() crash
  "Cannot set properties of null (setting 'textContent')" & toast "Tombol ini belum berfungsi
  (setCobekTab)".**
  1. **Isolasi error per-card di `renderDashboard()`** (`modules-render.js`): loop
     `DASH_RENDER_ORDER` sebelumnya memanggil `cardDef.render(dashCtx)` TANPA try/catch —
     kalau SATU card melempar error (mis. data anggaran/kategori yang sudah rusak), SISA card
     setelahnya di urutan render (`laporanMini`/`fi`/`pensiun`/`absensi`/`eduFund`/`refleksi`)
     ikut TIDAK ter-render ulang sama sekali, user cuma dapat toast generik "Ada error kecil"
     dari `_friendlyErrorNotice` tanpa tahu card mana yang bermasalah. Sekarang tiap card
     dibungkus try/catch sendiri, kegagalan dicatat `console.warn` & dilewati — card lain tetap
     lanjut normal.
  2. **`Budget.renderDashMini()` diperkeras** (`features-budget-laporan-carnotes-pelanggan.js`):
     4 elemen anak (`dashBudgetUsed`/`dashBudgetLimit`/`dashBudgetPct`/`dashBudgetBar`) dulu
     diambil & langsung ditulis TANPA null-check (beda dari pola card lain, mis.
     `renderDashLaporanMini` sudah `if(!trendEl||!katEl)return;`) — inilah sumber paling
     mungkin dari "Cannot set properties of null (setting 'textContent')" yang dilaporkan user
     persis di test self-test "renderDashboard() ikut memanggil mini-card Anggaran". Sekarang
     ke-4 elemen dicek dulu sebelum ditulis, fallback `card.style.display='none'` kalau ada yang
     hilang (bukan crash).
  3. **Alias kompatibilitas mundur `setCobekTab`→`setShopTab`** (`cobek.js`): tombol tab Bisnis
     Shop di-rename dari `setCobekTab` ke `setShopTab` saat redesign Etalase (lihat entri
     redesign di atas), tapi PWA yang service worker-nya belum sempat refresh HTML (skenario:
     app dibuka offline/cache lama) masih bisa menyimpan markup LAMA dengan
     `data-action="setCobekTab"` sementara bundle JS SUDAH ter-update ke versi baru → tombol itu
     memanggil fungsi yang sudah tidak ada, persis toast "Tombol ini belum berfungsi
     (setCobekTab)" yang dilaporkan user. Source `index.html`/`app_production.html`/`cobek.js`
     saat ini SUDAH 100% pakai `setShopTab` (dicek eksplisit, tidak ada sisa `setCobekTab` di
     source) — alias ini murni jaring pengaman transisi utk kombinasi HTML-lama+JS-baru di sisi
     klien, bukan tanda ada bug rename yang belum tuntas.
  **Verifikasi:** `npm test` → 1020/1020 pass (tidak ada test lama yang berubah perilakunya).
  `node build.js` → sukses, versi naik ke build #189
  (`kw83-test-pengaturan-search-5`/`kw-cache-v189`). Direproduksi & dikonfirmasi via Playwright +
  Chrome headless: `renderDashboard()` dgn `D.budgets` terisi tidak lagi melempar error tak
  tertangani; `setCobekTab('etalase', el)` dipanggil langsung → berhasil pindah tab tanpa error
  (membuktikan alias jalan). Sisa 1 kegagalan self-test yang TIDAK terkait laporan user
  (`loadMoreBbmList` tanpa aria-label) sudah ada SEBELUM perubahan ini & bukan bagian dari 2 bug
  yang dilaporkan — belum dikerjakan di sesi ini, lihat "BELUM DIKERJAKAN" di bawah.

- ✅ **[2026-07-11] Test `filter-laporan.js`** (lanjutan daftar nol-test
  ringan→berat dari bagian ke-33 `pengaturan-search.js`; sempat tertunda
  krn cabang kerja ini fokus ke redesign Etalase dulu). File 220 baris (221
  di versi sebelum redesign Etalase — beda cuma penamaan `cobek`→`shop`:
  `#page-cobek`→`#page-shop`, `setCobekTab`→`setShopTab`,
  `cobekTabName`→`shopTabName`, fungsinya identik). Test-nya sendiri
  awalnya ditulis & diverifikasi di snapshot v174 (belum ada redesign
  Etalase), lalu di-port ke sini dgn menyesuaikan penamaan tsb — bukan
  ditulis ulang dari nol. Cakupan: filter panel Keuangan (`kf*`) & Laporan
  (`f*`) — `txMatchesFilters`/`txMatchesSearch` (murni), `getLaporanFilters`/
  `getKeuFilters`/`resetLaporanFilter`/`resetKeuFilter`/`populateCatFilter`/
  `populateKeuFilters`/`onFKatChange`/`onKfKatChange`/`toggleKeuFilter`,
  simpan/pulihkan preferensi filter ke localStorage (`saveKeuFilterPrefs`/
  `loadKeuFilterPrefsIntoDOM`, dgn guard sekali-muat `_keuFilterPrefsLoaded`),
  badge jumlah filter aktif (`updateKfBadge`), paginasi list (`loadMoreTx`/
  `loadMoreLapTx`/`resetTxPageAndRender`, debounce pencarian
  `onKfSearchInput`), navigasi antar-list dgn scroll+highlight (`goToList`,
  termasuk cabang tab Shop `etalase`/`produsen`/`riwayat`/`pelanggan` &
  Car Notes), & modal ringkasan transaksi terfilter dari 3 scope
  dashboard/keuangan/laporan dgn paginasi 100/batch (`showFilteredTx`).
  **Tidak ada bug ditemukan** — murni menambah test yg sebelumnya nol.
  `npm test` → 1020/1020 pass (naik dari 969, +51 test baru). `node
  build.js` → sukses, versi naik ke build #188, `FILE-MAP.md` diregenerasi.
  Smoke-test browser (Playwright + Chrome headless) → bersih, 0
  `pageerror`, `✅ [smoke-test] OK`; dicoba juga live: `toggleKeuFilter()`,
  `resetKeuFilter()`, `showFilteredTx('dashboard','all',...)`,
  `goToList('page-etalase',null,undefined,'etalase')` — semua jalan &
  fungsi `setShopTab` (nama baru pasca-redesign) terkonfirmasi ada &
  terpanggil dgn benar. **Daftar nol-test ringan→berat BELUM tuntas** —
  sisa (per pengecekan `loadSource([...])` di seluruh `tests/*.test.js`
  sesi ini, TAPI cek ulang lagi krn 1x sudah kejadian ada file kelewat):
  `kasir.js`, `sewakios.js`, `linktx.js`, `modal-navigasi.js`,
  `payroll-absensi.js`, `renovasi.js`, `tagihan-kalender.js`,
  `backup-restore.js`, `features-aiwidget-reminder-gdrive-search.js`.
  (`cobek.js` SUDAH punya test — ditambahkan bareng redesign Etalase;
  `features-sheets-pwa-selftest.js` SEBAGIAN tercakup lewat `extractFunction`
  di `tests/parse-angka.test.js`, belum full.) Detail teknis test & jebakan
  `vm`/`fakeDom.js` yg ditemukan: lihat `CLAUDE.md`, catatan kerja
  2026-07-11 bagian ke-34.

- ✅ **[2026-07-11] Redesign tampilan kartu produk Etalase (tab Bisnis Shop → Etalase) jadi lebih profesional.**
  Sebelumnya kartu produk pakai layout generik `.tx-item` (sama seperti baris riwayat
  transaksi biasa). Sekarang pakai layout khusus `.shop-product-card` (di `styles.css`):
  - Badge stok berwarna sesuai level: 🔴 "Menipis" (≤2 pcs), 🟡 "Terbatas" (≤5 pcs),
    🟢 "Aman" (>5 pcs), plus garis aksen warna di sisi kiri kartu.
  - Tag kategori & produsen sbg pill terpisah (bukan teks digabung dgn "·").
  - Blok harga jelas: kalau produk punya "Diskon Default %" (field `pDiskon` yang
    sudah ada di form produk), harga normal dicoret & harga final + persen diskon
    ditonjolkan warna aksen; kalau tidak ada diskon, tampil harga jual polos.
    Harga modal & harga reseller tetap sbg info sekunder di bawahnya.
  - Badge margin (nominal + persen) & tombol edit/hapus dikelompokkan rapi di kanan.
  Perubahan di `cobek.js` (`Etalase.renderList()`), `styles.css` (kelas baru
  `.shop-product-*`), dan `index.html`/`app_production.html` (wrapper `#productList`
  dapat class `shop-product-grid`). Test baru ditambahkan di `tests/cobek.test.js`
  (badge stok per level, tampilan diskon vs tanpa diskon) — total 969 test, semua
  pass. Diverifikasi visual via Playwright + Chrome headless (screenshot tab Etalase
  dgn data produk contoh, termasuk smoke-test bawaan app tetap ✅ OK). Build dijalankan
  ulang (`node build.js`) sampai versi v187, bundle `app-bundle-a/b.min.js` &
  `index.html`/`app_production.html`/`sw.js` sudah konsisten di v187.
  **Belum dikerjakan (permintaan berikutnya dari user):** redesign POS Kasir, dan
  widget AI rekomendasi harga jual/reseller yang menghitung ongkos transport
  berdasarkan rute nyata (produsen → Pekalongan → konsumen, atau ambil di rumah)
  — saat ini `PriceReko.autoFillTransport()` masih pakai rata-rata Rp/liter dari
  log BBM tanpa memperhitungkan jarak/rute.

- ✅ **[2026-07-11] `cobek.js` — test suite (102 test) disesuaikan dgn rebranding "Cobek"→"Shop".**
  Sejak v163/v164, banyak identifier/DOM-id di `cobek.js` di-rename dari
  awalan `Cobek`/`cobek` jadi `Shop`/`shop` (mis. `resolveCobekKategori`→
  `resolveShopKategori`, `recordCobekSale`→`recordShopSale`, `renderCobek`→
  `renderShop`, id `cobekList`→`shopList`, `#page-cobek`→`#page-shop`, dst
  — murni rename, TIDAK ada perubahan logika/behavior, diverifikasi lewat
  `diff` baris-per-baris). Data layer TIDAK berubah: `D.cobek`,
  `D.cobekKategori`, dan properti `cobekLinkId` tetap memakai nama lama,
  begitu juga label `subcategory:'Cobek'` di transaksi. `tests/cobek.test.js`
  diupdate mengikuti mapping rename ini (78 token diverifikasi cocok satu-
  satu ke source baru). Dijalankan via `node --test tests/*.test.js`: 966
  test, semua pass, tidak ada regresi di file lain.

- ✅ **[2026-07-11] Bug: tombol "💰 Sudah Gajian?" SELALU reset minggu SEKARANG, walau user
  lagi browse ke minggu LAMA yang pending** (build #182). Ditemukan langsung dari pertanyaan
  user "absensi pending dimana lihat & dimana konfirmnya" — ternyata notif pending (fitur
  sebelumnya) mengarahkan user ke tombol yang secara diam-diam SELALU pakai `new Date()` utk
  hitung rentang minggu, bukan minggu yang sedang ditampilkan di layar (`Payroll.weekStart`,
  diubah via panah ‹ › di atas Riwayat Absensi). Akibatnya: notif pending tidak pernah bisa
  benar-benar diselesaikan lewat tombol itu — yang ke-reset/dicatat selalu minggu sekarang
  (kosong/salah), minggu lama yang dimaksud tetap nyangkut selamanya.
  **Fix** (`reset-gaji-mingguan.js`): `openWeeklyResetManual()` sekarang pakai `Payroll.weekStart`
  (fallback ke minggu real sekarang kalau modul Payroll belum termuat) sbg target rentang minggu,
  simpan ke `_wrLastStart`/`_wrLastEnd` (baru). `confirmWeeklyReset()` pakai `_wrLastStart`/
  `_wrLastEnd` yang sudah "dikunci" saat modal dibuka (bukan hitung ulang `new Date()`) — supaya
  konsisten dgn minggu yang ditampilkan ke user & aman dari race condition tanggal berganti persis
  saat modal terbuka. `checkWeeklySalaryReset()` (prompt otomatis tiap Sabtu) tetap pakai minggu
  real sekarang seperti semula (memang scope-nya cuma minggu berjalan), tapi ikut isi
  `_wrLastStart`/`_wrLastEnd` supaya konsisten.
  **Cara pakai sekarang (untuk user):** buka 📅 Absensi & Kalkulator Gaji Harian → tab Absensi →
  pakai panah ‹ › di atas "Riwayat Absensi" utk browse ke minggu yang pending (sesuai yg disebut
  di notif ⚠️) → kalau minggu itu ada isinya, tombol "💰 Sudah Gajian? Catat & Reset Minggu Ini"
  otomatis muncul di bawah ringkasan gaji minggu itu → tap → konfirmasi di modal "Sabtu Gajian!".
  `npm test` → 821/821 pass (1 test toast text sempat berubah krn typo saya sendiri, sudah
  dikembalikan persis semula agar tidak perlu ubah test). `node build.js` → sukses, versi naik
  ke 182. Lint & smoke-test browser BELUM dijalankan sesi ini (sandbox tanpa Chrome/Playwright).

- ✅ **[2026-07-11] Fitur Absensi: field "Tambahan Lain-lain (Rp)" + notif pending bisa di-dismiss**
  (build #181). 2 temuan dari user (lewat screenshot form Absensi):
  1. Form Absensi cuma punya field "Potongan Lain-lain", tidak ada lawannya untuk nominal
     tambahan (bonus/uang makan/dsb). **Fix:** tambah field `whTambahan` (mirror field
     Potongan) di `modals.js`, dibaca & dimasukkan ke rumus total di `payroll-absensi.js`
     (`total = pokok+lembur+tambahan-potongan`, berlaku di cabang jam biasa & borongan),
     disimpan sebagai `w.tambahan` di `D.workDays`, tampil di breakdown ringkasan gaji
     mingguan & di tiap item Riwayat Absensi, dan ikut ke-load/reset saat edit/batal-edit.
  2. Notif "⚠️ Ada absensi dari N minggu sebelumnya..." di atas Riwayat Absensi cuma teks HTML
     tanpa cara ditutup — muncul terus tiap buka Absensi walau user sudah paham. **Fix:**
     tambah tombol ✕ (`Payroll.dismissPendingOldWeeksBox`) yang menyimpan `weekStart` minggu
     yang lagi ditampilkan ke `D.payrollDismissedWeeks` (baru, default `[]`, migrasi di
     `features-helpers-global-security.js`). Box notif (`renderPendingOldWeeksBox`) sekarang
     pakai `pendingOldWeeksInfoVisible()` yang memfilter minggu ter-dismiss; badge status di
     Dashboard (`renderDashMini`) TETAP pakai `pendingOldWeeksInfo()` mentah (tidak ikut
     ke-filter) supaya status asli tidak disembunyikan permanen — kalau ada minggu pending
     BARU yang menumpuk lagi, notif otomatis muncul lagi (dismiss bukan "matikan selamanya").
  **Belum ada test otomatis ditambahkan** — `payroll-absensi.js` termasuk daftar nol-test
  yang belum digarap (lihat `FILE-MAP.md`). `npm test` → 821/821 pass (tidak ada yang berubah,
  murni menambah kode baru yang belum ada test-nya). `node build.js` → sukses, versi naik ke
  181. Lint & smoke-test browser BELUM dijalankan sesi ini (sandbox tanpa Chrome/Playwright
  saat itu) — perlu diverifikasi visual sebelum dianggap 100% selesai.

- ✅ **[2026-07-11] Test `reset-gaji-mingguan.js`** (lanjutan daftar nol-test
  ringan→berat, setelah `profil-pengaturan.js`). File 86 baris: `getWeekRange`
  (rentang minggu Minggu-Sabtu), `checkWeeklySalaryReset` (deteksi hari Sabtu
  + prompt sekali sehari + filter absensi dlm rentang minggu),
  `openWeeklyResetManual` (alur reset manual dari tombol Absensi/Kalkulator
  Gaji), `confirmWeeklyReset` (konfirmasi reset + catat Pemasukan otomatis,
  dgn fallback kategori & akun). Dipakai `class FakeDate extends Date` custom
  (bukan stub objek biasa) krn source butuh `new Date()`/`new Date(x)`
  berperilaku beda tapi tetap 1 class yg sama. **Tidak ada bug ditemukan** —
  murni menambah test yg sebelumnya nol. `npm test` → 813/813 pass (naik dari
  795, +18 test baru). `node build.js` → sukses, versi naik ke build #173.
  Smoke-test browser (Playwright + Chrome headless) → bersih, 0 `pageerror`.
  Detail lengkap: lihat `CLAUDE.md`, catatan kerja 2026-07-11 bagian ke-32.
  Sisa daftar nol-test berikutnya: `filter-laporan.js` (✅ SELESAI —
  lihat entri paling atas file ini; catatan: sesi berikutnya sempat
  loncat duluan ke `pengaturan-search.js` sebelum akhirnya balik
  mengerjakan `filter-laporan.js`, urutan bukan strict berurutan).

- ✅ **[2026-07-11] Test `profil-pengaturan.js`** (lanjutan daftar nol-test
  ringan→berat, setelah `error-handler.js`/`onboarding.js` &
  `diagnostik-versi.js`). File 81 baris: `autoSaveProfile` (baca form profil,
  fallback default, field opsional dgn guard sendiri2), `profilePTKPStatus`
  vs `profileJiwaKeluarga` (2 fungsi murni serupa tapi beda aturan clamp
  tanggungan), `updateProfilPTKPPreview`, `updateUsiaPreview`,
  `selectStatusKawin`/`selectTanggungan`/`selectStatusPekerjaan` (toggle chip
  + save), `toggleApiKeyHint`. **Tidak ada bug ditemukan** — murni menambah
  test yg sebelumnya nol. `npm test` → 795/795 pass (naik dari 764, +31 test
  baru). `node build.js` → sukses, versi naik ke build #172. Smoke-test
  browser (Playwright + Chrome headless) → bersih, 0 `pageerror`. Detail
  lengkap: lihat `CLAUDE.md`, catatan kerja 2026-07-11 bagian ke-31. Sisa
  daftar nol-test berikutnya: `reset-gaji-mingguan.js`.

- ✅ **[2026-07-11] Test `error-handler.js` + `onboarding.js`** (2 file paling
  ringan dari daftar nol-test tersisa, dikerjakan duluan sesuai urutan
  ringan→berat). `error-handler.js` (37 baris): throttle toast 3 detik,
  fallback console.warn kalau `toast()` belum siap, error di dalam
  `toast()` ditangkap diam-diam, & 2 listener global (`error`/
  `unhandledrejection`) — disuntik `window`/`Date`/`console` tiruan lewat
  `extraGlobals` krn stub bawaan `loadSource()` no-op (tidak bisa
  maju-mundurkan waktu / menyimpan handler). `onboarding.js` (40 baris):
  rumus estimasi gaji bulanan & sisa kiriman (`updateOnboardPreview`),
  validasi PIN 4 digit, & alur simpan profil+PIN (`finishOnboard`).
  **Tidak ada bug ditemukan** — murni menambah test yg sebelumnya nol.
  `npm test` → 733/733 pass (naik dari 715, +18 test baru). `node build.js`
  → sukses, versi naik ke `kw80-merge-advisor-card-dashcards-42` (build
  #167), `FILE-MAP.md` diregenerasi (kedua file otomatis hilang dari daftar
  nol-test). Detail lengkap: lihat `CLAUDE.md`, catatan kerja 2026-07-11
  bagian ke-28.

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

- Self-test "UI: elemen interaktif (data-action) yang cuma berisi ikon/emoji/tanpa teks wajib
  punya aria-label" gagal utk `<button data-action="loadMoreBbmList">` (BBM.renderList() di
  `features-budget-laporan-carnotes-pelanggan.js`, sekitar variabel `bbmMoreWrap`) — tombol
  "muat lebih banyak" ini dirender tanpa teks/aria-label. TIDAK terkait 2 bug dari screenshot
  user (sudah ada sebelum sesi ini), belum diperbaiki krn di luar scope laporan user kali ini.

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
