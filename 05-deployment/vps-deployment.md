# Deployment ke VPS (Nginx + PHP-FPM + PM2 + Certbot)

## 1. Spesifikasi Server

### Rekomendasi VPS

| Penyedia | Produk | Estimasi Harga |
|----------|--------|----------------|
| DigitalOcean | Droplet Basic | $6–12/bulan |
| Niagahoster | VPS Linux | Rp 50.000–150.000/bulan |
| Dewaweb | VPS | Rp 50.000–200.000/bulan |
| RackH | VPS | Rp 45.000/bulan |

### Spesifikasi Minimum

| Komponen | Minimum | Direkomendasikan |
|----------|---------|------------------|
| CPU | 1 vCPU | 2 vCPU |
| RAM | 1 GB | 2 GB |
| Storage | 20 GB SSD | 40 GB SSD |
| OS | Ubuntu 20.04 LTS | Ubuntu 22.04 LTS |

---

## 2. Persiapan Awal Server

### Langkah 1: Login ke Server via SSH

```bash
ssh root@[IP_SERVER]
# Contoh: ssh root@165.22.100.50
```

### Langkah 2: Update Sistem

```bash
apt update && apt upgrade -y
```

### Langkah 3: Buat Pengguna Non-Root

Menjalankan aplikasi sebagai `root` berbahaya. Buat user baru:

```bash
adduser bengkelin
usermod -aG sudo bengkelin

# Pindah ke user baru
su - bengkelin
```

### Langkah 4: Konfigurasi Firewall

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'   # Izinkan port 80 dan 443
sudo ufw enable

sudo ufw status
```

---

## 3. Instalasi PHP 8.x dan PHP-FPM

PHP-FPM (FastCGI Process Manager) adalah implementasi PHP yang berjalan sebagai daemon terpisah, jauh lebih efisien daripada `php artisan serve` untuk production.

```bash
# Tambahkan repository PHP (Ondrej PPA)
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# Install PHP 8.2 dan ekstensi yang dibutuhkan Laravel
sudo apt install -y \
    php8.2 \
    php8.2-fpm \
    php8.2-mysql \
    php8.2-mbstring \
    php8.2-xml \
    php8.2-bcmath \
    php8.2-curl \
    php8.2-zip \
    php8.2-gd \
    php8.2-intl \
    php8.2-tokenizer \
    php8.2-fileinfo \
    unzip \
    git \
    curl
```

### Verifikasi Instalasi

```bash
php -v
# PHP 8.2.x (cli)

php-fpm8.2 -v
# PHP 8.2.x (fpm-fcgi)
```

### Aktifkan PHP-FPM

```bash
sudo systemctl start php8.2-fpm
sudo systemctl enable php8.2-fpm   # Otomatis start saat server reboot

sudo systemctl status php8.2-fpm
```

---

## 4. Instalasi Composer

Composer adalah package manager PHP, digunakan untuk menginstal dependensi Laravel.

```bash
cd ~
curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer

composer --version
# Composer version 2.x.x
```

---

## 5. Instalasi MySQL

```bash
sudo apt install -y mysql-server

# Jalankan wizard keamanan MySQL
sudo mysql_secure_installation
# Jawab: Y untuk semua pertanyaan keamanan
```

### Buat Database dan User untuk Bengkelin

```bash
sudo mysql -u root -p
```

```sql
-- Buat database
CREATE DATABASE bengkelin_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Buat user khusus (jangan gunakan root)
CREATE USER 'bengkelin_user'@'localhost' IDENTIFIED BY 'PASSWORD_KUAT_DI_SINI';

-- Beri hak akses ke database
GRANT ALL PRIVILEGES ON bengkelin_db.* TO 'bengkelin_user'@'localhost';

FLUSH PRIVILEGES;
EXIT;
```

---

## 6. Instalasi Nginx

```bash
sudo apt install -y nginx

sudo systemctl start nginx
sudo systemctl enable nginx

