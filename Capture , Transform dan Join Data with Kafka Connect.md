# Capture Data

Buat Topik
Dalam kasus ini, kami ingin membiarkan produsen data menentukan skema, jadi alih-alih membuat topik di ksqlDB dengan kata kunci PARTITIONS, kami akan membuat topik di luar ksqlDB:

Buka Topik di Pusat Kontrol Confluent Anda

Pilih + Buat topik dan buat topik bernama ksqldb.stockapp.trades dengan 4 partisi dan pengaturan default.

Pilih + Buat topik dan buat topik bernama ksqldb.stockapp.users dengan 4 partisi dan pengaturan default.

Buat Konektor di ksqlDB
Buat konektor sumber untuk mengisi peristiwa perdagangan saham palsu.

```
CREATE SOURCE CONNECTOR `stockapp.trades` WITH (
    'connector.class' = 'io.confluent.kafka.connect.datagen.DatagenConnector',
    'kafka.topic' = 'ksqldb.stockapp.trades',
    'quickstart' = 'Stock_Trades',
    'max.interval' = 3000,
    'iterations' = 1000,
    'tasks.max' = '1'
);
```
Buat konektor sumber untuk mengisi data pengguna palsu.

```
CREATE SOURCE CONNECTOR `stockapp.users` WITH (
    'connector.class' = 'io.confluent.kafka.connect.datagen.DatagenConnector',
    'kafka.topic' = 'ksqldb.stockapp.users',
    'quickstart' = 'Users_',
    'max.interval' = 3000,
    'iterations' = 1000,
    'tasks.max' = '1'
);
```

Periksa Data di ksqlDB
Buat aliran bernama stockapp_trades dari topik ksqldb.stockapp.trades.

```
CREATE STREAM stockapp_trades WITH (
    KAFKA_TOPIC = 'ksqldb.stockapp.trades',
    VALUE_FORMAT = 'AVRO'
);
```

Kueri aliran perdagangan saham untuk melihat perdagangan yang mengalir secara real time. Hentikan kueri Anda setelah sekitar 30 detik.

```
SELECT * FROM stockapp_trades EMIT CHANGES;
```

Buat tabel bernama stockapp_users dari topik ksqldb.stockapp.users. Ini adalah tabel, bukan aliran karena kami biasanya hanya peduli dengan informasi profil terbaru untuk setiap pengguna.

```
CREATE TABLE stockapp_users (
userid STRING PRIMARY KEY,
registertime BIGINT,
regionid STRING,
gender STRING,
interests ARRAY<STRING>,
contactinfo MAP<STRING, STRING>
) WITH (
KAFKA_TOPIC = 'ksqldb.stockapp.users',
VALUE_FORMAT = 'AVRO'
);
```
Kueri tabel pengguna untuk melihat pembaruan yang mengalir secara langsung. Hentikan kueri Anda setelah sekitar 30 detik.

```
SELECT * FROM stockapp_users EMIT CHANGES;
```

Tentang Connector:

