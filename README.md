# Plant Management API

### Nama dan NIM
- Nama : Retno Silvia Ningrum
- NIM  : 24.01.55.7003

### Deskripsi Proyek
Plant management API adalah API berbasis rest yang digunakan untuk mengelola suatu data tanaman.
pengguna api ini dapat melakukan CRUD ( CREATE, READ, UPDATE, DELETE) pada data tanaman

# Query SQL Pembuatan Tabel
Berikut adalah query SQL untuk membuat tabel plants dalam database plant_management
- SQL database plant_management
```sql
CREATE DATABASE plant_management;

USE plant_management;

CREATE TABLE plants (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    type VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL CHECK(price > 0),
    stock INT NOT NULL CHECK(stock >= 0)
);
```
- Source Code
```sql
<?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'], '/'));
}

function getConnection() {
    $host = 'localhost';
    $db   = 'plant_management';
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = NULL) {
    http_response_code($status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}

function validatePlants($name, $type, $price, $stock) {
    $errors = [];
    if (empty($name)) {
        $errors[] = "Name is required";
    }
    if (empty($type)) {
        $errors[] = "Type is required";
    }
    if (!is_numeric($price) || $price <= 0) {
        $errors[] = "Price must be a positive number";
    }
    if (!is_numeric($stock) || $stock < 0) {
        $errors[] = "Stock must be a non-negative number";
    }
    return $errors;
}

$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            if ($request[0] === 'search') {
                $searchTerm = $_GET['term'] ?? '';
                $stmt = $db->prepare("SELECT * FROM plants WHERE name LIKE ? OR type LIKE ?");
                $searchTerm = "%$searchTerm%";
                $stmt->execute([$searchTerm, $searchTerm]);
                $plants = $stmt->fetchAll();
                response(200, $plants);
            } else {
                $id = $request[0];
                $stmt = $db->prepare("SELECT * FROM plants WHERE id = ?");
                $stmt->execute([$id]);
                $plant = $stmt->fetch();
                if ($plant) {
                    response(200, $plant);
                } else {
                    response(404, ["message" => "Plant not found"]);
                }
            }
        } else {
            $page = isset($_GET['page']) ? (int)$_GET['page'] : 1;
            $limit = isset($_GET['limit']) ? (int)$_GET['limit'] : 10;
            $offset = ($page - 1) * $limit;

            $stmt = $db->prepare("SELECT * FROM plants LIMIT ? OFFSET ?");
            $stmt->bindValue(1, $limit, PDO::PARAM_INT);
            $stmt->bindValue(2, $offset, PDO::PARAM_INT);
            $stmt->execute();
            $plants = $stmt->fetchAll();

            $totalStmt = $db->query("SELECT COUNT(*) FROM plants");
            $total = $totalStmt->fetchColumn();

            response(200, [
                'plants' => $plants,
                'total' => $total,
                'page' => $page,
                'limit' => $limit
            ]);
        }
        break;
    
    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        $errors = validatePlants($data->name ?? '', $data->type ?? '', $data->price ?? '', $data->stock ?? '');
        if (!empty($errors)) {
            response(400, ["errors" => $errors]);
        }
        $sql = "INSERT INTO plants (name, type, price, stock) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->type, $data->price, $data->stock])) {
            response(201, ["message" => "Plant created", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "Failed to create plant"]);
        }
        break;
    
    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "Plant ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        $errors = validatePlants($data->name ?? '', $data->type ?? '', $data->price ?? '', $data->stock ?? '');
        if (!empty($errors)) {
            response(400, ["errors" => $errors]);
        }
        $sql = "UPDATE plants SET name = ?, type = ?, price = ?, stock = ? WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->name, $data->type, $data->price, $data->stock, $id])) {
            if ($stmt->rowCount() > 0) {
                response(200, ["message" => "Plant updated"]);
            } else {
                response(404, ["message" => "Plant not found or no changes made"]);
            }
        } else {
            response(500, ["message" => "Failed to update plant"]);
        }
        break;
    
    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "Plant ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM plants WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            if ($stmt->rowCount() > 0) {
                response(200, ["message" => "Plant deleted"]);
            } else {
                response(404, ["message" => "Plant not found"]);
            }
        } else {
            response(500, ["message" => "Failed to delete plant"]);
        }
        break;
    
    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
?>
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
### SCREEN SHOOT
- GET All
- GET berdasarkan ID
- GET Berdasarkan Nama
- POST Menambahkan List Tanaman Baru
- PUT Mempebaharui List Tanaman berdasarkan ID
- DELETE Menghapus salah satu List Tanaman berdasarkan ID
