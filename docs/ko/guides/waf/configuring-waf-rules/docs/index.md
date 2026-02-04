# WAF 규칙 구성

## 종료 작업 이해

이해해야 할 가장 중요한 개념 중 하나는 *Allow*와 *Block*이 [종료 작업](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl-rule-actions.html)이라는 것입니다. 규칙이 일치하고 이러한 작업 중 하나를 가지면 더 이상 규칙이 평가되지 않습니다. 특히 웹 ACL 상단 근처에서 *Allow* 작업을 사용할 때 주의하십시오. *Allow* 작업이 있는 규칙은 후속 규칙이 차단했을 요청도 허용합니다(거짓 음성).

## 규칙 선택을 위한 일반적인 접근 방식

이 섹션에서는 WAF 규칙 선택을 위한 일반적인 사고 프로세스를 설명합니다. 웹 ACL의 기본 작업이 *Allow*라고 가정하면 웹 ACL 상단 근처에서 가능한 한 많은 원치 않는 요청을 차단하는 것이 합리적입니다. 가장 광범위한 원치 않는 트래픽에 적용되는 규칙을 상단에 배치하십시오. 좁은 기준을 가지거나 요청당 요금이 있는 규칙을 끝에 배치하십시오. 규칙의 정확한 순서를 규정하기보다는 상단, 중간 및 하단 범주로 그룹화합니다.

### 상단에 배치할 규칙

