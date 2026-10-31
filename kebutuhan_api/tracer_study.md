# SPESIFIKASI KEBUTUHAN API: SISTEM ALUMNI & TRACER STUDY
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** Portal Tracer Study Hamzanwadi | **Kritisitas:** Tinggi (IKU 1 & Evaluasi Lulusan)
> **Konsumen:** ETL Pipeline HARMONI (Worker Go / Asynq)
> **Base URL Contoh:** `https://tracer.hamzanwadi.ac.id`

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

Sistem Tracer Study mengumpulkan data umpan balik lulusan mengenai masa tunggu mendapatkan pekerjaan, relevansi kurikulum dengan dunia kerja, pendapatan pertama, serta sebaran industri tempat alumni berkarier.

### 8.1 Endpoint Data Alumni Terdaftar
- **Fungsi:** Mengambil master data alumni yang telah menyelesaikan yudisium dan status kontak aktif.
- **Method & Path:** `GET /api/v1/internal/tracer/alumni`
- **Query Parameter:** `tahun_lulus` (integer, wajib), `kode_prodi`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data alumni berhasil diambil",
  "data": [
    {
      "nim": "2010510005",
      "nama": "Muhammad Zaini",
      "kode_prodi": "PTI",
      "tahun_lulus": 2024,
      "nomor_ijazah": "12345/UNHAM/S1/2024",
      "nomor_hp": "081917xxxxxx",
      "email_aktif": "zaini.alumni@gmail.com",
      "status_pengisian_survei": "SUDAH_MENGISI",
      "updated_at": "2026-06-15T11:20:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 1,
    "total_records": 1350,
    "total_pages": 1350,
    "watermark_latest": "2026-06-15T11:20:00+08:00"
  }
}
```

---

### 8.2 Endpoint Hasil Kuesioner Tracer Study
- **Fungsi:** Mengambil data hasil survei alumni (untuk KPI Waktu Tunggu Lulusan dan Relevansi Bidang Kerja).
- **Tabel Tujuan DW:** `dw.fact_tracer_study`
- **Method & Path:** `GET /api/v1/internal/tracer/survei-hasil`
- **Query Parameter:** `tahun_lulus` (integer), `tahun_survei` (integer, wajib).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data survei tracer study berhasil diambil",
  "data": [
    {
      "id_respon_survei": "TRC-2025-00124",
      "nim": "2010510005",
      "kode_prodi": "PTI",
      "tahun_lulus": 2024,
      "tahun_survei": 2025,
      "tanggal_pengisian": "2025-06-14",
      "status_aktivitas": "BEKERJA",
      "masa_tunggu_bulan": 3,
      "nama_tempat_kerja": "PT Telkom Indonesia (Witel NTB)",
      "posisi_jabatan": "IT Support & Network Administrator",
      "tingkat_tempat_kerja": "NASIONAL",
      "kesesuaian_bidang": "SANGAT_SESUAI",
      "rentang_pendapatan": "RP_4_JUTA_SD_7_JUTA",
      "updated_at": "2026-06-14T14:10:00+08:00"
    },
    {
      "id_respon_survei": "TRC-2025-00125",
      "nim": "2010510022",
      "kode_prodi": "PTI",
      "tahun_lulus": 2024,
      "tahun_survei": 2025,
      "tanggal_pengisian": "2025-06-15",
      "status_aktivitas": "WIRAUSAHA",
      "masa_tunggu_bulan": 2,
      "nama_tempat_kerja": "Lombok Tech Studio",
      "posisi_jabatan": "Founder / Web Developer",
      "tingkat_tempat_kerja": "LOKAL",
      "kesesuaian_bidang": "SESUAI",
      "rentang_pendapatan": "RP_4_JUTA_SD_7_JUTA",
      "updated_at": "2026-06-15T09:30:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 920,
    "total_pages": 460,
    "watermark_latest": "2026-06-15T09:30:00+08:00"
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
