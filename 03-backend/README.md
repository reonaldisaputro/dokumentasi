# Bab 3 — Backend

## Daftar Isi

1. [Struktur Proyek](./struktur-proyek.md)
2. [Setup Environment](./setup-environment.md)
3. [Autentikasi & Otorisasi](./autentikasi.md)
4. [Implementasi Chatbot](./chatbot.md)

---

## Gambaran Umum Backend

Backend Bengkelin dibangun menggunakan **Laravel 9** — framework PHP berbasis MVC yang menyediakan ekosistem lengkap untuk membangun REST API. Berikut ringkasan teknologi yang digunakan:

| Teknologi | Versi | Fungsi |
|-----------|-------|--------|
| PHP | 8.0+ | Bahasa pemrograman utama |
| Laravel | 9.x | Framework web/API |
| Laravel Sanctum | 2.14 | Autentikasi token-based API |
| BotMan | 2.8 | Engine chatbot berbasis rule |
| Midtrans PHP | 2.5 | Payment gateway |
| MySQL | 8.0 | Database |
| Nginx | Latest | Web server & reverse proxy |
| Docker | 20.x | Kontainerisasi |

---

## Dependensi (composer.json)

```json
{
    "require": {
        "php": "^8.0",
        "laravel/framework": "^9.0",
        "laravel/sanctum": "^2.14",
        "botman/botman": "^2.8",
        "botman/driver-web": "^1.5",
        "midtrans/midtrans-php": "^2.5",
        "guzzlehttp/guzzle": "^7.2",
        "fruitcake/laravel-cors": "^2.0.5",
        "barryvdh/laravel-dompdf": "^2.0",
        "doctrine/dbal": "^3.6",
        "nesbot/carbon": "^2.71"
    }
}
```

### Penjelasan Dependensi

| Package | Fungsi |
|---------|--------|
| `laravel/sanctum` | Autentikasi token untuk SPA/Mobile API |
| `botman/botman` | Core library chatbot |
| `botman/driver-web` | Driver untuk chatbot berbasis HTTP (Web/Mobile) |
| `midtrans/midtrans-php` | SDK resmi Midtrans untuk payment gateway |
| `guzzlehttp/guzzle` | HTTP client untuk panggilan API eksternal |
| `fruitcake/laravel-cors` | Mengelola header CORS agar API bisa diakses dari mobile |
| `barryvdh/laravel-dompdf` | Generate PDF (untuk laporan/nota) |
| `doctrine/dbal` | Dibutuhkan untuk modifikasi kolom di migration |
