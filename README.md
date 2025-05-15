# Tutorial 9 Advance Programming
Muhammad Albyarto Ghazali (2306241695)


### Subscriber Reflection

> a. What is amqp?

AMQP (Advanced Message Queuing Protocol) adalah sebuah protokol komunikasi yang digunakan untuk pertukaran pesan antar sistem secara async. Protokol ini umum digunakan dalam arsitektur yang melibatkan message broker seperti RabbitMQ, di mana satu komponen sistem (publisher) mengirimkan pesan ke sebuah antrean (queue), dan komponen lain (subscriber) akan memproses message tersebut. Dengan AMQP, komunikasi antar service menjadi lebih terpisah dan tidak saling dependent secara langsung, sehingga sistem menjadi lebih fleksibel.

> b. What does it mean? guest:guest@localhost:5672 , what is the first guest, and what is the second guest, and what is localhost:5672 is for?

Pada bagian `amqp://guest:guest@localhost:5672`, ini adalah format URL koneksi yang digunakan untuk menghubungkan aplikasi ke server AMQP seperti RabbitMQ. Kata `guest` yang pertama adalah username, dan `guest` yang kedua adalah password yang digunakan untuk otentikasi ke server RabbitMQ. Secara default, RabbitMQ memang menyediakan akun `guest` dengan password `guest` untuk keperluan pengujian atau pengembangan. Selanjutnya, `localhost` menunjukkan bahwa server RabbitMQ berjalan di komputer lokal (komputer yang sama dengan aplikasi), dan `5672` adalah nomor port standar yang digunakan oleh protokol AMQP untuk menerima koneksi. Jadi, keseluruhan URL tersebut berarti bahwa aplikasi akan mencoba terhubung ke server RabbitMQ lokal menggunakan akun default melalui port standar AMQP.

---

### Simulation slow subscriber

![image](https://github.com/user-attachments/assets/2e500f6c-79e2-41fb-bd84-f98bd14c05b3)

Pada simulasi ini, saya mengubah subscriber menjadi lebih lambat dengan menambahkan delay 1 detik setiap kali memproses pesan. Setelah melakukan perubahan ini, saya menjalankan perintah `cargo run` pada program publisher sebanyak 8 kali dengan cepat. Setiap kali publisher dijalankan, pesan-pesan baru dikirimkan ke message broker dan masuk ke queue RabbitMQ. Karena subscriber sekarang memiliki delay, pesan-pesan tersebut tidak dapat langsung diproses dan akan menumpuk di queue.

Setelah menjalankan publisher sebanyak 8 kali, saya memeriksa grafik di RabbitMQ dan menemukan bahwa jumlah queue saya mencapai sekitar 25 pesan. Hal ini terjadi karena publisher terus mengirimkan pesan dengan cepat, sementara subscriber yang lambat hanya dapat memproses satu pesan setiap detiknya, sehingga antrean semakin panjang seiring waktu. Dalam situasi seperti ini, message broker (RabbitMQ) menyimpan pesan-pesan tersebut sementara menunggu subscriber untuk memprosesnya satu per satu.

---

### Running three subscribers

![image](https://github.com/user-attachments/assets/69a00cc5-4c25-4dc6-8ef0-52b2a8de9cc7)
![image](https://github.com/user-attachments/assets/35d4eea0-ce5d-424e-ac11-fe485bf12166)

Dalam simulasi ini, saya menjalankan tiga instance subscriber secara paralel, masing-masing dijalankan di console yang berbeda. Setiap subscriber terhubung ke queue yang sama dan bertugas untuk memproses event secara bersamaan. Saya kembali menjalankan perintah `cargo run` pada publisher sebanyak 8 kali secara cepat untuk mensimulasikan spike permintaan.

Hasilnya, saya mengamati bahwa jumlah queue pesan di RabbitMQ hanya mencapai sekitar 7 hingga 8 pesan saja, jauh lebih sedikit dibandingkan sebelumnya ketika hanya ada satu subscriber (yang mencapai sekitar 25). Hal ini terjadi karena beberapa subscriber dapat memproses pesan secara paralel, sehingga penumpukan pesan pada queue dapat segera dikurangi. Ini mencerminkan salah satu kekuatan dari arsitektur event-driven, yaitu kemampuannya untuk melakukan horizontal scaling pada sisi subscriber untuk menangani beban yang tinggi.

Berdasarkan pengamatan kode, salah satu hal yang dapat ditingkatkan adalah penggunaan thread pool atau asynchronous processing agar proses konsumsi pesan lebih efisien dan tidak dibatasi oleh blocking `thread::sleep`. Selain itu, menambahkan sistem logging yang lebih informatif juga dapat membantu dalam monitoring dan debugging saat sistem dijalankan dalam skala besar.
