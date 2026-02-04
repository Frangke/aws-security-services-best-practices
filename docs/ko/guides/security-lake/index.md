# Security Lake

## 소개

Amazon Security Lake 모범 사례 가이드에 오신 것을 환영합니다. 이 가이드의 목적은 AWS 계정에 저장된 목적별 데이터 레이크에 보안 데이터를 중앙 집중화하기 위해 Amazon Security Lake를 활용하는 규범적 지침을 제공하는 것입니다. GitHub를 통해 이 지침을 게시하면 서비스 개선 사항과 사용자 커뮤니티의 피드백을 포함하는 시기적절한 권장 사항을 신속하게 반복할 수 있습니다. 이 가이드는 처음으로 Security Lake를 배포하거나 기존 다중 계정 배포에서 Security Lake를 최적화하는 방법을 찾는 경우 모두에게 가치를 제공하도록 설계되었습니다.

## Amazon Security Lake란 무엇인가요?

Amazon Security Lake는 완전 관리형 보안 데이터 레이크 서비스입니다. Security Lake를 사용하여 AWS 환경, SaaS 제공업체, 온프레미스, 클라우드 소스 및 타사 소스의 보안 데이터를 AWS 계정에 저장된 목적별 데이터 레이크로 자동으로 중앙 집중화할 수 있습니다. Amazon Simple Storage Service(Amazon S3) 버킷으로 지원되므로 수집한 데이터의 소유권을 유지합니다. Security Lake는 보안 데이터 수집 및 통찰력 수집을 단순화합니다. 이는 워크로드, 애플리케이션 및 데이터의 보호를 개선한다는 궁극적인 목표로 광범위한 사용 사례를 지원합니다.

## Security Lake 활성화의 이점은 무엇인가요?

