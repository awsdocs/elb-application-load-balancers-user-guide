# Delete a target group<a name="delete-target-group"></a>

You can delete a target group if it is not referenced by the forward actions of any listener rules\. Deleting a target group does not affect the targets registered with the target group\. If you no longer need a registered EC2 instance, you can stop or terminate it\.

------
#### [ New console ]

**To delete a target group using the new console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group and choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes, delete**\.

------
#### [ Old console ]

**To delete a target group using the old console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select the target group and choose **Actions**, **Delete**\.

1. When prompted for confirmation, choose **Yes**\.

------

**To delete a target group using the AWS CLI**  
Use the [delete\-target\-group](https://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-target-group.html) command\.