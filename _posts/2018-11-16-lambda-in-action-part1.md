---
title: AWS Lambda 인 액션 Part 1
updated: 2018-11-16 05:12
---

##### greetingsOnDemand (Lambda 함수)
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

##### AWS CLI 설치(저는 OS X 상에서 진행했습니다. (aws.amazon.com/cli 참고))

$ curl -O https://bootstrap.pypa.io/get-pip.py
$ python3 get-pip.py --user
$ pip3 install awscli --upgrade --user
$ (echo 'alias aws=" ~/Library/Python/3.7/bin/aws"' >> ~/.bash_profile)
$ aws --version

##### AWS CLI 구성
https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-chap-getting-started.html?shortFooter=true

##### Policy
![Policies](https://github.com/orslow/orslow.github.io/tree/master/assets/img/policies.png)

$ aws lambda invoke --function-name greetingsOnDemand --payload '{"name":"John"}' output.txt
$ aws lambda invoke --function-name greetingsOnDemand --payload '{}' output.txt
$ aws lambda invoke help

