---
title: Mongodb/Hadoop/Hive 과제 전 설정
updated: 2018-11-18 18:00
---

##### 국민대학교 이경용 교수님이 내주신 과제


### GCP Dataproc / VM Instance
데이터프록 만들기.

![dataproc]( {{ site.baseurl }}/assets/img/mongohadoophive_exercise/dataproc.png)
![instance]( {{ site.baseurl }}/assets/img/mongohadoophive_exercise/instance.png)


<div class="divider"></div>


### gcloud

https://cloud.google.com/sdk/docs/quickstart-macos

$ cd ~/Downloads/
$ ./google-cloud-sdk/install.sh
$ gcloud init

이하 연동 및 ssh 접속하는거 [여기]([https://www.cyberciti.biz/faq/google-cloud-compute-engin-ssh-into-an-instance-from-linux-unix-appleosx/){:target="_blank} 참고

$ gcloud compute ssh cluster-us-west-1-m

<div class="divider"></div>

### MapReduce 


#### 실습자료
$ wget https://s3.*********.amazonaws.com/******/somedata.zip 학교 s3에서 받아오기(주소 빅데이터hw  슬라이드 참고)
$ unzip somedata.zip
$ rm -f somedata.zip
$ head somedata/movies.csv
$ hdfs dfs -mkdir -p /dataset/movielens
$ hdfs dfs -put somedata/* /dataset/movielens/


#### docker
$ curl -fsSL https://get.docker.com/ | sudo sh
$ sudo usermod -aG docker ${USER}
$ exit

후 다시 실행

$ docker run -d --name mongodb -p 27017:27017 mongo  (hadoop에서 mongodb 불러오고 쓰기위해 -p 옵션 설정)

$ docker cp somedata/movies.csv mongodb:/tmp/
$ docker cp somedata/tags.csv mongodb:/tmp/

$ docker exec -it mongodb /bin/bash

# mongoimport -d movielens -c tags --type csv --file /tmp/tags.csv --headerline
# mongoimport -d movielens -c movies --type csv --file /tmp/movies.csv --headerline

#### mongo-hadoop 라이브러리(worker 노드에서도 해야됨)

$ curl http://repo1.maven.org/maven2/org/mongodb/mongo-hadoop/mongo-hadoop-core/2.0.2/mongo-hadoop-core-2.0.2.jar > mongo-hadoop-core-2.0.2.jar

$ curl http://central.maven.org/maven2/org/mongodb/mongo-java-driver/3.8.2/mongo-java-driver-3.8.2.jar > mongo-java.driver-3.8.2.jar

$ sudo mv mongo-hadoop-core-2.0.2.jar mongo-java.driver-3.8.2.jar /usr/lib/hadoop-mapreduce/


#### 실행(과제 한것 중 하나 실행코드)

$ mkdir most_tagged
$ javac -classpath $HADOOP_CLASSPATH -d most_tagged MostTagged.java
$ jar -cvf MostTagged.jar -C most_tagged/ .
$ hadoop jar MostTagged.jar MostTagged 10.168.0.2/movielens.tags 10.168.0.2/movielens.most_tagged

