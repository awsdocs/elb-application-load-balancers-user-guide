# Limits for Your Application Load Balancers<a name="load-balancer-limits"></a>

To view the current limits for your Application Load Balancers, use the **Limits** page of the Amazon EC2 console or the [describe\-account\-limits](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-account-limits.html) \(AWS CLI\) command\. To request a limit increase, use the [Elastic Load Balancing Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-elastic-load-balancers)\.

Your AWS account has the following limits related to Application Load Balancers\.

**Regional Limits**
+ Load balancers per region: 20
+ Target groups per region: 3000

**Load Balancer Limits**
+ Listeners per load balancer: 50
+ Targets per load balancer: 1000
+ Subnets per Availability Zone per load balancer: 1
+ Security groups per load balancer: 5
+ Rules per load balancer \(not counting default rules\): 100
+ Certificates per load balancer \(not counting default certificates\): 25
+ Number of times a target can be registered per load balancer: 100

**Target Group Limits**
+ Load balancers per target group: 1
+ Targets per target group \(instances or IP addresses\): 1000
+ Targets per target group \(Lambda functions\): 1

**Rule Limits**
+ Match evaluations per rule: 5
+ Wildcards per rule: 5
+ Actions per rule: 2 \(one optional authentication action, one required action\)