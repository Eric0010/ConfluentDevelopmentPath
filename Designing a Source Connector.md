# Design Source Connector

Kafka Connect adalah kerangka kerja untuk integrasi data aliran skala besar dan waktu nyata menggunakan Kafka. Kita dapat meninjau komponen klaster Kafka Connect:

Konektor/Tugas: konektor sumber dan penerima membuat tugas untuk melakukan pekerjaan menyalin dari (sumber) dan menulis ke (penerima) sistem di luar Kafka.

Konverter: Tugas sumber mengeluarkan format standar untuk data yang ditujukan untuk Kafka. Dengan menggunakan format standar, Anda mengonfigurasi pilihan konverter untuk mengonversi (atau membuat serial) ke format yang akan ditulis ke Kafka. Di sisi lain, tugas penerima dikonfigurasi dengan konverter untuk mendeserialisasi data Kafka sebelum menyalin data ke tujuan.

workers: Kafka Connect dapat beroperasi dalam mode mandiri atau terdistribusi. Pekerja menghosting konektor dan tugas yang dikonfigurasi. Setiap kegagalan dalam klaster terdistribusi dideteksi dan tugas serta konektor diseimbangkan kembali untuk toleransi kesalahan.

![image](https://github.com/user-attachments/assets/b303c8a2-20c5-4e45-acee-c658bd6ac131)

Konektor Kafka yang telah dibuat sebelumnya
Banyak konektor yang telah dibuat sebelumnya sudah ada. Banyak yang didukung oleh Confluent atau mitra, dan banyak yang didukung komunitas. Sebelum memulai konektor Anda sendiri, periksa apakah sumber data Anda sudah memiliki konektor. Banyak konektor yang akan Anda temukan di repositori seperti GitHub adalah sumber yang bagus untuk dipelajari. Contoh konektor sumber:

[FileStreamSource](https://docs.confluent.io/current/connect/filestream_connector.html#filesource-connector) - menyalin dari file ke Kafka

[JdbcSourceConnector](https://docs.confluent.io/current/connect/kafka-connect-jdbc/source-connector/index.html) - menyalin perubahan dalam database JDBC ke Kafka

[ActiveMQSourceConnector](https://docs.confluent.io/current/connect/kafka-connect-activemq/index.html) - menyalin dari kluster ActiveMQ ke Kafka

Konektor dan Tasks

Konektor menerima konfigurasi untuk mengakses data sumber. Di dalam konektor, ini dipecah menjadi konfigurasi task. Ini membuat konfigurasi untuk task yang dapat dijalankan secara paralel pada connect workers. Misalnya, JdbcSourceConnector dapat menyalin data dari beberapa tabel. Konektor ditulis untuk menetapkan tabel ke beberapa task. Sebaliknya, FileStreamSourceConnector selalu membuat satu task karena satu file hanya akan digunakan oleh satu task.

![image](https://github.com/user-attachments/assets/0e197ddd-d2b5-472a-b92a-d0fe7fae7acc)

Mempartisi Data Source

Dengan menggunakan contoh yang kami gunakan di atas: JdbcSourceConnector dapat memecah pekerjaan saat menyalin aliran data dari beberapa tabel. Karena ini merupakan pola yang umum dalam konektor, kita dapat menggunakan metode ConnectorUtils#groupPartitions untuk menetapkan grup partisi ke task.

```
int maxTasks = 3;
List<String> tableNames = Arrays.asList("departments", "employees", "dept_emp", "dept_manager", "titles", "salaries");
List<List<String>> groups = ConnectorUtils.groupPartitions(tableNames, maxTasks);
System.out.println(groups);
```

Mengembalikan

[[departments, employees], [dept_emp, dept_manager], [titles, salary]]

Manajemen Offset

Saat Anda membuat konektor sumber, API org.apache.kafka.connect.source menyediakan mekanisme umum untuk menyimpan posisi konektor dalam aliran. Serta API untuk mengambil posisi mulai ulang jika terjadi penyeimbangan ulang.

![image](https://github.com/user-attachments/assets/e96fd84b-d4b3-4cab-a017-b2614484e25a)
