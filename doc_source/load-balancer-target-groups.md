# Target groups for your Application Load Balancers<a name="load-balancer-target-groups"></a>

Each *target group* is used to route requests to one or more registered targets\. When you create each listener rule, you specify a target group and conditions\. When a rule condition is met, traffic is forwarded to the corresponding target group\. You can create different target groups for different types of requests\. For example, create one target group for general requests and other target groups for requests to the microservices for your application\. For more information, see [Application Load Balancer components](introduction.md#application-load-balancer-components)\.

You define health check settings for your load balancer on a per target group basis\. Each target group uses the default health check settings, unless you override them when you create the target group or modify them later on\. After you specify a target group in a rule for a listener, the load balancer continually monitors the health of all targets registered with the target group that are in an Availability Zone enabled for the load balancer\. The load balancer routes requests to the registered targets that are healthy\.

**Topics**
+ [Routing configuration](#target-group-routing-configuration)
+ [Target type](#target-type)
+ [Registered targets](#registered-targets)
+ [Target group attributes](#target-group-attributes)
+ [Routing algorithm](#modify-routing-algorithm)
+ [Deregistration delay](#deregistration-delay)
+ [Slow start mode](#slow-start-mode)
+ [Sticky sessions](#sticky-sessions)
+ [Create a target group](create-target-group.md)
+ [Health checks for your target groups](target-group-health-checks.md)
+ [Register targets with your target group](target-group-register-targets.md)
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
+ 10\.0\.0\.0/8 \(RFC 1918\)
+ 100\.64\.0\.0/10 \(RFC 6598\)
+ 172\.16\.0\.0/12 \(RFC 1918\)
+ 192\.168\.0\.0/16 \(RFC 1918\)

These supported CIDR blocks enable you to register the following with a target group: ClassicLink instances, instances in a VPC that is peered to the load balancer VPC \(same Region or different Region\), AWS resources that are addressable by IP address and port \(for example, databases\), and on\-premises resources linked to AWS through AWS Direct Connect or a VPN connection\.

**Important**  
You can't specify publicly routable IP addresses\.

If you specify targets using an instance ID, traffic is routed to instances using the primary private IP address specified in the primary network interface for the instance\. If you specify targets using IP addresses, you can route traffic to an instance using any private IP address from one or more network interfaces\. This enables multiple applications on an instance to use the same port\. Each network interface can have its own security group\.

If the target type of your target group is `lambda`, you can register a single Lambda function\. When the load balancer receives a request for the Lambda function, it invokes the Lambda function\. For more information, see [Lambda functions as targets](lambda-functions.md)\.

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

`slow_start.duration_seconds`  
The time period, in seconds, during which the load balancer sends a newly registered target a linearly increasing share of the traffic to the target group\. The range is 30–900 seconds \(15 minutes\)\. The default is 0 seconds \(disabled\)\.

`stickiness.enabled`  
Indicates whether sticky sessions are enabled\.

`stickiness.lb_cookie.duration_seconds`  
The cookie expiration period, in seconds\. After this period, the cookie is considered stale\. The minimum value is 1 second and the maximum value is 7 days \(604800 seconds\)\. The default value is 1 day \(86400 seconds\)\.

`stickiness.type`  
The type of stickiness\. The possible value is `lb_cookie`\.

The following target group attribute is supported if the target group type is `lambda`:

`lambda.multi_value_headers.enabled`  
Indicates whether the request and response headers exchanged between the load balancer and the Lambda function include arrays of values or strings\. The possible values are `true` or `false`\. The default value is `false`\. For more information, see [Multi\-value headers](lambda-functions.md#multi-value-headers)\.

## Routing algorithm<a name="modify-routing-algorithm"></a>

By default, the round robin routing algorithm is used to route requests at the target group level\. You can specify the least outstanding requests routing algorithm instead\.

Consider using least outstanding requests when the requests for your application vary in complexity or your targets vary in processing capability\. Round robin is a good choice when the requests and targets are similar, or if you need to distribute requests equally among targets\. You can compare the effect of round robin versus least outstanding requests using the following CloudWatch metrics: **RequestCount**, **TargetConnectionErrorCount**, and **TargetResponseTime**\.

**Considerations**
+ You cannot enable both least outstanding requests and slow start mode\.
+ If you enable sticky sessions, this overrides the routing algorithm of the target group after the initial target selection\.
+ With HTTP/2, the load balancer converts the request to multiple HTTP/1\.1 requests, so least outstanding request treats each HTTP/2 request as multiple requests\.
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

If a deregistering target has no in\-flight requests and no active connections, Elastic Load Balancing immediately completes the deregistration process, without waiting for the deregistration delay to elapse\. However, even though target deregistration is complete, the status of the target will be displayed as `draining` until the deregistration delay elapses\.

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

After you enable slow start for a target group, its targets enter slow start mode when they are considered healthy by the target group\. A target in slow start mode exits slow start mode when the configured slow start duration period elapses or the target becomes unhealthy\. The load balancer linearly increases the number of requests that it can send to a target in slow start mode\. After a healthy target exits slow start mode, the load balancer can send it a a full share of requests\.

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

## Sticky sessions<a name="sticky-sessions"></a>

Sticky sessions are a mechanism to route requests to the same target in a target group\. This is useful for servers that maintain state information in order to provide a continuous experience to clients\. To use sticky sessions, the clients must support cookies\.

When a load balancer first receives a request from a client, it routes the request to a target, generates a cookie named AWSALB that encodes information about the selected target, encrypts the cookie, and includes the cookie in the response to the client\. The client should include the cookie that it receives in subsequent requests to the load balancer\. When the load balancer receives a request from a client that contains the cookie, if sticky sessions are enabled for the target group and the request goes to the same target group, the load balancer detects the cookie and routes the request to the same target\. If the cookie is present but cannot be decoded, or if it refers to a target that was deregistered or is unhealthy, the load balancer selects a new target and updates the cookie with information about the new target\.

You enable sticky sessions at the target group level\. You can also set the duration for the stickiness of the load balancer\-generated cookie, in seconds\. The duration is set with each request\. Therefore, if the client sends a request before each duration period expires, the sticky session continues\.

With CORS \(cross\-origin resource sharing\) requests, some browsers require `SameSite=None; Secure` to enable stickiness\. In this case, Elastic Load Balancing generates a second stickiness cookie, AWSALBCORS, which includes the same information as the original stickiness cookie plus this `SameSite` attribute\. Clients receive both cookies\.

Application Load Balancers support load balancer\-generated cookies only\. The contents of these cookies are encrypted using a rotating key\. You cannot decrypt or modify load balancer\-generated cookies\.

**Considerations**
+ If you are using multiple layers of Application Load Balancers, you can enable sticky sessions on one layer only, because the load balancers would use the same cookie name\.
+ WebSockets connections are inherently sticky\. If the client requests a connection upgrade to WebSockets, the target that returns an HTTP 101 status code to accept the connection upgrade is the target used in the WebSockets connection\. After the WebSockets upgrade is complete, cookie\-based stickiness is not used\.
+ Application Load Balancers use the Expires attribute in the cookie header instead of the Max\-Age header\.
+ Application Load Balancers do not support cookie values that are URL encoded\.

------
#### [ New console ]

**To enable sticky sessions using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** tab, in the **Attributes** section, choose **Edit**\.

1. On the **Edit attributes** page, do the following:

   1. Select **Stickiness**\.

   1. For **Stickiness duration**, specify a value between 1 second and 7 days\.

   1. Choose **Save changes**\.

------
#### [ Old console ]

**To enable sticky sessions using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit attributes** page, do the following:

   1. Select **Enable load balancer generated cookie stickiness**\.

   1. For **Stickiness duration**, specify a value between 1 second and 7 days\.

   1. Choose **Save**\.

------

**To enable sticky sessions using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command with the `stickiness.enabled` and `stickiness.lb_cookie.duration_seconds` attributes\.