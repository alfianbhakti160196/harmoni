# SPESIFIKASI KEBUTUHAN API: SISTEM INFORMASI KEUANGAN
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** Sistem Informasi Keuangan & Kas Kampus | **Kritisitas:** Kritis (Kesehatan Finansial & Serapan Anggaran)
> **Konsumen:** ETL Pipeline HARMONI (Worker Go / Asynq)
> **Base URL Contoh:** `https://keuangan.hamzanwadi.ac.id`

---

## 0. Standar Protokol & Format Response

Untuk menjamin keandalan dan konsistensi data yang masuk ke pipeline ETL HARMONI, seluruh API wajib mematuhi standar berikut:

### 0.1 Format Protokol & Autentikasi
- **Protokol:** HTTPS murni (TLS 1.2+).
- **Format Pertukaran Data:** JSON (`Content-Type: application/json; charset=utf-8`).
- **Autentikasi:** API Key atau Bearer Token via Header HTTP:
  ```http
  Authorization: Bearer <SECRET_INTERNAL_API_KEY>
  X-Client-Id: harmoni-etl
  ```
- **Zona Waktu:** Seluruh format waktu menggunakan ISO 8601 dengan offset WITA (`Asia/Makassar` / `+08:00`), contoh: `2026-10-30T10:00:00+08:00`. Format tanggal menggunakan `YYYY-MM-DD`.

### 0.2 Format Response Envelope Standar
Seluruh endpoint wajib membungkus payload respons dalam amplop (*envelope*) standar:

```json
{
  "status": "success",
  "message": "Data berhasil diambil",
  "data": [],
  "meta": {
    "page": 1,
    "page_size": 100,
    "total_records": 1250,
    "total_pages": 13,
    "watermark_latest": "2026-10-30T09:45:00+08:00"
  }
}
```

### 0.3 Mekanisme Incremental Sync (Watermarking) & Paginasi
Setiap endpoint data transaksi wajib mendukung dua parameter query berikut:
1. `updated_since`: String timestamp ISO 8601. Sistem sumber hanya mengembalikan record yang dibuat atau diubah pada atau setelah waktu tersebut (`updated_at >= updated_since`).
2. `page` (default: 1) dan `page_size` (default: 100, maksimal 500).

### 0.4 Format Response Error Standar
Jika terjadi kegagalan validasi, parameter tidak valid, atau kendala internal server:

```json
{
  "status": "error",
  "message": "Parameter updated_since tidak valid",
  "error_code": "INVALID_TIMESTAMP_FORMAT",
  "details": [
    {
      "field": "updated_since",
      "issue": "Format harus mengikuti ISO 8601 (YYYY-MM-DDTHH:mm:ss+08:00)"
    }
  ]
}
```

---

Sistem Keuangan menyediakan data pagu anggaran tahunan, realisasi serapan anggaran belanja operasional/program, serta pendapatan universitas.

