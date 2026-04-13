# Bab 2 — Database

## Daftar Isi

1. [Gambaran Umum](#gambaran-umum)
2. [Skema Tabel Lengkap](./skema-tabel.md)
3. [Desain ERD & Relasi](./desain-erd.md)

---

## Gambaran Umum

Bengkelin menggunakan **MySQL 8.0** sebagai sistem manajemen basis data relasional (RDBMS). Laravel berinteraksi dengan database melalui **Eloquent ORM** — lapisan abstraksi yang memungkinkan operasi database ditulis dalam PHP tanpa perlu menulis SQL secara langsung.

### Teknologi Database

| Komponen | Teknologi |
|----------|-----------|
| RDBMS | MySQL 8.0 |
| ORM | Laravel Eloquent |
| Migration | Laravel Database Migration |
| Seeder | Laravel Database Seeder |
| Query Builder | Laravel Query Builder / Eloquent |

### Daftar Tabel

Sistem Bengkelin terdiri dari **25 tabel** yang dibagi ke dalam beberapa kelompok:

#### Kelompok Pengguna (Authentication)
| Tabel | Fungsi |
|-------|--------|
| `users` | Data akun pengguna biasa |
| `pemilik_bengkels` | Data akun pemilik bengkel |
| `admins` | Data akun administrator |
| `personal_access_tokens` | Token autentikasi Sanctum |
| `password_reset_otps` | OTP untuk reset password |
| `password_resets` | Token reset password (default Laravel) |

#### Kelompok Bengkel & Layanan
| Tabel | Fungsi |
|-------|--------|
| `bengkels` | Data bengkel |
| `layanans` | Layanan yang disediakan bengkel |
| `jadwals` | Jadwal operasional bengkel |
| `specialists` | Master data spesialisasi bengkel |
| `bengkel_specialist` | Pivot: relasi bengkel dan spesialisasi |
| `merk_mobils` | Master data merk mobil |
| `bengkel_merk_mobil` | Pivot: relasi bengkel dan merk mobil |

#### Kelompok Lokasi
| Tabel | Fungsi |
|-------|--------|
| `kecamatans` | Data kecamatan |
| `kelurahans` | Data kelurahan |
| `category_kendaraans` | Kategori kendaraan (mobil, motor, dll) |

#### Kelompok Booking
| Tabel | Fungsi |
|-------|--------|
| `bookings` | Data booking layanan bengkel |
| `detail_layanan_bookings` | Detail layanan yang dipesan dalam booking |

#### Kelompok Transaksi & Produk
| Tabel | Fungsi |
|-------|--------|
| `products` | Produk/suku cadang yang dijual |
| `categories` | Kategori produk |
| `carts` | Keranjang belanja pengguna |
| `bengkel_carts` | Keranjang dari sisi bengkel |
| `transactions` | Data transaksi/pembayaran |
| `detail_transactions` | Detail item dalam satu transaksi |
| `ratings` | Rating dan ulasan |
| `withdraw_requests` | Permintaan penarikan dana bengkel |

#### Kelompok Sistem
| Tabel | Fungsi |
|-------|--------|
| `failed_jobs` | Antrian pekerjaan yang gagal |

---

## Konsep Migration Laravel

Migration adalah cara Laravel mengelola skema database menggunakan kode PHP. Setiap perubahan struktur tabel dicatat sebagai file migration yang bisa di-*rollback*.

```bash
# Menjalankan semua migration
php artisan migrate

# Rollback migration terakhir
php artisan migrate:rollback

# Reset dan jalankan ulang semua migration
php artisan migrate:fresh

# Jalankan migration + seeder
php artisan migrate:fresh --seed
```

### Struktur File Migration

Setiap file migration berada di `database/migrations/` dengan format penamaan:

```
{tahun}_{bulan}_{hari}_{urutan}_{nama_aksi}_{nama_tabel}.php
```

Contoh:
```
2023_05_17_152548_create_bengkels_table.php
2024_07_14_093224_create_kecamatans_table.php
2025_08_10_113720_create_ratings_table.php
```

Urutan angka di nama file menentukan urutan eksekusi migration — penting karena beberapa tabel memiliki *foreign key* yang merujuk ke tabel lain.
