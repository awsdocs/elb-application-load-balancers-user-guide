# Register targets with your target group<a name="target-group-register-targets"></a>

You register your targets with a target group\. When you create a target group, you specify its target type, which determines how you register its targets\. For example, you can register instance IDs, IP addresses, or Lambda functions\. For more information, see [Target groups for your Application Load Balancers](load-balancer-target-groups.md)\.

If demand on your currently registered targets increases, you can register additional targets in order to handle the demand\. When your target is ready to handle requests, register it with your target group\. The load balancer starts routing requests to the target as soon as the registration process completes and the target passes the initial health checks\.

If demand on your registered targets decreases, or you need to service a target, you can deregister it from your target group\. The load balancer stops routing requests to a target as soon as you deregister it\. When the target is ready to receive requests, you can register it with the target group again\.

When you deregister a target, the load balancer waits until in\-flight requests have completed\. This is known as *connection draining*\. The status of a target is `draining` while connection draining is in progress\.

When you deregister a target that was registered by IP address, you must wait for the deregistration delay to complete before you can register the same IP address again\.

If you are registering targets by instance ID, you can use your load balancer with an Auto Scaling group\. After you attach a target group to an Auto Scaling group and the group scales out, the instances launched by the Auto Scaling group are automatically registered with the target group\. If you detach the target group from the Auto Scaling group, the instances are automatically deregistered from the target group\. For more information, see [Attaching a load balancer to your Auto Scaling group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/attach-load-balancer-asg.html) in the *Amazon EC2 Auto Scaling User Guide*\.

## Target security groups<a name="target-security-groups"></a>

When you register EC2 instances as targets, you must ensure that the security groups for your instances allow the load balancer to communicate with your instances on both the listener port and the health check port\.


**Recommended rules**  

| 
| 
| Inbound | 
| --- |
|  Source  |  Port Range  |  Comment  | 
| load balancer security group | instance listener | Allow traffic from the load balancer on the instance listener port | 
| load balancer security group | health check | Allow traffic from the load balancer on the health check port | 

We also recommend that you allow inbound ICMP traffic to support Path MTU Discovery\. For more information, see [Path MTU Discovery](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/network_mtu.html#path_mtu_discovery) in the *Amazon EC2 User Guide for Linux Instances*\.

## Register or deregister targets<a name="register-deregister-targets"></a>

The target type of your target group determines how you register targets with that target group\. For more information, see [Target type](load-balancer-target-groups.md#target-type)\.

**Topics**
+ [Register or deregister targets by instance ID](#register-instances)
+ [Register or deregister targets by IP address](#register-ip-addresses)
+ [Register or deregister a Lambda function](#register-lambda-function)
+ [Register or deregister targets using the AWS CLI](#register-cli)

### Register or deregister targets by instance ID<a name="register-instances"></a>

The instance must be in the virtual private cloud \(VPC\) that you specified for the target group\. The instance must also be in the `running` state when you register it\.

------
#### [ New console ]

**To register or deregister targets by instance ID using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. Choose the **Targets** tab\.

1. To register instances, choose **Register targets**\. Select one or more instances, enter the default instance port as needed, and then choose **Include as pending below**\. When you are finished adding instances, choose **Register pending targets**\.

1. To deregister instances, select the instances and then choose **Deregister**\.

------
#### [ Old console ]

**To register or deregister targets by instance ID using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select your target group\.

1. On the **Targets** tab, choose **Edit**\.

1. To register instances, select them from **Instances**, modify the default instance port as needed, and choose **Add to registered**\.

1. To deregister instances, select them from **Registered instances** and choose **Remove**\.

1. Choose **Save**\.

------

### Register or deregister targets by IP address<a name="register-ip-addresses"></a>

The IP addresses that you register must be from one of the following CIDR blocks:
+ The subnets of the VPC for the target group
+ 10\.0\.0\.0/8 \(RFC 1918\)
+ 100\.64\.0\.0/10 \(RFC 6598\)
+ 172\.16\.0\.0/12 \(RFC 1918\)
+ 192\.168\.0\.0/16 \(RFC 1918\)

**Limits**
+ You cannot register the IP addresses of another Application Load Balancer in the same VPC\. If the other Application Load Balancer is in a VPC that is peered to the load balancer VPC, you can register its IP addresses\.

------
#### [ New console ]

**To register or deregister targets by IP address using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Chose the name of the target group to open its details page\.

1. Choose the **Targets** tab\.

1. To register IP addresses, choose **Register targets**\. For each IP address, select the network, enter the IP address and port, and choose **Include as pending below**\. When you are finished specifying addresses, choose **Register pending targets**\.

1. To deregister IP addresses, select the IP addresses and then choose **Deregister**\. If you have many registered IP addresses, you might find it helpful to add a filter or change the sort order\.

------
#### [ Old console ]

**To register or deregister targets by IP address**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select your target group\.

1. On the **Targets** tab, choose **Edit**\.

1. To register IP addresses, choose the **Register targets** icon \(the plus sign\) in the menu bar\. For each IP address, select the network, type the IP address and port, and choose **Add to list**\. When you are finished specifying addresses, choose **Register**\.

1. To deregister IP addresses, choose the **Deregister targets** icon \(the minus sign\) in the menu bar\. If you have many registered IP addresses, you might find it helpful to add a filter or change the sort order\. Select the IP addresses and then choose **Deregister**\.

1. To leave this screen, choose the **Back to target group** icon \(the back button\) in the menu bar\.

------

### Register or deregister a Lambda function<a name="register-lambda-function"></a>

You can register a single Lambda function with each target group\. Elastic Load Balancing must have permissions to invoke the Lambda function\. If you no longer need to send traffic to your Lambda function, you can deregister it\. After you deregister a Lambda function, in\-flight requests fail with HTTP 5XX errors\. To replace a Lambda function, it is better to create a new target group instead\. For more information, see [Lambda functions as targets](lambda-functions.md)\.

------
#### [ New console ]

**To register or deregister a Lambda function using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. Choose the **Targets** tab\.

1. If there is no Lambda function registered, choose **Register**\. Select the Lambda function and choose **Register**\.

1. To deregister a Lambda function, choose **Deregister**\. When prompted for confirmation, choose **Deregister**\.

------
#### [ Old console ]

**To register or deregister a Lambda function using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select your target group and choose the **Targets** tab\.

1. If there is no Lambda function registered, choose **Register**\. Select the Lambda function and choose **Register**\.

1. To deregister a Lambda function, choose **Deregister**\. When prompted for confirmation, choose **Deregister**\.

------

### Register or deregister targets using the AWS CLI<a name="register-cli"></a>

Use the [register\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/register-targets.html) command to add targets and the [deregister\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/deregister-targets.html) command to remove targets\.