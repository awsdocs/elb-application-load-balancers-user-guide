# Application Load Balancers<a name="application-load-balancers"></a>

A *load balancer* serves as the single point of contact for clients\. Clients send requests to the load balancer, and the load balancer sends them to targets, such as EC2 instances\. To configure your load balancer, you create [target groups](load-balancer-target-groups.md), and then register targets with your target groups\. You also create [listeners](load-balancer-listeners.md) to check for connection requests from clients, and listener rules to route requests from clients to the targets in one or more target groups\.

For more information, see [How Elastic Load Balancing works](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html) in the *Elastic Load Balancing User Guide*\.

**Topics**
+ [Subnets for your load balancer](#subnets-load-balancer)
+ [Load balancer security groups](#load-balancer-security-groups)
+ [Load balancer state](#load-balancer-state)
+ [Load balancer attributes](#load-balancer-attributes)
+ [IP address type](#ip-address-type)
+ [Connection idle timeout](#connection-idle-timeout)
+ [Deletion protection](#deletion-protection)
+ [Desync mitigation mode](#desync-mitigation-mode)
+ [Application Load Balancers and AWS WAF](#load-balancer-waf)
+ [Create a load balancer](create-application-load-balancer.md)
+ [Update Availability Zones](load-balancer-subnets.md)
+ [Update security groups](load-balancer-update-security-groups.md)
+ [Update the address type](load-balancer-ip-address-type.md)
+ [Update tags](load-balancer-tags.md)
+ [Delete a load balancer](load-balancer-delete.md)

## Subnets for your load balancer<a name="subnets-load-balancer"></a>

When you create an Application Load Balancer, you must specify one of the following types of subnets: Availability Zone, Local Zone, or Outpost\.

**Availability Zones**

You must select at least two Availability Zone subnets\. The following restrictions apply:
+ Each subnet must be from a different Availability Zone\.
+ To ensure that your load balancer can scale properly, verify that each Availability Zone subnet for your load balancer has a CIDR block with at least a `/27` bitmask \(for example, `10.0.0.0/27`\) and at least 8 free IP addresses\. Your load balancer uses these IP addresses to establish connections with the targets\.

**Local Zones**

You can specify a one or more Local Zone subnets\. The following restrictions apply:
+ You cannot use AWS WAF with the load balancer\.
+ You cannot use a Lambda function as a target\.

**Outposts**

You can specify a single Outpost subnet\. The following restrictions apply:
+ You must have installed and configured an Outpost in your on\-premises data center\. You must have a reliable network connection between your Outpost and its AWS Region\. For more information, see the [AWS Outposts User Guide](https://docs.aws.amazon.com/outposts/latest/userguide/)\.
+ The load balancer requires two instances on the Outpost for the load balancer nodes\. The supported instances are the general purpose, compute optimized, and memory optimized instances\. Initially, the instances are `large` instances\. The load balancer scales as needed, from `large` to `xlarge`, `xlarge` to `2xlarge`, and `2xlarge` to `4xlarge`\. If you need additional capacity, the load balancer adds `4xlarge` instances\. If you do not have sufficient instance capacity or available IP addresses to scale the load balancer, the load balancer reports an event to the [AWS Personal Health Dashboard](https://phd.aws.amazon.com/) and the load balancer state is `active_impaired`\.
+ You can register targets by instance ID or IP address\. If you register targets in the AWS Region for the Outpost, they are not used\.
+ The following features are not available: Lambda functions as targets, AWS WAF integration, authentication support, and integration with AWS Global Accelerator\.

## Load balancer security groups<a name="load-balancer-security-groups"></a>

A *security group* acts as a firewall that controls the traffic allowed to and from your load balancer\. You can choose the ports and protocols to allow for both inbound and outbound traffic\.

The rules for the security groups that are associated with your load balancer must allow traffic in both directions on both the listener and the health check ports\. Whenever you add a listener to a load balancer or update the health check port for a target group, you must review your security group rules to ensure that they allow traffic on the new port in both directions\. For more information, see [Recommended rules](load-balancer-update-security-groups.md#security-group-recommended-rules)\.

## Load balancer state<a name="load-balancer-state"></a>

A load balancer can be in one of the following states:

`provisioning`  
The load balancer is being set up\.

`active`  
The load balancer is fully set up and ready to route traffic\.

`active_impaired`  
The load balancer is routing traffic but does not have the resources it needs to scale\.

`failed`  
The load balancer could not be set up\.

## Load balancer attributes<a name="load-balancer-attributes"></a>

The following are the load balancer attributes:

`access_logs.s3.enabled`  
Indicates whether access logs stored in Amazon S3 are enabled\. The default is `false`\.

`access_logs.s3.bucket`  
The name of the Amazon S3 bucket for the access logs\. This attribute is required if access logs are enabled\. For more information, see [Bucket permissions](load-balancer-access-logs.md#access-logging-bucket-permissions)\.

`access_logs.s3.prefix`  
The prefix for the location in the Amazon S3 bucket\.

`deletion_protection.enabled`  
Indicates whether deletion protection is enabled\. The default is `false`\.

`idle_timeout.timeout_seconds`  
The idle timeout value, in seconds\. The default is 60 seconds\.

`routing.http.desync_mitigation_mode`  
Determines how the load balancer handles requests that might pose a security risk to your application\. The possible values are `monitor`, `defensive`, and `strictest`\. The default is `defensive`\.

`routing.http.drop_invalid_header_fields.enabled`  
Indicates whether HTTP headers with header fields that are not valid are removed by the load balancer \(`true`\), or routed to targets \(`false`\)\. The default is `false`\. Elastic Load Balancing requires that message header names contain only alphanumeric characters and hyphens\.

`routing.http2.enabled`  
Indicates whether HTTP/2 is enabled\. The default is `true`\.

## IP address type<a name="ip-address-type"></a>

You can set the types of IP addresses that clients can use with your internet\-facing load balancer\. Clients must use IPv4 addresses with internal load balancers\.

The following are the IP address types:

`ipv4`  
Clients must connect to the load balancer using IPv4 addresses \(for example, 192\.0\.2\.1\)

`dualstack`  
Clients can connect to the load balancer using both IPv4 addresses \(for example, 192\.0\.2\.1\) and IPv6 addresses \(for example, 2001:0db8:85a3:0:0:8a2e:0370:7334\)\.

When you enable dual\-stack mode for the load balancer, Elastic Load Balancing provides an AAAA DNS record for the load balancer\. Clients that communicate with the load balancer using IPv4 addresses resolve the A DNS record\. Clients that communicate with the load balancer using IPv6 addresses resolve the AAAA DNS record\.

The load balancer communicates with targets using IPv4 addresses, regardless of how the client communicates with the load balancer\. Therefore, the targets do not need IPv6 addresses\.

For more information, see [IP address types for your Application Load Balancer](load-balancer-ip-address-type.md)\.

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

## Desync mitigation mode<a name="desync-mitigation-mode"></a>

Desync mitigation mode protects your application from issues due to HTTP Desync\. The load balancer classifies each request based on its threat level, allows safe requests, and then mitigates risk as specified by the mitigation mode that you specify\. The desync mitigation modes are monitor, defensive, and strictest\. The default is the defensive mode, which provides durable mitigation against HTTP desync while maintaining the availability of your application\. You can switch to strictest mode to ensure that your application receives only requests that comply with RFC 7230\.

The http\_desync\_guardian library analyzes HTTP requests to prevent HTTP Desync attacks\. For more information, see [HTTP Desync Guardian](https://github.com/aws/http-desync-guardian) on github\.

**Classifications**

The classifications are as follows\. For more information, see [Classification reasons](load-balancer-access-logs.md#classification-reasons)\.
+ Compliant — Request complies with RFC 7230 and poses no known security threats\.
+ Acceptable — Request does not comply with RFC 7230 but poses no known security threats\.
+ Ambiguous — Request does not comply with RFC 7230 but poses a risk, as various web servers and proxies could handle it differently\.
+ Severe — Request poses a high security risk\. The load balancer blocks the request, serves a 400 response to the client, and closes the client connection\.

If a request does not comply with RFC 7230, the load balancer increments the `DesyncMitigationMode_NonCompliant_Request_Count` metric\. For more information, see [Application Load Balancer metrics](load-balancer-cloudwatch-metrics.md#load-balancer-metrics-alb)\.

**Modes**  
The following table describes how Application Load Balancers treat requests based on mode and classification\.


| Classification | Monitor mode | Defensive mode | Strictest mode | 
| --- | --- | --- | --- | 
| Compliant | Allowed | Allowed | Allowed | 
| Acceptable | Allowed | Allowed | Blocked | 
| Ambiguous | Allowed | Allowed¹ | Blocked | 
| Severe | Allowed | Blocked | Blocked | 

¹ Routes the requests but closes the client and target connections\.

**To update desync mitigation mode using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. For **Desync mitigation mode**, choose **Monitor**, **Defensive**, or **Strictest**\.

1. Choose **Save**\.

**To update desync mitigation mode using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command with the `routing.http.desync_mitigation_mode` attribute set to `monitor`, `defensive`, or `strictest`\.

## Application Load Balancers and AWS WAF<a name="load-balancer-waf"></a>

You can use AWS WAF with your Application Load Balancer to allow or block requests based on the rules in a web access control list \(web ACL\)\. For more information, see [Working with web ACLs](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl-working-with.html) in the *AWS WAF Developer Guide*\.

To check whether your load balancer integrates with AWS WAF, select your load balancer in the AWS Management Console and choose the **Integrated services** tab\.