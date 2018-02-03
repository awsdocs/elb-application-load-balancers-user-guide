# Limits for Your Application Load Balancers<a name="load-balancer-limits"></a>

To view the current limits for your Application Load Balancers, use the **Limits** page of the Amazon EC2 console or the [describe\-account\-limits](http://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-account-limits.html) \(AWS CLI\) command\. To request a limit increase, use the [Elastic Load Balancing Limits form](https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-elastic-load-balancers)\.

Your AWS account has the following limits related to Application Load Balancers\.

**Regional Limits**

+ Load balancers per region: 20 **\***

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

+ Targets per target group: 1000

**Rule Limits**

+ Conditions per rule: 2 \(one host condition, one path condition\)

+ Actions per rule: 1

+ Target groups per action: 1

**\*** This limit includes both your Application Load Balancers and your Classic Load Balancers\.