# PRD — Kain Nusantara ERP (lanjutan dari repo github.com/kakjsbsbs/KN)

## Problem statement (asli, 2026-09-17)
Lanjutkan development repo KN. Fitur dispatch/pengiriman masih sangat basic: tidak ada quick action di dasbor
(hanya list yang harus buat dispatch), belum ada history pengiriman, visualisasi status pengiriman yang sedang
diproses, ketersediaan armada internal, jenis pengiriman (kurir pihak ketiga dll). Pertanyaan: bagaimana fitur
customer ambil sendiri & validasinya? Data pengiriman (alamat kirim) masih bocor/terekspos pada SO yang ambil sendiri.

## Pilihan user
- Bangun semua sekaligus: dasbor dispatch + quick action, visualisasi status, ketersediaan armada, jenis pengiriman
  (internal / kurir pihak ketiga / ambil sendiri), history.
- Ambil sendiri: kode pickup unik + verifikasi identitas pengambil (nama + no. ID), tanpa driver/armada/alamat kirim.
- Kurir pihak ketiga: nama kurir, no. resi, biaya kirim, estimasi tiba.
- Armada: master kendaraan + driver dengan status (tersedia / dalam perjalanan / perawatan), status berubah otomatis.

## Arsitektur (yang disentuh)
- Backend FastAPI: `services/logistics_service.py` (moda self_pickup, kode pickup, handover, dashboard, history, redaksi kode),
  `services/fleet_service.py` (baru — master kendaraan `fleet_vehicles`, status sopir turunan), `routers/logistics.py`
  (endpoint baru), `schemas_logistics.py`, `entity_scope.py` (+fleet_vehicles scoped), `services/so_verify_service.py`
  (anti-bocor alamat pada SO ambil).
- Frontend React: `features/logistics/` → `LogisticsView` (tab Dasbor/Daftar/Riwayat/Armada), `DispatchDashboard`,
  `HistoryPanel`, `FleetPanel`, `PickupHandoverPanel`, `DeliveryCreateModal` (moda otomatis dari metode pemenuhan SO),
  `DeliveryDetailModal` (cabang pickup), `sales_admin/OrderPreviewCard` (sembunyikan alamat untuk SO ambil).
- Frontend TIDAK hot-reload: `setsid nohup bash /app/scripts/rebuild_frontend.sh > /app/.rebuild.out 2>&1 &`.
- Env: backend/.env CORS_ORIGINS harus daftar origin eksplisit (bukan `*`).

## Persona
Admin gudang / manajer (buat & kelola pengiriman, armada), petugas gudang (serah terima pickup), sopir (tugas hari ini),
sales / admin sales (pantau, bagikan kode pickup ke pelanggan).

## Sudah diimplementasikan (2026-09-17)
- Dasbor Pengiriman: KPI (SJ menunggu, menunggu diambil, diproses, di jalan, ETA hari ini, terlambat, terkirim hari ini),
  aksi cepat per SJ (Buat pengiriman / Siapkan pickup), antrean serah terima pickup, alur status (bar), moda (donut),
  ringkasan armada, daftar sedang diproses, baru selesai/gagal.
- Jenis pengiriman: Ekspedisi (kurir, resi, layanan, biaya kirim, ETA) · Armada sendiri (kendaraan master + sopir) ·
  Diambil pelanggan (kode pickup 6 karakter, tanggal ambil).
- Validasi ambil sendiri: SO `fulfillment_method=ambil` wajib moda self_pickup (dan sebaliknya ditolak); tidak ada
  alamat/plat/sopir; tahapan hanya lewat POST pickup-handover (kode cocok + nama pengambil [+ no. ID]); kode salah
  ditolak & dihitung; kode disembunyikan dari peran gudang/sopir (mereka mencocokkan kode yang disebut pengambil).
- Anti-bocor: `shipments/unassigned` & `order_preview` tidak mengirim alamat kirim untuk SO ambil.
- Riwayat: filter tanggal/moda/status/kata kunci, statistik (terkirim, gagal, rata-rata hari, total biaya ekspedisi), CSV.
- Armada: CRUD kendaraan, perawatan ⇄ tersedia, otomatis on_trip saat berangkat & lepas saat tiba/gagal/selesai,
  status sopir turunan dari pengiriman aktif; kendaraan maintenance/on_trip tak bisa dipilih.
- Seed demo: `scripts/seed_dispatch_demo.py`; smoke API: `scripts/smoke_dispatch.sh`.

## Backlog
- P1: notifikasi WA otomatis kode pickup ke pelanggan saat pengiriman pickup dibuat (sekarang tombol WA manual).
- P1: kadaluarsa kode pickup / regenerasi kode oleh admin; batas percobaan kode salah → kunci sementara.
- P1: tampilkan badge moda & kode pickup di Perjalanan Pesanan (OrderJourneyPanel) dan Meja Admin Gudang.
- P2: biaya kirim ekspedisi → jurnal beban pengiriman (GL) / tagihan ke pelanggan.
- P2: jadwal kendaraan (kalender), riwayat perawatan, KM.
- P2: peta posisi live untuk semua pengiriman aktif di dasbor.
