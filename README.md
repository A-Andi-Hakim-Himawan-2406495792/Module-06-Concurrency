## 🧩 Commit 1 Reflection Notes

Pada tahap ini, saya mempelajari bagaimana server menerima request dari browser menggunakan TcpListener dan TcpStream.

Setiap koneksi dibaca menggunakan BufReader, yang memungkinkan pembacaan data secara efisien. Request HTTP dibaca per baris menggunakan .lines(), kemudian dihentikan saat menemukan baris kosong dengan .take_while(|line| !line.is_empty()).

Baris kosong ini menandakan akhir dari HTTP header. Dari output terminal, terlihat bahwa browser mengirimkan request dalam bentuk teks yang berisi method (GET), host, user-agent, dan lain-lain.

Hal ini membantu saya memahami bagaimana komunikasi dasar antara browser dan server terjadi melalui protokol HTTP.

## 🌐 Commit 2 Reflection Notes

Pada tahap ini, server mulai mengirimkan response berupa HTML ke browser.

Response HTTP disusun secara manual, terdiri dari status line (HTTP/1.1 200 OK), header (Content-Length), dan body (isi HTML).

File HTML dibaca menggunakan fs::read_to_string, kemudian panjangnya dihitung untuk dimasukkan ke dalam header Content-Length. Header ini penting agar browser mengetahui ukuran data yang diterima.

Response kemudian digabungkan menggunakan format! dengan pemisah \r\n sesuai standar HTTP.

Dari tahap ini, saya memahami bahwa browser membutuhkan format response HTTP yang lengkap dan benar agar dapat menampilkan halaman dengan baik.

![Commit 2 Screenshot](/assets/images/commit2.jpeg)

## 🔍 Commit 3 Reflection Notes

Pada tahap ini, server mulai dapat membedakan request berdasarkan URL.

Request line pertama diambil dan dibandingkan. Jika request adalah "GET / HTTP/1.1", maka server mengembalikan hello.html, jika tidak maka mengembalikan 404.html.

Refactoring dilakukan dengan memisahkan status line dan nama file. Hal ini membuat kode lebih rapi dan mudah dikembangkan dibandingkan sebelumnya yang hanya mengembalikan satu response.

Dengan pendekatan ini, server sudah memiliki konsep routing sederhana.

![Commit 3 Screenshot](/assets/images/404.jpeg)

## 🐢 Commit 4 Reflection Notes

Pada tahap ini dilakukan simulasi request slow dengan menambahkan delay selama 10 detik pada endpoint /sleep.

Ketika membuka dua tab browser, satu mengakses /sleep dan satu lagi mengakses /, terlihat bahwa request kedua ikut tertunda.

Hal ini terjadi karena server masih menggunakan single thread, sehingga hanya dapat memproses satu request dalam satu waktu.

Dari sini terlihat bahwa single-threaded server tidak efisien untuk menangani banyak request secara bersamaan.

## ⚡ Commit 5 Reflection Notes
Pada tahap ini, server diubah menjadi multithreaded menggunakan ThreadPool.

ThreadPool bekerja dengan membuat sejumlah worker thread di awal, lalu setiap request yang masuk akan dikirim sebagai job ke worker melalui channel.

Digunakan mpsc (multi-producer, single-consumer) untuk mengirim job dari main thread ke worker thread. Untuk memungkinkan beberapa thread mengakses data yang sama, digunakan Arc dan Mutex.

Arc memungkinkan data dimiliki oleh beberapa thread
Mutex memastikan hanya satu thread yang mengakses data pada satu waktu (menghindari race condition)

Setelah menggunakan ThreadPool, server dapat menangani beberapa request secara paralel. Saat diuji kembali dengan endpoint /sleep, request lain tidak lagi ikut tertunda, yang menunjukkan bahwa server sudah tidak blocking.

## 🎁 Commit Bonus Reflection Notes

### (Function improvement)

Pada tahap ini, saya melakukan perbaikan pada fungsi pembuatan `ThreadPool`, yaitu dengan mengubah fungsi `new` menjadi `build` yang mengembalikan `Result<ThreadPool, PoolCreationError>`.

Sebelumnya, fungsi `new` menggunakan `assert!(size > 0)` untuk memastikan ukuran thread pool valid. Namun, pendekatan ini memiliki kelemahan karena jika kondisi tidak terpenuhi (misalnya `size = 0`), program akan langsung panic dan berhenti secara paksa. Hal ini kurang ideal, terutama untuk aplikasi yang lebih kompleks atau berjalan di lingkungan production, karena tidak memberikan kesempatan untuk menangani error secara elegan.

Dengan menggunakan `build`, validasi dilakukan menggunakan kondisi biasa (`if size == 0`) dan mengembalikan `Err(PoolCreationError)` jika input tidak valid. Pendekatan ini memungkinkan caller (misalnya di `main.rs`) untuk menangani error sesuai kebutuhan, misalnya dengan logging, retry, atau fallback ke nilai default.

Saya juga menyadari bahwa penggunaan `Result` merupakan idiom yang sangat penting dalam Rust, karena mendorong penanganan error secara eksplisit dan aman. Dibandingkan dengan panic, pendekatan ini membuat program lebih robust dan tidak mudah crash hanya karena kesalahan input yang sebenarnya bisa diantisipasi.

Selain itu, dengan tetap menggunakan `.unwrap()` di `main.rs`, saya memahami bahwa kita masih bisa memilih untuk memperlakukan error sebagai fatal, tetapi keputusan tersebut berada di level pemanggil, bukan dipaksakan di dalam fungsi `ThreadPool` itu sendiri. Ini memberikan fleksibilitas desain yang lebih baik.

Dari perubahan ini, saya belajar bahwa desain API yang baik tidak hanya fokus pada fungsi yang berjalan dengan benar, tetapi juga bagaimana menangani kondisi error secara aman dan fleksibel.
