# Setup Environment Backend

## 1. Persyaratan Sistem

| Komponen | Versi Minimum |
|----------|---------------|
| PHP | 8.0 |
| Composer | 2.x |
| MySQL | 8.0 |
| Node.js | 16.x (untuk build assets) |
| Docker | 20.x (untuk deployment) |
| Docker Compose | 2.x |

---

## 2. Instalasi Lokal (Tanpa Docker)

### Langkah 1: Clone Proyek

```bash
cd /path/to/project
```

### Langkah 2: Install Dependensi PHP

```bash
composer install
```

### Langkah 3: Salin File Environment

```bash
cp .env.example .env
```

### Langkah 4: Generate Application Key

```bash
php artisan key:generate
```

Perintah ini mengisi `APP_KEY` di file `.env` dengan nilai random yang digunakan untuk enkripsi.

### Langkah 5: Konfigurasi File `.env`

Buka file `.env` dan sesuaikan nilai-nilai berikut:

```env
# ── Konfigurasi Aplikasi ──────────────────────────────────
APP_NAME=Bengkelin
APP_ENV=local            # local / production
APP_KEY=                 # diisi otomatis oleh key:generate
APP_DEBUG=true           # false di production
APP_URL=http://localhost:8000

# ── Konfigurasi Database ──────────────────────────────────
DB_CONNECTION=mysql
DB_HOST=127.0.0.1        # Jika pakai Docker: nama service (mysql-db)
DB_PORT=3306
DB_DATABASE=bengkelin
DB_USERNAME=root
DB_PASSWORD=your_password

# ── Konfigurasi Email (untuk OTP & Notifikasi) ───────────
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your@gmail.com
MAIL_PASSWORD=your_app_password   # Google App Password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=your@gmail.com
MAIL_FROM_NAME="Bengkelin"

# ── Konfigurasi Midtrans ──────────────────────────────────
MIDTRANS_SERVER_KEY=SB-Mid-server-xxxxxxxxxxxx
MIDTRANS_CLIENT_KEY=SB-Mid-client-xxxxxxxxxxxx
MIDTRANS_IS_PRODUCTION=false   # true untuk live
MIDTRANS_IS_SANITIZED=true
MIDTRANS_IS_3DS=true

# ── Konfigurasi Filesystem ───────────────────────────────
FILESYSTEM_DISK=public
```

### Langkah 6: Jalankan Migrasi Database

```bash
# Buat database terlebih dahulu di MySQL
mysql -u root -p -e "CREATE DATABASE bengkelin;"

# Jalankan migrasi
php artisan migrate

# Opsional: isi data awal (seeder)
php artisan db:seed
```

### Langkah 7: Buat Symbolic Link Storage

```bash
php artisan storage:link
```

Perintah ini membuat link `public/storage` → `storage/app/public` agar file upload bisa diakses via URL.

### Langkah 8: Jalankan Development Server

```bash
php artisan serve
```

API siap diakses di `http://localhost:8000/api/`

---

## 3. Konfigurasi Guard Autentikasi

Laravel menggunakan konfigurasi di `config/auth.php` untuk mendefinisikan guard autentikasi. Bengkelin menambahkan guard khusus untuk owner:

```php
// config/auth.php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'sanctum',
        'provider' => 'users',
    ],

    // Guard untuk pemilik bengkel
    'owner-api' => [
        'driver' => 'sanctum',
        'provider' => 'owners',
    ],
],

'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],

    // Provider untuk pemilik bengkel
    'owners' => [
        'driver' => 'eloquent',
        'model' => App\Models\PemilikBengkel::class,
    ],
],
```

---

## 4. Konfigurasi CORS

Karena aplikasi mobile mengakses API dari domain/origin yang berbeda, CORS (Cross-Origin Resource Sharing) harus dikonfigurasi dengan benar.

File: `config/cors.php`

```php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],

    'allowed_methods' => ['*'],

    'allowed_origins' => ['*'],   // Di production, ganti dengan domain spesifik

    'allowed_origins_patterns' => [],

    'allowed_headers' => ['*'],

    'exposed_headers' => [],

    'max_age' => 0,

    'supports_credentials' => false,
];
```

---

## 5. Konfigurasi Sanctum

File: `config/sanctum.php`

```php
return [
    // Domain yang diizinkan untuk autentikasi
    'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
        '%s%s',
        'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
        env('APP_URL') ? ','.parse_url(env('APP_URL'), PHP_URL_HOST) : ''
    ))),

    'guard' => ['web'],

    // Token tidak expired secara default (null = tidak ada expiry)
    'expiration' => null,

    'middleware' => [
        'verify_csrf_token' => App\Http\Middleware\VerifyCsrfToken::class,
        'encrypt_cookies' => App\Http\Middleware\EncryptCookies::class,
    ],
];
```

---

## 6. Konfigurasi Midtrans

Midtrans dikonfigurasi di `config/midtrans.php` (atau langsung di `.env`):

```php
// Contoh penggunaan di CheckoutController
\Midtrans\Config::$serverKey = env('MIDTRANS_SERVER_KEY');
\Midtrans\Config::$isProduction = env('MIDTRANS_IS_PRODUCTION', false);
\Midtrans\Config::$isSanitized = env('MIDTRANS_IS_SANITIZED', true);
\Midtrans\Config::$is3ds = env('MIDTRANS_IS_3DS', true);
```

---

## 7. Perintah Artisan yang Sering Digunakan

```bash
# Melihat semua route yang terdaftar
php artisan route:list

# Melihat route yang terdaftar (filter untuk API)
php artisan route:list --path=api

# Membuat controller baru
php artisan make:controller API/NamaController

# Membuat model baru + migration
php artisan make:model NamaModel -m

# Membuat migration baru
php artisan make:migration add_kolom_to_tabel_name_table

# Jalankan ulang semua migration + seeder
php artisan migrate:fresh --seed

# Cache konfigurasi (untuk production)
php artisan config:cache
php artisan route:cache

# Bersihkan cache
php artisan cache:clear
php artisan config:clear
php artisan route:clear
```

---

## 8. Struktur File `.env.example`

File `.env.example` adalah template yang disertakan di repository. Ini **tidak** berisi nilai sensitif dan aman untuk di-commit ke git. Saat setup baru, developer menyalin file ini dan mengisinya sendiri.

```bash
cp .env.example .env
# kemudian edit .env sesuai kebutuhan
```

> **Penting:** File `.env` TIDAK boleh di-commit ke repository karena berisi kredensial database, API key, dan informasi sensitif lainnya. Pastikan `.env` ada di `.gitignore`.
