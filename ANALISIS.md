## Nama : Fachriel yoga wicaksono
## NIm : H1H024042

# Analisis Perbaikan

## Permasalahan 1

### Gejala
Saat menjalankan docker compose up -d langsung gagal, tidak ada container yang berhasil dibuat. Selain itu setelah diperbaiki sebagian, beberapa container tetap tidak bisa terhubung satu sama lain.

### Penyebab
- Baris services tidak diikuti tanda titik dua (:), menyebabkan file YAML tidak valid secara sintaks
- web1 memiliki DB_HOST: mysql, padahal nama service database adalah db
- web2 memiliki DB_PASS: wrongpassword, tidak sesuai password yang dikonfigurasi di service db (student123)
- Build context web3 mengarah ke ./web33 (typo), folder tersebut tidak ada
- Service web3 hanya terhubung ke network backend, tidak ke frontend, sehingga nginx tidak bisa menjangkau web3
- Named volume di bagian volumes: bernama database-data, tetapi service db mereferensikan db-data sehingga terjadi mismatch

### Solusi
- Menambahkan : pada baris services menjadi services:
- Mengubah DB_HOST: mysql menjadi DB_HOST: db pada service web1
- Mengubah DB_PASS: wrongpassword menjadi DB_PASS: student123 pada service web2
- Mengubah context: ./web33 menjadi context: ./web3 pada service web3
- Menambahkan frontend ke daftar networks pada service web3
- Menyamakan nama volume dengan mengubah db-data:/var/lib/mysql


## Permasalahan 2

### Gejala
Build web1 dan web3 gagal dengan error not found saat Docker mencoba pull image dari Docker Hub.

### Penyebab
- web1/Dockerfile menggunakan FROM php:8.2-apach (kurang huruf e di akhir)
- web3/Dockerfile menggunakan FROM php:8.2-apche (huruf a dan c tertukar)
- Image dengan nama tersebut tidak ditemukan di Docker Hub

### Solusi
- Mengubah FROM php:8.2-apach menjadi FROM php:8.2-apache pada web1/Dockerfile
> sed -i 's/php:8.2-apach$/php:8.2-apache/' web1/Dockerfile
- Mengubah FROM php:8.2-apche menjadi FROM php:8.2-apache pada web3/Dockerfile
> sed -i 's/php:8.2-apche/php:8.2-apache/' web3/Dockerfile



## Permasalahan 3

### Gejala
Container nginx langsung crash setelah start. Ditemukan dua error berbeda:
Setelah nginx berhasil jalan, web3 tidak pernah mendapat giliran dari load balancer.

### Penyebab
- File nginx.conf mengandung markdown blok yang ikut tersimpan di dalam file, nginx tidak bisa mem-parse sintaks tersebut
- Nama upstream tertulis web1 (kelebihan angka 1), tidak ada container dengan hostname tersebut
- Port upstream web3 ditulis web3:8080, padahal container web3 listen di port 80

### Solusi
- Menghapus semua baris markdown blok dengan sed -i '/^```/d' nginx/nginx.conf
- Mengubah server web11:80 menjadi server web1:80
> sed -i 's/web11/web1/g' nginx/nginx.con
- Mengubah server web3:8080 menjadi server web3:80
> sed -i 's/web3:8080/web3:80/' nginx/nginx.conf
- Karena perubahan config ada di dalam image, perlu dilakukan docker compose build --no-cache nginx agar image di-rebuild ulang


## Permasalahan 4

### Gejala
Container mysql-db crash saat inisialisasi database dengan error:

### Penyebab
File db/init.sql mengandung markdown blok di awal dan akhir file. MySQL tidak dapat mengeksekusi sintaks markdown tersebut sebagai perintah SQL yang valid.

### Solusi
Menghapus semua baris yang mengandung blok dengan sed -i '/^```/d' db/init.sql

## Permasalahan 5

### Gejala
Halaman web menampilkan teks "ganti ke namamu" & "ganti ke nimmu". (web*/index.php)

### Penyebab
File dibuat sebagai template dengan placeholder yang belum diganti.

### Solusi
Ganti $nama dan $nim dengan nilai yang benar