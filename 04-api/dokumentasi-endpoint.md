# Dokumentasi Endpoint API Bengkelin

> **Base URL:** `http://[server]/api`
>
> **Format Request Body:** `application/json` (kecuali upload file: `multipart/form-data`)
>
> **Format Response:** JSON dengan struktur `{ meta: {...}, data: {...} }`

---

## A. Autentikasi

### A1. Registrasi User

```
POST /api/register
```

**Request Body:**

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `name` | string | Ya | Nama lengkap (max 100) |
| `email` | string | Ya | Email unik |
| `password` | string | Ya | Min 8 karakter |
| `password_confirmation` | string | Ya | Harus sama dengan password |
| `phone_number` | string | Ya | Nomor telepon |
| `alamat` | string | Ya | Alamat lengkap |
| `kecamatan_id` | integer | Ya | ID kecamatan |
| `kelurahan_id` | integer | Ya | ID kelurahan |

**Contoh Request:**
```json
{
    "name": "Budi Santoso",
    "email": "budi@example.com",
    "password": "password123",
    "password_confirmation": "password123",
    "phone_number": "08123456789",
    "alamat": "Jl. Merdeka No. 1",
    "kecamatan_id": 1,
    "kelurahan_id": 5
}
```

**Response 200 OK:**
```json
{
    "meta": {
        "code": 200,
        "status": "success",
        "message": "User registered successfully"
    },
    "data": {
        "access_token": "1|abc123...",
        "token_type": "Bearer",
        "user": {
            "id": 1,
            "name": "Budi Santoso",
            "email": "budi@example.com",
            "phone_number": "08123456789",
            "alamat": "Jl. Merdeka No. 1",
            "kecamatan_id": 1,
            "kelurahan_id": 5,
            "created_at": "2025-01-01T10:00:00.000000Z"
        }
    }
}
```

---

### A2. Login User

```
POST /api/login
```

**Request Body:**

| Field | Tipe | Wajib |
|-------|------|-------|
| `email` | string | Ya |
| `password` | string | Ya |

**Contoh Request:**
```json
{
    "email": "budi@example.com",
    "password": "password123"
}
```

**Response 200 OK:**
```json
{
    "meta": { "code": 200, "status": "success", "message": "Login berhasil" },
    "data": {
        "access_token": "1|abc123...",
        "token_type": "Bearer",
        "user": { "id": 1, "name": "Budi Santoso", "email": "budi@example.com" }
    }
}
```

**Response 401 Unauthorized:**
```json
{
    "meta": { "code": 401, "status": "error", "message": "Email atau password salah" },
    "data": null
}
```

---

### A3. Registrasi Owner

```
POST /api/register-owner
```

**Request Body:**

| Field | Tipe | Wajib |
|-------|------|-------|
| `name` | string | Ya |
| `email` | string | Ya |
| `password` | string | Ya |
| `password_confirmation` | string | Ya |
| `phone_number` | string | Ya |

---

### A4. Login Owner

```
POST /api/login-owner
```

Request body sama seperti login user.

**Response** menyertakan `access_token` yang digunakan sebagai `auth:owner-api` token.

---

### A5. Reset Password (OTP) — User

**Langkah 1 — Kirim OTP:**
```
POST /api/user/forgot-password/send-otp
Body: { "email": "budi@example.com" }
```

**Langkah 2 — Verifikasi OTP:**
```
POST /api/user/forgot-password/verify-otp
Body: { "email": "budi@example.com", "otp": "123456" }
```

**Langkah 3 — Reset Password:**
```
POST /api/user/forgot-password/reset-password
Body: {
    "email": "budi@example.com",
    "password": "newpassword",
    "password_confirmation": "newpassword"
}
```

---

## B. Halaman Utama & Produk (Publik)

### B1. Data Halaman Utama

```
GET /api/home
```

Mengembalikan data untuk ditampilkan di halaman beranda: featured products, daftar bengkel, dll.

