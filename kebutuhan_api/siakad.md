# SPESIFIKASI KEBUTUHAN API: SISTEM INFORMASI AKADEMIK (SIAKAD / IOSYS)
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** SIAKAD Kampus (Berbasis IOSYS) | **Kritisitas:** Kritis (Single Source of Truth Akademik & SDM Pembelajaran)
> **Konsumen:** ETL Pipeline HARMONI (Worker Go / Asynq)
> **Base URL Contoh:** `https://siakad.hamzanwadi.ac.id`

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

SIAKAD adalah sumber kebenaran (*single source of truth*) utama untuk data civitas akademika, perkuliahan, mata kuliah, dan kelulusan.

### 1.1 Endpoint Data Mahasiswa
- **Fungsi:** Mengambil master profil mahasiswa, status terkini, program studi, angkatan, dan IPK kumulatif.
- **Tabel Tujuan DW:** `dw.dim_mahasiswa`
- **Method & Path:** `GET /api/v1/internal/siakad/mahasiswa`
- **Query Parameter:**
  - `prodi_kode` (string, opsional): Kode prodi, misal `PTI`, `SI`.
  - `angkatan` (integer, opsional): Contoh `2024`.
  - `status` (string, opsional): `A` (Aktif), `C` (Cuti), `N` (Non-aktif), `L` (Lulus), `K` (Keluar), `D` (DO).
  - `updated_since` (string, opsional): ISO timestamp.
  - `page`, `page_size` (integer, opsional).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data mahasiswa berhasil diambil",
  "data": [
    {
      "id_mahasiswa_siakad": "MHS-2022-PTI-0042",
      "nim": "2210510042",
      "id_registrasi_feeder": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
      "nama": "Ahmad Fauzi",
      "jenis_kelamin": "L",
      "kode_prodi": "PTI",
      "nama_prodi": "Pendidikan Teknologi Informasi",
      "jenjang": "S1",
      "angkatan": 2022,
      "semester_masuk": "20221",
      "jalur_masuk": "SNMPTN / PRESTASI",
      "jenis_pendaftaran": "BARU",
      "provinsi_asal": "Nusa Tenggara Barat",
      "kabupaten_asal": "Lombok Timur",
      "status_terakhir": "A",
      "ipk_terakhir": 3.65,
      "sks_total": 92,
      "dosen_pa_nidn": "0812058501",
      "dosen_pa_nama": "Dr. Irwan Hamzani, M.Pd.",
      "updated_at": "2026-10-28T14:20:10+08:00"
    },
    {
      "id_mahasiswa_siakad": "MHS-2022-PTI-0043",
      "nim": "2210510043",
      "id_registrasi_feeder": "7c2adb1a-8c4d-4e9a-9e1b-3c4d5e6f7a8b",
      "nama": "Siti Nurhaliza",
      "jenis_kelamin": "P",
      "kode_prodi": "PTI",
      "nama_prodi": "Pendidikan Teknologi Informasi",
      "jenjang": "S1",
      "angkatan": 2022,
      "semester_masuk": "20221",
      "jalur_masuk": "MANDIRI",
      "jenis_pendaftaran": "BARU",
      "provinsi_asal": "Nusa Tenggara Barat",
      "kabupaten_asal": "Lombok Tengah",
      "status_terakhir": "A",
      "ipk_terakhir": 3.82,
      "sks_total": 94,
      "dosen_pa_nidn": "0812058501",
      "dosen_pa_nama": "Dr. Irwan Hamzani, M.Pd.",
      "updated_at": "2026-10-29T10:15:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 8450,
    "total_pages": 4225,
    "watermark_latest": "2026-10-29T10:15:00+08:00"
  }
}
```

---

### 1.2 Endpoint Aktivitas Kuliah Mahasiswa (AKM Semesteran)
- **Fungsi:** Mengambil status registrasi semester, IPS, IPK, SKS diambil, dan SKS kumulatif per mahasiswa per semester (input utama KPI Mahasiswa Berisiko dan Drop-out).
- **Tabel Tujuan DW:** `dw.fact_mahasiswa_semester`
- **Method & Path:** `GET /api/v1/internal/siakad/aktivitas-kuliah`
- **Query Parameter:**
  - `semester_kode` (string, wajib): Contoh `20251`, `20252`.
  - `kode_prodi` (string, opsional).
  - `updated_since` (string, opsional).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data aktivitas kuliah mahasiswa berhasil diambil",
  "data": [
    {
      "nim": "2210510042",
      "semester_kode": "20251",
      "kode_prodi": "PTI",
      "status_semester": "A",
      "semester_ke": 7,
      "ips": 3.75,
      "ipk": 3.65,
      "sks_semester": 20,
      "sks_total": 92,
      "biaya_kuliah_ukt": 3500000.00,
      "status_bayar": "LUNAS",
      "tanggal_krs": "2025-09-05",
      "updated_at": "2026-10-15T11:00:00+08:00"
    },
    {
      "nim": "2110510010",
      "semester_kode": "20251",
      "kode_prodi": "PTI",
      "status_semester": "A",
      "semester_ke": 9,
      "ips": 1.75,
      "ipk": 2.45,
      "sks_semester": 6,
      "sks_total": 118,
      "biaya_kuliah_ukt": 3500000.00,
      "status_bayar": "LUNAS",
      "tanggal_krs": "2025-09-08",
      "updated_at": "2026-10-15T11:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 7820,
    "total_pages": 3910,
    "watermark_latest": "2026-10-15T11:00:00+08:00"
  }
}
```

