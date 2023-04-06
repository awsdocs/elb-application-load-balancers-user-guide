# Quotas for your Application Load Balancers<a name="load-balancer-limits"></a>

Your AWS account has default quotas, formerly referred to as limits, for each AWS service\. Unless otherwise noted, each quota is Region\-specific\. You can request increases for some quotas, and other quotas cannot be increased\.

To view the quotas for your Application Load Balancers, open the [Service Quotas console](https://console.aws.amazon.com/servicequotas/home)\. In the navigation pane, choose **AWS services** and select **Elastic Load Balancing**\. You can also use the [describe\-account\-limits](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-account-limits.html) \(AWS CLI\) command for Elastic Load Balancing\.

To request a quota increase, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html) in the *Service Quotas User Guide*\. If the quota is not yet available in Service Quotas, use the [Elastic Load Balancing limit increase form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-elastic-load-balancers)\.

**Load balancers**  
Your AWS account has the following quotas related to Application Load Balancers\.


| Name | Default | Adjustable | 
| --- | --- | --- | 
| Application Load Balancers per Region |  50  | [Yes](https://console.aws.amazon.com/servicequotas/home/services/elasticloadbalancing/quotas/L-53DA6B97) | 
| Certificates per Application Load Balancer \(excluding default certificates\) |  25  | [Yes](https://console.aws.amazon.com/servicequotas/home/services/elasticloadbalancing/quotas/L-9365A611) | 
| Listeners per Application Load Balancer |  50  | [Yes](https://console.aws.amazon.com/servicequotas/home/services/elasticloadbalancing/quotas/L-B6DF7632) | 
| Number of times a target can be registered per Application Load Balancer |  1,000  | No | 
| Target Groups per Action per Application Load Balancer |  5  | No | 
| Target Groups per Application Load Balancer |  100  | No | 
| Targets per Application Load Balancer |  1,000  | [Yes](https://console.aws.amazon.com/servicequotas/home/services/elasticloadbalancing/quotas/L-7E6692B2) | 

**Target groups**  
The following quotas are for target groups\.


| Name | Default | Adjustable | 
| --- | --- | --- | 
| Target Groups per Region  |  3,000 \* | [Yes](https://console.aws.amazon.com/servicequotas/home/services/elasticloadbalancing/quotas/L-B22855CB) | 
| Targets per Target Group per Region \(instances or IP addresses\) |  1,000  | [Yes](https://console.aws.amazon.com/servicequotas/home/services/elasticloadbalancing/quotas/L-A0D0B863) | 
| Targets per Target Group per Region \(Lambda functions\) | 1 | No | 
| Load balancers per target group | 1 | No | 

**\*** This quota is shared by Application Load Balancers and Network Load Balancers\.

**Rules**  
The following quotas are for rules\.


| Name | Default | Adjustable | 
| --- | --- | --- | 
| Rules per Application Load Balancer \(excluding default rules\) |  100  | [Yes](https://console.aws.amazon.com/servicequotas/home/services/elasticloadbalancing/quotas/L-7EED9B64) | 
| Condition Values per Rule |  5  | No | 
| Condition Wildcards per Rule |  5  | No | 
| Match evaluations per rule | 5 | No | 

**HTTP headers**  
The following are the size limits for HTTP headers\.


| Name | Default | Adjustable | 
| --- | --- | --- | 
| Request line | 16 K | No | 
| Single header | 16 K | No | 
| Entire response header | 32 K | No | 
| Entire request header | 64 K | No | 