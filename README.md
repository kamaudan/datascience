# datascience
This is a twitter streaming application using Apache spark 2.0, on Apache Zeppelin notebook.

Note the twitter streaming examples using apache spark available online were for version 1.0.... They don't work with version 2.0,  if you try  this:
import org.apache.spark.streaming.twitter._

it will output that twitter is not a member of org.apache.spark.streaming, this is because the logger that was used by version 1.0 was removed, to solve this I used apache bahir as below on zeppelin paragraph.

![screenshot from 2017-12-01 00-26-53](https://user-images.githubusercontent.com/3243281/33605032-1bfd96a0-d9c9-11e7-96a1-8c2b56a5ab3c.png)




You must configure authentication with a Twitter account.Got to twitter developer and get your api key, and other listed below. After you get API keys, you should fill out credential related values(apiKey, apiSecret, accessToken, accessTokenSecret) with your API keys on following script.

