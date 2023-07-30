# Topic 1. Network and Security

# 1. VPC
---
## internet G/W

internet과 통신할 수 있도록 돕는 게이트웨이. 대부분의 경우 라우팅 테이블에서 디폴트 라우팅으로 0.0.0.0/0 → igw 레코드를 추가하여 사용한다. 외부에서 igw가 연결된 subnet의 resource로의 요청을 정상적으로 처리한다. VPC와 1:1로 연결된다.


## Egress-only Intenet G/W

ipv6를 통한 인터넷과의 outbound 통신만을 허용한다. ngw와 유사하게 동작하지만 ipv6 트래픽만을 허용하며 security group을 연결할 수 없고, nacl을 활용하여 트래픽을 컨트롤해야한다.


## NAT G/W

기존에 nat instance를 통해 수행하던 NAT 과정을 aws에서 제공하는 기능으로 통합하여 수평적 확장이 가능한 리소스로 제공한다. 주로 사설 zone(igw가 연결되지 않은 서브넷)에서 인터넷과 통신하기 위해 자신의 사설 IP를 공인 IP로 변환 하여 인터넷 연결을 성립하도록 한다. 다만, 이는 pubilc ngw일 경우이고 private ngw일 경우 자신의 사설 IP를 변환하여 다른 사설 망 내 네트워크 장치에 접근할 수 있도록 돕는다. subnet과는 1:1로 지정할 수 있으며, 사설 subnet route table에서 레코드를 추가해주어야한다. ngw는 private resource가 밖으로 나가는 것만 도우므로 외부에서 들어오는 요청은 처리하지 않는다. EIP를 연결해서 사용하며, igw를 연결한 서브넷 인스턴스의 요청과 비교 시 표시되는 ip가 다르게 표현되며, ngw를 연결한 서브넷에서 인스턴스를 런치할 때 퍼블릭 IP 자동 할당 옵션을 통해 할당받은 IP가 존재하더라도 ngw의 IP가 표시된다.

[ngw가 연결된 인스턴스 출발의 트래픽 추적](./day1/ngw%EA%B0%80%20%EC%97%B0%EA%B2%B0%EB%90%9C%20%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%20%EC%B6%9C%EB%B0%9C%EC%9D%98%20%ED%8A%B8%EB%9E%98%ED%94%BD%20%EC%B6%94%EC%A0%81.md)


## VPC Peering

서로 다른 vpc 리소스 간의 통신을 가능케 하며, 전이적인 라우팅이 불가능하여 라우팅을 수행하여야하는 그룹의 구성원이 추가될 때마다 관리 비용이 비선형적으로 증가한다. 

때문에 많은 VPC를 다룰 시에는 TGW를 사용하는 것이 올바르지만, 저렴한 가격으로 소규모 인프라에서 채택할 수 있다.


[Peering으로 연결된 VPC 간의 private zone dns 쿼리](./day1/Peering%EC%9C%BC%EB%A1%9C%20%EC%97%B0%EA%B2%B0%EB%90%9C%20VPC%20%EA%B0%84%EC%9D%98%20private%20zone%20dns%20%EC%BF%BC%EB%A6%AC%20.md)


## DXGW VGW limit

TGW를 생성해서 TGW에 VGW를 연결하게 되면 된다. 하나의 tgw에 5000개까지 연결할 수 있고, 전용 DX 구성 시 4개까지의 Transit VIF를 설정할 수 있으므로 기존 20개까지만 연결 가능했던 한계를 사실한 무한정 늘릴 수 있다.
![](<./image/스크린샷 2023-07-31 오전 2.17.31.png>)


## VPC Endpoint, Private Link

VPC endpoint는 크게 2개, Gateway endpoint와 PrivateLink로 나뉠 수 있는데 PrivateLink는 여기에서 Interface endpoint와 Gateway Load Balancer Endpoint로 나뉠 수 있다. 

## Gateway endpoint

Gateway endpoint 같은 경우는 대부분의 자료에서 소개하듯 S3, DynamoDB와 같은 서비스를 통해 사용할 수 있는데, 만약 Gateway endpoint 사용 시 퍼블릭 IP를 그대로 사용하나, aws 측에서 내부 백본 망으로 접속을 진행하도록 하여 Amazon Network 망을 벗어나지 않는 퍼블릭 액세스를 가능하게 한다. 만약 gateway endpoint를 활용하되, 자신의 리소스에서의 접근만을 허용하고 싶다면 s3 버킷 정책을 통해 액세스를 제어할 수 있다.

