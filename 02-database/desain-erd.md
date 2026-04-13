# Desain ERD dan Relasi Antar Tabel

## 1. Entity Relationship Diagram (ERD)

```
┌───────────────┐       ┌─────────────────┐       ┌──────────────┐
│   kecamatans  │       │    kelurahans   │       │    admins    │
│               │       │                 │       │              │
│ PK id         │◄──┐   │ PK id           │       │ PK id        │
│    name       │   └───│ FK kecamatan_id │       │    name      │
└───────────────┘       │    name         │       │    email     │
       ▲                └────────┬────────┘       └──────────────┘
       │                         │
       │  ┌──────────────────────┘
       │  │
       │  ▼
┌──────┴──────────────────────────────────────────────────────────┐
│                          users                                  │
│ PK id                                                           │
│    name, email, phone_number, alamat, password                  │
│ FK kecamatan_id → kecamatans.id                                 │
│ FK kelurahan_id → kelurahans.id                                 │
└────────────────────────┬────────────────────────────────────────┘
                         │ 1
                         │
              ┌──────────┼──────────┐
              │ *        │ *        │ *
              ▼          ▼          ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ bookings │ │   carts  │ │  ratings │
        │          │ │          │ │          │
        │ PK id    │ │ PK id    │ │ PK id    │
        │ FK user  │ │ FK user  │ │ FK user  │
        │ FK beng. │ │ FK prod. │ │ FK prod. │
        │ tanggal  │ │ quantity │ │ FK trans.│
        │ waktu    │ └──────────┘ │ stars    │
        │ status   │              │ comment  │
        └────┬─────┘              └──────────┘
             │ 1
             │
             ▼ *
    ┌─────────────────────┐
    │ detail_layanan_     │
    │ bookings            │
    │                     │
    │ PK id               │
    │ FK booking_id       │
    │ FK layanan_id       │
    └─────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                      pemilik_bengkels                            │
│ PK id                                                            │
│    name, email, phone_number, password                           │
└───────────────────────┬──────────────────────────────────────────┘
                        │ 1
                        │
                        ▼ *
┌───────────────────────────────────────────────────────────────────┐
│                           bengkels                                │
│ PK id                                                             │
│    name, image, description, alamat                               │
│    latitude, longitude                                            │
│ FK pemilik_id → pemilik_bengkels.id                               │
│ FK kecamatan_id → kecamatans.id                                   │
│ FK kelurahan_id → kelurahans.id                                   │
└────┬────────┬────────┬────────┬────────┬──────────────────────────┘
     │        │        │        │        │
     │1       │1       │1       │        │
     ▼*       ▼*       ▼*       │        │
┌─────────┐ ┌──────┐ ┌──────┐   │        │
│layanans │ │jadwal│ │produc│   │ M:N    │ M:N
│         │ │  s   │ │  ts  │   │        │
│ PK id   │ │      │ │      │   ▼        ▼
│ FK beng │ │FK bg │ │FK bg │ ┌──────────────────┐
│ name    │ │waktu │ │name  │ │bengkel_specialist│
│ price   │ └──────┘ │price │ └──────────────────┘
└─────────┘          │stock │ ┌──────────────────┐
                     └──────┘ │bengkel_merk_mobil│
                              └──────────────────┘

TABEL TRANSAKSI:

┌──────────────────────────────────────────────────────────────────┐
│                         transactions                             │
│ PK id                                                            │
│    transaction_code, payment_status, shipping_status            │
│    ongkir, administrasi, grand_total, withdrawn_at               │
│ FK user_id → users.id                                            │
│ FK bengkel_id → bengkels.id                                      │
│ FK booking_id → bookings.id (nullable)                           │
└───────────────────────────┬──────────────────────────────────────┘
                            │ 1
                            │
                            ▼ *
              ┌─────────────────────────────┐
              │      detail_transactions    │
              │                             │
              │ PK id                       │
              │ FK transaction_id           │
              │ FK product_id (nullable)    │
              │ FK layanan_id (nullable)    │
              │    quantity, price          │
              └─────────────────────────────┘

TABEL PENARIKAN DANA:

bengkels ──1:*──→ withdraw_requests
```

---

## 2. Penjelasan Relasi

### 2.1 User dan Bengkel (Many-to-Many melalui Booking)

Seorang pengguna dapat melakukan booking ke banyak bengkel, dan satu bengkel dapat menerima booking dari banyak pengguna. Relasinya dijembatani oleh tabel `bookings`.

```php
// Model User
public function bookings() {
    return $this->hasMany(Booking::class);
}

// Model Booking
public function user() {
    return $this->belongsTo(User::class);
}
public function bengkel() {
    return $this->belongsTo(Bengkel::class);
}
```

### 2.2 Bengkel dan Specialist (Many-to-Many)

Satu bengkel bisa memiliki banyak spesialisasi, dan satu spesialisasi bisa dimiliki banyak bengkel. Relasinya menggunakan tabel pivot `bengkel_specialist`.

