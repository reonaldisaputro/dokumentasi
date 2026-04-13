# Arsitektur Sistem Bengkelin

## 1. Gambaran Arsitektur Keseluruhan

Bengkelin menggunakan arsitektur **Client-Server** dengan pola **REST API**. Backend berperan sebagai *single source of truth* yang melayani dua aplikasi mobile secara bersamaan.

```
┌───────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                             │
│                                                                   │
│   ┌──────────────────────┐       ┌──────────────────────┐         │
│   │  flutter_bengkelin_  │       │ bengkelin_owner_      │         │
│   │  user (Flutter)      │       │ flutter (Flutter)     │         │
│   │  - Aplikasi User     │       │ - Aplikasi Owner      │         │
│   └──────────┬───────────┘       └──────────┬────────────┘         │
│              │                              │                      │
└──────────────┼──────────────────────────────┼──────────────────────┘
               │       HTTPS / REST API       │
               ▼                              ▼
┌───────────────────────────────────────────────────────────────────┐
│                         SERVER LAYER                              │
│                                                                   │
│   ┌──────────────────────────────────────────────────────────┐    │
│   │              NGINX (Web Server / Reverse Proxy)          │    │
│   │              Port: 80/443                                │    │
│   └────────────────────────┬─────────────────────────────────┘    │
│                            │                                      │
│   ┌────────────────────────▼─────────────────────────────────┐    │
│   │            Laravel 9 Application (PHP-FPM)               │    │
│   │                                                          │    │
│   │   ┌─────────────┐  ┌──────────┐  ┌──────────────────┐   │    │
│   │   │  Routes     │  │ Sanctum  │  │   Controllers    │   │    │
│   │   │  api.php    │→ │  Auth    │→ │  (Business       │   │    │
│   │   │             │  │  Guard   │  │   Logic)         │   │    │
│   │   └─────────────┘  └──────────┘  └────────┬─────────┘   │    │
│   │                                           │              │    │
│   │   ┌────────────────────────────────────────▼──────────┐  │    │
│   │   │  Services & Models (Eloquent ORM)                 │  │    │
│   │   └───────────────────────────────────────────────────┘  │    │
│   │                                                          │    │
│   │   ┌──────────────┐  ┌───────────┐  ┌───────────────┐    │    │
│   │   │   BotMan     │  │ Midtrans  │  │  File Storage │    │    │
│   │   │  (Chatbot)   │  │ (Payment) │  │  (Laravel     │    │    │
│   │   │              │  │           │  │   Storage)    │    │    │
│   │   └──────────────┘  └───────────┘  └───────────────┘    │    │
│   └──────────────────────────────────────────────────────────┘    │
│                                                                   │
│   ┌──────────────────────────────────────────────────────────┐    │
│   │                    MySQL 8.0 Database                    │    │
│   └──────────────────────────────────────────────────────────┘    │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

---

## 2. Arsitektur Backend (Laravel MVC)

Laravel menggunakan pola **MVC (Model-View-Controller)** yang dimodifikasi untuk kebutuhan API:

```
HTTP Request
     │
     ▼
┌─────────────────────────────────────────────────────────────────┐
│                     routes/api.php                              │
│   Mendefinisikan semua endpoint dan mengarahkan ke Controller   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Middleware                                │
│   - auth:sanctum         → Verifikasi token JWT                 │
│   - auth:owner-api       → Verifikasi token Owner               │
│   - throttle             → Rate limiting                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Controller (app/Http/Controllers/API/)      │
│                                                                 │
│   Bertanggung jawab untuk:                                      │
│   - Validasi input request                                      │
│   - Memanggil Model/Service                                     │
│   - Mengembalikan respons JSON terstruktur                      │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Model (app/Models/)                        │
│                                                                 │
│   Eloquent ORM — Representasi tabel database dalam PHP          │
│   Mendefinisikan relasi antar tabel (hasMany, belongsTo, dll)   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     MySQL Database                              │
└─────────────────────────────────────────────────────────────────┘
```

### Pola Response API

Semua response API menggunakan helper `ResponseFormatter` dengan format yang konsisten:

```json
// Response Sukses
{
    "meta": {
        "code": 200,
        "status": "success",
        "message": "Data berhasil diambil"
    },
    "data": { ... }
}

