# Kafka Streams

Kafka Streams adalah sebuah library klien Java untuk membangun sebuah aplikasi dan microservice, dimana data input dan output data di simpan di dalam Kafka cluster.

![image](https://github.com/user-attachments/assets/5d8c8025-ed03-4d1a-8a5e-a2e451564e42)

Berikut adalah contoh code KStream.

Code tersebut menggunakan objek builder untuk membuat KStream dari topik pageviews-topik

Mengklompokan KStream dari key yang dimiliki pada record key

Memberikan sesi Timeout window di 5 menit

Menghitung record dari key pada setiap sesi window

Berikut adalah Definisi Stream.

![image](https://github.com/user-attachments/assets/a96194dc-934b-4414-be8d-d9cf7c19fb24)

Berikut Ilustrasi dari Stream

![image](https://github.com/user-attachments/assets/05050c3f-3250-4101-8a9a-4a0585956710)

Sesuai dengan tabel diatas tabel di sebelah kiri menunjukan bahwa KStream bisa merekam semua kejadian pada perubahan masing - masing Account.

Sedangkan, tabel di sebelah kanan yang mewakili Table hanya merekam hasil terakhir pada Account.

# Stream Processor Topology

![image](https://github.com/user-attachments/assets/ab26853c-9e07-46ec-800b-75244127dd6c)

Source Processor: Sebuah tipe spesial stream processor yang tidak memiliki upstream processor. Hanya mem-produce input stream kepada topologi dari 1 atau beberapa topik Kafka dengan cara melakukan consume record dari beberapa topik dan mengirimnya kepada downstream porcessornya.

Sink Processor: Sebuah tipe spesial stream processor yang tidak memiliki downstream processor. Hanya mengirim record yang disimpan dari upstream processon kepada topik Kafka yang dituju.

Berikut adalah beberapa fungsi - fungsi yang ada pada KStream:

![image](https://github.com/user-attachments/assets/10ab0af4-fa98-44ff-a238-c8207429d7e8)

Berikut adalah Tipe - tipe WIndow:

![image](https://github.com/user-attachments/assets/b3ebe3af-0134-467a-8c81-513e1957f47e)

Berikut adalah Contoh Simple Kafka Stream (Konfigurasi dan Topology):

# Konfigurasi

![image](https://github.com/user-attachments/assets/8e1530bc-02e9-4cb2-a11f-b9102f3213cc)

Baris 8 menunjukan Konfigurasi khusus untuk aplikasi Kafka Stream Consumer. Hal ini bisa membuat anda untuk merubah pengaturan producer di dalam aplikasi Kafka Stream.

Baris 10 dan 12 menunjukan key dan value dari SerDes. Di dalam contoh ini, tipe data yang ditunjukan akan sama untuk pesan inbound dan pesan outbound, supaya SerDes tidak akan di ganti ulang oleh code.

# Topology

![image](https://github.com/user-attachments/assets/18e0ec88-441f-49b0-8c63-2b27ab07cdbe)

Di dalam contoh ini, objek textLines KStream telah ditransform oleh mapValues untuk diubah menjadi semua karakter di dalam pesan untuk di uppercase, yang diartikan objek KStream bernama uppercasedWithMapValues.

Hal ini menjadikan KStream di produksi di dalam topik UppercasedTextLinesTopic.


