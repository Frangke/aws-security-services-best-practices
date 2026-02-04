# Security Hub

## 소개

AWS Security Hub 모범사례 가이드에 오신 것을 환영합니다. 이 가이드의 목적은 AWS 리소스에 대한 자동화되고 지속적인 보안 모범사례 검사를 위해 AWS Security Hub를 활용하는 규범적 지침을 제공하는 것입니다. GitHub를 통해 이 가이드를 게시함으로써 서비스 개선사항과 사용자 커뮤니티의 피드백을 포함한 시기적절한 권장사항을 제공하기 위해 신속한 반복이 가능합니다. 이 가이드는 단일 계정에서 처음으로 Security Hub를 배포하는 경우나 기존 다중 계정 배포에서 Security Hub를 최적화하는 방법을 찾고 있는 경우 모두에게 가치를 제공하도록 설계되었습니다.

## 이 가이드 사용 방법

이 가이드는 AWS 계정(및 리소스) 내에서 위협 및 악의적인 활동의 모니터링과 해결을 담당하는 보안 실무자를 대상으로 합니다. 모범사례는 더 쉬운 이해를 위해 세 가지 카테고리로 구성되어 있습니다. 각 카테고리에는 간략한 개요로 시작하여 지침 구현을 위한 세부 단계가 이어지는 일련의 모범사례가 포함되어 있습니다. 주제는 특정 순서로 읽을 필요가 없습니다:

