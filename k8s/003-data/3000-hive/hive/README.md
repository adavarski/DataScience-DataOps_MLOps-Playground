# Hive overview

Apache Hive is Data Warehouse software initially developed by Facebook and later given to the Apache Software Foundation. Organizations such as
Netflix and FINRA use Hive to query massive volumes of structured data across distributed storage systems, including Hadoop’s HDFS and Amazon
S3. Hive simplifies the complex MapReduce jobs typically required for querying Big Data by providing a standard SQL interface. While Hive is not
a database, it delivers the ability to project schema onto any structured data stored in HDFS or S3-compatible storage. Amazon’s AWS offers the
product Elastic MapReduce, including a version of Hive as a service. Apache Hive enables organizations to harness enormous quantities of
structured data not managed by formal database management systems, steady streams of IoT data, exports from legacy systems, and ad hoc data
ingestion. Apache Hive reduces the complexity and effort to perform Data Science activities, including business analytics, business intelligence, and
Machine Learning, by providing an SQL interface, metadata, and schema onto a vast Data Lake.

# Hive docker image build/test)

This section creates a custom Apache Hive container configured to use MySQL for the storage of schema and metadata related to objects residing
in an S3-compatible distributed storage system, such as the MinIO cluster (configured before). Apache Hive, like many Big Data applications evolved outside the Cloud-Native and Kubernetes ecosystems, therefore requiring a bit more effort in onboarding it into the cluster. The following starts with building a custom container suitable for use with Kubernetes and local experimentation.


This repository is for the demonstration of Apache Hive utilizing a MySQL database for metadata storage, specifically for the projection of schema atop S3 object storage.

Custom Hive container build instructions (download and uncompress both Apache Hive and its main dependency Apache Hadoop and extend Apache Hive’s capabilities by adding JAR files containing the functionality needed for connecting to S3-compatible object storage and MySQL for schema and metadata management)

```shell script
# Download Apache Hive
curl -L http://mirror.cc.columbia.edu/pub/software/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz -o ./src/apache-hive-3.1.2-bin.tar.gz

# Download Apache Hadoop
curl -L http://archive.apache.org/dist/hadoop/common/hadoop-3.1.2/hadoop-3.1.2.tar.gz -o ./src/hadoop-3.1.2.tar.gz

# Uncompress both
tar -xzvf ./src/apache-hive-3.1.2-bin.tar.gz -C ./src
tar -xzvf ./src/hadoop-3.1.2.tar.gz -C ./src

# Add Jars
./deps.sh

# create container
docker build -t davarski-hive-s3m:3.1.2 .
docker tag davarski-hive-s3m:3.1.2 davarski/hive-s3m:3.1.2-1.0.0
```
Note: Hive, like many Java-based applications, uses XML files for configuration, in this case, hive-site.xml. However, packaging configuration values containing
sensitive authentication tokens, passwords, and environment-specific services locations would be an anti-pattern causing security concerns and limiting
container reusability. Mounting a configuration file from a filesystem (or ConfigMaps in the case of Kubernetes) is a standard method of configuring
containers and provides considerable flexibility for admins or developers using the container; however, this method limits the ability to leverage values
from existing Secrets and ConfigMap values available in Kubernetes. The technique described in this section creates a configuration file template to be
populated by the container with environment variables at runtime (file named hive-site-template.xml). Shell script named entrypoint.sh as the container’s initial
process. The entry point script uses sed to replace values in the hive-site.xml configuration file with values from the environment variables passed in through the container runtime, defined in the previous section. After applying the configuration, the script runs the utility schematool to add any MySQL database and tables Hive requires to store schema and metadata. Finally, the entry point script starts both a Hive server and a Hive Metastore server.


# create MinIO bucket
```
$ mc mb minio-cluster/test1
```

# local test (remote S3:MinIO)
docker-compose up

# connect to hive CLI
docker exec -it hive /opt/hive/bin/hive

# create table
```
CREATE DATABASE IF NOT EXISTS test;
CREATE EXTERNAL TABLE IF NOT EXISTS test.message (id int,message string) row format delimited fields terminated by ',' lines terminated by "\n" location 's3a://test/messages';
INSERT INTO test.message VALUES (1, "Test1");
SELECT * FROM test.message;
```
Add container to registry:
```shell script
docker login
docker push davarski/hive-s3m:3.1.2-1.0.0
```