---

### B2. Daftar Produk

```
GET /api/products
```

**Query Parameters (opsional):**

| Parameter | Keterangan |
|-----------|------------|
| `page` | Nomor halaman (default: 1) |
| `category_id` | Filter berdasarkan kategori |

**Response:**
```json
{
    "meta": { "code": 200, "status": "success", "message": "..." },
    "data": {
        "current_page": 1,
        "data": [
            {
                "id": 1,
                "name": "Oli Shell Helix 10W-40",
                "description": "Oli mesin berkualitas tinggi",
                "price": 85000,
                "stock": 50,
                "image_url": "http://server/storage/products/oli.jpg",
                "bengkel": { "id": 1, "name": "Bengkel Jaya" },
                "category": { "id": 2, "name": "Pelumas" }
            }
        ],
        "total": 72,
        "per_page": 15
    }
}
```

---

### B3. Detail Produk

```
GET /api/products/{id}
```

---

## C. Bengkel (Publik)

### C1. Daftar Bengkel

```
GET /api/bengkel
```

**Query Parameters:**

| Parameter | Keterangan |
|-----------|------------|
| `kecamatan_id` | Filter berdasarkan kecamatan |

---

### C2. Detail Bengkel

```
GET /api/bengkel/{id}
```

**Response:**
```json
{
    "meta": { "code": 200, "status": "success" },
    "data": {
        "id": 1,
        "name": "Bengkel Jaya Motor",
        "description": "Bengkel terpercaya dengan teknisi berpengalaman",
        "alamat": "Jl. Industri No. 10, Bandung",
        "image_url": "http://server/storage/bengkel/bengkel1.jpg",
        "latitude": -6.9175,
        "longitude": 107.6191,
        "kecamatan": { "id": 1, "name": "Coblong" },
        "kelurahan": { "id": 5, "name": "Cipaganti" },
        "layanans": [
            { "id": 1, "name": "Ganti Oli", "price": 75000 },
            { "id": 2, "name": "Tune Up", "price": 200000 }
        ],
        "jadwals": [
            {
                "id": 1,
                "senin_buka": "08:00:00",
                "senin_tutup": "17:00:00"
            }
        ],
        "specialists": [
            { "id": 1, "name": "Mesin" },
            { "id": 2, "name": "AC Mobil" }
        ],
        "merk_mobils": [
            { "id": 1, "nama_merk": "Toyota" },
            { "id": 2, "nama_merk": "Honda" }
        ]
    }
}
```

---

### C3. Bengkel Terdekat

```
GET /api/bengkel/nearby?latitude=-6.2088&longitude=106.8456&radius=5
```

**Query Parameters:**

| Parameter | Tipe | Keterangan |
|-----------|------|------------|
| `latitude` | decimal | Koordinat lintang posisi user |
| `longitude` | decimal | Koordinat bujur posisi user |
| `radius` | integer | Radius pencarian dalam kilometer (default: 5) |

---

### C4. Cari Bengkel by Kecamatan

```
GET /api/service?kecamatan=Coblong
```

---

## D. Booking (User — `auth:sanctum`)

### D1. Buat Booking

```
POST /api/booking
Authorization: Bearer {token}
```

**Request Body:**

| Field | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `bengkel_id` | integer | Ya | ID bengkel yang dituju |
| `tanggal_booking` | date | Ya | Format: YYYY-MM-DD |
| `waktu_booking` | time | Ya | Format: HH:MM |
| `layanan_ids` | array | Ya | Array ID layanan yang dipilih |
| `brand` | string | Ya | Merk kendaraan (Toyota, Honda, dll) |
| `model` | string | Ya | Model kendaraan (Avanza, Jazz, dll) |
| `plat` | string | Ya | Nomor plat kendaraan |
| `tahun_pembuatan` | integer | Ya | Tahun pembuatan |
| `kilometer` | integer | Ya | Odometer saat ini |
| `transmisi` | string | Ya | `manual` atau `matic` |
| `catatan_tambahan` | string | Tidak | Catatan tambahan |

