# WAF 규칙 모니터링

## AWS WAF 메트릭 분석

[AWS WAF용 CloudWatch 메트릭](https://docs.aws.amazon.com/waf/latest/developerguide/monitoring-cloudwatch.html)을 사용하여 웹 ACL의 각 규칙에 의해 허용, 차단 또는 계산된 요청 수를 시간 경과에 따라 보여주는 대시보드에 그래프를 만들 수 있습니다. 레이블별로 동일한 메트릭을 볼 수도 있습니다.

이러한 메트릭은 몇 가지 시나리오에서 유용합니다.

* *Block* 모드로 전환하기로 결정하기 전에 *Count* 모드의 새 규칙과 일치하는 요청 속도를 모니터링합니다.
* 차단되거나 계산된 요청이 급증할 때 트리거되는 알람을 생성합니다. 이는 조사가 필요한 위협을 나타낼 수 있습니다.
* 지난 며칠 동안 가장 많은 차단된 요청에 기여하는 규칙을 확인합니다. 이를 통해 의도한 대로 작동하지 않는 규칙을 격리하는 데 도움이 될 수 있습니다.
* [Shield Advanced 규칙 그룹](https://docs.aws.amazon.com/waf/latest/developerguide/ddos-automatic-app-layer-response-rg.html)의 규칙이 많은 수의 요청을 차단하거나 계산하기 시작할 때 트리거되는 알람을 생성합니다.

## AWS WAF 로그 분석

CloudWatch Logs에 AWS WAF 로그를 저장할 때 Contributor Insights 및 Logs Insights를 사용하여 CloudWatch 대시보드로 로그를 시각화할 수 있습니다. 블로그 [Amazon CloudWatch 대시보드로 AWS WAF 로그 시각화](https://aws.amazon.com/blogs/security/visualize-aws-waf-logs-with-an-amazon-cloudwatch-dashboard/)를 참조하십시오.

[Amazon Athena를 사용하여 Amazon S3의 AWS WAF 로그를 분석](https://docs.aws.amazon.com/athena/latest/ug/waf-logs.html)할 수 있습니다. 문서는 시작점으로 예제 쿼리를 제공합니다.

Athena 쿼리를 활용하여 [Amazon QuickSight에서 대시보드를 생성](https://aws.amazon.com/blogs/security/enabling-serverless-security-analytics-using-aws-waf-full-logs/)할 수 있습니다.
