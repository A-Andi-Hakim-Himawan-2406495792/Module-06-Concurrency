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

