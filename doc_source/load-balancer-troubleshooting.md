# Troubleshoot your Application Load Balancers<a name="load-balancer-troubleshooting"></a>

The following information can help you troubleshoot issues with your Application Load Balancer\.

**Topics**
+ [A registered target is not in service](#target-not-inservice)
+ [Clients cannot connect to an internet\-facing load balancer](#client-cannot-connect)
+ [Load balancer shows elevated processing times](#elevated-processing-time)
+ [The load balancer sends a response code of 000](#response-code-000)
+ [The load balancer generates an HTTP error](#load-balancer-http-error-codes)
+ [A target generates an HTTP error](#target-http-errors)
+ [Multi\-Line headers are not supported](#multi-line-headers-issue)

## A registered target is not in service<a name="target-not-inservice"></a>

If a target is taking longer than expected to enter the `InService` state, it might be failing health checks\. Your target is not in service until it passes one health check\. For more information, see [Health checks for your target groups](target-group-health-checks.md)\.

Verify that your instance is failing health checks and then check for the following issues:

**A security group does not allow traffic**  
The security group associated with an instance must allow traffic from the load balancer using the health check port and health check protocol\. You can add a rule to the instance security group to allow all traffic from the load balancer security group\. Also, the security group for your load balancer must allow traffic to the instances\.

**A network access control list \(ACL\) does not allow traffic**  
The network ACL associated with the subnets for your instances must allow inbound traffic on the health check port and outbound traffic on the ephemeral ports \(1024\-65535\)\. The network ACL associated with the subnets for your load balancer nodes must allow inbound traffic on the ephemeral ports and outbound traffic on the health check and ephemeral ports\.

**The ping path does not exist**  
Create a target page for the health check and specify its path as the ping path\.

**The connection times out**  
First, verify that you can connect to the target directly from within the network using the private IP address of the target and the health check protocol\. If you can't connect, check whether the instance is over\-utilized, and add more targets to your target group if it is too busy to respond\. If you can connect, it is possible that the target page is not responding before the health check timeout period\. Choose a simpler target page for the health check or adjust the health check settings\.

**The target did not return a successful response code**  
By default, the success code is 200, but you can optionally specify additional success codes when you configure health checks\. Confirm the success codes that the load balancer is expecting and that your application is configured to return these codes on success\.

**The target response code was malformed or there was an error connecting to the target**  
Verify that your application responds to the load balancer's health check requests\. Some applications require additional configuration to respond to health checks, such as a virtual host configuration to respond to the HTTP host header sent by the load balancer\. The host header value contains the private IP address of the target, followed by the health check port\. For example, if your target’s private IP address is `10.0.0.10` and health check port is `8080`, the HTTP Host header sent by the load balancer in health checks is `Host: 10.0.0.10:8080`\. A virtual host configuration to respond to that host, or a default configuration, may be required to successfully health check your application\. Health check requests have the following attributes: the `User-Agent` is set to `ELB-HealthChecker/2.0`, the line terminator for message\-header fields is the sequence CRLF, and the header terminates at the first empty line followed by a CRLF\.

## Clients cannot connect to an internet\-facing load balancer<a name="client-cannot-connect"></a>

If the load balancer is not responding to requests, check for the following issues:

**Your internet\-facing load balancer is attached to a private subnet**  
You must specify public subnets for your load balancer\. A public subnet has a route to the Internet Gateway for your virtual private cloud \(VPC\)\.

**A security group or network ACL does not allow traffic**  
The security group for the load balancer and any network ACLs for the load balancer subnets must allow inbound traffic from the clients and outbound traffic to the clients on the listener ports\.

## Load balancer shows elevated processing times<a name="elevated-processing-time"></a>

The load balancer counts processing times differently based on configuration\.
+ If AWS WAF is associated with your Application Load Balancer and a client sends an HTTP POST request, the time to send the data for POST requests is reflected in the `request_processing_time` field in the load balancer access logs\. This behavior is expected for HTTP POST requests\.
+ If AWS WAF is not associated with your Application Load Balancer and a client sends an HTTP POST request, the time to send the data for POST requests is reflected in the `target_processing_time` field in the load balancer access logs\. This behavior is expected for HTTP POST requests\.

## The load balancer sends a response code of 000<a name="response-code-000"></a>

With HTTP/2 connections, if the compressed length of any of the headers exceeds 8 K bytes or if the number of requests served through one connection exceeds 10,000, the load balancer sends a GOAWAY frame and closes the connection with a TCP FIN\.

## The load balancer generates an HTTP error<a name="load-balancer-http-error-codes"></a>

The following HTTP errors are generated by the load balancer\. The load balancer sends the HTTP code to the client, saves the request to the access log, and increments the `HTTPCode_ELB_4XX_Count` or `HTTPCode_ELB_5XX_Count` metric\.

**Topics**
+ [HTTP 400: Bad request](#http-400-issues)
+ [HTTP 401: Unauthorized](#http-401-issues)
+ [HTTP 403: Forbidden](#http-403-issues)
+ [HTTP 405: Method not allowed](#http-405-issues)
+ [HTTP 408: Request timeout](#http-408-issues)
+ [HTTP 413: Payload too large](#http-413-issues)
+ [HTTP 414: URI too long](#http-414-issues)
+ [HTTP 460](#http-460-issues)
+ [HTTP 463](#http-463-issues)
+ [HTTP 464](#http-464-issues)
+ [HTTP 500: Internal server error](#http-500-issues)
+ [HTTP 501: Not implemented](#http-501-issues)
+ [HTTP 502: Bad gateway](#http-502-issues)
+ [HTTP 503: Service unavailable](#http-503-issues)
+ [HTTP 504: Gateway timeout](#http-504-issues)
+ [HTTP 505: Version not supported](#http-505-issues)
+ [HTTP 561: Unauthorized](#http-561-issues)

### HTTP 400: Bad request<a name="http-400-issues"></a>

Possible causes:
+ The client sent a malformed request that does not meet the HTTP specification\.
+ The request header exceeded 16 K per request line, 16 K per single header, or 64 K for the entire request header\.
+ The client closed the connection before sending the full request body\.

### HTTP 401: Unauthorized<a name="http-401-issues"></a>

You configured a listener rule to authenticate users, but one of the following is true:
+ You configured `OnUnauthenticatedRequest` to deny unauthenticated users or the IdP denied access\.
+ The size of the claims returned by the IdP exceeded the maximum size supported by the load balancer\.
+ A client submitted an HTTP/1\.0 request without a host header, and the load balancer was unable to generate a redirect URL\.
+ The requested scope doesn't return an ID token\.
+ You don't complete the login process before the client login timeout expires\. For more information see, [Client login timeout](listener-authenticate-users.md#client-login-timeout)\.

### HTTP 403: Forbidden<a name="http-403-issues"></a>

You configured an AWS WAF web access control list \(web ACL\) to monitor requests to your Application Load Balancer and it blocked a request\.

### HTTP 405: Method not allowed<a name="http-405-issues"></a>

The client used the TRACE method, which is not supported by Application Load Balancers\.

### HTTP 408: Request timeout<a name="http-408-issues"></a>

The client did not send data before the idle timeout period expired\. Sending a TCP keep\-alive does not prevent this timeout\. Send at least 1 byte of data before each idle timeout period elapses\. Increase the length of the idle timeout period as needed\.

### HTTP 413: Payload too large<a name="http-413-issues"></a>

The target is a Lambda function and the request body exceeds 1 MB\.

### HTTP 414: URI too long<a name="http-414-issues"></a>

The request URL or query string parameters are too large\.

### HTTP 460<a name="http-460-issues"></a>

The load balancer received a request from a client, but the client closed the connection with the load balancer before the idle timeout period elapsed\.

Check whether the client timeout period is greater than the idle timeout period for the load balancer\. Ensure that your target provides a response to the client before the client timeout period elapses, or increase the client timeout period to match the load balancer idle timeout, if the client supports this\.

### HTTP 463<a name="http-463-issues"></a>

The load balancer received an **X\-Forwarded\-For** request header with too many IP addresses\. The upper limit for IP addresses is 30\.

### HTTP 464<a name="http-464-issues"></a>

The load balancer received an incoming request protocol that is incompatible with the version config of the target group protocol\.

Possible causes:
+  The request protocol is an HTTP/1\.1, while the target group protocol version is a gRPC or HTTP/2\. 
+  The request protocol is a gRPC, while the target group protocol version is an HTTP/1\.1\. 
+  The request protocol is an HTTP/2 and the request is not POST, while target group protocol version is a gRPC\. 

### HTTP 500: Internal server error<a name="http-500-issues"></a>

Possible causes:
+ You configured an AWS WAF web access control list \(web ACL\) and there was an error executing the web ACL rules\.
+ The load balancer is unable to communicate with the IdP token endpoint or the IdP user info endpoint\. 
  + Verify that the IdP's DNS is publicly resolvable\.
  + Verify that the security groups for your load balancer and the network ACLs for your VPC allow outbound access to these endpoints\. 
  + Verify that your VPC has internet access\. If you have an internal\-facing load balancer, use a NAT gateway to enable internet access\.

### HTTP 501: Not implemented<a name="http-501-issues"></a>

The load balancer received a **Transfer\-Encoding** header with an unsupported value\. The supported values for **Transfer\-Encoding** are `chunked` and `identity`\. As an alternative, you can use the **Content\-Encoding** header\.

### HTTP 502: Bad gateway<a name="http-502-issues"></a>

Possible causes:
+ The load balancer received a TCP RST from the target when attempting to establish a connection\.
+ The load balancer received an unexpected response from the target, such as "ICMP Destination unreachable \(Host unreachable\)", when attempting to establish a connection\. Check whether traffic is allowed from the load balancer subnets to the targets on the target port\.
+ The target closed the connection with a TCP RST or a TCP FIN while the load balancer had an outstanding request to the target\. Check whether the keep\-alive duration of the target is shorter than the idle timeout value of the load balancer\.
+ The target response is malformed or contains HTTP headers that are not valid\.
+ The target response header exceeded 32 K for the entire response header\.
+ The load balancer encountered an SSL handshake error or SSL handshake timeout \(10 seconds\) when connecting to a target\.
+ The deregistration delay period elapsed for a request being handled by a target that was deregistered\. Increase the delay period so that lengthy operations can complete\.
+ The target is a Lambda function and the response body exceeds 1 MB\.
+ The target is a Lambda function that did not respond before its configured timeout was reached\.
+ The target is a Lambda function that returned an error or the function was throttled by the Lambda service\.

For more information see [How do I troubleshoot Application Load Balancer HTTP 502 errors](http://aws.amazon.com/premiumsupport/knowledge-center/elb-alb-troubleshoot-502-errors/) in the AWS Support Knowledge Center\.

### HTTP 503: Service unavailable<a name="http-503-issues"></a>

The target groups for the load balancer have no registered targets\.

### HTTP 504: Gateway timeout<a name="http-504-issues"></a>

Possible causes:
+ The load balancer failed to establish a connection to the target before the connection timeout expired \(10 seconds\)\.
+ The load balancer established a connection to the target but the target did not respond before the idle timeout period elapsed\.
+ The network ACL for the subnet did not allow traffic from the targets to the load balancer nodes on the ephemeral ports \(1024\-65535\)\.
+ The target returns a content\-length header that is larger than the entity body\. The load balancer timed out waiting for the missing bytes\.
+ The target is a Lambda function and the Lambda service did not respond before the connection timeout expired\.

### HTTP 505: Version not supported<a name="http-505-issues"></a>

The load balancer received an unexpected HTTP version request\. For example, the load balancer established an HTTP/1 connection but received an HTTP/2 request\.

### HTTP 561: Unauthorized<a name="http-561-issues"></a>

You configured a listener rule to authenticate users, but the IdP returned an error code when authenticating the user\. Check your access logs for the related [error reason code](load-balancer-access-logs.md#error-reason-codes)\.

## A target generates an HTTP error<a name="target-http-errors"></a>

The load balancer forwards valid HTTP responses from targets to the client, including HTTP errors\. The HTTP errors generated by a target are recorded in the `HTTPCode_Target_4XX_Count` and `HTTPCode_Target_5XX_Count` metrics\.

## Multi\-Line headers are not supported<a name="multi-line-headers-issue"></a>

Application Load Balancers do not support multi\-line headers, including the `message/http` media type header\. When a multi\-line header is provided the Application Load Balancer appends a colon character, "`:`", before passing it to the target\.