# Verifikasi
sudo systemctl status nginx
```

---

## 7. Deploy Kode Aplikasi Laravel

### Langkah 1: Clone Repository

```bash
cd /var/www
sudo git clone https://[url-repository]/bengkelin.git
sudo chown -R $USER:$USER /var/www/bengkelin
cd /var/www/bengkelin
```

> Jika repository private, gunakan SSH deploy key atau personal access token.

### Langkah 2: Install Dependensi PHP

```bash
composer install --optimize-autoloader --no-dev
```

Flag `--no-dev` mengecualikan package development (testing, dll) yang tidak diperlukan di production.

### Langkah 3: Konfigurasi File `.env`

```bash
cp .env.example .env
nano .env
```

Isi nilai untuk environment production:

```env
# ── Aplikasi ──────────────────────────────────────────────
APP_NAME=Bengkelin
APP_ENV=production
APP_KEY=                        # Akan diisi oleh key:generate
APP_DEBUG=false                 # WAJIB false di production
APP_URL=https://api.bengkelin.com

LOG_CHANNEL=stack
LOG_LEVEL=error

# ── Database ──────────────────────────────────────────────
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=bengkelin_db
DB_USERNAME=bengkelin_user
DB_PASSWORD=PASSWORD_KUAT_DI_SINI

# ── Email (untuk OTP reset password) ─────────────────────
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=noreply@bengkelin.com
MAIL_PASSWORD=GOOGLE_APP_PASSWORD
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=noreply@bengkelin.com
MAIL_FROM_NAME="Bengkelin"

# ── Midtrans (gunakan Production keys) ───────────────────
MIDTRANS_SERVER_KEY=Mid-server-PRODUCTION_KEY_ANDA
MIDTRANS_CLIENT_KEY=Mid-client-PRODUCTION_KEY_ANDA
MIDTRANS_IS_PRODUCTION=true
MIDTRANS_IS_SANITIZED=true
MIDTRANS_IS_3DS=true

# ── Storage ───────────────────────────────────────────────
FILESYSTEM_DISK=public
```

### Langkah 4: Generate Application Key

```bash
php artisan key:generate
```

Perintah ini mengisi `APP_KEY` di `.env` secara otomatis.

### Langkah 5: Jalankan Migrasi dan Seeder

```bash
php artisan migrate --force
php artisan db:seed --force
```

Flag `--force` diperlukan karena Laravel memblokir perintah destructive di environment production secara default.

### Langkah 6: Buat Symbolic Link Storage

```bash
php artisan storage:link
```

Ini membuat link `public/storage` → `storage/app/public` agar file upload (foto bengkel, foto produk) bisa diakses via URL.

### Langkah 7: Cache untuk Performa

```bash
php artisan config:cache   # Cache semua file konfigurasi
php artisan route:cache    # Cache tabel routing
php artisan view:cache     # Cache template Blade
php artisan optimize       # Jalankan semua optimasi sekaligus
```

### Langkah 8: Set Permission File

```bash
# PHP-FPM berjalan sebagai www-data, direktori ini harus bisa ditulis
sudo chown -R www-data:www-data /var/www/bengkelin/storage
sudo chown -R www-data:www-data /var/www/bengkelin/bootstrap/cache
sudo chmod -R 775 /var/www/bengkelin/storage
sudo chmod -R 775 /var/www/bengkelin/bootstrap/cache
```

---

## 8. Konfigurasi Nginx

### Buat File Konfigurasi Virtual Host

```bash
sudo nano /etc/nginx/sites-available/bengkelin
```

Isi dengan konfigurasi berikut:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name api.bengkelin.com;

    # Root mengarah ke folder public Laravel
    root /var/www/bengkelin/public;
    index index.php index.html;

    # Penanganan request utama — semua request masuk ke index.php
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Proses file PHP menggunakan PHP-FPM via socket Unix
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    }

    # Tolak akses ke file tersembunyi (.env, .git, dll)
    location ~ /\.(?!well-known).* {
        deny all;
    }

    # Atur header cache untuk file statis
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # Log
    access_log /var/log/nginx/bengkelin_access.log;
    error_log  /var/log/nginx/bengkelin_error.log;
}
```

### Aktifkan Konfigurasi

