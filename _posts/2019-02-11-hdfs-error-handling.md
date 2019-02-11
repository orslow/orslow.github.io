---
title: HDFS 에러났을 때 밀고 다시 시작하는 법
updated: 2019-02-11 17:10
---

### 에러내용

```
ls: Call From d45a7e62f02f/10.30.40.15 to master:9000 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
```

이유는 모르겠다.. 암튼 Connection refused.

```sh
stop-all.sh

# master container의 name 폴더 지우고 생성(나는 $HADOOP_HOME/dfs/name)
rm -rf $HADOOP_HOME/dfs/name
mkdir $HADOOP_HOME/dfs/name

# slave container들에게 접속해서 data 폴더 지우고 생성(나는 $HADOOP_HOME/dfs/data)
rm -rf $HADOO_HOME/dfs/data
mkdir $HADOO_HOME/dfs/data

# master 에서 네임노드 포멧
hdfs namenode -format

start-all.sh

jps

4004 SecondaryNameNode
4453 ResourceManager
3704 NameNode
15642 Jps

# namenode 켜졌으면 완료
```

hdfs 내에 있는 데이터 다 지워지니까 조심해야겠다.
데이터 살리면서 복구하는 방법이 결국에는 필요 할 것 같다.
