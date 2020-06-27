# Monitor your Application Load Balancers<a name="load-balancer-monitoring"></a>

You can use the following features to monitor your load balancers, analyze traffic patterns, and troubleshoot issues with your load balancers and targets\.

**CloudWatch metrics**  
You can use Amazon CloudWatch to retrieve statistics about data points for your load balancers and targets as an ordered set of time\-series data, known as *metrics*\. You can use these metrics to verify that your system is performing as expected\. For more information, see [CloudWatch metrics for your Application Load Balancer](load-balancer-cloudwatch-metrics.md)\.

**Access logs**  
You can use access logs to capture detailed information about the requests made to your load balancer and store them as log files in Amazon S3\. You can use these access logs to analyze traffic patterns and to troubleshoot issues with your targets\. For more information, see [Access logs for your Application Load Balancer](load-balancer-access-logs.md)\.

**Request tracing**  
You can use request tracing to track HTTP requests\. The load balancer adds a header with a trace identifier to each request it receives\. For more information, see [Request tracing for your Application Load Balancer](load-balancer-request-tracing.md)\.

**CloudTrail logs**  
You can use AWS CloudTrail to capture detailed information about the calls made to the Elastic Load Balancing API and store them as log files in Amazon S3\. You can use these CloudTrail logs to determine which calls were made, the source IP address where the call came from, who made the call, when the call was made, and so on\. For more information, see [Logging API calls for your Application Load Balancer using AWS CloudTrail](load-balancer-cloudtrail-logs.md)\.