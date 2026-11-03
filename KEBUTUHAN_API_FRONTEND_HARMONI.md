# SPESIFIKASI KEBUTUHAN REST API FRONTEND
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

Dokumen ini mendefinisikan kebutuhan endpoint REST API dan struktur data response JSON untuk kebutuhan integrasi frontend aplikasi **HARMONI** (`https://harmonites.adnantech.biz.id`) yang dibangun dengan **Vue 3**, **Bootstrap 5**, dan **Chart.js**.

---

## 1. STANDAR PROTOKOL & FORMAT UMUM API

### 1.1 Base URL & Versi API
- **Production Base URL:** `https://api.harmonites.adnantech.biz.id/api/v1`
- **Development / Staging Base URL:** `http://localhost:8000/api/v1`

### 1.2 Header Standar HTTP
Setiap request dari frontend ke backend wajib menyertakan header:
```http
Authorization: Bearer <JWT_ACCESS_TOKEN>
Content-Type: application/json
Accept: application/json
```

### 1.3 Format Standar Respons JSON (Envelope Pattern)

#### Format Sukses (HTTP 200 / 201)
```json
{
  "status": "success",
  "message": "Data berhasil dimuat.",
  "data": {},
  "meta": {
    "timestamp": "2026-11-03T14:02:40Z",
    "version": "v1.0.0"
  }
}
```

#### Format Sukses Paginated (HTTP 200)
```json
{
  "status": "success",
  "message": "Daftar data berhasil dimuat.",
  "data": [],
  "pagination": {
    "current_page": 1,
    "per_page": 10,
    "total_records": 1248,
    "total_pages": 125,
    "has_next": true,
    "has_prev": false
  },
  "meta": {
    "timestamp": "2026-11-03T14:02:40Z"
  }
}
```

#### Format Gagal / Error (HTTP 4xx / 5xx)
```json
{
  "status": "error",
  "message": "Deskripsi kesalahan yang ramah pengguna.",
  "error_code": "ERR_VALIDATION_FAILED",
  "errors": [
    {
      "field": "email",
      "message": "Format email tidak valid atau sudah terdaftar."
    }
  ],
  "meta": {
    "timestamp": "2026-11-03T14:02:40Z"
  }
}
```

---

## 2. MODUL ADMINISTRATOR SISTEM

Modul ini digunakan oleh peran `ADMIN` dan `PENGELOLA_DATA` untuk mengelola operasional teknis, pengguna, rute hak akses, integrasi data eksternal/internal, sinkronisasi ETL, audit mutu & log sistem.

---

### 2.1 Admin Dashboard (`/admin`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/admin`
- **Tujuan Halaman:** Menyajikan ringkasan statistik sistem, status operasional antrean sinkronisasi, koneksi API, log peringatan, serta grafik tren traffic & request.
- **Endpoint Utama:** `GET /api/v1/admin/dashboard`
- **Headers:** `Authorization: Bearer <token>`

#### Response JSON:
```json
{
  "status": "success",
  "message": "Data dashboard admin berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "id": "users_stat",
        "title": "Total Pengguna Aktif",
        "value": "1.248",
        "trend": "+12% bulan ini",
        "trend_type": "up",
        "icon": "bi-people-fill",
        "theme": "card-blue"
      },
      {
        "id": "integrations_stat",
        "title": "Status Integrasi API",
        "value": "4 / 4 Aktif",
        "trend": "Semua layanan normal",
        "trend_type": "up",
        "icon": "bi-hdd-network-fill",
        "theme": "card-emerald"
      },
      {
        "id": "sync_queue_stat",
        "title": "Antrean Sinkronisasi",
        "value": "0 Task",
        "trend": "Data valid & konsisten",
        "trend_type": "neutral",
        "icon": "bi-arrow-repeat",
        "theme": "card-gold"
      },
      {
        "id": "system_logs_stat",
        "title": "Peringatan Sistem (Log)",
        "value": "3 Error",
        "trend": "Perlu ditinjau",
        "trend_type": "warning",
        "icon": "bi-exclamation-triangle-fill",
        "theme": "card-purple"
      }
    ],
    "chart_aktivitas": {
      "labels": ["Senin", "Selasa", "Rabu", "Kamis", "Jumat", "Sabtu", "Minggu"],
      "datasets": [
        {
          "label": "Aktivitas Request API & Login",
          "backgroundColor": "rgba(13, 110, 253, 0.08)",
          "borderColor": "#0d6efd",
          "borderWidth": 2,
          "fill": true,
          "tension": 0.4,
          "data": [1200, 1900, 1500, 2200, 2800, 1400, 1100]
        }
      ]
    },
    "status_layanan": [
      {
        "id": "db_service",
        "name": "Database PostgreSQL (Data Warehouse)",
        "status": "Online",
        "latency": "12 ms",
        "uptime": "99.98%",
        "theme": "success"
      },
      {
        "id": "redis_cache",
        "name": "Redis Memory Cache & Queue",
        "status": "Online",
        "latency": "2 ms",
        "uptime": "99.99%",
        "theme": "success"
      },
      {
        "id": "worker_service",
        "name": "Background Job Sync Worker",
        "status": "Idle",
        "latency": "4 ms",
        "uptime": "99.95%",
        "theme": "primary"
      },
      {
        "id": "api_gateway",
        "name": "API Gateway & Reverse Proxy (Nginx)",
        "status": "Online",
        "latency": "8 ms",
        "uptime": "100%",
        "theme": "success"
      }
    ],
    "log_aktivitas_terkini": [
      {
        "id": "LOG-5091",
        "timestamp": "Hari ini, 14:20 WITA",
        "level": "INFO",
        "user": "rektor@hamzanwadi.ac.id",
        "action": "Login berhasil ke Dashboard Eksekutif",
        "ip": "192.168.1.45"
      },
      {
        "id": "LOG-5090",
        "timestamp": "Hari ini, 14:15 WITA",
        "level": "WARNING",
        "user": "admin@hamzanwadi.ac.id",
        "action": "Percobaan akses modul terbatas tanpa izin",
        "ip": "203.0.113.88"
      },
      {
        "id": "LOG-5089",
        "timestamp": "Hari ini, 13:50 WITA",
        "level": "ERROR",
        "user": "system@feeder",
        "action": "Koneksi timeout ke API Feeder PDDikti",
        "ip": "10.0.0.15"
      }
    ],
    "quick_actions": [
      { "title": "Kelola Pengguna Sistem", "path": "/admin/users", "icon": "bi-people" },
      { "title": "Konfigurasi API & Webhook", "path": "/admin/integrasi", "icon": "bi-hdd-network" },
      { "title": "Sinkronisasi Data Manual", "path": "/admin/sinkronisasi", "icon": "bi-arrow-repeat" },
      { "title": "Lihat Log Sistem & Audit", "path": "/admin/logs", "icon": "bi-journal-code" }
    ]
  }
}
```

---

### 2.2 Manajemen Pengguna (`/admin/users`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/admin/users`
- **Tujuan Halaman:** Mengelola akun pengguna (CRUD), penetapan role RBAC, penetapan unit kerja/fakultas/prodi, reset password, dan aktivasi status akun.

#### 2.2.1 Get List Users (Paginated & Filterable)
- **Endpoint:** `GET /api/v1/admin/users`
- **Query Params:**
  - `page` (integer, default: 1)
  - `per_page` (integer, default: 10)
  - `search` (string, filter nama / email / unit)
  - `role` (string: `ALL`, `REKTOR`, `WAKIL_REKTOR`, `DEKAN`, `KAPRODI`, `DOSEN`, `TENDIK`, `DIREKTORAT`, `PENGELOLA_DATA`, `ADMIN`)
  - `status` (string: `ALL`, `Aktif`, `Nonaktif`)

##### Response JSON:
```json
{
  "status": "success",
  "message": "Daftar pengguna berhasil dimuat.",
  "data": [
    {
      "id": 1,
      "name": "Dr. H. Muhammad, M.Pd.",
      "email": "muhammad@hamzanwadi.ac.id",
      "role": "REKTOR",
      "unit": "Rektorat",
      "status": "Aktif",
      "lastLogin": "Hari ini, 10:24 WITA",
      "created_at": "2025-01-15T08:00:00Z"
    },
    {
      "id": 2,
      "name": "Dr. Siti Nurhaliza, M.Sc.",
      "email": "siti.nurhaliza@hamzanwadi.ac.id",
      "role": "DEKAN",
      "unit": "Fakultas Teknik",
      "status": "Aktif",
      "lastLogin": "Kemarin, 14:15 WITA",
      "created_at": "2025-02-01T09:30:00Z"
    },
    {
      "id": 3,
      "name": "Ahmad Fauzi, M.Kom.",
      "email": "fauzi@hamzanwadi.ac.id",
      "role": "KAPRODI",
      "unit": "Pendidikan Teknologi Informasi",
      "status": "Aktif",
      "lastLogin": "2 hari lalu",
      "created_at": "2025-02-10T11:00:00Z"
    },
    {
      "id": 4,
      "name": "Prof. Dr. Hamzanwadi, M.Ed.",
      "email": "hamzanwadi@hamzanwadi.ac.id",
      "role": "WAKIL_REKTOR",
      "unit": "Bidang Akademik (WR I)",
      "status": "Aktif",
      "lastLogin": "Hari ini, 08:30 WITA",
      "created_at": "2025-01-20T10:00:00Z"
    },
    {
      "id": 5,
      "name": "Dewi Lestari, S.E.",
      "email": "dewi.lestari@hamzanwadi.ac.id",
      "role": "TENDIK",
      "unit": "Biro Keuangan",
      "status": "Aktif",
      "lastLogin": "3 hari lalu",
      "created_at": "2025-03-01T14:20:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 10,
    "total_records": 1248,
    "total_pages": 125,
    "has_next": true,
    "has_prev": false
  }
}
```

