# SPESIFIKASI KEBUTUHAN API: SISTEM ABSENSI & PRESENSI PEGAWAI
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** Sistem Presensi Pegawai Hamzanwadi | **Kritisitas:** Tinggi (Kedisiplinan & SSO Matching)
> **Konsumen:** ETL Pipeline HARMONI & Modul Auth SSO
> **Base URL Contoh:** `https://absensi.hamzanwadi.ac.id`

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

Sistem Absensi menyediakan verifikasi subjek SSO untuk auto-provisioning pengguna serta data riwayat presensi harian (hadir, terlambat, izin, sakit, cuti, dinas, alpa).

### 5.1 Endpoint Verifikasi Karyawan Internal (Integrasi SSO)
- **Fungsi:** Memverifikasi identitas pengguna yang login via Keycloak SSO dan menautkannya ke profil pegawai HARMONI.
- **Tabel Tujuan:** `auth.users` dan `dw.dim_pegawai`
- **Method & Path:** `GET /api/v1.1/internal/employees` (Endpoint yang sudah aktif pada server absensi kampus)
- **Query Parameter:**
  - `name` (string, opsional): Nama atau username SSO.
  - `nip` (string, opsional).
  - `sso_id` (string, opsional).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data pegawai internal berhasil ditemukan",
  "data": [
    {
      "employee_id": "EMP-0812058501",
      "sso_id": "8f3b2e1a-4c5d-6e7f-8a9b-0c1d2e3f4a5b",
      "nip": "198505122010121002",
      "nidn": "0812058501",
      "name": "Dr. Irwan Hamzani, M.Pd.",
      "username": "irwan.hamzani",
      "email": "irwan.hamzani@hamzanwadi.ac.id",
      "unit_id": "FTIK",
      "unit_name": "Fakultas Teknik dan Ilmu Komputer",
      "position": "Dosen Tetap / Ketua Program Studi",
      "role_category": "DOSEN",
      "employment_status": "TETAP",
      "is_active": true
    }
  ]
}
```

---

### 5.2 Endpoint Rekap Kehadiran Harian / Bulanan
- **Fungsi:** Mengambil data log presensi harian untuk menghitung persentase kehadiran bulanan, menit keterlambatan, dan kepatuhan jam kerja (untuk KPI Persentase Kehadiran Tendik).
- **Tabel Tujuan DW:** `dw.fact_kehadiran`
- **Method & Path:** `GET /api/v1/internal/absensi/kehadiran`
- **Query Parameter:**
  - `tanggal_mulai` (string, wajib): Contoh `2025-09-01`.
  - `tanggal_selesai` (string, wajib): Contoh `2025-09-30`.
  - `kode_unit` (string, opsional).
  - `nip` (string, opsional).
  - `updated_since` (string, opsional).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data kehadiran pegawai berhasil diambil",
  "data": [
    {
      "id_kehadiran": "ABS-20250915-001",
      "nip": "199208142018012001",
      "nama": "Suhartini, S.Kom.",
      "kode_unit": "BAAK",
      "tanggal": "2025-09-15",
      "status": "HADIR",
      "jam_masuk": "07:55:12",
      "jam_pulang": "16:05:30",
      "menit_terlambat": 0,
      "keterangan": "Hadir tepat waktu",
      "lokasi_presensi": "Gedung Rektorat Lantai 1",
      "updated_at": "2026-09-15T16:10:00+08:00"
    },
    {
      "id_kehadiran": "ABS-20250915-002",
      "nip": "199208142018012001",
      "nama": "Suhartini, S.Kom.",
      "kode_unit": "BAAK",
      "tanggal": "2025-09-16",
      "status": "TERLAMBAT",
      "jam_masuk": "08:24:45",
      "jam_pulang": "16:00:10",
      "menit_terlambat": 24,
      "keterangan": "Terlambat 24 menit",
      "lokasi_presensi": "Gedung Rektorat Lantai 1",
      "updated_at": "2026-09-16T16:05:00+08:00"
    },
    {
      "id_kehadiran": "ABS-20250915-003",
      "nip": "198505122010121002",
      "nama": "Dr. Irwan Hamzani, M.Pd.",
      "kode_unit": "FTIK",
      "tanggal": "2025-09-15",
      "status": "DINAS",
      "jam_masuk": null,
      "jam_pulang": null,
      "menit_terlambat": 0,
      "keterangan": "Tugas Asesor Akreditasi LAMINFOKOM",
      "nomor_surat_tugas": "ST/UNHAM/IX/2025/44",
      "updated_at": "2026-09-15T18:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 3,
    "total_records": 12400,
    "total_pages": 4134,
    "watermark_latest": "2026-09-16T16:05:00+08:00"
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
