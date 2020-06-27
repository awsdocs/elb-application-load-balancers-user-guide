# Delete a listener for your Application Load Balancer<a name="delete-listener"></a>

You can delete a listener at any time\. When you delete a load balancer, all its listeners are deleted\.

**To delete a listener using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. Select the check box for the HTTPS listener and choose **Delete**\.

1. When prompted for confirmation, choose **Yes, Delete**\.

**To delete a listener using the AWS CLI**  
Use the [delete\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-listener.html) command\.