https://aws.amazon.com/premiumsupport/knowledge-center/analyze-logs-athena/
How do I query Amazon Simple Storage Service (Amazon S3) server access logs in Amazon Athena?

Resolution
Amazon S3 stores server access logs as objects in an S3 bucket. You can use Athena to quickly analyze and query S3 access logs.

1.    Enable server access logging for your S3 bucket, if you haven't already. Note the values for Target bucket and Target prefix—you'll need both to specify the S3 location in an Athena query:

s3://awsexamplebucket-logs/prefix
2.    Open the Athena console.

3.    In the Query Editor, run a command similar to the following to create a database.
Note: It's a best practice to create the database in the same AWS Region as your S3 bucket.
```
create database s3_access_logs_db
```
4.    In the Query Editor, run a command similar to the following to create a table schema in the database that you created in step 3. The STRING and BIGINT data type values are the access log properties. You can query these properties in Athena. For LOCATION, enter the S3 bucket and prefix path from step 1.
```
CREATE EXTERNAL TABLE IF NOT EXISTS s3_access_logs_db.mybucket_logs(
         BucketOwner STRING,
         Bucket STRING,
         RequestDateTime STRING,
         RemoteIP STRING,
         Requester STRING,
         RequestID STRING,
         Operation STRING,
         Key STRING,
         RequestURI_operation STRING,
         RequestURI_key STRING,
         RequestURI_httpProtoversion STRING,
         HTTPstatus STRING,
         ErrorCode STRING,
         BytesSent BIGINT,
         ObjectSize BIGINT,
         TotalTime STRING,
         TurnAroundTime STRING,
         Referrer STRING,
         UserAgent STRING,
         VersionId STRING,
         HostId STRING,
         SigV STRING,
         CipherSuite STRING,
         AuthType STRING,
         EndPoint STRING,
         TLSVersion STRING
) 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
         'serialization.format' = '1', 'input.regex' = '([^ ]*) ([^ ]*) \\[(.*?)\\] ([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*) \\\"([^ ]*) ([^ ]*) (- |[^ ]*)\\\" (-|[0-9]*) ([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*) (\"[^\"]*\") ([^ ]*)(?: ([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*))?.*$' )
LOCATION 's3://awsexamplebucket-logs/prefix'
```
5.    Under Tables in the left pane, choose Preview table from the menu button that is next to the table name.

If you see data from the server access logs in the Results window (such as bucketowner, bucket, and requestdatetime), you successfully created the Athena table. You can now query the S3 server access logs.

Example queries

To find the log for a deleted object:
```
SELECT * FROM s3_access_logs_db.mybucket_logs WHERE 
key = 'images/picture.jpg' AND operation like '%DELETE%';
To show who deleted an object and when (time stamp, IP address, and AWS Identity and Access Management (IAM) user):
```
```
SELECT requestdatetime, remoteip, requester, key FROM s3_access_logs_db.mybucket_logs WHERE 
key = 'images/picture.jpg' AND operation like '%DELETE%';
```
To show all operations executed by an IAM user:
```
SELECT * FROM s3_access_logs_db.mybucket_logs WHERE 
requester='arn:aws:iam::123456789123:user/user_name';
```
To show all operations that were performed on an object in a specific time period:
```
SELECT *
FROM s3_access_logs_db.mybucket_logs
WHERE Key='prefix/images/picture.jpg' AND
parse_datetime(requestdatetime,'dd/MMM/yyyy:HH:mm:ss Z') 
BETWEEN parse_datetime('2017-02-18:07:00:00','yyyy-MM-dd:HH:mm:ss')
AND
parse_datetime('2017-02-18:08:00:00','yyyy-MM-dd:HH:mm:ss');
```
To show how much data was transferred by a specific IP address in a specific time period:
```
SELECT SUM(bytessent) as uploadtotal, 
SUM(objectsize) as downloadtotal, 
SUM(bytessent + objectsize) AS total FROM s3_access_logs_db.mybucket_logs WHERE remoteIP='1.2.3.4' and parse_datetime(requestdatetime,'dd/MMM/yyyy:HH:mm:ss Z') BETWEEN parse_datetime('2017-06-01','yyyy-MM-dd') and parse_datetime('2017-07-01','yyyy-MM-dd');
It's a best practice to create an S3 lifecycle policy for your server access logs bucket. Configure the lifecycle policy to periodically remove log files. Doing so reduces the amount of data that Athena analyzes for each query.
```
