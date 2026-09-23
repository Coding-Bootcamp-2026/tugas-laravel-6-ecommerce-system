# Brief Tugas: Aplikasi CRUD Laravel dengan Eloquent ORM (Sistem E-Commerce)

## Deskripsi Tugas
Dalam tugas ini, Anda diminta untuk membuat aplikasi berbasis web menggunakan framework Laravel dengan penekanan kuat pada penggunaan **Eloquent ORM**. Aplikasi ini akan mensimulasikan sistem E-Commerce (Toko Online) skala menengah. Anda akan mempraktikkan keseluruhan proses dari instalasi, merancang database yang cukup kompleks, menggunakan Eloquent untuk relasi antar tabel, hingga membuat operasi CRUD penuh.

---

## Spesifikasi ERD (Entity Relationship Diagram)
Aplikasi ini memiliki skema yang kompleks dengan **9 tabel** yang saling berelasi. Anda diwajibkan untuk membuat *Migration* dan *Model (Eloquent)* untuk kesembilan tabel di bawah ini:

1. **`users`** (Tabel Autentikasi Sistem bawaan Laravel)
   - **Kolom:** `id`, `name`, `email`, `password`, `timestamps`
   - **Relasi:** Memiliki satu Pelanggan (*Has One Customer*).

2. **`customers`** (Profil Detail Pelanggan)
   - **Kolom:** `id`, `user_id` (Foreign Key), `phone`, `address`, `timestamps`
   - **Relasi:** Milik satu User (*Belongs to User*), Memiliki banyak Pesanan (*Has Many Orders*).

3. **`categories`** (Kategori Produk)
   - **Kolom:** `id`, `name`, `description`, `timestamps`
   - **Relasi:** Memiliki banyak Produk (*Has Many Products*).

4. **`products`** (Data Produk Utama)
   - **Kolom:** `id`, `category_id` (Foreign Key), `name`, `description`, `price`, `stock`, `timestamps`
   - **Relasi:** Milik satu Kategori (*Belongs to Category*), Memiliki banyak Gambar (*Has Many Product Images*).

5. **`product_images`** (Galeri Gambar Produk)
   - **Kolom:** `id`, `product_id` (Foreign Key), `image_url`, `timestamps`
   - **Relasi:** Milik satu Produk (*Belongs to Product*).

6. **`orders`** (Data Transaksi Pesanan Utama)
   - **Kolom:** `id`, `customer_id` (Foreign Key), `order_date`, `total_amount`, `status`, `timestamps`
   - **Relasi:** Milik satu Pelanggan (*Belongs to Customer*), Memiliki banyak Item (*Has Many Order Items*), Memiliki satu Pembayaran (*Has One Payment*), Memiliki satu Pengiriman (*Has One Shipping Detail*).

7. **`order_items`** (Detail Produk di Dalam Pesanan)
   - **Kolom:** `id`, `order_id` (Foreign Key), `product_id` (Foreign Key), `quantity`, `price_at_purchase`, `timestamps`
   - **Relasi:** Milik satu Pesanan (*Belongs to Order*), Milik satu Produk (*Belongs to Product*).

8. **`payments`** (Riwayat Transaksi Pembayaran)
   - **Kolom:** `id`, `order_id` (Foreign Key), `payment_method`, `amount`, `payment_date`, `timestamps`
   - **Relasi:** Milik satu Pesanan (*Belongs to Order*).

9. **`shipping_details`** (Informasi Resi Logistik Pengiriman)
   - **Kolom:** `id`, `order_id` (Foreign Key), `shipping_address`, `tracking_number`, `status`, `timestamps`
   - **Relasi:** Milik satu Pesanan (*Belongs to Order*).

---

## Tahapan Pengerjaan Tugas Bagian 1: Instalasi & Database

### 1. Install Laravel Project
- Buat project Laravel baru menggunakan Composer dengan nama `ecommerce-app`.

### 2. Buat Database dan Konfigurasi Koneksi
- Buat database `db_ecommerce` di MySQL/MariaDB.
- Sesuaikan konfigurasi koneksi pada file `.env`.

### 3. Buat Migration
- Buat file migration untuk kesembilan tabel di atas secara berurutan. Tabel master (tanpa foreign key) harus dibuat lebih dahulu.
- Definisikan constraint *Foreign Key* dengan tipe `Cascade On Delete`.
- Jalankan perintah `php artisan migrate`.

### 4. Buat Model dengan Eloquent ORM
- Buat class Model untuk setiap tabel dan tentukan properti `$fillable`.
- Definisikan metode **Relasi Eloquent** (seperti `hasOne`, `hasMany`, `belongsTo`) di dalam class Model sesuai ERD.

### 5. Isi Data Dummy (Seeder)
- Buat *Database Seeder* untuk mengotomatiskan pengisian data awal.
- Pastikan ke-9 tabel terisi secara fungsional. Jalankan `php artisan db:seed`.

---

## Tahapan Pengerjaan Tugas Bagian 2: Implementasi Dashboard CRUD
Pada bagian ini, Anda wajib mendemonstrasikan implementasi CRUD (Create, Read, Update, Delete) yang dijabarkan per fitur. Gunakan *Route Resource* untuk mempermudah pengerjaan.

### 6. Desain Layout Master (Navbar)
- **Layout:** Buat file *master template* (contoh: `resources/views/layouts/app.blade.php`).
- **Styling:** Sisipkan library Bootstrap/Tailwind CSS. 
- **Navigasi:** Buat barisan **Navbar** di atas layar yang berisikan tautan navigasi untuk berpindah antar 4 menu utama: Kategori, Produk, Pelanggan, dan Pesanan.
- Sediakan blok `@yield('content')` untuk membungkus konten dinamis antar halaman.

