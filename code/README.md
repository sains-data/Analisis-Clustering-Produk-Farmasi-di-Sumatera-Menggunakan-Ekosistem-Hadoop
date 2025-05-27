Membaca Data dan menyimpan dalam format parquet
```scala
val df = spark.read
  .option("header", "true")
  .option("inferSchema", "true")
  .csv("/opt/gabung2.csv"")  // Ganti path sesuai lokasi Anda

df.write
  .mode("overwrite")
  .parquet("/opt/keluar_parquet/")  // Ganti path sesuai lokasi tujuan
```
Import Data yang digunakan
```scala
import org.apache.spark.sql.functions._
import org.apache.spark.ml.feature.VectorAssembler
import org.apache.spark.ml.feature.StandardScaler
import org.apache.spark.ml.clustering.KMeans
```
1. Segmentasi Produk Berdasarkan Karakteristik Harga, Diskon, dan Rating
```scala
// 1. Agregasi data per product_id: hitung rata-rata setiap fitur
val productFeatures = df.groupBy("product_id")
  .agg(
    avg("price_y").alias("avg_price_y"),
    avg("discount_percentage").alias("avg_discount_percentage"),
    avg("rating_y").alias("avg_rating_y"),
    avg("rating_x").alias("avg_rating_x"),
    avg("price_x").alias("avg_price_x")
  )

// 2. Buat fitur vector dari kolom-kolom tersebut
val featureCols = Array("avg_price_y", "avg_discount_percentage", "avg_rating_y", "avg_rating_x", "avg_price_x")
val assembler = new VectorAssembler()
  .setInputCols(featureCols)
  .setOutputCol("features")

val featuredData = assembler.transform(productFeatures)

// 3. Standarisasi fitur agar clustering lebih optimal
val scaler = new StandardScaler()
  .setInputCol("features")
  .setOutputCol("scaledFeatures")
  .setWithStd(true)
  .setWithMean(true)

val scalerModel = scaler.fit(featuredData)
val scaledData = scalerModel.transform(featuredData)

// 4. Cek klaster terbaik
import org.apache.spark.ml.clustering.KMeans
import org.apache.spark.ml.evaluation.ClusteringEvaluator

val evaluator = new ClusteringEvaluator()
val cost = (2 to 10).map { k =>
  val kmeans = new KMeans()
    .setK(k)
    .setFeaturesCol("scaledFeatures")
    .setSeed(1L)

  val model = kmeans.fit(scaledData)
  val predictions = model.transform(scaledData)
  val silhouette = evaluator.evaluate(predictions)
  println(s"Silhouette score for k=$k: $silhouette")
  (k, silhouette)
}

// 5. Terapkan KMeans yg terbaik
val kmeans = new KMeans()
  .setK(3)
  .setFeaturesCol("scaledFeatures")
  .setPredictionCol("cluster")

val model = kmeans.fit(scaledData)
val clusteredData = model.transform(scaledData)

// 6. Tampilkan hasil cluster per produk
clusteredData.select("product_id", "cluster", featureCols: _*).show(false)

// 7. Save file
clusteredData
  .write
  .mode("overwrite")     // overwrite jika folder sudah ada
  .parquet("output/clustered_produk")
```

2. Segmentasi Pasar Provinsi Berdasarkan Perilaku Harga, Diskon, dan Kepuasan Pelanggan untuk Optimalisasi Strategi Penjualan
```scala
// 1. Import Library
import org.apache.spark.sql.functions._
import org.apache.spark.ml.feature.VectorAssembler
import org.apache.spark.ml.feature.StandardScaler
import org.apache.spark.ml.clustering.KMeans

// 2. Agregasi per provinsi
val provinsiFeatures = df.groupBy("provinsi")
  .agg(
    avg("price_x").alias("avg_price_x"),
    avg("discount_percentage").alias("avg_discount_percentage"),
    avg("rating_x").alias("avg_rating_x")
  )

// 3. Susun vektor fitur
val featureColsProvinsi = Array("avg_price_x", "avg_discount_percentage", "avg_rating_x")
val assemblerProv = new VectorAssembler()
  .setInputCols(featureColsProvinsi)
  .setOutputCol("features")

val featuredProv = assemblerProv.transform(provinsiFeatures)

// 4. Standardisasi
val scalerProv = new StandardScaler()
  .setInputCol("features")
  .setOutputCol("scaledFeatures")
  .setWithStd(true)
  .setWithMean(true)

val scalerModelProv = scalerProv.fit(featuredProv)
val scaledProv = scalerModelProv.transform(featuredProv)

// penentuan index silhoute yg cocok
import org.apache.spark.ml.evaluation.ClusteringEvaluator

// Evaluator untuk clustering
val evaluatorProv = new ClusteringEvaluator()
  .setFeaturesCol("scaledFeatures")     // gunakan fitur hasil standardisasi
  .setPredictionCol("cluster")          // kolom prediksi klaster

// Hitung Silhouette Score
val silhouetteProv = evaluatorProv.evaluate(clusteredProv)

println(s"Silhouette Score untuk clustering provinsi: $silhouetteProv")


// 5. Klasterisasi dengan KMeans yg terbaik
val kmeansProv = new KMeans()
  .setK(6)
  .setSeed(1)
  .setFeaturesCol("scaledFeatures")
  .setPredictionCol("cluster")

val modelProv = kmeansProv.fit(scaledProv)
val clusteredProv = modelProv.transform(scaledProv)

// 6. Lihat hasil klaster tiap provinsi
clusteredProv.select("provinsi", "cluster", "avg_price_x", "avg_discount_percentage", "avg_rating_x").show(false)

// Simpan hasil clustering provinsi ke file Parquet
clusteredProv
  .select(
    col("provinsi"),
    col("cluster"),
    col("avg_price_x"),
    col("avg_discount_percentage"),
    col("avg_rating_x")
  )
  .write
  .mode("overwrite")  // Timpa folder jika sudah ada
  .parquet("output/clustering_result_provinsi_parquet")

```
3. egmentasi Produk Berdasarkan Harga, Diskon, dan Rating untuk Mengidentifikasi Produk Premium dan Ekonomis"
```scala
val productFeatures = df.groupBy("product_id").agg(
  avg("price_y").alias("avg_price_y"),
  avg("discount_percentage").alias("avg_discount_percentage"),
  avg("rating_y").alias("avg_rating_y"),
  avg("rating_x").alias("avg_rating_x"),
  avg("price_x").alias("avg_price_x")
)

val featureCols = Array("avg_price_y", "avg_discount_percentage", "avg_rating_y", "avg_rating_x", "avg_price_x")

val assembler = new VectorAssembler()
  .setInputCols(featureCols)
  .setOutputCol("features")

val featuredData = assembler.transform(productFeatures)

val scaler = new StandardScaler()
  .setInputCol("features")
  .setOutputCol("scaledFeatures")
  .setWithStd(true)
  .setWithMean(true)

val scalerModel = scaler.fit(featuredData)
val scaledData = scalerModel.transform(featuredData)

val kmeans = new KMeans()
  .setK(2) 
  .setFeaturesCol("scaledFeatures")
  .setPredictionCol("cluster")

val model = kmeans.fit(scaledData)
val clusteredData = model.transform(scaledData)

clusteredData.write.mode("overwrite").parquet("/opt/clusteredData.parquet")

```
