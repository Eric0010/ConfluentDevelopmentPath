# Consume Message

Sama seperti dengan Producer, untuk Consumer saya juga harus membuat sebuah properties:

![image](https://github.com/user-attachments/assets/02f10b28-887a-488e-a5c7-f2b74e9c5752)

ByteArraySerializer, IntegerSerializer, LongSerializer dan fungsi Serializer yang lainnya sudah termasuk di dalam client.

StringSerializer melakukan enkoding default ke UTF8. hal ini bisa diubah dengan mengatur property pada serializer.encoding

Di dalam class KafkaProducer, Kafka COnsumer di konfigurasi dengan objek di dalam Properties yang men-support class - class helper.

Maka coding diatas bisa dijadikan seperti pada gambar:

![image](https://github.com/user-attachments/assets/5ffc20c8-5c19-4df3-9278-f5c3760fd8b9)

Consume message dengan Kafka Consumer:

![image](https://github.com/user-attachments/assets/ce5b8622-63f3-4b09-b208-b6617379ad1f)

Metode poll(Duration) memiliki banyak fungsi, tetapi yang paling utama perlu diketahui adalah:

- Memberikan record dari partisi yang sudah didaftarkan.
- men-trigger tugas partisi jika dibutuhkan
- memberikan offset konsumer jika automatic offsets commit dinyalakan dan fungsi auto.commit.interval.ms sudah melebihi batas.

CLosing the Customer:

![image](https://github.com/user-attachments/assets/54afa5e8-789e-48cd-bf96-10740da53366)

close() berguna untuk menunggu timeout selama 30 detik untuk cleanup yang dibutuhkan. Jika auto-commit diaktifkan, hal ini bisa memunculkan offset yang sedang di consume di dalam waktu default timeout.