### 3.1 Endpoint Realisasi Anggaran Belanja Bulanan
- **Fungsi:** Mengambil data pagu dan realisasi pengeluaran per unit kerja per bulan (untuk KPI Serapan Anggaran).
- **Tabel Tujuan DW:** `dw.fact_keuangan_bulanan`
- **Method & Path:** `GET /api/v1/internal/keuangan/realisasi-anggaran`
- **Query Parameter:** `tahun` (integer, wajib), `bulan` (integer 1-12), `kode_unit`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data realisasi anggaran berhasil diambil",
  "data": [
    {
      "kode_unit": "FTIK",
      "nama_unit": "Fakultas Teknik dan Ilmu Komputer",
      "tahun": 2025,
      "bulan": 9,
      "jenis": "BELANJA",
      "kategori": "OPERASIONAL_PENDIDIKAN",
      "kode_akun": "5.2.1.01",
      "nama_akun": "Belanja Bahan dan Praktikum Laboratorium",
      "kode_program_kerja": "PROG-FTIK-2025-03",
      "anggaran_pagu": 120000000.00,
      "realisasi_bulan_ini": 18500000.00,
      "realisasi_kumulatif_sd_bulan": 92500000.00,
      "sisa_anggaran": 27500000.00,
      "persen_serapan": 77.08,
      "updated_at": "2026-10-02T17:00:00+08:00"
    },
    {
      "kode_unit": "PTI",
      "nama_unit": "Prodi Pendidikan Teknologi Informasi",
      "tahun": 2025,
      "bulan": 9,
      "jenis": "BELANJA",
      "kategori": "PENGEMBANGAN_SDM",
      "kode_akun": "5.2.2.04",
      "nama_akun": "Pelatihan dan Sertifikasi Kompetensi Dosen",
      "kode_program_kerja": "PROG-PTI-2025-01",
      "anggaran_pagu": 45000000.00,
      "realisasi_bulan_ini": 15000000.00,
      "realisasi_kumulatif_sd_bulan": 35000000.00,
      "sisa_anggaran": 10000000.00,
      "persen_serapan": 77.78,
      "updated_at": "2026-10-02T17:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 180,
    "total_pages": 90,
    "watermark_latest": "2026-10-02T17:00:00+08:00"
  }
}
```

---

### 3.2 Endpoint Pendapatan Universitas
- **Fungsi:** Mengambil data arus penerimaan kas universitas berdasarkan kategori (UKT Mahasiswa, Hibah, Kerjasama, Unit Bisnis).
- **Tabel Tujuan DW:** `dw.fact_keuangan_bulanan`
- **Method & Path:** `GET /api/v1/internal/keuangan/pendapatan`
- **Query Parameter:** `tahun` (integer, wajib), `bulan` (integer).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data pendapatan universitas berhasil diambil",
  "data": [
    {
      "kode_unit": "UNHAM",
      "nama_unit": "Universitas Hamzanwadi",
      "tahun": 2025,
      "bulan": 9,
      "jenis": "PENDAPATAN",
      "kategori": "UKT_MAHASISWA",
      "kode_akun": "4.1.1.01",
      "nama_akun": "Penerimaan UKT Semester Ganjil 2025/2026",
      "target_pagu": 28000000000.00,
      "realisasi_bulan_ini": 8450000000.00,
      "realisasi_kumulatif_sd_bulan": 26150000000.00,
      "persen_capaian": 93.39,
      "updated_at": "2026-10-02T17:00:00+08:00"
    },
    {
      "kode_unit": "LPPM",
      "nama_unit": "Lembaga Penelitian dan Pengabdian",
      "tahun": 2025,
      "bulan": 9,
      "jenis": "PENDAPATAN",
      "kategori": "HIBAH_EKSTERNAL",
      "kode_akun": "4.2.1.02",
      "nama_akun": "Penerimaan Dana Hibah BIMA Kemdiktisaintek Tahap 1",
      "target_pagu": 1500000000.00,
      "realisasi_bulan_ini": 1050000000.00,
      "realisasi_kumulatif_sd_bulan": 1050000000.00,
      "persen_capaian": 70.00,
      "updated_at": "2026-10-02T17:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 48,
    "total_pages": 24,
    "watermark_latest": "2026-10-02T17:00:00+08:00"
  }
}
```

---

---

## Panduan Pengujian & Checklist Kesiapan Integrasi

Sebelum menghubungkan API ini ke pipeline ETL HARMONI di server produksi:

- [ ] **Endpoint Aktif:** Endpoint mengembalikan status HTTP `200 OK` dengan format JSON yang valid.
- [ ] **Autentikasi Terverifikasi:** Request tanpa header `Authorization: Bearer ...` mengembalikan HTTP `401 Unauthorized`.
- [ ] **Watermarking Berfungsi:** Parameter `updated_since` benar-benar memfilter record dengan kondisi `updated_at >= updated_since`.
- [ ] **Paginasi Akurat:** Nilai `page`, `page_size`, `total_records`, dan `total_pages` pada objek `meta` sesuai dengan jumlah data yang dikembalikan.
- [ ] **Data Sensitif Terlindungi:** Tidak mengekspos field password, hash credential, atau informasi perbankan rahasia.
- [ ] **Performa Query:** Waktu respons API untuk batch 100 record tidak melebihi 2 detik.
