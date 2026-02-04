# Amazon Inspector

## 소개

Amazon Inspector 모범 사례 가이드에 오신 것을 환영합니다. 본 가이드의 목적은 Amazon EC2, AWS Lambda 함수 및 Amazon ECR과 같은 AWS 워크로드에서 소프트웨어 취약점 및 의도하지 않은 네트워크 노출을 지속적으로 모니터링하기 위해 Amazon Inspector를 활용하는 데 필요한 규범적 지침을 제공하는 것입니다. GitHub를 통해 본 가이드를 게시함으로써 서비스 개선 사항과 사용자 커뮤니티의 피드백을 포함한 시의적절한 권장 사항을 신속하게 반영할 수 있습니다. 본 가이드는 단일 계정에서 Inspector를 처음 배포하는 경우나 기존 다중 계정 배포에서 Inspector를 최적화하는 방법을 찾는 경우 모두에게 유용한 정보를 제공하도록 설계되었습니다.

## 본 가이드 사용 방법

본 가이드는 AWS 계정(및 리소스) 내에서 보안 이벤트, 비정상적인 활동 및 취약점의 모니터링과 해결을 담당하는 보안 실무자를 대상으로 합니다. 모범 사례는 보다 쉬운 이해를 위해 다양한 범주로 구성되어 있습니다. 각 범주에는 간략한 개요로 시작하여 지침 구현을 위한 세부 단계가 이어지는 일련의 모범 사례가 포함되어 있습니다. 주제는 특정 순서로 읽을 필요가 없습니다:

* [Amazon Inspector란 무엇인가](#what-is-amazon-inspector)
* [Amazon Inspector 활성화의 이점은 무엇인가](#what-are-the-benefits-of-enabling-amazon-inspector)
* [시작하기](#getting-started)
    * [AWS SSM agent](#aws-ssm-agent)
    * [배포 고려 사항](#deployment-considerations)
    * [리전 고려 사항](#region-considerations)
* [구현](#implementation)
    * [독립 실행형 계정 활성화](#stand-alone-account-enablement)
    * [다중 계정 조직 활성화](#multi-account-organization-enablement)
* [적용 범위](#coverage)
    * [Amazon EC2 스캔](#amazon-ec2-scanning)
    * [Windows 스캔](#windows-scanning)
    * [ECR 스캔](#ecr-scanning)
    * [Lambda 스캔](#lambda-scanning)
    * [CI/CD](#cicd-scanning)
    * [CIS 스캔](#cis-scans)
* [운영화](#operationalizing)
    * [Inspector 탐지 결과 조치](#actioning-inspector-findings)
    * [소프트웨어 자재 명세서(SBOM) 구성](#software-bill-of-materials-sbom-configuration)
    * [억제 규칙](#suppression-rules)
    * [취약점 데이터베이스 검색](#vulnerability-database-search)
* [비용 고려 사항](#cost-considerations)
* [리소스](#resources)

## Amazon Inspector란 무엇인가?

Amazon Inspector는 소프트웨어 취약점 및 의도하지 않은 네트워크 노출에 대해 AWS 워크로드를 지속적으로 모니터링하는 취약점 관리 서비스입니다. Amazon Inspector는 실행 중인 Amazon EC2 인스턴스, Amazon Elastic Container Registry(Amazon ECR)의 컨테이너 이미지 및 AWS Lambda 함수를 자동으로 검색하고 스캔합니다.

## Amazon Inspector 활성화의 이점은 무엇인가?

Amazon Inspector는 AWS Organization 전체에서 리소스를 지속적으로 검색합니다. Amazon Inspector를 배포한 후 리소스를 식별하고 취약점에 대해 자동으로 평가합니다. 환경의 지속적인 상태를 유지하면서 Amazon Inspector는 리소스가 더 이상 존재하지 않거나 더 중요하게는 새 리소스가 배포될 때를 이해하여 수동 구성 없이 이러한 리소스에 대한 취약점 평가를 시작할 수 있습니다.

새로운 취약점이 발견되더라도 항상 리소스의 현재 상태를 최신 상태로 유지할 수 있도록 취약점에 대해 Amazon EC2 인스턴스, ECR의 이미지 및 Lambda 함수를 포함한 리소스를 지속적으로 평가합니다. Amazon Inspector는 Amazon EC2 인스턴스에 새 패키지 설치, 패치 설치 및 리소스에 영향을 미치는 새로운 CVE(Common Vulnerabilities and Exposures)가 게시되는 것과 같이 새로운 취약점을 도입할 수 있는 변경 사항에 대응하여 리소스를 자동으로 모니터링함으로써 리소스의 수명 주기 전반에 걸쳐 환경을 계속 평가합니다. 기존 보안 스캔 소프트웨어와 달리 Amazon Inspector는 플릿의 성능에 미치는 영향이 최소화됩니다.

Amazon Inspector는 소프트웨어 취약점, 코드 취약점 또는 네트워크 구성 문제를 발견하면 탐지 결과를 생성합니다. 탐지 결과는 취약점을 설명하고 영향을 받는 리소스를 식별하며 취약점의 심각도를 평가하고 해결 지침을 제공합니다. Amazon Inspector는 또한 취약점의 심각도에 대한 컨텍스트를 제공하는 Amazon Inspector 점수를 제공합니다. [Amazon Inspector 점수](https://docs.aws.amazon.com/inspector/latest/user/findings-understanding-score.html#findings-understanding-inspector-score)에 대해 자세히 알아보려면 Inspector 문서를 방문하십시오. Amazon Inspector 콘솔을 사용하여 탐지 결과를 분석하거나 다른 AWS 서비스를 통해 탐지 결과를 보고 처리할 수 있습니다.

Organizations와의 통합을 통해 단일 위치에서 모든 계정의 취약점 태세를 신속하게 배포하고 확인할 수 있으며 조직에 새 계정이 추가될 때 Amazon Inspector가 자동으로 활성화되고 평가를 수행하는지 확인할 수 있습니다. 이를 통해 AWS 환경 전체에서 취약점 관리의 준비 상태를 유지하는 데 필요한 노력이 줄어듭니다.

## 시작하기

이 섹션에서는 AWS Organization에서 Amazon Inspector를 활성화하기 전에 고려해야 할 사항을 다룹니다.

### AWS SSM Agent

필수는 아니지만 Amazon Inspector를 사용하여 Amazon EC2 인스턴스에서 소프트웨어 취약점을 탐지할 때 인스턴스가 [AWS Systems Manager(SSM)](https://aws.amazon.com/systems-manager/)의 관리형 인스턴스인 것이 권장됩니다. SSM 관리형 인스턴스에는 AWS SSM Agent가 설치되어 실행 중이며 SSM에 인스턴스를 관리할 수 있는 권한이 있습니다. [AmazonSSMManagedInstanceCore](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonSSMManagedInstanceCore.html)는 인스턴스 프로파일을 연결할 때 사용하는 권장 정책입니다. 이 정책에는 Amazon Inspector EC2 스캔에 필요한 모든 권한이 있습니다. 이미 SSM을 사용하여 인스턴스를 관리하고 있는 경우 Amazon Inspector가 스캔을 시작하기 위해 추가 단계가 필요하지 않습니다.

지원되지 않는 운영 체제 때문에 SSM 에이전트를 가질 수 없거나 가지지 않는 인스턴스가 있는 경우에도 하이브리드 스캔 모드라고도 하는 Inspector [에이전트 없는 스캔](https://docs.aws.amazon.com/inspector/latest/user/scanning-ec2.html#agentless)을 사용하여 소프트웨어 취약점을 스캔할 수 있습니다. SSM 에이전트를 사용한 스캔은 이벤트 기반 패턴을 사용합니다. 예를 들어 인스턴스에 새 패키지가 설치되면 Inspector가 패키지 평가를 시작합니다. 에이전트 없는 스캔을 사용하면 Inspector는 24시간마다 한 번 인스턴스를 스캔합니다.

AWS SSM Agent는 [일부 Amazon Machine Images(AMI)](https://docs.aws.amazon.com/systems-manager/latest/userguide/ami-preinstalled-agent.html)에서 생성된 Amazon EC2 인스턴스에 기본적으로 설치됩니다. 자세한 내용은 AWS Systems Manager 사용 설명서의 [About SSM Agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/prereqs-ssm-agent.html)를 참조하십시오. 그러나 설치되어 있더라도 AWS SSM Agent를 수동으로 활성화하고 SSM에 인스턴스를 관리할 수 있는 권한을 부여해야 할 수 있습니다.

Amazon EC2 인스턴스에서 취약점을 찾는 데 필요한 가시성을 제공하는 것 외에도 AWS SSM 에이전트의 존재는 자동화된 패치 예약, 유지 관리 기간 생성, 패치 규정 준수 보고, 인스턴스 플릿 전체에서 자동화를 실행하는 메커니즘 제공 및 SSH 또는 RDP에 대한 원격 액세스를 위해 포트 22 또는 3389를 노출할 필요 없이 인스턴스에 대한 안전한 액세스 제공과 같은 활동에 가치를 제공합니다.

AWS SSM 에이전트 사용에 관심이 있는 경우 AWS Systems Manager에서 기본 호스트 관리 활성화에 대한 아래 섹션을 참조하십시오.

#### AWS Systems Manager(SSM)에서 Default Host Management 활성화

Amazon Inspector의 하이브리드 스캔에는 에이전트 기반 스캔과 에이전트 없는 스캔이 포함됩니다. 기본적으로 Amazon Inspector는 적격한 모든 Amazon EC2 인스턴스에서 이러한 스캔 방법을 사용합니다. 에이전트 기반 스캔은 SSM 에이전트를 사용하여 소프트웨어 인벤토리를 수집합니다. 에이전트 없는 스캔은 Amazon EBS 스냅샷을 사용하여 소프트웨어 인벤토리를 수집합니다. 기본적으로 SSM 에이전트는 Amazon Machine Images를 기반으로 하는 Amazon EC2 인스턴스에 이미 설치되어 있습니다. 그러나 경우에 따라 SSM 에이전트를 수동으로 활성화해야 할 수 있습니다. 자세한 내용은 AWS Systems Manager 사용 설명서의 [Working with the SSM agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html)를 참조하십시오. Systems Manager로 EC2 인스턴스를 자동으로 관리하려면 Default Host Management Configuration 설정을 사용하십시오.

Amazon EC2 인스턴스의 네트워크 도달 가능성 평가, 컨테이너 이미지의 취약점 스캔 또는 Lambda 함수의 취약점 스캔에는 에이전트가 필요하지 않습니다.

1. AWS Organizations의 조직 관리 계정에 로그인합니다.
2. Systems Manager로 이동하여 [Quick Setup](https://console.aws.amazon.com/systems-manager/quick-setup) 페이지를 엽니다.
3. **Default Host Management Configuration**에 대해 **Create**를 클릭합니다.
4. **Enable automatic updates of the SSM Agent every two weeks** 상자가 선택되어 있는지 확인합니다.
5. **Create**를 클릭합니다. "Your Default Host Management Configuration Quick Setup is being updated."라는 배너가 표시됩니다. 에이전트가 Systems Manager에 연결되고 EC2 인스턴스가 Systems Manager Inventory에 관리형 인스턴스로 표시되는 데 일반적으로 최대 30분이 걸립니다(Inspector가 에이전트 기반 스캔을 수행하는 데 필요).
6. Host Management Configuration이 완료될 때까지 30분을 기다리지 않고 다음 단계를 계속 진행할 수 있습니다.

### 배포 고려 사항

AWS Organization 전체에 Inspector를 배포하려면 콘솔, CLI 또는 API에서 수행하든 AWS 관리 계정과 보안 도구 계정에서 활성화해야 합니다. 보안 도구 계정의 개념에 익숙하지 않은 경우 Security Reference Architecture의 [권장 계정 구조](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/organizations.html)를 숙지하는 것이 좋습니다. 요약하자면, 이것은 Amazon Inspector, Amazon GuardDuty, AWS Security Hub 및 Amazon Detective와 같은 네이티브 AWS 보안 서비스의 위임된 관리자 계정으로 사용되는 AWS Organization의 전용 계정입니다.

### 리전 고려 사항

Amazon Inspector는 리전 서비스입니다. 즉, Amazon Inspector를 사용하려면 취약점 모니터링 기능을 원하는 모든 리전에서 활성화해야 합니다. [GitHub](https://github.com/aws-samples/inspector2-enablement-with-cli)의 AWS CLI 스크립트를 사용하여 모든 계정 및 리전에서 Inspector를 활성화하거나 콘솔에서 리전 간에 전환하여 이 작업을 수행할 수 있습니다.

자주 묻는 질문은 "회사가 적극적으로 사용하지 않는 리전에서 보안 서비스를 사용해야 합니까?"입니다. 위험 선호도, 예산 및 보상 통제 등 답변에 영향을 미칠 수 있는 많은 요인이 있기 때문에 이 질문에 대한 간단한 답변은 없습니다. 이 결정을 내릴 때 염두에 두어야 할 몇 가지 사항이 있습니다.

1. Inspector는 사용한 만큼만 비용을 지불하는 서비스입니다. 따라서 사용량이 최소인 리전이 있는 경우 Inspector를 켜서 리전에서 사용되는 리소스와 관련된 불균형한 비용을 발생시키지 않고 취약점 환경에 대한 가시성을 제공할 수 있습니다.
2. 이 리전의 모든 사용을 차단하는 서비스 제어 정책과 같이 사용되지 않는 리전과 관련된 보상 통제가 있는 경우. 향후 언제든지 이 보상 통제가 변경되는 것을 탐지하기 위해 탐지 통제를 갖는 것이 여전히 권장됩니다.

이 결정은 궁극적으로 귀사의 상황에 따라 귀사가 내려야 하는 결정이지만 보안의 경험 법칙은 조사 전에 더 많은 가시성을 확보하는 것이 더 낫다는 것입니다. 사후에 얻기가 어렵거나 불가능한 경우가 많기 때문입니다.


## 구현

이 섹션에서는 독립 실행형 계정 및 다중 계정 조직에서 Inspector를 활성화하기 위한 최소 요구 사항을 다룹니다.

### 독립 실행형 계정 활성화

![Inspector Getting started](../../images/I-Getting-Started.png)
*그림 1: Inspector 시작 페이지*

첫 번째 단계는 Inspector 콘솔로 이동하는 것입니다. Inspector 콘솔에 들어가면 시작 버튼이 있는 랜딩 페이지가 표시됩니다. "Get started"를 클릭합니다.

![Inspector activation](../../images/I-Activate.png)
*그림 2: Inspector 활성화 페이지*

"Get started"를 클릭하면 Inspector 활성화 페이지로 이동합니다. Service role permissions를 검토하여 Inspector가 기능을 제공하는 데 필요한 권한을 이해한 다음 "Activate Inspector"라고 표시된 노란색 상자를 선택하여 Inspector를 활성화합니다.

이 두 단계가 완료되면 이 계정에서 Inspector가 활성화됩니다. Inspector의 모든 관련 기능을 활성화했는지 확인하려면 아래 섹션을 참조하는 것이 중요합니다.

### 다중 계정 조직 활성화

AWS Organization에서 Inspector를 처음 구현할 때 위에서 언급한 대로 Inspector를 사용하려는 각 리전의 조직 관리 계정에서 위임된 관리자를 설정합니다. 모든 리전에서 동일한 계정을 사용해야 한다는 점을 명심하십시오.

![Inspector Getting started](../../images/I-Getting-Started.png)
*그림 3: Inspector 시작 페이지*

첫 번째 단계는 조직 관리 계정의 Inspector 콘솔로 이동하는 것입니다. Inspector 콘솔에 들어가면 시작 버튼이 있는 랜딩 페이지가 표시됩니다. "Get started"를 클릭합니다.

![Inspector delegated admin](../../images/I-Delegate-Admin.png)
*그림 4: 위임된 관리자 할당 페이지*

"Get started"를 클릭하면 Inspector 활성화 페이지로 이동합니다. 다음으로 Inspector 위임된 관리자 계정으로 지정하려는 계정의 계정 ID를 입력해야 합니다. 계정 ID를 입력한 후 "Delegate"를 선택합니다. 이 시점에서 위임된 관리자로 전환하여 AWS Organization 전체에서 Amazon Inspector 구성을 완료합니다.

![Inspector Getting started](../../images/I-Getting-Started.png)
*그림 5: Inspector 시작 페이지*

Inspector 위임된 관리자 계정에 들어가면 다시 시작 버튼이 있는 랜딩 페이지가 표시됩니다. "Get started"를 클릭합니다. 이것이 완료되면 Inspector가 활성화됩니다. 다룰 몇 가지 배포 고려 사항이 더 있습니다.

![Inspector Account management](../../images/I-Account-Management.png)
*그림 6: Inspector 계정 관리 페이지*

다음으로 모든 현재 계정 및 Amazon EC2 스캔, Amazon ECR 스캔, AWS Lambda 표준 스캔 및 AWS Lambda 코드 스캔을 포함한 모든 스캔 유형에 대한 스캔을 활성화해야 합니다. "Automatically activate Inspector for new member accounts"를 활성화하는 것도 강력히 권장됩니다. 이를 통해 조직에서 새 계정이 생성될 때 취약점 적용 범위에 격차가 없음을 이해할 수 있습니다. 이렇게 하면 새 계정 온보딩 프로세스가 자동화되고 모든 탐지 결과가 위임된 관리자 계정에 자동으로 집계되어 수동 작업이 줄어들고 시간이 절약됩니다.


* Inspector의 위임된 관리자 계정에 로그인합니다.
* Inspector 콘솔을 엽니다.
* [Account management](https://console.aws.amazon.com/inspector/v2/home?#/settings/account-management/accounts) 페이지에서 **Automatically activate Inspector for new member accounts**에서 **Automatically activate Inspector for new member accounts** 옵션을 켭니다.
* 모든 옵션이 선택되어 있는지 확인하고 **Save**를 클릭합니다. "You have successfully updated the auto-activate scan settings for your organization."이라는 메시지가 포함된 배너가 표시됩니다.
* **Organization** 아래에 계정이 나열된 위치로 스크롤합니다. 보유한 계정 수에 따라 오른쪽의 기어 아이콘을 클릭하여 페이지당 50개의 계정을 볼 수 있습니다.
* 페이지당 모든 계정을 선택합니다. **Activate** 드롭다운 메뉴를 클릭하고 모든 옵션을 선택한 다음 **Submit**을 클릭합니다.
* 몇 분 내에 리소스 스캔이 시작되고 [Findings](https://console.aws.amazon.com/inspector/v2/home?#/findings/all) 페이지에서 생성된 탐지 결과를 볼 수 있습니다.

또는 CLI를 사용하여 모든 계정 및 리전에서 Inspector를 활성화할 수 있습니다. [Github의 활성화 스크립트](https://github.com/aws-samples/inspector2-enablement-with-cli)를 확인하십시오.

## 적용 범위

이 섹션에서는 환경의 지원되는 모든 리소스에서 Inspector를 구성하기 위한 다양한 기능의 활성화를 다룹니다.

### Amazon EC2 스캔

Organization에서 Inspector를 활성화한 후 처리해야 하는 두 가지 Amazon EC2 스캔 관련 구성이 더 있습니다.

Amazon EC2용 Inspector Deep Inspection은 Inspector가 Linux 기반 Amazon EC2 인스턴스에서 애플리케이션 프로그래밍 언어 패키지에 대한 패키지 취약점을 탐지할 수 있는 기능을 제공합니다. Inspector를 활성화하는 새 계정은 기본적으로 딥 인스펙션이 활성화됩니다. 이전에 활성화된 계정이 있고 이것이 활성화되어 있는지 확인하려면 Inspector 콘솔의 계정 관리 페이지를 보고 아래 그림과 같이 딥 인스펙션 스캔이 비활성화된 계정을 찾을 수 있습니다. Inspector Deep inspection 및 딥 인스펙션 스캔 구성 방법에 대한 자세한 내용은 [Inspector Deep inspection 문서](https://docs.aws.amazon.com/inspector/latest/user/scanning-ec2.html#deep-inspection)를 참조하십시오.

![Inspector deep scan activated](../../images/I-Deep-Scan-Activated.png)
*그림 7: Inspector EC2 딥 스캔 활성화됨*

![Inspector deep scan deactivated](../../images/I-Deep-Scan-Deactivated.png)
*그림 8: Inspector EC2 딥 스캔 비활성화됨*

기본적으로 Deep Inspection 스캔은 아래의 기본 경로를 확인합니다.

* /usr/lib
* /usr/lib64
* /usr/local/lib
* /usr/local/lib64

모든 개별 계정은 최대 5개의 사용자 지정 경로를 추가할 수 있습니다. 위임된 관리자는 조직 전체에 적용될 추가 5개의 경로를 추가할 수 있습니다. 이는 조직의 계정당 최대 10개의 사용자 지정 경로를 스캔할 수 있습니다. 사용자 지정 경로의 예는 "/home/usr1/project01"입니다. Inspector Deep inspection에 대한 완전한 가시성을 제공하기 위해 조직에 적용 가능한 모든 사용자 지정 경로를 입력하는 것이 좋습니다.

![Inspector EC2 scan settings](../../images/I-EC2-Settings.png)
*그림 9: Inspector EC2 스캔 설정*

### Windows 스캔

Amazon Inspector는 지원되는 모든 Windows 인스턴스를 자동으로 검색하고 추가 작업 없이 지속적인 스캔에 포함합니다. Linux 기반 인스턴스에 대한 스캔과 달리 Amazon Inspector는 정기적으로 Windows 스캔을 실행합니다. Windows 인스턴스는 검색 시 처음 스캔된 다음 6시간마다 스캔됩니다. 그러나 기본 6시간 스캔 간격은 조정 가능합니다. 6시간이 원하는 스캔 빈도인 경우 조치가 필요하지 않습니다. 스캔 빈도를 조정하려면 aws ssm update-association CLI 명령을 사용하여 이 작업을 수행할 수 있습니다. 스캔 빈도를 조정하는 방법에 대한 단계는 [Inspector Windows 스캔 문서](https://docs.aws.amazon.com/inspector/latest/user/scanning-ec2.html#windows-scanning)를 참조하십시오.

### ECR 스캔

ECR 스캔을 처음 활성화하고 리포지토리가 지속적인 스캔으로 구성된 경우 Amazon Inspector는 30일 이내에 푸시했거나 지난 90일 이내에 가져온 모든 적격 이미지를 탐지합니다. Amazon Inspector는 지난 90일 이내에(기본적으로) 푸시되거나 가져온 이미지 또는 구성한 ECR 재스캔 기간 내에 푸시되거나 가져온 이미지를 계속 모니터링합니다. 환경의 애플리케이션에 더 이상 사용되지 않거나 보유하고 있는 다른 보상 통제를 기반으로 정의된 기간 후에 인스턴스 스캔을 중지하도록 선택할 수 있습니다. 이미지 푸시 날짜 또는 이미지 가져오기 날짜를 기반으로 재스캔하도록 Inspector를 구성할 수 있습니다. 예를 들어 푸시 날짜에 대해 60일을 선택하고 가져오기 날짜 구성에 대해 180일을 선택하면 Amazon Inspector는 지난 60일 동안 푸시되었거나 지난 180일 동안 한 번 이상 가져온 이미지를 계속 모니터링합니다. 재스캔 기간 설정을 결정할 때 조직의 애플리케이션 배포 패턴을 이해하는 것이 좋습니다. 예를 들어 이미지를 자주 빌드하는 경우 더 짧은 스캔 기간을 원할 수 있습니다. 또한 [ECR 수명 주기 정책](https://docs.aws.amazon.com/AmazonECR/latest/userguide/LifecyclePolicies.html)을 활용하는 경우 이미지가 ECR에 존재하는 기간에 영향을 미칠 수 있습니다. 이 기간을 설정하는 방법에 대한 자세한 지침은 [ECR 자동화된 재스캔 기간 문서](https://docs.aws.amazon.com/inspector/latest/user/scanning-ecr.html#scan-duration-ecr)를 참조하십시오.

![Inspector ECR scan settings](../../images/I-ECR-Settings.png)
*그림 10: Inspector ECR 스캔 설정*

### Lambda 스캔

Lambda 스캔에는 두 가지 다른 기능이 있습니다. 첫 번째는 프로그래밍 언어 및 패키지의 소프트웨어 취약점에 대해 Lambda 함수를 스캔하는 기능입니다. 두 번째는 코드 취약점에 대해 사용자 지정 권한 있는 애플리케이션 코드를 스캔하는 기능입니다. 계정 관리에서 이 두 가지가 모두 활성화되어 있는지 확인하여 완전한 Inspector 가시성을 제공하십시오. Lambda 스캔 또는 코드 스캔이 비활성화된 계정이 표시되면 [활성화에 대한 자세한 단계는 Lambda 스캔 문서](https://docs.aws.amazon.com/inspector/latest/user/scanning-lambda.html)를 참조하십시오.

![Inspector Lambda scanning](../../images/I-Auto-Enable.png)
*그림 11: Inspector 계정 관리 페이지*

### CI/CD 스캔

Amazon Inspector를 Jenkins와 같은 CI/CD 도구에 직접 통합할 수 있습니다. [Jenkins](https://docs.aws.amazon.com/inspector/latest/user/cicd-jenkins.html) 및 [TeamCity](https://docs.aws.amazon.com/inspector/latest/user/cicd-teamcity.html) 도구의 경우 Inspector에는 이러한 파이프라인에 직접 취약점 스캔을 추가할 수 있도록 설치할 수 있는 전용 플러그인이 있습니다. 이러한 플러그인은 탐지 결과 심각도를 기반으로 통과 또는 실패 메커니즘으로 사용할 수 있습니다.

Amazon Inspector가 CI/CD 솔루션에 대한 플러그인을 제공하지 않는 경우 Amazon Inspector SBOM Generator와 Amazon Inspector Scan API의 조합을 사용하여 자체 [사용자 지정 CI/CD 통합](https://docs.aws.amazon.com/inspector/latest/user/scanning-cicd.html)을 생성할 수 있습니다.

### CIS 스캔

기존 취약점 스캔 외에도 많은 고객이 CIS Benchmarks 준수를 평가하기 위한 스캔을 실행하기를 원합니다. 익숙하지 않은 경우 CIS(Center for Internet Security)의 CIS Benchmarks는 보안 실무자가 사이버 보안 방어를 구현하고 관리하는 데 도움이 되는 전 세계적으로 인정되고 합의 기반의 모범 사례 세트입니다. 전 세계 보안 전문가 커뮤니티와 함께 개발된 이 가이드라인은 조직이 새로운 위험에 대해 사전에 대비하는 데 도움이 됩니다. 기업은 CIS Benchmark 가이드라인을 구현하여 디지털 자산의 구성 기반 보안 취약점을 제한합니다. AWS Security Hub를 사용하여 CIS Benchmarks에 대해 AWS 환경을 확인할 수 있는 것처럼 Inspector를 사용하면 AWS의 운영 체제에 대해 CIS Benchmarks를 실행할 수 있습니다.

딥 스캔을 활성화해야 하고 인스턴스에 SSM 에이전트가 있어야 하는 것과 같은 [CIS 스캔 실행에 필요한 요구 사항](https://docs.aws.amazon.com/inspector/latest/user/scanning-cis.html#cis-requirements)을 숙지하는 것부터 시작하십시오.

다음으로 구성하기 전에 CIS 스캔에 대한 원하는 결과를 생각해 보는 것이 좋습니다. 예를 들어 아래에 나열된 것과 같은 구성 전에 질문에 답해야 합니다. 이는 완전한 목록이 아닙니다. 이렇게 하면 전반적인 보안 목표에 도움이 되지 않는 추가 탐지 결과를 생성하지 않고 CIS 스캔을 최대한 활용하는 데 도움이 됩니다.

* 이미지를 얼마나 자주 빌드합니까? 빈번한 스캔을 실행해야 합니까, 아니면 빈번한 이미지 빌드를 기반으로 일회성 스캔을 실행해야 합니까?
* 어떤 이미지/환경에서 레벨 1 대 레벨 2 스캔을 실행해야 합니까?
* 위임된 관리자 계정에서 AWS Organization 전체에 CIS 스캔을 푸시합니까, 아니면 개별 계정 소유자가 계정 수준에서 생성할 책임이 있습니까?
* 스캔 구성 설정을 구성할 때 인스턴스를 대상으로 사용할 수 있는 태그가 있습니까?


## 운영화

### Inspector 탐지 결과 조치

많은 조직에는 잘 확립된 취약점 관리 프로그램이 있지만 그렇지 않거나 현재 수행하는 작업이 Amazon Inspector와 작동하는지 확실하지 않은 경우 고려해야 할 몇 가지 중요한 주제를 강조하고자 합니다.

![Inspector finding detail](../../images/I-Finding-Detail.png)
*그림 12: Inspector 탐지 결과 세부 정보*

![Inspector score](../../images/I-Inspector-Score.png)
*그림 13: Inspector 점수 예제*

![Inspector code remediation](../../images/I-Code-Remediation.png)
*그림 14: 생성형 AI 기반 코드 해결 권장 사항*

Inspector 취약점 탐지 결과는 영향을 받는 리소스, CVE 및 생성형 AI 기반 코드 해결 제안에 대한 훌륭한 세부 정보를 제공하여 보안 및 애플리케이션 팀이 AWS 환경의 취약점을 더 빠르게 해결하는 데 필요한 컨텍스트를 제공합니다. Inspector 탐지 결과는 상태 저장형이기도 합니다. 즉, 취약점이 포함된 패키지를 업데이트하면 Inspector가 패키지 변경을 확인하고 평가를 시작하며 취약점이 실제로 해결된 경우 탐지 결과를 닫습니다. 이러한 이해는 Inspector 탐지 결과를 처리하는 방법을 이해하는 데 중요합니다. 아래에서는 고려해야 할 몇 가지 다른 주제에 대해 자세히 설명합니다.

1. 효과적인 취약점 관리 프로그램의 중요한 구성 요소는 보안 탐지 결과를 평가하고 우선순위를 지정하는 기능입니다. 여기에서 컨텍스트, 조직 기록 및 탐지 시스템 조정을 가져오는 것이 중요합니다. 보안 탐지 결과의 우선순위 지정은 적절한 대응 수준 속도를 설정하는 데 도움이 됩니다. 모든 중요 및 높은 심각도 탐지 결과의 조사를 우선순위로 지정하는 것이 좋습니다.
2. Inspector가 탐지 결과를 생성할 때 수행할 작업을 이해합니다. 예를 들어 Amazon Inspector에는 [여기](https://docs.aws.amazon.com/inspector/latest/user/findings-understanding-severity.html)에 설명된 5가지 다른 탐지 결과 심각도 수준이 있습니다. 조직의 팀이 탐지 결과를 얼마나 빨리 해결해야 하는지 이해하는 것이 중요합니다. 이는 심각도에 따라 달라질 가능성이 높습니다. 정보 제공 탐지 결과보다 중요한 탐지 결과에 더 빠르게 대응하기를 원할 것이기 때문입니다.
3. 대응 시간을 이해한 후 새로운 취약점에 대해 경고하는 방법에 집중하는 것이 중요합니다. 예를 들어 통합하려는 티켓팅 시스템이 있습니까? 그렇다면 AWS Security Hub가 도움을 줄 수 있는 것입니다. 이러한 경고를 해결을 위해 애플리케이션 팀에 직접 보내거나 먼저 보안을 통해 승인해야 할 수도 있습니다. Amazon EventBridge 및 AWS Security Hub와 Inspector의 통합을 통해 다양한 경고 및 추적 옵션이 있습니다. 조직에 가장 적합한 것이 무엇인지 이해하고 이 워크플로를 설정하는 것이 중요합니다.
4. 취약점 해결은 애플리케이션 코드에서 사용하는 패키지 업데이트의 영향을 이해하는 애플리케이션 소유자에게 푸시되어야 합니다. 보안 팀이 소프트웨어 패치를 담당하는 것은 안티 패턴입니다. 위험, 수용 및/또는 완화를 전달하기 위해 보안 팀 간에 양방향 통신 채널을 설정해야 합니다.
5. 이상적으로 모든 조직에서 우리는 인간의 반복적인 수동 작업과 관련된 시간과 잠재적 오류를 절약하기 위해 가능한 한 많이 자동화하기를 원합니다. 불행히도 이것은 항상 100% 수행될 수 없지만 항상 거기에 도달하기 위해 노력해야 합니다. 먼저 [AWS Systems Manager Patch Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager.html)와 같은 취약점 해결을 자동화하는 데 사용할 서비스를 작업합니다. 그런 다음 자동으로 해결하는 것이 편한 환경, 리소스 및 CVE를 이해합니다. 생성할 수 있는 자동화의 양은 리소스 타이밍 및 능력에 크게 의존합니다. 위험이 자동화를 더 진행하는 데 소비한 시간과 다른 우선순위가 높은 프로젝트 작업을 정당화할 만큼 크지 않은 교차점이 있을 것입니다. 이를 구축하기 시작하면 AWS에서 패치 관리를 자동화하는 데 도움이 될 수 있는 많은 다양한 리소스가 있습니다. 이에 대한 여러 리소스가 있으므로 아래 리소스 섹션에서 살펴봐야 할 유용한 리소스의 전용 섹션을 만들었습니다.
6. 가능한 경우 자동화하는 것 외에도 많은 고객이 Amazon Inspector를 사용하여 개발 수명 주기 초기에 리소스의 취약점을 찾고 프로덕션 환경에 배포되기 전에 해결합니다. 개발 및 테스트 중에 개발 및 스테이징 계정에서 Amazon Inspector를 사용하면 프로덕션에 배포하기 전에 애플리케이션에 존재하는 취약점을 이해할 수 있는 가시성을 제공합니다. 리소스 섹션에는 Inspector가 CI/CD 파이프라인에 어떻게 적합한지 다루는 여러 블로그도 있습니다.

![Inspector finding flow](../../images/I-Finding-Flow.png)
*그림 15: Inspector 탐지 결과 흐름*

### 소프트웨어 자재 명세서(SBOM) 구성

Amazon Inspector에서 소프트웨어 자재 명세서 또는 줄여서 SBOM을 내보낼 수 있습니다. 익숙하지 않은 경우 SBOM은 코드베이스의 모든 오픈 소스 및 타사 소프트웨어 구성 요소의 중첩된 인벤토리입니다. 이를 통해 일반적으로 사용되는 패키지 및 조직 전체의 관련 취약점과 같은 소프트웨어 공급에 대한 정보에 대한 가시성을 얻을 수 있습니다.

SBOM을 내보내려면 콘솔 또는 API를 사용하여 이를 설정해야 합니다. 이를 구성하는 방법에 대한 단계는 [Inspector SBOM 문서](https://docs.aws.amazon.com/inspector/latest/user/sbom-export.html)에서 찾을 수 있습니다. 이것이 일회성 내보내기라는 점을 명심하는 것이 중요합니다. 정기적으로 이 작업을 수행해야 하는 경우 정기적으로 create sbom export API를 사용하는 Lambda 함수를 설정하여 이러한 SBOM을 자동으로 생성하는 것이 좋습니다. 이렇게 하면 SBOM을 확인해야 하는 경우 최신 SBOM을 갖는 데 도움이 됩니다. 이것은 이벤트 기반일 수도 있습니다. 예를 들어 인스턴스가 생성될 때 SBOM 내보내기를 실행합니다.

![Inspector SBOM settings](../../images/I-SBOM.png)
*그림 16: Inspector SBOM 설정*

### 억제 규칙

보상 통제 때문에 적용되지 않거나 해결할 수 없고 수용된 위험으로 분류한 환경의 CVE가 있을 수 있습니다. 이러한 상황의 경우 Inspector에서 억제 규칙을 생성할 수 있습니다. 억제 규칙을 사용하여 지정된 기준과 일치하는 Amazon Inspector 탐지 결과를 자동으로 제외할 수 있습니다. 예를 들어 낮은 취약점 점수가 있는 모든 탐지 결과를 억제하는 규칙을 생성할 수 있습니다. 억제 규칙은 탐지 결과 자체에 영향을 미치지 않으며 Amazon Inspector가 탐지 결과를 생성하는 것을 방지하지 않습니다. 억제 규칙은 탐지 결과 목록을 필터링하는 데만 사용됩니다. Amazon Inspector가 억제 규칙과 일치하는 새 탐지 결과를 생성하면 서비스는 자동으로 탐지 결과의 상태를 Suppressed로 설정합니다. 억제 규칙 기준과 일치하는 탐지 결과는 기본적으로 콘솔에 표시되지 않습니다.

또한 Inspector 및 기타 AWS 보안 서비스의 집계 지점으로 AWS Security Hub를 사용하는 경우 자동화 규칙을 사용하여 탐지 결과를 처리할 수 있습니다. 예를 들어 자동화 규칙을 사용하여 특정 프로덕션 계정에 대한 모든 탐지 결과를 즉각적인 조치가 필요한 심각도로 업그레이드하거나 공개 리소스를 허용하지 않는 통제가 있는 특정 환경에 대한 탐지 결과의 심각도를 다운그레이드할 수 있습니다. AWS Security Hub 자동화 규칙에 대해 자세히 알아보려면 [AWS Security Hub 자동화 규칙 문서](https://docs.aws.amazon.com/securityhub/latest/userguide/automation-rules.html)를 참조하십시오.

### 취약점 데이터베이스 검색

Amazon Inspector는 고객이 최신 정보에 대해 환경을 평가할 수 있도록 최신 CVE로 취약점 데이터베이스를 지속적으로 업데이트합니다. 때때로 "Inspector가 이 CVE를 찾고 있습니까?"라고 물을 수 있습니다. AWS 콘솔의 취약점 데이터베이스 검색 기능을 사용하여 CVE(Common Vulnerability and Enumerations) ID(예: "CVE-2023-1264")를 제공하기만 하면 이 질문에 답하는 데 도움이 될 수 있습니다. 이를 통해 Inspector 스캔 엔진이 다루는 CVE를 확인하고 CVE에 대한 예비 조사를 수행할 수 있습니다.

![Inspector vulnerability database](../../images/I-Vuln-DB.png)
*그림 17: Inspector 취약점 데이터베이스 페이지*

## 비용 고려 사항

Amazon Inspector 가격은 가격 구성 요소를 다루는 가격 페이지와 환경에서 가격이 어떻게 보일 수 있는지에 대한 예를 다루는 여러 가격 예제에서 철저하게 다룹니다. 여기서는 이를 다루지 않고 대신 자주 묻는 두 가지 질문에 집중합니다. 1. 우리 조직은 비용에 민감하거나 비용 최적화 작업을 진행 중입니다. Amazon Inspector를 비용 효율적인 방식으로 사용하고 있는지 확인하고 싶습니다. 2. Inspector를 테스트하거나 환경에서 Inspector를 활성화하려고 하지만 시작하기 전에 비용을 추정하고 싶습니다.

Amazon Inspector는 환경의 사용량에 따라 요금을 부과하는 비용 효율적인 서비스입니다. Inspector를 비용 효율적으로 사용하고 있는지 확인하려면 사용량 기반이므로 환경에서 사용되지 않는 추가 리소스가 없는지 확인해야 한다는 것을 이해하는 것이 중요합니다. 이렇게 하면 Inspector 비용뿐만 아니라 Amazon EC2, ECR 및 Lambda 비용도 절약할 수 있습니다. 둘째, Amazon Inspector의 각 스캔 기능은 선택 사항이므로 필요한 것을 선택할 수 있습니다. 환경의 취약점에 대한 가시성 부족을 초래하고 불필요한 문제를 야기할 수 있으므로 기능을 비활성화하는 것은 권장되지 않지만 모든 조직에는 Amazon Inspector 기능을 사용할 때 고려해야 할 예산 제한이 있습니다. 사용할 수 있지만 신중하게 평가해야 하는 또 다른 옵션은 EC2 인스턴스에 대한 스캔을 제외하는 데 사용할 수 있는 태그입니다. 이 태그를 구성하는 방법에 대한 자세한 내용은 [Amazon EC2 인스턴스 스캔 문서](https://docs.aws.amazon.com/inspector/latest/user/scanning-ec2.html#exclude-ec2)를 참조하십시오.

Amazon Inspector로 비용을 추정하는 가장 좋은 방법은 15일 무료 평가판을 활용하는 것입니다. Inspector는 몇 분 안에 조직의 수백 또는 수천 개의 계정에서 켜고 끌 수 있습니다. Inspector를 활성화하면 15일 무료 평가판 이후 조직에서 Inspector를 실행하는 것과 관련된 비용을 확인할 수 있습니다.

## 문제 해결
### AWS Systems Manager(SSM) Agent 문제 해결
SSM 에이전트가 제대로 작동하지 않게 만드는 여러 문제가 있을 수 있습니다. Systems Manager Automation runbook을 사용하여 SSM이 관리할 수 없는 EC2 인스턴스를 자동으로 문제 해결할 수 있습니다

* [이 링크](https://us-east-1.console.aws.amazon.com/systems-manager/automation/execute/AWSSupport-TroubleshootManagedInstance?region=us-east-1)를 열어 Systems Manager Automation runbook을 구성합니다.
* 문제를 해결하려는 EC2 인스턴스의 AWS 리전으로 이동합니다.
* **Input parameters**에서 드롭다운을 "Show managed instances only"에서 "Show all instances"로 변경합니다.
* 문제를 해결하려는 EC2 인스턴스를 선택합니다.
* 다른 모든 설정을 그대로 두고 페이지 하단에서 **Execute**를 클릭합니다. 이 프로세스는 일반적으로 완료하는 데 최대 5분이 걸립니다.
* 자동화 문서의 Overall status가 Success가 되면 Outputs 섹션을 확장합니다.
* 출력을 검토하여 SSM 구성의 특정 문제를 확인합니다.


## 리소스

### 워크샵

* [Activation Days](https://awsactivationdays.splashthat.com/)
* [Amazon Inspector workshop](https://catalog.workshops.aws/inspector/en-US)
* [Amazon Detective workshop](https://catalog.workshops.aws/detective)
* [EKS security workshop](https://catalog.workshops.aws/containersecurity)

### 데모 비디오

* [Enhance workload security with agentless scanning and CI/CD integration](https://www.youtube.com/watch?v=5ngtzZHSwqU&list=PLB3flZ7qA4xu__uOEfpc-coXm04swNWva&index=5&pp=iAQB)
* [Inspector overview demo](https://www.youtube.com/watch?v=Nx8s7lwapoE&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=87&t=608s&pp=iAQB)
* [Vulnerability intelligence database search](https://www.youtube.com/watch?v=viAn4E7uwRU&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=18&pp=iAQB)
* [Windows support for continual EC2 vulnerability scanning](https://www.youtube.com/watch?v=ukvG_oRZ9iQ&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=19&pp=iAQB)
* [Software bill of materials export capability](https://www.youtube.com/watch?v=6dUvnDx4D-Y&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=20&pp=iAQB)
* [Inspector deep inspection of EC2 instances](https://www.youtube.com/watch?v=o0TAwqYN5rI&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=27&t=199s&pp=iAQB)
* [How to use Lambda code scanning](https://www.youtube.com/watch?v=VjIhTXeIgM0&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=37&pp=iAQB)
* [AWS Lambda functions support](https://www.youtube.com/watch?v=XXlY1yF_nUo&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=55&pp=iAQB)
* [Inspector for Lambda workloads](https://www.youtube.com/watch?v=BsrRibUfQls&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=61&pp=iAQB)
* [Inspector suppression rules demo](https://www.youtube.com/watch?v=JhtdPhuAVGM&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=90&pp=iAQB)

### 블로그

* [Use Amazon Inspector to manage your build and deploy pipelines for containerized applications](https://aws.amazon.com/blogs/security/use-amazon-inspector-to-manage-your-build-and-deploy-pipelines-for-containerized-applications/)
* [How to scan EC2 AMIs using Amazon Inspector](https://aws.amazon.com/blogs/security/how-to-scan-ec2-amis-using-amazon-inspector/)
* [Automate vulnerability management and remediation in AWS using Amazon Inspector and AWS Systems manager part 1](https://aws.amazon.com/blogs/mt/automate-vulnerability-management-and-remediation-in-aws-using-amazon-inspector-and-aws-systems-manager-part-1/)
* [Automate vulnerability management and remediation in AWS using Amazon Inspector and AWS Systems manager part 2](https://aws.amazon.com/blogs/mt/automate-vulnerability-management-and-remediation-in-aws-using-amazon-inspector-%20and-aws-systems-manager-part-2/)

### 기타 리소스

* [Building a scalable vulnerability management program on AWS - Guide](https://docs.aws.amazon.com/prescriptive-guidance/latest/vulnerability-management/introduction.html)
