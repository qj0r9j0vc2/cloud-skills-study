# AWS Lambda with Application Auto Scaling 실습

lambda 함수는 이전에 apigateway + lambda + dynamoDB 실습 때 생성한 함수를 사용한다.

# Register Scalable target

```bash
$ aws application-autoscaling register-scalable-target \
--service-namespace lambda \
--scalable-dimension lambda:function:ProvisionedConcurrency \
--resource-id function:rest-lambda:version-1 \
--min-capacity 0 \
--max-capacity 100
{
    "ScalableTargetARN": "arn:aws:application-autoscaling:ap-northeast-2:786584124104:scalable-target/07md4e1ccea3d5054da1a6988aeb97f16209"
}
```

![](<./image/스크린샷 2023-08-01 오전 4.36.14.png>)

정상적으로 등록 완료되어 arn이 나온 모습을 확인할 수 있다.

# Verify lambda function is registered correctly

```bash
$ aws application-autoscaling describe-scalable-targets --service-namespace lambda
```

![](<./image/스크린샷 2023-08-01 오전 4.36.28.png>)

# Schedule the Provisioned Concurrency

```bash
$ aws application-autoscaling put-scheduled-action --service-namespace lambda \
  --scalable-dimension lambda:function:ProvisionedConcurrency \
  --resource-id function:rest-lambda:version-1 \
  --scheduled-action-name scale-out \
  --schedule "cron(45 11 * * ? *)" \
  --scalable-target-action MinCapacity=250
```

위 명령을 통해 예약된 작업(cron: 매일 11시 45분)을 등록할 수 있다. 

이를 통해 실제 서비스에서 트래픽이 증가하는 기간을 측정하여 활용할 수 있으리라 기대할 수 있다.

```bash
$ aws application-autoscaling describe-scheduled-actions --service-namespace lambda 
```

위 명령을 통해 정상적으로 등록되었는지 확인할 수 있다.

![](<./image/스크린샷 2023-08-01 오전 4.44.44.png>)

아래 명령을 통해 15시 13분에 capacity 개수를 0으로 지정 할 수 있다.

```bash
$ aws application-autoscaling put-scheduled-action --service-namespace lambda \
  --scalable-dimension lambda:function:ProvisionedConcurrency \
  --resource-id function:rest-lambda:version-1 \
  --scheduled-action-name scale-in \
  --schedule "cron(15 13 * * ? *)" \
  --scalable-target-action MinCapacity=0,MaxCapacity=0
```

이후 실습한 것들은 모두 지워주자

# delete scheduled action

```bash
$ aws application-autoscaling delete-scheduled-action \
--service-namespace lambda \
--scheduled-action-name scale-out \
--resource-id function:rest-lambda:version-1 \
--scalable-dimension lambda:function:ProvisionedConcurrency
```

# deregister scalable target

```bash
$ aws application-autoscaling deregister-scalable-target \
--service-namespace lambda \
--resource-id function:rest-lambda:version-1 \
--scalable-dimension lambda:function:ProvisionedConcurrency
```

https://aws.amazon.com/ko/blogs/compute/scheduling-aws-lambda-provisioned-concurrency-for-recurring-peak-usage/