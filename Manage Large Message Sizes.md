# Ukuran Pesan Maksimum Default Kafka

Kafka tidak dioptimalkan untuk pesan besar. Saat Broker menerima kumpulan rekaman besar, buffer byte dialokasikan untuk menerima seluruh kumpulan rekaman. Semakin besar kumpulan rekaman, semakin besar kemungkinan masalah kinerja akan terjadi (misalnya, fragmentasi di tumpukan Java). Ukuran maksimum default adalah 1 MB, dan sangat disarankan untuk tidak mengubah default ini. Sebagian besar kasus penggunaan dapat dicapai dengan ukuran kumpulan rekaman kurang dari 1 MB.

Menambah batas ukuran pesan dapat menyebabkan:

Kinerja pengumpulan sampah JVM broker yang buruk

Lebih sedikit memori yang tersedia untuk bisnis broker penting lainnya

Lebih banyak sumber daya yang dibutuhkan untuk menangani permintaan

Permintaan pengambilan konsumen dan replika yang tidak efisien

Jika Anda harus mengubah batas ukuran untuk pesan Kafka, Anda melakukannya dengan memodifikasi pengaturan konfigurasi berikut:

![image](https://github.com/user-attachments/assets/e60aa3fa-29f5-4417-a744-f718d5dfa63f)

# Kompresi Pesan

![image](https://github.com/user-attachments/assets/93a1ad12-ee8a-44d1-ab03-a6661d5c9c27)

Mengaktifkan kompresi pada produsen akan mengurangi ukuran kumpulan rekaman yang menghasilkan penurunan bandwidth jaringan dan persyaratan penyimpanan.

KafkaProducer mengompresi pesan dan menambahkannya ke kumpulan rekaman

Kumpulan rekaman terkompresi dikirim ke broker

Broker menyimpan kumpulan rekaman terkompresi

Rekaman terkompresi dikirim ke konsumen

KafkaConsumer mendekompresi rekaman

Untuk mengaktifkan kompresi, ubah properti compression.type produsen dari none menjadi gzip, snappy, lz4, atau zstd.

Kompresi berlaku untuk kumpulan data penuh, jadi kemanjuran pengelompokan juga akan memengaruhi rasio kompresi (semakin banyak pengelompokan berarti kompresi yang lebih baik).

Jenis kompresi yang digunakan oleh produsen dicatat di header kumpulan rekaman.

Konsumen mendekompresi kumpulan pesan sesuai dengan jenis kompresi yang ditunjukkan di header.

# Lewati Referensi ke Penyimpanan Eksternal

![image](https://github.com/user-attachments/assets/9e6572cf-cc7e-4321-bf78-3fefcc53e04c)

Jika kompresi saja tidak cukup, maka strategi lain adalah menyimpan pesan besar dalam penyimpanan objek seperti S3, basis data dokumen seperti MongoDB, atau cache seperti Redis, lalu menghasilkan referensi ke lokasi sebenarnya data tersebut ke Kafka.

Konsumen akan membaca referensi tersebut dan mengambil data dari sistem eksternal.

Pola desain ini terkadang disebut "pemeriksaan klaim", karena berperilaku seperti klaim bagasi di bandara.

# Record Chunking

![image](https://github.com/user-attachments/assets/a5cef8ed-4232-42c2-9ae8-5908f933af74)

Jika kompresi tidak cukup dan penyampaian referensi tidak mencukupi, pertimbangkan untuk memecah muatan besar menjadi "potongan" dan mengirim potongan tersebut sebagai pesan individual.

Pada tingkat tinggi, ini akan mencakup:

Mengimplementasikan semacam kelas MessageChunker untuk memecah muatan besar menjadi daftar rekaman produsen Kafka

Mengirim rekaman ke Kafka secara atomik dan berurutan menggunakan produsen idempoten yang memanfaatkan API transaksional produsen

Mengonsumsi rekaman dan menyusunnya menjadi muatan asli setelah semua potongan tersedia
