# Konsep REST API

## 1. Apa itu REST API?

**REST (Representational State Transfer)** adalah gaya arsitektur untuk merancang sistem terdistribusi. REST API adalah antarmuka yang menggunakan prinsip REST untuk memungkinkan komunikasi antara client (mobile app) dan server (backend Laravel).

### Analogi Sederhana

Bayangkan REST API seperti menu restoran:
- **Client** = pelanggan yang memesan
- **Server** = dapur yang menyiapkan pesanan
- **Endpoint** = item menu
- **HTTP Method** = jenis aksi (pesan baru, minta ubah, minta hapus)
- **Request Body** = detail pesanan
- **Response** = makanan yang diantarkan

---

## 2. Prinsip-Prinsip REST

### 2.1 Stateless (Tanpa State)

Setiap request harus mengandung semua informasi yang diperlukan server untuk memprosesnya. Server tidak menyimpan "ingatan" tentang request sebelumnya.

```
Request 1: GET /api/profile (dengan token A) → Server proses, kirim data profil
Request 2: GET /api/cart   (dengan token A) → Server proses dari awal, kirim data cart

Server tidak tahu bahwa request 1 dan 2 dari orang yang sama
tanpa adanya token di header.
```

**Implikasi di Bengkelin:** Setiap request dari Flutter harus menyertakan header `Authorization: Bearer {token}`.

### 2.2 Uniform Interface (Antarmuka Seragam)

API menggunakan konvensi URL dan HTTP method yang konsisten:
- **Noun** (benda) untuk URL, bukan verb (kata kerja)
- **HTTP Method** menyatakan aksi

```
❌ Buruk (tidak RESTful):
GET  /api/getBengkel
POST /api/createBengkel
GET  /api/deleteBengkel?id=1

✅ Baik (RESTful):
GET    /api/bengkel          → ambil daftar
POST   /api/bengkel          → buat baru
GET    /api/bengkel/{id}     → ambil satu
PUT    /api/bengkel/{id}     → update
DELETE /api/bengkel/{id}     → hapus
```

### 2.3 Client-Server Separation

Client (Flutter) dan server (Laravel) adalah entitas terpisah yang berkomunikasi hanya melalui API. Keduanya bisa dikembangkan secara independen.

```
Flutter (client)          Laravel (server)
      │                         │
      │  REST API call          │
      │────────────────────────→│
      │                         │
      │  JSON response          │
      │←────────────────────────│
      │                         │
  Flutter hanya peduli      Laravel hanya peduli
  dengan data JSON,         dengan proses bisnis,
  bukan cara Laravel        bukan tampilan UI.
  memrosesnya.
```

---

## 3. HTTP Methods

| Method | Fungsi | Idempoten? | Body? |
|--------|--------|------------|-------|
| **GET** | Ambil data | Ya | Tidak |
| **POST** | Buat data baru | Tidak | Ya |
| **PUT** | Update data (ganti seluruhnya) | Ya | Ya |
| **PATCH** | Update data (sebagian) | Ya | Ya |
| **DELETE** | Hapus data | Ya | Tidak |

> **Idempoten** = memanggil berkali-kali menghasilkan efek yang sama.

### Contoh di Bengkelin

```
GET    /api/bengkel             → Ambil semua bengkel
GET    /api/bengkel/5           → Ambil bengkel dengan id=5
POST   /api/bengkel             → Daftarkan bengkel baru
PUT    /api/bengkel/5           → Update semua data bengkel id=5
DELETE /api/bengkel/5           → Hapus bengkel id=5
```

---

## 4. HTTP Status Code

Status code memberi tahu client tentang hasil request:

| Kode | Nama | Kapan Digunakan |
|------|------|-----------------|
| **200** | OK | Request berhasil (GET, PUT, DELETE) |
| **201** | Created | Data baru berhasil dibuat (POST) |
| **400** | Bad Request | Request tidak valid |
| **401** | Unauthorized | Tidak terautentikasi (token salah/tidak ada) |
| **403** | Forbidden | Terautentikasi tapi tidak punya izin |
| **404** | Not Found | Data/endpoint tidak ditemukan |
| **422** | Unprocessable Entity | Validasi gagal |
| **500** | Internal Server Error | Kesalahan di sisi server |

### Penggunaan di Bengkelin

```json
// 200 OK - Login berhasil
{
    "meta": { "code": 200, "status": "success", "message": "Login berhasil" },
    "data": { "access_token": "...", "user": {...} }
}

// 401 Unauthorized - Token tidak valid
{
    "message": "Unauthenticated."
}

// 422 Unprocessable Entity - Validasi gagal
{
    "meta": { "code": 422, "status": "error", "message": "Validasi gagal" },
    "data": {
        "email": ["Email sudah terdaftar."],
        "password": ["Password minimal 8 karakter."]
    }
}

// 404 Not Found - Data tidak ada
{
    "meta": { "code": 404, "status": "error", "message": "Bengkel tidak ditemukan" },
    "data": null
}
```

---

## 5. Request & Response Structure

### Format Request

Untuk request dengan body (POST, PUT), data dikirim dalam format JSON:

```http
POST /api/login
Content-Type: application/json
Accept: application/json

{
    "email": "user@example.com",
    "password": "password123"
}
```

Untuk upload file (foto bengkel, foto produk), gunakan `multipart/form-data`:

```http
POST /api/bengkel
Content-Type: multipart/form-data
Authorization: Bearer {token}

name=Bengkel Jaya
description=Bengkel terpercaya
image=[file binary]
```

### Format Response

```json
{
    "meta": {
        "code": 200,
        "status": "success",
        "message": "Deskripsi hasil operasi"
    },
    "data": {
        // Payload data
    }
}
```

---

## 6. Autentikasi API

### Bearer Token

```http
GET /api/profile
Authorization: Bearer 1|7Xy9MnZabcdefghijklmnopqrstuvwxyz12345
Content-Type: application/json
Accept: application/json
```

### Skenario Token Tidak Valid

```http
GET /api/profile
Authorization: Bearer token_yang_tidak_valid

Response:
HTTP/1.1 401 Unauthorized
{
    "message": "Unauthenticated."
}
```

---

## 7. Pagination

Untuk endpoint yang mengembalikan banyak data, Laravel menggunakan pagination:

```json
{
    "meta": {
        "code": 200,
        "status": "success",
        "message": "Data berhasil diambil"
    },
    "data": {
        "current_page": 1,
        "data": [...],
        "first_page_url": "http://localhost:8000/api/products?page=1",
        "from": 1,
        "last_page": 5,
        "last_page_url": "http://localhost:8000/api/products?page=5",
        "next_page_url": "http://localhost:8000/api/products?page=2",
        "path": "http://localhost:8000/api/products",
        "per_page": 15,
        "prev_page_url": null,
        "to": 15,
        "total": 72
    }
}
```

Untuk mengakses halaman selanjutnya: `GET /api/products?page=2`

---

## 8. Filtering & Searching

Beberapa endpoint mendukung parameter query untuk filter:

```
GET /api/bengkel?kecamatan_id=1
GET /api/bengkel/nearby?latitude=-6.2088&longitude=106.8456&radius=5
GET /api/products?category_id=3
```

---

## 9. CORS (Cross-Origin Resource Sharing)

Karena aplikasi Flutter mengakses API dari origin yang berbeda (IP device), server harus mengizinkan akses lintas origin.

**Header yang dikirim server:**
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization, Accept
```

Di Laravel, ini dikonfigurasi via `config/cors.php` dan middleware `HandleCors`.
