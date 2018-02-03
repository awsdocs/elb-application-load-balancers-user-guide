# Create a Listener for Your Application Load Balancer<a name="create-listener"></a>

A listener is a process that checks for connection requests\. You define a listener when you create your load balancer, and you can add listeners to your load balancer at any time\.

## Prerequisites<a name="listener-prereqs"></a>

+ You must specify a target group for the default listener rule\. For more information, see [Create a Target Group](create-target-group.md)\.

+ If you create an HTTPS listener, you must specify a certificate and a security policy\. The load balancer uses the certificate to terminate the connection and decrypt requests from clients before routing them to targets\. For more information, see [SSL Certificates](create-https-listener.md#https-listener-certificates)\. The load balancer uses the security policy when negotiating SSL connections with the clients\. For more information, see [Security Policies](create-https-listener.md#describe-ssl-policies)\.

## Add a Listener<a name="add-listener"></a>

You configure a listener with a protocol and a port for connections from clients to the load balancer, and a target group for the default listener rule\. For more information, see [Listener Configuration](load-balancer-listeners.md#listener-configuration)\.

**To add a listener using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select a load balancer, and choose **Listeners**, **Add listener**\.

1. For **Protocol**, choose **HTTP** or **HTTPS \(Secure HTTP\)**\.

1. For **Port**, keep the default port or type a different port\.

1. For **Default target group**, select an available target group\.

1. \[HTTPS Listener\] For **Select default certificate**, do one of the following:

   + If you created or imported a certificate using AWS Certificate Manager, select **Choose a certificate from ACM**, and then select the certificate from **Certificate name**\.

   + If you uploaded a certificate using IAM, select **Choose a certificate from IAM**, and then select the certificate from **Certificate name**\.

1. \[HTTPS Listener\] For **Security policy**, we recommend that you keep the default security policy\.

1. Choose **Create**\.

1. \(Optional\) To define additional listener rules that forward requests based on a path pattern or a hostname, see [Add a Rule](listener-update-rules.md#add-rule)\.

**To add a listener using the AWS CLI**  
Use the [create\-listener](http://docs.aws.amazon.com/cli/latest/reference/elbv2/create-listener.html) command to create the listener and default rule, and the [create\-rule](http://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) command to define additional listener rules\.