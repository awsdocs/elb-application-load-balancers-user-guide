# HTTP headers and Application Load Balancers<a name="x-forwarded-headers"></a>

HTTP requests and HTTP responses use header fields to send information about the HTTP messages\. HTTP headers are added automatically\. Header fields are colon\-separated name\-value pairs that are separated by a carriage return \(CR\) and a line feed \(LF\)\. A standard set of HTTP header fields is defined in RFC 2616, [Message Headers](https://datatracker.ietf.org/doc/html/rfc2616)\. There are also non\-standard HTTP headers available that are automatically added and widely used by the applications\. Some of the non\-standard HTTP headers have an `X-Forwarded` prefix\. Application Load Balancers support the following `X-Forwarded` headers\.

For more information about HTTP connections, see [Request routing](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html#request-routing) in the *Elastic Load Balancing User Guide*\.

**Topics**
+ [X\-Forwarded\-For](#x-forwarded-for)
+ [X\-Forwarded\-Proto](#x-forwarded-proto)
+ [X\-Forwarded\-Port](#x-forwarded-port)

## X\-Forwarded\-For<a name="x-forwarded-for"></a>

The `X-Forwarded-For` request header helps you identify the IP address of a client when you use an HTTP or HTTPS load balancer\. Because load balancers intercept traffic between clients and servers, your server access logs only contain the IP address of the load balancer\. To see the IP address of the client, use the `routing.http.xff_header_processing.mode` attribute\. This attribute enables you to modify, preserve, or remove the `X-Forwarded-For` header in the HTTP request before the Application Load Balancer sends the request to the target\. The possible values for this attribute are `append`, `preserve`, and `remove`\. The default value for this attribute is `append`\. 

### Append<a name="x-forwarded-for-append"></a>

By default, the Application Load Balancer stores the IP address of the client in the `X-Forwarded-For` request header and passes the header to your server\. If the `X-Forwarded-For` request header is not included in the original request, the load balancer creates one with the client IP address as the request value\. Otherwise, the load balancer adds the client IP address to the existing header and then passes the header to your server\. The `X-Forwarded-For` request header may contain multiple IP addresses that are comma separated\. The left\-most address is the client IP address where the request was first made\. This is followed by any subsequent proxy identifiers in a chain\. 

The `X-Forwarded-For` request header takes the following form:

```
X-Forwarded-For: client-ip-address
```

The following is an example `X-Forwarded-For` request header for a client with an IP address of `203.0.113.7`\.

```
X-Forwarded-For: 203.0.113.7
```

The following is an example `X-Forwarded-For` request header for a client with an IPv6 address of `2001:DB8::21f:5bff:febf:ce22:8a2e`\.

```
X-Forwarded-For: 2001:DB8::21f:5bff:febf:ce22:8a2e
```

When the client port preservation attribute \(`routing.http.xff_client_port.enabled`\) is enabled on the load balancer, the `X-Forwarded-For` request header includes the `client-port-number` appended to the `client-ip-address`, separated by a colon\. The header then takes the following form:

```
IPv4 -- X-Forwarded-For: client-ip-address:client-port-number
```

```
IPv6 -- X-Forwarded-For: [client-ip-address]:client-port-number
```

For IPv6, note that when the load balancer appends the `client-ip-address` to the existing header, it encloses the address in square brackets\.

The following is an example `X-Forwarded-For` request header for a client with an IPv4 address of `12.34.56.78` and a port number of `8080`\.

```
X-Forwarded-For: 12.34.56.78:8080
```

The following is an example `X-Forwarded-For` request header for a client with an IPv6 address of `2001:db8:85a3:8d3:1319:8a2e:370:7348` and a port number of `8080`\.

```
X-Forwarded-For: [2001:db8:85a3:8d3:1319:8a2e:370:7348]:8080
```

### Preserve<a name="x-forwarded-for-preserve"></a>

The `preserve` mode in the attribute ensures that the `X-Forwarded-For` header in the HTTP request is not modified in any way before it is sent to targets\.

### Remove<a name="x-forwarded-for-remove"></a>

The `remove` mode in the attribute removes the `X-Forwarded-For` header in the HTTP request before it is sent to targets\.

**Note**  
If you enable the client port preservation attribute \(`routing.http.xff_client_port.enabled`\), and also select `preserve` or `remove` for the `routing.http.xff_header_processing.mode` attribute, the Application Load Balancer overrides the client port preservation attribute\. It keeps the `X-Forwarded-For` header unchanged, or removes it depending on the mode you select, before it sends it to the targets\.

The following table shows examples of the `X-Forwarded-For` header that the target receives when you select either the `append`, `preserve` or the `remove` mode\. In this example, the IP address of the last hop is `127.0.0.1`\. 


|  Request description  |  Example request  | XFF in `append` mode  | XFF in `preserve` mode | XFF in `remove` mode | 
| --- | --- | --- | --- | --- | 
| Request is sent with no XFF header | GET /index\.html HTTP/1\.1 Host: example\.com | X\-Forwarded\-For: 127\.0\.0\.1 | Not present | Not present | 
| Request is sent with an XFF header and a client IP address\. | GET /index\.html HTTP/1\.1 Host: example\.com X\-Forwarded\-For: 127\.0\.0\.4 | X\-Forwarded\-For: 127\.0\.0\.4, 127\.0\.0\.1 | X\-Forwarded\-For: 127\.0\.0\.4 | Not present | 
| Request is sent with an XFF header with multiple client IP addresses\. | GET /index\.html HTTP/1\.1 Host: example\.com X\-Forwarded\-For: 127\.0\.0\.4, 127\.0\.0\.8 | X\-Forwarded\-For: 127\.0\.0\.4, 127\.0\.0\.8, 127\.0\.0\.1 | X\-Forwarded\-For: 127\.0\.0\.4, 127\.0\.0\.8 | Not present | 

**To modify, preserve, or remove the X\-Forwarded\-For header using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Attributes** tab, choose **Edit**\.

1. For **X\-Forward\-For header**, choose **Append** \(default\), **Preserve**, or **Remove**\.

1. Choose **Save changes**\.

**To modify, preserve, or remove the X\-Forwarded\-For header using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command with the `routing.http.xff_header_processing.mode` attribute\.

## X\-Forwarded\-Proto<a name="x-forwarded-proto"></a>

The `X-Forwarded-Proto` request header helps you identify the protocol \(HTTP or HTTPS\) that a client used to connect to your load balancer\. Your server access logs contain only the protocol used between the server and the load balancer; they contain no information about the protocol used between the client and the load balancer\. To determine the protocol used between the client and the load balancer, use the `X-Forwarded-Proto` request header\. Elastic Load Balancing stores the protocol used between the client and the load balancer in the `X-Forwarded-Proto` request header and passes the header along to your server\.

Your application or website can use the protocol stored in the `X-Forwarded-Proto` request header to render a response that redirects to the appropriate URL\.

The `X-Forwarded-Proto` request header takes the following form:

```
X-Forwarded-Proto: originatingProtocol
```

The following example contains an `X-Forwarded-Proto` request header for a request that originated from the client as an HTTPS request:

```
X-Forwarded-Proto: https
```

## X\-Forwarded\-Port<a name="x-forwarded-port"></a>

The `X-Forwarded-Port` request header helps you identify the destination port that the client used to connect to the load balancer\.