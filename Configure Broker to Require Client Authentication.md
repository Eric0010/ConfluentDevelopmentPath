# Konfigurasi Broker yang Memerlukan Autentikasi Klien

Jalankan kluster Kafka:

```
docker-compose -f ~/mtls/docker-compose.yml up -d
```
Verifikasi apakah instance Zookeeper dan broker Kafka sedang berjalan:

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
zk-3 /etc/confluent/docker/run Up 2181/tcp, 2888/tcp, 0.0.0.0:32181->32181/tcp, 3888/tcp

Pastikan broker aktif dan menanggapi permintaan SSL:

```
openssl s_client -connect kafka-1-external:19093 -tls1_3
```

Anda akan melihat output yang mirip dengan berikut ini:

CONNECTED(00000005)
depth=1 C = US, O = Confluent, L = MountainView, CN = confluent-ca
verifikasi error:num=19:self signed certificate in certificate chain
...
...
Tekan Ctrl+C untuk menutup koneksi.

Verifikasi Autentikasi Klien Saat Ini Tidak Diperlukan

Periksa berkas properti klien:

```
cat ~/mtls/client-creds/client-ssl.properties
```

Anda akan melihat keluaran berikut:

security.protocol = SSL
ssl.truststore.location=/home/training/mtls/client-creds/kafka.client.truststore.pkcs12
ssl.truststore.password=confluent
#ssl.keystore.location = /home/training/mtls/client-creds/kafka.client.keystore.pkcs12
#ssl.keystore.password = confluent
#ssl.key.password = confluent

Buat topik:

```
kafka-topics \
--bootstrap-server kafka-1-external:19093 \
--create \
--topic test-topic \
--partitions 1 \
--replication-factor 3 \
--command-config ~/mtls/client-creds/client-ssl.properties
```

Anda akan melihat output yang mirip dengan berikut ini:

[2021-03-25 18:41:31,437] WARN Konfigurasi 'ssl.truststore.location' diberikan tetapi bukan konfigurasi yang dikenal. (org.apache.kafka.clients.admin.AdminClientConfig)
[2021-03-25 18:41:31,437] WARN Konfigurasi 'ssl.truststore.password' diberikan tetapi bukan konfigurasi yang dikenal. (org.apache.kafka.clients.admin.AdminClientConfig)
Topik test-topic dibuat.

Hapus topik:

```
kafka-topics \
--bootstrap-server kafka-1-external:19093 \
--delete \
--topic test-topic \
--command-config ~/mtls/client-creds/client-ssl.properties
```

Konfigurasikan Broker untuk Memerlukan Autentikasi Klien

Di bagian ini, kita akan mengonfigurasi broker untuk mewajibkan autentikasi klien.

Buka docker-compose.yml di VSCode:

```
code ~/mtls/docker-compose.yml
```

Temukan pengaturan konfigurasi berikut untuk broker kafka-1:

```
94 #KAFKA_SSL_CLIENT_AUTH: "diperlukan"
```

Untuk mewajibkan otorisasi klien SSL, hapus karakter komentar #:

```
94 KAFKA_SSL_CLIENT_AUTH: "diperlukan"
```

Ulangi langkah 2-3 untuk broker kafka-2 dan kafka-3.

Biarkan docker-compose.yml terbuka di VSCode karena kami akan membuat pembaruan tambahan di bagian berikutnya dari aktivitas ini.

Buat Truststore Broker

Di bagian ini, kami akan membuat truststore broker. Kami juga akan menetapkan nilai properti broker ssl.truststore.location ssl.truststore.credentials yang sesuai di docker-compose.yml.

Buat truststore broker kafka-1 dan impor sertifikat CA:

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

Buat truststore broker kafka-2 dan kafka-3.

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

Verifikasi apakah instance Zookeeper dan broker Kafka sedang berjalan:

```
docker-compose -f ~/mtls/docker-compose.yml ps
```
Verifikasi apakah broker aktif dan merespons permintaan SSL:

```
openssl s_client -connect kafka-1-external:19093 -tls1_3
```

Anda akan melihat output yang mirip dengan berikut ini:

CONNECTED(00000005)
depth=1 C = US, O = Confluent, L = MountainView, CN = confluent-ca
verifikasi error:num=19:self signed certificate in certificate chain
...
...
Tekan Ctrl+C untuk menutup koneksi.

Verifikasi Otorisasi Klien Sekarang Diperlukan

Buat topik:

```
kafka-topics \
--bootstrap-server kafka-1-external:19093 \
--create \
--topic test-topic \
--partitions 1 \
--replication-factor 3 \
--command-config ~/mtls/client-creds/client-ssl.properties
```

Anda akan melihat output yang mirip dengan berikut ini:

[2021-03-25 18:47:30,605] WARN Konfigurasi 'ssl.truststore.location' disediakan tetapi bukan konfigurasi yang dikenal. (org.apache.kafka.clients.admin.AdminClientConfig)
[2021-03-25 18:47:30,606] PERINGATAN Konfigurasi 'ssl.truststore.password' diberikan tetapi bukan konfigurasi yang diketahui. (org.apache.kafka.clients.admin.AdminClientConfig)
[2021-03-25 18:47:31,276] ERROR [AdminClient clientId=adminclient-1] Koneksi ke node -1 (kafka-1-external/127.0.0.1:19093) gagal diautentikasi karena: Gagal memproses pesan pasca-jabat tangan, nama host SNI: kosong (org.apache.kafka.clients.NetworkClient)
[2021-03-25 18:47:31,278] WARN [AdminClient clientId=adminclient-1] Pembaruan metadata gagal karena kesalahan autentikasi (org.apache.kafka.clients.admin.internals.AdminMetadataManager)
org.apache.kafka.common.errors.SslAuthenticationException: Gagal memproses pesan pasca-jabat tangan, nama host SNI: kosong
Kesalahan saat menjalankan perintah topik: Gagal memproses pesan pasca-jabat tangan, nama host SNI: kosong
[2021-03-25 18:47:31,284] KESALAHAN org.apache.kafka.common.errors.SslAuthenticationException: Gagal memproses pesan pasca-jabat tangan, nama host SNI: kosong
(kafka.admin.TopicCommand$)

Periksa log broker kafka-1 untuk pesan kesalahan terkait:

```
docker-compose -f ~/mtls/docker-compose.yml logs kafka-1 | grep "Otentikasi gagal"
```

Anda akan melihat output yang mirip dengan berikut ini:

kafka-1 | [2021-03-15 21:50:46,673] INFO [SocketServer brokerId=101] Otentikasi gagal dengan /172.19.0.1 (errorMessage=Jabat tangan SSL gagal disebabkan oleh rantai sertifikat klien kosong) (org.apache.kafka.common.network.Selector)
