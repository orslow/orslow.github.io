---
title: HDFS / MongoDB Performance 관련 자료들
updated: 2019-01-18 13:00
---

## BigDataBench

### Sort

```sh
${HADOOP_HOME}/bin/hadoop jar  ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort /data4/sort/in /data4/sort/out

${HADOOP_HOME}/bin/hadoop jar  ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort -inFormat org.apache.hadoop.mapreduce.lib.input.TextInputFormat


# 뭘 하는진 모르겠는데 되긴 됨
${HADOOP_HOME}/bin/hadoop jar  ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort -inFormat org.apache.hadoop.mapreduce.lib.input.TextInputFormat -outFormat org.apache.hadoop.mapreduce.lib.output.TextOutputFormat -outKey org.apache.hadoop.io.LongWritable -outValue org.apache.hadoop.io.Text /tmpya /ho/
```

<div class="divider"></div>

```sh
sort [-r <reduces>] [-inFormat <input format class>] [-outFormat <output format class>] [-outKey <output key class>] [-outValue <output value class>] [-totalOrder <pcnt> <num samples> <max splits>] <input> <ou
tput>
Generic options supported are:
-conf <configuration file>        specify an application configuration file
-D <property=value>               define a value for a given property
-fs <file:///|hdfs://namenode:port> specify default filesystem URL to use, overrides 'fs.defaultFS' property from configurations.
-jt <local|resourcemanager:port>  specify a ResourceManager
-files <file1,...>                specify a comma-separated list of files to be copied to the map reduce cluster
-libjars <jar1,...>               specify a comma-separated list of jar files to be included in the classpath
-archives <archive1,...>          specify a comma-separated list of archives to be unarchived on the compute machines
```

<div class="divider"></div>

## TestDFSIO

```sh
# https://github.com/apache/hadoop/blob/trunk/hadoop-mapreduce-project/hadoop-mapreduce-client/hadoop-mapreduce-client-jobclient/src/test/java/org/apache/hadoop/fs/TestDFSIO.java
hadoop jar /usr/local/hadoop-3.1.1/share/hadoop/mapreduce/hadoop-*tests* TestDFSIO


# https://github.com/apache/hadoop/blob/trunk/hadoop-mapreduce-project/hadoop-mapreduce-client/hadoop-mapreduce-client-jobclient/src/test/java/org/apache/hadoop/fs/DFSCIOTest.java
hadoop jar /usr/local/hadoop-3.1.1/share/hadoop/mapreduce/hadoop-*tests* DFSCIOTest

# tera sort 관련
hadoop jar /usr/local/hadoop-3.1.1/share/hadoop/mapreduce/hadoop-*examples*.jar teragen
```

<div class="divider"></div>

[TestDFSIO 참고 사이트 1](https://medium.com/ymedialabs-innovation/hadoop-performance-evaluation-by-benchmarking-and-stress-testing-with-terasort-and-testdfsio-444b22c77db2){:target="_blank"}

[TestDFSIO 참고 사이트 2](https://community.pivotal.io/s/article/Running-DFSIO-MapReduce-benchmark-test){:target="_blank"}

<div class="divider"></div>


### 잘 되는지 해보기(1 master, 2 slave)


#### 1GB * 10 Write/Read

----- TestDFSIO ----- : write
Date & time: Wed Jan 23 03:16:03 UTC 2019
Number of files: 10
Total MBytes processed: 10240
Throughput mb/sec: 77.9
Average IO rate mb/sec: 167.31
IO rate std deviation: 109.19
Test exec time sec: 61.33

----- TestDFSIO ----- : read
Date & time: Wed Jan 23 03:17:41 UTC 2019
Number of files: 10
Total MBytes processed: 10240
Throughput mb/sec: 50.78
Average IO rate mb/sec: 54.84
IO rate std deviation: 13.24
Test exec time sec: 56.75


#### 1GB * 20 Write/Read

----- TestDFSIO ----- : write
Date & time: Wed Jan 23 03:09:41 UTC 2019
Number of files: 20
Total MBytes processed: 20480
Throughput mb/sec: 40.1
Average IO rate mb/sec: 104.26
IO rate std deviation: 100.28
Test exec time sec: 166.49

----- TestDFSIO ----- : read
Date & time: Wed Jan 23 03:13:29 UTC 2019
Number of files: 20
Total MBytes processed: 20480
Throughput mb/sec: 33.47
Average IO rate mb/sec: 43.78
IO rate std deviation: 44.97
Test exec time sec: 148.32


#### 1GB * 30 Write/Read

----- TestDFSIO ----- : write
Date & time: Wed Jan 23 03:28:54 UTC 2019
Number of files: 30
Total MBytes processed: 30720
Throughput mb/sec: 35.53
Average IO rate mb/sec: 71.33
IO rate std deviation: 81.16
Test exec time sec: 272.61

<div class="divider"></div>
