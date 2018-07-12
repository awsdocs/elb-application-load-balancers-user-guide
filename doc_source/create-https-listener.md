# HTTPS Listeners for Your Application Load Balancer<a name="create-https-listener"></a>

You can create a listener that uses encrypted connections \(also known as *SSL offload*\)\. This feature enables traffic encryption between your load balancer and the clients that initiate SSL or TLS sessions\.

To use an HTTPS listener, you must deploy an SSL/TLS server certificate on your load balancer\. The load balancer uses this certificate to terminate the connection and then decrypt requests from clients before sending them to the targets\.

Elastic Load Balancing uses a Secure Socket Layer \(SSL\) negotiation configuration, known as a security policy, to negotiate SSL connections between a client and the load balancer\. A security policy is a combination of protocols and ciphers\. The protocol establishes a secure connection between a client and a server and ensures that all data passed between the client and your load balancer is private\. A cipher is an encryption algorithm that uses encryption keys to create a coded message\. Protocols use several ciphers to encrypt data over the internet\. During the connection negotiation process, the client and the load balancer present a list of ciphers and protocols that they each support, in order of preference\. By default, the first cipher on the server's list that matches any one of the client's ciphers is selected for the secure connection\.

Application Load Balancers do not support SSL renegotiation for client or target connections\.

## SSL Certificates<a name="https-listener-certificates"></a>

The load balancer uses an X\.509 certificate \(SSL/TLS server certificate\)\. Certificates are a digital form of identification issued by a certificate authority \(CA\)\. A certificate contains identification information, a validity period, a public key, a serial number, and the digital signature of the issuer\.

When you create a certificate for use with your load balancer, you must specify a domain name\.

