# Lambda container 실습 및 코드 기반 함수와의 차이점 분석

# lambda_function

```bash
$ mkdir lambda_container
$ cd lambda_container
$ cat <<EOF > lambda_function.py> import sys
> def handler(event, context):
>   return 'Hello from AWS Lambda using Python' + sys.version + '!'
> EOF
$ cat <<EOF > requirements.txt
> boto3
> EOF
```

## Dockerfile

```docker
FROM public.ecr.aws/lambda/python:3.10

# Copy requirements.txt
COPY requirements.txt ${LAMBDA_TASK_ROOT}

# Copy function code
COPY lambda_function.py ${LAMBDA_TASK_ROOT}

# Install the specified packages
RUN pip install -r requirements.txt

# Set the CMD to your handler (could also be done as a parameter override outside of the Dockerfile)
CMD [ "lambda_function.handler" ]
```

모두 같은 위치에 두고, docker build를 수행한다.

```bash
$ docker buildx build --platform=linux/arm64,linux/amd64 --tag=docker-image:test .
```

# Test image

```bash
$ docker run -p 9000:8080 docker-image:test
$ curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'
"Hello from AWS Lambda using Python3.10.12 (main, Jul 11 2023, 13:56:08) [GCC 7.3.1 20180712 (Red Hat 7.3.1-15)]!"
```

정상적으로 작동하는 것을 확인할 수 있다.

# ECR upload

```bash
# ecr login
$ aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 111122223333.dkr.ecr.ap-northeast-2.amazonaws.com
# create repository 이후 응답에서 repositoryUri를 사용하여 추후 과정 진행
$ aws ecr create-repository --repository-name hello-world --image-scanning-configuration scanOnPush=true --image-tag-mutability MUTABLE

$ docker tag docker-image:test 111122223333.dkr.ecr.ap-northeast-2.amazonaws.com/hello-world:latest
$ docker push 111122223333.dkr.ecr.ap-northeast-2.amazonaws.com/hello-world:latest
```

![](<./image/스크린샷 2023-08-01 오후 4.07.02.png>)

## 실행 역할 생성

```bash
$ aws iam create-role --role-name lambda-ex --assume-role-policy-document '{"Version": "2012-10-17","Statement": [{ "Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole"}]}'
```

## Lambda 함수 생성

```bash
$ aws lambda create-function \
  --function-name hello-world \
  --package-type Image \
  --code ImageUri=786584124104.dkr.ecr.ap-northeast-2.amazonaws.com/hello-world:latest \
  --role arn:aws:iam::786584124104:role/lambda-ex
```

![](<./image/스크린샷 2023-08-01 오후 4.11.03.png>)

## 함수 호출

```bash
$ aws lambda invoke --function-name hello-world response.json
```

![](<./image/스크린샷 2023-08-01 오후 4.11.44.png>)

![](<./image/스크린샷 2023-08-01 오후 4.12.06.png>)

정상적으로 출력되는 것과 함께 함수의 생성을 확인하였다 

# 코드 기반 함수와의 차이점

기존 Lamdba에서 코드를 업로드하여 실행하는 방법의 경우, 기계 학습 또는 데이터 집약적 워크로드와 같이 상당히 종속성이 수반되는 대규모 워크로드를 구축하는 데에 어려움이 있었다.

Lambda function을 container image로 패키징 및 배포하게되면 최대 10G 크기까지 처리할 수 있으며 이를 통해 실행 환경 종속적인 코드를 손쉽게 수행할 수 있도록 구성할 수 있다. 

https://aws.amazon.com/ko/blogs/korea/new-for-aws-lambda-container-image-support/