---

### 1.3 Endpoint Kurikulum & Mata Kuliah
- **Fungsi:** Mengambil daftar mata kuliah, bobot SKS, kurikulum, dan ketersediaan RPS (untuk KPI Ketersediaan RPS).
- **Tabel Tujuan DW:** `dw.dim_mata_kuliah`
- **Method & Path:** `GET /api/v1/internal/siakad/mata-kuliah`
- **Query Parameter:** `kode_prodi`, `kurikulum_tahun`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data mata kuliah berhasil diambil",
  "data": [
    {
      "kode_prodi": "PTI",
      "kode_mk": "PTI3201",
      "nama_mk": "Kecerdasan Buatan dan Pembelajaran Mesin",
      "sks": 3,
      "kurikulum": "Kurikulum MBKM 2024",
      "semester_paket": 5,
      "is_wajib": true,
      "punya_rps": true,
      "url_rps": "https://siakad.hamzanwadi.ac.id/dokumen/rps/PTI3201.pdf",
      "updated_at": "2026-08-10T09:00:00+08:00"
    },
    {
      "kode_prodi": "PTI",
      "kode_mk": "PTI3202",
      "nama_mk": "Pengembangan Media Interaktif",
      "sks": 3,
      "kurikulum": "Kurikulum MBKM 2024",
      "semester_paket": 5,
      "is_wajib": true,
      "punya_rps": false,
      "url_rps": null,
      "updated_at": "2026-08-10T09:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 480,
    "total_pages": 240,
    "watermark_latest": "2026-08-10T09:00:00+08:00"
  }
}
```

---

### 1.4 Endpoint Kelas Kuliah & Dosen Pengajar
- **Fungsi:** Mengambil data kelas dibuka, distribusi nilai kelulusan MK, dan beban mengajar dosen (team teaching & beban SKS).
- **Tabel Tujuan DW:** `dw.fact_kelas_kuliah` & `dw.fact_pengajaran`
- **Method & Path:** `GET /api/v1/internal/siakad/kelas-kuliah`
- **Query Parameter:** `semester_kode` (wajib), `kode_prodi`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data kelas kuliah berhasil diambil",
  "data": [
    {
      "id_kelas_siakad": "KLS-20251-PTI3201-A",
      "semester_kode": "20251",
      "kode_prodi": "PTI",
      "kode_mk": "PTI3201",
      "nama_kelas": "A",
      "jumlah_peserta": 38,
      "jumlah_lulus": 35,
      "rata_nilai_angka": 81.45,
      "distribusi_nilai": {
        "A": 15,
        "B": 14,
        "C": 6,
        "D": 2,
        "E": 1
      },
      "pengajar": [
        {
          "nidn": "0812058501",
          "nama": "Dr. Irwan Hamzani, M.Pd.",
          "sks_beban": 1.5,
          "rencana_pertemuan": 16,
          "realisasi_pertemuan": 16
        },
        {
          "nidn": "0823048902",
          "nama": "Baiq Rina, M.Kom.",
          "sks_beban": 1.5,
          "rencana_pertemuan": 16,
          "realisasi_pertemuan": 15
        }
      ],
      "updated_at": "2026-10-25T16:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 1,
    "total_records": 1150,
    "total_pages": 1150,
    "watermark_latest": "2026-10-25T16:00:00+08:00"
  }
}
```

