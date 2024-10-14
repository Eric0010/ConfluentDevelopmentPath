# Produce Message to Apache Kafka

Berikut adalah cara - cara untuk membuat Producer untuk Apache Kafka:

Pertama, saya harus membuat properties seperti pada di gambar:

![image](https://github.com/user-attachments/assets/b1479c20-b56c-48ec-85d3-503e53cc77bc)

Ketika produce sebuah message, sangat penting untuk memisahkan konfigurasi dari aplikasi logic supaya logic yang sama bisa di run di berbagai environment di berbagai properties. Hal yang sama sudah terjadi di file .properties. Client kafka Java juga sama.

file .properties adalah sebuah file teks dimana properties diartikan dalam symbol '='. Seperti contoh berikut ini ```bootstrap.servers=broker-1:9092,broker-2:9092

Kedua, saya akan mengirim pesan dengan coding berikut ini:

![image](https://github.com/user-attachments/assets/1aad4fdf-2c38-465e-9a4e-200e4352adde)

Konstruktor ProducerRecord melakukan enkapsulasi key yang diesiakan dan nilai ke dalam message lengkap dengan header.

Ketika send() dipanggil, Kafka Producer melakukan serialisasi pada key dan melakukan run pada Partioner default, hal ini dilakukan untuk menentukan partisi mana yang dituju untuk menyimpan sebuah pesan pada topik menggunakan data yang sudah disediakan pada ProducerRecord.

Callbacks()

![image](https://github.com/user-attachments/assets/5dc35ade-80d2-44f5-9ddf-f80117788c7b)

Callbacks digunakan jika send() yang dijalankan mengalami kegagalan. Terdapat sebuah Klosur. Di dalam klosur, informasi yang tersedia bisa digunakan untuk melakukan callback yang sudah di inisiasi untuk record. Klosur adalah sebuah fungsi yang bisa di kirim sebagai sebuah objek dan memberikan sebuah akses pada beberapa variabel sebelumnya atau memberikan secara langsung pada konstruktor callback jika fungsi lambda tidak digunakan.

producer.close()

fungsi ini berguna untuk melakukan block sampai semua request sebelumnya sudah selesai.

