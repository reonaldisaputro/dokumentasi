# Skema Tabel Database Bengkelin

> Dokumen ini menjelaskan struktur kolom setiap tabel beserta keterangan tiap kolomnya.

---

## 1. Tabel `users`

Menyimpan data akun pengguna biasa (pelanggan).

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key, auto increment |
| `name` | VARCHAR(255) | Nama lengkap pengguna |
| `email` | VARCHAR(255) UNIQUE | Alamat email (digunakan untuk login) |
| `email_verified_at` | TIMESTAMP NULL | Waktu verifikasi email |
| `phone_number` | VARCHAR(255) | Nomor telepon |
| `alamat` | LONGTEXT | Alamat lengkap |
| `kecamatan_id` | BIGINT UNSIGNED FK | Referensi ke `kecamatans.id` |
| `kelurahan_id` | BIGINT UNSIGNED FK | Referensi ke `kelurahans.id` |
| `password` | VARCHAR(255) | Password ter-hash (bcrypt) |
| `remember_token` | VARCHAR(100) NULL | Token "ingat saya" |
| `created_at` | TIMESTAMP | Waktu pembuatan akun |
| `updated_at` | TIMESTAMP | Waktu terakhir diperbarui |

---

## 2. Tabel `pemilik_bengkels`

Menyimpan data akun pemilik bengkel.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key, auto increment |
| `name` | VARCHAR(255) | Nama pemilik bengkel |
| `email` | VARCHAR(255) UNIQUE | Email untuk login |
| `email_verified_at` | TIMESTAMP NULL | Waktu verifikasi email |
| `phone_number` | VARCHAR(255) | Nomor telepon |
| `password` | VARCHAR(255) | Password ter-hash (bcrypt) |
| `remember_token` | VARCHAR(100) NULL | Token "ingat saya" |
| `created_at` | TIMESTAMP | Waktu pembuatan akun |
| `updated_at` | TIMESTAMP | Waktu terakhir diperbarui |

---

## 3. Tabel `admins`

Menyimpan data akun administrator sistem.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `name` | VARCHAR(255) | Nama admin |
| `email` | VARCHAR(255) UNIQUE | Email admin |
| `email_verified_at` | TIMESTAMP NULL | Waktu verifikasi email |
| `password` | VARCHAR(255) | Password ter-hash |
| `remember_token` | VARCHAR(100) NULL | Token "ingat saya" |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 4. Tabel `bengkels`

Menyimpan data bengkel yang terdaftar di platform.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `pemilik_id` | BIGINT UNSIGNED FK | Referensi ke `pemilik_bengkels.id` |
| `name` | VARCHAR(100) | Nama bengkel |
| `image` | VARCHAR(255) | Nama file gambar (disimpan di `storage/bengkel/`) |
| `description` | TEXT | Deskripsi bengkel |
| `alamat` | LONGTEXT | Alamat lengkap bengkel |
| `latitude` | DECIMAL(10,8) NULL | Koordinat lintang (GPS) |
| `longitude` | DECIMAL(11,8) NULL | Koordinat bujur (GPS) |
| `kecamatan_id` | BIGINT UNSIGNED FK | Referensi ke `kecamatans.id` |
| `kelurahan_id` | BIGINT UNSIGNED FK | Referensi ke `kelurahans.id` |
| `created_at` | TIMESTAMP | Waktu pendaftaran |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

> **Catatan:** Kolom `latitude` dan `longitude` ditambahkan melalui migration terpisah untuk fitur pencarian bengkel terdekat (nearby search).

---

## 5. Tabel `layanans`

Menyimpan daftar layanan yang ditawarkan oleh setiap bengkel.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `name` | VARCHAR(100) | Nama layanan (contoh: Ganti Oli, Tune Up) |
| `price` | INT | Harga layanan dalam Rupiah |
| `bengkel_id` | BIGINT UNSIGNED FK | Referensi ke `bengkels.id` |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 6. Tabel `jadwals`

Menyimpan jadwal operasional bengkel per hari.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `bengkel_id` | BIGINT UNSIGNED FK | Referensi ke `bengkels.id` |
| `senin_buka` | TIME | Jam buka hari Senin |
| `senin_tutup` | TIME | Jam tutup hari Senin |
| `selasa_buka` | TIME | Jam buka hari Selasa |
| `selasa_tutup` | TIME | Jam tutup hari Selasa |
| `rabu_buka` | TIME | Jam buka hari Rabu |
| `rabu_tutup` | TIME | Jam tutup hari Rabu |
| `kamis_buka` | TIME | Jam buka hari Kamis |
| `kamis_tutup` | TIME | Jam tutup hari Kamis |
| `jumat_buka` | TIME | Jam buka hari Jumat |
| `jumat_tutup` | TIME | Jam tutup hari Jumat |
| `sabtu_buka` | TIME NULL | Jam buka hari Sabtu (opsional) |
| `sabtu_tutup` | TIME NULL | Jam tutup hari Sabtu (opsional) |
| `minggu_buka` | TIME NULL | Jam buka hari Minggu (opsional) |
| `minggu_tutup` | TIME NULL | Jam tutup hari Minggu (opsional) |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 7. Tabel `bookings`

