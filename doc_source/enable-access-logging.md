# Enable access logs for your Application Load Balancer<a name="enable-access-logging"></a>

When you enable access logs for your load balancer, you must specify the name of the S3 bucket where the load balancer will store the logs\. The bucket must have a bucket policy that grants Elastic Load Balancing permission to write to the bucket\.

**Topics**
+ [Create an S3 bucket](#access-log-create-bucket)
+ [Attach a policy to your S3 bucket](#attach-bucket-policy)
+ [Configure access logs](#enable-access-logs)
+ [Verify bucket permissions](#verify-bucket-permissions)

## Create an S3 bucket<a name="access-log-create-bucket"></a>

When you enable access logs, you must specify an S3 bucket for the access logs\. The bucket must meet the following requirements\.

**Requirements**
+ The bucket must be located in the same Region as the load balancer\. The bucket and the load balancer can be owned by different accounts\.
+ The prefix that you specify must not include `AWSLogs`\. We add the portion of the file name starting with `AWSLogs` after the bucket name and prefix that you specify\.
+ The only server\-side encryption option that's supported is Amazon S3\-managed keys \(SSE\-S3\)\. For more information, see [Amazon S3\-managed encryption keys \(SSE\-S3\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingServerSideEncryption.html)\.

Use one of the following options to create and configure an S3 bucket for access logs\.

**Options**
+ To create a bucket and enable access logs using the Elastic Load Balancing console, skip to [Configure access logs](#enable-access-logs) and select the option to have the console create the bucket and bucket policy for you\.
+ To use an existing bucket and add the required bucket policy using the Amazon S3 console, skip to [Attach a policy to your S3 bucket](#attach-bucket-policy)\.
+ To create a bucket and add the required bucket policy using the Amazon S3 console manually \(for example, if you are using the AWS CLI or an API to enable access logs\), use the procedure below and then continue to [Attach a policy to your S3 bucket](#attach-bucket-policy)\.

Use the following procedure to create a bucket manually using the Amazon S3 console, if you aren't using the Elastic Load Balancing console to create and configure the bucket on your behalf\.

**To create an S3 bucket manually using the Amazon S3 console**

1. Open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. Choose **Create bucket**\.

1. On the **Create bucket** page, do the following:

   1. For **Bucket name**, enter a name for your bucket\. This name must be unique across all existing bucket names in Amazon S3\. In some Regions, there might be additional restrictions on bucket names\. For more information, see [Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) in the *Amazon Simple Storage Service User Guide*\.

   1. For **AWS Region**, select the Region where you created your load balancer\.

   1. \(Optional\) Enable server\-side encryption using Amazon S3\-managed keys \(SSE\-S3\)\.

   1. Choose **Create bucket**\.

## Attach a policy to your S3 bucket<a name="attach-bucket-policy"></a>

Your S3 bucket must have a bucket policy that grants Elastic Load Balancing permission to write the access logs to the bucket\. Bucket policies are a collection of JSON statements written in the access policy language to define access permissions for your bucket\. Each statement includes information about a single permission and contains a series of elements\.

If you're using an existing bucket that already has an attached policy, you can add the statement for Elastic Load Balancing access logs to the policy\. If you do so, we recommend that you evaluate the resulting set of permissions to ensure that they are appropriate for the users that need access to the bucket for access logs\.

**To attach a bucket policy for access logs to your bucket**

1. Open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. Select the name of the bucket to open its details page\.

1. Choose **Permissions** and then choose **Bucket policy**, **Edit**\.

1. Create or update the bucket policy to grant the required permissions\. The bucket policy you'll use depends on the AWS Region of the bucket and the type of zone\.
   + \[Availability Zones and Local Zones\] For the Regions in the list below, use the following policy, which grants permissions to the specified Elastic Load Balancing account ID\.

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
     + Canada \(Central\) – 985666609251
     + Europe \(Frankfurt\) – 054676820928
     + Europe \(Ireland\) – 156460612806
     + Europe \(London\) – 652711504416
     + Europe \(Milan\) – 635631232127
     + Europe \(Paris\) – 009996457667
     + Europe \(Stockholm\) – 897822967062
     + Asia Pacific \(Hong Kong\) – 754344448648
     + Asia Pacific \(Tokyo\) – 582318560864
     + Asia Pacific \(Seoul\) – 600734575887
     + Asia Pacific \(Osaka\) – 383597477331
     + Asia Pacific \(Singapore\) – 114774131450
     + Asia Pacific \(Sydney\) – 783225319266
     + Asia Pacific \(Mumbai\) – 718504428378
     + Middle East \(Bahrain\) – 076674570225
     + South America \(São Paulo\) – 507241528517
     + AWS GovCloud \(US\-West\) – 048591011584
     + AWS GovCloud \(US\-East\) – 190560391635
   + \[Availability Zones and Local Zones\] For the Regions that do not appear in the list above, such as Middle East \(UAE\), use the following policy, which grants permissions to the specified log delivery service\.

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
   + \[Outpost\] Use the following policy, which grants permissions to the specified log delivery service\.

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

1. Choose **Save changes**\.

## Configure access logs<a name="enable-access-logs"></a>

Use the following procedure to configure access logs to capture and deliver log files to your S3 bucket\. Note that you can optionally have Elastic Load Balancing create the bucket and add the required policy, if you did not use the previous steps to do so manually\. If you specify an existing bucket, be sure that you own the bucket and that you added the required bucket policy\.

**To enable access logs using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Load Balancers**\.

1. Select your load balancer\.

1. On the **Description** tab, choose **Edit attributes**\.

1. On the **Edit load balancer attributes** page, do the following:

   1. For **Access logs**, select **Enable**\.

   1. For **S3 location**, enter the name of your S3 bucket, including any prefix \(for example, `my-loadbalancer-logs/my-app`\)\. You can enter the name of an existing bucket or a name for a new bucket\.

   1. \(Optional\) If the bucket does not exist, choose **Create this location for me**\. The name for a new bucket must be unique across all existing bucket names in Amazon S3 and follow DNS naming conventions\. For more information, see [Rules for bucket naming](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules) in the *Amazon Simple Storage Service User Guide*\.

   1. Choose **Save**\.

**To enable access logs using the AWS CLI**  
Use the [modify\-load\-balancer\-attributes](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-load-balancer-attributes.html) command\.

**To manage the S3 bucket for your access logs**  
Be sure to disable access logs before you delete the bucket that you configured for access logs\. Otherwise, if there is a new bucket with the same name and the required bucket policy but created in an AWS account that you don't own, Elastic Load Balancing could write the access logs for your load balancer to this new bucket\.

## Verify bucket permissions<a name="verify-bucket-permissions"></a>

After access logs are enabled for your load balancer, Elastic Load Balancing validates the S3 bucket and creates a test file to ensure that the bucket policy specifies the required permissions\. You can use the Amazon S3 console to verify that the test file was created\. The test file is not an actual access log file; it doesn't contain example records\.

**To verify that Elastic Load Balancing created a test file in your S3 bucket**

1. Open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. Select the name of the bucket that you specified for access logs\.

1. Navigate to the test file, `ELBAccessLogTestFile`, in the following location:

   ```
   my-bucket/prefix/AWSLogs/123456789012/ELBAccessLogTestFile
   ```