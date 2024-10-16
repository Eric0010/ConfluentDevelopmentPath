# Setup ksqlDB Dengan Platform Confluent

ksqlDB adalah basis data yang dibuat khusus yang dirancang untuk memfasilitasi pengembangan aplikasi pemrosesan aliran. Basis data ini menyatukan komponen standar yang dibutuhkan untuk menangani tugas-tugas umum yang terlibat dalam pemrosesan aliran, hanya memerlukan satu ketergantungan Apache Kafka. Selain itu, ksqlDB dirancang agar mudah dipahami oleh mereka yang sudah mengenal SQL, sehingga memberikan kurva pembelajaran yang lebih mudah untuk memulai dan menjalankannya dengan cepat.

Berikut langkah - langkah setup ksqlDB

luncurkan portal lab dan hubungkan ke mesin virtual.

Setelah terhubung, lingkungan Docker Compose akan otomatis diluncurkan. Periksa apakah layanan sudah aktif.

```
cd ksqldb-course
docker-compose ps
```

Buka http://localhost:9021 di browser untuk meluncurkan Pusat Kontrol Confluent.

Setelah Pusat Kontrol diluncurkan, Anda akan melihat halaman Beranda:

![image](https://github.com/user-attachments/assets/b55c1cd3-a2d6-4b75-99cc-4a900a8f1460)

Dari sini, klik kotak klaster produksi.

Selanjutnya, klik tab ksql di sebelah kiri jendela.

![image](https://github.com/user-attachments/assets/e8d87836-3d79-4aaf-b036-ab369a3b1de7)

Di Pusat Kontrol, klik aplikasi ksqlDB yang disebut ksqldb-class.

![image](https://github.com/user-attachments/assets/37e6cd3f-de68-4e06-9071-f8d8f53c6162)

Selanjutnya, mari buat aliran bernama test yang membuat topik bernama "ksqldb.test" dengan 1 partisi (Anda akan mempelajari lebih lanjut tentang topik dan partisi di modul berikutnya). Pastikan tab editor dipilih, lalu masukkan kueri dan klik Jalankan kueri. Jangan ragu untuk memasukkan lebih banyak rekaman ke dalam aliran.

Perhatikan bahwa saat Anda mengetik perintah secara manual (bukan menyalin/menempel), ada fitur pelengkapan otomatis untuk membantu Anda dengan perintah.

```
CREATE STREAM test (
name STRING KEY,
favorite_food STRING
) WITH (
KAFKA_TOPIC='ksqldb.test',
VALUE_FORMAT='DELIMITED',
PARTITIONS=1
);
INSERT INTO test (name, favorite_food) VALUES ('chuck', 'burritos');
```
Anda akan melihat output di jendela di bawah ini yang tampak seperti ini:

![image](https://github.com/user-attachments/assets/65cfcc79-2bd2-4f0c-a9ef-74bbaf4c8c10)