Security Lake는 통합된 AWS 서비스 및 타사 서비스에서 보안 관련 로그 및 이벤트 데이터의 수집을 자동화합니다. 또한 사용자 지정 가능한 보존 및 복제 설정으로 데이터의 수명 주기를 관리하는 데 도움이 됩니다. Security Lake는 수집된 데이터를 Apache Parquet 형식과 [Open Cybersecurity Schema Framework(OCSF)](https://github.com/ocsf)라는 표준 오픈 소스 스키마로 변환합니다. OCSF 지원을 통해 Security Lake는 AWS 및 광범위한 엔터프라이즈 보안 데이터 소스의 보안 데이터를 정규화하고 결합합니다.

다른 AWS 서비스 및 타사 서비스는 인시던트 대응 및 보안 데이터 분석을 위해 Security Lake에 저장된 데이터를 구독할 수 있습니다.

## 이 가이드 사용 방법

이 가이드는 AWS 및 하이브리드 환경 전반에 걸쳐 보안 로그를 수집하고 정규화하는 책임이 있는 보안 실무자를 대상으로 합니다. 모범 사례는 더 쉬운 소비를 위해 Security Lake 배포에 대해 고려해야 하는 순서대로 제시됩니다. 그러나 주제는 필요에 따라 순서에 관계없이 소비할 수 있습니다.

* [Security Lake 배포를 위한 전제 조건](#security-lake-배포를-위한-전제-조건)
    * [권한 및 역할](#권한-및-역할)
    * [위임된 관리](#위임된-관리)
    * [CloudTrail 관리 이벤트](#cloudtrail-관리-이벤트)
* [배포 계획](#배포-계획)
    * [보안 데이터를 수집할 리전](#보안-데이터를-수집할-리전)
    * [롤업 리전](#롤업-리전)
    * [기본 데이터 소스](#기본-데이터-소스)
    * [Security Lake 파트너](#security-lake-파트너)
    * [사용자 지정 데이터 소스](#사용자-지정-데이터-소스)
* [Security Lake 구현](#security-lake-구현)
* [Security Lake 구현 미세 조정](#security-lake-구현-미세-조정)
    * [로그 소스별 변경](#로그-소스별-변경)
    * [리전별 변경](#리전별-변경)
    * [계정 및 리전별 변경](#계정-및-리전별-변경)
    * [리전 및 계정 추가](#리전-및-계정-추가)
    * [S3 수명 주기 직접 변경](#s3-수명-주기-직접-변경)
* [타사 데이터 수집 구성](#타사-데이터-수집-구성)
    * [파트너 소스](#파트너-소스)
    * [Security Hub를 사용하여 결과를 Security Lake로 라우팅](#security-hub를-사용하여-결과를-security-lake로-라우팅)
    * [사용자 지정 소스](#사용자-지정-소스)
* [Security Lake 운영화](#security-lake-운영화)
    * [AWS 기본 분석](#aws-기본-분석)
    * [구독자 파트너](#구독자-파트너)
    * [OCSF 스키마 업데이트](#ocsf-스키마-업데이트)
* [비용 고려 사항](#비용-고려-사항)
    * [비용 분석 및 가시성](#비용-분석-및-가시성)
    * [내장된 비용 절감](#내장된-비용-절감)
    * [중복 기본 로그를 수집하지 않는지 확인](#중복-기본-로그를-수집하지-않는지-확인)
    * [CloudTrail 관리 이벤트 비용 절감](#cloudtrail-관리-이벤트-비용-절감)
    * [롤업 리전](#롤업-리전-1)
    * [로그를 Glacier로 이동](#로그를-glacier로-이동)
* [추가 리소스](#추가-리소스)

## Security Lake 배포를 위한 전제 조건

Security Lake를 배포하려면 다음 전제 조건을 미리 해결해야 합니다.

### 권한 및 역할

Amazon Security Lake를 시작하기 전에 서비스를 관리할 수 있는 권한이 있는지 확인해야 합니다. "AmazonSecurityLakeAdministrator"라는 AWS 관리형 정책을 사용할 수 있지만 항상 최소 권한을 염두에 두십시오. Security Lake를 프로그래밍 방식으로 활성화 또는 구성하거나 API 또는 CLI를 사용하여 액세스하려는 경우 [여기](https://docs.aws.amazon.com/security-lake/latest/userguide/getting-started.html#get-started-programmatic)에 명시된 IAM 역할을 생성해야 합니다. 콘솔을 사용하여 Security Lake를 활성화하고 구성하는 경우 이러한 역할이 자동으로 관리됩니다. 필수는 아니지만 일반적으로 이러한 이유로 콘솔을 사용하여 Security Lake를 배포하는 것이 좋습니다.

### 위임된 관리

AWS Organization 관리 계정을 사용하여 Security Lake 위임된 관리자 계정을 지정해야 합니다. AWS Organization에서 Security Lake 위임된 관리자로 사용하기에 가장 적합한 계정을 고려해야 합니다. [AWS 보안 참조 아키텍처](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/welcome.html)는 Security Lake의 위임된 관리로 [로그 아카이브 계정](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/log-archive.html)을 사용할 것을 권장합니다. 조직의 권장 계정 구조에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/organizations.html)에서 확인할 수 있습니다.

### CloudTrail 관리 이벤트

Security Lake로 CloudTrail 관리 이벤트를 수집하려면 읽기 및 쓰기 CloudTrail 관리 이벤트를 수집하는 CloudTrail 다중 리전 조직 추적이 하나 이상 활성화되어 있어야 합니다. Control Tower를 사용하는 경우 이는 Control Tower 구성의 일부일 가능성이 높습니다. CloudTrail을 통해 추적을 생성하고 관리하는 방법에 대한 정보는 *AWS CloudTrail 사용자 가이드*의 [조직에 대한 추적 생성](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/creating-trail-organization.html)을 참조하십시오. AWS Control Tower를 통해 추적을 생성하고 관리하는 방법에 대한 정보는 *AWS Control Tower 사용자 가이드*의 [AWS CloudTrail로 AWS Control Tower 작업 로깅](https://docs.aws.amazon.com/controltower/latest/userguide/logging-using-cloudtrail.html)을 참조하십시오.

## 배포 계획

Security Lake는 15일 무료 평가판을 제공합니다. Security Lake를 활성화하기 전에 다음 결정을 내려 이 무료 평가판을 최대한 활용하는 것이 좋습니다.

### 보안 데이터를 수집할 리전

Security Lake는 전역 가시성을 갖춘 리전 서비스입니다. 이를 통해 한 리전에서 Security Lake 콘솔에 액세스하고 원하는 모든 리전에서 로그 수집을 구성할 수 있습니다. 적극적으로 사용하지 않는 리전을 포함하여 모든 리전에서 Security Lake를 활성화하는 것이 좋습니다. Security Lake는 종량제 서비스입니다. 따라서 사용하지 않는 리전에서는 거의 데이터를 수집하지 않으므로 모든 리전에서 Security Lake를 활성화하는 것이 비용 효율적입니다. 그런 다음 예상치 못한 리전에서 활동이 발생하는 경우 관련 로그를 캡처했으며 조사에 사용할 수 있습니다. 또는 고객이 [이와 같은](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples_general.html#example-scp-deny-region) SCP를 사용하여 사용하지 않는 리전을 완전히 차단하도록 선택할 수 있습니다.

### 롤업 리전

"롤업 리전"을 사용하여 선택적으로 데이터를 집계할 수 있습니다. 롤업 리전은 지정한 "기여 리전"에서 보안 데이터의 복사본을 받도록 선택한 리전입니다. 이 점을 자세히 설명하려면 S3 버킷 이름은 전역적으로 고유하지만 S3에 저장된 데이터는 리전에 특정하다는 것을 기억하는 것이 중요합니다. Security Lake를 구성할 때 로그를 수집하는 각 리전에 대해 별도의 버킷이 생성됩니다. 롤업 리전을 사용하면 기여 리전이 데이터를 롤업 리전 버킷으로 "롤업"하여 모든 로그를 지정한 리전에 저장할 수 있습니다. 이는 데이터 거주 요구 사항에 필요한 유연성을 제공할 수 있습니다. 롤업 리전의 데이터는 이동되지 않고 복제됩니다. 중복 데이터를 최소화하려면 기여 리전에 대한 스토리지 클래스 전환을 사용하여 데이터의 만료 기간을 설정할 수 있습니다. 자체 위험 프로필 및 제어 환경을 기반으로 편안한 길이로 만료 기간을 설정하는 것이 좋습니다. 예를 들어 일부 고객은 주말에 발생하는 문제를 다루기에 충분히 길기 때문에 3일의 만료 기간이 비용과 로그 손실 위험 사이의 적절한 균형이라고 생각합니다. 다른 고객은 문제를 식별할 시간을 더 길게 갖기 위해 7일의 만료 기간을 선호합니다.

### 기본 데이터 소스

Security Lake를 배포하기 전에 수집하려는 기본 데이터 소스를 고려하십시오. Security Lake는 다음 기본 AWS 로그의 관리형 수집을 지원합니다:

* 기본 로그
    * AWS CloudTrail 관리 이벤트
    * Amazon Elastic Kubernetes Service(Amazon EKS) 감사 로그
    * Amazon Route 53 리졸버 쿼리 로그
    * AWS Security Hub 결과(Security Hub가 소비하는 결과 포함)
    * Amazon Virtual Private Cloud(Amazon VPC) Flow Logs
* 옵트인 로그
    * AWS CloudTrail 데이터 이벤트(S3, Lambda)
    * Amazon Web Application Firewall 로그

배포 중에 수집할 로그를 선택할 수 있습니다. 기본 로그는 이미 선택되어 있지만 수집하지 않도록 선택할 수 있습니다. AWS CloudTrail 데이터 이벤트 및 WAF 로그는 높은 볼륨과 수집과 관련된 비용으로 인해 기본적으로 수집되지 않습니다. 이러한 로그를 수집하려면 배포 중 또는 배포 후 구성 변경을 통해 수집을 옵트인해야 합니다.

모든 기본 AWS 로그 소스를 켜는 것이 좋습니다. Security Lake는 서비스 또는 리소스 수준에서 추가 구성을 수행할 필요 없이 이러한 로그를 대신 수집합니다(위의 전제 조건 섹션에 명시된 CloudTrail 관리 이벤트 제외). 그런 다음 Security Lake는 이 데이터를 [Open Cybersecurity Schema Framework(OCSF)](https://github.com/ocsf) 형식으로 변환하고 Apache Parquet 파일로 환경의 S3 버킷에 이러한 로그를 넣습니다. 옵트인 로그 소스는 일반적으로 매우 높은 볼륨이므로 이러한 로그를 수집하고 저장하는 데 더 높은 비용이 발생합니다. 많은 고객이 이러한 로그를 수집하는 데 가치를 보지만 이러한 로그 소스를 수집하기로 결정하기 전에 제공하는 보안 이점과 수집 비용을 완전히 이해하는 것이 좋습니다.

### Security Lake 파트너

Security Lake의 주요 이점은 Security Lake로 데이터를 보내거나 Security Lake에서 데이터를 소비하도록 설계된 사전 구축된 파트너 통합입니다. Security Lake 파트너에는 소스 파트너, 구독자 파트너 및 서비스 파트너의 세 가지 유형이 있습니다. [소스 파트너](https://aws.amazon.com/security-lake/partners/#Source_Partners)는 OCSF 형식으로 보안 로그 및 결과를 Security Lake로 보낼 수 있는 타사입니다. [구독자 파트너](https://aws.amazon.com/security-lake/partners/#Subscriber_Partners)는 Security Lake에서 데이터를 받는 타사입니다. 위협 탐지, 조사 및 인시던트 대응과 같은 보안 사용 사례에 도움이 되는 도구인 경우가 많습니다. [서비스 파트너](https://aws.amazon.com/security-lake/partners/#Service_Partners)는 Security Lake를 구축하거나 활용하는 데 도움을 주는 타사입니다.

Security Lake 15일 무료 평가판을 최대한 활용하려면 샌드박스 계정에서 통합을 테스트하여 프로덕션 데이터로 Security Lake를 테스트할 때 파트너 통합을 신속하게 배포할 수 있는지 확인할 수 있습니다. 각 파트너에는 [여기](https://docs.aws.amazon.com/security-lake/latest/userguide/integrations-third-party.html)에서 찾을 수 있는 특정 통합 세부 정보가 있습니다.

### 사용자 지정 데이터 소스

Security Lake는 사용자 지정 데이터 소스의 집계도 지원합니다. 현재 Security Lake 파트너가 아닌 도구를 사용하는 경우 사용자 지정 소스를 생성하면 여전히 로그 또는 결과를 Security Lake로 보낼 수 있습니다. 일부 고객은 기본 AWS 로그로 시작한 다음 파트너 소스 및 구독자를 추가한 다음 사용자 지정 소스를 추가하는 단계로 Security Lake를 배포하기로 선택합니다. 사용자 지정 소스를 설정하는 방법에 대한 자세한 내용은 [Security Lake 문서](https://docs.aws.amazon.com/security-lake/latest/userguide/custom-sources.html) 또는 이 [블로그](https://aws.amazon.com/blogs/security/patterns-for-consuming-custom-log-sources-in-amazon-security-lake/)를 참조하십시오.

## Security Lake 구현

이제 AWS Organization 전체에 Security Lake를 배포할 준비가 되었습니다. 배포 중에 Security Lake를 활성화하려는 계정 및 리전을 선택하고 수집하려는 데이터 소스를 지정합니다. 이때 선택적으로 롤업 리전 및 수명 주기 정책을 구성할 수도 있습니다. AWS 콘솔을 사용하여 Security Lake를 활성화하는 과정을 안내합니다. CLI 또는 API를 사용하여 Security Lake를 구성하는 방법에 대한 정보는 [Security Lake 문서](https://docs.aws.amazon.com/security-lake/latest/userguide/getting-started.html#get-started-console)를 참조하십시오.

1. Organization 관리 계정에서 Security Lake 콘솔을 엽니다: https://console.aws.amazon.com/securitylake
2. "시작하기"를 클릭합니다.
    ![image0](../../images/security-lake/Image0.jpg)
3. Security Lake의 위임된 관리자로 사용하려는 계정의 12자리 계정 ID를 입력한 다음 "위임"을 클릭합니다.
    ![image1](../../images/security-lake/Image1.jpg)
4. 위임된 관리자 계정으로 전환하고 Security Lake 콘솔을 엽니다: https://console.aws.amazon.com/securitylake. Organization 관리 계정에서 본 것과 동일한 랜딩 페이지가 표시됩니다. "시작하기"를 다시 클릭하면 Security Lake 활성화 페이지로 이동합니다.
5. 먼저 Security Lake로 수집할 기본 로그 소스를 선택합니다. 기본 소스를 수집하거나 이 창 상단의 라디오 버튼을 사용하여 소스를 지정하도록 선택할 수 있습니다. 로그 소스를 지정하도록 선택하면 수집하려는 로그를 선택해야 합니다.
    ![image2](../../images/security-lake/Image2.jpg)
    ![image3](../../images/security-lake/Image3.jpg)
6. 다음으로 이전에 선택한 로그 소스를 수집할 리전을 선택합니다. 다시 말하지만 기본 리전(지원되는 모든 리전)에서 수집하거나 수동으로 리전을 지정하는 옵션이 있습니다.
    ![image4](../../images/security-lake/Image4.jpg)
    ![image5](../../images/security-lake/Image5.jpg)
7. 다음으로 데이터를 수집할 계정을 선택합니다. 조직의 모든 계정, 특정 계정(CSV 형식으로 나열) 또는 현재 계정만 수집하도록 선택할 수 있습니다. 이 창 하단의 확인란은 조직에 추가된 새 계정에 Security Lake 수집 설정을 적용합니다. 이를 통해 조직이 성장하더라도 관리형 로그 수집이 가능합니다. 조직에 새 계정이 추가될 때 수동 작업을 줄이기 위해 이 기능을 활성화된 상태로 두는 것이 좋습니다. 이 동작을 원하지 않으면 이 확인란을 선택 취소하십시오. Security Lake를 이미 배포했고 이 설정이 활성화되어 있는지 확인하려면 이 [API 호출을 사용하여 확인](https://docs.aws.amazon.com/security-lake/latest/APIReference/API_GetDataLakeOrganizationConfiguration.html)하고 이 [API 호출을 사용하여 설정을 활성화](https://docs.aws.amazon.com/security-lake/latest/APIReference/API_CreateDataLakeOrganizationConfiguration.html)할 수 있습니다.
    ![image6](../../images/security-lake/Image6.jpg)
8. 초기 설정 중에는 Security Lake를 선택한 모든 로그 소스를 선택한 모든 계정 및 리전에서 수집하도록만 구성할 수 있습니다. 즉, 선택한 로그는 Security Lake를 활성화하는 모든 계정 및 리전에 적용됩니다. 다른 리전에서 다른 로그 소스를 수집하거나 특정 계정에 대한 로그 소스를 제외하는 것과 같은 보다 구체적인 요구 사항이 있는 경우 초기 배포 후 Security Lake 콘솔을 통해 구성할 수 있습니다.
9. 선택적으로 서비스 액세스 및 태그를 구성합니다. 서비스 액세스에 대한 자세한 내용은 [Security Lake 문서](https://docs.aws.amazon.com/security-lake/latest/userguide/getting-started.html#prerequisites)를 참조하십시오.
   ![image7](../../images/security-lake/Image7.jpg)
10. "다음"을 클릭하여 다음 옵션 페이지로 이동합니다. 여기에서 선택적으로 롤업 리전 및 스토리지 클래스 전환을 구성할 수 있습니다.
    ![image8](../../images/security-lake/Image8.jpg)
11. 롤업 리전을 구성하도록 선택하는 경우 "롤업 리전 추가"를 클릭하고 롤업 리전 및 기여 리전을 지정합니다. 롤업 리전에 기여하려는 각 리전에 대해 이 작업을 반복합니다. 리전은 기여 리전과 롤업 리전이 될 수 없습니다. 또한 아래 예와 같이 둘 이상의 롤업 리전을 생성할 수 있습니다.
    ![image9](../../images/security-lake/Image9.jpg)
12. 전체 Security Lake 배포에 적용될 스토리지 전환을 추가하려면 "전환 추가"를 클릭하고 원하는 스토리지 클래스를 지정하고 일수를 입력하여 지금 추가할 수 있습니다. 초기 배포 후 Security Lake 콘솔을 통해 각 리전에 대해 보다 세분화된 스토리지 전환을 생성할 수 있습니다.
    ![image10](../../images/security-lake/Image10.jpg)
13. 롤업 리전을 구성한 경우 복제를 위한 새 서비스 역할을 생성하거나 기존 역할을 사용하도록 선택하라는 메시지가 표시됩니다. 기존 역할 사용에 대한 자세한 내용은 [Security Lake 문서](https://docs.aws.amazon.com/security-lake/latest/userguide/manage-regions.html#add-rollup-region)를 참조하십시오.
    ![image11](../../images/security-lake/Image11.jpg)
14. 다음을 클릭하여 Security Lake 배포를 검토합니다. 이러한 설정은 나중에 변경할 수 있지만 지금 구성 옵션을 확인하는 데 시간을 할애하는 것이 좋습니다. 준비가 되면 "생성"을 클릭하여 Security Lake를 배포합니다.
15. 몇 분 후 선택한 모든 리전에서 Security Lake가 성공적으로 배포되었다는 확인이 표시됩니다. Security Lake는 즉시 데이터 수집을 시작합니다.

## Security Lake 구현 미세 조정

Security Lake의 초기 배포 후 Security Lake 콘솔을 통해 어떤 계정 및 리전에서 어떤 로그를 수집하는지 추가로 세분화할 수 있습니다. 초기 설정 중에 제외한 새 계정 및 리전에 대한 범위를 추가할 수도 있습니다.

### 로그 소스별 변경

로그 소스별로 Security Lake 구성을 변경하면 로그 소스를 선택하고 수집되는 정확한 리전을 지정할 수 있습니다. 이는 많은 리전에 대해 하나의 로그 소스 수집을 변경하는 가장 좋은 방법입니다. 이는 Security Lake 콘솔의 "소스" 페이지를 통해 수행됩니다.

이 페이지에서 로그 소스를 선택하고 오른쪽 상단의 "구성"을 클릭합니다. 그런 다음 지정한 리전에서 해당 로그 소스를 활성화할지 비활성화할지 선택할 수 있습니다. 이러한 변경 사항은 지정된 리전에 대해 Security Lake가 로그를 수집하도록 구성된 모든 계정에 영향을 미칩니다.

![image12](../../images/security-lake/Image12.jpg)
![image13](../../images/security-lake/Image13.jpg)

### 리전별 변경

리전별로 Security Lake 구성을 변경하면 리전을 선택하고 해당 리전에서 수집하려는 로그 소스를 지정할 수 있습니다. 이는 특정 리전의 수집 설정을 변경하는 가장 좋은 방법입니다. 이 방법을 사용하여 기본 S3 버킷의 스토리지 전환을 수정할 수도 있습니다. 이는 Security Lake 콘솔의 "리전" 페이지를 통해 수행됩니다.

이 페이지에서 리전을 선택하고 페이지 오른쪽 상단의 "편집"을 클릭합니다. 다음으로 "다음에 대한 모든 계정의 소스 재정의..."로 시작하는 문장 옆의 확인란을 클릭합니다. 그런 다음 선택한 리전의 모든 계정에 대해 수집된 로그 소스를 구성할 수 있습니다. 동일한 페이지에서 S3 스토리지 전환을 구성하고 태그를 추가할 수도 있습니다.

![image14](../../images/security-lake/Image14.jpg)
![image15](../../images/security-lake/Image15.jpg)
![image16](../../images/security-lake/Image16.jpg)

### 계정 및 리전별 변경

계정/리전 쌍별로 Security Lake 구성을 변경하면 어디에서 어떤 로그 소스를 수집할지 가장 세분화된 변경이 가능합니다. 이는 Security Lake 콘솔의 "계정" 페이지를 통해 수행됩니다.

이 페이지에서 계정 및 리전 쌍을 선택하고 오른쪽 상단의 "편집"을 클릭합니다. 그런 다음 해당 특정 리전의 특정 계정에 대한 로그 소스 수집을 변경할 수 있습니다.

![image17](../../images/security-lake/Image17.jpg)
![image18](../../images/security-lake/Image18.jpg)

### 리전 및 계정 추가

Security Lake를 초기 설정할 때 모든 계정 및 리전을 포함하지 않은 경우 Security Lake 콘솔의 "리전" 및 "계정" 페이지를 통해 추가할 수 있습니다. 각 페이지의 오른쪽 상단에는 새 리전 또는 계정을 추가하는 주황색 버튼이 있습니다. 그런 다음 전체 리전 또는 지정한 새 계정(쉼표로 구분된 계정 ID 목록)에 대해 원하는 로그 소스를 지정하는 유사한 설정 마법사가 표시됩니다.

### S3 수명 주기 직접 변경

Security Lake의 기본 스토리지 아키텍처는 S3이므로 버킷 및 접두사 수준에서 직접 수명 주기 관리를 구성할 수 있습니다. 이를 통해 ASL 콘솔에서 지원하는 대량 변경보다 버킷 수명 주기를 더 세밀하게 사용자 지정할 수 있습니다. 그러나 S3에서 직접 수행한 변경 사항은 Security Lake 콘솔에 반영되지 않습니다. 자세한 내용은 [S3 문서](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-transition-general-considerations.html)를 참조하십시오.

## 타사 데이터 수집 구성

Security Lake를 사용하면 AWS 로그 소스를 쉽게 수집할 수 있지만 데이터 레이크의 주요 개념 중 하나는 이 데이터에서 통찰력을 도출할 수 있도록 다양한 데이터 소스를 수집할 수 있다는 것입니다.

### 파트너 소스

타사 로그 소스를 수집하려면 먼저 지원되는 [Security Lake 소스 파트너](https://aws.amazon.com/security-lake/partners/#Source_Partners)인지 확인하십시오. Security Lake 소스 파트너는 이미 솔루션에서 OCSF 형식으로 Security Lake로 로그 또는 결과를 보낼 수 있는 통합을 구축했습니다. 이 통합의 세부 사항은 파트너마다 다르지만 모든 파트너 통합 문서에 대한 링크는 [여기](https://docs.aws.amazon.com/security-lake/latest/userguide/integrations-third-party.html)에서 찾을 수 있습니다. 파트너가 게시한 해당 지침을 따르십시오.

### Security Hub를 사용하여 결과를 Security Lake로 라우팅

Security Hub는 광범위한 타사 소스에서 보안 결과 수집을 지원합니다. 그런 다음 이러한 결과를 Security Lake로 보낼 수 있습니다. 타사 보안 소스에서 Security Lake로 보안 데이터를 보내려고 하는데 소스 파트너가 아닌 경우 이 [목록](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-partner-providers.html)을 검토하여 Security Hub 통합이 있는지 확인하는 것이 좋습니다.

### 사용자 지정 소스

Amazon Security Lake에서 사용자 지정 소스를 사용하여 모든 소스에서 로그 또는 결과를 수집할 수 있습니다. 예를 들어 온프레미스 DNS 서버, SaaS 애플리케이션, 사용자 지정 애플리케이션 또는 다른 클라우드 제공업체에서 로그를 수집할 수 있습니다. 이러한 유형의 로그 소스를 Security Lake로 가져오기 전에 OCSF로 필요한 변환을 담당합니다. [Glue](https://aws.amazon.com/glue/), [Data Pipeline](https://docs.aws.amazon.com/datapipeline/latest/DeveloperGuide/what-is-datapipeline.html) 및 [Kinesis](https://aws.amazon.com/kinesis/)와 같은 AWS 서비스는 사용자 지정 로그 소스를 OCSF로 변환하는 데 도움이 될 수 있습니다. [AWS AppFabric](https://aws.amazon.com/appfabric/)은 [지원되는 SaaS 애플리케이션](https://aws.amazon.com/appfabric/supported-applications/)의 로그를 OCSF로 변환하여 도움을 줄 수도 있습니다. AppFabric과 Security Lake의 통합에 대한 자세한 내용은 AppFabric [문서](https://docs.aws.amazon.com/appfabric/latest/adminguide/security-lake.html#security-lake-audit-log-ingestion)를 참조하십시오.

Security Lake에서 사용자 지정 소스를 생성하는 방법을 이해하려면 [Security Lake 문서](https://docs.aws.amazon.com/security-lake/latest/userguide/custom-sources.html)를 참조하십시오. 사용자 지정 소스에서 데이터를 수집하는 방법에 대한 패턴은 이 [Security Lake 블로그 게시물](https://aws.amazon.com/blogs/security/get-custom-data-into-amazon-security-lake-through-ingesting-azure-activity-logs/)을 참조하는 것이 좋습니다.

## Security Lake 운영화

보안 로그가 중앙에서 수집되고 OCSF 형식으로 정규화되면 자연스럽게 수집하는 데이터에서 통찰력과 가치를 도출하고 싶을 것입니다. 이는 AWS 기본 분석 솔루션을 사용하거나 Security Lake 구독자 파트너 솔루션을 사용하여 수행할 수 있습니다.

### AWS 기본 분석

Security Lake 데이터는 궁극적으로 S3에 상주하므로 활용할 수 있는 광범위한 AWS 기본 분석 솔루션이 있습니다. Security Lake와 함께 사용하기 위해 고려하는 것이 좋은 기본 솔루션은 [Athena](https://aws.amazon.com/athena/), [QuickSight](https://aws.amazon.com/quicksight/?amazon-quicksight-whats-new.sort-by=item.additionalFields.postDateTime&amazon-quicksight-whats-new.sort-order=desc) 및 [OpenSearch](https://aws.amazon.com/opensearch-service/)입니다.

Athena를 사용하면 Security Lake 데이터에 대해 SQL 쿼리를 실행할 수 있습니다. Security Lake가 배포되면 기본적으로 Athena와 통합됩니다. Athena를 사용한 적이 없는 경우 Athena가 쿼리 결과를 쓸 S3 버킷을 구성해야 합니다. 이 작업을 수행하는 데 도움이 필요하면 Athena [문서](https://docs.aws.amazon.com/athena/latest/ug/query-results-specify-location-console.html)를 참조하십시오. Security Lake 데이터에 대해 실행할 수 있는 쿼리 예제와 추가 지침은 [여기](https://docs.aws.amazon.com/security-lake/latest/userguide/subscriber-query-examples.html)에서 찾을 수 있습니다. Athena는 자주 액세스하지 않는 보안 데이터에 권장되는 분석 접근 방식입니다. 또한 보안 팀이 임시 조사 및 보고서를 실행할 수 있도록 합니다. 경우에 따라 Athena를 사용하면 고객이 다운스트림의 더 비용이 많이 드는 분석 솔루션에서 보존 기간을 단축하거나 거의 사용되지 않는 로그 전송을 완전히 중단할 수 있습니다.

QuickSight를 사용하면 Security Lake 데이터를 기반으로 대시보드 및 보고서를 생성하고 이해 관계자와 공유할 수 있습니다. 이는 Security Lake와 함께 사용할 비용 효율적인 보고 기능을 찾는 고객에게 권장되는 접근 방식입니다. QuickSight는 또한 [Amazon Q in QuickSight](https://aws.amazon.com/quicksight/q/)를 통해 생성형 AI 기능을 제공하여 사용자가 자연어를 사용하여 보안 데이터와 상호 작용할 수 있도록 합니다. Security Lake와 함께 QuickSight를 사용하는 데 관심이 있는 경우 사전 구축된 AWS 솔루션 "[Security Insights on AWS](https://aws.amazon.com/solutions/implementations/security-insights-on-aws/)"를 평가하는 것이 좋습니다.

OpenSearch는 AWS에서 관리형 서비스로 배포할 수 있는 오픈 소스 검색 및 분석 제품군입니다. OpenSearch는 Security Lake용 수집 파이프라인과 Security Lake와의 제로 ETL 통합을 지원합니다. 수집 파이프라인 접근 방식은 거의 실시간 검색 및 분석 기능을 찾거나 Security Lake 데이터를 기반으로 한 경고가 필요한 Security Lake 고객에게 권장됩니다. 제로 ETL 접근 방식은 추가 단계와 비용 없이 Security Lake 데이터를 쿼리하려는 고객에게 권장됩니다. 제로 ETL 접근 방식은 특히 인덱싱된 뷰 및 대시보드와 같은 OpenSearch 기능을 사용하여 비용 효율성과 성능 간의 균형을 제공합니다. 두 배포 옵션 모두 다음 블로그 게시물에 문서화되어 있습니다: [수집 파이프라인 접근 방식](https://aws.amazon.com/blogs/big-data/generate-security-insights-from-amazon-security-lake-data-using-amazon-opensearch-ingestion/) 및 [제로 ETL 접근 방식](https://aws.amazon.com/blogs/aws/introducing-amazon-opensearch-service-zero-etl-integration-for-amazon-security-lake/).

### 구독자 파트너

[구독자 파트너](https://aws.amazon.com/security-lake/partners/#Subscriber_Partners)는 Security Lake 데이터를 활용하는 추가 방법을 제공합니다. 많은 인기 있는 SIEM, SOAR, XDR 및 보안 분석 도구가 Security Lake 파트너입니다. 즉, Security Lake 배포에서 데이터를 수집할 수 있습니다. 이러한 파트너는 Security Lake 데이터로 수행할 수 있는 작업을 확장하는 데 중요한 역할을 합니다. Security Lake는 또한 이러한 보안 분석 솔루션의 구현 및 관리를 단순화할 수 있습니다. 고객은 Security Lake를 관리형 솔루션으로 사용하여 기본 AWS 로그를 수집하고 하나의 계정 및 리전으로 집계한 다음 다운스트림 분석 솔루션과 하나의 통합 지점을 가질 수 있습니다.

Security Lake 구독자에는 두 가지 액세스 모델이 있습니다. 로그를 파트너 솔루션으로 보낼 수 있는 데이터 액세스와 파트너 솔루션이 Security Lake 데이터에 대해 쿼리를 실행하고 결과를 반환할 수만 있는 쿼리 액세스입니다. 두 유형의 Security Lake 구독자를 구성할 때 구독자가 액세스할 수 있는 로그 소스를 선택할 수 있습니다. 구독자 구성 및 액세스 관리에 대한 추가 세부 정보는 Security Lake [문서](https://docs.aws.amazon.com/security-lake/latest/userguide/subscriber-management.html)를 참조하십시오.

구독자의 사용 사례는 상관 관계 또는 대응 및 수정을 위해 특정 로그 데이터를 SIEM 또는 SOAR 솔루션으로 보내는 것일 수 있습니다. 또 다른 예는 개발 팀이 별도의 개별 계정 CloudTrail 추적을 생성할 필요 없이 계정의 문제를 해결하기 위해 CloudTrail 데이터에 액세스할 수 있도록 하는 교차 계정 구독자를 생성하는 것입니다. [SageMaker를 사용하여 기계 학습 분석 실행](https://github.com/aws-samples/amazon-security-lake-machine-learning) 또는 보안 로그에서 가치를 도출하는 데 도움이 될 수 있는 긴 [타사 통합](https://docs.aws.amazon.com/security-lake/latest/userguide/integrations-third-party.html) 목록과 통합하는 것과 같은 다른 많은 사용 사례도 있습니다. 각 구독자 파트너는 또한 솔루션과 Security Lake 간의 통합을 올바르게 설정하는 방법에 대한 지침을 제공합니다. 이러한 통합에 대한 지침은 [여기](https://docs.aws.amazon.com/security-lake/latest/userguide/integrations-third-party.html)에서 제공됩니다. 그러나 모든 구독자는 아래 예와 같이 구독자 페이지의 Security Lake 콘솔 내에서 일부 구성이 필요합니다.

![image19](../../images/security-lake/Image19.png)
![image20](../../images/security-lake/Image20.png)
![image21](../../images/security-lake/Image21.png)

### OCSF 스키마 업데이트

Security Lake는 새로운 버전의 OCSF가 릴리스됨에 따라 주기적으로 스키마 업데이트를 거칩니다. 이 경우 Security Lake 배포를 새 스키마 버전으로 업데이트할 시기를 선택할 수 있습니다. 그러나 파트너는 릴리스되는 즉시 새 스키마를 지원하지 않을 수 있습니다. 업데이트하기 전에 새 스키마 버전에 대한 파트너 지원을 확인하는 것이 좋습니다.

## 비용 고려 사항

Security Lake를 사용하면 선불 비용 없이 사용한 만큼만 지불합니다. Security Lake 가격은 데이터 수집 및 데이터 정규화의 두 가지 차원을 기반으로 합니다. 월별 비용은 AWS 서비스에서 수집된 로그 및 이벤트 데이터의 기가바이트당 볼륨에 따라 결정됩니다. 타사 또는 자체 데이터를 가져오는 데는 요금이 부과되지 않습니다. 가격에 대한 자세한 내용은 [Security Lake 가격 페이지](https://aws.amazon.com/security-lake/pricing/)를 참조하십시오. 이 섹션에서는 중요한 비용 고려 사항을 강조하고자 합니다.

### 비용 분석 및 가시성

Security Lake는 데이터 레이크를 성공적으로 관리하고 오케스트레이션하기 위해 사용자를 대신하여 다른 서비스를 프로비저닝합니다. 여기에는 S3, Lake Formation, Glue, Lambda, EventBridge, Kinesis 및 SQS가 포함됩니다. 이러한 각 서비스는 각각의 가격 구조에 따라 독립적으로 비용을 생성합니다. 생성된 비용은 개별 서비스에 보고되며 Security Lake 가격 구조에 포함되지 않거나 Cost Explorer 또는 Security Lake 콘솔의 사용량 페이지와 같은 도구에서 Security Lake에 연결된 요금으로 다시 보고되지 않습니다. Security Lake와 관련된 오케스트레이션 비용의 대부분은 Security Lake를 실행하도록 선택한 위임된 관리자 계정과 연결됩니다.

Security Lake의 가격 구조에는 AWS 로그 소스의 수집 및 정규화에 대한 요금이 포함됩니다. 이러한 요금은 Cost Explorer와 같은 도구에서 Security Lake와 연결되며 Security Lake 콘솔의 사용량 페이지에 표시됩니다. 이러한 요금은 로그가 시작된 계정과 연결됩니다. 즉, 대부분의 경우 Security Lake와 직접 연결된 요금의 대부분은 구성원 계정에 연결됩니다.

### 내장된 비용 절감

Security Lake로 모든 로그를 수집하면 환경의 S3 버킷에 모든 로그를 저장하게 됩니다. 고객은 이러한 로그에 사용할 S3 스토리지 계층을 선택할 수 있으며 $0.023/GB의 표준 스토리지부터 $0.00099/GB의 딥 아카이브까지 모든 것을 선택할 수 있습니다. 이러한 유연성 외에도 Security Lake는 이러한 로그를 사전 텍스트 원시 로그 볼륨의 90% 이상 압축 비율을 허용하는 Apache Parquet 파일로 저장합니다.

### 중복 기본 로그를 수집하지 않는지 확인

Security Lake를 활성화해도 기존 로그 수집 설정은 변경되지 않습니다. 즉, 이미 수집하고 있던 기본 로그 소스를 수집하도록 Security Lake를 활성화하면 동일한 데이터의 복사본을 두 개 이상 수집하기 시작합니다(로그 형식은 다르지만). Security Lake를 활성화한 후 이전 로그 수집을 비활성화할 수 있습니다. 그러나 그렇게 하기 전에 이전 로그를 사용하는 내부 이해 관계자 또는 워크로드를 식별하고 Security Lake 사용으로 전환하는 방법을 계획하는 것이 좋습니다.

### CloudTrail 관리 이벤트 비용 절감

CloudTrail 관리 이벤트를 수집하려면 위에서 언급한 대로 CloudTrail 조직 추적을 생성해야 합니다. 이로 인해 CloudTrail이 선택한 버킷으로 로그를 보냅니다. 이러한 로그의 첫 번째 복사본 전달은 무료이지만 S3 스토리지 비용이 발생합니다. Security Lake는 CloudTrail 관리 이벤트의 복사본을 수집하고 Security Lake S3 버킷에 이러한 로그를 저장합니다. 이로 인해 CloudTrail 로그가 중복됩니다. CloudTrail 버킷에 [S3 수명 주기 정책](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)을 설정하여 짧은 기간 후에 로그 파일을 만료하는 것이 좋으며, 이는 조직에서 결정하는 내용에 따라 크게 달라집니다. 이러한 로그 파일은 동시에 전달되므로 만료 기간은 매우 짧을 수 있습니다.

### 롤업 리전

롤업 리전을 사용하는 경우 하나 이상의 리전 버킷에서 단일 리전의 버킷으로 데이터를 집계합니다. 이제 롤업 리전의 버킷에 복사본이 존재하므로 각 리전 버킷에서 로그가 중복됩니다. 일부 고객은 복원력을 위해 로그의 여러 복사본을 유지하는 것을 선호합니다. 그러나 비용 최적화를 위해 이전 섹션에서 논의한 CloudTrail 관리 이벤트의 중복과 동일한 방식으로 이 데이터 중복을 처리하는 것이 좋습니다. [S3 수명 주기 정책](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)을 생성하여 짧은 기간 후에 이러한 로그를 만료합니다.

### 로그를 Glacier로 이동

Glacier 스토리지 클래스는 S3 스토리지 비용을 줄일 수 있지만 보안 로그 데이터와 상호 작용하는 방법에 따라 신중하게 고려해야 하는 절충안이 있습니다. Glacier 스토리지 클래스는 자주 액세스할 필요가 없는 데이터의 장기 저장을 위해 설계되었습니다. 낮은 스토리지 비용에 대한 절충안으로 이러한 클래스에는 최소 보존 기간, 최소 객체 크기, 각 객체에 대한 추가 메타데이터, 느린 검색 시간 및 직접 검색 비용과 같은 것들이 있습니다. 전체 분석은 [S3 사용자 가이드](https://docs.aws.amazon.com/AmazonS3/latest/userguide/glacier-storage-classes.html)를 참조하십시오. Glacier 스토리지 클래스의 가격 구조를 완전히 이해하여 특정 사용 사례에 대해 스토리지 절감액이 추가 비용 차원보다 큰지 확인하십시오. 또한 필요할 때 데이터를 검색하는 방법을 고려하고 해당 지연이 조직에 허용되는지 확인해야 합니다.

## 추가 리소스

[문서: Amazon Security Lake 사용자 가이드](https://docs.aws.amazon.com/security-lake/latest/userguide/what-is-security-lake.html)

[문서: Amazon Security Lake API 참조](https://docs.aws.amazon.com/security-lake/latest/APIReference/Welcome.html)

[비디오: AWS re:Inforce 2023 - IPG를 활용한 Amazon Security Lake로 보안 데이터 레이크 구축](https://www.youtube.com/watch?v=qcXB6Y_7bNo&pp=ygUUYW1hem9uIHNlY3VyaXR5IGxha2U%3D)

[비디오: AWS Organizations로 Amazon Security Lake 시작 및 관리 방법](https://www.youtube.com/watch?v=fKGhscpwN-k&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=35&t=90s&pp=iAQB)

[비디오: Amazon Security Lake 비용 이해](https://www.youtube.com/watch?v=CbzLCntmgiY)

[비디오: Amazon Athena 및 Amazon QuickSight와 함께하는 Amazon Security Lake](https://www.youtube.com/watch?v=M0GviMezp3w&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=41&t=1s&pp=iAQB)

[비디오: Amazon Security Lake 사용자 지정 소스](https://www.youtube.com/watch?v=8MDP3LX2A-A&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=26&t=3s&pp=iAQB)

[비디오: Splunk와 Amazon Security Lake 통합](https://www.youtube.com/watch?v=VSwSwNFPz14&list=PLhr1KZpdzukfJzNDd8eCJH_TGg24ZTwP6&index=4&t=190s&pp=iAQB)

[블로그: Amazon Security Lake에서 사용자 지정 로그 소스를 소비하기 위한 패턴](https://aws.amazon.com/blogs/security/patterns-for-consuming-custom-log-sources-in-amazon-security-lake/)

[블로그: Azure 활동 로그 수집을 통해 Amazon Security Lake로 사용자 지정 데이터 가져오기](https://aws.amazon.com/blogs/security/get-custom-data-into-amazon-security-lake-through-ingesting-azure-activity-logs/)

[블로그: Amazon Security Lake에서 게시한 이벤트를 Amazon OpenSearch Service로 수집, 변환 및 전달](https://aws.amazon.com/blogs/big-data/ingest-transform-and-deliver-events-published-by-amazon-security-lake-to-amazon-opensearch-service/)

[블로그: Amazon Security Lake가 사전 위협 분석을 위한 보안 데이터 관리를 단순화하는 데 어떻게 도움이 되는지](https://aws.amazon.com/blogs/security/how-amazon-security-lake-is-helping-customers-simplify-security-data-management-for-proactive-threat-analysis/)

[블로그: Amazon Security Lake에서 로그를 수집하기 위한 Amazon OpenSearch 클러스터 배포 방법](https://aws.amazon.com/blogs/security/how-to-deploy-an-amazon-opensearch-cluster-to-ingest-logs-from-amazon-security-lake/)

[블로그: Amazon OpenSearch Ingestion을 사용하여 Amazon Security Lake 데이터에서 보안 통찰력 생성](https://aws.amazon.com/blogs/big-data/generate-security-insights-from-amazon-security-lake-data-using-amazon-opensearch-ingestion/)

[블로그: 보안 분석을 단순화하기 위한 Amazon OpenSearch Service와 Amazon Security Lake 통합 소개](https://aws.amazon.com/blogs/aws/introducing-amazon-opensearch-service-zero-etl-integration-for-amazon-security-lake/)

[OCSF Github 페이지](https://github.com/ocsf)
