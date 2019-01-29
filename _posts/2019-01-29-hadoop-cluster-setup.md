---
title: hadoop 3.2.0 cluster setup
updated: 2019-01-29 14:37
---

#### 연구실에서 만들어놓은 도커 이미지로 구성해서 쓰니까 내부를 전혀 모르겠어서 직접 해보기로 했다.

GCP에서 ubuntu 16.04 instance 네 개(1master, 3workers) 만듦.
hadoop 3.2.0 버전으로 진행.

<div class="divider"></div>

### 하둡 유저 추가(모든 인스턴스에)

첫 접속은 console.cloud.google.com에서 SSH로.
나는 모든 유저 이름 eon으로 통일.

```sh
sudo adduser eon
# 비번설정 후 enter 여러번 누르기

sudo vi /etc/sudoers

# 아래 내용 추가
eon     ALL=(ALL:ALL) ALL

su - eon
# enter password

mkdir ~/.ssh
vi ~/.ssh/authorized_keys
# 로컬에서 만든 id_rsa.pub 추가 
```

#### 로컬에서(OS X)
```sh
vi ~/.bash_profile

# 편하게 접속하기 위해 alias 만들어놓기(아래는 내가 임의로 지어낸 IP주소)
alias master="ssh eon@100.13.1.1"
alias worker1="ssh eon@100.13.1.2"
alias worker2="ssh eon@100.13.1.3"
alias worker3="ssh eon@100.13.1.4"
```

<div class="divider"></div>

### 자바 설치(모든 인스턴스에)

``sh
#!/bin/bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:webupd8team/java -y
sudo apt-get update
echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | sudo debconf-set-selections
sudo apt-get install oracle-java8-installer -y
sudo add-apt-repository -r ppa:webupd8team/java -y
```

<div class="divider"></div>

### /etc/hosts 에 master/workers 등록하기(master에서만!)

```sh
sudo vi /etc/hosts
```

master의 ip주소와 worker들의 주소 write
```
127.0.0.1 localhost master

#100.13.1.1 master
100.13.1.2 worker1
100.13.1.3 worker2
100.13.1.4 worker3
```

- namenode는 core-site.xml에서 지정한대로 가고 secondary namenode는 default로 instance 이름으로 따라가서 연결됨.

- 100.13.1.1 을 master 로 할당해주면 연결이 안되고 127.0.0.1에 묶어야 되던데.. ssh 접속은 되는데 100.13.1.1로 하면 왜 안될까?

- internal IP / external IP? 둘다 접속은 되던데 정확한 차이는 뭘까?

<div class="divider"></div>

### ssh접속을 위해 master에서 키 생성 후 각 worker 에게 추가 해주기

ssh-copy-id 명령어로 등록 할 수 있는 것 같은데.. 잘 안돼서 직접 각 worker들의 ~/.ssh/authorized_keys 에 붙여넣음

```sh
ssh-keygen -t rsa -P ""

# master의 authorized_keys 에도 등록해줘야 함!
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

cat ~/.ssh/id_rsa.pub
# copy해서 각 worker들의 ~/.ssh/authorized_keys 에 붙여넣기

# master, 각 worker 접속되면 잘 된 것
ssh master
exit
ssh worker1
exit
ssh worker2
exit
ssh worker3
exit
```

master에서 접속중인 계정이름과 worker의 authorized_keys에 등록한 계정 이름이 다르면 {dest_hostname}@worker1 이런식으로 해야 접속이 되는데 굳이 다르게 해야 할 이유 없다면 같게 하는게 좋은 것 같다(나는 eon으로 모두 같게 진행)

<div class="divider"></div>

### hadoop 설치/설정

#### 설치(master에서만!)

```sh
cd
wget http://mirror.apache-kr.org/hadoop/common/hadoop-3.2.0/hadoop-3.2.0.tar.gz
# 링크 깨졌으면 https://hadoop.apache.org/releases.html 가보기

tar -zxf hadoop-3.2.0.tar.gz

mv hadoop-3.2.0 hadoop

# 어디 두든 상관없음 나는 /usr/local/ 에
mv hadoop /usr/local

vi ~/.bashrc

# 아래내용 추가
export HADOOP_HOME=/usr/local/hadoop
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

source ~/.bashrc

# 폴더에 권한 주기
chown -R eon /usr/local/hadoop/
```

#### 설정 후 worker들에게 나눠주기


```sh
cd $HADOOP_HOME/etc/hadoop/
vi hadoop-env.sh

# JAVA_HOME 잡아주기
export JAVA_HOME=/usr/lib/jvm/java-8-oracle

vi core-site.xml

# Namenode URI 잡아주기
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
    </property>
</configuration>

vi hdfs-site.xml

# worker node로 hadoop폴더 통째로 복사해줄거라 일단 data폴더도 만들기
mkdir /usr/local/hadoop/dfs
mkdir /usr/local/hadoop/dfs/name
mkdir /usr/local/hadoop/dfs/data

# Namenode 가 저장할 경로 지정. 위와 마찬가지로 worker 노드에서 필요한 데이터 경로도 지정.
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/usr/local/hadoop/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/usr/local/hadoop/dfs/data</value>
    </property>
</configuration>

# /usr/local 은 권한이 없어서 worker의 eon 폴더로 복사 후 이동
scp -r /usr/local/hadoop worker1:/home/eon
scp -r /usr/local/hadoop worker2:/home/eon
scp -r /usr/local/hadoop worker3:/home/eon

ssh worker1
mv hadoop /usr/local/
exit
ssh worker2
mv hadoop /usr/local/
exit
ssh worker3
mv hadoop /usr/local/
exit

# worker들에서는 chown 안해줘도 hadoop폴더 r/w 가능하던데 권한까지 옮겨지는건지?

```

#### 해보기

```sh
# 시작해보기
start-dfs.sh

jps
# SecondaryNameNode
# NameNode
# Jps

ssh worker1
jps
# DataNode
# Jps

exit
```

<div class="divider"></div>


[참고 사이트](https://hadoop.apache.org/docs/r3.2.0/hadoop-project-dist/hadoop-common/ClusterSetup.html){:target="_blank"}

In general, it is recommended that HDFS and YARN run as separate users. 라고 써져있는데 나눠써야하나?

도커에서 만들어서 해보기
