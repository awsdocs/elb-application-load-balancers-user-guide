# Target group health<a name="target-group-health"></a>

By default, a target group is considered healthy as long as it has at least one healthy target\. If you have a large fleet, having only one healthy target serving traffic is not sufficient\. Instead, you can specify a minimum count or percentage of targets that must be healthy, and what actions the load balancer takes when the healthy targets fall below the specified threshold\. This improves availability\.

## Unhealthy state actions<a name="unhealthy-state-actions"></a>

You can configure healthy thresholds for the following actions:
+ DNS failover – When the healthy targets in a zone fall below the threshold, we mark the IP addresses of the load balancer node for the zone as unhealthy in DNS\. Therefore, when clients resolve the load balancer DNS name, the traffic is routed only to healthy zones\.
+ Routing failover – When the healthy targets in a zone fall below the threshold, the load balancer sends traffic to all targets available to the load balancer node, including unhealthy targets\. This increases the chances that a client connection succeeds, especially when targets temporarily fail to pass health checks, and reduces the risk of overloading the healthy targets\.

## Requirements and considerations<a name="target-group-health-considerations"></a>
+ If you specify both types of thresholds for an action \(count and percentage\), the load balancer takes the action when either threshold is breached\.
+ If you specify thresholds for both actions, the threshold for DNS failover must be greater than or equal to the threshold for routing failover, so that DNS failover occurs either with or before routing failover\.
+ If you specify the threshold as a percentage, we calculate the value dynamically, based on the total number of targets registered with the target groups\.
+ The total number of targets is based on whether cross\-zone load balancing is off or on\. If cross\-zone load balancing is off, each node sends traffic only to the targets in its own zone, which means that the thresholds apply to the number of targets in each enabled zone separately\. If cross\-zone load balancing is on, each node sends traffic to all targets in all enabled zones, which means that the specified thresholds apply to the total number targets in all enabled zones\.
+ With DNS failover, we remove the IP addresses for the unhealthy zones from the DNS hostname for the load balancer\. However, the local client DNS cache might contain these IP addresses until the time\-to\-live \(TTL\) in the DNS record expires \(60 seconds\)\.
+ When DNS failover occurs, this impacts all target groups associated with the load balancer\. Ensure that you have enough capacity in your remaining zones to handle this additional traffic, especially if cross\-zone load balancing is off\.
+ With DNS failover, if all load balancer zones are considered unhealthy, the load balancer sends traffic to all zones, including the unhealthy zones\.
+ There are factors other than whether there are enough healthy targets that might lead to DNS failover, such as the health of the Availability Zone\.

## Monitoring<a name="target-group-health-monitoring"></a>

To monitor the health of your target groups, see [CloudWatch metrics for target group health](load-balancer-cloudwatch-metrics.md#target-group-health-metric-table)\.

## Example<a name="target-group-health-examples"></a>

The following example demonstrates how target group health settings are applied\.

**Scenario**
+ A load balancer that supports two Availability Zones, A and B
+ Each Availability Zone contains 10 registered targets
+ The target group has the following target group health settings:
  + DNS failover \- 50%
  + Routing failover \- 50%
+ Six targets fail in Availability Zone B

**If cross\-zone load balancing is off**
+ The load balancer node in each Availability Zone can send traffic only to the 10 targets in its Availability Zone\.
+ There are 10 healthy targets in Availability Zone A, which meets the required percentage of healthy targets\. The load balancer continues to distribute traffic between the 10 healthy targets\.
+ There are only 4 healthy targets in Availability Zone B, which is 40% of the targets for the load balancer node in Availability Zone B\. Because this is less than the required percentage of healthy targets, the load balancer takes the following actions:
  + DNS failover \- Availability Zone B is marked as unhealthy in DNS\. Because clients can't resolve the load balancer name to the load balancer node in Availability Zone B, and Availability Zone A is healthy, clients send new connections to Availability Zone A\.
  + Routing failover \- When new connections are sent explicitly to Availability Zone B, the load balancer distributes traffic to all targets in Availability Zone B, including the unhealthy targets\. This prevents outages among the remaining healthy targets\.

**If cross\-zone load balancing is on**
+ Each load balancer node can send traffic to all 20 registered targets across both Availability Zones\.
+ There are 10 healthy targets in Availability Zone A and 4 healthy targets in Availability Zone B, for a total of 14 healthy targets\. This is 70% of the targets for the load balancer nodes in both Availability Zones, which meets the required percentage of healthy targets\.
+ The load balancer distributes traffic between the 14 healthy targets in both Availability Zones\.

## Modify target group health settings<a name="modify-target-group-health-settings"></a>

You can modify the target group health settings for your target group as follows\.

**To modify target group health settings using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **Load Balancing**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Attributes** tab, choose **Edit**\.

1. Check whether cross\-zone load balancing is turned on or turned off\. Update this setting as needed to ensure you have enough capacity to handle the additional traffic if a zone fails\.

1. Expand **Target group health requirements**\.

1. For **Configuration type**, we recommend that you choose **Unified configuration**, which sets the same threshold for both actions\.

1. For **Healthy state requirements**, do one of the following:
   + Choose **Minimum healthy target count**, and then enter a number from 1 to the maximum number of targets for your target group\.
   + Choose **Minimum healthy target percentage**, and then enter a number from 1 to 100\.

1. Choose **Save changes**\.

**To modify target group health settings using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command\. The following example sets the healthy threshold for both unhealthy state actions to 50%\.

```
aws elbv2 modify-target-group-attributes \
--target-group-arn arn:aws:elasticloadbalancing:region:123456789012:targetgroup/my-targets/73e2d6bc24d8a067 \
--attributes Key=target_group_health.dns_failover.minimum_healthy_targets.percentage,Value=50 \
  Key=target_group_health.unhealthy_state_routing.minimum_healthy_targets.percentage,Value=50
```