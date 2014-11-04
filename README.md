## MFG Commons AWS Scala API

[Scaladoc](http://mfglabs.github.io/commons-aws/api/current/)

## S3

It enhances default AWS `AmazonS3Client` for Scala :
  - asynchronous/non-blocking wrapper using an external pool of threads managed internally.
  - a few file management/streaming facilities.

> It is based on [Opensource Pellucid wrapper](https://github.com/pellucidanalytics/aws-wrap).

To use rich MFGLabs AWS S3 wrapper, you just have to add the following:


In your build.sbt (plus the classic `~/.sbt/.s3credentials`):

```scala
resolvers ++= Seq(
  "MFG releases" at "s3://mfg-mvn-repo/releases",
  "MFG snapshots" at "s3://mfg-mvn-repo/snapshots",
  "MFG thirdparty" at "s3://mfg-mvn-repo/thirdparty",
  "Pellucid public" at "http://dl.bintray.com/content/pellucid/maven"
)


libraryDependencies ++= Seq(
...
  "com.mfglabs" %% "commons-aws" % "0.1-SNAPSHOT"
...
)

```


In your code:

```scala
import com.mfglabs.commons.aws.s3
import s3._ // brings implicit extensions

// Create the client
val S3 = new AmazonS3Client()
// Use it
for {
  _   <- S3.uploadStream(bucket, "big.txt", Enumerator.fromFile(new java.io.File(s"big.txt")))
  l   <- S3.listFiles(bucket)
  _   <- S3.deleteFile(bucket, "big.txt")
  l2  <- S3.listFiles(bucket)
} yield (l, l2)
```

Please remark that you don't need any implicit `scala.concurrent.ExecutionContext` as it's directly provided
and managed by [[AmazonS3Client]] itself.

There are also smart `AmazonS3Client` constructors that can be provided with custom.
`java.util.concurrent.ExecutorService` if you want to manage your pools of threads.

## Postgres extensions

Several utils to integrate AWS services with postgresql.

Add in your build.sbt:
```scala
libraryDependencies ++= Seq(
  "com.mfglabs" %% "commons-aws-postgres" % "0.1-SNAPSHOT"
)
```

You can for instance stream a multipart file from S3 directly to a postgres table:
```scala
val pgConnection = PostgresConnectionInfo("jdbc:postgresql:metadsp", "atamborrino", "password")
val S3 = new s3.AmazonS3Client()
val pg = new PostgresExtensions(pgConnection, S3)

val s3bucket = "mfg-test"
val s3path = "report.csv" // will stream sequentially report.csv.part0001, report.csv.part0002, ...

pg.streamMultipartFileFromS3(s3bucket, s3path, dbSchema = "public", dbTableName = "test_postgres_aws_s3")
```