* 요청 플러드 차단을 위한 [속도 기반 규칙](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html)
* [Amazon IP 평판 목록](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-ip-rep.html#aws-managed-rule-groups-ip-rep-amazon) 관리형 규칙 그룹
* [익명 IP 목록](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-ip-rep.html#aws-managed-rule-groups-ip-rep-anonymous) 관리형 규칙 그룹
* 원산지 지역을 기반으로 요청을 차단하거나 속도 제한하기 위한 [지리적 기반 규칙](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-geo-match.html)

### 중간에 배치할 규칙

* 예상되는 HTTP 요청 필드(사용자 에이전트, 헤더)를 검증하는 사용자 지정 규칙
* OWASP Top 10 위협을 차단하기 위한 [AWS Core 규칙 세트](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-baseline.html#aws-managed-rule-groups-baseline-crs)
* [SQL 데이터베이스](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-use-case.html#aws-managed-rule-groups-use-case-sql-db) 관리형 규칙 그룹(SQL을 사용하는 애플리케이션에만 해당)
* [알려진 잘못된 입력](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-baseline.html#aws-managed-rule-groups-baseline-known-bad-inputs) 관리형 규칙 그룹(Java를 사용하는 애플리케이션에만 해당)

### 하단에 배치할 규칙

* [Bot Control](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-bot.html) 규칙 그룹(적용 가능성을 제한하기 위한 범위 축소 문과 함께)
* [Fraud Control 계정 탈취 방지](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-atp.html) 규칙 그룹(적용 가능성을 제한하기 위한 범위 축소 문과 함께)

## 웹 ACL 용량 단위(WCU)

각 웹 ACL에는 [웹 ACL 용량 단위](https://docs.aws.amazon.com/waf/latest/developerguide/aws-waf-capacity-units.html)(WCU)로 측정되는 용량이 있습니다. 이 용량은 웹 ACL에 추가하는 규칙 및 규칙 그룹에서 사용됩니다. 기본적으로 용량은 1,500 WCU입니다.

이 제한에 불필요하게 도달하지 않도록 웹 ACL에 보호되는 애플리케이션에 필요한 규칙만 포함되어 있는지 확인하십시오. 이제 제한 증가를 요청하지 않고도 웹 ACL당 최대 5,000 WCU를 사용할 수 있지만 추가 요금이 발생합니다.

## 웹 ACL의 기본 작업 선택

웹 ACL에 종료 작업이 있는 일치하는 규칙이 없으면 웹 ACL은 [기본 작업](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl-default-action.html)을 적용합니다. 두 가지 가능한 기본 작업이 있습니다: *Allow*와 *Block*. 대부분의 고객은 기본 작업을 *Allow*로 설정합니다.

대부분의 요청을 허용하고 공격자만 차단하려면 *Allow* 기본 작업을 사용하십시오. 예를 들어 악의적인 필드가 있거나 신뢰할 수 없는 소스 IP 또는 지역에서 발생한 요청만 차단하려고 할 수 있습니다.

특정 요청을 허용하고 다른 모든 것을 차단하려면 *Block* 기본 작업을 사용하십시오. 예를 들어 특정 IP 범위의 요청이나 특정 값을 포함하는 요청만 허용하려고 할 수 있습니다.

## 속도 기반 규칙

WAF [속도 기반 규칙](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html)은 최대 10,000개의 소스 IP(속도 기반 규칙당)에서 요청을 계산하고 클라이언트가 후행 5분 창에서 측정된 임계값을 초과하면 요청을 차단합니다. 이것은 AWS WAF의 자주 간과되는 기능이지만 웹 ACL에 추가할 수 있는 가장 간단하고 가치 있는 규칙 중 하나입니다. 대부분의 고객은 모든 웹 ACL에 최소한 하나의 속도 기반 규칙을 가져야 합니다.

속도 기반 규칙을 활용하는 방법에 대한 전체 설명은 블로그 [가장 중요한 세 가지 AWS WAF 속도 기반 규칙](https://aws.amazon.com/blogs/security/three-most-important-aws-waf-rate-based-rules/)을 참조하십시오. 요약하면 가장 중요한 세 가지 속도 기반 규칙은 다음과 같습니다:

1. 모든 요청에 적용되는 포괄적인 속도 기반 규칙
2. 더 제한적인 제한으로 애플리케이션의 특정 부분을 보호하기 위한 URI별 속도 기반 규칙
3. 알려진 악의적인 소스 IP의 요청 속도를 제한하는 속도 기반 규칙

AWS Shield Advanced는 Shield Advanced로 보호되는 ALB 및 CloudFront 배포에서 DDoS 관련 사용량 급증으로 인한 확장 요금으로부터 보호하기 위해 DDoS 비용 보호를 제공합니다. AWS Support를 통해 [크레딧을 요청](https://docs.aws.amazon.com/waf/latest/developerguide/request-refund.html)할 수 있습니다. CloudFront 및 ALB 보호 리소스에 대한 크레딧을 받을 자격이 되려면 AWS WAF 웹 ACL을 연결하고 웹 ACL에 속도 기반 규칙을 구현해야 합니다.

## 규칙 레이블 사용

[레이블](https://docs.aws.amazon.com/waf/latest/developerguide/waf-labels.html)은 일치하는 규칙에 의해 웹 요청에 추가되는 메타데이터입니다. 레이블을 사용하여 한 규칙의 결과를 다른 규칙에서 사용할 수 있도록 합니다. 레이블은 [레이블 일치 문](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-label-match.html)을 사용하여 동일한 웹 ACL의 하위(상위가 아님) 다른 규칙에서 검사할 수 있습니다.

종료 작업이 있는 규칙에 의해 추가된 레이블은 다른 규칙에서 검사할 수 없습니다. 이러한 레이블은 [WAF 로그 레코드](https://docs.aws.amazon.com/waf/latest/developerguide/logging-fields.html) 및 [CloudWatch 메트릭 차원](https://docs.aws.amazon.com/waf/latest/developerguide/monitoring-cloudwatch.html#waf-metrics)에 포함되므로 종료 규칙의 동작을 분석하고 시각화할 수 있습니다.

레이블은 일반적으로 관리형 규칙의 동작을 보강하는 데 사용됩니다. 첫 번째 단계는 관리형 규칙의 작업을 *Block*에서 *Count*로 전환하는 것입니다. 그런 다음 관리형 규칙의 레이블과 요청을 차단해야 하는지 결정하는 다른 조건과 일치하는 다른 규칙을 아래에 만듭니다.

레이블 사용에 대한 자세한 내용은 [AWS WAF용 AWS 관리형 규칙의 동작을 사용자 지정하는 방법](https://aws.amazon.com/blogs/security/how-to-customize-behavior-of-aws-managed-rules-for-aws-waf/)을 참조하십시오.

## 관리형 규칙 그룹 사용

### 관리형 규칙 그룹 제공업체

[AWS 관리형 규칙 그룹](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups.html)은 Amazon의 위협 연구 팀에서 유지 관리합니다. 이러한 규칙에는 Bot Control 및 Fraud Control 규칙 그룹을 제외하고 요청당 요금이 없습니다. 대부분의 고객에게 적합한 AWS 관리형 규칙 그룹에 대한 권장 사항은 위를 참조하십시오.

AWS Marketplace를 사용하여 [AWS 파트너가 제공하는 관리형 규칙](https://docs.aws.amazon.com/waf/latest/developerguide/marketplace-managed-rule-groups.html)을 구독할 수 있습니다. 이러한 규칙 그룹은 WAF 웹 ACL에서 실행되지만 추가 요금이 발생합니다.

Firewall Manager 정책에서 AWS Marketplace 규칙 그룹을 사용하려면 조직의 각 계정이 먼저 해당 규칙 그룹을 구독해야 합니다.

### 버전 관리

관리형 규칙 그룹 제공업체는 규칙을 업데이트해야 할 수 있습니다. [버전 관리](https://docs.aws.amazon.com/waf/latest/developerguide/waf-managed-rule-groups-versioning.html)를 지원하는 규칙 그룹의 경우 제공업체가 사용하는 버전을 관리하도록 선택하거나(기본값) 버전 설정을 직접 관리할 수 있습니다(정적).

기본 버전을 사용하는 경우 새 버전의 [알림을 구독](https://docs.aws.amazon.com/waf/latest/developerguide/waf-using-managed-rule-groups-sns-topic.html)하십시오. 이를 통해 새 기본값이 되기 전에 테스트할 시간을 주기 위해 예정된 변경 사항에 대해 알 수 있습니다.

정적 버전을 사용하는 경우 새 정적 버전을 인식할 수 있도록 알림을 구독하는 것도 중요합니다. 애플리케이션에 최신 보호 기능이 있는지 확인하려면 버전을 최신 상태로 유지하십시오. 테스트하기 전에 강제로 업그레이드되지 않도록 [버전 만료를 추적](https://docs.aws.amazon.com/waf/latest/developerguide/waf-using-managed-rule-groups-expiration.html)하십시오.

버전 관리에 대한 자세한 내용은 [AWS WAF용 AWS 관리형 규칙의 동작을 사용자 지정하는 방법](https://aws.amazon.com/blogs/security/how-to-customize-behavior-of-aws-managed-rules-for-aws-waf/)을 참조하십시오.

AWS 관리형 [IP 평판 규칙 그룹](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-ip-rep.html)은 버전을 사용하지 않습니다. 이러한 규칙 그룹은 Amazon 위협 인텔리전스의 진화를 기반으로 자주 업데이트됩니다.

### 범위 축소 문

[범위 축소 문](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-scope-down-statements.html)은 평가되는 요청 세트를 좁히기 위해 관리형 규칙 그룹 또는 속도 기반 규칙 내부에 추가하는 문입니다.

Bot Control 및 Fraud Protection과 같은 고급 관리형 규칙 그룹에서 범위 축소 문을 사용하여 규칙 그룹에서 평가해야 하는 요청을 지정하십시오. 범위 축소 문에서 제외된 요청에 대해서는 요금이 부과되지 않으므로 비용을 최적화하는 효과적인 방법입니다.

범위 축소 문에 대한 자세한 내용은 [AWS WAF용 AWS 관리형 규칙의 동작을 사용자 지정하는 방법](https://aws.amazon.com/blogs/security/how-to-customize-behavior-of-aws-managed-rules-for-aws-waf/)을 참조하십시오.

### 거짓 양성 처리

WAF 규칙은 종종 HTTP 요청 필드에 나타나는 패턴을 찾아 요청이 악의적인지 판단합니다. HTTP 요청에 나타날 수 있는 데이터의 무한한 다양성으로 인해 WAF 규칙이 악의적이지 않은 요청을 차단하는 경우가 있습니다. 이를 거짓 양성이라고 합니다.

거짓 양성을 유발하는 규칙을 확인하려면 WAF 로그를 쿼리하여 요청을 차단한 규칙의 레이블을 식별하십시오. 이 규칙이 요청을 잘못 차단하고 있으므로 이러한 요청을 허용하는 규칙을 작성하려는 본능이 있을 수 있습니다. _Allow_는 종료 작업이고 규칙이 다른 이유로 차단되어야 했던 요청을 허용할 수 있기 때문에 문제가 됩니다.

관리형 규칙으로 인한 거짓 양성을 수정하려면 다음 두 단계를 따르십시오. 예를 들어 [알려진 잘못된 입력](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-baseline.html#aws-managed-rule-groups-baseline-known-bad-inputs)에 대한 AWS 관리형 규칙 그룹 내의 `Log4JRCE_BODY` 규칙이 URI `/reports`에 대해 거짓 양성을 유발한다고 가정합니다.

1. `Log4JRCE_BODY`의 작업을 _Block_에서 _Count_로 전환합니다. 이 규칙과 일치하는 요청은 `awswaf:managed:aws:known-bad-inputs:Log4JRCE_Body`로 레이블이 지정됩니다.
2. 관리형 규칙 그룹 아래에 다음 동작을 가진 규칙을 작성합니다:

```
IF Label = awswaf:managed:aws:known-bad-inputs:Log4JRCE_Body AND
   URI != "/reports"
THEN Block
```

관리형 규칙을 _Count_ 모드로 설정하면 관리형 규칙에 의해 레이블이 지정된 요청을 차단하는 해당 규칙을 작성해야 합니다. 그렇지 않으면 거짓 음성이 발생합니다.

거짓 양성 처리에 대한 자세한 내용은 [AWS WAF용 AWS 관리형 규칙의 동작을 사용자 지정하는 방법](https://aws.amazon.com/blogs/security/how-to-customize-behavior-of-aws-managed-rules-for-aws-waf/)을 참조하십시오.

## 대용량 HTTP 요청 처리

AWS WAF에는 검사할 수 있는 HTTP 요청 구성 요소의 크기와 수에 제한이 있습니다. 자세한 내용은 [AWS WAF에서 초과 크기 웹 요청 구성 요소 처리](https://docs.aws.amazon.com/waf/latest/developerguide/waf-oversize-request-components.html)를 참조하십시오. 다음은 크기 제한에 대한 요약입니다.

* AWS WAF는 CloudFront 웹 ACL에 대해 최대 64KB의 요청 본문을 검사할 수 있습니다. 리전 웹 ACL의 경우 AWS WAF는 최대 8KB의 본문을 검사할 수 있습니다.
* 모든 웹 ACL에 대해 AWS WAF는 8KB의 헤더 또는 쿠키 또는 처음 200개의 헤더 또는 쿠키 중 먼저 도달하는 제한을 검사할 수 있습니다.

일부 초과 크기 요청을 허용해야 하는 경우 해당 요청만 명시적으로 *Allow*하는 규칙을 추가하십시오. 동일한 구성 요소를 검사하는 웹 ACL의 다른 규칙보다 먼저 실행되도록 해당 규칙의 우선 순위를 지정하십시오. 다음은 특정 URI 및 HTTP 메서드에 대해서만 초과 크기 요청을 허용하는 예입니다.

```
IF Body size > 8,192 (with oversize handling set to Match) AND
   URI path starts with "/upload" AND
   HTTP method exactly matches "POST"
THEN Allow
```

다른 모든 요청의 경우 크기 제한이 있는 구성 요소를 검사하고 제한을 초과하는 요청을 *Block*하는 규칙을 추가하십시오. 이렇게 하면 검사되지 않은 요청 콘텐츠가 애플리케이션에 도달하는 것을 방지합니다.

```
IF Body size > 8,192 (with oversize handling set to Match) OR
   All headers size > 8,192 (with oversize handling set to Match)
THEN Block
```
