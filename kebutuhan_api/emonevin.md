# SPESIFIKASI KEBUTUHAN API: SISTEM E-MONEVIN (PENJAMINAN MUTU & KINERJA)
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** Sistem E-MONEVIN / LPM Hamzanwadi | **Kritisitas:** Kritis (SPMI, AMI, & Akreditasi)
> **Konsumen:** ETL Pipeline HARMONI (Worker Go / Asynq)
> **Base URL Contoh:** `https://emonevin.hamzanwadi.ac.id`

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

E-MONEVIN dikelola oleh Badan Penjaminan Mutu (BPM) untuk memantau ketercapaian program kerja unit, indikator standar mutu (SPMI), temuan Audit Mutu Internal (AMI), serta status masa berlaku akreditasi.

### 7.1 Endpoint Program Kerja & Realisasi Kinerja Unit
- **Fungsi:** Mengambil data program kerja, target kinerja, jadwal, realisasi kegiatan, dan serapan anggaran (untuk dashboard Kepala Unit & Dekan).
- **Tabel Tujuan DW:** `dw.fact_program_kerja`
- **Method & Path:** `GET /api/v1/internal/emonevin/program-kerja`
- **Query Parameter:** `tahun` (integer, wajib), `kode_unit`, `status`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data program kerja unit berhasil diambil",
  "data": [
    {
      "id_program_emonevin": "PROG-FTIK-2025-01",
      "kode_unit": "FTIK",
      "nama_unit": "Fakultas Teknik dan Ilmu Komputer",
      "tahun": 2025,
      "kode_program": "P.01.2025",
      "nama_program": "Peningkatan Akreditasi Internasional Program Studi",
      "indikator": "Tersusunnya dokumen Self Assessment Report (SAR) standar IABEE",
      "target": "1 Dokumen SAR Lengkap",
      "pj_nip": "198505122010121002",
      "pj_nama": "Dr. Irwan Hamzani, M.Pd.",
      "tanggal_mulai": "2025-02-01",
      "tanggal_selesai": "2025-11-30",
      "status": "BERJALAN",
      "progres_persen": 75,
      "pagu_anggaran": 50000000.00,
      "realisasi_anggaran": 37500000.00,
      "updated_at": "2026-10-15T13:45:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 1,
    "total_records": 64,
    "total_pages": 64,
    "watermark_latest": "2026-10-15T13:45:00+08:00"
  }
}
```

---

### 7.2 Endpoint Capaian Standar Mutu SPMI
- **Fungsi:** Mengambil indikator standar mutu internal (Pendidikan, Penelitian, Pengabdian, Tata Kelola) beserta nilai target dan capaian faktualnya.
- **Tabel Tujuan DW:** `dw.fact_indikator_mutu`
- **Method & Path:** `GET /api/v1/internal/emonevin/indikator-mutu`
- **Query Parameter:** `tahun` (integer), `kode_unit`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data capaian indikator mutu SPMI berhasil diambil",
  "data": [
    {
      "kode_unit": "PTI",
      "tahun": 2025,
      "kode_standar": "STD-PEND-04",
      "nama_standar": "Persentase Kelulusan Tepat Waktu Mahasiswa S1",
      "target": 75.0,
      "capaian": 72.5,
      "satuan": "PERSEN",
      "status": "BELUM_TERCAPAI",
      "analisis_penyebab": "Kendala pada penyelesaian revisi tugas akhir mahasiswa di semester 8",
      "updated_at": "2026-10-10T10:00:00+08:00"
    },
    {
      "kode_unit": "PTI",
      "tahun": 2025,
      "kode_standar": "STD-SDM-02",
      "nama_standar": "Rasio Dosen Berpendidikan Doktor (S3)",
      "target": 35.0,
      "capaian": 41.67,
      "satuan": "PERSEN",
      "status": "TERCAPAI",
      "analisis_penyebab": "3 Dosen menyelesaikan studi S3 tepat waktu pada tahun berjalan",
      "updated_at": "2026-10-10T10:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 112,
    "total_pages": 56,
    "watermark_latest": "2026-10-10T10:00:00+08:00"
  }
}
```

---

### 7.3 Endpoint Temuan Audit Mutu Internal (AMI) & Tindak Lanjut
- **Fungsi:** Mengambil daftar temuan audit mutu, status rekomendasi korektif, dan tingkat kepatuhan tenggat penyelesaian.
- **Tabel Tujuan DW:** `dw.fact_temuan`
- **Method & Path:** `GET /api/v1/internal/emonevin/temuan`
- **Query Parameter:** `tahun`, `kode_unit`, `status` (`TERBUKA`, `PROSES`, `SELESAI`, `TERLAMBAT`).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data temuan audit mutu berhasil diambil",
  "data": [
    {
      "id_temuan_emonevin": "AMI-2025-FTIK-003",
      "kode_unit": "PTI",
      "sumber_temuan": "AMI",
      "tanggal_temuan": "2025-06-20",
      "uraian_temuan": "Sebanyak 15% mata kuliah yang aktif dibuka belum mengunggah dokumen Rencana Evaluasi Pembelajaran pada portal RPS",
      "kategori": "MINOR",
      "rekomendasi": "Kaprodi wajib melakukan verifikasi kelengkapan RPS sebelum perkuliahan minggu ke-3",
      "pj_nip": "198505122010121002",
      "pj_nama": "Dr. Irwan Hamzani, M.Pd.",
      "tenggat": "2025-09-30",
      "status": "SELESAI",
      "progres_persen": 100,
      "tanggal_penyelesaian": "2025-09-25",
      "updated_at": "2026-09-26T08:30:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 1,
    "total_records": 38,
    "total_pages": 38,
    "watermark_latest": "2026-09-26T08:30:00+08:00"
  }
}
```

---

### 7.4 Endpoint Status Akreditasi Institusi & Prodi
- **Fungsi:** Mengambil status akreditasi BAN-PT/LAM, peringkat, tanggal SK, dan masa kedaluwarsa untuk peringatan dini re-akreditasi.
- **Tabel Tujuan DW:** `dw.fact_akreditasi`
- **Method & Path:** `GET /api/v1/internal/emonevin/akreditasi`
- **Query Parameter:** `kode_unit` (opsional).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data status akreditasi berhasil diambil",
  "data": [
    {
      "kode_unit": "PTI",
      "nama_unit": "Pendidikan Teknologi Informasi",
      "jenis_unit": "PRODI",
      "lembaga": "LAMDIK",
      "peringkat": "BAIK_SEKALI",
      "nomor_sk": "412/SK/LAMDIK/Ak/S/IV/2023",
      "tanggal_sk": "2023-04-12",
      "tanggal_kadaluarsa": "2028-04-12",
      "is_berlaku": true,
      "sisa_masa_berlaku_hari": 529,
      "updated_at": "2026-01-10T09:00:00+08:00"
    },
    {
      "kode_unit": "UNHAM",
      "nama_unit": "Universitas Hamzanwadi",
      "jenis_unit": "UNIVERSITAS",
      "lembaga": "BAN-PT",
      "peringkat": "BAIK_SEKALI",
      "nomor_sk": "88/SK/BAN-PT/Ak/PT/II/2024",
      "tanggal_sk": "2024-02-20",
      "tanggal_kadaluarsa": "2029-02-20",
      "is_berlaku": true,
      "sisa_masa_berlaku_hari": 843,
      "updated_at": "2026-02-25T10:00:00+08:00"
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
