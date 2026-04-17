## 🧩 Commit 1 Reflection Notes

Pada tahap ini, saya mempelajari bagaimana server menerima request dari browser menggunakan TcpListener dan TcpStream.

Setiap koneksi dibaca menggunakan BufReader, yang memungkinkan pembacaan data secara efisien. Request HTTP dibaca per baris menggunakan .lines(), kemudian dihentikan saat menemukan baris kosong dengan .take_while(|line| !line.is_empty()).

Baris kosong ini menandakan akhir dari HTTP header. Dari output terminal, terlihat bahwa browser mengirimkan request dalam bentuk teks yang berisi method (GET), host, user-agent, dan lain-lain.

Hal ini membantu saya memahami bagaimana komunikasi dasar antara browser dan server terjadi melalui protokol HTTP.