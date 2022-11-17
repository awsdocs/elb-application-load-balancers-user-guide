# Cross\-zone load balancing for target groups<a name="disable-cross-zone"></a>

The nodes for your load balancer distribute requests from clients to registered targets\. When cross\-zone load balancing is on, each load balancer node distributes traffic across the registered targets in all registered Availability Zones\. When cross\-zone load balancing is off, each load balancer node distributes traffic only across the registered targets in its Availability Zone\. This could be if zonal failure domains are preferred over regional, ensuring that a healthy zone isn't impacted by an unhealthy zone, or for overall latency improvements\.

With Application Load Balancers, cross\-zone load balancing is always turned on at the load balancer level, and cannot be turned off\. For target groups, the default is to use the load balancer setting, but you can override the default by explicitly turning cross\-zone load balancing off at the target group level\.

**Considerations**
+ Target stickiness is not supported when cross\-zone load balancing is off\.
+ Lambda functions as targets are not supported when cross\-zone load balancing is off\.
+ Attempting to turn off cross\-zone load balancing through the `ModifyTargetGroupAttributes` API if any targets have parameter `AvailabilityZone` set to `all` results in an error\.
+ When registering targets, the `AvailabilityZone` parameter is required\. Specific Availability Zone values are only allowed when cross\-zone load balancing is off\. Otherwise, the parameter is ignored and treated as `all`\.

**Best practices**
+ Plan for enough target capacity across all Availability Zones that you expect to utilize, per target group\. If you can't plan for enough capacity across all participating Availability Zones, we recommend that you keep cross\-zone load balancing on\.
+ When configuring your Application Load Balancer with multiple target groups, ensure all target groups are participating in the same Availability Zones, within the configured Region\. This is to avoid an Availability Zone being empty while cross\-zone load balancing is off, as this triggers a 503 error for all HTTP requests that enter the empty Availability Zone\.
+ Avoid creating empty subnets\. Application Load Balancers expose zonal IP addresses through DNS for the empty subnets, which triggers 503 errors for HTTP requests\.
+ There can be occurrences where a target group with cross\-zone load balancing turned off has enough planned target capacity per Availability Zone, but all targets in an Availability Zone become unhealthy\. When there is at least one target group with all unhealthy targets, the IP addresses of the load balancer nodes are removed from DNS\. After the target group has at least one healthy target, the IP addresses are restored to DNS\.

## Turn off cross\-zone load balancing<a name="cross_zone_console_disable"></a>

You can turn off cross\-zone load balancing for your Application Load Balancer target groups at any time\.

**To turn off cross\-zone load balancing using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Load Balancing**, select **Target Groups**\.

1. Select the name of the target group to open its details page\.

1. On the **Attributes** tab, select **Edit**\.

1. On the **Edit target group attributes** page, select **Off** for **Cross\-zone load balancing**\.

1. Choose **Save changes**\.

**To turn off cross\-zone load balancing using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command and set the `load_balancing.cross_zone.enabled` attribute to `false`\.

```
aws elbv2 modify-target-group-attributes --target-group-arn my-targetgroup-arn --attributes Key=load_balancing.cross_zone.enabled,Value=false
```

The following is an example response:

```
{
    "Attributes": [
        {
            "Key": "load_balancing.cross_zone.enabled",
            "Value": "false"
        },
    ]
}
```

## Turn on cross\-zone load balancing<a name="cross_zone_console_enable"></a>

You can turn on cross\-zone load balancing for your Application Load Balancer target groups at any time\. The cross\-zone load balancing setting at the target group level overrides the setting at the load balancer level\.

**To turn on cross\-zone load balancing using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Load Balancing**, select **Target Groups**\.

1. Select the name of the target group to open its details page\.

1. On the **Attributes** tab, select **Edit**\.

1. On the **Edit target group attributes** page, select **On** for **Cross\-zone load balancing**\.

1. Choose **Save changes**\.

**To turn on cross\-zone load balancing using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command and set the `load_balancing.cross_zone.enabled` attribute to `true`\.

```
aws elbv2 modify-target-group-attributes --target-group-arn my-targetgroup-arn --attributes Key=load_balancing.cross_zone.enabled,Value=true
```

The following is an example response:

```
{
    "Attributes": [
        {
            "Key": "load_balancing.cross_zone.enabled",
            "Value": "true"
        },
    ]
}
```