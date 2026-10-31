# HARMONI — University Intelligence Dashboard
### Universitas Hamzanwadi

Dokumentasi arsitektur sistem, spesifikasi API, dan panduan teknis implementasi untuk dashboard eksekutif dan analitik pimpinan **Universitas Hamzanwadi**.

---

## 📚 Panduan & Dokumentasi Utama

### 1. Cetak Biru Teknis & Panduan Implementasi
- **[`perancangan.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/perancangan.md)**: Dokumen arsitektur lengkap (Tech Stack Go, Integrasi SSO Hamzanwadi Keycloak OIDC, DDL PostgreSQL multi-skema, RBAC, Data Scope, Machine Learning/Forecasting, Asynq Worker ETL, dan panduan langkah-demi-langkah fase 0–8).
- **[`docs/PENYESUAIAN_FRONTEND_REKTOR.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/docs/PENYESUAIAN_FRONTEND_REKTOR.md)**: Pemetaan antarmuka 11 halaman live Rektor (`https://harmonites.adnantech.biz.id/rektor/*`) ke endpoint backend Go dan query database Data Warehouse.

---

## 🔌 Spesifikasi Kebutuhan API Per Aplikasi (`kebutuhan_api/`)

Kebutuhan API dari masing-masing sistem telah dipisahkan menjadi dokumen mandiri lengkap dengan contoh request dan **respons JSON riil**:

### A. Sistem Internal Kampus (Hamzanwadi)
1. **[`kebutuhan_api/siakad.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/siakad.md)** *(atau [`siakad.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/siakad.md))*: Sistem Informasi Akademik (Master Mahasiswa, Riwayat Semester, Mata Kuliah, KHS, Kelulusan, Penugasan Dosen).
2. **[`kebutuhan_api/pmb.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/pmb.md)** *(atau [`pmb.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/pmb.md))*: Sistem PMB Online (Pendaftar, Jalur Seleksi, Nilai Ujian, Registrasi, Kuota & Daya Tampung).
3. **[`kebutuhan_api/keuangan.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/keuangan.md)** *(atau [`keuangan.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/keuangan.md))*: Sistem Keuangan Kampus (Pembayaran SPP/UKT Mahasiswa, Anggaran & Realisasi Belanja Unit).
4. **[`kebutuhan_api/sdm.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/sdm.md)** *(atau [`sdm.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/sdm.md))*: SIMPEG (Master Dosen & Tendik, Riwayat Jafung & Pangkat).
5. **[`kebutuhan_api/absensi.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/absensi.md)** *(atau [`absensi.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/absensi.md))*: Sistem Absensi Pegawai (Log Presensi Harian, Verifikasi SSO Subject ID `sub` Keycloak).
6. **[`kebutuhan_api/penelitian.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/penelitian.md)** *(atau [`penelitian.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/penelitian.md))*: LPPM (Penelitian, Pengabdian Masyarakat, Publikasi Jurnal, Paten/HKI).
7. **[`kebutuhan_api/emonevin.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/emonevin.md)** *(atau [`emonevin.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/emonevin.md))*: Sistem E-MONEVIN (Standar SPMI, Audit Mutu Internal / AMI, Evaluasi Kinerja Unit).
8. **[`kebutuhan_api/tracer_study.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/tracer_study.md)** *(atau [`tracer_study.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/tracer_study.md))*: Sistem Alumni & Tracer Study (Masa Tunggu, Keselarasan Kerja, Direktori Alumni).

### B. Sistem & Sumber Eksternal Resmi
9. **[`kebutuhan_api/pddikti.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/pddikti.md)** *(atau [`pddikti.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/pddikti.md))*: Web Service Resmi PDDIKTI Neo Feeder (JSON-RPC).
10. **[`kebutuhan_api/sister.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/sister.md)** *(atau [`sister.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/sister.md))*: Web Service Resmi SISTER Cloud Kemdiktisaintek.
11. **[`kebutuhan_api/sinta.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/sinta.md)** *(atau [`sinta.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/sinta.md))*: Integrasi Data SINTA v3 Kemdiktisaintek.
12. **[`kebutuhan_api/scopus.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/scopus.md)** *(atau [`scopus.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/scopus.md))*: Elsevier Scopus Developer API & Google Scholar.
13. **[`kebutuhan_api/akreditasi.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/akreditasi.md)** *(atau [`akreditasi.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/akreditasi.md))*: Integrasi Data Akreditasi BAN-PT & Lembaga Akreditasi Mandiri (LAM).

👉 **Indeks Lengkap & Matriks Pemetaan Kunci:** Lihat [`kebutuhan_api/README.md`](file:///home/alfian-bhakti/Documents/Projek/harmoni/dashboard-harmoni/kebutuhan_api/README.md).
