# Create a Listener for Your Application Load Balancer<a name="create-listener"></a>

A listener is a process that checks for connection requests\. You define a listener when you create your load balancer, and you can add listeners to your load balancer at any time\.

## Prerequisites<a name="listener-prereqs"></a>
+ To add a forward action to the default listener rule, you must specify an available target group\. For more information, see [Create a Target Group](create-target-group.md)\.
+ To create an HTTPS listener, you must specify a certificate and a security policy\. The load balancer uses the certificate to terminate the connection and decrypt requests from clients before routing them to targets\. For more information, see [SSL Certificates](create-https-listener.md#https-listener-certificates)\. The load balancer uses the security policy when negotiating SSL connections with the clients\. For more information, see [Security Policies](create-https-listener.md#describe-ssl-policies)\.

## Add a Listener<a name="add-listener"></a>

You configure a listener with a protocol and a port for connections from clients to the load balancer, and a target group for the default listener rule\. For more information, see [Listener Configuration](load-balancer-listeners.md#listener-configuration)\.

**To add a listener using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select a load balancer, and choose **Listeners**, **Add listener**\.

1. For **Protocol : port**, choose **HTTP** or **HTTPS**\. Keep the default port or type a different port\.

1. \(Optional, HTTPS listeners\) To authenticate users, for **Default actions**, choose **Add action**, **Authenticate** and provide the requested information\. To save the action, choose the checkmark icon\. For more information, see [Authenticate Users Using an Application Load Balancer](listener-authenticate-users.md)\.

1. For **Default actions**, do one of the following:
   + Choose **Add action**, **Forward to** and choose a target group\.
   + Choose **Add action**, **Redirect to** and provide the URL for the redirect\. For more information, see [Redirect Actions](load-balancer-listeners.md#redirect-actions)\.
   + Choose **Add action**, **Return fixed response** and provide a response code and optional response body\. For more information, see [Fixed\-Response Actions](load-balancer-listeners.md#fixed-response-actions)\.

   To save the action, choose the checkmark icon\.

1. \[HTTPS listeners\] For **Security policy**, we recommend that you keep the default security policy\.

1. \[HTTPS listeners\] For **Default SSL certificate**, do one of the following:
   + If you created or imported a certificate using AWS Certificate Manager, choose **From ACM** and choose the certificate\.
   + If you uploaded a certificate using IAM, choose **From IAM** and choose the certificate\.

1. Choose **Save**\.

1. \(Optional\) To define additional listener rules that forward requests based on a path pattern or a hostname, see [Add a Rule](listener-update-rules.md#add-rule)\.

**To add a listener using the AWS CLI**  
Use the [create\-listener](http://docs.aws.amazon.com/cli/latest/reference/elbv2/create-listener.html) command to create the listener and default rule, and the [create\-rule](http://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) command to define additional listener rules\.