#### 2.2.2 Create User Baru
- **Endpoint:** `POST /api/v1/admin/users`
- **Request Body:**
```json
{
  "name": "Dr. Irwan Hamzani, M.Pd.",
  "email": "irwan.hamzani@hamzanwadi.ac.id",
  "password": "PasswordAman123!",
  "role": "DOSEN",
  "unit": "Fakultas Keguruan dan Ilmu Pendidikan",
  "status": "Aktif"
}
```
##### Response JSON (HTTP 201 Created):
```json
{
  "status": "success",
  "message": "Pengguna baru berhasil ditambahkan.",
  "data": {
    "id": 6,
    "name": "Dr. Irwan Hamzani, M.Pd.",
    "email": "irwan.hamzani@hamzanwadi.ac.id",
    "role": "DOSEN",
    "unit": "Fakultas Keguruan dan Ilmu Pendidikan",
    "status": "Aktif",
    "created_at": "2026-11-03T14:15:00Z"
  }
}
```

#### 2.2.3 Update Pengguna & Role
- **Endpoint:** `PUT /api/v1/admin/users/{id}`
- **Request Body:**
```json
{
  "name": "Dr. Siti Nurhaliza, M.Sc.",
  "role": "DEKAN",
  "unit": "Fakultas Teknik",
  "status": "Aktif"
}
```
##### Response JSON (HTTP 200):
```json
{
  "status": "success",
  "message": "Data pengguna berhasil diperbarui.",
  "data": {
    "id": 2,
    "name": "Dr. Siti Nurhaliza, M.Sc.",
    "email": "siti.nurhaliza@hamzanwadi.ac.id",
    "role": "DEKAN",
    "unit": "Fakultas Teknik",
    "status": "Aktif"
  }
}
```

#### 2.2.4 Toggle Status Aktif / Nonaktif
- **Endpoint:** `PATCH /api/v1/admin/users/{id}/status`
- **Request Body:**
```json
{
  "status": "Nonaktif"
}
```
##### Response JSON:
```json
{
  "status": "success",
  "message": "Status pengguna berhasil diubah menjadi Nonaktif.",
  "data": {
    "id": 3,
    "status": "Nonaktif"
  }
}
```

#### 2.2.5 Reset Password Pengguna
- **Endpoint:** `POST /api/v1/admin/users/{id}/reset-password`
- **Request Body:**
```json
{
  "new_password": "NewSecretPassword2026!"
}
```
##### Response JSON:
```json
{
  "status": "success",
  "message": "Password pengguna berhasil diperbarui dan instruksi dikirim ke email."
}
```

---

### 2.3 Peran & Hak Akses Rute (`/admin/routes`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/admin/routes`
- **Tujuan Halaman:** Konfigurasi matriks hak akses RBAC (Role-Based Access Control) untuk setiap modul dan rute menu dinamis di frontend.

#### 2.3.1 Get Master Peran & Matriks Rute
- **Endpoint:** `GET /api/v1/admin/routes`
- **Response JSON:**
```json
{
  "status": "success",
  "message": "Matriks rute dan hak akses berhasil dimuat.",
  "data": {
    "roles_master": [
      { "id": "REKTOR", "name": "Rektor", "description": "Akses eksekutif seluruh data strategis universitas" },
      { "id": "WAKIL_REKTOR", "name": "Wakil Rektor", "description": "Akses bidang kerja dan indikator strategis" },
      { "id": "DEKAN", "name": "Dekan", "description": "Akses agregat fakultas dan program studi binaan" },
      { "id": "KAPRODI", "name": "Ketua Program Studi", "description": "Akses operasional akademik dan mutu prodi" },
      { "id": "DOSEN", "name": "Dosen", "description": "Akses tridarma diri sendiri (Read-only)" },
      { "id": "TENDIK", "name": "Karyawan / Tendik", "description": "Akses kepegawaian dan layanan unit" },
      { "id": "DIREKTORAT", "name": "Direktorat / Kepala Unit", "description": "Akses program kerja, KPI dan anggaran unit" },
      { "id": "PENGELOLA_DATA", "name": "Pengelola Data", "description": "Validasi kualitas dan sinkronisasi ETL" },
      { "id": "ADMIN", "name": "Administrator Sistem", "description": "Akses konfigurasi teknis dan sistem" }
    ],
    "routes_matrix": [
      {
        "id": "dash-app",
        "category": "Dashboard Global",
        "name": "Dashboard Utama Eksekutif",
        "path": "/app",
        "roles": ["REKTOR", "WAKIL_REKTOR", "DEKAN", "KAPRODI", "DOSEN", "TENDIK", "DIREKTORAT", "PENGELOLA_DATA", "ADMIN"]
      },
      {
        "id": "rek-dash",
        "category": "Modul Rektor",
        "name": "Dashboard Eksekutif Rektor",
        "path": "/rektor/dashboard",
        "roles": ["REKTOR"]
      },
      {
        "id": "rek-akad",
        "category": "Modul Rektor",
        "name": "Analisis Akademik Rektor",
        "path": "/rektor/akademik",
        "roles": ["REKTOR", "WAKIL_REKTOR"]
      },
      {
        "id": "rek-keu",
        "category": "Modul Rektor",
        "name": "Pengawasan Anggaran & Keuangan",
        "path": "/rektor/keuangan",
        "roles": ["REKTOR", "WAKIL_REKTOR"]
      },
      {
        "id": "rek-ai",
        "category": "Modul Rektor",
        "name": "Pusat Kendali Prediksi & AI",
        "path": "/rektor/ai",
        "roles": ["REKTOR", "WAKIL_REKTOR"]
      },
      {
        "id": "adm-usr",
        "category": "Administrator",
        "name": "Manajemen Pengguna & RBAC",
        "path": "/admin/users",
        "roles": ["ADMIN", "PENGELOLA_DATA"]
      },
      {
        "id": "adm-rte",
        "category": "Administrator",
        "name": "Konfigurasi Rute & Modul",
        "path": "/admin/routes",
        "roles": ["ADMIN"]
      },
      {
        "id": "adm-syn",
        "category": "Administrator",
        "name": "Sinkronisasi Data Multi-Sistem",
        "path": "/admin/sinkronisasi",
        "roles": ["ADMIN", "PENGELOLA_DATA"]
      },
      {
        "id": "adm-log",
        "category": "Administrator",
        "name": "Log Sistem & Keamanan",
        "path": "/admin/logs",
        "roles": ["ADMIN"]
      }
    ]
  }
}
```

#### 2.3.2 Update Hak Akses Peran pada Rute
- **Endpoint:** `PUT /api/v1/admin/routes/{route_id}/permissions`
- **Request Body:**
```json
{
  "roles": ["ADMIN", "REKTOR", "WAKIL_REKTOR"]
}
```
##### Response JSON:
```json
{
  "status": "success",
  "message": "Hak akses rute /rek-akad berhasil diperbarui.",
  "data": {
    "route_id": "rek-akad",
    "roles": ["ADMIN", "REKTOR", "WAKIL_REKTOR"]
  }
}
```

---

### 2.4 Integrasi Data & Gateway API (`/admin/integrasi`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/admin/integrasi`
- **Tujuan Halaman:** Mengelola daftar koneksi pihak ketiga (Third-party API & Webhook), status kesehatan koneksi, latensi, dan log riwayat pemanggilan.

#### 2.4.1 Get List Integrasi & Webhook Logs
- **Endpoint:** `GET /api/v1/admin/integrasi`
- **Response JSON:**
```json
{
  "status": "success",
  "message": "Data integrasi dan webhook berhasil dimuat.",
  "data": {
    "integrasi_list": [
      {
        "id": 1,
        "name": "WhatsApp Gateway (Kirimi.id)",
        "category": "Messaging API",
        "status": "Connected",
        "latency": "142 ms",
        "endpoint": "https://api.kirimi.id/v1/webhook",
        "icon": "bi-whatsapp",
        "theme": "success",
        "last_checked": "5 menit lalu"
      },
      {
        "id": 2,
        "name": "Payment Gateway (Midtrans QRIS)",
        "category": "Financial API",
        "status": "Connected",
        "latency": "98 ms",
        "endpoint": "https://api.midtrans.com/v2/charge",
        "icon": "bi-wallet2",
        "theme": "primary",
        "last_checked": "2 menit lalu"
      },
      {
        "id": 3,
        "name": "SISTER Kemendikbud API",
        "category": "Academic Sync",
        "status": "Warning",
        "latency": "1.2 s",
        "endpoint": "https://sister.kemdikbud.go.id/ws",
        "icon": "bi-mortarboard",
        "theme": "warning",
        "last_checked": "10 menit lalu"
      },
      {
        "id": 4,
        "name": "Single Sign-On (Google OAuth)",
        "category": "Authentication",
        "status": "Connected",
        "latency": "65 ms",
        "endpoint": "https://accounts.google.com/o/oauth2",
        "icon": "bi-google",
        "theme": "gold",
        "last_checked": "1 menit lalu"
      },
      {
        "id": 5,
        "name": "PDDIKTI Neo Feeder Kemdiktisaintek",
        "category": "National Database",
        "status": "Connected",
        "latency": "320 ms",
        "endpoint": "http://feeder.hamzanwadi.ac.id:8082/ws/live2.php",
        "icon": "bi-cloud-arrow-down",
        "theme": "success",
        "last_checked": "8 menit lalu"
      }
    ],
    "webhook_logs": [
      {
        "id": "LOG-9921",
        "service": "WhatsApp Gateway (Kirimi.id)",
        "event": "Notifikasi Realisasi Anggaran Rektor",
        "timestamp": "Hari ini, 12:45 WITA",
        "status": "200 OK",
        "latency": "142 ms"
      },
      {
        "id": "LOG-9920",
        "service": "Payment Gateway (Midtrans QRIS)",
        "event": "Notifikasi Pembayaran UKT Sukses",
        "timestamp": "Hari ini, 11:30 WITA",
        "status": "200 OK",
        "latency": "98 ms"
      },
      {
        "id": "LOG-9919",
        "service": "SISTER Kemendikbud API",
        "event": "Tarik Riwayat Fungsional Dosen",
        "timestamp": "Hari ini, 09:15 WITA",
        "status": "504 Gateway Timeout",
        "latency": "5.4 s"
      }
    ]
  }
}
```

