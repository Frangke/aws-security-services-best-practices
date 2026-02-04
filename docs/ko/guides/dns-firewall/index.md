# DNS Firewall 모범 사례 가이드

## 개요

Amazon Route 53 Resolver DNS Firewall은 VPC에서 아웃바운드 DNS 쿼리를 제어하고 필터링할 수 있는 관리형 방화벽 서비스입니다. 알려진 악성 도메인에 대한 DNS 쿼리를 차단하고 DNS 프로토콜을 사용한 유출 시도를 차단하여 DNS 기반 위협으로부터 워크로드를 보호하는 데 도움이 됩니다.

## Amazon Route 53 Resolver DNS Firewall 활성화의 이점은 무엇인가요?

Amazon Route 53 Resolver DNS Firewall을 활성화하면 다음과 같은 주요 이점을 제공합니다:

* **향상된 보안**: 멀웨어, 피싱, 명령 및 제어 공격을 포함한 DNS 기반 위협으로부터 VPC를 보호합니다.
* **운영 오버헤드 감소**: 자동으로 업데이트되는 AWS 관리형 도메인 목록을 활용하여 보안 팀의 부담을 줄입니다.
* **사용자 정의 가능한 보호**: 특정 보안 요구 사항을 해결하거나 알려진 위협을 차단하기 위해 사용자 정의 도메인 목록을 생성하고 관리합니다.
* **고급 위협 탐지**: DNS Firewall Advanced 규칙 그룹을 활용하여 터널링 및 유출과 같은 정교한 DNS 공격으로부터 보호합니다.
* **중앙 집중식 관리**: AWS Firewall Manager와 함께 사용하면 여러 계정 및 VPC에서 DNS Firewall 규칙을 쉽게 배포하고 관리할 수 있습니다.
* **비용 최적화**: DNS 계층에서 악성 트래픽을 필터링하여 Network Firewall과 같은 다운스트림 보안 제어에서 불필요한 데이터 처리 비용을 줄입니다.
* **원활한 통합**: 기존 AWS 서비스 및 현재 보안 아키텍처와 쉽게 통합됩니다.
* **확장성**: 추가 인프라 관리 없이 DNS 트래픽을 처리하도록 자동으로 확장됩니다.

Route 53 Resolver DNS Firewall을 구현함으로써 조직은 보안 태세를 크게 향상시키고 DNS 기반 위협으로부터 AWS 리소스를 보호할 수 있습니다.

## 모범 사례

### AWS 관리형 도메인 목록으로 방어 계층 구현

* 첫 번째 방어선으로 AWS 관리형 도메인 목록을 활용합니다
* 이러한 목록은 AWS Security에 의해 자동으로 업데이트됩니다

[참조: AWS 관리형 도메인 목록 문서](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver-dns-firewall-managed-domain-lists.html)

### DNS Firewall Advanced 규칙 그룹 활용

* DNS Firewall Advanced 규칙 그룹을 구현하여 다음으로부터 보호합니다:
    * DNS 터널링
    * 도메인 생성 알고리즘

[참조: DNS Firewall Advanced 기능](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/firewall-advanced.html)

### AWS Firewall Manager로 중앙 집중식 관리

* AWS Firewall Manager를 사용하여:
    * 조직 전체에 DNS Firewall 규칙을 일관되게 배포합니다
    * 새 VPC가 생성될 때 자동으로 보호합니다
    * 계정 전체에서 규칙을 중앙에서 관리합니다

[Firewall Manager 문서](https://docs.aws.amazon.com/waf/latest/developerguide/getting-started-fms-dns-firewall.html)

### DNS 쿼리 로깅 활성화 및 구성

* 다음을 위해 DNS 쿼리 로깅을 활성화합니다:
    * 보안 조사 및 위협 헌팅
    * 트래픽 패턴 분석
    * Amazon CloudWatch Logs 또는 S3로 로깅을 구성합니다
    * 적절한 로그 보존 정책을 설정합니다

[참조: DNS 쿼리 로깅 구성](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/firewall-resolver-query-logs-configuring.html)

### 소스에 가까운 곳에서 악성 트래픽 차단

* DNS Firewall을 초기 필터링 메커니즘으로 사용합니다
* Network Firewall에 도달하기 전에 DNS 계층에서 악성 트래픽을 차단합니다
* 불필요한 데이터 처리 비용을 줄입니다
* 다른 보안 제어와 함께 구현합니다

## 구현 가이드

### 초기 설정

1. DNS Firewall 규칙 그룹을 생성합니다
2. AWS 관리형 도메인 목록 및 DNS Firewall Advanced 규칙을 연결합니다
3. 필요한 경우 사용자 정의 도메인 목록을 구성합니다
4. 적절한 작업(ALLOW, ALERT, BLOCK)으로 사용자 정의 규칙을 생성합니다
5. 규칙 그룹을 VPC와 연결합니다

[참조: 시작 가이드](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver-dns-firewall-getting-started.html)

## 모니터링 및 유지 관리

* DNS 쿼리 로그를 정기적으로 검토합니다
* 규칙 구성을 검토하고 조정합니다
* 규칙 효과성을 검증합니다

## 권장 규칙 그룹 구성

* 권장 DNS Firewall 규칙 그룹 구성은 다음 링크를 참조하십시오: [권장 규칙 그룹 구성](https://github.com/aws-samples/amazon-route-53-resolver-dns-firewall-automation-examples/blob/main/sample-rule-group/template.yaml) 

## 추가 리소스

* [DNS Firewall 개요](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver-dns-firewall-overview.html)
* [자동화된 허용 목록 생성기 솔루션](https://github.com/aws-samples/amazon-route-53-resolver-dns-firewall-automation-examples/tree/main/AllowListGenerator)
* [AWS 보안 블로그 - 고급 DNS 위협으로부터 보호](https://aws.amazon.com/blogs/security/protect-against-advanced-dns-threats-with-amazon-route-53-resolver-dns-firewall/)
* [도메인 목록 관리 문서](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver-dns-firewall-managed-domain-lists.html)
