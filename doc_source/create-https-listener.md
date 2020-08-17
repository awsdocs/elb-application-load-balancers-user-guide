# Create an HTTPS listener for your Application Load Balancer<a name="create-https-listener"></a>

A listener is a process that checks for connection requests\. You define a listener when you create your load balancer, and you can add listeners to your load balancer at any time\.

You can create an HTTPS listener, which uses encrypted connections \(also known as *SSL offload*\)\. This feature enables traffic encryption between your load balancer and the clients that initiate SSL or TLS sessions\.

The information on this page helps you create an HTTPS listener for your load balancer\. To add an HTTP listener to your load balancer, see [Create an HTTP listener for your Application Load Balancer](create-listener.md)\.

**Contents**
+ [SSL certificates](#https-listener-certificates)
  + [Default certificate](#default-certificate)
  + [Certificate list](#sni-certificate-list)
  + [Certificate renewal](#ssl-certificate-renewal)
+ [Security policies](#describe-ssl-policies)
+ [Add an HTTPS listener](#add-https-listener)
+ [Update an HTTPS listener](#update-https-listener)

## SSL certificates<a name="https-listener-certificates"></a>

To use an HTTPS listener, you must deploy at least one SSL/TLS server certificate on your load balancer\. The load balancer uses a server certificate to terminate the front\-end connection and then decrypt requests from clients before sending them to the targets\.

The load balancer requires X\.509 certificates \(SSL/TLS server certificates\)\. Certificates are a digital form of identification issued by a certificate authority \(CA\)\. A certificate contains identification information, a validity period, a public key, a serial number, and the digital signature of the issuer\.

When you create a certificate for use with your load balancer, you must specify a domain name\.

We recommend that you create certificates for your load balancer using [AWS Certificate Manager \(ACM\)](https://aws.amazon.com/certificate-manager/)\. ACM integrates with Elastic Load Balancing so that you can deploy the certificate on your load balancer\. For more information, see the [AWS Certificate Manager User Guide](https://docs.aws.amazon.com/acm/latest/userguide/)\.

**Important**  
ACM supports RSA certificates with a 4096 key length and EC certificates\. However, you cannot install these certificates on your load balancer through integration with ACM\. You must upload these certificates to IAM in order to use them with your load balancer\.

Alternatively, you can use SSL/TLS tools to create a certificate signing request \(CSR\), then get the CSR signed by a CA to produce a certificate, then import the certificate into ACM or upload the certificate to AWS Identity and Access Management \(IAM\)\. For more information about importing certificates into ACM, see [Importing certificates](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html) in the *AWS Certificate Manager User Guide*\. For more information about uploading certificates to IAM, see [Working with server certificates](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_server-certs.html) in the *IAM User Guide*\.

### Default certificate<a name="default-certificate"></a>

When you create an HTTPS listener, you must specify exactly one certificate\. This certificate is known as the *default certificate*\. You can replace the default certificate after you create the HTTPS listener\. For more information, see [Replace the default certificate](listener-update-certificates.md#replace-default-certificate)\.

If you specify additional certificates in a [certificate list](#sni-certificate-list), the default certificate is used only if a client connects without using the Server Name Indication \(SNI\) protocol to specify a hostname or if there are no matching certificates in the certificate list\.

If you do not specify additional certificates but need to host multiple secure applications through a single load balancer, you can use a wildcard certificate or add a Subject Alternative Name \(SAN\) for each additional domain to your certificate\.

### Certificate list<a name="sni-certificate-list"></a>

After you create an HTTPS listener, it has a default certificate and an empty certificate list\. You can optionally add certificates to the certificate list for the listener\. Using a certificate list enables the load balancer to support multiple domains on the same port and provide a different certificate for each domain\. For more information, see [Add certificates to the certificate list](listener-update-certificates.md#add-certificates)\.

The load balancer uses a smart certificate selection algorithm with support for SNI\. If the hostname provided by a client matches a single certificate in the certificate list, the load balancer selects this certificate\. If a hostname provided by a client matches multiple certificates in the certificate list, the load balancer selects the best certificate that the client can support\. Certificate selection is based on the following criteria in the following order:
+ Public key algorithm \(prefer ECDSA over RSA\)
+ Hashing algorithm \(prefer SHA over MD5\)
+ Key length \(prefer the largest\)
+ Validity period

The load balancer access log entries indicate the hostname specified by the client and the certificate presented to the client\. For more information, see [Access log entries](load-balancer-access-logs.md#access-log-entry-format)\.

### Certificate renewal<a name="ssl-certificate-renewal"></a>

Each certificate comes with a validity period\. You must ensure that you renew or replace each certificate for your load balancer before its validity period ends\. This includes the default certificate and certificates in a certificate list\. Renewing or replacing a certificate does not affect in\-flight requests that were received by the load balancer node and are pending routing to a healthy target\. After a certificate is renewed, new requests use the renewed certificate\. After a certificate is replaced, new requests use the new certificate\.

You can manage certificate renewal and replacement as follows:
+ Certificates provided by AWS Certificate Manager and deployed on your load balancer can be renewed automatically\. ACM attempts to renew certificates before they expire\. For more information, see [Managed renewal](https://docs.aws.amazon.com/acm/latest/userguide/acm-renewal.html) in the *AWS Certificate Manager User Guide*\.
+ If you imported a certificate into ACM, you must monitor the expiration date of the certificate and renew it before it expires\. For more information, see [Importing certificates](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html) in the *AWS Certificate Manager User Guide*\.
+ If you imported a certificate into IAM, you must create a new certificate, import the new certificate to ACM or IAM, add the new certificate to your load balancer, and remove the expired certificate from your load balancer\.

## Security policies<a name="describe-ssl-policies"></a>

Elastic Load Balancing uses a Secure Socket Layer \(SSL\) negotiation configuration, known as a security policy, to negotiate SSL connections between a client and the load balancer\. A security policy is a combination of protocols and ciphers\. The protocol establishes a secure connection between a client and a server and ensures that all data passed between the client and your load balancer is private\. A cipher is an encryption algorithm that uses encryption keys to create a coded message\. Protocols use several ciphers to encrypt data over the internet\. During the connection negotiation process, the client and the load balancer present a list of ciphers and protocols that they each support, in order of preference\. By default, the first cipher on the server's list that matches any one of the client's ciphers is selected for the secure connection\.

Application Load Balancers do not support SSL renegotiation for client or target connections\.

When you create a TLS listener, you must select a security policy\. You can update the security policy as needed\. For more information, see [Update the security policy](listener-update-certificates.md#update-security-policy)\.

You can choose the security policy that is used for front\-end connections\. The `ELBSecurityPolicy-2016-08` security policy is always used for backend connections\. Application Load Balancers do not support custom security policies\.

Elastic Load Balancing provides the following security policies for Application Load Balancers:
+ `ELBSecurityPolicy-2016-08` \(default\)
+ `ELBSecurityPolicy-TLS-1-0-2015-04`
+ `ELBSecurityPolicy-TLS-1-1-2017-01`
+ `ELBSecurityPolicy-TLS-1-2-2017-01`
+ `ELBSecurityPolicy-TLS-1-2-Ext-2018-06`
+ `ELBSecurityPolicy-FS-2018-06`
+ `ELBSecurityPolicy-FS-1-1-2019-08`
+ `ELBSecurityPolicy-FS-1-2-2019-08`
+ `ELBSecurityPolicy-FS-1-2-Res-2019-08`
+ `ELBSecurityPolicy-2015-05` \(identical to `ELBSecurityPolicy-2016-08`\)

We recommend the `ELBSecurityPolicy-2016-08` policy for compatibility\. You can use one of the `ELBSecurityPolicy-FS` policies if you require Forward Secrecy \(FS\)\. You can use one of the `ELBSecurityPolicy-TLS` policies to meet compliance and security standards that require disabling certain TLS protocol versions, or to support legacy clients that require deprecated ciphers\. Only a small percentage of internet clients require TLS version 1\.0\. To view the TLS protocol version for requests to your load balancer, enable access logging for your load balancer and examine the access logs\. For more information, see [Access Logs](load-balancer-access-logs.md)\.

The following table describes the default policy, `ELBSecurityPolicy-2016-08`, and the `ELBSecurityPolicy-TLS` polices\. The `ELBSecurityPolicy-` has been removed from policy names in the heading row so that they fit\.


| Security policy | 2016\-08 | TLS\-1\-0\-2015\-04 † | TLS\-1\-1\-2017\-01 | TLS\-1\-2\-2017\-01 | TLS\-1\-2\-Ext\-2018\-06 | 
| --- |--- |--- |--- |--- |--- |
| **TLS Protocols** | 
| --- |
| Protocol\-TLSv1 | ♦ | ♦ |  |  |  | 
| Protocol\-TLSv1\.1 | ♦ | ♦ | ♦ |  |  | 
| Protocol\-TLSv1\.2 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| **TLS Ciphers** | 
| --- |
| ECDHE\-ECDSA\-AES128\-GCM\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES128\-GCM\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES128\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES128\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES128\-SHA | ♦ | ♦ | ♦ |  | ♦ | 
| ECDHE\-RSA\-AES128\-SHA | ♦ | ♦ | ♦ |  | ♦ | 
| ECDHE\-ECDSA\-AES256\-GCM\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES256\-GCM\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES256\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES256\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES256\-SHA | ♦ | ♦ | ♦ |  | ♦ | 
| ECDHE\-ECDSA\-AES256\-SHA | ♦ | ♦ | ♦ |  | ♦ | 
| AES128\-GCM\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| AES128\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| AES128\-SHA | ♦ | ♦ | ♦ |  | ♦ | 
| AES256\-GCM\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| AES256\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| AES256\-SHA | ♦ | ♦ | ♦ |  | ♦ | 
| DES\-CBC3\-SHA |  | ♦ |  |  |  | 

† Do not use this policy unless you must support a legacy client that requires the DES\-CBC3\-SHA cipher, which is a weak cipher\.

The following table describes the default policy, `ELBSecurityPolicy-2016-08`, and the `ELBSecurityPolicy-FS` policies\. The `ELBSecurityPolicy-` has been removed from policy names in the heading row so that they fit\.


| Security policy | 2016\-08 | FS\-2018\-06 | FS\-1\-1\-2019\-08 | FS\-1\-2\-2019\-08 | FS\-1\-2\-Res\-2019\-08 | 
| --- |--- |--- |--- |--- |--- |
| **TLS Protocols** | 
| --- |
| Protocol\-TLSv1 | ♦ | ♦ |  |  |  | 
| Protocol\-TLSv1\.1 | ♦ | ♦ | ♦ |  |  | 
| Protocol\-TLSv1\.2 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| **TLS Ciphers** | 
| --- |
| ECDHE\-ECDSA\-AES128\-GCM\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES128\-GCM\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES128\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES128\-SHA256 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES128\-SHA | ♦ | ♦ | ♦ | ♦ |  | 
| ECDHE\-RSA\-AES128\-SHA | ♦ | ♦ | ♦ | ♦ |  | 
| ECDHE\-ECDSA\-AES256\-GCM\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES256\-GCM\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-ECDSA\-AES256\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES256\-SHA384 | ♦ | ♦ | ♦ | ♦ | ♦ | 
| ECDHE\-RSA\-AES256\-SHA | ♦ | ♦ | ♦ | ♦ |  | 
| ECDHE\-ECDSA\-AES256\-SHA | ♦ | ♦ | ♦ | ♦ |  | 
| AES128\-GCM\-SHA256 | ♦ |  |  |  |  | 
| AES128\-SHA256 | ♦ |  |  |  |  | 
| AES128\-SHA | ♦ |  |  |  |  | 
| AES256\-GCM\-SHA384 | ♦ |  |  |  |  | 
| AES256\-SHA256 | ♦ |  |  |  |  | 
| AES256\-SHA | ♦ |  |  |  |  | 

To view the configuration of a security policy for Application Load Balancers using the AWS CLI, use the [describe\-ssl\-policies](https://docs.aws.amazon.com/cli/latest/reference/elbv2/describe-ssl-policies.html) command\.

## Add an HTTPS listener<a name="add-https-listener"></a>

You configure a listener with a protocol and a port for connections from clients to the load balancer, and a target group for the default listener rule\. For more information, see [Listener configuration](load-balancer-listeners.md#listener-configuration)\.

**Prerequisites**
+ To add a forward action to the default listener rule, you must specify an available target group\. For more information, see [Create a target group](create-target-group.md)\.
+ To create an HTTPS listener, you must specify a certificate and a security policy\. The load balancer uses the certificate to terminate the connection and decrypt requests from clients before routing them to targets\. The load balancer uses the security policy when negotiating SSL connections with the clients\.

**To add an HTTPS listener using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select a load balancer, and choose **Listeners**, **Add listener**\.

1. For **Protocol : port**, choose **HTTPS** and keep the default port or enter a different port\.

1. \(Optional\) To authenticate users, for **Default actions**, choose **Add action**, **Authenticate** and provide the requested information\. To save the action, choose the checkmark icon\. For more information, see [Authenticate users using an Application Load Balancer](listener-authenticate-users.md)\.

1. For **Default actions**, do one of the following:
   + Choose **Add action**, **Forward to** and choose a target group\.
   + Choose **Add action**, **Redirect to** and provide the URL for the redirect\. For more information, see [Redirect actions](load-balancer-listeners.md#redirect-actions)\.
   + Choose **Add action**, **Return fixed response** and provide a response code and optional response body\. For more information, see [Fixed\-response actions](load-balancer-listeners.md#fixed-response-actions)\.

   To save the action, choose the checkmark icon\.

1. For **Security policy**, we recommend that you keep the default security policy\.

1. For **Default SSL certificate**, do one of the following:
   + If you created or imported a certificate using AWS Certificate Manager, choose **From ACM** and choose the certificate\.
   + If you uploaded a certificate using IAM, choose **From IAM** and choose the certificate\.

1. Choose **Save**\.

1. \(Optional\) To define additional listener rules that forward requests based on a path pattern or a hostname, see [Add a rule](listener-update-rules.md#add-rule)\.

1. \(Optional\) To add a certificate list for use with the SNI protocol, see [Add certificates to the certificate list](listener-update-certificates.md#add-certificates)\.

**To add an HTTPS listener using the AWS CLI**  
Use the [create\-listener](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-listener.html) command to create the listener and default rule, and the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) command to define additional listener rules\.

## Update an HTTPS listener<a name="update-https-listener"></a>

After you create an HTTPS listener, you can replace the default certificate, update the certificate list, or replace the security policy\. For more information, see [Update an HTTPS listener for your Application Load Balancer](listener-update-certificates.md)\.