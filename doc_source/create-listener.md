# Create an HTTP listener for your Application Load Balancer<a name="create-listener"></a>

A listener is a process that checks for connection requests\. You define a listener when you create your load balancer, and you can add listeners to your load balancer at any time\.

The information on this page helps you create an HTTP listener for your load balancer\. To add an HTTPS listener to your load balancer, see [Create an HTTPS listener for your Application Load Balancer](create-https-listener.md)\.

## Prerequisites<a name="listener-prereqs"></a>
+ To add a forward action to the default listener rule, you must specify an available target group\. For more information, see [Create a target group](create-target-group.md)\.

## Add an HTTP listener<a name="add-listener"></a>

You configure a listener with a protocol and a port for connections from clients to the load balancer, and a target group for the default listener rule\. For more information, see [Listener configuration](load-balancer-listeners.md#listener-configuration)\.

**To add an HTTP listener using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select a load balancer, and choose **Listeners**, **Add listener**\.

1. For **Protocol : port**, choose **HTTP** and keep the default port or enter a different port\.

1. For **Default actions**, do one of the following:
   + Choose **Add action**, **Forward to** and choose a target group\.
   + Choose **Add action**, **Redirect to** and provide the URL for the redirect\. For more information, see [Redirect actions](load-balancer-listeners.md#redirect-actions)\.
   + Choose **Add action**, **Return fixed response** and provide a response code and optional response body\. For more information, see [Fixed\-response actions](load-balancer-listeners.md#fixed-response-actions)\.

   To save the action, choose the checkmark icon\.

1. Choose **Save**\.

1. \(Optional\) To define additional listener rules that forward requests based on a path pattern or a hostname, see [Add a rule](listener-update-rules.md#add-rule)\.

**To add an HTTP listener using the AWS CLI**  
Use the [create\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-listener.html) command to create the listener and default rule, and the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) command to define additional listener rules\.