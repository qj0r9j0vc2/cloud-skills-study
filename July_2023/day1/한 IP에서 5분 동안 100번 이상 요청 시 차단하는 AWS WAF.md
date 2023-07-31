# 한 IP에서 5분 동안 100번 이상 요청 시 차단하는 AWS WAF

WAF에서 Create WebACL, 이후 다음으로 넘어간다.


![](<./image/스크린샷 2023-07-31 오전 7.49.26.png>)
![](<./image/스크린샷 2023-07-31 오전 8.02.59.png>)
![](<./image/스크린샷 2023-07-31 오전 8.03.21.png>)

> AWS WAF rate-based rules can only determine access in 5 minutes.
So please consider installing a third party WAF.
https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html

https://repost.aws/questions/QU_hJELbmLQt6YtIdvTBVmow/how-to-create-customize-aws-waf-rate-based-rule-for-1min-time-window