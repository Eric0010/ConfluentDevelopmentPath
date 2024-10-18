# Kafka Rebalancing

Rebalancing adalah Ketika beberapa Group Consumer sudah dibuat, Group coordinator akan menetapkan pasrtisi tersebut ke pada setiap konsumer di dalam group tersebut.

Setiap Consumer akan bertanggung jawab atas pengkonsumsian data dari beberapa partisi.

Bagaimana Rebalancing bekerja?

Kafka menyediakan beberapa stategi partisi untuk menghasilkan bagiamana partisi akan ditetapkan ketika proses rebalncing, ini yang dinamakann assignor.

Startegi partisi yang default adalah Round-Robin, dimana kafka akan menetapkan partisi kepada konsumer 1 sama lain.

Kafka juga menyediakan 'range' dan 'cooperative sticky' untuk beberapa strategi penugasan, yang akan berguna untuk beberapa use case khusus.

Kapan Rebalance terjadi?

Kafka akan memberitahu setiap consumer yang ada di grup dengan mengirim pesan GroupCoordinator.

Setiap Consumer akan merespon dengan pesan Join Group, mengindikasikan ketersediaan untuk berpartisipasi di rebalance.

Kafka kemudian akan menggunakan strategi penetapan partisi untuk menetapkan setiap consumer yang ada di dalam sebuah group.

Di waktu rebalnce, Kafka bisa saja melakukan pemberhentian data untuk sementara. Hal ini penting untuk meyakinkan semua consumer mempunyai data terkini mengenai penugasan partisi sebelum melakukan pengkonsumsain data kembali.

# Consumer Joins atau Leave

Ketika Consumer bergabung atau meninggalkan sebuah group, Kafka harus melakukan rebalancing kepada partisi pada consumer yang tersedia. Hal ini bisa saja terjadi ketika shutdown/restart atau aplikasi sedang mengalami peningkatan/penurunan. Detak jantung adalah salah satu contoh yang mengindikasikan, jika consumer masih beroperasi dan aktik berpartisipasi di dalam sebuah group. Jika Consumer gagal untuk mengirim 1 detak jantung pada interval khusus, maka bisa disimpulkan consumer tersebut sudah tidak aktif, dan grup koordinator harus menginisiasi rebalancing untuk melakukan penugasan kembali kepada partisi kepada consumer lain.