![image](https://github.com/user-attachments/assets/95ef370d-897b-4e46-916d-03ce7d629be4)

Mengonfigurasi konektor memerlukan pengetahuan tentang sistem eksternal

Contoh: Jika Anda menggunakan konektor Datagen untuk menghasilkan data uji, Anda perlu mengetahui fungsi berbagai parameter konfigurasi, seperti max.interval

Contoh: Jika Anda mengambil perubahan dari tabel basis data dengan konektor JDBC, Anda perlu mengetahui URL koneksi JDBC untuk mengakses basis data Anda dengan aman.

Setiap konektor memiliki dokumentasinya sendiri yang menjelaskan fungsi setiap parameter konfigurasi.

Ilustrasi menunjukkan cara membuat konektor menggunakan sintaks SQL langsung di dalam aplikasi ksqlDB Anda.

Kirim perintah ke ksqlDB

ksqlDB mengirimkan permintaan ke layanan Kafka Connect

Kafka Connect memindahkan data antara sistem eksternal dan Topik Kafka.

Aplikasi Streaming Acara Sejauh Ini
-- Capture Data

```
-- Capture Data

CREATE SOURCE CONNECTOR `stockapp.trades` WITH (
    'connector.class' = 'io.confluent.kafka.connect.datagen.DatagenConnector',
    'kafka.topic' = 'ksqldb.stockapp.trades',
    'quickstart' = 'Stock_Trades',
    'max.interval' = 3000,
    'iterations' = 1000,
    'tasks.max' = '1'
);

CREATE SOURCE CONNECTOR `stockapp.users` WITH (
    'connector.class' = 'io.confluent.kafka.connect.datagen.DatagenConnector',
    'kafka.topic' = 'ksqldb.stockapp.users',
    'quickstart' = 'Users_',
    'max.interval' = 3000,
    'iterations' = 1000,
    'tasks.max' = '1'
);

CREATE TABLE stockapp_users (
    userid STRING PRIMARY KEY,
    registertime BIGINT,
    regionid STRING,
    gender STRING,
    interests ARRAY<STRING>,
    contactinfo MAP<STRING, STRING>
) WITH (
    KAFKA_TOPIC = 'ksqldb.stockapp.users',
    VALUE_FORMAT = 'AVRO'
);

CREATE STREAM stockapp_trades
WITH (
    KAFKA_TOPIC = 'ksqldb.stockapp.trades',
    VALUE_FORMAT = 'AVRO'
);
```

# Transform

Untuk membuat Transform, saya ingin membuat seperti berikut:

Membuat aliran peristiwa perdagangan saham yang disebut stockapp_trades dari topik ksqldb.stockapp.trades.

Membuat aliran baru stockapp_trades_transformed yang berasal dari aliran stockapp_trades:

- menghitung bidang baru dollar_amount yang merupakan jumlah dolar dari setiap perdagangan
    
- Menyembunyikan semua huruf dalam atribut "akun" dengan tanda bintang ('*')
    
- Memilih hanya pesan yang atribut "simbol"-nya diakhiri dengan "T"
    
- Bidang-bidang tersebut harus berupa dollar_amount, account_masked, symbol, dan userid.
    
Dari Code sebelumnya, saya akan menambahkan Stream Transform:

```
-- Capture Data

CREATE SOURCE CONNECTOR `stockapp.trades` WITH (
    'connector.class' = 'io.confluent.kafka.connect.datagen.DatagenConnector',
    'kafka.topic' = 'ksqldb.stockapp.trades',
    'quickstart' = 'Stock_Trades',
    'max.interval' = 3000,
    'iterations' = 1000,
    'tasks.max' = '1'
);

CREATE SOURCE CONNECTOR `stockapp.users` WITH (
    'connector.class' = 'io.confluent.kafka.connect.datagen.DatagenConnector',
    'kafka.topic' = 'ksqldb.stockapp.users',
    'quickstart' = 'Users_',
    'max.interval' = 3000,
    'iterations' = 1000,
    'tasks.max' = '1'
);

CREATE TABLE stockapp_users (
    userid STRING PRIMARY KEY,
    registertime BIGINT,
    regionid STRING,
    gender STRING,
    interests ARRAY<STRING>,
    contactinfo MAP<STRING, STRING>
) WITH (
    KAFKA_TOPIC = 'ksqldb.stockapp.users',
    VALUE_FORMAT = 'AVRO'
);

CREATE STREAM stockapp_trades
WITH (
    KAFKA_TOPIC = 'ksqldb.stockapp.trades',
    VALUE_FORMAT = 'AVRO'
);

-- Perform Continuous Transformations

CREATE STREAM stockapp_trades_transformed AS
    SELECT
        CAST(price AS DECIMAL(7,2)) * quantity / 100 AS dollar_amount,
        MASK(account, '*', '*', NULL, NULL) AS account_masked,
        symbol,
        userid
    FROM stockapp_trades
    WHERE symbol LIKE '%T'
    EMIT CHANGES;
```

# Join

Untuk Join, saya ingin menggabungkan data sebagai berikut:

Membuat aliran stockapp_transformed_enriched dengan menggabungkan aliran stockapp_trades_transformed dengan stockapp_users. 

Aliran yang diperkaya harus menyertakan kolom berikut:

- userid

- dolar_amount

- regionid

- interests

- contactinfo

Saya juga harus melakukan query pada aliran stockapp_trades_transformed_enriched untuk melihat rekaman yang diubah mengalir secara real time. Hentikan permintaan Anda setelah 30 detik atau lebih dengan command:

```
SELECT * FROM stockapp_trades_transformed_enriched EMIT CHANGES;
```

Maka Code yang dilakukan akan seperti berikut:

```
-- Capture Data

CREATE SOURCE CONNECTOR `stockapp.trades` WITH (
    'connector.class' = 'io.confluent.kafka.connect.datagen.DatagenConnector',
    'kafka.topic' = 'ksqldb.stockapp.trades',
    'quickstart' = 'Stock_Trades',
    'max.interval' = 3000,
    'iterations' = 1000,
    'tasks.max' = '1'
);

CREATE SOURCE CONNECTOR `stockapp.users` WITH (
    'connector.class' = 'io.confluent.kafka.connect.datagen.DatagenConnector',
    'kafka.topic' = 'ksqldb.stockapp.users',
    'quickstart' = 'Users_',
    'max.interval' = 3000,
    'iterations' = 1000,
    'tasks.max' = '1'
);

CREATE TABLE stockapp_users (
    userid STRING PRIMARY KEY,
    registertime BIGINT,
    regionid STRING,
    gender STRING,
    interests ARRAY<STRING>,
    contactinfo MAP<STRING, STRING>
) WITH (
    KAFKA_TOPIC = 'ksqldb.stockapp.users',
    VALUE_FORMAT = 'AVRO'
);

CREATE STREAM stockapp_trades
WITH (
    KAFKA_TOPIC = 'ksqldb.stockapp.trades',
    VALUE_FORMAT = 'AVRO'
);

-- Perform Continuous Transformations

CREATE STREAM stockapp_trades_transformed AS
    SELECT
        CAST(price AS DECIMAL(7,2)) * quantity / 100 AS dollar_amount,
        MASK(account, '*', '*', NULL, NULL) AS account_masked,
        symbol,
        userid
    FROM stockapp_trades
    WHERE symbol LIKE '%T'
    EMIT CHANGES;

CREATE STREAM stockapp_trades_transformed_enriched AS
    SELECT s.userid, s.dollar_amount, s.symbol,
           u.regionid, u.interests, u.contactinfo
    FROM stockapp_trades_transformed s
    LEFT JOIN stockapp_users u
        ON s.userid = u.userid
    EMIT CHANGES;
```
