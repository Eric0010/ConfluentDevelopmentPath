# Membuat Source Connector

Untuk lebih memahami kelas yang digunakan untuk membangun konektor sumber, Anda akan menerapkan konektor sumber yang dibuat dengan sangat sederhana.

IncrementSourceConnector mengambil daftar penambahan untuk menghasilkan angka baru dalam urutan penambahan lalu tidur selama 1 detik. Konektor membuat rekaman terstruktur dengan nilai urutan, dan informasi diagnostik threadId dan nama host. Nilai diagnostik digunakan untuk menggambarkan penyeimbangan ulang.

Konektor menyalin nilai urutan ke topik dengan nama awalan dan nilai penambahan, misalnya seq_1.

Saat mengerjakan konektor sederhana ini, pertimbangkan bagaimana konsep yang Anda pelajari dapat diterapkan ke sumber data yang saat ini Anda gunakan. Memahami konsep ini juga membantu Anda lebih memahami bagaimana konektor yang ada melakukan tugasnya.

Implementasi

Untuk menulis konektor sumber, kami mengimplementasikan kelas yang memperluas [SourceConnector](https://docs.confluent.io/current/connect/javadocs/org/apache/kafka/connect/source/SourceConnector.html) dan [SourceTask](https://docs.confluent.io/current/connect/javadocs/org/apache/kafka/connect/source/SourceTask.html).

Data sumber yang Anda salin juga dapat berubah seiring waktu. Misalnya, JDBCSourceConnector mengonfigurasi ulang tugas jika tabel baru muncul di database. Jika ini merupakan persyaratan untuk konektor sumber Anda, baca tentang properti konteks [ConnectorContext](https://docs.confluent.io/current/connect/javadocs/org/apache/kafka/connect/connector/ConnectorContext.html) dari SourceConnector.

SourceConnector

Kode konektor Anda berada dalam kelas yang berasal dari SourceConnector. Metode abstrak yang penting dijelaskan di bawah ini.

void start(Map<String, String> props) kode konektor Anda menerima konfigurasi dalam metode start.

List<Map<String, String>> taskConfigs(int maxTasks) kode Anda mengembalikan daftar pengaturan konfigurasi tugas hingga nomor maxTasks. Di sinilah Anda memutuskan cara mempartisi penyalinan data.

SourceTask

Tugas Anda yang melakukan penyalinan data berasal dari SourceTask.

void start(Map<String, String> props) kode tugas Anda menerima konfigurasi yang dibuat dalam SourceConnector#taskConfigs dalam metode start.

List<SourceRecord> poll() Connect "memeriksa" tugas Anda untuk data baru.

Copying Data
Di dalam metode poll() kode Anda mengembalikan daftar kelas SourceRecord yang diisi dengan data yang disalin dari sumber Anda. Kelas SourceRecord juga menetapkan pengimbang aliran pemrosesan Anda.

```
records.add(
new SourceRecord(offsetKey(increment), // sourcePartition
offsetValue(++offset), // sourceOffset
topic, // topic
null, // partition
VALUE_SCHEMA, // valueSchema
struct // value
)
);
```

Skema dan Struct

Output konektor Anda dapat berupa tipe standar Java, atau struktur data dari paket org.apache.kafka.connect.data. Jika data Anda berupa rekaman terstruktur, Anda dapat mengisi Struct dan menyediakan skema.

```
Skema schema = SchemaBuilder.struct().name("com.example.Person")
.field("name", Schema.STRING_SCHEMA).field("age", Schema.INT32_SCHEMA).build();
Struct struct = new Struct(schema).put("name", "Bobby McGee").put("age", 21);
```

Serialisasi output Anda ditangani oleh konverter yang dikonfigurasi. Dalam aktivitas berikut, Anda akan mengonfigurasi AvroConverter. AvroConverter mendaftarkan skema Anda dengan Confluent Schema Registry dan digunakan untuk menulis byte serial Avro ke topik Kafka Anda.

Periksa Source dan Konfigurasi

Pindah ke direktori IncrementSourceConnector dan buka proyek di Visual Studio Code:

```
$ cd IncrementSourceConnector
$ code .
```

Periksa konfigurasi dan kode source.

Pengaturan connect-avro-standalone.properties untuk pekerja koneksi mandiri. Perhatikan pengaturan plugin.path=extensions ini adalah jalur relatif ke lokasi tempat pekerja koneksi mandiri akan mencari plugin.

Pengaturan connector1.properties untuk konektor yang dijalankan di pekerja.

Implementasi konektor src/main/java/com/example/IncrementSourceConnector.java.

Implementasi tugas src/main/java/com/example/IncrementSourceTask.java.

Biasakan diri Anda dengan implementasi IncrementSourceConnector dan IncrementSourceTask. Temukan semua pernyataan log.info. Anda akan melihat output dari pencatatan log saat menjalankan konektor.

Bangun Konektor JAR

Dari jendela terminal, buat JAR:
```
$ ./gradlew shadowJar
```

Jika Anda sekarang melihat di direktori ekstensi, Anda akan melihat berkas connect-increment-0.0.1.jar yang berisi implementasi konektor sumber.

Jalankan Kafka Connect Standalone
Di sini, kami menggunakan perintah docker compose untuk mengaktifkan kluster pengembangan Kafka. Konektor telah dikonfigurasi dengan AvroConverter dan akan memerlukan Schema Registry untuk menyimpan skema. Dari jendela terminal, jalankan kluster Kafka:
```
$ docker-compose up -d zookeeper kafka schema-registry
```

Mungkin perlu beberapa detik agar Schema Registry tersedia. Pertama, periksa apakah REST API Schema Registry sudah siap. Terus coba perintah curl hingga Anda melihat respons {"mode":"READWRITE"} yang berhasil.
```
$ curl localhost:8081/mode
```
Jalankan pekerja koneksi mandiri:
```
$ connect-standalone connect-avro-standalone.properties connector1.properties
```

Skrip connect-standalone akan menampilkan banyak informasi yang akan bergulir keluar layar.

IncrementSourceConnector.java mencatat konfigurasi setiap tugas dengan baris ini:

```
log.info("Konfigurasi tugas: (awalan: {}, increments {})", topicPrefix, String.join(",", taskIncrements));
```

Dapatkah Anda menemukan keluaran pencatatan ini di keluaran connect-standalone? Berapa banyak konfigurasi tugas yang dibuat?

Lihat Data Topik
Dari jendela terminal baru, gunakan kafka-avro-console-consumer untuk menggunakan semua topik yang dimulai dengan seq_. Biarkan konsumen ini berjalan hingga Anda diperintahkan untuk mengakhirinya.

```
$ kafka-avro-console-consumer --bootstrap-server localhost:9092 --whitelist 'seq_.+'
{"threadId":37,"hostname":"training","value":3}
{"threadId":37,"hostname":"training","value":300}
{"threadId":37,"hostname":"training","value":4}
{"threadId":37,"hostname":"training","value":400}
```
Perhatikan bahwa semua pesan ditulis dari threadId yang sama. Akhiri proses connect-standalone dengan CTRL-c.

Melakukan Komitmen Offset
Pada langkah sebelumnya, Anda mengakhiri pekerja connect-standalone Anda. Perhatikan pesan yang berisi Stopping task di log keluaran, diikuti dengan pesan Committing offsets. Selama proses shutdown, pekerja connect melakukan komitmen pada posisi yang terakhir disimpan oleh tugas.

Di mana offset untuk konsumsi Anda disimpan? File konfigurasi connect-avro-standalone.properties mengonfigurasi pekerja mandiri untuk menyimpan offset dalam file di /tmp/connect.offsets. File tersebut berformat biner, tetapi Anda dapat mengenali nilai partisi dan offset yang digunakan untuk membuat SourceRecord di IncrementSourceTask.

Di jendela terminal, gunakan string untuk menemukan offset yang disimpan oleh IncrementSourceTask di /tmp/connect.offsets:

```
$ strings /tmp/connect.offsets
java.util.HashMap
loadFactorI
thresholdxp?@
-["kafka-connect-increment",{"increment":100}]uq
{"position":17}uq
+["kafka-connect-increment",{"increment":1}]uq
{"position":17}x
```

Jalankan kembali pekerja connect-standalone

```
$ connect-standalone connect-avro-standalone.properties connector1.properties
```

Anda akan melihat output dari pencatatan IncrementSourceTask yang berisi pesan Kami menemukan offset untuk increment…​. Tugas konektor telah berhasil dilanjutkan dari offset. Kembali ke konsumen di jendela terminal kedua. Amati bahwa urutan pesan telah berlanjut dari tempat pesan sebelumnya berakhir.

Pembersihan

Akhiri proses connect-standalone dengan CTRL-c. Akhiri proses kafka-avro-console-consumer dengan CTRL-c.

Matikan kluster Kafka Anda dengan perintah docker-compose down.
