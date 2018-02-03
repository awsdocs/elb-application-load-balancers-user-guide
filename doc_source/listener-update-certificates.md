# Update Server Certificates<a name="listener-update-certificates"></a>

When you create a secure listener, you specify a default certificate\. You can also create a certificate list for the listener by adding additional certificates\.

Each certificate comes with a validity period\. You must ensure that you renew or replace the certificate before its validity period ends\. Renewing or replacing a certificate does not affect in\-flight requests that were received by the load balancer node and are pending routing to a healthy target\. After a certificate is renewed, new requests use the renewed certificate\. After a certificate is replaced, new requests use the new certificate\.

You can manage certificate renewal and replacement as follows:

+ Certificates provided by AWS Certificate Manager and deployed on your load balancer can be renewed automatically\. ACM attempts to renew certificates before they expire\. For more information, see [Managed Renewal](http://docs.aws.amazon.com/acm/latest/userguide/acm-renewal.html) in the *AWS Certificate Manager User Guide*\.

+ If you imported a certificate into ACM, you must monitor the expiration date of the certificate and renew it before it expires\. For more information, see [Importing Certificates](http://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html) in the *AWS Certificate Manager User Guide*\.

+ If you imported a certificate into IAM, you must create a new certificate, import the new certificate to ACM or IAM, add the new certificate to your load balancer, and remove the expired certificate from your load balancer\.

## Add Certificates<a name="add-certificates"></a>

You can add certificates to the certificate list for your listener using the following procedure\. The default certificate for a listener is not added to the certificate list by default, but you can add the default certificate to the certificate list\.

**To add certificates using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. For the secure listener to update, choose **View/edit certificates**, which displays the default certificate followed by any other certificates that you've added to the listener\.

1. Choose the **Add certificates** icon \(the plus sign\) in the menu bar, which displays the default certificate followed by any other certificates managed by ACM and IAM\. If you've already added a certificate to the listener, its checkbox is selected and disabled\.

1. To add certificates that are already managed by ACM or IAM, select the certificates and choose **Add**\.

1. If you have a certificate that isn't managed by ACM or IAM, import it to ACM and add it to your listener as follows:

   1. Choose **Import certificate**\.

   1. For **Certificate private key**, paste the PEM\-encoded, unencrypted private key for the certificate\.

   1. For **Certificate body**, paste the PEM\-encoded certificate\.

   1. \(Optional\) For **Certificate chain**, paste the PEM\-encoded certificate chain\.

   1. Choose **Import**\. The newly imported certificate appears in the list of available certificates and is selected\.

   1. Choose **Add**\.

1. To leave this screen, choose the **Back to the load balancer** icon \(the back button\) in the menu bar\.

**To add a certificate using the AWS CLI**  
Use the [add\-listener\-certificate](http://docs.aws.amazon.com/cli/latest/reference/elbv2/add-listener-certificate.html) command\.

## Replace the Default Certificate<a name="replace-default-certificate"></a>

You can replace the default certificate for your listener using the following procedure\.

**To change the default certificate using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. Select the listener and choose **Actions**, **Edit**\.

1. For **Select default certificate**, do one of the following:

   + If you created or imported a certificate using AWS Certificate Manager, select **Choose an existing certificate from AWS Certificate Manager \(ACM\)**, and then select the certificate from **Certificate name**\.

   + If you uploaded a certificate using IAM, select **Choose an existing certificate from AWS Identity and Access Management \(IAM\)**, and then select the certificate from **Certificate name**\.

1. Choose **Save**\.

**To change the default certificate using the AWS CLI**  
Use the [modify\-listener](http://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-listener.html) command\.

## Remove Certificates<a name="remove-certificate"></a>

You can remove the nondefault certificates for a secure listener at any time\. You cannot remove the default certificate for a secure listener using this procedure\.

**To remove certificates using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. For the secure listener to update, choose **View/edit certificates**, which displays the default certificate followed by any other certificates that you've added to the listener\.

1. Choose the **Remove certificates** icon \(the minus sign\) in the menu bar\.

1. Select the certificates and choose **Remove**\.

1. To leave this screen, choose the **Back to the load balancer** icon \(the back button\) in the menu bar\.

**To remove a certificate using the AWS CLI**  
Use the [remove\-listener\-certificate](http://docs.aws.amazon.com/cli/latest/reference/elbv2/remove-listener-certificate.html) command\.