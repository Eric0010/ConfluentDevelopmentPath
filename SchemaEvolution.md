# Schema Evolution and Compatibility for Schema Registry on Confluent Platform

Aspek penting dari manajemen data adalah evolusi skema. Setelah skema awal ditetapkan, aplikasi mungkin perlu mengembangkannya dari waktu ke waktu. Ketika ini terjadi, sangat penting bagi konsumen downstream untuk dapat menangani data yang dikodekan dengan skema lama dan baru dengan lancar.

# Schema Evolution

Saat menggunakan format serdes seperti Avro, JSON Schema, dan Protobuf, perlu diingat pentingnya mengelola skema dan pertimbangkan bagaimana skema ini harus berkembang. Confluent Schema Registry dibuat untuk tujuan tersebut. Pemeriksaan kompatibilitas skema diimplementasikan dalam Schema Registry dengan membuat versi setiap skema. Jenis kompatibilitas menentukan bagaimana Schema Registry membandingkan skema baru dengan versi skema sebelumnya, untuk subjek tertentu. Saat skema pertama kali dibuat untuk subjek, skema tersebut mendapatkan id unik dan nomor versi, yaitu versi 1. Saat skema diperbarui (jika lolos pemeriksaan kompatibilitas), skema tersebut mendapatkan id unik baru dan nomor versi yang bertambah, yaitu versi 2.

Berikut adalah Tabel yang menyajikan ringkasan jenis perubahan skema yang diizinkan untuk berbagai jenis kompatibilitas, untuk subjek tertentu. Jenis kompatibilitas default Confluent Schema Registry adalah BACKWARD. Semua jenis kompatibilitas dijelaskan secara lebih rinci di bagian di bawah ini.

