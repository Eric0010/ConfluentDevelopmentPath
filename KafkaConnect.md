# Kafka Connect

Kafka Connect adalah sebuah framework yang digunakan untuk streaming data antara Apache Kafka dengan sistem data yang lain.

Kafka connect bersifat open source dan merupakan bagian dari distribusi Apache Kafka sendiri.

Kafka connect = mudah, terjangkau, dapat diandalkan.

![image](https://github.com/user-attachments/assets/4b93f810-876a-4a6e-ba12-a31df333ec7f)

Komponen dari Kafka Connect adalah connector, connector merupakan sebuah pekerja yang melakukan penyalinan data antara Kafka dengan system lainnya. Connector di bagi menjadi 2 yaitu:

- Source Connector: melakukan pembacaan data dari sistem data eksternal ke Kafka
    Secara Internal, Source Connector menggunakan Kafka Producer
- Sink Connector: menulis data dari Kafka ke sistem data eksternal.
    Secara Internal, Sink Connector menggunakan Kafka Consumer

Contoh UseCase Kafka Connect:

![image](https://github.com/user-attachments/assets/be103c8b-2ba7-44cf-a4c5-ecce9a86a463)

Melakukan Stream data secara keseluruhan dari database SQL ke Kafka, ada 2 mode:
- Bulk: Memuat data di dalam tabel secara keseluruhan.
- Change data capture: Memuat data di dalam tabel hanya jika terjadi perubahan.

Melakukan Stream data dari Kafka Topics ke Hadoop File System untuk memproses batch.

Di dalam Confluent hub, anda juga bisa mencari lebih dari 100 connector.

![image](https://github.com/user-attachments/assets/483c9188-ac2f-4cf3-81ac-d97887ec51aa)

Cara melakukan Instalasi Connector:

```
confluent-hub install debezium/sebezium-connector-mysql: latest
```
Selanjutnya setelah melakukan penginstallan, anda juga harus menemukan dan menyalin alamat file connector Plugin yang ingin dipakai ke file worker Kafka connect

![image](https://github.com/user-attachments/assets/66bfa32d-fe41-49af-8a0c-f9a2533bdde6)

Keunggulan dari Kafka Connect:

Fleksibel dan terjangkau

- untuk menambahkan beberapa resource ke dalam Connect, hanya perlu menambahkan beberapa worker saja.
- beberapa worker connector bisa menemukan satu sama lain dan memuatnya secara otomatis sehingga pekerjaan yang dilakukan seimbang.
- memperbolehkan menjalankan beberapa connector dan tugas sekaligus di waktu yang bersamaan.

Dapat diandalkan dan men-toleransi kesalahan

- Jika salah satu worker gagal, maka semua tugas dan connector yang ada di dalamnya secara otomatis me restart di worker yang lain.
- Jika salah satu tugas dan connector gagal, maka seorang user bisa memutuskan solusi yang diinginkan.

# Konfigurasi connector Kafka Connect

![image](https://github.com/user-attachments/assets/d7b2fce7-6223-47f6-b1fd-feb74c43757a)

Beberapa Connector juga bisa di konfigurasi di Confluent Connect Center (Source dan Sink).

![image](https://github.com/user-attachments/assets/ece7f9a6-57d9-4b20-abb2-8b4f38d25590)

Berikut adalah contoh file connector yang berbentuk JSON file.

connector bisa ditambahkan, diubah dan dihapus via REST API di dalam port 8083.

Anda bisa menjalankan Kafka connect di dalam 2 mode:

# Mode Distributed

- konfigurasi bisa dijalankan via REST API.
- Perubahan yang terjadi akan dipertahankan ketika proses sebuah worker di restart.
- Konfigurasi data connector di simpan di dalam Kafka topic yang bersifat spesial.
- Permintaan REST bisa dibuat di worker mana pun.

# Mode Standalone

- konfigurasi juga bisa dijalankan via REST API.
      Untuk Standalone, biasanya konfigurasi dilakukan via file properties.
- Perubahan yang terjadi via REST API di mode Standalone tidak akan dipertahankan ketika proses sebuah worker di restart.
- C3 memanfaatkan REST API untuk membiarkan seorang user meng-konfigurasi dan mengatur connector lewat GUI.
  
