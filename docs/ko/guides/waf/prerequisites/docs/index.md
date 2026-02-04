# WAF 전제 조건

첫 번째 AWS WAF 웹 ACL을 생성하기 전에 다음 주제를 고려하십시오.

## AWS WAF로 보호할 리소스 식별

AWS WAF로 기본적으로 보호할 수 있는 리소스 유형을 이해하는 것이 중요합니다. 사용 사례가 직접 지원되지 않는 경우 고려해야 할 아키텍처 변경 사항이 있을 수 있습니다. 이 섹션에서는 가장 일반적인 사용 사례에 대한 모범 사례를 설명합니다.

### AWS WAF에서 기본적으로 지원하는 AWS 서비스

AWS WAF는 [다른 AWS 서비스와 기본적으로 통합](https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works-resources.html)됩니다. 보호하려는 리소스에 웹 ACL을 연결하기만 하면 됩니다.

![](../images/waf-supported-resources.png)
**그림 1:** AWS와 연결할 수 있는 전역 및 리전 AWS 리소스

WAF의 필요성이 불분명하거나 리소스를 웹 ACL과 직접 연결할 수 없는 사용 사례가 있습니다. 다음 섹션에서는 이러한 상황에 대한 지침을 제공합니다.

### 프라이빗 Application Load Balancer

프라이빗 Application Load Balancer(ALB)는 [인터넷 게이트웨이](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)로의 경로가 없는 VPC 서브넷을 사용합니다. 즉, ALB는 인터넷에서 직접 도달할 수 없습니다.

![](../images/waf-private-alb.png)
**그림 2:** 프라이빗 Application Load Balancer 보호

일반적으로(항상은 아니지만) 위험이 비용을 정당화하지 않기 때문에 AWS WAF로 프라이빗 ALB를 보호하는 것은 드뭅니다. 프라이빗 ALB를 보호할 수 있는 몇 가지 상황은 다음과 같습니다:

* 인터넷 또는 제어하지 않는 네트워크에서 필터링되지 않은 트래픽을 간접적으로 처리하는 경우 AWS WAF로 프라이빗 ALB를 보호하십시오. 소스 IP 주소가 보존되지 않으면 [IP 주소를 전달](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-forwarded-ip-address.html)하는 HTTP 헤더를 검사해야 할 수 있습니다. 일반적으로 이 헤더는 `X-Forwarded-For`입니다.
* 프라이빗 ALB가 좁은 [VPC 보안 그룹 인그레스 규칙](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html)과 같은 다른 완화 제어 없이 자체 네트워크의 위협에 노출되는 경우 AWS WAF로 보호하는 것을 고려하십시오.

### Amazon S3 버킷을 가리키는 Amazon CloudFront 배포

Amazon S3 버킷을 가리키는 오리진만 있는 CloudFront 배포를 보호하는 것은 드뭅니다. AWS는 악의적인 요청으로부터 Amazon S3 엔드포인트를 보호할 책임이 있습니다.

![](../images/waf-cloudfront-s3.png)
**그림 3:** Amazon S3 버킷 오리진이 있는 CloudFront 배포 보호

AWS WAF [지리적 일치 문](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-geo-match.html)을 사용하여 원산지 지역을 기반으로 요청을 차단할 수 있습니다. 특정 지리적 위치의 사용자가 CloudFront에서 배포하는 콘텐츠에 액세스하지 못하도록 해야 하는 경우 CloudFront에서 기본 [지리적 제한](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/georestrictions.html)을 사용할 수 있습니다. CloudFront 지리적 제한 기능을 사용하는 경우 이 기능은 차단된 요청을 AWS WAF로 전달하지 않습니다. 지리 및 기타 AWS WAF 기준을 기반으로 요청을 차단하려면 AWS WAF 지리적 일치 문을 사용하고 CloudFront 지리적 제한 기능을 사용하지 마십시오.

[오리진 액세스 제어](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)(OAC)를 사용하여 Amazon S3가 CloudFront 배포의 요청만 허용하도록 버킷에 대한 액세스를 제한하십시오.

### Network Load Balancer

