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
+ [Host header preservation](#host-header-preservation)
+ [AWS WAF](#load-balancer-waf)
+ [Create a load balancer](create-application-load-balancer.md)
+ [Update Availability Zones](load-balancer-subnets.md)
+ [Update security groups](load-balancer-update-security-groups.md)
+ [Update the address type](load-balancer-ip-address-type.md)
+ [Update tags](load-balancer-tags.md)
+ [Delete a load balancer](load-balancer-delete.md)

## Subnets for your load balancer<a name="subnets-load-balancer"></a>

When you create an Application Load Balancer, you must specify one of the following types of subnets: Availability Zone, Local Zone, or Outpost\.<a name="availability-zones"></a>

**Availability Zones**

You must select at least two Availability Zone subnets\. The following restrictions apply:
+ Each subnet must be from a different Availability Zone\.
+ To ensure that your load balancer can scale properly, verify that each Availability Zone subnet for your load balancer has a CIDR block with at least a `/27` bitmask \(for example, `10.0.0.0/27`\) and at least 8 free IP addresses per subnet\. Your load balancer uses these IP addresses to establish connections with the targets\. Depending on your traffic profile, the load balancer can scale higher and consume up to a maximum of 100 IP addresses distributed across all enabled subnets\. <a name="local-zones"></a>

**Local Zones**

You can specify one or more Local Zone subnets\. The following restrictions apply:
+ You cannot use AWS WAF with the load balancer\.
+ You cannot use a Lambda function as a target\.<a name="outposts"></a>

**Outposts**

You can specify a single Outpost subnet\. The following restrictions apply:
+ You must have installed and configured an Outpost in your on\-premises data center\. You must have a reliable network connection between your Outpost and its AWS Region\. For more information, see the [AWS Outposts User Guide](https://docs.aws.amazon.com/outposts/latest/userguide/)\.
+ The load balancer requires two `large` instances on the Outpost for the load balancer nodes\. The supported instance types are shown in the following table\. The load balancer scales as needed, resizing the nodes one size at a time \(from `large` to `xlarge`, then `xlarge` to `2xlarge`, and then `2xlarge` to `4xlarge`\)\. After scaling the nodes to the largest instance size, if you need additional capacity, the load balancer adds `4xlarge` instances as load balancer nodes\. If you do not have sufficient instance capacity or available IP addresses to scale the load balancer, the load balancer reports an event to the [AWS Health Dashboard](https://phd.aws.amazon.com/) and the load balancer state is `active_impaired`\.
+ You can register targets by instance ID or IP address\. If you register targets in the AWS Region for the Outpost, they are not used\.
+ The following features are not available: Lambda functions as targets, AWS WAF integration, sticky sessions, authentication support, and integration with AWS Global Accelerator\.

An Application Load Balancer can be deployed on c5/c5d, m5/m5d, or r5/r5d instances on an Outpost\. The following table shows the size and EBS volume per instance type that the load balancer can use on an Outpost: 


| Instance type and size | EBS volume \(GB\) | 
| --- | --- | 
| c5/c5d | 
| large | 50 | 
| xlarge | 50 | 
| 2xlarge | 50 | 
| 4xlarge | 100 | 
| m5/m5d | 
| large | 50 | 
| xlarge | 50 | 
| 2xlarge | 100 | 
| 4xlarge | 100 | 
| r5/r5d | 
| large | 50 | 
| xlarge | 100 | 
| 2xlarge | 100 | 
| 4xlarge | 100 | 

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
The name of the Amazon S3 bucket for the access logs\. This attribute is required if access logs are enabled\. For more information, see [Enable access logs](enable-access-logging.md)\.

`access_logs.s3.prefix`  
The prefix for the location in the Amazon S3 bucket\.

`deletion_protection.enabled`  
Indicates whether deletion protection is enabled\. The default is `false`\.

`idle_timeout.timeout_seconds`  
The idle timeout value, in seconds\. The default is 60 seconds\. 

`ipv6.deny_all_igw_traffic`  
Blocks internet gateway \(IGW\) access to the load balancer, preventing unintended access to your internal load balancer through an internet gateway\. It is set to `false` for internet\-facing load balancers and `true` for internal load balancers\. This attribute does not prevent non\-IGW internet access \(such as, through peering, Transit Gateway, AWS Direct Connect, or AWS VPN\)\.

`routing.http.desync_mitigation_mode`  
Determines how the load balancer handles requests that might pose a security risk to your application\. The possible values are `monitor`, `defensive`, and `strictest`\. The default is `defensive`\.

`routing.http.drop_invalid_header_fields.enabled`  
Indicates whether HTTP headers with header fields that are not valid are removed by the load balancer \(`true`\), or routed to targets \(`false`\)\. The default is `false`\. Elastic Load Balancing requires that valid HTTP header names conform to the regular expression `[-A-Za-z0-9]+`, as described in the HTTP Field Name Registry\. Each name consists of alphanumeric characters or hyphens\. Select `true` if you want HTTP headers that do not conform to this pattern, to be removed from requests\.

`routing.http.preserve_host_header.enabled`  
Indicates whether the Application Load Balancer should preserve the `Host` header in the HTTP request and send it to targets without any change\. The possible values are `true` and `false`\. The default is `false`\.

`routing.http.x_amzn_tls_version_and_cipher_suite.enabled`  
Indicates whether the two headers \(`x-amzn-tls-version` and `x-amzn-tls-cipher-suite`\), which contain information about the negotiated TLS version and cipher suite, are added to the client request before sending it to the target\. The `x-amzn-tls-version` header has information about the TLS protocol version negotiated with the client, and the `x-amzn-tls-cipher-suite` header has information about the cipher suite negotiated with the client\. Both headers are in OpenSSL format\. The possible values for the attribute are `true` and `false`\. The default is `false`\.

`routing.http.xff_client_port.enabled`  
Indicates whether the `X-Forwarded-For` header should preserve the source port that the client used to connect to the load balancer\. The possible values are `true` and `false`\. The default is `false`\.

`routing.http.xff_header_processing.mode`  
Enables you to modify, preserve, or remove the `X-Forward-For` header in the HTTP request before the Application Load Balancer sends the request to the target\. The possible values are `append`, `preserve`, and `remove`\. The default is `append`\.  
+ If the value is `append`, the Application Load Balancer adds the client IP address \(of the last hop\) to the `X-Forward-For` header in the HTTP request before it sends it to targets\.
+ If the value is `preserve`, the Application Load Balancer preserves the `X-Forward-For` header in the HTTP request, and sends it to targets without any change\.
+ If the value is `remove`, the Application Load Balancer removes the `X-Forward-For` header in the HTTP request before it sends it to targets\.

`routing.http2.enabled`  
Indicates whether HTTP/2 is enabled\. The default is `true`\.

`waf.fail_open.enabled`  
Indicates whether to allow a AWS WAF\-enabled load balancer to route requests to targets if it is unable to forward the request to AWS WAF\. The possible values are `true` and `false`\. The default is `false`\.

**Note**  
The `routing.http.drop_invalid_header_fields.enabled` attribute was introduced to offer HTTP desync protection\. The `routing.http.desync_mitigation_mode` attribute was added to provide more comprehensive protection from HTTP desync for your applications\. You aren't required to use both attributes and may choose either, depending on your application's requirements\.

## IP address type<a name="ip-address-type"></a>

You can set the types of IP addresses that clients can use to access your internet\-facing and internal load balancers\.

The following are the IP address types:

`ipv4`  
Clients must connect to the load balancer using IPv4 addresses \(for example, 192\.0\.2\.1\)

`dualstack`  
Clients can connect to the load balancer using both IPv4 addresses \(for example, 192\.0\.2\.1\) and IPv6 addresses \(for example, 2001:0db8:85a3:0:0:8a2e:0370:7334\)\.

**Dualstack load balancer considerations**
+ The load balancer communicates with targets based on the IP address type of the target group\.
+ When you enable dualstack mode for the load balancer, Elastic Load Balancing provides an AAAA DNS record for the load balancer\. Clients that communicate with the load balancer using IPv4 addresses resolve the A DNS record\. Clients that communicate with the load balancer using IPv6 addresses resolve the AAAA DNS record\.
+ Access to your internal dualstack load balancers through the internet gateway is blocked to prevent unintended internet access\. However, this does not prevent non\-IWG internet access \(such as, through peering, Transit Gateway, AWS Direct Connect, or AWS VPN\)\. 

## Connection idle timeout<a name="connection-idle-timeout"></a>

For each request that a client makes through a load balancer, the load balancer maintains two connections\. The front\-end connection is between a client and the load balancer\. The backend connection is between the load balancer and a target\. The load balancer has a configured idle timeout period that applies to its connections\. If no data has been sent or received by the time that the idle timeout period elapses, the load balancer closes the connection\. To ensure that lengthy operations such as file uploads have time to complete, send at least 1 byte of data before each idle timeout period elapses, and increase the length of the idle timeout period as needed\.

For backend connections, we recommend that you enable the HTTP keep\-alive option for your EC2 instances\. You can enable HTTP keep\-alive in the web server settings for your EC2 instances\. If you enable HTTP keep\-alive, the load balancer can reuse backend connections until the keep\-alive timeout expires\. We also recommend that you configure the idle timeout of your application to be larger than the idle timeout configured for the load balancer\. Otherwise, if the application closes the TCP connection to the load balancer ungracefully, the load balancer might send a request to the application before it receives the packet indicating that the connection is closed\. If this is the case, then the load balancer sends an HTTP 502 Bad Gateway error to the client\.

By default, Elastic Load Balancing sets the idle timeout value for your load balancer to 60 seconds\. Use the following procedure to set a different idle timeout value\.

**To update the idle timeout value using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit load balancer attributes** page, enter a value for **Idle timeout**, in seconds\. The valid range is from 1 through 4000\.

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

Desync mitigation mode protects your application from issues due to HTTP desync\. The load balancer classifies each request based on its threat level, allows safe requests, and then mitigates risk as specified by the mitigation mode that you specify\. The desync mitigation modes are monitor, defensive, and strictest\. The default is the defensive mode, which provides durable mitigation against HTTP desync while maintaining the availability of your application\. You can switch to strictest mode to ensure that your application receives only requests that comply with [RFC 7230](https://tools.ietf.org/html/rfc7230)\.

The http\_desync\_guardian library analyzes HTTP requests to prevent HTTP desync attacks\. For more information, see [HTTP Desync Guardian](https://github.com/aws/http-desync-guardian) on GitHub\.

**Classifications**

The classifications are as follows:
+ Compliant — Request complies with RFC 7230 and poses no known security threats\.
+ Acceptable — Request does not comply with RFC 7230 but poses no known security threats\.
+ Ambiguous — Request does not comply with RFC 7230 but poses a risk, as various web servers and proxies could handle it differently\.
+ Severe — Request poses a high security risk\. The load balancer blocks the request, serves a 400 response to the client, and closes the client connection\.

If a request does not comply with RFC 7230, the load balancer increments the `DesyncMitigationMode_NonCompliant_Request_Count` metric\. For more information, see [Application Load Balancer metrics](load-balancer-cloudwatch-metrics.md#load-balancer-metrics-alb)\.

The classification for each request is included in the load balancer access logs\. If the request does not comply, the access logs include a classification reason code\. For more information, see [Classification reasons](load-balancer-access-logs.md#classification-reasons)\.

**Modes**  
The following table describes how Application Load Balancers treat requests based on mode and classification\.


| Classification | Monitor mode | Defensive mode | Strictest mode | 
| --- | --- | --- | --- | 
| Compliant | Allowed | Allowed | Allowed | 
| Acceptable | Allowed | Allowed | Blocked | 
| Ambiguous | Allowed | Allowed¹ | Blocked | 
| Severe | Allowed | Blocked | Blocked | 

¹ Routes the requests but closes the client and target connections\. You might incur additional charges if your load balancer receives a large number of Ambiguous requests in Defensive mode\. This is because the increased number of new connections per second contributes to the Load Balancer Capacity Units \(LCU\) used per hour\. You can use the `NewConnectionCount` metric to compare how your load balancer establishes new connections in Monitor mode and Defensive mode\.

**To update desync mitigation mode using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. For **Desync mitigation mode**, choose **Monitor**, **Defensive**, or **Strictest**\.

1. Choose **Save**\.

**To update desync mitigation mode using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command with the `routing.http.desync_mitigation_mode` attribute set to `monitor`, `defensive`, or `strictest`\.

## Host header preservation<a name="host-header-preservation"></a>

When you enable the **Preserve host header **attribute, the Application Load Balancer preserves the `Host` header in the HTTP request, and sends the header to targets without any modification\. If the Application Load Balancer receives multiple `Host` headers, it preserves all of them\. Listener rules are applied only to the first `Host` header received\.

By default, when the **Preserve host header **attribute is not enabled, the Application Load Balancer modifies the `Host` header in the following manner: 

**When host header preservation is not enabled, and listener port is a non\-default port**: When not using the default ports \(ports 80 or 443\) we append the port number to the host header if it isn’t already appended by the client\. For example, the `Host` header in the HTTP request with `Host: www.example.com` would be modified to `Host: www.example.com:8080`, if the listener port is a non\-default port such as `8080`\. 

**When host header preservation is not enabled, and the listener port is a default port \(port 80 or 443\)**: For default listener ports \(either port 80 or 443\), we do not append the port number to the outgoing host header\. Any port number that was already in the incoming host header, is removed\. 

The following table shows more examples of how Application Load Balancers treat host headers in the HTTP request based on listener port\.


|  Listener port  |  Example request  |  Host header in the request  |  Host header preservation is disabled \(default behavior\)  | Host header preservation is enabled | 
| --- | --- | --- | --- | --- | 
| Request is sent on default HTTP/HTTPS listener\. | GET /index\.html HTTP/1\.1 Host: example\.com | example\.com | example\.com | example\.com | 
| Request is sent on default HTTP listener and host header has a port \(80 or 443\)\. | GET /index\.html HTTP/1\.1 Host: example\.com:80 | example\.com:80 | example\.com | example\.com:80 | 
| Request has an absolute path\. | GET https://dns\_name/index\.html HTTP/1\.1 Host: example\.com | example\.com | dns\_name | example\.com | 
| Request is sent on a non\-default listener port and host header has port \(for example, 8080\)\. | GET /index\.html HTTP/1\.1 Host: example\.com | example\.com | example\.com:8080 | example\.com | 

**To enable host header preservation**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **Load Balancing**, choose **Load Balancers**\.

1. Select the load balancer you want to use\.

1. On the **Description** tab, choose **Edit attributes**\.

1. For **Preserve host header **, choose **Enable**\.

1. Choose **Save**\.

**To enable host header preservation using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command with the `routing.http.preserve_host_header.enabled` attribute set to `true`\.

## Application Load Balancers and AWS WAF<a name="load-balancer-waf"></a>

You can use AWS WAF with your Application Load Balancer to allow or block requests based on the rules in a web access control list \(web ACL\)\. For more information, see [Working with web ACLs](https://docs.aws.amazon.com/waf/latest/developerguide/web-acl-working-with.html) in the *AWS WAF Developer Guide*\.

To check whether your load balancer integrates with AWS WAF, select your load balancer in the AWS Management Console and choose the **Integrated services** tab\.

By default, if the load balancer cannot get a response from AWS WAF, it returns an HTTP 500 error and does not forward the request\. If you need your load balancer to forward requests to targets even if it is unable to contact AWS WAF, you can enable the AWS WAF fail open attribute\.

**To enable AWS WAF fail open using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. For **AWS WAF fail open**, choose **Enable**\.

1. Choose **Save**\.

**To enable AWS WAF fail open using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command with the `waf.fail_open.enabled` attribute set to `true`\.