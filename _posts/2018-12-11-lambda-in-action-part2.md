---
title: AWS Lambda 인 액션 Part 2
updated: 2018-12-11 19:00
---

### CH4

아마존 IAM 들어가서 역할/정책 보기 


#### 정책 작성법 예시


##### Policy_RW_S3.json

```
{
	"Version": "2012-10-17",  # 최신버전 적어두기
		"Statement": 
		{
			"Effect": "Allow", 	# Allow or Deny
				"Action": [
				"s3:ListBucket",  # Bucket 내 객체 리스팅
				"s3:GetBucketLocation"  # Bucket이 속해있는 regeion 값
				],
				"Resource": "arn:aws:s3:::BUCKET"
		},
		{
			"Effect": "Allow",
			"Action": [   # 쓰기/읽기/삭제 할 수 있도록
				"s3:PutObject",  
			"s3:GetObject",
			"s3:DeleteObject"
				],
			"Resource": "arn:aws:s3:::BUCKET/*"  # 여기서만
		}
	]
}
```

아직 사용 할 만한 곳이 없음

##### Policy_RW_DynamoDB.json

```
{
	"Version": "2012-10-17",
		"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"dynamodb:GetItem",
			"dynamodb:BatchGetItem",
			"dynamodb:PutItem",
			"dynamodb:UpdateItem",
			"dynamodb:BatchWriteItem",
			"dynamodb:DeleteItem"
				],
			"Resource":
				"arn:aws:dynamodb:<region>:<account-id>:table/<table-name>"
		}
	]
}
```

요런식으로 쓴다

동적변수도 쓸 수 있다는데 [여기](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/reference_policies_variables.html#policy-vars-infotouse){:target="_blank"} 참고


파트 1에서 함수 작성 후 역할 설정할 때 role로 myBasicExecution  만들었던거 생각

##### lambda_basic_execution.json

```
{
	"Version": "2012-10-17",
		"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"logs:CreateLogGroup",
			"logs:CreateLogStream",
			"logs:PutLogEvents"
				],
			"Resource": "arn:aws:logs:*:*:*"
		}
	]
}
```

### CH5

npm이 있어야한다 머시기

```sh
$ aws s3 mb s3://{bucket-name}
```
