# Security groups for your Application Load Balancer<a name="load-balancer-update-security-groups"></a>

You must ensure that your load balancer can communicate with registered targets on both the listener port and the health check port\. Whenever you add a listener to your load balancer or update the health check port for a target group used by the load balancer to route requests, you must verify that the security groups associated with the load balancer allow traffic on the new port in both directions\. If they do not, you can edit the rules for the currently associated security groups or associate different security groups with the load balancer\.

## Recommended rules<a name="security-group-recommended-rules"></a>

The following rules are recommended for an internet\-facing load balancer\.


| 
| 
| Inbound | 
| --- |
|  Source  |  Port Range  |  Comment  | 
| 0\.0\.0\.0/0 | *listener* | Allow all inbound traffic on the load balancer listener port | 
|   Outbound   | 
| --- |
|  Destination  |  Port Range  |  Comment  | 
| *instance security group* | *instance listener* | Allow outbound traffic to instances on the instance listener port | 
| *instance security group* | *health check* | Allow outbound traffic to instances on the health check port | 

The following rules are recommended for an internal load balancer\.


| 
| 
| Inbound | 
| --- |
|  Source  |  Port Range  |  Comment  | 
| *VPC CIDR* | *listener* | Allow inbound traffic from the VPC CIDR on the load balancer listener port | 
|   Outbound   | 
| --- |
|  Destination  |  Port Range  |  Comment  | 
| *instance security group* | *instance listener* | Allow outbound traffic to instances on the instance listener port | 
| *instance security group* | *health check* | Allow outbound traffic to instances on the health check port | 

## Update the associated security groups<a name="update-group"></a>

You can update the security groups associated with your load balancer at any time\.

**To update security groups using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Description** tab, under **Security**, choose **Edit security groups**\.

1. To associate a security group with your load balancer, select it\. To remove a security group from your load balancer, clear it\. 

1. Choose **Save**\.

**To update security groups using the AWS CLI**  
Use the [set\-security\-groups](https://docs.aws.amazon.com/cli/latest/reference/elbv2/set-security-groups.html) command\.