Menyimpan data pemesanan layanan bengkel oleh pengguna.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `user_id` | BIGINT UNSIGNED FK | Referensi ke `users.id` |
| `bengkel_id` | BIGINT UNSIGNED FK | Referensi ke `bengkels.id` |
| `tanggal_booking` | DATE | Tanggal servis yang dipesan |
| `waktu_booking` | TIME | Jam servis yang dipesan |
| `brand` | VARCHAR(255) | Merk kendaraan |
| `model` | VARCHAR(255) | Model/tipe kendaraan |
| `plat` | VARCHAR(255) | Nomor plat kendaraan |
| `tahun_pembuatan` | YEAR | Tahun pembuatan kendaraan |
| `kilometer` | INT | Odometer kendaraan saat ini |
| `transmisi` | ENUM | Jenis transmisi: `manual` atau `matic` |
| `booking_status` | ENUM | Status booking (lihat tabel status) |
| `catatan_tambahan` | TEXT NULL | Catatan tambahan dari pengguna |
| `created_at` | TIMESTAMP | Waktu booking dibuat |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

**Status Booking (Enum):**

| Nilai | Keterangan |
|-------|------------|
| `PENDING` | Baru dibuat, menunggu konfirmasi owner |
| `CONFIRMED` | Dikonfirmasi oleh owner bengkel |
| `REJECTED` | Ditolak oleh owner bengkel |
| `IN_PROGRESS` | Kendaraan sedang dalam perbaikan |
| `DONE` | Layanan selesai |
| `CANCELLED` | Dibatalkan oleh pengguna |

---

## 8. Tabel `detail_layanan_bookings`

Menyimpan detail layanan apa saja yang dipilih dalam satu booking.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `booking_id` | BIGINT UNSIGNED FK | Referensi ke `bookings.id` |
| `layanan_id` | BIGINT UNSIGNED FK | Referensi ke `layanans.id` |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

> Satu booking dapat memiliki banyak layanan (relasi one-to-many).

---

## 9. Tabel `products`

Menyimpan produk/suku cadang yang dijual bengkel.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `bengkel_id` | BIGINT UNSIGNED FK | Referensi ke `bengkels.id` |
| `category_id` | BIGINT UNSIGNED FK NULL | Referensi ke `categories.id` |
| `name` | VARCHAR(255) | Nama produk |
| `description` | TEXT | Deskripsi produk |
| `price` | INT | Harga produk |
| `stock` | INT | Stok tersedia |
| `image` | VARCHAR(255) | Nama file gambar produk |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 10. Tabel `categories`

Menyimpan kategori produk otomotif.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `name` | VARCHAR(255) | Nama kategori |
| `slug` | VARCHAR(255) UNIQUE | URL-friendly identifier |
| `icon` | VARCHAR(255) NULL | Icon class CSS |
| `description` | TEXT NULL | Deskripsi kategori |
| `is_active` | BOOLEAN | Status aktif (1=aktif, 0=nonaktif) |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 11. Tabel `carts`

Menyimpan item yang ditambahkan ke keranjang belanja pengguna.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `user_id` | BIGINT UNSIGNED FK | Referensi ke `users.id` |
| `product_id` | BIGINT UNSIGNED FK | Referensi ke `products.id` |
| `quantity` | INT | Jumlah item |
| `created_at` | TIMESTAMP | Waktu ditambahkan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 12. Tabel `transactions`

Menyimpan data transaksi pembelian.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `transaction_code` | VARCHAR(255) | Kode unik transaksi |
| `user_id` | BIGINT UNSIGNED FK | Referensi ke `users.id` |
| `bengkel_id` | BIGINT UNSIGNED FK | Referensi ke `bengkels.id` |
| `booking_id` | BIGINT UNSIGNED FK NULL | Referensi ke `bookings.id` (jika dari booking) |
| `payment_status` | VARCHAR(255) | Status pembayaran dari Midtrans |
| `shipping_status` | VARCHAR(255) | Status pengiriman produk |
| `ongkir` | INT | Biaya ongkos kirim |
| `administrasi` | INT | Biaya administrasi platform |
| `grand_total` | INT | Total pembayaran keseluruhan |
| `withdrawn_at` | TIMESTAMP NULL | Waktu dana dicairkan ke bengkel |
| `created_at` | TIMESTAMP | Waktu transaksi dibuat |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 13. Tabel `detail_transactions`

Menyimpan detail item dalam satu transaksi.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `transaction_id` | BIGINT UNSIGNED FK | Referensi ke `transactions.id` |
| `product_id` | BIGINT UNSIGNED FK NULL | Referensi ke `products.id` |
| `layanan_id` | BIGINT UNSIGNED FK NULL | Referensi ke `layanans.id` |
| `quantity` | INT | Jumlah item |
| `price` | INT | Harga per item saat transaksi |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 14. Tabel `ratings`

