# Target groups for your Application Load Balancers<a name="load-balancer-target-groups"></a>

Target groups route requests to one or more registered targets, such as EC2 instances, using the protocol and port number that you specify\. You can register a target with multiple target groups\. You can configure health checks on a per target group basis\. Health checks are performed on all targets registered to a target group that is specified in a listener rule for your load balancer\.

Each target group is used to route requests to one or more registered targets\. When you create each listener rule, you specify a target group and conditions\. When a rule condition is met, traffic is forwarded to the corresponding target group\. You can create different target groups for different types of requests\. For example, create one target group for general requests and other target groups for requests to the microservices for your application\. You can use each target group with only one load balancer\. For more information, see [Application Load Balancer components](introduction.md#application-load-balancer-components)\.

You define health check settings for your load balancer on a per target group basis\. Each target group uses the default health check settings, unless you override them when you create the target group or modify them later on\. After you specify a target group in a rule for a listener, the load balancer continually monitors the health of all targets registered with the target group that are in an Availability Zone enabled for the load balancer\. The load balancer routes requests to the registered targets that are healthy\.

**Topics**
+ [Routing configuration](#target-group-routing-configuration)
+ [Target type](#target-type)
+ [IP address type](#target-group-ip-address-type)
+ [Protocol version](#target-group-protocol-version)
+ [Registered targets](#registered-targets)
+ [Target group attributes](#target-group-attributes)
+ [Routing algorithm](#modify-routing-algorithm)
+ [Deregistration delay](#deregistration-delay)
+ [Slow start mode](#slow-start-mode)
+ [Create a target group](create-target-group.md)
+ [Health checks for your target groups](target-group-health-checks.md)
+ [Cross\-zone load balancing for target groups](disable-cross-zone.md)
+ [Target group health](target-group-health.md)
+ [Register targets with your target group](target-group-register-targets.md)
+ [Sticky sessions for your Application Load Balancer](sticky-sessions.md)
+ [Lambda functions as targets](lambda-functions.md)
+ [Tags for your target group](target-group-tags.md)
+ [Delete a target group](delete-target-group.md)

## Routing configuration<a name="target-group-routing-configuration"></a>

By default, a load balancer routes requests to its targets using the protocol and port number that you specified when you created the target group\. Alternatively, you can override the port used for routing traffic to a target when you register it with the target group\.

Target groups support the following protocols and ports:
+ **Protocols**: HTTP, HTTPS
+ **Ports**: 1\-65535

If a target group is configured with the HTTPS protocol or uses HTTPS health checks, the TLS connections to the targets use the security settings from the `ELBSecurityPolicy-2016-08` policy\. The load balancer establishes TLS connections with the targets using certificates that you install on the targets\. The load balancer does not validate these certificates\. Therefore, you can use self\-signed certificates or certificates that have expired\. Because the load balancer is in a virtual private cloud \(VPC\), traffic between the load balancer and the targets is authenticated at the packet level, so it is not at risk of man\-in\-the\-middle attacks or spoofing even if the certificates on the targets are not valid\.

## Target type<a name="target-type"></a>

When you create a target group, you specify its target type, which determines the type of target you specify when registering targets with this target group\. After you create a target group, you cannot change its target type\.

The following are the possible target types:

`instance`  
The targets are specified by instance ID\.

`ip`  
The targets are IP addresses\.

`lambda`  
The target is a Lambda function\.

When the target type is `ip`, you can specify IP addresses from one of the following CIDR blocks:
+ The subnets of the VPC for the target group
+ 10\.0\.0\.0/8 \([RFC 1918](https://tools.ietf.org/html/rfc1918)\)
+ 100\.64\.0\.0/10 \([RFC 6598](https://tools.ietf.org/html/rfc6598)\)
+ 172\.16\.0\.0/12 \(RFC 1918\)
+ 192\.168\.0\.0/16 \(RFC 1918\)

These supported CIDR blocks enable you to register the following with a target group: ClassicLink instances, instances in a VPC that is peered to the load balancer VPC \(same Region or different Region\), AWS resources that are addressable by IP address and port \(for example, databases\), and on\-premises resources linked to AWS through AWS Direct Connect or a Site\-to\-Site VPN connection\.

**Important**  
You can't specify publicly routable IP addresses\.

If you specify targets using an instance ID, traffic is routed to instances using the primary private IP address specified in the primary network interface for the instance\. If you specify targets using IP addresses, you can route traffic to an instance using any private IP address from one or more network interfaces\. This enables multiple applications on an instance to use the same port\. Each network interface can have its own security group\.

If the target type of your target group is `lambda`, you can register a single Lambda function\. When the load balancer receives a request for the Lambda function, it invokes the Lambda function\. For more information, see [Lambda functions as targets](lambda-functions.md)\.

## IP address type<a name="target-group-ip-address-type"></a>

When creating a new target group, you can select the IP address type of your target group\. This controls the IP version used to communicate with targets and check their health status\. 

Application Load Balancers support both IPv4 and IPv6 target groups\. The default selection is IPv4\.

**Considerations**
+ All IP addresses within a target group must have the same IP address type\. For example, you can't register an IPv4 target with an IPv6 target group\.
+ IPv6 target groups can only be used with `dualstack` load balancers\. 
+ IPv6 target groups only support IP type targets\.

## Protocol version<a name="target-group-protocol-version"></a>

By default, Application Load Balancers send requests to targets using HTTP/1\.1\. You can use the protocol version to send requests to targets using HTTP/2 or gRPC\.

The following table summarizes the result for the combinations of request protocol and target group protocol version\.


| Request protocol | Protocol version | Result | 
| --- | --- | --- | 
| HTTP/1\.1 | HTTP/1\.1 | Success | 
| HTTP/2 | HTTP/1\.1 | Success | 
| gRPC | HTTP/1\.1 | Error | 
| HTTP/1\.1 | HTTP/2 | Error | 
| HTTP/2 | HTTP/2 | Success | 
| gRPC | HTTP/2 | Success if targets support gRPC | 
| HTTP/1\.1 | gRPC | Error | 
| HTTP/2 | gRPC | Success if a POST request | 
| gRPC | gRPC | Success | 

**Considerations for the gRPC protocol version**
+ The only supported listener protocol is HTTPS\.
+ The only supported action type for listener rules is `forward`\.
+ The only supported target types are `instance` and `ip`\.
+ The load balancer parses gRPC requests and routes the gRPC calls to the appropriate target groups based on the package, service, and method\.
+ The load balancer supports unary, client\-side streaming, server\-side streaming, and bi\-directional streaming\.
+ You must provide a custom health check method with the format `/package.service/method`\.
+ You must specify the gRPC status codes to use when checking for a successful response from a target\.
+ You cannot use Lambda functions as targets\.

**Considerations for the HTTP/2 protocol version**
+ The only supported listener protocol is HTTPS\.
+ The only supported action type for listener rules is `forward`\.
+ The only supported target types are `instance` and `ip`\.
+ The load balancer supports streaming from clients\. The load balancer does not support streaming to the targets\.

## Registered targets<a name="registered-targets"></a>

Your load balancer serves as a single point of contact for clients and distributes incoming traffic across its healthy registered targets\. You can register each target with one or more target groups\.

If demand on your application increases, you can register additional targets with one or more target groups in order to handle the demand\. The load balancer starts routing requests to a newly registered target as soon as the registration process completes and the target passes the initial health checks\.

If demand on your application decreases, or you need to service your targets, you can deregister targets from your target groups\. Deregistering a target removes it from your target group, but does not affect the target otherwise\. The load balancer stops routing requests to a target as soon as it is deregistered\. The target enters the `draining` state until in\-flight requests have completed\. You can register the target with the target group again when you are ready for it to resume receiving requests\.

If you are registering targets by instance ID, you can use your load balancer with an Auto Scaling group\. After you attach a target group to an Auto Scaling group, Auto Scaling registers your targets with the target group for you when it launches them\. For more information, see [Attaching a load balancer to your Auto Scaling group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/attach-load-balancer-asg.html) in the *Amazon EC2 Auto Scaling User Guide*\.

**Limits**
+ You cannot register the IP addresses of another Application Load Balancer in the same VPC\. If the other Application Load Balancer is in a VPC that is peered to the load balancer VPC, you can register its IP addresses\.

## Target group attributes<a name="target-group-attributes"></a>

The following target group attributes are supported if the target group type is `instance` or `ip`:

`deregistration_delay.timeout_seconds`  
The amount of time for Elastic Load Balancing to wait before deregistering a target\. The range is 0–3600 seconds\. The default value is 300 seconds\.

`load_balancing.algorithm.type`  
The load balancing algorithm determines how the load balancer selects targets when routing requests\. The value is `round_robin` or `least_outstanding_requests`\. The default is `round_robin`\.

`load_balancing.cross_zone.enabled`  
Indicates whether cross zone load balancing is enabled\. The value is `true`, `false` or `use_load_balancer_configuration`\. The default is `use_load_balancer_configuration`\.

`slow_start.duration_seconds`  
The time period, in seconds, during which the load balancer sends a newly registered target a linearly increasing share of the traffic to the target group\. The range is 30–900 seconds \(15 minutes\)\. The default is 0 seconds \(disabled\)\.

`stickiness.enabled`  
Indicates whether sticky sessions are enabled\. The value is `true` or `false`\. The default is `false`\.

`stickiness.app_cookie.cookie_name`  
The name of the application cookie\. The application cookie name cannot have the following prefixes: `AWSALB`, `AWSALBAPP`, or `AWSALBTG`; they're reserved for use by the load balancer\.

`stickiness.app_cookie.duration_seconds`  
The application\-based cookie expiration period, in seconds\. After this period, the cookie is considered stale\. The minimum value is 1 second and the maximum value is 7 days \(604800 seconds\)\. The default value is 1 day \(86400 seconds\)\.

`stickiness.lb_cookie.duration_seconds`  
The duration\-based cookie expiration period, in seconds\. After this period, the cookie is considered stale\. The minimum value is 1 second and the maximum value is 7 days \(604800 seconds\)\. The default value is 1 day \(86400 seconds\)\.

`stickiness.type`  
The type of stickiness\. The possible values are `lb_cookie` and `app_cookie`\.

`target_group_health.dns_failover.minimum_healthy_targets.count`  
The minimum number of targets that must be healthy\. If the number of healthy targets is below this value, mark the zone as unhealthy in DNS, so that traffic is routed only to healthy zones\. The possible values are `off` or an integer from 1 to the maximum number of targets\. The default is `off`\.

`target_group_health.dns_failover.minimum_healthy_targets.percentage`  
The minimum percentage of targets that must be healthy\. If the percentage of healthy targets is below this value, mark the zone as unhealthy in DNS, so that traffic is routed only to healthy zones\. The possible values are `off` or an integer from 1 to 100\. The default is `off`\.

`target_group_health.unhealthy_state_routing.minimum_healthy_targets.count`  
The minimum number of targets that must be healthy\. If the number of healthy targets is below this value, send traffic to all targets, including unhealthy targets\. The range is 1 to the maximum number of targets\. The default is 1\.

`target_group_health.unhealthy_state_routing.minimum_healthy_targets.percentage`  
The minimum percentage of targets that must be healthy\. If the percentage of healthy targets is below this value, send traffic to all targets, including unhealthy targets\. The possible values are `off` or an integer from 1 to 100\. The default is `off`\.

The following target group attribute is supported if the target group type is `lambda`:

`lambda.multi_value_headers.enabled`  
Indicates whether the request and response headers exchanged between the load balancer and the Lambda function include arrays of values or strings\. The possible values are `true` or `false`\. The default value is `false`\. For more information, see [Multi\-value headers](lambda-functions.md#multi-value-headers)\.

## Routing algorithm<a name="modify-routing-algorithm"></a>

By default, the round robin routing algorithm is used to route requests at the target group level\. You can specify the least outstanding requests routing algorithm instead\.

Consider using least outstanding requests when the requests for your application vary in complexity or your targets vary in processing capability\. Round robin is a good choice when the requests and targets are similar, or if you need to distribute requests equally among targets\. You can compare the effect of round robin versus least outstanding requests using the following CloudWatch metrics: **RequestCount**, **TargetConnectionErrorCount**, and **TargetResponseTime**\.

**Considerations**
+ You cannot enable both least outstanding requests and slow start mode\.
+ If you enable sticky sessions, the routing algorithm of the target group is overridden after the initial target selection\.
+ When the load balancer supports HTTP/2 to targets that can only support HTTP/1\.1, it converts the request to multiple HTTP/1\.1 requests, so least outstanding request treats each HTTP/2 request as multiple requests\.
+ When you use least outstanding requests with WebSockets, the target is selected using least outstanding requests\. The load balancer creates a connection to this target and sends all messages over this connection\.

------
#### [ New console ]

**To modify the routing algorithm using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** tab, in the **Attributes** section, choose **Edit**\.

1. On the **Edit attributes** page, for **Load balancing algorithm**, choose **Round robin** or **Least outstanding requests**\.

1. Choose **Save changes**\.

------
#### [ Old console ]

**To modify the routing algorithm using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit attributes** page, for **Load balancing algorithm**, choose **Round robin** or **Least outstanding requests**, and then choose **Save**\.

------

**To modify the routing algorithm using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command with the `load_balancing.algorithm.type` attribute\.

## Deregistration delay<a name="deregistration-delay"></a>

Elastic Load Balancing stops sending requests to targets that are deregistering\. By default, Elastic Load Balancing waits 300 seconds before completing the deregistration process, which can help in\-flight requests to the target to complete\. To change the amount of time that Elastic Load Balancing waits, update the deregistration delay value\.

The initial state of a deregistering target is `draining`\. After the deregistration delay elapses, the deregistration process completes and the state of the target is `unused`\. If the target is part of an Auto Scaling group, it can be terminated and replaced\.

If a deregistering target has no in\-flight requests and no active connections, Elastic Load Balancing immediately completes the deregistration process, without waiting for the deregistration delay to elapse\. However, even though target deregistration is complete, the status of the target is displayed as `draining` until the deregistration delay timeout expires\. After the timeout expires, the target transitions to an `unused` state\.

If a deregistering target terminates the connection before the deregistration delay elapses, the client receives a 500\-level error response\.

------
#### [ New console ]

**To update the deregistration delay value using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** tab, in the **Attributes** section, choose **Edit**\.

1. On the **Edit attributes** page, change the value of **Deregistration delay** as needed\.

1. Choose **Save changes**\.

------
#### [ Old console ]

**To update the deregistration delay value using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit attributes** page, change the value of **Deregistration delay** as needed, and then choose **Save**\.

------

**To update the deregistration delay value using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command with the `deregistration_delay.timeout_seconds` attribute\.

## Slow start mode<a name="slow-start-mode"></a>

By default, a target starts to receive its full share of requests as soon as it is registered with a target group and passes an initial health check\. Using slow start mode gives targets time to warm up before the load balancer sends them a full share of requests\.

After you enable slow start for a target group, its targets enter slow start mode when they are considered healthy by the target group\. A target in slow start mode exits slow start mode when the configured slow start duration period elapses or the target becomes unhealthy\. The load balancer linearly increases the number of requests that it can send to a target in slow start mode\. After a healthy target exits slow start mode, the load balancer can send it a full share of requests\.

**Considerations**
+ When you enable slow start for a target group, the healthy targets registered with the target group do not enter slow start mode\.
+ When you enable slow start for an empty target group and then register targets using a single registration operation, these targets do not enter slow start mode\. Newly registered targets enter slow start mode only when there is at least one healthy target that is not in slow start mode\.
+ If you deregister a target in slow start mode, the target exits slow start mode\. If you register the same target again, it enters slow start mode when it is considered healthy by the target group\.
+ If a target in slow start mode becomes unhealthy, the target exits slow start mode\. When the target becomes healthy, it enters slow start mode again\.
+ You cannot enable both slow start mode and least outstanding requests\.

------
#### [ New console ]

**To update the slow start duration value using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** tab, in the **Attributes** section, choose **Edit**\.

1. On the **Edit attributes** page, change the value of **Slow start duration** as needed\. To disable slow start mode, set the duration to 0\.

1. Choose **Save changes**\.

------
#### [ Old console ]

**To update the slow start duration value using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit attributes** page, change the value of **Slow start duration** as needed, and then choose **Save**\. To disable slow start mode, set the duration to 0\.

------

**To update the slow start duration value using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command with the `slow_start.duration_seconds` attribute\.