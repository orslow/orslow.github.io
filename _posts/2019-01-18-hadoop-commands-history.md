---
title: Hadoop 명령어 모아놓기
updated: 2019-01-18 13:00
---

##### Should be merged with BigdataBench post


## Sort

```sh

${HADOOP_HOME}/bin/hadoop jar  ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort /data4/sort/in /data4/sort/out

${HADOOP_HOME}/bin/hadoop jar  ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort -inFormat org.apache.hadoop.mapreduce.lib.input.TextInputFormat


# 뭘 하는진 모르겠는데 되긴 됨
${HADOOP_HOME}/bin/hadoop jar  ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort -inFormat org.apache.hadoop.mapreduce.lib.input.TextInputFormat -outFormat org.apache.hadoop.mapreduce.lib.output.TextOutputFormat -outKey org.apache.hadoop.io.LongWritable -outValue org.apache.hadoop.io.Text /tmpya /ho/


```

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









hadoop jar /usr/local/hadoop-3.1.1/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.1-tests.jar TestDFSIO

성능측정해주는 TestDFSIO 들어있는 jar 파일
