# Implementasi Slicing Flutter

## 1. Setup Proyek Flutter

### Buat Proyek Baru

```bash
flutter create flutter_bengkelin_user
flutter create bengkelin_owner_flutter
```

### Tambahkan Dependensi (`pubspec.yaml`)

```yaml
dependencies:
  flutter:
    sdk: flutter
  google_fonts: ^6.2.1       # Font Poppins
  dio: ^5.8.0                # HTTP client
  flutter_dotenv: ^5.2.1     # Load .env config
  flutter_secure_storage: ^9.2.4  # Simpan token
  shared_preferences: ^2.0.15     # Simpan data ringan
  fluttertoast: ^8.2.12      # Notifikasi toast
  webview_flutter: ^4.13.0   # Midtrans Snap payment
  geolocator: ^14.0.2        # GPS / lokasi user
  intl: ^0.20.2              # Format tanggal & angka
  http: ^1.4.0               # HTTP requests
  logger: ^2.6.0             # Debug logging
```

```bash
flutter pub get
```

### Struktur Folder

```
lib/
├── config/
│   ├── app_color.dart      ← Warna & TextStyle global
│   ├── endpoint.dart       ← Semua URL API
│   ├── network.dart        ← HTTP client (Dio)
│   └── pref.dart           ← Session / token storage
├── model/                  ← Data class dari JSON
├── viewmodel/              ← Logic & API calls
├── views/                  ← Semua halaman (Screen)
├── widget/                 ← Komponen reusable
└── main.dart
```

---

## 2. Konfigurasi Dasar

### `config/app_color.dart` — Design System

Definisikan semua warna dan style teks di satu file agar konsisten di seluruh aplikasi:

```dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

class AppColor {
  // Primary
  static const Color colorPrimaryBrown   = Color(0xFF964A1B);
  static const Color colorSecondaryBrown = Color(0xFFD06F32);
  static const Color colorPrimaryBlue    = Color(0xFF2D5BA6);

  // Background
  static const Color whiteBg = Color(0xFFF3EFEE);

  // Text
  static const Color colortextBlack = Color(0xFF272727);
  static const Color colorGrey      = Color(0xFFA5A5A5);

  // Status
  static const Color colorRed     = Color(0xFFCA1919);
  static const Color colorSuccess = Color(0xFF76F278);
}

// TextStyle siap pakai
TextStyle poppinsBold     = GoogleFonts.poppins(fontWeight: FontWeight.w700);
TextStyle poppinsSemiBold = GoogleFonts.poppins(fontWeight: FontWeight.w600);
TextStyle poppinsMedium   = GoogleFonts.poppins(fontWeight: FontWeight.w500);
TextStyle poppinsRegular  = GoogleFonts.poppins(fontWeight: FontWeight.w400);
```

### `config/endpoint.dart` — URL API

```dart
class Endpoint {
  static const String authLoginUrl    = '/api/login';
  static const String authRegisterUrl = '/api/register';
  static const String bengkelUrl      = '/api/bengkel';
  static const String productUrl      = '/api/products';
  static const String bookingUrl      = '/api/booking';
  static const String cartUrl         = '/api/cart';
  static const String checkoutUrl     = '/api/checkout';
  // ... tambahkan endpoint lain
}
```

### `config/network.dart` — HTTP Client (Dio)

```dart
import 'package:dio/dio.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

final baseUrl = dotenv.env['BASEURL_STAGING_CLIENT']!;

class Network {
  static Future<dynamic> postApi(String url, dynamic body) async {
    var dio = Dio(BaseOptions(baseUrl: baseUrl));
    Response res = await dio.post(url, data: body);
    return res.data;
  }

  static Future<dynamic> getApiWithHeaders(
      String url, Map<String, dynamic> headers) async {
    var dio = Dio(BaseOptions(baseUrl: baseUrl, headers: headers));
    Response res = await dio.get(url);
    return res.data;
  }

  // Tambahkan postApiWithHeaders, putApi, deleteApi sesuai kebutuhan
}
```

### `config/pref.dart` — Manajemen Token

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class Session {
  final _storage = const FlutterSecureStorage();

  Future<void> saveUserToken(String token) async {
    await _storage.write(key: 'user_token', value: token);
  }

  Future<String?> getUserToken() async {
    return await _storage.read(key: 'user_token');
  }

  Future<void> clearSession() async {
    await _storage.deleteAll();
  }
}
```

---

## 3. Membuat Model (Data Class)

Model adalah representasi data yang diterima dari API. Dibuat dari struktur JSON response.

**Contoh: `model/user_model.dart`**

```dart
// Dari JSON response:
// { "id": 1, "name": "Budi", "email": "budi@mail.com" }

class UserModel {
  final int id;
  final String name;
  final String email;
  final String? phoneNumber;

