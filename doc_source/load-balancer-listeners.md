# Listeners for Your Application Load Balancers<a name="load-balancer-listeners"></a>

Before you start using your Application Load Balancer, you must add one or more *listeners*\. A listener is a process that checks for connection requests, using the protocol and port that you configure\. The rules that you define for a listener determine how the load balancer routes requests to the targets in one or more target groups\.

**Topics**
+ [Listener Configuration](#listener-configuration)
+ [Listener Rules](#listener-rules)
+ [Redirect Actions](#redirect-actions)
+ [Fixed\-Response Actions](#fixed-response-actions)
+ [Host Conditions](#host-conditions)
+ [Path Conditions](#path-conditions)
+ [Create an HTTP Listener](create-listener.md)
+ [Create an HTTPS Listener](create-https-listener.md)
+ [Update Listener Rules](listener-update-rules.md)
+ [Update an HTTPS Listener](listener-update-certificates.md)
+ [Authenticate Users](listener-authenticate-users.md)
+ [Delete a Listener](delete-listener.md)

## Listener Configuration<a name="listener-configuration"></a>

Listeners support the following protocols and ports:
+ **Protocols**: HTTP, HTTPS
+ **Ports**: 1\-65535

You can use an HTTPS listener to offload the work of encryption and decryption to your load balancer so that your applications can focus on their business logic\. If the listener protocol is HTTPS, you must deploy at least one SSL server certificate on the listener\. For more information, see [Create an HTTPS Listener for Your Application Load Balancer](create-https-listener.md)\.

Application Load Balancers provide native support for WebSockets\. You can use WebSockets with both HTTP and HTTPS listeners\.

Application Load Balancers provide native support for HTTP/2 with HTTPS listeners\. You can send up to 128 requests in parallel using one HTTP/2 connection\. The load balancer converts these to individual HTTP/1\.1 requests and distributes them across the healthy targets in the target group\. Because HTTP/2 uses front\-end connections more efficiently, you might notice fewer connections between clients and the load balancer\. You can't use the server\-push feature of HTTP/2\.

For more information, see [Request Routing](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html#request-routing) in the *Elastic Load Balancing User Guide*\.

## Listener Rules<a name="listener-rules"></a>

Each listener has a default rule, and you can optionally define additional rules\. Each rule consists of a priority, one or more actions, an optional host condition, and an optional path condition\.

### Default Rules<a name="listener-default-rule"></a>

When you create a listener, you define actions for the default rule\. Default rules can't have conditions\. If no conditions for any of a listener's rules are met, then the action for the default rule is performed\.

The following is an example of a default rule as shown in the console:

![\[The default rule for a listener.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/default_rule.png)

### Rule Priority<a name="listener-rule-priority"></a>

Each rule has a priority\. Rules are evaluated in priority order, from the lowest value to the highest value\. The default rule is evaluated last\. You can change the priority of a nondefault rule at any time\. You cannot change the priority of the default rule\. For more information, see [Reorder Rules](listener-update-rules.md#update-rule-priority)\.

### Rule Actions<a name="listener-rule-actions"></a>

Each rule action has a type, an order, and information required to perform the action\. The following are the supported action types:

`authenticate-cognito`  
\[HTTPS listeners\] Use Amazon Cognito to authenticate users\.

`authenticate-oidc`  
\[HTTPS listeners\] Use an identity provider that is compliant with OpenID Connect \(OIDC\) to authenticate users\.

`fixed-response`  
Return a custom HTTP response\.

`forward`  
Forward requests to the specified target group\.

`redirect`  
Redirect requests from one URL to another\.

The action with the lowest order value is performed first\. The action to be performed last must be a `forward`, `redirect`, or `fixed-response` action\. Each rule must include exactly one of the following types of actions: `forward`, `redirect`, or `fixed-response`\.

You can edit a rule at any time\. For more information, see [Edit a Rule](listener-update-rules.md#edit-rule)\.

### Rule Conditions<a name="listener-rule-conditions"></a>

There are two types of rule conditions: host and path\. Each rule can have up to one host condition and up to one path condition\. When the conditions for a rule are met, then its action is performed\.

## Redirect Actions<a name="redirect-actions"></a>

You can use `redirect` actions to redirect client requests from one URL to another\. You can configure redirects as either temporary \(HTTP 302\) or permanent \(HTTP 301\) based on your needs\.

A URI consists of the following components:

```
protocol://hostname:port/path?query
```

You must modify at least one of the following components to avoid a redirect loop: protocol, hostname, port, or path\. Any components that you do not modify retain their original values\.

*protocol*  
The protocol \(HTTP or HTTPS\)\. You can redirect HTTP to HTTP, HTTP to HTTPS, and HTTPS to HTTPS\. You cannot redirect HTTPS to HTTP\.

*hostname*  
The hostname\.

*port*  
The port \(1 to 65535\)\.

*path*  
The absolute path, starting with the leading "/"\.

*query*  
The query parameters\.

You can reuse URI components of the original URL in the target URL using the following reserved keywords:
+ `#{protocol}` \- Retains the protocol\. Use in the protocol and query components
+ `#{host}` \- Retains the domain\. Use in the hostname, path, and query components
+ `#{port}` \- Retains the port\. Use in the port, path, and query components
+ `#{path}` \- Retains the path\. Use in the path and query components
+ `#{query}` \- Retains the query parameters\. Use in the query component

For example, the following rule sets up a permanent redirect to a URL that uses the HTTPS protocol and the specified port \(40443\), but retains the original hostname, path, and query parameters\. This screen is equivalent to "https://\#\{host\}:40443/\#\{path\}?\#\{query\}"\.

![\[A rule that redirects the request to a URL that uses the HTTPS protocol and the specified port (40443), but retains the original domain, path, and query parameters of the original URL.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/redirect_https_port.png)

The following rule sets up a permanent redirect to a URL that retains the original protocol, port, hostname, and query parameters, and uses the `#{path}` keyword to create a modified path\. This screen is equivalent to "\#\{protocol\}://\#\{host\}:\#\{port\}/new/\#\{path\}?\#\{query\}"\.

![\[A rule that redirects the request to a URL that retains the original protocol, port, hostname, and query parameters, and uses the #{path} keyword to create a modified path.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/redirect_path.png)

When a `redirect` action is taken, the action is recorded in the access logs\. For more information, see [Access Log Entries](load-balancer-access-logs.md#access-log-entry-format)\. The count of successful `redirect` actions is reported in the `HTTP_Redirect_Count` metric\. For more information, see [Application Load Balancer Metrics](load-balancer-cloudwatch-metrics.md#load-balancer-metrics-alb)\.

To add a `redirect` action to an existing rule or create a new rule with a `redirect` action, see [Listener Rules for Your Application Load Balancer](listener-update-rules.md)\.

## Fixed\-Response Actions<a name="fixed-response-actions"></a>

You can use `fixed-response` actions to drop client requests and return a custom HTTP response\. You can use this action to return a 2XX, 4XX, or 5XX response code and an optional message\.

When a `fixed-response` action is taken, the action and the URL of the redirect target are recorded in the access logs\. For more information, see [Access Log Entries](load-balancer-access-logs.md#access-log-entry-format)\. The count of successful `fixed-response` actions is reported in the `HTTP_Fixed_Response_Count` metric\. For more information, see [Application Load Balancer Metrics](load-balancer-cloudwatch-metrics.md#load-balancer-metrics-alb)\.

To add a `fixed-response` action to an existing rule or create a new rule with a `fixed-response` action, see [Listener Rules for Your Application Load Balancer](listener-update-rules.md)\.

## Host Conditions<a name="host-conditions"></a>

You can use host conditions to define rules that forward requests to different target groups based on the host name in the host header \(also known as *host\-based routing*\)\. This enables you to support multiple domains using a single load balancer\.

Each host condition has one hostname\. If the hostname in the host header matches the hostname in a listener rule exactly, the request is routed using that rule\.

A hostname is case\-insensitive, can be up to 128 characters in length, and can contain any of the following characters:
+ A–Z, a–z, 0–9
+ \- \.
+ \* \(matches 0 or more characters\)
+ ? \(matches exactly 1 character\)

You can include up to three wildcard characters\. You must include at least one "\." character\. You can include only alphabetical characters after the final "\." character\.

**Example hostnames**
+ **example\.com**
+ **test\.example\.com**
+ **\*\.example\.com**

The rule **\*\.example\.com** matches **test\.example\.com** but doesn't match **example\.com**\.

**Console Example**  
The following is an example of a rule with a host condition as shown in the console\. If the hostname in the host header matches **\*\.example\.com**, the request is forwarded to the target group named **my\-web\-servers**\. For more information, see [Add a Rule](listener-update-rules.md#add-rule)\.

![\[A listener rule with a host condition.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/host_condition.png)

## Path Conditions<a name="path-conditions"></a>

You can use path conditions to define rules that forward requests to different target groups based on the URL in the request \(also known as *path\-based routing*\)\.

Each path condition has one path pattern\. If the path of the URL in a request matches the path pattern in a listener rule, the request is routed using that rule\. The path pattern is applied only to the path of the URL, not to its query parameters\.

A path pattern is case\-sensitive, can be up to 128 characters in length, and can contain any of the following characters\. You can include up to three wildcard characters\.
+ A–Z, a–z, 0–9
+ \_ \- \. $ / \~ " ' @ : \+
+ & \(using &amp;\)
+ \* \(matches 0 or more characters\)
+ ? \(matches exactly 1 character\)

**Example path patterns**
+ `/img/*`
+ `/js/*`

The path pattern is used to route requests but does not alter them\. For example, if a rule has a path pattern of `/img/*`, the rule would forward a request for `/img/picture.jpg` to the specified target group as a request for `/img/picture.jpg`\.

**Console Example**  
The following is an example of a rule with a path condition as shown in the console\. If the URL in the request matches `/img/*`, the request is forwarded to the target group named **my\-targets**\. For more information, see [Add a Rule](listener-update-rules.md#add-rule)\.

![\[A listener rule with a path condition.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/path_condition.png)