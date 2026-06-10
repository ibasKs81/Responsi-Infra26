# Analisis Perbaikan

## Permasalahan 1

### Gejala
yaml:Mapping values not allowe 

### Penyebab
Penyebabnya adalah baris 3 kolom 5 pada yaml typo/ ada masalah penulisan/indentasi
### Solusi
Cek bagian tersebut dan ternyata kurang ":" jadi tambahkan agar bisa terbaca sistem

---

## Permasalahan 2: Volume Database Tidak Dikenali

### Gejala
Muncul pesan error: `service "db" refers to undefined volume db-data` saat menjalankan `docker compose up`.

### Penyebab
Terdapat ketidakcocokan penamaan volume. Pada bagian `services` -> `db`, volume dipanggil dengan nama `db-data`. Namun, pada bagian `volumes:` di paling bawah file YAML, volume dideklarasikan dengan nama `database-data`.

### Solusi
Mengubah nama deklarasi volume di bagian paling bawah `docker-compose.yml` agar sama persis dengan yang dipanggil oleh service `db`, yaitu menjadi `db-data:`.

---

## Permasalahan 3: Koneksi Web ke Database Gagal & Kesalahan Direktori Build

### Gejala
Web server tidak dapat terhubung ke database. Selain itu, terdapat pesan error saat build image web3 karena direktori tidak ditemukan.

### Penyebab
1. Pada `docker-compose.yml` service `web1`, `web2`, dan `web3`, variabel `DB_HOST` menunjuk ke `mysql`, padahal nama service database yang benar adalah `db`.
2. Variabel `DB_PASS` pada `web2` menggunakan password yang salah (`wrongpassword`).
3. Direktori *build context* untuk service `web3` salah ketik menjadi `./web33`.

### Solusi
1. Mengubah nilai `DB_HOST: mysql` menjadi `DB_HOST: db` pada ketiga service web.
2. Menyamakan password database pada `web2` menjadi `DB_PASS: student123`.
3. Memperbaiki *build context* web3 menjadi `context: ./web3`.

---

## Permasalahan 4: Nginx Upstream Error (Bad Gateway)

### Gejala
Nginx tidak bisa melakukan *load balancing* dengan benar. Jika diakses, akan muncul error *host not found* atau web tertentu tidak bisa dimuat.

### Penyebab
Terdapat *typo* pada blok `upstream backend` di dalam file `nginx/nginx.conf`:
- Alamat server web1 salah ketik menjadi `server web11:80;`.
- Alamat server web3 mengarah ke port yang salah, yaitu `server web3:8080;` padahal container berjalan di port 80.

### Solusi
Memperbaiki konfigurasi `nginx.conf` menjadi `server web1:80;` dan `server web3:80;`.

---

## Permasalahan 5: Kegagalan Build Image karena Typo Base Image

### Gejala
Muncul error saat proses *build* container: `failed to solve: php:8.2-apche: failed to resolve source metadata... not found`.

### Penyebab
Terdapat *typo* pada penulisan *base image* di baris pertama `Dockerfile` milik web server. Di `web1` tertulis `php:8.2-apach` dan di `web3` tertulis `php:8.2-apche`. Image tersebut tidak eksis di Docker Hub. Selain itu, port yang diekspos (`EXPOSE 8080`) tidak sesuai dengan *default port* Apache.

### Solusi
Mengubah baris pertama pada `Dockerfile` di web1, web2, dan web3 menjadi `FROM php:8.2-apache`. Disarankan juga mengubah `EXPOSE 8080` menjadi `EXPOSE 80` agar sinkron dengan port internal *container*.

---

## Permasalahan 6: Container Nginx dan Init SQL Crash (Exited)

### Gejala
Saat dijalankan `docker ps`, container `nginx-lb` tidak muncul (mati sesaat setelah dihidupkan). Pada log memunculkan pesan: `nginx: [emerg] unknown directive "
http://googleusercontent.com/immersive_entry_chip/0