---

### 1.5 Endpoint Data Kelulusan
- **Fungsi:** Mengambil data kelulusan wisuda, masa studi, IPK kelulusan, dan status tepat waktu.
- **Tabel Tujuan DW:** `dw.fact_kelulusan`
- **Method & Path:** `GET /api/v1/internal/siakad/kelulusan`
- **Query Parameter:** `tahun_lulus`, `kode_prodi`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data kelulusan mahasiswa berhasil diambil",
  "data": [
    {
      "nim": "2010510005",
      "nama": "Muhammad Zaini",
      "kode_prodi": "PTI",
      "semester_lulus": "20242",
      "tahun_lulus": 2024,
      "tanggal_lulus": "2024-08-15",
      "masa_studi_semester": 8,
      "masa_studi_bulan": 48,
      "ipk": 3.78,
      "predikat": "Dengan Pujian (Cumlaude)",
      "is_tepat_waktu": true,
      "nomor_ijazah": "12345/UNHAM/S1/2024",
      "nomor_sk_yudisium": "SK-YUD/2024/08/112",
      "updated_at": "2026-08-20T10:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 1,
    "total_records": 1420,
    "total_pages": 1420,
    "watermark_latest": "2026-08-20T10:00:00+08:00"
  }
}
```

---

### 1.6 Endpoint Ringkasan Eksekutif & Statistik Utama Akademik
- **Fungsi:** Mengambil indikator agregat level universitas untuk dashboard eksekutif Rektor dan Wakil Rektor Akademik, meliputi:
  - **Total Mahasiswa Aktif** (berdasarkan status registrasi semester berjalan / KRS aktif)
  - **Rata-rata IPK Universitas** (IPK kumulatif seluruh mahasiswa aktif di tingkat universitas)
  - **Jumlah Program Studi Aktif** (jenjang D3, S1, S2, Profesi)
  - **Beban Dosen / Rasio** (Rasio perbandingan dosen tetap terhadap mahasiswa aktif tingkat universitas, rata-rata beban SKS mengajar dosen, dan status kepatuhan standar rasio)
- **Tabel Tujuan DW / Data Mart:** `dw.fact_mahasiswa_semester`, `dw.fact_pengajaran`, `mart.dm_akademik_statistik_prodi`
- **Method & Path:** `GET /api/v1/internal/siakad/ringkasan-akademik`
- **Query Parameter:**
  - `semester_kode` (string, opsional): Kode semester akademik, contoh `20251`, `20252` (default: semester aktif berjalan).
  - `tahun_akademik` (integer, opsional): Tahun akademik, contoh `2025`.
  - `jenjang` (string, opsional): Filter jenjang tertentu, contoh `S1`, `S2`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data ringkasan eksekutif akademik berhasil diambil dari SIAKAD (IOSYS)",
  "data": {
    "semester_kode": "20251",
    "nama_semester": "Semester Ganjil 2025/2026",
    "tahun_akademik": 2025,
    "total_mahasiswa_aktif": 8432,
    "total_mahasiswa_baru": 2150,
    "rata_rata_ipk_universitas": 3.38,
    "jumlah_prodi": 32,
    "beban_dosen": {
      "total_dosen_tetap_mengajar": 312,
      "rasio_dosen_mahasiswa": "1 : 22",
      "rasio_angka": 22.05,
      "standar_rasio_nasional": "1 : 30",
      "status_rasio": "IDEAL",
      "rata_rata_sks_mengajar": 11.45,
      "total_kelas_aktif": 1150
    },
    "ringkasan_per_jenjang": [
      {
        "jenjang": "S1",
        "jumlah_prodi": 28,
        "mahasiswa_aktif": 7890,
        "rata_rata_ipk": 3.37,
        "rasio_dosen_mhs": "1 : 22.5"
      },
      {
        "jenjang": "S2",
        "jumlah_prodi": 3,
        "mahasiswa_aktif": 420,
        "rata_rata_ipk": 3.55,
        "rasio_dosen_mhs": "1 : 12.0"
      },
      {
        "jenjang": "PROFESI",
        "jumlah_prodi": 1,
        "mahasiswa_aktif": 122,
        "rata_rata_ipk": 3.62,
        "rasio_dosen_mhs": "1 : 10.5"
      }
    ],
    "updated_at": "2026-10-31T08:00:00+08:00"
  },
  "meta": {
    "source_system": "SIAKAD_IOSYS",
    "generated_at": "2026-10-31T08:00:00+08:00"
  }
}
```

