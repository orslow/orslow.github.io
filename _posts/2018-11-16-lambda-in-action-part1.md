---
title: AWS Lambda 인 액션 Part 1(작성 중)
updated: 2018-11-16 05:12
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

Execution role -> Create a new role from one or more templates -> myBasicExecution

생성 후 상단에 json 형식으로 값(name)을 넣고 테스트 해볼 수 있습니다(log도 확인).
<div class="divider"></div>

#### *AWS CLI 설치(저는 OS X 상에서 진행했습니다. ([https://aws.amazon.com/cli](https://aws.amazon.com/cli){:target="_blank"} 참고))

$ curl -O https://bootstrap.pypa.io/get-pip.py

$ python3 get-pip.py --user

$ pip3 install awscli --upgrade --user

$ (echo 'alias aws=" ~/Library/Python/3.7/bin/aws"' >> ~/.bash_profile)

$ aws --version
<div class="divider"></div>

#### [AWS CLI 구성(링크)](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-chap-getting-started.html?shortFooter=true){:target="_blank"}


#### Policy 넣어주기
![Policies]( {{ site.baseurl }}/assets/img/lambda-in-action-part1/policies.png)

$ aws lambda invoke --function-name greetingsOnDemand --payload '{"name":"John"}' output.txt

$ aws lambda invoke --function-name greetingsOnDemand --payload '{}' output.txt

$ aws lambda invoke help


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

| 리소스          | HTTP 동사      | AWS Lambda를 사용한 메소드   |
| ------------- |-------------| -----              |
| /greeting      | GET| greetingsOnDemand |

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
작성 후 테스트 
