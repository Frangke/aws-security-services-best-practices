# Amazon Macie

## 소개

Amazon Macie 모범 사례 가이드에 오신 것을 환영합니다. 이 가이드의 목적은 Amazon S3 자산의 데이터를 지속적으로 모니터링하기 위해 Amazon Macie를 활용하는 규범적 지침을 제공하는 것입니다. GitHub를 통해 이 지침을 게시하면 서비스 개선 사항과 사용자 커뮤니티의 피드백을 포함하는 시기적절한 권장 사항을 신속하게 반복할 수 있습니다. 이 가이드는 단일 계정에서 처음으로 Macie를 배포하거나 기존 다중 계정 배포에서 Macie를 최적화하는 방법을 찾는 경우 모두에게 가치를 제공하도록 설계되었습니다.

## 이 가이드 사용 방법

이 가이드는 AWS 계정(및 리소스) 내에서 위협, 악의적 활동 및 취약점의 모니터링 및 해결을 담당하는 보안 실무자를 대상으로 합니다. 모범 사례는 더 쉬운 소비를 위해 세 가지 범주로 구성되어 있습니다. 각 범주에는 간략한 개요로 시작하여 지침 구현을 위한 세부 단계가 이어지는 해당 모범 사례 세트가 포함되어 있습니다. 주제는 특정 순서로 읽을 필요가 없습니다:

