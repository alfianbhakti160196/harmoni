# SPESIFIKASI KEBUTUHAN API: INTEGRASI AKREDITASI BAN-PT & LAM
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** Portal SAPTO BAN-PT & LAM (LAMDIK, LAM-PTKes, LAM-INFOKOM) | **Kritisitas:** Kritis (Status Legalitas Prodi)
> **Konsumen:** ETL Pipeline HARMONI
> **Mekanisme:** RESTful API & Scheduled Scraping Worker

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

Data status akreditasi program studi dan perguruan tinggi yang ditarik dari web service/portal pemantauan BAN-PT, LAMDIK, LAMINFOKOM, dan LAMEMBA.

### 13.1 Endpoint Status Akreditasi Institusi & Seluruh Prodi
- **Method & Path:** `GET /api/v1/external/akreditasi/status-pt`
- **Query Parameter:** `kode_pt=081042`
- **Tabel Tujuan DW:** `dw.fact_akreditasi`

#### Contoh Response JSON:
```json
{
  "status": "success",
  "data": {
    "institusi": {
      "kode_pt": "081042",
      "nama_pt": "Universitas Hamzanwadi",
      "lembaga": "BAN-PT",
      "peringkat": "BAIK_SEKALI",
      "skor_akreditasi": 348.50,
      "nomor_sk": "88/SK/BAN-PT/Ak/PT/II/2024",
      "tanggal_sk": "2024-02-20",
      "tanggal_kadaluarsa": "2029-02-20",
      "is_berlaku": true
    },
    "program_studi": [
      {
        "kode_prodi": "PTI",
        "nama_prodi": "Pendidikan Teknologi Informasi",
        "jenjang": "S1",
        "lembaga": "LAMDIK",
        "peringkat": "BAIK_SEKALI",
        "skor_akreditasi": 352.00,
        "nomor_sk": "412/SK/LAMDIK/Ak/S/IV/2023",
        "tanggal_sk": "2023-04-12",
        "tanggal_kadaluarsa": "2028-04-12",
        "is_berlaku": true
      },
      {
        "kode_prodi": "IF",
        "nama_prodi": "Teknik Informatika",
        "jenjang": "S1",
        "lembaga": "LAMINFOKOM",
        "peringkat": "UNGGUL",
        "skor_akreditasi": 368.00,
        "nomor_sk": "105/SK/LAM-INFOKOM/Ak/S/VIII/2024",
        "tanggal_sk": "2024-08-28",
        "tanggal_kadaluarsa": "2029-08-28",
        "is_berlaku": true
      }
    ]
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
