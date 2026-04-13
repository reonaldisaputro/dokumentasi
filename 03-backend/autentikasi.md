# Autentikasi & Otorisasi

## 1. Konsep Autentikasi di Bengkelin

Bengkelin menggunakan **Laravel Sanctum** untuk autentikasi API berbasis token. Sanctum menghasilkan **Personal Access Token** — sebuah string acak yang disimpan di database dan dikirim bersama setiap request sebagai bukti identitas.

### Perbedaan dengan Session-based Auth

| Aspek | Session-based | Token-based (Sanctum) |
|-------|---------------|----------------------|
| Penyimpanan | Cookie di browser | Header `Authorization: Bearer {token}` |
| Cocok untuk | Web aplikasi | Mobile app, SPA, API |
| State | Stateful | Stateless |
| Expiry | Berbasis session server | Bisa dikonfigurasi, default: tidak expired |

---

## 2. Dua Guard Autentikasi

Bengkelin memiliki **dua jenis pengguna** dengan guard terpisah:

```
┌─────────────────────────────────────────────────────────┐
│   Guard: auth:sanctum       Guard: auth:owner-api       │
│   Model: User               Model: PemilikBengkel        │
│   Tabel: users              Tabel: pemilik_bengkels      │
│                                                         │
│   Endpoint:                 Endpoint:                   │
│   POST /api/login           POST /api/login-owner       │
│   POST /api/register        POST /api/register-owner    │
└─────────────────────────────────────────────────────────┘
```

Semua token dari kedua guard disimpan di tabel `personal_access_tokens` dengan kolom `tokenable_type` yang membedakan apakah token milik `User` atau `PemilikBengkel`.

---

## 3. Implementasi AuthController

### 3.1 Registrasi User

```php
// POST /api/register
public function register(Request $request)
{
    // 1. Validasi input
    $request->validate([
        'name'         => 'required|string|max:100',
        'email'        => 'required|string|email|max:100|unique:users,email',
        'password'     => ['required', 'confirmed', Rules\Password::defaults()],
        'phone_number' => 'required|string',
        'alamat'       => 'required|string',
        'kecamatan_id' => 'required|exists:kecamatans,id',
        'kelurahan_id' => 'required|exists:kelurahans,id',
    ]);

    // 2. Buat user baru (password di-hash otomatis via model)
    $user = User::create([
        'name'         => $request->name,
        'email'        => $request->email,
        'password'     => Hash::make($request->password),
        'alamat'       => $request->alamat,
        'phone_number' => $request->phone_number,
        'kecamatan_id' => $request->kecamatan_id,
        'kelurahan_id' => $request->kelurahan_id,
    ]);

    // 3. Buat token untuk user baru
    $token = $user->createToken('auth_token')->plainTextToken;

    // 4. Return response sukses
    return ResponseFormatter::success([
        'access_token' => $token,
        'token_type'   => 'Bearer',
        'user'         => $user,
    ], 'User registered successfully');
}
```

### 3.2 Login User

```php
// POST /api/login
public function login(Request $request)
{
    // 1. Validasi input
    $request->validate([
        'email'    => ['required', 'email'],
        'password' => ['required'],
    ]);

    // 2. Cari user berdasarkan email
    $user = User::where('email', $request->email)->first();

    // 3. Verifikasi password
    if (!$user || !Hash::check($request->password, $user->password)) {
        return ResponseFormatter::error(
            null,
            'Email atau password salah',
            401
        );
    }

    // 4. Hapus token lama (opsional, untuk single session)
    $user->tokens()->delete();

    // 5. Buat token baru
    $token = $user->createToken('auth_token')->plainTextToken;

    return ResponseFormatter::success([
        'access_token' => $token,
        'token_type'   => 'Bearer',
        'user'         => $user,
    ], 'Login berhasil');
}
```

### 3.3 Logout

```php
// POST /api/logout (dengan middleware auth:sanctum)
public function logout(Request $request)
{
    // Hapus token yang sedang digunakan
    $request->user()->currentAccessToken()->delete();

    return ResponseFormatter::success(null, 'Logout berhasil');
}
```

---

## 4. Cara Menggunakan Token di Mobile

Setelah login berhasil, aplikasi mobile menyimpan `access_token` di local storage (SharedPreferences / Hive). Setiap request yang memerlukan autentikasi harus menyertakan token di header:

```
Authorization: Bearer 1|abcdefghijklmnopqrstuvwxyz1234567890
```

Contoh dalam Flutter (Dart):

```dart
import 'package:http/http.dart' as http;

final response = await http.get(
  Uri.parse('https://api.bengkelin.com/api/profile'),
  headers: {
    'Authorization': 'Bearer $token',
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
);
```

---

## 5. Middleware Autentikasi

Middleware digunakan di `routes/api.php` untuk melindungi endpoint:

