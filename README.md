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