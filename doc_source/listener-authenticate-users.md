# Authenticate users using an Application Load Balancer<a name="listener-authenticate-users"></a>

You can configure an Application Load Balancer to securely authenticate users as they access your applications\. This enables you to offload the work of authenticating users to your load balancer so that your applications can focus on their business logic\.

The following use cases are supported:
+ Authenticate users through an identity provider \(IdP\) that is OpenID Connect \(OIDC\) compliant\.
+ Authenticate users through well\-known social IdPs, such as Amazon, Facebook, or Google, through the user pools supported by Amazon Cognito\.
+ Authenticate users through corporate identities, using SAML, LDAP, or Microsoft AD, through the user pools supported by Amazon Cognito\.

## Prepare to use an OIDC\-compliant IdP<a name="oidc-requirements"></a>

Do the following if you are using an OIDC\-compliant IdP with your Application Load Balancer:
+ Create a new OIDC app in your IdP\. You must configure a client ID and a client secret\.
+ Get the following endpoints published by the IdP: authorization, token, and user info\. You can locate this information in the well\-known config\.
+ Allow one of the following redirect URLs in your IdP app, whichever your users will use, where DNS is the domain name of your load balancer and CNAME is the DNS alias for your application:
  + https://*DNS*/oauth2/idpresponse
  + https://*CNAME*/oauth2/idpresponse

## Prepare to use Amazon Cognito<a name="cognito-requirements"></a>

Do the following if you are using Amazon Cognito user pools with your Application Load Balancer:
+ Create a user pool\. For more information, see [Amazon Cognito user pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html) in the *Amazon Cognito Developer Guide*\.
+ Create a user pool client\. You must configure the client to generate a client secret, use code grant flow, and support the same OAuth scopes that the load balancer uses\. For more information, see [Configuring a user pool app client](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-client-apps.html) in the *Amazon Cognito Developer Guide*\.
+ Create a user pool domain\. For more information, see [Adding a Domain name for your user pool](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-domain.html) in the *Amazon Cognito Developer Guide*\.
+ Verify that the requested scope returns an ID token\. For example, the default scope, `openid` returns an ID token but the `aws.cognito.signin.user.admin` scope does not\.
+ To federate with a social or corporate IdP, enable the IdP in the federation section\. For more information, see [Add social sign\-in to a user pool](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-configuring-federation-with-social-idp.html) or [Add sign\-in with a SAML IdP to a user pool](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-configuring-federation-with-saml-2-0-idp.html) in the *Amazon Cognito Developer Guide*\.
+ Allow the following redirect URLs in the callback URL field for Amazon Cognito, where DNS is the domain name of your load balancer, and CNAME is the DNS alias for your application \(if you are using one\):
  + https://*DNS*/oauth2/idpresponse
  + https://*CNAME*/oauth2/idpresponse
+ Allow your user pool domain on your IdP app's callback URL\. Use the format for your IdP\. For example:
  + https://*domain\-prefix*\.auth\.*region*\.amazoncognito\.com/saml2/idpresponse
  + https://*user\-pool\-domain*/oauth2/idpresponse

To enable an IAM user to configure a load balancer to use Amazon Cognito to authenticate users, you must grant the user permission to call the `cognito-idp:DescribeUserPoolClient` action\.

## Prepare to use Amazon CloudFront<a name="cloudfront-requirements"></a>

Enable the following settings if you are using a CloudFront distribution in front of your Application Load Balancer:
+ Forward request headers \(all\) — Ensures that CloudFront does not cache responses for authenticated requests\. This prevents them from being served from the cache after the authentication session expires\. Alternatively, to reduce this risk while caching is enabled, owners of a CloudFront distribution can set the time\-to\-live \(TTL\) value to expire before the authentication cookie expires\.
+ Query string forwarding and caching \(all\) — Ensures that the load balancer has access to the query string parameters required to authenticate the user with the IdP\.
+ Cookie forwarding \(all\) — Ensures that CloudFront forwards all authentication cookies to the load balancer\.

