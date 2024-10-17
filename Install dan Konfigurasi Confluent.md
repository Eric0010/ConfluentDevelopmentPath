# Install Confluent

![image](https://github.com/user-attachments/assets/f2b9029e-756e-4574-a1a0-88525ec80845)

Confluent Platform melengkapi Apache Kafka dengan kemampuan canggih yang dirancang untuk memungkinkan kasus penggunaan waktu nyata, menyederhanakan operasi perusahaan dalam skala besar, dan memenuhi persyaratan keamanan yang ketat.

Berikut adalah cara yang paling mudah untuk menginstall Confluent:

```
curl -O $confluent_url tar xzf confluent-$version
```
atau

```
sudo apt-get install confluent-platform
```
Menginstal Confluent Platform melalui arsip TAR mempercepat penyiapan lingkungan pengembangan lokal.

Pengelola paket seperti apt dan yum adalah cara paling andal untuk menginstal server kelas produksi. Pengelola paket memungkinkan pembaruan yang lebih mudah dan secara otomatis menangani banyak konfigurasi sistem seperti menyiapkan layanan systemd dan meningkatkan batas penanganan file yang terbuka.

![image](https://github.com/user-attachments/assets/b006cb0b-a7bf-4b57-80a3-faeeb2d89be2)

Namun, yang lebih penting daripada mengikuti langkah-langkah instalasi yang mudah adalah mempelajari apa saja yang termasuk dalam instalasi.

Komponen yang termasuk dalam Confluent Platform:

- File konfigurasi yang penting

- Alat CLI yang berguna

- Skrip penting dan variabel lingkungan yang dibutuhkan

# Instalasi Confluent menggunakan Tarball

Instal Confluent Platform dari Tarball
Buka terminal di lingkungan lab Fresh Install dan instal Java 11 dengan pelatihan kata sandi sudo.

```
$ sudo apt-get install openjdk-11-jre-headless
```

Confluent Platform memerlukan Java 8 atau Java 11. Jika Anda memiliki beberapa instalasi Java, tetapkan variabel JAVA_HOME ke lokasi Java 8 atau 11.

Unduh dan ekstrak arsip TAR Confluent Platform 7.1.1.

```
$ curl -O http://packages.confluent.io/archive/7.1/confluent-7.1.1.tar.gz
$ tar xzf confluent-7.1.1.tar.gz
```
Berikut cara untuk menjelajahi direktori di Visual Studio Code.

```
$ code ~/confluent-7.1.1
```

Konfigurasikan Confluent CLI

Tetapkan variabel CONFLUENT_HOME ke lokasi penginstalan dan tambahkan ke .bashrc.

```
$ export CONFLUENT_HOME=${HOME}/confluent-7.1.1 \
&& echo "export CONFLUENT_HOME=$CONFLUENT_HOME" >> ~/.bashrc
```

Tambahkan biner Confluent Platform (yang menyertakan Confluent CLI) ke variabel PATH.

```
$ echo "export PATH=$CONFLUENT_HOME/bin:${PATH}" >> ~/.bashrc
```
Aktifkan pelengkapan bash untuk Confluent CLI dan sumberkan .bashrc agar semua perubahan berlaku.

```
$ ~/confluent-7.1.1/bin/confluent completion bash | sudo tee /etc/bash_completion.d/confluent \
&& echo "source /etc/bash_completion.d/confluent" >> ~/.bashrc \
&& source ~/.bashrc
```

Sekarang CLI confluent tersedia di PATH dan memiliki pelengkapan otomatis Tab. Ini akan berguna.

Mulai Layanan Lokal Confluent

Mulai seluruh Platform Confluent.

```
$ confluent local services start
```

Output dari perintah sebelumnya menunjukkan lokasi data dan konfigurasi di /tmp/confluent.<some number>. Jelajahi direktori tersebut secara singkat di Visual Studio Code.

```
$ code /tmp/confluent.<number> # ganti <number> dengan nomor Anda
```

Jika Anda mengalami masalah, Anda dapat confluent local destroy untuk memulai lagi. Cara yang lebih ekstrem adalah dengan ```sudo rm -rf /tmp/*```.

Menghasilkan dan Mengonsumsi Data Avro

Buat file skema Avro untuk data yang akan Anda hasilkan ke Kafka.

```
$ cat <<EOF > ~/temperature_reading.avsc
{
"namespace": "io.confluent.examples",
"type": "record",
"name": "temperature_reading",
"fields": [
{"name": "city", "type": "string"},
{"name": "temp", "type": "int", "doc": "temperature in Fahrenheit"} ]
}
EOF
```

Buka jendela terminal kedua dan atur kedua jendela berdampingan.

Di jendela terminal kanan, siapkan konsumen baris perintah untuk membaca kunci dan nilai dari pesan di topik suhu Kafka.

```
$ confluent local services \
kafka consume temperature \
--property print.key=true \
--property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
--value-format avro
```

confluent local services kafka tahu untuk mencari Kafka di localhost:9092. Properti print.key=true berarti kita akan melihat kunci setiap peristiwa selain nilainya. Kita menyediakan deserializer untuk kunci tersebut. --value-format avro berarti kita mengharapkan data serial Avro untuk nilai tersebut.

Di terminal kiri, mulai produsen baris perintah untuk menghasilkan peristiwa ke topik suhu Kafka.

```
$ confluent local services \
kafka produce temperature \
--property parse.key=true --property key.separator=, \
--property key.serializer=org.apache.kafka.common.serialization.StringSerializer \
--value-format avro \
--property value.schema.file=$HOME/temperature_reading.avsc
```

Parse.key=true berarti kita memproduksi peristiwa dengan kunci dan nilai, bukan hanya nilai. key.separator=, berarti kita menggunakan koma untuk memisahkan kunci dan nilai. Kita menentukan serializer String untuk kunci tersebut. Format nilai adalah Avro dan skema untuk nilai tersebut disediakan dengan file temperature_reading.avsc yang kita buat sebelumnya.

Tulis beberapa peristiwa suhu ke topik suhu. Tekan Enter untuk mengirimkan setiap peristiwa.

```
alameda,{"city":"alameda","temp":58}
ashland,{"city":"ashland","temp":62}
nairobi,{"city":"nairobi","temp":65}
sydney,{"city":"sydney","temp":75}
```

Coba tambahkan peristiwa yang nilainya tidak cocok dengan skema. Mengapa gagal?

```
jaipur,{"city":"jaipur","temp":"87"}
```

CLI lokal confluent tahu cara mendaftarkan dan mencari skema dengan layanan Schema Registry di http://localhost:8081
Hentikan semua proses produsen dan konsumen yang sedang berjalan dengan Ctrl+C.

(Opsional) Jelajahi Confluent Control Center

Buka browser dan navigasikan ke http://localhost:9021 untuk membuka Confluent Control Center.

Pilih Cluster 1 dari menu sebelah kiri.

Hancurkan penyebaran Platform Confluent lokal Anda.
```
$ confluent local destroy
```

# Jelajahi Skrip run-class dan Variabel Environment

Apa Itu Skrip run-class?
Luncurkan mesin virtual lab yang disebut "Confluent Platform Preinstalled" untuk membuat lingkungan lab tempat CP telah diinstal sebelumnya dengan manajer paket apt.

Lihat berkas unit systemd Confluent Server, khususnya baris ExecStart yang menentukan cara memulai layanan.

```
$ systemctl cat confluent-server
```

Di mana layanan Confluent Server mencari berkas properti konfigurasinya?

/etc/kafka/server.properties

Skrip apa yang dijalankan untuk memulai layanan?

/usr/bin/kafka-server-start

Buka skrip kafka-server-start dan perhatikan bahwa skrip tersebut merupakan pembungkus tipis di sekitar skrip kafka-run-class.

```
$ code $(which kafka-server-start)
```

Variabel lingkungan apa yang diekspor ke proses kafka-run-class dan apa fungsinya?

KAFKA_LOG4J_OPTS diekspor dengan nilai default
```
"-Dlog4j.configuration=file:/etc/kafka/log4j.properties"
```

Variabel ini digunakan untuk memuat file konfigurasi dari /etc/kafka/log4j.properties ke dalam Java Virtual Machine untuk mengonfigurasi pengaturan pencatatan log4j.

Variabel KAFKA_HEAP_OPTS diekspor dengan nilai default "-Xmx1G -Xms1G", yang akan menetapkan 1 Gigabyte memori ke tumpukan JVM broker Kafka.

Urutkan Variabel Lingkungan

Buka skrip kafka-run-class itu sendiri.

dengan command berikut:

```
$ code $(which kafka-run-class)
```

Pelajari variabel lingkungan dalam skrip kafka-run-class dan buat peringkat 5 variabel terpenting. Tidak ada jawaban pasti untuk pertanyaan ini, tetapi contoh respons disediakan untuk Anda.

CLASSPATH: Variabel ini penting untuk memuat kelas Java ke dalam Java Virtual Machine (JVM) dari file .jar yang diinstal oleh manajer paket. Secara default, CLASSPATH dibuat dari berbagai direktori di bawah /usr/share/java/. Mengetahui hal ini dapat membantu saat men-debug kesalahan saat kelas tidak dimuat karena suatu alasan. Di sisi klien Kafka, CLASSPATH umumnya dimodifikasi untuk menginstrumentasikan aplikasi klien dengan interseptor pemantauan untuk mengirim metrik klien ke Confluent Control Center. Interseptor pemantauan confluent terletak di /usr/share/java/monitoring-interceptors/monitoring-interceptors-6.0.0.jar.

KAFKA_HEAP_OPTS: Broker Kafka menggunakan tumpukan Java untuk mereplikasi partisi dengan sekitar 1 Megabyte per replika. Ukuran tumpukan JVM default untuk broker adalah 1 Gigabyte, yang sering kali terlalu kecil untuk penerapan produksi besar. Biasanya 4-6 GB direkomendasikan, dengan penerapan yang sangat besar pada 12 GB.

Contoh: KAFKA_HEAP_OPTS="-Xms6g -Xmx6g"

KAFKA_JVM_PERFORMANCE_OPTS: Variabel lingkungan ini dapat digunakan untuk menyetel pengumpulan sampah dan konfigurasi JVM lainnya. Berikut adalah contoh penggunaan rekomendasi produksi Confluent:

KAFKA_JVM_PERFORMANCE_OPTS="-XX:MetaspaceSize=96m -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:G1HeapRegionSize=16M -XX:MinMetaspaceFreeRatio=50 -XX:MaxMetaspaceFreeRatio=80"

JMX_PORT: Jika port JMX (Java Management Extensions) tidak ditentukan, maka broker Kafka tidak akan memaparkan metriknya melalui JMX.

Contoh: JMX_PORT=9990

JMX_OPTS: Dalam produksi, merupakan hal yang umum untuk menyesuaikan opsi JMX guna mengamankan koneksi dengan SSL. Berikut adalah contoh di mana klien JMX dan broker Kafka harus saling mengautentikasi melalui SSL:

KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote \ -Djavax.net.ssl.keyStore=</path/to/file>.keystore \ -Djavax.net.ssl.keyStorePassword=<keystore-password> \ -Dcom.sun.management.jmxremote.ssl.need.client.auth=true \ -Djavax.net.ssl.trustStore=</path/to/file>.truststore \ -Djavax.net.ssl.trustStorePassword=<truststore-password> \ -Dcom.sun.management.jmxremote.registry.ssl=true"

Sesuaikan Variabel Lingkungan dalam systemd
Biasanya, kustomisasi variabel lingkungan dilakukan dengan mengganti file unit systemd yang disediakan. Dalam kasus ini, kami akan mengganti confluent-server.service untuk menambah memori yang tersedia untuk heap JVM.

Buat direktori untuk layanan. Kata sandi sudo sedang dilatih.

```
$ sudo mkdir /etc/systemd/system/confluent-server.service.d
```

Buat file penggantian systemd untuk layanan confluent-server guna menambah memori heap hingga 4 GB.

```
$ cat << EOF | sudo tee /etc/systemd/system/confluent-server.service.d/override.conf
[Layanan]
Environment="KAFKA_HEAP_OPTS=-Xms4g -Xmx4g"
EOF
```

systemd akan secara otomatis menggabungkan pengaturan di /etc/systemd/system/<service>.service.d/override.conf dengan file layanan asli /lib/systemd/system/<service>.service sebelum memulai layanan.

Muat ulang daemon systemd untuk mengambil alih layanan.

```
$ sudo systemctl daemon-reload
```

Mulai ZooKeeper.

```
$ sudo systemctl start confluent-zookeeper
```

Mulai Confluent Server.

```
$ sudo systemctl start confluent-server
```

Periksa apakah broker Kafka berjalan dengan ukuran heap JVM yang dikonfigurasi.

```
$ systemctl status confluent-server
```

Berikut command jika ingin mulai Confluent Control Center menggunakan konfigurasi pengembangan lokal.

```
$ control-center-start /etc/confluent-control-center/control-center-dev.properties
```

Anda akan melihat output seperti ini:

```

● confluent-server.service - Apache Kafka - broker
Loaded: loaded (/lib/systemd/system/confluent-server.service; disabled; vendor preset: enabled)
Drop-In: /etc/systemd/system/confluent-server.service.d
└─override.conf
Active: active (running) sejak Thu 2020-11-05 22:13:06 UTC; 20 detik yang lalu
Dokumen: http://docs.confluent.io/
PID Utama: 19983 (java)
Tugas: 93 (batas: 4915)
CGroup: /system.slice/confluent-server.service
└─19983 java -Xms4g -Xmx4g -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 ...

```

