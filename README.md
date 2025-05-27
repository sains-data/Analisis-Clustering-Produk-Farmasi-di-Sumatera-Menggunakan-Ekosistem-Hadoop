1. Import data dan pilih fitur yang digunakan untuk klaster
```scla
import org.apache.spark.sql.functions._
import org.apache.spark.ml.feature.{VectorAssembler, StandardScaler}
import org.apache.spark.ml.clustering.KMeans

val productAgg = df.groupBy("product_id")
  .agg(
    count("*").alias("jumlah_transaksi"),
    avg("price_y").alias("avg_price_y"),
    avg("discount_percentage").alias("avg_discount_percentage"),
    avg("rating_y").alias("avg_rating_y"),
    avg("rating_x").alias("avg_rating_x")
  )
```
2. Vektorisasi
```scala
2. Vektorisasi
val featureCols = Array("jumlah_transaksi", "avg_price_y", "avg_discount_percentage", "avg_rating_y", "avg_rating_x")
val assembler = new VectorAssembler().setInputCols(featureCols).setOutputCol("features")
val featuredData = assembler.transform(productAgg)
```
3. Standarisasi
```scala

val scaler = new StandardScaler().setInputCol("features").setOutputCol("scaledFeatures").setWithStd(true).setWithMean(true)
val scalerModel = scaler.fit(featuredData)
val scaledData = scalerModel.transform(featuredData)
```
4. Melihat klaster terbaik berdasarkan index silhoute
```scala
import org.apache.spark.ml.clustering.KMeans
import org.apache.spark.ml.evaluation.ClusteringEvaluator

val kRange = 2 to 10

// Prepare fitur vector seperti yang sudah kamu lakukan sebelumnya
val featureCols = Array("avg_price_x", "avg_discount_percentage", "avg_rating_x")
val assembler = new VectorAssembler()
  .setInputCols(featureCols)
  .setOutputCol("features")

val featureData = assembler.transform(cityFeatures)  // cityFeatures adalah DataFrame hasil groupBy + agg

val scaler = new StandardScaler()
  .setInputCol("features")
  .setOutputCol("scaledFeatures")
  .setWithStd(true)
  .setWithMean(true)

val scalerModel = scaler.fit(featureData)
val scaledData = scalerModel.transform(featureData)

val evaluator = new ClusteringEvaluator()
  .setFeaturesCol("scaledFeatures")
  .setPredictionCol("prediction")
  .setMetricName("silhouette")

val wssseList = scala.collection.mutable.ArrayBuffer.empty[(Int, Double)]
val silhouetteList = scala.collection.mutable.ArrayBuffer.empty[(Int, Double)]

for (k <- kRange) {
  val kmeans = new KMeans()
    .setK(k)
    .setSeed(1)
    .setFeaturesCol("scaledFeatures")
    .setPredictionCol("prediction")

  val model = kmeans.fit(scaledData)
  val predictions = model.transform(scaledData)

  // WSSSE (within set sum of squared errors)
  val wssse = model.summary.trainingCost
  wssseList += ((k, wssse))

  // Silhouette score
  val silhouette = evaluator.evaluate(predictions)
  silhouetteList += ((k, silhouette))
}

// Tampilkan hasil
println("k \t WSSSE \t\t Silhouette Score")
for(i <- kRange.indices) {
  println(s"${kRange(i)} \t ${wssseList(i)._2} \t ${silhouetteList(i)._2}")
}

```
5. KMeans Clustering
```scala
val kmeans = new KMeans().setK(3).setFeaturesCol("scaledFeatures").setPredictionCol("cluster")
val model = kmeans.fit(scaledData)
val clusteredData = model.transform(scaledData)
```
6. Tampilkan hasil cluster
```scala
clusteredData.select("product_id", "jumlah_transaksi", "cluster").show(false)
```
Menyimpan hasil clusteredProv ke file Parquet
```scala
clusteredProv
  .select("provinsi", "cluster", "avg_price_x", "avg_discount_percentage", "avg_rating_x")
  .write
  .mode("overwrite")
  .parquet("/opt/cluster_provinsi_parquet/")
```