**Contoh Request:**
```json
{
    "bengkel_id": 1,
    "tanggal_booking": "2025-02-15",
    "waktu_booking": "09:00",
    "layanan_ids": [1, 3],
    "brand": "Toyota",
    "model": "Avanza",
    "plat": "B 1234 XYZ",
    "tahun_pembuatan": 2020,
    "kilometer": 25000,
    "transmisi": "manual",
    "catatan_tambahan": "Bunyi aneh di rem belakang"
}
```

**Response 200 OK:**
```json
{
    "meta": { "code": 200, "status": "success", "message": "Booking berhasil dibuat" },
    "data": {
        "id": 10,
        "user_id": 1,
        "bengkel_id": 1,
        "tanggal_booking": "2025-02-15",
        "waktu_booking": "09:00:00",
        "booking_status": "PENDING",
        "detail_layanan_bookings": [
            { "layanan_id": 1, "name": "Ganti Oli", "price": 75000 },
            { "layanan_id": 3, "name": "Cek Rem", "price": 50000 }
        ]
    }
}
```

---

### D2. Cek Waktu yang Sudah Terpesan

```
GET /api/booking/{bengkel_id}/booked-times?date=2025-02-15
Authorization: Bearer {token}
```

**Response:**
```json
{
    "meta": { "code": 200, "status": "success" },
    "data": ["09:00", "10:00", "13:00"]
}
```

Waktu-waktu ini tidak bisa dipilih saat booking.

---

### D3. Daftar Booking User

```
GET /api/user/bookings
Authorization: Bearer {token}
```

---

## E. Keranjang & Checkout (User — `auth:sanctum`)

### E1. Lihat Keranjang

```
GET /api/cart
Authorization: Bearer {token}
```

### E2. Tambah ke Keranjang

```
POST /api/cart/add
Authorization: Bearer {token}
Body: { "product_id": 5, "quantity": 2 }
```

### E3. Update Keranjang

```
PUT /api/cart/{id}
Authorization: Bearer {token}
Body: { "quantity": 3 }
```

### E4. Hapus dari Keranjang

```
DELETE /api/cart/{id}
Authorization: Bearer {token}
```

### E5. Checkout

```
POST /api/checkout
Authorization: Bearer {token}
```

**Request Body:**
```json
{
    "cart_ids": [1, 2, 3],
    "shipping_address": "Jl. Merdeka No. 10, Bandung"
}
```

**Response:**
```json
{
    "meta": { "code": 200, "status": "success", "message": "Checkout berhasil" },
    "data": {
        "transaction_id": 25,
        "transaction_code": "TRX-2025-001",
        "grand_total": 185000,
        "snap_token": "abc123def456...",
        "redirect_url": "https://app.sandbox.midtrans.com/snap/v2/vtweb/abc123"
    }
}
```

Client menggunakan `snap_token` untuk membuka halaman pembayaran Midtrans.

---

## F. Manajemen Bengkel (Owner — `auth:owner-api`)

### F1. Lihat Data Bengkel Sendiri

```
GET /api/my-bengkel
Authorization: Bearer {owner_token}
```

### F2. Daftarkan Bengkel

```
POST /api/bengkel
Authorization: Bearer {owner_token}
Content-Type: multipart/form-data
```

**Request Body (form-data):**

| Field | Tipe | Keterangan |
|-------|------|------------|
| `name` | string | Nama bengkel |
| `description` | string | Deskripsi |
| `alamat` | string | Alamat lengkap |
| `kecamatan_id` | integer | ID kecamatan |
| `kelurahan_id` | integer | ID kelurahan |
| `latitude` | decimal | Koordinat GPS |
| `longitude` | decimal | Koordinat GPS |
| `image` | file | Foto bengkel (jpg/png) |

### F3. Update Data Bengkel

