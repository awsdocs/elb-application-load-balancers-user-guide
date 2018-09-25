# IP Address Types for Your Application Load Balancer<a name="load-balancer-ip-address-type"></a>

You can configure your Application Load Balancer to route IPv4 traffic only or to route both IPv4 and IPv6 traffic\. For more information, see [IP Address Type](application-load-balancers.md#ip-address-type)\.

**IPv6 Requirements**
+ An Internet\-facing load balancer\.
+ Your virtual private cloud \(VPC\) has subnets with associated IPv6 CIDR blocks\. For more information, see [IPv6 Addresses](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-instance-addressing.html#ipv6-addressing) in the *Amazon EC2 User Guide*\.

**To update the IP address type using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. Choose **Actions**, **Edit IP address type**\.

1. For **IP address type**, choose **ipv4** to support IPv4 addresses only or **dualstack** to support both IPv4 and IPv6 addresses\.

1. Choose **Save**\.

**To update the IP address type using the AWS CLI**  
Use the [set\-ip\-address\-type](https://docs.aws.amazon.com/cli/latest/reference/elbv2/set-ip-address-type.html) command\.