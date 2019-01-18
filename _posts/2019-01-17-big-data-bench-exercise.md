---
title: BigdataBench 쓰는 과정 정리
updated: 2019-01-17 11:30
---

### Docker

```sh
curl -fsSL https://get.docker.com/ | sudo sh
sudo usermod -aG docker ${USER}
exit
# ssh login 다시
docker pull kmubigdata/ubuntu-hadoop
```

[컨테이너 만들고 설정](https://github.com/kmu-bigdata/ubuntu-hadoop#1-overlay-network-만드는-방법){:target="_blank"}

-> 가져다 썼지만 도커로 하둡 클러스터 구성하는거 공부해야 됨.

### BigdataBench

```sh
docker exec -it master /bin/bash
apt-get update
apt-get install wget
# 3.1
# wget http://prof.ict.ac.cn/bdb_uploads/bdb_3_1/packages/BigDataBench_V3.1_Hadoop.tar.gz
# 4.0
wget http://prof.ict.ac.cn/bdb_uploads/bdb_4/packages/BigDataBench_V4.0_Hadoop.tar.gz
tar -zxvf BigDataBench_V4.0_Hadoop.tar.gz
cd BigDataBench_V4.0_Hadoop
vi conf.properties
# BigdataBench_HOME 경로 잡아주기
vi prepar.sh
# BigdataBench_Home -> BigdataBench_HOME 수정
./prepar.sh
# Sort 하러가기
cd MicroBenchmark/OfflineAnalytics/Sort/
vi genData_Sort.sh
# 3, 4번쨰 줄 경로 설정해주고(-rm -r도 수정) jar파일 경로 확인.  mapreduce.randomwriter.bytespermap 부분 바꾸고싶으면 바꾸기
vi run_Sort.sh
# 4번째 줄 파일 input경로 output경로 설정해주기

ldconfig
```

진행하면서 내용 추가/수정

