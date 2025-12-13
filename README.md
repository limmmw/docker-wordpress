# docker-wordpress

Template Docker Compose sederhana untuk menjalankan WordPress menggunakan image resmi dan konfigurasi eksternal.

Direktori & file penting

- `docker-compose.yml` - definisi layanan Docker (WordPress). Menggunakan image `wordpress:latest` dan mengikat port host `8081` ke kontainer `80`.
- `.env` / `.env.example` - variabel lingkungan untuk koneksi database eksternal dan konfigurasi WordPress tambahan.
- `php-custom.ini` - konfigurasi PHP khusus (memory limit, upload size, opcache) yang dipasang ke kontainer.
- `conf/` - contoh konfigurasi Nginx reverse-proxy untuk meneruskan traffic ke WordPress yang berjalan pada `localhost:8081`.
- `wordpress/` - volume berkas untuk konten WordPress pada host (direktori ini akan dibuat/diisi saat kontainer jalan jika kosong).

Ringkasan fitur

- Mendukung penggunaan MySQL eksternal (tidak menjalankan servis database di-compose).
- Menyediakan `php-custom.ini` untuk kebutuhan plugin berat (Elementor, dsb.).
- Contoh konfigurasi Nginx siap dipakai sebagai reverse-proxy dengan timeout dan buffer lebih besar untuk proses berat.

Variabel lingkungan (di `.env`)

Contoh `.env`:

DB_HOST=host.docker.internal:3306
DB_USER=wordpress_user
DB_PASSWORD=password_anda
DB_NAME=wordpress_db

WORDPRESS_CONFIG_EXTRA=define('WP_HOME', 'https://example.com'); define('WP_SITEURL', 'https://example.com'); if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') { $_SERVER['HTTPS'] = 'on'; }

Penjelasan singkat:
- `DB_HOST` - host:port MySQL eksternal. `host.docker.internal` mengarah ke host dari dalam container (berguna di macOS/Windows; di Linux Anda mungkin perlu mengganti ke IP host atau nama host DB).
- `DB_USER`, `DB_PASSWORD`, `DB_NAME` - kredensial database MySQL.
- `WORDPRESS_CONFIG_EXTRA` - potongan PHP untuk dimasukkan ke `wp-config.php` lewat image WordPress. Digunakan untuk mengatur URL situs dan protokol ketika menggunakan reverse-proxy.

Cara menggunakan (lokal)

1. Copy `.env.example` ke `.env` dan sesuaikan nilai-nilai database dan URL.

2. (Opsional) Buat direktori volume untuk WordPress jika belum ada:

```bash
mkdir -p wordpress
```

3. Jalankan Docker Compose:

```bash
docker compose up -d
```

4. Akses WordPress di browser melalui http://localhost:8081 (atau melalui reverse-proxy Anda jika telah terpasang pada domain).

Menggunakan dengan Nginx reverse-proxy

- Contoh konfigurasi Nginx ada di `conf/`. File tersebut meneruskan traffic ke `http://localhost:8081` dan menambahkan header proxy untuk mengaktifkan HTTPS/forwarding.
- Pastikan `server_name` di `conf/` diganti sesuai domain Anda.

Tips produksi & keamanan

- Jangan menyimpan file `.env` dalam kontrol versi (sudah ditambahkan ke `.gitignore`).
- Gunakan MySQL yang dikelola atau database terpisah di jaringan privat untuk produksi.
- Gunakan HTTPS (Let's Encrypt) pada reverse-proxy dan atur header `X-Forwarded-Proto` dengan benar.
- Untuk performa produksi, pertimbangkan menambahkan caching (Varnish, Redis, plugin caching WP) dan penyimpanan file objek (S3) untuk media static.

Troubleshooting umum

- Jika WordPress tidak bisa terhubung ke DB: periksa `DB_HOST` dan kredensial. Pastikan DB menerima koneksi dari IP host Docker.
- Jika file upload error / limit: periksa `php-custom.ini` (upload_max_filesize, post_max_size) dan `client_max_body_size` di Nginx.
- Jika assets 404: pastikan volume `./wordpress` memiliki izin yang benar dan owner sesuai (opsional: chown ke UID/GID yang sama dengan container PHP-FPM).

Tambahan kecil

- `php-custom.ini` sudah di-mount read-only ke dalam container sehingga konfigurasi PHP akan aktif tanpa mengubah image.

Kontribusi

Silakan buka issue atau pull request untuk perbaikan, dokumentasi tambahan, atau integrasi fitur (cache, backup, dsb.).

Lisensi

Proyek ini tidak menyertakan lisensi spesifik. Tambahkan file `LICENSE` jika ingin menentukan lisensi.
