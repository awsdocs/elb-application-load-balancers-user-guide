# Enable access logs for your Application Load Balancer<a name="enable-access-logging"></a>

When you enable access logs for your load balancer, you must specify the name of the S3 bucket where the load balancer will store the logs\. The bucket must have a bucket policy that grants Elastic Load Balancing permission to write to the bucket\.

**Topics**
+ [Step 1: Create an S3 bucket](#access-log-create-bucket)
+ [Step 2: Attach a policy to your S3 bucket](#attach-bucket-policy)
+ [Step 3: Configure access logs](#enable-access-logs)
+ [Step 4: Verify bucket permissions](#verify-bucket-permissions)
+ [Troubleshooting](#bucket-permissions-troubleshooting)

## Step 1: Create an S3 bucket<a name="access-log-create-bucket"></a>

When you enable access logs, you must specify an S3 bucket for the access logs\. You can use an existing bucket, or create a bucket specifically for access logs\. The bucket must meet the following requirements\.

**Requirements**
+ The bucket must be located in the same Region as the load balancer\. The bucket and the load balancer can be owned by different accounts\.
+ The prefix that you specify must not include `AWSLogs`\. We add the portion of the file name starting with `AWSLogs` after the bucket name and prefix that you specify\.
+ The only server\-side encryption option that's supported is Amazon S3\-managed keys \(SSE\-S3\)\. For more information, see [Amazon S3\-managed encryption keys \(SSE\-S3\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingServerSideEncryption.html)\.

**To create an S3 bucket using the Amazon S3 console**

1. Open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. Choose **Create bucket**\.

1. On the **Create bucket** page, do the following:

   1. For **Bucket name**, enter a name for your bucket\. This name must be unique across all existing bucket names in Amazon S3\. In some Regions, there might be additional restrictions on bucket names\. For more information, see [Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) in the *Amazon Simple Storage Service User Guide*\.

   1. For **AWS Region**, select the Region where you created your load balancer\.

   1. For **Default encryption**, choose **Amazon S3\-managed keys \(SSE\-S3\)**\.

   1. Choose **Create bucket**\.

## Step 2: Attach a policy to your S3 bucket<a name="attach-bucket-policy"></a>

Your S3 bucket must have a bucket policy that grants Elastic Load Balancing permission to write the access logs to the bucket\. Bucket policies are a collection of JSON statements written in the access policy language to define access permissions for your bucket\. Each statement includes information about a single permission and contains a series of elements\.

If you're using an existing bucket that already has an attached policy, you can add the statement for Elastic Load Balancing access logs to the policy\. If you do so, we recommend that you evaluate the resulting set of permissions to ensure that they are appropriate for the users that need access to the bucket for access logs\.

**Available bucket policies**  
The bucket policy that you'll use depends on the AWS Region and the type of zone\. Each expandable section below contains a bucket policy and information about when to use that policy\.

### Regions available as of August 2022 or later<a name="bucket-policy-logdelivery"></a>

This policy grants permissions to the specified log delivery service\. Use this policy for load balancers in Availability Zones and Local Zones in the following Regions:
+ Asia Pacific \(Hyderabad\)
+ Asia Pacific \(Melbourne\)
+ Europe \(Spain\)
+ Europe \(Zurich\)
+ Middle East \(UAE\)

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "logdelivery.elasticloadbalancing.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::bucket-name/prefix/AWSLogs/aws-account-id/*"
    }
  ]
}
```

### Regions available before August 2022<a name="bucket-policy-account-ids"></a>

This policy grants permissions to the specified Elastic Load Balancing account ID\. Use this policy for load balancers in Availability Zones or Local Zones in the Regions in the list below\.

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
    }
  ]
}
```

Replace *elb\-account\-id* with the ID of the AWS account for Elastic Load Balancing for your Region:
+ US East \(N\. Virginia\) – 127311923021
+ US East \(Ohio\) – 033677994240
+ US West \(N\. California\) – 027434742980
+ US West \(Oregon\) – 797873946194
+ Africa \(Cape Town\) – 098369216593
+ Asia Pacific \(Hong Kong\) – 754344448648
+ Asia Pacific \(Jakarta\) – 589379963580
+ Asia Pacific \(Mumbai\) – 718504428378
+ Asia Pacific \(Osaka\) – 383597477331
+ Asia Pacific \(Seoul\) – 600734575887
+ Asia Pacific \(Singapore\) – 114774131450
+ Asia Pacific \(Sydney\) – 783225319266
+ Asia Pacific \(Tokyo\) – 582318560864
+ Canada \(Central\) – 985666609251
+ Europe \(Frankfurt\) – 054676820928
+ Europe \(Ireland\) – 156460612806
+ Europe \(London\) – 652711504416
+ Europe \(Milan\) – 635631232127
+ Europe \(Paris\) – 009996457667
+ Europe \(Stockholm\) – 897822967062
+ Middle East \(Bahrain\) – 076674570225
+ South America \(São Paulo\) – 507241528517
+ AWS GovCloud \(US\-West\) – 048591011584
+ AWS GovCloud \(US\-East\) – 190560391635

### Outposts Zones<a name="bucket-policy-outposts"></a>

The following policy grants permissions to the specified log delivery service\. Use this policy for load balancers in Outposts Zones\.

```
{
    "Effect": "Allow",
    "Principal": {
        "Service": "logdelivery.elb.amazonaws.com"
    },
    "Action": "s3:PutObject",
    "Resource": "arn:aws:s3:::bucket-name/prefix/AWSLogs/your-aws-account-id/*",
    "Condition": {
        "StringEquals": {
            "s3:x-amz-acl": "bucket-owner-full-control"
        }
    }
}
```

**To attach a bucket policy for access logs to your bucket using the Amazon S3 console**

1. Open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. Select the name of the bucket to open its details page\.

1. Choose **Permissions** and then choose **Bucket policy**, **Edit**\.

1. Update the bucket policy to grant the required permissions\.

1. Choose **Save changes**\.

## Step 3: Configure access logs<a name="enable-access-logs"></a>

Use the following procedure to configure access logs to capture and deliver log files to your S3 bucket\.

**Requirements**  
Your bucket must meet the requirements described in [step 1](#access-log-create-bucket), and you must attach a bucket policy as described in [step 2](#attach-bucket-policy)\.

**To enable access logs using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Load Balancers**\.

1. Select the name of your load balancer to open its details page\.

1. On the **Attributes** tab, choose **Edit**\.

1. On the **Edit load balancer attributes** page, do the following:

   1. For **Monitoring**, turn on **Access logs**\.

   1. Choose **Browse S3** and select a bucket\. Alternatively, enter the location of your S3 bucket, including any prefix\.

   1. Choose **Save changes**\.

**To enable access logs using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command\.

**To manage the S3 bucket for your access logs**  
Be sure to disable access logs before you delete the bucket that you configured for access logs\. Otherwise, if there is a new bucket with the same name and the required bucket policy but created in an AWS account that you don't own, Elastic Load Balancing could write the access logs for your load balancer to this new bucket\.

## Step 4: Verify bucket permissions<a name="verify-bucket-permissions"></a>

After access logs are enabled for your load balancer, Elastic Load Balancing validates the S3 bucket and creates a test file to ensure that the bucket policy specifies the required permissions\. You can use the Amazon S3 console to verify that the test file was created\. The test file is not an actual access log file; it doesn't contain example records\.

**To verify that Elastic Load Balancing created a test file in your S3 bucket**

1. Open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. Select the name of the bucket that you specified for access logs\.

1. Navigate to the test file, `ELBAccessLogTestFile`, in the following location:

   ```
   my-bucket/prefix/AWSLogs/123456789012/ELBAccessLogTestFile
   ```

## Troubleshooting<a name="bucket-permissions-troubleshooting"></a>

If you receive an access denied error, the following are possible causes:
+ The bucket policy does not grant Elastic Load Balancing permission to write access logs to the bucket\.
+ The bucket uses an unsupported server\-side encryption option\. The bucket must use Amazon S3\-managed keys \(SSE\-S3\)\.