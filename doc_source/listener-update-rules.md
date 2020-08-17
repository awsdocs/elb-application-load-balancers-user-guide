# Listener rules for your Application Load Balancer<a name="listener-update-rules"></a>

The rules that you define for your listener determine how the load balancer routes requests to the targets in one or more target groups\.

Each rule consists of a priority, one or more actions, and one or more conditions\. For more information, see [Listener rules](load-balancer-listeners.md#listener-rules)\.

**Note**  
The console displays a relative sequence number for each rule, not the rule priority\. You can get the priority of a rule by describing it using the AWS CLI or the Elastic Load Balancing API\.

## Requirements<a name="update-rule-requirements"></a>
+ Each rule must include exactly one of the following actions: `forward`, `redirect`, or `fixed-response`, and it must be the last action to be performed\.
+ Each rule can include zero or one of the following conditions: `host-header`, `http-request-method`, `path-pattern`, and `source-ip`, and zero or more of the following conditions: `http-header` and `query-string`\.
+ You can specify up to three comparison strings per condition and up to five per rule\.
+ A `forward` action routes requests to its target group\. Before you add a `forward` action, create the target group and add targets to it\. For more information, see [Create a target group](create-target-group.md)\.

## Add a rule<a name="add-rule"></a>

You define a default rule when you create a listener, and you can define additional nondefault rules at any time\.

**To add a rule using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. For the listener to update, choose **View/edit rules**\.

1. Choose the **Add rules** icon \(the plus sign\) in the menu bar, which adds **Insert Rule** icons at the locations where you can insert a rule in the priority order\.  
![\[The Add rules icon on the menu bar.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/menu_bar_add_rules.png)

1. Choose one of the **Insert Rule** icons added in the previous step\.

1. Add one or more conditions as follows:

   1. To add a host header condition, choose **Add condition**, **Host header** and enter the hostname \(for example, **\*\.example\.com**\)\. To save the condition, choose the checkmark icon\.

      The maximum size of each string is 128 characters\. The comparison is not case\-sensitive\. The following wildcard characters are supported: \* and ?\.

   1. To add a path condition, choose **Add condition**, **Path** and enter the path pattern \(for example, `/img/*`\)\. To save the condition, choose the checkmark icon\.

      The maximum size of each string is 128 characters\. The comparison is case\-sensitive\. The following wildcard characters are supported: \* and ?\.

   1. To add an HTTP header condition, choose **Add condition**, **Http header**\. Enter the name of the header and add one or more comparison strings\. To save the condition, choose the checkmark icon\.

      The maximum size of each header name is 40 characters, the header name is not case\-sensitive, and wildcards are not supported\. The maximum size of each comparison string is 128 characters and the following wildcard characters are supported: \* and ?\. The comparison is not case\-sensitive\.

   1. To add an HTTP request method condition, choose **Add condition**, **Http request method** and add one or more method names\. To save the condition, choose the checkmark icon\.

      The maximum size of each name is 40 characters\. The allowed characters are A\-Z, hyphen \(\-\), and underscore \(\_\)\. The comparison is case sensitive\. Wildcards are not supported\.

   1. To add a query string condition, choose **Add condition**, **Query string** and add one or more key/value pairs\. For each key/value pair, you can omit the key and specify only the value\. To save the condition, choose the checkmark icon\.

      The maximum size of each string is 128 characters\. The comparison is not case\-sensitive\. The following wildcard characters are supported: \* and ?\.

   1. To add a source IP condition, choose **Add condition**, **Source IP** and add one or more CIDR blocks\. To save the condition, choose the checkmark icon\.

      You can use both IPv4 and IPv6 addresses\. Wildcards are not supported\.

1. \(Optional, HTTPS listener\) To authenticate users, choose **Add action**, **Authenticate** and provide the requested information\. To save the action, choose the checkmark icon\. For more information, see [Authenticate users using an Application Load Balancer](listener-authenticate-users.md)\.

1. Add one of the following actions:
   + To add a forward action, choose **Add action**, **Forward to** and choose one or more target groups\. If you use more than one target group, select a weight for each target group and optionally enable target group stickiness\. If you enable target group stickiness and there is more than one target group, you must also enable sticky sessions on the target groups\. To save the action, choose the checkmark icon\. For more information, see [Forward actions](load-balancer-listeners.md#forward-actions)\.
   + To add a redirect action, choose **Add action**, **Redirect to** and provide the URL for the redirect\. To save the action, choose the checkmark icon\. For more information, see [Redirect actions](load-balancer-listeners.md#redirect-actions)\.
   + To add a fixed\-response action, choose **Add action**, **Return fixed response** and provide a response code and optional response body\. To save the action, choose the checkmark icon\. For more information, see [Fixed\-response actions](load-balancer-listeners.md#fixed-response-actions)\.  
![\[The Insert Rule interface.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/add_rule.png)

1. Choose **Save**\.

1. \(Optional\) To change the order of the rule, use the arrows and then choose **Save**\. The default rule always has the **last** priority\.

1. To leave this screen, choose the **Back to the load balancer** icon \(the back button\) in the menu bar\.

**To add a rule using the AWS CLI**  
Use the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) command to create the rule\. Use the [describe\-rules](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-rules.html) command to view information about the rule\.

## Edit a rule<a name="edit-rule"></a>

You can edit the action and conditions for a rule at any time\. Rule updates do not take effect immediately, so requests could be routed using the previous rule configuration for a short time after you update a rule\. Any in\-flight requests are completed\.

**To edit a rule using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. For the listener to update, choose **View/edit rules**\.

1. Choose the **Edit rules** icon \(the pencil\) in the menu bar\.  
![\[The Edit rules icon on the menu bar.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/menu_bar_edit_rules.png)

1. For the rule to edit, choose the **Edit rules** icon \(the pencil\)\.

1. \(Optional\) Modify the conditions and actions as needed\. For example, you can edit a condition or action \(pencil icon\), add a condition, add an authenticate action to a rule for an HTTPS listener, or delete a condition or action \(trash can icon\)\. You can't add conditions to the default rule\.  
![\[The Edit Rule interface.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/edit_rule.png)

1. Choose **Update**\.

1. To leave this screen, choose the **Back to the load balancer** icon \(the back button\) in the menu bar\.

**To edit a rule using the AWS CLI**  
Use the [modify\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-rule.html) command\.

## Reorder rules<a name="update-rule-priority"></a>

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
Use the [set\-rule\-priorities](https://docs.aws.amazon.com/cli/latest/reference/elbv2/set-rule-priorities.html) command\.

## Delete a rule<a name="delete-rule"></a>

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
Use the [delete\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/delete-rule.html) command\.