  UserModel({
    required this.id,
    required this.name,
    required this.email,
    this.phoneNumber,
  });

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id:          json['id'],
      name:        json['name'],
      email:       json['email'],
      phoneNumber: json['phone_number'],
    );
  }
}
```

**Contoh: `model/bengkel_model.dart`**

```dart
class BengkelModel {
  final int id;
  final String name;
  final String? imageUrl;
  final String alamat;
  final double? latitude;
  final double? longitude;

  BengkelModel({
    required this.id,
    required this.name,
    this.imageUrl,
    required this.alamat,
    this.latitude,
    this.longitude,
  });

  factory BengkelModel.fromJson(Map<String, dynamic> json) {
    return BengkelModel(
      id:        json['id'],
      name:      json['name'],
      imageUrl:  json['image_url'],
      alamat:    json['alamat'],
      latitude:  json['latitude']?.toDouble(),
      longitude: json['longitude']?.toDouble(),
    );
  }
}
```

---

## 4. Membuat ViewModel

ViewModel bertugas memanggil API dan mengolah data sebelum ditampilkan ke View.

**Contoh: `viewmodel/auth_viewmodel.dart`**

```dart
import 'dart:io';
import 'package:flutter/material.dart';
import '../config/endpoint.dart';
import '../config/network.dart';
import '../config/pref.dart';

class AuthViewmodel {
  Future<Map<String, dynamic>> login({
    required String email,
    required String password,
  }) async {
    final body = {'email': email, 'password': password};
    final resp = await Network.postApi(Endpoint.authLoginUrl, body);
    return resp;
  }

  Future<Map<String, dynamic>> register({
    required String name,
    required String email,
    required String password,
    required String phone,
    required int kecamatanId,
    required int kelurahanId,
  }) async {
    final body = {
      'name': name, 'email': email, 'password': password,
      'password_confirmation': password, 'phone_number': phone,
      'alamat': 'Jl. Contoh', 'kecamatan_id': kecamatanId,
      'kelurahan_id': kelurahanId,
    };
    final resp = await Network.postApi(Endpoint.authRegisterUrl, body);
    return resp;
  }
}
```

**Contoh: `viewmodel/bengkel_viewmodel.dart`**

```dart
import 'dart:io';
import '../config/endpoint.dart';
import '../config/network.dart';
import '../config/pref.dart';
import '../model/bengkel_model.dart';

class BengkelViewmodel {
  Future<List<BengkelModel>> fetchBengkels() async {
    String? token = await Session().getUserToken();
    final headers = {HttpHeaders.authorizationHeader: 'Bearer $token'};

    final resp = await Network.getApiWithHeaders(
      Endpoint.bengkelUrl, headers,
    );

    final List data = resp['data']['data'];
    return data.map((e) => BengkelModel.fromJson(e)).toList();
  }

  Future<BengkelModel> fetchBengkelDetail(int id) async {
    final resp = await Network.getApiWithHeaders(
      '${Endpoint.bengkelUrl}/$id', {},
    );
    return BengkelModel.fromJson(resp['data']);
  }
}
```

---

## 5. Membuat View (Halaman UI)

View menampilkan data dari ViewModel. Gunakan `StatefulWidget` jika halaman perlu state (loading, data berubah).

### Pola Umum Setiap Halaman

```dart
class NamaHalaman extends StatefulWidget {
  const NamaHalaman({super.key});

  @override
  State<NamaHalaman> createState() => _NamaHalamanState();
}

class _NamaHalamanState extends State<NamaHalaman> {
  bool _isLoading = false;
  List<dynamic> _data = [];

  @override
  void initState() {
    super.initState();
    _loadData();           // Panggil data saat halaman pertama dibuka
  }

  Future<void> _loadData() async {
    setState(() => _isLoading = true);
    try {
      // Panggil viewmodel
      final result = await NamaViewmodel().fetchData();
      setState(() => _data = result);
    } catch (e) {
      // Tampilkan pesan error
    } finally {
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: AppColor.whiteBg,
      appBar: AppBar(title: Text('Judul', style: poppinsSemiBold)),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: _data.length,
              itemBuilder: (ctx, i) => CardItem(data: _data[i]),
            ),
    );
  }
}
```

---

## 6. Contoh Slicing Halaman Nyata

### 6.1 Login Page

```dart
class LoginPage extends StatefulWidget { ... }

class _LoginPageState extends State<LoginPage> {
  final _emailCtrl    = TextEditingController();
  final _passwordCtrl = TextEditingController();
  bool _isLoading     = false;

