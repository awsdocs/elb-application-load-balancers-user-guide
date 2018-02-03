# Tutorial: Use Path\-Based Routing with Your Application Load Balancer<a name="tutorial-load-balancer-routing"></a>

You can create a listener with rules to forward requests based on the URL path\. This is known as *path\-based routing*\. If you are running microservices, you can route traffic to multiple back\-end services using path\-based routing\. For example, you can route general requests to one target group and requests to render images to another target group\.

## Before You Begin<a name="routing-prerequisites"></a>

+ Launch your EC2 instances in a virtual private cloud \(VPC\)\. Ensure that the security groups for these instances allow access on the listener port and the health check port\. For more information, see [Target Security Groups](target-group-register-targets.md#target-security-groups)\.

+ Verify that your microservices are deployed on the EC2 instances that you plan to register\.

## Create Your Load Balancer<a name="routing-create-load-balancer"></a>

**To create a load balancer that uses path\-based routing**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar, select the same region that you selected for your EC2 instances\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Create a target group for the first set of targets as follows:

   1. Choose **Create target group**\.

   1. Specify a name, protocol, port, and VPC for the target group, and then choose **Create**\.

   1. Select the new target group\.

   1. On the **Targets** tab, choose **Edit**\.

   1. For **Instances**, select one or more instances\. Specify a port for the instances, choose **Add to registered**, and then choose **Save**\.

      Note that the status of the instances is `initial` until the instances are registered and have passed health checks, and then it is `unused` until you configure the target group to receive traffic from the load balancer\.

1. Create a target group for the second set of targets as follows:

   1. Choose **Create target group**\.

   1. Specify a name, protocol, port, and VPC for the target group, and then choose **Create**\.

   1. On the **Targets** tab, choose **Edit**\.

   1. For **Instances**, select one or more instances\. Specify a port for the instances, choose **Add to registered**, and then choose **Save**\.

      Note that the status of the instances is `initial` until the instances are registered and have passed health checks, and then it is `unused` until you configure the target group to receive traffic from the load balancer\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Choose **Create Load Balancer**\.

1. For **Select load balancer type**, choose **Application Load Balancer**\.

1. Choose **Continue**\.

1. Complete the **Configure Load Balancer** page as follows:

   1. For **Name**, type a name for your load balancer\.

      The name of your Application Load Balancer must be unique within your set of Application Load Balancers and Network Load Balancers for the region, can have a maximum of 32 characters, can contain only alphanumeric characters and hyphens, and must not begin or end with a hyphen\.

   1. For **Scheme**, an Internet\-facing load balancer routes requests from clients over the Internet to targets\. An internal load balancer routes requests to targets using private IP addresses\.

   1. For **Listeners**, the default is a listener that accepts HTTP traffic on port 80\. You can keep the default listener settings, modify the protocol or port of the listener, or choose **Add** to add another listener\.

   1. For **Availability Zones**, select the VPC that you used for your EC2 instances\. Select at least two Availability Zones\. If there is one subnet for an Availability Zone, it is selected\. If there is more than one subnet for an Availability Zone, select one of the subnets\. Note that you can select only one subnet per Availability Zone\.

   1. Choose **Next: Configure Security Settings**\.

1. \(Optional\) If you created a secure listener in the previous step, complete the **Configure Security Settings** page as follows:

   1. If you created or imported a certificate using AWS Certificate Manager, select **Choose an existing certificate from AWS Certificate Manager \(ACM\)**, and then select the certificate from **Certificate name**\.

   1. If you uploaded a certificate using IAM, select **Choose an existing certificate from AWS Identity and Access Management \(IAM\)**, and then select your certificate from **Certificate name**\.

   1. If you have a certificate to upload but ACM is not supported in your region, choose **Upload a new SSL Certificate to AWS Identity and Access Management \(IAM\)**\. For **Certificate name**, type a name for the certificate\. For **Private Key**, copy and paste the contents of the private key file \(PEM\-encoded\)\. In **Public Key Certificate**, copy and paste the contents of the public key certificate file \(PEM\-encoded\)\. In **Certificate Chain**, copy and paste the contents of the certificate chain file \(PEM\-encoded\), unless you are using a self\-signed certificate and it's not important that browsers implicitly accept the certificate\.

   1. For **Select policy**, keep the default security policy\.

1. Choose **Next: Configure Security Groups**\.

1. Complete the **Configure Security Groups** page as follows:

   1. Select **Create a new security group**\.

   1. Type a name and description for the security group, or keep the default name and description\. This new security group contains a rule that allows traffic to the port that you selected for your load balancer on the **Configure Load Balancer** page\.

   1. Choose **Next: Configure Routing**\.

1. Complete the **Configure Routing** page as follows:

   1. For **Target group**, choose `Existing target group`\.

   1. For **Name**, choose the first target group that you created\.

   1. Choose **Next: Register Targets**\.

1. On the **Register Targets** page, the instances that you registered with the target group appear under **Registered instances**\. You can't modify the targets registered with the target group until after you complete the wizard\. Choose **Next: Review**\.

1. On the **Review** page, choose **Create**\.

1. After you are notified that your load balancer was created successfully, choose **Close**\.

1. Select the newly created load balancer\.

1. On the **Listeners** tab, use the arrow to view the rules for the listener, and then choose **Add rule**\. Specify the rule as follows:

   1. For **Target group name**, choose the second target group that you created\.

   1. For **Path pattern** specify the exact pattern to be used for path\-based routing \(for example, `/img/*`\)\. For more information, see [Listener Rules](load-balancer-listeners.md#listener-rules)\.

   1. Choose **Save**\.