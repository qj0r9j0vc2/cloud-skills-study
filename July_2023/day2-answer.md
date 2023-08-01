# **Topic2. Serverless**

# 1. Serverless 기초 개념

- serverless란? 필요 시에만 동적으로 컴퓨팅 리소스를 할당하여 유휴 상태에서의 낭비를 막고 컴퓨팅 리소스 관리를 제거한 컴퓨팅 모델
- 장점: 관리 부담 없음. 호출한 만큼만 비용이 부과됨
- 단점: 제한된 실행 시간. 느린 실행 속도

# 2. Event-driven architecture

- EDA란? 발행자와 소비자가 각각 이벤트를 발행하고 소비하는 구조의 아키텍처.
- 장점:
    - 본인이 처리(소비)해야할 범위만 신경쓰면 끝이므로 서비스 간에 느슨한 결합도를 가질 수 있음. → MSA에 적합₩
- 단점:
    - 비교적 어려운 초기 구성(MQ 구축, Message 정의 등등…)
    - 오류 발생 시 이를 처리하는 이벤트를 발행하는 등 서비스 전체에 걸쳐 롤백이 필요한 상황 발생 시 이를 처리하는 로직이 복잡하게 구성될 수 있다.
    - 사용자 요청이 처리되는 흐름을 파악하기 어려움. → 디버깅 하기 어려움

# 3. Serverless 기반 REST API 구축

- FaaS(Function as a Service): Function을 클라우드 컴퓨팅 자원에 등록하고, 이를 호출하는 만큼 비용을 청구한다. (AWS Lamdba, MS Azure Function 등…)

[Lambda, DynamoDB, API Gateway 활용하여 REST API 구축](./day2/Lambda,%20DynamoDB,%20API%20Gateway%20활용하여%20REST%20API%20구축.md)

[function URL 사용](./day2/function%20URL%20사용.md)

## API Gateway, Lambda Function URL 비교

api gateway와 통합된 Lambda는 HTTP, WebSocket을 지원하고 API Key, IAM, Cognito, Lambda 등을 통해 사용자 인증을 수행할 수 있는 반면 Lambda function URL은 오직 http만을 지원하고 IAM을 활용한 인증방식만을 사용할 수 있다. 뿐만 아니라 도메인 등록(function URL에선 CF를 활용하여야함) 등에서 function URL이 확연히 부족한 기능을 가지고 있다. 다만, 가격 면에서 충분히 이점을 가지고 있으며 간단한 기능을 가지는 lambda함수, 또는 15분 이상의 처리 시간이 소요되는 경우 Lambda FunctionURL을 활용하여 구성할 수 있으며, WebSocket을 활용하거나 확장된 인증 정책을 사용하고 싶을 경우 API Gateway를 결합해서 사용할 수 있다.

# 4. Lambda (Deep Dive)

## Cold Start

lambda 함수는 요청 입력 시에 실행중인 컨테이너를 조회하고, 만약 컨테이너가 존재하지 않는다면 새로운 컨테이너를 작동시키게 되는데 이를 cold start라고 한다. 컨테이너는 실행 이후 5분 간 존재하는데, 이 5분 안에 다른 요청이 입력되어 작동하는 상태를 warm start라고 한다. cold start를 극복하기 위한 방법으로는 메모리 추가 할당, container가 잠들지 못하게 하여 이를 재사용, 또는 프로비저닝된 동시성 기능 활성을 통한 대기중인 컨테이너 유지 등이 있고 여기에서 더 나아가 트래픽이 급증하는 시간대에 맞춰 프로비저닝 오토스케일링을 진행할 수도 있다.

## Lambda 동시성

**동시성이란 lambda가 동시에 처리하는 전송 중인 요청의 수**이며, 이 수치에 따라 실행 환경의 개별 인스턴스를 프로비저닝한다.

기본적으로 lambda는 별도의 격리된 공간(컨테이너)에서 작업을 수행하는데, 이때 복수 개의 요청 입력 시 lambda는 다른 컨테이너를 프로비저닝하여 요청을 처리한다. 이때 1초 동안 처리할 수 있는 요청의 개수를 **동시성**이라고 한다.

초당 100개의 요청을 입력받는 함수일 때, 요청 지속 시간이 500ms(0.5s)이면 이에 대한 동시성은 50이다. 만약 요청 지속 시간이 1s라면 동시성은 100이 되고, 200ms이면 동시성은 20이다. 

