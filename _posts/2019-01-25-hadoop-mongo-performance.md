---
title: HDFS / MongoDB Performance 관련
updated: 2019-01-24 16:10
---

### 하둡의 native library를 이용한 sort

sequential file로 만들고, sequential file 형식을 받아서 sorting함(sequence file은 hdfs dfs -text <src> 로 읽을 수 있음)

```sh
# sequence file 1GB짜리 두개 생성.
hadoop jar ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar randomwriter -D mapreduce.randomwriter.bytespermap=1073741824 -D mapreduce.randomwriter.mapsperhost=1 /data/sort/in

# sorting
hadoop jar ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort /data/sort/in /data/sort/out

# 텍스트로 가져오기(용량 크니까 tail 명령어 같은걸로 잘라서 보기)
hdfs dfs -text /data4/sort/out/* > result

# (추가) Format에 옵션을 줬을 때 돌아감. 형식은 어떻게 맞춰야 하는지 아직 모르겠다.
hadoop jar ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar sort -inFormat org.apache.hadoop.mapreduce.lib.input.TextInputFormat -outFormat org.apache.hadoop.mapreduce.lib.output.TextOutputFormat -outKey org.apache.hadoop.io.LongWritable -outValue org.apache.hadoop.io.Text /tmpya /ho/
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

[TestDFSIO 참고 사이트 1](https://medium.com/ymedialabs-innovation/hadoop-performance-evaluation-by-benchmarking-and-stress-testing-with-terasort-and-testdfsio-444b22c77db2){:target="_blank"}

[TestDFSIO 참고 사이트 2](https://community.pivotal.io/s/article/Running-DFSIO-MapReduce-benchmark-test){:target="_blank"}


### 실행 결과

![test_result]( {{ site.baseurl }}/assets/img/hadoop-commands-history/test_result.png)

-> IO performance 보는 걸로는 쓸만 할 것 같다.

<div class="divider"></div>
<div class="divider"></div>

## 직접 작성한 코드로 실행한 MapReduce 결과를 비교해보기

#### 갑자기 datanode 떨어졌었는데 해결법

```sh
# 내리기
docker exec -ti master bash 
stop-all.sh
exit

# slave container 접속해서 /usr/local/hadoop/dfs/data 지우기!
docker exec -it slave1 bash 
rm -rf /usr/local/hadoop/dfs/data/
exit

docker exec -it slave2 bash 
rm -rf /usr/local/hadoop/dfs/data/
exit

# 올리기
docker exec -ti master bash 
hadoop namenode -format
start-all.sh

# jps 명령어 이용해서 확인하기
```

#### 작업 listing/kill

```sh
# version >=2.3.0
yarn application -list
yarn application -kill $ApplicationId

# version <2.3.0
hadoop job -list
hadoop job -kill $jobId
```

#### jar 만들고 돌리기(run.sh)

```sh
#!/bin/bash

rm Sort.jar

rm -rf sort

mkdir sort

hdfs dfs -rm -r /sorted

javac -classpath $HADOOP_CLASSPATH -d sort Sort.java

jar -cvf Sort.jar -C sort/ .

hadoop jar Sort.jar Sort /unsored /sorted
```
