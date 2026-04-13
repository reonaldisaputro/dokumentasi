# Tahap Desain UI

## 1. Alur Tahap Desain

Sebelum menulis satu baris kode Flutter, desain tampilan dikerjakan terlebih dahulu menggunakan tools desain. Urutan prosesnya:

```
Identifikasi Halaman      →    Wireframe         →    UI Design (Figma)
(Fitur apa saja            (Sketsa kasar          (Desain berwarna,
 yang butuh halaman?)       layout & alur)         tipografi, aset)
                                                          │
                                                          ▼
                                                  Handoff ke Developer
                                                  (export aset, ukuran,
                                                   warna, font)
```

---

## 2. Tools yang Digunakan

| Tools | Fungsi |
|-------|--------|
| **Figma** | Desain UI utama (wireframe + high-fidelity mockup) |
| **FigJam** | Mapping alur pengguna (user flow diagram) |
| **Figma Inspect** | Melihat nilai CSS/ukuran/warna untuk developer |

---

## 3. Identifikasi Halaman

Sebelum desain dimulai, daftar semua halaman yang dibutuhkan berdasarkan fitur:

### Aplikasi User (`flutter_bengkelin_user`)

| No | Nama Halaman | Fungsi |
|----|-------------|--------|
| 1 | Splash Screen | Loading awal, cek status login |
| 2 | Login | Masuk dengan email & password |
| 3 | Register | Daftar akun baru |
| 4 | Forgot Password (3 langkah) | OTP → verifikasi → reset password |
| 5 | Home | Beranda: bengkel unggulan, produk, banner |
| 6 | Service Page | Daftar bengkel berdasarkan lokasi |
| 7 | Service Category | Filter berdasarkan kategori/spesialisasi |
| 8 | Bengkel Detail | Profil bengkel lengkap |
| 9 | Booking Form | Form isi data kendaraan & pilih jadwal |
| 10 | Product Page | Daftar semua produk marketplace |
| 11 | Product Detail | Detail satu produk |
| 12 | Cart | Keranjang belanja |
| 13 | Checkout | Ringkasan pesanan & proses pembayaran |
| 14 | Payment WebView | Halaman pembayaran Midtrans Snap |
| 15 | List Booking | Riwayat semua booking |
| 16 | Booking Detail | Detail satu booking |
| 17 | List Transaksi | Riwayat semua transaksi |
| 18 | Transaction Detail | Detail satu transaksi |
| 19 | Profile | Profil user |
| 20 | Edit Profile | Form ubah data profil |
| 21 | Chat Assistant | Chatbot |

### Aplikasi Owner (`bengkelin_owner_flutter`)

| No | Nama Halaman | Fungsi |
|----|-------------|--------|
| 1 | Splash Screen | Loading awal |
| 2 | Login | Masuk sebagai owner |
| 3 | Register | Daftar akun owner |
| 4 | Forgot Password | Reset password owner |
| 5 | Home/Dashboard | Ringkasan data bengkel |
| 6 | Bengkel List/Create/Edit | CRUD data bengkel |
| 7 | Layanan List/Create/Edit | CRUD layanan |
| 8 | Jadwal List/Form | Kelola jam operasional |
| 9 | Produk List/Create/Edit | CRUD produk |
| 10 | Booking List/Detail | Kelola booking masuk |
| 11 | Transaksi List/Detail | Kelola transaksi |
| 12 | Withdraw List/Create | Pengajuan pencairan dana |
| 13 | Profile | Profil owner |

---

## 4. Design System

Sebelum mendesain halaman satu per satu, tentukan **Design System** — panduan konsisten untuk seluruh tampilan:

### Palet Warna (dari `AppColor`)

| Nama | Hex | Penggunaan |
|------|-----|------------|
| `colorPrimaryBrown` | `#964A1B` | Warna utama (tombol, aksen) |
| `colorSecondaryBrown` | `#D06F32` | Warna sekunder |
| `colorPrimaryBlue` | `#2D5BA6` | Aksi penting, link |
| `whiteBg` | `#F3EFEE` | Background halaman |
| `colortextBlack` | `#272727` | Teks utama |
| `colorGrey` | `#A5A5A5` | Teks placeholder / keterangan |
| `colorRed` | `#CA1919` | Error, hapus, status negatif |
| `colorSuccess` | `#76F278` | Status sukses |

### Tipografi

Seluruh teks menggunakan **font Poppins** (Google Fonts):

| Style | Weight | Penggunaan |
|-------|--------|------------|
| `poppinsBold` | 700 | Judul halaman, heading |
| `poppinsSemiBold` | 600 | Sub-judul, label penting |
| `poppinsMedium` | 500 | Body text utama |
| `poppinsRegular` | 400 | Teks keterangan, placeholder |

### Komponen Berulang

Identifikasi komponen yang muncul di banyak halaman agar didesain sekali dan dipakai ulang:

- **Card Bengkel** — foto, nama, lokasi, rating
- **Card Produk** — foto, nama, harga, tombol tambah cart
- **Card Booking** — bengkel, tanggal, status badge
- **Input Field** — text field dengan style konsisten
- **Primary Button** — tombol aksi utama (coklat)
- **Status Badge** — chip warna sesuai status booking/transaksi
- **AppBar kustom** — header halaman

---

## 5. Wireframe vs High-Fidelity

### Wireframe (Lo-Fi)

Dibuat cepat untuk memvalidasi layout dan alur **tanpa** memperhatikan warna atau detail visual:

```
┌─────────────────┐
│  [Logo]  [Menu] │   ← AppBar
├─────────────────┤
│ ┌─────────────┐ │
│ │   Banner    │ │   ← Hero banner
│ └─────────────┘ │
│                 │
│ Bengkel Terdekat│   ← Section title
│ ┌────┐  ┌────┐ │
│ │card│  │card│ │   ← Bengkel cards
│ └────┘  └────┘ │
│                 │
│ Produk Terbaru  │
│ ┌────┐  ┌────┐ │
│ │card│  │card│ │
│ └────┘  └────┘ │
├─────────────────┤
│ [Home][Srvc][Pf]│   ← Bottom Navigation
└─────────────────┘
```

### High-Fidelity Mockup

Setelah wireframe disetujui, dibuat desain final di Figma dengan:
- Warna sesuai design system
- Font Poppins dengan ukuran dan weight yang tepat
- Foto/ikon placeholder
- Padding dan spacing yang konsisten (kelipatan 4px atau 8px)
- Status bar, safe area iOS/Android diperhatikan

---

## 6. Panduan Handoff Desain ke Flutter

Setelah desain selesai, developer membaca desain Figma menggunakan **Figma Inspect** untuk mendapatkan:

| Informasi | Cara Baca di Figma | Kode Flutter |
|-----------|-------------------|--------------|
| Ukuran & posisi | Inspect panel → W, H, X, Y | `width:`, `height:`, `Padding` |
| Warna | Inspect → Fill color | `Color(0xFF...)` |
| Font size | Inspect → Text → Size | `fontSize: 16` |
| Font weight | Inspect → Text → Weight | `FontWeight.w600` |
| Border radius | Inspect → Corner radius | `BorderRadius.circular(12)` |
| Jarak antar elemen | Spacing di Figma | `SizedBox`, `Padding`, `gap` |
| Aset gambar | Export sebagai PNG/SVG | Simpan ke `assets/` |
