## 📘 Rust Web Server — Tutorial Modul 6

Tutorial ini mengacu pada The Rust Programming Language Chapter 20.
Pada modul ini, saya membangun web server sederhana menggunakan Rust dari nol, mulai dari server single-thread hingga multithreaded menggunakan ThreadPool.

Melalui proses ini, saya tidak hanya mengikuti implementasi, tetapi juga memahami bagaimana web server bekerja di balik framework modern seperti Django atau Express.

---

## 🧩 Commit 1 Reflection Notes

### (Handle-connection, check response)

Pada tahap ini, saya mempelajari bagaimana server menerima koneksi dari browser menggunakan `TcpListener` dan `TcpStream`.

Setiap koneksi yang masuk diproses oleh fungsi `handle_connection`. Untuk membaca data dari client, digunakan `BufReader` yang membantu membaca stream secara efisien dengan buffering.

Request HTTP dibaca baris per baris menggunakan `.lines()`, kemudian dihentikan saat menemukan baris kosong dengan `.take_while(|line| !line.is_empty())`. Baris kosong ini menandakan akhir dari HTTP header.

Dari output terminal, terlihat bahwa request HTTP memiliki struktur seperti:

* Method (GET)
* Path (/)
* Header (Host, User-Agent, dll)

Dari sini saya memahami bahwa komunikasi antara browser dan server sebenarnya berupa teks dengan format tertentu (HTTP)

---

## 🌐 Commit 2 Reflection Notes

### (Returning HTML)

Pada tahap ini, server mulai mengirimkan response berupa halaman HTML ke browser.

Response HTTP disusun secara manual dengan struktur:

* Status line (`HTTP/1.1 200 OK`)
* Header (`Content-Length`)
* Body (isi file HTML)

File HTML dibaca menggunakan `fs::read_to_string`, lalu panjangnya dihitung untuk mengisi header `Content-Length`. Header ini penting agar browser mengetahui kapan response selesai diterima.

Response kemudian digabung menggunakan `format!` dengan separator `\r\n`, yang merupakan standar dalam protokol HTTP.

Dari tahap ini, saya memahami bahwa browser hanya akan menampilkan halaman jika format response HTTP valid. Tidak cukup hanya mengirim HTML, tetapi harus sesuai dengan struktur protokol.

📸 Screenshot hasil:

```md
![Commit 2 Screenshot](/assets/images/commit2.jpeg)
```

---

## 🔍 Commit 3 Reflection Notes

### (Validating request and selectively responding)

Pada tahap ini, saya mengimplementasikan mekanisme routing sederhana berdasarkan request dari browser.

Request line pertama diambil menggunakan:

```rust
let request_line = buf_reader.lines().next().unwrap().unwrap();
```

Struktur request ini berisi method, path, dan versi HTTP, contohnya:

```text
GET / HTTP/1.1
```

Dengan memanfaatkan informasi ini, saya membuat logika:

* Jika path adalah `/`, maka server mengembalikan `hello.html`
* Jika tidak, maka server mengembalikan `404.html`

Refactoring dilakukan dengan memisahkan `status_line` dan `filename` ke dalam satu tuple:

```rust
let (status_line, filename) = ...
```

Pendekatan ini membuat kode:

* Lebih rapi (tidak duplikasi)
* Lebih mudah dikembangkan (tinggal tambah kondisi lainnya)
* Lebih modular

Dari sini saya menyadari bahwa ini adalah bentuk paling sederhana dari routing yang biasanya disediakan oleh framework. Dengan mengimplementasikannya sendiri, saya jadi lebih memahami bagaimana request dipetakan ke response.

📸 Screenshot hasil:

```md
![Commit 3 Screenshot](/assets/images/404.jpeg)
```

---

## 🐢 Commit 4 Reflection Notes

### (Simulation of slow request)

Pada tahap ini dilakukan simulasi request lambat dengan menambahkan delay menggunakan:

```rust
thread::sleep(Duration::from_secs(10));
```

Endpoint `/sleep` akan menyebabkan server menunggu selama 10 detik sebelum memberikan response.

Saat diuji dengan dua tab browser:

* Tab pertama mengakses `/sleep`
* Tab kedua mengakses `/`