#### 2.4.2 Test Connection (Ping Service)
- **Endpoint:** `POST /api/v1/admin/integrasi/{id}/ping`
- **Response JSON:**
```json
{
  "status": "success",
  "message": "Koneksi ke SISTER Kemendikbud API berhasil diverifikasi.",
  "data": {
    "id": 3,
    "status": "Connected",
    "latency": "285 ms",
    "tested_at": "2026-11-03T14:22:10Z"
  }
}
```

#### 2.4.3 Update Konfigurasi Integrasi
- **Endpoint:** `PUT /api/v1/admin/integrasi/{id}/config`
- **Request Body:**
```json
{
  "endpoint": "https://api.kirimi.id/v1/webhook",
  "api_key": "sec_live_99238471928374",
  "timeout_seconds": 30
}
```
##### Response JSON:
```json
{
  "status": "success",
  "message": "Konfigurasi integrasi berhasil disimpan."
}
```

---

### 2.5 Sinkronisasi Data & ETL (`/admin/sinkronisasi`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/admin/sinkronisasi`
- **Tujuan Halaman:** Memantau status berkala sinkronisasi data dari berbagai sistem sumber (SISTER, PDDIKTI, SIAKAD, SIMPEG), memicu sinkronisasi manual, dan mencatat riwayat pemrosesan data.

#### 2.5.1 Get Status Sinkronisasi & Riwayat
- **Endpoint:** `GET /api/v1/admin/sinkronisasi`
- **Response JSON:**
```json
{
  "status": "success",
  "message": "Status sinkronisasi data berhasil dimuat.",
  "data": {
    "entitas_sync": [
      {
        "id": 1,
        "name": "Data Dosen & NIDN",
        "source": "SISTER Kemendikbud",
        "totalRecords": "342 Dosen",
        "lastSync": "Hari ini, 08:30 WITA",
        "status": "Success",
        "icon": "bi-person-badge",
        "theme": "success"
      },
      {
        "id": 2,
        "name": "Data Mahasiswa Aktif (PDDikti)",
        "source": "Feeder PDDikti",
        "totalRecords": "8.432 Mahasiswa",
        "lastSync": "Hari ini, 04:00 WITA",
        "status": "Success",
        "icon": "bi-mortarboard",
        "theme": "primary"
      },
      {
        "id": 3,
        "name": "Kurikulum & Mata Kuliah",
        "source": "SIAKAD Utama",
        "totalRecords": "210 MK",
        "lastSync": "Kemarin, 22:15 WITA",
        "status": "Success",
        "icon": "bi-journal-code",
        "theme": "gold"
      },
      {
        "id": 4,
        "name": "Nilai & Transkrip Akademik",
        "source": "Feeder PDDikti",
        "totalRecords": "45.120 Record",
        "lastSync": "2 hari lalu",
        "status": "Warning",
        "icon": "bi-file-earmark-spreadsheet",
        "theme": "warning"
      }
    ],
    "riwayat_sync": [
      {
        "id": "SYN-8821",
        "entity": "Data Dosen & NIDN",
        "source": "SISTER Kemendikbud",
        "type": "Incremental Sync",
        "startedAt": "03 Nov 2026, 08:30 WITA",
        "duration": "42 detik",
        "recordsProcessed": "12 diperbarui",
        "status": "Success"
      },
      {
        "id": "SYN-8820",
        "entity": "Data Mahasiswa Aktif (PDDikti)",
        "source": "Feeder PDDikti",
        "type": "Full Sync",
        "startedAt": "03 Nov 2026, 04:00 WITA",
        "duration": "4 menit 18 detik",
        "recordsProcessed": "8.432 record",
        "status": "Success"
      },
      {
        "id": "SYN-8819",
        "entity": "Nilai & Transkrip Akademik",
        "source": "Feeder PDDikti",
        "type": "Incremental Sync",
        "startedAt": "01 Nov 2026, 18:00 WITA",
        "duration": "1 menit 05 detik",
        "recordsProcessed": "23 gagal, 1.450 sukses",
        "status": "Warning"
      }
    ]
  }
}
```

#### 2.5.2 Trigger Sinkronisasi Manual
- **Endpoint:** `POST /api/v1/admin/sinkronisasi/trigger`
- **Request Body:**
```json
{
  "entity_id": 1,
  "sync_mode": "incremental"
}
```
##### Response JSON (HTTP 202 Accepted):
```json
{
  "status": "success",
  "message": "Proses sinkronisasi telah dijadwalkan di background worker.",
  "data": {
    "job_id": "JOB-SYNC-99102",
    "entity_name": "Data Dosen & NIDN",
    "status": "Queued",
    "queued_at": "2026-11-03T14:25:00Z"
  }
}
```

---

### 2.6 Kualitas Data & Audit Mutu (`/admin/kualitas`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/admin/kualitas`
- **Tujuan Halaman:** Memantau metrik kualitas data warehouse (kelengkapan, konsistensi, duplikasi), kepatuhan dokumen SPMI, serta temuan audit internal.

#### 2.6.1 Get Ringkasan Kualitas Data & Dokumen Mutu
- **Endpoint:** `GET /api/v1/admin/kualitas`
- **Response JSON:**
```json
{
  "status": "success",
  "message": "Metrik kualitas data dan mutu berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "title": "Capaian IKU Universitas",
        "value": "88.4%",
        "trend": "Target 85% tercapai",
        "icon": "bi-trophy-fill",
        "theme": "card-gold"
      },
      {
        "title": "Audit Mutu Internal (AMI)",
        "value": "100%",
        "trend": "Siklus 2025/2026 Selesai",
        "icon": "bi-clipboard-check-fill",
        "theme": "card-emerald"
      },
      {
        "title": "Temuan Audit (Rekomendasi)",
        "value": "12 Temuan",
        "trend": "8 Selesai, 4 Proses",
        "icon": "bi-exclamation-octagon-fill",
        "theme": "card-purple"
      },
      {
        "title": "Borang Akreditasi Aktif",
        "value": "5 Prodi",
        "trend": "Persiapan LAM / BAN-PT",
        "icon": "bi-file-earmark-text-fill",
        "theme": "card-blue"
      }
    ],
    "dokumen_spmi": [
      {
        "id": "DOC-01",
        "title": "Kebijakan SPMI Universitas Hamzanwadi 2026",
        "category": "Standar Mutu",
        "auditor": "LPM Universitas",
        "status": "Valid",
        "date": "10 Jan 2026"
      },
      {
        "id": "DOC-02",
        "title": "Manual SPMI - Standar Pembelajaran & Penilaian",
        "category": "Manual Mutu",
        "auditor": "LPM Universitas",
        "status": "Valid",
        "date": "15 Feb 2026"
      },
      {
        "id": "DOC-03",
        "title": "Laporan Hasil Audit Mutu Internal Siklus IV",
        "category": "Laporan AMI",
        "auditor": "Tim Auditor Internal",
        "status": "Review",
        "date": "20 Okt 2026"
      }
    ],
    "anomali_data": [
      {
        "id": "ANM-101",
        "tabel": "dim_dosen",
        "isu": "Ditemukan 2 record NIDN duplikat pada dosen NIDK",
        "tingkat": "Tinggi",
        "jumlah_baris": 2,
        "status": "Perlu Validasi"
      },
      {
        "id": "ANM-102",
        "tabel": "fact_mahasiswa_krs",
        "isu": "14 mahasiswa terdaftar aktif tanpa paket KRS semester berjalan",
        "tingkat": "Sedang",
        "jumlah_baris": 14,
        "status": "Investigasi"
      }
    ]
  }
}
```

#### 2.6.2 Trigger Scan Integritas Data
- **Endpoint:** `POST /api/v1/admin/kualitas/scan`
- **Response JSON:**
```json
{
  "status": "success",
  "message": "Pemeriksaan integritas dan kualitas data berhasil dijalankan.",
  "data": {
    "total_checked_records": 128450,
    "issues_found": 16,
    "accuracy_score": "98.7%",
    "scanned_at": "2026-11-03T14:28:00Z"
  }
}
```

---

### 2.7 Log Sistem & Audit Trail (`/admin/logs`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/admin/logs`
- **Tujuan Halaman:** Memantau log audit keamanan, riwayat login, peringatan sistem, error API, dan jejak aktivitas admin.

#### 2.7.1 Get System Logs (Filtered & Paginated)
- **Endpoint:** `GET /api/v1/admin/logs`
- **Query Params:**
  - `page` (integer, default: 1)
  - `per_page` (integer, default: 10)
  - `level` (string: `ALL`, `INFO`, `WARNING`, `ERROR`, `FATAL`)
  - `search` (string, filter action / user / IP)
  - `timeframe` (string: `24h`, `7d`, `30d`)