```php
// Model Bengkel
public function specialists() {
    return $this->belongsToMany(Specialist::class, 'bengkel_specialist');
}
```

### 2.3 Bengkel dan Merk Mobil (Many-to-Many)

Satu bengkel bisa menangani banyak merk mobil, menggunakan tabel pivot `bengkel_merk_mobil`.

```php
// Model Bengkel
public function merkMobils() {
    return $this->belongsToMany(MerkMobil::class, 'bengkel_merk_mobil');
}
```

### 2.4 Booking dan Layanan (Many-to-Many melalui detail_layanan_bookings)

Satu booking bisa mencakup beberapa layanan sekaligus.

```php
// Model Booking
public function detail_layanan_bookings() {
    return $this->hasMany(DetailLayananBooking::class);
}
```

### 2.5 Transaksi dan Detail Transaksi (One-to-Many)

Satu transaksi bisa memuat banyak item (produk atau layanan).

```php
// Model Transaction
public function detail_transactions() {
    return $this->hasMany(DetailTransaction::class);
}
```

### 2.6 Wilayah: Kecamatan → Kelurahan (One-to-Many)

Satu kecamatan memiliki banyak kelurahan. Baik `users` maupun `bengkels` memiliki referensi ke kedua tabel ini.

```php
// Model User
public function kecamatan() {
    return $this->belongsTo(Kecamatan::class);
}
public function kelurahan() {
    return $this->belongsTo(Kelurahan::class);
}
```

---

## 3. Ringkasan Jenis Relasi

| Relasi | Entitas A | Entitas B | Jenis | Via |
|--------|-----------|-----------|-------|-----|
| User → Booking | users | bookings | 1:N | - |
| Bengkel → Booking | bengkels | bookings | 1:N | - |
| Booking → DetailLayanan | bookings | detail_layanan_bookings | 1:N | - |
| DetailLayanan → Layanan | detail_layanan_bookings | layanans | N:1 | - |
| PemilikBengkel → Bengkel | pemilik_bengkels | bengkels | 1:N | - |
| Bengkel → Layanan | bengkels | layanans | 1:N | - |
| Bengkel → Produk | bengkels | products | 1:N | - |
| Bengkel → Jadwal | bengkels | jadwals | 1:N | - |
| Bengkel ↔ Specialist | bengkels | specialists | M:N | bengkel_specialist |
| Bengkel ↔ MerkMobil | bengkels | merk_mobils | M:N | bengkel_merk_mobil |
| User → Cart | users | carts | 1:N | - |
| Produk → Cart | products | carts | 1:N | - |
| User → Transaction | users | transactions | 1:N | - |
| Transaction → Detail | transactions | detail_transactions | 1:N | - |
| Bengkel → Withdraw | bengkels | withdraw_requests | 1:N | - |
| User → Rating | users | ratings | 1:N | - |
| Kecamatan → Kelurahan | kecamatans | kelurahans | 1:N | - |

---

## 4. Indexing Database

Index database digunakan untuk mempercepat operasi pencarian. Beberapa index penting dalam Bengkelin:

| Tabel | Kolom yang diindex | Alasan |
|-------|-------------------|--------|
| `users` | `email` | Login selalu mencari berdasarkan email |
| `pemilik_bengkels` | `email` | Login owner berdasarkan email |
| `bengkels` | `pemilik_id`, `kecamatan_id` | Filter bengkel berdasarkan owner dan lokasi |
| `bookings` | `user_id`, `bengkel_id` | Pencarian booking per user dan per bengkel |
| `transactions` | `user_id`, `bengkel_id` | Riwayat transaksi |
| `ratings` | `product_id`, `transaction_id` | Query rating produk |
| `personal_access_tokens` | `tokenable_id`, `tokenable_type` | Pencarian token saat autentikasi |

---

## 5. Desain Keputusan Database

### Mengapa `pemilik_bengkels` dipisah dari `users`?

Dua guard autentikasi (`auth:sanctum` untuk user, `auth:owner-api` untuk owner) memerlukan model terpisah. Ini juga membuat validasi bisnis lebih bersih — pemilik bengkel memiliki atribut dan hak akses yang berbeda dari pengguna biasa.

### Mengapa kolom `latitude` dan `longitude` ditambahkan terpisah?

Fitur geolocation (pencarian bengkel terdekat) ditambahkan setelah desain awal. Ini umum dalam pengembangan iteratif — migration `2025_08_07_191537_add_lat_long_to_bengkels_table.php` menambahkan kedua kolom tanpa merusak data yang sudah ada.

### Mengapa `product_id` dan `layanan_id` di `ratings` bisa NULL?

Rating awalnya hanya untuk produk, lalu dikembangkan untuk mendukung rating layanan juga (via migration `2025_11_08_162742_modify_ratings_table_for_layanan.php`). Kolom `product_id` dibuat nullable agar kompatibel dengan data lama.
