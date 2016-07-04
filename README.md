# Collaborative filtering with Spark

## Commands
1. Copy files to HDFS
 * hadoop fs -copyFromLocal lastfm-dataset-360K/usersha1-artmbid-artname-plays.tsv /user/ds
 * hadoop fs -copyFromLocal lastfm-dataset-360K/usersha1-profile.tsv /user/ds
 * run spark shell (spark-shell --driver-memory 6g)
 * val rawdata = sc.textFile("hdfs://localhost:9000/user/ds/usersha1-artmbid-artname-plays.tsv")
     * val splitData = rawdata.map{case line => line.split('\t')}.map{case Array(userId, artistId, artistAlias, listenCount) => (userId.hashCode, artistId.hashCode, artistAlias, listenCount.trim.toInt)}
 **splitData.map(_._1.toDouble).stats()
 * val artistAlias = splitData.map(tp => (tp._2, tp._3)).collectAsMap
 * sc.broadcast(artistAlias)
 * val userArtistListenCount = splitData.map(tp => (tp._1, tp._2, tp._4))
 * import org.apache.spark.mllib.recommendation._
 * val trainData = userArtistListenCount.map{case (userId, artistId, count) => Rating(userId, artistId, count)}.cache
 * val model = ALS.trainImplicit(trainData, 10, 5, 0.01, 1.0)
 * val rawArtistForUser = splitData.filter{case (user, _, _, _) => user == 1201881167}.map(_._3).foreach(println)
 * val recommendation = model.recommendProducts(1201881167, 5)
 * val recommendedProductIDs = recommendation.map(_.product).toSet
 * artistAlias.filter{case (artist, alias)=>recommendedProductIDs.contains(artist)}.map(_._2).foreach(println)

## References
1. [Last.fm dataset] (http://www.dtic.upf.edu/~ocelma/MusicRecommendationDataset/lastfm-360K.html)