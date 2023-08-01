# Lambda Authorizer를 통한 인증/인가 실습

![](<./image/스크린샷 2023-08-01 오후 11.39.05.png>)

함수를 생성한다.

```
// A simple token-based authorizer example to demonstrate how to use an authorization token 
// to allow or deny a request. In this example, the caller named 'user' is allowed to invoke 
// a request if the client-supplied token value is 'allow'. The caller is not allowed to invoke 
// the request if the token value is 'deny'. If the token value is 'unauthorized' or an empty
// string, the authorizer function returns an HTTP 401 status code. For any other token value, 
// the authorizer returns an HTTP 500 status code. 
// Note that token values are case-sensitive.

export const handler =  function(event, context, callback) {
    var token = event.authorizationToken;
    switch (token) {
        case 'allow':
            callback(null, generatePolicy('user', 'Allow', event.methodArn));
            break;
        case 'deny':
            callback(null, generatePolicy('user', 'Deny', event.methodArn));
            break;
        case 'unauthorized':
            callback("Unauthorized");   // Return a 401 Unauthorized response
            break;
        default:
            callback("Error: Invalid token"); // Return a 500 Invalid token response
    }
};

// Help function to generate an IAM policy
var generatePolicy = function(principalId, effect, resource) {
    var authResponse = {};
    
    authResponse.principalId = principalId;
    if (effect && resource) {
        var policyDocument = {};
        policyDocument.Version = '2012-10-17'; 
        policyDocument.Statement = [];
        var statementOne = {};
        statementOne.Action = 'execute-api:Invoke'; 
        statementOne.Effect = effect;
        statementOne.Resource = resource;
        policyDocument.Statement[0] = statementOne;
        authResponse.policyDocument = policyDocument;
    }
    
    // Optional output with custom properties of the String, Number or Boolean type.
    authResponse.context = {
        "stringKey": "stringval",
        "numberKey": 123,
        "booleanKey": true
    };
    return authResponse;
}
```

위 내용을 함수 내용에 작성한 뒤 배포한다.

# API Gateway 만들기
![](<./image/스크린샷 2023-08-01 오후 11.41.45.png>)

예제로 생성한다.

![](<./image/스크린샷 2023-08-01 오후 11.43.53.png>)

이후 권한 부여자 섹션에 들어가서 위 옵션으로 생성을 진행한다.


![](<./image/스크린샷 2023-08-01 오후 11.44.16.png>)

allow로 입력 후 테스트 진행 시 정상적으로 수행된다.

![](<./image/스크린샷 2023-08-01 오후 11.50.02.png>)

이후 메서드 실행으로 이동하여 승인 방식을 설정하고 배포하면 끝난다.