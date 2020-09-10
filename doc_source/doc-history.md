# Document history for Application Load Balancers<a name="doc-history"></a>

The following table describes the releases for Application Load Balancers\.


| Feature | Description | Date | 
| --- | --- | --- | 
| Outpost support | You can provision an Application Load Balancer on your Outpost\. The load balancer operates in a single subnet and scales automatically using capacity from the Outpost\. | September 8, 2020 | 
| Desync mitigation mode | This release adds support for desync migitation mode\. For more information, see [Desync mitigation mode](application-load-balancers.md#desync-mitigation-mode)\. | August 17, 2020 | 
| Least outstanding requests | This release adds support for the least outstanding requests algorithm\. For more information, see [Routing algorithm](load-balancer-target-groups.md#modify-routing-algorithm)\. | November 25, 2019 | 
| Weighted target groups | This release adds support for forward actions with multiple target groups\. Requests are distributed to these target groups based on the weight you specify for each target group\. For more information, see [Forward actions](load-balancer-listeners.md#forward-actions)\. | November 19, 2019 | 
| New attribute | This release adds support for the routing\.http\.drop\_invalid\_header\_fields\.enabled attribute\. For more information, see [Load balancer attributes](application-load-balancers.md#load-balancer-attributes)\. | November 11, 2019 | 
| Advanced request routing | This release extends the existing support for host header and path\-based routing by adding conditions for your listener rules based on standard and custom HTTP headers and methods, query parameters, and source IP addresses\. For more information, see [Rule condition types](load-balancer-listeners.md#rule-condition-types)\. | March 27, 2019 | 
| Lambda functions as a target | This release add support to register your Lambda functions as a target\. For more information, see [Lambda functions as targets](lambda-functions.md)\. | November 29, 2018 | 
| Fixed\-response actions | This release adds support for the load balancer to return a custom HTTP response\. For more information, see [Fixed\-response actions](load-balancer-listeners.md#fixed-response-actions)\. | July 25, 2018 | 
| Redirect actions | This release adds support for the load balancer to redirect requests to a different URL\. For more information, see [Redirect actions](load-balancer-listeners.md#redirect-actions)\. | July 25, 2018 | 
| Security policies for FS and TLS 1\.2 | This release adds security policies for Forward Secrecy \(FS\) and TLS 1\.2\. For more information, see [Security policies](create-https-listener.md#describe-ssl-policies)\. | June 6, 2018 | 
| Authentication support | This release adds support for the load balancer to authenticate users of your applications using their corporate or social identities before routing requests\. For more information, see [Authenticate users using an Application Load Balancer](listener-authenticate-users.md)\. | May 30, 2018 | 
| Slow start mode | This release adds support for slow start mode, which gradually increases the share of requests the load balancer sends to a newly registered target while it warms up\. For more information, see [Slow start mode](load-balancer-target-groups.md#slow-start-mode)\. | March 24, 2018 | 
| Resource\-level permissions | This release adds support for resource\-level permissions and tagging condition keys\. For more information, see [Authentication and access control](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/load-balancer-authentication-access-control.html) in the Elastic Load Balancing User Guide\. | May 10, 2018 | 
| SNI support | This release adds support for Server Name Indication \(SNI\)\. For more information, see [SSL certificates](create-https-listener.md#https-listener-certificates)\. | October 10, 2017 | 
| IP addresses as targets | This release adds support for registering IP addresses as targets\. For more information, see [Target type](load-balancer-target-groups.md#target-type)\. | August 31, 2017 | 
| Host\-based routing | This release adds support for routing requests based on the host names in the host header\. For more information, see [Host conditions](load-balancer-listeners.md#host-conditions)\. | April 5, 2017 | 
| Security policies for TLS 1\.1 and TLS 1\.2 | This release adds security policies for TLS 1\.1 and TLS 1\.2\. For more information, see [Security policies](create-https-listener.md#describe-ssl-policies)\. | February 6, 2017 | 
| IPv6 support | This release adds support for IPv6 addresses\. For more information, see [IP address type](application-load-balancers.md#ip-address-type)\. | January 25, 2017 | 
| Request tracing | This release adds support for request tracing\. For more information, see [Request tracing for your Application Load Balancer](load-balancer-request-tracing.md)\. | November 22, 2016 | 
| Percentiles support for the TargetResponseTime metric | This release adds support for the new percentile statistics supported by Amazon CloudWatch\. For more information, see [Statistics for Application Load Balancer metrics](load-balancer-cloudwatch-metrics.md#metric-statistics)\. | November 17, 2016 | 
| New load balancer type | This release of Elastic Load Balancing introduces Application Load Balancers\. | August 11, 2016 | 