```php
// Endpoint yang HANYA bisa diakses oleh User yang sudah login
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/profile', [ProfileUserController::class, 'show']);
    Route::post('/booking', [BookingController::class, 'store']);
    // ...
});

// Endpoint yang HANYA bisa diakses oleh Owner yang sudah login
Route::middleware('auth:owner-api')->group(function () {
    Route::get('/my-bengkel', [BengkelController::class, 'myBengkel']);
    Route::post('/bengkel', [BengkelController::class, 'store']);
    // ...
});
```

Jika request dilakukan tanpa token atau dengan token tidak valid, Sanctum akan otomatis mengembalikan response:

```json
{
    "message": "Unauthenticated."
}
```

---

## 6. Fitur Lupa Password (OTP)

Bengkelin mengimplementasikan reset password menggunakan **OTP (One-Time Password)** 6 digit yang dikirim via email.

### Alur Reset Password

```
1. User kirim email ke:  POST /api/user/forgot-password/send-otp
         ↓
2. Sistem buat OTP 6 digit → simpan ke tabel password_reset_otps
   (OTP expire setelah 10 menit)
         ↓
3. Kirim OTP ke email user via SMTP (Laravel Mail)
         ↓
4. User masukkan OTP ke: POST /api/user/forgot-password/verify-otp
         ↓
5. Sistem verifikasi OTP (cek kecocokan + belum expired)
   → tandai is_verified = true
         ↓
6. User masukkan password baru ke: POST /api/user/forgot-password/reset-password
         ↓
7. Sistem update password di tabel users
   → hapus OTP dari tabel password_reset_otps
```

### Implementasi ForgotPasswordUserController

```php
// Langkah 1: Kirim OTP
public function sendOtp(Request $request)
{
    $request->validate(['email' => 'required|email|exists:users,email']);

    // Generate OTP 6 digit
    $otp = str_pad(random_int(0, 999999), 6, '0', STR_PAD_LEFT);

    // Simpan ke database
    PasswordResetOtp::updateOrCreate(
        ['email' => $request->email, 'user_type' => 'user'],
        [
            'otp'         => $otp,
            'is_verified' => false,
            'expires_at'  => now()->addMinutes(10),
        ]
    );

    // Kirim via email
    Mail::to($request->email)->send(new OtpMail($otp));

    return ResponseFormatter::success(null, 'OTP berhasil dikirim ke email');
}

// Langkah 2: Verifikasi OTP
public function verifyOtp(Request $request)
{
    $request->validate([
        'email' => 'required|email',
        'otp'   => 'required|string|size:6',
    ]);

    $record = PasswordResetOtp::where([
        'email'     => $request->email,
        'otp'       => $request->otp,
        'user_type' => 'user',
    ])->first();

    if (!$record || now()->isAfter($record->expires_at)) {
        return ResponseFormatter::error(null, 'OTP tidak valid atau sudah expired', 422);
    }

    $record->update(['is_verified' => true]);

    return ResponseFormatter::success(null, 'OTP valid');
}

// Langkah 3: Reset Password
public function resetPassword(Request $request)
{
    $request->validate([
        'email'    => 'required|email',
        'password' => 'required|confirmed|min:8',
    ]);

    // Pastikan OTP sudah diverifikasi
    $record = PasswordResetOtp::where([
        'email'       => $request->email,
        'user_type'   => 'user',
        'is_verified' => true,
    ])->first();

    if (!$record) {
        return ResponseFormatter::error(null, 'Verifikasi OTP diperlukan', 403);
    }

    // Update password
    User::where('email', $request->email)
        ->update(['password' => Hash::make($request->password)]);

    // Hapus record OTP
    $record->delete();

    return ResponseFormatter::success(null, 'Password berhasil direset');
}
```

---

## 7. Keamanan Token

### Praktik yang Diterapkan

1. **Password di-hash** menggunakan `bcrypt` via `Hash::make()` — tidak pernah disimpan sebagai plaintext.
2. **Token dihapus saat logout** — tidak bisa digunakan lagi setelah user keluar.
3. **OTP memiliki waktu kedaluwarsa** — 10 menit, setelah itu tidak valid.
4. **Guard terpisah** — token user tidak bisa digunakan untuk mengakses endpoint owner, dan sebaliknya.
5. **Validasi input** — setiap request divalidasi ketat sebelum diproses.

### Format Token Sanctum

Token Sanctum memiliki format:

```
{id}|{random_string}
```

Contoh: `1|7Xy9MnZabcdefghijklmnopqrstuvwxyz12345`

- Bagian `{id}` adalah ID record di tabel `personal_access_tokens`
- Bagian `{random_string}` adalah token aktual yang di-hash di database

Ketika divalidasi, Laravel:
1. Memisahkan ID dari token
2. Mencari record di `personal_access_tokens` berdasarkan ID
3. Membandingkan hash dari token yang dikirim dengan yang tersimpan