```
PUT /api/bengkel/{id}
Authorization: Bearer {owner_token}
Content-Type: multipart/form-data
```

---

## G. Manajemen Layanan (Owner — `auth:owner-api`)

### G1. Daftar Layanan

```
GET /api/layanans
Authorization: Bearer {owner_token}
```

### G2. Tambah Layanan

```
POST /api/layanans
Authorization: Bearer {owner_token}
Body: {
    "name": "Ganti Kampas Kopling",
    "price": 350000
}
```

### G3. Update Layanan

```
PUT /api/layanans/{id}
Authorization: Bearer {owner_token}
Body: { "price": 400000 }
```

### G4. Hapus Layanan

```
DELETE /api/layanans/{id}
Authorization: Bearer {owner_token}
```

---

## H. Manajemen Booking Masuk (Owner — `auth:owner-api`)

### H1. Daftar Booking Masuk

```
GET /api/bengkel-booking
Authorization: Bearer {owner_token}
```

### H2. Update Status Booking

```
PUT /api/bengkel-booking/{id}
Authorization: Bearer {owner_token}
Body: { "booking_status": "CONFIRMED" }
```

**Nilai `booking_status` yang valid:**
- `CONFIRMED` — Terima booking
- `REJECTED` — Tolak booking
- `IN_PROGRESS` — Kendaraan sedang dikerjakan
- `DONE` — Selesai

---

## I. Chatbot

### Kirim Pesan ke Chatbot

```
POST /api/chatbot
```

**Request Body:**
```json
{
    "message": "1"
}
```

**Response:**
```json
{
    "messages": [
        {
            "type": "text",
            "text": "Layanan Pengiriman Produk:\n1. Informasi pickup\n2. Informasi jasa kurir"
        }
    ]
}
```

**Daftar Perintah yang Tersedia:**

| Input | Respons |
|-------|---------|
| `0` | Tampilkan menu utama |
| `1` | Sub-menu pengiriman produk |
| `2` | Informasi layanan bengkel |
| `3` | Informasi cara registrasi akun |
| lainnya | Pesan error/panduan |

---

## J. Rating

### J1. Kirim Rating

```
POST /api/ratings
Authorization: Bearer {token}
Body: {
    "product_id": 5,
    "transaction_id": 25,
    "detail_transaction_id": 30,
    "stars": 5,
    "comment": "Produk sangat bagus, sesuai deskripsi!"
}
```

### J2. Lihat Rating Produk

```
GET /api/ratings/product/{product_id}
```

---

## K. Withdraw (Penarikan Dana — `auth:owner-api`)

### Ajukan Penarikan Dana

```
POST /api/withdraw-requests
Authorization: Bearer {owner_token}
Body: {
    "amount": 500000,
    "name": "Joko Widodo",
    "bank": "BCA",
    "number": "1234567890"
}
```

---

## L. Webhook Midtrans

### Callback Pembayaran

```
POST /api/midtrans-callback
```

Endpoint ini dipanggil otomatis oleh server Midtrans setelah pembayaran berhasil atau gagal. Tidak perlu autentikasi karena dipanggil dari server Midtrans.

**Aksi yang dilakukan:**
- Memverifikasi signature key dari Midtrans
- Update `payment_status` di tabel `transactions`
- Trigger notifikasi ke bengkel/penjual (jika ada)

---

## M. Data Referensi (Publik)

### M1. Daftar Kecamatan

```
GET /api/service/kecamatan
```

### M2. Daftar Kelurahan

```
GET /api/service/kelurahan/{kecamatan_id}
```

### M3. Daftar Spesialisasi

```
GET /api/specialists
```

### M4. Daftar Kategori Produk

```
GET /api/categories
```

### M5. Daftar Merk Mobil

```
GET /api/merk-mobil
```

### M6. Daftar Jadwal Bengkel

```
GET /api/jadwals
Authorization: Bearer {token}
```