이때 100개의 요청을 처리해야하는 상황에서 요청 지속시간이 500ms, 즉 하나의 실행 환경에서 초당 2개의 요청을 처리할 수 있으므로 50개의 실행 환경이 필요하다. 고로, 위 상황에서 각각 초당 요청을 처리하기 위한 실행 환경의 개수는 50, 100, 20이다. 

다만 하나의 실행 환경에서 초당 처리할 수 있는 최대 요청의 개수는 10개인데, 때문에 100ms 미만의 요청 처리 속도를 가진 작업에 대해서는 모두 동일하게 취급된다. 때문에 초당 200개의 요청에서 평균 요청 기간이 50ms인 함수일 경우 동시성은 10이지만, 실행 환경에서 처리할 수 있는 요청은 100ms 이하일 때 모두 동일하게 취급하므로 20개의 실행 환경이 필요하다. 이를 동시성과 requests per second가 다르다고 표현한다.

추가적으로, 이러한 동시성은 기본적으로 리전 당 1000으로 제한되어 있으며, 추가적으로 요청한다 하여도 제한되어있어 한도를 초과하면 일부 실행환경이 제거되는데, 이때 제거되는 실행환경을 관리하기 위해 **예약된 동시성**과 **프로비저닝된 동시성**을 활용할 수 있다. 

예약된 동시성의 경우 대역폭을 분리한 것과 같이 특정 개수를 함수에 할당하여 해당 함수는 개수만큼의 동시성을 가지고 활동할 수 있으며, 이러한 동시성은 다른 함수가 침범할 수 없다. 

## 프로비저닝된 동시성

프로비저닝된 동시성의 경우 실행 환경 최초 실행 시 발생하는 cold start를 방지하기 위해 미리 대기시켜놓는 것으로써 프로비저닝된 동시성 설정 시 즉시 리전 별로 설정되어있는 500 ~ 3000 만큼의 동시성 Burst가 이루어지고 이후 500 단위로 목표로 설정한 지점까지 프로비저닝한다.

[프로비저닝된 동시성 구성 실습](./day2/프로비저닝된%20동시성%20구성%20실습.md)

[AWS Lambda with Application Auto Scaling 실습](./day2/AWS%20Lambda%20with%20Application%20Auto%20Scaling%20실습.md)

## Lambda invocation types

- Sychronous
    - 가장 간단한 호출 모델
    - API call에 따라 즉시 호출되어 함수를 실행한다.
    - 예시: API Gateway + AWS Lambda + DynamoDB
- Asychronous
    - queue에 이벤트를 전송하고 추가적인 정보 없는 성공 응답을 받음으로써 호출 수행
    - 이후 queue로부터 이벤트를 읽고 람다 함수를 실행
    - 예시: S3/SNS + Lambda + DynamoDB
        - S3에 새로운 개체가 작성될 때 람다를 비동기로 호출.
        - 이를 위해 invocation type parameter를 event로 지정하여야함.
- Poll-Based
    - AWS Stream 및 Queue 기반 서비스들과의 통합을 가능케함.
    - 서비스들이 Lambda 함수를 직접 호출하는 것이 아닌, Lambda 함수가 Stream 또는 Queue에서 Poll하는 것.
    - 예시: SQS + Lambda

[Lambda container 실습 및 코드 기반 함수와의 차이점 분석](./day2/Lambda%20container%20실습%20및%20코드%20기반%20함수와의%20차이점%20분석.md)



# 5. DynamoDB (Deep Dive)

## NoSQL의 개념, RDBMS와의 차이점

정형화된 데이터를 처리하는 것에 초점을 맞춘 2세대 관계형 데이터베이스와 달리 NoSQL은 RDBMS가 제공하는 안정성과 일관성 유지의 장점을 포기하고 데이터 구조를 미리 정해놓지 않는 형태로 비정형 데이터를 처리한다. 때문에 저렴한 비용으로 여러 대의 컴퓨터에 데이터를 분산, 저장, 처리하는 것이 가능해졌고 스키마 없이 동작하므로 데이터 구조를 미리 정의할 필요가 없으며 수시로 그 구조를 수정할 수 있어 비정형 데이터를 효율적으로 관리할 수 있다.

다만, 위에서 서술한 특징으로 인해 트랜잭션을 제공하지 않고 SQL 대신 별도의 기술을 통해 데이터의 내용을 분석하여야한다. 

## DynamoDB 파티션 키, 정렬 키

