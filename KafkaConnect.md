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