![](<./image/스크린샷 2023-07-31 오전 2.25.43.png>)

## Private Link

private link 같은 경우 또한 내부에서 접속하도록 돕지만, private IP를 통해 접근할 수 있도록 한다. endpoint의 ENI를 통해 서비스에 액세스할 수 있으며, Gateway Endpoint의 경우, S3 요청을 게이트웨이로 요청하고 게이트웨이를 통과한 트래픽은 S3로 즉시 전달되지만 Private Link에 연결된 endpoint ENI는 요청을 즉시 S3로 전달한다.

GWLB endpoint는 어플라이언스에 특화되어 VPC를 오가는 트래픽을 전달받아 확인하여 보안 감사를 진행할 수 있다. market place를 통해 IPS IDS 시스템 등을 구축할 수 있으며 운용을 3자 서비스에게 위임하여 비용을 절감할 수 있다.

GWLB endpoint는 GatewayLoadBalancer를 통해 작업을 수행하지만 interface endpoint같은 경우 생산자 VPC에서 NLB를 생성하게되며, 소비자가 NLB로 향하는 endpoint로 요청을 처리한다. 또한 NLB에서 ALB를 트래픽 전달 구성으로 활용할 수 있게 됨에 따라 더욱 다양하게 활용할 수 있으며, 인가가 이루어진 클라이언트의 VPC를 대상으로 서비스를 제공하여 유연한 형태의 서비스 공급망으로써 활용할 수 있다.



# 2. CloudFront
## CDN 개념

사용자의 요청이 PoP로 전달되어 백본망을 통해 접근하도록 구성함으로써 공격자로부터의 원본 서버 보호와 더불어 캐싱된 데이터를 통한 빠른 서비스 공급을 기대할 수 있다.


## 최종 사용자 IP 얻기

- X-Forwarded-For 헤더 활용
    - Viewer Protocol Policy와 X-Forwarded-For가 포함된Whitelist를 옵션으로 포함하는 Forward Headers를 구성한다.
    - 여러 proxy를 통해 처리된 요청일 경우 XFF에 여러 IP가 포함될 수 있으므로 첫 IP만 사용


## 캐시 업데이트 방법

- CF 캐싱은 기본적으로 24시간까지 캐시를 유지하므로 이를 무효화하여 업데이트된 내용을 적용시킬 수 있다.
- distrbution에서 원하는 대상의 경로를 입력하여 invalidation을 생성한다.


## CF 사용시 인증서 구성

- CF 구성
    - ACM 인증서 요청
        - CF 인증서 적용을 위해선 us-east-1에서 작업을 수행해야함.
        - DNS 레코드에 키-값 레코드 추가하여 소유권 확인
    - CF에서 사용자 정의 인증서를 사용하도록 구성.
    - Route53에서 Cloudfront 배포 등록
- ALB 구성(옵션) - CF에서 원본 프로토콜을 HTTP로 전달하도록 구성하면 가능함.
    - ec2가 실행 중인 리전에서 ACM 인증서 생성
    - 보안 리스너 설정에서 SSL 인증서로 전에 생성한 인증서 사용

alb → ec2 사이의 트래픽은 기본적으로 http를 사용하지만 보안 요구사항 등으로 인해 모든 전송 과정에서의 암호화가 필요할 경우 self-signed certificate을 활용할 수 있다.

위 상황에서 모든 요청이 cf를 통해서만 허용된다면, cf → alb 통신 과정을 암호화하지 않고 인증서를 가져오는 과정에서 발생하는 latency를 줄일 수 있다. 다만, cf origin 통신은 keepalive를 사용하므로 인증서를 사용하는 것을 권장한다.


## CORS(Cross Origin Resource Sharing)

