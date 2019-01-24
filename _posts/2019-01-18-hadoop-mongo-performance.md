---
title: HDFS / MongoDB Performance 관련 자료들
updated: 2019-01-24 16:10
---

### 하둡의 native library를 이용한 sort

sequential file로 만들고, sequential file 형식을 받아서 sorting함(sequence file은 hdfs dfs -text <src> 로 읽을 수 있음)

```sh
{HADOOP_HOME}/bin/hadoop jar ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort /data/sort/in /data/sort/out

#{HADOOP_HOME}/bin/hadoop jar ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort -inFormat org.apache.hadoop.mapreduce.lib.input.TextInputFormat

# Format에 옵션을 줬을 때 돌아가긴 하는데 결과값이 이상함 아무튼 example
${HADOOP_HOME}/bin/hadoop jar  ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort -inFormat org.apache.hadoop.mapreduce.lib.input.TextInputFormat -outFormat org.apache.hadoop.mapreduce.lib.output.TextOutputFormat -outKey org.apache.hadoop.io.LongWritable -outValue org.apache.hadoop.io.Text /tmpya /ho/
```

<div class="divider"></div>

```sh
# sort 사용법
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
https://github.com/apache/hadoop/blob/trunk/hadoop-mapreduce-project/hadoop-mapreduce-client/hadoop-mapreduce-client-jobclient/src/test/java/org/apache/hadoop/fs/TestDFSIO.java
hadoop jar /usr/local/hadoop-3.1.1/share/hadoop/mapreduce/hadoop-*tests* TestDFSIO

https://github.com/apache/hadoop/blob/trunk/hadoop-mapreduce-project/hadoop-mapreduce-client/hadoop-mapreduce-client-jobclient/src/test/java/org/apache/hadoop/fs/DFSCIOTest.java
hadoop jar /usr/local/hadoop-3.1.1/share/hadoop/mapreduce/hadoop-*tests* DFSCIOTest

# tera sort 관련
hadoop jar /usr/local/hadoop-3.1.1/share/hadoop/mapreduce/hadoop-*examples*.jar teragen
```

[TestDFSIO 참고 사이트 1](https://medium.com/ymedialabs-innovation/hadoop-performance-evaluation-by-benchmarking-and-stress-testing-with-terasort-and-testdfsio-444b22c77db2){:target="_blank"}

[TestDFSIO 참고 사이트 2](https://community.pivotal.io/s/article/Running-DFSIO-MapReduce-benchmark-test){:target="_blank"}


### 실행 결과

![test_result]( {{ site.baseurl }}/assets/img/hadoop-commands-history/test_result.png)

<div class="divider"></div>

