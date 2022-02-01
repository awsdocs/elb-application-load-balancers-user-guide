# Sticky sessions for your Application Load Balancer<a name="sticky-sessions"></a>

By default, an Application Load Balancer routes each request independently to a registered target based on the chosen load\-balancing algorithm\. However, you can use the sticky session feature \(also known as session affinity\) to enable the load balancer to bind a user's session to a specific target\. This ensures that all requests from the user during the session are sent to the same target\. This feature is useful for servers that maintain state information in order to provide a continuous experience to clients\. To use sticky sessions, the client must support cookies\.

Application Load Balancers support both duration\-based cookies and application\-based cookies\. The key to managing sticky sessions is determining how long your load balancer should consistently route the user's request to the same target\. Sticky sessions are enabled at the target group level\. You can use a combination of duration\-based stickiness, application\-based stickiness, and no stickiness across all of your target groups\. 

The content of load balancer generated cookies are encrypted using a rotating key\. You cannot decrypt or modify load balancer generated cookies\. 

For both stickiness types, the Application Load Balancer resets the expiry of the cookies it generates after every request\. If a cookie expires, the session is no longer sticky and the client should remove the cookie from its cookie store\. 

**Requirements**
+ An HTTP/HTTPS load balancer\.
+ At least one healthy instance in each Availability Zone\.

