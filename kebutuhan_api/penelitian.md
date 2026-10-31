# SPESIFIKASI KEBUTUHAN API: SISTEM PENELITIAN & PENGABDIAN (LPPM)
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** Sistem LPPM Hamzanwadi | **Kritisitas:** Kritis (Kinerja Riset, Publikasi, PKM, & HKI)
> **Konsumen:** ETL Pipeline HARMONI (Worker Go / Asynq)
> **Base URL Contoh:** `https://lppm.hamzanwadi.ac.id`

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

LPPM menyediakan data luaran tridarma dosen berupa penelitian, pengabdian kepada masyarakat (PKM), pendanaan hibah internal/eksternal, publikasi jurnal, dan perolehan HKI.

### 6.1 Endpoint Data Penelitian Dosen
- **Fungsi:** Mengambil catatan penelitian, pendanaan, anggota dosen, dan status luaran.
- **Tabel Tujuan DW:** `dw.fact_penelitian` & `dw.penelitian_anggota`
- **Method & Path:** `GET /api/v1/internal/lppm/penelitian`
- **Query Parameter:** `tahun` (integer), `sumber_dana`, `nidn_ketua`, `status`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data penelitian berhasil diambil",
  "data": [
    {
      "id_penelitian_lppm": "LIT-2025-0042",
      "judul": "Pengembangan Sistem Deteksi Dini Putus Studi Mahasiswa Berbasis Algoritma Stacking Ensemble",
      "tahun": 2025,
      "skema": "Penelitian Terapan Unggulan Perguruan Tinggi (PTUPT)",
      "sumber_dana": "DIKTI",
      "jumlah_dana": 65000000.00,
      "status": "SELESAI",
      "kode_unit_pelaksana": "PTI",
      "anggota": [
        {
          "nidn": "0812058501",
          "nama": "Dr. Irwan Hamzani, M.Pd.",
          "peran": "KETUA"
        },
        {
          "nidn": "0823048902",
          "nama": "Baiq Rina, M.Kom.",
          "peran": "ANGGOTA"
        }
      ],
      "luaran": [
        {
          "jenis": "JURNAL",
          "judul": "Early Dropout Detection in Higher Education Using Machine Learning",
          "nama_media": "International Journal of Education and Development",
          "status_luaran": "PUBLISHED"
        },
        {
          "jenis": "HKI",
          "judul": "Program Komputer Algoritma Stacking Ensemble Mahasiswa Berisiko",
          "nomor_pendaftaran": "EC00202511200",
          "status_luaran": "TERDAFTAR"
        }
      ],
      "updated_at": "2026-10-18T15:20:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 1,
    "total_records": 124,
    "total_pages": 124,
    "watermark_latest": "2026-10-18T15:20:00+08:00"
  }
}
```

---

### 6.2 Endpoint Data Pengabdian kepada Masyarakat (PKM)
- **Fungsi:** Mengambil kegiatan PKM dosen, lokasi mitra, pendanaan, dan anggota tim.
- **Tabel Tujuan DW:** `dw.fact_pengabdian` & `dw.pengabdian_anggota`
- **Method & Path:** `GET /api/v1/internal/lppm/pengabdian`
- **Query Parameter:** `tahun`, `sumber_dana`, `nidn`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data pengabdian masyarakat berhasil diambil",
  "data": [
    {
      "id_pengabdian_lppm": "PKM-2025-0018",
      "judul": "Pendampingan Digitalisasi Manajemen Sekolah dan Asesmen Berbasis Komputer di SMP Sasak Ranah Lombok",
      "tahun": 2025,
      "skema": "Program Kemitraan Masyarakat (PKM)",
      "sumber_dana": "INTERNAL",
      "jumlah_dana": 15000000.00,
      "lokasi": "Kecamatan Masbagik, Kabupaten Lombok Timur",
      "mitra": "SMP Sasak Ranah",
      "status": "SELESAI",
      "kode_unit_pelaksana": "PTI",
      "anggota": [
        {
          "nidn": "0812058501",
          "nama": "Dr. Irwan Hamzani, M.Pd.",
          "peran": "KETUA"
        }
      ],
      "luaran": [
        {
          "jenis": "JURNAL",
          "judul": "Pemberdayaan Guru Melalui Pelatihan CBT Digital",
          "nama_media": "Jurnal Pengabdian Masyarakat Hamzanwadi",
          "status_luaran": "PUBLISHED"
        }
      ],
      "updated_at": "2026-09-30T11:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 1,
    "total_records": 98,
    "total_pages": 98,
    "watermark_latest": "2026-09-30T11:00:00+08:00"
  }
}
```

---

### 6.3 Endpoint Publikasi Ilmiah & Indeksasi
- **Fungsi:** Mengambil publikasi karya ilmiah dosen yang tercatat di repositori/LPPM untuk disinkronkan dengan data SINTA, Google Scholar, dan Scopus.
- **Tabel Tujuan DW:** `dw.fact_publikasi` & `dw.publikasi_penulis`
- **Method & Path:** `GET /api/v1/internal/lppm/publikasi`
- **Query Parameter:** `tahun`, `indeks` (`SCOPUS`, `SINTA1`, dll.), `nidn`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data publikasi ilmiah berhasil diambil",
  "data": [
    {
      "id_publikasi_lppm": "PUB-2025-0112",
      "judul": "Machine Learning Predictive Analytics for Student Retention in Indonesian Islamic Higher Education",
      "tahun": 2025,
      "jenis": "JURNAL",
      "nama_media": "IEEE Access",
      "issn_isbn": "2169-3536",
      "doi": "10.1109/ACCESS.2025.3214567",
      "url": "https://doi.org/10.1109/ACCESS.2025.3214567",
      "indeks": "SCOPUS",
      "kuartil": "Q1",
      "jumlah_sitasi": 8,
      "penulis": [
        {
          "nidn": "0812058501",
          "nama": "Dr. Irwan Hamzani, M.Pd.",
          "urutan": 1,
          "is_korespondensi": true
        }
      ],
      "updated_at": "2026-10-10T14:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 1,
    "total_records": 310,
    "total_pages": 310,
    "watermark_latest": "2026-10-10T14:00:00+08:00"
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