##### Response JSON:
```json
{
  "status": "success",
  "message": "Log sistem berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "title": "Total Log (24 Jam)",
        "value": "1.482",
        "trend": "+8.2% dari kemarin",
        "icon": "bi-journal-text",
        "theme": "card-blue"
      },
      {
        "title": "Log Aktivitas (Info)",
        "value": "1.420",
        "trend": "Normal operation",
        "icon": "bi-info-circle-fill",
        "theme": "card-emerald"
      },
      {
        "title": "Peringatan (Warning)",
        "value": "48",
        "trend": "Perlu pengawasan",
        "icon": "bi-exclamation-triangle-fill",
        "theme": "card-gold"
      },
      {
        "title": "Kesalahan (Error)",
        "value": "14",
        "trend": "Tercatat di server utama",
        "icon": "bi-x-octagon-fill",
        "theme": "card-purple"
      }
    ],
    "logs": [
      {
        "id": "LOG-5091",
        "timestamp": "27 Okt 2026, 14:20 WITA",
        "level": "INFO",
        "user": "rektor@hamzanwadi.ac.id",
        "action": "Login berhasil ke Dashboard Eksekutif",
        "ip": "192.168.1.45"
      },
      {
        "id": "LOG-5090",
        "timestamp": "27 Okt 2026, 14:15 WITA",
        "level": "WARNING",
        "user": "admin@hamzanwadi.ac.id",
        "action": "Percobaan akses modul terbatas tanpa izin",
        "ip": "203.0.113.88"
      },
      {
        "id": "LOG-5089",
        "timestamp": "27 Okt 2026, 13:50 WITA",
        "level": "ERROR",
        "user": "system@feeder",
        "action": "Koneksi timeout ke API Feeder PDDikti",
        "ip": "10.0.0.15"
      },
      {
        "id": "LOG-5088",
        "timestamp": "27 Okt 2026, 12:30 WITA",
        "level": "INFO",
        "user": "akademik@hamzanwadi.ac.id",
        "action": "Pembaruan status akreditasi S1 Informatika",
        "ip": "192.168.1.20"
      },
      {
        "id": "LOG-5087",
        "timestamp": "27 Okt 2026, 11:10 WITA",
        "level": "INFO",
        "user": "keuangan@hamzanwadi.ac.id",
        "action": "Ekspor laporan pembayaran SPP Q3",
        "ip": "192.168.1.12"
      }
    ]
  },
  "pagination": {
    "current_page": 1,
    "per_page": 10,
    "total_records": 1482,
    "total_pages": 149
  }
}
```

---

### 2.8 Konfigurasi Sistem (`/admin/konfigurasi`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/admin/konfigurasi`
- **Tujuan Halaman:** Mengelola setelan umum instansi, kebijakan keamanan & sesi, kredensial SMTP mailer, dan parameter pemeliharaan sistem.

#### 2.8.1 Get All Configuration Sections
- **Endpoint:** `GET /api/v1/admin/konfigurasi`
- **Response JSON:**
```json
{
  "status": "success",
  "message": "Konfigurasi sistem berhasil dimuat.",
  "data": {
    "umum": {
      "appName": "HARMONI Universitas Hamzanwadi",
      "appTagline": "Pusat Kendali Mutu & Akademik Terpadu",
      "adminEmail": "admin@hamzanwadi.ac.id",
      "academicYear": "2025/2026 Ganjil",
      "timezone": "WITA (Asia/Makassar)"
    },
    "keamanan": {
      "sessionTimeout": "120",
      "maxLoginAttempts": "5",
      "enable2FA": true,
      "forceHttps": true
    },
    "email": {
      "mailHost": "smtp.mailtrap.io",
      "mailPort": "587",
      "mailUsername": "app_harmoni_user",
      "mailEncryption": "TLS",
      "senderName": "Sistem Informasi HARMONI"
    },
    "sistem": {
      "maintenanceMode": false,
      "debugMode": false,
      "backupFrequency": "Harian"
    }
  }
}
```

#### 2.8.2 Update Konfigurasi per Bagian
- **Endpoint:** `PUT /api/v1/admin/konfigurasi/{section}`
- **Contoh Section:** `umum`, `keamanan`, `email`, `sistem`
- **Request Body (untuk `section=umum`):**
```json
{
  "appName": "HARMONI Universitas Hamzanwadi",
  "appTagline": "Pusat Kendali Mutu & Akademik Terpadu",
  "adminEmail": "admin@hamzanwadi.ac.id",
  "academicYear": "2025/2026 Ganjil",
  "timezone": "WITA (Asia/Makassar)"
}
```
##### Response JSON:
```json
{
  "status": "success",
  "message": "Konfigurasi bagian umum berhasil disimpan."
}
```

#### 2.8.3 Test Kirim Email SMTP
- **Endpoint:** `POST /api/v1/admin/konfigurasi/test-email`
- **Request Body:**
```json
{
  "target_email": "test-admin@hamzanwadi.ac.id"
}
```
##### Response JSON:
```json
{
  "status": "success",
  "message": "Email percobaan berhasil dikirim ke test-admin@hamzanwadi.ac.id."
}
```

---

### 2.9 Monitoring Sistem & Server (`/admin/monitoring`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/admin/monitoring`
- **Tujuan Halaman:** Real-time health check server (CPU, RAM, Disk, Uptime), status service engine (Nginx, MariaDB, Redis, Worker), dan performa throughput aplikasi.

#### 2.9.1 Get System Health & Performance
- **Endpoint:** `GET /api/v1/admin/monitoring`
- **Response JSON:**
```json
{
  "status": "success",
  "message": "Metrik monitoring server berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "title": "Penggunaan CPU",
        "value": "34.2%",
        "status": "Normal (4 Core)",
        "icon": "bi-cpu-fill",
        "theme": "card-blue"
      },
      {
        "title": "Memori RAM",
        "value": "6.2 GB / 16 GB",
        "status": "38.7% Terpakai",
        "icon": "bi-memory",
        "theme": "card-emerald"
      },
      {
        "title": "Penyimpanan Disk",
        "value": "112 GB / 500 GB",
        "status": "22.4% Terpakai",
        "icon": "bi-hdd-fill",
        "theme": "card-gold"
      },
      {
        "title": "Uptime Server",
        "value": "99.98%",
        "status": "18 Hari, 4 Jam Online",
        "icon": "bi-shield-check",
        "theme": "card-purple"
      }
    ],
    "services": [
      {
        "id": 1,
        "name": "Web Server (Nginx / Vite)",
        "port": "80 / 5173",
        "status": "Running",
        "pid": "PID 4821",
        "uptime": "18 hari"
      },
      {
        "id": 2,
        "name": "Database Engine (PostgreSQL / MariaDB)",
        "port": "5432 / 3306",
        "status": "Running",
        "pid": "PID 1102",
        "uptime": "18 hari"
      },
      {
        "id": 3,
        "name": "Background Queue Worker (Redis)",
        "port": "Redis 6379",
        "status": "Running",
        "pid": "PID 3320",
        "uptime": "5 hari"
      },
      {
        "id": 4,
        "name": "AI Inference Engine (Python Fastify)",
        "port": "8081",
        "status": "Running",
        "pid": "PID 5501",
        "uptime": "12 hari"
      }
    ],
    "network_metrics": {
      "requests_per_second": 48.5,
      "average_latency_ms": 68.2,
      "error_rate_percent": 0.04,
      "active_db_connections": 24
    }
  }
}
```

---

## 3. MODUL EKSEKUTIF REKTOR

Modul ini digunakan oleh **Rektor**, **Wakil Rektor**, dan jajaran pimpinan universitas untuk analisis strategis multi-fakultas, pengawasan indikator kinerja institusi, serta pemanfaatan analitik prediktif AI.

---

### 3.1 Dashboard Eksekutif (`/rektor/dashboard`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/dashboard`
- **Tujuan Halaman:** Pusat kendali eksekutif menyajikan 4 kartu metrik utama universitas, grafik tren pendaftaran mahasiswa baru 12 bulan, dan log aksi kegiatan pimpinan.
- **Endpoint Utama:** `GET /api/v1/rektor/dashboard`
- **Headers:** `Authorization: Bearer <token>`

