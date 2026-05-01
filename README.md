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