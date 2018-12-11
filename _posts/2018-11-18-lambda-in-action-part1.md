---
title: AWS Lambda 인 액션 Part 1
updated: 2018-11-18 02:48
---

#### greetingsOnDemand (Lambda 함수코드)
```javascript
console.log('Loading function');

exports.handler = (event, context, callback) => {
    console.log('Received event: ',
    JSON.stringify(event, null, 2));
    console.log('name =', event.name);
    var name ='';
    if ('name' in event) {
        name = event['name'];
    } else {
        name = 'World';
    }
    var greetings = 'Hello ' + name + '!';
    console.log(greetings);
    callback(null, greetings);
}
```

##### 코드 작성 후 하단에서 역할 정하기

Execution role -> Create a new role from one or more templates -> myBasicExecution

생성 후 상단에 json 형식으로 값(name)을 넣고 테스트 해볼 수 있다(log도 확인).


<div class="divider"></div>


#### *AWS CLI 설치(난 OS X 상에서 진행 ([https://aws.amazon.com/cli](https://aws.amazon.com/cli){:target="_blank"} 참고))

```sh
$ curl -O https://bootstrap.pypa.io/get-pip.py
$ python get-pip.py --user
$ pip install awscli --upgrade --user
$ echo 'alias aws=" ~/Library/Python/2.7/bin/aws"' >> ~/.bash_profile
# aws 깔린 위치 잘 찾아서 하기
# 터미널 다시 시작
$ aws --version
$ aws configure
# AWS IAM에서 만들고 받아서 치면 됨( 위 사이트에 자세히 적혀있음
```


<div class="divider"></div>

#### [AWS CLI 구성(링크)](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-chap-getting-started.html?shortFooter=true){:target="_blank"}


#### Policy 넣어주기
![Policies]( {{ site.baseurl }}/assets/img/lambda-in-action-part1/policies.png)

```sh
$ aws lambda invoke --function-name greetingsOnDemand --payload '{"name":"John"}' output.txt

$ aws lambda invoke --function-name greetingsOnDemand --payload '{}' output.txt

$ aws lambda invoke help
```


#### customGreetingsOnDemand (Lambda 함수코드)
```javascript
console.log('Loading function');

exports.handler = (event, context, callback) => {
    console.log('Received event: ',
        JSON.stringify(event, null, 2));
    console.log('greet = ', event.greet);
    console.log('name = ', event.name);
    var greet = '';
    if ('greet' in event) {
        greet = event.greet;
    } else {
        greet = 'Hello';
    }
    var name = '';
    if( 'name' in event) {
        name = event.name;
    } else {
        name = 'World';
    }
    var greetings = greet + ' ' + name + '!';
    console.log(greetings);
    callback(null, greetings);
}
```

### Lambda 함수 API Gateway 통합 

| 리소스 | HTTP 동사  | AWS Lambda를 사용한 메소드   |
| ------ |-------------| -----  |
| /greeting | GET| greetingsOnDemand |

Amazon API Gateway -> Create API -> 새 API, API이름 'My Utilities'

리소스(/greeting) 생성 -> 메소드(GET) 생성 -> GET 설정에서 Lambda 함수 연동(greetingsOnDemand)

![Method Flow]( {{ site.baseurl }}/assets/img/lambda-in-action-part1/flow.png)

Method Request -> URL Query String Parameters에 name 추가
Integration Request -> 하단 Mapping Templates -> When there are no templates defined (recommended) ->
Add mapping template -> 'application/json' 작성 후 클릭

```
#set($name = $input.params('name'))

{
#if($name != "")
    "name": "$name"
#end
}
```
작성 후 테스트 진행(name이 빈 문자열일때도 처리)


## 응답 변환하기
Integration Response -> 펼치기 -> Mapping Templates -> 'application/json'

```
{ "greeting": "$input.path('$')" }
```

## Deploy

Actions -> Deploy API -> New Stage -> Name: Prod ... -> Deploy

curl https://{my endpoint}/prod/greeting

curl https://{my endpoint}/prod/greeting?name=John


## * name을 resource 경로로 처리하기

![resource by path]( {{ site.baseurl}}/assets/img/lambda-in-action-part1/resource_by_path.png)

curl https://{my endpoint}/prod/user/John/greet

<div class="divider"></div>

### IP주소 따내는거 만들기

#### Lambda 함수(만들기)

```javascript
exports.handler = (event, context, callback) => {
    callback(null, event.myip);
}
```

#### API Gateway 측 Method Execution

![get_ip_gatewayside]( {{ site.baseurl}}/assets/img/lambda-in-action-part1/get_ip_gatewayside.png)


curl https://{my endpoint}/prod/my-ip

<div class="divider"></div>

### 연습

curl https://{my endpoint}/prod/user/{username}/say/{greeting} 하면

{username} {greeting}! response로 나오도록 만들기

```
#set($name = $input.params('username'))
#set($greet = $input.params('greet'))
{
#if($greet != "")
    "greet": "$greet"
    #if($name != "")
    ,
    #end
#end

#if($name != "")
    "name": "$name"
#end


}
```

이 언어 문법 아직 이해가 잘 되지 않는다.