* [Security Hub란 무엇인가](#what-is-security-hub)
* [Security Hub 활성화의 이점](#what-are-the-benefits-of-enabling-security-hub)
* [시작하기](#getting-started)
    * [배포 고려사항](#deployment-considerations)
    * [리전 고려사항](#region-considerations)
* [구현](#implementation)
    * [구성](#Configuration)
    * [보안 도구 통합](#integrate-your-security-tools)
    * [보안 표준 활성화](#enable-security-standards)
* [운영화](#operationalizing)
    * [중요 및 높음 수준 탐지 결과에 대한 조치](#take-action-on-critical-and-high-findings)
    * [사용자 정의 인사이트 생성](#create-customized-insights)
    * [사용 가능한 해결 지침 활용](#leverage-available-remediation-instructions)
    * [보안 표준 제어 미세 조정](#fine-tuning-security-standard-controls)
    * [자동화 규칙](#automation-rules)
    * [자동화된 대응 및 해결](#automated-response-and-remediation)
    * [타사 통합](#3rd-party-integrations)
* [비용 고려사항](#cost-considerations)
    * [AWS Config](#aws-config)
* [리소스](#resources)

## Security Hub란 무엇인가?

AWS Security Hub는 AWS 리소스에 대한 자동화되고 지속적인 보안 모범사례 검사를 수행하여 잘못된 구성을 식별하고, 보안 경고(즉, 탐지 결과)를 표준화된 형식으로 집계하여 보다 쉽게 강화, 조사 및 해결할 수 있도록 하는 클라우드 보안 태세 관리(CSPM) 서비스입니다. 보안 팀, 규정 준수 팀, 클라우드 아키텍트, 인시던트 대응 팀, 위험 관리 팀 및 MSSP가 사용할 수 있습니다. Security Hub는 현재 소규모 스타트업부터 대기업에 이르기까지 모든 규모의 고객이 사용하고 있습니다.

## Security Hub 활성화의 이점은 무엇인가?

Security Hub는 AWS 계정, 워크로드 및 리소스의 보안을 관리하고 개선하는 복잡성과 노력을 줄여줍니다. 특정 리전 내에서 몇 분 안에 Security Hub를 활성화할 수 있으며, 이 서비스는 일상적으로 가질 수 있는 기본적인 보안 질문에 답하는 데 도움이 됩니다. 주요 이점은 다음과 같습니다:

* 한 번의 클릭으로 보안 모범사례로부터의 편차를 감지합니다. Security Hub는 [AWS Foundational Security Best Practices standard](https://docs.aws.amazon.com/securityhub/latest/userguide/fsbp-standard.html) 및 [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services/), [National Institute of Standards and Technology (NIST)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final), [AWS Resource Tagging Standard](https://docs.aws.amazon.com/securityhub/latest/userguide/standards-tagging.html), [Payment Card Industry Data Security Standard (PCI DSS)](https://www.pcisecuritystandards.org/)를 포함한 기타 지원되는 업계 모범사례 및 표준의 제어에 대해 지속적이고 자동화된 계정 및 리소스 수준 구성 검사를 실행합니다. [Security Hub에서 지원되는 표준 및 제어](https://docs.aws.amazon.com/securityhub/latest/userguide/standards-reference.html)에 대해 자세히 알아보세요.
* AWS 및 파트너 서비스로부터 표준화된 데이터 형식으로 보안 탐지 결과를 자동으로 집계합니다. Security Hub는 Amazon GuardDuty의 위협 탐지 결과, Amazon Inspector의 취약성 탐지 결과, Amazon Macie의 민감한 데이터 탐지 결과와 같이 AWS 계정 전체에서 활성화된 보안 서비스로부터 탐지 결과를 수집합니다. Security Hub는 또한 표준화된 AWS Security Finding Format을 사용하여 파트너 보안 제품으로부터 탐지 결과를 수집하므로 시간이 많이 소요되는 데이터 파싱 및 정규화 작업이 필요하지 않습니다. 고객은 모든 계정의 탐지 결과에 액세스할 수 있는 관리자 계정을 지정할 수 있습니다.
* 자동화된 대응 및 해결 작업으로 평균 해결 시간을 단축합니다. Security Hub [Amazon EventBridge와의 통합](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-cloudwatch-events.html) 및 기타 [통합](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-partner-providers.html)을 사용하여 사용자 정의 자동화된 대응, 해결 및 강화 워크플로를 생성하여 Security Orchestration Automation and Response (SOAR) 및 Security Information and Event Management (SIEM) 워크플로를 만들 수 있습니다. 또한 Security Hub Automation Rules를 사용하여 거의 실시간으로 탐지 결과를 자동으로 업데이트하거나 억제할 수 있습니다.

## 시작하기

AWS Security Hub의 중요한 고려사항은 Security Hub를 관리할 수 있는 적절한 권한이 있는지 확인하고 AWS Organization에서 Security Hub 위임 관리자 계정으로 가장 적합한 계정을 고려해야 한다는 것입니다. 권한을 시작하려면 Security Hub를 관리하는 데 사용하는 역할에 최소한 AWS 관리형 정책 이름 "AWSSecurityHubFullAccess"가 있는지 확인하세요. 또 다른 고려사항은 AWS Security Hub가 대부분의 제어에 대한 보안 검사를 수행하기 위해 서비스 연결 AWS Config 규칙을 사용하므로 모든 계정에서 AWS Config가 활성화되어 있는지 확인하는 것입니다. 자세한 내용은 이 [문서](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-prereq-config.html)를 참조하세요.

### 배포 고려사항

AWS Organization 전체에 Security Hub를 배포하려면 콘솔, CLI 또는 API에서 수행하든 AWS 관리 계정과 보안 도구 계정에서 활성화해야 합니다. 보안 도구 계정의 개념에 익숙하지 않은 경우 Security Reference Architecture의 [권장 계정 구조](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/organizations.html)를 숙지하는 것이 좋습니다. 요약하자면, 이것은 Amazon Inspector, Amazon GuardDuty, Amazon Macie, Amazon Detective와 같은 네이티브 AWS 보안 서비스의 위임 관리자 계정으로 사용되는 AWS Organization의 전용 계정입니다.

### 리전 고려사항

Amazon Security Hub는 리전별 서비스입니다. 즉, Security Hub를 사용하려면 Security Hub를 활용하려는 모든 리전에서 활성화해야 합니다. AWS API를 사용하여 모든 계정과 리전에서 Security Hub를 활성화하거나 콘솔에서 리전 간 전환을 통해 이 작업을 수행할 수 있습니다. AWS Security Hub가 제공하는 또 다른 기능은 [Cross-Region aggregation](https://docs.aws.amazon.com/securityhub/latest/userguide/finding-aggregation.html)이라고 하며, 여기서 여러 리전의 탐지 결과, 탐지 결과 업데이트, 인사이트, 제어 규정 준수 상태 및 보안 점수를 선택한 단일 집계 리전으로 집계할 수 있습니다. 그런 다음 집계 리전에서 이 모든 데이터를 관리하여 교차 리전 배포를 단순화할 수 있습니다.

## 구현

AWS Organization에서 처음으로 Security Hub를 구현할 때 위에서 언급한 것처럼 Security Hub를 사용하려는 각 리전의 조직 관리 계정에서 위임 관리자를 설정합니다. 이 프로세스에 대한 자세한 내용은 [Security Hub 문서](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-accounts.html)를 참조하거나 아래 단계를 따르세요. 완료되면 이 가이드에서 다룰 Security Hub를 완전히 구성하기 위한 다른 단계가 있습니다.

### 구성

조직 관리 계정에서 위임 관리자를 설정하면 Security Hub가 활성화되지만 조직의 모든 기존 계정에 대한 적용 범위가 누락됩니다. 따라서 다음으로 계정 구성 설정으로 이동하거나 API를 사용하여 Security Hub [중앙 구성 정책](https://docs.aws.amazon.com/securityhub/latest/userguide/central-configuration-intro.html)을 사용하여 모든 멤버 계정에서 Security Hub를 활성화해야 합니다. 이 구성 정책을 사용하면 Security Hub가 활성화된 계정, 조직 단위 또는 리전과 활성화하려는 표준을 지정할 수 있습니다. 기본 정책 옵션은 모든 리전, 모든 계정을 선택하고 AWS Foundational Security Best Practices 표준을 활성화하며 AWS Organization에 추가된 새 계정에 대해 동일한 구성을 활성화합니다. 이렇게 하면 가시성 부족을 방지하고 앞으로 조직의 계정에 대해 개별적으로 Security Hub를 활성화하는 수동 작업을 절약할 수 있습니다. 중앙 구성을 사용하면 중앙 구성을 설정할 때 사용된 리전으로 탐지 결과 집계도 설정됩니다.

![Security Hub enable](../../images/SH-Enable.png)
*그림 1: Security Hub 계정 페이지*

중앙 구성을 사용하지 않고 대신 새 조직 계정에 로컬 구성을 사용하도록 선택할 수 있지만, 이 기능을 사용하면 각 계정과 리전에서 설정을 수동으로 구성해야 합니다. 로컬 구성을 사용하거나 중앙 구성 사용을 시작하지 않은 경우에도 [Security Hub 문서](https://docs.aws.amazon.com/securityhub/latest/userguide/finding-aggregation-enable.html)에 따라 집계 리전을 구성할 수 있습니다.

### 보안 도구 통합

Security Hub에 다양한 보안 도구를 통합할 수 있습니다. 여기에는 지원되는 [AWS 서비스](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-internal-providers.html), [타사 공급업체](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-partner-providers.html), AWS 사용자 정의 config 규칙, 심지어 자체 [사용자 정의 애플리케이션](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-custom-providers.html)과 같은 다양한 데이터 소스로부터 Security Hub로 다양한 탐지 결과를 수집하는 것이 포함됩니다. Security Hub 콘솔 통합 페이지는 각 통합에 대한 세부 정보와 활성화하기 위해 수행해야 하는 작업을 제공합니다. 뿐만 아니라 Security Hub에서 관리하는 이러한 탐지 결과를 지원되는 티켓팅 시스템, 채팅 소프트웨어 또는 SIEM 솔루션과 같은 다른 AWS 서비스 또는 통합된 타사 도구로 전달하여 탐지 결과의 경고 및 관리를 수행할 수 있습니다.

![Security Hub Finding Flow](../../images/SH-Finding-Flow.png)
*그림 2: Security Hub 탐지 결과 흐름*

### 보안 표준 활성화

기본적으로 기본 정책을 사용하여 Security Hub를 활성화하면 AWS Foundational Best Practices가 선택됩니다. 기본값으로 시작한 다음 CIS 또는 NIST 표준과 같이 비즈니스 요구사항을 충족하는 데 필요한 다른 보안 표준을 활성화하는 것이 좋습니다.

![Security Hub Standards](../../images/SH-Standards.png)
*그림 3: Security Hub 표준*

## 운영화

### 중요 및 높음 수준 탐지 결과에 대한 조치

대부분의 고객은 우선적으로 대응해야 할 중요 및 높음 심각도 탐지 결과에 집중합니다. 이러한 단계에서 언급되고 아래 그림과 같이 필터 옵션을 사용하는 것이 좋습니다. 중요 및 높음 탐지 결과가 무엇인지 이해하면 대응할 탐지 결과 유형을 이해할 수 있습니다. 그런 다음 이 작업을 완료하는 데 필요한 런북과 자동화를 생성할 수 있습니다.

* Severity label 및 Status로 탐지 결과를 필터링합니다. 필터는 대소문자를 구분한다는 점을 유념하세요.
* 검토 및 해결합니다.

![Security Hub finding](../../images/SH-Finding.png)
*그림 4: Security Hub 탐지 결과*

### 사용자 정의 인사이트 생성

AWS Security Hub의 인사이트를 사용하면 다양한 시각화를 통해 탐지 결과를 볼 수 있습니다. 대부분의 고객은 기본 인사이트를 사용하고 주의가 필요한 탐지 결과 추세에 집중하기 위해 사용자 정의 인사이트를 생성합니다. 다음은 인사이트 생성의 몇 가지 모범사례입니다:

* 환경의 컨텍스트를 사용하여 인사이트를 생성합니다.
* 'Group By' 필터를 사용하여 인사이트를 생성합니다. 예를 들어 AWS 리소스별로 탐지 결과를 그룹화하는 'ResourceType'을 사용하거나 다중 계정 설정에서 AWS 계정별로 탐지 결과를 그룹화하는 AWS account ID를 사용할 수 있습니다.
* 'Group By'를 사용하기 전에 필터를 추가하여 인사이트에 집중합니다. 예를 들어 Status EQUALS FAILED입니다.
* 팀이 작업 중인 보안 프로그램의 진행 상황을 시각화하고 추적하는 데 도움이 되는 인사이트를 생성합니다. 예를 들어 시간 경과에 따른 중요 취약성 감소입니다.

### 사용 가능한 해결 지침 활용

보안 또는 규정 준수 표준의 각 Security Hub 탐지 결과에는 관련 해결 지침이 있습니다. 이는 주어진 탐지 결과에 대응하는 방법에 대한 귀중한 인사이트를 제공할 수 있습니다.

![SH finding remediation](../../images/SH-Finding-Remediation.png)
*그림 5: Security Hub 탐지 결과 해결 지침*

### 보안 표준 제어 미세 조정

일부 고객은 보안 표준을 활성화할 때 환경에 적용되지 않는 하나 이상의 제어가 있을 수 있습니다. 이러한 탐지 결과의 경우 비활성화하고 기록 참조를 위해 이유에 대한 메모를 추가할 수 있습니다. 이는 아래와 같이 제어를 선택하고 제어 비활성화 버튼을 클릭하여 수행할 수 있습니다. 이렇게 하면 해당 제어에 대한 추가 탐지 결과 생성이 중지됩니다. 기존 탐지 결과의 경우 3-5일 후에 자동으로 아카이브됩니다.

![SH Control status](../../images/SH-Control-Status.png)
*그림 6: Security Hub 제어 상태*

일부 Security Hub 제어는 제어 평가 방법에 영향을 미치는 파라미터를 사용합니다. 일반적으로 이러한 제어는 Security Hub가 정의한 기본 파라미터 값에 대해 평가됩니다. 그러나 이러한 제어의 하위 집합에 대해서는 파라미터 값을 사용자 정의할 수 있습니다. 제어에 대한 파라미터 값을 사용자 정의하면 Security Hub는 지정한 값에 대해 제어 평가를 시작합니다. 이는 환경에 적용 가능한 특정 정보로 제어를 업데이트해야 하는 경우 활용할 수 있는 훌륭한 기능입니다. 사용자 정의 제어 파라미터는 아래와 같이 구성 정책을 사용하여 단일 또는 다중 계정 수준에서 구성할 수 있습니다. [사용자 정의 제어 파라미터 구성](https://docs.aws.amazon.com/securityhub/latest/userguide/custom-control-parameters.html)에 대한 자세한 내용은 문서를 참조하세요.

![SH Control parameters](../../images/SH-Control-Parameters.png)
*그림 7: Security Hub 사용자 정의 제어 파라미터 정책 구성*

### 자동화 규칙

[Automation Rules](https://docs.aws.amazon.com/securityhub/latest/userguide/automation-rules.html)를 사용하면 코드 없이 Security Hub 탐지 결과를 자동으로 업데이트하거나 억제할 수 있습니다. 이 기능을 사용하면 관리자가 규칙을 생성하여 클라우드 보안 태세 관리를 간소화하고 조직의 모든 계정에서 탐지 결과에 대해 조치를 취합니다. 이는 탐지 결과 수집 시점에 규칙이 실행되므로 거의 실시간으로 발생합니다. 이는 보안 팀의 반복적인 작업을 제거하고 평균 대응 시간을 줄이는 데 도움이 될 수 있습니다. 예를 들어 가장 일반적인 고객 사용 사례 중 하나는 개발자 계정에서 식별된 동일한 탐지 결과보다 집중해야 하는 주요 프로덕션 계정에 대한 탐지 결과 심각도를 높이는 것입니다. 프로덕션 계정 ID와 관련된 탐지 결과의 심각도 수준을 더 높은 심각도로 높이거나 개발자 계정 ID와 관련된 탐지 결과의 심각도를 낮추어 팀이 중요한 탐지 결과에만 집중할 수 있도록 할 수 있습니다. 또 다른 사용 사례는 리소스 태그를 사용하여 탐지 결과와 연결된 리소스와 자동화 규칙이 탐지 결과로 수행할 작업을 추가로 이해하는 것입니다.

![SH Automation Rules page](../../images/SH-Automation-Rules-Page.png)
*그림 8: Security Hub 자동화 규칙 페이지*

다음은 자동화 규칙과 관련된 몇 가지 예입니다:

* 탐지 결과가 특정 프로덕션 계정에 영향을 미치는 경우 탐지 결과 심각도를 HIGH에서 CRITICAL로 변경합니다.
* 'Informational' 심각도 레이블이 있는 Security Hub 탐지 결과를 "Suppressed" 워크플로 상태로 변경합니다.
* 탐지 결과의 리소스 ID가 특정 리소스(예: PII가 있는 S3 버킷)를 참조하는 경우 탐지 결과 심각도를 CRITICAL로 설정합니다.

예를 들어 프로덕션 환경에 대한 탐지 결과의 심각도를 높음에서 중요로 높이는 자동화 규칙을 생성했다고 가정해 보겠습니다. 이 자동화 규칙의 일부로 기준이 일치하면 아래 이미지와 같이 탐지 결과 심각도가 높음에서 중요로 높아지는 자동화된 작업이 수행됩니다.

![SH Automation rule](../../images/SH-Automation-Rule.png)
*그림 9: Security Hub 자동화 규칙*

다음은 자동화 규칙과 관련된 몇 가지 고려사항입니다:

* 규칙 설명을 포함하면 팀이 대응자 및 리소스 소유자에게 컨텍스트를 제공할 수 있습니다.
* Security Hub 관리자 계정만 자동화 규칙을 생성, 삭제, 편집 및 볼 수 있습니다.
* 자동화된 해결은 Security Hub 관리자 계정의 각 리전에서 생성되어야 합니다.
* 기준을 정의하고 멤버 계정 ID를 포함합니다.
* Security Hub는 12-24시간마다 또는 연결된 리소스가 상태를 변경할 때 제어 탐지 결과를 업데이트합니다.
* 규칙 순서가 중요합니다 - 여러 규칙이 동일한 탐지 결과 또는 탐지 결과 필드에 적용될 수 있습니다. 가장 낮은 숫자 값이 먼저입니다.
* 여러 탐지 결과가 동일한 규칙 순서를 가진 경우 Security Hub는 UpdatedAt 필드에 대해 더 이른 값을 가진 규칙을 먼저 적용합니다(즉, 가장 최근에 편집된 규칙이 마지막에 적용됨).
* Security Hub는 현재 관리자 계정에 대해 최대 100개의 자동화 규칙을 지원합니다.

### 자동화된 보안 대응

이 AWS 솔루션은 AWS Security Hub와 함께 작동하는 애드온으로, 보안 위협에 대한 업계 규정 준수 표준 및 모범사례를 기반으로 사전 정의된 대응 및 해결 작업을 제공합니다. Security Hub 고객이 일반적인 보안 탐지 결과를 해결하고 AWS에서 보안 태세를 개선하는 데 도움이 됩니다. 자세한 내용은 [이 문서](https://aws.amazon.com/solutions/implementations/automated-security-response-on-aws/)를 참조하세요.

![SH ASR](../../images/SH-ASR.png)
*그림 10: 자동화된 보안 대응 다이어그램*

### 타사 통합

Security Hub 내에서 타사 지원 파트너와의 통합을 사용할 수 있습니다. 통합을 사용하여 대응을 자동화하는 한 가지 예는 탐지 결과를 티켓팅 시스템으로 전달하는 것입니다. 예를 들어 ServiceNow ITSM과 Security Hub의 통합을 통해 Security Hub의 보안 탐지 결과를 ServiceNow ITSM 내에서 볼 수 있습니다. 또한 Security Hub로부터 탐지 결과를 받을 때 자동으로 인시던트 또는 문제를 생성하도록 ServiceNow를 구성할 수 있습니다. 이러한 인시던트 및 문제에 대한 업데이트는 Security Hub의 탐지 결과 업데이트로 이어집니다.

![SH and SNOW integration](../../images/SH-Snow-Integration.png)
*그림 11: Security Hub 및 ServiceNow 통합 다이어그램*

## 비용 고려사항

Security Hub는 세 가지 차원으로 가격이 책정됩니다: 보안 검사 수량, 탐지 결과 수집 이벤트 수량, 월별 처리된 규칙 평가 수량입니다. AWS Organizations 지원을 통해 Security Hub를 사용하면 여러 AWS 계정을 연결하고 해당 계정의 탐지 결과를 통합하여 전체 조직의 보안 검사, 탐지 결과 수집 이벤트 및 자동화 규칙 평가에 대한 계층화된 가격을 누릴 수 있습니다.

Security Hub를 처음 시작하거나 새 계정에서 활성화하는 경우 AWS Security Hub에는 [30일 무료 평가판](https://aws.amazon.com/security-hub/pricing/?nc=sn&loc=3)이 있습니다. 평가판에는 전체 Security Hub 기능 세트 및 보안 모범사례 검사가 포함됩니다. Security Hub가 활성화된 각 리전의 모든 AWS 계정은 무료 평가판을 받습니다. 무료 평가판 기간 동안 동일한 계정과 리전에서 Security Hub를 계속 사용하는 경우 월별 청구서 예상치를 받게 됩니다.

* AWS Foundational Security Best Practices 및 CIS 표준은 기본적으로 켜져 있습니다.
* 표준을 활성화하면 해당 표준의 "모든" 제어가 기본적으로 활성화됩니다. 그런 다음 활성화된 표준 내에서 특정 제어를 비활성화 및 활성화하거나 전체 표준을 비활성화할 수 있습니다.
* 절충점이 있습니다: 제어를 비활성화하여 비용을 낮추면 가시성 부족으로 인해 잠재적으로 더 높은 위험이 발생할 수 있습니다.
* 글로벌 기록을 실행하는 리전을 제외한 모든 리전에서 글로벌 리소스를 다루는 제어를 끕니다. 예:
  * 중앙 집중식 S3 버킷이 있는 계정 및 리전을 제외한 모든 계정 및 리전에서 CloudTrail 로깅과 관련된 CIS 2.3 및 2.6 제어를 비활성화할 수 있습니다.
  * 글로벌 기록을 실행하는 리전을 제외한 모든 리전에서 글로벌 리소스를 다루는 CIS 1.2-1.14, 1.16, 1.22 및 2.5 제어를 비활성화합니다.
* Security Hub로 공급되는 통합에서 유용하지 않은 탐지 결과를 필터링합니다. 이렇게 하면 탐지 결과 수집 이벤트 비용이 줄어듭니다.
* 환경과 잠재적으로 관련이 없는 보안 검사를 끕니다. 예를 들어 엄격한 CI/CD 프로세스 및 일일 롤아웃과 같은 보완 제어가 있기 때문에 VPC 내에서 Lambda를 사용할 계획이 없는 경우입니다. 그러나 이 예에서는 Lambda와 같은 서비스 사용을 제어하기 위한 메커니즘(예: 허용/거부 목록 SCP)이 있어야 한다는 점도 고려해야 합니다.

### AWS Config

AWS Config를 사용하면 기록된 구성 항목 수, 활성 AWS Config 규칙 평가 수 및 계정의 적합성 팩 평가 수에 따라 요금이 부과됩니다. 구성 항목은 AWS 계정의 리소스 구성 상태 레코드입니다. AWS Config 규칙 평가는 AWS 계정의 AWS Config 규칙에 의한 리소스의 규정 준수 상태 평가입니다. 적합성 팩 평가는 적합성 팩 내의 AWS Config 규칙에 의한 리소스 평가입니다.

* 필요한 리전에서만 글로벌 리소스를 기록합니다. 이렇게 하면 기록된 구성 항목 수가 줄어듭니다.
* Security Hub 외부에서 Config를 사용하지 않는 경우 규정 준수 기록 타임라인을 끕니다 - 각 개별 리소스 규정 준수 상태의 기록을 추적합니다. 이는 Config에서 모든 리소스 유형 기록을 활성화한 경우 기본적으로 켜져 있습니다.
* Security Hub 제어에서 지원하지 않는 리소스 기록을 비활성화합니다 - Config를 CMDB로 사용하는 경우 위험이 발생할 수 있습니다. 또한 Security Hub 제어 확장을 고려하여 환경에서 사용하는 리소스 유형의 기록을 비활성화하는 것은 권장되지 않습니다.
* [Security Hub를 위한 AWS Config 비용 최적화](https://aws.amazon.com/blogs/security/optimize-aws-config-for-aws-security-hub-to-effectively-manage-your-cloud-security-posture/)에 언급된 다른 팁이 있습니다.

여기서 언급할 중요한 사항은 AWS Config가 Security Hub의 30일 평가판 버전에 포함되지 않는다는 것입니다. 따라서 Security Hub에서 보안 표준을 활성화하려는 경우 사용된 모든 AWS Config 규칙에 대해 요금이 부과됩니다.

## 리소스

### 워크샵

* [Activation Days](https://awsactivationdays.splashthat.com/)
* [Threat Detection and Response workshop](https://catalog.workshops.aws/security/en-US)
* [Amazon Detective workshop](https://catalog.workshops.aws/detective)
* [EKS security workshop](https://catalog.workshops.aws/containersecurity)
* [Amazon Macie workshop](https://catalog.workshops.aws/data-discovery)

### 비디오

* [Customize and contextualize security with AWS Security Hub](https://www.youtube.com/watch?v=nghb507nVtM&list=PLB3flZ7qA4xu__uOEfpc-coXm04swNWva&index=2&pp=iAQB)
* [AWS Security Hub - Bidirectional integration with ServiceNow ITSM](https://www.youtube.com/watch?v=OYTi0sjEggE)
* [Re:inforce Security Hub Automation Rules](https://www.youtube.com/watch?v=t10Mgi8ZgVw)
* [AWS Security Hub integration with AWS Control Tower](https://www.youtube.com/watch?v=Ev3giJRpHWw&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=2&pp=iAQB)
* [AWS Security Hub automation rules](https://www.youtube.com/watch?v=XaMfO_MERH8&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=23&pp=iAQB)
* [Using Security Hub finding history feature](https://www.youtube.com/watch?v=mz_yRIDxX5M&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=28&pp=iAQB)
* [Subscribing to Security Hub announcements](https://www.youtube.com/watch?v=iolGhikAigw&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=70&t=2s&pp=iAQB)
* [Visualize Security Hub findings using Amazon Quicksight](https://www.youtube.com/watch?v=qfBptS8qogE&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=86&t=2s&pp=iAQB)
* [Cross-region finding aggregation](https://www.youtube.com/watch?v=KcRmxehmRvk&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=93&pp=iAQB)
* [Bidirectional integration with Atlassian Jira Service Management](https://www.youtube.com/watch?v=uEKwu0M8S3M&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=97&t=1s&pp=iAQB)

### 블로그

* [Optimize AWS Config for AWS Security Hub to effectively manage your cloud security posture](https://aws.amazon.com/blogs/security/optimize-aws-config-for-aws-security-hub-to-effectively-manage-your-cloud-security-posture/)
* [Consolidating controls in Security Hub: The new controls view and consolidated findings](https://aws.amazon.com/blogs/security/consolidating-controls-in-security-hub-the-new-controls-view-and-consolidated-findings/)
* [AWS Security Hub launches a new capability for automating actions to update findings](https://aws.amazon.com/blogs/security/aws-security-hub-launches-a-new-capability-for-automating-actions-to-update-findings/)
* [Get details on security finding changes with the new Finding History feature in Security Hub](https://aws.amazon.com/blogs/security/get-details-on-security-finding-changes-with-the-new-finding-history-feature-in-security-hub/)
* [Three recurring Security Hub usage patterns and how to deploy them](https://aws.amazon.com/blogs/security/three-recurring-security-hub-usage-patterns-and-how-to-deploy-them/)
* [How to subscribe to the new Security Hub Announcements topic for Amazon SNS](https://aws.amazon.com/blogs/security/how-to-subscribe-to-the-new-security-hub-announcements-topic-for-amazon-sns/)
* [How to export AWS Security Hub findings to CSV format](https://aws.amazon.com/blogs/security/how-to-export-aws-security-hub-findings-to-csv-format/)
* [Automatically block suspicious DNS activity with Amazon GuardDuty and Route 53 Resolver DNS Firewall](https://aws.amazon.com/blogs/security/automatically-block-suspicious-dns-activity-with-amazon-guardduty-and-route-53-resolver-dns-firewall/)
* [How to build a multi-Region AWS Security Hub analytic pipeline and visualize Security Hub data](https://aws.amazon.com/blogs/security/how-to-build-a-multi-region-aws-security-hub-analytic-pipeline/)
* [How to enrich AWS Security Hub findings with account metadata](https://aws.amazon.com/blogs/security/how-to-enrich-aws-security-hub-findings-with-account-metadata/)
