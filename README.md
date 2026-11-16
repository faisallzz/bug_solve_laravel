# Bug Battle: Kasus 01, 02, dan 03

Repository ini berisi penyelesaian tugas **Problem Solving Bug** pada Laravel.

## Daftar Isi

- [Struktur Repository](#struktur-repository)
- [Cara Menjalankan](#cara-menjalankan)
- [Kasus 01: CRUD Manajemen Buku](#kasus-01-crud-manajemen-buku)
- [Kasus 02: Relasi Buku dan Kategori](#kasus-02-relasi-buku-dan-kategori)
- [Kasus 03: Pencarian dan Filter Buku](#kasus-03-pencarian-dan-filter-buku)

---

## Struktur Repository

```
.
├── bug-battle-case-01/
├── bug-battle-case-02/
├── bug-battle-case-03/
├── screenshots/
│   ├── kasus-01/
│   ├── kasus-02/
│   └── kasus-03/
└── README.md
```

## Cara Menjalankan

Prasyarat: PHP, Composer, MySQL (Laragon).

Untuk setiap kasus, masuk ke foldernya lalu:

```bash
composer install
copy .env.example .env        # Windows (Linux/Mac: cp .env.example .env)
php artisan key:generate
```

Atur bagian database di `.env`, lalu buat database sesuai kasus.

| Kasus | Nama Database | Perintah |
|---|---|---|
| 01 | `bug_battle_case_01` | `php artisan migrate` |
| 02 | `bug_battle_case_02` | `php artisan migrate` lalu `php artisan db:seed --class=CategorySeeder` |
| 03 | `bug_battle_case_03` | `php artisan migrate` lalu `php artisan db:seed` |

Jalankan server dan buka `http://127.0.0.1:8000/books`:

```bash
php artisan optimize:clear
php artisan serve
```

---

## Kasus 01: CRUD Manajemen Buku

**Fokus:** Route, Controller, Model, Migration, Blade, validasi dasar.

### Gejala yang diselidiki

1. Halaman daftar buku tidak dapat dibuka dengan benar.
2. Form tambah buku memiliki perilaku yang tidak sesuai.
3. Data buku tidak tampil setelah proses penyimpanan.
4. Tombol hapus tidak bekerja sebagaimana mestinya.

### Kondisi akhir kode

| Komponen | Implementasi |
|---|---|
| Route | `GET /books`, `GET /books/create`, `POST /books`, `DELETE /books/{id}` |
| Controller | `index`, `create`, `store` (dengan validasi), `destroy` (`findOrFail`) |
| Model | `$fillable`: `title`, `author`, `published_year` |
| View | `index` memakai `@forelse ($books as $book)`, form hapus memakai `@method('DELETE')` |
| Validasi | `title` dan `author` required, string, max 255. `published_year` required, integer, min 1000, max tahun berjalan |

### Hasil pengujian

**1. Daftar buku kosong**

![Kasus 01 - Daftar kosong](screenshots/kasus-01/kasus01-01-daftar-kosong.png)

**2. Form tambah buku (data valid)**

![Kasus 01 - Form tambah](screenshots/kasus-01/kasus01-02-form-tambah.png)

**3. Setelah simpan: pesan sukses dan data tampil**

![Kasus 01 - Setelah simpan](screenshots/kasus-01/kasus01-03-setelah-simpan.png)

**4. Validasi gagal**

![Kasus 01 - Validasi gagal](screenshots/kasus-01/kasus01-04-validasi-gagal.png)

**5. Konfirmasi hapus**

![Kasus 01 - Konfirmasi hapus](screenshots/kasus-01/kasus01-05-konfirmasi-hapus.png)

**6. Setelah hapus**

![Kasus 01 - Setelah hapus](screenshots/kasus-01/kasus01-06-setelah-hapus.png)

---

## Kasus 02: Relasi Buku dan Kategori

**Fokus:** Migration, foreign key, Eloquent relationship (`belongsTo`, `hasMany`), eager loading.

### Gejala yang diselidiki

1. Migrasi tabel buku dapat gagal atau relasinya tidak terjamin.
2. Daftar buku gagal mengambil kategori.
3. Nama kategori tidak tampil dengan benar.
4. Form buku tidak menampilkan pilihan kategori secara benar.

### Kondisi akhir kode

| Komponen | Implementasi |
|---|---|
| Migration | `categories` (`..._000001`) dibuat sebelum `books` (`..._000002`) |
| Foreign key | `foreignId('category_id')->constrained()->cascadeOnDelete()` |
| Model `Book` | `belongsTo(Category::class)` |
| Model `Category` | `hasMany(Book::class)` |
| Controller | `Book::with('category')` pada `index`, `Category::orderBy('name')` pada `create` |
| View | `$book->category->name` dan dropdown dari `$categories` |

### Hasil pengujian

**1. Form tambah buku dengan dropdown kategori**

![Kasus 02 - Form tambah](screenshots/kasus-02/kasus02-01-form-tambah.png)

**2. Validasi gagal**

![Kasus 02 - Validasi gagal](screenshots/kasus-02/kasus02-02-validasi-gagal.png)

**3. Daftar buku dengan beberapa kategori**

![Kasus 02 - Daftar buku](screenshots/kasus-02/kasus02-03-daftar-buku.png)

**4. Bukti relasi di Tinker**

![Kasus 02 - Tinker](screenshots/kasus-02/kasus02-04-tinker.png)

**5. Foreign key pada database**

![Kasus 02 - Foreign key database](screenshots/kasus-02/kasus02-05-database-fk.png)

---

## Kasus 03: Pencarian dan Filter Buku

**Fokus:** Request, query string, Eloquent `where` / `orWhere`, `LIKE`, filter kategori, Blade.

### Gejala yang diselidiki

1. Kata kunci dari form tidak terbaca oleh Controller.
2. Pencarian hanya menemukan kecocokan persis.
3. Filter kategori tidak bekerja.
4. Variabel hasil pencarian tidak tersedia di Blade.
5. Input pencarian tidak dipertahankan setelah submit.

### Kondisi akhir kode

```php
$books = Book::with('category')
    ->when($keyword, function ($query) use ($keyword) {
        $query->where(function ($q) use ($keyword) {
            $q->where('title', 'like', "%{$keyword}%")
              ->orWhere('author', 'like', "%{$keyword}%");
        });
    })
    ->when($request->category, function ($query) use ($request) {
        $query->where('category_id', $request->category);
    })
    ->get();
```

Poin penting:

- Nama parameter form (`keyword`, `category`) sama dengan yang dibaca Controller.
- Pencarian parsial memakai `LIKE %keyword%`.
- Kondisi judul dan penulis dibungkus satu grup `where(function ...)`, sehingga `orWhere` tidak merusak filter kategori.
- Variabel `books` dan `categories` dikirim lewat `compact()`.
- Input dipertahankan lewat `request('keyword')` dan atribut `selected` pada kategori.
- Hasil kosong ditangani oleh `@forelse ... @empty` dengan pesan "Buku tidak ditemukan."

### Hasil pengujian

**1. Semua data (tanpa filter)**

![Kasus 03 - Semua data](screenshots/kasus-03/kasus03-01-semua-data.png)

**2. Pencarian parsial judul (`?keyword=lar`)**

![Kasus 03 - Pencarian judul](screenshots/kasus-03/kasus03-02-cari-judul.png)

**3. Pencarian penulis (`?keyword=andi`)**

![Kasus 03 - Pencarian penulis](screenshots/kasus-03/kasus03-03-cari-penulis.png)

**4. Filter kategori (`?category=2`)**

![Kasus 03 - Filter kategori](screenshots/kasus-03/kasus03-04-filter-kategori.png)

**5. Gabungan keyword dan kategori, hasil kosong (`?keyword=python&category=2`)**

![Kasus 03 - Gabungan dan kosong](screenshots/kasus-03/kasus03-05-gabungan-kosong.png)

**6. Input dipertahankan setelah submit**

![Kasus 03 - Input dipertahankan](screenshots/kasus-03/kasus03-06-input-dipertahankan.png)

---

## Catatan

- File `.env` tidak disertakan di repository. Gunakan `.env.example` sebagai acuan.
- Folder `vendor/` tidak disertakan. Jalankan `composer install` terlebih dahulu.