# K8s:
```
kubectl apply -f ../10-mysql-metadata_backend.yml  
kubectl apply -f ../20-service.yml  
kubectl apply -f ../30-deployment.yml 
```
```
mc mb minio0cluster/test2
CREATE DATABASE IF NOT EXISTS test2;
CREATE EXTERNAL TABLE IF NOT EXISTS test2.message (id int,message string) row format delimited fields terminated by ',' lines terminated by "\n" location 's3a://test2/messages';
INSERT INTO test2.message VALUES (1, "Test1");
SELECT * FROM test2.message;
```

Example:

```
$ mc mb minio-cluster/test2
Bucket created successfully `minio-cluster/test2`.

$ kubectl get svc/hive -n data
NAME   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                        AGE
hive   ClusterIP   10.43.178.75   <none>        10000/TCP,9083/TCP,10002/TCP   36m

$ kubectl get pod -n data |grep hive
hive-dccc9f446-6wsg2                1/1     Running   0          36m

$ kubectl logs hive-dccc9f446-6wsg2 -n data
cp: cannot stat '/hive-site-template.xml': No such file or directory
Setting MySQL endpoint: mysql-service:3306
Setting S3 endpoint: http://minio-service:9000
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/hive/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:	 jdbc:mysql://mysql-service:3306/objectmetastore?createDatabaseIfNotExist=true&useSSL=false
Metastore Connection Driver :	 com.mysql.jdbc.Driver
Metastore connection User:	 root
Starting metastore schema initialization to 3.1.0
Initialization script hive-schema-3.1.0.mysql.sql
....
Initialization script completed
schemaTool completed
2020-12-11 08:55:12: Starting Hive Metastore Server
2020-12-11 08:55:12: Starting HiveServer2
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/hive/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/hive/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Hive Session ID = 0f7fda0a-3345-4041-89f7-c2716b4615d5


$ kubectl exec -it hive-dccc9f446-6wsg2  /opt/hive/bin/hive -n data
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/hive/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Hive Session ID = 0824f4d1-f6c7-4db6-9939-dcd8475d4da5

Logging initialized using configuration in jar:file:/opt/hive/lib/hive-common-3.1.2.jar!/hive-log4j2.properties Async: true
Hive Session ID = 806c444e-b85c-4980-bb76-9713fb1e4393
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
hive> CREATE DATABASE IF NOT EXISTS test2;
OK
Time taken: 0.76 seconds
hive> CREATE EXTERNAL TABLE IF NOT EXISTS test2.message (id int,message string) row format delimited fields terminated by ',' lines terminated by "\n" location 's3a://test2/messages';
OK
Time taken: 2.152 seconds
hive> INSERT INTO test2.message VALUES (1, "Test1");
Query ID = root_20201211092448_bc67f163-7b0f-4d93-b6a4-f5fad73c6383
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2020-12-11 09:24:53,986 Stage-1 map = 0%,  reduce = 0%
2020-12-11 09:24:54,997 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local137323622_0001
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Loading data to table test2.message
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 0 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 9.052 seconds
hive> INSERT INTO test2.message VALUES (2, "Test2");
Query ID = root_20201211092520_596cb1bf-1bdd-4381-b40f-53e6ef97128f
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2020-12-11 09:25:23,234 Stage-1 map = 0%,  reduce = 0%
2020-12-11 09:25:24,273 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local73448910_0002
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Loading data to table test2.message
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 0 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 6.661 seconds
hive> SELECT * FROM test2.message;
OK
1	Test1
2	Test2
Time taken: 0.312 seconds, Fetched: 2 row(s)
hive> exit;

$ mc ls minio-cluster/test2/messages
[2020-12-11 11:26:04 EET]     0B messages/
$ mc ls minio-cluster/test2/messages/
[2020-12-11 11:24:56 EET]     8B 000000_0
[2020-12-11 11:25:24 EET]     8B 000000_0_copy_1
$ mc cat minio-cluster/test2/messages/000000_0
1,Test1
$ mc cat minio-cluster/test2/messages/000000_0_copy_1
2,Test2


```
