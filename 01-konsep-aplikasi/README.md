# Bab 1 — Konsep Aplikasi

## Daftar Isi

1. [Latar Belakang & Tujuan](#latar-belakang--tujuan)
2. [Arsitektur Sistem](./arsitektur-sistem.md)
3. [Alur Pengguna](./alur-pengguna.md)

---

## Latar Belakang & Tujuan

### Permasalahan

Pemilik kendaraan di Indonesia sering kesulitan menemukan bengkel yang terpercaya, memiliki jadwal yang sesuai, dan menawarkan layanan yang dibutuhkan. Di sisi lain, pemilik bengkel kesulitan menjangkau lebih banyak pelanggan secara digital.

### Solusi

**Bengkelin** hadir sebagai platform marketplace bengkel yang:
- Mempertemukan pengguna dan pemilik bengkel dalam satu ekosistem digital
- Memudahkan proses booking layanan bengkel secara online
- Menyediakan marketplace produk/suku cadang otomotif
- Dilengkapi chatbot untuk membantu pengguna baru memahami cara penggunaan aplikasi

### Tujuan Penelitian

1. Merancang dan membangun sistem marketplace bengkel berbasis mobile
2. Mengimplementasikan fitur booking layanan dengan manajemen jadwal
3. Mengintegrasikan payment gateway (Midtrans) untuk transaksi yang aman
4. Membangun chatbot berbasis rule-based menggunakan BotMan
5. Men-deploy backend menggunakan Docker di server VPS

---

## Aktor dan Peran

Sistem Bengkelin memiliki **tiga aktor** utama:

| Aktor | Aplikasi | Peran |
|-------|----------|-------|
| **User** | flutter_bengkelin_user | Mencari bengkel, melakukan booking, membeli produk |
| **Owner Bengkel** | bengkelin_owner_flutter | Mengelola bengkel, menerima booking, mengelola produk/layanan |
| **Admin** | Web Dashboard (Laravel) | Mengelola seluruh data, verifikasi, dan konfigurasi sistem |

---

## Ruang Lingkup Fitur

### Aplikasi User
- Registrasi & Login dengan email
- Lupa password (OTP via email)
- Pencarian bengkel berdasarkan kecamatan/kelurahan
- Pencarian bengkel terdekat (geolocation/radius)
- Detail bengkel: layanan, jadwal, produk, spesialisasi
- Booking layanan bengkel
- Pembelian produk (marketplace)
- Keranjang belanja
- Checkout & pembayaran via Midtrans
- Riwayat booking & transaksi
- Rating dan ulasan
- Chatbot interaktif

### Aplikasi Owner Bengkel
- Registrasi & Login pemilik bengkel
- Lupa password (OTP via email)
- Manajemen profil & data bengkel
- Manajemen layanan (CRUD)
- Manajemen produk (CRUD)
- Manajemen jadwal operasional
- Kelola spesialisasi dan merk kendaraan yang ditangani
- Menerima & memperbarui status booking
- Menerima & memproses transaksi
- Pengajuan penarikan dana (withdraw)