```bash
# Buat symlink dari sites-available ke sites-enabled
sudo ln -s /etc/nginx/sites-available/bengkelin /etc/nginx/sites-enabled/

# Hapus konfigurasi default Nginx (opsional)
sudo rm /etc/nginx/sites-enabled/default

# Test konfigurasi — pastikan tidak ada error
sudo nginx -t
# Output: nginx: configuration file /etc/nginx/nginx.conf test is successful

# Reload Nginx
sudo systemctl reload nginx
```

---

## 9. SSL/HTTPS dengan Certbot (Let's Encrypt)

SSL certificate gratis dari Let's Encrypt diperlukan agar API bisa diakses via HTTPS — wajib untuk keamanan data (token, password, dll).

### Langkah 1: Install Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
```

### Langkah 2: Dapatkan Sertifikat SSL

```bash
sudo certbot --nginx -d api.bengkelin.com
```

Certbot akan:
1. Memverifikasi kepemilikan domain
2. Mendownload sertifikat SSL
3. **Otomatis memodifikasi konfigurasi Nginx** untuk mendukung HTTPS
4. Mengatur redirect HTTP → HTTPS

Saat diminta, pilih opsi:
- Email: masukkan email aktif untuk notifikasi
- Agree to Terms: `A`
- Redirect: `2` (redirect semua HTTP ke HTTPS)

### Hasil Konfigurasi Nginx Setelah Certbot

Certbot secara otomatis menambahkan blok SSL ke konfigurasi Nginx:

```nginx
server {
    listen 443 ssl;
    server_name api.bengkelin.com;

    ssl_certificate     /etc/letsencrypt/live/api.bengkelin.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.bengkelin.com/privkey.pem;
    include             /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam         /etc/letsencrypt/ssl-dhparams.pem;

    root /var/www/bengkelin/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}

# Redirect HTTP ke HTTPS
server {
    listen 80;
    server_name api.bengkelin.com;
    return 301 https://$server_name$request_uri;
}
```

### Langkah 3: Verifikasi Auto-Renewal

Sertifikat Let's Encrypt berlaku 90 hari. Certbot mengatur pembaruan otomatis via cron:

```bash
# Test proses renewal
sudo certbot renew --dry-run

# Cek timer systemd yang mengurus renewal
sudo systemctl status certbot.timer
```

---

## 10. PM2 — Daemon Manager untuk Proses Background

**PM2** (Process Manager 2) adalah tool yang menjaga proses background tetap berjalan terus-menerus, bahkan setelah server restart. Dalam konteks Laravel, PM2 digunakan untuk menjalankan:

- **`queue:work`** — memproses antrian pekerjaan (email OTP, notifikasi)
- **`schedule:run`** — menjalankan task terjadwal Laravel

### Langkah 1: Install Node.js dan PM2

```bash
# Install Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Install PM2 secara global
sudo npm install -g pm2

pm2 --version
```

### Langkah 2: Jalankan Queue Worker

Laravel Queue Worker memproses job yang masuk ke antrian — contohnya pengiriman email OTP reset password.

```bash
pm2 start "php /var/www/bengkelin/artisan queue:work --sleep=3 --tries=3 --max-time=3600" \
    --name="bengkelin-queue" \
    --interpreter=none
```

**Penjelasan flag:**

| Flag | Keterangan |
|------|------------|
| `--sleep=3` | Tunggu 3 detik jika tidak ada job baru |
| `--tries=3` | Coba ulang job yang gagal maksimal 3 kali |
| `--max-time=3600` | Worker restart setelah 1 jam (cegah memory leak) |

### Langkah 3: Jalankan Laravel Scheduler

Laravel Scheduler menjalankan task terjadwal yang didefinisikan di `app/Console/Kernel.php`. PM2 menjalankannya setiap menit:

```bash
pm2 start "php /var/www/bengkelin/artisan schedule:run" \
    --name="bengkelin-scheduler" \
    --cron="* * * * *" \
    --interpreter=none
