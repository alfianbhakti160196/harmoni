# SPESIFIKASI KEBUTUHAN API: SISTEM PENERIMAAN MAHASISWA BARU (PMB)
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** Sistem PMB Online | **Kritisitas:** Tinggi (Faktor Utama Analisis Tren Mahasiswa)
> **Konsumen:** ETL Pipeline HARMONI (Worker Go / Asynq)
> **Base URL Contoh:** `https://pmb.hamzanwadi.ac.id`

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

Sistem PMB menyediakan data corong konversi (*funnel*) calon mahasiswa dari tahap pendaftaran hingga registrasi ulang, serta sebaran asal sekolah dan geografis.

### 2.1 Endpoint Data Pendaftar PMB
- **Fungsi:** Mengambil daftar transaksi peserta PMB, pilihan program studi, hasil seleksi, dan status pembayaran registrasi.
- **Tabel Tujuan DW:** `dw.fact_pmb`
- **Method & Path:** `GET /api/v1/internal/pmb/pendaftar`
- **Query Parameter:** `tahun_pmb` (integer, wajib), `gelombang`, `jalur`, `status`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data pendaftar PMB berhasil diambil",
  "data": [
    {
      "no_pendaftaran": "PMB2025001201",
      "tahun_pmb": 2025,
      "gelombang": "Gelombang 1",
      "jalur": "PRESTASI_UNGGULAN",
      "kode_prodi_pilihan1": "PTI",
      "kode_prodi_pilihan2": "SI",
      "kode_prodi_diterima": "PTI",
      "tanggal_daftar": "2025-02-10",
      "status": "REGISTRASI",
      "tanggal_lulus_seleksi": "2025-03-01",
      "tanggal_registrasi": "2025-03-10",
      "nominal_registrasi": 4500000.00,
      "asal_sekolah": "SMKN 1 Selong",
      "jenis_sekolah": "SMK",
      "provinsi_asal": "Nusa Tenggara Barat",
      "kabupaten_asal": "Lombok Timur",
      "updated_at": "2026-03-15T08:30:00+08:00"
    },
    {
      "no_pendaftaran": "PMB2025001202",
      "tahun_pmb": 2025,
      "gelombang": "Gelombang 1",
      "jalur": "REGULER_TES",
      "kode_prodi_pilihan1": "PGSD",
      "kode_prodi_pilihan2": "PAUD",
      "kode_prodi_diterima": "PGSD",
      "tanggal_daftar": "2025-02-12",
      "status": "LULUS_SELEKSI",
      "tanggal_lulus_seleksi": "2025-03-01",
      "tanggal_registrasi": null,
      "nominal_registrasi": 0,
      "asal_sekolah": "MAN 1 Lombok Timur",
      "jenis_sekolah": "MA",
      "provinsi_asal": "Nusa Tenggara Barat",
      "kabupaten_asal": "Lombok Timur",
      "updated_at": "2026-03-05T09:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 3450,
    "total_pages": 1725,
    "watermark_latest": "2026-03-15T08:30:00+08:00"
  }
}
```

---

### 2.2 Endpoint Statistik Corong PMB (Funnel Aggregates)
- **Fungsi:** Menyediakan statistik agregat siap pakai untuk dashboard eksekutif Rektor dan Kaprodi.
- **Method & Path:** `GET /api/v1/internal/pmb/statistik`
- **Query Parameter:** `tahun_pmb` (integer).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Statistik PMB berhasil diambil",
  "data": {
    "tahun_pmb": 2025,
    "total_pendaftar": 3450,
    "total_berkas_lengkap": 3200,
    "total_ikut_seleksi": 3150,
    "total_lulus_seleksi": 2800,
    "total_registrasi": 2350,
    "persen_konversi": 83.93,
    "sebaran_prodi": [
      {
        "kode_prodi": "PTI",
        "nama_prodi": "Pendidikan Teknologi Informasi",
        "pendaftar": 380,
        "lulus": 320,
        "registrasi": 285,
        "persen_konversi": 89.06
      },
      {
        "kode_prodi": "PGSD",
        "nama_prodi": "Pendidikan Guru Sekolah Dasar",
        "pendaftar": 620,
        "lulus": 510,
        "registrasi": 460,
        "persen_konversi": 90.20
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
