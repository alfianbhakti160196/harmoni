# SPESIFIKASI KEBUTUHAN API: WEB SERVICE RESMI PDDIKTI NEO FEEDER
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** PDDIKTI Neo Feeder Kemdiktisaintek (Lokal Kampus) | **Kritisitas:** Kritis (Pelaporan Nasional)
> **Protokol:** JSON-RPC over HTTP/HTTPS
> **Base URL Contoh:** `http://feeder.hamzanwadi.ac.id:8082/ws/live2.php`

---

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

---

## Panduan Pengujian & Checklist Kesiapan Integrasi

Sebelum menghubungkan API ini ke pipeline ETL HARMONI di server produksi:

- [ ] **Endpoint Aktif:** Endpoint mengembalikan status HTTP `200 OK` dengan format JSON yang valid.
- [ ] **Autentikasi Terverifikasi:** Request tanpa header `Authorization: Bearer ...` mengembalikan HTTP `401 Unauthorized`.
- [ ] **Watermarking Berfungsi:** Parameter `updated_since` benar-benar memfilter record dengan kondisi `updated_at >= updated_since`.
- [ ] **Paginasi Akurat:** Nilai `page`, `page_size`, `total_records`, dan `total_pages` pada objek `meta` sesuai dengan jumlah data yang dikembalikan.
- [ ] **Data Sensitif Terlindungi:** Tidak mengekspos field password, hash credential, atau informasi perbankan rahasia.
- [ ] **Performa Query:** Waktu respons API untuk batch 100 record tidak melebihi 2 detik.