dynamoDB는 파티션에 데이터를 저장하며, 다음과 같은 상황에서 파티션을 늘린다. 

- 기존 파티션이 지원할 수 있는 한도를 초과하여 테이블의 할당된 처리량 설정을 늘리는 경우
- 기존 파티션 용량이 다 차서  추가 스토리지 공간이 필요한 경우

이때, DynamoDB에서 파티션을 결정하는 기준이 파티션 키이다. DynamoDB는 파티션 키의 해시 함수 값 결과에 따라 저장할 파티션의 위치를 결정한다. 이는 파티션 키가 복합 키일지라도 동일하게 적용된다.

이렇게 파티션 키에 따라 달라진 파티션 내부에서는 정렬 키를 기준으로 항목을 저장한다. 때문에 만약 Partition Key의 역할을 하는 칼럼이 type이고, Name을 정렬 키로 가진 경우 type == Dog인 항목 조회 시 Name 순으로(Ex. Alex, Brown, Charlie, Elice…) 정렬되어 조회된다.

### 파티션 키 설계 모범 사례

파티션 키에 따라 배치하는 파티션이 달라지므로, 이 값은 되도록 많은 수의 고유값을 가질 수 있는 칼럼으로 선택하는 것이 권장되며 또한 조회 조건으로 선정되는 칼럼을 선택할 필요가 있다. 정렬 키의 경우에서는 조회 시 자주 사용되는 정렬 조건(ex. created_at)을 정렬 키로 가짐으로써 조회 대상 데이터를 대상으로 불필요한 정렬 연산이 추가되지 않도록 구성하는 것이 올바르다.

- 가장 자주 수행하는 쿼리
    - select * from student where grade_num = 1 order by score desc;
- PartionKey: grade_num
- SortedKey: score

### 파티션 키 설계 실패 사례

모범 사례와 반대되도록, 데이터 분포도(칼럼이 가지는 고유도)가 낮으며, 하나의 값에 데이터가 집중되어있는 구조를 띄고, 가장 자주 수행하는 or 많은 부하를 주는 쿼리에서 사용되는 조건 문 칼럼이 PartitionKey, SortedKey 어느 것에도 포함되지 않는 경우일 때 파티션 키 설계에 실패했다고 어느정도 판단할 수 있다.

- 데이터 분포도:
    - user_type
        - student: 95%
        - teacher: 5%
- 가장 자주 수행하는 쿼리
    - select * from user where create_at ≤ 2019-02-23 order by score desc;
- PartitionKey: user_type
- SortedKey: email

## GSI(Global Secondary Index), LSI(Local Secondary Index)

- Secondary Index란?
    - 파티션 키를 통하여 원하는 파티션에 빠르게 접근할 수는 있지만, 쿼리를 수행 함에 있어 더욱 다양한 조건들이 포함될 수 밖에 없으며, 이를 PartitionKey에 전적으로 맡기는 것은 성능 상으로도 그다지 좋은 선택이 아니다. 때문에 RDB에서처럼 새로운 Index를 추가할 수 있는데, 그것이 Secondary Index이다.
- GSI(Global Secondary Index)
    - GSI는 파티션 키를 선두에 두지 않고 새로운 인덱스 조합을 제공함으로써 파티션에 종속적이지 않는다. 특성상 데이터의 추가와 삭제에 LSI에 비하여 더 많은 부하를 가져오게 되지만, 정제하고자 하는 데이터의 조건이 모든 파티션에 넓게 펴져있는 상태일 경우 탁월한 선택지가될 수 있다.
- LSI(Local Secondary Index)
    - LSI는 파티션 키를 인덱스의 선두에 배치함으로써 인덱스의 구조를 파티션에 종속적으로 배치한다. 때문에 GSI에 비하여 비교적 성능 부하가 덜할 수 있으며, 파티션 키를 지속적으로 대부분의 쿼리에서 조회 조건으로 활용하는 경우 탁월한 선택지가 될 수 있다.

## Global Table

Global table이 배치된 리전 간에 지속적으로 동기화 상태를 유지하여 범 리전적으로 단일된 것처럼 여겨지는 DynamoDB를 제공한다. multi-az에서 이젠 multi-region이 된 것이다. 뿐만 아니라 1초 이내에 모든 변경 사항이 공유되는 동기화 과정을 통해 일관성을 지키고 client에서 받아들이기로써는 사실상 로컬에서의 Read/Write 과정만 존재하는 것이므로 성능 상의 이점도 기대할 수 있다.