We recommend that you create certificates for your load balancer using [AWS Certificate Manager \(ACM\)](https://aws.amazon.com/certificate-manager/)\. ACM integrates with Elastic Load Balancing so that you can deploy the certificate on your load balancer\. For more information, see the [AWS Certificate Manager User Guide](http://docs.aws.amazon.com/acm/latest/userguide/)\.

Alternatively, you can use SSL/TLS tools to create a certificate signing request \(CSR\), then get the CSR signed by a CA to produce a certificate, then import the certificate into ACM or upload the certificate to AWS Identity and Access Management \(IAM\)\. For more information about importing certificates into ACM, see [Importing Certificates](http://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html) in the *AWS Certificate Manager User Guide*\. For more information about uploading certificates to IAM, see [Working with Server Certificates](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_server-certs.html) in the *IAM User Guide*\.

**Important**  
You cannot install certificates with 4096\-bit RSA keys or EC keys on your load balancer through integration with ACM\. You must upload certificates with 4096\-bit RSA keys or EC keys to IAM in order to use them with your load balancer\.

When you create an HTTPS listener, you specify a default certificate\. You can create an optional certificate list for the listener by adding more certificates\. This enables a load balancer to support multiple domains on the same port and provide a different certificate for each domain\. The default certificate for a listener is not added to the certificate list by default\. For more information, see [Update Server Certificates](listener-update-certificates.md)\.

Clients can use the Server Name Identification \(SNI\) protocol extension to specify the hostname they are trying to reach\. If the hostname doesn't match a certificate in the certificate list, the load balancer selects the default certificate\. If the hostname matches a single certificate in the certificate list, the load balancer selects this certificate\. If a hostname provided by a client matches multiple certificates in the certificate list, the load balancer selects the best certificate that the client can support\. Certificate selection is based on the following criteria in the following order:
+ Public key algorithm \(prefer ECDSA over RSA\)
+ Hashing algorithm \(prefer SHA over MD5\)
+ Key length \(prefer the largest\)
+ Validity period

The load balancer access log entries indicate the hostname specified by the client and the certificate presented to the client\. For more information, see [Access Log Entries](load-balancer-access-logs.md#access-log-entry-format)\.

## Security Policies<a name="describe-ssl-policies"></a>

You can choose the security policy that is used for front\-end connections\. The `ELBSecurityPolicy-2016-08` security policy is always used for backend connections\. Application Load Balancers do not support custom security policies\.

Elastic Load Balancing provides the following security policies for Application Load Balancers:
+ `ELBSecurityPolicy-2016-08`
+ `ELBSecurityPolicy-FS-2018-06`
+ `ELBSecurityPolicy-TLS-1-2-2017-01`
+ `ELBSecurityPolicy-TLS-1-2-Ext-2018-06`
+ `ELBSecurityPolicy-TLS-1-1-2017-01`
+ `ELBSecurityPolicy-2015-05`
+ `ELBSecurityPolicy-TLS-1-0-2015-04`

We recommend the `ELBSecurityPolicy-2016-08` policy for general use\. You can use the `ELBSecurityPolicy-FS-2018-06` policy if you require Forward Secrecy \(FS\)\. You can use one of the `ELBSecurityPolicy-TLS` policies to meet compliance and security standards that require disabling certain TLS protocol versions, or to support legacy clients that require deprecated ciphers\. Only a small percentage of internet clients require TLS version 1\.0\. To view the TLS protocol version for requests to your load balancer, enable access logging for your load balancer and examine the access logs\. For more information, see [Access Logs](load-balancer-access-logs.md)\.

The following table describes the security policies defined for Application Load Balancers\.


| Security Policy | 2016\-08 \* | FS\-2018\-06 | TLS\-1\-2 | TLS\-1\-2\-Ext | TLS\-1\-1 | TLS\-1\-0 † | 
| --- | --- | --- | --- | --- | --- | --- | 
| TLS Protocols | 
| Protocol\-TLSv1 | ♦ | ♦ |  |  |  | ♦ | 
| Protocol\-TLSv1\.1 | ♦ | ♦ |  |  | ♦ | ♦ | 
| Protocol\-TLSv1\.2 | ♦ | ♦ | ♦ | ♦ | ♦ | ♦ | 
| TLS Ciphers | 
| ECDHE\-ECDSA\-AES128\-GCM\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES128\-GCM\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES128\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES128\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES128\-SHA | ♦ | ♦ |  | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES128\-SHA | ♦ | ♦ |  | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES256\-GCM\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES256\-GCM\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES256\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES256\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES256\-SHA | ♦ | ♦ |  | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES256\-SHA | ♦ | ♦ |  | ♦ | ♦ | ♦ | 
| AES128\-GCM\-SHA256 | ♦ |  | ♦ | ♦ | ♦ | ♦ | 
| AES128\-SHA256 | ♦ |  | ♦ | ♦ | ♦ | ♦ | 
| AES128\-SHA | ♦ |  |  | ♦ | ♦ | ♦ | 
| AES256\-GCM\-SHA384 | ♦ |  | ♦ | ♦ | ♦ | ♦ | 
| AES256\-SHA256 | ♦ |  | ♦ | ♦ | ♦ | ♦ | 
| AES256\-SHA | ♦ |  |  | ♦ | ♦ | ♦ | 
| DES\-CBC3\-SHA |  |  |  |  |  | ♦ | 

\* The `ELBSecurityPolicy-2016-08` and `ELBSecurityPolicy-2015-05` security policies for Application Load Balancers are identical\.

† Do not use this security policy unless you must support a legacy client that requires the DES\-CBC3\-SHA cipher, which is a weak cipher\.

To view the configuration of a security policy for Application Load Balancers using the AWS CLI, use the [describe\-ssl\-policies](http://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-ssl-policies.html) command\.

## Update the Security Policy<a name="update-security-policy"></a>

When you create an HTTPS listener, you can select the security policy that meets your needs\. When a new security policy is added, you can update your HTTPS listener to use the new security policy\. Application Load Balancers do not support custom security policies\.

**To update the security policy using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select the load balancer and choose **Listeners**\.

1. Select the check box for the HTTPS listener and choose **Edit**\.

1. For **Security policy**, choose a security policy\.

1. Choose **Update**\.

**To update the security policy using the AWS CLI**  
Use the [modify\-listener](http://docs.aws.amazon.com/cli/latest/reference/elbv2/modify-listener.html) command\.