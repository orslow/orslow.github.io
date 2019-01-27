---
title: 리눅스 유저 추가/제거
updated: 2019-01-27 14:17
---

#### SSH로 접속 할 수 있게 하도록 하려고 유저를 만듦

### on local

```sh
ssh-keygen -t rsa

# 적절한 폴더에 원하는 이름으로 만들기(나는 default로 설정된 경로, 파일명으로 만들었다)

cat /Users/eon/.ssh/id_rsa.pub

# 복사하기
```

<div class="divider"></div>

### on server

```sh
# sudo 권한 주기
sudo vi /etc/sudoers

# eon이라는 이름으로 만듦
adduser eon
# password만 설정하고 기억

su - eon

mkdir ~/.ssh

vi ~/.ssh/authorized_keys

# local에서 복사 한 내용 붙여넣기

# 추가 되었는지 확인
cut -d: -f1 /etc/passwd

```

<div class="divider"></div>

### 유저 삭제

```sh
userdel eon

# 유저에서 실행되고 있는 프로세스 모두 kill하고 지우기
killall -u eon

# 홈폴더 삭제 옵션 줘서 다 삭제 (필요한 파일이 있다면 미리 꺼내놓기)
deluser --remove-home -f username
```

