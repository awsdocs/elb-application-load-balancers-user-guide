# Health checks for your target groups<a name="target-group-health-checks"></a>

Your Application Load Balancer periodically sends requests to its registered targets to test their status\. These tests are called *health checks*\.

Each load balancer node routes requests only to the healthy targets in the enabled Availability Zones for the load balancer\. Each load balancer node checks the health of each target, using the health check settings for the target groups with which the target is registered\. After your target is registered, it must pass one health check to be considered healthy\. After each health check is completed, the load balancer node closes the connection that was established for the health check\.

If a target group contains only unhealthy registered targets, the load balancer nodes route requests across its unhealthy targets\.

Health checks do not support WebSockets\.

## Health check settings<a name="health-check-settings"></a>

You configure health checks for the targets in a target group as described in the following table\. The setting names used in the table are the names used in the API\. The load balancer sends a health check request to each registered target every **HealthCheckIntervalSeconds** seconds, using the specified port, protocol, and ping path\. Each health check request is independent and the result lasts for the entire interval\. The time that it takes for the target to respond does not affect the interval for the next health check request\. If the health checks exceed **UnhealthyThresholdCount** consecutive failures, the load balancer takes the target out of service\. When the health checks exceed **HealthyThresholdCount** consecutive successes, the load balancer puts the target back in service\.


| Setting | Description | 
| --- | --- | 
| **HealthCheckProtocol** |  The protocol the load balancer uses when performing health checks on targets\. The possible protocols are HTTP and HTTPS\. The default is the HTTP protocol\.  | 
| **HealthCheckPort** |  The port the load balancer uses when performing health checks on targets\. The default is to use the port on which each target receives traffic from the load balancer\.  | 
| **HealthCheckPath** |  The ping path that is the destination on the targets for health checks\. Specify a valid URI \(/*path*?*query*\)\. The default is /\.  | 
| **HealthCheckTimeoutSeconds** |  The amount of time, in seconds, during which no response from a target means a failed health check\. The range is 2–120 seconds\. The default is 5 seconds if the target type is `instance` or `ip` and 30 seconds if the target type is `lambda`\.  | 
| **HealthCheckIntervalSeconds** |  The approximate amount of time, in seconds, between health checks of an individual target\. The range is 5–300 seconds\. The default is 30 seconds if the target type is `instance` or `ip` and 35 seconds if the target type is `lambda`\.  | 
| **HealthyThresholdCount** |  The number of consecutive successful health checks required before considering an unhealthy target healthy\. The range is 2–10\. The default is 5\.  | 
| **UnhealthyThresholdCount** |  The number of consecutive failed health checks required before considering a target unhealthy\. The range is 2–10\. The default is 2\.  | 
| **Matcher** |  The HTTP codes to use when checking for a successful response from a target\. The possible values are from 200 to 499\. You can specify multiple values \(for example, "200,202"\) or a ranges of values \(for example, "200\-299"\)\. The default value is 200\. These are called **Success codes** in the console\.  | 

## Target health status<a name="target-health-states"></a>

Before the load balancer sends a health check request to a target, you must register it with a target group, specify its target group in a listener rule, and ensure that the Availability Zone of the target is enabled for the load balancer\. Before a target can receive requests from the load balancer, it must pass the initial health checks\. After a target passes the initial health checks, its status is `Healthy`\.

The following table describes the possible values for the health status of a registered target\.


| Value | Description | 
| --- | --- | 
| `initial` |  The load balancer is in the process of registering the target or performing the initial health checks on the target\. Related reason codes: `Elb.RegistrationInProgress` \| `Elb.InitialHealthChecking`  | 
| `healthy` |  The target is healthy\. Related reason codes: None  | 
| `unhealthy` |  The target did not respond to a health check or failed the health check\. Related reason codes: `Target.ResponseCodeMismatch` \| `Target.Timeout` \| `Target.FailedHealthChecks` \| `Elb.InternalError`  | 
| `unused` |  The target is not registered with a target group, the target group is not used in a listener rule, the target is in an Availability Zone that is not enabled, or the target is in the stopped or terminated state\. Related reason codes: `Target.NotRegistered` \| `Target.NotInUse` \| `Target.InvalidState` \| `Target.IpUnusable`  | 
| `draining` |  The target is deregistering and connection draining is in process\. Related reason code: `Target.DeregistrationInProgress`  | 
| `unavailable` |  Health checks are disabled for the target group\. Related reason code: `Target.HealthCheckDisabled`  | 

## Health check reason codes<a name="target-health-reason-codes"></a>

If the status of a target is any value other than `Healthy`, the API returns a reason code and a description of the issue, and the console displays the same description in a tooltip\. Reason codes that begin with `Elb` originate on the load balancer side and reason codes that begin with `Target` originate on the target side\.


| Reason code | Description | 
| --- | --- | 
| `Elb.InitialHealthChecking` |  Initial health checks in progress  | 
| `Elb.InternalError` |  Health checks failed due to an internal error  | 
| `Elb.RegistrationInProgress` |  Target registration is in progress  | 
| `Target.DeregistrationInProgress` |  Target deregistration is in progress  | 
| `Target.FailedHealthChecks` |  Health checks failed  | 
| `Target.HealthCheckDisabled` |  Health checks are disabled  | 
| `Target.InvalidState` |  Target is in the stopped state Target is in the terminated state Target is in the terminated or stopped state Target is in an invalid state  | 
| `Target.IpUnusable` |  The IP address cannot be used as a target, as it is in use by a load balancer  | 
| `Target.NotInUse` |  Target group is not configured to receive traffic from the load balancer Target is in an Availability Zone that is not enabled for the load balancer  | 
| `Target.NotRegistered` |  Target is not registered to the target group  | 
| `Target.ResponseCodeMismatch` |  Health checks failed with these codes: \[*code*\]  | 
| `Target.Timeout` |  Request timed out  | 

## Check the health of your targets<a name="check-target-health"></a>

You can check the health status of the targets registered with your target groups\.

------
#### [ New console ]

**To check the health of your targets using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Targets** tab, the **Status** column indicates the status of each target\.

1. If the status is any value other than `Healthy`, the **Status details** column contains more information\.

------
#### [ Old console ]

**To check the health of your targets using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group\.

1. On the **Targets** tab, the **Status** column indicates the status of each target\.

1. If the status is any value other than `Healthy`, view the tooltip for more information\.

------

**To check the health of your targets using the AWS CLI**  
Use the [describe\-target\-health](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-target-health.html) command\. The output of this command contains the target health state\. If the status is any value other than `Healthy`, the output also includes a reason code\.

**To receive email notifications about unhealthy targets**  
Use CloudWatch alarms to trigger a Lambda function to send details about unhealthy targets\. For step\-by\-step instructions, see the following blog post: [Identifying unhealthy targets of your load balancer](http://aws.amazon.com/blogs/networking-and-content-delivery/identifying-unhealthy-targets-of-elastic-load-balancer/)\.

## Modify the health check settings of a target group<a name="modify-health-check-settings"></a>

You can modify the health check settings for your target group at any time\.

------
#### [ New console ]

**To modify the health check settings of a target group using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** tab, in the **Health check settings** section, choose **Edit**\.

1. On the **Edit health check settings** page, modify the settings as needed, and then choose **Save changes**\.

------
#### [ Old console ]

**To modify the health check settings of a target group using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group\.

1. On the **Health checks** tab, choose **Edit**\.

1. On the **Edit target group** page, modify the settings as needed, and then choose **Save**\.

------

**To modify the health check settings of a target group using the AWS CLI**  
Use the [modify\-target\-group](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group.html) command\.