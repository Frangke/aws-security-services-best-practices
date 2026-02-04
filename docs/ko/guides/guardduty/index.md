# Amazon GuardDuty

## 소개

Amazon GuardDuty 모범 사례 가이드에 오신 것을 환영합니다. 본 가이드의 목적은 AWS 계정 및 리소스의 지속적인 모니터링을 위해 Amazon GuardDuty를 활용하는 데 필요한 규범적 지침을 제공하는 것입니다. GitHub를 통해 본 가이드를 게시함으로써 서비스 개선 사항과 사용자 커뮤니티의 피드백을 포함한 시의적절한 권장 사항을 신속하게 반영할 수 있습니다. 본 가이드는 단일 계정에서 GuardDuty를 처음 배포하는 경우나 기존 다중 계정 배포에서 GuardDuty를 최적화하는 방법을 찾는 경우 모두에게 유용한 정보를 제공하도록 설계되었습니다.

## 본 가이드 사용 방법

본 가이드는 AWS 계정(및 리소스) 내에서 위협 및 악의적인 활동의 모니터링과 해결을 담당하는 보안 실무자를 대상으로 합니다. 모범 사례는 보다 쉬운 이해를 위해 세 가지 범주로 구성되어 있습니다. 각 범주에는 간략한 개요로 시작하여 지침 구현을 위한 세부 단계가 이어지는 일련의 모범 사례가 포함되어 있습니다. 주제는 특정 순서로 읽을 필요가 없습니다:

