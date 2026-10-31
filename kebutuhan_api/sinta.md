# SPESIFIKASI KEBUTUHAN API: INTEGRASI DATA SINTA (KEMDIKTISAINTEK)
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** Science and Technology Index (SINTA v3) | **Kritisitas:** Tinggi (Pemeringkatan Riset Perguruan Tinggi)
> **Konsumen:** ETL Pipeline HARMONI
> **Base URL Contoh:** `https://api.sinta.kemdiktisaintek.go.id`

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

SINTA adalah portal indeksasi publikasi, sitasi, dan pemeringkatan peneliti di Indonesia yang dikelola oleh Kemdiktisaintek.

### 11.1 Endpoint Profil Author Dosen SINTA
- **Fungsi:** Mengambil skor SINTA 3 tahun, skor SINTA keseluruhan, jumlah dokumen terindeks Scopus/WOS/Garuda, dan h-index.
- **Tabel Tujuan DW:** `dw.dim_pegawai` (kolom metrik SINTA) dan `dw.fact_indeks_penulis`
- **Method & Path:** `GET /api/v1/sinta/authors/{sinta_id}`
- **Headers:** `X-API-KEY: <SINTA_PARTNER_KEY>`

#### Contoh Response JSON:
```json
{
  "status": "success",
  "data": {
    "sinta_id": "6021894",
    "nidn": "0812058501",
    "nama": "Dr. Irwan Hamzani, M.Pd.",
    "afiliasi": "Universitas Hamzanwadi",
    "kode_pt": "081042",
    "sinta_score_3yr": 284.50,
    "sinta_score_overall": 642.00,
    "scopus_h_index": 5,
    "google_scholar_h_index": 12,
    "wos_h_index": 2,
    "dokumen_scopus": 14,
    "dokumen_google_scholar": 48,
    "dokumen_garuda": 22,
    "sitasi_scopus": 86,
    "sitasi_google_scholar": 520,
    "updated_at": "2026-10-25T08:00:00+08:00"
  }
}
```

---

### 11.2 Endpoint Peringkat Afiliasi Universitas Hamzanwadi
- **Fungsi:** Mengambil metrik performa riset institusi secara keseluruhan untuk dashboard eksekutif Rektor.
- **Method & Path:** `GET /api/v1/sinta/affiliations/081042/overview`
- **Response JSON:**
  ```json
  {
    "status": "success",
    "data": {
      "kode_pt": "081042",
      "nama_pt": "Universitas Hamzanwadi",
      "sinta_score_3yr": 18450.00,
      "sinta_score_overall": 42100.00,
      "ranking_nasional": 128,
      "ranking_ptn_pts_wilayah": 4,
      "total_authors": 312,
      "total_departments": 24,
      "total_scopus_documents": 412,
      "total_scholar_documents": 2480,
      "total_garuda_documents": 1150
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
