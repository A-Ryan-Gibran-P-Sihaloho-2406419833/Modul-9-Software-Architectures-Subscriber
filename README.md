## Memahami Subscriber dan Message Broker

**a. Apa itu AMQP?**
AMQP (Advanced Message Queuing Protocol) adalah protokol lapisan aplikasi berstandar terbuka yang dirancang untuk *middleware* 
berorientasi pesan. Protokol ini menyediakan berbagai fitur seperti orientasi pesan, antrean (*queuing*), routing 
(termasuk *point-to-point* dan *publish-and-subscribe*), keandalan, dan keamanan, yang memungkinkan berbagai sistem 
terdistribusi untuk berkomunikasi satu sama lain dengan lancar.

**b. Apa maksudnya? `guest:guest@localhost:5672`**
Ini adalah *connection string* (URI) yang digunakan untuk terhubung ke *message broker* RabbitMQ:
- Kata **`guest`** yang pertama adalah **username** bawaan (*default*) yang digunakan untuk autentikasi dengan server RabbitMQ.
- Kata **`guest`** yang kedua adalah **password** bawaan yang terhubung dengan *username* tersebut.
- **`localhost:5672`** menentukan *host* dan *port*. Ini berarti server RabbitMQ sedang berjalan di mesin lokal komputer 
(`localhost`) dan listening koneksi AMQP yang masuk pada *port* bawaannya yaitu `5672`.


## Simulation Slow Subscriber

![Slow Subscriber Queue](img_1.png)

**Mengapa total antrean (Queued messages) bisa memuncak?**
Total antrean melonjak tajam (misalnya mencapai 20 atau lebih) karena kita menyimulasikan *slow subscriber* dengan menambahkan *delay* (`thread::sleep`) selama 1 detik untuk setiap pesan yang diproses. Sementara itu, *publisher* mengirimkan puluhan pesan secara instan. Karena kecepatan *publisher* mengirim data jauh melebihi kecepatan *subscriber* memproses data, *message broker* (RabbitMQ) menampung pesan-pesan tersebut di dalam antrean (Queue) sambil menunggu giliran untuk dikonsumsi oleh *subscriber*.


## Reflection and Running at least three subscribers

![Multiple Subscribers Queue](img.png)

**Refleksi: Mengapa antrean (*spike*) menurun lebih cepat?**
Penurunan grafik antrean yang sangat cepat terjadi karena beban kerja tidak lagi ditanggung oleh satu program saja. Dengan menjalankan tiga program *subscriber* secara bersamaan (paralel) yang terhubung ke antrean yang sama, RabbitMQ mendistribusikan pesan-pesan tersebut secara *Round-Robin* ke ketiga *worker*. Artinya, kecepatan pemrosesan data (konsumsi) meningkat tiga kali lipat, sehingga penumpukan pesan di dalam *queue* bisa diurai dengan sangat efisien.

**Adakah hal dari kode Publisher dan Subscriber yang bisa di-improve?**
1. **Penggunaan Asynchronous Sleep:** Pada *subscriber*, penggunaan `thread::sleep` akan memblokir (*block*) seluruh *thread*, yang mana kurang optimal dalam ekosistem `tokio` (asynchronous). Sebaiknya menggunakan `tokio::time::sleep` agar *thread* tidak terblokir dan bisa menangani *task* lain.
2. **Error Handling yang Lebih Baik:** Penggunaan `.unwrap()` pada koneksi `CrosstownBus::new_queue_listener` sangat berisiko membuat program langsung *panic* (crash) jika koneksi ke RabbitMQ terputus. Sebaiknya menggunakan penanganan *error* yang lebih baik seperti `match` atau `?`.
3. **Hardcoded URL:** URL koneksi RabbitMQ (`amqp://guest:guest@localhost:5672`) ditulis langsung di dalam kode. Untuk praktik terbaik (*best practice*), URL ini seharusnya dipisahkan ke dalam variabel *environment* (`.env`) agar lebih fleksibel saat program di-deploy ke produksi.