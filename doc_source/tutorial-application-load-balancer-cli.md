# Tutorial: Create an Application Load Balancer Using the AWS CLI<a name="tutorial-application-load-balancer-cli"></a>

This tutorial provides a hands\-on introduction to Application Load Balancers through the AWS CLI\.

## Before You Begin<a name="prerequisites-aws-cli"></a>
+ Use the following command to verify that you are running a version of the AWS CLI that supports Application Load Balancers\.

  ```
  aws elbv2 help
  ```

  If you get an error message that elbv2 is not a valid choice, update your AWS CLI\. For more information, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.
+ Launch your EC2 instances in a virtual private cloud \(VPC\)\. Ensure that the security groups for these instances allow access on the listener port and the health check port\. For more information, see [Target Security Groups](target-group-register-targets.md#target-security-groups)\.

## Create Your Load Balancer<a name="create-load-balancer-aws-cli"></a>

To create your first load balancer, complete the following steps\.

**To create a load balancer**

1. Use the [create\-load\-balancer](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-load-balancer.html) command to create a load balancer\. You must specify two subnets that are not from the same Availability Zone\.

   ```
   aws elbv2 create-load-balancer --name my-load-balancer  \
   --subnets subnet-12345678 subnet-23456789 --security-groups sg-12345678
   ```

   The output includes the Amazon Resource Name \(ARN\) of the load balancer, with the following format:

   ```
   arn:aws:elasticloadbalancing:us-east-2:123456789012:loadbalancer/app/my-load-balancer/1234567890123456
   ```

1. Use the [create\-target\-group](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-target-group.html) command to create a target group, specifying the same VPC that you used for your EC2 instances:

   ```
   aws elbv2 create-target-group --name my-targets --protocol HTTP --port 80 \
   --vpc-id vpc-12345678
   ```

   The output includes the ARN of the target group, with this format:

   ```
   arn:aws:elasticloadbalancing:us-east-2:123456789012:targetgroup/my-targets/1234567890123456
   ```

1. Use the [register\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/register-targets.html) command to register your instances with your target group:

   ```
   aws elbv2 register-targets --target-group-arn targetgroup-arn  \
   --targets Id=i-12345678 Id=i-23456789
   ```

1. Use the [create\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-listener.html) command to create a listener for your load balancer with a default rule that forwards requests to your target group:

   ```
   aws elbv2 create-listener --load-balancer-arn loadbalancer-arn \
   --protocol HTTP --port 80  \
   --default-actions Type=forward,TargetGroupArn=targetgroup-arn
   ```

   The output contains the ARN of the listener, with the following format:

   ```
   arn:aws:elasticloadbalancing:us-east-2:123456789012:listener/app/my-load-balancer/1234567890123456/1234567890123456
   ```

1. \(Optional\) You can verify the health of the registered targets for your target group using this [describe\-target\-health](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-target-health.html) command:

   ```
   aws elbv2 describe-target-health --target-group-arn targetgroup-arn
   ```

## Add an HTTPS Listener<a name="https-listener-aws-cli"></a>

If you have a load balancer with an HTTP listener, you can add an HTTPS listener as follows\.

**To add an HTTPS listener to your load balancer**

1. Create an SSL certificate for use with your load balancer using one of the following methods:
   + Create or import the certificate using AWS Certificate Manager \(ACM\)\. For more information, see [Request a Certificate](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request.html) or [Importing Certificates](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html) in the *AWS Certificate Manager User Guide*\.
   + Upload the certificate using AWS Identity and Access Management \(IAM\)\. For more information, see [Working with Server Certificates](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_server-certs.html) in the *IAM User Guide*\.

1. Use the [create\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-listener.html) command to create the listener with a default rule that forwards requests to your target group\. You must specify an SSL certificate when you create an HTTPS listener\. Note that you can specify an SSL policy other than the default using the `--ssl-policy` option\.

   ```
   aws elbv2 create-listener --load-balancer-arn loadbalancer-arn \
   --protocol HTTPS --port 443  \
   --certificates CertificateArn=certificate-arn \
   --default-actions Type=forward,TargetGroupArn=targetgroup-arn
   ```

## Add Targets Using Port Overrides<a name="port-overrides-aws-cli"></a>

If you have multiple ECS containers on a single instance, each container accepts connections on a different port\. You can register the instance with the target group multiple times, each time with a different port\.

**To add targets using port overrides**

1. Use the [create\-target\-group](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-target-group.html) command to create a target group:

   ```
   aws elbv2 create-target-group --name my-targets --protocol HTTP --port 80 \
   --vpc-id vpc-12345678
   ```

1. Use the [register\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/register-targets.html) command to register your instances with your target group\. Notice that the instance IDs are the same for each container, but the ports are different\.

   ```
   aws elbv2 register-targets --target-group-arn targetgroup-arn  \
   --targets Id=i-12345678,Port=80 Id=i-12345678,Port=766
   ```

1. Use the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) command to add a rule to your listener that forwards requests to the target group:

   ```
   aws elbv2 create-rule --listener-arn listener-arn --priority 10 \
   --actions Type=forward,TargetGroupArn=targetgroup-arn
   ```

## Add Path\-Based Routing<a name="path-based-routing-aws-cli"></a>

If you have a listener with a default rule that forwards requests to one target group, you can add a rule that forwards requests to another target group based on URL\. For example, you can route general requests to one target group and requests to display images to another target group\.

**To add a rule to a listener with a path pattern**

1. Use the [create\-target\-group](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-target-group.html) command to create a target group:

   ```
   aws elbv2 create-target-group --name my-targets --protocol HTTP --port 80 \
   --vpc-id vpc-12345678
   ```

1. Use the [register\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/register-targets.html) command to register your instances with your target group:

   ```
   aws elbv2 register-targets --target-group-arn targetgroup-arn  \
   --targets Id=i-12345678 Id=i-23456789
   ```

1. Use the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) command to add a rule to your listener that forwards requests to the target group if the URL contains the specified pattern:

   ```
   aws elbv2 create-rule --listener-arn listener-arn --priority 10 \
   --conditions Field=path-pattern,Values='/img/*' \
   --actions Type=forward,TargetGroupArn=targetgroup-arn
   ```

## Delete Your Load Balancer<a name="delete-aws-cli"></a>

When you no longer need your load balancer and target group, you can delete them as follows:

```
aws elbv2 delete-load-balancer --load-balancer-arn loadbalancer-arn
aws elbv2 delete-target-group --target-group-arn targetgroup-arn
```