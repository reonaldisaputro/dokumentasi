# Bab 5 — Deployment

## Daftar Isi

1. [Konsep Deployment](#konsep-deployment)
2. [Deployment dengan Docker](./docker.md)
3. [Deployment ke VPS](./vps-deployment.md)

---

## Konsep Deployment

**Deployment** adalah proses mempublikasikan aplikasi dari lingkungan pengembangan (lokal) ke server yang bisa diakses oleh pengguna nyata (produksi).

### Perbedaan Environment

| Aspek | Development (Lokal) | Production (Server) |
|-------|---------------------|---------------------|
| `APP_ENV` | `local` | `production` |
| `APP_DEBUG` | `true` | `false` |
| `APP_URL` | `http://localhost:8000` | `https://domain.com` |
| Database | MySQL lokal | MySQL server |
| File storage | Local disk | Server/cloud |
| Error display | Detail lengkap | Pesan umum saja |
| Cache | Tidak dicache | Config & route dicache |

### Strategi Deployment Bengkelin

Bengkelin menggunakan **Docker** untuk membungkus aplikasi dan seluruh dependensinya ke dalam container yang konsisten:

```
Lokal (Mac/Windows/Linux)
    ↓  push ke Git
Server VPS (Ubuntu)
    ↓  git pull
    ↓  docker-compose up
Aplikasi berjalan di server
    ↓  
Bisa diakses via: https://api.bengkelin.com
```

**Keuntungan menggunakan Docker:**
1. **Konsistensi** — Aplikasi berjalan sama persis di semua environment
2. **Isolasi** — Tiap service (PHP, Nginx, MySQL) terpisah, tidak konflik
3. **Portabilitas** — Mudah dipindah ke server lain
4. **Skalabilitas** — Mudah ditambah container jika traffic meningkat
