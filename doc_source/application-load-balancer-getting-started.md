# Getting started with Application Load Balancers<a name="application-load-balancer-getting-started"></a>

This tutorial provides a hands\-on introduction to Application Load Balancers through the AWS Management Console, a web\-based interface\. To create your first Application Load Balancer, complete the following steps\.

**Topics**
+ [Before you begin](#prerequisites)
+ [Step 1: Select a load balancer type](#select-load-balancer-type)
+ [Step 2: Configure your load balancer and listener](#configure-load-balancer-listener)
+ [Step 3: Configure a security group for your load balancer](#configure-security-groups)
+ [Step 4: Configure your target group](#configure-target-group)
+ [Step 5: Register targets with your target group](#add-targets)
+ [Step 6: Create and test your load balancer](#test-load-balancer)
+ [Step 7: Delete your load balancer \(optional\)](#delete-load-balancer)

Alternatively, to create a Network Load Balancer, see [Getting started with Network Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancer-getting-started.html) in the *User Guide for Network Load Balancers*\. To create a Classic Load Balancer, see [Create a Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-getting-started.html) in the *User Guide for Classic Load Balancers*\.

For demos of common load balancer configurations, see [Elastic Load Balancing demos](https://exampleloadbalancer.com/)\.

## Before you begin<a name="prerequisites"></a>
+ Decide which two Availability Zones you will use for your EC2 instances\. Configure your virtual private cloud \(VPC\) with at least one public subnet in each of these Availability Zones\. These public subnets are used to configure the load balancer\. You can launch your EC2 instances in other subnets of these Availability Zones instead\.
+ Launch at least one EC2 instance in each Availability Zone\. Be sure to install a web server, such as Apache or Internet Information Services \(IIS\), on each EC2 instance\. Ensure that the security groups for these instances allow HTTP access on port 80\.

## Step 1: Select a load balancer type<a name="select-load-balancer-type"></a>

Elastic Load Balancing supports three types of load balancers\. For this tutorial, you create an Application Load Balancer\.

**To create an Application Load Balancer**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar, choose a region for your load balancer\. Be sure to select the same region that you used for your EC2 instances\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Choose **Create Load Balancer**\.

1. For **Application Load Balancer**, choose **Create**\.

## Step 2: Configure your load balancer and listener<a name="configure-load-balancer-listener"></a>

On the **Configure Load Balancer** page, complete the following procedure\.

**To configure your load balancer and listener**

1. For **Name**, enter a name for your load balancer\.

   The name of your Application Load Balancer must be unique within your set of Application Load Balancers and Network Load Balancers for the region, can have a maximum of 32 characters, can contain only alphanumeric characters and hyphens, must not begin or end with a hyphen, and must not begin with "internal\-"\.

1. For **Scheme** and **IP address type**, keep the default values\.

1. For **Listeners**, keep the default, which is a listener that accepts HTTP traffic on port 80\.

1. For **Availability Zones**, select the VPC that you used for your EC2 instances\. For each Availability Zone that you used to launch your EC2 instances, select the Availability Zone and then select the public subnet for that Availability Zone\.

1. Choose **Next: Configure Security Settings**\.

1. For this tutorial, you are not creating an HTTPS listener\. Choose **Next: Configure Security Groups**\.

## Step 3: Configure a security group for your load balancer<a name="configure-security-groups"></a>

The security group for your load balancer must allow it to communicate with registered targets on both the listener port and the health check port\. The console can create a security group for your load balancer on your behalf, with rules that specify the correct protocols and ports\. If you prefer, you can create and select your own security group instead\. For more information, see [Recommended rules](load-balancer-update-security-groups.md#security-group-recommended-rules)\.

On the **Configure Security Groups** page, complete the following procedure to have Elastic Load Balancing create a security group for your load balancer on your behalf\.

**To configure a security group for your load balancer**

1. Choose **Create a new security group**\.

1. Type a name and description for the security group, or keep the default name and description\. This new security group contains a rule that allows traffic to the load balancer listener port that you selected on the **Configure Load Balancer** page\.

1. Choose **Next: Configure Routing**\.

## Step 4: Configure your target group<a name="configure-target-group"></a>

Create a target group, which is used in request routing\. The default rule for your listener routes requests to the registered targets in this target group\. The load balancer checks the health of targets in this target group using the health check settings defined for the target group\. On the **Configure Routing** page, complete the following procedure\.

**To configure your target group**

1. For **Target group**, keep the default, **New target group**\.

1. For **Name**, enter a name for the new target group\.

1. Keep the default target type \(**Instance**\), protocol \(**HTTP**\), and port \(**80**\)\.

1. For **Health checks**, keep the default settings\.

1. Choose **Next: Register Targets**\.

## Step 5: Register targets with your target group<a name="add-targets"></a>

On the **Register Targets** page, complete the following procedure\.

**To register your instances with the target group**

1. For **Instances**, select one or more instances\.

1. Keep the default port \(**80**\) and choose **Add to registered**\.

1. When you have finished selecting instances, choose **Next: Review**\.

## Step 6: Create and test your load balancer<a name="test-load-balancer"></a>

Before creating the load balancer, review the settings that you selected\. After creating the load balancer, verify that it's sending traffic to your EC2 instances\.

**To create and test your load balancer**

1. On the **Review** page, choose **Create**\.

1. After you are notified that your load balancer was created successfully, choose **Close**\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the newly created target group\.

1. On the **Targets** tab, verify that your instances are ready\. If the status of an instance is `initial`, it's probably because the instance is still in the process of being registered, or it has not passed the minimum number of health checks to be considered healthy\. After the status of at least one instance is `healthy`, you can test your load balancer\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the newly created load balancer\.

1. On the **Description** tab, copy the DNS name of the load balancer \(for example, my\-load\-balancer\-1234567890\.us\-west\-2\.elb\.amazonaws\.com\)\. Paste the DNS name into the address field of an Internet\-connected web browser\. If everything is working, the browser displays the default page of your server\.

1. \(Optional\) To define additional listener rules, see [Add a rule](listener-update-rules.md#add-rule)\.

## Step 7: Delete your load balancer \(optional\)<a name="delete-load-balancer"></a>

As soon as your load balancer becomes available, you are billed for each hour or partial hour that you keep it running\. When you no longer need a load balancer, you can delete it\. As soon as the load balancer is deleted, you stop incurring charges for it\. Note that deleting a load balancer does not affect the targets registered with the load balancer\. For example, your EC2 instances continue to run\.

**To delete your load balancer**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the checkbox for the load balancer, and then choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.