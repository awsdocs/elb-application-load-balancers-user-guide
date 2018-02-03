# AWS CloudTrail Logging for Your Application Load Balancer<a name="load-balancer-cloudtrail-logs"></a>

Elastic Load Balancing is integrated with AWS CloudTrail, which captures API calls to AWS made by or on behalf of your AWS account, and delivers log files to an Amazon S3 bucket that you specify\. There is no cost to use CloudTrail\. However, the standard rates for Amazon S3 apply\.

CloudTrail logs calls to the AWS APIs, including the Elastic Load Balancing API, whether you use them directly or indirectly through the AWS Management Console\. You can use the information collected by CloudTrail to determine what API call was made, what source IP address was used, who made the call, when it was made, and so on\.

To learn more about CloudTrail, including how to configure and enable it, see the [AWS CloudTrail User Guide](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\. For the complete list of Elastic Load Balancing API actions, see the [Elastic Load Balancing API Reference version 2015\-12\-01](http://docs.aws.amazon.com/elasticloadbalancing/latest/APIReference/)\.

To monitor other actions for your load balancer, such as when a client makes a request to your load balancer, use access logs\. For more information, see [Access Logs for Your Application Load Balancer](load-balancer-access-logs.md)\.


+ [Enable CloudTrail Event Logging](#enable-cloudtrail-logging)
+ [Elastic Load Balancing Event Records in CloudTrail Log Files](#cloudtrail-event-records)

## Enable CloudTrail Event Logging<a name="enable-cloudtrail-logging"></a>

If you haven't done so already, use the following steps to enable CloudTrail event logging for your account\.

**To enable CloudTrail event logging**

1. Open the CloudTrail console at [https://console\.aws\.amazon\.com/cloudtrail/](https://console.aws.amazon.com/cloudtrail/)\.

1. Choose **Get Started Now**\.

1. For **Trail name**, type a name for your trail\.

1. Leave **Apply trail to all regions** as **Yes**\.

1. Choose an existing S3 bucket for your CloudTrail log files, or create a new one\. To create a new bucket, type a unique name for **S3 bucket**\. To use an existing bucket, change **Create a new S3 bucket** to **No** and then select your bucket from **S3 bucket**\.

1. Choose **Turn on**\.

The log files are written to your S3 bucket in the following location:

```
my-bucket/AWSLogs/123456789012/CloudTrail/region/yyyy/mm/dd/
```

For more information, see the [AWS CloudTrail User Guide](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

## Elastic Load Balancing Event Records in CloudTrail Log Files<a name="cloudtrail-event-records"></a>

The log files from CloudTrail contain event information in JSON format\. An event record represents a single AWS API call and includes information about the requested action, such as the user that requested the action, the date and the time of the request, the request parameter, and the response elements\.

The log files include events for all AWS API calls for your AWS account, not just Elastic Load Balancing API calls\. You can locate calls to the Elastic Load Balancing API by checking for `eventSource` elements with the value `elasticloadbalancing.amazonaws.com`\. To view a record for a specific action, such as `CreateLoadBalancer`, check for `eventName` elements with the action name\. To view records for calls made to the Amazon EC2 API by Elastic Load Balancing on your behalf, check for records with an `eventSource` element with the value `ec2.amazonaws.com` and an `invokedBy` element with the value `elasticloadbalancing.amazonaws.com`\. For example, when your load balancer scales to handle a change in traffic volume, it calls the Amazon EC2 API to create and delete network interfaces\.

The following example shows CloudTrail log records for a user who created a load balancer and then deleted it using the AWS CLI\. You can identify the CLI using the `userAgent` elements\. You can identify the requested API calls using the `eventName` elements\. Information about the user \(`Alice`\) can be found in the `userIdentity` element\. For more information about the different elements and values in a CloudTrail log file, see [CloudTrail Event Reference](http://docs.aws.amazon.com/awscloudtrail/latest/userguide/event_reference_top_level.html) in the *AWS CloudTrail User Guide*\.

```
{
   "Records": [
   . . .
   {
      "eventVersion: "1.03",
      "userIdentity": { 
         "type": "IAMUser",
         "principalId": "123456789012",
         "arn": "arn:aws:iam::123456789012:user/Alice",
         "accountId": "123456789012",
         "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
         "userName": "Alice"
      },
      "eventTime": "2016-04-01T15:31:48Z",
      "eventSource": "elasticloadbalancing.amazonaws.com",
      "eventName": "CreateLoadBalancer",
      "awsRegion": "us-west-2",
      "sourceIPAddress": "198.51.100.1",
      "userAgent": "aws-cli/1.10.10 Python/2.7.9 Windows/7 botocore/1.4.1",
      "requestParameters": {
         "subnets": ["subnet-8360a9e7","subnet-b7d581c0"],
         "securityGroups": ["sg-5943793c"],
         "name": "my-load-balancer",
         "scheme": "internet-facing"
      },
      "responseElements": {
         "loadBalancers":[{
            "type": "application",
            "loadBalancerName": "my-load-balancer",
            "vpcId": "vpc-3ac0fb5f",
            "securityGroups": ["sg-5943793c"],
            "state": {"code":"provisioning"},
            "availabilityZones": [
               {"subnetId":"subnet-8360a9e7","zoneName":"us-west-2a"},
               {"subnetId":"subnet-b7d581c0","zoneName":"us-west-2b"}
            ],
            "dNSName": "my-load-balancer-1836718677.us-west-2.elb.amazonaws.com",
            "canonicalHostedZoneId": "Z2P70J7HTTTPLU",
            "createdTime": "Apr 11, 2016 5:23:50 PM",
            "loadBalancerArn": "arn:aws:elasticloadbalancing:us-west-2:123456789012:loadbalancer/app/my-load-balancer/ffcddace1759e1d0",
            "scheme": "internet-facing"
         }]
      },
      "requestID": "b9960276-b9b2-11e3-8a13-f1ef1EXAMPLE",
      "eventID": "6f4ab5bd-2daa-4d00-be14-d92efEXAMPLE",
      "eventType": "AwsApiCall",
      "apiVersion": "2015-12-01",
      "recipientAccountId": "123456789012"
   },
   . . .
   {
      "eventVersion: "1.03",
      "userIdentity": { 
         "type": "IAMUser",
         "principalId": "123456789012",
         "arn": "arn:aws:iam::123456789012:user/Alice",
         "accountId": "123456789012",
         "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
         "userName": "Alice"
      },
      "eventTime": "2016-04-01T15:31:48Z",
      "eventSource": "elasticloadbalancing.amazonaws.com",
      "eventName": "DeleteLoadBalancer",
      "awsRegion": "us-west-2",
      "sourceIPAddress": "198.51.100.1",
      "userAgent": "aws-cli/1.10.10 Python/2.7.9 Windows/7 botocore/1.4.1",
      "requestParameters": {
         "loadBalancerArn": "arn:aws:elasticloadbalancing:us-west-2:123456789012:loadbalancer/app/my-load-balancer/ffcddace1759e1d0"
      },
      "responseElements": null,
      "requestID": "349598b3-000e-11e6-a82b-298133eEXAMPLE",
      "eventID": "75e81c95-4012-421f-a0cf-babdaEXAMPLE",
      "eventType": "AwsApiCall",
      "apiVersion": "2015-12-01",
      "recipientAccountId": "123456789012"
   },
   . . . 
]}
```

You can also use one of the Amazon partner solutions that integrate with CloudTrail to read and analyze your CloudTrail log files\. For more information, see the [AWS CloudTrail Partners](https://aws.amazon.com/cloudtrail/partners/) page\.