  Future<void> _login() async {
    setState(() => _isLoading = true);
    try {
      final resp = await AuthViewmodel().login(
        email: _emailCtrl.text,
        password: _passwordCtrl.text,
      );
      // Simpan token
      await Session().saveUserToken(resp['data']['access_token']);
      // Navigasi ke home
      Navigator.pushReplacement(
        context, MaterialPageRoute(builder: (_) => const HomePage()),
      );
    } catch (e) {
      Fluttertoast.showToast(msg: 'Login gagal, cek email & password');
    } finally {
      setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: AppColor.whiteBg,
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Logo
            Image.asset('assets/logo.png', height: 80),
            const SizedBox(height: 32),

            // Email Field
            TextField(
              controller: _emailCtrl,
              keyboardType: TextInputType.emailAddress,
              decoration: InputDecoration(
                labelText: 'Email',
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(12),
                ),
              ),
            ),
            const SizedBox(height: 16),

            // Password Field
            TextField(
              controller: _passwordCtrl,
              obscureText: true,
              decoration: InputDecoration(
                labelText: 'Password',
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(12),
                ),
              ),
            ),
            const SizedBox(height: 24),

            // Tombol Login
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: _isLoading ? null : _login,
                style: ElevatedButton.styleFrom(
                  backgroundColor: AppColor.colorPrimaryBrown,
                  padding: const EdgeInsets.symmetric(vertical: 16),
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                ),
                child: _isLoading
                    ? const CircularProgressIndicator(color: Colors.white)
                    : Text('Masuk', style: poppinsSemiBold.copyWith(
                        color: Colors.white, fontSize: 16,
                      )),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

### 6.2 Card Bengkel (Reusable Widget)

Komponen yang dipakai di banyak halaman dibuat sebagai widget terpisah:

```dart
// widget/bengkel_card.dart
class BengkelCard extends StatelessWidget {
  final BengkelModel bengkel;
  final VoidCallback onTap;

  const BengkelCard({super.key, required this.bengkel, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: Container(
        margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
        decoration: BoxDecoration(
          color: Colors.white,
          borderRadius: BorderRadius.circular(12),
          boxShadow: [BoxShadow(color: Colors.black12, blurRadius: 6)],
        ),
        child: Row(
          children: [
            // Foto bengkel
            ClipRRect(
              borderRadius: const BorderRadius.only(
                topLeft: Radius.circular(12),
                bottomLeft: Radius.circular(12),
              ),
              child: Image.network(
                bengkel.imageUrl ?? '',
                width: 90, height: 90,
                fit: BoxFit.cover,
                errorBuilder: (_, __, ___) =>
                    Container(width: 90, height: 90, color: AppColor.colorGrey2),
              ),
            ),
            // Info bengkel
            Expanded(
              child: Padding(
                padding: const EdgeInsets.all(12),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(bengkel.name,
                        style: poppinsSemiBold.copyWith(fontSize: 14)),
                    const SizedBox(height: 4),
                    Text(bengkel.alamat,
                        style: poppinsRegular.copyWith(
                          fontSize: 12, color: AppColor.colorGrey,
                        ),
                        maxLines: 2, overflow: TextOverflow.ellipsis),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**Cara pakai di halaman:**
```dart
BengkelCard(
  bengkel: _bengkelList[i],
  onTap: () => Navigator.push(
    context,
    MaterialPageRoute(
      builder: (_) => BengkelDetailPage(id: _bengkelList[i].id),
    ),
  ),
)
```

---

## 7. Navigasi Antar Halaman

```dart
// Push halaman baru (bisa kembali)
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => const BengkelDetailPage()),
);

// Replace halaman saat ini (tidak bisa kembali)
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (_) => const HomePage()),
);

// Kembali ke halaman sebelumnya
Navigator.pop(context);

// Kirim data ke halaman tujuan melalui constructor
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => BookingFormPage(bengkelId: 5)),
);
```

---

## 8. Splash Screen & Cek Session

Halaman pertama yang dibuka. Tugasnya mengecek apakah user sudah login atau belum, lalu mengarahkan ke halaman yang sesuai.

```dart
class SplashScreen extends StatefulWidget { ... }

class _SplashScreenState extends State<SplashScreen> {
  @override
  void initState() {
    super.initState();
    _checkLogin();
  }

  Future<void> _checkLogin() async {
    // Tunda 2 detik untuk tampilkan splash
    await Future.delayed(const Duration(seconds: 2));

    final token = await Session().getUserToken();

    if (!mounted) return;

    // Jika token ada → ke home, jika tidak → ke login
    Navigator.pushReplacement(
      context,
      MaterialPageRoute(
        builder: (_) => token != null ? const HomePage() : const LoginPage(),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: AppColor.colorPrimaryBrown,
      body: Center(
        child: Image.asset('assets/logo_white.png', width: 160),
      ),
    );
  }
}
```

---

## 9. Menjalankan Aplikasi

```bash
# Cek device yang tersambung
flutter devices

# Jalankan di emulator/device
flutter run

# Build APK untuk testing
flutter build apk --debug

# Build APK release (siap distribusi)
flutter build apk --release
```

### File `.env` Flutter

```env
BASEURL_STAGING_CLIENT=http://[IP_SERVER]:8000
BASEURL_PRODUCTION=https://api.bengkelin.com
```

Pastikan `.env` terdaftar di `pubspec.yaml`:

```yaml
flutter:
  assets:
    - assets/
    - .env
```
