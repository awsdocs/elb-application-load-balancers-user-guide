# Application Load Balancers<a name="application-load-balancers"></a>

A *load balancer* serves as the single point of contact for clients\. Clients send requests to the load balancer, and the load balancer sends them to targets, such as EC2 instances\. To configure your load balancer, you create [target groups](load-balancer-target-groups.md), and then register targets with your target groups\. You also create [listeners](load-balancer-listeners.md) to check for connection requests from clients, and listener rules to route requests from clients to the targets in one or more target groups\.

For more information, see [How Elastic Load Balancing works](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html) in the *Elastic Load Balancing User Guide*\.

**Topics**
+ [Subnets for your load balancer](#subnets-load-balancer)
+ [Load balancer security groups](#load-balancer-security-groups)
+ [Load balancer state](#load-balancer-state)
+ [Load balancer attributes](#load-balancer-attributes)
+ [IP address type](#ip-address-type)
+ [Deletion protection](#deletion-protection)
+ [Connection idle timeout](#connection-idle-timeout)
+ [Application Load Balancers and AWS WAF](#load-balancer-waf)
+ [Create a load balancer](create-application-load-balancer.md)
+ [Update Availability Zones](load-balancer-subnets.md)
+ [Update security groups](load-balancer-update-security-groups.md)
+ [Update the address type](load-balancer-ip-address-type.md)
+ [Update tags](load-balancer-tags.md)
+ [Delete a load balancer](load-balancer-delete.md)

## Subnets for your load balancer<a name="subnets-load-balancer"></a>

When you create an Application Load Balancer, you must enable at least two Availability Zones\. To enable an Availability Zone, you specify one subnet from that Availability Zone\.

To ensure that your load balancer can scale properly, verify that each Availability Zone subnet for your load balancer has a CIDR block with at least a `/27` bitmask \(for example, `10.0.0.0/27`\) and has at least 8 free IP addresses\. Your load balancer uses these IP addresses to establish connections with the targets\.

Alternatively, you can specify a Local Zone subnet for your load balancer instead of specifying two Availability Zone subnets\. If you do so, the following restrictions apply:
+ If you enable a Local Zone subnet, you cannot also enable an Availability Zone subnet\.
+ You cannot use AWS WAF with the load balancer\.
+ You cannot use a Lambda function as a target\.

## Load balancer security groups<a name="load-balancer-security-groups"></a>

A *security group* acts as a firewall that controls the traffic allowed to and from your load balancer\. You can choose the ports and protocols to allow for both inbound and outbound traffic\.

The rules for the security groups associated with your load balancer security group must allow traffic in both directions on both the listener and the health check ports\. Whenever you add a listener to a load balancer or update the health check port for a target group, you must review your security group rules to ensure that they allow traffic on the new port in both directions\. For more information, see [Recommended rules](load-balancer-update-security-groups.md#security-group-recommended-rules)\.

## Load balancer state<a name="load-balancer-state"></a>

A load balancer can be in one of the following states:

`provisioning`  
The load balancer is being set up\.

`active`  
The load balancer is fully set up and ready to route traffic\.

`failed`  
The load balancer could not be set up\.

## Load balancer attributes<a name="load-balancer-attributes"></a>

The following are the load balancer attributes:

`access_logs.s3.enabled`  
Indicates whether access logs stored in Amazon S3 are enabled\. The default is `false`\.

`access_logs.s3.bucket`  
The name of the S3 bucket for the access logs\. This attribute is required if access logs are enabled\. For more information, see [Bucket permissions](load-balancer-access-logs.md#access-logging-bucket-permissions)\.

`access_logs.s3.prefix`  
The prefix for the location in the S3 bucket\.

`deletion_protection.enabled`  
Indicates whether deletion protection is enabled\. The default is `false`\.

`idle_timeout.timeout_seconds`  
The idle timeout value, in seconds\. The default is 60 seconds\.

`routing.http.drop_invalid_header_fields.enabled`  
Indicates whether HTTP headers with header fields that are not valid are removed by the load balancer \(`true`\) or routed to targets \(`false`\)\. The default is `false`\. Elastic Load Balancing requires that message header names contain only alphanumeric characters and hyphens\.

`routing.http2.enabled`  
Indicates whether HTTP/2 is enabled\. The default is `true`\.

## IP address type<a name="ip-address-type"></a>

You can set the types of IP addresses that clients can use with your Internet\-facing load balancer\. Clients must use IPv4 addresses with internal load balancers\.

The following are the IP address types:

`ipv4`  
Clients must connect to the load balancer using IPv4 addresses \(for example, 192\.0\.2\.1\)

`dualstack`  
Clients can connect to the load balancer using both IPv4 addresses \(for example, 192\.0\.2\.1\) and IPv6 addresses \(for example, 2001:0db8:85a3:0:0:8a2e:0370:7334\)\.

When you enable dual\-stack mode for the load balancer, Elastic Load Balancing provides an AAAA DNS record for the load balancer\. Clients that communicate with the load balancer using IPv4 addresses resolve the A DNS record\. Clients that communicate with the load balancer using IPv6 addresses resolve the AAAA DNS record\.

The load balancer communicates with targets using IPv4 addresses, regardless of how the client communicates with the load balancer\. Therefore, the targets do not need IPv6 addresses\.

For more information, see [IP address types for your Application Load Balancer](load-balancer-ip-address-type.md)\.

## Deletion protection<a name="deletion-protection"></a>

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

## Connection idle timeout<a name="connection-idle-timeout"></a>

For each request that a client makes through a load balancer, the load balancer maintains two connections\. The front\-end connection is between a client and the load balancer\. The back\-end connection is between the load balancer and a target\. The load balancer has a configured idle timeout period that applies to its connections\. If no data has been sent or received by the time that the idle timeout period elapses, the load balancer closes the connection\. To ensure that lengthy operations such as file uploads have time to complete, send at least 1 byte of data before each idle timeout period elapses, and increase the length of the idle timeout period as needed\.

For back\-end connections, we recommend that you enable the HTTP keep\-alive option for your EC2 instances\. You can enable HTTP keep\-alive in the web server settings for your EC2 instances\. If you enable HTTP keep\-alive, the load balancer can reuse back\-end connections until the keep\-alive timeout expires\. We also recommend that you configure the idle timeout of your application to be larger than the idle timeout configured for the load balancer\.

By default, Elastic Load Balancing sets the idle timeout value for your load balancer to 60 seconds\. Use the following procedure to set a different idle timeout value\.

**To update the idle timeout value using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit load balancer attributes** page, enter a value for **Idle timeout**, in seconds\. The valid range is 1\-4000\.

1. Choose **Save**\.

**To update the idle timeout value using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command with the `idle_timeout.timeout_seconds` attribute\.

## Application Load Balancers and AWS WAF<a name="load-balancer-waf"></a>

You can use AWS WAF with your Application Load Balancer to allow or block requests based on the rules in a web access control list \(web ACL\)\. For more information, see [Working with web ACLs](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl-working-with.html) in the *AWS WAF Developer Guide*\.

To check whether your load balancer integrates with AWS WAF, select your load balancer in the AWS Management Console and choose the **Integrated services** tab\.