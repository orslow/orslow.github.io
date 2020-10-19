---
title: Setting Zeppelin Using Docker
updated: 2020-10-19 20:50
---

Spark 관련해서 가벼운 작업들 테스트, 결과 확인 가볍게 하고싶을 때 Zeppelin 컨테이너 하나 만들어두니까 편합니다.

Spark 외에도 다양한 Interpreter를 지원합니다. 저는 Spark를 쓰기위해 사용합니다.

Docker를 사용하면 환경 만드는 것이 간단합니다.

Docker는 설치되어 있다고 가정하고 진행합니다. [Docker installation](https://docs.docker.com/engine/install/){:target="_blank"}

<div class="divider"></div>


### Zeppelin 이미지 당겨오기

Tag가 latest면 오류가 나서 직접 붙여줍니다. (현재 0.9.0)

```sh
docker pull apache/zeppelin:0.9.0
```

![pull_image]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/pull_image.png)

<div class="divider"></div>


### 컨테이너 만들기

이미지를 사용해 컨테이너를 만듭니다. 포트 매핑을 해줘야하고 볼륨 옵션(-v)은 안써도 됩니다. 저는 함께 쓰는 서버에서 컨테이너가 갑자기 사라지거나 하는 경우 때문에 백업용으로 습관적으로 쓰려고 하고있습니다.

포트 번호는 웬만하면 8080은 피해서 매핑을 해줍니다(보안상).

`docker run -p <port_number>:8080 -itd --name <container_name> -v <local_backup_folder>:/zeppelin/notebook apache/zeppelin:0.9.0`

아래는 작성 예시입니다. 18080으로 매핑했고 컨테이너 이름이 my-zeppelin-container라 가정하고 진행하겠습니다.

```sh
docker run -p 18080:8080 -itd --name my-zeppelin-container -v /home/ubuntu/container_backup/zeppelin:/zeppelin/notebook apache/zeppelin:0.9.0

docker ps
```

![run_container]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/run_container.png)

<div class="divider"></div>


### 18080포트에 접속해서 Zeppelin 확인하기

컨테이너를 만든 머신의 IP에 포트번호 18080을 붙여 접속해봅니다.

![zeppelin_hello]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/zeppelin_hello.png)

잘 접속이 됩니다. 하지만 IP주소와 포트가 뚫리면 익명의 아무나(anonymous) 노트북에 손을 댈 수 있으니 보안 설정을 해줘야 합니다.

<div class="divider"></div>


### 컨테이너에 접속해서 설정파일 세팅하기

컨테이너에 접속해서 설정파일을 만져줘야 합니다.

```sh
docker exec -ti my-zeppelin-container bash # 컨테이너에 접속

ls
```

![ls_home]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/ls_home.png)

폴더 중 conf 폴더에서 뭔가를 만져주면 될 것 같습니다.

```sh
cd conf/

ls
```

![ls_conf]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/ls_conf.png)


Zeppelin은 Apachie Shiro라는 framework를 사용해 보안을 관리한다고 합니다.

shiro 설정파일 템플릿을 사용해서 설정을 합시다.

```sh
cp shiro.ini.template shiro.ini

vi shiro.ini
```

빔을 사용해 수정을 해야하는데 빔이 없습니다. 설치를 해야합니다.

```sh
sudo apt install vim -y
```

sudo 명령어가 없다고 나옵니다.

```sh
su -
```

zeppelin 유저의 비밀번호를 모릅니다.


```sh
exit
```

컨테이너에서 빠져 나온 후에 컨테이너에 root 유저로 접속합니다.

```sh
docker exec -ti -u root my-zeppelin-container bash
```

빔을 설치합니다.

```sh
apt update && apt upgrade -y
apt install vim -y
```

shiro.ini 파일을 열어봅니다.

```sh
vi conf/shiro.ini
```

18번째 줄에 보면 친절하게 설명이 나와있습니다. 

![shiro_user_1]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/shiro_user_1.png)

\<username\> = \<password\>, \<roleName1\>, \<roleName2\>, ... 형식으로 작성하면 됩니다.

저는 나머지 유저들은 주석처리하고 비밀번호가 0000이고 역할이 admin인 jueon 유저를 생성하겠습니다.

![shiro_user_2]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/shiro_user_2.png)


유저를 작성한 후 아래로 내려 93번째 줄을 보면 role을 설정할 수 있도록 되어있습니다. AWS에서의 role과 비슷하게 보면 될 것 같습니다.

![shiro_role_1]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/shiro_role_1.png)

디테일하게 role을 나누는 부분은 좀 더 공부를 해야할 것 같습니다. [Shiro INI configuration](https://shiro.apache.org/configuration.html#ini-sections){:target="_blank"}

저는 admin role만 사용할 것이기 때문에 나머지는 주석처리 하겠습니다.

![shiro_role_2]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/shiro_role_2.png)

여기까지 수정하고 저장 후 나옵니다(:wq).

<div class="divider"></div>


### 컨테이너 재시작 후 변경사항 확인

변경사항들을 적용하기 위해 컨테이너를 재시작합니다.

```sh
exit # 컨테이너에서 나오기
docker restart my-zeppelin-container bash
```

다시 브라우저를 열어 컨테이너를 만든 머신의 IP에 포트번호 18080을 붙여 접속해봅니다.

![zeppelin_secure_hello]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/zeppelin_secure_hello.png)

이제는 로그인을 해야 노트북에 접근할 수 있습니다.

우측 상단의 로그인 버튼을 눌러 진행합니다.

shiro.ini 파일에서 만들었던 유저 이름 jueon과 비밀번호 0000을 입력해 로그인을 합니다.

![zeppelin_secure_login]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/zeppelin_secure_login.png)

Notebook 버튼이 생겼습니다.

![zeppelin_secure_logined]( {{ site.baseurl }}/assets/img/2020-10-19-zeppelin/zeppelin_secure_logined.png)

사용하면 됩니다.
