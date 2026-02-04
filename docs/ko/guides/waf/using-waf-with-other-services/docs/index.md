# 다른 서비스와 함께 AWS WAF 사용

## AWS Firewall Manager(FMS)

Firewall Manager에서 [WAF 정책을 생성](https://docs.aws.amazon.com/waf/latest/developerguide/waf-policies.html)할 수 있습니다. 웹 ACL의 상단과 하단에 원하는 규칙 그룹을 지정합니다. AWS Organization에서 리소스의 범위를 정의하면 FMS가 각 구성원 계정에 웹 ACL을 생성하고 범위 내 리소스와 연결합니다. FMS 관리형 웹 ACL은 구성원 계정에서 사용자 지정할 수 있지만 상단 및 하단 규칙 그룹은 수정할 수 없습니다.

### AWS WAF용 FMS 보안 정책에 포함할 규칙 결정

AWS WAF용 FMS 보안 정책은 일반적으로 중앙 보안 팀에서 관리합니다. 이 팀은 조직 전체에 규칙 기준선을 시행할 책임이 있습니다. 고유한 기준선이 필요하므로 고유한 정책이 필요한 조직의 일부가 있을 수 있습니다.

FMS 관리형 웹 ACL은 구성원 계정에서 사용자 지정해야 할 수 있습니다. 관리형 규칙이 해결해야 하는 거짓 양성을 유발하거나 기준선에서 완화되지 않는 애플리케이션별 위협이 있을 수 있습니다. FMS 정책에 애플리케이션별 변경을 수행하는 대신 웹 ACL을 업데이트하는 것이 좋습니다. 일반적으로 각각 몇 개의 리소스에 적용되는 많은 FMS 정책보다 광범위한 리소스에 적용되는 몇 개의 FMS 정책을 갖는 것이 일반적입니다.

### CloudFormation을 사용하여 FMS 관리형 웹 ACL 업데이트

[AWS WAF V2 리소스](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_WAFv2.html)는 CloudFormation 템플릿을 사용하여 완전히 정의할 수 있습니다. Firewall Manager를 사용하여 웹 ACL을 관리할 때 몇 가지 고려 사항이 있습니다.

CloudFormation을 사용하여 AWS WAF용 [Firewall Manager 정책](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-fms-policy.html)을 정의할 수 있습니다. 정책에는 FMS에서 생성한 웹 ACL의 상단과 하단에 원하는 AWS WAF 규칙의 정의가 포함됩니다. CloudFormation(또는 다른 [코드형 인프라](https://docs.aws.amazon.com/whitepapers/latest/introduction-devops-aws/infrastructure-as-code.html) 도구)을 사용할 때 몇 가지 영향이 있습니다.

CloudFormation 템플릿 형식은 [AWS::WAFev2::WebACL](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-wafv2-webacl.html)과 [AWS::FMS::Policy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-fms-policy.html) 간에 동일하지 않습니다. 기존 AWS WAF 웹 ACL을 Firewall Manager 정책의 기반으로 사용하려면 웹 ACL을 JSON으로 다운로드하거나(AWS WAF 콘솔에서) [GetWebACL](https://docs.aws.amazon.com/waf/latest/APIReference/API_GetWebACL.html) API를 사용해야 합니다. 이 JSON을 사용하여 FMS 정책 리소스의 [SecurityServicePolicyData](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-fms-policy-securityservicepolicydata.html) 요소를 구성합니다. 간단한 경우 FMS 콘솔에서 AWS WAF 규칙을 수동으로 다시 만드는 것이 더 쉬울 수 있습니다.

구성원 계정은 AWS 콘솔 또는 CLI 또는 SDK를 사용하여 FMS에서 생성한 웹 ACL을 사용자 지정할 수 있습니다. 그러나 FMS 관리형 웹 ACL에 사용자 지정 규칙을 추가하는 표준 CloudFormation 템플릿을 만들 수 없습니다. AWS WAF 규칙은 AWS::WAFev2::WebACL 리소스 내부에 정의됩니다. 웹 ACL이 이미 존재하므로 CloudFormation 템플릿에서 정의할 수 없습니다. 현재 이 문제를 해결하는 두 가지 옵션이 있습니다.

1. FMS 관리형 웹 ACL을 CloudFormation 스택으로 [가져온](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resource-import.html) 다음 사용자 지정 규칙으로 스택을 업데이트합니다.
2. [AWS::WAFv2::RuleGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-wafv2-rulegroup.html)에서 사용자 지정 규칙을 정의한 다음 FMS 관리형 웹 ACL을 검색하고 규칙 그룹을 참조하는 규칙을 생성하는 코드가 있는 Lambda 지원 [사용자 지정 리소스](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html)를 사용합니다.

### FMS 관리형 규칙 그룹으로 인한 거짓 양성 처리

대규모 조직에서는 FMS 관리형 웹 ACL에 하나 이상의 구성원 계정에 대해 거짓 양성을 유발하는 규칙이 포함되는 것이 드물지 않습니다. 여러 FMS 정책을 사용하여 이러한 상황을 처리할 수 있습니다.

1. 거짓 양성 처리를 허용하기 위해 문제가 있는 규칙을 *Count* 모드로 설정한 *예외 정책*. 이 정책의 범위에는 특정 태그가 있는 리소스만 포함됩니다. 구성원 계정은 거짓 양성을 적절하게 처리하는 규칙을 추가할 책임이 있습니다.
2. *기본 정책*은 모든 관리형 규칙을 *Block* 모드로 설정합니다. 이 정책은 거짓 양성에 대해 우려하지 않는 리소스를 보호하는 데 사용됩니다. 이 정책의 범위는 예외 정책에서 사용하는 태그가 있는 리소스를 제외합니다.

## Amazon GuardDuty

Amazon GuardDuty에서 생성한 결과를 기반으로 AWS WAF 규칙 생성을 자동화할 수 있습니다. 블로그 [Amazon GuardDuty 및 AWS WAF v2를 사용하여 의심스러운 호스트를 자동으로 차단하는 방법](https://aws.amazon.com/blogs/security/how-to-use-amazon-guardduty-and-aws-waf-v2-to-automatically-block-suspicious-hosts/)을 참조하십시오.

## AWS Shield Advanced

[Shield Advanced 자동 애플리케이션 계층 DDoS 완화](https://docs.aws.amazon.com/waf/latest/developerguide/ddos-automatic-app-layer-response.html)는 애플리케이션 계층 공격을 자동으로 감지하고 비정상적인 트래픽을 격리하고 차단하는 AWS WAF 규칙을 생성하는 Shield Advanced 기능입니다. 이 기능을 활성화하면 [Shield Advanced에서 관리하는 규칙 그룹](https://docs.aws.amazon.com/waf/latest/developerguide/ddos-automatic-app-layer-response-rg.html)이 보호된 리소스의 웹 ACL 하단에 추가됩니다.

[Firewall Manager의 AWS Shield 정책](https://docs.aws.amazon.com/waf/latest/developerguide/shield-policies.html)을 사용하여 이 기능을 자동으로 구성할 수 있습니다.

## AWS Transfer Family

AWS Transfer Family는 파일 전송 프로토콜을 관리하는 서비스입니다. 블로그 [AWS Web Application Firewall 및 Amazon API Gateway로 AWS Transfer Family 보안](https://aws.amazon.com/blogs/storage/securing-aws-transfer-family-with-aws-web-application-firewall-and-amazon-api-gateway/)을 참조하십시오.