#### Response JSON:
```json
{
  "status": "success",
  "message": "Data dashboard eksekutif Rektor berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "title": "Total Mahasiswa Aktif",
        "value": "8.432",
        "trend": "+5.2% dari tahun lalu",
        "icon": "bi-people-fill",
        "theme": "gold",
        "badgeBg": "bg-gold-subtle text-gold"
      },
      {
        "title": "Akreditasi Institusi",
        "value": "Unggul",
        "trend": "Valid hingga 2028",
        "icon": "bi-shield-check",
        "theme": "emerald",
        "badgeBg": "bg-emerald-subtle text-emerald"
      },
      {
        "title": "Serapan Anggaran (Q3)",
        "value": "78%",
        "trend": "Optimal & On-track",
        "icon": "bi-pie-chart-fill",
        "theme": "blue",
        "badgeBg": "bg-blue-subtle text-blue"
      },
      {
        "title": "Publikasi Jurnal Ilmiah",
        "value": "156",
        "trend": "+12% semester ini",
        "icon": "bi-journal-bookmark-fill",
        "theme": "purple",
        "badgeBg": "bg-purple-subtle text-purple"
      }
    ],
    "chart_tren": {
      "labels": ["Jan", "Feb", "Mar", "Apr", "Mei", "Jun", "Jul", "Ags", "Sep", "Okt", "Nov", "Des"],
      "datasets": [
        {
          "label": "Pendaftar Baru",
          "borderColor": "#b8860b",
          "backgroundColor": "rgba(184, 134, 11, 0.08)",
          "fill": true,
          "tension": 0.4,
          "borderWidth": 2.5,
          "data": [150, 220, 380, 520, 850, 1300, 1600, 1150, 700, 350, 250, 180]
        },
        {
          "label": "Mahasiswa Diterima",
          "borderColor": "#10b981",
          "backgroundColor": "rgba(16, 185, 129, 0.05)",
          "fill": true,
          "tension": 0.4,
          "borderWidth": 2.5,
          "data": [60, 90, 150, 230, 400, 580, 720, 550, 280, 120, 90, 70]
        }
      ]
    },
    "aktivitas_terkini": [
      {
        "id": 1,
        "action": "Persetujuan Nota Kesepahaman (MoU) Mitra Internasional UTM Malaysia",
        "time": "15 menit yang lalu",
        "type": "approval",
        "badge": "MoU"
      },
      {
        "id": 2,
        "action": "Evaluasi Anggaran Triwulanan Bersama Wakil Rektor II",
        "time": "2 jam yang lalu",
        "type": "system",
        "badge": "Keuangan"
      },
      {
        "id": 3,
        "action": "Penerbitan SK Kebijakan Kurikulum MBKM Mandiri 2026",
        "time": "5 jam yang lalu",
        "type": "doc",
        "badge": "Akademik"
      }
    ],
    "shortcuts": [
      { "title": "Validasi SK Mutu", "path": "/rektor/mutu", "icon": "bi-patch-check" },
      { "title": "Monitor Anggaran", "path": "/rektor/keuangan", "icon": "bi-cash-stack" },
      { "title": "Analisis Prediktif AI", "path": "/rektor/ai", "icon": "bi-cpu" },
      { "title": "Arsip Laporan Kinerja", "path": "/rektor/laporan", "icon": "bi-journal-richtext" }
    ]
  }
}
```

---

### 3.2 Kinerja Akademik (`/rektor/akademik`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/akademik`
- **Tujuan Halaman:** Menampilkan data mahasiswa aktif, rata-rata IPK universitas, sebaran proporsi mahasiswa per fakultas, serta status akreditasi per program studi.
- **Endpoint Utama:** `GET /api/v1/rektor/akademik`

#### Response JSON:
```json
{
  "status": "success",
  "message": "Data akademik eksekutif berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "title": "Total Mahasiswa Aktif",
        "value": "8.432",
        "note": "Rasio Ideal Terkendali",
        "icon": "bi-mortarboard-fill",
        "theme": "card-gold"
      },
      {
        "title": "Rata-rata IPK Universitas",
        "value": "3.38",
        "note": "+0.04 dari Periode Lalu",
        "icon": "bi-graph-up-arrow",
        "theme": "card-emerald"
      },
      {
        "title": "Prodi Unggul",
        "value": "14 Prodi",
        "note": "42% dari Total Program",
        "icon": "bi-patch-check-fill",
        "theme": "card-blue"
      },
      {
        "title": "Beban Dosen (Rasio)",
        "value": "1 : 22",
        "note": "Sesuai Standar SNDikti",
        "icon": "bi-people-fill",
        "theme": "card-purple"
      }
    ],
    "chart_akreditasi_fakultas": {
      "labels": ["Fakultas Teknik", "Fakultas Keguruan", "Fakultas Agama Islam", "Fakultas Kesehatan", "Fakultas Ekonomi"],
      "datasets": [
        {
          "label": "Unggul / A",
          "backgroundColor": "#0f172a",
          "borderRadius": 8,
          "barThickness": 20,
          "data": [4, 6, 3, 2, 3]
        },
        {
          "label": "Baik Sekali / B",
          "backgroundColor": "#b8860b",
          "borderRadius": 8,
          "barThickness": 20,
          "data": [2, 1, 2, 3, 2]
        }
      ]
    },
    "chart_sebaran_mahasiswa": {
      "labels": ["Fakultas Keguruan", "Fakultas Teknik", "Fakultas Kesehatan", "Fakultas Ekonomi", "Fakultas Agama Islam"],
      "datasets": [
        {
          "backgroundColor": ["#0f172a", "#b8860b", "#0d6efd", "#10b981", "#8b5cf6"],
          "borderWidth": 0,
          "data": [35, 25, 18, 12, 10]
        }
      ]
    },
    "tabel_prodi": [
      {
        "no": "01",
        "prodi": "S1 Informatika",
        "fakultas": "Fakultas Teknik",
        "akreditasi": "Unggul (A)",
        "validUntil": "2028",
        "status": "Aktif"
      },
      {
        "no": "02",
        "prodi": "S1 Sistem Informasi",
        "fakultas": "Fakultas Teknik",
        "akreditasi": "Baik Sekali (B)",
        "validUntil": "2027",
        "status": "Aktif"
      },
      {
        "no": "03",
        "prodi": "S1 Pendidikan Teknologi Informasi",
        "fakultas": "Fakultas Keguruan",
        "akreditasi": "Baik Sekali (B)",
        "validUntil": "2028",
        "status": "Aktif"
      },
      {
        "no": "04",
        "prodi": "S1 Manajemen",
        "fakultas": "Fakultas Ekonomi",
        "akreditasi": "Unggul (A)",
        "validUntil": "2029",
        "status": "Aktif"
      }
    ]
  }
}
```

---

### 3.3 Kemahasiswaan & Alumni (`/rektor/kemahasiswaan`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/kemahasiswaan`
- **Tujuan Halaman:** Analisis capaian prestasi mahasiswa tingkat nasional/internasional, sebaran distribusi beasiswa (KIP-K, Yayasan), keaktifan Ormawa/UKM, serta agenda kegiatan lomba.
- **Endpoint Utama:** `GET /api/v1/rektor/kemahasiswaan`

#### Response JSON:
```json
{
  "status": "success",
  "message": "Data kemahasiswaan berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "title": "Total Mahasiswa Aktif",
        "value": "8.432",
        "trend": "Terdaftar PDDikti",
        "icon": "bi-people-fill",
        "color": "text-primary",
        "bg": "bg-primary-soft"
      },
      {
        "title": "Prestasi Mahasiswa (2026)",
        "value": "142 Juara",
        "trend": "Nasional & Internasional",
        "icon": "bi-trophy-fill",
        "color": "text-gold",
        "bg": "bg-gold-soft"
      },
      {
        "title": "Penerima Beasiswa",
        "value": "1.850 Mhs",
        "trend": "KIP-Kuliah & Yayasan",
        "icon": "bi-award-fill",
        "color": "text-success",
        "bg": "bg-success-soft"
      },
      {
        "title": "Aktivitas Ormawa/UKM",
        "value": "38 Unit",
        "trend": "Aktif Berjalan",
        "icon": "bi-kanban-fill",
        "color": "text-info",
        "bg": "bg-info-soft"
      }
    ],
    "chart_tren_prestasi": {
      "labels": ["Jan", "Feb", "Mar", "Apr", "Mei", "Jun", "Jul", "Ags", "Sep", "Okt", "Nov", "Des"],
      "datasets": [
        {
          "label": "Tingkat Nasional",
          "backgroundColor": "#b8860b",
          "borderRadius": 6,
          "data": [8, 12, 15, 10, 18, 22, 14, 16, 20, 15, 12, 10]
        },
        {
          "label": "Tingkat Internasional",
          "backgroundColor": "#198754",
          "borderRadius": 6,
          "data": [2, 3, 5, 4, 6, 8, 5, 7, 9, 6, 4, 3]
        }
      ]
    },
    "chart_distribusi_beasiswa": {
      "labels": ["KIP-Kuliah", "Beasiswa Yayasan", "Beasiswa Pemerintah Daerah", "Beasiswa Prestasi / Tahfidz"],
      "datasets": [
        {
          "backgroundColor": ["#b8860b", "#0d6efd", "#198754", "#6f42c1"],
          "data": [45, 25, 20, 10]
        }
      ]
    },
    "tabel_agenda": [
      {
        "no": 1,
        "kegiatan": "Pekan Olahraga Mahasiswa Nasional (POMNAS)",
        "ormawa": "BEM Universitas",
        "tanggal": "12-18 Nov 2026",
        "peserta": "120 Atlet",
        "status": "Persiapan"
      },
      {
        "no": 2,
        "kegiatan": "National University Debating Championship (NUDC)",
        "ormawa": "UKM Bahasa Inggris",
        "tanggal": "04-06 Des 2026",
        "peserta": "12 Mahasiswa",
        "status": "Terdaftar"
      },
      {
        "no": 3,
        "kegiatan": "Pameran Inovasi Teknologi Mahasiswa",
        "ormawa": "HIMATIF Teknik",
        "tanggal": "20-22 Des 2026",
        "peserta": "45 Tim",
        "status": "Ditinjau"
      }
    ]
  }
}
```

---

### 3.4 SDM & Kepegawaian (`/rektor/sdm`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/sdm`
- **Tujuan Halaman:** Pemantauan profil dosen tetap & tendik, persentase kelulusan Serdos, rasio dosen-mahasiswa, grafik distribusi jenjang jabatan fungsional, dan direktori pejabat struktural universitas.
- **Endpoint Utama:** `GET /api/v1/rektor/sdm`