![image](https://github.com/user-attachments/assets/d6fa1e0e-5908-4573-ae3f-796f92849555)

# Avro, Protobuf, dan Skema JSON memiliki aturan kompatibilitas yang berbeda

Aturan evolusi dan kompatibilitas skema bervariasi berdasarkan format skema. Skenario dan contoh kode sumber yang diberikan pada halaman ini ditujukan untuk Avro , yang merupakan serializer/deserializer pertama yang didukung Confluent. Avro dikembangkan dengan mempertimbangkan evolusi skema, dan spesifikasinya dengan jelas menyatakan aturan untuk kompatibilitas mundur; sedangkan aturan dan tata bahasa untuk Skema JSON dan Protobuf dapat lebih bernuansa.

Meskipun konsep umum berlaku untuk semua format, detail tentang cara penerapan kompatibilitas akan berbeda-beda, tergantung pada apakah Anda menggunakan Avro, JSON Schema, atau Protobuf. Untuk penjelasan lebih mendalam dan contoh spesifik tentang format yang lebih baru, lihat posting blog dan dokumentasi berikut:

Protobuf

Blog post: [Understanding Protobuf Compatibility](https://yokota.blog/2021/08/26/understanding-protobuf-compatibility/)
Confluent Cloud documentation: [Protobuf compatibility rules](https://docs.confluent.io/cloud/current/sr/fundamentals/serdes-develop/serdes-protobuf.html#protobuf-schema-compatibility-rules)
Confluent Platform documentation: [Protobuf compatibility rules](https://docs.confluent.io/platform/7.7/schema-registry/fundamentals/serdes-develop/serdes-protobuf.html#protobuf-schema-compatibility-rules)

Perhatikan bahwa praktik terbaik untuk Protobuf adalah menggunakan BACKWARD_TRANSITIVE, karena menambahkan jenis pesan baru tidak kompatibel dengan penerusan.

Skema JSON

Blog post: Understanding [JSON Schema Compatibility](https://yokota.blog/2021/03/29/understanding-json-schema-compatibility/)
Confluent Cloud documentation: [JSON Schema compatibility rules](https://docs.confluent.io/cloud/current/sr/fundamentals/serdes-develop/serdes-json.html#json-schema-compatibility-rules)
Confluent Platform documentation: [Protobuf compatibility rules](https://docs.confluent.io/platform/7.7/schema-registry/fundamentals/serdes-develop/serdes-json.html#json-schema-compatibility-rules) [Compatibility checks](https://docs.confluent.io/platform/7.7/schema-registry/fundamentals/serdes-develop/index.html#compatibility-checks)

# Backward compatibility

BACKWARD compatibility berarti bahwa konsumen yang menggunakan skema baru dapat membaca data yang dihasilkan dengan skema terakhir. Misalnya, jika ada tiga skema untuk subjek yang berubah dalam urutan X-2 , X-1 , dan X maka BACKWARD compatibility memastikan bahwa konsumen yang menggunakan skema baru X dapat memproses data yang ditulis oleh produsen yang menggunakan skema X atau X-1 , tetapi tidak harus X-2 . Jika konsumen yang menggunakan skema baru perlu dapat memproses data yang ditulis oleh semua skema yang terdaftar, bukan hanya dua skema terakhir, maka gunakan BACKWARD_TRANSITIVE sebagai ganti BACKWARD. Misalnya, jika ada tiga skema untuk subjek yang berubah dalam urutan X-2 , X-1 , dan X maka BACKWARD_TRANSITIVE compatibility memastikan bahwa konsumen yang menggunakan skema X baru dapat memproses data yang ditulis oleh produsen yang menggunakan skema X , X-1 , atau X-2 .

BACKWARD: konsumen yang menggunakan skema X dapat memproses data yang dihasilkan dengan skema X atau X-1
BACKWARD_TRANSITIVE: konsumen yang menggunakan skema X dapat memproses data yang dihasilkan dengan skema X , X-1 , atau X-2

Contoh perubahan kompatibilitas Backward adalah penghapusan suatu bidang. Konsumen yang dikembangkan untuk memproses kejadian tanpa bidang ini akan dapat memproses kejadian yang ditulis dengan skema lama dan berisi bidang tersebut – konsumen akan mengabaikan bidang tersebut.

Pertimbangkan kasus di mana semua data di Kafka juga dimuat ke HDFS, dan Anda ingin menjalankan kueri SQL (misalnya, menggunakan Apache Hive) pada semua data. Di sini, penting bahwa kueri SQL yang sama terus berfungsi bahkan saat data mengalami perubahan seiring waktu. Untuk mendukung kasus penggunaan semacam ini, Anda dapat mengembangkan skema dengan cara yang kompatibel dengan versi lama. Semua Format, Serializer, dan Deserializer memiliki aturan tentang perubahan apa yang diizinkan dalam skema baru agar kompatibel dengan versi lama. Misalnya, berikut adalah aturan Avro untuk kompatibilitas. Jika semua skema dikembangkan dengan cara yang kompatibel dengan versi lama, kita selalu dapat menggunakan skema terbaru untuk mengkueri semua data secara seragam.

# Forward compatibility

FORWARD compatibility berarti bahwa data yang dihasilkan dengan skema baru dapat dibaca oleh konsumen menggunakan skema terakhir, meskipun mereka mungkin tidak dapat menggunakan kapabilitas penuh dari skema baru. Misalnya, jika ada tiga skema untuk subjek yang berubah dalam urutan X-2 , X-1 , dan X maka FORWARD compatibility memastikan bahwa data yang ditulis oleh produsen menggunakan skema baru X dapat diproses oleh konsumen menggunakan skema X atau X-1 , tetapi tidak harus X-2 . Jika data yang dihasilkan dengan skema baru perlu dibaca oleh konsumen menggunakan semua skema terdaftar, bukan hanya dua skema terakhir, maka gunakan FORWARD_TRANSITIVE sebagai ganti FORWARD. Misalnya, jika ada tiga skema untuk subjek yang berubah dalam urutan X-2 , X-1 , dan X maka FORWARD_TRANSITIVE compatibility memastikan bahwa data yang ditulis oleh produsen menggunakan skema baru X dapat diproses oleh konsumen menggunakan skema X , X-1 , atau X-2 .

FORWARD:data yang dihasilkan menggunakan skema X dapat dibaca oleh konsumen dengan skema X atau X-1
FORWARD_TRANSITIVE: data yang dihasilkan menggunakan skema X dapat dibaca oleh konsumen dengan skema X , X-1 , atau X-2

Contoh modifikasi skema yang kompatibel ke depan adalah menambahkan kolom baru. Dalam sebagian besar format data, konsumen yang ditulis untuk memproses peristiwa tanpa kolom baru akan dapat terus melakukannya bahkan saat mereka menerima peristiwa baru yang berisi kolom baru.

Pertimbangkan kasus penggunaan saat konsumen memiliki logika aplikasi yang terikat pada versi skema tertentu. Saat skema berevolusi, logika aplikasi mungkin tidak segera diperbarui. Oleh karena itu, Anda harus dapat memproyeksikan data dengan skema yang lebih baru ke skema (yang lebih lama) yang dipahami aplikasi. Untuk mendukung kasus penggunaan ini, Anda dapat mengembangkan skema dengan cara yang kompatibel ke depan: data yang dikodekan dengan skema baru dapat dibaca dengan skema lama.

# Full compatibility

FULL compatibility berarti skema kompatibel baik ke belakang maupun ke depan. Skema berevolusi dengan cara yang sepenuhnya kompatibel: data lama dapat dibaca dengan skema baru, dan data baru juga dapat dibaca dengan skema terakhir. Misalnya, jika ada tiga skema untuk subjek yang berubah dalam urutan X-2 , X-1 , dan X maka FULLkompatibilitas memastikan bahwa konsumen yang menggunakan skema X yang baru dapat memproses data yang ditulis oleh produsen yang menggunakan skema X atau X-1 , tetapi tidak harus X-2 , dan bahwa data yang ditulis oleh produsen yang menggunakan skema X yang baru dapat diproses oleh konsumen yang menggunakan skema X atau X-1 , tetapi tidak harus X-2 . Jika skema baru perlu kompatibel ke depan dan ke belakang dengan semua skema yang terdaftar, bukan hanya dua skema terakhir, maka gunakan FULL_TRANSITIVE sebagai ganti FULL. Misalnya, jika ada tiga skema untuk subjek yang berubah dalam urutan X-2 , X-1 , dan X maka FULL_TRANSITIVE kompatibilitas memastikan bahwa konsumen yang menggunakan skema X baru dapat memproses data yang ditulis oleh produsen menggunakan skema X , X-1 , atau X-2 , dan bahwa data yang ditulis oleh produsen menggunakan skema X baru dapat diproses oleh konsumen yang menggunakan skema X , X-1 , atau X-2 .

FULL: kompatibilitas mundur dan maju antara skema X dan X-1
FULL_TRANSITIVE: kompatibilitas mundur dan maju antara skema X , X-1 , dan X-2

Di Avro dan Protobuf, Anda dapat menentukan kolom dengan nilai default. Dalam hal ini, menambahkan atau menghapus kolom dengan nilai default merupakan perubahan yang sepenuhnya kompatibel.

Aturan kompatibilitas untuk jenis skema yang didukung dijelaskan dalam Pemeriksaan kompatibilitas dalam Format, Serializer, dan Deserializer .

Skema JSON tidak secara eksplisit mendefinisikan aturan kompatibilitas, jadi postingan blog ini menjelaskan lebih lanjut cara kerja kompatibilitas Skema JSON , termasuk kompatibilitas penuh.

# No compatibility checking

NONE adalah jenis kompatibilitas berarti pemeriksaan kompatibilitas skema dinonaktifkan.

Terkadang kita membuat perubahan yang tidak kompatibel. Misalnya, mengubah jenis bidang dari Number menjadi String. Dalam kasus ini, Anda perlu upgrade semua produsen dan konsumen ke versi skema baru secara bersamaan, atau lebih mungkin – membuat topik baru dan mulai memigrasikan aplikasi untuk menggunakan topik dan skema baru, sehingga terhindar dari kebutuhan untuk menangani dua versi yang tidak kompatibel dalam topik yang sama.

# Transitive property

Pemeriksaan kompatibilitas transitif penting setelah Anda memiliki lebih dari dua versi skema untuk subjek tertentu. Jika kompatibilitas dikonfigurasi sebagai transitif, maka ia memeriksa kompatibilitas skema baru terhadap semua skema yang terdaftar sebelumnya; jika tidak, ia memeriksa kompatibilitas skema baru hanya terhadap skema terbaru.

Misalnya, jika ada tiga skema untuk subjek yang berubah dalam urutan X-2 , X-1 , dan X maka:

transitif: memastikan kompatibilitas antara X-2 <==> X-1 dan X-1 <==> X dan X-2 <==> X
non-transitif: memastikan kompatibilitas antara X-2 <==> X-1 dan X-1 <==> X , tetapi tidak harus X-2 <==> X

Lihat [contoh perubahan skema](https://github.com/confluentinc/schema-registry/issues/209) yang kompatibel secara bertahap, tetapi tidak kompatibel secara transitif.

Tipe kompatibilitas default Confluent Schema Registry BACKWARDbersifat nontransitif, yang berarti tidak BACKWARD_TRANSITIVE. Akibatnya, skema baru diperiksa kompatibilitasnya hanya terhadap skema terbaru.

# Order of upgrading clients

Tipe kompatibilitas yang dikonfigurasi memiliki implikasi pada urutan pemutakhiran aplikasi klien, yaitu, produsen menggunakan skema untuk menulis peristiwa ke Kafka dan konsumen menggunakan skema untuk membaca peristiwa dari Kafka. Bergantung pada tipe kompatibilitas:

BACKWARD atau BACKWARD_TRANSITIVE: tidak ada jaminan bahwa konsumen yang menggunakan skema lama dapat membaca data yang dihasilkan menggunakan skema baru. Oleh karena itu, tingkatkan semua konsumen sebelum Anda mulai menghasilkan peristiwa baru.

FORWARD atau FORWARD_TRANSITIVE: tidak ada jaminan bahwa konsumen yang menggunakan skema baru dapat membaca data yang dihasilkan menggunakan skema lama. Oleh karena itu, pertama-tama tingkatkan semua produsen untuk menggunakan skema baru dan pastikan data yang telah dihasilkan menggunakan skema lama tidak tersedia bagi konsumen, lalu tingkatkan konsumen.

FULL atau FULL_TRANSITIVE: ada jaminan bahwa konsumen yang menggunakan skema lama dapat membaca data yang dihasilkan menggunakan skema baru dan bahwa konsumen yang menggunakan skema baru dapat membaca data yang dihasilkan menggunakan skema lama. Oleh karena itu, Anda dapat memutakhirkan produsen dan konsumen secara mandiri.

NONE: pemeriksaan kompatibilitas dinonaktifkan. Oleh karena itu, Anda perlu berhati-hati tentang kapan harus memutakhirkan klien.

