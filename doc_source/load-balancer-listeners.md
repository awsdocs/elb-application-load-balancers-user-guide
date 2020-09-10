# Listeners for your Application Load Balancers<a name="load-balancer-listeners"></a>

Before you start using your Application Load Balancer, you must add one or more *listeners*\. A listener is a process that checks for connection requests, using the protocol and port that you configure\. The rules that you define for a listener determine how the load balancer routes requests to its registered targets\.

**Topics**
+ [Listener configuration](#listener-configuration)
+ [Listener rules](#listener-rules)
+ [Rule action types](#rule-action-types)
+ [Rule condition types](#rule-condition-types)
+ [Create an HTTP listener](create-listener.md)
+ [Create an HTTPS listener](create-https-listener.md)
+ [Update listener rules](listener-update-rules.md)
+ [Update an HTTPS listener](listener-update-certificates.md)
+ [Authenticate users](listener-authenticate-users.md)
+ [Delete a listener](delete-listener.md)

## Listener configuration<a name="listener-configuration"></a>

Listeners support the following protocols and ports:
+ **Protocols**: HTTP, HTTPS
+ **Ports**: 1\-65535

You can use an HTTPS listener to offload the work of encryption and decryption to your load balancer so that your applications can focus on their business logic\. If the listener protocol is HTTPS, you must deploy at least one SSL server certificate on the listener\. For more information, see [Create an HTTPS listener for your Application Load Balancer](create-https-listener.md)\.

Application Load Balancers provide native support for WebSockets\. You can use WebSockets with both HTTP and HTTPS listeners\.

Application Load Balancers provide native support for HTTP/2 with HTTPS listeners\. You can send up to 128 requests in parallel using one HTTP/2 connection\. The load balancer converts these to individual HTTP/1\.1 requests and distributes them across the healthy targets in the target group\. Because HTTP/2 uses front\-end connections more efficiently, you might notice fewer connections between clients and the load balancer\. You can't use the server\-push feature of HTTP/2\.

For more information, see [Request routing](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html#request-routing) in the *Elastic Load Balancing User Guide*\.

## Listener rules<a name="listener-rules"></a>

Each listener has a default rule, and you can optionally define additional rules\. Each rule consists of a priority, one or more actions, and one or more conditions\. You can add or edit rules at any time\. For more information, see [Edit a rule](listener-update-rules.md#edit-rule)\.

### Default rules<a name="listener-default-rule"></a>

When you create a listener, you define actions for the default rule\. Default rules can't have conditions\. If the conditions for none of a listener's rules are met, then the action for the default rule is performed\.

The following is an example of a default rule as shown in the console:

![\[The default rule for a listener.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/default_rule.png)

### Rule priority<a name="listener-rule-priority"></a>

Each rule has a priority\. Rules are evaluated in priority order, from the lowest value to the highest value\. The default rule is evaluated last\. You can change the priority of a nondefault rule at any time\. You cannot change the priority of the default rule\. For more information, see [Reorder rules](listener-update-rules.md#update-rule-priority)\.

### Rule actions<a name="listener-rule-actions"></a>

Each rule action has a type, an order, and the information required to perform the action\. For more information, see [Rule action types](#rule-action-types)\.

### Rule conditions<a name="listener-rule-conditions"></a>

Each rule condition has a type and configuration information\. When the conditions for a rule are met, then its actions are performed\. For more information, see [Rule condition types](#rule-condition-types)\.

## Rule action types<a name="rule-action-types"></a>

The following are the supported action types for a rule:

`authenticate-cognito`  
\[HTTPS listeners\] Use Amazon Cognito to authenticate users\. For more information, see [Authenticate users using an Application Load Balancer](listener-authenticate-users.md)\.

`authenticate-oidc`  
\[HTTPS listeners\] Use an identity provider that is compliant with OpenID Connect \(OIDC\) to authenticate users\.

`fixed-response`  
Return a custom HTTP response\. For more information, see [Fixed\-response actions](#fixed-response-actions)\.

`forward`  
Forward requests to the specified target groups\. For more information, see [Forward actions](#forward-actions)\.

`redirect`  
Redirect requests from one URL to another\. For more information, see [Redirect actions](#redirect-actions)\.

The action with the lowest order value is performed first\. Each rule must include exactly one of the following actions: `forward`, `redirect`, or `fixed-response`, and it must be the last action to be performed\.

### Fixed\-response actions<a name="fixed-response-actions"></a>

You can use `fixed-response` actions to drop client requests and return a custom HTTP response\. You can use this action to return a 2XX, 4XX, or 5XX response code and an optional message\.

When a `fixed-response` action is taken, the action and the URL of the redirect target are recorded in the access logs\. For more information, see [Access log entries](load-balancer-access-logs.md#access-log-entry-format)\. The count of successful `fixed-response` actions is reported in the `HTTP_Fixed_Response_Count` metric\. For more information, see [Application Load Balancer metrics](load-balancer-cloudwatch-metrics.md#load-balancer-metrics-alb)\.

**Example fixed response action for the AWS CLI**  
You can specify an action when you create or modify a rule\. For more information, see the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) and [modify\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-rule.html) commands\. The following action sends a fixed response with the specified status code and message body\.  

```
[
  {
      "Type": "fixed-response",
      "FixedResponseConfig": {
          "StatusCode": "200",
          "ContentType": "text/plain",
          "MessageBody": "Hello world"
      }
  }
]
```

### Forward actions<a name="forward-actions"></a>

You can use `forward` actions to route requests to one or more target groups\. If you specify multiple target groups for a `forward` action, you must specify a weight for each target group\. Each target group weight is a value from 0 to 999\. Requests that match a listener rule with weighted target groups are distributed to these target groups based on their weights\. For example, if you specify two target groups, each with a weight of 10, each target group receives half the requests\. If you specify two target groups, one with a weight of 10 and the other with a weight of 20, the target group with a weight of 20 receives twice as many requests as the other target group\.

By default, configuring a rule to distribute traffic between weighted target groups does not guarantee that sticky sessions are honored\. To ensure that sticky sessions are honored, enable target group stickiness for the rule\. When the load balancer first routes a request to a weighted target group, it generates a cookie named AWSALBTG that encodes information about the selected target group, encrypts the cookie, and includes the cookie in the response to the client\. The client should include the cookie that it receives in subsequent requests to the load balancer\. When the load balancer receives a request that matches a rule with target group stickiness enabled and contains the cookie, the request is routed to the target group specified in the cookie\.

Application Load Balancers do not support cookie values that are URL encoded\.

With CORS \(cross\-origin resource sharing\) requests, some browsers require `SameSite=None; Secure` to enable stickiness\. In this case, Elastic Load Balancing generates a second cookie, AWSALBTGCORS, which includes the same information as the original stickiness cookie plus this `SameSite` attribute\. Clients receive both cookies\.

**Example forward action with one target group**  
You can specify an action when you create or modify a rule\. For more information, see the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) and [modify\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-rule.html) commands\. The following action forwards requests to the specified target group\.  

```
[
  {
      "Type": "forward",
      "ForwardConfig": {
          "TargetGroups": [
              {
                  "TargetGroupArn": "arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/my-targets/73e2d6bc24d8a067"
              }
          ]
      }
  }
]
```

**Example forward action with two weighted target groups**  
The following action forwards requests to the two specified target groups, based on the weight of each target group\.  

```
[
  {
      "Type": "forward",
      "ForwardConfig": {
          "TargetGroups": [
              {
                  "TargetGroupArn": "arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/blue-targets/73e2d6bc24d8a067",
                  "Weight": 10
              },
              {
                  "TargetGroupArn": "arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/green-targets/09966783158cda59",
                  "Weight": 20
              }
          ]
      }
  }
]
```

**Example forward action with stickiness enabled**  
If you have a forward rule with multiple target groups and one or more of the target groups has [sticky sessions](load-balancer-target-groups.md#sticky-sessions) enabled, you must enable target group stickiness\.  
The following action forwards requests to the two specified target groups, with target group stickiness enabled\. Requests that do not contain the stickiness cookies are routed based on the weight of each target group\.  

```
[
  {
      "Type": "forward",
      "ForwardConfig": {
          "TargetGroups": [
              {
                  "TargetGroupArn": "arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/blue-targets/73e2d6bc24d8a067",
                  "Weight": 10
              },
              {
                  "TargetGroupArn": "arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/green-targets/09966783158cda59",
                  "Weight": 20
              }
          ],
          "TargetGroupStickinessConfig": {
              "Enabled": true,
              "DurationSeconds": 1000
          }
      }
  }
]
```

### Redirect actions<a name="redirect-actions"></a>

You can use `redirect` actions to redirect client requests from one URL to another\. You can configure redirects as either temporary \(HTTP 302\) or permanent \(HTTP 301\) based on your needs\.

A URI consists of the following components:

```
protocol://hostname:port/path?query
```

You must modify at least one of the following components to avoid a redirect loop: protocol, hostname, port, or path\. Any components that you do not modify retain their original values\.

*protocol*  
The protocol \(HTTP or HTTPS\)\. You can redirect HTTP to HTTP, HTTP to HTTPS, and HTTPS to HTTPS\. You cannot redirect HTTPS to HTTP\.

*hostname*  
The hostname\. A hostname is not case\-sensitive, can be up to 128 characters in length, and consists of alpha\-numeric characters, wildcards \(\* and ?\), and hyphens \(\-\)\.

*port*  
The port \(1 to 65535\)\.

*path*  
The absolute path, starting with the leading "/"\. A path is case\-sensitive, can be up to 128 characters in length, and consists of alpha\-numeric characters, wildcards \(\* and ?\), & \(using &amp;\), and the following special characters: \_\-\.$/\~"'@:\+\.

*query*  
The query parameters\. The maximum length is 128 characters\.

You can reuse URI components of the original URL in the target URL using the following reserved keywords:
+ `#{protocol}` \- Retains the protocol\. Use in the protocol and query components\.
+ `#{host}` \- Retains the domain\. Use in the hostname, path, and query components\.
+ `#{port}` \- Retains the port\. Use in the port, path, and query components\.
+ `#{path}` \- Retains the path\. Use in the path and query components\.
+ `#{query}` \- Retains the query parameters\. Use in the query component\.

When a `redirect` action is taken, the action is recorded in the access logs\. For more information, see [Access log entries](load-balancer-access-logs.md#access-log-entry-format)\. The count of successful `redirect` actions is reported in the `HTTP_Redirect_Count` metric\. For more information, see [Application Load Balancer metrics](load-balancer-cloudwatch-metrics.md#load-balancer-metrics-alb)\.

**Example redirect actions using the console**  
The following rule sets up a permanent redirect to a URL that uses the HTTPS protocol and the specified port \(40443\), but retains the original hostname, path, and query parameters\. This screen is equivalent to "https://\#\{host\}:40443/\#\{path\}?\#\{query\}"\.  

![\[A rule that redirects the request to a URL that uses the HTTPS protocol and the specified port (40443), but retains the original domain, path, and query parameters of the original URL.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/redirect_https_port.png)
The following rule sets up a permanent redirect to a URL that retains the original protocol, port, hostname, and query parameters, and uses the `#{path}` keyword to create a modified path\. This screen is equivalent to "\#\{protocol\}://\#\{host\}:\#\{port\}/new/\#\{path\}?\#\{query\}"\.  

![\[A rule that redirects the request to a URL that retains the original protocol, port, hostname, and query parameters, and uses the #{path} keyword to create a modified path.\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/redirect_path.png)

**Example redirect action for the AWS CLI**  
You can specify an action when you create or modify a rule\. For more information, see the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) and [modify\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-rule.html) commands\. The following action redirects an HTTP request to an HTTPS request on port 443, with the same host name, path, and query string as the HTTP request\.  

```
[
  {
      "Type": "redirect",
      "RedirectConfig": {
          "Protocol": "HTTPS",
          "Port": "443",
          "Host": "#{host}",
          "Path": "/#{path}",
          "Query": "#{query}",
          "StatusCode": "HTTP_301"
      }
  }
]
```

## Rule condition types<a name="rule-condition-types"></a>

The following are the supported condition types for a rule:

`host-header`  
Route based on the host name of each request\. For more information, see [Host conditions](#host-conditions)\.

`http-header`  
Route based on the HTTP headers for each request\. For more information, see [HTTP header conditions](#http-header-conditions)\.

`http-request-method`  
Route based on the HTTP request method of each request\. For more information, see [HTTP request method conditions](#http-request-method-conditions)\.

`path-pattern`  
Route based on path patterns in the request URLs\. For more information, see [Path conditions](#path-conditions)\.

`query-string`  
Route based on key/value pairs or values in the query strings\. For more information, see [Query string conditions](#query-string-conditions)\.

`source-ip`  
Route based on the source IP address of each request\. For more information, see [Source IP address conditions](#source-ip-conditions)\.

Each rule can optionally include up to one of each of the following conditions: `host-header`, `http-request-method`, `path-pattern`, and `source-ip`\. Each rule can also optionally include one or more of each of the following conditions: `http-header` and `query-string`\.

You can specify up to three match evaluations per condition\. For example, for each `http-header` condition, you can specify up to three strings to be compared to the value of the HTTP header in the request\. The condition is satisfied if one of the strings matches the value of the HTTP header\. To require that all of the strings are a match, create one condition per match evaluation\.

You can specify up to five match evaluations per rule\. For example, you can create a rule with five conditions where each condition has one match evaluation\.

You can include wildcard characters in the match evaluations for the `http-header`, `host-header`, `path-pattern`, and `query-string` conditions\. There is a limit of five wildcard characters per rule\.

Rules are applied only to visible ASCII characters; control characters \(0x00 to 0x1f and 0x7f\) are excluded\.

For demos, see [Advanced request routing](https://exampleloadbalancer.com/advanced_request_routing_demo.html)\.

### HTTP header conditions<a name="http-header-conditions"></a>

You can use HTTP header conditions to configure rules that route requests based on the HTTP headers for the request\. You can specify the names of standard or custom HTTP header fields\. The header name and the match evaluation are not case\-sensitive\. The following wildcard characters are supported in the comparison strings: \* \(matches 0 or more characters\) and ? \(matches exactly 1 character\)\. Wildcard characters are not supported in the header name\.

**Example HTTP header condition for the AWS CLI**  
You can specify conditions when you create or modify a rule\. For more information, see the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) and [modify\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify_rule.html) commands\. The following condition is satisfied by requests with a User\-Agent header that matches one of the specified strings\.  

```
[
  {
      "Field": "http-header",
      "HttpHeaderConfig": {
          "HttpHeaderName": "User-Agent",
          "Values": ["*Chrome*", "*Safari*"]
      }
  }
]
```

### HTTP request method conditions<a name="http-request-method-conditions"></a>

You can use HTTP request method conditions to configure rules that route requests based on the HTTP request method of the request\. You can specify standard or custom HTTP methods\. The match evaluation is case\-sensitive\. Wildcard characters are not supported; therefore, the method name must be an exact match\.

We recommend that you route GET and HEAD requests in the same way, because the response to a HEAD request may be cached\.

**Example HTTP method condition for the AWS CLI**  
You can specify conditions when you create or modify a rule\. For more information, see the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) and [modify\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify_rule.html) commands\. The following condition is satisfied by requests that use the specified method\.  

```
[
  {
      "Field": "http-request-method",
      "HttpRequestMethodConfig": {
          "Values": ["CUSTOM-METHOD"]
      }
  }
]
```

### Host conditions<a name="host-conditions"></a>

You can use host conditions to define rules that route requests based on the host name in the host header \(also known as *host\-based routing*\)\. This enables you to support multiple subdomains and different top\-level domains using a single load balancer\.

A hostname is not case\-sensitive, can be up to 128 characters in length, and can contain any of the following characters:
+ A–Z, a–z, 0–9
+ \- \.
+ \* \(matches 0 or more characters\)
+ ? \(matches exactly 1 character\)

You must include at least one "\." character\. You can include only alphabetical characters after the final "\." character\.

**Example hostnames**
+ **example\.com**
+ **test\.example\.com**
+ **\*\.example\.com**

The rule **\*\.example\.com** matches **test\.example\.com** but doesn't match **example\.com**\.

**Example host header condition for the AWS CLI**  
You can specify conditions when you create or modify a rule\. For more information, see the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) and [modify\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify_rule.html) commands\. The following condition is satisfied by requests with a host header that matches the specified string\.  

```
[
  {
      "Field": "host-header",
      "HostHeaderConfig": {
          "Values": ["*.example.com"]
      }
  }
]
```

### Path conditions<a name="path-conditions"></a>

You can use path conditions to define rules that route requests based on the URL in the request \(also known as *path\-based routing*\)\.

The path pattern is applied only to the path of the URL, not to its query parameters\. It is applied only to visible ASCII characters; control characters \(0x00 to 0x1f and 0x7f\) are excluded\.

A path pattern is case\-sensitive, can be up to 128 characters in length, and can contain any of the following characters\.
+ A–Z, a–z, 0–9
+ \_ \- \. $ / \~ " ' @ : \+
+ & \(using &amp;\)
+ \* \(matches 0 or more characters\)
+ ? \(matches exactly 1 character\)

**Example path patterns**
+ `/img/*`
+ `/img/*/pics`

The path pattern is used to route requests but does not alter them\. For example, if a rule has a path pattern of `/img/*`, the rule would forward a request for `/img/picture.jpg` to the specified target group as a request for `/img/picture.jpg`\.

**Example path pattern condition for the AWS CLI**  
You can specify conditions when you create or modify a rule\. For more information, see the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) and [modify\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify_rule.html) commands\. The following condition is satisfied by requests with a URL that contains the specified string\.  

```
[
  {
      "Field": "path-pattern",
      "PathPatternConfig": {
          "Values": ["/img/*"]
      }
  }
]
```

### Query string conditions<a name="query-string-conditions"></a>

You can use query string conditions to configure rules that route requests based on key/value pairs or values in the query string\. The match evaluation is not case\-sensitive\. The following wildcard characters are supported: \* \(matches 0 or more characters\) and ? \(matches exactly 1 character\)\.

**Example query string condition for the AWS CLI**  
You can specify conditions when you create or modify a rule\. For more information, see the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) and [modify\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify_rule.html) commands\. The following condition is satisfied by requests with a query string that includes either a key/value pair of "version=v1" or any key set to "example"\.  

```
[
  {
      "Field": "query-string",
      "QueryStringConfig": {
          "Values": [
            {
                "Key": "version", 
                "Value": "v1"
            },
            {
                "Value": "*example*"
            }
          ]
      }
  }
]
```

### Source IP address conditions<a name="source-ip-conditions"></a>

You can use source IP address conditions to configure rules that route requests based on the source IP address of the request\. The IP address must be specified in CIDR format\. You can use both IPv4 and IPv6 addresses\. Wildcard characters are not supported\.

If a client is behind a proxy, this is the IP address of the proxy, not the IP address of the client\.

This condition is not satisfied by the addresses in the X\-Forwarded\-For header\. To search for addresses in the X\-Forwarded\-For header, use an `http-header` condition\.

**Example source IP condition for the AWS CLI**  
You can specify conditions when you create or modify a rule\. For more information, see the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) and [modify\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify_rule.html) commands\. The following condition is satisfied by requests with a source IP address in one of the specified CIDR blocks\.  

```
[
  {
      "Field": "source-ip",
      "SourceIpConfig": {
          "Values": ["192.0.2.0/24", "198.51.100.10/32"]
      }
  }
]
```