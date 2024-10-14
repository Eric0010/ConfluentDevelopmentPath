# Produce dan Consume Message menggunakan Avro

# Data Schema

Data schema adalah yang menjelaskan struktur sebuah data. Data schema bisa digunakan sebagai kontrak pada kedua belah pihak, yakni producer data dan consumer data. Dengan data schema yang diberikan, consumer selalu mengetahui apa yang diharapkan dari sisi producer.

Jika tidak memiliki data schema, maka kedua belah pihak tidak mengetahui arti struktur data yang dikirim dan yang akan di consume id dalam waktu tertentu. Ini dinamakan Schema Evolution.

# Serialisasi dan Deserialisasi

Serialisasi adalah cara untuk merrpresentasikan data di dalam memory didalam bentuk bebrapa seri byte. Serialisasi juga dibutuhkan untuk mentransfer sebuah data lewat koneksi atau menyimpannya pada sebuah penyimpanan.

Deserialisasi adalah sebuah proses mengubah stream beberapa bytes kembali menjadi sebuah objek data

Serialisasi sangat pentik pada Kafka, karena Kafka menggunakan beberapa array byte sebagai formart untuk mengirim dan menyimpan sebuah data pada broker.

Tetapi, Serialisasi dan Deserialisasi bukan merupakan sebuah standart antara operating sistem dan bahasa pemrograman.

Kafka terus mencoba mengatasi masalah konversi data dari byte ke objek data menggunakan kelas - kelas serialisasi, tetapi hal ini di dasarkan pada tipe data yang simple dan mungkin tidak cukup untuk tipe data yang kompleks.

# Avro

![image](https://github.com/user-attachments/assets/df28625d-b62b-4179-b5b7-242676f23db8)

Berikut adalah contoh dari Schema Avro.

Schema Avro biasa dituangkan ke dalam file JSON atau IDL.

Avro mempunyai 3 cara untuk membuat records yaitu:

Generic:
- Menulis schema secara manual
- Mambuat objekGenericRecord secara manual dari schema

Reflection
- Membuat tipe data secara manual
- Menghasilkan schema dari code

Specific
- Menulis schema secara manual
- Menghasilkan kelas - kelas dari schema untuk dimasukan ke sebuah program.

Di dalam Schema Avro di wajibkan untuk memiliki tipe data dari masing - masing pesan yang ingin dihasilkan di dalam bahasa pemrograman dan schema tentunya di dalam format JSON.

Schema Registry juga di perlukan dalam Produce dan Consume menggunakan Avro.

Schema Registry adalah sebuah komponen yang menyediakan pengaturan dan layanan untuk mencari sebuah schema. Schema Registry juga menyediakan interface RESTful untuk menyimpan dan menerima beberapa Schema Avro.

Berikut adalah Flow dari Schema Registry:

![image](https://github.com/user-attachments/assets/f8898a6b-47aa-483e-ae96-e00987328970)

key dan nilai dari pesan sudah di serialisasi secara independen.

Producer melakukan serialisasi data dan menggunakan ID dari Schema.

Consumer akan menggunakan ID Schema untuk deserialisasi data yang diterima.

Schema Registry menjadi komunikasi pada pesan pertama pada schema yang baru.

Berikut command untuk menggunakan Schema Registry:

# Post

![image](https://github.com/user-attachments/assets/c4c957c1-6805-485b-837b-d2a1a77ab1e0)

# Get

![image](https://github.com/user-attachments/assets/08ceb6f8-ce98-4811-beae-2ac006704cb9)

Schema Registry Get berguna untuk integrasi secara terus menerus atau pengiriman pipeline scara terus menerus seperti data pemerintahan.

# Schema Evolution

Schema Avro dapat sewaktu - waktu mengalami evolusi seiring terjadinya perubahan didalam code terjadi.

Schema registry mengharuskan kesesuaian pengaturan subjek ketika ada versi schema baru saja terdaftarkan.

Berikut adalah contoh Producer dan Consumer menggunakan Avro:

# Producer

![image](https://github.com/user-attachments/assets/cdd52998-c671-44f1-b234-369447688ff8)

Di baris ke 5 saya sudah mempunyai kelas serialisasi menggunakan Avro yang akan digunakan untuk melakukan serialisasi pesan.

Di baris ke 6 dan ke 7, saya sudah melakukan konfigurasu untuk menggunakan Schema Registry.

# Consumer

![image](https://github.com/user-attachments/assets/a8d7746d-6320-44c8-af38-cc5d224191ac)

Pada Code Consumer ini tidak ada code yang langsung menggambarkan untuk menggunakan Schema Registry. Interaksi dengan Schema Registry bisa di di komunikasikan dengan Serialisasi Avro dan Deserialisasi Avro.

Contoh Schema di Confluent:

![image](https://github.com/user-attachments/assets/2969d204-7963-42e2-9f5c-fed5e8033180)


