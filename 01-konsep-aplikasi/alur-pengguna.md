# Alur Pengguna (User Flow)

## 1. Alur Registrasi & Login

### User (Pengguna Biasa)

```
[Buka Aplikasi]
      │
      ▼
[Layar Splash / Onboarding]
      │
      ├── [Sudah punya akun?] ──→ [Form Login]
      │                                │
      │                           [Masukkan Email & Password]
      │                                │
      │                           [POST /api/login]
      │                                │
      │                     ┌──────────┴──────────┐
      │                     ▼                     ▼
      │                 [Sukses]             [Gagal]
      │                     │                     │
      │              [Simpan Token]         [Tampilkan Error]
      │              [Masuk ke Home]
      │
      └── [Belum punya akun?] ──→ [Form Registrasi]
                                       │
                                  [Isi: Nama, Email,
                                   Password, Telepon,
                                   Alamat, Kecamatan,
                                   Kelurahan]
                                       │
                                  [POST /api/register]
                                       │
                                  [Token diterima]
                                       │
                                  [Masuk ke Home]
```

### Owner (Pemilik Bengkel)

```
[Buka Aplikasi Owner]
      │
      ▼
[Form Login / Register Owner]
      │
      ├── [Login] ──→ [POST /api/login-owner]
      │                     │
      │              [auth:owner-api token]
      │              [Masuk ke Dashboard Owner]
      │
      └── [Register] ──→ [Form: Nama, Email, Password, Telepon]
                              │
                         [POST /api/register-owner]
                              │
                         [Token diterima]
                         [Masuk ke Dashboard Owner]
```

---

## 2. Alur Booking Layanan (User)

```
[Home] → [Cari Bengkel]
              │
         ┌────┴────────────────────────────────┐
         │                                     │
    [Cari by Lokasi]                   [Cari Bengkel Terdekat]
    GET /api/service?kecamatan=...     GET /api/bengkel/nearby
         │                                     │
         └──────────────┬──────────────────────┘
                        │
                   [Pilih Bengkel]
                   GET /api/bengkel/{id}
                        │
                   [Lihat Detail Bengkel]
                   - Nama, Deskripsi, Lokasi
                   - Daftar Layanan
                   - Jadwal Operasional
                   - Spesialisasi & Merk Kendaraan
                   - Rating
                        │
                   [Pilih Layanan]
                        │
                   [Pilih Tanggal & Waktu]
                   GET /api/booking/{bengkel_id}/booked-times
                   (untuk melihat slot yang sudah terisi)
                        │
                   [Isi Data Kendaraan]
                   - Brand, Model, Plat Nomor
                   - Tahun Pembuatan
                   - Kilometer
                   - Transmisi (Manual/Matic)
                        │
                   [Tambahkan Catatan (opsional)]
                        │
                   [Konfirmasi Booking]
                   POST /api/booking
                        │
               ┌────────┴────────┐
               ▼                 ▼
           [Sukses]          [Gagal]
               │                 │
        [Booking dibuat]    [Tampilkan pesan error]
        [Status: PENDING]
               │
        [Notifikasi ke Owner]
               │
        [Owner Update Status]
        → CONFIRMED / REJECTED
               │
        [User mendapat update]
               │
        [Layanan Selesai]
        [Status: DONE]
               │
        [User bisa memberikan Rating]
        POST /api/ratings
```

---

## 3. Alur Transaksi Produk (User)

```
[Home/Marketplace]
      │
[Browsing Produk]
GET /api/products
      │
[Lihat Detail Produk]
GET /api/products/{id}
      │
[Tambah ke Keranjang]
POST /api/cart/add
      │
[Lihat Keranjang]
GET /api/cart
      │
[Update/Hapus Item]
PUT /api/cart/{id}
DELETE /api/cart/{id}
      │
[Checkout]
POST /api/checkout (membuat transaksi & mendapat snap_token Midtrans)
      │
[Halaman Pembayaran Midtrans]
(WebView atau in-app payment)
      │
      ├── [Bayar Sukses]
      │        │
      │   [Midtrans kirim webhook]
      │   POST /api/midtrans-callback
      │        │
      │   [payment_status → PAID]
      │        │
      │   [Notifikasi ke Bengkel/Seller]
      │
      └── [Batal/Gagal]
               │
          [payment_status → FAILED/CANCELLED]
```

---

## 4. Alur Manajemen Bengkel (Owner)

```
[Login Owner]
      │
[Dashboard Owner]
      │
      ├── [Setup Bengkel] (jika belum ada)
      │   POST /api/bengkel
      │   - Nama Bengkel
      │   - Deskripsi
      │   - Alamat, Kecamatan, Kelurahan
      │   - Koordinat (Latitude, Longitude)
      │   - Upload Foto
      │
      ├── [Tambah Layanan]
      │   POST /api/layanans
      │   - Nama layanan, Harga, Deskripsi
      │
      ├── [Tambah Produk]
      │   POST /api/products-owner
      │   - Nama produk, Harga, Stok, Foto
      │
      ├── [Atur Jadwal]
      │   POST /api/jadwals
      │   - Hari, Jam Buka, Jam Tutup
      │
      ├── [Kelola Booking Masuk]
      │   GET /api/bengkel-booking
      │   PUT /api/bengkel-booking/{id}
      │   → Update status: CONFIRMED / REJECTED / DONE
      │
      ├── [Kelola Transaksi]
      │   GET /api/transaction-owner
      │   PUT /api/transaction-owner/{id}
      │   → Update shipping_status
      │
      └── [Ajukan Penarikan Dana]
          POST /api/withdraw-requests
```

---

## 5. Alur Chatbot

```
[User membuka fitur Chatbot]
      │
[Tampil Menu Utama]
POST /api/chatbot { "message": "0" }
      │
┌─────────────────────────────────────┐
│ Selamat datang di menu utama.       │
│ Pilih opsi berikut:                 │
│ 1. Pengiriman Produk                │
│ 2. Layanan Bengkel                  │
│ 3. Cara Mendaftar Akun              │
│ 0. Kembali ke Menu Utama            │
└─────────────────────────────────────┘
      │
      ├── Input "1" → Sub-menu Pengiriman Produk
      │                    │
      │               ├── Input "1" → Info Pickup
      │               └── Input "2" → Info Jasa Kurir
      │
      ├── Input "2" → LayananBengkelConversation
      │               (percakapan multi-step tentang layanan)
      │
      ├── Input "3" → Info cara registrasi
      │               (link ke halaman register)
      │
      └── Input lain → "Harap masukkan angka pilihan yang valid"
```

---

## 6. Ringkasan Status Booking

| Status | Deskripsi | Diset Oleh |
|--------|-----------|------------|
| `PENDING` | Booking baru dibuat, menunggu konfirmasi | Sistem (otomatis) |
| `CONFIRMED` | Owner telah menerima booking | Owner |
| `REJECTED` | Owner menolak booking | Owner |
| `IN_PROGRESS` | Kendaraan sedang dalam perbaikan | Owner |
| `DONE` | Layanan selesai | Owner |
| `CANCELLED` | Dibatalkan oleh user | User |

## 7. Ringkasan Status Transaksi (Payment)

| Status | Deskripsi |
|--------|-----------|
| `PENDING` | Transaksi dibuat, menunggu pembayaran |
| `PAID` / `settlement` | Pembayaran berhasil (dari Midtrans) |
| `FAILED` | Pembayaran gagal |
| `CANCELLED` | Transaksi dibatalkan |
| `EXPIRED` | Waktu pembayaran habis |
