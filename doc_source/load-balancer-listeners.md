# Listeners for Your Application Load Balancers<a name="load-balancer-listeners"></a>

Before you start using your Application Load Balancer, you must add one or more *listeners*\. A listener is a process that checks for connection requests, using the protocol and port that you configure\. The rules that you define for a listener determine how the load balancer routes requests to the targets in one or more target groups\.

Contents

## Listener Configuration<a name="listener-configuration"></a>

Listeners support the following protocols and ports:

+ **Protocols**: HTTP, HTTPS

+ **Ports**: 1\-65535

You can use an HTTPS listener to offload the work of encryption and decryption to your load balancer so that your targets can focus on their main work\. If the listener protocol is HTTPS, you must deploy exactly one SSL server certificate on the listener\. For more information, see [HTTPS Listeners for Your Application Load Balancer](create-https-listener.md)\.

Application Load Balancers provide native support for Websockets\. You can use WebSockets with both HTTP and HTTPS listeners\.

Application Load Balancers provide native support for HTTP/2 with HTTPS listeners\. You can send up to 128 requests in parallel using one HTTP/2 connection\. The load balancer converts these to individual HTTP/1\.1 requests and distributes them across the healthy targets in the target group using the round robin routing algorithm\. Because HTTP/2 uses front\-end connections more efficiently, you might notice fewer connections between clients and the load balancer\. Note that you can't use the server\-push feature of HTTP/2\.

## Listener Rules<a name="listener-rules"></a>

Each listener has a default rule, and you can optionally define additional rules\. Each rule consists of a priority, action, optional host condition, and optional path condition\.

### Default Rules<a name="listener-default-rule"></a>

When you create a listener, you define an action for the default rule\. Default rules can't have conditions\. If no conditions for any of a listener's rules are met, then the action for the default rule is taken\.

The following is an example of a default rule as shown in the console:

![\[The default rule for a listener.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/default_rule.png)

### Rule Priority<a name="listener-rule-priority"></a>

Each rule has a priority\. Rules are evaluated in priority order, from the lowest value to the highest value\. The default rule is evaluated last\. You can change the priority of a nondefault rule at any time\. You cannot change the priority of the default rule\. For more information, see [Reorder Rules](listener-update-rules.md#update-rule-priority)\.

### Rule Actions<a name="listener-rule-actions"></a>

Each rule action has a type and a target group\. Currently, the only supported type is `forward`, which forwards requests to the target group\. You can change the target group for a rule at any time\. For more information, see [Edit a Rule](listener-update-rules.md#edit-rule)\.

### Rule Conditions<a name="listener-rule-conditions"></a>

There are two types of rule conditions: host and path\. When the conditions for a rule are met, then its action is taken\.

## Host Conditions<a name="host-conditions"></a>

You can use host conditions to define rules that forward requests to different target groups based on the host name in the host header \(also known as *host\-based routing*\)\. This enables you to support multiple domains using a single load balancer\.

Each host condition has one hostname\. If the hostname in the host header matches the hostname in a listener rule exactly, the request is routed using that rule\.

A hostname is case\-insensitive, can be up to 128 characters in length, and can contain any of the following characters\. Note that you can include up to three wildcard characters\.

+ A–Z, a–z, 0–9

+ \- \.

+ \* \(matches 0 or more characters\)

+ ? \(matches exactly 1 character\)

**Example hostnames**

+ **example\.com**

+ **test\.example\.com**

+ **\*\.example\.com**

Note that **\*\.example\.com** will match **test\.example\.com** but won't match **example\.com**\.

**Console Example**  
The following is an example of a rule with a host condition as shown in the console\. If the hostname in the host header matches **\*\.example\.com**, the request is forwarded to the target group named **my\-web\-servers**\. For more information, see [Add a Rule](listener-update-rules.md#add-rule)\.

![\[A listener rule with a host condition.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/host_condition.png)

## Path Conditions<a name="path-conditions"></a>

You can use path conditions to define rules that forward requests to different target groups based on the URL in the request \(also known as *path\-based routing*\)\.

Each path condition has one path pattern\. If the URL in a request matches the path pattern in a listener rule exactly, the request is routed using that rule\.

A path pattern is case\-sensitive, can be up to 128 characters in length, and can contain any of the following characters\. Note that you can include up to three wildcard characters\.

+ A–Z, a–z, 0–9

+ \_ \- \. $ / \~ " ' @ : \+

+ & \(using &amp;\)

+ \* \(matches 0 or more characters\)

+ ? \(matches exactly 1 character\)

**Example path patterns**

+ `/img/*`

+ `/js/*`

Note that the path pattern is used to route requests but does not alter them\. For example, if a rule has a path pattern of `/img/*`, the rule would forward a request for `/img/picture.jpg` to the specified target group as a request for `/img/picture.jpg`\.

**Console Example**  
The following is an example of a rule with a path condition as shown in the console\. If the URL in the request matches `/img/*`, the request is forwarded to the target group named **my\-targets**\. For more information, see [Add a Rule](listener-update-rules.md#add-rule)\.

![\[A listener rule with a path condition.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/path_condition.png)