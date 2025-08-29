# Proyek Big Data: Analisis Clustering Produk Farmasi di Sumatera

## Deskripsi Proyek
Proyek ini bertujuan untuk menganalisis dan mengelompokkan produk farmasi yang beredar di wilayah Sumatera menggunakan ekosistem Hadoop. Sistem ini mengumpulkan, memproses, dan menganalisis data produk farmasi dengan pendekatan clustering menggunakan metode K-Means.

## Tujuan
Proyek ini bertujuan untuk membantu perusahan farmasi dalam mengelompokkan produk berdasarkan kategori wilayah, kateogri obat , dan informasi lainnya yang relevan sehingga pengelolaan ketersediaan obat yang lebih optimal dan tepa sasaran

## Dataset
Data yang digunakan merupakan data transaksi farmasi tahun *2020-2023* yang mencakup informasi produk, pelanggan, kategori, diskon, rating dan lokasi cabang. Untuk penelitian ini hanya difokuskan untuk seluruh cabang yang ada di Pulau Sumatera.

## Lingkup Sistem
Sistem ini menyimpan data dalam tiga lapisan (bronze, silver, gold) dan menyediakan hasil analitik terstruktur yang dapat digunakan oleh BI tools atau query langsung untuk mendukung rekomendasi strategis bagi pelaku industri farmasi.

## Fungsi Utama Sistem
1. Mengambil dan menyimpan data ekspor mentah (bronze).
2. Membersihkan dan mentransformasi data (silver).
3. Mengagregasi data untuk analisis (gold).
4. Menyediakan akses ke data melalui Hive dan dashboard visualisasi.


## Arsitektur Sistem


![apa (1)](https://github.com/user-attachments/assets/543a9794-a522-4789-9e06-3c11811be5e5)


Sistem dibangun berdasaran *Medalion Architectur*, yang membagi prpses pipeline menjaid tiga lapisan:
- **Bronze Layer**: Menyimpan data mentah dalam format CSV.
- **Silver Layer**: Menyimpan data yang telah dibersihkan dalam format Parquet.
- **Gold Layer**: Mengelola data menjadi klaster dan menyimpan data hasil klaster yang siap untuk analitik.

Konfigurasi cluster lokal:
- 1 NameNode
- 2 DataNode
- 1 Spark Master
- 2 Spark Worker

## Proses Analitik
Analisis clustering dilakukan dengan *K-Means Clustering* dari pustaka *Apache Spark MLlib*:

1. Membaca data dari Gold Layer (format Parquet)
2. Preprocessing: normalisasi, imputasi missing value
3. Split data ke training dan testing
4. Latih model K-Means
5. Evaluasi performa model
6. Simpan dan deploy model untuk penggunaan lebih lanjut
7. Visualisasi Hasil


## Metodologi Proyek
Proyek ini menggunakan pendekatan Waterfall, yang mencakup:

1. Analisis Kebutuhan
2. Perancangan Sistem
3. Implementasi
4. Deployment
5. Testing

## Teknologi yang Digunakan
- **Hadoop Distributed File System (HDFS)**: Penyimpanan terdistribusi.
- **Apache Spark**: Pemrosesan data batch.
- **Apache Hive**: Query SQL terhadap data besar.
- **Docker**: Lingkungan multi-container untuk pengoperasian sistem.
- **Phyton**: Tempat Pembuatan Visual hasil klaster


## Implementasi Pipeline
Pipeline batch dibangun untuk memproses data secara bertahap dari ingest hingga analitik. Proses mencakup:

1. Ingest data ke HDFS
2. Transformasi dan pembersihan dengan Spark
3. Simpan hasil ke Hive
4. Analisis clustering dengan Spark MLlib
5. Visualisasi hasil


## Pengujian
Pengujian dilakukan untuk memastikan seluruh proses pipeline berjalan dengan baik, termasuk:
- Unit Test
- Integration Test
- Data Quality Test
- Performance Test
- End-to-End Test

##👥 Tim Proyek
|           Nama          |     NIM     |
| ----------------------- | ----------- |
| Afifah Syaharani        |  121450097  |
| Ukasyah Muntaha         |  122450028  |
| Renta Siahaan           |  122450070  |
| M. Deriansyah Okutra    |  122450101  |