[Global Table 실습](./day2/Global%20Table%20실습.md)


## DynamoDB TTL

- dynamoDB에서는 항목을 관리할 때 unix time으로 TTL을 설정할 수 있는데, TTL 활성화 시 파티션별 백그라운드 프로세스가 만료된 TTL 항목을 지속적으로 평가하여 삭제를 수행한다. 이때 삭제되는 로그는 시스템 삭제로 기록된다.
- 다만, 스캐너가 작동하여 지속적으로 ttl 상태를 평가하는 것이므로 테이블의 크기에 따라 실제 삭제되기까지 일부 추가 시간이 경과될 수 있다.

[DynamoDB TTL 실습](./day2/DynamoDB%20TTL%20실습.md)



# 6. API Gateway (Deep Dive)

## API Gateway vs ALB

- 확장성
    - API Gateway: 10000RPS(requests per second) 제한
    - ALB: Not limited
- 안정성 가용성
    - 둘 다 aws managed service
    - API Gateway: 개발자가 아무 것도 신경 안써도 됨
    - ALB: 개발자가 하나보다 많은 AZ를 선택하도록 요구함으로써 높은 레벨의 가용성 달성
- Intergration
    - API Gateway: 다양한 aws 서비스들과 통합하는 데에 더욱 뛰어남
    - ALB: EC2 instnace, ECS containers, IP addresses로 연결 가능
- Requests Routing Capabilities
    - API Gateway: Path-based routing
        - url에 따라 어떤 리소스가 요청을 받도록 할 것인지 구성할 수 있다.
    - ALB: rule-based routing
        - 다음 항목들의 상태에 따른 다양한 조합을 가질 수 있다.
            - Requester Hostname
            - Requester IP address
            - HTTP Headers
            - HTTP Request method
            - Key/value pairs incoming as query strings
- Costs
    - API Gateway
        - **Rest APIs**: from $1.51 to $3.50 per million requests
        - **HTTP APIs**: from $0.90 to $1.00 per million requests
        - **WebSockets**: from $0.80 to $1.00 per million requests, plus $0.25 per million connection minutes
    - ALB
        - based on time:
            - it is straightforward: $0.0225 per hour.
        - based on resource usage:
            - LCU-hour 마다 $0.008
            - LCU는 ALB에 의해 처리된 트래픽을 측정한다.
            - 하나의 LCU가 지원하는 것(만약 아래 기준 초과 시 시간 당 추가적인 LCU 비용을 부과할 것)
                - 25 new connections per second
                - 3,000 active connections per minute
                - 1 GB of traffic per hour for EC2 instances, or 0.4 GB per hour for Lambda functions
                - 1,000 routing rule evaluations per second
                

## API Gateway HTTP API vs REST API

- HTTP API
    - 최소한의 기능만을 지원
    - 더 낮은 가격
    - 지원하는 엔드포인트 유형
        - 리전
    - 보안
        - mTLS
    - Authentication
        - IAM
        - Cognito
        - AWS Lambda
        - JWT
- REST API
    - 다양한 기능 지원
        - API 키, 클라이언트별 제한, 요청 검증, WAF 통합, 프라이빗 API 엔드포인트
    - 지원하는 엔드포인트 유형
        - 엣지 최적화, 리전, 프라이빗
    - 보안
        - mTLS
        - backend certificate
        - AWS WAF
    - Authentication
        - IAM
        - 리소스 정책
        - Cognito
        - AWS Lambda
    

## Lambda Proxy vs Non-Proxy

- Lambda Proxy
    - Lambda 함수와 clinet 간의 직접적인 상호작용에 의존한다.
    - client로부터의 요청이 직접 lambda에게 전달되며 모든 요청 및 응답에는 APIGateway가 관여하지 않는다.
    - 오직 API Gateway와만이 결합할 수 있으며, 다른 모든 AWS 서비스 작업에서 사용할 수 없다.
- Non-proxy
    - 요청와 응답은 lambda에게 전송되거나 전달받았을 때 수정될 수 있으며, API Gateway는 요청과 응답에 대한 완전한 제어를 얻는다.
    - 요청과 응답에 대한 APIGateway의 필수적인 데이터 매핑과 응답 코드 설정 등이 필요하나, 다른 서비스에서 동일한 Lambda 함수를 활용할 수 있다.

[Lambda Authorizer를 통한 인증/인가 실습](./day2/Lambda%20Authorizer를%20통한%20인증:인가%20실습.md)