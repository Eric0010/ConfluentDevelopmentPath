#  Enable SSL on Kafka Broker

![image](https://github.com/user-attachments/assets/48087ba0-f7ae-4c69-bf62-42f1677118a2)

Untuk mengenkripsi pesan Kafka saat transit antara Klien Kafka dan Broker Kafka menggunakan SSL, pertama-tama kita perlu mengaktifkan SSL pada Broker Kafka dengan menyelesaikan langkah-langkah berikut:

# Buat Penyimpanan Key dan Sertifikat SSL

![image](https://github.com/user-attachments/assets/4cf8687b-545f-4f69-9e76-750d0d69243a)

Untuk membuat penyimpanan key dan sertifikat SSL:

Buat berkas konfigurasi Otoritas Sertifikat (CA) yang digunakan oleh perintah openssl:

```
[ policy_match ]
countryName = match
stateOrProvinceName = match
organizationName = match
organizationalUnitName = optional
commonName = supplied
emailAddress = optional

[ req ]
prompt = no
distinguished_name = dn
default_md = sha256
default_bits = 4096
x509_extensions = v3_ca

[ dn ]
countryName = {country code, e.g., US}
organizationName = {organization name}
localityName = {locality, e.g., city name}
commonName = {certificate common name, e.g., <company name>-ca}

[ v3_ca ]
subjectKeyIdentifier=hash
basicConstraints = critical,CA:true
authorityKeyIdentifier=keyid:always,issuer:always
keyUsage = critical,keyCertSign,cRLSign
```

Buat kunci & sertifikat CA:

```
openssl req -new -nodes \
   -x509 \
   -days {validity} \
   -newkey rsa:2048 \
   -keyout ca.key \
   -out ca.crt \
   -config ca.cnf
```

Ubah kunci CA ke format pem:

```
cat ca.crt ca.key > ca.pem
```

Buat permintaan penandatanganan kunci & sertifikat server (berkas .csr):

```
openssl req -new \
    -newkey rsa:2048 \
    -keyout kafka.server.key \
    -out kafka.server.csr \
    -config kafka.server.cnf \
    -nodes
```

Tandatangani sertifikat server dengan CA:

```
openssl x509 -req \
    -days {validity} \
    -in kafka.server.csr \
    -CA ca.crt \
    -CAkey ca.key \
    -CAcreateserial \
    -out kafka.server.crt \
    -extfile kafka.server.cnf \
    -extensions v3_req
```

Ubah sertifikat server ke format pkcs12:

```
openssl pkcs12 -export \
    -in kafka.server.crt \
    -inkey kafka.server.key \
    -chain \
    -CAfile ca.pem \
    -name localhost \
    -out kafka.server.p12 \
    -password pass:test1234
```

Buat penyimpanan kunci server:

```
sudo keytool -importkeystore \
    -deststorepass test1234 \
    -destkeystore kafka.server.keystore.pkcs12 \
    -srckeystore kafka.server.p12 \
    -deststoretype PKCS12  \
    -srcstoretype PKCS12 \
    -noprompt \
    -srcstorepass test1234
```

# Konfigurasikan Properti SSL Broker Kafka

Identifikasi Lokasi dan Kata Sandi Penyimpanan Kunci Broker
Untuk mengaktifkan SSL pada Broker Kafka, kita perlu:

Menambahkan pengaturan penyimpanan kunci ke berkas properti broker

contoh pengaturan
```
ssl.keystore.location=/var/ssl/private/kafka.server.keystore.pkcs12
ssl.keystore.password=test1234
ssl.key.password=test1234
```

# Konfigurasikan Listener SSL Broker Kafka
```
listeners=SSL://:9093,SASL_SSL://:9094
```

Dalam contoh ini, kami mendefinisikan dua listener SSL

Listener SSL://:9093 tidak memerlukan autentikasi klien kecuali ssl.client.auth=required juga ditetapkan

Listener SASL_SSL://:9094 memerlukan autentikasi klien SASL
