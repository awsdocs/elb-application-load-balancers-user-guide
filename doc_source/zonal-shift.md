# Zonal shift<a name="zonal-shift"></a>

Zonal shift is a capability in Amazon Route 53 Application Recovery Controller \(Route 53 ARC\)\. With zonal shift, you can shift a load balancer resource away from an impaired Availability Zone with a single action\. This way, you can continue operating from other healthy Availability Zones in an AWS Region\.



When you start a zonal shift, your load balancer stops sending traffic for the resource to the affected Availability Zone\. Route 53 ARC creates the zonal shift immediately\. However, it can take a short time, typically up to a few minutes, to complete existing, in\-progress connections in the affected Availability Zone\. For more information, see [How a zonal shift works: health checks and zonal IP addresses](https://docs.aws.amazon.com/r53recovery/latest/dg/arc-zonal-shift.how-it-works.html) in the *Amazon Route 53 Application Recovery Controller Developer Guide*\.

Zonal shifts are only supported on Application Load Balancers and Network Load Balancers with cross\-zone load balancing turned off\. If you turn on cross\-zone load balancing, you can't start a zonal shift\. For more information, see [Resources supported for zonal shifts](https://docs.aws.amazon.com/r53recovery/latest/dg/arc-zonal-shift.resource-types.html) in the *Amazon Route 53 Application Recovery Controller Developer Guide*\.

Before you use a zonal shift, review the following:
+ Cross\-zone load balancing isn't supported with zonal shifts\. You must turn off cross\-zone load balancing to use this capability\.
+ Zonal shift isn't supported when you use an Application Load Balancer as an accelerator endpoint in AWS Global Accelerator\.
+ You can start a zonal shift for a specific load balancer only for a single Availability Zone\. You can't start a zonal shift for multiple Availability Zones\.
+ AWS proactively removes zonal load balancer IP addresses from DNS when multiple infrastructure issues impact services\. Always check current Availability Zone capacity before you start a zonal shift\. If your load balancers have cross\-zone load balancing turned off and you use a zonal shift to remove a zonal load balancer IP address, the Availability Zone affected by the zonal shift also loses target capacity\.
+ When an Application Load Balancer is a target of a Network Load Balancer, always start the zonal shift from the Network Load Balancer\. If you start a zonal shift from the Application Load Balancer, the Network Load Balancer doesn't recognize the shift and continues to send traffic to the Application Load Balancer\.

For more guidance and information, see [Best practices with Route 53 ARC zonal shifts](https://docs.aws.amazon.com/r53recovery/latest/dg/route53-arc-best-practices.html#zonalshift.route53-arc-best-practices.zonal-shifts) in the *Amazon Route 53 Application Recovery Controller Developer Guide*\.

## Start a zonal shift<a name="start_zonal_shift"></a>

The steps in this procedure explain how to start a zonal shift using the Amazon EC2 console\. For steps to start a zonal shift using the Route 53 ARC console, see [Starting a zonal shift](https://docs.aws.amazon.com/r53recovery/latest/dg/arc-zonal-shift.start.html) in the *Amazon Route 53 Application Recovery Controller Developer Guide*\.

**To start a zonal shift using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Load Balancing**, choose **Load Balancers**\.

1. Select the load balancer name\.

1. On the **Integrations** tab, under **Route 53 Application Recovery Controller**, choose **Start zonal shift**\.

1. Select the Availability Zone that you want to move traffic away from\.

1. Choose or enter an expiration for the zonal shift\. A zonal shift can initially be set from 1 minute up to three days \(72 hours\)\.

   All zonal shifts are temporary\. You must set an expiration, but you can update active shifts later to set a new expiration\.

1. Enter a comment\. You can update the zonal shift later to edit the comment, if you like\.

1. Select the check box to acknowledge that starting a zonal shift will reduce capacity for your application by shifting traffic away from the Availability Zone\.

1. Choose **Start**\.

**To start a zonal shift using the AWS CLI**  
To work with zonal shift programmatically, see the [Zonal Shift API Reference Guide](https://docs.aws.amazon.com/arc-zonal-shift/latest/api/)\.

## Update a zonal shift<a name="update_zonal_shift"></a>

The steps in this procedure explain how to update a zonal shift using the Amazon EC2 console\. For steps to update a zonal shift using the Amazon Route 53 Application Recovery Controller console, see [Updating a zonal shift](https://docs.aws.amazon.com/r53recovery/latest/dg/arc-zonal-shift.update-cancel.html) in the *Amazon Route 53 Application Recovery Controller Developer Guide*\.

**To update a zonal shift using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Load Balancing**, choose **Load Balancers**\.

1. Select a load balancer name that has an active zonal shift\.

1. On the **Integrations** tab, under **Route 53 Application Recovery Controller**, choose **Update zonal shift**\.

   This opens the Route 53 ARC console to continue the update\.

1. For **Set zonal shift expiration**, optionally select or enter an expiration\.

1. For **Comment**, optionally edit the existing comment or enter a new comment\.

1. Choose **Update**\.

**To update a zonal shift using the AWS CLI**  
To work with zonal shift programmatically, see the [Zonal Shift API Reference Guide](https://docs.aws.amazon.com/arc-zonal-shift/latest/api/)\.

## Cancel a zonal shift<a name="cancel_zonal_shift"></a>

The steps in this procedure explain how to cancel a zonal shift using the Amazon EC2 console\. For steps to cancel a zonal shift using the Amazon Route 53 Application Recovery Controller console, see [Canceling a zonal shift](https://docs.aws.amazon.com/r53recovery/latest/dg/arc-zonal-shift.update-cancel.html) in the *Amazon Route 53 Application Recovery Controller Developer Guide*\.

**To cancel a zonal shift using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Load Balancing**, choose **Load Balancers**\.

1. Select a load balancer name that has an active zonal shift\.

1. On the **Integrations** tab, under **Route 53 Application Recovery Controller**, choose **Cancel zonal shift**\.

   This opens the Route 53 ARC console to continue the cancelation\.

1. Choose **Cancel zonal shift**\.

1. On the confirmation dialog, choose **Confirm**\.

**To cancel a zonal shift using the AWS CLI**  
To work with zonal shift programmatically, see the [Zonal Shift API Reference Guide](https://docs.aws.amazon.com/arc-zonal-shift/latest/api/)\.