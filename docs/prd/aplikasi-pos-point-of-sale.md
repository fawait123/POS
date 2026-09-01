# Aplikasi POS (Point of Sale)

## Versi
- Versi: 1.0
- Tanggal: 2026-08-09
- Status: Draft

## Ringkasan / Overview
Sistem POS terpadu untuk usaha ritel/warung dengan empat modul inti: penjualan (transaksi kasir), inventory (stok barang), akuntansi (pencatatan arus keuangan), dan laporan (analisa penjualan). Tujuannya agar pemilik toko bisa jual barang, pantau stok, rekam keuangan, dan lihat performa bisnis dari satu antarmuka tanpa beralih antar-sistem. Nilai: eliminasi pekerjaan manual entri data antar-modul, stok tidak kebonoran karena salah sinkronisasi, dan laporan keuangan akurat tanpa spreadsheet manual.

## Sasaran & Non-Sasaran
- Sasaran: operator kasir, admin toko, pemilik usaha kecil-menengah (UKM) hingga menengah, pemilik bisnis fisik dengan 1–50 SKU.
- Non-Sasaran: perusahaan manufaktur dengan batch tracking kompleks, e-commerce multi-pengguna skala besar, bisnis yang butuh integrasi ERP penuh.

## Persona & Use Case
- Persona utama: Admin Toko — kelola produk, kategori, harga, stok, dan lihat laporan.
- Persona utama: Kasir — proses transaksi cepat via kasir (mouse/touchscreen), cetak struk, bayar tunai/kartu.
- Persona utama: Pemilik — pantau arus kas, margin, dan stok terendah dari dashboard.
- Use case inti: Kasir scan barang → sistem otomatis kurangi stok → jika stok kurang muncul warning → bayar → transaksi tersimpan ke akuntansi → admin lihat laporan penjualan harian.

## Persyaratan Fungsional (Requirements)
1. FR-1: Modul Penjualan
   - Deskripsi: Kasir bisa tambah barang ke keranjang, pilih metode bayar (tunai/kartu/kredit), diskon, dan bayar. Sistem otomatis hitung subtotal, diskon, pajak, total, dan cetak struk.
   - Acceptance criteria: Transaksi tersimpan real-time ke DB; struk bisa dicetak/barcode; total akurat setelah diskon & pajak; refund bisa dilakukan.

2. FR-2: Modul Inventory
   - Deskripsi: CRUD produk (SKU, nama, kategori, harga beli/jual, stok), stok otomatis turun saat terjual, stok masuk manual (pembelian), threshold minimum, dan alert stok rendah.
   - Acceptance criteria: Stok berkurang otomatis per transaksi; produk di bawah threshold muncul alert; stok bisa di-koreksi manual dengan audit log.

3. FR-3: Modul Akuntansi
   - Deskripsi: Setiap transaksi penjualan tercatat sebagai arus kas masuk; pembelian stok tercatat arus kas keluar; laba rugi per periode; saldo kas.
   - Acceptance criteria: Saldo kas akurat per transaksi; laporan laba rugi bisa diekspor; audit trail tersambung ke transaksi dan inventory.

4. FR-4: Modul Laporan
   - Deskripsi: Laporan penjualan harian/bulanan, top produk, produk terlaris, margin per produk, dan stok terendah. Filter periode & kategori.
   - Acceptance criteria: Grafik & tabel render real-time; export CSV/PDF; filter periode akurat.

5. FR-5: Multi-user & Hak Akses
   - Deskripsi: Role-based access (kasir/admin/pemilik), session aman, data per-user terisolasi.
   - Acceptance criteria: Kasir tidak bisa akses laporan; token session valid; logout membatalkan akses.

## Persyaratan Non-Fungsional
- Performa: Halaman kasir responsif < 200ms; transaksi optimistik (UI langsung update, sync ke DB). Support 100+ produk di satu listing tanpa lag.
- Keamanan: Password di-hash (bcrypt); token JWT/signed cookie; validasi input server-side (Golang); CORS & rate-limit di VPS; backup DB berkala.
- Skalabilitas: Arsitektur API-driven (Golang) di-behind-frontend (Nuxt SSR); Postgres sebagai source of truth; siap horizontal scale via load balancer jika traffic naik.
- Ketersediaan: Backup otomatis, retry logic, error boundary UI.
- Kompatibilitas: Web app (Nuxt SSR) + opsional client native (Tauri) untuk kasir layar sentuh.

## Batasan & Risiko
- Batasan teknis: Nuxt (JS) di-frontend, Golang di-backend → perlu bridge API (REST/GraphQL). Tidak monolitik = kompleksitas deployment VPS manual lebih tinggi.
- Batasan bisnis: Target UKM → fitur tidak over-engineered; cloud ops terlalu mahal, jadi VPS manual.
- Risiko: Sinkronisasi stok vs transaksi salah → mitigasi: transaksi atomic (DB transaction) + optimistik rollback. Kerusakan DB → mitigasi: backup + seed. Integrasi dua bahasa lambat → mitigasi: contract-first API (OpenAPI) + mock.

## Task Breakdown (Outline)
1. Setup VPS manual (OS, firewall, reverse proxy, domain).
2. Scaffold monorepo: Nuxt frontend + Golang API (Go modules) + Postgres.
3. Contract-first OpenAPI spec; backend skeleton.
4. Auth & role (kasir/admin/pemilik).
5. Modul Inventory: CRUD produk, kategori, stok, threshold.
6. Modul Penjualan: kasir, keranjang, pembayaran, cetak struk.
7. Integrasi: penjualan ↔ inventory (stok turun otomatis, atomic).
8. Modul Akuntansi: pencatatan arus kas, laba rugi, saldo.
9. Modul Laporan: query, grafik, filter, export.
10. Multi-user, hak akses, session.
11. Testing (unit backend + e2e frontend).
12. Deployment & backup otomatis.
13. QA, fix, launch.

## Stack Teknologi
- Frontend: **Nuxt.js** (SSR) — UI kasir, dashboard, laporan. Terhubung ke backend via REST/GraphQL.
- Backend: **Golang** (Go modules) — API service layer, business logic, ORM, auth. Performa tinggi untuk transaksi & query.
- Database: **PostgreSQL** — source of truth, relational data (produk, transaksi, arus kas, laporan).
- Server: **VPS manual** — deploy self-hosted, full control atas firewall, backup, monitoring.

→ skipped: detail teknis per modul (ORM choice, API endpoint list, schema ERD) — tambahkan saat breakdown task detail.
→ skipped: opsi client native Tauri untuk kasir — tambahkan jika kasir layar sentuh jadi prioritas.