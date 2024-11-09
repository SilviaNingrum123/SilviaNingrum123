# Plant Management API

### Nama dan NIM
-Nama : Retno Silvia Ningrum
-NIM  : 24.01.55.7003

### Deskripsi Proyek
Plant management API adalah API berbasis rest yang digunakan untuk mengelola suatu data tanaman.
pengguna api ini dapat melakukan CRUD ( CREATE, READ, UPDATE, DELETE) pada data tanaman

# Query SQL Pembuatan Tabel
Berikut adalah query SQL untuk membuat tabel plants dalam database plant_management
```sql
CREATE DATABASE plant_management;

USE plant_management;

CREATE TABLE plants (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    type VARCHAR(255) NOT NULL,
    price DECIMAL(10, 2) NOT NULL CHECK(price > 0),
    stock INT NOT NULL CHECK(stock >= 0)
);
```
### DAFTAR END POINT
 | method     | Endpoint | Deskripsi       |
|----------|------|------------|
|  GET  | /plants   | Memanggil semua tanaman     |
| GET  | /plants/{id}   |  Memanggil detail tanaman berdasarkan ID    |
| GET  | /plants/search?term={term}   | Memanggil tanaman berdasarkan nama/jenis   |
|  POST  | /plants   | Menambahkan tanaman baru    |
| PUT  | /plants/{id}   | Memperbarui tanaman berdasarkan ID    |
| DELETE  | /plants/{id}   | Menghapus tanaman berdasarkan ID   |

### CARA INSTALASI dan PENGGUNAAN
1. Clone Repository atau unduh file API ini.
2. Setup Database dengan MySQL, buat database plant_management, dan jalankan query pembuatan tabel di atas.
3. Konfigurasi Koneksi Database:
  - Ubah nilai $user dan $pass dalam fungsi getConnection() di file plant_api.php sesuai konfigurasi MySQL Anda.
4. Jalankan Server Lokal:
  - Letakkan file API di folder server lokal (misalnya, htdocs untuk XAMPP).
  - Akses URL: http://localhost/plant_management/plant_api.php
5. Gunakan Postman untuk menguji setiap endpoint API.
