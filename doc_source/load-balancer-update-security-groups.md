# Security groups for your Application Load Balancer<a name="load-balancer-update-security-groups"></a>

You must ensure that your load balancer can communicate with registered targets on both the listener port and the health check port\. Whenever you add a listener to your load balancer or update the health check port for a target group used by the load balancer to route requests, you must verify that the security groups associated with the load balancer allow traffic on the new port in both directions\. If they do not, you can edit the rules for the currently associated security groups or associate different security groups with the load balancer\. In a VPC, you provide the security group for your load balancer, which enables you to choose the ports and protocols to allow\. For example, you can open Internet Control Message Protocol \(ICMP\) connections for the load balancer to respond to ping requests \(however, ping requests are not forwarded to any instances\)\.

## Recommended rules<a name="security-group-recommended-rules"></a>

The following rules are recommended for an internet\-facing load balancer\.


| 
| 
| Inbound | 
| --- |
|  Source  |  Port Range  |  Comment  | 
|  0\.0\.0\.0/0  |  *listener*  |  Allow all inbound traffic on the load balancer listener port  | 
|   Outbound   | 
| --- |
|  Destination  |  Port Range  |  Comment  | 
|  *instance security group*  |  *instance listener*  |  Allow outbound traffic to instances on the instance listener port  | 
|  *instance security group*  |  *health check*  |  Allow outbound traffic to instances on the health check port  | 

The following rules are recommended for an internal load balancer\.


| 
| 
| Inbound | 
| --- |
|  Source  |  Port Range  |  Comment  | 
|  *VPC CIDR*  |  *listener*  |  Allow inbound traffic from the VPC CIDR on the load balancer listener port  | 
|   Outbound   | 
| --- |
|  Destination  |  Port Range  |  Comment  | 
|  *instance security group*  |  *instance listener*  |  Allow outbound traffic to instances on the instance listener port  | 
|  *instance security group*  |  *health check*  |  Allow outbound traffic to instances on the health check port  | 

The following rules are recommended for an Application Load Balancer used as a target of a Network Load Balancer\.


| 
| 
| Inbound | 
| --- |
|  Source  |  Port Range  |  Comment  | 
|  *client IP addresses/CIDR*  |  *`alb `listener*  |  Allow inbound client traffic on the load balancer listener port  | 
|  *VPC CIDR*  |  *`alb `listener*  |  Allow inbound client traffic via AWS PrivateLink on the load balancer listener port  | 
|  *VPC CIDR*  |  *`alb `listener*  |  Allow inbound health traffic from the Network Load Balancer  | 
|   Outbound   | 
| --- |
|  Destination  |  Port Range  |  Comment  | 
|  *instance security group*  |  *instance listener*  |  Allow outbound traffic to instances on the instance listener port  | 
|  *instance security group*  |  *health check*  |  Allow outbound traffic to instances on the health check port  | 

Note that the security groups for your Application Load Balancer use connection tracking to track information about traffic coming from the Network Load Balancer\. This happens regardless of the security group rules set for your Application Load Balancer\. To learn more about Amazon EC2 connection tracking, see [Security group connection tracking](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-group-connection-tracking.html) in the *Amazon EC2 User Guide for Linux Instances*\.

We also recommend that you allow inbound ICMP traffic to support Path MTU Discovery\. For more information, see [Path MTU Discovery](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/network_mtu.html#path_mtu_discovery) in the *Amazon EC2 User Guide for Linux Instances*\.

## Update the associated security groups<a name="update-group"></a>

You can update the security groups associated with your load balancer at any time\.

**To update security groups using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Security** tab, choose **Edit**\.

1. To associate a security group with your load balancer, select it\. To remove a security group association, choose the **X** icon for the security group\.

1. Choose **Save changes**\.

**To update security groups using the AWS CLI**  
Use the [set\-security\-groups](https://docs.aws.amazon.com/cli/latest/reference/elbv2/set-security-groups.html) command\.