**Considerations**
+ For application\-based cookies, cookie names have to be specified individually for each target group\. However, for duration\-based cookies, `AWSALB` is the only name used across all target groups\.
+ If you are using multiple layers of Application Load Balancers, you can enable sticky sessions across all layers with application\-based cookies\. However, with duration\-based cookies, you can enable sticky sessions only on one layer, because `AWSALB` is the only name available\.
+ Application\-based stickiness does not work with weighted target groups\.
+ If you have a [forward action](load-balancer-listeners.md#forward-actions) with multiple target groups, and sticky sessions are enabled for one or more of the target groups, you must enable stickiness at the target group level\.
+ WebSocket connections are inherently sticky\. If the client requests a connection upgrade to WebSockets, the target that returns an HTTP 101 status code to accept the connection upgrade is the target used in the WebSockets connection\. After the WebSockets upgrade is complete, cookie\-based stickiness is not used\.
+ Application Load Balancers use the `Expires` attribute in the cookie header instead of the `Max-Age` attribute\.
+ Application Load Balancers do not support cookie values that are URL encoded\.

## Duration\-based stickiness<a name="duration-based-stickiness"></a>

Duration\-based stickiness routes requests to the same target in a target group using a load balancer generated cookie \(`AWSALB`\)\. The cookie is used to map the session to the target\. If your application does not have its own session cookie, you can specify your own stickiness duration and manage how long your load balancer should consistently route the user's request to the same target\. 

When a load balancer first receives a request from a client, it routes the request to a target \(based on the chosen algorithm\), and generates a cookie named `AWSALB`\. It encodes information about the selected target, encrypts the cookie, and includes the cookie in the response to the client\. The load balancer generated cookie has its own expiry of 7 days which is non\-configurable\.

In subsequent requests, the client should include the `AWSALB` cookie\. When the load balancer receives a request from a client that contains the cookie, it detects it and routes the request to the same target\. If the cookie is present but cannot be decoded, or if it refers to a target that was deregistered or is unhealthy, the load balancer selects a new target and updates the cookie with information about the new target\.

With cross\-origin resource sharing \(CORS\) requests, some browsers require `SameSite=None; Secure` to enable stickiness\. In this case, the load balancer generates a second stickiness cookie, `AWSALBCORS`, which includes the same information as the original stickiness cookie plus the `SameSite` attribute\. Clients receive both cookies\.

**To enable duration\-based stickiness using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Load Balancing**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** tab, in the **Attributes** section, choose **Edit**\.

1. On the **Edit attributes** page, do the following:

   1. Select **Stickiness**\.

   1. For **Stickiness type**, select **Load balancer generated cookie**\.

   1. For **Stickiness duration**, specify a value between 1 second and 7 days\.

   1. Choose **Save changes**\.

**To enable duration\-based stickiness using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command with the `stickiness.enabled` and `stickiness.lb_cookie.duration_seconds` attributes\.

Use the following command to enable duration\-based stickiness\.

```
aws elbv2 modify-target-group-attributes --target-group-arn ARN --attributes Key=stickiness.enabled,Value=true Key=stickiness.lb_cookie.duration_seconds,Value=time-in-seconds
```

Your output should be similar to the following example\.

```
     {     
        "Attributes": [
           ...
            {
                 "Key": "stickiness.enabled",
                 "Value": "true"
              },           
              {
                  "Key": "stickiness.lb_cookie.duration_seconds",
                  "Value": "86500"
              
              },
           ...
          ]
      }
```

## Application\-based stickiness<a name="application-based-stickiness"></a>

Application\-based stickiness gives you the flexibility to set your own criteria for client\-target stickiness\. When you enable application\-based stickiness, the load balancer routes the first request to a target within the target group based on the chosen algorithm\. The target is expected to set a custom application cookie that matches the cookie configured on the load balancer to enable stickiness\. This custom cookie can include any of the cookie attributes required by the application\.

When the Application Load Balancer receives the custom application cookie from the target, it automatically generates a new encrypted application cookie to capture stickiness information\. This load balancer generated application cookie captures stickiness information for each target group that has application\-based stickiness enabled\. 

The load balancer generated application cookie does not copy the attributes of the custom cookie set by the target\. It has its own expiry of 7 days which is non\-configurable\. In the response to the client, the Application Load Balancer only validates the name with which the custom cookie was configured at the target group level and not the value or the expiry attribute of the custom cookie\. As long as the name matches, the load balancer sends both cookies, the custom cookie set by the target, and the application cookie generated by the load balancer, in the response to the client\. 

In subsequent requests, clients have to send back both cookies to maintain stickiness\. The load balancer decrypts the application cookie, and checks whether the configured duration of stickiness is still valid\. It then uses the information in the cookie to send the request to the same target within the target group to maintain stickiness\. The load balancer also proxies the custom application cookie to the target without inspecting or modifying it\. In subsequent responses, the expiry of the load balancer generated application cookie and the duration of stickiness configured on the load balancer are reset\. To maintain stickiness between client and target, the expiry of the cookie, and the duration of stickiness should not elapse\.

If a target fails or becomes unhealthy, the load balancer stops routing requests to that target, and chooses a new healthy target based on the chosen load balancing algorithm\. The load balancer treats the session as now being "stuck" to the new healthy target, and continues routing requests to the new healthy target even if the failed target comes back\.

With cross\-origin resource sharing \(CORS\) requests, to enable stickiness, the load balancer adds the `SameSite=None; Secure` attributes to the load balancer generated application cookie only if the user\-agent version is Chromium80 or above\.

Since most browsers limit cookies to 4K in size, the load balancer shards application cookies greater than 4K into multiple cookies\. Application Load Balancers support cookies up to 16K in size and can therefore create up to 4 shards that it sends to the client\. The application cookie name that the client sees begins with “AWSALBAPP\-" and includes a fragment number\. For example, if the cookie size is 0\-4K, the client sees AWSALBAPP\-0\. If the cookie size is 4\-8k, the client sees AWSALBAPP\-0 and AWSALBAPP\-1, and so on\.

**To enable application\-based stickiness using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Load Balancing**, choose **Target Groups**\.

1. Choose the name of the target group to open its details page\.

1. On the **Group details** tab, in the **Attributes** section, choose **Edit**\.

1. On the **Edit attributes** page, do the following:

   1. Select **Stickiness**\.

   1. For **Stickiness type**, select **Application\-based cookie**\.

   1. For **Stickiness duration**, specify a value between 1 second and 7 days\.

   1. For **App cookie name**, enter a name for your application\-based cookie\.

      Do not use `AWSALB`, `AWSALBAPP`, or `AWSALBTG` for the cookie name; they're reserved for use by the load balancer\.

   1. Choose **Save changes**\.

**To enable application\-based stickiness using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command with the following attributes:
+ `stickiness.enabled`
+ `stickiness.type`
+ `stickiness.app_cookie.cookie_name`
+ `stickiness.app_cookie.duration_seconds`

Use the following command to enable application\-based stickiness\.

```
aws elbv2 modify-target-group-attributes --target-group-arn ARN --attributes Key=stickiness.enabled,Value=true Key=stickiness.type,Value=app_cookie Key=stickiness.app_cookie.cookie_name Value=my-cookie-name Key=stickiness.app_cookie.duration_seconds Value=time-in-seconds
```

Your output should be similar to the following example\.

```
 
       {
            "Attributes": [
                ...
                {
                    "Key": "stickiness.enabled",
                    "Value": "true"
                },
                {
                    "Key": "stickiness.app_cookie.cookie_name",
                    "Value": "MyCookie"
                },
                {
                    "Key": "stickiness.type",
                    "Value": "app_cookie"
                },
                {
                    "Key": "stickiness.app_cookie.duration_seconds",
                    "Value": "86500"
                },
                ...
            ]
        }
```

**Manual rebalancing**  
When scaling up, if the number of targets increase considerably, there is potential for unequal distribution of load due to stickiness\. In this scenario, you can rebalance the load on your targets using the following two options:
+ Set an expiry on the cookie generated by the application that is prior to the current date and time\. This will prevent clients from sending the cookie to the Application Load Balancer, which will restart the process of establishing stickiness\.
+ Set a very short duration on the load balancer’s application\-based stickiness configuration, for example, 1 second\. This forces the Application Load Balancer to reestablish stickiness even if the cookie set by the target has not expired\.