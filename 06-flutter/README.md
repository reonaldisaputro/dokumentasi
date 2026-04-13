# Bab 6 — Desain UI & Slicing Flutter

## Daftar Isi

1. [Tahap Desain UI](./design-ui.md)
2. [Implementasi Slicing Flutter](./slicing-flutter.md)

---

## Gambaran Umum

Bengkelin memiliki **dua aplikasi mobile** yang dibangun dengan Flutter:

| Aplikasi | Folder | Pengguna |
|----------|--------|----------|
| Aplikasi User | `flutter_bengkelin_user` | Pelanggan / pemilik kendaraan |
| Aplikasi Owner | `bengkelin_owner_flutter` | Pemilik bengkel |

### Arsitektur Aplikasi: MVVM

Kedua aplikasi menggunakan pola **MVVM (Model-View-ViewModel)**:

```
┌──────────┐     ┌─────────────┐     ┌────────────────┐
│  Model   │◄────│  ViewModel  │◄────│     View       │
│          │     │             │     │  (UI / Widget) │
│  Data &  │────►│  Business   │────►│                │
│  API     │     │  Logic      │     │  Menampilkan   │
│  calls   │     │             │     │  data &        │
└──────────┘     └─────────────┘     │  menerima input│
                                     └────────────────┘
```

```
lib/
├── model/        ← Struktur data (dari JSON API)
├── viewmodel/    ← Logic: panggil API, olah data
├── views/        ← Halaman UI (Screen)
├── widget/       ← Komponen UI yang bisa dipakai ulang
├── config/       ← Konfigurasi: warna, endpoint, network
└── main.dart     ← Entry point aplikasi
```

### Stack Teknologi Flutter

| Package | Fungsi |
|---------|--------|
| `dio` | HTTP client untuk panggil API |
| `google_fonts` | Font Poppins |
| `shared_preferences` | Simpan data ringan (token, setting) |
| `flutter_secure_storage` | Simpan token secara aman (terenkripsi) |
| `webview_flutter` | Tampilkan halaman Midtrans Snap |
| `geolocator` | Ambil lokasi GPS user |
| `fluttertoast` | Notifikasi toast |
| `flutter_dotenv` | Load konfigurasi dari file `.env` |