다른 도메인 간에 리소스 요청 시 발생하는 오류. (ex. https://api.sample.io를 호출하는 localhost:3000)

### Cloudfront CORS

cloudfront에서는 오리진 요청 정책, 또는 Legacy cache setttings로 origin에 대한 cors 요청을 활성화할 수 있다.

- 오리진 요청 정책
    - CORS-CustomOrigin
        - 오리진이 사용자 지정 오리진일 때 CORS 요청을 활성화하는 헤더 포함
        - 오리진 요청에 포함된 헤더
            - Origin
    - CORS-S3Origin
        - 오리진이 Amazon S3일 때 CORS 요청을 활성화하는 헤더 포함
        - 오리진 요청에 포함된 헤더
            - Origin
            - Access-Control-Request-Headers
            - Access-Control-Request-Method
    ![](<./image/스크린샷 2023-07-31 오전 4.10.54.png>)

    
- legacy cache settings
    - 다음 헤더 포함 → “Origin”을 포함하도록 구성
    ![](<./image/스크린샷 2023-07-31 오전 4.10.28.png>)
    

만약 s3를 오리진으로 구성하고 cors를 활성화하고자 할 경우, s3에서 cors 설정을 구성하여야한다.


## Cloudfront Lambda@Edge

lambda@Edge를 활용하게 되면 backend에서 준 정보에 추가로 http response header에 정보를 추가할 수 있다.

Cloudfront 배포를 Lambda@Edge 함수와 연결 시 다음과 같은 위치에 배치할 수 있으며, 다음 중 오리진 응답과 최종 사용자 응답에 Lambda 함수를 추가하여 추가적인 작업을 수행할 수 있도록 구성할 수 있다.

- CloudFront가 최종 사용자의 요청을 수신할 때(최종 사용자 요청)
- CloudFront가 오리진에 요청을 전달하기 전(오리진 요청)
- CloudFront가 오리진의 응답을 수신할 때(오리진 응답)
- CloudFront가 최종 사용자에게 응답을 반환하기 전(최종 사용자 응답)

자세한 설명은 다음 참조 - https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/lambda-at-the-edge.html



# 3. LB
## draining time

draining time은 노드가 종료되기 전에 모든 연결이 끝날 때 까지 기다리는 시간이다.

ELB는 deregistration 작업 수행 시 deregistering된 타겟으로 요청을 전달하는 것을 중지한다. 다만, 이때 기존에 수행하던 작업이 완료되기까지 기본적으로 300초 동안 대기하는데, 이것을 draining time(Deregistration delay)라고 부른다. 

deregistration 작업 수행 시 타겟은 draining 상태로 변경되며, 이후 deregistration delay가 경과한 이후부턴 타겟의 상태가 unused가되고 auto scaling group에서 제거된다. 

만약 draining time(dereigstration delay)이 경과되기 전에 타겟이 연결을 종료한다면 클라이언트는 500번 대의 오류를 응답받는다.


## ALB https redirect

alb 생성 → listeners and rules → add listener

![](<./image/스크린샷 2023-07-31 오전 5.09.26.png>)


## NLB, ALB 차이

NLB는 사용자의 L4 환경에서 패킷을 확인하고 전달한다. 하지만 ALB는 L7에서 작동하여 IP, Port, Header까지 모두 확인 후 전달한다. 이때 ALB를 통과한 패킷은 SourceIP가 유동적으로 변경되는 ALB의 주소로 전달되며 대신에 실제 사용자의 IP를 X-Forwarded-For에 표시된다.(X-Forwarded-로 시작하여 IP와 Port, Proto 등의 정보까지 조회 가능). 

alb는 security group을 함께 적용하여 default에서의 요청을 허용하도록 구성할 수 있지만 nlb에서는 프록싱하지 않기에 client가 ec2의 security group에 영향을 받는다. 때문에 nlb 80 → ec2 8080의 예시에서는 ec2에 0.0.0.0/0 → 8080 허용 인바운드를 추가하여야한다.

또한 ALB에서는 http keep-alive를 활용하고, NLB에서는 tcp keep-alive를 활용하는데 이 때문에 http keep-alive를 활용하는 alb에서는 load balancer가 ec2보다 먼저 keepalive가 종료될 수 있도록 keepalive를 구성하여 loadbalancer가 ec2측에서 종료된 keepalive를 활용하여 요청이 잘못 처리되는 일을 방지하여야하고(만약 이럴 경우 502 bad gateway가 응답된다), nlb에서는 커널의 keepalive 파라미터를 수정하여 nlb의 timeout보다 빠르게 종료를 알려 사용자와 ec2 간의 커넥션이 종료되도록 하여 nlb에서 연결을 종료시켜 rst 패킷이 전달되는 사태를 방지하여야한다.


## ALB stickiness

alb stickiness는 클라이언트의 쿠키를 활용하여 세션 중에 처리되는 요청이 고정된 노드에게 전달되도록 한다. 

해당 옵션을 통해 기존에 인가된 사용자가 새로운 노드에서 요청을 처리하게 되어 로그인 정보를 찾지 못하고 미인가된 사용자로 취급되는 것을 방지하여 사용자 권한 인증에 있어 원활한 서비스 사용 경험을 제공할 수 있다. 

다만 위와 같은 상황에서는 로그인 정보를 노드 별로 저장하여 발생하는 문제이므로 alb stickiness를 사용하지 않고 모든 노드의 애플리케이션이 공통된 하나의 ElastiCache 서버를 사용하거나, 인증/인가 과정에서 jwt를 채택하는  등 사용자 권한 인증/인가 과정에 사용하는 아키텍처를 수정하여 문제를 극복할 수 있다.

### 4. IAM
- iam user
    - 하나의 사용자를 이야기하며, aws 루트 계정에 종속적이다. iam group을 통해 그룹에 속하여 그룹의 정책을 일괄적으로 부여받거나, 직접 정책을 연결하여 권한을 부여받을 수 있다.
- iam role
    - 정책들과 미리 연결되어 그룹에 권한을 부여하는 대신, 역할을 부여하여 관리할 수 있다. 또한 역할을 활용하여 임시자격증명을 생성할 수 있어 보안 향상, 강력한 권한 제어를 기대할 수 있다.
- iam policy
    - 사용자와 그룹, 역할 등에 연결될 때 해당 권한을 정의하는 aws 객체이다. 크게 관리형 정책과 인라인 정책으로 나뉜다.

## 인증과 인가

- 인증: 올바른 사용자인가
- 인가: 해당 권한이 있는가

## IAM Identity providers

IAM 자격 증명 공급자를 통해 기존에 존재하는 facebook, google 등과 같은 IdP를 활용해 사용자 자격 증명에 사용할 수 있으며 이러한 외부 사용자 자격 증명에 대해 AWS 리소스 사용 권한을 부여할 수 있다.


[Assume-role 실습](./day1/Assume-role%20%EC%8B%A4%EC%8A%B5.md)


## EKS pod에서 [http://169.254.169.254](http://169.254.169.254/)에 대한 호출 실패

- IMDS 조회 시 TOKEN 미포함
    - [169.254.169.254](http://169.254.169.254/)에 요청하여 IMDS를 활용할 때 버전이 1일 때 요청 → 응답으로 메타 데이터가 출력되지만 버전 2에서는 토큰을 미리 준비하여야한다.
    - imds v2 사용을 옵션으로 전환하면 v1 방식으로 요청을 수행할 수 있다.
        ![](<./image/스크린샷 2023-07-31 오전 7.24.44.png>)
        
        ```yaml
        [ec2-user@ip-192-168-1-181 ~]$ curl http://169.254.169.254/latest/meta-data/
        ami-id
        ami-launch-index
        ami-manifest-path
        block-device-mapping/
        events/
        hostname
        identity-credentials/
        instance-action
        instance-id
        instance-life-cycle
        instance-type
        local-hostname
        local-ipv4
        mac
        metrics/
        network/
        placement/
        profile
        public-hostname
        public-ipv4
        public-keys/
        reservation-id
        security-groups
        services/
        ```
        
- network 문제



# 5. WAF/Shield
## WAF와 Shield의 차이

- WAF(Web Application Firewall)
    - 이름부터 웹 어플리케이션 방화벽.
    - 7계층을 타깃으로 하여 지능형 위협을 완화하고 일반적인 익스플로잇으로부터 웹 어플리케이션 보호가 주 목적.
- Shield
    - 오직 DDos 방어에 초점을 맞추며 shield standard의 경우 aws 리소스에서 무료로 작동하며 3, 4계층에서 방어하고, Shield Advanced의 경우 3 ~ 7계층의 DDoS 공격을 방어한다.


## WAF WCU

WCU(Web ACL capacity units)는 WAF가 처리할 수 있는 용량을 의미한다. WCU부족 시 500 단위로 증설할 수 있으며 이에 따라 가격이 추가적으로 부과된다.


[한 IP에서 5분 동안 100번 이상 요청 시 차단하는 AWS WAF](./day1/%ED%95%9C%20IP%EC%97%90%EC%84%9C%205%EB%B6%84%20%EB%8F%99%EC%95%88%20100%EB%B2%88%20%EC%9D%B4%EC%83%81%20%EC%9A%94%EC%B2%AD%20%EC%8B%9C%20%EC%B0%A8%EB%8B%A8%ED%95%98%EB%8A%94%20AWS%20WAF.md)


[log4shell을 막는 aws WAF 구성](./day1/log4shell%EC%9D%84%20%EB%A7%89%EB%8A%94%20aws%20WAF%20%EA%B5%AC%EC%84%B1.md)
