# Lambda Functions as Targets<a name="lambda-functions"></a>

You can register your Lambda functions as targets and configure a listener rule to forward requests to the target group for your Lambda function\. When the load balancer forwards the request to a target group with a Lambda function as a target, it invokes your Lambda function and passes the content of the request to the Lambda function, in JSON format\.

**Limits**
+ The maximum size of the request body that you can send to a Lambda function is 1 MB\. For related size limits, see [HTTP Header Limits](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html#http-header-limits)\.
+ The maximum size of the response JSON that the Lambda function can send is 1 MB\.
+ WebSockets are not supported\. Upgrade requests are rejected with an HTTP 400 code\.

**Topics**
+ [Prepare the Lambda Function](#prepare-lambda-function)
+ [Create a Target Group for the Lambda Function](#register-lambda-function)
+ [Receive Events From the Load Balancer](#receive-event-from-load-balancer)
+ [Respond to the Load Balancer](#respond-to-load-balancer)
+ [Multi\-Value Headers](#multi-value-headers)
+ [Enable Health Checks](#enable-health-checks-lambda)
+ [Deregister the Lambda Function](#deregister-lambda-function)

## Prepare the Lambda Function<a name="prepare-lambda-function"></a>

**Permissions to Invoke the Lambda Function**  
If you create the target group and register the Lambda function using the AWS Management Console, the console adds the required permissions to your Lambda function policy on your behalf\. Otherwise, after you create the target group and register the function using the AWS CLI, you must use the [add\-permission](https://docs.aws.amazon.com/cli/latest/reference/lambda/add-permission.html) command to grant Elastic Load Balancing permission to invoke your Lambda function\. We recommend that you include the `--source-arn` parameter to restrict function invocation to the specified target group\.

```
aws lambda add-permission \
--function-name lambda-function-arn-with-alias-name \ 
--statement-id elb1 \
--principal elasticloadbalancing.amazonaws.com \
--action lambda:InvokeFunction \
--source-arn target-group-arn
```

**Lambda Function Versioning**  
You can register one Lambda function per target group\. To ensure that you can change your Lambda function and that the load balancer always invokes the current version of the Lambda function, create a function alias and include the alias in the function ARN when you register the Lambda function with the load balancer\. For more information, see [AWS Lambda Function Versioning and Aliases](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html) and [Traffic Shifting Using Aliases](https://docs.aws.amazon.com/lambda/latest/dg/lambda-traffic-shifting-using-aliases.html) in the *AWS Lambda Developer Guide*\.

**Function Timeout**  
The load balancer waits until your Lambda function responds or times out\. We recommend that you configure the timeout of the Lambda function based on your expected run time\. For information about the default timeout value and how to change it, see [Basic AWS Lambda Function Configuration](https://docs.aws.amazon.com/lambda/latest/dg/resource-model.html)\. For information about the maximum timeout value you can configure, see [AWS Lambda Limits](https://docs.aws.amazon.com/lambda/latest/dg/limits.html)\.

## Create a Target Group for the Lambda Function<a name="register-lambda-function"></a>

Create a target group, which is used in request routing\. If the request content matches a listener rule with an action to forward it to this target group, the load balancer invokes the registered Lambda function\.

**To create a target group and register the Lambda function**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Choose **Create target group**\.

1. For **Target group name**, type a name for the target group\.

1. For **Target type**, select **Lambda function**\.

1. For **Lambda function**, do one of the following:
   + Select the Lambda function
   + Create a new Lambda function and select it
   + Register the Lambda function after you create the target group

1. \(Optional\) To enable health checks, choose **Health check**, **Enable**\.

1. Choose **Create**\.

**To create a target group and deregister the Lambda function using the AWS CLI**  
Use the [create\-target\-group](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-target-group.html) and [register\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/register-targets.html) commands\.

## Receive Events From the Load Balancer<a name="receive-event-from-load-balancer"></a>

The load balancer supports Lambda invocation for requests over both HTTP and HTTPS\. The load balancer sends an event in JSON format\. The load balancer adds the following headers to every request: `X-Amzn-Trace-Id`, `X-Forwarded-For`, `X-Forwarded-Port`, and `X-Forwarded-Proto`\.

If the content type is one of the following types, the load balancer sends the body to the Lambda function as is and sets `isBase64Encoded` to `false`: text/\*, application/json, application/javascript, and application/xml\. For all other types, the load balancer Base64 encodes the body and sets `isBase64Encoded` to `true`\.

The following is an example event\.

```
{
    "requestContext": {
        "elb": {
            "targetGroupArn": "arn:aws:elasticloadbalancing:region:123456789012:targetgroup/my-target-group/6d0ecf831eec9f09"
        }
    },
    "httpMethod": "GET",
    "path": "/",
    "queryStringParameters": {parameters},
    "headers": {
        "accept": "text/html,application/xhtml+xml",
        "accept-language": "en-US,en;q=0.8",
        "content-type": "text/plain",
        "cookie": "cookies",
        "host": "lambda-846800462-us-east-2.elb.amazonaws.com",
        "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6)",
        "x-amzn-trace-id": "Root=1-5bdb40ca-556d8b0c50dc66f0511bf520",
        "x-forwarded-for": "72.21.198.66",
        "x-forwarded-port": "443",
        "x-forwarded-proto": "https"
    },
    "isBase64Encoded": false,
    "body": "request_body"
}
```

## Respond to the Load Balancer<a name="respond-to-load-balancer"></a>

The response from your Lambda function must include the Base64 encoding status, status code, status description, and headers\. You can omit the body\. The `statusDescription` header must contain the status code and reason phrase, separated by a single space\.

To include a binary content in the body of the response, you must Base64 encode the content and set `isBase64Encoded` to `true`\. The load balancer decodes the content to retrieve the binary content and sends it to the client in the body of the HTTP response\.

The load balancer does not honor hop\-by\-hop headers, such as `Connection` or `Transfer-Encoding`\. You can omit the `Content-Length` header because the load balancer computes it before sending responses to clients\.

The following is an example response from a Lambda function\.

```
{
    "isBase64Encoded": false,
    "statusCode": 200,
    "statusDescription": "200 OK",
    "headers": {
        "Set-cookie": "cookies",
        "Content-Type": "application/json"
    },
    "body": "Hello from Lambda (optional)"
}
```

For Lambda function templates that work with Application Load Balancers, see [application\-load\-balancer\-serverless\-app](https://github.com/aws/elastic-load-balancing-tools/tree/master/application-load-balancer-serverless-app) on github\. Alternatively, open the [Lambda console](https://console.aws.amazon.com/lambda), create a function, and select one of the following from the AWS Serverless Application Repository:
+ ALB\-Lambda\-Target\-HelloWorld
+ ALB\-Lambda\-Target\-UploadFiletoS3
+ ALB\-Lambda\-Target\-BinaryResponse
+ ALB\-Lambda\-Target\-WhatisMyIP

## Multi\-Value Headers<a name="multi-value-headers"></a>

If requests from a client or responses from a Lambda function contain headers with multiple values or contains the same header multiple times, you can enable support for multi\-value header syntax\. After you enable multi\-value headers, the request and response headers exchanged between the load balancer and the Lambda function use arrays\. Otherwise, the load balancer uses the last value it receives\.

**Topics**
+ [Requests with Multi\-Value Headers](#multi-value-headers-request)
+ [Responses with Multi\-Value Headers](#multi-value-headers-response)
+ [Enable Multi\-Value Headers](#enable-multi-value-headers)

### Requests with Multi\-Value Headers<a name="multi-value-headers-request"></a>

The following example request has two query parameters with the same key:

```
http://www.example.com?&myKey=val1&myKey=val2
```

With the default format, the load balancer uses the last value sent by the client and sends you an event that includes the following:

```
"queryStringParameters": { "myKey": "val2"},
```

With multi\-value headers, the load balancer uses both key values sent by the client and sends you an event that includes the following:

```
"multiValueQueryStringParameters": { "myKey": ["val1", "val2"] },
```

Similarly, suppose that the client sends a request with two cookies in the header:

```
"cookie": "name1=value1",
"cookie": "name2=value2",
```

With the default format, the load balancer uses the last cookie sent by the client and sends you an event that includes the following:

```
"headers": {
    "cookie": "name2=value2",
    ...
},
```

With multi\-value headers, the load balancer uses both cookies sent by the client and sends you an event that includes the following:

```
"multiValueHeaders": {
    "cookie": ["name1=value1", "name2=value2"],
    ...
},
```

### Responses with Multi\-Value Headers<a name="multi-value-headers-response"></a>

With the default format, you can specify a single cookie:

```
{
  "headers": {
      "Set-cookie": "cookie-name=cookie-value;Domain=myweb.com;Secure;HttpOnly",
      "Content-Type": "application/json"
  },
}
```

With multi\-value headers, you can specify multiple cookies as follows:

```
{
  "multiValueHeaders": {
      "Set-cookie": ["cookie-name=cookie-value;Domain=myweb.com;Secure;HttpOnly","cookie-name=cookie-value;Expires=May 8, 2019"],
      "Content-Type": ["application/json"]
  },
}
```

### Enable Multi\-Value Headers<a name="enable-multi-value-headers"></a>

You can enable or disable multi\-value headers for a target group with the target type `lambda`\.

**To enable multi\-value headers using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select your target group\.

1. On the **Description** tab, choose **Edit attributes**\.

1. Select **Enable**\.

1. Choose **Save**\.

**To enable multi\-value headers using the AWS CLI**  
Use the [modify\-target\-group\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group-attributes.html) command with the `lambda.multi_value_headers.enabled` attribute\.

## Enable Health Checks<a name="enable-health-checks-lambda"></a>

By default, health checks are disabled for target groups of type `lambda`\. You can enable health checks in order to implement DNS failover with Amazon Route 53\. The Lambda function can check the health of a downstream service before responding to the health check request\. If the response from the Lambda function indicates a health check failure, the health check failure is passed to Route 53\. You can configure Route 53 to fail over to a backup application stack\.

You are charged for health checks as you are for any Lambda function invocation\.

The following is the format of the health check event sent to your Lambda function\. To check whether an event is a health check event, check the value of the user\-agent field\. The user agent for health checks is `ELB-HealthChecker/2.0`\.

```
{
    "requestContext": {
        "elb": {
            "targetGroupArn": "arn:aws:elasticloadbalancing:region:123456789012:targetgroup/my-target-group/6d0ecf831eec9f09"
        }
    },
    "httpMethod": "GET",  
    "path": "/",  
    "queryStringParameters": {},  
    "headers": {
        "user-agent": "ELB-HealthChecker/2.0"
    },  
    "body": "",  
    "isBase64Encoded": false
}
```

**To enable health checks for a target group**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select your target group\.

1. On the **Health checks** tab, choose **Edit health check**\.

1. Select **Enable**\.

1. Choose **Save**\.

**To enable health checks for a target group using the AWS CLI**  
Use the [modify\-target\-group](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-target-group.html) command with the `--health-check-enabled` option\.

## Deregister the Lambda Function<a name="deregister-lambda-function"></a>

If you no longer need to send traffic to your Lambda function, you can deregister it\. After you deregister a Lambda function, in\-flight requests fail with HTTP 5XX errors\.

To replace a Lambda function, we recommend that you create a new target group, register the new function with the new target group, and update the listener rules to use the new target group instead of the existing one\.

**To deregister the Lambda function**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Target Groups**\.

1. Select your target group\.

1. On the **Targets** tab, choose **Deregister**\.

1. Choose **Deregister**\.

**To deregister the Lambda function using the AWS CLI**  
Use the [deregister\-targets](https://docs.aws.amazon.com/cli/latest/reference/elbv2/deregister-targets.html) command\.