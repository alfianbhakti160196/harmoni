# SPESIFIKASI KEBUTUHAN API SISTEM INTERNAL & EKSTERNAL
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> Dokumen ini adalah **spesifikasi teknis kebutuhan API resmi** yang dikonsumsi oleh modul ETL (Extract-Transform-Load) HARMONI untuk mengisi Data Warehouse (`dw`), Data Mart (`mart`), dan mesin analitik/KPI/AI.
> Mencakup **8 sistem internal** (SIAKAD, PMB, Keuangan, SIMPEG, Absensi, LPPM, E-MONEVIN, Tracer Study) dan **5 sistem eksternal resmi** (PDDIKTI Neo Feeder, SISTER, SINTA, Scopus/Scholar, BAN-PT/LAM).
>
> **Karakteristik Integrasi:** Read-Only, RESTful JSON over HTTPS / Web Service JSON-RPC, Incremental Sync (watermarking), dan diamankan dengan API Token/Key.

---

## DAFTAR ISI

0. [Standar Protokol & Konvensi API](#0-standar-protokol--konvensi-api)
1. [Sistem Informasi Akademik (SIAKAD / IOSYS)](#1-sistem-informasi-akademik-siakad--iosys)
2. [Sistem Penerimaan Mahasiswa Baru (PMB)](#2-sistem-penerimaan-mahasiswa-baru-pmb)
3. [Sistem Informasi Keuangan](#3-sistem-informasi-keuangan)
4. [Sistem Informasi Kepegawaian & SDM (SIMPEG)](#4-sistem-informasi-kepegawaian--sdm-simpeg)
5. [Sistem Absensi & Presensi Pegawai](#5-sistem-absensi--presensi-pegawai)
6. [Sistem Informasi Penelitian & Pengabdian (LPPM)](#6-sistem-informasi-penelitian--pengabdian-lppm)
7. [Sistem E-MONEVIN (Penjaminan Mutu & Kinerja Unit)](#7-sistem-e-monevin-penjaminan-mutu--kinerja-unit)
8. [Sistem Alumni & Tracer Study](#8-sistem-alumni--tracer-study)
9. [Web Service Resmi PDDIKTI (Neo Feeder Kemdiktisaintek)](#9-web-service-resmi-pddikti-neo-feeder-kemdiktisaintek)
10. [Web Service Resmi SISTER Kemdiktisaintek](#10-web-service-resmi-sister-kemdiktisaintek)
11. [Integrasi Data SINTA (Science and Technology Index)](#11-integrasi-data-sinta-science-and-technology-index)
12. [Integrasi Data Scopus (Elsevier Developer API) & Google Scholar](#12-integrasi-data-scopus-elsevier-developer-api--google-scholar)
13. [Integrasi Data Akreditasi BAN-PT & Lembaga Akreditasi Mandiri (LAM)](#13-integrasi-data-akreditasi-ban-pt--lembaga-akreditasi-mandiri-lam)
14. [Matriks Pemetaan Identitas Antar-Sistem (Identity Key Matching)](#14-matriks-pemetaan-identitas-antar-sistem-identity-key-matching)
15. [Panduan Pengujian & Checklist Kesiapan Integrasi](#15-panduan-pengujian--checklist-kesiapan-integrasi)

---

## 0. Standar Protokol & Konvensi API

Untuk menjamin keandalan dan konsistensi data yang masuk ke pipeline HARMONI, seluruh API sistem internal wajib mematuhi standar berikut:

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

---

## 1. Sistem Informasi Akademik (SIAKAD / IOSYS)

SIAKAD (aplikasi berbasis IOSYS) adalah sumber kebenaran (*single source of truth*) utama untuk data civitas akademika, perkuliahan, mata kuliah, kelulusan, ringkasan eksekutif universitas, sebaran fakultas, status akreditasi prodi, serta profil dosen tetap (termasuk sertifikasi serdos) dan tenaga kependidikan.

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

## 2. Sistem Penerimaan Mahasiswa Baru (PMB)

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

## 3. Sistem Informasi Keuangan

Sistem Keuangan menyediakan data pagu anggaran tahunan, realisasi serapan anggaran belanja operasional/program, serta pendapatan universitas.

### 3.1 Endpoint Realisasi Anggaran Belanja Bulanan
- **Fungsi:** Mengambil data pagu dan realisasi pengeluaran per unit kerja per bulan (untuk KPI Serapan Anggaran).
- **Tabel Tujuan DW:** `dw.fact_keuangan_bulanan`
- **Method & Path:** `GET /api/v1/internal/keuangan/realisasi-anggaran`
- **Query Parameter:** `tahun` (integer, wajib), `bulan` (integer 1-12), `kode_unit`.

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data realisasi anggaran berhasil diambil",
  "data": [
    {
      "kode_unit": "FTIK",
      "nama_unit": "Fakultas Teknik dan Ilmu Komputer",
      "tahun": 2025,
      "bulan": 9,
      "jenis": "BELANJA",
      "kategori": "OPERASIONAL_PENDIDIKAN",
      "kode_akun": "5.2.1.01",
      "nama_akun": "Belanja Bahan dan Praktikum Laboratorium",
      "kode_program_kerja": "PROG-FTIK-2025-03",
      "anggaran_pagu": 120000000.00,
      "realisasi_bulan_ini": 18500000.00,
      "realisasi_kumulatif_sd_bulan": 92500000.00,
      "sisa_anggaran": 27500000.00,
      "persen_serapan": 77.08,
      "updated_at": "2026-10-02T17:00:00+08:00"
    },
    {
      "kode_unit": "PTI",
      "nama_unit": "Prodi Pendidikan Teknologi Informasi",
      "tahun": 2025,
      "bulan": 9,
      "jenis": "BELANJA",
      "kategori": "PENGEMBANGAN_SDM",
      "kode_akun": "5.2.2.04",
      "nama_akun": "Pelatihan dan Sertifikasi Kompetensi Dosen",
      "kode_program_kerja": "PROG-PTI-2025-01",
      "anggaran_pagu": 45000000.00,
      "realisasi_bulan_ini": 15000000.00,
      "realisasi_kumulatif_sd_bulan": 35000000.00,
      "sisa_anggaran": 10000000.00,
      "persen_serapan": 77.78,
      "updated_at": "2026-10-02T17:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 180,
    "total_pages": 90,
    "watermark_latest": "2026-10-02T17:00:00+08:00"
  }
}
```

---

### 3.2 Endpoint Pendapatan Universitas
- **Fungsi:** Mengambil data arus penerimaan kas universitas berdasarkan kategori (UKT Mahasiswa, Hibah, Kerjasama, Unit Bisnis).
- **Tabel Tujuan DW:** `dw.fact_keuangan_bulanan`
- **Method & Path:** `GET /api/v1/internal/keuangan/pendapatan`
- **Query Parameter:** `tahun` (integer, wajib), `bulan` (integer).

#### Contoh Response JSON:
```json
{
  "status": "success",
  "message": "Data pendapatan universitas berhasil diambil",
  "data": [
    {
      "kode_unit": "UNHAM",
      "nama_unit": "Universitas Hamzanwadi",
      "tahun": 2025,
      "bulan": 9,
      "jenis": "PENDAPATAN",
      "kategori": "UKT_MAHASISWA",
      "kode_akun": "4.1.1.01",
      "nama_akun": "Penerimaan UKT Semester Ganjil 2025/2026",
      "target_pagu": 28000000000.00,
      "realisasi_bulan_ini": 8450000000.00,
      "realisasi_kumulatif_sd_bulan": 26150000000.00,
      "persen_capaian": 93.39,
      "updated_at": "2026-10-02T17:00:00+08:00"
    },
    {
      "kode_unit": "LPPM",
      "nama_unit": "Lembaga Penelitian dan Pengabdian",
      "tahun": 2025,
      "bulan": 9,
      "jenis": "PENDAPATAN",
      "kategori": "HIBAH_EKSTERNAL",
      "kode_akun": "4.2.1.02",
      "nama_akun": "Penerimaan Dana Hibah BIMA Kemdiktisaintek Tahap 1",
      "target_pagu": 1500000000.00,
      "realisasi_bulan_ini": 1050000000.00,
      "realisasi_kumulatif_sd_bulan": 1050000000.00,
      "persen_capaian": 70.00,
      "updated_at": "2026-10-02T17:00:00+08:00"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 2,
    "total_records": 48,
    "total_pages": 24,
    "watermark_latest": "2026-10-02T17:00:00+08:00"
  }
}
```

---

## 4. Sistem Informasi Kepegawaian & SDM (SIMPEG)

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

## 5. Sistem Absensi & Presensi Pegawai

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

## 6. Sistem Informasi Penelitian & Pengabdian (LPPM)

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

## 7. Sistem E-MONEVIN (Penjaminan Mutu & Kinerja Unit)

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

## 8. Sistem Alumni & Tracer Study

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

## 9. Web Service Resmi PDDIKTI (Neo Feeder Kemdiktisaintek)

Neo Feeder adalah sistem pelaporan data akademik resmi perguruan tinggi ke pangkalan data kementerian. HARMONI menggunakan protokol **JSON-RPC** resmi Neo Feeder melalui endpoint Web Service `POST /ws/live2.php`.

### 9.1 Mekanisme Autentikasi (`GetToken`)
- **Fungsi:** Mengambil token sesi akses ke Neo Feeder menggunakan username dan password resmi operator PDDIKTI.
- **Method & Path:** `POST /ws/live2.php`
- **Request Body (JSON):**
  ```json
  {
    "act": "GetToken",
    "username": "081042@hamzanwadi.ac.id",
    "password": "PASSWORD_FEEDER_DIRAHASIAKAN"
  }
  ```
- **Response JSON:**
  ```json
  {
    "error_code": 0,
    "error_desc": "",
    "data": {
      "token": "4a7b9c2d1e8f0a3b5c7d9e1f2a4b6c8d"
    }
  }
  ```
  *(Token di-cache di Redis selama 60 menit untuk request selanjutnya).*

---

### 9.2 Fungsi `GetListMahasiswa` (Data PDDikti)
- **Fungsi:** Mengambil data mahasiswa yang terdaftar di pelaporan nasional PDDikti untuk cross-validation dengan SIAKAD.
- **Tabel Tujuan DW:** `dw.dim_mahasiswa` (pencocokan `id_registrasi_feeder` & `nim`).
- **Request Body (JSON):**
  ```json
  {
    "act": "GetListMahasiswa",
    "token": "4a7b9c2d1e8f0a3b5c7d9e1f2a4b6c8d",
    "filter": "id_prodi='8b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d' AND id_periode='20251'",
    "order": "nim asc",
    "limit": 50,
    "offset": 0
  }
  ```
- **Response JSON:**
  ```json
  {
    "error_code": 0,
    "error_desc": "",
    "data": [
      {
        "id_registrasi_mahasiswa": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
        "id_mahasiswa": "3c2a1b0f-9e8d-7c6b-5a4f-3e2d1c0b9a8f",
        "nim": "2210510042",
        "nama_mahasiswa": "Ahmad Fauzi",
        "jenis_kelamin": "L",
        "id_prodi": "8b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
        "nama_program_studi": "Pendidikan Teknologi Informasi",
        "id_periode_masuk": "20221",
        "nama_status_mahasiswa": "Aktif",
        "id_jalur_daftar": 1,
        "nama_jalur_daftar": "SNMPTN",
        "ipk": 3.65,
        "total_sks": 92
      }
    ]
  }
  ```

---

### 9.3 Fungsi `GetAktivitasKuliahMahasiswa` (AKM PDDikti)
- **Fungsi:** Mengambil pelaporan status perkuliahan semesteran, IPS, IPK, dan SKS semester yang sudah tervalidasi di PDDikti.
- **Tabel Tujuan DW:** `dw.fact_mahasiswa_semester`
- **Request Body (JSON):**
  ```json
  {
    "act": "GetAktivitasKuliahMahasiswa",
    "token": "4a7b9c2d1e8f0a3b5c7d9e1f2a4b6c8d",
    "filter": "id_semester='20251'",
    "order": "",
    "limit": 50,
    "offset": 0
  }
  ```
- **Response JSON:**
  ```json
  {
    "error_code": 0,
    "error_desc": "",
    "data": [
      {
        "id_registrasi_mahasiswa": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
        "nim": "2210510042",
        "nama_mahasiswa": "Ahmad Fauzi",
        "id_semester": "20251",
        "nama_semester": "2025/2026 Ganjil",
        "id_status_mahasiswa": "A",
        "nama_status_mahasiswa": "Aktif",
        "ips": 3.75,
        "ipk": 3.65,
        "sks_semester": 20,
        "total_sks": 92
      }
    ]
  }
  ```

---

### 9.4 Fungsi `GetDosenPengajarKelasKuliah`
- **Fungsi:** Mengambil beban SKS mengajar dosen resmi yang dilaporkan ke kementerian.
- **Tabel Tujuan DW:** `dw.fact_pengajaran`
- **Request Body (JSON):**
  ```json
  {
    "act": "GetDosenPengajarKelasKuliah",
    "token": "4a7b9c2d1e8f0a3b5c7d9e1f2a4b6c8d",
    "filter": "id_semester='20251'",
    "limit": 50,
    "offset": 0
  }
  ```
- **Response JSON:**
  ```json
  {
    "error_code": 0,
    "error_desc": "",
    "data": [
      {
        "id_aktivitas_mengajar": "7f8e9d0c-1b2a-3c4d-5e6f-7a8b9c0d1e2f",
        "id_registrasi_dosen": "5a4b3c2d-1e0f-9a8b-7c6d-5e4f3a2b1c0d",
        "nidn": "0812058501",
        "nama_dosen": "Dr. Irwan Hamzani, M.Pd.",
        "id_kelas_kuliah": "KLS-20251-PTI3201-A",
        "nama_kelas_kuliah": "A",
        "id_mata_kuliah": "MK-PTI3201",
        "nama_mata_kuliah": "Kecerdasan Buatan dan Pembelajaran Mesin",
        "sks_substansi_total": 3.0,
        "sks_beban": 1.5,
        "rencana_minggu_pertemuan": 16,
        "realisasi_minggu_pertemuan": 16
      }
    ]
  }
  ```

---

### 9.5 Fungsi `GetListKelulusanMahasiswa`
- **Fungsi:** Mengambil data kelulusan mahasiswa dan nomor ijazah nasional (PIN PDDikti).
- **Tabel Tujuan DW:** `dw.fact_kelulusan`
- **Request Body (JSON):**
  ```json
  {
    "act": "GetListKelulusanMahasiswa",
    "token": "4a7b9c2d1e8f0a3b5c7d9e1f2a4b6c8d",
    "filter": "id_periode_keluar='20242'",
    "limit": 50,
    "offset": 0
  }
  ```
- **Response JSON:**
  ```json
  {
    "error_code": 0,
    "error_desc": "",
    "data": [
      {
        "id_keluar": "1a2b3c4d-5e6f-7a8b-9c0d-1e2f3a4b5c6d",
        "nim": "2010510005",
        "nama_mahasiswa": "Muhammad Zaini",
        "id_jenis_keluar": "1",
        "nama_jenis_keluar": "Lulus",
        "tanggal_keluar": "2024-08-15",
        "id_periode_keluar": "20242",
        "keterangan": "Yudisium Wisuda Periode II 2024",
        "nomor_sk_yudisium": "SK-YUD/2024/08/112",
        "nomor_seri_ijazah": "12345/UNHAM/S1/2024",
        "ipk": 3.78
      }
    ]
  }
  ```

---

## 10. Web Service Resmi SISTER Kemdiktisaintek

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

## 11. Integrasi Data SINTA (Science and Technology Index)

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

## 12. Integrasi Data Scopus (Elsevier Developer API) & Google Scholar

Digunakan untuk memvalidasi sitasi internasional bereputasi, status kuartil jurnal (Q1–Q4), dan publikasi terindeks Scopus.

### 12.1 Scopus Author Retrieval API
- **Protokol:** HTTPS REST API Elsevier (`https://api.elsevier.com/content/author`)
- **Headers:**
  ```http
  X-ELS-APIKey: <ELSEVIER_API_KEY>
  X-ELS-Insttoken: <ELSEVIER_INST_TOKEN>
  Accept: application/json
  ```
- **Method & Path:** `GET /content/author?author_id=57210984500&view=STANDARD`

#### Contoh Response JSON:
```json
{
  "author-retrieval-response": [
    {
      "coredata": {
        "dc:identifier": "AUTHOR_ID:57210984500",
        "eid": "9-s2.0-57210984500",
        "document-count": "14",
        "cited-by-count": "86",
        "citation-count": "86"
      },
      "author-profile": {
        "preferred-name": {
          "surname": "Hamzani",
          "given-name": "Irwan"
        },
        "affiliation-current": {
          "affiliation": {
            "@id": "60105432",
            "affilname": "Universitas Hamzanwadi"
          }
        },
        "h-index": "5"
      }
    }
  ]
}
```

---

### 12.2 Scopus Search API (Pencarian Publikasi & Kuartil)
- **Method & Path:** `GET /content/search/scopus?query=AU-ID(57210984500)&count=25`
- **Response JSON:**
  ```json
  {
    "search-results": {
      "opensearch:totalResults": "14",
      "entry": [
        {
          "dc:title": "Machine Learning Predictive Analytics for Student Retention in Indonesian Islamic Higher Education",
          "prism:publicationName": "IEEE Access",
          "prism:issn": "21693536",
          "prism:doi": "10.1109/ACCESS.2025.3214567",
          "prism:coverDate": "2025-05-15",
          "prism:aggregationType": "Journal",
          "citedby-count": "8",
          "subtypeDescription": "Article",
          "kuartil_simpanan": "Q1"
        }
      ]
    }
  }
  ```

---

## 13. Integrasi Data Akreditasi BAN-PT & Lembaga Akreditasi Mandiri (LAM)

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

## 14. Matriks Pemetaan Identitas Antar-Sistem (Identity Key Matching)

Tabel berikut menunjukkan keterkaitan kunci identitas (*natural keys*) yang digunakan pipeline ETL HARMONI untuk menghubungkan rekaman data dari **8 sistem internal** dan **5 sistem eksternal**:

| Entitas Utama | Kunci Internal Kampus | Kunci Sistem Eksternal Resmi | Skema & Kolom Tujuan HARMONI |
|---|---|---|---|
| **Dosen** | `nidn` (SIMPEG, SIAKAD / IOSYS, LPPM), `nip` (Absensi) | `id_sdm` (SISTER), `sinta_id` (SINTA), `scopus_id` (Scopus) | `dw.dim_pegawai (nidn, nip, sso_subject, sinta_id, scopus_id)` |
| **Karyawan / Tendik** | `nip` (SIMPEG, SIAKAD / IOSYS, Absensi) | `sso_id` (Keycloak SSO Hamzanwadi) | `dw.dim_pegawai (nip, sso_subject)` |
| **Mahasiswa** | `nim` (SIAKAD / IOSYS, PMB, Tracer) | `id_registrasi_mahasiswa` (PDDIKTI Neo Feeder) | `dw.dim_mahasiswa (nim, id_registrasi_feeder)` |
| **Program Studi / Unit** | `kode_prodi` (SIAKAD / IOSYS, PMB, Keuangan, E-MONEVIN) | `id_prodi` (Neo Feeder & SISTER UUID) | `ref.unit (kode, id_prodi_feeder)` |
| **Semester** | `semester_kode` (SIAKAD / IOSYS: `20251`) | `id_semester` (PDDIKTI: `20251`) | `ref.semester (kode)` |
| **Mata Kuliah** | `kode_mk` + `kurikulum` (SIAKAD / IOSYS) | `id_mata_kuliah` (Neo Feeder UUID) | `dw.dim_mata_kuliah (kode_mk, kurikulum)` |
| **Publikasi Ilmiah** | `judul_normal` (LPPM) | `doi` (Scopus / SINTA), `issn_isbn` | `dw.fact_publikasi (doi, judul_normal, indeks)` |
| **Akreditasi** | `kode_prodi` (SIAKAD / IOSYS), `kode_unit` (E-MONEVIN) | `nomor_sk` (BAN-PT, LAMDIK, LAMINFOKOM) | `dw.fact_akreditasi (unit_id, lembaga, nomor_sk)` |

---

## 15. Panduan Pengujian & Checklist Kesiapan Integrasi

Sebelum menghubungkan sistem internal dan eksternal ke pipeline ETL HARMONI di server produksi, tim pengembang dan administrator wajib memverifikasi hal-hal berikut:

- [ ] **Endpoint Internal & Eksternal Aktif:** Seluruh endpoint mengembalikan status HTTP 200 OK atau response error_code 0 (Neo Feeder).
- [ ] **Otentikasi Token Terlindungi:** Kunci API sistem sumber (`SSO_CLIENT_SECRET`, `SISTER_TOKEN`, `ELSEVIER_API_KEY`) tersimpan aman dalam format terenkripsi AES-256-GCM di `etl.sumber_data`.
- [ ] **Filter Inkremental Berfungsi:** Parameter query `updated_since` (Internal API) dan filter `id_periode` (Feeder) hanya menarik data baru/berubah.
- [ ] **Format JSON Sesuai Kontrak:** Struktur nama field, tipe data angka (`float/int`), dan format tanggal ISO 8601 persis mengikuti contoh response pada dokumen ini.
- [ ] **Pencocokan Identitas Berjalan Baik:** Identity matching di `etl.identity_map` mencapai rasio otomatis $\ge 95\%$ untuk NIDN, NIP, dan NIM.
- [ ] **Performa Query Sumber Optimal:** Waktu respons API untuk batch 100 baris tidak melebihi 2 detik.
- [ ] **IP Whitelisting Server:** IP Server HARMONI telah didaftarkan pada whitelist firewall Neo Feeder dan server database internal.

---
*Dokumen spesifikasi kebutuhan API internal dan eksternal HARMONI ini disahkan sebagai standar integrasi resmi data Universitas Hamzanwadi.*