#### Response JSON:
```json
{
  "status": "success",
  "message": "Data SDM kepegawaian berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "label": "Total Dosen Tetap",
        "count": "312",
        "trend": "+4% vs tahun lalu",
        "trendType": "up",
        "icon": "bi-person-badge",
        "theme": "gold"
      },
      {
        "label": "Tenaga Kependidikan",
        "count": "100",
        "trend": "Stabil",
        "trendType": "neutral",
        "icon": "bi-people",
        "theme": "indigo"
      },
      {
        "label": "Tersertifikasi (Serdos)",
        "count": "78%",
        "trend": "Target Tercapai",
        "trendType": "up",
        "icon": "bi-patch-check",
        "theme": "emerald"
      },
      {
        "label": "Rasio Dosen : Mhs",
        "count": "1 : 21",
        "trend": "Ideal SNikti",
        "trendType": "up",
        "icon": "bi-pie-chart",
        "theme": "violet"
      }
    ],
    "chart_jafung": {
      "labels": ["Asisten Ahli", "Lektor", "Lektor Kepala", "Guru Besar", "Tenaga Pengajar"],
      "datasets": [
        {
          "label": "Jumlah Dosen",
          "backgroundColor": "#c59b27",
          "hoverBackgroundColor": "#b8860b",
          "borderRadius": 8,
          "barThickness": 22,
          "data": [95, 140, 55, 12, 10]
        }
      ]
    },
    "pejabat_strategis": [
      {
        "name": "Dr. H. Muhammad, M.Pd.",
        "title": "Dekan Fakultas Keguruan & Ilmu Pendidikan",
        "nip": "19820312 200501 1 003",
        "jafung": "Lektor Kepala",
        "education": "S3 (Doktor)",
        "status": "Aktif"
      },
      {
        "name": "Ir. Ahmad Zulkarnain, M.T.",
        "title": "Dekan Fakultas Teknik",
        "nip": "19790521 200312 1 002",
        "jafung": "Lektor",
        "education": "S2 (Magister)",
        "status": "Aktif"
      },
      {
        "name": "Dr. Siti Aminah, S.H., M.H.",
        "title": "Kaprodi Ilmu Hukum",
        "nip": "19851004 201012 2 001",
        "jafung": "Lektor Kepala",
        "education": "S3 (Doktor)",
        "status": "Aktif"
      }
    ]
  }
}
```

---

### 3.5 Keuangan & Anggaran (`/rektor/keuangan`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/keuangan`
- **Tujuan Halaman:** Monitoring pagu anggaran tahun berjalan, serapan realisasi triwulan, target kumulatif vs realisasi aktual, serta efisiensi anggaran per fakultas/unit kerja.
- **Endpoint Utama:** `GET /api/v1/rektor/keuangan`

#### Response JSON:
```json
{
  "status": "success",
  "message": "Data keuangan dan anggaran eksekutif berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "label": "Total Pagu Anggaran",
        "count": "Rp 42.5 M",
        "trend": "Tahun Anggaran 2026",
        "icon": "bi-wallet2",
        "theme": "gold"
      },
      {
        "label": "Realisasi Serapan (Q3)",
        "count": "Rp 33.1 M",
        "trend": "78% Terserap",
        "trendType": "up",
        "icon": "bi-graph-up-arrow",
        "theme": "emerald"
      },
      {
        "label": "Sisa Anggaran (Sisa Pagu)",
        "count": "Rp 9.4 M",
        "trend": "Aman / Terkendali",
        "icon": "bi-shield-check",
        "theme": "blue"
      },
      {
        "label": "Pendapatan UKT / Non-UKT",
        "count": "Rp 28.6 M",
        "trend": "+12% vs lalu",
        "trendType": "up",
        "icon": "bi-cash-stack",
        "theme": "violet"
      }
    ],
    "chart_serapan": {
      "labels": ["Jan", "Feb", "Mar", "Apr", "Mei", "Jun", "Jul", "Ags", "Sep", "Okt", "Nov", "Des"],
      "datasets": [
        {
          "label": "Target Serapan (%)",
          "borderColor": "#94a3b8",
          "borderDash": [4, 4],
          "backgroundColor": "transparent",
          "tension": 0.3,
          "data": [10, 20, 30, 40, 50, 60, 70, 80, 90, 95, 98, 100]
        },
        {
          "label": "Realisasi Aktual (%)",
          "borderColor": "#b8860b",
          "backgroundColor": "rgba(184, 134, 11, 0.08)",
          "fill": true,
          "tension": 0.3,
          "data": [8, 18, 28, 42, 54, 62, 78, 82, 88, 0, 0, 0]
        }
      ]
    },
    "tabel_unit_anggaran": [
      {
        "no": 1,
        "unit": "Fakultas Teknik",
        "pagu": "Rp 8.500.000.000",
        "realisasi": "Rp 7.100.000.000",
        "persentase": "83.5%",
        "status": "Optimal"
      },
      {
        "no": 2,
        "unit": "Fakultas Keguruan & Ilmu Pendidikan",
        "pagu": "Rp 10.200.000.000",
        "realisasi": "Rp 8.200.000.000",
        "persentase": "80.3%",
        "status": "Optimal"
      },
      {
        "no": 3,
        "unit": "Fakultas Agama Islam",
        "pagu": "Rp 5.400.000.000",
        "realisasi": "Rp 4.100.000.000",
        "persentase": "75.9%",
        "status": "Perlu Akselerasi"
      },
      {
        "no": 4,
        "unit": "Lembaga Penelitian & Pengabdian (LPPM)",
        "pagu": "Rp 6.000.000.000",
        "realisasi": "Rp 5.100.000.000",
        "persentase": "85.0%",
        "status": "Optimal"
      }
    ]
  }
}
```

---

### 3.6 Penelitian & Pengabdian (`/rektor/penelitian`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/penelitian`
- **Tujuan Halaman:** Pemantauan capaian tridarma riset, total dana hibah berjalan, publikasi terindeks internasional (Scopus/WoS), perolehan paten HKI, serta daftar penelitian strategis.
- **Endpoint Utama:** `GET /api/v1/rektor/penelitian`

#### Response JSON:
```json
{
  "status": "success",
  "message": "Data penelitian dan pengabdian berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "label": "Total Hibah Berjalan",
        "count": "Rp 3.8 M",
        "trend": "58 Judul Riset Aktif",
        "icon": "bi-journal-check",
        "theme": "gold"
      },
      {
        "label": "Publikasi Scopus/WOS",
        "count": "86 Paper",
        "trend": "Q1 & Q2 Terindeks",
        "icon": "bi-award",
        "theme": "emerald"
      },
      {
        "label": "Perolehan Paten & HKI",
        "count": "24 HKI",
        "trend": "+6 dari Semester Lalu",
        "icon": "bi-shield-check",
        "theme": "blue"
      },
      {
        "label": "Dana Hibah Eksternal",
        "count": "Rp 2.6 M",
        "trend": "Kemdiktisaintek & DUDI",
        "icon": "bi-cash-coin",
        "theme": "violet"
      }
    ],
    "chart_sebaran_fakultas": {
      "labels": ["Fakultas Teknik", "Fakultas Keguruan", "Fakultas Agama Islam", "Fakultas Kesehatan", "Fakultas Ekonomi"],
      "datasets": [
        {
          "label": "Jumlah Judul Hibah",
          "backgroundColor": "#b8860b",
          "borderRadius": 8,
          "data": [18, 22, 8, 12, 10]
        }
      ]
    },
    "tabel_penelitian": [
      {
        "no": 1,
        "judul": "Pengembangan Sistem Deteksi Dini Putus Studi Mahasiswa Berbasis Machine Learning",
        "ketua": "Dr. Irwan Hamzani, M.Pd.",
        "fakultas": "Fakultas Keguruan",
        "skema": "PTUPT Kemdiktisaintek",
        "dana": "Rp 65.000.000",
        "status": "Selesai"
      },
      {
        "no": 2,
        "judul": "Pemanfaatan Energi Terbarukan Sel Surya pada Irigasi Pertanian NTB",
        "ketua": "Ir. Ahmad Zulkarnain, M.T.",
        "fakultas": "Fakultas Teknik",
        "skema": "Penelitian Terapan Unggulan",
        "dana": "Rp 50.000.000",
        "status": "Berjalan"
      },
      {
        "no": 3,
        "judul": "Pemberdayaan UMKM Halal Berbasis Komunitas Lokal Lombok Timur",
        "ketua": "Dr. Siti Aminah, M.H.",
        "fakultas": "Fakultas Agama Islam",
        "skema": "Pengabdian Kemitraan Masyarakat",
        "dana": "Rp 35.000.000",
        "status": "Berjalan"
      }
    ]
  }
}
```

---

### 3.7 Kerja Sama & Kemitraan (`/rektor/kerjasama`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/kerjasama`
- **Tujuan Halaman:** Pemetaan jejaring kemitraan universitas, rincian bentuk kerja sama (MBKM, riset kolaboratif, pertukaran pelajar), serta direktori dokumen MoU/MoA aktif.
- **Endpoint Utama:** `GET /api/v1/rektor/kerjasama`

