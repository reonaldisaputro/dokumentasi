# Bab 4 — API (Application Programming Interface)

## Daftar Isi

1. [Konsep REST API](./konsep-rest-api.md)
2. [Dokumentasi Endpoint](./dokumentasi-endpoint.md)

---

## Gambaran Umum API Bengkelin

Bengkelin mengekspos **REST API** yang dikonsumsi oleh kedua aplikasi mobile (user dan owner). Semua endpoint dimulai dengan prefix `/api/`.

### Base URL

```
Development : http://localhost:8000/api
Production  : https://[domain-server]/api
```

### Format Response

Semua response menggunakan format JSON dengan struktur yang konsisten:

```json
{
    "meta": {
        "code": 200,
        "status": "success",
        "message": "Keterangan"
    },
    "data": { ... }
}
```

### Autentikasi

Endpoint yang memerlukan autentikasi menggunakan header:

```
Authorization: Bearer {access_token}
```

Token diperoleh dari endpoint login.

---

## Ringkasan Endpoint

### Endpoint Publik (Tanpa Autentikasi)

| Method | Endpoint | Fungsi |
|--------|----------|--------|
| POST | `/register` | Registrasi user baru |
| POST | `/login` | Login user |
| POST | `/register-owner` | Registrasi owner baru |
| POST | `/login-owner` | Login owner |
| POST | `/chatbot` | Kirim pesan ke chatbot |
| GET | `/home` | Data halaman utama |
| GET | `/products` | Daftar semua produk |
| GET | `/products/{id}` | Detail produk |
| GET | `/bengkel` | Daftar bengkel |
| GET | `/bengkel/{id}` | Detail bengkel |
| GET | `/bengkel/nearby` | Bengkel terdekat (geolocation) |
| GET | `/specialists` | Daftar spesialisasi |
| GET | `/categories` | Daftar kategori produk |
| GET | `/merk-mobil` | Daftar merk mobil |
| POST | `/user/forgot-password/send-otp` | Kirim OTP reset password user |
| POST | `/user/forgot-password/verify-otp` | Verifikasi OTP user |
| POST | `/user/forgot-password/reset-password` | Reset password user |
| POST | `/owner/forgot-password/send-otp` | Kirim OTP reset password owner |
| POST | `/owner/forgot-password/verify-otp` | Verifikasi OTP owner |
| POST | `/owner/forgot-password/reset-password` | Reset password owner |
| POST | `/midtrans-callback` | Webhook callback Midtrans |

### Endpoint User (Memerlukan `auth:sanctum`)

| Method | Endpoint | Fungsi |
|--------|----------|--------|
| GET | `/profile` | Lihat profil user |
| PUT | `/profile` | Update profil user |
| GET | `/profile/bookings` | Riwayat booking user |
| GET | `/profile/bookings/{id}` | Detail booking |
| GET | `/profile/transactions` | Riwayat transaksi user |
| GET | `/profile/transactions/{id}` | Detail transaksi |
| GET | `/user/bookings` | Daftar booking milik user |
| GET | `/user/bookings/{id}` | Detail booking user |
| POST | `/booking` | Buat booking baru |
| GET | `/booking/{bengkel_id}/booked-times` | Cek slot waktu yang sudah terpesan |
| GET | `/cart` | Lihat keranjang |
| POST | `/cart/add` | Tambah item ke keranjang |
| PUT | `/cart/{id}` | Update item keranjang |
| DELETE | `/cart/{id}` | Hapus item dari keranjang |
| POST | `/checkout` | Proses pembayaran |
| POST | `/ratings` | Kirim rating |
| GET | `/ratings/product/{product}` | Lihat rating produk |
| GET | `/withdraw-requests` | Daftar pengajuan withdraw |
| POST | `/withdraw-requests` | Buat pengajuan withdraw |

### Endpoint Owner (Memerlukan `auth:owner-api`)

| Method | Endpoint | Fungsi |
|--------|----------|--------|
| GET | `/owner/profile` | Lihat profil owner |
| POST | `/owner/logout` | Logout owner |
| GET | `/my-bengkel` | Data bengkel milik owner |
| POST | `/bengkel` | Daftarkan bengkel baru |
| PUT | `/bengkel/{id}` | Update data bengkel |
| DELETE | `/bengkel/{id}` | Hapus bengkel |
| GET | `/bengkel/{id}/merk-mobil` | Lihat merk mobil bengkel |
| POST | `/bengkel/{id}/merk-mobil` | Tambah merk mobil bengkel |
| PUT | `/bengkel/{id}/merk-mobil` | Update merk mobil bengkel |
| DELETE | `/bengkel/{id}/merk-mobil/{merkMobilId}` | Hapus merk mobil bengkel |
| GET | `/jadwals` | Lihat jadwal operasional |
| POST | `/jadwals` | Tambah jadwal |
| PUT | `/jadwals/{id}` | Update jadwal |
| DELETE | `/jadwals/{id}` | Hapus jadwal |
| GET | `/layanans` | Lihat daftar layanan |
| POST | `/layanans` | Tambah layanan |
| PUT | `/layanans/{id}` | Update layanan |
| DELETE | `/layanans/{id}` | Hapus layanan |
| GET | `/products-owner` | Lihat produk milik bengkel |
| POST | `/products-owner` | Tambah produk |
| PUT | `/products-owner/{id}` | Update produk |
| DELETE | `/products-owner/{id}` | Hapus produk |
| GET | `/bengkel-booking` | Lihat booking masuk |
| GET | `/bengkel-booking/{id}` | Detail booking |
| PUT | `/bengkel-booking/{id}` | Update status booking |
| GET | `/transaction-owner` | Lihat transaksi |
| GET | `/transaction-owner/{id}` | Detail transaksi |
| PUT | `/transaction-owner/{id}` | Update status pengiriman |