AWS WAF 웹 ACL은 Network Load Balancer(NLB)와 연결할 수 없습니다. NLB가 HTTP 트래픽을 처리하는 경우 대신 Application Load Balancer(ALB) 사용을 고려하십시오. 정적 IP 주소에서 수신 대기하기 위해 NLB를 사용하는 경우 [NLB의 대상으로 ALB를 생성](https://aws.amazon.com/blogs/networking-and-content-delivery/application-load-balancer-type-target-group-for-network-load-balancer/)하고 AWS WAF로 ALB를 보호할 수 있습니다. 기본적으로 NLB는 ALB 대상으로 전송된 트래픽의 클라이언트 IP를 보존합니다. 이는 IP 기반 AWS WAF 규칙이 제대로 작동하는 데 중요합니다.

![](../images/waf-nlb.png)
**그림 4:** AWS WAF와 함께 ALB를 사용하여 NLB 대상 보호

### AWS 외부에서 호스팅되는 HTTP 엔드포인트

[IP 주소 대상](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html#target-group-ip-address-type)이 있는 ALB를 생성할 수 있습니다. VPC에 엔드포인트 네트워크로의 경로가 있는 경우 모든 [RFC 1918](https://datatracker.ietf.org/doc/html/rfc1918) 주소를 대상으로 지정할 수 있습니다. RFC 1918 외부의 IP 대상을 사용해야 하는 경우 AWS Support에 문의하십시오.

![](../images/waf-non-aws-target.png)
**그림 5:** AWS WAF를 사용하여 AWS 외부의 엔드포인트 보호

### CloudFront 배포 뒤의 Application Load Balancer

CloudFront 배포의 오리진인 Application Load Balancer(ALB)를 보호하기 위해 AWS WAF를 사용하는 것은 드뭅니다. CloudFront 엣지 로케이션에서 원치 않는 트래픽을 필터링하려면 CloudFront 배포와 웹 ACL을 연결하십시오. CloudFront는 `X-Forwarded-For` HTTP 헤더에서 클라이언트 IP를 ALB로 전달하지만 제대로 작동하지 않는 일부 IP 기반 규칙이 있습니다. 예를 들어 [Amazon IP 평판 목록](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-ip-rep.html#aws-managed-rule-groups-ip-rep-amazon) 및 [익명 IP 목록](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-ip-rep.html#aws-managed-rule-groups-ip-rep-anonymous)은 현재 전달된 IP를 지원하지 않습니다.

![](../images/waf-cloudfront-alb.png)
**그림 6:** AWS WAF를 사용하여 CloudFront 뒤의 ALB 보호

CloudFront는 ALB 오리진이 인터넷에서 도달할 수 있어야 합니다. ALB가 CloudFront 배포의 인터넷 트래픽만 수락하도록 하려면 다음 단계를 따르십시오.

1. [Amazon CloudFront용 AWS 관리형 접두사 목록을 사용하여 오리진에 대한 액세스 제한](https://aws.amazon.com/blogs/networking-and-content-delivery/limit-access-to-your-origins-using-the-aws-managed-prefix-list-for-amazon-cloudfront/)
2. [CloudFront에서 추가한 고객 헤더가 포함된 경우에만 요청에 응답하도록 ALB 구성](https://aws.amazon.com/blogs/security/how-to-enhance-amazon-cloudfront-origin-security-with-aws-waf-and-aws-secrets-manager/). 이렇게 하면 오리진이 CloudFront 배포의 요청만 수락합니다.

### CloudFront가 아닌 콘텐츠 전송 네트워크 뒤의 AWS 리소스

애플리케이션이 CloudFront 이외의 CDN을 사용하는 경우 리전 AWS WAF 웹 ACL로 오리진을 보호할 수 있습니다. AWS WAF는 클라이언트 IP를 직접 볼 수 없으므로 CDN이 HTTP 헤더에서 클라이언트 IP를 전달하는 것에 의존해야 합니다. 일부 AWS 관리형 규칙 그룹은 전달된 IP를 지원하지 않습니다. 전달된 IP를 사용할 수 있는 규칙 문은 [전달된 IP 주소](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-forwarded-ip-address.html)를 참조하십시오.

## AWS WAF 로그 저장

AWS WAF 로그는 Amazon S3 또는 Amazon CloudWatch Logs에 저장할 수 있습니다. Amazon Kinesis Data Firehose를 사용하여 Amazon S3를 포함한 다양한 대상에 WAF 로그를 전달하도록 WAF를 구성할 수 있습니다.

### S3에 AWS WAF 로그 저장

Amazon S3는 AWS WAF 로그를 저장하는 가장 비용 효율적인 옵션입니다. S3 버킷에 로그를 저장하도록 AWS WAF를 구성하는 두 가지 방법이 있습니다.

1. Kinesis Data Firehose 사용
2. 대상 버킷을 지정하여 직접. 이것이 더 간단하지만 로그 볼륨이 월 800억 이벤트를 초과하지 않는 한 KDF를 사용하는 것보다 비용이 더 많이 듭니다.

AWS WAF 로그에 Amazon S3를 사용하는 경우 일반적으로 중앙 버킷에 저장됩니다. 웹 ACL에서 모든 계정 또는 리전의 버킷 ARN을 지정할 수 있습니다. 필요한 버킷 정책 변경에 대한 정보는 [문서](https://docs.aws.amazon.com/waf/latest/developerguide/logging-s3.html)를 참조하십시오.

WAF 동작을 모니터링하기 위해 Amazon S3의 로그를 사용하는 방법에 대한 정보는 [WAF 규칙 모니터링](../../monitoring-waf-rules/docs/)을 참조하십시오.

### CloudWatch Logs에 AWS WAF 로그 저장

[구독 필터](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Subscriptions.html)를 사용하여 사용자 지정 처리, 분석 또는 다른 시스템으로 로드하기 위해 Amazon Kinesis 또는 AWS Lambda와 같은 다른 서비스로 AWS WAF 로그를 전달할 수 있습니다.

[메트릭 필터](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringLogData.html)를 사용하여 로그 이벤트를 구문 분석하고 숫자 값을 CloudWatch 메트릭으로 추출할 수 있습니다. 이러한 메트릭은 알람 또는 대시보드 구성에 사용할 수 있습니다.

중앙 계정의 CloudWatch 로그 그룹으로 AWS WAF 로그를 중앙 집중화할 수 있지만 이는 기본적으로 지원되지 않으며 간단하지 않은 설정이 필요합니다. 먼저 웹 ACL과 동일한 리전(전역 웹 ACL의 경우 `us-east-1`)에 CloudWatch 로그 그룹을 설정합니다. 그런 다음 [AWS Lambda를 사용한 구독 필터](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/SubscriptionFilters.html#LambdaFunctionExample)를 정의합니다. Lambda 함수는 중앙 계정의 CloudWatch 로그 그룹에 로그 이벤트를 씁니다. 현재로서는 KDF를 사용하여 계정 간에 데이터를 스트리밍할 수 없습니다.

WAF 동작을 모니터링하기 위해 CloudWatch Logs의 로그를 사용하는 방법에 대한 정보는 [WAF 규칙 모니터링](../../monitoring-waf-rules/docs/)을 참조하십시오.

## WAF 로그 필터링

민감한 데이터가 포함될 수 있는 필드의 로깅을 피하려면 [AWS WAF 로그에서 필드를 생략](https://docs.aws.amazon.com/waf/latest/developerguide/logging-management.html)할 수 있습니다. AWS WAF는 HTTP 본문을 로깅하지 않습니다.

AWS WAF 로그는 최대 100개의 규칙 레이블을 포함할 수 있으므로 상세합니다. AWS WAF 로그 저장 비용을 최적화하려면 로그를 유지하거나 삭제할지 결정하는 조건을 지정할 수 있습니다([로그 필터링](https://docs.aws.amazon.com/waf/latest/developerguide/logging-management.html) 참조). HTTP 요청 필드의 값이 아닌 규칙 레이블 및 규칙 작업을 기반으로만 필터링할 수 있습니다.

## 비용 추정

[AWS WAF 가격](https://aws.amazon.com/waf/pricing/)을 검토하는 동안 고려해야 할 몇 가지 주제가 있습니다.

요청당 가격은 두 가지 요인에 따라 증가합니다.

* 웹 ACL이 기본 할당인 1,500을 초과하여 사용하는 500 WCU당 백만 요청당 추가 $0.20
* 기본 본문 검사 제한(CloudFront 웹 ACL의 경우 16KB, 리전 웹 ACL의 경우 8KB)을 초과하여 분석된 각 추가 16KB당 백만 요청당 추가 $0.30

다음 AWS WAF 요금은 AWS Shield Advanced로 보호되는 리소스에 대해 면제됩니다.

* AWS WAF에서 분석한 백만 요청당 $0.60
* 웹 ACL당 월 $5.00

Shield Advanced는 이러한 AWS WAF 요금을 포함하지 않습니다. 이 목록은 AWS WAF에서 더 많은 고급 기능이 지원됨에 따라 시간이 지남에 따라 증가할 수 있습니다.

* 1,500 WCU 이상 사용 시 추가 요금
* 기본 제한보다 큰 본문 검사 시 추가 요금
* Fraud Control 계정 생성 사기 방지 요금
* Fraud Control 계정 탈취 방지 요금
* Bot Control 요금
* CAPTCHA 및 Challenge 작업 요금

CloudFront는 AWS WAF에 의해 차단된 경우에도 요청에 대해 요금을 부과합니다. AWS Shield Advanced는 Shield Advanced로 보호되는 CloudFront 배포에서 DDoS 관련 사용량 급증으로 인한 확장 요금으로부터 보호하기 위해 DDoS 비용 보호를 제공합니다. AWS Support를 통해 [크레딧을 요청](https://docs.aws.amazon.com/waf/latest/developerguide/request-refund.html)할 수 있습니다. CloudFront 및 ALB 보호 리소스에 대한 크레딧을 받을 자격이 되려면 AWS WAF 웹 ACL을 연결하고 웹 ACL에 속도 기반 규칙을 구현해야 합니다.

## WAF 규칙의 소유권

간단한 조직에서는 애플리케이션 팀이 일반적으로 자체 AWS WAF 규칙을 관리합니다. 웹 ACL을 생성하고 그 안의 모든 규칙을 소유합니다. 또한 자체 계정에 AWS WAF 로그를 보관하는 경향이 있습니다.

대규모 조직은 모든 웹 애플리케이션에 AWS WAF 규칙의 최소 기준선이 있어야 한다는 회사 전체 보안 정책을 갖는 경향이 있습니다. 보안 팀은 규정 준수 보고 및 자동 수정을 위해 웹 애플리케이션을 감사해야 합니다. 고객은 AWS Firewall Manager를 사용하여 필수 규칙을 시행하는 AWS WAF 정책을 정의합니다. 애플리케이션 팀은 애플리케이션별 규칙으로 FMS 관리형 웹 ACL을 사용자 지정할 수 있습니다. 이러한 방식으로 웹 ACL에는 필수 규칙과 사용자 지정 규칙이 혼합되어 있습니다.

애플리케이션 팀은 조직이 사용하는 책임 모델을 알고 있어야 합니다. 보안 팀은 필수 규칙을 업데이트하려고 할 수 있으며 이를 위해서는 조정된 테스트가 필요합니다. 또한 FMS를 사용하는 경우 AWS WAF 로그는 일반적으로 애플리케이션 팀이 액세스해야 할 수 있는 중앙 S3 버킷에 저장됩니다.

## 사용할 웹 ACL 수

AWS WAF 웹 ACL은 ALB와 같은 애플리케이션 리소스와 연결됩니다. 웹 ACL은 동일한 계정의 하나 이상의 리소스와 연결할 수 있습니다. 웹 ACL을 공유하거나 전용으로 사용해야 하는 경우는 언제입니까? 답은 단순성, 격리 및 비용 간의 균형이 필요합니다.

간단하게 유지하려면 하나의 웹 ACL을 사용하여 여러 리소스를 보호할 수 있습니다. 이는 여러 리소스가 동일한 AWS WAF 규칙을 공유한다는 것을 의미합니다. 한 애플리케이션에 사용자 지정 규칙이 필요한 경우 동일한 웹 ACL을 공유하는 다른 애플리케이션의 요청에 영향을 줄 수 있습니다. 각 애플리케이션에 고유한 규칙이 있는 경우 공유 대신 전용 웹 ACL을 선호할 수 있습니다.

비용은 웹 ACL 공유가 합리적인지 결정하는 또 다른 요소입니다. AWS WAF 요금은 요청 볼륨과 웹 ACL당 및 규칙당 요금을 기반으로 합니다. 계정의 애플리케이션이 월 1천만 건 이상의 요청을 처리하는 경우 요청 볼륨이 지배적인 가격 요소입니다. 이 경우 하나 대신 많은 웹 ACL을 사용해도 비용에 큰 영향을 미치지 않습니다. 반면에 보호할 리소스가 많고 해당 리소스가 월 소수의 요청을 처리하는 경우 비용을 최적화하기 위해 웹 ACL을 통합하는 것이 합리적일 수 있습니다.
