# Application Load Balancers<a name="application-load-balancers"></a>

A *load balancer* serves as the single point of contact for clients\. Clients send requests to the load balancer, and the load balancer sends them to targets, such as EC2 instances, in two or more Availability Zones\. To configure your load balancer, you create [target groups](load-balancer-target-groups.md), and then register targets with your target groups\. You also create [listeners](load-balancer-listeners.md) to check for connection requests from clients, and listener rules to route requests from clients to the targets in one or more target groups\.

For more information, see [How Elastic Load Balancing Works](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html) in the *Elastic Load Balancing User Guide*\.

**Topics**
+ [Subnets for Your Load Balancer](#subnets-load-balancer)
+ [Load Balancer Security Groups](#load-balancer-security-groups)
+ [Load Balancer State](#load-balancer-state)
+ [Load Balancer Attributes](#load-balancer-attributes)
+ [IP Address Type](#ip-address-type)
+ [Deletion Protection](#deletion-protection)
+ [Connection Idle Timeout](#connection-idle-timeout)
+ [Application Load Balancers and AWS WAF](#load-balancer-waf)
+ [Create a Load Balancer](create-application-load-balancer.md)
+ [Update Availability Zones](load-balancer-subnets.md)
+ [Update Security Groups](load-balancer-update-security-groups.md)
+ [Update the Address Type](load-balancer-ip-address-type.md)
+ [Update Tags](load-balancer-tags.md)
+ [Delete a Load Balancer](load-balancer-delete.md)

## Subnets for Your Load Balancer<a name="subnets-load-balancer"></a>

When you create a load balancer, you must specify one public subnet from at least two Availability Zones\. You can specify only one public subnet per Availability Zone\.

To ensure that your load balancer can scale properly, verify that each subnet for your load balancer has a CIDR block with at least a `/27` bitmask \(for example, `10.0.0.0/27`\) and has at least 8 free IP addresses\. Your load balancer uses these IP addresses to establish connections with the targets\.

## Load Balancer Security Groups<a name="load-balancer-security-groups"></a>

A *security group* acts as a firewall that controls the traffic allowed to and from your load balancer\. You can choose the ports and protocols to allow for both inbound and outbound traffic\.

The rules for the security groups associated with your load balancer security group must allow traffic in both directions on both the listener and the health check ports\. Whenever you add a listener to a load balancer or update the health check port for a target group, you must review your security group rules to ensure that they allow traffic on the new port in both directions\. For more information, see [Recommended Rules](load-balancer-update-security-groups.md#security-group-recommended-rules)\.

## Load Balancer State<a name="load-balancer-state"></a>

A load balancer can be in one of the following states:

`provisioning`  
The load balancer is being set up\.

`active`  
The load balancer is fully set up and ready to route traffic\.

`failed`  
The load balancer could not be set up\.

## Load Balancer Attributes<a name="load-balancer-attributes"></a>

The following are the load balancer attributes:

`access_logs.s3.enabled`  
Indicates whether access logs stored in Amazon S3 are enabled\. The default is `false`\.

`access_logs.s3.bucket`  
The name of the S3 bucket for the access logs\. This attribute is required if access logs are enabled\. For more information, see [Bucket Permissions](load-balancer-access-logs.md#access-logging-bucket-permissions)\.

`access_logs.s3.prefix`  
The prefix for the location in the S3 bucket\.

`deletion_protection.enabled`  
Indicates whether deletion protection is enabled\. The default is `false`\.

`idle_timeout.timeout_seconds`  
The idle timeout value, in seconds\. The default is 60 seconds\.

`routing.http2.enabled`  
Indicates whether HTTP/2 is enabled\. The default is `true`\.

## IP Address Type<a name="ip-address-type"></a>

You can set the IP address type of your Internet\-facing load balancer when you create it or after it is active\. Note that internal load balancers must use IPv4 addresses\.

The following are the load balancer IP address types:

`ipv4`  
The load balancer supports only IPv4 addresses \(for example, 192\.0\.2\.1\)

`dualstack`  
The load balancer supports both IPv4 and IPv6 addresses \(for example, 2001:0db8:85a3:0:0:8a2e:0370:7334\)\.

Clients that communicate with the load balancer using IPv4 addresses resolve the A record and clients that communicate with the load balancer using IPv6 addresses resolve the AAAA record\. However, the load balancer communicates with its targets using IPv4 addresses, regardless of how the client communicates with the load balancer\.

For more information, see [IP Address Types for Your Application Load Balancer](load-balancer-ip-address-type.md)\.

## Deletion Protection<a name="deletion-protection"></a>

To prevent your load balancer from being deleted accidentally, you can enable deletion protection\. By default, deletion protection is disabled for your load balancer\.

If you enable deletion protection for your load balancer, you must disable it before you can delete the load balancer\.

**To enable deletion protection using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit load balancer attributes** page, select **Enable** for **Delete Protection**, and then choose **Save**\.

1. Choose **Save**\.

**To disable deletion protection using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit load balancer attributes** page, clear **Enable** for **Delete Protection**, and then choose **Save**\.

1. Choose **Save**\.

**To enable or disable deletion protection using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command with the `deletion_protection.enabled` attribute\.

## Connection Idle Timeout<a name="connection-idle-timeout"></a>

For each request that a client makes through a load balancer, the load balancer maintains two connections\. A front\-end connection is between a client and the load balancer, and a back\-end connection is between the load balancer and a target\. The load balancer manages an idle timeout that is triggered when no data is sent over a front\-end connection for a specified time period\. If no data has been sent or received by the time that the idle timeout period elapses, the load balancer closes the connection\.

By default, Elastic Load Balancing sets the idle timeout value to 60 seconds\. Therefore, if the target doesn't send some data at least every 60 seconds while the request is in flight, the load balancer can close the front\-end connection\. To ensure that lengthy operations such as file uploads have time to complete, send at least 1 byte of data before each idle timeout period elapses, and increase the length of the idle timeout period as needed\.

For back\-end connections, we recommend that you enable the HTTP keep\-alive option for your EC2 instances\. You can enable HTTP keep\-alive in the web server settings for your EC2 instances\. If you enable HTTP keep\-alive, the load balancer can reuse back\-end connections until the keep\-alive timeout expires\. We also recommend that you configure the idle timeout of your application to be larger than the idle timeout configured for the load balancer\.

**To update the idle timeout value using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit load balancer attributes** page, type a value for **Idle timeout**, in seconds\. The valid range is 1\-4000\. The default is 60 seconds\.

1. Choose **Save**\.

**To update the idle timeout value using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command with the `idle_timeout.timeout_seconds` attribute\.

## Application Load Balancers and AWS WAF<a name="load-balancer-waf"></a>

You can use AWS WAF with your Application Load Balancer to allow or block requests based on the rules in a web access control list \(web ACL\)\. For more information, see [Working with Web ACLs](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl-working-with.html) in the *AWS WAF Developer Guide*\.

To check whether your load balancer integrates with AWS WAF, select your load balancer in the AWS Management Console and choose the **Integrated services** tab\.