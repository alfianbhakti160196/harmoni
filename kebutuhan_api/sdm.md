# SPESIFIKASI KEBUTUHAN API: SISTEM INFORMASI KEPEGAWAIAN & SDM (SIMPEG)
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** SIMPEG Hamzanwadi | **Kritisitas:** Kritis (Master Data Dosen & Tenaga Kependidikan)
> **Konsumen:** ETL Pipeline HARMONI (Worker Go / Asynq) & Modul SSO Auto-Provisioning
> **Base URL Contoh:** `https://simpeg.hamzanwadi.ac.id`

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

SIMPEG menyediakan master data seluruh sumber daya manusia universitas (dosen tetap, dosen tidak tetap, dan tenaga kependidikan/karyawan).

### 4.1 Endpoint Data Pegawai (Dosen & Tendik)
- **Fungsi:** Mengambil data identitas kepegawaian, jabatan struktural, jabatan fungsional, dan unit penempatan.
- **Tabel Tujuan DW:** `dw.dim_pegawai`
- **Method & Path:** `GET /api/v1/internal/simpeg/pegawai`
- **Query Parameter:** `jenis_pegawai` (`DOSEN` / `TENDIK`), `status_aktif`, `kode_unit`, `updated_since`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data pegawai berhasil diambil",
  "data": [
    {
      "id_pegawai_simpeg": "PEG-0812058501",
      "nip": "198505122010121002",
      "nidn": "0812058501",
      "nidk": null,
      "nuptk": "4538763665200002",
      "nama": "Dr. Irwan Hamzani, M.Pd.",
      "gelar_depan": "Dr.",
      "gelar_belakang": "M.Pd.",
      "jenis_kelamin": "L",
      "email_institusi": "irwan.hamzani@hamzanwadi.ac.id",
      "jenis_pegawai": "DOSEN",
      "status_kepegawaian": "TETAP",
      "status_aktif": "AKTIF",
      "kode_unit_kerja": "FTIK",
      "kode_homebase_prodi": "PTI",
      "jabatan_struktural": "Ketua Program Studi",
      "jabatan_fungsional_kode": "L",
      "jabatan_fungsional_nama": "Lektor",
      "angka_kredit": 245.50,
      "pangkat_golongan": "Penata Tk. I / III/d",
      "pendidikan_terakhir": "S3",
      "bidang_keilmuan": "Pendidikan Ilmu Komputer",
      "is_serdos": true,
      "sinta_id": "6021894",
      "scholar_id": "aBcDeFgAAAAJ",
      "scopus_id": "57210984500",
      "tmt_kerja": "2010-12-01",
      "updated_at": "2026-10-20T14:30:00+08:00"
    },
    {
      "id_pegawai_simpeg": "PEG-1992081401",
      "nip": "199208142018012001",
      "nidn": null,
      "nidk": null,
      "nuptk": null,
      "nama": "Suhartini, S.Kom.",
      "gelar_depan": null,
      "gelar_belakang": "S.Kom.",
      "jenis_kelamin": "P",
      "email_institusi": "suhartini@hamzanwadi.ac.id",
      "jenis_pegawai": "TENDIK",
      "status_kepegawaian": "TETAP",
      "status_aktif": "AKTIF",
      "kode_unit_kerja": "BAAK",
      "kode_homebase_prodi": null,
      "jabatan_struktural": "Staf Administrasi Akademik",
      "jabatan_fungsional_kode": null,
      "jabatan_fungsional_nama": null,
      "angka_kredit": null,
      "pangkat_golongan": "Penata Muda / III/a",
      "pendidikan_terakhir": "S1",
      "bidang_keilmuan": "Sistem Informasi",
      "is_serdos": false,
      "sinta_id": null,
      "scholar_id": null,
      "scopus_id": null,
      "tmt_kerja": "2018-01-01",
      "updated_at": "2026-10-21T09:15:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 520,
    "total_pages": 260,
    "watermark_latest": "2026-10-21T09:15:00+08:00"
  }
}
```

---

### 4.2 Endpoint Riwayat Jabatan Fungsional Dosen
- **Fungsi:** Mengambil riwayat SK jabatan akademik untuk menghitung tren dan memetakan progres kenaikan jabatan berikutnya.
- **Tabel Tujuan DW:** `dw.riwayat_jabatan_fungsional`
- **Method & Path:** `GET /api/v1/internal/simpeg/riwayat-jabfung`
- **Query Parameter:** `nidn`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Riwayat jabatan fungsional berhasil diambil",
  "data": [
    {
      "nidn": "0812058501",
      "jabatan_fungsional_kode": "AA",
      "jabatan_fungsional_nama": "Asisten Ahli",
      "nomor_sk": "SK-JAB/2014/110",
      "tmt": "2014-04-01",
      "angka_kredit": 150.00
    },
    {
      "nidn": "0812058501",
      "jabatan_fungsional_kode": "L",
      "jabatan_fungsional_nama": "Lektor",
      "nomor_sk": "SK-JAB/2019/420",
      "tmt": "2019-10-01",
      "angka_kredit": 200.00
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
