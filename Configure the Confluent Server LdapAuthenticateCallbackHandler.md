# Konfigurasi Autentikasi LDAP

![image](https://github.com/user-attachments/assets/892930fa-b16b-4941-871f-40056fad68bf)

Bagian penekanan normal dari diagram di atas menggambarkan proses autentikasi klien Kafka melalui LDAP yang menjadi fokus kursus ini.

Sisa diagram yang penekanannya dikurangi menggambarkan proses autentikasi dan otorisasi Kafka lengkap yang disediakan oleh Kontrol Akses Berbasis Peran (RBAC) Confluent Platform. Implementasi RBAC Confluent Platform dibahas secara terperinci dalam kursus Mengelola Akses ke Confluent Platform dengan Kontrol Akses Berbasis Peran.

Ada dua persyaratan utama untuk mengaktifkan autentikasi klien LDAP

Konfigurasikan SASL untuk mengautentikasi pengguna/prinsip menggunakan LDAP

Pengaturan koneksi layanan direktori LDAP

```
ldap.java.naming.provider.url=ldaps://directory-service:10636
# Autentikasi ke LDAP
ldap.java.naming.security.principal=uid=admin,ou=system
ldap.java.naming.security.credentials=secret
ldap.java.naming.security.authentication=simple
```

Pengaturan pencarian pengguna

```
# Temukan pengguna
ldap.user.search.base=ou=users,dc=confluent,dc=io
ldap.user.name.attribute=uid
ldap.user.object.class=inetOrgPerson
```

Konfigurasikan broker listener yang akan digunakan LdapAuthenticateCallbackHandler

```
listener.name.client.sasl.enabled.mechanisms=PLAIN
listener.name.client.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule diperlukan;
listener.name.client.plain.sasl.server.callback.handler.class=io.confluent.security.auth.provider.ldap.LdapAuthenticateCallbackHandler
```

Jika Kontrol Akses Berbasis Peran (RBAC) Confluent Platform akan digunakan untuk mengotorisasi akses ke prinsip berdasarkan keanggotaan mereka dalam grup layanan direktori LDAP, maka pengaturan pencarian grup juga harus dikonfigurasi.

```
ldap.search.mode=GROUPS
ldap.group.search.base=ou=groups,dc=confluent,dc=io
ldap.group.object.class=groupOfNames
ldap.group.name.attribute=cn
ldap.group.member.attribute=member
ldap.group.member.attribute.pattern=cn=(.*),ou=users,dc=confluent,dc=io
```
