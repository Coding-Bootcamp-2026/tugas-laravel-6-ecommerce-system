# Brief Tugas: Aplikasi CRUD Laravel dengan Eloquent ORM (Sistem E-Commerce)

## Deskripsi Tugas
Dalam tugas ini, Anda diminta untuk membuat aplikasi berbasis web menggunakan framework Laravel dengan penekanan kuat pada penggunaan **Eloquent ORM**. Aplikasi ini akan mensimulasikan sistem E-Commerce (Toko Online) skala menengah. Anda akan mempraktikkan keseluruhan proses dari instalasi, merancang database yang cukup kompleks, menggunakan Eloquent untuk relasi antar tabel, hingga membuat operasi CRUD penuh.

---

## Spesifikasi ERD (Entity Relationship Diagram)
Aplikasi ini memiliki skema yang kompleks dengan **9 tabel** yang saling berelasi. Anda diwajibkan untuk membuat *Migration* dan *Model (Eloquent)* untuk kesembilan tabel di bawah ini:

1. **`users`** (Tabel Autentikasi bawaan Laravel)
   - Kolom: `id`, `name`, `email`, `password`, `timestamps`
2. **`customers`** (Profil Pelanggan)
   - Kolom: `id`, `user_id` (FK), `phone`, `address`, `timestamps`
   - Relasi: *Belongs to* `users`
3. **`categories`** (Kategori Produk)
   - Kolom: `id`, `name`, `description`, `timestamps`
4. **`products`** (Data Produk Utama)
   - Kolom: `id`, `category_id` (FK), `name`, `description`, `price`, `stock`, `timestamps`
   - Relasi: *Belongs to* `categories`
5. **`product_images`** (Galeri Gambar Produk)
   - Kolom: `id`, `product_id` (FK), `image_url`, `timestamps`
   - Relasi: *Belongs to* `products`
6. **`orders`** (Data Transaksi Pesanan)
   - Kolom: `id`, `customer_id` (FK), `order_date`, `total_amount`, `status`, `timestamps`
   - Relasi: *Belongs to* `customers`
7. **`order_items`** (Detail Item Produk yang Dipesan)
   - Kolom: `id`, `order_id` (FK), `product_id` (FK), `quantity`, `price_at_purchase`, `timestamps`
   - Relasi: *Belongs to* `orders`, *Belongs to* `products`
8. **`payments`** (Data Pembayaran Transaksi)
   - Kolom: `id`, `order_id` (FK), `payment_method`, `amount`, `payment_date`, `timestamps`
   - Relasi: *Belongs to* `orders`
9. **`shipping_details`** (Informasi Pengiriman Barang)
   - Kolom: `id`, `order_id` (FK), `shipping_address`, `tracking_number`, `status`, `timestamps`
   - Relasi: *Belongs to* `orders`

---

## Tahapan Pengerjaan Tugas

### 1. Install Laravel Project
- Buka terminal/command prompt.
- Buat project Laravel baru menggunakan Composer dengan nama `ecommerce-app`:
  ```bash
  composer create-project laravel/laravel ecommerce-app
  ```

### 2. Buat Database dan Konfigurasi Koneksi
- Buat database baru di MySQL (misal menggunakan phpMyAdmin atau DataGrip) dengan nama `db_ecommerce`.
- Buka file `.env` di project Laravel Anda, ubah konfigurasi agar terhubung dengan database:
  ```env
  DB_CONNECTION=mysql
  DB_HOST=127.0.0.1
  DB_PORT=3306
  DB_DATABASE=db_ecommerce
  DB_USERNAME=root
  DB_PASSWORD=
  ```

### 3. Buat Migration
- Buat file migration untuk kesembilan tabel di atas (`php artisan make:migration ...`).
- **Penting:** Perhatikan urutan pembuatan migration. Tabel *master* (seperti `users` dan `categories`) harus dibuat lebih dahulu. Setelah itu baru buat tabel yang membutuhkan *Foreign Key* (seperti `products` dan `orders`).
- Definisikan constraint Foreign Key dengan tipe *Cascade* pada kolom yang berelasi, lalu jalankan `php artisan migrate`.

