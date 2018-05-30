# Delete a Target Group<a name="delete-target-group"></a>

If a target group is not referenced by any actions, you can delete it\. Deleting a target group does not affect the targets registered with the target group\. If you no longer need a registered EC2 instance, you can stop or terminate it\.

**To delete a target group using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group and choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes**\.

**To delete a target group using the AWS CLI**  
Use the [delete\-target\-group](http://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-target-group.html) command\.