Menyimpan rating dan ulasan dari pengguna.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `user_id` | BIGINT UNSIGNED FK | Referensi ke `users.id` |
| `product_id` | BIGINT UNSIGNED NULL FK | Referensi ke `products.id` (untuk rating produk) |
| `layanan_id` | BIGINT UNSIGNED NULL | Referensi ke `layanans.id` (untuk rating layanan) |
| `transaction_id` | BIGINT UNSIGNED FK | Referensi ke `transactions.id` |
| `detail_transaction_id` | BIGINT UNSIGNED | Referensi ke `detail_transactions.id` |
| `stars` | TINYINT | Jumlah bintang (1-5) |
| `comment` | TEXT NULL | Komentar/ulasan |
| `created_at` | TIMESTAMP | Waktu dibuat |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

> **Constraint UNIQUE:** Kombinasi `(user_id, detail_transaction_id)` harus unik — satu pengguna hanya bisa memberi satu rating per item transaksi.

---

## 15. Tabel `withdraw_requests`

Menyimpan pengajuan penarikan dana dari bengkel.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `bengkel_id` | BIGINT UNSIGNED FK | Referensi ke `bengkels.id` |
| `amount` | DECIMAL(10,2) | Jumlah dana yang ditarik |
| `status` | ENUM | `pending` atau `transferred` |
| `image` | VARCHAR(255) NULL | Bukti transfer (upload admin) |
| `name` | VARCHAR(255) NULL | Nama penerima dana |
| `bank` | VARCHAR(255) NULL | Nama bank penerima |
| `number` | VARCHAR(255) NULL | Nomor rekening penerima |
| `created_at` | TIMESTAMP | Waktu pengajuan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 16. Tabel `specialists`

Master data spesialisasi bengkel.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `name` | VARCHAR(255) | Nama spesialisasi (contoh: AC, Kaki-kaki, Mesin) |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 17. Tabel `bengkel_specialist` (Pivot)

Relasi many-to-many antara bengkel dan spesialisasi.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `bengkel_id` | BIGINT UNSIGNED FK | Referensi ke `bengkels.id` |
| `specialist_id` | BIGINT UNSIGNED FK | Referensi ke `specialists.id` |

---

## 18. Tabel `merk_mobils`

Master data merk kendaraan yang didukung.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `nama_merk` | VARCHAR(100) | Nama merk (Toyota, Honda, dll) |
| `logo` | VARCHAR(255) NULL | File logo merk |
| `deskripsi` | TEXT NULL | Deskripsi singkat |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 19. Tabel `bengkel_merk_mobil` (Pivot)

Relasi many-to-many: bengkel bisa menangani beberapa merk kendaraan.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `bengkel_id` | BIGINT UNSIGNED FK | Referensi ke `bengkels.id` |
| `merk_mobil_id` | BIGINT UNSIGNED FK | Referensi ke `merk_mobils.id` |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 20. Tabel `kecamatans`

Data wilayah kecamatan.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `name` | VARCHAR(255) | Nama kecamatan |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 21. Tabel `kelurahans`

Data wilayah kelurahan.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `kecamatan_id` | BIGINT UNSIGNED FK | Referensi ke `kecamatans.id` |
| `name` | VARCHAR(255) | Nama kelurahan |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 22. Tabel `personal_access_tokens`

Dikelola otomatis oleh Laravel Sanctum untuk menyimpan token API.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `tokenable_type` | VARCHAR(255) | Nama class model (User atau PemilikBengkel) |
| `tokenable_id` | BIGINT UNSIGNED | ID record yang memiliki token |
| `name` | VARCHAR(255) | Nama token (contoh: `auth_token`) |
| `token` | VARCHAR(64) UNIQUE | Hash dari token aktual |
| `abilities` | TEXT NULL | Hak akses token (JSON) |
| `last_used_at` | TIMESTAMP NULL | Terakhir digunakan |
| `created_at` | TIMESTAMP | Waktu dibuat |
| `updated_at` | TIMESTAMP | Waktu diperbarui |

---

## 23. Tabel `password_reset_otps`

Menyimpan OTP untuk fitur lupa password.

| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `id` | BIGINT UNSIGNED (PK) | Primary key |
| `email` | VARCHAR(255) | Email yang meminta reset |
| `otp` | VARCHAR(6) | Kode OTP 6 digit |
| `user_type` | VARCHAR(255) | `user` atau `owner` |
| `is_verified` | BOOLEAN | Status: sudah diverifikasi atau belum |
| `expires_at` | TIMESTAMP | Waktu kedaluwarsa OTP |
| `created_at` | TIMESTAMP | Waktu pembuatan |
| `updated_at` | TIMESTAMP | Waktu diperbarui |
