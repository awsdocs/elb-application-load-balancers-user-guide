# Update an HTTPS listener for your Application Load Balancer<a name="listener-update-certificates"></a>

After you create an HTTPS listener, you can replace the default certificate, update the certificate list, or replace the security policy\.

**Topics**
+ [Replace the default certificate](#replace-default-certificate)
+ [Add certificates to the certificate list](#add-certificates)
+ [Remove certificates from the certificate list](#remove-certificates)
+ [Update the security policy](#update-security-policy)

## Replace the default certificate<a name="replace-default-certificate"></a>

You can replace the default certificate for your listener using the following procedure\. For more information, see [SSL certificates](create-https-listener.md#https-listener-certificates)\.

**To change the default certificate using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Listeners** tab, select the text in the **Protocol:Port** column to open the detail page for the listener\.

1. On the **Certificates** tab, choose **Change default**\.

1. For **ACM and IAM certificates**, select a certificate\.

1. Choose **Save as default**\.

**To change the default certificate using the AWS CLI**  
Use the [modify\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-listener.html) command\.

## Add certificates to the certificate list<a name="add-certificates"></a>

You can add certificates to the certificate list for your listener using the following procedure\. When you first create an HTTPS listener, the certificate list is empty\. You can add one or more certificates\. You can optionally add the default certificate to ensure that this certificate is used with the SNI protocol even if it is replaced as the default certificate\. For more information, see [SSL certificates](create-https-listener.md#https-listener-certificates)\.

**To add certificates to the certificate list using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Listeners** tab, select the text in the **Protocol:Port** column to open the detail page for the listener\.

1. On the **Certificates** tab, choose **Add certificate**\.

1. For **ACM and IAM certificates**, select the certificates and choose **Include as pending below**\.

1. If you have a certificate that isn't managed by ACM or IAM, choose **Import certificate**, complete the form, and choose **Import**\.

1. Choose **Add pending certificates**\.

**To add a certificate to the certificate list using the AWS CLI**  
Use the [add\-listener\-certificates](https://docs.aws.amazon.com/cli/latest/reference/elbv2/add-listener-certificates.html) command\.

## Remove certificates from the certificate list<a name="remove-certificates"></a>

You can remove certificates from the certificate list for an HTTPS listener using the following procedure\. To remove the default certificate for an HTTPS listener, see [Replace the default certificate](#replace-default-certificate)\.

**To remove certificates from the certificate list using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Listeners** tab, select the text in the **Protocol:Port** column to open the detail page for the listener\.

1. On the **Certificates** tab, select the check boxes for the certificates and choose **Remove**\.

1. When prompted for confirmation, enter **confirm** and choose **Remove**\.

**To remove a certificate from the certificate list using the AWS CLI**  
Use the [remove\-listener\-certificates](https://docs.aws.amazon.com/cli/latest/reference/elbv2/remove-listener-certificates.html) command\.

## Update the security policy<a name="update-security-policy"></a>

When you create an HTTPS listener, you can select the security policy that meets your needs\. When a new security policy is added, you can update your HTTPS listener to use the new security policy\. Application Load Balancers do not support custom security policies\. For more information, see [Security policies](create-https-listener.md#describe-ssl-policies)\.

**To update the security policy using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, choose **Load Balancers**\.

1. Select the load balancer\.

1. On the **Listeners** tab, select the text in the **Protocol:Port** column to open the detail page for the listener\.

1. On the **Details** tab, choose **Edit**\.

1. For **Security policy**, choose a security policy\.

1. Choose **Save changes**\.

**To update the security policy using the AWS CLI**  
Use the [modify\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-listener.html) command\.