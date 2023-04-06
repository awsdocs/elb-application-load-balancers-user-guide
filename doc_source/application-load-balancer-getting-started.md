# Getting started with Application Load Balancers<a name="application-load-balancer-getting-started"></a>

This tutorial provides a hands\-on introduction to Application Load Balancers through the AWS Management Console, a web\-based interface\. To create your first Application Load Balancer, complete the following steps\.

**Topics**
+ [Before you begin](#prerequisites)
+ [Step 1: Configure your target group](#configure-target-group)
+ [Step 2: Choose a load balancer type](#select-load-balancer-type)
+ [Step 3: Configure your load balancer and listener](#configure-load-balancer-listener)
+ [Step 4: Test your load balancer](#test-load-balancer)
+ [Step 5: \(Optional\) Delete your load balancer](#delete-load-balancer)

For demos of common load balancer configurations, see [Elastic Load Balancing demos](https://exampleloadbalancer.com/)\.

## Before you begin<a name="prerequisites"></a>
+ Decide which two Availability Zones you will use for your EC2 instances\. Configure your virtual private cloud \(VPC\) with at least one public subnet in each of these Availability Zones\. These public subnets are used to configure the load balancer\. You can launch your EC2 instances in other subnets of these Availability Zones instead\.
+ Launch at least one EC2 instance in each Availability Zone\. Be sure to install a web server, such as Apache or Internet Information Services \(IIS\), on each EC2 instance\. Ensure that the security groups for these instances allow HTTP access on port 80\.

## Step 1: Configure your target group<a name="configure-target-group"></a>

Create a target group, which is used in request routing\. The default rule for your listener routes requests to the registered targets in this target group\. The load balancer checks the health of targets in this target group using the health check settings defined for the target group\. 

**To configure your target group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **Load Balancing**, choose **Target Groups**\.

1. Choose **Create target group**\.

1. Under **Basic configuration**, keep the **Target type** as instance\. 

1. For **Target group name**, enter a name for the new target group\.

1. Keep the default protocol \(**HTTP**\) and port \(**80**\)\.

1. Select the **VPC** containing your instances\. Keep the protocol version as **HTTP1**\. 

1. For **Health checks**, keep the default settings\.

1. Choose **Next**\.

1. On the **Register targets** page, complete the following steps\. This is an optional step for creating the load balancer\. However, you must register this target if you want to test your load balancer and ensure that it is routing traffic to this target\.

   1. For **Available instances**, select one or more instances\.

   1. Keep the default port 80, and choose **Include as pending below**\.

1. Choose **Create target group**\.

## Step 2: Choose a load balancer type<a name="select-load-balancer-type"></a>

Elastic Load Balancing supports different types of load balancers\. For this tutorial, you create an Application Load Balancer\.

**To create a Application Load Balancer**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar, choose a Region for your load balancer\. Be sure to choose the same Region that you used for your EC2 instances\.

1. In the navigation pane, under **Load Balancing**, choose **Load Balancers**\.

1. Choose **Create Load Balancer**\.

1. For **Application Load Balancer**, choose **Create**\.

## Step 3: Configure your load balancer and listener<a name="configure-load-balancer-listener"></a>

To create an Application Load Balancer, you must first provide basic configuration information for your load balancer, such as a name, scheme, and IP address type\. Then, you provide information about your network, and one or more listeners\. A listener is a process that checks for connection requests\. It is configured with a protocol and a port for connections from clients to the load balancer\. For more information about supported protocols and ports, see [Listener configuration](load-balancer-listeners.md#listener-configuration)\.

**To configure your load balancer and listener**

1. For **Load balancer name**, enter a name for your load balancer\. For example, `my-alb`\. 

1. For **Scheme** and **IP address type**, keep the default values\.

1. For **Network mapping**, select the VPC that you used for your EC2 instances\. Select at least two Availability Zones and one subnet per zone\. For each Availability Zone that you used to launch your EC2 instances, select the Availability Zone and then select one public subnet for that Availability Zone\.

1. For **Security groups**, we select the default security group for the VPC that you selected in the previous step\. You can choose a different security group instead\. The security group must include rules that allow the load balancer to communicate with registered targets on both the listener port and the health check port\. For more information, see [Security group rules](load-balancer-update-security-groups.md)\.

1. For **Listeners and routing**, keep the default protocol and port, and select your target group from the list\. This configures a listener that accepts HTTP traffic on port 80 and forwards traffic to the selected target group by default\. For this tutorial, you are not creating an HTTPS listener\.

1. For **Default action**, select the target group that you created and registered in Step 1: Configure your target group\. 

1. \(Optional\) Add a tag to categorize your load balancer\. Tag keys must be unique for each load balancer\. Allowed characters are letters, spaces, numbers \(in UTF\-8\), and the following special characters: \+ \- = \. \_ : / @\. Do not use leading or trailing spaces\. Tag values are case\-sensitive\.

1. Review your configuration, and choose **Create load balancer**\. A few default attributes are applied to your load balancer during creation\. You can view and edit them after creating the load balancer\. For more information, see [Load balancer attributes](application-load-balancers.md#load-balancer-attributes)\.

## Step 4: Test your load balancer<a name="test-load-balancer"></a>

 After creating the load balancer, verify that it's sending traffic to your EC2 instances\.

**To test your load balancer**

1. After you are notified that your load balancer was created successfully, choose **Close**\.

1. In the navigation pane, under **Load Balancing**, choose **Target Groups**\.

1. Select the newly created target group\.

1. Choose **Targets** and verify that your instances are ready\. If the status of an instance is `initial`, it's probably because the instance is still in the process of being registered, or it has not passed the minimum number of health checks to be considered healthy\. After the status of at least one instance is `healthy`, you can test your load balancer\.

1. In the navigation pane, under **Load Balancing**, choose **Load Balancers**\.

1. Select the newly created load balancer\.

1. Choose **Description** and copy the DNS name of the load balancer \(for example, my\-load\-balancer\-1234567890abcdef\.elb\.us\-east\-2\.amazonaws\.com\)\. Paste the DNS name into the address field of an internet\-connected web browser\. If everything is working, the browser displays the default page of your server\.

1. \(Optional\) To define additional listener rules, see [Add a rule](listener-update-rules.md#add-rule)\.

## Step 5: \(Optional\) Delete your load balancer<a name="delete-load-balancer"></a>

As soon as your load balancer becomes available, you are billed for each hour or partial hour that you keep it running\. When you no longer need a load balancer, you can delete it\. As soon as the load balancer is deleted, you stop incurring charges for it\. Note that deleting a load balancer does not affect the targets registered with the load balancer\. For example, your EC2 instances continue to run after deleting the load balancer created in this guide\.

**To delete your load balancer**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, under **Load Balancing**, choose **Load Balancers**\.

1. Select the checkbox for the load balancer, choose **Actions**, then choose **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.