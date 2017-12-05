# datascience
This is a twitter streaming application using Apache spark 2.0, on Apache Zeppelin notebook.

Note the twitter streaming examples using apache spark available online were for version 1.0.... They don't work with version 2.0,  if you try  this:
import org.apache.spark.streaming.twitter._

it will output that twitter is not a member of org.apache.spark.streaming, this is because the logger that was used by version 1.0 was removed, to solve this I used apache bahir as below on zeppelin paragraph.

%dep
z.reset()
z.load("org.apache.bahir:spark-streaming-twitter_2.11:2.0.0")
z.load("org.twitter4j:twitter4j-core:3.0.3")
z.load("org.twitter4j:twitter4j-media-support:3.0.3")
z.load("org.twitter4j:twitter4j-async:3.0.3")
z.load("org.twitter4j:twitter4j-examples:3.0.3")
z.load("org.twitter4j:twitter4j-stream:3.0.3")




You must configure authentication with a Twitter account.Got to twitter developer and get your api key, and other listed below. After you get API keys, you should fill out credential related values(apiKey, apiSecret, accessToken, accessTokenSecret) with your API keys on following script.

This will create a RDD of Tweet objects and register these stream data as a table:
import org.apache.spark.streaming._
import org.apache.spark.streaming.twitter._
import org.apache.spark.storage.StorageLevel
import scala.io.Source
import scala.collection.mutable.HashMap
import java.io.File
import org.apache.log4j.Logger
import org.apache.log4j.Level
import sys.process.stringSeqToProcess

/** Configures the Oauth Credentials for accessing Twitter */

def configureTwitterCredentials(apiKey: String, apiSecret: String, accessToken: String, accessTokenSecret: String) {
  val configs = new HashMap[String, String] ++= Seq(
    "apiKey" -> apiKey, "apiSecret" -> apiSecret, "accessToken" -> accessToken, "accessTokenSecret" -> accessTokenSecret)
  println("Configuring Twitter OAuth")
  configs.foreach{ case(key, value) =>
    if (value.trim.isEmpty) {
      throw new Exception("Error setting authentication - value for " + key + " not set")
    }
    val fullKey = "twitter4j.oauth." + key.replace("api", "consumer")
    System.setProperty(fullKey, value.trim)
    println("\tProperty " + fullKey + " set as [" + value.trim + "]")
  }
  println()
}

// Configure Twitter credentials
val apiKey = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
val apiSecret = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
val accessToken = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
val accessTokenSecret = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
configureTwitterCredentials(apiKey, apiSecret, accessToken, accessTokenSecret)

import org.apache.spark.streaming.twitter._
val ssc = new StreamingContext(sc, Seconds(2))
val tweets = TwitterUtils.createStream(ssc, None)
val twt = tweets.window(Seconds(60))

case class Tweet(createdAt:Long, text:String)
twt.map(status=>
  Tweet(status.getCreatedAt().getTime()/1000, status.getText())
).foreachRDD(rdd=>
  // Below line works only in spark 1.3.0.
  // For spark 1.1.x and spark 1.2.x,
  // use rdd.registerTempTable("tweets") instead.
  
  
  
  
  
  rdd.toDF().registerAsTable("tweets")
)

twt.print

ssc.start()

You can make user-defined function and use it in Spark SQL. Let's try it by making function named sentiment. This function will return one of the three attitudes(positive, negative, neutral) towards the parameter.


def sentiment(s:String) : String = {
    val positive = Array("like", "love", "good", "great", "happy", "cool", "the", "one", "that")
    val negative = Array("hate", "bad", "stupid", "is")

    var st = 0;

    val words = s.split(" ")    
    positive.foreach(p =>
        words.foreach(w =>
            if(p==w) st = st+1
        )
    )

    negative.foreach(p=>
        words.foreach(w=>
            if(p==w) st = st-1
        )
    )
    if(st>0)
        "positivie"
    else if(st<0)
        "negative"
    else
        "neutral"
}
