# Quotas for your Application Load Balancers<a name="load-balancer-limits"></a>

Your AWS account has default quotas, formerly referred to as limits, for each AWS service\. Unless otherwise noted, each quota is Region\-specific\. You can request increases for some quotas, and other quotas cannot be increased\.

To view the quotas for your Application Load Balancers, open the [Service Quotas console](https://console.aws.amazon.com/servicequotas/home)\. In the navigation pane, choose **AWS services** and select **Elastic Load Balancing**\. You can also use the [describe\-account\-limits](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-account-limits.html) \(AWS CLI\) command for Elastic Load Balancing\.

To request a quota increase, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html) in the *Service Quotas User Guide*\. If the quota is not yet available in Service Quotas, use the [Elastic Load Balancing limit increase form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-elastic-load-balancers)\.

Your AWS account has the following quotas related to Application Load Balancers\.

**Regional**
+ Load balancers per Region: 50
+ Target groups per Region: 3000 **\***

**Load balancer**
+ Listeners per load balancer: 50
+ Targets per load balancer: 1000
+ Target groups per load balancer: 100
+ Subnets per Availability Zone per load balancer: 1
+ Security groups per load balancer: 5
+ Rules per load balancer \(not counting default rules\): 100
+ Certificates per load balancer \(not counting default certificates\): 25
+ Number of times a target can be registered per load balancer: 100

**Target group**
+ Load balancers per target group: 1
+ Targets per target group \(instances or IP addresses\): 1000
+ Targets per target group \(Lambda functions\): 1

**Rule**
+ Target groups per action: 5
+ Match evaluations per rule: 5
+ Wildcards per rule: 5
+ Actions per rule: 2 \(one optional authentication action, one required action\)

**\*** This quota is shared by target groups for your Application Load Balancers and Network Load Balancers\.