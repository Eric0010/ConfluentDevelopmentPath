# Menentukan Offset Saat Konsumen Memulai

Pertama, kita akan meninjau metode default yang digunakan konsumen untuk menentukan offset awalnya.

Properti konsumen auto.offset.reset menentukan apa yang harus dilakukan jika tidak ada posisi yang dikomit (yang akan terjadi saat grup pertama kali diinisialisasi) atau saat offset berada di luar rentang, yaitu, kurang dari offset awal log saat ini atau lebih besar dari offset akhir log.

![image](https://github.com/user-attachments/assets/dfb83c4f-afc1-4ef0-86d5-e48e8d26341a)

Berikut adalah contoh commandnya:

```
kafka-consumer-groups \
    --bootstrap-server kafka-1:9092 \
    --group my-group \
    --topic my_topic \
    --reset-offsets \
    --execute \
    --to-datetime 2021-06-01T17:14:23.933
```
Berikut contoh perulangan offset ke stempel waktu tertentu:

![image](https://github.com/user-attachments/assets/b93d0d11-586b-4525-9460-763a05bdf9ac)

Dalam contoh ini, konsumen akan mencari offset yang sesuai dengan MY_TIMESTAMP untuk setiap partisi topik my_topic.

Berikut contoh perulangan Ulang Offset Saat Ini ke Awal:

![image](https://github.com/user-attachments/assets/d4b96428-d795-4db3-a9e0-2f0f039f6958)

Dalam contoh ini, konsumen akan mencariToBeginning dari semua partisi topik my_topic