// Response Error
{
    "meta": {
        "code": 422,
        "status": "error",
        "message": "Validasi gagal"
    },
    "data": {
        "email": ["Email sudah terdaftar"]
    }
}
```

---

## 3. Arsitektur Autentikasi

Bengkelin menggunakan **Laravel Sanctum** dengan dua guard terpisah:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SISTEM AUTENTIKASI                           │
│                                                                 │
│  Guard: auth:sanctum          Guard: auth:owner-api             │
│  ┌────────────────────┐       ┌────────────────────┐            │
│  │    Tabel: users    │       │ Tabel: pemilik_     │            │
│  │                    │       │ bengkels            │            │
│  │  - id              │       │  - id               │            │
│  │  - name            │       │  - name             │            │
│  │  - email           │       │  - email            │            │
│  │  - password        │       │  - password         │            │
│  └────────────────────┘       └────────────────────┘            │
│                                                                 │
│  Token disimpan di tabel: personal_access_tokens                │
│  Format Header: Authorization: Bearer {token}                   │
└─────────────────────────────────────────────────────────────────┘
```

**Kenapa dua guard?**

Karena User dan Owner adalah entitas yang berbeda dengan hak akses berbeda:
- `User` → tabel `users` → akses ke fitur booking, marketplace
- `Owner` → tabel `pemilik_bengkels` → akses ke fitur manajemen bengkel

---

## 4. Arsitektur Chatbot (BotMan)

Chatbot menggunakan **BotMan** dengan driver Web (komunikasi via HTTP POST):

```
Mobile App
    │
    │  POST /api/chatbot
    │  Body: { "message": "1" }
    ▼
┌───────────────────────────────────────────┐
│           ChatbotController               │
│                                           │
│   BotMan::hears('{message}')              │
│        │                                 │
│        ├─ '1' → Menu Pengiriman Produk   │
│        ├─ '2' → LayananBengkelConversation│
│        ├─ '3' → Info Registrasi           │
│        └─ '0' → Menu Utama               │
│                                           │
│   Conversation (percakapan multi-step):   │
│   - LayananBengkelConversation            │
│   - PengirimProdukConversation            │
└───────────────────────────────────────────┘
    │
    ▼
Response JSON → Ditampilkan di aplikasi mobile
```

Chatbot bersifat **rule-based** (berbasis aturan), artinya setiap input pengguna dipetakan ke jawaban yang sudah ditentukan. Ini berbeda dengan chatbot berbasis AI/NLP.

---

## 5. Arsitektur Payment Gateway (Midtrans)

```
User App           Backend (Laravel)         Midtrans Server
   │                      │                       │
   │  POST /checkout       │                       │
   │─────────────────────→│                       │
   │                      │  Create transaction   │
   │                      │──────────────────────→│
   │                      │                       │
   │                      │  Return snap_token    │
   │                      │←──────────────────────│
   │  Return snap_token   │                       │
   │←─────────────────────│                       │
   │                      │                       │
   │  [User bayar via Midtrans Snap]              │
   │────────────────────────────────────────────→│
   │                      │                       │
   │                      │  POST /midtrans-      │
   │                      │  callback (webhook)   │
   │                      │←──────────────────────│
   │                      │                       │
   │                      │  Update payment_status│
   │                      │  di database          │
```

---

## 6. Kontainerisasi dengan Docker

Seluruh backend berjalan dalam **Docker containers** yang terisolasi:

```
docker-compose.yml
┌────────────────────────────────────────────────────┐
│                                                    │
│  ┌──────────────┐  ┌──────────────┐               │
│  │  nginx-web   │  │  laravel-app │               │
│  │  Port: 8000  │  │  PHP-FPM     │               │
│  │  (web server)│←→│  (backend)   │               │
│  └──────────────┘  └──────┬───────┘               │
│                           │                       │
│  ┌──────────────┐  ┌──────▼───────┐               │
│  │  phpmyadmin  │  │  mysql-db    │               │
│  │  Port: 8080  │  │  Port: 3307  │               │
│  │  (DB GUI)    │←→│  (database)  │               │
│  └──────────────┘  └──────────────┘               │
│                                                    │
│  Network: laravel (internal bridge network)        │
└────────────────────────────────────────────────────┘
```

Setiap service berkomunikasi melalui **internal Docker network** bernama `laravel`, sehingga tidak ada port yang terbuka ke publik kecuali yang didefinisikan di `ports:`.