#### Response JSON:
```json
{
  "status": "success",
  "message": "Data kemitraan dan kerja sama berhasil dimuat.",
  "data": {
    "total_mitra": 85,
    "kategori_mitra": [
      { "name": "Mitra Industri / DUDI", "count": "34 Mitra" },
      { "name": "Perguruan Tinggi", "count": "25 Mitra" },
      { "name": "Instansi Pemerintah", "count": "16 Mitra" },
      { "name": "Mitra Luar Negeri", "count": "10 Mitra" }
    ],
    "bentuk_kerjasama": [
      {
        "name": "Magang & Penempatan Kerja (MBKM)",
        "percentage": 30,
        "count": "25 Mitra",
        "color": "bg-success"
      },
      {
        "name": "Riset Kolaboratif & Publikasi",
        "percentage": 25,
        "count": "21 Mitra",
        "color": "bg-warning"
      },
      {
        "name": "Student & Lecturer Exchange",
        "percentage": 20,
        "count": "16 Mitra",
        "color": "bg-primary"
      },
      {
        "name": "Pengabdian Masyarakat Kolaboratif",
        "percentage": 10,
        "count": "10 Mitra",
        "color": "bg-info"
      }
    ],
    "tabel_mitra": [
      {
        "no": 1,
        "instansi": "PT Telkom Indonesia Witel NTB",
        "kategori": "Industri",
        "lingkup": "Nasional",
        "dokumen": "MoU & MoA",
        "nomorSk": "MoU/UNHAM/2024/011",
        "validUntil": "2027-05-10",
        "status": "Aktif"
      },
      {
        "no": 2,
        "instansi": "Universiti Teknologi Malaysia (UTM)",
        "kategori": "Perguruan Tinggi",
        "lingkup": "Internasional",
        "dokumen": "MoU",
        "nomorSk": "MoU/UNHAM/2025/002",
        "validUntil": "2030-01-15",
        "status": "Aktif"
      },
      {
        "no": 3,
        "instansi": "Pemerintah Daerah Lombok Timur",
        "kategori": "Pemerintah",
        "lingkup": "Daerah",
        "dokumen": "MoA",
        "nomorSk": "MoA/UNHAM/2024/089",
        "validUntil": "2028-11-20",
        "status": "Aktif"
      }
    ]
  }
}
```

---

### 3.8 Penjaminan Mutu & Akreditasi (`/rektor/mutu`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/mutu`
- **Tujuan Halaman:** Evaluasi indeks mutu institusi, ketercapaian 8 standar SPMI nasional, penyelesaian temuan Audit Mutu Internal (AMI), dan skor kepuasan stakeholder.
- **Endpoint Utama:** `GET /api/v1/rektor/mutu`

#### Response JSON:
```json
{
  "status": "success",
  "message": "Data penjaminan mutu dan akreditasi berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "label": "Indeks Mutu Institusi",
        "value": "3.85",
        "note": "Skala 4.00 (Unggul)",
        "icon": "bi-award",
        "badge": "Optimal",
        "theme": "gold"
      },
      {
        "label": "Siklus SPMI Aktif",
        "value": "Siklus IV",
        "note": "Evaluasi Tahap III",
        "icon": "bi-arrow-repeat-circle",
        "badge": "Berjalan",
        "theme": "dark"
      },
      {
        "label": "Penyelesaian Temuan AMI",
        "value": "94%",
        "note": "Tepat Waktu",
        "icon": "bi-patch-check",
        "badge": "Tinggi",
        "theme": "emerald"
      },
      {
        "label": "Kepuasan Stakeholder",
        "value": "4.65",
        "note": "Skala 5.00",
        "icon": "bi-graph-up-arrow",
        "badge": "Excellent",
        "theme": "purple"
      }
    ],
    "standar_spmi": [
      {
        "code": "ST-01",
        "name": "Standar Kompetensi Lulusan",
        "score": "98%",
        "status": "Tercapai",
        "lead": "Dr. H. Muhammad"
      },
      {
        "code": "ST-02",
        "name": "Standar Isi Pembelajaran",
        "score": "95%",
        "status": "Tercapai",
        "lead": "Ir. Ahmad Zulkarnain"
      },
      {
        "code": "ST-03",
        "name": "Standar Proses Pembelajaran",
        "score": "92%",
        "status": "Tercapai",
        "lead": "Dr. Siti Aminah"
      },
      {
        "code": "ST-04",
        "name": "Standar Penilaian Mahasiswa",
        "score": "96%",
        "status": "Tercapai",
        "lead": "Rian Hidayat, M.Kom."
      }
    ],
    "tabel_audit_ami": [
      {
        "id": "AMI-01",
        "unit": "Fakultas Teknik",
        "auditor": "Tim Auditor I (Dr. Hartono)",
        "findings": 2,
        "status": "Selesai 100%",
        "type": "success"
      },
      {
        "id": "AMI-02",
        "unit": "Fakultas Keguruan & Ilmu Pendidikan",
        "auditor": "Tim Auditor II (Dr. Siti Aminah)",
        "findings": 4,
        "status": "Dalam Proses 75%",
        "type": "warning"
      },
      {
        "id": "AMI-03",
        "unit": "Biro Administrasi Akademik",
        "auditor": "Tim Auditor III (Ir. Hasanuddin)",
        "findings": 1,
        "status": "Selesai 100%",
        "type": "success"
      }
    ]
  }
}
```

---

### 3.9 Sarana & Prasarana (`/rektor/sarpras`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/sarpras`
- **Tujuan Halaman:** Monitoring aset fisik kampus, ruang kuliah/laboratorium, tingkat utilisasi beban zonasi gedung, serta pemeliharaan fasilitas.
- **Endpoint Utama:** `GET /api/v1/rektor/sarpras`

#### Response JSON:
```json
{
  "status": "success",
  "message": "Data sarana dan prasarana berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "label": "Total Aset Fisik",
        "value": "2.480",
        "unit": "Unit",
        "desc": "Inventaris Kampus",
        "icon": "bi-box-seam",
        "theme": "gold"
      },
      {
        "label": "Gedung & Fasilitas",
        "value": "24",
        "unit": "Gedung",
        "desc": "Utama & Penunjang",
        "icon": "bi-buildings",
        "theme": "blue"
      },
      {
        "label": "Ruang Kuliah & Lab",
        "value": "130",
        "unit": "Ruang",
        "desc": "Utilisasi 92%",
        "icon": "bi-door-open",
        "theme": "emerald"
      },
      {
        "label": "Pemeliharaan",
        "value": "14",
        "unit": "Unit",
        "desc": "Dalam Servis / Perbaikan",
        "icon": "bi-tools",
        "theme": "purple"
      }
    ],
    "zonasi_kampus": [
      {
        "zone": "Zona A - Rektorat & Biro Pusat",
        "buildings": 4,
        "status": "Optimal",
        "load": "85%"
      },
      {
        "zone": "Zona B - Fakultas Teknik & Lab Komputer",
        "buildings": 6,
        "status": "Padat",
        "load": "96%"
      },
      {
        "zone": "Zona C - FKIP & Gedung Kuliah Bersama",
        "buildings": 8,
        "status": "Optimal",
        "load": "90%"
      },
      {
        "zone": "Zona D - FAI & Gelanggang Mahasiswa",
        "buildings": 6,
        "status": "Stabil",
        "load": "72%"
      }
    ],
    "tabel_aset": [
      {
        "tag": "AST-IT-992",
        "item": "Projektor Epson EB-X400",
        "location": "Lab Komputer 3 (Teknik)",
        "custodian": "Fakultas Teknik",
        "condition": "Sangat Baik"
      },
      {
        "tag": "AST-HV-104",
        "item": "AC Daikin 2 PK Inverter",
        "location": "Ruang Sidang Utama Rektorat",
        "custodian": "Biro Umum",
        "condition": "Perawatan"
      },
      {
        "tag": "AST-LB-551",
        "item": "Mikroskop Binokuler Optika B-150",
        "location": "Lab Biologi FKIP",
        "custodian": "Fakultas Keguruan",
        "condition": "Sangat Baik"
      }
    ]
  }
}
```

---

### 3.10 Laporan Eksekutif (`/rektor/laporan`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/laporan`
- **Tujuan Halaman:** Arsip dokumen laporan strategis tahunan, laporan triwulanan rektor, borang LED akreditasi, dan tautan unduhan formal (PDF/XLSX).

#### 3.10.1 Get Daftar Arsip Laporan
- **Endpoint:** `GET /api/v1/rektor/laporan`
- **Response JSON:**
```json
{
  "status": "success",
  "message": "Daftar arsip laporan eksekutif berhasil dimuat.",
  "data": {
    "kpi_cards": [
      {
        "label": "Total Arsip Laporan",
        "value": "30 Dokumen",
        "trend": "Terarsip Digital",
        "icon": "bi-journal-richtext",
        "theme": "gold"
      },
      {
        "label": "Laporan Triwulanan",
        "value": "12 Dokumen",
        "trend": "Periode 2026",
        "icon": "bi-file-earmark-bar-graph",
        "theme": "blue"
      },
      {
        "label": "Borang Akreditasi",
        "value": "8 Dokumen",
        "trend": "LED & SPMI",
        "icon": "bi-shield-check",
        "theme": "emerald"
      },
      {
        "label": "Laporan Keuangan",
        "value": "6 Dokumen",
        "trend": "Audit Akuntabilitas",
        "icon": "bi-wallet2",
        "theme": "purple"
      }
    ],
    "chart_kategori": {
      "labels": ["Eksekutif & Kinerja", "Mutu & Akreditasi", "Keuangan & Hibah", "SDM & Dosen", "Kerjasama & Mitra"],
      "datasets": [
        {
          "label": "Jumlah Dokumen Laporan",
          "backgroundColor": "#b8860b",
          "borderRadius": 8,
          "barThickness": 24,
          "data": [8, 7, 6, 5, 4]
        }
      ]
    },
    "daftar_laporan": [
      {
        "id": "REP-2026-01",
        "title": "Laporan Kinerja Rektor Triwulan III 2026",
        "category": "Eksekutif",
        "date": "15 Oktober 2026",
        "size": "4.2 MB",
        "format": "PDF",
        "url": "/api/v1/rektor/laporan/unduh/REP-2026-01"
      },
      {
        "id": "REP-2026-02",
        "title": "Laporan Evaluasi Diri (LED) Akreditasi Institusi",
        "category": "Mutu & Akreditasi",
        "date": "28 September 2026",
        "size": "12.8 MB",
        "format": "PDF",
        "url": "/api/v1/rektor/laporan/unduh/REP-2026-02"
      },
      {
        "id": "REP-2026-03",
        "title": "Rekapitulasi Realisasi Anggaran & Belanja Q3",
        "category": "Keuangan",
        "date": "01 Oktober 2026",
        "size": "3.5 MB",
        "format": "XLSX",
        "url": "/api/v1/rektor/laporan/unduh/REP-2026-03"
      },
      {
        "id": "REP-2026-04",
        "title": "Laporan Kinerja Dosen & Beban Kerja (BKD) Gasal 2025/2026",
        "category": "SDM & Dosen",
        "date": "10 September 2026",
        "size": "5.1 MB",
        "format": "PDF",
        "url": "/api/v1/rektor/laporan/unduh/REP-2026-04"
      }
    ]
  }
}
```

