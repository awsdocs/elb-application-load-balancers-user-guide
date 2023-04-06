# Availability Zones for your Application Load Balancer<a name="load-balancer-subnets"></a>

You can enable or disable the Availability Zones for your load balancer at any time\. After you enable an Availability Zone, the load balancer starts routing requests to the registered targets in that Availability Zone\. Your load balancer is most effective if you ensure that each enabled Availability Zone has at least one registered target\.

After you disable an Availability Zone, the targets in that Availability Zone remain registered with the load balancer, but the load balancer will not route requests to them\.

**To update Availability Zones using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Network mapping** tab, choose **Edit subnets**\.

1. To enable an Availability Zone, select its check box and select one subnet\. If there is only one available subnet, it is selected for you\.

1. To change the subnet for an enabled Availability Zone, choose one of the other subnets from the list\.

1. To disable an Availability Zone, clear its check box\.

1. Choose **Save changes**\.

**To update Availability Zones using the AWS CLI**  
Use the [set\-subnets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/set-subnets.html) command\.