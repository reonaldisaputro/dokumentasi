# Struktur Proyek Backend (Laravel)

## Pohon Direktori

```
bengkelin/
├── app/
│   ├── Console/              # Artisan commands kustom
│   ├── Conversations/        # Kelas percakapan BotMan (chatbot)
│   │   ├── LayananBengkelConversation.php
│   │   └── PengirimProdukConversation.php
│   ├── Enums/                # Enumerasi nilai konstan
│   │   ├── BookingStatus.php
│   │   └── BookingType.php
│   ├── Exceptions/           # Handler exception kustom
│   │   └── Handler.php
│   ├── Helpers/              # Kelas utilitas
│   │   └── ResponseFormatter.php   ← Standarisasi format JSON response
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── API/          # Semua controller REST API
│   │   │   │   ├── AuthController.php
│   │   │   │   ├── BengkelBookingController.php
│   │   │   │   ├── BengkelController.php
│   │   │   │   ├── BengkelTransactionController.php
│   │   │   │   ├── BookingController.php
│   │   │   │   ├── CartController.php
│   │   │   │   ├── CategoryController.php
│   │   │   │   ├── ChatbotController.php
│   │   │   │   ├── ChatApiController.php
│   │   │   │   ├── CheckoutController.php
│   │   │   │   ├── DashboardController.php
│   │   │   │   ├── ForgotPasswordOwnerController.php
│   │   │   │   ├── ForgotPasswordUserController.php
│   │   │   │   ├── JadwalController.php
│   │   │   │   ├── LayananController.php
│   │   │   │   ├── MerkMobilController.php
│   │   │   │   ├── PageController.php
│   │   │   │   ├── ProductController.php
│   │   │   │   ├── ProfileUserController.php
│   │   │   │   ├── RatingController.php
│   │   │   │   ├── ServiceController.php
│   │   │   │   ├── SpecialistController.php
│   │   │   │   └── WithdrawRequestController.php
│   │   │   └── (web controllers untuk admin panel)
│   │   ├── Kernel.php        # Registrasi middleware
│   │   └── Middleware/       # Middleware kustom
│   ├── Models/               # Eloquent ORM models
│   │   ├── Admin.php
│   │   ├── Bengkel.php
│   │   ├── BengkelCart.php
│   │   ├── Booking.php
│   │   ├── Cart.php
│   │   ├── Category.php
│   │   ├── CategoryKendaraan.php
│   │   ├── DetailLayananBooking.php
│   │   ├── DetailTransaction.php
│   │   ├── Jadwal.php
│   │   ├── Kecamatan.php
│   │   ├── Kelurahan.php
│   │   ├── Layanan.php
│   │   ├── MerkMobil.php
│   │   ├── PemilikBengkel.php
│   │   ├── Product.php
│   │   ├── Rating.php
│   │   ├── Specialist.php
│   │   ├── Transaction.php
│   │   ├── User.php
│   │   └── WithdrawRequest.php
│   ├── Notifications/        # Notifikasi email
│   ├── Providers/            # Service providers
│   └── Services/             # Business logic layer
├── bootstrap/                # Bootstrap aplikasi Laravel
├── config/                   # Konfigurasi aplikasi
│   ├── app.php
│   ├── auth.php              ← Konfigurasi guard autentikasi
│   ├── cors.php              ← Konfigurasi CORS
│   ├── database.php
│   └── sanctum.php
├── database/
│   ├── factories/            # Factory untuk testing/seeding
│   ├── migrations/           # File migrasi database
│   └── seeders/              # Data awal
├── docker/
│   ├── nginx/
│   │   └── default.conf      # Konfigurasi Nginx
│   └── php/
│       └── Dockerfile        # Dockerfile PHP-FPM
├── public/                   # Web root (entry point)
│   └── index.php
├── resources/                # Views blade (admin panel)
├── routes/
│   ├── api.php               ← Semua endpoint API terdefinisi di sini
│   ├── web.php               ← Route web (admin panel)
│   └── console.php
├── storage/
│   └── app/public/
│       ├── bengkel/          # Foto bengkel
│       └── products/         # Foto produk
├── tests/                    # Unit dan feature tests
├── .env                      # Environment variables (rahasia)
├── .env.example              # Template environment variables
├── artisan                   # CLI Laravel
├── composer.json             # Dependensi PHP
├── docker-compose.yml        # Konfigurasi Docker
└── phpunit.xml               # Konfigurasi testing
```

---

## Penjelasan Komponen Utama

### 1. `app/Http/Controllers/API/`

