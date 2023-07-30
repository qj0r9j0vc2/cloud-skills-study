# Assume-role 실습
> User01, user02, user03 IAM user와 dev, ops라는 IAM role을 생성하고, user01은 dev role만 assume 가능하도록 하고 user02, user03은 ops라는 role만 assume 할 수 있도록 구성해 보세요.

## 사용자 user01, user02, user03 생성
![](<../image/스크린샷 2023-07-31 오전 7.01.29.png>)


## dev, ops 역할 생성
![](<../image/스크린샷 2023-07-31 오전 6.38.14.png>)

## user01 arn 복사, dev 신뢰 정책에 추가
![](<../image/스크린샷 2023-07-31 오전 6.55.59.png>)
![](<../image/스크린샷 2023-07-31 오전 6.57.51.png>)



## Assume role
```
[ec2-user@ip-192-168-1-181 ~]$ aws configure
AWS Access Key ID [None]: AKIA3OJARHLEJ7NERVZU
AWS Secret Access Key [None]: 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
Default region name [None]: ap-northeast-2
Default output format [None]:
[ec2-user@ip-192-168-1-181 ~]$ aws sts get-caller-identity
{
    "UserId": "AIDA3OJARHLEOZBLBAZBJ",
    "Account": "786584124104",
    "Arn": "arn:aws:iam::786584124104:user/user01"
}
[ec2-user@ip-192-168-1-181 ~]$ aws sts assume-role --role-arn arn:aws:iam::786584124104:role/dev --role-session-name "dev-user01"
{
    "Credentials": {
        ...
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "AROA3OJARHLEPR76MLZ67:dev-user01",
        "Arn": "arn:aws:sts::786584124104:assumed-role/dev/dev-user01"
    }
}
[ec2-user@ip-192-168-1-181 ~]$ aws sts assume-role --role-arn arn:aws:iam::786584124104:role/ops --role-session-name "ops-user01"

An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:iam::786584124104:user/user01 is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::786584124104:role/ops
```
user01에서 dev와 ops를 각각 assume하고 ops에 대해 인가받은 않았음을 확인하였다. 
이후, user02, user03에서도 동일한 작업 반복 및 확인



![](<../image/스크린샷 2023-07-31 오전 7.08.33.png>)

```
[ec2-user@ip-192-168-1-181 ~]$ aws configure
AWS Access Key ID [****************RVZU]: AKIA3OJARHLEIBHI7N63
AWS Secret Access Key [****************at+i]: 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
Default region name [ap-northeast-2]:
Default output format [None]:
[ec2-user@ip-192-168-1-181 ~]$ aws sts get-caller-identity
{
    "UserId": "AIDA3OJARHLEIH4ZM4VZX",
    "Account": "786584124104",
    "Arn": "arn:aws:iam::786584124104:user/user03"
}
[ec2-user@ip-192-168-1-181 ~]$ aws sts assume-role --role-arn arn:aws:iam::786584124104:role/dev --role-session-name "dev-user03"

An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:iam::786584124104:user/user03 is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::786584124104:role/dev
[ec2-user@ip-192-168-1-181 ~]$ aws sts assume-role --role-arn arn:aws:iam::786584124104:role/ops --role-session-name "ops-user03"
{
    "Credentials": {
        ...
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "AROA3OJARHLEB76HW3XWX:ops-user03",
        "Arn": "arn:aws:sts::786584124104:assumed-role/ops/ops-user03"
    }
}
[ec2-user@ip-192-168-1-181 ~]$
```