# Amazon Detective

## 소개

Amazon Detective 모범 사례 가이드에 오신 것을 환영합니다. 이 가이드의 목적은 AWS 리소스와 관련된 보안 문제를 조사하기 위해 Amazon Detective를 활용하는 규범적 지침을 제공하는 것입니다. GitHub를 통해 이 지침을 게시하면 서비스 개선 사항과 사용자 커뮤니티의 피드백을 포함하는 시기적절한 권장 사항을 신속하게 반복할 수 있습니다. 이 가이드는 단일 계정에서 처음으로 Detective를 배포하거나 기존 다중 계정 배포에서 Detective를 최적화하는 방법을 찾는 경우 모두에게 가치를 제공하도록 설계되었습니다.

## 이 가이드 사용 방법

이 가이드는 AWS 계정(및 리소스) 내에서 위협 및 악의적 활동의 모니터링 및 해결을 담당하는 보안 실무자를 대상으로 합니다. 모범 사례는 더 쉬운 소비를 위해 범주로 구성되어 있습니다. 각 범주에는 간략한 개요로 시작하여 지침 구현을 위한 세부 단계가 이어지는 해당 모범 사례 세트가 포함되어 있습니다. 주제는 특정 순서로 읽을 필요가 없습니다:

* [Amazon Detective란 무엇인가요?](#amazon-detective란-무엇인가요)
* [Amazon Detective는 누구를 위한 것인가요?](#amazon-detective는-누구를-위한-것인가요)
* [Amazon Detective 활성화의 이점은 무엇인가요?](#amazon-detective-활성화의-이점은-무엇인가요)
* [시작하기](#시작하기)
    * [GuardDuty 활성화](#guardduty-활성화)
    * [Detective를 활성화할 위치](#detective를-활성화할-위치)
    * [리전 고려 사항](#리전-고려-사항)
* [구현](#구현)
    * [활성화](#활성화)
    * [Organization의 다른 AWS 계정 등록](#organization의-다른-aws-계정-등록)
    * [선택적 데이터 피드 활성화](#선택적-데이터-피드-활성화)
* [운영화](#운영화)
    * [액세스 제공](#액세스-제공)
    * [결과 조사](#결과-조사)
      * [GuardDuty에서 조사 시작](#guardduty에서-조사-시작)
      * [Finding Group에서 조사 시작](#finding-group에서-조사-시작)
      * [Detective에서 위협 헌팅](#detective에서-위협-헌팅)
      * [Detective 임베디드 URL 작성](#detective-임베디드-url-작성)
* [비용 고려 사항](#비용-고려-사항)
* [리소스](#리소스)

## Amazon Detective란 무엇인가요?

Amazon Detective는 보안 결과 또는 의심스러운 활동의 근본 원인을 분석, 조사 및 신속하게 식별할 수 있게 해줍니다. Detective는 AWS 리소스에서 로그 데이터를 자동으로 수집하고 기계 학습, 통계 분석 및 그래프 이론을 사용하여 더 빠르고 효율적인 보안 조사를 시각화하고 수행할 수 있도록 지원합니다.

## Amazon Detective는 누구를 위한 것인가요?

Amazon Detective는 AWS에서 보안 문제를 조사하는 데 도움이 되는 관리형 보안 서비스를 활용하려는 고객에게 적합합니다. 많은 고객이 많은 도구를 보유한 대규모 조직일 수 있으며 분석가가 인시던트를 신속하게 분류할 수 있도록 하려고 합니다. 다른 고객은 더 적은 IT 보안 직원을 보유하고 있으며 팀이 AWS에서 보안 문제를 효율적으로 조사하는 데 도움이 되는 효과적인 도구가 필요할 수 있습니다. Detective는 보안 분석가, 보안 운영 엔지니어 및 인시던트 대응 팀을 위한 것입니다. 대규모 고객은 도구 확산을 경험했고 SIEM 도구를 통합하려는 경우 Amazon Detective를 사용하면 이점을 얻을 수 있습니다. 고객은 EKS 감사 로그 및 VPC 플로우 로그의 긴밀한 통합으로 인해 Amazon Elastic Kubernetes Service(EKS) 클러스터에 대한 원격 측정 및 가시성을 위해 Amazon Detective에서 특히 가치를 발견했습니다.

## Amazon Detective 활성화의 이점은 무엇인가요?

Detective는 인시던트 대응자와 보안 엔지니어가 특정 보안 이벤트와 관련된 리소스를 결정하는 데 도움을 줍니다. GuardDuty 결과, VPC Flow Logs, CloudTrail, EKS 감사 로그 및 Security Hub 결과의 데이터 피드를 사용하여 동작 그래프를 구축하여 고객이 리소스 연결, 새로 관찰된 지리적 위치와 같은 비정상적인 활동, 역할이 다른 역할에 의해 가정된 시기 이해 및 환경의 다른 많은 사용 사례를 이해하는 데 도움을 줍니다. 많은 고객이 로그 및 원격 측정 수집 상관 관계 도구를 구축하지 않았거나 지식이나 리소스가 없을 수 있으며 이것이 Amazon Detective를 완벽하게 적합하게 만듭니다.

이미 도구, 지식 및 인력을 갖춘 많은 고객은 허용 가능한 응답 시간 내에 보안 이벤트를 성공적으로 탐색하고 관련 로그를 발견할 수 있습니다. 일반적으로 이를 위해서는 VPC Flow 로그, CloudTrail 로그, EKS 감사 로그를 수집하고 로그 집계 도구로 전송하기 위해 타사 도구를 설치, 구성, 운영 및 모니터링해야 합니다. 이러한 데이터 피드에 대한 가시성이 내장되어 있는 Amazon Detective를 사용하여 별도로 집계 및 저장하는 것과 비교하여 비용 절충 및 절약된 시간을 고려하십시오. 대부분의 고객은 로그 집계 도구에 쿼리를 작성할 필요가 없고 대신 Detective가 제공하는 통찰력에 의존하기 때문에 Detective가 조사 시간을 단축하는 데 효과적이라는 것을 발견했습니다.

많은 고객이 Detective와 GuardDuty를 함께 사용하면 이벤트 후 AWS에서 기본적으로 조사할 수 있는 기능을 제공한다고 관찰합니다. GuardDuty는 위협에 대해 알려주지만 Detective는 해당 위협에 대한 스토리를 알려줍니다. 종종 고객으로부터 특정 서비스의 보안 결과 양이 팀을 압도하고 명확한 우선 순위 없이 남겨둔다는 이야기를 듣습니다. Detective는 팀이 실제 보안 이벤트에 집중할 수 있도록 합니다.

Detective는 런타임 조사를 지원하기 위해 Amazon EKS 워크로드에 적합합니다. Detective를 사용하면 EKS 워크로드 내부를 보고 포드 구성, 포드 이미지, 이미지 소스, EKS 클러스터 내의 VPC 플로우 및 클러스터 내 Kubernetes 사용자별 활동을 검사할 수 있습니다. 많은 AWS 계정에서 Amazon EKS를 실행하는 경우 Detective가 백엔드에서 데이터를 수집하고 중앙 집중식 보안 계정에 중앙 집중화하는 기능은 보안 이벤트 중 근본 원인까지의 시간을 줄이는 데 도움이 됩니다. 컨테이너 기반 워크로드에 Detective를 사용하는 방법에 대한 심층 연습은 [컨테이너 보안 워크샵](https://catalog.workshops.aws/containersecurity)을 자세히 살펴보십시오.

## 시작하기

### GuardDuty 활성화

Amazon Detective를 활성화하기 전에 Amazon GuardDuty를 활성화하는 것이 전제 조건입니다. GuardDuty를 활성화하고 대규모로 Organization의 AWS 계정을 등록하려면 [Amazon GuardDuty 문서](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)를 따르십시오.

환경에서 Amazon EKS를 실행하는 경우 [GuardDuty에서 EKS 감사 및 런타임 보호](https://docs.aws.amazon.com/guardduty/latest/ug/kubernetes-protection.html)를 활성화하는 것이 좋습니다. 아래 활성화 권장 사항에서 Detective에서 EKS 감사 로그를 소비하는 방법에 대해 논의할 것입니다.

GuardDuty CloudWatch 알림 빈도를 기본값인 6시간에서 15분으로 업데이트하는 것이 좋습니다. 6시간으로 두면 반복되는 결과에 대한 업데이트가 Detective에 반영되는 데 최대 6시간이 걸릴 수 있습니다. 이 빈도를 변경하든 변경하지 않든 새 결과는 항상 결과가 생성된 후 5분 이내에 Detective로 전송됩니다. 이 기간을 단축해도 추가 비용이 발생하지 않습니다. 알림 빈도 설정에 대한 정보는 Amazon GuardDuty 사용자 가이드의 [Amazon CloudWatch Events로 GuardDuty 결과 모니터링](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings_cloudwatch.html)을 참조하십시오.

### Detective를 활성화할 위치

[보안 참조 아키텍처](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/organizations.html)(SRA) 백서에 설명된 대로 나머지 AWS 보안 서비스(GuardDuty, Security Hub, Macie 등)와 동일한 위임된 관리자 AWS 계정에서 Amazon Detective를 활성화하는 것이 좋습니다. 아직 전용 보안 AWS 계정이 없는 경우 지금이 계정을 만들 완벽한 기회입니다. AWS Control Tower를 실행하는 경우 종종 "감사" 또는 "보안" 계정이라고 하는 계정이 이미 있습니다.

Detective는 현재 각 동작 그래프에서 최대 1200개의 AWS 계정을 지원합니다. 1200개 이상의 계정에 대해 Detective를 활성화하는 데 관심이 있는 경우 AWS 계정 팀에 문의하십시오. 다른 Detective 할당량은 [리전 및 할당량](https://docs.aws.amazon.com/detective/latest/adminguide/regions-limitations.html)에 대한 문서를 참조하십시오.

### 리전 고려 사항

Amazon Detective는 리전 서비스입니다. 즉, Amazon Detective를 사용하려면 Detective를 사용하여 인시던트 대응 기능을 원하는 모든 리전에서 활성화해야 합니다.

자주 묻는 질문은 "회사가 적극적으로 사용하지 않는 리전에서 보안 서비스를 사용해야 합니까?"입니다. Detective의 비용은 특정 계정 및 리전에서 수집된 데이터 양을 기반으로 합니다. AWS 계정 또는 리전에 활동이 없으면 Detective는 비용이 발생하지 않습니다. 예상 비용이 허용 가능한지 확인하려면 Detective의 사용량 섹션을 검토하는 것이 좋습니다. AWS Organization에서 사용할 수 있도록 계획하는 모든 AWS 계정 및 리전에서 Detective를 활성화하는 것이 좋습니다. 이렇게 하면 누군가가 리전에서 리소스를 생성하려고 시도하는 경우 후속 보안 문제에 필요한 경우 가시성을 확보할 수 있습니다.

AWS 콘솔, AWS CLI 또는 Amazon Detective Python 스크립트를 사용하여 AWS Organization의 계정 전체에서 Detective를 활성화할 수 있습니다. [여기에서 스크립트 및 해당 내용을 검토하십시오](https://docs.aws.amazon.com/detective/latest/adminguide/detective-github-scripts.html).

## 구현

### 활성화

원하는 보안 도구 계정에 Amazon Detective용 위임된 관리자를 생성하려면 [활성화 문서](https://docs.aws.amazon.com/detective/latest/adminguide/accounts-designate-admin.html)를 따르십시오. 그런 다음 AWS 콘솔을 사용하여 보안 도구 계정을 열고 Detective 서비스로 이동합니다.

Detective는 활성화하기가 매우 쉽습니다. 설치할 것이 없으며 다른 AWS 서비스와의 통합이 원활합니다. Detective 서비스에서 주황색 시작하기 버튼을 클릭하여 Detective를 활성화합니다.

![Detective 시작](../../images/DT-Getting-Started.png)

*그림 1: Detective 시작 페이지*

Detective가 생성할 서비스 연결 역할을 검토하여 필요한 권한을 이해한 다음 Amazon Detective 활성화를 클릭합니다.

![Detective 활성화](../../images/DT-Enable.png)

*그림 2: Detective 활성화*

### Organization의 다른 AWS 계정 등록

Detective를 활성화한 후 Organization의 다른 AWS 계정을 등록해야 합니다. Detective 콘솔에서 계정 관리로 이동합니다. 계정 관리 페이지에서 아래 그림과 같이 "모든 계정 활성화" 버튼을 선택하여 조직의 모든 계정을 등록할 수 있습니다.

![모든 계정 활성화](../../images/DT-Enable-All.png)

*그림 3: 모든 Detective 계정 활성화*

조직의 모든 계정에 대해 활성화한 후 아래 그림과 같이 "자동 활성화" 버튼을 켰는지 확인하십시오. 이를 통해 Detective 위임된 관리자는 팀이 어떤 작업도 수행할 필요 없이 조직에서 생성된 모든 새 계정에서 Detective를 자동으로 활성화할 수 있습니다.

![Detective 자동 활성화](../../images/DT-Auto-Enable.png)

*그림 4: Detective 자동 활성화 기능*

### 선택적 데이터 피드 활성화

일반 섹션으로 이동합니다.

Detective를 방금 활성화한 경우 이미 AWS 보안 서비스 및 EKS 감사 로그의 결과를 수집하고 있습니다. 이러한 통합이 릴리스되기 전에 Detective를 사용하고 있었다면 수동으로 활성화해야 합니다.

![소스 패키지 업데이트](../../images/DT-Sources.png)

*그림 5: Detective 소스 패키지*

### Amazon Security Lake 통합 활성화

선택적으로 Detective를 Amazon Security Lake와 통합하도록 구성하여 Detective 콘솔에서 직접 Security Lake에 저장된 로그를 쿼리하고 검색할 수 있습니다. 이 통합을 통해 보안 분석가는 Detective의 요약 및 시각화를 기반으로 보안 조사를 시작할 수 있습니다. 더 깊이 파고들어 로그를 검색해야 하는 경우 Detective는 조사 중인 시간 및 엔터티로 범위가 지정된 Amazon Athena를 사용하여 사전 구축된 쿼리를 제공합니다. 이 쿼리를 사용하여 잠재적 보안 문제를 나타내는 CloudTrail 또는 VPC 플로우 로그의 미리 보기를 가져오거나 모든 로그를 CSV 파일로 다운로드할 수 있습니다.

![SL 통합](../../images/DT-SL-Integration.png)
*그림 6: Security Lake 통합 활성화 페이지*

## 운영화

### 액세스 제공

Amazon Detective는 VPC Flow Logs, CloudTrail Logs, EKS 감사 로그 및 Security Hub 결과를 소비하고 GuardDuty의 이벤트와 리소스 ID를 동작 그래프로 연관시켜 작동합니다. 이 그래프는 조사를 수행할 때 Detective에서 따라갈 수 있는 것으로, 예를 들어 어떤 IP가 어떤 EC2 인스턴스와 관련이 있고 어떤 EC2 인스턴스 프로필과 관련이 있으며 인스턴스 프로필을 생성한 IAM 역할과 관련이 있는지 확인할 수 있습니다.

![Detective 플로우](../../images/DT-Flow.png)

*그림 7: Detective 워크플로 다이어그램*

결과적으로 IP 주소, 패키지 취약점, IAM 역할 및 페더레이션 사용자 형태로 Detective에 공급되는 정보가 있을 수 있습니다. 인시던트 조사자 IAM 역할에 최소 권한을 적용하고 싶을 것입니다.

대부분의 고객은 Security Hub, Inspector, Macie 및 GuardDuty 위임된 관리자를 포함한 많은 AWS 보안 서비스가 할당된 보안 도구 계정에 보안 및 인시던트 대응 팀에 대한 액세스를 제공하기로 선택합니다. AWS 계정에 대한 전체 관리 권한을 팀에 제공하지 않는 것이 모범 사례이며 보안 팀도 다르지 않습니다. Detective를 사용하는 인시던트 대응자에게 작업을 수행할 수 있는 최소 권한을 부여하는 것이 좋으며, 이를 시작하는 좋은 방법은 인시던트 대응자의 IAM 역할에 AWS IAM 관리형 정책 AmazonDetectiveInvestigatorAccess를 연결하는 것입니다. 설정 > 일반 탭에서 이 정책 및 기타 정책을 볼 수 있습니다.

![Detective IAM 정책](../../images/DT-IAM-Policies.png)

*그림 8: Detective IAM 정책*

### 결과 조사

Detective를 사용하여 조사를 시작하는 네 가지 주요 방법이 있습니다. 아래에서 이러한 방법을 자세히 다룰 것입니다.

1. GuardDuty에서 **Detective로 조사** 옵션 사용
2. Detective에서 **Finding Group** 검사
3. Detective의 검색 및 지리 그래프를 사용한 위협 헌팅
4. Splunk에서 임베디드 링크 사용

#### GuardDuty에서 조사 시작

대부분의 고객은 결과에서 **Detective로 조사** 옵션을 사용하여 Amazon GuardDuty를 통해 위협이 탐지된 후 조사를 시작합니다. GuardDuty 결과가 없는 경우에도 아래 방법 #3에서 설명하는 것처럼 Detective를 사용하여 활동을 조사하거나 위협을 헌팅할 수 있습니다. GuardDuty 결과를 열면 오른쪽에 세부 정보 창이 나타납니다. **Detective로 조사** 옵션을 클릭합니다.

![GuardDuty 결과](../../images/DT-GD-Finding.png)

*그림 9: GuardDuty 결과 세부 정보 페이지의 조사 링크*

Detective가 수집한 모든 관련 리소스가 있는 패널이 표시됩니다. GuardDuty 결과부터 시작하겠습니다.

![Detective 조사 옵션](../../images/DT-Investigate-Options.png)

*그림 10: GuardDuty 결과의 Detective 조사 옵션*

여기에서 VPC ID, 서브넷 ID 및 기타 관련 정보를 볼 수 있습니다.

![Detective 엔터티](../../images/DT-Entities.png)

*그림 11: GuardDuty 결과와 관련된 엔터티*

EC2 인스턴스를 탐색하여 관련 정보를 확인하겠습니다.

![Detective EC2 인스턴스](../../images/DT-Instance.png)

*그림 12: Detective의 인스턴스 프로필 페이지*

여기에서 EC2가 구축된 시기, 상주하는 AWS 계정, EC2 인스턴스를 생성한 IAM 역할 또는 사용자를 확인할 수 있습니다.

생성자 IAM 역할을 클릭하면 리소스 상호 작용 탭을 보고 이 EC2 인스턴스를 생성한 IAM ID, 사용자의 첫 번째 관찰 시기, 가장 최근 발생 및 범위 시간 동안 이 사용자가 이 역할을 가정한 다양한 시간을 식별할 수 있습니다.

![Detective 역할](../../images/DT-Role.png)

*그림 13: Detective의 역할 프로필 페이지*

이 연습을 위해 문제의 사용자의 역할 가정과 관련하여 이상한 발생을 발견했다고 가정하겠습니다. 이를 염두에 두고 이 시점에서 인시던트 대응 계획을 실행하고 EC2 인스턴스를 격리 및 스냅샷하고 IAM 역할의 과거 세션에 대한 자격 증명을 취소하고 인스턴스를 생성한 페더레이션 사용자를 추가로 조사할 수 있습니다. 조직에 현재 인시던트 대응 플레이북이 없는 경우 시작하는 데 도움이 되는 [공개적으로 사용 가능한 샘플](https://github.com/aws-samples/aws-incident-response-playbooks)이 있습니다.

#### Finding Group에서 조사 시작

Amazon Detective Finding Group을 사용하면 단일 보안 이벤트와 관련된 여러 활동을 검사할 수 있습니다. 위협 행위자가 AWS 환경을 손상시키려고 시도하는 경우 일반적으로 여러 보안 결과 및 비정상적인 동작으로 이어지는 일련의 작업을 수행합니다. 이러한 작업은 종종 시간과 엔터티에 걸쳐 분산됩니다. 보안 결과를 개별적으로 조사하면 중요성을 잘못 해석하고 근본 원인을 찾기 어려울 수 있습니다. Amazon Detective는 결과와 엔터티 간의 관계를 추론하고 함께 그룹화하는 그래프 분석 기술을 적용하여 이 문제를 해결합니다. Finding Group을 관련된 엔터티 및 결과를 조사하기 위한 시작점으로 취급하는 것이 좋습니다. GuardDuty가 위협 이벤트를 보여줄 수 있는 반면 Detective는 이벤트와 관련하여 환경에서 발생한 다른 일의 스토리를 알려줄 수 있습니다.

Finding Group으로 처음 이동하면(아래 예시 이미지) Finding Group의 생성형 AI 기반 요약이 표시됩니다. 이를 통해 데이터를 살펴보기 전에 Finding Group과 관련된 활동을 이해할 수 있습니다.

![Detective 생성형 AI](../../images/DT-Gen-AI.png)
*그림 14: Detective 생성형 AI Finding Group 요약*

Detective는 위협 이벤트 및 리소스를 MITRE ATT&CK 프레임워크와 연관시키려고 시도합니다. [MITRE](https://attack.mitre.org/matrices/enterprise/)는 악의적인 행위자가 시스템을 공격할 때 수행하는 전술, 기술 및 절차(TTP) 매트릭스를 게시합니다. 이러한 상관된 TTP는 Detective에 의해 Finding Group에 게시됩니다. 이것은 이벤트를 구성하는 이벤트, 리소스 및 보안 결과의 시각적 표현입니다.

![Detective Finding Group TTP](../../images/DT-Finding-TTPs.png)

*그림 15: Detective Finding Group과 관련된 TTP*

Finding Group은 관련 리소스와 연결 방법을 시각화하는 생성된 그래프로 표현됩니다. 이 예에서는 무차별 대입 GuardDuty 결과와 관련된 많은 다른 IP 주소가 있으며 이러한 리소스 중 일부는 이 도메인 생성 알고리즘(DGA) 요청과 같은 다른 GuardDuty 결과와 연결되어 있습니다. DGA는 명령 및 제어(C&C) 서버와의 랑데부 지점으로 사용할 수 있는 많은 수의 도메인 이름을 주기적으로 생성하는 데 사용됩니다. GuardDuty 결과 조사 중에 전체 보안 문제에 포함된 다른 리소스가 있다는 것이 명확하지 않을 수 있습니다. Detective Finding Group은 이러한 점들을 연결하는 데 도움이 됩니다.

![Detective Finding Group 그래프](../../images/DT-Graph.png)

*그림 16: Detective Finding Group 그래프 시각화*

이러한 Finding Group을 사용하여 공격이 시작되거나 계속된 다른 IP, EC2 또는 AWS 계정을 조사합니다.

이러한 데이터 소스를 개별적으로 수집하는 경우 이벤트를 리소스와 연관시키는 데이터를 원시 데이터로 사용할 수 있지만 보안 분석가는 로그를 크롤링하기 위해 개별 쿼리를 작성한 다음 나중에 시각화 도구로 그래프를 작성하는 방법을 알아야 합니다. Detective는 이러한 조사 노력과 쿼리 생성 활동을 단축합니다. 대부분의 고객이 기존 SIEM 도구를 Detective로 교체하지는 않지만 SIEM과 함께 Detective를 사용하면 조사 시간을 줄이는 데 효과적입니다.

Detective의 효과를 로그를 별도로 수집하고 집계하는 것과 비교하십시오. 데이터 소스를 개별적으로 수집하고 로그 집계 도구에 저장하는 경우 보안 분석가는 필요한 데이터를 찾기 위해 쿼리를 작성하고 별도로 시각화 도구로 그래프를 작성하는 전문 지식이 필요합니다. Detective는 기본 제공 AWS 데이터 피드를 사용하여 시각화를 제공함으로써 이 경고 --> 로그 --> 쿼리 --> 시각화 프로세스를 단축합니다. 대부분의 고객이 기존 SIEM 도구를 Detective로 교체하지는 않지만 SIEM과 함께 Detective를 사용하면 조사 시간을 줄이는 데 효과적입니다.

#### Detective에서 위협 헌팅

고객은 AWS 리소스 또는 이벤트와 연관시키려는 흥미로운 정보가 있는 경우 Detective를 사용할 수 있습니다. 일반적으로 Detective는 AWS 리소스가 의심스러운 공용 IP와 통신하는 위치를 조사하는 데 사용됩니다. Amazon Detective 콘솔로 이동하여 왼쪽 탐색 창에서 검색 옵션을 클릭합니다. EC2 인스턴스, IP 주소, Kubernetes 주체 또는 컨테이너 이미지를 포함한 다양한 종류의 리소스를 검색할 수 있습니다.

![Detective 검색 옵션](../../images/DT-Search-Options.png)

*그림 17: Detective 검색 옵션*

![Detective IP 검색](../../images/DT-Search-IP.png)

*그림 18: Detective에서 IP 검색*

중요 참고 사항: IP 주소와 관련하여 어떤 IP가 귀하의 것이고 어떤 IP가 공용 IP인지 기억하는 것이 중요합니다. 대부분의 고객 IPv4 주소는 RFC1918 범위(10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)에 있습니다. VPC의 IP 스키마에서 공용 IP를 사용하는 경우 IP를 소유한 사람을 확인하기 위해 때때로 조회를 수행해야 할 수 있습니다.

정보 검색 외에도 [Detective 조사](https://docs.aws.amazon.com/detective/latest/userguide/detective-investigations.html)를 통해 침해 지표(IoC)를 사용하여 IAM 사용자 및 IAM 역할을 조사할 수 있습니다. IoC는 리소스가 보안 인시던트에 관련되어 있는지 확인하는 데 도움이 될 수 있습니다. Detective 조사를 통해 [불가능한 이동](https://www.mimecast.com/blog/impossible-travel-tests-limits-of-anomalous-detection/), 플래그가 지정된 IP 주소 및 Finding Group을 포함한 공격 전술을 조사할 수 있습니다. 초기 보안 조사 단계를 수행하고 Detective가 식별한 위험을 강조하는 보고서를 생성하여 보안 이벤트를 이해하고 대응하는 데 도움을 줍니다.

조사를 시작하려면 **조사 실행**을 선택합니다. 원하는 날짜/시간을 선택하고 확인합니다.

![Detective 조사 시작](../../images/DT-Start-Investigation.png)
*그림 19: IAM 역할 엔터티 페이지의 조사 시작 버튼*

![Detective 조사 확인](../../images/DT-Confirm-Investigation.png)
*그림 20: 조사 설정 확인*

조사가 실행되면 TTP, 위협 인텔리전스 플래그가 지정된 IP 주소, 불가능한 이동, 관련 Finding Group, 관련 결과, 새로운 지리적 위치, 새로운 사용자 에이전트 및 새로운 ASO를 포함한 새로운 정보를 사용할 수 있습니다. 모든 정보는 콘솔에서 제공되며 JSON 형식으로 다운로드할 수 있습니다.

![Detective 조사 결과](../../images/DT-Investigation-Results.png)
*그림 21: Detective 샘플 조사 결과*

#### Detective 임베디드 URL 작성

Detective를 사용하여 보안 이벤트 조사를 시작하는 네 번째 방법은 Splunk Trumpet 프로젝트를 통해 Splunk에서 임베디드 링크를 사용하는 것입니다. Splunk Trumpet 프로젝트를 사용하면 AWS 서비스에서 Splunk로 데이터를 보낼 수 있습니다. Trumpet 프로젝트를 구성하여 Amazon GuardDuty 결과에 대한 Detective URL을 생성할 수 있습니다. 그런 다음 이러한 URL을 사용하여 Splunk에서 해당 Detective 결과 프로필로 직접 피벗할 수 있습니다. Trumpet 프로젝트는 [GitHub](https://github.com/splunk/splunk-aws-project-trumpet)에서 사용할 수 있습니다.

![Detective URL](../../images/DT-URLs.png)

*그림 22: Detective 임베디드 URL 작성*

## 비용 고려 사항

Amazon Detective 가격은 가격 차원 및 일부 가격 예시를 포함하여 [가격 페이지](https://aws.amazon.com/detective/pricing/)에서 자세히 다룹니다. 이 가이드에서는 Detective 지출을 제어하기 위한 몇 가지 옵션을 다룰 것입니다.

1. 30일 무료 평가판을 활용하여 Detective의 가격 책정 방식에 대한 아이디어를 얻으십시오. Detective를 활성화한 후 며칠 후에 돌아와서 왼쪽의 사용량 탭을 확인하십시오.
2. Detective 사용량 페이지에서 **이 계정의 예상 비용** 및 **모든 계정의 예상 비용**에서 예상 비용을 볼 수 있습니다. **구성원 계정별 수집된 볼륨** 옵션을 사용하여 어떤 계정이 비용을 유발하는지 더 잘 이해할 수 있습니다.
3. 일부 고객은 비용을 제어하는 방법으로 샌드박스 또는 비프로덕션으로 분류된 AWS 계정을 제외하기로 선택합니다. 또한 VPC Flow Logs 및 AWS CloudTrail 데이터를 별도로 수집하는 보조 도구가 있는 경우 축소하고 Detective로 보강할 수 있는지 고려하십시오.

![Detective 사용량 페이지](../../images/DT-Usage.png)

*그림 23: Detective 사용량 페이지*

## 리소스

### 워크샵

* [Activation Days](https://awsactivationdays.splashthat.com/)
* [Amazon GuardDuty 워크샵](https://catalog.workshops.aws/guardduty)
* [Amazon Detective 워크샵](https://catalog.workshops.aws/detective)
* [EKS 보안 워크샵](https://catalog.workshops.aws/containersecurity)

### 비디오

* [AWS re:Inforce 2022 - Amazon Detective를 사용하여 보안 조사 개선(TDR302)](https://www.youtube.com/watch?v=vd_VHg6-xWc&t=1796s&pp=ygUTcmVpbmZvcmNlIGRldGVjdGl2ZQ%3D%3D)
* [AWS re:Inforce 2023 - Amazon Detective로 보안 분석 간소화(TDR210)](https://www.youtube.com/watch?v=TWJtvq8pgw8&t)
* [보안 조사를 위해 Amazon Detective를 사용하는 방법](https://www.youtube.com/watch?v=AtUZgdtZCYo)
* [Detective Finding Group에 Amazon Inspector 결과 포함](https://www.youtube.com/watch?v=U5xu6Dy3Pb0&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=9&pp=iAQB)
* [Amazon GuardDuty 위협 탐지를 위한 조사](https://www.youtube.com/watch?v=dFR4Dk4h1Go&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=15&pp=iAQB)
* [Amazon Detective를 사용하여 보안 결과에 대한 근본 원인 분석 수행](https://www.youtube.com/watch?v=nuYHzN02f60&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=31&pp=iAQB)
* [Amazon Detective 시각화 데모](https://www.youtube.com/watch?v=TZZuQrC8ZtA&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=43&pp=iAQB)
* [관련 결과를 그룹화하여 GuardDuty 결과를 조사하는 시간 단축](https://www.youtube.com/watch?v=rPnpPBIRRv8&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=63&pp=iAQB)
* [보안 시나리오 조사 연습](https://www.youtube.com/watch?v=Rz8MvzPfTZA&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=85&pp=iAQB)
* [AWS Organizations와 함께 Amazon Detective 배포](https://www.youtube.com/watch?v=aZRnzBErRsE&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=88&t=1s&pp=iAQB)

### 블로그

* [Amazon Detective Finding Group을 사용하여 보안 인시던트 조사를 개선하는 방법](https://aws.amazon.com/blogs/security/how-to-improve-security-incident-investigations-using-amazon-detective-finding-groups/)
* [Detective Finding Group 시각화로 보안 조사 개선](https://aws.amazon.com/blogs/security/improve-your-security-investigations-with-detective-finding-groups-visualizations/)
* [Amazon GuardDuty를 사용하여 Amazon EKS 클러스터의 보안 문제를 탐지하는 방법 – 파트 1](https://aws.amazon.com/blogs/security/how-to-detect-security-issues-in-amazon-eks-clusters-using-amazon-guardduty-part-1/)
* [Amazon Detective를 사용하여 Amazon EKS 클러스터의 보안 문제를 조사하고 조치하는 방법 – 파트 2](https://aws.amazon.com/blogs/security/how-to-investigate-and-take-action-on-security-issues-in-amazon-eks-clusters-with-amazon-detective-part-2/)
