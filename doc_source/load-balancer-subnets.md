# Availability Zones for Your Application Load Balancer<a name="load-balancer-subnets"></a>

You can enable or disable the Availability Zones for your load balancer at any time\. After you enable an Availability Zone, the load balancer starts routing requests to the registered targets in that Availability Zone\. Your load balancer is most effective if you ensure that each enabled Availability Zone has at least one registered target\.

After you disable an Availability Zone, the targets in that Availability Zone remain registered with the load balancer, but the load balancer will not route requests to them\.

**To update Availability Zones using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Description** tab, under **Basic Configuration**, choose **Edit Availability Zones**\.

1. To enable an Availability Zone, select the check box for that Availability Zone\. If there is one subnet for that Availability Zone, it is selected\. If there is more than one subnet for that Availability Zone, select one of the subnets\. Note that you can select only one subnet per Availability Zone\.

1. To change the subnet for an enabled Availability Zone, choose **Change subnet** and select one of the other subnets\.

1. To remove an Availability Zone, clear the check box for that Availability Zone\.

1. Choose **Save**\.

**To update Availability Zones using the AWS CLI**  
Use the [set\-subnets](http://docs.aws.amazon.com/cli/latest/reference/elbv2/set-subnets.html) command\.