---

### 1.7 Endpoint Proporsi & Sebaran Mahasiswa per Fakultas
- **Fungsi:** Mengambil data proporsi dan sebaran kuantitas mahasiswa aktif per fakultas (persentase terhadap total universitas, rasio gender L/P, jumlah mahasiswa baru, rata-rata IPK fakultas, serta rincian prodi di bawah naungannya) untuk visualisasi diagram donat (*doughnut chart*) dan grafik perbandingan kinerja antar-fakultas pada dashboard pimpinan.
- **Tabel Tujuan DW / Data Mart:** `ref.unit`, `dw.dim_mahasiswa`, `dw.fact_mahasiswa_semester`, `mart.dm_akademik_statistik_prodi`
- **Method & Path:** `GET /api/v1/internal/siakad/sebaran-fakultas`
- **Query Parameter:**
  - `semester_kode` (string, opsional): Kode semester, contoh `20251`.
  - `status` (string, opsional): Filter status mahasiswa, default `A` (Aktif).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data proporsi dan sebaran mahasiswa per fakultas berhasil diambil dari SIAKAD (IOSYS)",
  "data": {
    "semester_kode": "20251",
    "total_mahasiswa_universitas": 8432,
    "fakultas": [
      {
        "id_fakultas_siakad": "FAK-01",
        "kode_fakultas": "FIP",
        "nama_fakultas": "Fakultas Ilmu Pendidikan",
        "jumlah_prodi": 8,
        "total_mahasiswa": 3120,
        "persentase_proporsi": 37.00,
        "gender": {
          "laki_laki": 1120,
          "perempuan": 2000
        },
        "mahasiswa_baru": 780,
        "rata_rata_ipk": 3.42,
        "sebaran_prodi": [
          {
            "kode_prodi": "PGSD",
            "nama_prodi": "Pendidikan Guru Sekolah Dasar",
            "jenjang": "S1",
            "mahasiswa_aktif": 1250,
            "proporsi_fakultas": 40.06
          },
          {
            "kode_prodi": "PTI",
            "nama_prodi": "Pendidikan Teknologi Informasi",
            "jenjang": "S1",
            "mahasiswa_aktif": 680,
            "proporsi_fakultas": 21.79
          }
        ]
      },
      {
        "id_fakultas_siakad": "FAK-02",
        "kode_fakultas": "FMIPA",
        "nama_fakultas": "Fakultas Matematika dan IPA",
        "jumlah_prodi": 5,
        "total_mahasiswa": 1650,
        "persentase_proporsi": 19.57,
        "gender": {
          "laki_laki": 620,
          "perempuan": 1030
        },
        "mahasiswa_baru": 410,
        "rata_rata_ipk": 3.35,
        "sebaran_prodi": [
          {
            "kode_prodi": "PEND-MAT",
            "nama_prodi": "Pendidikan Matematika",
            "jenjang": "S1",
            "mahasiswa_aktif": 480,
            "proporsi_fakultas": 29.09
          }
        ]
      },
      {
        "id_fakultas_siakad": "FAK-03",
        "kode_fakultas": "FT",
        "nama_fakultas": "Fakultas Teknik",
        "jumlah_prodi": 6,
        "total_mahasiswa": 1820,
        "persentase_proporsi": 21.58,
        "gender": {
          "laki_laki": 1250,
          "perempuan": 570
        },
        "mahasiswa_baru": 460,
        "rata_rata_ipk": 3.31,
        "sebaran_prodi": [
          {
            "kode_prodi": "IF",
            "nama_prodi": "Teknik Informatika",
            "jenjang": "S1",
            "mahasiswa_aktif": 720,
            "proporsi_fakultas": 39.56
          },
          {
            "kode_prodi": "SI",
            "nama_prodi": "Sistem Informasi",
            "jenjang": "S1",
            "mahasiswa_aktif": 510,
            "proporsi_fakultas": 28.02
          }
        ]
      },
      {
        "id_fakultas_siakad": "FAK-04",
        "kode_fakultas": "FEB",
        "nama_fakultas": "Fakultas Ekonomi dan Bisnis",
        "jumlah_prodi": 4,
        "total_mahasiswa": 1142,
        "persentase_proporsi": 13.54,
        "gender": {
          "laki_laki": 430,
          "perempuan": 712
        },
        "mahasiswa_baru": 310,
        "rata_rata_ipk": 3.40,
        "sebaran_prodi": [
          {
            "kode_prodi": "MANAJEMEN",
            "nama_prodi": "Manajemen",
            "jenjang": "S1",
            "mahasiswa_aktif": 620,
            "proporsi_fakultas": 54.29
          }
        ]
      },
      {
        "id_fakultas_siakad": "FAK-05",
        "kode_fakultas": "FIKES",
        "nama_fakultas": "Fakultas Ilmu Kesehatan",
        "jumlah_prodi": 4,
        "total_mahasiswa": 700,
        "persentase_proporsi": 8.30,
        "gender": {
          "laki_laki": 180,
          "perempuan": 520
        },
        "mahasiswa_baru": 190,
        "rata_rata_ipk": 3.48,
        "sebaran_prodi": [
          {
            "kode_prodi": "FARMASI",
            "nama_prodi": "Farmasi",
            "jenjang": "S1",
            "mahasiswa_aktif": 390,
            "proporsi_fakultas": 55.71
          }
        ]
      }
    ],
    "updated_at": "2026-10-31T08:00:00+08:00"
  },
  "meta": {
    "source_system": "SIAKAD_IOSYS",
    "total_fakultas": 5,
    "watermark_latest": "2026-10-31T08:00:00+08:00"
  }
}
```

---

### 1.8 Endpoint Status Akreditasi Program Studi
- **Fungsi:** Mengambil data master legalitas dan status akreditasi seluruh program studi yang tercatat di SIAKAD (IOSYS), mencakup lembaga pengakreditasi (BAN-PT, LAMDIK, LAM-INFOKOM, LAMEMBA, LAM-PTKes), peringkat (Unggul, Baik Sekali, Baik, A, B), nomor & tanggal SK, masa berlaku, sisa hari kedaluwarsa, dan status peringatan dini (*early warning system* akreditasi) untuk visualisasi Stacked Bar (Unggul/A vs Baik Sekali/B) dan tabel status legalitas prodi di dashboard pimpinan.
- **Tabel Tujuan DW / Data Mart:** `dw.fact_akreditasi`, `ref.unit`, `mart.dm_mutu_akreditasi_prodi`
- **Method & Path:** `GET /api/v1/internal/siakad/akreditasi-prodi`
- **Query Parameter:**
  - `kode_fakultas` (string, opsional): Filter kode fakultas, misal `FT`, `FIP`.
  - `jenjang` (string, opsional): Filter jenjang, misal `S1`, `S2`.
  - `peringkat` (string, opsional): Filter peringkat, misal `Unggul`, `Baik Sekali`.
  - `status_peringatan` (string, opsional): `Aman`, `Persiapan (<12 Bulan)`, `Kritis (<6 Bulan)`, `Kedaluwarsa`.
  - `updated_since` (string, opsional): ISO timestamp untuk incremental sync.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data status akreditasi program studi berhasil diambil dari SIAKAD (IOSYS)",
  "data": {
    "rekap_peringkat": {
      "total_prodi": 32,
      "unggul_atau_a": 14,
      "baik_sekali_atau_b": 15,
      "baik_atau_c": 3,
      "persentase_unggul_baik_sekali": 90.63
    },
    "daftar_akreditasi": [
      {
        "id_prodi_siakad": "PRODI-PTI",
        "kode_prodi": "PTI",
        "nama_prodi": "Pendidikan Teknologi Informasi",
        "jenjang": "S1",
        "kode_fakultas": "FIP",
        "nama_fakultas": "Fakultas Ilmu Pendidikan",
        "lembaga_akreditasi": "LAMDIK",
        "peringkat": "Baik Sekali",
        "nilai_angka": 352,
        "nomor_sk": "412/SK/LAMDIK/Ak/S/IV/2023",
        "tanggal_sk": "2023-04-12",
        "tanggal_mulai_berlaku": "2023-04-12",
        "tanggal_kedaluwarsa": "2028-04-12",
        "sisa_hari_berlaku": 528,
        "status_peringatan": "Aman",
        "is_berlaku": true,
        "dokumen_sk_url": "https://siakad.hamzanwadi.ac.id/dokumen/akreditasi/sk_lamdik_pti_2023.pdf",
        "updated_at": "2026-09-10T11:00:00+08:00"
      },
      {
        "id_prodi_siakad": "PRODI-IF",
        "kode_prodi": "IF",
        "nama_prodi": "Teknik Informatika",
        "jenjang": "S1",
        "kode_fakultas": "FT",
        "nama_fakultas": "Fakultas Teknik",
        "lembaga_akreditasi": "LAM-INFOKOM",
        "peringkat": "Unggul",
        "nilai_angka": 368,
        "nomor_sk": "105/SK/LAM-INFOKOM/Ak/S/VIII/2024",
        "tanggal_sk": "2024-08-28",
        "tanggal_mulai_berlaku": "2024-08-28",
        "tanggal_kedaluwarsa": "2029-08-28",
        "sisa_hari_berlaku": 1032,
        "status_peringatan": "Aman",
        "is_berlaku": true,
        "dokumen_sk_url": "https://siakad.hamzanwadi.ac.id/dokumen/akreditasi/sk_laminfokom_if_2024.pdf",
        "updated_at": "2026-09-15T09:30:00+08:00"
      },
      {
        "id_prodi_siakad": "PRODI-PGSD",
        "kode_prodi": "PGSD",
        "nama_prodi": "Pendidikan Guru Sekolah Dasar",
        "jenjang": "S1",
        "kode_fakultas": "FIP",
        "nama_fakultas": "Fakultas Ilmu Pendidikan",
        "lembaga_akreditasi": "LAMDIK",
        "peringkat": "Unggul",
        "nilai_angka": 372,
        "nomor_sk": "208/SK/LAMDIK/Ak/S/V/2022",
        "tanggal_sk": "2022-05-18",
        "tanggal_mulai_berlaku": "2022-05-18",
        "tanggal_kedaluwarsa": "2027-05-18",
        "sisa_hari_berlaku": 198,
        "status_peringatan": "Persiapan (<12 Bulan)",
        "is_berlaku": true,
        "dokumen_sk_url": "https://siakad.hamzanwadi.ac.id/dokumen/akreditasi/sk_lamdik_pgsd_2022.pdf",
        "updated_at": "2026-10-01T14:15:00+08:00"
      }
    ]
  },
  "meta": {
    "page": 1,
    "page_size": 100,
    "total_records": 32,
    "total_pages": 1,
    "watermark_latest": "2026-10-01T14:15:00+08:00"
  }
}
```

