# Certificate Services

## 소개
AWS Certificate Manager (ACM) 및 AWS Private CA 가이드에 오신 것을 환영합니다. 이 가이드의 목적은 안전하고 효과적인 공개 키 인프라(PKI)를 유지하는 데 사용되는 인증서를 생성하고 제어하기 위해 AWS Certificate Manager (ACM) 및 AWS Private Certificate Authority (AWS Private CA)를 활용하는 규범적 지침을 제공하는 것입니다. GitHub를 통해 이 가이드를 게시함으로써 서비스 개선 사항과 사용자 커뮤니티의 피드백을 포함한 시의적절한 권장 사항을 신속하게 반복할 수 있습니다. 이 가이드는 단일 계정에서 처음으로 인증서 시스템을 배포하는 경우든, 기존 다중 계정 인프라에서 인프라를 최적화하는 방법을 찾고 있는 경우든 가치를 제공하도록 설계되었습니다.

## 이 가이드 사용 방법

이 가이드는 AWS 계정(및 리소스) 내에서 인증서 작업을 담당하는 보안 실무자를 대상으로 합니다. 모범 사례는 더 쉬운 이해를 위해 다양한 카테고리로 구성되어 있습니다. 각 카테고리에는 간략한 개요로 시작하여 지침 구현을 위한 세부 단계가 이어지는 해당 모범 사례 세트가 포함되어 있습니다. 주제는 특정 순서로 읽을 필요가 없습니다:

* [AWS Certificate Manager (ACM)이란 무엇입니까?](#aws-certificate-manager-acm이란-무엇입니까)
* [ACM 사용의 이점은 무엇입니까?](#acm-사용의-이점은-무엇입니까)
* [ACM 시작하기](#acm-시작하기)
    * [인증서 고려사항](#인증서-고려사항)
    * [데이터 보호 고려사항](#데이터-보호-고려사항)
    * [AWS CloudFormation 고려사항](#aws-cloudformation-고려사항)
    * [AWS CloudTrail 활성화](#aws-cloudtrail-활성화)
* [ACM 구현](#acm-구현)
    * [배포 고려사항](#배포-고려사항)
    * [도메인 이름 고려사항](#도메인-이름-고려사항)
    * [ACM 인증서에 대한 액세스 제어](#acm-인증서에-대한-액세스-제어)
    * [인증서 피닝](#인증서-피닝)
    * [인증서 투명성 로깅 옵트아웃](#인증서-투명성-로깅-옵트아웃)
* [ACM 운영화](#acm-운영화)
    * [Amazon EventBridge를 사용한 만료 인증서 자동 알림 설정](#amazon-eventbridge를-사용한-만료-인증서-자동-알림-설정)
    * [CloudWatch 알람을 사용한 만료 인증서 자동 알림 설정](#cloudwatch-알람을-사용한-만료-인증서-자동-알림-설정)
    * [인증서 인벤토리 관리](#인증서-인벤토리-관리)
    * [규정 준수 및 감사](#규정-준수-및-감사)
    * [가져온 인증서의 관리형 갱신](#가져온-인증서의-관리형-갱신)
    * [갱신 상태 추적](#갱신-상태-추적)
* [AWS Private Certificate Authority (AWS Private CA)란 무엇입니까?](#aws-private-certificate-authority-aws-private-ca란-무엇입니까)
* [AWS Private CA 사용의 이점은 무엇입니까?](#aws-private-ca-사용의-이점은-무엇입니까)
* [AWS Private CA 시작하기](#aws-private-ca-시작하기)
    * [인증서 고려사항](#인증서-고려사항-1)
    * [데이터 보호 고려사항](#데이터-보호-고려사항-1)
    * [검증 고려사항](#검증-고려사항)
    * [도메인 이름 고려사항](#도메인-이름-고려사항-1)
    * [배포 고려사항](#배포-고려사항-1)
    * [가능한 경우 Root CA 사용 최소화](#가능한-경우-root-ca-사용-최소화)
* [AWS Private CA 구현](#aws-private-ca-구현)
    * [인증 기관 계층 설계](#인증-기관-계층-설계)
    * [키 알고리즘 및 길이 선택](#키-알고리즘-및-길이-선택)
    * [인증서 템플릿 구성](#인증서-템플릿-구성)
    * [CA 공유](#ca-공유)
    * [AWS 서비스와의 통합](#aws-서비스와의-통합)
    * [액세스 제어 및 IAM 정책 보안](#액세스-제어-및-iam-정책-보안)
    * [관리형 폐기](#관리형-폐기)
* [Kubernetes용 커넥터](#kubernetes용-커넥터)
* [SCEP용 커넥터](#scep용-커넥터)
* [Active Directory용 커넥터](#active-directory용-커넥터)
* [교차 계정 CA 공유](#교차-계정-ca-공유)
* [ACM과 AWS Private CA의 차이점은 무엇입니까?](#acm과-aws-private-ca의-차이점은-무엇입니까)
* [비용 고려사항](#비용-고려사항)
* [AWS Private CA 모범 사례 체크리스트](#aws-private-ca-모범-사례-체크리스트)
* [리소스](#리소스)

## AWS Certificate Manager (ACM)이란 무엇입니까?

ACM은 AWS 서비스와 함께 사용할 공개 및 프라이빗 SSL/TLS X.509 인증서의 프로비저닝, 저장 및 갱신 관리를 간소화하는 AWS 서비스입니다.

TLS는 전송 중인 데이터를 암호화하는 데 중요한 역할을 합니다. TLS의 보안은 암호화 및 디지털 서명을 기반으로 하며, 인증 기관이 기계의 신원을 증명하는 문서에 서명하고 TLS 연결을 시작하는 클라이언트가 이를 디지털로 검증합니다. 인증서 기반 서명은 인증 기관(CA)이 발급한 디지털 인증서를 사용하여 서명을 특정 신원에 연결하고 서명자의 신원을 검증합니다. ACM은 두 가지 경로를 통해 인증서를 발급합니다: 인터넷 도메인용 Amazon의 공개 인증 기관의 공개 인증서와 내부 리소스용 AWS Private CA를 통한 프라이빗 인증서입니다.

![ACM Overview](../../images/ACM-overview.png)
*그림 1: ACM 개요*

ACM은 인증서 및 관련 키 관리와 관련된 암호화 복잡성을 간소화합니다. ACM을 통해 인증서를 중앙에서 관리하고, 네이티브 AWS 서비스와 원활하게 통합하며, 인증서의 수명 주기를 자동화할 수 있습니다.

## ACM 사용의 이점은 무엇입니까?

ACM은 AWS Management Console, AWS CLI 또는 API를 통한 중앙 집중식 제어를 활성화하여 공개 및 프라이빗 SSL/TLS 인증서의 관리 및 배포를 간소화합니다. 인증서 사용은 감사 및 거버넌스 목적으로 AWS CloudTrail 로그를 통해 모니터링할 수 있습니다.

이러한 TLS 인증서는 Elastic Load Balancer, Amazon CloudFront 배포 및 Amazon API Gateway의 API를 포함한 다양한 AWS 서비스와 함께 사용하기 위해 추가 비용 없이 발급될 수 있습니다. 또한 타사 CA의 SSL/TLS 인증서를 가져올 수도 있습니다.

ACM은 Amazon에서 발급한 모든 SSL/TLS 인증서에 대해 관리형 갱신을 제공합니다. DNS 검증을 사용하는 인증서는 자동으로 갱신됩니다. Private CA IssueCertificate API 또는 가져온 인증서에서 발급한 프라이빗 인증서의 경우 사용자는 자동 만료 알림을 구현해야 합니다.

## ACM 시작하기

### 인증서 고려사항

ACM은 세 가지 유형의 SSL/TLS 인증서를 관리합니다: 공개(도메인 또는 이메일 검증), 프라이빗 및 가져온 인증서입니다. 사용자는 공개적으로 신뢰할 수 있는 인증서를 요청하거나, 타사 인증서를 가져오거나, AWS Private CA에서 호스팅되는 고객 소유 CA에서 발급한 프라이빗 인증서를 요청할 수 있습니다. ACM 인증서는 Elastic Load Balancing, CloudFront 및 API Gateway와 같은 통합 AWS 서비스와만 호환됩니다. 전체 목록은 [ACM과 통합된 서비스](https://docs.aws.amazon.com/acm/latest/userguide/acm-services.html)를 참조하십시오.

ACM에서 요청한 인증서의 유효 기간은 13개월로 고정되어 있으며 공개 또는 프라이빗 인증서 모두에 대해 수정할 수 없습니다. 조직에서 다른 유효 기간이 필요한 경우 타사 CA 공급업체에서 인증서를 가져오는 것을 고려해야 합니다.

## 데이터 보호 고려사항

ACM에서 공개 인증서를 요청하면 서비스가 사용자를 대신하여 공개/프라이빗 키 쌍을 생성합니다. 공개 키는 인증서의 일부가 되고 ACM은 KMS를 사용하여 암호화 컨텍스트로 프라이빗 키를 암호화합니다. 그런 다음 ACM은 인증서와 해당 암호화된 프라이빗 키를 저장합니다. AWS는 동일한 KMS 키를 사용하여 특정 AWS 계정 및 리전의 모든 인증서에 대한 프라이빗 키를 암호화하며, 모든 KMS 호출은 ACM에서 내부적으로 처리됩니다.

대칭 암호화 키를 사용한 KMS 암호화 작업은 암호화 컨텍스트(데이터에 대한 추가 컨텍스트 정보를 포함하는 비밀이 아닌 키-값 쌍의 선택적 세트)를 허용합니다. 이 컨텍스트는 KMS Encrypt 작업에 삽입되어 API 복호화 호출의 감사 가능성을 향상시킬 수 있습니다. ACM은 암호화되는 프라이빗 키의 인증서 ARN, AWS 리전 및 계정 ID를 포함하는 값으로 컨텍스트를 자동으로 설정하며, 이는 암호문에 암호화 방식으로 바인딩됩니다. 모든 ACM 인증서에 대한 고유한 암호화 컨텍스트는 인증서 간에 논리적 격리를 생성합니다.

ACM 공개 인증서에 대한 중요한 고려사항은 다음과 같습니다:

* KMS 작업에 직접 액세스할 수 없습니다
* ACM이 KMS를 호출하는 방법을 수정할 수 없습니다
* 암호화 컨텍스트를 수정할 수 없습니다
* 공개 인증서의 프라이빗 키를 내보낼 수 없습니다

### AWS CloudFormation 고려사항

AWS CloudFormation을 사용하면 사용하려는 AWS 리소스를 설명하는 템플릿을 만들 수 있습니다. 그런 다음 AWS CloudFormation이 해당 리소스를 프로비저닝하고 구성합니다. AWS CloudFormation은 Elastic Load Balancing, Amazon CloudFront 및 Amazon API Gateway와 같은 ACM에서 지원하는 리소스를 프로비저닝할 수 있습니다.

빠른 개발 주기에는 종종 빠른 반복이 필요합니다. AWS CloudFormation을 사용하여 여러 테스트 환경을 빠르게 생성하고 삭제하는 경우 각 환경에 대해 별도의 ACM 인증서를 생성하지 않는 것이 좋습니다. 그렇게 하면 인증서 할당량이 소진될 수 있습니다. 대신 테스트에 사용하는 모든 도메인 이름을 포함하는 와일드카드 인증서를 생성하십시오. 예를 들어 버전 번호만 다른 도메인 이름(예: \\<version>.service.example.com)에 대해 ACM 인증서를 반복적으로 생성하는 경우 대신 <*>.service.example.com에 대한 단일 와일드카드 인증서를 생성하십시오. AWS CloudFormation이 테스트 환경을 생성하는 데 사용하는 템플릿에 와일드카드 인증서를 포함하십시오.

AWS CloudFormation을 사용하여 인증서를 요청할 때 도메인 검증은 특정 조건에서만 처리됩니다: 도메인이 Amazon Route 53에서 호스팅되고, AWS 계정에 있으며, DNS 검증을 사용해야 합니다. 이러한 조건이 충족되지 않으면 CloudFormation 스택은 CREATE_IN_PROGRESS 상태로 유지되어 추가 스택 작업이 지연됩니다.

### AWS CloudTrail 활성화

ACM 사용을 시작하기 전에 CloudTrail 로깅을 활성화하십시오. CloudTrail을 사용하면 AWS Management Console, AWS SDK, AWS Command Line Interface 및 상위 수준 Amazon Web Services를 통해 수행된 API 호출을 포함하여 계정에 대한 AWS API 호출 기록을 검색하여 AWS 배포를 모니터링할 수 있습니다. 또한 ACM API를 호출한 사용자 및 계정, 호출이 이루어진 소스 IP 주소 및 호출이 발생한 시기를 식별할 수 있습니다. API를 사용하여 CloudTrail을 애플리케이션에 통합하고, 조직의 추적 생성을 자동화하고, 추적 상태를 확인하고, 관리자가 CloudTrail 로깅을 켜고 끄는 방법을 제어할 수 있습니다. CloudTrail은 또한 [조직에서 만료되는 TLS 인증서에 대해 알리거나 조치를 취하는 이벤트 기반 워크플로를 활성화](https://aws.amazon.com/blogs/security/how-to-manage-certificate-lifecycles-using-acm-event-driven-workflows/)하여 인증서 수명 주기 관리를 간소화합니다.

자세한 내용은 [추적 생성](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html)을 참조하십시오. [AWS Certificate Manager와 함께 CloudTrail 사용](https://docs.aws.amazon.com/acm/latest/userguide/cloudtrail.html)으로 이동하여 ACM 작업에 대한 예제 추적을 볼 수도 있습니다.

## ACM 구현

### 배포 고려사항

ACM은 리전 서비스입니다. 즉, ACM 인증서는 사용될 AWS 리소스와 동일한 리전에서 발급되어야 하며 AWS 계정 간에 공유할 수 없습니다. Amazon CloudFront에 대한 중요한 예외가 있습니다: CloudFront용 모든 ACM 인증서는 미국 동부(버지니아 북부) 리전(us-east-1)에서 발급되어야 합니다. CloudFront 배포와 연결되면 이러한 인증서는 구성된 모든 지리적 위치에 배포됩니다.

### 도메인 검증 고려사항

CA가 사이트에 대한 인증서를 발급하기 전에 ACM은 요청에 지정한 모든 도메인을 소유하거나 제어하는지 확인해야 합니다. 이메일 또는 DNS를 사용하여 검증을 수행할 수 있습니다.

![ACM Domain Validation](../../images/ACM-validation-1.png)
*그림 2: ACM 도메인 검증*

Amazon은 이메일 검증보다 DNS 검증을 사용할 것을 권장합니다. DNS 검증은 몇 가지 주요 이점을 제공합니다:

* Route 53과의 직접 통합을 통해 ACM을 통해 레코드를 업데이트할 수 있습니다
* DNS 레코드가 그대로 유지되고 인증서가 적극적으로 사용되는 한 ACM은 DNS 검증 인증서를 자동으로 갱신합니다
* 갱신을 위해 수동 개입이 필요하지 않습니다

이메일 검증 인증서는 다른 접근 방식이 필요합니다. 갱신 알림은 만료 45일 전부터 요청된 도메인에 대한 5개의 일반적인 시스템 주소(예: admin 또는 postmaster)로 전송됩니다. 이러한 인증서는 도메인 소유자의 수동 승인이 필요합니다. 동일한 ARN을 가진 갱신된 인증서가 발급되려면 나열된 모든 도메인을 검증해야 하며, 이메일 구성이 올바르지 않으면 잠재적인 오류나 지연이 발생할 수 있습니다.

### 도메인 이름 고려사항

ACM 인증서는 단일 도메인 이름, 여러 특정 도메인 이름, 와일드카드 도메인(무제한 수의 하위 도메인 보호) 또는 이들의 조합을 보호할 수 있습니다. 생성된 후에는 도메인 이름을 추가하거나 제거하여 인증서를 수정할 수 없습니다. 변경하려면 전체 수정된 도메인 이름 목록으로 새 인증서를 요청하고 이전에 검증된 이름을 포함하여 모든 도메인의 소유권을 검증해야 합니다.

기존 ACM 인증서에서 도메인 이름을 추가하거나 제거할 수 없습니다. 대신 수정된 도메인 이름 목록으로 새 인증서를 요청해야 합니다. 예를 들어 인증서에 5개의 도메인 이름이 있고 4개를 더 추가하려는 경우 9개의 도메인 이름이 모두 포함된 새 인증서를 요청해야 합니다. 모든 새 인증서와 마찬가지로 원래 인증서에 대해 이전에 검증한 이름을 포함하여 요청의 모든 도메인 이름에 대한 소유권을 검증해야 합니다.

이메일 검증을 사용하는 경우 각 도메인에 대해 최대 8개의 검증 이메일 메시지를 받으며, 그 중 최소 1개는 72시간 이내에 조치를 취해야 합니다. 예를 들어 5개의 도메인 이름이 있는 인증서를 요청하면 최대 40개의 검증 메시지를 받으며, 그 중 최소 5개는 72시간 이내에 조치를 취해야 합니다. 인증서 요청의 도메인 이름 수가 증가함에 따라 이메일을 사용하여 도메인 소유권을 검증하는 데 필요한 작업도 증가합니다.

대신 DNS 검증을 사용하는 경우 검증하려는 FQDN에 대한 데이터베이스에 하나의 새 DNS 레코드를 작성해야 합니다. ACM은 생성할 레코드를 보내고 나중에 데이터베이스를 쿼리하여 레코드가 추가되었는지 확인합니다. 레코드를 추가하면 도메인을 소유하거나 제어한다고 주장합니다. 앞의 예에서 5개의 도메인 이름이 있는 인증서를 요청하는 경우 5개의 DNS 레코드를 생성해야 합니다. 가능한 경우 DNS 검증을 사용하는 것이 좋습니다.

### ACM 인증서에 대한 액세스 제어

KMS 또는 Secrets Manager와 같은 다른 데이터 보호 서비스와 달리 ACM은 리소스 정책 또는 리소스 제어 정책(RCP)을 지원하지 않습니다. 대신 ACM 인증서에 대한 액세스는 ID 기반 IAM 정책 또는 AWS Organizations를 사용하는 경우 서비스 제어 정책(SCP)을 통해 제어할 수 있습니다.

정책에서 계정 수준 분리를 사용하여 계정 수준에서 인증서에 액세스할 수 있는 사람을 제어하십시오. 프로덕션 인증서를 테스트 및 개발 인증서와 별도의 계정에 보관하십시오. 계정 수준 분리를 사용할 수 없는 경우 인증서 프라이빗 키를 보호하는 KMS 키에 대한 정책에서 kms:CreateGrant 작업을 거부하여 특정 역할에 대한 액세스를 제한할 수 있습니다. 이는 계정의 어떤 역할이 높은 수준에서 인증서에 서명하거나 사용할 수 있는지 제한합니다. 권한 부여 용어를 포함한 권한 부여에 대한 자세한 내용은 *AWS Key Management Service 개발자 가이드*의 [AWS KMS의 권한 부여](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html)를 참조하십시오.

계정별로 `kms:CreateGrant` 사용을 제한하는 것보다 더 세분화된 제어를 원하는 경우 `kms:EncryptionContext` 조건 키를 사용하여 특정 인증서로 `kms:CreateGrant`를 제한할 수 있습니다. 키로 `arn:aws:acm`을 지정하고 제한할 ARN의 값을 지정합니다. 다음 예제 정책은 특정 인증서의 사용을 방지하지만 다른 인증서는 허용합니다.

```
{
   "Version": "2012-10-17",
   "Statement": [
       {
           "Sid": "VisualEditor0",
           "Effect": "Deny",
           "Action": "kms:CreateGrant",
           "Resource": "*",
           "Condition": {
               "StringEquals": {
                   "kms:EncryptionContext:aws:acm:arn": "arn:aws:acm:us-east-1:111122223333:certificate/[certificate-id-here]"
               }
           }
       }
   ]
}
```

### 인증서 피닝

인증서 피닝(또는 SSL 피닝)은 인증서 계층 구조 대신 원격 호스트를 X.509 인증서 또는 공개 키와 직접 연결하여 원격 호스트를 검증하기 위해 애플리케이션에서 사용할 수 있는 프로세스로, SSL/TLS 인증서 체인 검증을 우회합니다. 루트 CA에서 아래로 전체 인증서 체인을 검증하는 대신 애플리케이션은 해당 호스트에 대해 미리 정의된 인증서 또는 키만 신뢰합니다. 이 신뢰할 수 있는 인증서는 개발 중에 포함되거나 호스트에 대한 첫 번째 연결 중에 캡처될 수 있습니다.

애플리케이션이 ACM 인증서를 피닝하지 않는 것이 좋습니다. ACM은 AWS Certificate Manager에서 [관리형 인증서 갱신](https://docs.aws.amazon.com/acm/latest/userguide/managed-renewal.html)을 수행하여 만료되기 전에 Amazon에서 발급한 SSL/TLS 인증서를 자동으로 갱신합니다. 인증서를 갱신하기 위해 ACM은 새 공개-프라이빗 키 쌍을 생성합니다. 애플리케이션이 ACM 인증서를 피닝하고 인증서가 새 공개 키로 성공적으로 갱신되면 애플리케이션이 도메인에 연결할 수 없을 수 있습니다.

인증서를 피닝하기로 결정한 경우 다음 옵션은 애플리케이션이 도메인에 연결하는 것을 방해하지 않습니다:

* ACM에 [자체 인증서를 가져온](https://docs.aws.amazon.com/acm/latest/userguide/managed-renewal.html) 다음 애플리케이션을 가져온 인증서에 피닝합니다. ACM은 가져온 인증서를 자동으로 갱신하려고 시도하지 않습니다.
* 공개 인증서를 사용하는 경우 애플리케이션을 사용 가능한 모든 [Amazon 루트 인증서](https://www.amazontrust.com/repository/)에 피닝합니다. 프라이빗 인증서를 사용하는 경우 애플리케이션을 CA의 루트 인증서에 피닝합니다.

### 인증서 투명성 로깅 옵트아웃

2018년 4월 30일부터 Google Chrome은 인증서 투명성 로그에 기록되지 않은 공개 SSL/TLS 인증서를 더 이상 신뢰하지 않습니다. 따라서 2018년 4월 24일부터 Amazon CA는 모든 새 인증서 및 갱신을 최소 두 개의 공개 로그에 게시하기 시작했습니다. 인증서가 로깅되면 제거할 수 없습니다.

로깅은 인증서를 요청하거나 인증서가 갱신될 때 자동으로 수행되지만 옵트아웃을 선택할 수 있습니다. 그렇게 하는 일반적인 이유는 보안 및 개인 정보 보호에 대한 우려입니다. 예를 들어 내부 호스트 도메인 이름을 로깅하면 잠재적인 공격자에게 그렇지 않으면 공개되지 않을 내부 네트워크에 대한 정보를 제공합니다. 또한 로깅은 새 제품 또는 출시되지 않은 제품 및 웹사이트의 이름을 유출할 수 있습니다.

인증서를 요청할 때 투명성 로깅을 옵트아웃하려면 [request-certificate](https://docs.aws.amazon.com/cli/latest/reference/acm/request-certificate.html) AWS CLI 명령 또는 [RequestCertificate](https://docs.aws.amazon.com/acm/latest/APIReference/API_RequestCertificate.html) API 작업의 options 파라미터를 사용하십시오. 인증서가 2018년 4월 24일 이전에 발급되었고 갱신 중에 로깅되지 않도록 하려면 [update-certificate-options](https://docs.aws.amazon.com/cli/latest/reference/acm/update-certificate-options.html) 명령 또는 [UpdateCertificateOptions](https://docs.aws.amazon.com/acm/latest/APIReference/API_UpdateCertificateOptions.html) API 작업을 사용하여 옵트아웃할 수 있습니다.

**제한 사항**

* 콘솔을 사용하여 투명성 로깅을 활성화하거나 비활성화할 수 없습니다.
* 인증서가 갱신 기간(일반적으로 인증서 만료 60일 전)에 들어간 후에는 로깅 상태를 변경할 수 없습니다. 상태 변경이 실패하면 오류 메시지가 생성되지 않습니다.

인증서가 로깅되면 로그에서 제거할 수 없습니다. 그 시점에서 옵트아웃해도 효과가 없습니다. 인증서를 요청할 때 로깅을 옵트아웃한 다음 나중에 다시 옵트인하기로 선택하면 인증서가 갱신될 때까지 로깅되지 않습니다. 인증서를 즉시 로깅하려면 새 인증서를 발급하는 것이 좋습니다.

다음 예제는 [request-certificate](https://docs.aws.amazon.com/cli/latest/reference/acm/request-certificate.html) 명령을 사용하여 새 인증서를 요청할 때 인증서 투명성을 비활성화하는 방법을 보여줍니다.

```
aws acm request-certificate \
--domain-name www.example.com \
--validation-method DNS \
--options CertificateTransparencyLoggingPreference=DISABLED \
```

앞의 명령은 새 인증서의 ARN을 출력합니다.

```
{ 
    "CertificateArn": "arn:aws:acm:region:account:certificate/certificate_ID" 
}
```

이미 인증서가 있고 갱신될 때 로깅되지 않도록 하려면 [update-certificate-options](https://docs.aws.amazon.com/cli/latest/reference/acm/update-certificate-options.html) 명령을 사용하십시오. 이 명령은 값을 반환하지 않습니다.

```
aws acm update-certificate-options \
--certificate-arn arn:aws:acm:region:account:\
certificate/certificate_ID \
--options CertificateTransparencyLoggingPreference=DISABLED
```

## ACM 운영화

### Amazon EventBridge를 사용한 만료 인증서 자동 알림 설정

ACM은 Private CA Issue Certificates API 또는 가져온 인증서를 사용하여 발급된 인증서에 대한 갱신을 관리하거나 만료 알림을 보내지 않습니다. 대신 ACM Certificate Approaching Expiration 이벤트를 활용하여 만료 이벤트를 알릴 수 있습니다.

1. 관련 표준 **SNS** 주제를 생성합니다.
2. **EventBridge** 콘솔을 엽니다. 탐색 창에서 **Buses**를 선택한 다음 **Rules**를 선택합니다.
3. **Create Rule**을 클릭합니다.
4. 규칙 이름을 입력합니다. 다른 모든 설정은 기본값으로 둡니다.

![EventBridge Rule Detail](../../images/ACM-eventbridge-1.png)
*그림 3: Eventbridge 규칙 세부 정보 정의*

4. **Event Pattern**으로 스크롤합니다. **AWS Service**에서 **Certificate Manager**를 선택합니다.
5. **Event Type**에서 **ACM Certificate Approaching Expiration**을 선택합니다.

![EventBridge Event Pattern](../../images/ACM-eventbridge-2.png)
*그림 4: Eventbridge 이벤트 패턴 정의*

6. **Next**를 클릭합니다.
7. 1단계에서 생성한 SNS 주제를 선택합니다.

![Eventbridge Target](../../images/ACM-eventbridge-3.png)
*그림 5: Eventbridge 대상 지정*

8. **Next**를 클릭한 다음 5단계로 건너뜁니다.
9. 선택 사항을 검토한 다음 **Create rule**을 클릭합니다.

이제 규칙이 인증서가 만료에 가까워지면 SNS 주제를 통해 알립니다. ACM은 만료 45일 전부터 모든 활성 인증서(공개, 프라이빗 및 가져온)에 대해 매일 만료 이벤트를 보냅니다. 만료 알림 타이밍을 변경하려면 ACM API의 `PutAccountConfiguration` 작업을 사용하여 **DaysBeforeExpiry**에 대해 1-45일 사이의 값을 선택하십시오.

EventBridge 규칙은 인증서가 가져온 것인지, 프라이빗인지 또는 ACM에서 발급한 것인지에 관계없이 알립니다. 예를 들어 특정 인증서에 대해서만 알리는 보다 세분화된 알림을 원할 수 있습니다.

### CloudWatch 알람을 사용한 만료 인증서 자동 알림 설정

[CloudWatch 알람](https://docs.aws.amazon.com/acm/latest/userguide/cloudwatch-metrics.html)은 AWS 환경에서 인증서 만료를 모니터링하고 자동 알림을 통해 서비스 중단을 방지합니다. 이 서비스는 인증서가 만료 임계값에 가까워지면 이메일, SMS 또는 인시던트 관리 시스템을 통해 알림을 보냅니다.

AWS 콘솔에는 한 가지 특정 제한 사항이 있습니다: CloudWatch 알람은 알람 구성당 하나의 인증서 메트릭만 모니터링할 수 있습니다. 여러 인증서가 있는 환경의 경우 AWS CLI 또는 Infrastructure as Code (IaC) 도구를 사용하여 프로그래밍 방식으로 알람을 생성하고 관리하십시오.

확장 가능한 인증서 모니터링을 위한 두 가지 솔루션이 있습니다:

1. 정의된 일정에 따라 인증서 만료 확인을 실행하는 AWS Lambda 함수.
2. [AWS 계정 전체에서 인증서 수명 주기를 추적하는 AWS Config 규칙](https://repost.aws/knowledge-center/acm-certificate-expiration).

### 인증서 인벤토리 관리

조직은 [AWS Config 규칙](https://github.com/aws-samples/ACM-flag-CA-issuer)을 구현하여 인프라 전체에서 인증서 사용 및 배포 패턴을 적극적으로 추적할 수 있습니다. 구조화된 리소스 태깅 전략은 각 인증서에 특정 소유권 및 목적 메타데이터를 할당하여 명확한 책임과 추적을 가능하게 합니다. 조직은 일반적으로 AWS Organizations 및 AWS Config를 사용하여 자동화된 인벤토리 보고서를 생성하여 인증서 배포 전체에 대한 포괄적인 가시성을 제공합니다. 인증서-리소스 매핑의 정기적인 감사는 정확성을 보장하고 인증서 관리의 잠재적인 격차를 식별하는 데 도움이 됩니다.

### 규정 준수 및 감사

규정 준수 및 감사 프로세스는 모든 인증서 작업에 대한 AWS CloudTrail 로그를 통해 세부 기록을 유지하는 데 중점을 둡니다. 이러한 로그를 통해 자동화된 보고 시스템이 사용되지 않는 인증서를 식별하고 만료가 임박한 인증서를 추적하여 사전 예방적 인증서 관리를 지원할 수 있습니다. 조직은 규정 준수 검증 스크립트를 구현하여 내부 정책 및 규제 요구 사항에 대한 인증서 구성을 확인합니다. 정기적인 보안 검토는 인증서 정책 및 배포를 평가하여 보안 표준 및 규정 준수 의무와의 지속적인 일치를 보장합니다.

### 가져온 인증서의 관리형 갱신

AWS Config는 리소스 태그 및 수정 규칙을 사용하여 ACM에서 가져온 인증서의 갱신을 자동화합니다. 자동으로 갱신되는 AWS 발급 인증서와 달리 가져온 인증서는 갱신을 위해 이 특정 자동화가 필요합니다. 자동화 프로세스는 고정된 순서를 따릅니다: AWS Config 규칙이 만료되는 가져온 인증서를 식별하고 Lambda 함수를 트리거하며, 그런 다음 리소스 태그의 인증서 메타데이터를 사용하여 갱신을 실행합니다. 함수는 새 인증서를 검색하고 ACM으로 가져오며 관련 AWS 리소스를 자동으로 업데이트합니다. 이 솔루션은 수동 갱신을 제거하고 인증서 관련 중단을 방지하며 모든 인증서 유형에 걸쳐 관리를 표준화합니다. 구현에는 AWS Config 규칙, Lambda 함수 및 리소스 태그의 정확한 구성이 필요합니다.

### 갱신 상태 추적

AWS Config는 정의된 규칙 및 메타데이터 태그를 사용하여 ACM에서 가져온 인증서 갱신을 모니터링합니다. 각 인증서는 갱신 대기 중, 갱신 진행 중, 갱신 완료 또는 갱신 실패의 네 가지 상태 태그 중 하나를 유지합니다. Config 규칙은 상태 변경 시 EventBridge 알림을 트리거하여 갱신 활동에 대한 시기적절한 알림을 보장합니다.

조직은 AWS Config 대시보드 및 집계기 보고서를 통해 갱신 상태를 추적할 수 있으며, 시스템은 감사 목적으로 모든 갱신 작업을 기록합니다. 이 자동화는 인증서 수명 주기의 지속적인 모니터링을 보장하고 갱신 관련 중단을 방지합니다.

## AWS Private Certificate Authority (AWS Private CA)란 무엇입니까?

AWS Private CA는 조직을 위한 내부 공개 키 인프라(PKI)를 생성하고 유지하도록 설계된 고가용성 관리형 서비스입니다. 이 서비스는 프라이빗 인증 기관 운영과 관련된 초기 투자 및 지속적인 유지 관리 비용을 제거하면서 인증서 수명 주기 관리를 간소화합니다.

## AWS Private CA 사용의 이점은 무엇입니까?

### 가용성 및 확장성 향상

AWS Private CA의 가용성 및 확장성의 기본 이점은 [서비스 수준 계약(SLA)](https://aws.amazon.com/private-ca/sla/)을 통해 강력한 99.9% 가용성 보장을 제공합니다. Private CA는 온프레미스 PKI 인프라 관리와 관련된 복잡성과 오버헤드를 제거합니다. 서비스의 유연한 아키텍처는 비즈니스 요구가 증가함에 따라 원활한 확장을 지원하는 동시에 Active Directory, Kubernetes 및 MDM(Mobile Device Management)용 SCEP를 포함한 여러 커넥터 유형을 통해 다양한 배포 옵션을 제공합니다. 조직은 다양한 인증서 모드(단기 및 범용)를 활용하여 특정 보안 요구 사항에 맞출 수 있으며, 인증서 발급 및 관리에 대한 맞춤형 접근 방식을 가능하게 합니다.

### 보안 태세 강화

보안은 여러 보호 계층을 통해 강화됩니다. AWS Private CA는 FIPS 140-3 검증 하드웨어 보안 모듈(HSM)을 사용하여 CA 키를 보호하고 중요한 PKI 구성 요소에 대한 엔터프라이즈급 보안을 제공합니다. AWS Identity and Access Management (IAM)와의 통합은 인증 기관에 대한 세분화된 액세스 제어 및 권한 관리를 가능하게 합니다. 보안 대응 기능은 OCSP(Online Certificate Status Protocol) 및 CRL(Certificate Revocation List)을 포함한 효율적인 인증서 폐기 메커니즘을 통해 강화되어 조직이 잠재적인 인증서 손상에 신속하게 대응하고 강력한 보안 태세를 유지할 수 있습니다.

### 인증서 수명 주기 관리 간소화

인증서 수명 주기 관리는 자동화 및 통합을 통해 간소화됩니다. AWS Private CA는 AWS Certificate Manager와 원활하게 작동하여 AWS 및 비AWS 리소스 모두에 대한 인증서 발급, 갱신 및 폐기와 같은 중요한 작업을 자동화합니다. 이 자동화는 전통적으로 PKI 관리와 관련된 수동 오버헤드를 제거하여 IT 팀의 운영 부담을 줄입니다.

일상적인 PKI 관리 작업을 자동화함으로써 조직은 효율적인 인증서 운영을 유지하면서 전략적 보안 이니셔티브로 리소스를 리디렉션할 수 있습니다.

### 규정 준수 및 규제 준수 가속화

이 서비스는 중앙 집중식 관리 및 포괄적인 감사 기능을 통해 규정 준수 및 규제 준수를 가속화합니다. AWS CloudTrail 및 Amazon CloudWatch와의 통합은 통합된 가시성과 세부 감사 추적을 제공하여 업계 규정 및 표준 준수를 간소화합니다. AWS Private CA를 사용하면 조직이 엄격한 액세스 제어 및 보안 모범 사례를 유지하면서 여러 AWS 계정에서 CA를 쉽게 공유할 수 있습니다. 플랫폼의 사용자 지정 가능한 인증서 템플릿은 조직별 ID 및 데이터 보호 정책을 지원하여 웹 서비스에서 IoT 장치에 이르기까지 다양한 사용 사례를 수용하면서 비용을 최적화하고 보안 표준을 유지합니다.

## AWS Private CA 시작하기

### 인증서 고려사항

AWS Private CA를 사용하면 AWS에서 프라이빗 CA를 생성하고 운영할 수 있습니다. ACM의 공개 인증서와 달리 Private CA를 사용하면 프라이빗 인증서를 발급하기 위한 루트 및 하위 CA로 자체 PKI 계층을 설정할 수 있습니다. 이러한 인증서는 AWS 서비스 및 온프레미스 애플리케이션 모두에서 사용할 수 있습니다.

인증서 유형:

* Root CA - PKI 계층의 최상위 기관
* Subordinate CA - 루트 CA의 권한 하에 인증서를 발급합니다

![Certificate Types](../../images/PCA-hierarchy.png)
*그림 6: 간단한 3단계 CA 계층*

ACM은 인증서에 대해 고정된 13개월 유효 기간을 제공하는 반면 Private CA는 CA 및 발급된 인증서에 대한 유연한 구성을 제공합니다:

* Root CA 유효성에는 제한이 없습니다. Private CA의 루트 인증서 기본값은 10년입니다.
* Subordinate CA는 사용자 지정 유효 기간을 가질 수 있습니다. Private CA의 하위 CA 인증서 기본 유효 기간은 3년입니다.

Private CA에서 관리하는 인증서는 이를 발급한 CA의 유효 기간보다 짧거나 같은 유효 기간을 가져야 합니다.

### 데이터 보호 고려사항

AWS Private CA를 사용하여 프라이빗 인증 기관을 생성하면 서비스가 루트 또는 하위 CA에 대한 프라이빗/공개 키 쌍을 생성합니다. CloudTrail을 사용하여 CA와 관련된 모든 암호화 작업을 모니터링하고 감사할 수 있습니다.

### 검증 고려사항

AWS Private CA는 내부 인증서용으로 설계되었으므로 공개 도메인 검증에 의존하지 않고 발급 정책 및 검증 방법을 제어합니다. CA 구성에서 인증서 주체 및 관련 확장 또는 제약 조건을 정의합니다. 인증서 템플릿(허용된 구성 정의), 사용자 지정 검증 규칙 및 IAM 정책을 통해 구현된 액세스 제어에 의존하는 자체 검증 프로세스를 구현할 수 있습니다. 조직은 사용자 지정 인증서 확장, 키 사용 제한 및 명명 규칙을 포함하여 자체 검증 기준을 정의할 수 있습니다. 신뢰는 다음을 통해 설정됩니다:

* **IAM 정책**: 인증서를 발급, 폐기 또는 관리할 수 있는 사람을 제어합니다. [IAM 조건 컨텍스트 키를 사용하여 승인된 계정 및 사용자가 특정 도메인에 대한 인증서를 요청하도록 허용하거나 거부](https://aws.amazon.com/blogs/security/how-to-use-aws-certificate-manager-to-enforce-certificate-issuance-controls/)할 수도 있습니다.
* **인증서 템플릿**: 인증서 파라미터(예: 키 사용, 확장 키 사용)를 적용하는 사전 정의 또는 사용자 지정 템플릿.
* **폐기**: API/콘솔을 통해 인증서를 폐기하고 CRL(Certificate Revocation List)을 S3 버킷에 게시하거나 OCSP를 활성화합니다.
* **이름 제약 조건**: 인증서 오용을 완화하기 위한 가드레일을 설정하는 DNS 이름 제약 조건. 예를 들어 [CA가 특정 도메인 이름을 사용하여 리소스에 인증서를 발급하지 못하도록 제한하는 DNS 이름 제약 조건을 설정](https://aws.amazon.com/blogs/security/how-to-enforce-dns-name-constraints-in-aws-private-ca/)할 수 있습니다.

교차 계정 인증서 발급의 경우 요청을 검증하고 승인하려면 명시적 IAM 권한을 구성해야 합니다.

### 도메인 이름 고려사항

AWS Private CA는 ACM의 공개 도메인 검증 요구 사항 없이 내부 및 외부 도메인 이름 모두에 대한 인증서를 발급할 수 있는 유연한 도메인 이름 고려사항을 제공합니다. 비공개 도메인(.internal, .local, .corp 등), 프라이빗 IP 주소 및 사용자 지정 내부 명명 체계를 포함하여 유효한 도메인 명명 규칙을 사용할 수 있습니다. 인증서에는 주체 고유 이름(DN) 구성 요소(예: Common Name, Organization, Country 등) 및 DNS 이름, IP 주소, 내부 호스트 이름 및 와일드카드 도메인을 지원하는 여러 주체 대체 이름(SAN)이 포함될 수 있습니다.

### 배포 고려사항

AWS Private CA는 리전 서비스입니다. 리소스가 배포된 AWS 리전 또는 인증서를 사용할 계획인 리전과 동일한 리전에 프라이빗 CA를 생성해야 합니다. 그러나 인증서는 리전에 바인딩되지 않으며 다른 리전에 있는 리소스에 대한 인증서를 발급할 수 있습니다. AWS Certificate Manager (ACM)와 함께 AWS Private CA를 사용하여 Elastic Load Balancing, API Gateway 및 CloudFront와 같은 AWS 서비스에 인증서를 배포할 수 있습니다.

### 가능한 경우 Root CA 사용 최소화

고객은 계정 간에 Root CA를 공유하지 않고 가능한 한 제한적으로 유지해야 합니다. Root CA는 MFA가 활성화된 격리된 계정에 배치되어야 하며 하위 CA에 서명하는 것 외에는 교차 계정 액세스가 없어야 합니다.

루트 CA는 일반적으로 중간 CA에 대한 인증서를 발급하는 데만 사용해야 합니다. 이를 통해 루트 CA를 자체 계정에 안전하게 저장할 수 있으며 중간 CA는 최종 엔터티 인증서를 발급하는 일상적인 작업을 수행합니다.

![PCA Hierarchy](../../images/PCA-hierarchy-2.png)
*그림 7: 최소 Root 사용을 위한 모범 사례 CA 계층*

## AWS Private CA 구현

### 인증 기관 계층 설계

루트 및 하위 CA 계층은 지리적 분포, 사업부 분리 및 특정 사용 사례 요구 사항을 고려하여 신중하게 설계되어야 합니다. 계층 설계는 모든 인증서 작업의 프레임워크를 설정하고 PKI 인프라의 보안 및 관리 가능성에 직접 영향을 미칩니다. 잘 설계된 CA 계층은 각 CA에 적합한 세분화된 보안 제어, 더 나은 부하 분산 및 보안을 위한 관리 작업 분할, 일상 작업을 위한 제한적이고 폐기 가능한 신뢰를 가진 CA 사용, 유효 기간 및 인증서 경로 제한을 제공합니다.

AWS Private CA에서는 최대 5단계의 인증서 계층을 만들 수 있습니다. 루트 CA는 분기당 최대 4단계의 하위 CA를 가진 여러 분기를 가질 수 있습니다. 각각 자체 루트를 가진 여러 계층을 만들 수도 있습니다.

CA 계층은 트리의 맨 위에서 가장 강력한 보안을 가져야 하며 루트 CA와 해당 프라이빗 키를 보호해야 합니다. 손상된 루트는 전체 PKI의 신뢰를 파괴할 수 있기 때문입니다. 결과적으로 루트 및 상위 수준 하위 CA는 드물게만 사용해야 하며(예: 다른 CA 인증서에 서명하거나 CRL 또는 OCSP 응답자를 구성해야 하는 경우) 엄격하게 제어되고 감사되어야 합니다. 더 일상적인 관리 작업은 계층의 하위 수준 CA에 부여되어야 합니다.

### 키 알고리즘 및 길이 선택

Private CA는 [다음 RSA 및 타원 곡선 알고리즘](https://docs.aws.amazon.com/privateca/latest/userguide/PcaWelcome.html#supported-algorithms)을 지원합니다. 암호화 알고리즘 및 키 길이(RSA 2048 또는 4096비트)는 보안 및 성능 요구 사항의 균형을 맞추기 위해 선택해야 하며, 선택은 인증서 처리 오버헤드, 호환성 요구 사항 및 장기 보안 태세에 영향을 미칩니다. 진화하는 암호화 표준 및 새로운 보안 위협과의 지속적인 일치를 보장하기 위해 조직 내에서 이러한 선택에 대한 정기적인 평가를 구현해야 합니다.

### 인증서 템플릿 구성

Private CA는 구성 템플릿을 사용하여 CA 인증서 및 최종 엔터티 인증서를 발급합니다. 네 가지 유형의 템플릿이 지원됩니다:

* Base 템플릿 - 통과 파라미터가 허용되지 않는 사전 정의된 템플릿
* CSRPassthrough 템플릿 - CSR 통과를 허용하여 해당 기본 템플릿 버전을 확장하는 템플릿(예: 인증서 서명 요청의 확장 값이 발급된 인증서로 복사됨). (참고: CSR에 템플릿 정의와 충돌하는 확장 값이 포함된 경우 템플릿 정의가 더 높은 우선순위를 가짐).
* APIPassthrough 템플릿 - API 통과 또는 동적 값을 허용하여 해당 기본 템플릿 버전을 확장하는 템플릿(예: 관리자 또는 인증서를 요청한 엔터티가 알 수 없거나 템플릿에서 정의할 수 없거나 CSR에서 사용할 수 없는 중간 시스템에 알려진 동적 값). (참고: APIPassthrough 파라미터에 템플릿 정의와 충돌하는 확장 값이 포함된 경우 템플릿 정의가 더 높은 우선순위를 가짐).
* APICSRPassthrough 템플릿 - API 및 CSR 통과를 모두 허용하여 해당 기본 템플릿 버전을 확장하는 템플릿. (참고: 템플릿 정의, API 통과 값 또는 CSR 통과 확장이 충돌하는 경우 우선순위는 템플릿 정의, APIPassthrough, CSR 통과 확장 순임).

인증서 템플릿은 인증서 발급 프로세스를 표준화하고 간소화합니다. 조직은 다양한 사용 사례 및 보안 요구 사항에 맞는 적절한 유효 기간, 확장 및 제약 조건으로 템플릿을 정의해야 합니다. 잘 설계된 템플릿은 운영 유연성을 유지하면서 조직 정책을 적용합니다. 이러한 표준화는 인적 오류를 줄이고 인증서 발급의 일관성을 보장하며 조직 전체에서 인증서 수명 주기 관리를 간소화합니다.

### CA 공유

AWS Private CA는 교차 계정 공유 또는 다른 계정에 중앙 집중식 CA를 사용하여 최종 엔터티 인증서 또는 하위 CA 인증서를 발급할 수 있는 권한을 부여하는 기능을 제공합니다. 이는 AWS Resource Access Manager (RAM)를 사용하여 권한을 관리하거나 Private CA API 또는 CLI를 사용하여 CA에 리소스 기반 정책을 연결하여 수행할 수 있습니다.

[RAM을 사용하여 Private CA를 공유](https://repost.aws/knowledge-center/acm-share-pca-with-another-account)하는 경우 RAM은 리전 서비스이며 리소스 공유는 리전별이라는 점을 명심하십시오. 다른 AWS 계정의 주체와 ACM PCA 리소스 공유는 리소스 공유와 동일한 AWS 리전의 리소스(및 us-east-1에서 생성된 지원되는 글로벌 리소스)에만 액세스해야 합니다.

### AWS 서비스와의 통합

통합 계획은 AWS Private CA를 필수 AWS 서비스, 특히 자동화된 수명 주기 관리를 위한 ACM과 연결하는 것으로 구성됩니다. 기존 AWS 인프라를 평가하여 IoT 및 Direct Connect와 같은 서비스와의 통합 지점을 식별해야 합니다. 통합 전략은 AWS 에코시스템의 전체 기능을 활용하면서 자동화, 확장성 및 효율적인 인증서 배포를 강조해야 합니다.

### 액세스 제어 및 IAM 정책 보안

AWS Private CA 작업에는 AWS IAM을 통한 정확한 액세스 제어 구현이 필요합니다. 조직은 최소 권한 원칙을 적용하는 역할 및 정책을 생성하여 인증서 발급, 폐기 및 CA 관리에 대한 특정 권한을 정의해야 합니다. CA 관리를 제어하는 효과적인 방법은 권한 분리를 통하는 것입니다. 즉, 사용자 A는 CA의 상태만 변경할 수 있고 사용자 B는 CA를 통해 인증서를 설치하고 발급할 수만 있습니다. 따라서 새 CA를 생성하고 활성화하려면 두 사용자가 모두 필요합니다. 이러한 권한 분리는 CA를 비활성화하는 역할과 삭제할 수 있는 역할 간에도 구현되어야 합니다.

정기적인 정책 검토는 운영 효율성을 유지하면서 보안 요구 사항과의 지속적인 일치를 보장합니다. 이러한 구조화된 액세스 제어 접근 방식은 무단 액세스를 방지하고 규정 준수 요구 사항을 지원합니다.

### 관리형 폐기

RevokeCertificate API는 키가 손상되거나 연결된 도메인이 유효하지 않게 되는 등의 경우 예정된 만료 전에 AWS Private CA 인증서를 폐기합니다. AWS는 인증서를 사용하는 클라이언트가 폐기 상태를 확인할 수 있는 두 가지 완전 관리형 메커니즘인 OCSP 및 CRL(Certificate Revocation List)을 제공합니다.

![Revocation Chart](../../images/PCA-revocation.png)
*그림 8: AWS 완전 관리형 폐기 메커니즘 비교*

**OCSP**

AWS Private CA는 인증서가 폐기되었음을 엔드포인트에 알리는 완전 관리형 OCSP 솔루션을 제공합니다. 고객은 Private CA 콘솔, API, CLI 또는 CloudFormation을 통해 CA에서 OCSP를 활성화할 수 있습니다. OCSP를 사용하면 클라이언트가 동기 상태를 반환하는 권한 있는 폐기 데이터베이스를 쿼리합니다. OCSP가 활성화되면 Private CA는 새 인증서의 AIA(Authority Information Access) 확장에 OCSP 응답자의 URL을 포함합니다.

OCSP에 대한 몇 가지 고려사항:

* OCSP 응답은 폐기된 인증서의 새 상태를 반영하는 데 최대 60분이 걸릴 수 있습니다
* APIPassthrough 및 CSRPassthrough 인증서 템플릿은 OCSP 응답자가 활성화된 경우 AIA와 작동하지 않습니다
* 관리형 OCSP 서비스의 엔드포인트는 공용 인터넷에서 사용할 수 있습니다. OCSP를 원하지만 공용 엔드포인트를 선호하지 않는 고객은 대체 인프라를 운영해야 합니다.
* OCSP는 온디맨드로 가격이 책정됩니다.

**CRL**

CRL은 폐기된 모든 인증서 목록과 폐기 이유를 포함한 관련 정보를 포함하는 파일입니다. CA를 구성할 때 AWS Private CA가 완전한 CRL을 생성할지 또는 분할된 CRL(CRL이 여러 개의 작은 CRL로 분할됨)을 생성할지 선택할 수 있습니다. 각 CRL은 50,000개의 인증서만 폐기할 수 있으므로 분할된 CRL이 많을수록 지원되는 각 리전에 대해 최대 1,000,000개 제한까지 폐기할 수 있는 인증서가 더 많습니다.

CRL에 대한 몇 가지 고려사항:

* CRL은 인증서가 폐기된 후 약 30분 후에 업데이트됩니다. CRL 업데이트가 실패하면 Private CA는 15분마다 추가 시도를 계속합니다
* 다운로드 및 처리 요구 사항으로 인해 CRL은 OCSP보다 더 많은 메모리가 필요합니다. 폐기 목록 또는 분할된 CRL 캐싱을 고려하십시오.
* CRL을 완전에서 분할로 업데이트하면 AWS Private CA는 필요에 따라 새 파티션을 생성하고 원본을 포함한 모든 CRL에 IDP 확장을 추가합니다.
* 분할된 CRL은 프라이빗 CA가 발급할 수 있는 인증서 수를 크게 늘리고 CA를 자주 교체하지 않아도 됩니다.

## Kubernetes용 커넥터

AWS Private CA는 오픈 소스 cert-manager 플러그인인 aws-privateca-issuer를 통해 Kubernetes와의 강력한 통합을 도입했습니다. 이 솔루션을 통해 조직은 엄격한 보안 제어 및 규정 준수 요구 사항을 유지하면서 Kubernetes 워크로드에 대한 인증서를 관리할 수 있습니다. 플러그인은 클러스터 내에 민감한 프라이빗 키를 저장할 필요를 없애 보안 위험을 크게 줄이고 인증서 수명 주기 관리를 간소화합니다.

aws-privateca-issuer 플러그인은 Amazon EKS, AWS의 자체 관리형 Kubernetes 및 온프레미스 Kubernetes 환경 전반에 걸쳐 배포를 지원하는 유연성을 제공합니다. 조직은 x86 및 ARM 아키텍처를 모두 포함하여 인프라 선택에 관계없이 이 솔루션을 활용할 수 있습니다. 이 통합은 향상된 감사 가능성과 인증서 작업에 대한 제어를 제공하므로 엄격한 규제 요구 사항이 있는 기업에 특히 유용합니다. 보안 팀은 개발 팀이 익숙한 Kubernetes 워크플로를 통해 인증서 요구 사항을 효율적으로 관리할 수 있도록 하면서 인증서 발급에 대한 중앙 집중식 감독을 유지할 수 있습니다.

![Connector for Kubernetes](../../images/PCA-kubernetes.png)
*그림 9: Kubernetes용 AWS Private CA 커넥터*

이 아키텍처는 AWS Private CA를 사용하여 Amazon Elastic Kubernetes Service (Amazon EKS)에서 엔드 투 엔드 암호화를 설정하는 방법을 보여줍니다. 이 엔드 투 엔드 암호화 예제의 경우 트래픽은 클라이언트에서 시작되어 샘플 앱 내부에서 실행되는 Ingress 컨트롤러 서버에서 종료됩니다.

### SCEP용 커넥터

[AWS Private Certificate Authority (AWS Private CA) Connector for SCEP](https://www.youtube.com/watch?v=sqlyaQqyUnw) (Simple Certificate Enrollment Protocol)는 모바일 장치 인증서 관리를 위한 간소화된 솔루션을 제공합니다. 이 미리 보기 기능은 인기 있는 MDM(Mobile Device Management) 솔루션과 원활하게 통합되어 모바일 장치의 보안을 강화하고 인증서 배포를 간소화합니다.

커넥터는 모바일 장치 인증서를 관리하는 조직에 몇 가지 주요 이점을 제공합니다. 프라이빗 인증서로 모바일 장치를 보호하는 프로세스를 간소화하여 운영 비용과 복잡성을 크게 줄입니다. 조직은 AWS Private CA 커넥터와 함께 단일 인증 기관을 활용하여 다양한 사용 사례에서 여러 CA가 필요하지 않으므로 인증서 관리를 간소화하고 인프라 복잡성을 줄일 수 있습니다.

이 솔루션의 주요 장점은 PKI 운영의 비용 효율성과 효율성입니다. 관리형 SCEP 서비스와 함께 관리형 프라이빗 CA를 활용함으로써 조직은 PKI 인프라에 대한 시간과 비용을 모두 절약할 수 있습니다. 또한 이 솔루션은 상업적으로 사용 가능한 MDM 솔루션과 함께 작동하는 AWS Private CA를 통해 모바일 장치의 자동 등록을 가능하게 하여 장치 온보딩 및 인증서 배포 프로세스를 간소화합니다.

![Connector for SCEP](../../images/PCA-SCEP.png)
*그림 10: SCEP용 AWS Private CA 커넥터*

다이어그램에 설명된 아키텍처는 MDM 솔루션이 SCEP용 AWS Private CA 커넥터를 통해 모바일 장치와 통신한 다음 인증서 관리를 위해 AWS Private CA와 인터페이스하는 간단한 흐름을 보여줍니다. 이 통합 접근 방식은 대규모로 장치 인증서를 관리하는 안전하고 효율적인 방법을 제공합니다.

### Active Directory용 커넥터

AWS Private CA Connector for Active Directory는 Microsoft Active Directory (AD) 환경 내에서 사용자 및 컴퓨터에 대한 인증서를 프로비저닝하는 데 도움이 될 수 있습니다. AWS Private CA를 AD와 결합하면 관리형 프라이빗 CA를 활용하면서 기존 AD 인프라를 사용하여 인증서 등록, 배포 및 신뢰를 처리할 수 있습니다. AWS Private CA Connector for Active Directory를 사용하면 온프레미스 엔터프라이즈 또는 기타 타사 CA를 소유한 관리형 프라이빗 CA로 교체하여 AD에서 관리하는 사용자, 그룹 및 컴퓨터에 인증서 등록을 제공할 수 있습니다.

AWS Private CA를 Active Directory와 통합하려면 AWS Cloud에서 관리형 Microsoft Active Directory를 제공하는 AWS Directory Service를 사용할 수 있습니다. AWS Directory Service를 사용하여 디렉터리를 생성하면 AWS Private CA가 디렉터리를 신뢰하고 Active Directory에 대해 인증된 사용자 및 장치에 인증서를 발급하도록 구성할 수 있습니다. AD용 커넥터는 온프레미스 AD 환경(AWS Directory Services에서 제공하는 [Active Directory Connector](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/directory_ad_connector.html) 사용) 및 [AWS Managed Microsoft AD](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/directory_microsoft_ad.html)에서 호스팅되는 AD 환경 모두에 대해 도메인 가입 장치에 인증서를 발급하는 데 사용할 수 있습니다.

![Connector for AD](../../images/PCA-AD.png)
*그림 11: Active Directory용 AWS Private CA 커넥터*

이 아키텍처는 AWS Private CA 없이 온프레미스 ADCS 구현과 AD용 커넥터와 함께 AWS Private CA를 사용하는 구현 간의 차이를 보여줍니다.

### 교차 계정 CA 공유

AWS Private CA는 조직 내의 다른 AWS 계정과 프라이빗 인증 기관(CA)을 공유할 수 있는 교차 계정 공유를 지원합니다.

AWS RAM(Resource Access Manager)을 사용하여 ACM Private CA를 공유하여 다른 AWS 계정과 리소스 공유를 생성할 수 있습니다. AWS RAM을 사용하여 리소스 기반 정책을 적용하면 AWS Private CA 관리자가 다른 AWS 계정의 사용자와 직접 또는 AWS Organizations를 통해 CA에 대한 액세스를 공유할 수 있습니다. 기본 계정은 Private CA에 정책을 연결하여 인증서 발급, 인증서 폐기 또는 CA 인증서 내보내기와 같은 작업을 수행할 수 있는 보조 계정 또는 IAM 역할을 지정합니다. 그런 다음 보조 계정은 IAM 정책을 사용하여 사용자 또는 역할이 공유 CA와 상호 작용할 수 있도록 합니다. 이 설정은 인증서 발급을 신뢰할 수 있는 계정에 위임하면서 CA의 수명 주기에 대한 중앙 집중식 제어를 보장합니다.

다음과 같은 다른 엔터티와 ACM Private CA를 공유할 수도 있습니다:

* AWS Identity and Access Management (IAM) 사용자 및 IAM 역할과 같은 기타 주체.
* 조직 단위(OU).
* 계정이 구성원인 전체 AWS 조직.

![Cross-account CA Sharing](../../images/PCA-crossaccount.png)
*그림 12: 교차 계정 CA 공유*

이 아키텍처는 AWS Private CA를 사용한 교차 계정 CA 공유에 대한 자세한 개요를 제공합니다. Root CA, Issuing CA 및 RAM Shared CA가 함께 작동하여 계정 간에 신뢰 체인을 유지하면서 중앙 집중식 인증서 관리를 가능하게 하는 방법을 보여줍니다.

### ACM과 AWS Private CA의 차이점은 무엇입니까?

AWS Private CA와 ACM은 별개이지만 상호 보완적인 서비스입니다. AWS Private CA를 사용하면 조직이 AWS 내에서 자체 CA를 설정하고 운영할 수 있으며, 인증서 계층 및 정책에 대한 완전한 제어와 함께 내부 및 외부 리소스 모두에 대한 프라이빗 인증서를 발급할 수 있습니다. 반대로 ACM은 주로 AWS 서비스용 SSL/TLS 인증서 관리에 중점을 두고 Amazon Trust Services CA의 무료 공개 인증서와 갱신 및 배포를 포함한 자동화된 인증서 수명 주기 관리를 제공합니다.

Private CA는 CA 운영 및 인증서 발급과 관련된 비용으로 모든 리소스에 대한 인증서를 발급할 수 있는 더 광범위한 유연성을 제공하는 반면 ACM은 무료 공개 인증서 및 심층 서비스 통합으로 AWS 서비스에 특화된 인증서 관리를 간소화합니다. 이러한 서비스는 실제로 종종 함께 작동하며 AWS Private CA는 프라이빗 인증서 생성을 처리하고 ACM은 AWS 서비스 내에서 배포 및 수명 주기를 관리하여 조직에 포괄적인 인증서 관리 솔루션을 제공합니다.

* 웹사이트, 애플리케이션 또는 AWS에서 실행되는 API와 같은 AWS 서비스에 대해 무료로 자동 관리되는 공개 인증서가 필요한 경우 ACM을 사용하십시오.
* 내부 리소스, AWS 외부 애플리케이션에 대한 자체 프라이빗 인증서를 생성하고 관리해야 하거나 인증서 발급 및 정책에 대한 완전한 제어가 필요한 경우 AWS Private CA를 사용하십시오.

### 비용 고려사항

AWS Certificate Manager (ACM)는 공개 TLS 인증서를 무료로 제공합니다. AWS PrivateCA 가격은 세 부분 모델을 따릅니다: 운영 비용에 대한 CA당 고정 월별 요금, 프라이빗 인증서 발급에 대한 사용량 기반 요금 및 OCSP 검증 요금(CRL 사용은 무료로 유지됨).

AWS Private CA를 시작하거나 새 계정에서 활성화하는 경우 AWS Private CA는 각 리전의 계정에서 생성된 첫 번째 프라이빗 CA에 대해 30일 무료 평가판을 제공합니다. ACM을 통해 요청한 인증서의 프라이빗 키 및 인증서를 처음 내보낼 때 일회성 요금을 포함하여 평가판 기간 동안 발급된 모든 인증서에 대해 비용을 지불합니다. 이 평가판 기간은 전체 CA 운영 비용을 발생시키지 않고 서비스의 기능을 평가하고 조직의 PKI 요구 사항을 충족하는지 확인할 수 있는 기회를 제공합니다.

## AWS Private CA 모범 사례 체크리스트

모범 사례는 AWS Private CA를 효과적으로 사용하는 데 도움이 되는 권장 사항입니다. 다음 모범 사례는 현재 AWS Private CA 고객의 실제 경험을 기반으로 합니다.

* CA 구조 및 정책을 문서화합니다. 루트 CA에서 최종 엔터티 인증서를 발급하지 마십시오.
* 초기 CA 생성 중에 CA 유효 기간을 정의하고 설정합니다.
* Root CA 사용을 중간 CA 인증서 발급으로만 제한합니다.
* 효율적인 관리를 위해 2-3단계의 간단한 CA 계층을 유지합니다.
* 다른 CA와 별도로 전용 AWS 계정에 루트 CA를 배포합니다.
* 고유한 IAM 권한을 통해 관리자와 인증서 발급자 역할 간의 분리를 유지합니다.
* 자동화된 인증서 상태 관리를 위해 OCSP 또는 CRL을 통한 인증서 폐기를 구현합니다.
* 프라이빗 CA를 생성하고 운영하기 전에 AWS CloudTrail 로깅을 활성화합니다.
* 인증서 가져오기 또는 CA 교체를 통해 CA 프라이빗 키를 정기적으로 교체합니다.
* 권장 삭제 프로세스에 따라 사용하지 않는 CA를 제거합니다.
* PKI 세부 정보를 보호하기 위해 CRL 스토리지 버킷에 대해 S3 Block Public Access를 활성화합니다.
* Kubernetes 환경에서 AWS Private CA를 사용할 때 Amazon EKS 모범 사례 가이드를 따르십시오.

## 리소스

### 워크샵
* [Activation Days](https://awsactivationdays.splashthat.com/)
* [AWS Encryption in Transit Workshop](https://catalog.workshops.aws/certificatemanager/en-US)
* [Hands-on ACM Troubleshooting Labs](https://catalog.workshops.aws/acm-troubleshooting/en-US)

### 비디오
* [AWS Private Certificate Authority (AWS Private CA) Connector for SCEP](https://www.youtube.com/watch?v=sqlyaQqyUnw)

### 블로그
* [How to manage certificate lifecycles using ACM event-driven workflows](https://aws.amazon.com/blogs/security/how-to-manage-certificate-lifecycles-using-acm-event-driven-workflows/)
* [How to enforce DNS name constraints in AWS Private cA](https://aws.amazon.com/blogs/security/how-to-enforce-dns-name-constraints-in-aws-private-ca/)