### 7. Fitur 1: Manajemen Kategori (Tabel `categories`)
- **Route:** Definisikan route resource `categories`.
- **Controller:** Generate `CategoryController` menggunakan Artisan.
- **View Read (Index):** Tampilkan daftar Kategori dalam format tabel HTML (ID, Nama, Deskripsi).
- **View Create & Store:** Buat form untuk field Nama dan Deskripsi. Gunakan `Category::create()` pada Controller untuk menyimpan data baru ke tabel.
- **View Edit & Update:** Buat form serupa dengan mode *pre-filled* (terisi nilai lama). Gunakan metode `$category->update()` untuk menyimpan perubahan.
- **Delete:** Hapus kategori menggunakan relasi model `delete()`.

### 8. Fitur 2: Manajemen Produk (Tabel `products` dan `product_images`)
- **Route & Controller:** Definisikan rute dan buat `ProductController`.
- **View Read (Index):** Tampilkan daftar Produk. Pastikan Anda menerapkan teknik **Eager Loading** (`Product::with('category')`) di controller agar *loading* data kategori tidak terkena masalah N+1 Query.
- **View Create & Store (Multi-Tabel Insert):**
  - Di method `create`, kirim data `Category::all()` untuk membuat Dropdown (Select) pilihan Kategori di halaman view HTML.
  - Pada View Form Tambah, tambahkan 1 input kolom teks opsional untuk **URL Gambar**.
  - Saat `store`, insert data utama ke tabel `products` lebih dulu. Setelah itu, jika input URL Gambar tidak kosong, catat ke tabel `product_images` dengan merujuk ke `$product->id` yang baru saja terbuat.
- **View Edit, Update, & Show:** Buat rincian selengkapnya untuk memodifikasi produk. Halaman Show harus menampilkan detail kategori beserta *link* gambar jika ada.
- **Delete:** Menghapus data produk. Record gambar di `product_images` harusnya otomatis musnah terhapus berkat fitur *Cascade On Delete*.

### 9. Fitur 3: Manajemen Pelanggan (Tabel `users` dan `customers`)
- **Route & Controller:** Buat `CustomerController` dan definisikan rute resource.
- **View Read (Index):** Tampilkan tabel daftar Pelanggan dengan mengkombinasikan data dari relasi `user` (Nama, Email) dan data `customer` sendiri (Telepon, Alamat).
- **View Create & Store (Multi-Tabel Insert Lanjutan):**
  - Rancang satu form HTML berisikan kolom: Nama, Email, Password, Telepon, dan Alamat.
  - Pada method `store`, operasi Insert harus berurutan. Pertama, lakukan `User::create(...)` untuk menyimpan akun sistem beserta password yang di-hash. Kedua, simpan rincian identitas sisanya ke dalam tabel `customers` yang membawa variabel `$user->id`.
- **View Edit & Update:** Pisahkan proses update menjadi 2 baris *query* update terpisah di dalam controller (update tabel user, kemudian update tabel customer).
- **Delete:** Hapus berdasarkan model User. Identitasnya di tabel `customers` akan otomatis terhapus bersama.

### 10. Fitur 4: Manajemen Pesanan (Tabel `orders`, `order_items`, `payments`, `shipping_details`)
- **Route & Controller:** Buat rute resource dan `OrderController`.
- **View Read (Index):** Tampilkan rangkuman transaksi ringkas (Kode Pesanan, Nama Pelanggan, Tanggal, Total Pembayaran, Status Pesanan).
- **View Create & Store (Insert ke 4 Tabel Sekaligus):**
  - Form Pembuatan Pesanan harus menampilkan dua Dropdown utama: Pilihan Pelanggan dan Pilihan Produk (yang stoknya > 0). Lengkapi juga dengan kolom Kuantitas beli, Alamat Kirim, dan Metode Bayar (Dropown bank transfer/e-wallet).
  - Saat di-submit (`store`), susun urutan simpan/insert sebagai berikut:
    1. Buat record transaksi utama di tabel `orders` dengan nominal total (Harga Produk × Kuantitas).
    2. Simpan catatan barang apa yang dibeli ke tabel `order_items`. Di tahap ini, lakukan perintah kurangi Stok (*decrement*) pada produk di database.
    3. Catat metode dan bukti lunas ke tabel `payments`.
    4. Masukkan data alamat ekspedisi pembeli ke tabel `shipping_details`.
- **View Show (Detail Faktur):** Buat sebuah rincian halaman selayaknya Invoice resmi. Anda harus merelasikan dan memanggil data dari ke-4 tabel tersebut sekaligus dalam satu halaman (Detail Pengiriman, Informasi Pembeli, Bukti Bayar, serta Daftar Item Keranjang Belanja).
- **View Edit & Update Status:** Pada bagian *Update*, pengguna hanya diizinkan mengubah dan memperbarui *Status Pesanan* (misalnya dari "Processing" diubah menjadi "Shipped" atau "Completed").

---

## Ketentuan Ekstra
- Jangan lupa menyisipkan logika **Validasi** sederhana (`$request->validate()`) di tiap fungsi simpan/ubah Controller agar data yang masuk ke database terjamin kesesuaian tipenya.
- Implementasikan UI dengan rapi agar proses transisi pindah halaman (*flash message alert* dll) berjalan secara responsif.
