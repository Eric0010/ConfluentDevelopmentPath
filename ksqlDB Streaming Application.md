# ksqlDB Streaming

Mem-proses sebuah aplikasi streaming sebelum ksqlDB

arsitektur yang biasa digunakan di dalam streaming aplikasi biasanya menggunakan connector dan producer dari API untuk masuk ke dalam Kafka, memproses stream tersebut dengan Kafka Stream atau dengan streaming Framework. Lalu, menggunakan connector untuk menyampaikan hasil yang diproses ke database.

ksqlDB ingin menyederhanakan proses arsitektur ini untuk membuat streaming app lebih mudah untuk dibuat.

![image](https://github.com/user-attachments/assets/326944e6-bf0d-4e44-ad1e-5241034007be)

ksqlDB sangat mengurangi kompleksitas operasional yang diperlukan untuk membangun aplikasi pemrosesan stream, yang memungkinkan devloper dan analis membangun sistem real time tanpa memerlukan waktu dan overhead yang signifikan. 
ksqlDB menggabungkan kekuatan pemrosesan stream real time dengan nuansa database yang mudah dipahami, melalui sintaks SQL yang ringan dan familier. Dan karena ksqlDB secara asli didukung oleh Apache Kafka, ksqlDB memanfaatkan platform streaming peristiwa yang mendasarinya dan telah teruji.

ksqlDB adalah proyek sumber terbuka di bawah Lisensi Komunitas Confluent, yang berarti pada dasarnya ia gratis untuk digunakan siapa saja, tetapi tidak seorang pun dapat menjual layanan ksqlDB terkelola mereka sendiri.

Berikut arsitektur dan Komponen dari ksqlDB

![image](https://github.com/user-attachments/assets/10666052-4e6e-4b56-83ca-618be47390bf)

Salah satu konsep kunci adalah bahwa engine proses stream ksqlDB didukung oleh Kafka Streams. Setiap kueri dalam ksqIDB sebenarnya adalah aplikasi Kafka Streams kecil di balik layar.

Berikut adalah ksqlDB Use case:

![image](https://github.com/user-attachments/assets/5134af0d-ec57-4d2e-95e0-648c74ccafde)

Berikut command untuk membuat table di dalam ksqlDB:

![image](https://github.com/user-attachments/assets/55b2298b-0dd3-4bbe-bf1b-7a978737d526)

command diatas memanggil "possibele_fraud" dengan melihat seberapa kali field "card_number" muncul di sebuah stream, stream yang bernama "authorization_attempts" lebih dari 5 detik di hopping window.

jika "card_number" muncul lebih dari 3x di window tersebut, maka tabel ini akan membuat pesan dengan "card number" sebagai key dan terhitung sebagai value di tabel "possible_fraud".

Berikut adalah beberapa Statement dari ksqlDB

![image](https://github.com/user-attachments/assets/1d14da99-3164-4fff-b533-e80fa3d8f5cd)

# Non-Persistent and Persistent Queries

Hasil dari query yang tidak persisten tidak akan di buat di dalam topik Kafka dan hanya akan di print out di dalam console.

Hasil dari query yang persistent akan di buat di dalam kafka Topic.

![image](https://github.com/user-attachments/assets/f4d1a551-91d9-4514-b67c-c4e1c159fdeb)

# Pull dan Push Query

Pull Query

![image](https://github.com/user-attachments/assets/0b2dd109-0ce9-4379-9fe3-dd8e45b2f8b5)

Kelebihan:

- Mencari value dari tabel yang sudah di materialisasi
- respon tunggal akan di kembalikan pada request http

Pull Query adalah query yang dimana client akan menerima sebuah hasil yang bersifat "sekarang". seperti query yang melawab hubungan database yang tradisional. Pull Queries sangat bagus untuk control flow yang bersifat synchronous.

Push Query

![image](https://github.com/user-attachments/assets/6465a80b-fb3d-4553-b91e-846e380d1ff1)

Kelbihan:

- tersubscribe otomatis ke dalam stream.
- ditunjukan di dalam EMIT CHANGES
- response http yang terpisah - pisah dari sebuah panjang yang tidak terbatas.

Push Query adalah query yang dimana client akan melihat sebuah hasil pada saat query berubah di masa realtime. perubahan akan di dorong di bawah long-lived connection. Push queries juga berguna untuk membangun sebuah microservices yang sudah diprogram dan terkontrol, aplikasi realtime atau control flow yang berisifat asynchronous.

Berikut adalah cara membuat Stream dan tabel dari kafka Topic:

![image](https://github.com/user-attachments/assets/a52c60ec-ce4e-4913-aea1-3f336ddd640a)



