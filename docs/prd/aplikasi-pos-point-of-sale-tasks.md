---
prd: "Aplikasi POS (Point of Sale)"
task_count: 13
---
## T1. Setup infrastruktur VPS

### Deskripsi
Persiapan server fisik/manual: instal OS, konfigurasi firewall, pasang reverse proxy (Nginx/Caddy), atur domain dan SSL, siapkan lingkungan untuk Nuxt frontend dan Golang backend.

### Acceptance Criteria
- [ ] Server online dengan domain + SSL valid; firewall hanya membuka port yang diperlukan; reverse proxy mem-proxy frontend ke port API dan backend ke DB; health check endpoint merespons 200.

## T2. Scaffold monorepo

### Deskripsi
Buat struktur monorepo: paket Nuxt (SSR) untuk frontend, proyek Golang (Go modules) untuk API service layer, dan Postgres sebagai source of truth. Pisahkan frontend dan backend melalui REST/GraphQL bridge.

### Acceptance Criteria
- [ ] Monorepo bisa build tanpa error; Nuxt dev server jalan di port frontend; Golang API compile dan serve; Postgres instance bisa diakses; frontend bisa memanggil endpoint backend.

## T3. Contract-first OpenAPI spec dan backend skeleton

### Deskripsi
Buat spec OpenAPI yang mendefinisikan model dan endpoint. Backend skeleton menyediakan dependency injection, logger, error handling, dan router yang membaca spec. Schema database (produk, transaksi, arus kas, laporan) dibuat di Postgres.

### Acceptance Criteria
- [ ] OpenAPI spec valid dan ter-generate; server startup tanpa panic; endpoint root dan health check tersedia; migrasi database berjalan dan tabel sesuai spec ter-buat.

## T4. Implementasi auth dan role

### Deskripsi
Buat sistem autentikasi dan otorisasi berbasis role (kasir/admin/pemilik). Gunakan signed cookie atau JWT, hash password bcrypt, isolasi data per-user. Session aman dengan logout.

### Acceptance Criteria
- [ ] Login/logout berfungsi; token session valid dan kadaluarsa; password di-hash; kasir tidak bisa akses endpoint admin/pemilik; data per-user ter-isolasi; logout membatalkan akses.

## T5. Modul Inventory: CRUD produk, kategori, stok, threshold

### Deskripsi
Buat endpoint CRUD produk (SKU, nama, kategori, harga beli/jual, stok), kategori, penyesuaian stok manual dengan audit log, dan threshold minimum. Stok menjadi source of truth.

### Acceptance Criteria
- [ ] CRUD produk dan kategori jalan; stok bisa di-koreksi manual dengan audit log tercatat; produk di bawah threshold memicu alert; request tanpa hak akses ditolak.

## T6. Modul Penjualan: kasir, keranjang, pembayaran, cetak struk

### Deskripsi
Buat UI dan endpoint kasir: tambah barang ke keranjang, pilih metode bayar (tunai/kartu/kredit), diskon, pajak. Sistem otomatis hitung subtotal, diskon, pajak, total. Cetak struk/barcode.

### Acceptance Criteria
- [ ] Transaksi tersimpan real-time ke DB; total akurat setelah diskon & pajak; struk bisa dicetak/barcode; refund bisa dilakukan; transaksi tersimpan.

## T7. Integrasi penjualan dengan inventory (stok turun otomatis, atomic)

### Deskripsi
Kaitkan modul penjualan dengan inventory: saat transaksi tersimpan, stok otomatis berkurang dalam DB transaction atomic. Implementasi optimistik rollback — UI langsung update, sinkron ke DB, rollback jika gagal.

### Acceptance Criteria
- [ ] Stok berkurang otomatis per transaksi; jika transaksi gagal stok dikembalikan (rollback); transaksi atomic lewat DB transaction; UI update optimistik dan sinkron dengan DB.

## T8. Modul Akuntansi: pencatatan arus kas, laba rugi, saldo

### Deskripsi
Buat pencatatan arus kas: penjualan arus kas masuk, pembelian stok arus kas keluar. Hitung laba rugi per periode dan saldo kas. Audit trail tersambung ke transaksi dan inventory.

### Acceptance Criteria
- [ ] Saldo kas akurat per transaksi; arus kas masuk/keluar tercatat; laporan laba rugi per periode bisa diekspor; audit trail tersambung ke transaksi dan inventory.

## T9. Modul Laporan: query, grafik, filter, export

### Deskripsi
Buat laporan penjualan harian/bulanan, top produk, produk terlaris, margin per produk, stok terlarus. Filter periode & kategori. Render grafik dan tabel real-time. Export CSV/PDF.

### Acceptance Criteria
- [ ] Grafik & tabel render real-time; filter periode & kategori akurat; export CSV/PDF berfungsi; margin dan stok terlarus terhitung benar.

## T10. Multi-user, hak akses, session

### Deskripsi
Buat manajemen multi-user: isolasi data per-user di semua modul, validasi session server-side, proteksi endpoint, dan pembatalan akses saat logout. Integrasikan dengan auth.

### Acceptance Criteria
- [ ] Data per-user ter-isolasi di semua modul; token session valid dan divalidasi server-side; CORS & rate-limit dikonfigurasi; logout membatalkan akses; backup DB berkala berjalan.

## T11. Testing: unit backend dan e2e frontend

### Deskripsi
Buat test unit untuk backend (business logic, ORM, endpoint) dan e2e untuk frontend (alur kasir, laporan, hak akses).

### Acceptance Criteria
- [ ] Test unit backend passing; e2e frontend passing untuk alur kasir, laporan, dan hak akses; coverage untuk path kritis.

## T12. Deployment dan backup otomatis

### Deskripsi
Deploy monorepo ke VPS manual. Konfigurasi backup otomatis database, retry logic, error boundary UI, dan monitoring.

### Acceptance Criteria
- [ ] Aplikasi live di VPS; backup database otomatis berjalan; error boundary UI berfungsi; monitoring menampilkan status kesehatan.

## T13. QA, fix, launch

### Deskripsi
Lakukan QA menyeluruh: uji semua modul, perbaiki bug, verifikasi acceptance criteria. Lanjutkan ke produksi.

### Acceptance Criteria
- [ ] Semua acceptance criteria terpenuhi; bug kritis diperbaiki; aplikasi berjalan stabil di produksi.
