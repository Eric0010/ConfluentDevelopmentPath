# Konfigurasi Autentikasi mTLS Broker

Dengan autentikasi mtutual TLS (mTLS) – juga dikenal sebagai “two way authentication” – klien dan server Kafka menggunakan sertifikat TLS untuk memverifikasi identitas satu sama lain guna memastikan bahwa lalu lintas aman dan tepercaya di kedua arah.

Berikut adalah UseCase mTLS:

![image](https://github.com/user-attachments/assets/70df5be4-9cc4-4a86-a3c5-7dfc01d3dd26)

Persyaratan Autentikasi mTLS Broker

Tetapkan properti broker client.auth ke true

Tetapkan protokol antar broker ke SSL menggunakan salah satu dari dua metode berikut:

inter.broker.protocol=SSL

inter.broker.listener.name=<nama listener> dan listener.security.protocol.map=<nama listener>:SSL

Buat truststore broker

Broker menggunakan truststore SSL untuk mengautentikasi broker lain dalam kluster menggunakan mTLS.

Identifikasi lokasi truststore broker dalam berkas properti broker

Berikut adalah cara untuk melakukan konfigurasi Autentikasi mTLS broker:

buat Broker Truststores
Di bagian ini, kita akan membuat broker truststore. Kita juga akan mengatur nilai properti broker ssl.truststore.location ssl.truststore.credentials yang sesuai di docker-compose.yml.

1. Buat truststore broker kafka-1 dan impor sertifikat CA:

```
sudo keytool -keystore ~/mtls/kafka-1-creds/kafka.kafka-1.truststore.pkcs12 \
-alias CARoot \
-import \
-file ~/mtls/ca.crt \
-storepass confluent \
-noprompt \
-storetype PKCS12
```

Anda akan melihat output berikut:

Sertifikat telah ditambahkan ke keystore

Simpan kredensial truststore ke dalam file:
```
echo "confluent" > ~/mtls/kafka-1-creds/kafka-1_truststore_creds
```

Parameter ssl.truststore.credentials akan ditetapkan ke nama file ini.

Buat truststore broker Kafka-2 dan Kafka-3. 
```
sudo ~/mtls/scripts/truststore-create-kafka-2-3.sh
```

Kembali ke VSCode dan temukan pengaturan konfigurasi berikut untuk broker kafka-1:

```
91 #KAFKA_SSL_TRUSTSTORE_FILENAME: kafka.kafka-1.truststore.jks
92 #KAFKA_SSL_TRUSTSTORE_CREDENTIALS: kafka-1_truststore_creds
```

Untuk mengatur lokasi untuk truststore broker dan kredensial truststore, hapus karakter komentar #:

```
91 KAFKA_SSL_TRUSTSTORE_FILENAME: kafka.kafka-1.truststore.jks
92 KAFKA_SSL_TRUSTSTORE_CREDENTIALS: kafka-1_truststore_creds
```

Ulangi langkah-langkah 4-5 untuk broker kafka-2 dan kafka-3.

Simpan docker-compose.yml dan keluar dari VSCode.

Aktifkan konfigurasi broker yang diperbarui:

```
docker-compose -f ~/mtls/docker-compose.yml up -d
```

Pastikan instance Zookeeper dan broker Kafka berjalan:

```
docker-compose -f ~/mtls/docker-compose.yml ps
```

Pastikan broker aktif dan merespons permintaan SSL:

```
openssl s_client -connect kafka-1-external:19093 -tls1_3
```

Anda akan melihat output yang mirip dengan berikut ini:

CONNECTED(00000005)
depth=1 C = US, O = Confluent, L = MountainView, CN = confluent-ca
verifikasi error:num=19:self signed certificate in certificate chain

Tekan Ctrl+C untuk menutup koneksi.

2. Atur Inter Broker Listener untuk Menggunakan Protokol SSL
Buka docker-compose.yml di VSCode:
```
code ~/mtls/docker-compose.yml
```

Di bagian lingkungan kontainer kafka-1:, cari pengaturan listener.security.protocol.map:
```
86 KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,SSL:SSL,BROKER:PLAINTEXT
```

Ubah BROKER:PLAINTEXT menjadi BROKER:SSL:

```
86 KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,SSL:SSL,BROKER:SSL
```

Ulangi langkah 2-3 untuk kontainer kafka-2 dan kafka-3.

Simpan docker-compose.yml dan keluar dari VSCode.

Jalankan kluster Kafka yang diperbarui:

```
docker-compose -f ~/mtls/docker-compose.yml up -d
```

Pastikan instans Zookeeper dan broker Kafka berjalan:

```
docker-compose -f ~/mtls/docker-compose.yml ps
```

Anda akan melihat output berikut:

Nama Perintah Status Port
--------------------------------------------------------------------------------------------------------------------------
kafka-1 /etc/confluent/docker/run Up 0.0.0.0:19092->19092/tcp, 0.0.0.0:19093->19093/tcp, 9092/tcp
kafka-2 /etc/confluent/docker/run Up 0.0.0.0:29092->29092/tcp, 0.0.0.0:29093->29093/tcp, 9092/tcp
kafka-3 /etc/confluent/docker/menjalankan 0.0.0.0:39092->39092/tcp, 0.0.0.0:39093->39093/tcp, 9092/tcp
zk-1 /etc/confluent/docker/menjalankan 0.0.0.0:12181->12181/tcp, 2181/tcp, 2888/tcp, 3888/tcp
zk-2 /etc/confluent/docker/menjalankan 2181/tcp, 0.0.0.0:22181->22181/tcp, 2888/tcp, 3888/tcp
zk-3 /etc/confluent/docker/run Naik 2181/tcp, 2888/tcp, 0.0.0.0:32181->32181/tcp, 3888/tcp

3. Verifikasikan Lalu Lintas Antar Broker SSL Beroperasi
   
Buat topik baru:

```
kafka-topics \
--bootstrap-server kafka-1-external:19093 \
--command-config ~/mtls/client-creds/client-ssl.properties \
--create \
--topic test-topic \
--partitions 1 \
--replication-factor 3
```
Buat catatan untuk test-topic:

```
kafka-console-producer \
--bootstrap-server kafka-1-external:19093,kafka-2-external:29093 \
--topic test-topic \
--producer.config ~/mtls/client-creds/client-ssl.properties
```

Di > prompt, ketik “Security is good” dan tekan Enter.

Tambahkan beberapa pesan lagi lalu tekan Ctrl+D untuk keluar dari konsol Producer.

Konsumsi rekaman dari test-topic:

```
kafka-console-consumer \
--bootstrap-server kafka-1-external:19093,kafka-2-external:29093 \
--group test-consumer-group \
--topic test-topic \
--consumer.config ~/mtls/client-creds/client-ssl.properties \
--from-beginning
```

Tekan Ctrl+C untuk keluar dari konsol Consumer.

Jelaskan topik pengujian:

```
kafka-topics \
--bootstrap-server kafka-1-external:19093 \
--command-config ~/mtls/client-creds/client-ssl.properties \
--describe \
--topic topik pengujian
```

Anda akan melihat output yang mirip dengan berikut ini:

...
Topik: topik pengujian TopicId: UT8DpdGMQYC1LuE4SFj0zw PartitionCount: 1 ReplicationFactor: 1 Konfigurasi:
Topik: topik pengujian Partisi: 0 Pemimpin: 102 Replika: 102 Isr: 102 Offline:
