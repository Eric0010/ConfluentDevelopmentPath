# Partisi

# Memutuskan Berapa Angka yang tepat untuk Partisi

Sebagai informasi, Arsitek Solusi di Confluent secara anekdot mengamati bahwa 30 partisi sudah cukup untuk menskalakan sebagian besar aplikasi. Angka 30 dapat dibagi dengan 2, 3, 5, 6, 10, 15, dan 30, jadi ada banyak pilihan untuk menskalakan jumlah konsumen juga. Biaya overhead untuk partisi dapat diabaikan hingga ada lebih dari 4000 per broker atau 200.000 per kluster, jadi "pembagian berlebih" saat topik dibuat adalah tindakan yang direkomendasikan.

Jumlah partisi yang disarankan: maks (t/p, t/c)
• t: target throughput
p: Throughput Produsen per Partisi
• c: Throughput Konsumen per Partisi

Variasikan parameter Produsen:
• Faktor replikasi
• Ukuran pesan
• Permintaan dalam penerbangan per koneksi
(maks.permintaan.dalam.penerbangan.per.koneksi)
• Ukuran batch (ukuran.batch)
• Waktu tunggu batch (linger.ms)

Variasikan parameter Konsumen:
• Ukuran pengambilan (min.fetch.bytes)
• Waktu tunggu pengambilan (max.wait.ms)

Panduan diatas adalah panduan untuk menetapkan jumlah partisi untuk topik jika desain memerlukan target throughput tertentu, tetapi ini bukanlah jawaban yang lengkap. Hal penting yang perlu diingat adalah bahwa pengujian kinerja diperlukan untuk membuat rencana yang baik untuk jumlah partisi.

Faktor pembatasnya kemungkinan besar adalah konsumen, jadi jumlah partisi kemungkinan besar akan bersifat t/c. Topik harus berukuran sedemikian rupa sehingga Konsumen dapat mengimbangi throughput dari sudut pandang fisik (kecepatan NIC) dan komputasi (waktu pemrosesan per poling).

# Memutuskan Berapa Angka yang tepat untuk Consumer

Bagaimana jumlah partisi topik memengaruhi skalabilitas grup konsumen?

Mengapa semua topik harus memiliki jumlah partisi yang sangat dapat dibagi?

Dengan strategi penugasan partisi default (RangeAssignor), apa yang akan terjadi jika grup konsumen yang terdiri dari 100 konsumen berlangganan 100 topik, setiap topik dengan 1 partisi?

Skalabilitas grup konsumen dibatasi oleh jumlah partisi, dan dengan strategi penugasan partisi default (RangeAssignor), berlangganan beberapa topik juga memiliki pertimbangan lain.

Pertimbangkan 1 topik sejenak. Karena cara partisi ditetapkan, topik sebaiknya memiliki sejumlah partisi yang sangat dapat dibagi, sehingga grup konsumen dapat diskalakan secara efektif ke banyak konsumen sambil mempertahankan beban yang seimbang. Dengan 30 partisi, misalnya, grup konsumen dapat diskalakan ke 1, 2, 3, 5, 6, 10, 15, atau 30 konsumen dengan beban yang seimbang.

Jika grup konsumen berlangganan beberapa topik dengan strategi penugasan partisi default, maka topik-topik ini mungkin memiliki jumlah partisi yang berbeda, sehingga menjadi lebih penting bahwa semua topik memiliki jumlah partisi yang sangat dapat dibagi. Jika satu topik memiliki 30 partisi, dan yang lain memiliki 24, maka grup konsumen dapat mengonsumsi dari kedua topik dengan beban seimbang untuk 1, 2, 3, atau 6 konsumen.

Jika grup konsumen yang terdiri dari 100 konsumen berlangganan 100 topik, masing-masing dengan 1 partisi, maka semua 100 partisi akan ditetapkan ke konsumen pertama dan akan ada 99 konsumen yang menganggur. RangeAssignor menetapkan partisi topik demi topik, masing-masing dengan cara yang persis sama. Ini adalah pertanyaan penting untuk dibahas karena mengungkap kesalahpahaman umum yang menyebabkan orang memasukkan lebih banyak konsumen dalam grup konsumen dan tidak memahami mengapa kelambatan konsumen terus bertambah.
