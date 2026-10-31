# SPESIFIKASI KEBUTUHAN API: WEB SERVICE RESMI SISTER KEMDIKTISAINTEK
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** SISTER Cloud Kemdiktisaintek | **Kritisitas:** Kritis (Data Nasional Dosen & Portofolio BKD)
> **Konsumen:** ETL Pipeline HARMONI
> **Base URL Contoh:** `https://sister-api.kemdiktisaintek.go.id`

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

SISTER adalah platform resmi karier dosen nasional (Beban Kerja Dosen, Sertifikasi Dosen, Kenaikan Jabatan Fungsional, dan Rekam Jejak Tridarma).

### 10.1 Standar Header & Autentikasi SISTER
- **Protokol:** HTTPS REST API (`https://sister.hamzanwadi.ac.id/ws.php/1.0/`)
- **Headers:**
  ```http
  Authorization: Bearer <SISTER_BEARER_TOKEN>
  Accept: application/json
  ```

---

### 10.2 Endpoint Data Master Dosen (`GET /ws.php/1.0/dosen`)
- **Fungsi:** Mengambil profil dosen terverifikasi nasional, NIDN, NIP, status keaktifan, dan homebase kementerian.
- **Tabel Tujuan DW:** `dw.dim_pegawai`
- **Response JSON:**
  ```json
  {
    "status": "success",
    "data": [
      {
        "id_sdm": "6a5b4c3d-2e1f-0a9b-8c7d-6e5f4a3b2c1d",
        "nama_dosen": "Dr. Irwan Hamzani, M.Pd.",
        "nidn": "0812058501",
        "nip": "198505122010121002",
        "jenis_kelamin": "L",
        "status_keaktifan": "Aktif",
        "status_kepegawaian": "Dosen Tetap",
        "ikatan_kerja": "Dosen Tetap Yayasan",
        "kode_pt": "081042",
        "nama_pt": "Universitas Hamzanwadi",
        "id_prodi": "8b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
        "nama_prodi": "Pendidikan Teknologi Informasi",
        "jenjang_pendidikan_terakhir": "S3",
        "gelar_akademik": "Dr., M.Pd."
      }
    ]
  }
  ```

---

### 10.3 Endpoint Jabatan Fungsional (`GET /ws.php/1.0/jabatan_fungsional`)
- **Fungsi:** Mengambil riwayat jabatan fungsional (Lektor, Lektor Kepala, Guru Besar), angka kredit kumulatif, nomor SK, dan TMT.
- **Tabel Tujuan DW:** `dw.riwayat_jabatan_fungsional` dan `dw.dim_pegawai`
- **Response JSON:**
  ```json
  {
    "status": "success",
    "data": [
      {
        "id_sdm": "6a5b4c3d-2e1f-0a9b-8c7d-6e5f4a3b2c1d",
        "nidn": "0812058501",
        "jabatan_fungsional": "Lektor",
        "angka_kredit": 200.0,
        "nomor_sk": "SK-JAB/2019/420",
        "tanggal_sk": "2019-09-15",
        "tmt_jabatan": "2019-10-01",
        "kelebihan_angka_kredit": 45.50
      }
    ]
  }
  ```

---

### 10.4 Endpoint Sertifikasi Dosen (`GET /ws.php/1.0/sertifikasi_dosen`)
- **Fungsi:** Mengambil status lulus Serdos resmi (untuk KPI Persentase Dosen Tersertifikasi).
- **Tabel Tujuan DW:** `dw.dim_pegawai.is_serdos`
- **Response JSON:**
  ```json
  {
    "status": "success",
    "data": [
      {
        "id_sdm": "6a5b4c3d-2e1f-0a9b-8c7d-6e5f4a3b2c1d",
        "nidn": "0812058501",
        "jenis_sertifikasi": "Dosen Reguler",
        "bidang_studi": "Pendidikan Ilmu Komputer",
        "nomor_peserta": "160812058501",
        "nomor_sertifikat": "1610810420110",
        "tahun_sertifikasi": 2016,
        "status_sertifikasi": "LULUS"
      }
    ]
  }
  ```

---

### 10.5 Endpoint Penelitian & Pengabdian Nasional (`GET /ws.php/1.0/penelitian`)
- **Fungsi:** Mengambil rekam jejak hibah penelitian kementerian (BIMA / Simlitabmas) yang diakui secara nasional.
- **Tabel Tujuan DW:** `dw.fact_penelitian`
- **Response JSON:**
  ```json
  {
    "status": "success",
    "data": [
      {
        "id_penelitian": "PEN-SISTER-2025-091",
        "id_sdm_ketua": "6a5b4c3d-2e1f-0a9b-8c7d-6e5f4a3b2c1d",
        "nidn_ketua": "0812058501",
        "judul_penelitian": "Pengembangan Sistem Deteksi Dini Putus Studi Mahasiswa Berbasis Algoritma Stacking Ensemble",
        "tahun_usulan": 2024,
        "tahun_pelaksanaan": 2025,
        "skema_penelitian": "Penelitian Terapan Unggulan Perguruan Tinggi (PTUPT)",
        "sumber_dana": "Kemdiktisaintek (BIMA)",
        "dana_disetujui": 65000000.0,
        "status_penelitian": "Selesai"
      }
    ]
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
