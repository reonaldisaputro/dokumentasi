# Dokumentasi Teknis Aplikasi Bengkelin

> **Judul Skripsi:** Pengembangan Marketplace Bengkel Mobil Berbasis Mobile dengan Fitur Chatbot
>
> **Teknologi Utama:** Laravel 9 (Backend), Flutter (Mobile), MySQL (Database), Docker (Deployment)

---

## Daftar Isi

| No | Bab | Deskripsi |
|----|-----|-----------|
| 1 | [Konsep Aplikasi](./01-konsep-aplikasi/README.md) | Gambaran umum, arsitektur sistem, dan alur pengguna |
| 2 | [Database](./02-database/README.md) | Desain ERD, skema tabel, dan relasi antar entitas |
| 3 | [Backend](./03-backend/README.md) | Struktur proyek, setup environment, autentikasi, dan chatbot |
| 4 | [API](./04-api/README.md) | Konsep REST API dan dokumentasi seluruh endpoint |
| 5 | [Deployment](./05-deployment/README.md) | Panduan deployment VPS dengan Nginx, PM2, dan Certbot |
| 6 | [Flutter UI & Slicing](./06-flutter/README.md) | Tahap desain UI (Figma) hingga implementasi slicing Flutter |

---

## Ringkasan Proyek

**Bengkelin** adalah aplikasi marketplace bengkel mobil berbasis mobile yang mempertemukan pengguna (pemilik kendaraan) dengan bengkel-bengkel di sekitar mereka. Aplikasi ini memiliki dua sisi:

- **Aplikasi User** (`flutter_bengkelin_user`) — Untuk pemilik kendaraan yang ingin mencari dan memesan layanan bengkel.
- **Aplikasi Owner** (`bengkelin_owner_flutter`) — Untuk pemilik bengkel yang ingin mendaftarkan dan mengelola bengkelnya.

### Fitur Utama

| Fitur | Deskripsi |
|-------|-----------|
| Pencarian Bengkel | Cari bengkel berdasarkan lokasi, kategori kendaraan, atau spesialisasi |
| Booking Layanan | Reservasi jadwal servis dengan memilih layanan yang diinginkan |
| Marketplace Produk | Jual beli produk/suku cadang otomotif |
| Chatbot | Asisten virtual untuk menjawab pertanyaan seputar penggunaan aplikasi |
| Pembayaran Online | Integrasi Midtrans untuk pembayaran yang aman |
| Rating & Ulasan | Sistem penilaian untuk bengkel dan layanan |
| Penarikan Dana | Fitur withdraw untuk pemilik bengkel |

### Stack Teknologi

```
┌─────────────────────────────────────────────────────┐
│                    FRONTEND                         │
│   flutter_bengkelin_user   bengkelin_owner_flutter  │
│         (Flutter)                (Flutter)          │
└─────────────────┬───────────────────┬───────────────┘
                  │    REST API       │
                  ▼                   ▼
┌─────────────────────────────────────────────────────┐
│                    BACKEND                          │
│              Laravel 9 (PHP 8.x)                    │
│         Laravel Sanctum (Authentication)            │
│         BotMan (Chatbot Engine)                     │
│         Midtrans (Payment Gateway)                  │
└─────────────────────┬───────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│                   DATABASE                          │
│                  MySQL 8.0                          │
└─────────────────────────────────────────────────────┘
```

---

## Cara Membaca Dokumentasi Ini

Dokumentasi ini dibagi menjadi 5 bab yang disarankan dibaca secara berurutan:

1. **Mulai dari Bab 1** untuk memahami konsep dan arsitektur besar aplikasi.
2. **Lanjut ke Bab 2** untuk memahami struktur data dan database.
3. **Baca Bab 3** untuk implementasi backend secara teknis.
4. **Bab 4** menjelaskan seluruh endpoint API yang tersedia.
5. **Bab 5** adalah panduan untuk men-deploy aplikasi ke server produksi.