Terlihat bahwa request kedua ikut tertunda hingga request pertama selesai. Hal ini terjadi karena server masih menggunakan single thread, sehingga hanya dapat memproses satu request dalam satu waktu.

Dari sini saya memahami bahwa:

* Single-threaded server bersifat blocking
* Satu request lambat dapat menghambat request lain
* Pendekatan ini tidak scalable untuk banyak user

---

## ⚡ Commit 5 Reflection Notes

### (Multithreaded server using Threadpool)

Pada tahap ini, server diubah menjadi multithreaded menggunakan ThreadPool untuk mengatasi masalah blocking yang terjadi pada server single-threaded sebelumnya.

Perubahan penting pada tahap ini adalah penambahan file baru, yaitu `src/lib.rs`. File ini digunakan untuk memisahkan implementasi ThreadPool dari `main.rs`, sehingga kode menjadi lebih modular dan terstruktur. Dengan pemisahan ini, `main.rs` hanya berfokus pada logic server, sedangkan pengelolaan thread ditangani oleh modul terpisah.

ThreadPool diimplementasikan dengan konsep:

1. Membuat sejumlah worker thread di awal (pre-spawned threads)
2. Menyimpan worker dalam sebuah vector
3. Mengirimkan setiap request sebagai job ke worker melalui channel

Di `main.rs`, perubahan utama adalah mengganti pemanggilan langsung:

```rust
handle_connection(stream);
```

menjadi:

```rust
pool.execute(|| {
    handle_connection(stream);
});
```

Dengan demikian, setiap request tidak lagi diproses langsung oleh main thread, tetapi dikirim ke worker thread dalam ThreadPool.

Komunikasi antar thread menggunakan `mpsc` (multi-producer, single-consumer):

* Main thread bertindak sebagai producer yang mengirim job
* Worker thread bertindak sebagai consumer yang menerima job

Karena receiver digunakan oleh banyak thread, digunakan:

* `Arc` (Atomic Reference Counting) → agar data dapat dimiliki bersama oleh beberapa thread
* `Mutex` → untuk memastikan hanya satu thread yang mengakses data pada satu waktu, sehingga menghindari race condition

Alur kerja sistem:

```text
Request masuk → dikirim ke channel → worker mengambil job → dieksekusi
```

Setelah implementasi ini, saya melakukan pengujian ulang dengan endpoint `/sleep`. Berbeda dengan sebelumnya, request lain tidak lagi ikut tertunda. Hal ini menunjukkan bahwa server sudah dapat menangani beberapa request secara paralel.

Dari tahap ini, saya memahami bahwa penggunaan ThreadPool tidak hanya memungkinkan concurrency, tetapi juga lebih efisien dibanding membuat thread baru setiap kali ada request, karena thread dapat digunakan kembali (reuse), sehingga penggunaan resource menjadi lebih optimal.

---

## 🎁 Commit Bonus Reflection Notes

### (Function improvement)

Pada tahap ini, saya memperbaiki desain fungsi `ThreadPool` dengan mengganti `new` menjadi `build` yang mengembalikan `Result`.

Sebelumnya:

```rust
assert!(size > 0);
```

Jika kondisi tidak terpenuhi, program akan panic dan langsung berhenti.

Setelah diubah:

```rust
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError>
```

Jika input tidak valid:

```rust
return Err(PoolCreationError);
```

Perubahan ini memberikan beberapa keuntungan:

* Program tidak langsung crash
* Error dapat ditangani secara fleksibel
* Lebih sesuai dengan prinsip error handling Rust

Saya juga menyadari bahwa penggunaan `Result` memaksa kita untuk memikirkan kemungkinan error sejak awal.

Menariknya, di `main.rs` saya masih menggunakan `.unwrap()`, yang berarti saya tetap memilih untuk panic jika terjadi error. Namun, sekarang keputusan tersebut berada di level pemanggil, bukan di dalam fungsi `ThreadPool`.

Dari sini saya belajar bahwa desain API yang baik tidak hanya tentang fungsi yang berjalan, tetapi juga bagaimana fungsi tersebut menangani error secara aman dan fleksibel.

---