Berisi semua controller yang menangani request API. Setiap controller bertanggung jawab atas satu domain/entitas:

| Controller | Domain |
|-----------|--------|
| `AuthController` | Registrasi, login, logout (User & Owner) |
| `BengkelController` | CRUD bengkel, pencarian, nearby |
| `BookingController` | Buat & lihat booking (dari sisi user) |
| `BengkelBookingController` | Kelola booking masuk (dari sisi owner) |
| `LayananController` | CRUD layanan bengkel |
| `JadwalController` | CRUD jadwal operasional |
| `ProductController` | CRUD produk (dari sisi owner) |
| `PageController` | Halaman publik (home, daftar produk) |
| `CartController` | Keranjang belanja (dari sisi user) |
| `CheckoutController` | Proses pembayaran Midtrans |
| `BengkelTransactionController` | Transaksi dari sisi owner |
| `ChatbotController` | Handler chatbot BotMan |
| `ProfileUserController` | Profil & riwayat user |
| `WithdrawRequestController` | Pengajuan penarikan dana |
| `RatingController` | Submit & baca rating |
| `ForgotPasswordUserController` | Reset password via OTP (user) |
| `ForgotPasswordOwnerController` | Reset password via OTP (owner) |

### 2. `app/Helpers/ResponseFormatter.php`

Helper class untuk memastikan semua response API memiliki format yang konsisten:

```php
class ResponseFormatter
{
    protected static $response = [
        'meta' => [
            'code'    => 200,
            'status'  => 'success',
            'message' => null,
        ],
        'data' => null,
    ];

    public static function success($data = null, $message = null)
    {
        self::$response['meta']['code']    = 200;
        self::$response['meta']['status']  = 'success';
        self::$response['meta']['message'] = $message;
        self::$response['data']            = $data;

        return response()->json(self::$response, 200);
    }

    public static function error($data = null, $message = null, $code = 400)
    {
        self::$response['meta']['code']    = $code;
        self::$response['meta']['status']  = 'error';
        self::$response['meta']['message'] = $message;
        self::$response['data']            = $data;

        return response()->json(self::$response, $code);
    }
}
```

### 3. `app/Enums/BookingStatus.php`

Mendefinisikan nilai-nilai enum untuk status booking agar konsisten:

```php
enum BookingStatus: string
{
    case PENDING     = 'PENDING';
    case CONFIRMED   = 'CONFIRMED';
    case REJECTED    = 'REJECTED';
    case IN_PROGRESS = 'IN_PROGRESS';
    case DONE        = 'DONE';
    case CANCELLED   = 'CANCELLED';
}
```

### 4. `app/Conversations/`

Berisi kelas percakapan multi-langkah untuk chatbot BotMan:

```php
class LayananBengkelConversation extends Conversation
{
    public function run()
    {
        $this->askLayanan();
    }

    private function askLayanan()
    {
        $this->ask('Informasi apa yang Anda butuhkan tentang layanan bengkel?', function(Answer $answer) {
            // Proses jawaban dan tindak lanjut
        });
    }
}
```

### 5. `routes/api.php`

File ini adalah inti dari definisi API. Semua endpoint didefinisikan di sini dengan struktur:

```php
// Endpoint publik (tidak perlu autentikasi)
Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);

// Endpoint yang membutuhkan autentikasi User
Route::middleware('auth:sanctum')->group(function () {
    // ...endpoint user...
});

// Endpoint yang membutuhkan autentikasi Owner
Route::middleware('auth:owner-api')->group(function () {
    // ...endpoint owner...
});
```

---

## Cara Kerja Request Lifecycle

Setiap HTTP request yang masuk ke backend melewati tahapan berikut:

```
1. Request HTTP masuk ke Nginx (port 80/443)
         ↓
2. Nginx meneruskan ke PHP-FPM (FastCGI)
         ↓
3. public/index.php — entry point Laravel
         ↓
4. Bootstrap aplikasi (Service Container, Service Providers)
         ↓
5. HTTP Kernel — jalankan middleware global
   (CORS, throttle, dll)
         ↓
6. Router (routes/api.php) — cocokkan URL ke handler
         ↓
7. Middleware route-level
   (auth:sanctum atau auth:owner-api jika diperlukan)
         ↓
8. Controller — validasi input, panggil Model/Service
         ↓
9. Model (Eloquent) — query database MySQL
         ↓
10. Controller — format response menggunakan ResponseFormatter
         ↓
11. Response JSON dikirim kembali ke client
```
