# log4shell을 막는 aws WAF 구성

![](<../image/스크린샷 2023-07-31 오전 8.10.42.png>)
![](<../image/스크린샷 2023-07-31 오전 8.14.13.png>)

> AWSManagedRulesKnownBadInputsRuleSet를 Web ACL에 추가한 다음,  해당 Web ACL을 CloudFront 배포 지점, ALB, API Gateway 또는 AppSync GraphQL API와 연결하시면 됩니다.

https://aws.amazon.com/ko/blogs/korea/aws-security-bulletins-cve-2021-44228/