#### 3.10.2 Unduh Dokumen Laporan
- **Endpoint:** `GET /api/v1/rektor/laporan/unduh/{id}`
- **Headers Respons:** `Content-Type: application/pdf` atau `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`
- **Content-Disposition:** `attachment; filename="Laporan_Kinerja_Rektor_Q3_2026.pdf"`

---

### 3.11 Pusat Kendali Prediksi & AI (`/rektor/ai`)
- **URL Frontend:** `https://harmonites.adnantech.biz.id/rektor/ai`
- **Tujuan Halaman:** University Intelligence Engine menyajikan proyeksi pertumbuhan penerimaan mahasiswa 2027-2028, status 6 pilar model machine learning universitas, serta kotak pencarian tanya-jawab AI berbasis bahasa alami (Natural Language Query).

#### 3.11.1 Get Inisialisasi AI Dashboard & 6 Pilar Model
- **Endpoint:** `GET /api/v1/rektor/ai`
- **Response JSON:**
```json
{
  "status": "success",
  "message": "Model analitik AI eksekutif berhasil dimuat.",
  "data": {
    "chart_skenario": {
      "labels": ["2022", "2023", "2024", "2025", "2026 (Aktual)", "2027 (Prediksi)", "2028 (Prediksi)"],
      "datasets": [
        {
          "label": "Tren Historis & Aktual",
          "borderColor": "#64748b",
          "backgroundColor": "transparent",
          "borderDash": [4, 4],
          "tension": 0.3,
          "data": [3200, 3500, 3800, 4100, 4450, null, null]
        },
        {
          "label": "Proyeksi Skenario AI (Penerimaan Mahasiswa)",
          "borderColor": "#b8860b",
          "backgroundColor": "rgba(184, 134, 11, 0.08)",
          "fill": true,
          "tension": 0.3,
          "data": [null, null, null, null, 4450, 4900, 5400]
        }
      ]
    },
    "pilar_ai": [
      {
        "id": 1,
        "number": "01",
        "title": "Proyeksi Penerimaan Mahasiswa",
        "desc": "Memproyeksikan penerimaan mahasiswa baru berdasarkan tren historis PMB dan faktor demografi regional.",
        "icon": "bi-graph-up-arrow",
        "badge": "Predictive Engine",
        "modelStatus": "Akurat (98.4%)"
      },
      {
        "id": 2,
        "number": "02",
        "title": "Analisis Tren Pendapatan",
        "desc": "Menganalisis tren pendapatan institusi secara komprehensif dengan simulasi skenario inflasi & registrasi.",
        "icon": "bi-cash-coin",
        "badge": "Financial AI",
        "modelStatus": "Stabil"
      },
      {
        "id": 3,
        "number": "03",
        "title": "Identifikasi Risiko Kelulusan",
        "desc": "Mendeteksi indikasi risiko penurunan kelulusan tepat waktu atau potensi mahasiswa drop-out sejak dini.",
        "icon": "bi-shield-exclamation",
        "badge": "Risk Analytics",
        "modelStatus": "Monitoring"
      },
      {
        "id": 4,
        "number": "04",
        "title": "Komparasi Kinerja Fakultas",
        "desc": "Membandingkan produktivitas antar fakultas secara objektif untuk menetapkan area prioritas intervensi.",
        "icon": "bi-pie-chart-fill",
        "badge": "Comparative Metrics",
        "modelStatus": "Active Sync"
      },
      {
        "id": 5,
        "number": "05",
        "title": "Prioritas Investasi Strategis",
        "desc": "Memberikan rekomendasi alokasi belanja infrastruktur laboratorium & IT berdasarkan rasio kebutuhan riil.",
        "icon": "bi-lightning-charge",
        "badge": "Strategic ROI",
        "modelStatus": "Optimized"
      },
      {
        "id": 6,
        "number": "06",
        "title": "Mitigasi Risiko Strategis",
        "desc": "Penyusunan alternatif skenario tindakan preventif beserta unit penanggung jawab dan tenggat mitigasi.",
        "icon": "bi-check2-circle",
        "badge": "Action Matrix",
        "modelStatus": "Ready"
      }
    ]
  }
}
```

#### 3.11.2 Natural Language Query AI Search Box
- **Endpoint:** `POST /api/v1/rektor/ai/query`
- **Request Body:**
```json
{
  "question": "Bagaimana proyeksi penerimaan mahasiswa baru dan serapan anggaran untuk Fakultas Teknik di tahun 2027?"
}
```
##### Response JSON:
```json
{
  "status": "success",
  "message": "Hasil analisis AI berhasil digenerate.",
  "data": {
    "query": "Bagaimana proyeksi penerimaan mahasiswa baru dan serapan anggaran untuk Fakultas Teknik di tahun 2027?",
    "summary": "Berdasarkan model prediktif ARIMA dan tren 5 tahun terakhir, pendaftar Fakultas Teknik diproyeksikan tumbuh sebesar 10.1% pada tahun 2027 (estimasi 1.150 mahasiswa baru). Realisasi serapan anggaran diproyeksikan mencapai 87.5% apabila alokasi belanja modal lab teknik direalisasikan di Q1.",
    "key_metrics": [
      { "label": "Proyeksi Mahasiswa Baru", "value": "1.150 Mhs", "trend": "+10.1%" },
      { "label": "Estimasi Serapan Anggaran", "value": "Rp 7.8 M (87.5%)", "trend": "Optimal" },
      { "label": "Confidence Level", "value": "94.8%", "trend": "Tinggi" }
    ],
    "actionable_recommendations": [
      {
        "priority": "Tinggi",
        "action": "Penambahan 1 ruang laboratorium komputasi cerdas guna menjaga rasio ideal pemakaian fasilitas.",
        "unit": "Fakultas Teknik & Biro Umum",
        "target_deadline": "Juni 2027"
      },
      {
        "priority": "Sedang",
        "action": "Akselerasi penerbitan SK beasiswa kemitraan industri untuk menjaring calon mahasiswa berprestasi.",
        "unit": "Biro Kemahasiswaan",
        "target_deadline": "April 2027"
      }
    ],
    "sources_used": [
      "Data Mart PMB 2021-2026",
      "Data Warehouse Keuangan (Pagu & Realisasi)",
      "Kapasitas Lab & Sarpras Fakultas Teknik"
    ]
  }
}
```

---

## 4. STANDAR PENANGANAN KODE ERROR HTTP

Backend wajib mengembalikan kode status HTTP standar sesuai panduan RFC 7231:

| Kode HTTP | Status Label | Kapan Digunakan |
| :--- | :--- | :--- |
| **200** | OK | Permintaan berhasil (GET, PUT, PATCH). |
| **201** | Created | Data entitas baru berhasil dibuat (POST). |
| **202** | Accepted | Permintaan diterima untuk diproses asinkron di antrean (contoh: Trigger Sinkronisasi). |
| **400** | Bad Request | Parameter kueri atau sintaks request tidak valid. |
| **401** | Unauthorized | Token Bearer JWT tidak disertakan atau telah kedaluwarsa. |
| **403** | Forbidden | Pengguna memiliki token valid namun tidak memiliki hak akses role ke resource ini. |
| **404** | Not Found | Endpoint atau ID resource yang dicari tidak ditemukan. |
| **422** | Unprocessable Entity | Validasi input formulir gagal (misal email duplikat, password kurang panjang). |
| **500** | Internal Server Error | Kesalahan pada server internal atau kegagalan query database tak terduga. |
| **503** | Service Unavailable | Database utama atau layanan eksternal sedang pemeliharaan / down. |

---

## 5. REKOMENDASI IMPLEMENTASI PADA FRONTEND VUE 3

Frontend disarankan membuat satu instance HTTP client menggunakan **Axios** dengan konfigurasi interceptor terpusat:

```typescript
// src/api/http.ts
import axios from 'axios'

const http = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || 'https://api.harmonites.adnantech.biz.id/api/v1',
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json'
  }
})

// Request Interceptor: Sisipkan Token JWT secara otomatis
http.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('access_token')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  (error) => Promise.reject(error)
)

// Response Interceptor: Tangani sesi kedaluwarsa (401)
http.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('access_token')
      window.location.href = '/login?session_expired=true'
    }
    return Promise.reject(error)
  }
)

export default http
```

Contoh pemanggilan di komponen view Vue 3:
```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import http from '@/api/http'

const loading = ref(true)
const kpiCards = ref([])
const chartData = ref(null)

const loadDashboard = async () => {
  try {
    loading.value = true
    const response = await http.get('/rektor/dashboard')
    kpiCards.value = response.data.data.kpi_cards
    chartData.value = response.data.data.chart_tren
  } catch (error) {
    console.error('Gagal memuat dashboard rektor:', error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  loadDashboard()
})
</script>
```

---

*Dokumen ini disusun untuk Tim Pengembang Frontend & Backend HARMONI (University Intelligence Dashboard) Universitas Hamzanwadi.*
