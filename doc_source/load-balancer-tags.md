# Tags for your Application Load Balancer<a name="load-balancer-tags"></a>

Tags help you to categorize your load balancers in different ways, for example, by purpose, owner, or environment\.

You can add multiple tags to each load balancer\. Tag keys must be unique for each load balancer\. If you add a tag with a key that is already associated with the load balancer, it updates the value of that tag\.

When you are finished with a tag, you can remove it from your load balancer\.

**Restrictions**
+ Maximum number of tags per resource—50
+ Maximum key length—127 Unicode characters
+ Maximum value length—255 Unicode characters
+ Tag keys and values are case sensitive\. Allowed characters are letters, spaces, and numbers representable in UTF\-8, plus the following special characters: \+ \- = \. \_ : / @\. Do not use leading or trailing spaces\.
+ Do not use the `aws:` prefix in your tag names or values because it is reserved for AWS use\. You can't edit or delete tag names or values with this prefix\. Tags with this prefix do not count against your tags per resource limit\. 

**To update the tags for a load balancer using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Tags** tab, choose **Add/Edit Tags**, and then do one or more of the following:

   1. To update a tag, edit the values of **Key** and **Value**\.

   1. To add a new tag, choose **Create Tag** and then enter values for **Key** and **Value**\.

   1. To delete a tag, choose the delete icon \(X\) next to the tag\.

1. When you have finished updating tags, choose **Save**\.

**To update the tags for a load balancer using the AWS CLI**  
Use the [add\-tags](https://docs.aws.amazon.com/cli/latest/reference/elbv2/add-tags.html) and [remove\-tags](https://docs.aws.amazon.com/cli/latest/reference/elbv2/remove-tags.html) commands\.