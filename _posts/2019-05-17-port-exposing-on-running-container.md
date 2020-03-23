---
title: 컨테이너 다시 만들지 않고 docker container 포트 여는 법
updated: 2019-05-17 15:18
---

### 개요

1. docker container stop하기 
2. docker service stop하기(service docker stop)
3. docker config file 수정하기(config.v2.json, hostconfig.json)
4. start docker service (service docker start)
5. docker container start하기

### config file 수정

추가하려는 port number를 8080과 18080이라 가정하고 진행

```sh
docker stop [container_id]
sudo su
service docker stop
vi /var/lib/docker/containers/[container_id]<tab>/config.v2.json
```

가독성을 위해 들여쓰기로 적은것이고 파일에는 띄어쓰기 없이 json형식 쭉 작성.

```
...
"ExposedPorts": {
	{
		"8080/tcp":{},
		"18080/tcp":{}
	}
}
....
"Ports": {
	"8080/tcp": [
		{
			"HostIp": "",
			"HostPort": "8080"
		}
	],
	"18080/tcp": [
		{
			"HostIp": "",
			"HostPort": "18080"
		}
	]
}
...
```

```sh
vi /var/lib/docker/containers/[container_id]<tab>/hostconfig.json
```

```
...
"PortBindings": {
	"8080/tcp": [
		{
			"HostIp": "",
			"HostPort": "8080"
		}
	],
	"18080/tcp": [
		{
			"HostIp": "",
			"HostPort": "18080"
		}
	]
}
...
```

```sh
service docker start
docker start [container_id]
docker ps
# 포트 연결된 것 확인
```

