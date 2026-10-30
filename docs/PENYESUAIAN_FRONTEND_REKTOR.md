# PANDUAN PENYESUAIAN FRONTEND MODUL REKTOR
## Sinkronisasi Kontrak API Backend Go dengan Live Frontend `harmonites.adnantech.biz.id`

> Dokumen ini adalah **panduan integrasi langsung** antara modul eksekutif Rektor pada frontend yang telah aktif di:
> **`https://harmonites.adnantech.biz.id/rektor/*`**
> dengan **Backend API Golang** dan **Data Warehouse HARMONI**.
>
> **Tujuan:** Mengganti data statis/mocking reactive `ref(...)` pada komponen Vue 3 menjadi pemanggilan REST API dinamis (`GET /api/v1/rektor/*`) dengan struktur JSON yang 100% cocok.

---

## DAFTAR ISI

1. [Standar UI Kit & Skema Warna Frontend](#1-standar-ui-kit--skema-warna-frontend)
2. [Pola Integrasi Komponen Vue 3](#2-pola-integrasi-komponen-vue-3)
3. [Spesifikasi Teknis 11 Halaman Rektor](#3-spesifikasi-teknis-11-halaman-rektor)
   - 3.1 [Dashboard Eksekutif (`/rektor/dashboard`)](#31-dashboard-eksekutif-rektordashboard)
   - 3.2 [Akademik (`/rektor/akademik`)](#32-akademik-rektorakademik)
   - 3.3 [Kemahasiswaan (`/rektor/kemahasiswaan`)](#33-kemahasiswaan-rektorkemahasiswaan)
   - 3.4 [SDM Pegawai (`/rektor/sdm`)](#34-sdm-pegawai-rektorsdm)
   - 3.5 [Keuangan & Anggaran (`/rektor/keuangan`)](#35-keuangan--anggaran-rektorkeuangan)
   - 3.6 [Penelitian & Pengabdian (`/rektor/penelitian`)](#36-penelitian--pengabdian-rektorpenelitian)
   - 3.7 [Kerja Sama & Kemitraan (`/rektor/kerjasama`)](#37-kerja-sama--kemitraan-rektorkerjasama)
   - 3.8 [Mutu & Akreditasi (`/rektor/mutu`)](#38-mutu--akreditasi-rektormutu)
   - 3.9 [Sarana & Prasarana (`/rektor/sarpras`)](#39-sarana--prasarana-rektorsarpras)
   - 3.10 [Laporan Eksekutif (`/rektor/laporan`)](#310-laporan-eksekutif-rektorlaporan)
   - 3.11 [Pusat Kendali Prediksi & AI (`/rektor/ai`)](#311-pusat-kendali-prediksi--ai-rektorai)
4. [Matriks Sumber Data Internal ke Halaman Rektor](#4-matriks-sumber-data-internal-ke-halaman-rektor)

---

## 1. Standar UI Kit & Skema Warna Frontend

Berdasarkan inspeksi langsung pada build bundle `harmonites.adnantech.biz.id`:
- **CSS Framework:** Bootstrap 5.3.0 (`bootstrap.min.css`)
- **Ikon UI:** Bootstrap Icons 1.11.0 (`bootstrap-icons.css`, prefix `bi bi-*`)
- **Visualisasi Grafik:** Chart.js 4.x
- **Palet Warna Eksekutif (Gold Theme):**
  - **Gold Utama:** `#b8860b` / `#c59b27` (`text-gold`, `bg-gold-subtle`)
  - **Navy Dark (Latar & Border):** `#0f172a`
  - **Emerald Success:** `#10b981` / `#198754` (`text-success`, `bg-success-soft`)
  - **Blue Info:** `#0d6efd` (`text-primary`, `bg-primary-soft`)
  - **Purple / Violet:** `#8b5cf6` / `#6f42c1`

---

## 2. Pola Integrasi Komponen Vue 3

Pada frontend live saat ini, data masih tersimpan dalam deklarasi reaktif lokal. Pola migrasi ke API backend Go adalah sebagai berikut:

```vue
<!-- Pola Standardisasi Pemanggilan API pada Halaman Rektor -->
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import http from '@/api/http'

const loading = ref(true)
const kpiCards = ref([])
const chartData = ref({ labels: [], datasets: [] })
const tableData = ref([])

onMounted(async () => {
  try {
    loading.value = true
    // Panggil endpoint terdedikasi
    const { data } = await http.get('/rektor/dashboard')
    const res = data.data

    kpiCards.value = res.kpi_cards
    chartData.value = res.chart_tren
    tableData.value = res.aktivitas_terkini
  } catch (error) {
    console.error('Gagal memuat data eksekutif:', error)
  } finally {
    loading.value = false
  }
})
</script>
```

---

## 3. Spesifikasi Teknis 11 Halaman Rektor

### 3.1 Dashboard Eksekutif (`/rektor/dashboard`)
- **View File Frontend:** `RektorView.vue` (`RektorView-DvqE3zN0.js`)
- **Endpoint Backend:** `GET /api/v1/rektor/dashboard`
- **Komponen UI:**
  1. **Shortcuts Panel (4 Menu Cepat):** Validasi SK (`/rektor/mutu`), Monitor Anggaran (`/rektor/keuangan`), Analisis AI (`/rektor/ai`), Arsip Laporan (`/rektor/laporan`).
  2. **4 Kartu KPI:** Total Mahasiswa Aktif, Akreditasi Institusi, Serapan Anggaran Q3, Publikasi Jurnal Ilmiah.
  3. **Grafik Garis Ganda:** Tren Pendaftar Baru vs Mahasiswa Diterima (12 Bulan).
  4. **Activity Feed Panel:** Log kegiatan penting universitas (Persetujuan MoU, Evaluasi Anggaran, dll.).

#### Contoh Response JSON Backend:
```json
{
  "status": "success",
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
        "action": "Persetujuan Nota Kesepahaman (MoU) Mitra Internasional UTM",
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
        "action": "Penerbitan SK Kebijakan Kurikulum MBKM Mandiri",
        "time": "5 jam yang lalu",
        "type": "doc",
        "badge": "Akademik"
      }
    ]
  }
}
```

---

### 3.2 Akademik (`/rektor/akademik`)
- **View File Frontend:** `RektorAkademikView.vue` (`RektorAkademikView-OCDwUymN.js`)
- **Endpoint Backend:** `GET /api/v1/rektor/akademik`
- **Komponen UI:**
  1. **4 Kartu Metrik Bento:** Total Mahasiswa Aktif (`8.432`), Rata-rata IPK Universitas (`3.38`), Prodi Unggul (`14 Prodi`), Beban Dosen Rasio (`1 : 22`).
  2. **Grafik Stacked Bar:** Akreditasi Prodi per Fakultas (Unggul/A vs Baik Sekali/B).
  3. **Grafik Doughnut:** Sebaran Proporsi Mahasiswa per Fakultas.
  4. **Tabel Data Prodi:** Daftar prodi, fakultas, jenjang, status akreditasi, dan masa berlaku.

#### Contoh Response JSON Backend:
```json
{
  "status": "success",
  "data": {
    "kpi_cards": [
      {"title": "Total Mahasiswa Aktif", "value": "8.432", "note": "Rasio Ideal Terkendali", "icon": "bi-mortarboard-fill", "theme": "card-gold"},
      {"title": "Rata-rata IPK Universitas", "value": "3.38", "note": "+0.04 dari Periode Lalu", "icon": "bi-graph-up-arrow", "theme": "card-emerald"},
      {"title": "Prodi Unggul", "value": "14 Prodi", "note": "42% dari Total Program", "icon": "bi-patch-check-fill", "theme": "card-blue"},
      {"title": "Beban Dosen (Rasio)", "value": "1 : 22", "note": "Sesuai Standar SNDikti", "icon": "bi-people-fill", "theme": "card-purple"}
    ],
    "chart_akreditasi_fakultas": {
      "labels": ["Fakultas Teknik", "Fakultas Keguruan", "Fakultas Agama Islam", "Fakultas Kesehatan", "Fakultas Ekonomi"],
      "datasets": [
        {"label": "Unggul / A", "backgroundColor": "#0f172a", "borderRadius": 8, "barThickness": 20, "data": [4, 6, 3, 2, 3]},
        {"label": "Baik Sekali / B", "backgroundColor": "#b8860b", "borderRadius": 8, "barThickness": 20, "data": [2, 1, 2, 3, 2]}
      ]
    },
    "chart_sebaran_mahasiswa": {
      "labels": ["Fakultas Keguruan", "Fakultas Teknik", "Fakultas Kesehatan", "Fakultas Ekonomi", "Fakultas Agama Islam"],
      "datasets": [
        {"backgroundColor": ["#0f172a", "#b8860b", "#0d6efd", "#10b981", "#8b5cf6"], "borderWidth": 0, "data": [35, 25, 18, 12, 10]}
      ]
    },
    "tabel_prodi": [
      {"no": "01", "prodi": "S1 Informatika", "fakultas": "Fakultas Teknik", "akreditasi": "Unggul (A)", "validUntil": "2028", "status": "Aktif"},
      {"no": "02", "prodi": "S1 Sistem Informasi", "fakultas": "Fakultas Teknik", "akreditasi": "Baik Sekali (B)", "validUntil": "2027", "status": "Aktif"},
      {"no": "03", "prodi": "S1 Pendidikan Teknologi Informasi", "fakultas": "Fakultas Keguruan", "akreditasi": "Baik Sekali (B)", "validUntil": "2028", "status": "Aktif"}
    ]
  }
}
```

---

### 3.3 Kemahasiswaan (`/rektor/kemahasiswaan`)
- **View File Frontend:** `RektorKemahasiswaanView.vue` (`RektorKemahasiswaanView-BOp54KOH.js`)
- **Endpoint Backend:** `GET /api/v1/rektor/kemahasiswaan`
- **Komponen UI:**
  1. **4 Kartu Stat:** Total Mahasiswa Aktif (`8.432`), Prestasi Mahasiswa 2026 (`142 Juara`), Penerima Beasiswa (`1.850 Mhs`), Aktivitas Ormawa (`38 Unit`).
  2. **Grafik Batang Ganda:** Tren Prestasi Nasional vs Internasional per bulan.
  3. **Grafik Donat:** Distribusi Alokasi Beasiswa (KIP-K 45%, Yayasan 25%, Pemda 20%, Tahfidz 10%).
  4. **Tabel Kegiatan & Prestasi:** Agenda kegiatan Ormawa dan kompetisi mahasiswa terkini.

#### Contoh Response JSON Backend:
```json
{
  "status": "success",
  "data": {
    "kpi_cards": [
      {"title": "Total Mahasiswa Aktif", "value": "8.432", "trend": "Terdaftar PDDikti", "icon": "bi-people-fill", "color": "text-primary", "bg": "bg-primary-soft"},
      {"title": "Prestasi Mahasiswa (2026)", "value": "142 Juara", "trend": "Nasional & Internasional", "icon": "bi-trophy-fill", "color": "text-gold", "bg": "bg-gold-soft"},
      {"title": "Penerima Beasiswa", "value": "1.850 Mhs", "trend": "KIP-Kuliah & Yayasan", "icon": "bi-award-fill", "color": "text-success", "bg": "bg-success-soft"},
      {"title": "Aktivitas Ormawa/UKM", "value": "38 Unit", "trend": "Aktif Berjalan", "icon": "bi-kanban-fill", "color": "text-info", "bg": "bg-info-soft"}
    ],
    "chart_tren_prestasi": {
      "labels": ["Jan", "Feb", "Mar", "Apr", "Mei", "Jun", "Jul", "Ags", "Sep", "Okt", "Nov", "Des"],
      "datasets": [
        {"label": "Tingkat Nasional", "backgroundColor": "#b8860b", "borderRadius": 6, "data": [8, 12, 15, 10, 18, 22, 14, 16, 20, 15, 12, 10]},
        {"label": "Tingkat Internasional", "backgroundColor": "#198754", "borderRadius": 6, "data": [2, 3, 5, 4, 6, 8, 5, 7, 9, 6, 4, 3]}
      ]
    },
    "chart_distribusi_beasiswa": {
      "labels": ["KIP-Kuliah", "Beasiswa Yayasan", "Beasiswa Pemerintah Daerah", "Beasiswa Prestasi / Tahfidz"],
      "datasets": [
        {"backgroundColor": ["#b8860b", "#0d6efd", "#198754", "#6f42c1"], "data": [45, 25, 20, 10]}
      ]
    },
    "tabel_agenda": [
      {"no": 1, "kegiatan": "Pekan Olahraga Mahasiswa Nasional (POMNAS)", "ormawa": "BEM Universitas", "tanggal": "12-18 Nov 2026", "peserta": "120 Atlet", "status": "Persiapan"},
      {"no": 2, "kegiatan": "National University Debating Championship", "ormawa": "UKM Bahasa Inggris", "tanggal": "04-06 Des 2026", "peserta": "12 Mahasiswa", "status": "Terdaftar"}
    ]
  }
}
```

---

### 3.4 SDM Pegawai (`/rektor/sdm`)
- **View File Frontend:** `RektorSdmView.vue` (`RektorSdmView-BWJiulAJ.js`)
- **Endpoint Backend:** `GET /api/v1/rektor/sdm`
- **Komponen UI:**
  1. **4 Kartu Stat:** Total Dosen Tetap (`312`), Tenaga Kependidikan (`100`), Tersertifikasi Serdos (`78%`), Rasio Dosen : Mhs (`1 : 21`).
  2. **Grafik Batang Horizontal:** Distribusi Jabatan Fungsional Dosen (Asisten Ahli 95, Lektor 140, Lektor Kepala 55, Guru Besar 12, Tenaga Pengajar 10).
  3. **Direktori Profile Cards:** Daftar pimpinan fakultas/prodi strategis (Dekan, Kaprodi, Guru Besar).

#### Contoh Response JSON Backend:
```json
{
  "status": "success",
  "data": {
    "kpi_cards": [
      {"label": "Total Dosen Tetap", "count": "312", "trend": "+4% vs tahun lalu", "trendType": "up", "icon": "bi-person-badge", "theme": "gold"},
      {"label": "Tenaga Kependidikan", "count": "100", "trend": "Stabil", "trendType": "neutral", "icon": "bi-people", "theme": "indigo"},
      {"label": "Tersertifikasi (Serdos)", "count": "78%", "trend": "Target Tercapai", "trendType": "up", "icon": "bi-patch-check", "theme": "emerald"},
      {"label": "Rasio Dosen : Mhs", "count": "1 : 21", "trend": "Ideal SNikti", "trendType": "up", "icon": "bi-pie-chart", "theme": "violet"}
    ],
    "chart_jafung": {
      "labels": ["Asisten Ahli", "Lektor", "Lektor Kepala", "Guru Besar", "Tenaga Pengajar"],
      "datasets": [
        {"label": "Jumlah Dosen", "backgroundColor": "#c59b27", "hoverBackgroundColor": "#b8860b", "borderRadius": 8, "barThickness": 22, "data": [95, 140, 55, 12, 10]}
      ]
    },
    "pejabat_strategis": [
      {"name": "Dr. H. Muhammad, M.Pd.", "title": "Dekan Fakultas Keguruan", "nip": "19820312 200501 1 003", "jafung": "Lektor Kepala", "education": "S3 (Doktor)", "status": "Aktif"},
      {"name": "Ir. Ahmad Zulkarnain, M.T.", "title": "Dekan Fakultas Teknik", "nip": "19790521 200312 1 002", "jafung": "Lektor", "education": "S2 (Magister)", "status": "Aktif"},
      {"name": "Dr. Siti Aminah, S.H., M.H.", "title": "Kaprodi Ilmu Hukum", "nip": "19851004 201012 2 001", "jafung": "Lektor Kepala", "education": "S3 (Doktor)", "status": "Aktif"}
    ]
  }
}
```

---

### 3.5 Keuangan & Anggaran (`/rektor/keuangan`)
- **View File Frontend:** `RektorKeuanganView.vue` (`RektorKeuanganView-B_d0TAzz.js`)
- **Endpoint Backend:** `GET /api/v1/rektor/keuangan`
- **Komponen UI:**
  1. **4 Kartu Stat:** Total Pagu Anggaran (`Rp 42.5 M`), Realisasi Serapan Q3 (`Rp 33.1 M / 78%`), Sisa Pagu (`Rp 9.4 M`), Pendapatan UKT / Non-UKT (`Rp 28.6 M`).
  2. **Grafik Garis:** Target Serapan Kumulatif vs Realisasi Aktual (12 Bulan).
  3. **Tabel Realisasi Fakultas/Unit:** Pagu, realisasi, persentase serapan, dan status efisiensi.

#### Contoh Response JSON Backend:
```json
{
  "status": "success",
  "data": {
    "kpi_cards": [
      {"label": "Total Pagu Anggaran", "count": "Rp 42.5 M", "trend": "Tahun Anggaran 2026", "icon": "bi-wallet2", "theme": "gold"},
      {"label": "Realisasi Serapan (Q3)", "count": "Rp 33.1 M", "trend": "78% Terserap", "trendType": "up", "icon": "bi-graph-up-arrow", "theme": "emerald"},
      {"label": "Sisa Anggaran (Sisa Pagu)", "count": "Rp 9.4 M", "trend": "Aman / Terkendali", "icon": "bi-shield-check", "theme": "blue"},
      {"label": "Pendapatan UKT / Non-UKT", "count": "Rp 28.6 M", "trend": "+12% vs lalu", "trendType": "up", "icon": "bi-cash-stack", "theme": "violet"}
    ],
    "chart_serapan": {
      "labels": ["Jan", "Feb", "Mar", "Apr", "Mei", "Jun", "Jul", "Ags", "Sep", "Okt", "Nov", "Des"],
      "datasets": [
        {"label": "Target Serapan (%)", "borderColor": "#94a3b8", "borderDash": [4, 4], "backgroundColor": "transparent", "tension": 0.3, "data": [10, 20, 30, 40, 50, 60, 70, 80, 90, 95, 98, 100]},
        {"label": "Realisasi Aktual (%)", "borderColor": "#b8860b", "backgroundColor": "rgba(184, 134, 11, 0.08)", "fill": true, "tension": 0.3, "data": [8, 18, 28, 42, 54, 62, 78, 82, 88, 0, 0, 0]}
      ]
    },
    "tabel_unit_anggaran": [
      {"no": 1, "unit": "Fakultas Teknik", "pagu": "Rp 8.500.000.000", "realisasi": "Rp 7.100.000.000", "persentase": "83.5%", "status": "Optimal"},
      {"no": 2, "unit": "Fakultas Keguruan & Ilmu Pendidikan", "pagu": "Rp 10.200.000.000", "realisasi": "Rp 8.200.000.000", "persentase": "80.3%", "status": "Optimal"},
      {"no": 3, "unit": "Fakultas Agama Islam", "pagu": "Rp 5.400.000.000", "realisasi": "Rp 4.100.000.000", "persentase": "75.9%", "status": "Perlu Akselerasi"}
    ]
  }
}
```

---

### 3.6 Penelitian & Pengabdian (`/rektor/penelitian`)
- **View File Frontend:** `RektorPenelitianView.vue` (`RektorPenelitianView-AZr7HahE.js`)
- **Endpoint Backend:** `GET /api/v1/rektor/penelitian`
- **Komponen UI:**
  1. **4 Kartu Stat:** Total Hibah Berjalan (`Rp 3.8 M`), Publikasi Scopus/WOS (`86 Paper`), Perolehan Paten & HKI (`24 HKI`), Dana Hibah Eksternal (`Rp 2.6 M`).
  2. **Grafik Sebaran Hibah per Fakultas.**
  3. **Tabel Judul Penelitian & Pengabdian Strategis.**

#### Contoh Response JSON Backend:
```json
{
  "status": "success",
  "data": {
    "kpi_cards": [
      {"label": "Total Hibah Berjalan", "count": "Rp 3.8 M", "trend": "58 Judul Riset Aktif", "icon": "bi-journal-check", "theme": "gold"},
      {"label": "Publikasi Scopus/WOS", "count": "86 Paper", "trend": "Q1 & Q2 Terindeks", "icon": "bi-award", "theme": "emerald"},
      {"label": "Perolehan Paten & HKI", "count": "24 HKI", "trend": "+6 dari Semester Lalu", "icon": "bi-shield-check", "theme": "blue"},
      {"label": "Dana Hibah Eksternal", "count": "Rp 2.6 M", "trend": "Kemdiktisaintek & DUDI", "icon": "bi-cash-coin", "theme": "violet"}
    ],
    "chart_sebaran_fakultas": {
      "labels": ["Fakultas Teknik", "Fakultas Keguruan", "Fakultas Agama Islam", "Fakultas Kesehatan", "Fakultas Ekonomi"],
      "datasets": [
        {"label": "Jumlah Judul Hibah", "backgroundColor": "#b8860b", "borderRadius": 8, "data": [18, 22, 8, 12, 10]}
      ]
    },
    "tabel_penelitian": [
      {"no": 1, "judul": "Pengembangan Sistem Deteksi Dini Putus Studi Mahasiswa", "ketua": "Dr. Irwan Hamzani, M.Pd.", "fakultas": "Fakultas Keguruan", "skema": "PTUPT Kemdiktisaintek", "dana": "Rp 65.000.000", "status": "Selesai"},
      {"no": 2, "judul": "Pemanfaatan Energi Terbarukan Sel Surya pada Irigasi Pertanian", "ketua": "Ir. Ahmad Zulkarnain, M.T.", "fakultas": "Fakultas Teknik", "skema": "Penelitian Terapan", "dana": "Rp 50.000.000", "status": "Berjalan"}
    ]
  }
}
```

---

### 3.7 Kerja Sama & Kemitraan (`/rektor/kerjasama`)
- **View File Frontend:** `RektorKerjasamaView.vue` (`RektorKerjasamaView-mK4x_L7J.js`)
- **Endpoint Backend:** `GET /api/v1/rektor/kerjasama`
- **Komponen UI:**
  1. **Bento Card Total Mitra:** 85 Mitra Aktif (Industri/DUDI 34, PT 25, Pemerintah 16, Luar Negeri 10).
  2. **Progress Bars Bentuk Kemitraan:** MBKM (30%), Pertukaran Mahasiswa/Dosen (20%), Riset Bersama (25%), PKM (10%).
  3. **Tabel Direktori Mitra Strategis:** Nama instansi, kategori mitra, nomor dokumen, masa berlaku, status.

#### Contoh Response JSON Backend:
```json
{
  "status": "success",
  "data": {
    "total_mitra": 85,
    "kategori_mitra": [
      {"name": "Mitra Industri / DUDI", "count": "34 Mitra"},
      {"name": "Perguruan Tinggi", "count": "25 Mitra"},
      {"name": "Instansi Pemerintah", "count": "16 Mitra"},
      {"name": "Mitra Luar Negeri", "count": "10 Mitra"}
    ],
    "bentuk_kerjasama": [
      {"name": "Magang & Penempatan Kerja (MBKM)", "percentage": 30, "count": "25 Mitra", "color": "bg-success"},
      {"name": "Riset Kolaboratif & Publikasi", "percentage": 25, "count": "21 Mitra", "color": "bg-warning"},
      {"name": "Student & Lecturer Exchange", "percentage": 20, "count": "16 Mitra", "color": "bg-primary"},
      {"name": "Pengabdian Masyarakat Kolaboratif", "percentage": 10, "count": "10 Mitra", "color": "bg-info"}
    ],
    "tabel_mitra": [
      {"no": 1, "instansi": "PT Telkom Indonesia Witel NTB", "kategori": "Industri", "lingkup": "Nasional", "dokumen": "MoU & MoA", "nomorSk": "MoU/UNHAM/2024/011", "validUntil": "2027-05-10", "status": "Aktif"},
      {"no": 2, "instansi": "Universiti Teknologi Malaysia (UTM)", "kategori": "Perguruan Tinggi", "lingkup": "Internasional", "dokumen": "MoU", "nomorSk": "MoU/UNHAM/2025/002", "validUntil": "2030-01-15", "status": "Aktif"}
    ]
  }
}
```

---

### 3.8 Mutu & Akreditasi (`/rektor/mutu`)
- **View File Frontend:** `RektorMutuView.vue` (`RektorMutuView-pn34aKkL.js`)
- **Endpoint Backend:** `GET /api/v1/rektor/mutu`
- **Komponen UI:**
  1. **4 Kartu Metrik:** Indeks Mutu Institusi (`3.85`), Siklus SPMI Aktif (`Siklus IV`), Penyelesaian Temuan AMI (`94%`), Kepuasan Stakeholder (`4.65`).
  2. **List Ketercapaian Standar SPMI:** Kode standar ST-01 s/d ST-08 beserta lead auditor dan skor.
  3. **Tabel Status Audit Mutu Internal (AMI) per Unit.**

#### Contoh Response JSON Backend:
```json
{
  "status": "success",
  "data": {
    "kpi_cards": [
      {"label": "Indeks Mutu Institusi", "value": "3.85", "note": "Skala 4.00 (Unggul)", "icon": "bi-award", "badge": "Optimal", "theme": "gold"},
      {"label": "Siklus SPMI Aktif", "value": "Siklus IV", "note": "Evaluasi Tahap III", "icon": "bi-arrow-repeat-circle", "badge": "Berjalan", "theme": "dark"},
      {"label": "Penyelesaian Temuan AMI", "value": "94%", "note": "Tepat Waktu", "icon": "bi-patch-check", "badge": "Tinggi", "theme": "emerald"},
      {"label": "Kepuasan Stakeholder", "value": "4.65", "note": "Skala 5.00", "icon": "bi-graph-up-arrow", "badge": "Excellent", "theme": "purple"}
    ],
    "standar_spmi": [
      {"code": "ST-01", "name": "Standar Kompetensi Lulusan", "score": "98%", "status": "Tercapai", "lead": "Dr. H. Muhammad"},
      {"code": "ST-02", "name": "Standar Isi Pembelajaran", "score": "95%", "status": "Tercapai", "lead": "Ir. Ahmad Zulkarnain"},
      {"code": "ST-03", "name": "Standar Proses Pembelajaran", "score": "92%", "status": "Tercapai", "lead": "Dr. Siti Aminah"},
      {"code": "ST-04", "name": "Standar Penilaian Mahasiswa", "score": "96%", "status": "Tercapai", "lead": "Rian Hidayat, M.Kom."}
    ],
    "tabel_audit_ami": [
      {"id": "AMI-01", "unit": "Fakultas Teknik", "auditor": "Tim Auditor I (Dr. Hartono)", "findings": 2, "status": "Selesai 100%", "type": "success"},
      {"id": "AMI-02", "unit": "Fakultas Keguruan & Ilmu Pendidikan", "auditor": "Tim Auditor II (Dr. Siti Aminah)", "findings": 4, "status": "Dalam Proses 75%", "type": "warning"}
    ]
  }
}
```

---

### 3.9 Sarana & Prasarana (`/rektor/sarpras`)
- **View File Frontend:** `RektorSarprasView.vue` (`RektorSarprasView-DduHOJzG.js`)
- **Endpoint Backend:** `GET /api/v1/rektor/sarpras`
- **Komponen UI:**
  1. **4 Kartu Stat:** Total Aset Fisik (`2.480 Unit`), Gedung & Fasilitas (`24 Gedung`), Ruang Kuliah & Lab (`130 Ruang`), Pemeliharaan (`14 Unit`).
  2. **Grid Zonasi Kampus:** Beban utilisasi Zona A (Rektorat), Zona B (Fakultas Teknik & Lab), Zona C (FKIP), Zona D (FAI).
  3. **Tabel Inventaris Aset Fisik:** Kode tag aset, nama barang, lokasi, penanggung jawab, dan kondisi.

#### Contoh Response JSON Backend:
```json
{
  "status": "success",
  "data": {
    "kpi_cards": [
      {"label": "Total Aset Fisik", "value": "2.480", "unit": "Unit", "desc": "Inventaris Kampus", "icon": "bi-box-seam", "theme": "gold"},
      {"label": "Gedung & Fasilitas", "value": "24", "unit": "Gedung", "desc": "Utama & Penunjang", "icon": "bi-buildings", "theme": "blue"},
      {"label": "Ruang Kuliah & Lab", "value": "130", "unit": "Ruang", "desc": "Utilisasi 92%", "icon": "bi-door-open", "theme": "emerald"},
      {"label": "Pemeliharaan", "value": "14", "unit": "Unit", "desc": "Dalam Servis / Perbaikan", "icon": "bi-tools", "theme": "purple"}
    ],
    "zonasi_kampus": [
      {"zone": "Zona A - Rektorat & Biro Pusat", "buildings": 4, "status": "Optimal", "load": "85%"},
      {"zone": "Zona B - Fakultas Teknik & Lab", "buildings": 6, "status": "Padat", "load": "96%"},
      {"zone": "Zona C - FKIP & Kuliah Umum", "buildings": 8, "status": "Optimal", "load": "90%"},
      {"zone": "Zona D - FAI & Mahasiswa", "buildings": 6, "status": "Stabil", "load": "72%"}
    ],
    "tabel_aset": [
      {"tag": "AST-IT-992", "item": "Projektor Epson EB-X400", "location": "Lab Komputer 3 (Teknik)", "custodian": "Fakultas Teknik", "condition": "Sangat Baik"},
      {"tag": "AST-HV-104", "item": "AC Daikin 2 PK Inverter", "location": "Ruang Sidang Utama Rektorat", "custodian": "Biro Umum", "condition": "Perawatan"}
    ]
  }
}
```

---

### 3.10 Laporan Eksekutif (`/rektor/laporan`)
- **View File Frontend:** `RektorLaporanView.vue` (`RektorLaporanView-C3F5_q6x.js`)
- **Endpoint Backend:** `GET /api/v1/rektor/laporan`
- **Komponen UI:**
  1. **4 Kartu Stat:** Total Arsip Laporan (`30 Dokumen`), Laporan Triwulanan (`12 Dokumen`), Borang Akreditasi (`8 Dokumen`), Laporan Keuangan (`6 Dokumen`).
  2. **Grafik Bar Kategori:** Distribusi jumlah dokumen per domain.
  3. **Direktori Dokumen Eksekutif:** Judul laporan, kategori, tanggal rilis, ukuran, format (PDF/XLSX), dan tautan unduh.

#### Contoh Response JSON Backend:
```json
{
  "status": "success",
  "data": {
    "kpi_cards": [
      {"label": "Total Arsip Laporan", "value": "30 Dokumen", "trend": "Terarsip Digital", "icon": "bi-journal-richtext", "theme": "gold"},
      {"label": "Laporan Triwulanan", "value": "12 Dokumen", "trend": "Periode 2026", "icon": "bi-file-earmark-bar-graph", "theme": "blue"},
      {"label": "Borang Akreditasi", "value": "8 Dokumen", "trend": "LED & SPMI", "icon": "bi-shield-check", "theme": "emerald"},
      {"label": "Laporan Keuangan", "value": "6 Dokumen", "trend": "Audit Akuntabilitas", "icon": "bi-wallet2", "theme": "purple"}
    ],
    "chart_kategori": {
      "labels": ["Eksekutif & Kinerja", "Mutu & Akreditasi", "Keuangan & Hibah", "SDM & Dosen", "Kerjasama & Mitra"],
      "datasets": [
        {"label": "Jumlah Dokumen Laporan", "backgroundColor": "#b8860b", "borderRadius": 8, "barThickness": 24, "data": [8, 7, 6, 5, 4]}
      ]
    },
    "daftar_laporan": [
      {"id": "REP-2026-01", "title": "Laporan Kinerja Rektor Triwulan III 2026", "category": "Eksekutif", "date": "15 Oktober 2026", "size": "4.2 MB", "format": "PDF", "url": "/api/v1/laporan/unduh/REP-2026-01"},
      {"id": "REP-2026-02", "title": "Laporan Evaluasi Diri (LED) Akreditasi Institusi", "category": "Mutu & Akreditasi", "date": "28 September 2026", "size": "12.8 MB", "format": "PDF", "url": "/api/v1/laporan/unduh/REP-2026-02"},
      {"id": "REP-2026-03", "title": "Rekapitulasi Realisasi Anggaran & Belanja Q3", "category": "Keuangan", "date": "01 Oktober 2026", "size": "3.5 MB", "format": "XLSX", "url": "/api/v1/laporan/unduh/REP-2026-03"}
    ]
  }
}
```

---

### 3.11 Pusat Kendali Prediksi & AI (`/rektor/ai`)
- **View File Frontend:** `RektorAiView.vue` (`RektorAiView-DFg6J-G9.js`)
- **Endpoint Backend:**
  - `GET /api/v1/rektor/ai` (Data inisialisasi grafik & 6 pilar AI)
  - `POST /api/v1/rektor/ai/query` (Penjawab pertanyaan analitik bahasa alami)
- **Komponen UI:**
  1. **AI Hero Search Box:** Input interaktif bagi Rektor untuk menanyakan analitik data universitas secara langsung.
  2. **Grafik Proyeksi Skenario AI:** Tren Historis & Aktual (2022-2026) vs Proyeksi Skenario AI (2027-2028).
  3. **6 Bento Cards Pilar Utama AI:**
     - 01: Proyeksi Penerimaan Mahasiswa (`Predictive Engine`, `Akurat 98.4%`)
     - 02: Analisis Tren Pendapatan (`Financial AI`, `Stabil`)
     - 03: Identifikasi Risiko Kelulusan (`Risk Analytics`, `Monitoring`)
     - 04: Komparasi Kinerja Fakultas (`Comparative Metrics`, `Active Sync`)
     - 05: Prioritas Investasi Strategis (`Strategic ROI`, `Optimized`)
     - 06: Mitigasi Risiko Strategis (`Action Matrix`, `Ready`)
  4. **Metodologi Kerangka AI:** Identifikasi Masalah $\rightarrow$ Indikator Pendukung $\rightarrow$ Prediksi Dampak $\rightarrow$ Alternatif Tindakan $\rightarrow$ Prioritas $\rightarrow$ Monitoring.

#### Contoh Response JSON Backend (`GET /api/v1/rektor/ai`):
```json
{
  "status": "success",
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
        "desc": "Memproyeksikan penerimaan mahasiswa berdasarkan tren historis dan faktor pendukung yang tersedia.",
        "icon": "bi-graph-up-arrow",
        "badge": "Predictive Engine",
        "modelStatus": "Akurat (98.4%)"
      },
      {
        "id": 2,
        "number": "02",
        "title": "Analisis Tren Pendapatan",
        "desc": "Menganalisis tren pendapatan institusi secara komprehensif dengan asumsi skenario yang dinyatakan.",
        "icon": "bi-cash-coin",
        "badge": "Financial AI",
        "modelStatus": "Stabil"
      },
      {
        "id": 3,
        "number": "03",
        "title": "Identifikasi Risiko Kelulusan",
        "desc": "Mengidentifikasi risiko penurunan kelulusan tepat waktu atau potensi ketidakseimbangan sumber daya.",
        "icon": "bi-shield-exclamation",
        "badge": "Risk Analytics",
        "modelStatus": "Monitoring"
      },
      {
        "id": 4,
        "number": "04",
        "title": "Komparasi Kinerja Fakultas",
        "desc": "Membandingkan kinerja antar fakultas secara objektif untuk menemukan area spesifik yang memerlukan intervensi.",
        "icon": "bi-pie-chart-fill",
        "badge": "Comparative Metrics",
        "modelStatus": "Active Sync"
      },
      {
        "id": 5,
        "number": "05",
        "title": "Prioritas Investasi Strategis",
        "desc": "Merekomendasikan prioritas alokasi investasi berdasarkan tingkat kebutuhan mendesak dan dampak prediktif.",
        "icon": "bi-cpu-fill",
        "badge": "Strategic ROI",
        "modelStatus": "Optimized"
      },
      {
        "id": 6,
        "number": "06",
        "title": "Mitigasi Risiko Strategis",
        "desc": "Mengidentifikasi risiko strategis jangka panjang dan menawarkan opsi alternatif tindakan korektif.",
        "icon": "bi-compass-fill",
        "badge": "Action Matrix",
        "modelStatus": "Ready"
      }
    ]
  }
}
```

#### Contoh Response Query NLP AI (`POST /api/v1/rektor/ai/query`):
```json
{
  "status": "success",
  "data": {
    "pertanyaan": "Bagaimana proyeksi serapan anggaran akhir tahun Fakultas Teknik?",
    "jawaban": "Berdasarkan laju serapan saat ini (83.5% hingga Q3), Fakultas Teknik diproyeksikan mencapai serapan akhir tahun sebesar 94.2% (kategori Optimal), dengan sisa pagu efisiensi sekitar Rp 490 juta.",
    "indikator_terkait": [
      {"kpi": "KEU_SERAPAN_ANGGARAN", "nilai": 83.5, "target": 75.0, "status": "HIJAU"}
    ],
    "rekomendasi_tindakan": "Alokasikan sisa anggaran pemeliharaan laboratorium sebelum batas tutup buku 15 Desember 2026."
  }
}
```

---

## 4. Matriks Sumber Data Internal ke Halaman Rektor

Tabel berikut menunjukkan hubungan langsung antara halaman frontend Rektor dengan API sistem internal yang telah dispesifikasikan pada `kebutuhan_api.md`:

| Halaman Rektor di Frontend | Sistem Sumber Data Internal | Endpoint Kebutuhan API Terkait | Tabel Tujuan Data Mart HARMONI |
|---|---|---|---|
| `/rektor/dashboard` | SIAKAD, PMB, Keuangan, LPPM | `/pmb/statistik`, `/keuangan/realisasi-anggaran`, `/lppm/publikasi` | `mart.mv_pmb_funnel`, `mart.mv_keuangan_unit_bulan` |
| `/rektor/akademik` | SIAKAD, E-MONEVIN | `/siakad/mahasiswa`, `/siakad/kelas-kuliah`, `/emonevin/akreditasi` | `mart.mv_mahasiswa_prodi_semester`, `dw.fact_akreditasi` |
| `/rektor/kemahasiswaan` | SIAKAD, Kemahasiswaan | `/siakad/mahasiswa`, Data Prestasi & Beasiswa Internal | `dw.fact_prestasi_mahasiswa`, `dw.fact_beasiswa` |
| `/rektor/sdm` | SIMPEG, Absensi | `/simpeg/pegawai`, `/simpeg/riwayat-jabfung`, `/absensi/kehadiran` | `dw.dim_pegawai`, `mart.mv_kehadiran_bulan` |
| `/rektor/keuangan` | Sistem Keuangan | `/keuangan/realisasi-anggaran`, `/keuangan/pendapatan` | `dw.fact_keuangan_bulanan`, `mart.mv_keuangan_unit_bulan` |
| `/rektor/penelitian` | LPPM | `/lppm/penelitian`, `/lppm/pengabdian`, `/lppm/publikasi` | `dw.fact_penelitian`, `dw.fact_publikasi` |
| `/rektor/kerjasama` | Biro Kerjasama & MBKM | Data Kemitraan & Perjanjian Kerjasama Internal | `dw.fact_kerjasama` |
| `/rektor/mutu` | E-MONEVIN (BPM) | `/emonevin/indikator-mutu`, `/emonevin/temuan`, `/emonevin/akreditasi` | `dw.fact_indikator_mutu`, `dw.fact_temuan` |
| `/rektor/sarpras` | SIM Aset / Sarpras | Data Inventarisasi Sarpras & Master Gedung | `dw.fact_sarpras` |
| `/rektor/laporan` | E-Arsip / Tata Usaha Rektorat | Master Dokumen Laporan Kinerja Eksekutif | `dw.dokumen` |
| `/rektor/ai` | Data Mart & Mesin AI HARMONI | Hasil Eksekusi Forecasting Holt & Model Rekomendasi | `ai.prediksi`, `ai.rekomendasi`, `ai.skor_risiko` |

---
*Dokumen penyesuaian frontend ini memastikan seluruh 11 halaman eksekutif Rektor pada `harmonites.adnantech.biz.id` dapat bertransisi dengan mulus dari data mock ke sistem produksi.*
