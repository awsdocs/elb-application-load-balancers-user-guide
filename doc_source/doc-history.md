# Document history for Application Load Balancers<a name="doc-history"></a>

The following table describes the releases for Application Load Balancers\.

| Change | Description | Date | 
| --- |--- |--- |
| [Target group health](#doc-history) | This release adds support to configure the minimum count or percentage of targets that must be healthy, and what actions the load balancer takes when the threshold is not met\. | November 17, 2022 | 
| [Cross\-zone load balancing](#doc-history) | This release adds support to configure cross\-zone load balancing at the level group level\. | November 17, 2022 | 
| [IPv6 target groups](#doc-history) | This release adds support to configure IPv6 target groups for Application Load Balancers\. | September 20, 2021 | 
| [Client port preservation ](#doc-history) | This release adds an attribute to preserve the source port that the client used to connect to the load balancer\.  | July 29, 2021 | 
| [TLS headers](#doc-history) | This release adds an attribute to indicate that the TLS headers, which contain information about the negotiated TLS version and cipher suite, are added to the client request before sending it to the target | July 21, 2021 | 
| [Additional ACM certificates](#doc-history) | This release supports RSA certificates with 2048, 3072, and 4096\-bit key lengths, and all ECDSA certificates\.  | July 14, 2021 | 
| [Application\-based stickiness](#doc-history) | This release adds an application\-based cookie to support sticky sessions for your load balancer\. | February 8, 2021 | 
| [Least outstanding requests](#doc-history) | This release adds support for the least outstanding requests algorithm\. | November 25, 2020 | 
| [Security policy for FS supporting TLS version 1\.2](#doc-history) | This release adds a security policy for Forward Secrecy \(FS\) supporting TLS version 1\.2\. | November 24, 2020 | 
| [WAF fail open support](#doc-history) | This release adds support for configuring the behavior of your load balancer if it integrates with AWS WAF\. | November 13, 2020 | 
| [gRPC and HTTP/2 support](#doc-history) | This release adds support for gRPC workloads and end\-to\-end HTTP/2\. | October 29, 2020 | 
| [Outpost support](#doc-history) | You can provision an Application Load Balancer on your Outpost\. | September 8, 2020 | 
| [Desync mitigation mode](#doc-history) | This release adds support for desync mitigation mode\. | August 17, 2020 | 
| [Weighted target groups](#doc-history) | This release adds support for forward actions with multiple target groups\. Requests are distributed to these target groups based on the weight you specify for each target group\. | November 19, 2019 | 
| [New attribute](#doc-history) | This release adds support for the routing\.http\.drop\_invalid\_header\_fields\.enabled attribute\. | November 15, 2019 | 
| [Advanced request routing](#doc-history) | This release adds support for additional condition types for your listener rules\. | March 27, 2019 | 
| [Lambda functions as a target](#doc-history) | This release adds support for registering Lambda functions as a target\. | November 29, 2018 | 
| [Redirect actions](#doc-history) | This release adds support for the load balancer to redirect requests to a different URL\. | July 25, 2018 | 
| [Fixed\-response actions](#doc-history) | This release adds support for the load balancer to return a custom HTTP response\. | July 25, 2018 | 
| [Security policies for FS and TLS 1\.2](#doc-history) | This release adds support for two additional predefined security policies\. | June 6, 2018 | 
| [User authentication](#doc-history) | This release adds support for the load balancer to authenticate users of your applications using their corporate or social identities before routing requests\. | May 30, 2018 | 