---

### 1.9 Endpoint Ringkasan SDM: Dosen Tetap, Tenaga Kependidikan & Sertifikasi Pendidik (Serdos)
- **Fungsi:** Mengambil data ringkasan dan rincian Sumber Daya Manusia (SDM) yang terkelola di SIAKAD (IOSYS), meliputi:
  - **Total Dosen Tetap** (status aktif homebase prodi)
  - **Total Tenaga Kependidikan (Tendik)** (staf administrasi di tingkat unit/fakultas/laboratorium)
  - **Jumlah & Persentase Tersertifikasi (Serdos)** (dosen tetap yang memegang sertifikat pendidik)
  - Distribusi kualifikasi pendidikan dosen (S2 vs S3 Doktor) dan jabatan fungsional (Tenaga Pengajar hingga Guru Besar)
  - Agregasi per fakultas dan daftar data pegawai untuk memasok kartu statistik SDM Dashboard Rektor & Warek SDM.
- **Tabel Tujuan DW / Data Mart:** `dw.dim_pegawai`, `mart.dm_sdm_rekap_komposisi`, `mart.dm_sdm_kinerja_tendik`
- **Method & Path:** `GET /api/v1/internal/siakad/sdm-statistik`
- **Query Parameter:**
  - `kode_fakultas` (string, opsional): Filter kode fakultas.
  - `jenis_pegawai` (string, opsional): `DOSEN` atau `TENDIK`.
  - `status_serdos` (boolean, opsional): `true` (sudah serdos) atau `false` (belum serdos).
  - `status_kepegawaian` (string, opsional): `TETAP`, `KONTRAK`, `PNS_DPK`.
  - `updated_since` (string, opsional): ISO timestamp untuk incremental sync.
  - `page`, `page_size` (integer, opsional).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data statistik SDM dosen tetap, tendik, dan sertifikasi pendidik berhasil diambil dari SIAKAD (IOSYS)",
  "data": {
    "rekap_universitas": {
      "total_dosen_tetap": 312,
      "total_tenaga_kependidikan": 100,
      "dosen_tersertifikasi_serdos": 243,
      "persentase_tersertifikasi": 77.88,
      "dosen_belum_serdos": 69,
      "komposisi_pendidikan_dosen": {
        "s3_doktor": 58,
        "s2_magister": 254,
        "persen_s3": 18.59
      },
      "komposisi_jafung_dosen": {
        "guru_besar": 12,
        "lektor_kepala": 48,
        "lektor": 115,
        "asisten_ahli": 105,
        "tenaga_pengajar": 32
      },
      "rasio_dosen_mahasiswa": "1 : 21.6"
    },
    "rekap_per_fakultas": [
      {
        "kode_fakultas": "FIP",
        "nama_fakultas": "Fakultas Ilmu Pendidikan",
        "dosen_tetap": 110,
        "tenaga_kependidikan": 30,
        "tersertifikasi_serdos": 88,
        "persen_serdos": 80.00,
        "dosen_s3": 22
      },
      {
        "kode_fakultas": "FT",
        "nama_fakultas": "Fakultas Teknik",
        "dosen_tetap": 68,
        "tenaga_kependidikan": 20,
        "tersertifikasi_serdos": 52,
        "persen_serdos": 76.47,
        "dosen_s3": 12
      },
      {
        "kode_fakultas": "FMIPA",
        "nama_fakultas": "Fakultas Matematika dan IPA",
        "dosen_tetap": 58,
        "tenaga_kependidikan": 18,
        "tersertifikasi_serdos": 47,
        "persen_serdos": 81.03,
        "dosen_s3": 14
      },
      {
        "kode_fakultas": "FEB",
        "nama_fakultas": "Fakultas Ekonomi dan Bisnis",
        "dosen_tetap": 44,
        "tenaga_kependidikan": 16,
        "tersertifikasi_serdos": 34,
        "persen_serdos": 77.27,
        "dosen_s3": 6
      },
      {
        "kode_fakultas": "FIKES",
        "nama_fakultas": "Fakultas Ilmu Kesehatan",
        "dosen_tetap": 32,
        "tenaga_kependidikan": 16,
        "tersertifikasi_serdos": 22,
        "persen_serdos": 68.75,
        "dosen_s3": 4
      }
    ],
    "pegawai_list": [
      {
        "id_pegawai_siakad": "PEG-DSN-0812058501",
        "nip": "198505122010121003",
        "nidn": "0812058501",
        "nama_lengkap": "Dr. Irwan Hamzani, M.Pd.",
        "jenis_kelamin": "L",
        "jenis_pegawai": "DOSEN",
        "status_kepegawaian": "TETAP",
        "status_aktif": "AKTIF",
        "homebase_fakultas": "FIP",
        "homebase_prodi": "PTI",
        "pendidikan_terakhir": "S3",
        "jabatan_fungsional": "Lektor Kepala",
        "is_serdos": true,
        "nomor_serdos": "19081205850123",
        "tahun_serdos": 2019,
        "bidang_studi_serdos": "Pendidikan Teknologi Informasi",
        "updated_at": "2026-10-18T10:00:00+08:00"
      },
      {
        "id_pegawai_siakad": "PEG-TND-1990082001",
        "nip": "199008202015042001",
        "nidn": null,
        "nama_lengkap": "Khadijah, S.Kom.",
        "jenis_kelamin": "P",
        "jenis_pegawai": "TENDIK",
        "status_kepegawaian": "TETAP",
        "status_aktif": "AKTIF",
        "homebase_fakultas": "FT",
        "homebase_prodi": null,
        "unit_kerja": "Bagian Tata Usaha Fakultas Teknik",
        "pendidikan_terakhir": "S1",
        "jabatan_struktural": "Kepala Sub Bagian Tata Usaha FT",
        "is_serdos": false,
        "nomor_serdos": null,
        "tahun_serdos": null,
        "bidang_studi_serdos": null,
        "updated_at": "2026-10-20T11:30:00+08:00"
      }
    ]
  },
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 412,
    "total_pages": 206,
    "watermark_latest": "2026-10-20T11:30:00+08:00"
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