```

Flag `--cron="* * * * *"` membuat PM2 menjalankan perintah setiap menit, persis seperti cron job.

### Langkah 4: Simpan Konfigurasi PM2

```bash
pm2 save
```

Perintah ini menyimpan daftar proses yang sedang berjalan ke disk.

### Langkah 5: Aktifkan Startup Otomatis

Agar PM2 dan semua proses di dalamnya otomatis berjalan setelah server reboot:

```bash
pm2 startup

# Perintah di atas akan menampilkan satu baris perintah sudo yang harus dijalankan.
# Contoh output:
# sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u bengkelin --hp /home/bengkelin
# Salin dan jalankan perintah tersebut.
```

---

## 11. Verifikasi Deployment

### Cek Status Layanan

```bash
# Status Nginx
sudo systemctl status nginx

# Status PHP-FPM
sudo systemctl status php8.2-fpm

# Status MySQL
sudo systemctl status mysql

# Status proses PM2
pm2 status
pm2 logs bengkelin-queue
```

### Test API

```bash
# Test endpoint publik
curl https://api.bengkelin.com/api/home

# Test health check
curl -I https://api.bengkelin.com
# Harus mengembalikan: HTTP/2 200
```

### Cek Log Error

```bash
# Log Nginx
sudo tail -f /var/log/nginx/bengkelin_error.log

# Log Laravel
tail -f /var/www/bengkelin/storage/logs/laravel.log
```

---

## 12. Konfigurasi Midtrans untuk Production

Setelah server live, daftarkan URL callback di dashboard Midtrans:

1. Login ke **dashboard.midtrans.com**
2. Pilih **Settings → Configuration**
3. Isi **Payment Notification URL:**
   ```
   https://api.bengkelin.com/api/midtrans-callback
   ```
4. Aktifkan **Production Mode** dan ganti ke production keys di `.env`

---

## 13. Prosedur Update Aplikasi

Setiap kali ada perubahan kode yang perlu di-deploy:

```bash
cd /var/www/bengkelin

# 1. Pull perubahan terbaru
git pull origin main

# 2. Install dependensi baru (jika ada)
composer install --optimize-autoloader --no-dev

# 3. Jalankan migrasi baru (jika ada)
php artisan migrate --force

# 4. Perbarui cache
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 5. Restart queue worker agar pakai kode terbaru
pm2 restart bengkelin-queue

# 6. Reload PHP-FPM (opsional, untuk clear OPcache)
sudo systemctl reload php8.2-fpm
```

---

## 14. Checklist Deployment

### Sebelum Deploy

- [ ] Domain sudah diarahkan ke IP VPS (A record di DNS)
- [ ] Semua kode sudah di-push ke repository
- [ ] Migration terbaru sudah ada di `database/migrations/`

### Saat Deploy Pertama Kali

- [ ] PHP 8.x, Composer, MySQL, Nginx, Node.js, PM2 sudah terinstal
- [ ] Database dan user MySQL sudah dibuat
- [ ] Kode sudah di-clone ke `/var/www/bengkelin`
- [ ] `composer install --no-dev` berhasil
- [ ] `.env` sudah dikonfigurasi (APP_KEY, DB, MAIL, MIDTRANS)
- [ ] `php artisan key:generate` sudah dijalankan
- [ ] `php artisan migrate --force` berhasil
- [ ] `php artisan storage:link` sudah dijalankan
- [ ] Permission `storage/` dan `bootstrap/cache/` sudah diset ke `www-data`
- [ ] Konfigurasi Nginx sudah aktif dan `nginx -t` sukses
- [ ] SSL sudah terpasang via Certbot
- [ ] PM2 queue worker berjalan
- [ ] PM2 startup diaktifkan (`pm2 startup && pm2 save`)

### Verifikasi Akhir

- [ ] `https://api.bengkelin.com/api/home` mengembalikan response JSON
- [ ] Register/login user berhasil
- [ ] Upload foto bengkel/produk tersimpan dan bisa diakses
- [ ] Email OTP terkirim saat test reset password
- [ ] `pm2 status` menunjukkan semua proses `online`
- [ ] `sudo certbot renew --dry-run` tidak ada error
