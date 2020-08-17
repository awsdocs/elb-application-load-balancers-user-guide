# CloudWatch metrics for your Application Load Balancer<a name="load-balancer-cloudwatch-metrics"></a>

Elastic Load Balancing publishes data points to Amazon CloudWatch for your load balancers and your targets\. CloudWatch enables you to retrieve statistics about those data points as an ordered set of time\-series data, known as *metrics*\. Think of a metric as a variable to monitor, and the data points as the values of that variable over time\. For example, you can monitor the total number of healthy targets for a load balancer over a specified time period\. Each data point has an associated time stamp and an optional unit of measurement\.

You can use metrics to verify that your system is performing as expected\. For example, you can create a CloudWatch alarm to monitor a specified metric and initiate an action \(such as sending a notification to an email address\) if the metric goes outside what you consider an acceptable range\.

Elastic Load Balancing reports metrics to CloudWatch only when requests are flowing through the load balancer\. If there are requests flowing through the load balancer, Elastic Load Balancing measures and sends its metrics in 60\-second intervals\. If there are no requests flowing through the load balancer or no data for a metric, the metric is not reported\.

For more information, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)\.

**Topics**
+ [Application Load Balancer metrics](#load-balancer-metrics-alb)
+ [Metric dimensions for Application Load Balancers](#load-balancer-metric-dimensions-alb)
+ [Statistics for Application Load Balancer metrics](#metric-statistics)
+ [View CloudWatch metrics for your load balancer](#view-metric-data)

## Application Load Balancer metrics<a name="load-balancer-metrics-alb"></a>

The `AWS/ApplicationELB` namespace includes the following metrics for load balancers\.


| Metric | Description | 
| --- | --- | 
| ActiveConnectionCount |  The total number of concurrent TCP connections active from clients to the load balancer and from the load balancer to targets\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| ClientTLSNegotiationErrorCount |  The number of TLS connections initiated by the client that did not establish a session with the load balancer due to a TLS error\. Possible causes include a mismatch of ciphers or protocols or the client failing to verify the server certificate and closing the connection\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| ConsumedLCUs |  The number of load balancer capacity units \(LCU\) used by your load balancer\. You pay for the number of LCUs that you use per hour\. For more information, see [Elastic Load Balancing pricing](https://aws.amazon.com/elasticloadbalancing/pricing/)\. **Reporting criteria**: Always reported **Statistics**: All [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| DesyncMitigationMode\_NonCompliant\_Request\_Count |  The number of requests that do not comply with RFC 7230\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| DroppedInvalidHeaderRequestCount |  The number of requests where the load balancer removed HTTP headers with header fields that are not valid before routing the request\. The load balancer removes these headers only if the `routing.http.drop_invalid_header_fields.enabled` attribute is set to `true`\. **Reporting criteria**: Always reported **Statistics**: All [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| ForwardedInvalidHeaderRequestCount |  The number of requests routed by the load balancer that had HTTP headers with header fields that are not valid\. The load balancer forwards requests with these headers only if the `routing.http.drop_invalid_header_fields.enabled` attribute is set to `false`\. **Reporting criteria**: Always reported **Statistics**: All [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTP\_Fixed\_Response\_Count |  The number of fixed\-response actions that were successful\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTP\_Redirect\_Count |  The number of redirect actions that were successful\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTP\_Redirect\_Url\_Limit\_Exceeded\_Count |  The number of redirect actions that couldn't be completed because the URL in the response location header is larger than 8K\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTPCode\_ELB\_3XX\_Count |  The number of HTTP 3XX redirection codes that originate from the load balancer\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTPCode\_ELB\_4XX\_Count |  The number of HTTP 4XX client error codes that originate from the load balancer\. Client errors are generated when requests are malformed or incomplete\. These requests were not received by the target, other than in the case where the load balancer returns an [HTTP 460 error code](load-balancer-troubleshooting.md#http-460-issues)\. This count does not include any response codes generated by the targets\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. Note that `Minimum`, `Maximum`, and `Average` all return 1\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTPCode\_ELB\_5XX\_Count |  The number of HTTP 5XX server error codes that originate from the load balancer\. This count does not include any response codes generated by the targets\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. Note that `Minimum`, `Maximum`, and `Average` all return 1\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTPCode\_ELB\_500\_Count |  The number of HTTP 500 error codes that originate from the load balancer\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTPCode\_ELB\_502\_Count |  The number of HTTP 502 error codes that originate from the load balancer\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTPCode\_ELB\_503\_Count |  The number of HTTP 503 error codes that originate from the load balancer\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTPCode\_ELB\_504\_Count |  The number of HTTP 504 error codes that originate from the load balancer\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| IPv6ProcessedBytes |  The total number of bytes processed by the load balancer over IPv6\. This count is included in `ProcessedBytes`\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| IPv6RequestCount |  The number of IPv6 requests received by the load balancer\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. Note that `Minimum`, `Maximum`, and `Average` all return 1\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| NewConnectionCount |  The total number of new TCP connections established from clients to the load balancer and from the load balancer to targets\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| NonStickyRequestCount |  The number of requests where the load balancer chose a new target because it couldn't use an existing sticky session\. For example, the request was the first request from a new client and no stickiness cookie was presented, a stickiness cookie was presented but it did not specify a target that was registered with this target group, the stickiness cookie was malformed or expired, or an internal error prevented the load balancer from reading the stickiness cookie\. **Reporting criteria**: Stickiness is enabled on the target group\. **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| ProcessedBytes |  The total number of bytes processed by the load balancer over IPv4 and IPv6\. This count includes traffic to and from clients and Lambda functions, and traffic from an Identity Provider \(IdP\) if user authentication is enabled\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| RejectedConnectionCount |  The number of connections that were rejected because the load balancer had reached its maximum number of connections\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| RequestCount |  The number of requests processed over IPv4 and IPv6\. This count includes only the requests with a response generated by a target of the load balancer\. **Reporting criteria**: Always reported **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| RuleEvaluations |  The number of rules processed by the load balancer given a request rate averaged over an hour\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 

The `AWS/ApplicationELB` namespace includes the following metrics for targets\.


| Metric | Description | 
| --- | --- | 
| HealthyHostCount |  The number of targets that are considered healthy\. **Reporting criteria**: Reported if health checks are enabled **Statistics**: The most useful statistics are `Average`, `Minimum`, and `Maximum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| HTTPCode\_Target\_2XX\_Count, HTTPCode\_Target\_3XX\_Count, HTTPCode\_Target\_4XX\_Count, HTTPCode\_Target\_5XX\_Count |  The number of HTTP response codes generated by the targets\. This does not include any response codes generated by the load balancer\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. Note that `Minimum`, `Maximum`, and `Average` all return 1\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| RequestCountPerTarget |  The average number of requests received by each target in a target group\. You must specify the target group using the `TargetGroup` dimension\. This metric does not apply if the target is a Lambda function\. **Reporting criteria**: Always reported **Statistics**: The only valid statistic is `Sum`\. Note that this represents the average not the sum\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| TargetConnectionErrorCount |  The number of connections that were not successfully established between the load balancer and target\. This metric does not apply if the target is a Lambda function\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| TargetResponseTime |  The time elapsed, in seconds, after the request leaves the load balancer until a response from the target is received\. This is equivalent to the `target_processing_time` field in the access logs\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistics are `Average` and `pNN.NN` \(percentiles\)\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| TargetTLSNegotiationErrorCount |  The number of TLS connections initiated by the load balancer that did not establish a session with the target\. Possible causes include a mismatch of ciphers or protocols\. This metric does not apply if the target is a Lambda function\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| UnHealthyHostCount |  The number of targets that are considered unhealthy\. **Reporting criteria**: Reported if health checks are enabled **Statistics**: The most useful statistics are `Average`, `Minimum`, and `Maximum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 

The `AWS/ApplicationELB` namespace includes the following metrics for Lambda functions that are registered as targets\.


| Metric | Description | 
| --- | --- | 
| LambdaInternalError |  The number of requests to a Lambda function that failed because of an issue internal to the load balancer or AWS Lambda\. To get the error reason codes, check the error\_reason field of the access log\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| LambdaTargetProcessedBytes |  The total number of bytes processed by the load balancer for requests to and responses from a Lambda function\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| LambdaUserError |  The number of requests to a Lambda function that failed because of an issue with the Lambda function\. For example, the load balancer did not have permission to invoke the function, the load balancer received JSON from the function that is malformed or missing required fields, or the size of the request body or response exceeded the maximum size of 1 MB\. To get the error reason codes, check the error\_reason field of the access log\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 

The `AWS/ApplicationELB` namespace includes the following metrics for user authentication\.


| Metric | Description | 
| --- | --- | 
| ELBAuthError |  The number of user authentications that could not be completed because an authenticate action was misconfigured, the load balancer couldn't establish a connection with the IdP, or the load balancer couldn't complete the authentication flow due to an internal error\. To get the error reason codes, check the error\_reason field of the access log\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| ELBAuthFailure |  The number of user authentications that could not be completed because the IdP denied access to the user or an authorization code was used more than once\. To get the error reason codes, check the error\_reason field of the access log\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| ELBAuthLatency |  The time elapsed, in milliseconds, to query the IdP for the ID token and user info\. If one or more of these operations fail, this is the time to failure\. **Reporting criteria**: There is a nonzero value **Statistics**: All statistics are meaningful\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| ELBAuthRefreshTokenSuccess |  The number of times the load balancer successfully refreshed user claims using a refresh token provided by the IdP\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| ELBAuthSuccess |  The number of authenticate actions that were successful\. This metric is incremented at the end of the authentication workflow, after the load balancer has retrieved the user claims from the IdP\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 
| ELBAuthUserClaimsSizeExceeded |  The number of times that a configured IdP returned user claims that exceeded 11K bytes in size\. **Reporting criteria**: There is a nonzero value **Statistics**: The only meaningful statistic is `Sum`\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html)  | 

## Metric dimensions for Application Load Balancers<a name="load-balancer-metric-dimensions-alb"></a>

To filter the metrics for your Application Load Balancer, use the following dimensions\.


| Dimension | Description | 
| --- | --- | 
| AvailabilityZone |  Filters the metric data by Availability Zone\.  | 
| LoadBalancer |  Filters the metric data by load balancer\. Specify the load balancer as follows: app/*load\-balancer\-name*/*1234567890123456* \(the final portion of the load balancer ARN\)\.  | 
| TargetGroup |  Filters the metric data by target group\. Specify the target group as follows: targetgroup/*target\-group\-name*/*1234567890123456* \(the final portion of the target group ARN\)\.  | 

## Statistics for Application Load Balancer metrics<a name="metric-statistics"></a>

CloudWatch provides statistics based on the metric data points published by Elastic Load Balancing\. Statistics are metric data aggregations over specified period of time\. When you request statistics, the returned data stream is identified by the metric name and dimension\. A dimension is a name\-value pair that uniquely identifies a metric\. For example, you can request statistics for all the healthy EC2 instances behind a load balancer launched in a specific Availability Zone\.

The `Minimum` and `Maximum` statistics reflect the minimum and maximum reported by the individual load balancer nodes\. For example, suppose there are 2 load balancer nodes\. One node has `HealthyHostCount` with a `Minimum` of 2, a `Maximum` of 10, and an `Average` of 6, while the other node has `HealthyHostCount` with a `Minimum` of 1, a `Maximum` of 5, and an `Average` of 3\. Therefore, the load balancer has a `Minimum` of 1, a `Maximum` of 10, and an `Average` of about 4\.

The `Sum` statistic is the aggregate value across all load balancer nodes\. Because metrics include multiple reports per period, `Sum` is only applicable to metrics that are aggregated across all load balancer nodes\.

The `SampleCount` statistic is the number of samples measured\. Because metrics are gathered based on sampling intervals and events, this statistic is typically not useful\. For example, with `HealthyHostCount`, `SampleCount` is based on the number of samples that each load balancer node reports, not the number of healthy hosts\.

A percentile indicates the relative standing of a value in a data set\. You can specify any percentile, using up to two decimal places \(for example, p95\.45\)\. For example, the 95th percentile means that 95 percent of the data is below this value and 5 percent is above\. Percentiles are often used to isolate anomalies\. For example, suppose that an application serves the majority of requests from a cache in 1\-2 ms, but in 100\-200 ms if the cache is empty\. The maximum reflects the slowest case, around 200 ms\. The average doesn't indicate the distribution of the data\. Percentiles provide a more meaningful view of the application's performance\. By using the 99th percentile as an Auto Scaling trigger or a CloudWatch alarm, you can target that no more than 1 percent of requests take longer than 2 ms to process\.

## View CloudWatch metrics for your load balancer<a name="view-metric-data"></a>

You can view the CloudWatch metrics for your load balancers using the Amazon EC2 console\. These metrics are displayed as monitoring graphs\. The monitoring graphs show data points if the load balancer is active and receiving requests\.

Alternatively, you can view metrics for your load balancer using the CloudWatch console\.

**To view metrics using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. To view metrics filtered by target group, do the following:

   1. In the navigation pane, choose **Target Groups**\.

   1. Select your target group, and then choose the **Monitoring** tab\.

   1. \(Optional\) To filter the results by time, select a time range from **Showing data for**\.

   1. To get a larger view of a single metric, select its graph\.

1. To view metrics filtered by load balancer, do the following:

   1. In the navigation pane, choose **Load Balancers**\.

   1. Select your load balancer, and then choose the **Monitoring** tab\.

   1. \(Optional\) To filter the results by time, select a time range from **Showing data for**\.

   1. To get a larger view of a single metric, select its graph\.

**To view metrics using the CloudWatch console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. In the navigation pane, choose **Metrics**\.

1. Select the **ApplicationELB** namespace\.

1. \(Optional\) To view a metric across all dimensions, enter its name in the search field\.

1. \(Optional\) To filter by dimension, select one of the following:
   + To display only the metrics reported for your load balancers, choose **Per AppELB Metrics**\. To view the metrics for a single load balancer, enter its name in the search field\.
   + To display only the metrics reported for your target groups, choose **Per AppELB, per TG Metrics**\. To view the metrics for a single target group, enter its name in the search field\.
   + To display only the metrics reported for your load balancers by Availability Zone, choose **Per AppELB, per AZ Metrics**\. To view the metrics for a single load balancer, enter its name in the search field\. To view the metrics for a single Availability Zone, enter its name in the search field\.
   + To display only the metrics reported for your load balancers by Availability Zone and target group, choose **Per AppELB, per AZ, per TG Metrics**\. To view the metrics for a single load balancer, enter its name in the search field\. To view the metrics for a single target group, enter its name in the search field\. To view the metrics for a single Availability Zone, enter its name in the search field\.

**To view metrics using the AWS CLI**  
Use the following [list\-metrics](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/list-metrics.html) command to list the available metrics:

```
aws cloudwatch list-metrics --namespace AWS/ApplicationELB
```

**To get the statistics for a metric using the AWS CLI**  
Use the following [get\-metric\-statistics](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/get-metric-statistics.html) command get statistics for the specified metric and dimension\. Note that CloudWatch treats each unique combination of dimensions as a separate metric\. You can't retrieve statistics using combinations of dimensions that were not specially published\. You must specify the same dimensions that were used when the metrics were created\.

```
aws cloudwatch get-metric-statistics --namespace AWS/ApplicationELB \
--metric-name UnHealthyHostCount --statistics Average  --period 3600 \
--dimensions Name=LoadBalancer,Value=app/my-load-balancer/50dc6c495c0c9188 \
Name=TargetGroup,Value=targetgroup/my-targets/73e2d6bc24d8a067 \
--start-time 2016-04-18T00:00:00Z --end-time 2016-04-21T00:00:00Z
```

The following is example output:

```
{
    "Datapoints": [
        {
            "Timestamp": "2016-04-18T22:00:00Z",
            "Average": 0.0,
            "Unit": "Count"
        },
        {
            "Timestamp": "2016-04-18T04:00:00Z",
            "Average": 0.0,
            "Unit": "Count"
        },
        ...
    ],
    "Label": "UnHealthyHostCount"
}
```