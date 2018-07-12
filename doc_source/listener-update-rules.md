# Listener Rules for Your Application Load Balancer<a name="listener-update-rules"></a>

The rules that you define for your listener determine how the load balancer routes requests to the targets in one or more target groups\.

Each rule consists of a priority, one or more actions, an optional host condition, and an optional path condition\. For more information, see [Listener Rules](load-balancer-listeners.md#listener-rules)\.

**Note**  
The console displays a relative sequence number for each rule, not the rule priority\. You can get the priority of a rule by describing it using the AWS CLI or the Elastic Load Balancing API\.

## Prerequisites<a name="update-rule-prerequisites"></a>

A rule routes requests to its target group\. Before you create a rule or update the target group for a rule, create the target group and add targets to it\. For more information, see [Create a Target Group](create-target-group.md)\.

## Add a Rule<a name="add-rule"></a>

You define a default rule when you create a listener, and you can define additional nondefault rules at any time\.

**To add a rule using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. For the listener to update, choose **View/edit rules**\.

1. Choose the **Add rules** icon \(the plus sign\) in the menu bar, which adds **Insert Rule** icons at the locations where you can insert a rule in the priority order\.  
![\[The Add rules icon on the menu bar.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/menu_bar_add_rules.png)

1. Define the rule as follows:

   1. Choose **Insert Rule**\.

   1. \(Optional\) To configure host\-based routing, choose **Add condition**, **Host is** and type the hostname \(for example, **\*\.example\.com**\)\. To save the condition, choose the checkmark icon\.

   1. \(Optional\) To configure path\-based routing, choose **Add condition**, **Path is** and type the path pattern \(for example, `/img/*`\)\. To save the condition, choose the checkmark icon\.

   1. \(Optional, HTTPS listeners\) To authenticate users, choose **Add action**, **Authenticate** and provide the requested information\. For more information, see [Authenticate Users Using an Application Load Balancer](listener-authenticate-users.md)\.

   1. To add a forward action, choose **Add action**, **Forward to** and choose a target group\. Each rule must have exactly one forward action\.

   1. \(Optional\) To change the order of the rule, use the arrows\. The default rule always has the **last** priority\.

   1. Choose **Save**\.  
![\[The Insert Rule interface.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/add_rule.png)

1. To leave this screen, choose the **Back to the load balancer** icon \(the back button\) in the menu bar\.

**To add a rule using the AWS CLI**  
Use the [create\-rule](http://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) command to create the rule\. Use the [describe\-rules](http://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-rules.html) command to view information about the rule\.

## Edit a Rule<a name="edit-rule"></a>

You can edit the action and conditions for a rule at any time\.

**To edit a rule using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. For the listener to update, choose **View/edit rules**\.

1. Choose the **Edit rules** icon \(the pencil\) in the menu bar\.  
![\[The Edit rules icon on the menu bar.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/menu_bar_edit_rules.png)

1. For the rule to edit, choose the **Edit rules** icon \(the pencil\)\.

1. \(Optional\) Modify the conditions and actions as needed\. For example, you can edit a condition or action \(pencil icon\), add a path condition if you don't have one already, add a host condition if you don't have one already, add an authenticate action for a rule for an HTTPS listener, or delete a condition or action \(trash can icon\)\. You can't add conditions to the default rule\. Each rule must have exactly one forward action\.  
![\[The Edit Rule interface.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/edit_rule.png)

1. Choose **Update**\.

1. To leave this screen, choose the **Back to the load balancer** icon \(the back button\) in the menu bar\.

**To edit a rule using the AWS CLI**  
Use the [modify\-rule](http://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-rule.html) command\.

## Reorder Rules<a name="update-rule-priority"></a>

Rules are evaluated in priority order, from the lowest value to the highest value\. The default rule is evaluated last\. You can change the priority of a nondefault rule at any time\. You cannot change the priority of the default rule\.

**Note**  
The console displays a relative sequence number for each rule, not the rule priority\. When you reorder rules using the console, they get new rule priorities based on the existing rule priorities\. To set the priority of a rule to a specific value, use the AWS CLI or the Elastic Load Balancing API\.

**To reorder rules using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. For the listener to update, choose **View/edit rules**\.

1. Choose the **Reorder rules** icon \(the arrows\) in the menu bar\.  
![\[The Reorder rules icon on the menu bar.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/menu_bar_reorder_rules.png)

1. Select the check box next to a rule, and then use the arrows to give the rule a new priority\. The default rule always has the last priority\.

1. When you have finished reordering rules, choose **Save**\.

1. To leave this screen, choose the **Back to the load balancer** icon \(the back button\) in the menu bar\.

**To update rule priorities using the AWS CLI**  
Use the [set\-rule\-priorities](http://docs.aws.amazon.com/cli/latest/reference/elbv2/set-rule-priorities.html) command\.

## Delete a Rule<a name="delete-rule"></a>

You can delete the nondefault rules for a listener at any time\. You cannot delete the default rule for a listener\. When you delete a listener, all its rules are deleted\.

**To delete a rule using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. For the listener to update, choose **View/edit rules**\.

1. Choose the **Delete rules** icon \(the minus sign\) in the menu bar\.

1. Select the check box for the rule and choose **Delete**\. You can't delete the default rule for the listener\.

1. To leave this screen, choose the **Back to the load balancer** icon \(the back button\) in the menu bar\.

**To delete a rule using the AWS CLI**  
Use the [delete\-rule](http://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-rule.html) command\.