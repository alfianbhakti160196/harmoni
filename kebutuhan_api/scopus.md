# SPESIFIKASI KEBUTUHAN API: ELSEVIER SCOPUS & GOOGLE SCHOLAR
## University Intelligence Dashboard (HARMONI) — Universitas Hamzanwadi

> **Sistem Sumber:** Elsevier Developer API & Google Scholar | **Kritisitas:** Tinggi (Indeksasi Internasional & Sitasi)
> **Konsumen:** ETL Pipeline HARMONI
> **Base URL Contoh:** `https://api.elsevier.com`

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

---

## Panduan Pengujian & Checklist Kesiapan Integrasi

Sebelum menghubungkan API ini ke pipeline ETL HARMONI di server produksi:

- [ ] **Endpoint Aktif:** Endpoint mengembalikan status HTTP `200 OK` dengan format JSON yang valid.
- [ ] **Autentikasi Terverifikasi:** Request tanpa header `Authorization: Bearer ...` mengembalikan HTTP `401 Unauthorized`.
- [ ] **Watermarking Berfungsi:** Parameter `updated_since` benar-benar memfilter record dengan kondisi `updated_at >= updated_since`.
- [ ] **Paginasi Akurat:** Nilai `page`, `page_size`, `total_records`, dan `total_pages` pada objek `meta` sesuai dengan jumlah data yang dikembalikan.
- [ ] **Data Sensitif Terlindungi:** Tidak mengekspos field password, hash credential, atau informasi perbankan rahasia.
- [ ] **Performa Query:** Waktu respons API untuk batch 100 record tidak melebihi 2 detik.
