# Access logs for your Application Load Balancer<a name="load-balancer-access-logs"></a>

Elastic Load Balancing provides access logs that capture detailed information about requests sent to your load balancer\. Each log contains information such as the time the request was received, the client's IP address, latencies, request paths, and server responses\. You can use these access logs to analyze traffic patterns and troubleshoot issues\.

Access logging is an optional feature of Elastic Load Balancing that is disabled by default\. After you enable access logging for your load balancer, Elastic Load Balancing captures the logs and stores them in the Amazon S3 bucket that you specify as compressed files\. You can disable access logging at any time\.

Each access log file is automatically encrypted using SSE\-S3 before it is stored in your S3 bucket and decrypted when you access it\. You do not need to take any action; the encryption and decryption is performed transparently\. Each log file is encrypted with a unique key, which is itself encrypted with a master key that is regularly rotated\. For more information, see [Protecting data using server\-side encryption with Amazon S3\-managed encryption keys \(SSE\-S3\)](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html) in the *Amazon Simple Storage Service Developer Guide*\.

There is no additional charge for access logs\. You are charged storage costs for Amazon S3, but not charged for the bandwidth used by Elastic Load Balancing to send log files to Amazon S3\. For more information about storage costs, see [Amazon S3 pricing](https://aws.amazon.com/s3/pricing/)\.

**Topics**
+ [Access log files](#access-log-file-format)
+ [Access log entries](#access-log-entry-format)
+ [Bucket permissions](#access-logging-bucket-permissions)
+ [Enable access logging](#enable-access-logging)
+ [Disable access logging](#disable-access-logging)
+ [Processing access log files](#log-processing-tools)

## Access log files<a name="access-log-file-format"></a>

Elastic Load Balancing publishes a log file for each load balancer node every 5 minutes\. Log delivery is eventually consistent\. The load balancer can deliver multiple logs for the same period\. This usually happens if the site has high traffic\.

The file names of the access logs use the following format:

```
bucket[/prefix]/AWSLogs/aws-account-id/elasticloadbalancing/region/yyyy/mm/dd/aws-account-id_elasticloadbalancing_region_load-balancer-id_end-time_ip-address_random-string.log.gz
```

*bucket*  
The name of the S3 bucket\.

*prefix*  
The prefix \(logical hierarchy\) in the bucket\. If you don't specify a prefix, the logs are placed at the root level of the bucket\.

*aws\-account\-id*  
The AWS account ID of the owner\.

*region*  
The Region for your load balancer and S3 bucket\.

*yyyy*/*mm*/*dd*  
The date that the log was delivered\.

*load\-balancer\-id*  
The resource ID of the load balancer\. If the resource ID contains any forward slashes \(/\), they are replaced with periods \(\.\)\.

*end\-time*  
The date and time that the logging interval ended\. For example, an end time of 20140215T2340Z contains entries for requests made between 23:35 and 23:40\.

*ip\-address*  
The IP address of the load balancer node that handled the request\. For an internal load balancer, this is a private IP address\.

*random\-string*  
A system\-generated random string\.

The following is an example log file name:

```
s3://my-bucket/prefix/AWSLogs/123456789012/elasticloadbalancing/us-east-2/2016/05/01/123456789012_elasticloadbalancing_us-east-2_app.my-loadbalancer.1234567890abcdef_20140215T2340Z_172.160.001.192_20sg8hgm.log.gz
```

You can store your log files in your bucket for as long as you want, but you can also define Amazon S3 lifecycle rules to archive or delete log files automatically\. For more information, see [Object lifecycle management](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-lifecycle-mgmt.html) in the *Amazon Simple Storage Service Developer Guide*\.

## Access log entries<a name="access-log-entry-format"></a>

Elastic Load Balancing logs requests sent to the load balancer, including requests that never made it to the targets\. For example, if a client sends a malformed request, or there are no healthy targets to respond to the request, the request is still logged\. Note that Elastic Load Balancing does not log health check requests\.

Each log entry contains the details of a single request \(or connection in the case of WebSockets\) made to the load balancer\. For WebSockets, an entry is written only after the connection is closed\. If the upgraded connection can't be established, the entry is the same as for an HTTP or HTTPS request\.

**Important**  
Elastic Load Balancing logs requests on a best\-effort basis\. We recommend that you use access logs to understand the nature of the requests, not as a complete accounting of all requests\.

**Topics**
+ [Syntax](#access-log-entry-syntax)
+ [Actions taken](#actions-taken)
+ [Classification reasons](#classification-reasons)
+ [Error reason codes](#error-reason-codes)
+ [Examples](#access-log-entry-examples)

### Syntax<a name="access-log-entry-syntax"></a>

The following table describes the fields of an access log entry, in order\. All fields are delimited by spaces\. When new fields are introduced, they are added to the end of the log entry\. You should ignore any fields at the end of the log entry that you were not expecting\.


| Field | Description | 
| --- | --- | 
| type | The type of request or connection\. The possible values are as follows \(ignore any other values\): [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html)  | 
| time | The time when the load balancer generated a response to the client, in ISO 8601 format\. For WebSockets, this is the time when the connection is closed\. | 
| elb | The resource ID of the load balancer\. If you are parsing access log entries, note that resources IDs can contain forward slashes \(/\)\. | 
| client:port | The IP address and port of the requesting client\. | 
| target:port |  The IP address and port of the target that processed this request\. If the client didn't send a full request, the load balancer can't dispatch the request to a target, and this value is set to \-\. If the target is a Lambda function, this value is set to \-\. If the request is blocked by AWS WAF, this value is set to \- and the value of elb\_status\_code is set to 403\.  | 
| request\_processing\_time |  The total time elapsed \(in seconds, with millisecond precision\) from the time the load balancer received the request until the time it sent the request to a target\. This value is set to \-1 if the load balancer can't dispatch the request to a target\. This can happen if the target closes the connection before the idle timeout or if the client sends a malformed request\. This value can also be set to \-1 if the registered target does not respond before the idle timeout\.  | 
| target\_processing\_time |  The total time elapsed \(in seconds, with millisecond precision\) from the time the load balancer sent the request to a target until the target started to send the response headers\. This value is set to \-1 if the load balancer can't dispatch the request to a target\. This can happen if the target closes the connection before the idle timeout or if the client sends a malformed request\. This value can also be set to \-1 if the registered target does not respond before the idle timeout\.  | 
| response\_processing\_time |  The total time elapsed \(in seconds, with millisecond precision\) from the time the load balancer received the response header from the target until it started to send the response to the client\. This includes both the queuing time at the load balancer and the connection acquisition time from the load balancer to the client\. This value is set to \-1 if the load balancer can't send the request to a target\. This can happen if the target closes the connection before the idle timeout or if the client sends a malformed request\.  | 
| elb\_status\_code | The status code of the response from the load balancer\. | 
| target\_status\_code | The status code of the response from the target\. This value is recorded only if a connection was established to the target and the target sent a response\. Otherwise, it is set to \-\. | 
| received\_bytes |  The size of the request, in bytes, received from the client \(requester\)\. For HTTP requests, this includes the headers\. For WebSockets, this is the total number of bytes received from the client on the connection\.  | 
| sent\_bytes |  The size of the response, in bytes, sent to the client \(requester\)\. For HTTP requests, this includes the headers\. For WebSockets, this is the total number of bytes sent to the client on the connection\.  | 
| "request" |  The request line from the client, enclosed in double quotes and logged using the following format: HTTP method \+ protocol://host:port/uri \+ HTTP version\. The load balancer preserves the URL sent by the client, as is, when recording the request URI\. It does not set the content type for the access log file\. When you process this field, consider how the client sent the URL\.  | 
| "user\_agent" |  A User\-Agent string that identifies the client that originated the request, enclosed in double quotes\. The string consists of one or more product identifiers, product\[/version\]\. If the string is longer than 8 KB, it is truncated\.  | 
| ssl\_cipher |  \[HTTPS listener\] The SSL cipher\. This value is set to \- if the listener is not an HTTPS listener\.  | 
| ssl\_protocol |  \[HTTPS listener\] The SSL protocol\. This value is set to \- if the listener is not an HTTPS listener\.  | 
| target\_group\_arn |  The Amazon Resource Name \(ARN\) of the target group\.  | 
| "trace\_id" |  The contents of the **X\-Amzn\-Trace\-Id** header, enclosed in double quotes\.  | 
| "domain\_name" |  \[HTTPS listener\] The SNI domain provided by the client during the TLS handshake, enclosed in double quotes\. This value is set to \- if the client doesn't support SNI or the domain doesn't match a certificate and the default certificate is presented to the client\.  | 
| "chosen\_cert\_arn" |  \[HTTPS listener\] The ARN of the certificate presented to the client, enclosed in double quotes\. This value is set to `session-reused` if the session is reused\. This value is set to \- if the listener is not an HTTPS listener\.  | 
| matched\_rule\_priority |  The priority value of the rule that matched the request\. If a rule matched, this is a value from 1 to 50,000\. If no rule matched and the default action was taken, this value is set to 0\. If an error occurs during rules evaluation, it is set to \-1\. For any other error, it is set to \-\.  | 
| request\_creation\_time |  The time when the load balancer received the request from the client, in ISO 8601 format\.  | 
| "actions\_executed" |  The actions taken when processing the request, enclosed in double quotes\. This value is a comma\-separated list that can include the values described in [Actions taken](#actions-taken)\. If no action was taken, such as for a malformed request, this value is set to \-\.  | 
| "redirect\_url" |  The URL of the redirect target for the location header of the HTTP response, enclosed in double quotes\. If no redirect actions were taken, this value is set to \-\.  | 
| "error\_reason" |  The error reason code, enclosed in double quotes\. If the request failed, this is one of the error codes described in [Error reason codes](#error-reason-codes)\. If the actions taken do not include an authenticate action or the target is not a Lambda function, this value is set to \-\.  | 
| "target:port\_list" |  A space\-delimited list of IP addresses and ports for the targets that processed this request, enclosed in double quotes\. Currently, this list can contain one item and it matches the target:port field\. If the client didn't send a full request, the load balancer can't dispatch the request to a target, and this value is set to \-\. If the target is a Lambda function, this value is set to \-\. If the request is blocked by AWS WAF, this value is set to \- and the value of elb\_status\_code is set to 403\.  | 
| "target\_status\_code\_list" |  A space\-delimited list of status codes from the responses of the targets, enclosed in double quotes\. Currently, this list can contain one item and it matches the target\_status\_code field\. This value is recorded only if a connection was established to the target and the target sent a response\. Otherwise, it is set to \-\. | 
| "classification" |  The classification for desync mitigation, enclosed in double quotes\. If the request does not comply with RFC 7230, the possible values are Acceptable, Ambiguous, and Severe\. If the request complies with RFC 7230, this value is set to \-\.  | 
| "classification\_reason" |  The classification reason code, enclosed in double quotes\. If the request does not comply with RFC 7230, this is one of the classification codes described in [Classification reasons](#classification-reasons)\. If the request complies with RFC 7230, this value is set to \-\.  | 

### Actions taken<a name="actions-taken"></a>

The load balancer stores the actions that it takes in the actions\_executed field of the access log\.
+ `authenticate` — The load balancer validated the session, authenticated the user, and added the user information to the request headers, as specified by the rule configuration\.
+ `fixed-response` — The load balancer issued a fixed response, as specified by the rule configuration\.
+ `forward` — The load balancer forwarded the request to a target, as specified by the rule configuration\.
+ `redirect` — The load balancer redirected the request to another URL, as specified by the rule configuration\.
+ `waf` — The load balancer forwarded the request to AWS WAF to determine whether the request should be forwarded to the target\. If this is the final action, AWS WAF determined that the request should be rejected\.
+ `waf-failed` — The load balancer attempted to forward the request to AWS WAF, but this process failed\.

### Classification reasons<a name="classification-reasons"></a>

If a request does not comply with RFC 7230, the load balancer stores one of the following codes in the classification\_reason field of the access log\. For more information, see [Desync mitigation mode](application-load-balancers.md#desync-mitigation-mode)\.


| Code | Description | Classification | 
| --- | --- | --- | 
| `AmbiguousUri` | The request URI contains control characters\. | Ambiguous | 
| `BadContentLength` | The Content\-Length header contains a value that cannot be parsed or is not a valid number\. | Severe | 
| `BadHeader` | A header contains a null character or carriage return\. | Severe | 
| `BadTransferEncoding` | The Transfer\-Encoding header contains a bad value\. | Severe | 
| `BadUri` | The request URI contains a null character or carriage return\. | Severe | 
| `BadMethod` | The request method is malformed\. | Severe | 
| `BadVersion` | The request version is malformed\. | Severe | 
| `BothTeClPresent` | The request contains both a Transfer\-Encoding header and a Content\-Length header\. | Ambiguous | 
| `DuplicateContentLength` | There are multiple Content\-Length headers with the same value\. | Ambiguous | 
| `EmptyHeader` | A header is empty or there is a line with only spaces\. | Ambiguous | 
| `GetHeadZeroContentLength` | There is a Content\-Length header with a value of 0 for a GET or HEAD request\. | Acceptable | 
| `MultipleContentLength` | There are multiple Content\-Length headers with different values\. | Severe | 
| `MultipleTransferEncodingChunked` | There are multiple Transfer\-Encoding: chunked headers\. | Severe | 
| `NonCompliantHeader` | A header contains a non\-ASCII or control character\. | Acceptable | 
| `NonCompliantVersion` | The request version contains a bad value\. | Acceptable | 
| `SpaceInUri` | The request URI contains a space that is not URL encoded\. | Acceptable | 
| `SuspiciousHeader` | There is a header that can be normalized to Transfer\-Encoding or Content\-Length using common text normalization techniques\. | Ambiguous | 
| `UndefinedContentLengthSemantics` | There is no Content\-Length header defined for a GET or HEAD request\. | Ambiguous | 
| `UndefinedTransferEncodingSemantics` | There is no Transfer\-Encoding header defined for GET or HEAD request\. | Ambiguous | 

### Error reason codes<a name="error-reason-codes"></a>

If the load balancer cannot complete an authenticate action, the load balancer stores one of the following reason codes in the error\_reason field of the access log\. The load balancer also increments the corresponding CloudWatch metric\. For more information, see [Authenticate users using an Application Load Balancer](listener-authenticate-users.md)\.


| Code | Description | Metric | 
| --- | --- | --- | 
| `AuthInvalidCookie` | The authentication cookie is not valid\. | `ELBAuthFailure` | 
| `AuthInvalidGrantError` | The authorization grant code from the token endpoint is not valid\. | `ELBAuthFailure` | 
| `AuthInvalidIdToken` | The ID token is not valid\. | `ELBAuthFailure` | 
| `AuthInvalidStateParam` | The state parameter is not valid\. | `ELBAuthFailure` | 
| `AuthInvalidTokenResponse` | The response from the token endpoint is not valid\. | `ELBAuthFailure` | 
| `AuthInvalidUserinfoResponse` | The response from the user info endpoint is not valid\. | `ELBAuthFailure` | 
| `AuthMissingCodeParam` | The authentication response from the authorization endpoint is missing a query parameter named 'code'\. | `ELBAuthFailure` | 
| `AuthMissingHostHeader` | The authentication response from the authorization endpoint is missing a host header field\. | `ELBAuthError` | 
| `AuthMissingStateParam` | The authentication response from the authorization endpoint is missing a query parameter named 'state'\. | `ELBAuthFailure` | 
| `AuthTokenEpRequestFailed` | There is an error response \(non\-2XX\) from the token endpoint\. | `ELBAuthError` | 
| `AuthTokenEpRequestTimeout` | The load balancer is unable to communicate with the token endpoint\. | `ELBAuthError` | 
| `AuthUnhandledException` | The load balancer encountered an unhandled exception\. | `ELBAuthError` | 
| `AuthUserinfoEpRequestFailed` | There is an error response \(non\-2XX\) from the IdP user info endpoint\. | `ELBAuthError` | 
| `AuthUserinfoEpRequestTimeout` | The load balancer is unable to communicate with the IdP user info endpoint\. | `ELBAuthError` | 
| `AuthUserinfoResponseSizeExceeded` | The size of the claims returned by the IdP exceeded 11K bytes\. | `ELBAuthUserClaimsSizeExceeded` | 

If a request to a weighted target group fails, the load balancer stores one of the following error codes in the error\_reason field of the access log\.


| Code | Description | 
| --- | --- | 
| `AWSALBTGCookieInvalid` | The AWSALBTG cookie, which is used with weighted target groups, is not valid\. For example, the load balancer returns this error when cookie values are URL encoded\. | 
| `WeightedTargetGroupsUnhandledException` | The load balancer encountered an unhandled exception\. | 

If a request to a Lambda function fails, the load balancer stores one of the following reason codes in the error\_reason field of the access log\. The load balancer also increments the corresponding CloudWatch metric\. For more information, see the Lambda [Invoke](https://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html) action\.


| Code | Description | Metric | 
| --- | --- | --- | 
| `LambdaAccessDenied` | The load balancer did not have permission to invoke the Lambda function\. | `LambdaUserError` | 
| `LambdaBadRequest` | Lambda invocation failed because the client request headers or body did not contain only UTF\-8 characters\. | `LambdaUserError` | 
| `LambdaConnectionTimeout` | An attempt to connect to Lambda timed out\. | `LambdaInternalError` | 
| `LambdaEC2AccessDeniedException` | Amazon EC2 denied access to Lambda during function initialization\. | `LambdaUserError` | 
| `LambdaEC2ThrottledException` | Amazon EC2 throttled Lambda during function initialization\. | `LambdaUserError` | 
| `LambdaEC2UnexpectedException` | Amazon EC2 encountered an unexpected exception during function initialization\. | `LambdaUserError` | 
| `LambdaENILimitReachedException` | Lambda couldn't create a network interface in the VPC specified in the configuration of the Lambda function because the limit for network interfaces was exceeded\. | `LambdaUserError` | 
| `LambdaInvalidResponse` | The response from the Lambda function is malformed or is missing required fields\. | `LambdaUserError` | 
| `LambdaInvalidRuntimeException` | The specified version of the Lambda runtime is not supported\. | `LambdaUserError` | 
| `LambdaInvalidSecurityGroupIDException` | The security group ID specified in the configuration of the Lambda function is not valid\. | `LambdaUserError` | 
| `LambdaInvalidSubnetIDException` | The subnet ID specified in the configuration of the Lambda function is not valid\. | `LambdaUserError` | 
| `LambdaInvalidZipFileException` | Lambda could not unzip the specified function zip file\. | `LambdaUserError` | 
| `LambdaKMSAccessDeniedException` | Lambda could not decrypt environment variables because access to the KMS key was denied\. Check the KMS permissions of the Lambda function\. | `LambdaUserError` | 
| `LambdaKMSDisabledException` | Lambda could not decrypt environment variables because the specified KMS key is disabled\. Check the KMS key settings of the Lambda function\. | `LambdaUserError` | 
| `LambdaKMSInvalidStateException` | Lambda could not decrypt environment variables because the state of the KMS key is not valid\. Check the KMS key settings of the Lambda function\. | `LambdaUserError` | 
| `LambdaKMSNotFoundException` | Lambda could not decrypt environment variables because the KMS key was not found\. Check the KMS key settings of the Lambda function\. | `LambdaUserError` | 
| `LambdaRequestTooLarge` | The size of the request body exceeded 1 MB\. | `LambdaUserError` | 
| `LambdaResourceNotFound` | The Lambda function could not be found\. | `LambdaUserError` | 
| `LambdaResponseTooLarge` | The size of the response exceeded 1 MB\. | `LambdaUserError` | 
| `LambdaServiceException` | Lambda encountered an internal error\. | `LambdaInternalError` | 
| `LambdaSubnetIPAddressLimitReachedException` | Lambda could not set up VPC access for the Lambda function because one or more subnets have no available IP addresses\. | `LambdaUserError` | 
| `LambdaThrottling` | The Lambda function was throttled because there were too many requests\. | `LambdaUserError` | 
| `LambdaUnhandled` | The Lambda function encountered an unhandled exception\. | `LambdaUserError` | 
| `LambdaUnhandledException` | The load balancer encountered an unhandled exception\. | `LambdaInternalError` | 

If the load balancer encounters an error when forwarding requests to AWS WAF, it stores one of the following error codes in the error\_reason field of the access log\.


| Code | Description | 
| --- | --- | 
| `WAFConnectionError` | The load balancer cannot connect to AWS WAF\. | 
| `WAFConnectionTimeout` | The connection to AWS WAF timed out\. | 
| `WAFResponseReadTimeout` | A request to AWS WAF timed out\. | 
| `WAFServiceError` | AWS WAF returned a 5XX error\. | 
| `WAFUnhandledException` | The load balancer encountered an unhandled exception\. | 

### Examples<a name="access-log-entry-examples"></a>

The following are example log entries\. Note that the text appears on multiple lines only to make them easier to read\.

**Example HTTP Entry**  
The following is an example log entry for an HTTP listener \(port 80 to port 80\):

```
http 2018-07-02T22:23:00.186641Z app/my-loadbalancer/50dc6c495c0c9188 
192.168.131.39:2817 10.0.0.1:80 0.000 0.001 0.000 200 200 34 366 
"GET http://www.example.com:80/ HTTP/1.1" "curl/7.46.0" - - 
arn:aws:elasticloadbalancing:us-east-2:123456789012:targetgroup/my-targets/73e2d6bc24d8a067
"Root=1-58337262-36d228ad5d99923122bbe354" "-" "-" 
0 2018-07-02T22:22:48.364000Z "forward" "-" "-" 10.0.0.1:80 200 "-" "-"
```

**Example HTTPS Entry**  
The following is an example log entry for an HTTPS listener \(port 443 to port 80\):

```
https 2018-07-02T22:23:00.186641Z app/my-loadbalancer/50dc6c495c0c9188 
192.168.131.39:2817 10.0.0.1:80 0.086 0.048 0.037 200 200 0 57 
"GET https://www.example.com:443/ HTTP/1.1" "curl/7.46.0" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 
arn:aws:elasticloadbalancing:us-east-2:123456789012:targetgroup/my-targets/73e2d6bc24d8a067
"Root=1-58337281-1d84f3d73c47ec4e58577259" "www.example.com" "arn:aws:acm:us-east-2:123456789012:certificate/12345678-1234-1234-1234-123456789012"
1 2018-07-02T22:22:48.364000Z "authenticate,forward" "-" "-" 10.0.0.1:80 200 "-" "-"
```

**Example HTTP/2 Entry**  
The following is an example log entry for an HTTP/2 stream\.

```
h2 2018-07-02T22:23:00.186641Z app/my-loadbalancer/50dc6c495c0c9188 
10.0.1.252:48160 10.0.0.66:9000 0.000 0.002 0.000 200 200 5 257 
"GET https://10.0.2.105:773/ HTTP/2.0" "curl/7.46.0" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2
arn:aws:elasticloadbalancing:us-east-2:123456789012:targetgroup/my-targets/73e2d6bc24d8a067
"Root=1-58337327-72bd00b0343d75b906739c42" "-" "-"
1 2018-07-02T22:22:48.364000Z "redirect" "https://example.com:80/" "-" 10.0.0.66:9000 200 "-" "-"
```

**Example WebSockets Entry**  
The following is an example log entry for a WebSockets connection\.

```
ws 2018-07-02T22:23:00.186641Z app/my-loadbalancer/50dc6c495c0c9188 
10.0.0.140:40914 10.0.1.192:8010 0.001 0.003 0.000 101 101 218 587 
"GET http://10.0.0.30:80/ HTTP/1.1" "-" - - 
arn:aws:elasticloadbalancing:us-east-2:123456789012:targetgroup/my-targets/73e2d6bc24d8a067
"Root=1-58337364-23a8c76965a2ef7629b185e3" "-" "-"
1 2018-07-02T22:22:48.364000Z "forward" "-" "-" 10.0.1.192:8010 101 "-" "-"
```

**Example Secured WebSockets Entry**  
The following is an example log entry for a secured WebSockets connection\.

```
wss 2018-07-02T22:23:00.186641Z app/my-loadbalancer/50dc6c495c0c9188 
10.0.0.140:44244 10.0.0.171:8010 0.000 0.001 0.000 101 101 218 786
"GET https://10.0.0.30:443/ HTTP/1.1" "-" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 
arn:aws:elasticloadbalancing:us-west-2:123456789012:targetgroup/my-targets/73e2d6bc24d8a067
"Root=1-58337364-23a8c76965a2ef7629b185e3" "-" "-"
1 2018-07-02T22:22:48.364000Z "forward" "-" "-" 10.0.0.171:8010 101 "-" "-"
```

**Example Entries for Lambda Functions**  
The following is an example log entry for a request to a Lambda function that succeeded:

```
http 2018-11-30T22:23:00.186641Z app/my-loadbalancer/50dc6c495c0c9188
192.168.131.39:2817 - 0.000 0.001 0.000 200 200 34 366
"GET http://www.example.com:80/ HTTP/1.1" "curl/7.46.0" - -
arn:aws:elasticloadbalancing:us-east-2:123456789012:targetgroup/my-targets/73e2d6bc24d8a067
"Root=1-58337364-23a8c76965a2ef7629b185e3" "-" "-"
0 2018-11-30T22:22:48.364000Z "forward" "-" "-" "-" "-" "-" "-"
```

The following is an example log entry for a request to a Lambda function that failed:

```
http 2018-11-30T22:23:00.186641Z app/my-loadbalancer/50dc6c495c0c9188
192.168.131.39:2817 - 0.000 0.001 0.000 502 - 34 366
"GET http://www.example.com:80/ HTTP/1.1" "curl/7.46.0" - -
arn:aws:elasticloadbalancing:us-east-2:123456789012:targetgroup/my-targets/73e2d6bc24d8a067
"Root=1-58337364-23a8c76965a2ef7629b185e3" "-" "-"
0 2018-11-30T22:22:48.364000Z "forward" "-" "LambdaInvalidResponse" "-" "-" "-" "-"
```

## Bucket permissions<a name="access-logging-bucket-permissions"></a>

When you enable access logging, you must specify an S3 bucket for the access logs\. The bucket must meet the following requirements\.

**Requirements**
+ The bucket must be located in the same Region as the load balancer\.
+ Amazon S3\-Managed Encryption Keys \(SSE\-S3\) is required\. No other encryption options are supported\.
+ The bucket must have a bucket policy that grants Elastic Load Balancing permission to write the access logs to your bucket\. Bucket policies are a collection of JSON statements written in the access policy language to define access permissions for your bucket\. Each statement includes information about a single permission and contains a series of elements\.

Use one of the following options to prepare an S3 bucket for access logging\.

**Options**
+ To create a bucket and enable access logging using the Elastic Load Balancing console, skip to [Enable access logging](#enable-access-logging) and select the option to have the console create the bucket and bucket policy for you\.
+ To use an existing bucket and add the required bucket policy using the Amazon S3 console, use the following procedure but skip the steps marked "\[Skip to use existing bucket\]"\.
+ To create a bucket and add the required bucket policy using the Amazon S3 console \(for example, if you are using the AWS CLI or an API to enable access logging\), use the following procedure\.

**To prepare an Amazon S3 bucket for access logging**

1. Open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. \[Skip to use existing bucket\] Choose **Create bucket**\.

1. \[Skip to use existing bucket\] On the **Create bucket** page, do the following:

   1. For **Bucket name**, enter a name for your bucket\. This name must be unique across all existing bucket names in Amazon S3\. In some Regions, there might be additional restrictions on bucket names\. For more information, see [Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) in the *Amazon Simple Storage Service Developer Guide*\.

   1. For **Region**, select the Region where you created your load balancer\.

   1. Choose **Create**\.

1. Select the bucket\. Choose **Permissions** and then choose **Bucket Policy**\.

1. If you are creating a new bucket policy, copy the entire policy document to the policy editor, then replace the placeholders with the corresponding information\. If you are editing an existing bucket policy, copy only the new statement from the policy document \(the text between the \[ and \] of the `Statement` element\)\.

   \[Availability Zones and Local Zones\] Use the following policy\. Update the placeholders for the name and prefix for your bucket, the ID of the AWS account for Elastic Load Balancing \(based on the Region for your load balancer\), and the ID of your AWS account\.

   ```
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::elb-account-id:root"
         },
         "Action": "s3:PutObject",
         "Resource": "arn:aws:s3:::bucket-name/prefix/AWSLogs/your-aws-account-id/*"
       },
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "delivery.logs.amazonaws.com"
         },
         "Action": "s3:PutObject",
         "Resource": "arn:aws:s3:::bucket-name/prefix/AWSLogs/your-aws-account-id/*",
         "Condition": {
           "StringEquals": {
             "s3:x-amz-acl": "bucket-owner-full-control"
           }
         }
       },
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "delivery.logs.amazonaws.com"
         },
         "Action": "s3:GetBucketAcl",
         "Resource": "arn:aws:s3:::bucket-name"
       }
     ]
   }
   ```

   The following table contains the account IDs to use in place of *elb\-account\-id* in your bucket policy\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html)

   \* These Regions requires a separate account\. For more information, see [AWS GovCloud \(US\-West\)](https://aws.amazon.com/govcloud-us/) and [China \(Beijing\)](http://www.amazonaws.cn/en/)\.

   \[Outpost\] Use the following policy\. Update the placeholders for the name and prefix for your bucket and the ID of your AWS account\.

   ```
   {
       "Effect": "Allow",
       "Principal": {
           "Service": "logdelivery.elb.amazonaws.com"
       },
       "Action": "s3.PutObject",
       "Resource": "arn:aws:s3:::bucket-name/prefix/AWSLogs/your-aws-account-id/*",
       "Condition": {
           "StringEquals": {
               "s3:x-amz-acl": "bucket-owner-full-control"
           }
       }
   }
   ```

1. Choose **Save**\.

## Enable access logging<a name="enable-access-logging"></a>

When you enable access logging for your load balancer, you must specify the name of the S3 bucket where the load balancer will store the logs\. The bucket must be in the same Region as your load balancer, and must have a bucket policy that grants Elastic Load Balancing permission to write the access logs to the bucket\. The bucket can be owned by a different account than the account that owns the load balancer\.

**To enable access logging using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Load Balancers**\.

1. Select your load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit load balancer attributes** page, do the following:

   1. For **Access logs**, select **Enable**\.

   1. For **S3 location**, enter the name of your S3 bucket, including any prefix \(for example, `my-loadbalancer-logs/my-app`\)\. You can specify the name of an existing bucket or a name for a new bucket\. If you specify an existing bucket, be sure that you own this bucket and that you configured the required bucket policy\.

   1. \(Optional\) If the bucket does not exist, choose **Create this location for me**\. You must specify a name that is unique across all existing bucket names in Amazon S3 and follows the DNS naming conventions\. For more information, see [Rules for bucket naming](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules) in the *Amazon Simple Storage Service Developer Guide*\.

   1. Choose **Save**\.

**To enable access logging using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command\.

**To verify that Elastic Load Balancing created a test file in your S3 bucket**

After access logging is enabled for your load balancer, Elastic Load Balancing validates the S3 bucket and creates a test file to ensure that the bucket policy specifies the required permissions\. You can use the Amazon S3 console to verify that the test file was created\. Note that the test file is not an actual access log file; it doesn't contain example records\.

1. Open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. For **All Buckets**, select your S3 bucket\.

1. Navigate to the bucket you specified for access logging and look for `ELBAccessLogTestFile`\. For example, if you used the console to create the bucket and bucket policy, the path is as follows:

   ```
   my-bucket/prefix/AWSLogs/123456789012/ELBAccessLogTestFile
   ```

**To manage the S3 bucket for your access logs**  
After you enable access logging, be sure to disable access logging before you delete the bucket with your access logs\. Otherwise, if there is a new bucket with the same name and the required bucket policy but created in an AWS account that you don't own, Elastic Load Balancing could write the access logs for your load balancer to this new bucket\.

## Disable access logging<a name="disable-access-logging"></a>

You can disable access logging for your load balancer at any time\. After you disable access logging, your access logs remain in your S3 bucket until you delete the them\. For more information, see [Working with buckets](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/BucketOperations.html) in the *Amazon Simple Storage Service Console User Guide*\.

**To disable access logging using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Load Balancers**\.

1. Select your load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. For **Access logs**, clear **Enable**\.

1. Choose **Save**\.

**To disable access logging using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command\.

## Processing access log files<a name="log-processing-tools"></a>

The access log files are compressed\. If you open the files using the Amazon S3 console, they are uncompressed and the information is displayed\. If you download the files, you must uncompress them to view the information\.

If there is a lot of demand on your website, your load balancer can generate log files with gigabytes of data\. You might not be able to process such a large amount of data using line\-by\-line processing\. Therefore, you might have to use analytical tools that provide parallel processing solutions\. For example, you can use the following analytical tools to analyze and process access logs:
+ Amazon Athena is an interactive query service that makes it easy to analyze data in Amazon S3 using standard SQL\. For more information, see [Querying Application Load Balancer logs](https://docs.aws.amazon.com/athena/latest/ug/application-load-balancer-logs.html) in the *Amazon Athena User Guide*\.
+ [Loggly](https://www.loggly.com/docs/s3-ingestion-auto/)
+ [Splunk](https://splunkbase.splunk.com/app/1274/)
+ [Sumo logic](https://www.sumologic.com/application/elb/)