### 4. Buat Model dengan Eloquent ORM
- Buat class Model untuk setiap tabel.
- Tentukan array `$fillable` (untuk keamanan *Mass Assignment*) di semua Model.
- Definisikan metode **Relasi Eloquent** (seperti `hasOne`, `hasMany`, `belongsTo`) di setiap file Model sesuai dengan struktur ERD di atas.

### 5. Isi Data Dummy (Seeder)
- Buat *Seeder* atau *Factory* minimal untuk tabel `categories` dan `products` agar Anda memiliki data untuk diuji coba di halaman View.
- Masukkan minimal 5 kategori dan 15 produk dummy ke dalam database.
- Jalankan perintah `php artisan db:seed`.

> **Fokus Utama CRUD:**
> *Untuk tahapan 6 hingga 12, Anda diminta untuk berfokus membangun antarmuka web dan operasi CRUD khusus untuk entitas/tabel **Data Produk (`products`)**.*

### 6. Buat Route
- Buka file `routes/web.php`.
- Definisikan rute untuk mengelola Data Produk. Sangat disarankan memanfaatkan fitur *Route Resource*:
  ```php
  Route::resource('products', ProductController::class);
  ```

### 7. Buat Controller
- Generate controller dengan perintah: `php artisan make:controller ProductController --resource`.
- Pastikan seluruh proses pengambilan dan manipulasi data di dalam Controller **wajib menggunakan metode Eloquent ORM** (seperti `Product::create()`, `Product::findOrFail()`, dll), bukan DB Query Builder konvensional.

### 8. Buat View Read (Tampilkan Semua Data)
- Pada method `index` di Controller, ambil seluruh data produk dari database. Gunakan teknik **Eager Loading** Eloquent (contoh: `Product::with('category')->get()`) untuk mencegah masalah performa *N+1 Query Problem*.
- Buat file `resources/views/products/index.blade.php`.
- Tampilkan data di HTML dengan rapi (misal menggunakan tabel yang memuat Nama Produk, Kategori, Harga, dan Stok). Sediakan tombol opsi: "Tambah Produk", "Detail", "Edit", dan "Hapus".

### 9. Buat View Create (Tambah Data Baru)
- Pada method `create`, ambil seluruh data dari tabel `categories` menggunakan Eloquent (`Category::all()`) dan lewatkan (passing) ke form View.
- Di View `resources/views/products/create.blade.php`, buat form HTML. Gunakan *tag select/dropdown* untuk memilih Kategori.
- Pada method `store`, validasi request yang masuk, lalu simpan baris data produk baru ke tabel menggunakan metode mass-assignment Eloquent. *Redirect* kembali ke halaman *index* dengan *flash message*.

### 10. Buat View Detail Data (Show)
- Pada method `show($id)`, ambil data produk spesifik menggunakan method `Product::findOrFail($id)`.
- Passing data produk tersebut ke `resources/views/products/show.blade.php`.
- Tampilkan seluruh kolom/informasi dari produk, termasuk nama kategori dari produk tersebut (memanfaatkan relasi Eloquent).

### 11. Buat View Update (Edit Data)
- Pada method `edit($id)`, muat rincian dari satu produk beserta semua data Kategori.
- Tampilkan form edit di `resources/views/products/edit.blade.php` dengan fitur *pre-filled* (input sudah terisi otomatis berdasarkan data lama).
- Pada method `update`, lakukan proses update menggunakan fungsi bawaan Eloquent (seperti `$product->update()`), kemudian *redirect* kembali ke halaman *index*.

### 12. Proses Delete (Hapus Data)
- Pada method `destroy($id)`, muat objek produk, lalu hapus rekaman data menggunakan fungsi Eloquent `$product->delete()`.
- Kembalikan pengguna (redirect) ke halaman index disertai pesan konfirmasi bahwa produk telah berhasil dihapus.