## Configure user authentication<a name="configure-user-authentication"></a>

You configure user authentication by creating an authenticate action for one or more listener rules\. The `authenticate-cognito` and `authenticate-oidc` action types are supported only with HTTPS listeners\. For descriptions of the corresponding fields, see [AuthenticateCognitoActionConfig](https://docs.aws.amazon.com/elasticloadbalancing/latest/APIReference/API_AuthenticateCognitoActionConfig.html) and [AuthenticateOidcActionConfig](https://docs.aws.amazon.com/elasticloadbalancing/latest/APIReference/API_AuthenticateOidcActionConfig.html) in the *Elastic Load Balancing API Reference version 2015\-12\-01*\.

The load balancer sends a session cookie to the client to maintain authentication status\. This cookie always contains the `secure` attribute, because user authentication requires an HTTPS listener\. This cookie contains the `SameSite=None` attribute with CORS \(cross\-origin resource sharing\) requests\.

Application Load Balancers do not support cookie values that are URL encoded\.

By default, the `SessionTimeout` field is set to 7 days\. If you want shorter sessions, you can configure a session timeout as short as 1 second\. For more information, see [Authentication logout and session timeout](#authentication-logout-timeout)\.

Set the `OnUnauthenticatedRequest` field as appropriate for your application\. For example:
+ **Applications that require the user to log in using a social or corporate identity**—This is supported by the default option, `authenticate`\. If the user is not logged in, the load balancer redirects the request to the IdP authorization endpoint and the IdP prompts the user to log in using its user interface\.
+ **Applications that provide a personalized view to a user that is logged in or a general view to a user that is not logged in**—To support this type of application, use the `allow` option\. If the user is logged in, the load balancer provides the user claims and the application can provide a personalized view\. If the user is not logged in, the load balancer forwards the request without the user claims and the application can provide the general view\.
+ **Single\-page applications with JavaScript that loads every few seconds**—By default, after the authentication session cookie expires, the AJAX calls are redirected to the IdP and are blocked\. If you use the `deny` option, the load balancer returns an HTTP 401 Unauthorized error to these AJAX calls\.

The load balancer must be able to communicate with the IdP token endpoint \(`TokenEndpoint`\) and the IdP user info endpoint \(`UserInfoEndpoint`\)\. Verify that the security groups for your load balancer and the network ACLs for your VPC allow outbound access to these endpoints\. Verify that your VPC has internet access\. If you have an internal\-facing load balancer, use a NAT gateway to enable the load balancer to access these endpoints\.

Use the following [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/elbv2/create-rule.html) command to configure user authentication\.

```
aws elbv2 create-rule --listener-arn listener-arn --priority 10 \
--conditions Field=path-pattern,Values="/login" --actions file://actions.json
```

The following is an example of the `actions.json` file that specifies an `authenticate-oidc` action and a `forward` action\.

```
[{
    "Type": "authenticate-oidc",
    "AuthenticateOidcConfig": {
        "Issuer": "https://idp-issuer.com",
        "AuthorizationEndpoint": "https://authorization-endpoint.com",
        "TokenEndpoint": "https://token-endpoint.com",
        "UserInfoEndpoint": "https://user-info-endpoint.com",
        "ClientId": "abcdefghijklmnopqrstuvwxyz123456789",
        "ClientSecret": "123456789012345678901234567890",
        "SessionCookieName": "my-cookie",
        "SessionTimeout": 3600,
        "Scope": "email",
        "AuthenticationRequestExtraParams": {
            "display": "page",
            "prompt": "login"
        },
        "OnUnauthenticatedRequest": "deny"
    },
    "Order": 1
},
{
    "Type": "forward",
    "TargetGroupArn": "arn:aws:elasticloadbalancing:region-code:account-id:targetgroup/target-group-name/target-group-id",
    "Order": 2
}]
```

The following is an example of the `actions.json` file that specifies an `authenticate-cognito` action and a `forward` action\.

```
[{
    "Type": "authenticate-cognito",
    "AuthenticateCognitoConfig": {
        "UserPoolArn": "arn:aws:cognito-idp:region-code:account-id:userpool/user-pool-id",
        "UserPoolClientId": "abcdefghijklmnopqrstuvwxyz123456789",
        "UserPoolDomain": "userPoolDomain1",
        "SessionCookieName": "my-cookie",
        "SessionTimeout": 3600,
        "Scope": "email",
        "AuthenticationRequestExtraParams": {
            "display": "page",
            "prompt": "login"
        },
        "OnUnauthenticatedRequest": "deny"
    },
    "Order": 1
},
{
    "Type": "forward",
    "TargetGroupArn": "arn:aws:elasticloadbalancing:region-code:account-id:targetgroup/target-group-name/target-group-id",
    "Order": 2
}]
```

For more information, see [Listener rules](load-balancer-listeners.md#listener-rules)\.

## Authentication flow<a name="authentication-flow"></a>

Elastic Load Balancing uses the OIDC authorization code flow, which includes the following steps\.

1. When the conditions for a rule with an authenticate action are met, the load balancer checks for an authentication session cookie in the request headers\. If the cookie is not present, the load balancer redirects the user to the IdP authorization endpoint so that the IdP can authenticate the user\.

1. After the user is authenticated, the IdP redirects the user back to the load balancer with an authorization grant code\. The load balancer presents the code to the IdP token endpoint to get the ID token and access token\.

1. After the load balancer validates the ID token, it exchanges the access token with the IdP user info endpoint to get the user claims\.

1. The load balancer creates the authentication session cookie and sends it to the client so that the client's user agent can send the cookie to the load balancer when making requests\. Because most browsers limit a cookie to 4K in size, the load balancer shards a cookie that is greater than 4K in size into multiple cookies\. If the total size of the user claims and access token received from the IdP is greater than 11K bytes in size, the load balancer returns an HTTP 500 error to the client and increments the `ELBAuthUserClaimsSizeExceeded` metric\.

1. The load balancer sends the user claims to the target in HTTP headers\. For more information, see [User claims encoding and signature verification](#user-claims-encoding)\.

1. If the IdP provides a valid refresh token in the ID token, the load balancer saves the refresh token and uses it to refresh the user claims each time the access token expires, until the session times out or the IdP refresh fails\. If the user logs out, the refresh fails and the load balancer redirects the user to the IdP authorization endpoint\. This enables the load balancer to drop sessions after the user logs out\. For more information, see [Authentication logout and session timeout](#authentication-logout-timeout)\.

## User claims encoding and signature verification<a name="user-claims-encoding"></a>

After your load balancer authenticates a user successfully, it sends the user claims received from the IdP to the target\. The load balancer signs the user claim so that applications can verify the signature and verify that the claims were sent by the load balancer\.

The load balancer adds the following HTTP headers:

`x-amzn-oidc-accesstoken`  
The access token from the token endpoint, in plain text\.

`x-amzn-oidc-identity`  
The subject field \(`sub`\) from the user info endpoint, in plain text\.

`x-amzn-oidc-data`  
The user claims, in JSON web tokens \(JWT\) format\.

Applications that require the full user claims can use any standard JWT library to verify the JWT tokens\. These tokens follow the JWT format but are not ID tokens\. The JWT format includes a header, payload, and signature that are base64 URL encoded and includes padding characters at the end\. The JWT signature is ECDSA \+ P\-256 \+ SHA256\.

The JWT header is a JSON object with the following fields:

```
{
   "alg": "algorithm",
   "kid": "12345678-1234-1234-1234-123456789012",
   "signer": "arn:aws:elasticloadbalancing:region-code:account-id:loadbalancer/app/load-balancer-name/load-balancer-id", 
   "iss": "url",
   "client": "client-id",
   "exp": "expiration"
}
```

The JWT payload is a JSON object that contains the user claims received from the IdP user info endpoint\.

```
{
   "sub": "1234567890",
   "name": "name",
   "email": "alias@example.com",
   ...
}
```

Because the load balancer does not encrypt the user claims, we recommend that you configure the target group to use HTTPS\. If you configure your target group to use HTTP, be sure to restrict the traffic to your load balancer using security groups\. We also recommend that you verify the signature before doing any authorization based on the claims\. To get the public key, get the key ID from the JWT header and use it to look up the public key from the following regional endpoint:

```
https://public-keys.auth.elb.region.amazonaws.com/key-id
```

For AWS GovCloud \(US\-West\), the endpoint is as follows:

```
https://s3-us-gov-west-1.amazonaws.com/aws-elb-public-keys-prod-us-gov-west-1/key-id
```

For AWS GovCloud \(US\-East\), the endpoint is as follows:

```
https://s3-us-gov-east-1.amazonaws.com/aws-elb-public-keys-prod-us-gov-east-1/key-id
```

The following example shows how to get the public key in Python 3\.x:

```
import jwt
import requests
import base64
import json

# Step 1: Get the key id from JWT headers (the kid field)
encoded_jwt = headers.dict['x-amzn-oidc-data']
jwt_headers = encoded_jwt.split('.')[0]
decoded_jwt_headers = base64.b64decode(jwt_headers)
decoded_jwt_headers = decoded_jwt_headers.decode("utf-8")
decoded_json = json.loads(decoded_jwt_headers)
kid = decoded_json['kid']

# Step 2: Get the public key from regional endpoint
url = 'https://public-keys.auth.elb.' + region + '.amazonaws.com/' + kid
req = requests.get(url)
pub_key = req.text

# Step 3: Get the payload
payload = jwt.decode(encoded_jwt, pub_key, algorithms=['ES256'])
```

The following example shows how to get the public key in Python 2\.7:

```
import jwt
import requests
import base64
import json

# Step 1: Get the key id from JWT headers (the kid field)
encoded_jwt = headers.dict['x-amzn-oidc-data']
jwt_headers = encoded_jwt.split('.')[0]
decoded_jwt_headers = base64.b64decode(jwt_headers)
decoded_json = json.loads(decoded_jwt_headers)
kid = decoded_json['kid']

# Step 2: Get the public key from regional endpoint
url = 'https://public-keys.auth.elb.' + region + '.amazonaws.com/' + kid
req = requests.get(url)
pub_key = req.text

# Step 3: Get the payload
payload = jwt.decode(encoded_jwt, pub_key, algorithms=['ES256'])
```

## Authentication logout and session timeout<a name="authentication-logout-timeout"></a>

When an application needs to log out an authenticated user, it should set the expiration time of the authentication session cookie to \-1 and redirect the client to the IdP logout endpoint \(if the IdP supports one\)\. To prevent users from reusing a deleted cookie, we recommend that you configure as short an expiration time for the access token as is reasonable\. If a client provides a load balancer with a session cookie that has an expired access token with a non\-NULL refresh token, the load balancer contacts the IdP to determine whether the user is still logged in\.

The refresh token and the session timeout work together as follows:
+ If the session timeout is shorter than the access token expiration, the load balancer honors the session timeout\. If the user has an active session with the IdP, the user might not be prompted to log in again\. Otherwise, the user is redirected to log in\.
+ If the session timeout is longer than the access token expiration and the IdP does not support refresh tokens, the load balancer keeps the authentication session until it times out and then has the user log in again\.
+ If the session timeout is longer than the access token expiration and the IdP supports refresh tokens, the load balancer refreshes the user session each time the access token expires\. The load balancer has the user log in again only after the authentication session times out or the refresh flow fails\.