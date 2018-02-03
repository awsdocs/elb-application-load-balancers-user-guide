# Tutorial: Use Microservices as Targets with Your Application Load Balancer<a name="tutorial-target-ecs-containers"></a>

You can use a microservices architecture to structure your application as services that you can develop and deploy independently\. You can install one or more of these services on each EC2 instance, with each service accepting connections on a different port\. You can use a single Application Load Balancer to route requests to all the services for your application\. When you register an EC2 instance with a target group, you can register it multiple times; for each service, register the instance using the port for the service\.

**Important**  
When you deploy your services using Amazon Elastic Container Service \(Amazon ECS\), you can use dynamic port mapping to support multiple tasks from a single service on the same container instance\. Amazon ECS manages updates to your services by automatically registering and deregistering containers with your target group using the instance ID and port for each container\. For more information, see [Service Load Balancing](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-load-balancing.html) in the *Amazon Elastic Container Service Developer Guide*\.

## Before You Begin<a name="containers-prerequisites"></a>

+ Launch your EC2 instances\. Ensure that the security groups for the instances allow access from the load balancer security group on the listener ports and the health check ports\. For more information, see [Target Security Groups](target-group-register-targets.md#target-security-groups)\.

+ Deploy your services to your EC2 instances \(for example, using containers\.\)

## Create Your Load Balancer<a name="containers-create-load-balancer"></a>

**To create a load balancer that uses multiple services as targets**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation bar, select the same region that you selected for your EC2 instances\.

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

   1. If you uploaded a certificate using IAM, select **Choose an existing certificate from AWS Identity and Access Management \(IAM\)**, and then select the certificate from **Certificate name**\.

   1. If you have a certificate to upload but ACM is not supported in your region, choose **Upload a new SSL Certificate to AWS Identity and Access Management \(IAM\)**\. For **Certificate name**, type a name for the certificate\. For **Private Key**, copy and paste the contents of the private key file \(PEM\-encoded\)\. In **Public Key Certificate**, copy and paste the contents of the public key certificate file \(PEM\-encoded\)\. In **Certificate Chain**, copy and paste the contents of the certificate chain file \(PEM\-encoded\), unless you are using a self\-signed certificate and it's not important that browsers implicitly accept the certificate\.

   1. For **Select policy**, keep the default security policy\.

1. Choose **Next: Configure Security Groups**\.

1. Complete the **Configure Security Groups** page as follows:

   1. Select **Create a new security group**\.

   1. Type a name and description for the security group, or keep the default name and description\. This new security group contains a rule that allows traffic to the port that you selected for your load balancer on the **Configure Load Balancer** page\.

   1. Choose **Next: Configure Routing**\.

1. Complete the **Configure Routing** page as follows:

   1. For **Target group**, keep the default, `New target group`\.

   1. For **Name**, type a name for the new target group\.

   1. Set **Protocol** and **Port** as needed\.

   1. For **Health checks**, keep the default health check settings\.

   1. Choose **Next: Register Targets**\.

1. For **Register Targets**, do the following:

   1. For **Instances**, select an EC2 instance\.

   1. Type the port used by the service, and then choose **Add to registered**\.

   1. Repeat for each service to register\. When you are finished, choose **Next: Review**\.

1. On the **Review** page, choose **Create**\.

1. After you are notified that your load balancer was created successfully, choose **Close**\.