* [Amazon Macie란 무엇인가요?](#amazon-macie란-무엇인가요)
* [Macie 활성화의 이점은 무엇인가요?](#macie-활성화의-이점은-무엇인가요)
* [시작하기](#시작하기)
    * [배포 고려 사항](#배포-고려-사항)
    * [리전 고려 사항](#리전-고려-사항)
* [구현](#구현)
    * [독립 실행형 계정 활성화](#독립-실행형-계정-활성화)
    * [다중 계정 조직 활성화](#다중-계정-조직-활성화)
    * [계정 범위](#계정-범위)
    * [검색 결과](#검색-결과)
    * [Security Hub에 게시](#security-hub에-게시)
    * [자동화된 민감한 데이터 검색 vs 민감한 데이터 검색 작업](#자동화된-민감한-데이터-검색-vs-민감한-데이터-검색-작업)
    * [리소스 범위](#리소스-범위)
* [Macie 결과 운영화](#macie-결과-운영화)
    * [Macie 결과 조치](#macie-결과-조치)
    * [민감한 데이터 탐지에 영향 주기](#민감한-데이터-탐지에-영향-주기)
    * [억제 규칙](#억제-규칙)
* [비용 고려 사항](#비용-고려-사항)
* [리소스](#리소스)

## Amazon Macie란 무엇인가요?

Amazon Macie는 기계 학습과 패턴 매칭을 사용하여 민감한 데이터를 검색하고, 데이터 보안 위험에 대한 가시성을 제공하며, 이러한 위험에 대한 자동화된 보호를 가능하게 하는 데이터 보안 서비스입니다.

조직의 Amazon Simple Storage Service(Amazon S3) 데이터 자산의 보안 태세를 관리하는 데 도움이 되도록 Macie는 S3 버킷의 인벤토리를 제공하고 보안 및 액세스 제어를 위해 버킷을 자동으로 평가하고 모니터링합니다. Macie가 데이터의 보안 또는 개인 정보 보호와 관련된 잠재적 문제(예: 공개적으로 액세스 가능하게 된 버킷)를 감지하면 Macie는 필요에 따라 검토하고 해결할 수 있도록 결과를 생성합니다.

Macie는 또한 민감한 데이터의 검색 및 보고를 자동화하여 조직이 Amazon S3에 저장하는 데이터를 더 잘 이해할 수 있도록 합니다. 민감한 데이터를 탐지하기 위해 Macie가 제공하는 기본 제공 기준 및 기술, 사용자가 정의하는 사용자 지정 기준 또는 이 둘의 조합을 사용할 수 있습니다. Macie가 S3 객체에서 민감한 데이터를 탐지하면 Macie는 발견한 민감한 데이터를 알리기 위해 결과를 생성합니다.

## Macie 활성화의 이점은 무엇인가요?

Macie는 공개적으로 액세스 가능한 버킷, AWS Organization 외부에서 공유되는 버킷 또는 민감한 데이터가 포함된 버킷의 수와 같은 통계를 자동으로 제공하는 AWS 관리형 서비스입니다. 이러한 통계는 Amazon S3 데이터의 보안 태세와 데이터 자산에서 민감한 데이터가 있을 수 있는 위치에 대한 통찰력을 제공합니다. 통계와 데이터는 특정 S3 버킷 및 객체에 대한 심층 조사를 수행하기 위한 결정을 안내할 수 있습니다. Macie는 관리형 서비스이므로 이러한 통찰력을 제공하는 데 필요한 인프라를 설정하고 관리하는 대신 Amazon Macie 콘솔 또는 Amazon Macie API를 사용하여 결과, 통계 및 기타 데이터를 검토하고 분석하는 데 시간을 할애할 수 있습니다. 또한 Macie와 Amazon EventBridge 및 AWS Security Hub의 통합을 활용하여 다른 서비스, 애플리케이션 및 시스템을 사용하여 결과를 모니터링, 처리 및 해결할 수 있습니다.

Macie는 자동화 및 AWS Organizations 통합을 통해 AWS Organization 내의 모든 계정에서 모든 S3 버킷 및 객체에 대한 민감한 데이터 검색 및 버킷 태세 평가를 제공하여 이를 쉽게 수행할 수 있습니다. 전체 S3 데이터 자산에 대한 중앙 집중식 가시성을 제공합니다.

Macie의 가장 일반적인 사용 사례는 다음과 같습니다:

* Macie 자동화된 데이터 검색 및 AWS 서비스(예: AWS Security Hub 및 Amazon EventBridge)와의 기본 통합을 통해 민감한 데이터를 검색하고 조치하는 것이 더욱 비용 효율적이고 시간 효율적입니다.
* 회사 A가 회사 B에 인수되고 있습니다. 합병의 일환으로 회사 A는 규정 준수 의무를 준수하고 있는지 확인하기 위해 회사 B의 데이터에서 민감한 데이터의 존재를 이해해야 합니다.
* 회사 A는 데이터 중심 회사이며 비즈니스 요구 사항을 위해 엄청난 양의 데이터를 보관합니다. 회사 A는 또한 민감한 데이터 자산과 이 데이터의 액세스 패턴을 이해함으로써 민감한 데이터가 신중하게 보호되도록 하고자 합니다.
* 귀사에 보안 문제가 발생했으며 액세스된 데이터 유형을 이해해야 합니다.

## 시작하기

Amazon Macie를 시작하기 전에 Macie를 관리할 수 있는 권한이 있는지 확인하고 AWS Organization에서 Macie 위임된 관리자 계정으로 가장 적합한 계정을 고려해야 합니다. 권한을 시작하려면 Macie를 관리하는 데 사용하는 역할에 최소한 AWS 관리형 정책 이름 "AmazonMacieFullAccess"가 있는지 확인하십시오.

### 배포 고려 사항

AWS Organization 전체에 Macie를 배포하려면 콘솔, CLI 또는 API를 통해 수행되는지 여부에 관계없이 AWS 관리 계정과 보안 도구 계정에서 활성화해야 합니다. 보안 도구 계정의 개념에 익숙하지 않은 경우 보안 참조 아키텍처의 [권장 계정 구조](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/organizations.html)를 숙지하는 것이 좋습니다. 요약하자면, 이것은 Amazon Inspector, Amazon GuardDuty, AWS Security Hub 및 Amazon Detective와 같은 기본 AWS 보안 서비스의 위임된 관리자 계정으로 사용되는 AWS Organization의 전용 계정입니다.

### 리전 고려 사항

Amazon Macie는 리전 서비스입니다. 즉, Macie를 사용하려면 민감한 데이터를 검색하려는 모든 리전에서 활성화해야 합니다. AWS API를 사용하거나 콘솔에서 리전 간 전환을 통해 모든 계정 및 리전에서 Macie를 활성화할 수 있습니다.

## 구현

이 섹션에서는 독립 실행형 계정과 다중 계정 조직에서 Macie를 활성화하기 위한 최소 요구 사항을 다룹니다.

### 독립 실행형 계정 활성화

![Macie 시작 페이지](../../images/MC-Getting-Started.png)
*그림 1: Macie 시작*

첫 번째 단계는 Macie 콘솔로 이동하는 것입니다. Macie 콘솔에 있으면 시작 버튼이 있는 랜딩 페이지가 표시됩니다. "시작하기"를 클릭합니다.

![Macie 활성화 페이지](../../images/MC-Enable.png)
*그림 2: Macie 활성화*

"시작하기"를 클릭하면 Macie 활성화 페이지로 이동합니다. 서비스 역할 권한을 검토하여 Macie가 기능을 제공하는 데 필요한 권한을 이해한 다음 "Macie 활성화"라는 노란색 상자를 선택하여 Macie를 활성화합니다.

이 두 단계가 완료되면 이 계정에서 Macie가 활성화됩니다. Macie의 모든 관련 기능을 활성화했는지 확인하려면 아래 섹션을 참조하는 것이 중요합니다.

### 다중 계정 조직 활성화

위에서 언급한 대로 AWS Organization에서 처음으로 Macie를 구현할 때 Macie를 사용하려는 각 리전의 조직 관리 계정에서 위임된 관리자를 설정합니다. 완료되면 이 섹션에서 다룰 Amazon Macie를 완전히 구성하기 위한 다른 단계가 있습니다.

![Macie 시작 페이지](../../images/MC-Getting-Started.png)
*그림 3: Macie 시작*

첫 번째 단계는 Organizations 관리 계정의 Macie 콘솔로 이동하는 것입니다. Macie 콘솔에 있으면 시작 버튼이 있는 랜딩 페이지가 표시됩니다. "시작하기"를 클릭합니다.

![Macie 위임된 관리자 활성화](../../images/MC-Delegated-Enable.png)
*그림 4: Macie 위임된 관리자 활성화*

"시작하기"를 클릭하면 Macie 활성화 페이지로 이동합니다. 다음으로 Macie 위임된 관리자 계정으로 지정하려는 계정의 계정 ID를 입력해야 합니다. 계정 ID를 입력한 후 "위임"을 선택합니다. 이 시점에서 위임된 관리자로 전환하여 AWS Organization 전체에서 Amazon Macie 구성을 완료합니다.

### 계정 범위

조직 관리 계정에서 위임된 관리자를 설정하면 위임된 관리자 계정에서 Macie가 활성화되지만 조직에 이미 존재하는 계정 전체에서 범위가 누락됩니다. 따라서 다음으로 계정 설정으로 이동하거나 API를 사용하여 모든 구성원 계정에서 Macie를 활성화해야 합니다. 또한 "자동 활성화" 토글 스위치를 켜기로 확인하고 싶을 것입니다. 이렇게 하면 Organization의 모든 새 계정이 자동으로 Macie를 활성화하여 가시성 부족을 방지하고 조직의 계정에 대해 개별적으로 Macie를 활성화하는 수동 작업을 절약할 수 있습니다. 계정 상태가 활성화됨으로 표시되고 "자동 활성화" 토글 스위치가 아래 그림과 같이 켜짐을 나타내는지 확인하십시오.

![Macie 계정 페이지](../../images/MC-Account-Page.png)
*그림 5: Macie 계정 페이지*

### 검색 결과

Macie를 활성화한 후 처음 30일 이내에 검색 결과를 위한 S3 버킷을 설정하는 것이 중요합니다. Macie는 민감한 데이터 검색 결과를 90일 동안만 저장하기 때문입니다. 결과에 액세스하고 장기 저장 및 보존을 활성화하려면 결과를 S3 버킷에 저장하도록 Macie를 구성해야 합니다. 이렇게 하면 앞으로 모든 검색 결과의 기록을 유지할 수 있습니다. 컨텍스트를 위해 검색 결과는 민감한 데이터가 식별되지 않은 경우에도 Macie가 확인한 모든 파일에 대한 항목을 생성합니다. 이 데이터는 특정 파일이 스캔되었는지 확인하고, Macie가 파일을 스캔할 수 없는 위치를 식별하고, 민감한 데이터가 발견된 파일을 식별하는 데 도움이 될 수 있습니다. 이 리포지토리를 구성하는 방법을 알아보려면 [민감한 데이터 검색 결과 저장 및 보존](https://docs.aws.amazon.com/macie/latest/user/discovery-results-repository-s3.html)을 참조하십시오.

### Security Hub에 게시

기본적으로 Macie는 정책 및 민감한 데이터 결과를 이벤트로 Amazon EventBridge에 자동으로 게시하고 정책 결과를 Security Hub에 게시합니다. 민감한 데이터 결과도 Security Hub에 게시하여 티켓팅 시스템으로 결과를 보내거나 Lambda를 사용하여 Security Hub의 결과를 기반으로 리소스를 해결하는 등 다른 보안 서비스 결과에 대해 생성했을 수 있는 워크플로 또는 자동화를 활용하는 것이 좋습니다.

![Macie 게시](../../images/MC-Publishing.png)
*그림 6: Macie 결과 게시 설정*

### 자동화된 민감한 데이터 검색 vs 민감한 데이터 검색 작업

조직 전체에서 사용되는 대규모 S3 버킷(고객 서비스, 백엔드 소프트웨어, 타사, 정적 웹 콘텐츠 호스팅 등)이 있는 경우 ADD를 활성화하는 것이 좋습니다. ADD는 S3 자산을 지능적으로 샘플링하고 크롤링합니다. 유사한 객체를 찾고 해당 유사한 객체의 일부를 조사하여 매일 매우 가벼운 수준의 S3 버킷을 샘플링합니다. 이렇게 하면 TB/PB 단위의 대규모 데이터 세트가 있더라도 ADD를 통해 데이터의 높은 신뢰도 민감도 수준을 얻을 수 있습니다.

![Macie 요약 페이지](../../images/MC-Summary.png)
*그림 7: Macie 요약 페이지*

특정 버킷에 대한 심층 분석을 수행해야 하는 경우 찾고 있는 스캔 기능에 매우 구체적일 수 있는 민감한 데이터 검색 작업을 실행할 수도 있습니다. 예를 들어 특정 관리형 데이터 식별자, 사용자 지정 데이터 식별자를 찾거나 특정 파일 유형만 찾거나 특정 태그를 기반으로 데이터를 포함/제외할 수 있습니다. 대다수의 고객은 자동화된 검색에서 결과를 얻은 다음 특정 데이터 세트를 자세히 살펴보기 때문에 개별 작업을 실행할 필요가 없습니다.

민감한 데이터 검색 작업을 실행하는 몇 가지 사용 사례는 불확실성이 있을 수 없는 데이터 세트가 있는 고객이므로 이 데이터 세트 전체를 스캔하여 주어진 데이터 세트에서 민감한 데이터의 모든 인스턴스가 있는 위치를 알 수 있도록 결정합니다. 또 다른 사용 사례는 데이터 수집을 기반으로 스캔을 실행하는 것입니다. 데이터 검색 작업을 통해 이 데이터가 수집되는 위치와 스캔되는 빈도를 대상으로 지정할 수 있습니다. 마지막으로 자동화된 검색을 통해 전체 S3 자산에서보다 더 작은 데이터 하위 집합에서 사용자 지정 데이터 식별자를 사용할 수 있습니다. 민감한 데이터 검색 작업을 통해 이 사용자 지정 데이터 식별자를 더 작은 데이터 세트에 집중할 수 있습니다.

민감한 데이터 검색 작업을 구성하는 방법에 대한 자세한 내용은 [Amazon Macie 문서](https://docs.aws.amazon.com/macie/latest/user/discovery-jobs.html)를 참조하십시오.

### 리소스 범위

Macie가 S3의 데이터를 스캔하려면 데이터가 [지원되는 파일 형식](https://docs.aws.amazon.com/macie/latest/user/discovery-supported-storage.html)이어야 하고 [Macie를 허용하는 권한](https://docs.aws.amazon.com/macie/latest/user/monitoring-restrictive-s3-buckets.html)이 있는 버킷에 있어야 합니다. 대부분의 버킷에는 버킷 정책에 명시적 허용이 있지만 버킷 정책에 명시적 거부가 있는 버킷의 경우 버킷 정책에서 Macie 서비스 연결 역할을 허용하도록 추가해야 합니다. 리소스 범위 페이지에서 Macie는 버킷 문제를 나열하므로 Macie가 스캔할 수 없는 버킷을 더 쉽게 확인하고 해결할 수 있습니다.

![Macie 리소스 범위](../../images/MC-Coverage.png)
*그림 8: Macie S3 범위 통찰력*

## Macie 결과 운영화

### Macie 결과 조치

보안 서비스의 결과를 운영화하기 위해 권장하는 첫 번째 단계는 결과가 제공하는 세부 정보를 이해한 다음 조직의 도구 기능과 소유권 모델을 고려하여 결과에 어떻게 대응할지 이해하는 것입니다. 예를 들어, 새로운 민감한 데이터 결과가 생성될 때 티켓을 생성하기 위해 통합할 수 있는 티켓팅 시스템이 있습니까? 계정 소유자가 자동으로 티켓을 받고 해결을 담당합니까? 중앙 보안 팀이 분류 및 경고를 담당합니까? 이것들은 답해야 할 몇 가지 질문일 뿐입니다.

![Macie 결과](../../images/MC-Findings.png)
*그림 9: Macie 결과*

Macie 결과는 민감한 데이터 결과의 버킷 및 객체 위치와 같은 세부 정보를 제공하며, 민감한 데이터가 무엇이었는지와 주어진 결과 내의 위치에 대한 정보도 제공하여 해결을 더 쉽게 만듭니다. 티켓팅 시스템 또는 이메일 알림을 통한 자동화를 통해 이 경고가 수행되는지 여부에 관계없이 계정 소유자와 애플리케이션 팀이 Macie에서 발견한 민감한 데이터를 해결하고 정리하는 책임을 지도록 하는 것부터 시작하는 것이 좋습니다. 민감한 데이터 결과와 환경에 대한 친숙함을 확립한 후에는 시간을 절약하고 수동 인간 작업으로 인한 잠재적 오류를 줄이기 위해 가능한 곳에서 자동화하는 것이 좋습니다. 이것은 Lambda를 사용하여 민감한 데이터가 발견된 S3 버킷에 대해 모두 거부 S3 버킷 정책을 생성하는 것과 같은 것일 수 있습니다. 이러한 유형의 솔루션은 조직의 위험 선호도와 환경의 태그 지정 및 보상 제어와 같은 다른 요소에 크게 의존합니다. Macie 결과를 기반으로 자동화를 생성하는 방법을 자세히 다루는 여러 블로그를 리소스 섹션에 연결했습니다.

### 민감한 데이터 탐지에 영향 주기

고객들은 종종 Amazon Macie가 찾고 있는 것을 어떻게 조정할 수 있는지 묻습니다. S3 데이터 자산의 데이터를 찾기 위해 Macie를 더 효과적으로 조정하려면 허용 목록, 사용자 지정 데이터 식별자를 생성하고 찾고자 하는 관리형 데이터 식별자를 지정할 수 있는 옵션이 있습니다.

Macie에는 자격 증명, 금융 정보, 개인 건강 정보 및 개인 식별 정보와 같은 민감한 데이터 유형과 관련된 관리형 데이터 식별자가 있습니다. 사용자 지정 데이터 식별자를 사용하면 조직의 특정 시나리오, 지적 재산 또는 독점 데이터(예: 직원 ID, 고객 계정 번호 또는 내부 데이터 분류)를 반영하는 자체 탐지 기준을 생성할 수 있습니다.

허용 목록은 S3 객체에서 민감한 데이터를 검사할 때 Macie가 무시할 특정 텍스트 및 텍스트 패턴을 정의합니다.

아래 그림과 같이 자동화된 검색 설정에서 이러한 각 메커니즘을 조정할 수 있습니다. Macie는 사용되는 데이터 식별자의 양에 따라 요금이 부과되지 않으므로 모든 관리형 데이터 식별자를 사용한 다음 허용 목록을 사용하여 거짓 긍정을 조정하는 것이 좋습니다. 그런 다음 조직에 특정한 데이터를 찾아야 하는 경우 사용자 지정 데이터 식별자를 사용하여 Macie의 탐지 기능을 확장하십시오.

![Macie 데이터 식별자](../../images/MC-Identifiers.png)
*그림 10: Macie 관리형 데이터 식별자*

### 억제 규칙

결과 분석을 간소화하기 위해 억제 규칙을 생성하고 사용할 수 있습니다. 억제 규칙은 Amazon Macie가 자동으로 결과를 보관하려는 경우를 정의하는 속성 기반 필터 기준 세트입니다. 억제 규칙은 결과 클래스를 검토했고 다시 알림을 받고 싶지 않은 상황에서 유용합니다. 억제 규칙을 구성하는 방법에 대해 자세히 알아보려면 [문서를 참조](https://docs.aws.amazon.com/macie/latest/user/findings-suppression.html)하십시오.

## 비용 고려 사항

Amazon Macie를 가장 비용 효율적으로 사용하려면 Macie의 자동화된 데이터 검색 기능을 사용하는 것이 좋습니다. 이 기능은 샘플링 기술을 사용하여 버킷에서 대표적인 S3 객체를 효과적으로 식별하고 선택하여 민감한 데이터 검색을 비용 효율적으로 만듭니다. 예를 들어 Macie 자동화된 데이터 검색은 한 달에 몇 백 달러만으로 100TB 데이터 자산의 데이터를 샘플링할 수 있습니다.

민감한 데이터 검색 작업을 구성하는 경우 작업을 생성하기 전에 Macie는 스캔될 데이터 양을 기반으로 작업 실행의 잠재적 비용 추정치를 보여줍니다. 이를 통해 작업 비용을 이해하고 비용 영향을 이해하는 데 도움이 될 수 있습니다. Macie 콘솔의 사용량 페이지를 통해 예방적 제어 모니터링, 민감한 데이터 검색 작업 및 자동화된 민감한 데이터 검색으로 세분화된 전체 환경과 관련된 월별 Macie 비용을 이해할 수 있으며, 이는 객체 분석 및 객체 모니터링으로 더 세분화됩니다.

Amazon Macie를 처음 시작하거나 새 계정에서 활성화하는 경우 Amazon Macie에는 계정당 최대 150GB의 자동화된 민감한 데이터 검색과 버킷 인벤토리를 포함하는 30일 무료 평가판이 있습니다. 민감한 데이터 검색 작업은 [30일 무료 평가판](https://aws.amazon.com/macie/pricing/)에 포함되지 않습니다.

자세한 내용과 가격 예시는 [Macie 가격 페이지](https://aws.amazon.com/macie/pricing/)에서 확인할 수 있습니다.

## 리소스

### 워크샵

* [Activation Days](https://awsactivationdays.splashthat.com/)
* [Amazon Macie 워크샵](https://catalog.us-east-1.prod.workshops.aws/workshops/9982e0dc-0ccf-4116-ad12-c053b0ab31c6/en-US)
* [Amazon Detective 워크샵](https://catalog.workshops.aws/detective)

### 비디오

* [추가 유형의 민감한 데이터 검색을 위한 추가 지원](https://www.youtube.com/watch?v=Cu0Cy_kk_Zc&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=34&pp=iAQB)
* [허용 목록으로 민감한 데이터 결과 미세 조정](https://www.youtube.com/watch?v=JmQ_Hybh2KI&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=49&pp=iAQB)
* [자동화된 데이터 검색 개요](https://www.youtube.com/watch?v=PVnFYotwqyo&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=52&t=3s&pp=iAQB)
* [원클릭 임시 검색](https://www.youtube.com/watch?v=-rY4G8ER8Gg&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=62&t=3s&pp=iAQB)
* [Amazon Macie가 키워드를 사용하여 민감한 데이터를 검색하는 방법](https://www.youtube.com/watch?v=GBaIAwLYN-o&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=69&pp=iAQB)

### 블로그

* [Amazon Macie를 사용하여 민감한 데이터 검색 비용을 줄이는 방법](https://aws.amazon.com/blogs/security/how-to-use-amazon-macie-to-reduce-the-cost-of-discovering-sensitive-data/)
* [Athena 및 Quicksight로 Macie 민감한 데이터 검색 결과를 쿼리하고 시각화하는 방법](https://aws.amazon.com/blogs/security/how-to-query-and-visualize-macie-sensitive-data-discovery-results-with-athena-and-quicksight/)
* [Amazon Macie를 사용하여 S3 버킷의 민감한 데이터를 미리 보는 방법](https://aws.amazon.com/blogs/security/how-to-use-amazon-macie-to-preview-sensitive-data-in-s3-buckets/)
* [IAM Access Analyzer 결과를 Amazon Macie와 연관시키기](https://aws.amazon.com/blogs/security/correlate-iam-access-analyzer-findings-with-amazon-macie/)
* [Amazon Macie, Amazon EventBridge 및 Slack을 사용한 민감한 데이터 검색에서 알림 워크플로 생성](https://aws.amazon.com/blogs/security/creating-a-notification-workflow-from-sensitive-data-discover-with-amazon-macie-amazon-eventbridge-aws-lambda-and-slack/)
* [Amazon Macie 결과를 해결하기 위한 자동화된 ChatOps 솔루션 배포](https://aws.amazon.com/blogs/security/deploy-an-automated-chatops-solution-for-remediating-amazon-macie-findings/)