* [GuardDuty 배포](#guardduty-배포)
    * [단일 계정 배포](#단일-계정-배포)
    * [다중 계정 배포](#다중-계정-배포)
        * [위임된 관리자 활성화](#위임된-관리자-활성화)
        * [조직에 대한 자동 활성화 기본 설정 구성](#조직에-대한-자동-활성화-기본-설정-구성)
        * [조직에 멤버로 계정 추가](#조직에-멤버로-계정-추가)
    * [GuardDuty 보호 플랜](#guardduty-보호-플랜)
        * [S3 멀웨어 보호 활성화](#s3-멀웨어-보호-활성화)
        * [런타임 모니터링](#런타임-모니터링)
            * [EC2용 런타임 모니터링 활성화](#ec2용-런타임-모니터링-배포)
            * [ECS용 런타임 모니터링 활성화](#ecs용-런타임-모니터링-배포)
            * [EKS용 런타임 모니터링 활성화](#eks용-런타임-모니터링-배포)
* [GuardDuty 탐지 결과 운영화](#guardduty-탐지-결과-운영화)
    * [탐지 결과 필터링](#탐지-결과-필터링)
    * [잠재적 노이즈 감소](#잠재적-노이즈-감소)
    * [높은 우선순위 탐지 결과에 대한 자동 알림](#높은-우선순위-탐지-결과에-대한-자동-알림)
    * [일반적인 탐지 결과 유형에 대한 자동 해결](#일반적인-탐지-결과-유형에-대한-자동-해결)
* [비용 최적화](#비용-최적화)
    * [CloudTrail 및/또는 S3 데이터 이벤트 사용량이 높은 경우](#cloudtrail-및또는-s3-데이터-이벤트-사용량이-높은-경우)
    * [VPC Flow Logs](#vpc-flow-logs)
    * [DNS Query Logs](#dns-query-logs)
    * [기타 고려 사항](#기타-고려-사항)
* [문제 해결](#문제-해결)
* [리소스](#리소스)

## Amazon GuardDuty란 무엇인가?

GuardDuty는 AWS 계정, Amazon Elastic Compute Cloud(Amazon EC2) 인스턴스, AWS Lambda 함수, Amazon Elastic Kubernetes Service(Amazon EKS) 클러스터, Amazon Elastic Container Service(Amazon ECS), Amazon Aurora 로그인 활동 및 Amazon Simple Storage Service(Amazon S3)에 저장된 데이터를 악의적인 활동에 대해 지속적으로 모니터링하는 지능형 위협 탐지 서비스입니다. 비정상적인 동작, 자격 증명 유출 또는 명령 및 제어 인프라(C2) 통신과 같은 잠재적인 악의적 활동이 탐지되면 GuardDuty는 보안 가시성 및 해결 지원에 사용할 수 있는 상세한 보안 탐지 결과를 생성합니다. 또한 Amazon GuardDuty Malware Protection 기능을 사용하면 Amazon EC2 인스턴스 및 컨테이너 워크로드에 연결된 Amazon Elastic Block Store(Amazon EBS) 볼륨에서 악성 파일을 탐지할 수 있습니다.

GuardDuty는 AWS CloudTrail 이벤트 로그, AWS CloudTrail 관리 이벤트, Amazon VPC Flow Logs 및 DNS 로그와 같은 [기본 데이터 소스](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_data-sources.html)를 모니터링합니다. 기본 데이터 소스의 활성화는 필요하지 않습니다. Amazon GuardDuty는 해당 서비스에서 직접 독립적인 데이터 스트림을 가져옵니다.

기본 데이터 소스 외에도 GuardDuty는 AWS 환경의 다른 AWS 서비스에서 추가 데이터를 사용하여 잠재적인 보안 위협을 모니터링하고 분석할 수 있습니다. 이러한 서비스에는 다음이 포함됩니다:

* [Amazon EKS](https://docs.aws.amazon.com/guardduty/latest/ug/kubernetes-protection.html) – GuardDuty는 EKS Audit Logs 및 GuardDuty 보안 에이전트를 통한 운영 체제 수준 이벤트를 모니터링합니다.
* [AWS Lambda](https://docs.aws.amazon.com/guardduty/latest/ug/lambda-protection.html) – GuardDuty는 의심스러운 네트워크 트래픽에 대한 VPC Flow Logs를 포함한 네트워크 활동 로그를 모니터링합니다.
* [Amazon EC2 및 Containers](https://docs.aws.amazon.com/guardduty/latest/ug/malware-protection.html) – GuardDuty는 연결된 Amazon Elastic Block Store(Amazon EBS) 볼륨에서 멀웨어 존재 지표(예: 비트코인 채굴 활동)에 대한 탐지 결과를 모니터링합니다.
* [Amazon Aurora](https://docs.aws.amazon.com/guardduty/latest/ug/rds-protection.html) – GuardDuty는 잠재적 위협에 대한 관계형 데이터베이스 서비스 로그인 활동을 모니터링하고 프로파일링합니다.
* [Amazon S3](https://docs.aws.amazon.com/guardduty/latest/ug/s3-protection.html) – GuardDuty는 AWS CloudTrail S3 데이터 이벤트를 모니터링하여 Amazon S3 리소스의 잠재적 위협을 식별합니다. AWS CloudTrail S3 관리 이벤트는 GuardDuty가 활성화된 후 기본적으로 모니터링됩니다.
* [Runtime Monitoring](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring.html) - Runtime Monitoring은 운영 체제 수준, 네트워킹 및 파일 이벤트를 관찰하고 분석하여 환경의 특정 AWS 워크로드에서 잠재적 위협을 탐지하는 데 도움을 줍니다.
* [S3 Malware Protection](https://docs.aws.amazon.com/guardduty/latest/ug/gdu-malware-protection-s3.html) - Malware Protection for S3는 선택한 Amazon Simple Storage Service(Amazon S3) 버킷에 새로 업로드된 객체를 스캔하여 멀웨어의 잠재적 존재를 탐지하는 데 도움을 줍니다. S3 객체 또는 기존 S3 객체의 새 버전이 선택한 버킷에 업로드되면 GuardDuty가 자동으로 멀웨어 스캔을 시작합니다. S3 멀웨어 보호는 전체 S3 자산에 배포하기 위한 것이 아니라는 점에 유의해야 합니다. S3 Malware protection은 신뢰할 수 없는 버킷에 업로드된 객체를 스캔하는 비용 효율적인 솔루션을 제공하도록 특별히 구축되었습니다. 예를 들어, 제3자가 문서나 파일을 보내고 애플리케이션에서 처리하기 전에 멀웨어가 없는지 확인해야 하는 시나리오에서 사용됩니다.

## GuardDuty 활성화의 이점은 무엇인가?

GuardDuty는 리소스와 완전히 독립적으로 작동하도록 설계되어 워크로드의 성능이나 가용성에 영향을 미치지 않습니다. 이 서비스는 통합된 위협 인텔리전스, 기계 학습(ML) 이상 탐지 및 멀웨어 스캔을 갖춘 완전 관리형 서비스입니다. GuardDuty는 기존 이벤트 관리 및 워크플로 시스템과 통합되도록 설계된 상세하고 실행 가능한 경고를 제공합니다. 초기 비용이 없으며 분석된 이벤트에 대해서만 비용을 지불하고 관리할 인프라나 위협 인텔리전스 피드 구독이 필요하지 않습니다.


## GuardDuty 배포

### 단일 계정 배포

독립 실행형 계정으로 GuardDuty를 활성화하거나 AWS Organizations와 함께 GuardDuty 관리자로 활성화하기 위한 최소 요구 사항은 아래에 설명되어 있습니다.

GuardDuty를 사용하는 첫 번째 단계는 계정에서 활성화하는 것입니다. 활성화되면 GuardDuty는 현재 리전에서 보안 위협에 대한 모니터링을 즉시 시작합니다. 독립 실행형 계정의 경우:

1. GuardDuty 콘솔을 엽니다: https://console.aws.amazon.com/guardduty/
2. Get Started를 선택합니다.
![GuardDuty landing page](../../images/GD-Landing-Page.png)
*그림 1: GuardDuty 랜딩 페이지*
3. Enable GuardDuty를 선택합니다.
![Enable GuardDuty](../../images/Enable-GD.png)
*그림 2: GuardDuty 활성화*

이 단계가 완료되면 GuardDuty는 로그 데이터 수집을 시작하고 악의적이거나 의심스러운 활동에 대해 환경을 모니터링합니다.

#### S3 Malware Protection 단일 계정 배포

전체 AWS 환경에서 GuardDuty를 사용하는 것을 권장하지만 S3 Malware protection은 단일 계정에서 나머지 GuardDuty와 독립적으로 활성화할 수 있습니다. 자세한 내용은 [Get started with Malware Protection for S3 only](https://docs.aws.amazon.com/guardduty/latest/ug/malware-protection-s3-get-started-independent.html) 문서를 참조하십시오.

![S3 Malware Protection Only](../../images/GD-S3-Malware-Only.png)
*그림 3: S3 Malware Protection만 사용*

### 다중 계정 배포

이 프로세스의 전제 조건으로, 관리하려는 모든 계정과 동일한 조직에 있어야 하며 조직 내에서 GuardDuty의 관리자를 위임하기 위해 AWS Organizations 관리 계정에 대한 액세스 권한이 있어야 합니다. 관리자를 위임하려면 추가 권한이 필요할 수 있습니다. 자세한 내용은 [Permissions required to designate a delegated administrator](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_organizations.html#organizations_permissions)를 참조하십시오.

#### 위임된 관리자 활성화

1. 관리 계정을 사용하여 AWS Organizations 콘솔을 엽니다: https://console.aws.amazon.com/organizations/
2. GuardDuty 콘솔을 엽니다: https://console.aws.amazon.com/guardduty/

GuardDuty가 이미 계정에서 활성화되어 있습니까?

* GuardDuty가 아직 활성화되지 않은 경우 Get Started를 선택한 다음 Welcome to GuardDuty 페이지에서 GuardDuty 위임된 관리자를 지정할 수 있습니다.
* GuardDuty가 활성화된 경우 Settings 페이지에서 GuardDuty 위임된 관리자를 지정할 수 있습니다.

3. 조직의 GuardDuty 위임된 관리자로 지정하려는 계정의 12자리 AWS 계정 ID를 입력하고 Delegate를 선택합니다.

![GuardDuty Auto Enable off](../../images/GD-Auto-Enable-Off.png)
*그림 4: GuardDuty 자동 활성화 꺼짐*

#### 조직에 대한 자동 활성화 기본 설정 구성

1. GuardDuty 콘솔을 엽니다: https://console.aws.amazon.com/guardduty/
2. 탐색 창에서 Accounts를 선택합니다.
3. Accounts 페이지는 GuardDuty 관리자에게 조직에 속한 멤버 계정을 대신하여 GuardDuty 및 선택적 보호 플랜을 자동 활성화하는 구성 옵션을 제공합니다. Add accounts by invitation 옆에서 이 옵션을 찾을 수 있습니다.

이 지원은 AWS 리전에서 GuardDuty 및 지원되는 모든 선택적 보호 플랜을 구성하는 데 사용할 수 있습니다. 멤버 계정을 대신하여 GuardDuty에 대해 다음 구성 옵션 중 하나를 선택할 수 있습니다:

* Enable for all accounts – 조직의 모든 멤버 계정에 대해 해당 옵션을 자동으로 활성화하려면 선택합니다. 여기에는 조직에 가입하는 새 계정과 조직에서 일시 중단되거나 제거되었을 수 있는 계정이 포함됩니다.
* Auto-enable for new accounts – 새 멤버 계정이 조직에 가입할 때 GuardDuty를 자동으로 활성화하려면 선택합니다.
* Do not enable – 조직의 모든 계정에 대해 해당 옵션을 활성화하지 않으려면 선택합니다. 이 경우 GuardDuty 관리자가 각 계정을 개별적으로 관리합니다.

![Manage Auto-enable preferences](../../images/GD-Auto-Enable-Preferences.png)

*그림 5: GuardDuty 자동 활성화 기본 설정*

4. Save Changes를 선택합니다.

#### 조직에 멤버로 계정 추가

이 절차는 AWS Organizations를 통해 GuardDuty 위임된 관리자 계정에 멤버 계정을 추가하는 방법을 다룹니다. GuardDuty에서 멤버를 연결하는 방법에 대한 자세한 내용은 [Managing multiple accounts in Amazon GuardDuty AWS service integrations with GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_accounts.html)를 참조하십시오.

1. 위임된 관리자 계정에 로그인합니다.
2. GuardDuty 콘솔을 엽니다: https://console.aws.amazon.com/guardduty/
3. 탐색 패널에서 Accounts를 선택합니다.
계정 테이블에는 Via Organizations 또는 By invitation을 통해 추가된 모든 계정이 표시됩니다.
4. 멤버로 추가하려는 하나 또는 여러 계정 ID를 선택합니다. 이러한 계정 ID는 Type이 Via Organizations여야 합니다.
5. Status 열의 드롭다운 화살표를 선택하여 Not a member 상태별로 계정을 정렬한 다음 현재 리전에서 GuardDuty가 활성화되지 않은 각 계정을 선택할 수 있습니다.
6. Confirm을 선택하여 계정을 멤버로 추가합니다. 이 작업은 선택한 모든 계정에 대해 GuardDuty도 활성화합니다. 계정의 Status가 Enabled로 변경됩니다.
7. (권장) 각 AWS 리전에서 이 단계를 반복합니다. 이렇게 하면 위임된 관리자가 GuardDuty를 활성화한 모든 리전에서 멤버 계정에 대한 탐지 결과 및 기타 구성을 관리할 수 있습니다.

### GuardDuty 보호 플랜

계정에서 GuardDuty를 활성화한 후 추가 보호 유형을 선택하는 것이 좋습니다. GuardDuty 보호 플랜은 Amazon EKS, Amazon S3, Amazon Aurora, Amazon EC2, Amazon ECS 및 AWS Lambda에 대한 집중적인 위협 탐지를 추가하는 [추가 기능](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty-features-activation-model.html)입니다. 각 GuardDuty 보호가 제공하는 이점에 대한 자세한 내용은 [Amazon GuardDuty User Guide](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)의 보호 섹션을 참조하십시오.

#### S3 Malware Protection

콘솔, CLI, API 또는 [CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-guardduty-malwareprotectionplan.html) 또는 [Terraform](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/guardduty_malware_protection_plan)과 같은 코드형 인프라를 통해 S3 Malware protection을 활성화할 수 있습니다. 전체 계정이나 계정들이 아닌 한 번에 하나의 버킷에 대해 특별히 활성화한다는 점을 명심하십시오. 정확한 단계는 [배포 문서](https://docs.aws.amazon.com/guardduty/latest/ug/enable-malware-protection-s3-bucket.html)를 참조하되, 배포가 어떻게 보이는지 이해할 수 있도록 아래에 이미지를 배치했습니다.

배포하기 전에 GuardDuty가 KMS 키 작업을 허용하는 기능을 포함하여 버킷의 S3 객체를 스캔할 수 있는 액세스 권한을 부여하는 역할이 존재하고 버킷이 이 역할에 대한 액세스를 명시적으로 거부하지 않는지 확인해야 합니다. 이 역할 생성 및 지원 암호화 메커니즘에 대한 자세한 내용은 [전제 조건 문서](https://docs.aws.amazon.com/guardduty/latest/ug/malware-protection-s3-iam-policy-prerequisite.html) 및 [할당량 문서](https://docs.aws.amazon.com/guardduty/latest/ug/malware-protection-s3-quotas-guardduty.html)를 참조하십시오.

배포할 때 GuardDuty가 멀웨어가 발견되었는지 여부에 따라 객체에 태그를 지정할지 여부를 결정해야 합니다(NO_THREATS_FOUND, THREATS_FOUND, UNSUPPORTED, ACCESS_DENIED 또는 FAILED). 이 태그 지정을 통해 태그에 따라 자동화된 워크플로를 생성할 수 있습니다. [S3 Malware 자동화 워크플로](#s3-malware-protection-automation-workflow) 섹션에서 이에 대한 예를 제공합니다.

![S3 Malware Protection console](../../images/GD-S3-Malware-Console.png)
*그림 7: S3 Malware Protection 콘솔*

![S3 Malware Protection configuration](../../images/GD-S3-Malware-Configuration.png)
*그림 8: S3 Malware Protection 구성*


#### Runtime Monitoring

EKS, EC2 및 ECS용 GuardDuty 런타임 모니터링 보호를 포함한 GuardDuty 런타임 모니터링의 경우 적용 범위를 원하는 리소스와 런타임 에이전트를 배포하는 방법을 추가로 범위 지정할 수 있습니다. GuardDuty가 런타임 모니터링을 배포하도록 허용하는 것이 좋습니다. 이렇게 하면 각각 에이전트, 사이드카 또는 EKS 관리형 애드온을 사용하여 EC2, ECS 및 EKS용 VPC 엔드포인트 및 에이전트가 배포됩니다. 이렇게 하면 현재 리소스에 대한 적용 범위가 보장되지만 향후 생성되는 새 리소스에도 적용됩니다. 이 에이전트는 [EBPF 기술](https://ebpf.io/what-is-ebpf/)을 기반으로 구축되었습니다. GuardDuty 자동화를 사용하면 조직 전체에서 새 리소스 적용 범위를 처리하는 데 필요한 수동 작업이 절약됩니다. 그러나 이 구성을 직접 관리하도록 선택할 수 있습니다.

GuardDuty가 환경에서 런타임 모니터링을 제공하는 데 필요한 현재 및 향후 VPC 엔드포인트와 에이전트를 모두 포함하는 데 필요한 리소스를 배포하도록 허용하기로 선택한 경우 포함 또는 제외 태그를 사용하여 런타임 모니터링이 적용되는 리소스를 추가로 범위 지정할 수 있습니다. 예를 들어, 모니터링하지 않으려는 EKS 클러스터가 몇 개 있는 경우 포함 태그를 사용하려는 몇 개의 리소스에만 모니터링을 적용하려는 반대 시나리오보다 제외 태그를 사용하는 것이 더 효율적입니다. GuardDuty 문서는 이 기능을 구현하는 데 필요한 [사용 사례 예제 및 특정 태그](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring-configuration.html)를 제공합니다.

GuardDuty 런타임 모니터링을 구성할 때 런타임 모니터링에 대한 [전제 조건](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring-prerequisites.html)을 이해하는 것이 중요합니다. 이를 통해 지원되는 OS, 커널 버전, GuardDuty 에이전트의 CPU 및 메모리 제한 등에 대한 정보를 얻을 수 있습니다. 배포 후 런타임 모니터링 배포의 [적용 범위를 평가](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring-assessing-coverage.html)하여 문제를 해결하십시오.

#### EC2용 Runtime Monitoring 배포
이 배포 가이드는 다음을 가정합니다:

* EC2 인스턴스가 지원되는 Amazon Machine Images(AMI), 인스턴스 유형 및 운영 체제에서 실행되고 있습니다.
* AWS Organization에 대해 기본 GuardDuty 적용 범위가 이미 활성화되어 있습니다.

전제 조건에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/guardduty/latest/ug/prereq-runtime-monitoring-ec2-support.html)를 참조하십시오.

이 가이드에 설명된 단계를 따르면 GuardDuty Runtime Monitoring을 활용하여 EC2 인스턴스의 보안 태세를 강화하고 AWS 환경 내에서 실시간 위협 탐지 및 대응 기능을 활성화하는 방법을 배울 수 있습니다.

##### AWS Systems Manager(SSM) Default Host Management 구성

GuardDuty가 런타임 이벤트를 모니터링하려는 Amazon EC2 인스턴스는 AWS Systems Manager(SSM) 관리형이어야 합니다. AWS Systems Manager의 기능인 Quick Setup을 사용하면 AWS Organizations의 조직에 추가된 모든 계정 및 리전에 대해 Default Host Management Configuration을 활성화할 수 있습니다. 이렇게 하면 조직의 모든 Amazon Elastic Compute Cloud(EC2) 인스턴스에서 SSM Agent가 최신 상태로 유지되고 Systems Manager에 연결할 수 있습니다.

1. Organization 관리 계정에서 [AWS Systems Manager 콘솔](https://console.aws.amazon.com/systems-manager/)을 방문합니다.
2. 왼쪽 탐색 창에서 **Quick Setup**을 선택합니다.
3. "Default Host Management Configuration"에서 "Create"를 선택합니다.
4. 옵션을 기본값으로 두고 "Create"를 선택합니다.

---
**Default Host Management를 구성할 때 EC2 인스턴스에서 AWS Systems Manager(SSM) Agent가 이미 실행 중이라고 가정하면 에이전트가 Systems Manager에 연결되고 EC2 인스턴스가 Fleet Manager에 관리형 인스턴스로 표시될 때까지 최대 30분을 기다려야 할 수 있습니다. 그 시점에서 Systems Manager는 인스턴스에 Amazon GuardDuty Runtime Monitoring SSM Plugin을 설치합니다.**

---


##### SSM Host Management 검증
Default Host Management를 활성화한 후 호스트가 SSM에 표시되는 데 일반적으로 30분이 걸립니다.

5. [AWS Systems Manager 콘솔](https://console.aws.amazon.com/systems-manager/)을 방문하여 왼쪽 탐색 창의 **Node Management** 섹션에서 **Fleet Manager**를 선택합니다.
6. Fleet Manager에서 EC2 인스턴스를 볼 수 있는지 확인합니다. 또는 [AWS Resource Explorer](https://docs.aws.amazon.com/systems-manager/latest/userguide/Resource-explorer-quick-setup.html)를 사용하여 AWS 계정 또는 전체 AWS 조직에서 리소스를 검색하고 발견할 수 있습니다.

---
**Default Host Management를 활성화한 후 30분이 지나도 Fleet Manager에 EC2 인스턴스가 표시되지 않으면 이 페이지 하단의 SSM Agent 문제 해결 섹션을 참조하십시오.**

---

##### EC2용 GuardDuty Runtime Monitoring 활성화

7. GuardDuty 위임된 관리자 계정에서 [Amazon GuardDuty 콘솔](https://console.aws.amazon.com/guardduty/)을 방문합니다.
8. 왼쪽 탐색 창의 **Protection plans** 섹션에서 **Runtime Monitoring**을 선택합니다.
9. **Runtime Monitoring configuration**에서 **Enable**을 클릭합니다. 메시지가 표시되면 **Confirm**을 클릭합니다.
10. **Automated agent configuration**에서 모든 계정에 대해 **Enable**을 클릭합니다(권장).
11. **Runtime coverage** 탭으로 전환한 다음 **EC2 instance runtime coverage** 탭을 엽니다. 5분 이내에 EC2 인스턴스가 "Healthy" 상태를 표시하기 시작합니다. 이미 전제 조건 구성을 충족하는 EC2 인스턴스에서 런타임 모니터링이 "Healthy"가 되는 데 최대 10분이 걸릴 수 있습니다.
12. AWS Organization에 대해 활성화된 모든 리전에 대해 3-5단계를 반복합니다.



#### ECS용 Runtime Monitoring 배포
이 배포 가이드는 다음을 가정합니다:

* AWS Organization에 대해 기본 GuardDuty 적용 범위가 이미 활성화되어 있습니다.
* 위임된 GuardDuty 관리자 계정에 대한 액세스 권한이 있습니다.
* 지원되는 버전(최소 1.4.0 또는 LATEST)에서 실행되는 Fargate 플랫폼.
* EC2 컨테이너 인스턴스가 지원되는 ECS-AMI, 인스턴스 유형 및 운영 체제에서 실행되고 있습니다.

Fargate(ECS만 해당) 클러스터의 전제 조건에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/guardduty/latest/ug/prereq-runtime-monitoring-ecs-support.html)를 참조하십시오.

Amazon EC2에서 실행되는 Amazon ECS의 전제 조건에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-guard-duty-configure-manual-guard-duty.html)를 참조하십시오.

이 가이드에 설명된 단계를 따르면 GuardDuty Runtime Monitoring을 활용하여 ECS 클러스터의 보안 태세를 강화하고 AWS 환경 내에서 실시간 위협 탐지 및 대응 기능을 활성화하는 방법을 배울 수 있습니다. Runtime Monitoring이 Fargate(Amazon ECS만 해당)와 작동하는 방식에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/guardduty/latest/ug/how-runtime-monitoring-works-ecs-fargate.html)를 참조하십시오.

1. 위임된 GuardDuty 관리자 계정에서 Amazon GuardDuty의 [**Runtime Monitoring**](https://us-east-1.console.aws.amazon.com/guardduty/home?region=us-east-1#/runtime-monitoring) 페이지를 엽니다.
2. **Runtime Monitoring configuration**에서 **Enable for all accounts**를 클릭합니다. 메시지가 표시되면 **Save**를 클릭합니다. 이렇게 하면 모든 계정에서 GuardDuty ECS 모니터링이 켜집니다.
3. **Automated agent configuration**에서 **AWS Fargate(ECS only)**에 대해 **Enable for all accounts**를 클릭합니다. GuardDuty는 사용자를 대신하여 AWS Fargate의 ECS 클러스터에 에이전트를 배포하고 관리합니다.
4. **Runtime coverage** 탭으로 전환한 다음 **ECS clusters runtime coverage** 탭을 엽니다.

서비스의 일부인 작업을 GuardDuty가 모니터링하도록 하려면 Runtime Monitoring을 활성화한 후 새 서비스 배포가 필요합니다. 특정 ECS 서비스에 대한 마지막 배포가 Runtime Monitoring을 활성화하기 전에 시작된 경우 서비스를 다시 시작하거나 `forceNewDeployment`를 사용하여 서비스를 업데이트할 수 있습니다.

서비스를 업데이트하는 단계는 다음 리소스를 참조하십시오:
* *Amazon Elastic Container Service Developer Guide*의 [Updating an Amazon ECS service using the console](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/update-service-console-v2.html)
* *Amazon Elastic Container Service API Reference*의 [UpdateService](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_UpdateService.html)
* *AWS CLI Command Reference*의 [update-service](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/update-service.html)

#### EKS용 Runtime Monitoring 배포
이 배포 가이드는 다음을 가정합니다:

* Amazon EKS 클러스터가 Amazon EC2 인스턴스에서 실행되고 있습니다. GuardDuty는 AWS Fargate에서 실행되는 Amazon EKS 클러스터를 지원하지 않습니다.
* Amazon EKS 클러스터가 지원되는 운영 체제 및 Kubernetes 버전에서 실행되고 있어야 합니다.
* AWS Organization에 대해 기본 GuardDuty 적용 범위가 이미 활성화되어 있습니다.
* 위임된 GuardDuty 관리자 계정에 대한 액세스 권한이 있습니다.

전제 조건에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/guardduty/latest/ug/prereq-runtime-monitoring-ec2-support.html)를 참조하십시오.

이 가이드에 설명된 단계를 따르면 GuardDuty Runtime Monitoring을 활용하여 EKS 클러스터의 보안 태세를 강화하고 AWS 환경 내에서 실시간 위협 탐지 및 대응 기능을 활성화하는 방법을 배울 수 있습니다. Runtime Monitoring이 Amazon EKS 클러스터와 작동하는 방식에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/guardduty/latest/ug/how-runtime-monitoring-works-eks.html#eksrunmon-approach-to-monitor-eks-clusters)를 참조하십시오.

다중 계정 환경에서는 위임된 GuardDuty 관리자 계정만 멤버 계정에 대한 Automated agent configuration을 활성화 또는 비활성화하고 조직의 멤버 계정에 속한 EKS 클러스터에 대한 Automated agent를 관리할 수 있습니다. GuardDuty 멤버 계정은 자신의 계정에서 이 구성을 수정할 수 없습니다. 위임된 GuardDuty 관리자 계정은 AWS Organizations를 사용하여 멤버 계정을 관리합니다. 다중 계정 환경에 대한 자세한 내용은 [Managing multiple accounts](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_accounts.html)를 참조하십시오.

1. Amazon GuardDuty에서 [**Runtime Monitoring**](https://us-east-1.console.aws.amazon.com/guardduty/home?region=us-east-1#/runtime-monitoring) 페이지를 엽니다.
2. **Runtime Monitoring configuration**에서 **Enable for all accounts**를 클릭합니다. 메시지가 표시되면 **Save**를 클릭합니다.
3. **Automated agent configuration**에서 **Amazon EKS**에 대해 **Enable for all accounts**를 클릭합니다. GuardDuty는 사용자를 대신하여 AWS Fargate의 EKS 클러스터에 에이전트를 배포하고 관리합니다.
4. **Runtime coverage** 탭으로 전환한 다음 **EKS clusters runtime coverage** 탭을 엽니다.



## GuardDuty 탐지 결과 운영화

계정에서 GuardDuty를 활성화한 후 GuardDuty는 [기본 데이터 소스](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_data-sources.html) 모니터링을 시작하고 선택적으로 활성화된 리소스 보호 유형과 관련된 기능을 분석합니다. GuardDuty를 활성화한 후 권장되는 모범 사례는 30일 평가판(계정당 활성화됨)을 활용하여 계정 및 리소스에 대해 기계 학습으로 도출된 정상 활동의 기준선을 이해하는 것입니다. 평가판 기간 동안 GuardDuty는 위협 기반 및 규칙 기반 인텔리전스를 사용하여 거의 실시간으로 [탐지 결과](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings.html)를 생성합니다.

잠재적인 보안 문제는 GuardDuty 콘솔에서 탐지 결과로 표시됩니다. 모든 GuardDuty 탐지 결과에는 잠재적 위험에 따라 [심각도 수준](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_findings.html#guardduty_findings-severity)(낮음, 중간, 높음) 및 해당 값(1.0 – 8.9)이 할당됩니다. 값이 높은 탐지 결과는 더 큰 보안 위험을 나타냅니다. 탐지 결과에 할당된 심각도 수준은 탐지 결과에서 강조 표시된 잠재적 보안 문제에 대한 대응을 결정하는 데 도움이 됩니다.
![GuardDuty Severity levels](../../images/GD-Severity-Levels.png)
*그림 9: GuardDuty 심각도 수준*

팀이 [GuardDuty Finding Types](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html)에 익숙해지는 것이 좋습니다. 이를 통해 계정에서 볼 수 있는 탐지 결과, 이러한 탐지 결과와 관련된 세부 정보 및 잠재적인 [해결 조치](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_remediate.html)에 대한 통찰력을 얻을 수 있습니다. 다양한 GuardDuty 탐지 결과를 이해한 후 고객이 필터링, 알림 및 잠재적 자동 해결을 고려하도록 권장합니다. 이는 다음 섹션에서 다룰 것입니다. 또한 환경에서 GuardDuty 탐지 결과에 대응하기 위한 [인시던트 대응 플레이북](https://github.com/aws-samples/aws-incident-response-playbooks) 작성을 시작하는 것이 좋습니다. 이는 아래의 여러 단계를 결합하고 환경 및 팀에 특정한 다루지 않은 세부 정보도 포함할 것입니다.

## Amazon GuardDuty Extended Threat Detection

GuardDuty Extended Threat Detection은 AWS 계정 내에서 데이터 소스, 여러 유형의 AWS 리소스 및 시간에 걸쳐 있는 다단계 공격을 자동으로 탐지합니다. 이 기능을 통해 GuardDuty는 다양한 유형의 데이터 소스를 모니터링하여 관찰하는 여러 이벤트의 시퀀스에 초점을 맞춥니다. Extended Threat Detection은 이러한 이벤트를 상관 관계하여 AWS 환경에 대한 잠재적 위협으로 나타나는 시나리오를 식별한 다음 공격 시퀀스 탐지 결과를 생성합니다.


단일 탐지 결과가 전체 공격 시퀀스를 포함할 수 있습니다. 예를 들어 다음과 같은 시나리오를 탐지할 수 있습니다:

1. 위협 행위자가 컴퓨팅 워크로드에 대한 무단 액세스를 얻습니다.
2. 그런 다음 행위자는 권한 상승 및 지속성 확립과 같은 일련의 작업을 수행합니다.
3. 마지막으로 행위자는 Amazon S3 리소스에서 데이터를 유출합니다.

Extended Threat Detection은 AWS 자격 증명 오용과 관련된 손상 및 AWS 계정의 데이터 손상 시도와 관련된 위협 시나리오를 다룹니다. 이러한 위협 시나리오의 특성상 GuardDuty는 모든 공격 시퀀스 탐지 결과 유형을 Critical로 간주합니다.

몇 가지 고려 사항

자동 활성화:

1. ETD는 모든 GuardDuty 계정에 대해 자동으로 켜집니다.
2. ETD에 대한 수동 활성화가 필요하지 않습니다.
3. ETD는 추가 비용이 없습니다.

S3 Protection:

1. ETD와 별도의 기능으로 유지됩니다.
2. 원하는 경우 활성화해야 합니다.
3. ETD와 함께 자동으로 활성화되지 않습니다.

기능:

1. ETD는 GuardDuty의 위협 탐지 기능을 자동으로 향상시킵니다.
2. 선택적 S3 Protection을 포함한 다른 GuardDuty 기능과 함께 작동합니다.

GuardDuty 콘솔의 Protection plans에서 Extended Threat Detection 페이지를 방문하십시오.
Extended Threat Detection이 활성화되어 있는지 확인하십시오.

### 탐지 결과 필터링


GuardDuty를 처음 사용하는 경우 탐지 결과를 보거나 필터링하는 가장 쉬운 방법은 Summary 탭을 사용하는 것입니다. Summary 대시보드는 현재 AWS 리전에서 생성된 마지막 10,000개의 GuardDuty 탐지 결과에 대한 집계된 보기를 표시합니다. Last 2 days(기본값), Last 7 days 또는 Last 30 days를 선택할 수 있습니다. 이 대시보드는 6개의 위젯을 제공하며 그 중 3개는 보기를 사용자 지정할 수 있는 필터 기능을 포함합니다.
![GuardDuty Summary page](../../images/GD-Summary-page.png)
*그림 10: GuardDuty 요약 페이지*

위임된 관리자가 조직의 GuardDuty 탐지 결과에 더 익숙해지면 Findings 탭을 사용하여 고급 필터링 기술을 사용할 수 있습니다. Findings 탭은 지정한 기준과 일치하거나 일치하지 않는 탐지 결과를 필터링할 수 있는 [80개 이상의 탐지 결과 속성](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_filter-findings.html#filter_criteria)을 노출합니다. 일반적인 필터 기술은 1. 리소스가 손상되었다는 높은 표시가 있는 위협(예: 높은 심각도 탐지 결과 보기) 2. 탐지 결과 유형이 원치 않는 청구 요금에 대한 잠재적 위험을 나타내는 경우(예: CryptoMining)에 초점을 맞춘 것입니다.

비트코인 채굴을 표면화하는 것은 생성할 수 있는 탐지 결과 필터의 한 예입니다. 비트코인은 다른 통화, 제품 및 서비스와 교환할 수 있는 전 세계 암호화폐 및 디지털 결제 시스템입니다. 비트코인은 비트코인 채굴에 대한 보상이며 위협 행위자가 매우 선호합니다. EC2 인스턴스 중 비트코인 채굴 목적으로 손상된 인스턴스가 있는지 확인하려면 이 속성 및 값 쌍을 사용할 수 있습니다: Severity:High, Finding type:CryptoCurrency:EC2/BitcoinTool.B!DNS. 이 필터를 적용하면 비트코인 또는 기타 암호화폐 관련 활동과 관련된 도메인 이름을 쿼리하는 EC2 인스턴스의 보기가 제공됩니다.
![GuardDuty Finding page](../../images/GD-Findings-Page.png)
*그림 11: GuardDuty 탐지 결과 페이지*

참고: 자주 사용하는 필터를 저장하여 향후 노력을 줄일 수 있습니다. 지정된 속성과 해당 값(필터 기준)을 필터로 저장하려면 Save를 선택합니다. 필터 이름과 설명을 입력한 다음 Done을 선택합니다.

### 잠재적 노이즈 감소

AWS Organization 내에서 계정 및 워크로드가 증가함에 따라 특정 구성으로 인해 GuardDuty 탐지 결과가 증가할 수 있습니다. 일부 탐지 결과는 가치가 낮거나 조치를 취하지 않으려는 위협으로 간주될 수 있습니다. 환경에 가장 영향을 미치는 보안 위협을 더 쉽게 인식하여 신속한 해결 조치를 가능하게 하려면 억제 규칙을 배포하는 것이 좋습니다. [억제 규칙](https://docs.aws.amazon.com/guardduty/latest/ug/findings_suppression-rule.html)은 지정된 기준과 일치하는 새 탐지 결과를 자동으로 보관하는 데 사용되는 값과 쌍을 이루는 필터 속성으로 구성된 기준 집합입니다.

억제 규칙을 생성한 후 억제 규칙이 적용되는 한 규칙에 정의된 기준과 일치하는 새 탐지 결과가 자동으로 보관됩니다. 기존 필터를 사용하여 억제 규칙을 생성할 수 있습니다. 억제 규칙은 전체 탐지 결과 유형을 억제하거나 특정 탐지 결과 유형의 특정 인스턴스만 억제하도록 보다 세분화된 필터 기준을 정의하도록 구성할 수 있습니다. 억제 규칙은 언제든지 편집할 수 있습니다. 가능한 한 세분화하여 억제하는 것이 좋습니다. 이렇게 하면 탐지 결과 유형의 전체 적용 범위를 잃지 않고 특정 상황에 대해서만 탐지 결과를 줄이는 데 도움이 됩니다.

억제된 탐지 결과는 AWS Security Hub, Amazon S3, Amazon Detective 또는 Amazon CloudWatch로 전송되지 않으므로 Security Hub, 타사 SIEM 또는 기타 경고 및 티켓팅 애플리케이션을 통해 GuardDuty 탐지 결과를 사용하는 경우 탐지 결과 노이즈 수준이 감소합니다.

GuardDuty는 억제 규칙과 일치하더라도 계속 탐지 결과를 생성하지만 해당 탐지 결과는 자동으로 보관됨으로 표시됩니다. 보관된 탐지 결과는 GuardDuty에 90일 동안 저장되며 해당 기간 동안 언제든지 볼 수 있습니다. GuardDuty 콘솔에서 탐지 결과 테이블에서 Archived를 선택하거나 service.archived가 true인 findingCriteria 기준과 함께 ListFindings API를 사용하여 GuardDuty API를 통해 억제된 탐지 결과를 볼 수 있습니다.

억제 규칙의 일반적인 사용 사례는 EC2 인스턴스를 스캔하는 알려진 리소스를 필터링하는 것입니다. 이 리소스는 환경의 특정 리소스에 대해 의도된 것일 수 있는 탐지 결과 유형 Recon:EC2/Portscan을 초래할 수 있습니다. 이 시나리오의 경우 리소스 태그 지정과 Finding Type의 조합을 사용하여 관련 탐지 결과를 억제하는 것이 좋습니다.

### 높은 우선순위 탐지 결과에 대한 자동 알림

환경에 가장 영향을 미치는 탐지 결과만 표면화하도록 억제 규칙 필터를 생성한 후 권장되는 다음 조치는 높은 우선순위 탐지 결과의 알림을 자동화하는 것입니다. Amazon EventBridge는 AWS 서비스에서 생성된 이벤트를 사용하여 대규모로 이벤트 기반 애플리케이션을 더 쉽게 구축할 수 있게 해주는 서버리스 이벤트 버스입니다. GuardDuty는 새로 생성된 탐지 결과에 대해 Amazon EventBridge용 이벤트를 생성합니다. 모든 탐지 결과는 동적입니다. 즉, GuardDuty가 동일한 보안 문제와 관련된 새로운 활동을 탐지하면 새로 집계된 탐지 결과로 새 정보로 원래 탐지 결과를 업데이트합니다. 따라서 EventBridge가 잠재적으로 처리할 새 이벤트를 생성합니다. 위임된 관리자는 선택적으로 GuardDuty Settings의 Findings export options에서 EventBridge에 새 탐지 결과를 게시하는 빈도를 변경할 수 있습니다. 기본적으로 EventBridge는 6시간마다 업데이트되지만 1시간 또는 15분 간격으로 변경할 수 있습니다.

모든 GuardDuty 탐지 결과에는 탐지 결과 ID가 할당됩니다. GuardDuty는 고유한 탐지 결과 ID가 있는 모든 탐지 결과에 대해 EventBridge 이벤트를 생성합니다. Amazon EventBridge를 GuardDuty와 함께 사용하면 GuardDuty 탐지 결과와 관련된 보안 문제에 대응하는 데 도움이 되는 작업을 자동화할 수 있습니다. 그러한 작업 중 하나는 선호하는 알림 채널로 알림을 보내는 것입니다. 일반적으로 사용되는 알림 채널은 이메일 및 Webhook을 통한 수신 메시지를 지원하는 엔터프라이즈 메시징 애플리케이션입니다.

또 다른 일반적인 시나리오는 추적 및 이벤트 상관 관계를 위해 GuardDuty 탐지 결과를 티켓팅 시스템 또는 SIEM 솔루션으로 보내는 것입니다. [티켓팅 시스템 또는 SIEM 솔루션으로 이러한 탐지 결과를 보내기](https://docs.aws.amazon.com/securityhub/latest/partnerguide/prepare-receive-findings.html) 전에 Security Hub를 AWS의 보안 탐지 결과에 대한 중앙 집계 지점으로 활용하는 것이 좋습니다. EventBridge를 활용하여 GuardDuty 탐지 결과를 티켓팅 시스템 또는 SIEM 솔루션으로 직접 보낼 수도 있습니다. EventBridge를 통해 정보를 보내는 방법에 대한 지침은 솔루션 문서를 참조하십시오.

선호하는 알림 채널로 이메일을 활용하는 경우 GuardDuty는 EventBridge를 통해 Amazon Simple Notification Services(Amazon SNS)와 통합됩니다. Amazon Simple Notification Service(Amazon SNS)는 애플리케이션 간(A2A) 및 애플리케이션 대 사람(A2P) 통신을 위한 완전 관리형 메시징 서비스입니다. 알림 워크플로는 아래 이미지에 표시된 대로입니다.
![GuardDuty Notification workflow](../../images/GD-Notification-Workflow.png)
*그림 12: GuardDuty 알림 워크플로*

억제 규칙과 일치하지 않는 모든 GuardDuty 탐지 결과는 자동으로 계정 이벤트 버스로 전달됩니다. 억제되지 않은 탐지 결과에 대해 Amazon SNS를 통한 이메일 알림을 배포하는 데 필요한 구성은 두 가지뿐입니다:

1. SNS 주제 생성 및 구독
2. EventBridge 규칙 생성

SNS는 이 워크플로에서 EventBridge 작업의 대상 서비스입니다. SNS는 두 단계만 필요합니다: 1. 새 Topic 생성(유형을 Standard로 선택) 및 Topic Name 제공, 2. Email을 프로토콜로 사용하여 Subscription 생성. Endpoint에 이메일 주소를 입력하고 Create subscription을 클릭하면 콘솔에서 필요한 단계가 완료됩니다. 마지막 단계는 엔드포인트에 사용된 이메일 주소로 전송된 이메일 내에서 Confirm subscription을 클릭하는 것입니다.
![GuardDuty SNS email](../../images/GD-SNS-Email.png)
*그림 13: GuardDuty SNS 이메일*

![SNS Subscription Confirmed](../../images/GD-Subscription-Confirmed.png)

*그림 14: SNS 구독 확인*

EventBridge의 기본은 [이벤트](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-events.html)를 [대상](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html)으로 라우팅하는 [규칙](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html)을 생성하는 것입니다. 아래 예에서는 이벤트 패턴이 있는 규칙이 구성됩니다. 이렇게 하면 이벤트가 정의된 패턴과 일치할 때 규칙이 실행됩니다. 이 규칙은 위협 행위자로 활용되는 리소스라는 추가 세부 정보와 함께 SSH 무차별 대입 공격 동작을 나타내는 EC2 인스턴스를 찾습니다.

SNS 주제/구독이 모두 준비되면 이 EventBridge 규칙을 배포하면 등록된 이메일 주소로 이메일 알림이 자동으로 전송됩니다. 높은 심각도 탐지 결과의 알림을 보낼 때 팀과 연결된 이메일 별칭을 사용하는 것이 좋습니다.
![GuardDuty EventBridge rule](../../images/GD-EventBridge-Rule.png)
*그림 15: GuardDuty EventBridge 규칙*

팁: 사용자 지정과 함께 이메일 알림을 위해 SNS를 대상으로 하는 GuardDuty 탐지 결과에 대한 EventBridge 규칙을 구축하는 방법에 대한 자세한 지침은 이 [AWS Knowledge Center 비디오](https://www.youtube.com/watch?v=HSon9kZ0mCw)를 참조하십시오.


### 일반적인 탐지 결과 유형에 대한 자동 해결

*Stealth:S3/ServerAccessLoggingDisabled*
GuardDuty 탐지 결과 유형을 사용한 자동화의 일반적인 사용 사례는 잘못된 구성(의도적 또는 비의도적)의 결과로 S3 및 EC2 관련 탐지 결과를 처리하는 것입니다. 예를 들어, 탐지 결과 유형 Stealth:S3/ServerAccessLoggingDisabled는 AWS 환경 내의 버킷에 대해 S3 서버 액세스 로깅이 비활성화되어 있음을 알려줍니다. 비활성화된 경우 식별된 S3 버킷에 액세스하려는 시도에 대한 웹 요청 로그가 생성되지 않지만 [DeleteBucket](https://docs.aws.amazon.com/AmazonS3/latest/API/API_DeleteBucket.html)과 같은 버킷에 대한 S3 관리 API 호출은 여전히 추적됩니다. 이 버킷에 대해 CloudTrail을 통해 S3 데이터 이벤트 로깅이 활성화된 경우 버킷 내 객체에 대한 웹 요청은 여전히 추적됩니다. 이는 잘못된 구성의 결과이거나 탐지를 회피하기 위한 무단 사용자의 기술의 일부일 수 있습니다.

관련 탐지 결과 세부 정보를 사용하여 관리자는 의심스러운 활동에 관련된 AWS 리소스, 이 활동이 발생한 시기 및 추가 정보에 대해 알 수 있습니다. 정보를 수집한 후 관리자는 탐지 결과에서 S3 버킷으로 전환하여 Server Access Logging을 다시 활성화할 수 있습니다. 이를 자동으로 해결하면 관리자의 시간이 절약되고 잠재적 위협 행위자가 API 호출을 수행할 수 있었던 방법에 대한 집중적인 조사가 가능합니다.

일반적인 해결 접근 방식은 AWS Config Managed Rule을 사용하여 [s3-bucket-logging-enabled](https://docs.aws.amazon.com/config/latest/developerguide/s3-bucket-logging-enabled.html) 구성 변경 시 트리거하여 GuardDuty 탐지를 자동화 작업에서 분리합니다. 트리거되면 AWS Config는 즉시 S3 버킷을 평가하고 로깅이 비활성화된 경우 NON_COMPLIANT 상태를 표시합니다. 해결 작업을 AWS Config 규칙과 연결하고 수동 개입 없이 비준수 리소스를 처리하기 위해 자동으로 실행하도록 선택할 수 있습니다. 이 시나리오에서 비준수 s3-bucket-logging-enabled 상태는 [AWS-ConfigureS3BucketLogging](https://docs.aws.amazon.com/systems-manager-automation-runbooks/latest/userguide/automation-aws-configures3bucketlogging.html)이라는 [Systems Manager Automation](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html) 문서를 트리거합니다. AWS Config 규칙 Action을 구성할 때 Automatic remediation 및 AWS-ConfigureS3BucketLogging을 Remediation action detail로 선택하면 S3 버킷이 Config 규칙을 준수하지 않을 때마다 S3 서버 액세스 로깅이 다시 활성화됩니다.

![GuardDuty Finding automation workflow](../../images/GD-Finding-Automation.png)

*그림 16: GuardDuty 탐지 결과 자동화*

*Backdoor:EC2/C&CActivity.B*
알려진 명령 및 제어 서버와 연결된 IP 주소를 쿼리하는 EC2 인스턴스는 자동 해결의 또 다른 일반적인 사용 사례입니다. 이 탐지 결과는 AWS 환경 내의 나열된 인스턴스가 알려진 명령 및 제어(C&C) 서버와 연결된 IP를 쿼리하고 있음을 알려줍니다. 나열된 인스턴스가 손상되었을 수 있습니다. 이 유형의 탐지 결과에는 인스턴스를 종료하기 전에 볼륨의 스냅샷을 수행하는 것과 같은 작업을 포함할 수 있는 해결을 위한 여러 단계가 있습니다. 이 스냅샷은 포렌식 팀에 정보를 제공할 수 있습니다. 그러나 이 탐지 결과를 해결하는 첫 번째 단계는 인스턴스를 격리하는 것입니다.

Automated Notifications 섹션에 설명된 것과 동일한 이벤트 워크플로를 활용하여 인스턴스의 보안 그룹을 변경하여 인터넷에 대한 인바운드/아웃바운드 연결을 중지할 수 있습니다. [AWS Lambda 함수](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)는 Backdoor:EC2/C&CActivity.B와 일치하도록 설정된 EventBridge 규칙의 대상이 됩니다. (EventBridge는 여러 대상을 지원할 수 있습니다. 이 시나리오의 모범 사례에는 SNS 주제 대상도 포함됩니다)
다음은 이 작업을 수행하기 위해 Python으로 작성된 Lambda 함수의 예입니다:

```Python
import boto3

ec2_client = boto3.client('ec2')

def lambda_handler(event, context):
    try:
        # Check if the event is triggered by GuardDuty finding
        if 'detail' in event and 'type' in event['detail'] and event['detail']['type'] == 'Backdoor:EC2/C&CActivity.B':
            # Extract relevant information from the GuardDuty finding
            instanceId = event['detail']['resource']['instanceDetails']['instanceId']
            SecurityGroupId = "sg-XXXXXXXXXXXX"  # Replace with the desired Security Group ID

            # Construct the modification command
            command = {
                'InstanceId': instanceId,
                'Groups': [SecurityGroupId]
            }

            # Send the command to modify the EC2 instance's security group
            ec2_client.modify_instance_attribute(**command)

            return 'Updated security group for instance: {}'.format(instanceId)
        else:
            return 'No action taken. Event is not a GuardDuty finding of type Backdoor:EC2/C&CActivity.B'
    except Exception as e:
        print('Error:', str(e))
        raise e
```

### S3 Malware Protection 자동화 워크플로

GuardDuty에 멀웨어 스캔 결과에 따라 NO_THREATS_FOUND, THREATS_FOUND, UNSUPPORTED, ACCESS_DENIED 또는 FAILED와 같은 태그를 추가할 수 있는 기능을 제공하면 자동화된 워크플로를 생성할 수 있습니다. 예를 들어, 신뢰할 수 없는 제3자가 애플리케이션으로 가져오기 전에 S3에 문서를 업로드하도록 허용하는 경우 GuardDuty S3 멀웨어 보호를 사용하여 이러한 문서를 스캔한 다음 부여된 태그에 따라 애플리케이션 버킷 또는 격리 버킷으로 이동할 수 있습니다. 입문서로 GuardDuty S3 Malware Protection이 파일을 처리하고 다른 AWS 서비스와 통합하는 방법을 보여주는 아래 그림을 포함했습니다.

![S3 Malware Protection automation workflow](../../images/GD-S3-Automation-Workflow.png)
*그림 17: S3 Malware Protection 자동화 워크플로*

첫 번째 단락에서 설명한 예를 달성하기 위해 [Amazon EventBridge](https://aws.amazon.com/eventbridge/) 및 AWS Lambda를 사용하여 AWS 계정에서 S3 Malware Protection이 구성될 때 [기본적으로 구성](https://docs.aws.amazon.com/guardduty/latest/ug/monitor-with-eventbridge-s3-malware-protection.html)되는 기본 이벤트 버스에 게시되는 S3 객체 스캔 결과를 기반으로 자동화된 솔루션을 구축할 수 있습니다. 그런 다음 GuardDuty가 부여한 객체 태그를 기반으로 파일을 처리하도록 Lambda 함수를 구성할 수 있습니다.

[GuardDuty 문서](https://docs.aws.amazon.com/guardduty/latest/ug/monitor-with-eventbridge-s3-malware-protection.html)에서 EventBridge 규칙의 예를 볼 수 있습니다. 여기에서 해당 EventBridge 규칙을 트리거로 사용하도록 Lambda 함수를 구성한 다음 필요에 따라 파일을 이동해야 합니다. 이를 수행하는 Python 코드 예제는 아래와 같습니다. 이것은 예제 코드이며 사용 사례에 맞게 변경해야 한다는 점을 명심하십시오.

```Python
#Script that evaluates messages about the scan status of S3 malware scanning and then moves the files to
#the appropriate bucket based on the status.


import boto3
import json
import os
import logging
from botocore.exceptions import ClientError
logger = logging.getLogger()
logger.setLevel(logging.INFO)

gdclient = boto3.client('guardduty')
s3client = boto3.client('s3')

infected_bucket = os.environ['INFECTED_BUCKET']
clean_bucket = os.environ['CLEAN_BUCKET']

def lambda_handler(event, context):
    logger.info('Event Data')
    logger.info(event)

    scan_status = event["detail"]["scanStatus"]
    bucket_name = event["detail"]["s3ObjectDetails"]["bucketName"]
    object_key = event["detail"]["s3ObjectDetails"]["objectKey"]
    scan_result = event["detail"]["scanResultDetails"]["scanResultStatus"]
    
    if scan_status == 'COMPLETED':
        logger.info("Scan status is COMPLETED")

        if scan_result == 'THREATS_FOUND':
            logger.info("Threats found in the object")
            logger.info("Moving file to infected bucket: %s",infected_bucket)
            
            copy_source = {'Bucket': bucket_name, 'Key': object_key}
            
            try:
                response = s3client.copy_object(
                    Bucket=infected_bucket,
                    CopySource=copy_source,
                    Key=object_key
                )
                
                print(response)
                
                logger.info("File copy successful")
                logger.info("Deleting object from source bucket")
                response = s3client.delete_object(
                    Bucket=bucket_name,
                    Key=object_key)
                    
                print (response)
                    
            except ClientError:
                raise

        elif scan_result == 'NO_THREATS_FOUND':
            logger.info("No threats found in the object")
            logger.info("Moving file to clean bucket: %s",clean_bucket)
            
            copy_source = {'Bucket': bucket_name, 'Key': object_key}
            
            try:
                response = s3client.copy_object(
                    Bucket=clean_bucket,
                    CopySource=copy_source,
                    Key=object_key
                )
                
                print(response)
                
                logger.info("File copy successful")
                logger.info("Deleting object from source bucket")
                response = s3client.delete_object(
                    Bucket=bucket_name,
                    Key=object_key)
                    
                print (response)
                
            except ClientError:
                raise
            

        else:
            logger.info("scan_result is: %s not moving", scan_result)
```

## 비용 최적화

계정에서 GuardDuty를 처음 활성화하면 해당 계정에 각 리전에 대해 30일 무료 평가판이 자동으로 제공됩니다. 이후 각 기능에도 무료 평가판이 있습니다. 가격 및 무료 평가판에 대한 자세한 내용은 [GuardDuty 가격 페이지](https://aws.amazon.com/guardduty/pricing/)를 참조하십시오. 이는 주어진 계정에서 월별 GuardDuty 비용을 이해하는 중요한 첫 번째 단계입니다. 평가판 기간 동안 관리자는 Account ID, Data source, Feature 및 S3 buckets를 기반으로 비용 추정치를 볼 수 있습니다. AWS 조직에서 GuardDuty를 활성화하는 경우 관리자는 모든 멤버 계정에 대한 비용 메트릭을 모니터링할 수 있습니다.

AWS 계정 관리자는 비용 증가가 발생하는 경우 GuardDuty와 관련된 요금을 조사해야 할 수 있습니다. 월별 GuardDuty 비용 증가의 일반적인 이유는 다음과 같습니다:

### CloudTrail 및/또는 S3 데이터 이벤트 사용량이 높은 경우

CloudTrail 요금 증가는 관리 이벤트 비용 또는 데이터 이벤트 비용에서 발생하는지에 따라 여러 이유로 인한 것일 수 있습니다. AWS 로그 데이터를 쿼리하는 방법은 여러 가지가 있지만 정확한 API 호출 및 관련 서비스를 보기 위해 Athena를 사용하는 예를 보여드리겠습니다. Athena를 사용하여 CloudTrail API 볼륨을 이해하려면:

1. 아직 없는 경우 새 CloudTrail 트레일을 생성합니다 [2]. 여기에서 관리 및 데이터 이벤트를 모두 고려하도록 선택합니다. 이 분석 범위를 벗어나 트레일이 필요하지 않은 경우 트레일을 삭제하거나 데이터 로깅을 비활성화할 수 있습니다.
2. CloudTrail 로그가 저장된 S3 위치에서 가져오는 Athena 테이블을 생성합니다 [3]
3. 이 테이블이 생성되면 Athena 콘솔로 이동하여 24시간 동안 쿼리를 수행하여 급증에 기여한 상위 이벤트 이름 및 이벤트 소스를 이해합니다

```SQL
SELECT eventName,count(eventName) AS eventVolume,eventSource
FROM your_athena_tablename
WHERE eventtime between '2021-01-24' AND '2021-01-25'
GROUP BY eventName, eventSource
ORDER BY eventVolume DESC limit 10;
```

4. 대량의 활동을 담당한 이벤트 및 이벤트 소스를 이해한 후 SQL 쿼리를 변경하여 시간, 사용자 에이전트, IP 또는 주체별로 데이터를 보는 것이 적합할 수 있습니다. 이렇게 하면 증가된 API 호출의 정확한 소스를 식별하는 데 도움이 됩니다

*[2] Creating a trail - <https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-a-trail-using-the-console-first-time.html>*
*[3] Querying AWS CloudTrail Logs - Using the CloudTrail Console to Create an Athena Table for CloudTrail Logs  - <https://docs.aws.amazon.com/athena/latest/ug/cloudtrail-logs.html#create-cloudtrail-table-ct>*

### VPC Flow Logs

VPC Flow log 관련 비용 증가는 GuardDuty가 플로우 로그를 분석하여 AWS 계정 및 워크로드에서 악의적이거나 무단이거나 예상치 못한 동작을 식별한 결과일 수 있습니다. 이 데이터 소스는 월별 기가바이트(GB)당 요금이 부과되며 계층화된 볼륨 할인이 제공됩니다.
Athena를 사용하여 VPC Flow log 볼륨을 이해하려면 다음 단계를 수행합니다:

1. Flow logs가 S3에 게시되고 있는지 확인합니다 [4]
2. 플로우 로그가 저장된 S3 위치에서 가져오는 Athena 테이블을 생성합니다 [5]
3. 이 테이블이 생성되면 Athena 콘솔로 이동하여 24시간 동안 쿼리를 수행하여 급증에 기여한 상위 플로우를 이해합니다

```SQL
SELECT COUNT(version) AS Flows,
        SUM(numpackets) AS Packetcount,
         sourceaddress
FROM vpc_flow_logs
WHERE date = DATE('2021-01-31')
GROUP BY  sourceaddress
ORDER BY  Flows DESC LIMIT 10;
```

4. 또는 VPC 플로우 로그가 최종 대상으로 CloudWatch 로그로 설정된 경우 고객은 CloudWatch insights를 사용하여 상위 통신자 정보를 추출할 수 있습니다 [6]. 다음은 insights 콘솔에서 직접 호출할 수 있는 몇 가지 샘플 쿼리입니다 [7]

*[4] Publishing flow logs to Amazon S3 - <https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-s3.html>*
*[5] Querying Amazon VPC Flow Logs - <https://docs.aws.amazon.com/athena/latest/ug/vpc-flow-logs.html>*
*[6] Analyzing Log Data with CloudWatch Logs Insights - <https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html>*
*[7] Sample Queries - <https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax-examples.html>*

Querying flow logs using Amazon Athena - <https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-athena.html>*

### DNS Query logs

DNS 로그 관련 비용 증가는 GuardDuty가 DNS 요청 및 응답을 분석하여 AWS 계정 및 워크로드에서 악의적이거나 무단이거나 예상치 못한 동작을 식별한 결과일 수 있습니다. 이 데이터 소스는 월별 기가바이트(GB)당 요금이 부과되며 계층화된 볼륨 할인이 제공됩니다.
VPC Flow logs와 마찬가지로 계정은 DNS 플로우 로그를 CloudWatch 또는 S3로 내보낼 수 있습니다 [8]. DNS의 상위 통신자는 CloudTrail Insights에서 쿼리할 수 있습니다:

1. Route53으로 이동하여 'Query Logging'이 활성화되어 있는지 확인합니다
2. 로거 구성을 확인하고 'CloudWatch Logs log group'이 대상 유형인지 확인합니다
3. 'VPC's that queries are logged for' 아래에 하나 이상의 VPC가 있는지 확인합니다
4. 'Destination ARN'의 일부로 표시된 로그 그룹을 클릭합니다
5. 왼쪽 탭에서 Insights를 클릭합니다
6. 쿼리가 로깅되는 로그 그룹을 선택합니다
7. 시간 범위를 선택하고 쿼리를 제공하여 가장 많은 아웃바운드 쿼리가 있는 인스턴스 및 호스트 이름을 이해합니다

```SQL
stats count(*) as numRequests by srcids.instance,query_name
    | sort numRequests desc
    | limit 10
```

또는 DNS 로그가 S3 버킷으로 라우팅되는 경우 Athena를 사용하여 S3에서 이 데이터를 가져오는 테이블을 생성할 수 있습니다 [9]. 수행할 단계:

1. Route53으로 이동하여 'Query Logging'이 활성화되어 있는지 확인합니다
2. 로거 구성을 확인하고 'S3 bucket'이 대상 유형인지 확인합니다
3. 'VPC's that queries are logged for' 아래에 하나 이상의 VPC가 있는지 확인합니다
4. Athena로 이동하여 다음 명령을 사용하여 테이블을 생성합니다

```SQL
CREATE EXTERNAL TABLE IF NOT EXISTS default.route53query (
  `version` float,
  `account_id` string,
  `region` string,
  `vpc_id` string,
  `query_timestamp` string,
  `query_name` string,
  `query_type` string,
  `query_class` string,
  `rcode` string,
  `answers` array<string>,
  `srcaddr` string,
  `srcport` int,
  `transport` string,
  `srcids` string,
  `isMatchedDomain` string
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
WITH SERDEPROPERTIES (
  'serialization.format' = '1'
) LOCATION 's3://<bucket name>/AWSLogs/<account number>/vpcdnsquerylogs/'
TBLPROPERTIES ('has_encrypted_data'='false');
```

5. 다음 쿼리를 사용하여 데이터를 쿼리합니다

```SQL
SELECT count(version) AS numRequests,
         srcids,
        query_name
FROM route53query
WHERE query_timestamp
    BETWEEN '2021-01-14'
        AND '2021-01-15'
GROUP BY  srcids,query_name
ORDER BY  numRequests DESC limit 10;
```

*[8] Log your VPC DNS queries with Route 53 Resolver Query Logs - https://aws.amazon.com/blogs/aws/log-your-vpc-dns-queries-with-route-53-resolver-query-logs/*
*[9] How to automatically parse Route 53 Resolver query logs - https://aws.amazon.com/blogs/networking-and-content-delivery/how-to-automatically-parse-route-53-resolver-query-logs/*
--

### 기타 고려 사항

* 때때로 CloudTrail S3 및 VPC/DNS 플로우 로그 분석에 대해 월초에 발생하는 비용 급증에 대한 이해가 필요합니다. 이는 이러한 데이터 소스의 가격 구조 때문일 가능성이 높습니다. GuardDuty는 계층화된 할인을 제공하며 월 동안 더 많은 이벤트가 분석됨에 따라 사용하기에 저렴해집니다. 더 나은 이해를 위해 가격 페이지(<https://aws.amazon.com/guardduty/pricing/>)를 참조하십시오.
* GuardDuty 비용은 새 애플리케이션 배포 또는 반복적인 'Describe' API 호출을 수행하는 타사 보안/스캔 도구 사용을 포함한 계정의 구성 변경으로 인해 증가할 수 있습니다. 이로 인해 CloudTrail 이벤트 및/또는 트래픽(VPC 및 DNS 로그)이 증가할 수 있습니다.
* CloudTrail 이벤트 기록의 CSV 다운로드에 의존하여 이벤트 볼륨을 비교하는 것은 권장되지 않습니다. 이벤트 필터가 있거나 불완전한 데이터가 있을 수 있기 때문입니다. 이 기능은 더 작은 CSV 다운로드에 더 적합합니다.
* CloudTrail insights는 이러한 조사를 보강하는 데 유용할 수 있습니다. CloudTrail Insights는 CloudTrail 쓰기 관리 이벤트를 지속적으로 모니터링하고 수학적 모델을 사용하여 계정에 대한 API 및 서비스 이벤트 활동의 정상 수준을 결정합니다.
* 새 멤버 계정이 추가되거나 GuardDuty 서비스가 장기간 비활성화된 후 재개되는 경우 해당 비용도 급증으로 인식될 수 있습니다.
* S3 보호는 비활성화할 수 있지만 GuardDuty는 추가 데이터 소스를 제거할 수 없습니다. 그러나 VPC 수준에서 DNS가 비활성화된 경우 해당 로그는 GuardDuty에서 처리되지 않습니다. 계정은 요금이 부과되지 않거나 DNS 기반 결과가 없습니다.
* GuardDuty는 보안 가치에 최적화되어 있으며 고객의 CloudTrail에 전달될 일부 낮은 위험 이벤트 처리에 대해 고객에게 요금을 부과하지 않습니다. 따라서 이벤트 수가 정확히 일치하지 않을 수 있습니다.
* 계정에 대해 Runtime Monitoring이 활성화된 경우 GuardDuty 에이전트가 배포되고 활성화된 인스턴스의 VPC Flow Logs 분석에 대한 요금이 부과되지 않습니다.


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

### Shared VPCs 문제 해결
AWS 계정의 EC2 인스턴스는 다른 AWS 계정이 소유한 [공유 VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html)의 서브넷에서 실행될 수 있습니다. 이 시나리오에서 GuardDuty 콘솔은 Runtime Monitoring 섹션에 나열된 EC2 인스턴스에 대해 "VPC Endpoint Creation Failed" 오류 메시지를 표시할 수 있습니다.
* 공유 VPC를 소유한 AWS 계정에서 GuardDuty EC2 Runtime Monitoring을 활성화합니다.
* 공유 VPC 및 VPC를 소유한 AWS 계정에서 실행 중인 EC2 인스턴스가 하나 이상 있는지 확인합니다.

### 사용자 지정 재귀 DNS 아키텍처 문제 해결

"VPC Endpoint Creation Failed" 오류 메시지가 표시되는 또 다른 이유는 기본 [Amazon Route 53 Resolver](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html)(종종 ".2 resolver"라고 함) 대신 사용자 지정 DNS 서버를 사용하기 때문일 수 있습니다. GuardDuty는 Route 53 resolver에 의존하여 GuardDuty VPC 엔드포인트를 찾는 방법을 알기 때문에 VPC 엔드포인트 구성에 문제가 발생할 수 있습니다. 재귀 DNS에 AWS 관리형 DNS resolver를 사용하는 것이 좋지만 일부 고객은 사용자 지정 DNS resolver를 사용하기로 결정합니다. 사용자 지정 DNS 서버를 계속 사용하면서 이 문제를 해결하려면 두 가지 옵션이 있습니다:

**옵션 1:** 최선의 옵션은 "Enable DNS resolution" 상자를 선택한 상태로 두어 GuardDuty가 Route 53 resolver를 사용하여 GuardDuty VPC 엔드포인트를 찾을 수 있도록 하는 것입니다. 사용자 지정 DNS 서버에 대해 구성한 DHCP 옵션 세트는 호스트가 사용자 지정 DNS 서버를 사용하도록 호스트의 resolver.conf 파일을 설정합니다. GuardDuty 에이전트는 .2 resolver 옵션을 사용합니다. VPC에서 Route 53 DNS Resolver를 활성화하려면 아래 단계를 따르십시오.

* [Amazon VPC Console](https://console.aws.amazon.com/vpc/)을 방문합니다.
* DNS 설정을 변경하려는 VPC를 선택합니다.
* 오른쪽 상단 모서리에서 **Actions** 드롭다운을 사용하고 **VPC settings** 편집을 클릭합니다


![Image](../../images/edit-vpc-settings.png)


* VPC 설정 페이지에서 **Enable DNS resolution** 상자가 선택되어 있는지 확인합니다. 이렇게 하면 이 VPC에 대해 Route53 DNS Resolver가 활성화됩니다.


![Image2](../../images/dns-settings.png)


* Route 53 resolver를 사용할 수 있는 트래픽을 제한하려고 사용자 지정 DNS 서버를 구성한 경우 Route 53 DNS 방화벽을 사용하여 GuardDuty 엔드포인트에 대한 확인을 허용하고 다른 것은 허용하지 않을 수 있습니다. 이렇게 하려면 Route 53 DNS 방화벽 정책에서 두 개의 규칙을 만들어야 합니다. 최상위 우선순위를 가진 규칙은 "guardduty-data.<region>.amazonaws.com"에 대한 확인을 허용합니다. 규칙 평가 우선순위에서 따라오는 규칙은 "*."를 거부해야 합니다. 이러한 규칙의 조합은 Route 53 resolver를 통해 GuardDuty 엔드포인트에 대한 확인을 허용하면서 다른 확인은 거부합니다. (GuardDuty는 의심스러운 활동에 대해 이 Route53 resolver 트래픽도 모니터링합니다.)**



**옵션 2:** [중앙 집중식 엔드포인트 VPC 아키텍처](https://aws.amazon.com/blogs/networking-and-content-delivery/centralize-access-using-vpc-interface-endpoints/)를 사용하는 경우 사용자 지정 DNS 서버에 항목을 생성하여 GuardDuty 엔드포인트를 확인할 수 있습니다. 이렇게 하려면 GuardDuty 에이전트가 확인하려는 중앙 VPC에 특정한 엔드포인트를 확인한 다음 DNS 항목으로 추가해야 합니다.

### ECS Task Execution Role 오류 문제 해결
문제 유형 **Agent exited** 및 **CannotPullContainerError**라는 추가 정보 오류 메시지가 표시되면 ECS Fargate 작업 실행 역할에 대한 IAM 권한이 부족하기 때문일 가능성이 높습니다. Fargate 작업은 작업 실행 역할을 **반드시** 사용해야 합니다. 이 역할은 작업에 사용자를 대신하여 GuardDuty 보안 에이전트를 검색, 업데이트 및 관리할 수 있는 권한을 부여합니다. 이 문제를 해결하려면 아래 단계를 따르십시오.

* [AWS IAM 콘솔](https://console.aws.amazon.com/iam/)을 방문합니다.
* 왼쪽 탐색 창에서 **Roles**를 선택한 다음 **Create role**을 선택합니다.
* **Trusted entity type**에서 **AWS service**를 선택합니다.
* **Service or use case**에서 **Elastic Container Service**를 선택한 다음 **Elastic Container Service Task** 사용 사례를 선택합니다.
* **Next**를 선택합니다.
* **Add permissions** 섹션에서 **AmazonECSTaskExecutionRolePolicy**를 검색한 다음 정책을 선택합니다.
* **Next**를 선택합니다.
* **Role name**에 **ecsTaskExecutionRole**을 입력합니다.
* 역할을 검토한 다음 **Create role**을 선택합니다.


---
**Runtime Monitoring은 GuardDuty를 통해서만 Amazon ECS 클러스터(AWS Fargate)에 대한 보안 에이전트 관리를 지원합니다. Amazon ECS 클러스터에서 보안 에이전트를 수동으로 관리하는 것은 지원되지 않습니다.**

---

## 리소스

### 워크샵

* [Activation Days](https://awsactivationdays.splashthat.com/)
* [Amazon GuardDuty workshop](https://catalog.workshops.aws/guardduty)
* [Amazon Detective workshop](https://catalog.workshops.aws/detective)
* [Container security workshop](https://catalog.workshops.aws/containersecurity)

### 비디오

* [Introducing GuardDuty ECS Runtime Monitoring](https://www.youtube.com/watch?v=nuMOaQctNgE&list=PLB3flZ7qA4xu__uOEfpc-coXm04swNWva&index=4&pp=iAQB)
* [Amazon GuardDuty EKS Protection Overview](https://www.youtube.com/watch?v=mxNyxJ_Mo8k&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=14&pp=iAQB)
* [Amazon GuardDuty S3 Protection Overview](https://www.youtube.com/watch?v=yNUdNH31BUw&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=15&pp=iAQB)
* [Amazon GuardDuty findings summary view](https://www.youtube.com/watch?v=4rR37ZBJWdI&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=30&pp=iAQB)
* [Enable GuardDuty Lambda Protection to monitor your Lambda execution environment](https://www.youtube.com/watch?v=dpc6jvtHB0g&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=38&pp=iAQB)
* [GuardDuty EKS Runtime Monitoring](https://www.youtube.com/watch?v=t3rVVilJWEk&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=40&pp=iAQB)
* [Amazon GuardDuty RDS Protection](https://www.youtube.com/watch?v=f_CFtrAG8Nw&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=41&pp=iAQB)
* [Amazon Guardduty Extended Threat Detection](https://www.youtube.com/embed/xLqGwoSaoPw?si=_wprcGe8l9oypKFR)
* [S3 Malware Protection overview](https://youtu.be/uweeumMAif4)

### 블로그

* [Measure cluster performance impact of Amazon GuardDuty EKS Agent](https://aws.amazon.com/blogs/containers/measure-cluster-performance-impact-of-amazon-guardduty-eks-agent/)
* [How AWS threat intelligence deters threat actors](https://aws.amazon.com/blogs/security/how-aws-threat-intelligence-deters-threat-actors/)
* [Improve your security investigations with Detective finding groups visualizations](https://aws.amazon.com/blogs/security/improve-your-security-investigations-with-detective-finding-groups-visualizations/)
* [Three ways to accelerate incident response in the cloud: insights from re:Inforce 2023](https://aws.amazon.com/blogs/security/three-ways-to-accelerate-incident-response-in-the-cloud-insights-from-reinforce-2023/)
* [Detect threats to your data stored in RDS databases by using GuardDuty](https://aws.amazon.com/blogs/security/detect-threats-to-your-data-stored-in-rds-databases-by-using-guardduty/)
* [Reduce triage time for security investigations with Amazon Detective visualizations and export data](https://aws.amazon.com/blogs/security/reduce-triage-time-for-security-investigations-with-detective-visualizations-and-export-data/)
* [How to use Amazon GuardDuty and AWS WAF v2 to automatically block suspicious hosts](https://aws.amazon.com/blogs/security/how-to-use-amazon-guardduty-and-aws-waf-v2-to-automatically-block-suspicious-hosts/)
* [How to improve security incident investigations using Amazon Detective finding groups](https://aws.amazon.com/blogs/security/how-to-improve-security-incident-investigations-using-amazon-detective-finding-groups/)
* [Automatically block suspicious DNS activity with Amazon GuardDuty and Route 53 Resolver DNS Firewall](https://aws.amazon.com/blogs/security/automatically-block-suspicious-dns-activity-with-amazon-guardduty-and-route-53-resolver-dns-firewall/)
* [How to use new Amazon GuardDuty EKS Protection findings](https://aws.amazon.com/blogs/security/how-to-use-new-amazon-guardduty-eks-protection-findings/)
