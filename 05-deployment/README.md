# Bab 5 — Deployment

## Daftar Isi

1. [Konsep Deployment](#konsep-deployment)
2. [Deployment ke VPS](./vps-deployment.md)

---

## Konsep Deployment

**Deployment** adalah proses mempublikasikan aplikasi dari lingkungan pengembangan (lokal) ke server yang bisa diakses pengguna nyata (produksi).

### Perbedaan Environment

| Aspek | Development (Lokal) | Production (Server) |
|-------|---------------------|---------------------|
| `APP_ENV` | `local` | `production` |
| `APP_DEBUG` | `true` | `false` |
| `APP_URL` | `http://localhost:8000` | `https://domain.com` |
| Web Server | `php artisan serve` | Nginx + PHP-FPM |
| Proses Daemon | Manual | PM2 |
| HTTPS | Tidak | Ya (Certbot / Let's Encrypt) |
| Error display | Detail lengkap | Pesan umum saja |
| Cache | Tidak dicache | Config & route dicache |

### Stack Deployment Bengkelin

```
Internet
    │
    ▼
┌──────────────────────────────────────┐
│           VPS (Ubuntu)               │
│                                      │
│  ┌──────────────────────────────┐    │
│  │   Nginx (Port 80 / 443)      │    │  ← Web server + reverse proxy
│  │   + Certbot (SSL)            │    │
│  └──────────────┬───────────────┘    │
│                 │ FastCGI            │
│  ┌──────────────▼───────────────┐    │
│  │   PHP-FPM (PHP 8.x)          │    │  ← Proses PHP application
│  └──────────────────────────────┘    │
│                                      │
│  ┌──────────────────────────────┐    │
│  │   MySQL 8.0                  │    │  ← Database
│  └──────────────────────────────┘    │
│                                      │
│  ┌──────────────────────────────┐    │
│  │   PM2                        │    │  ← Daemon manager untuk
│  │   - queue:work               │    │    background Laravel processes
│  │   - schedule:run             │    │
│  └──────────────────────────────┘    │
└──────────────────────────────────────┘
```

### Peran Setiap Komponen

| Komponen | Peran |
|----------|-------|
| **Nginx** | Menerima request HTTP/HTTPS, melayani file statis, meneruskan request PHP ke PHP-FPM |
| **PHP-FPM** | Menjalankan kode PHP Laravel secara efisien sebagai proses terpisah |
| **MySQL** | Menyimpan seluruh data aplikasi |
| **PM2** | Menjaga proses background Laravel (queue worker, scheduler) tetap berjalan sebagai daemon |
| **Certbot** | Mendapatkan dan memperpanjang sertifikat SSL/TLS dari Let's Encrypt secara otomatis |
