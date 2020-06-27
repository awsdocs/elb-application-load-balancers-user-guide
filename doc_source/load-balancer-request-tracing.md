# Request tracing for your Application Load Balancer<a name="load-balancer-request-tracing"></a>

You can use request tracing to track HTTP requests from clients to targets or other services\. When the load balancer receives a request from a client, it adds or updates the **X\-Amzn\-Trace\-Id** header before sending the request to the target\. Any services or applications between the load balancer and the target can also add or update this header\.

If you enable access logs, the contents of the **X\-Amzn\-Trace\-Id** header are logged\. For more information, see [Access logs for your Application Load Balancer](load-balancer-access-logs.md)\.

## Syntax<a name="request-tracing-syntax"></a>

The **X\-Amzn\-Trace\-Id** header contains fields with the following format:

```
Field=version-time-id
```

*Field*  
The name of the field\. The supported values are `Root` and `Self`\.  
An application can add arbitrary fields for its own purposes\. The load balancer preserves these fields but does not use them\.

*version*  
The version number\.

*time*  
The epoch time, in seconds\.

*id*  
The trace identifier\.

**Examples**  
If the **X\-Amzn\-Trace\-Id** header is not present on an incoming request, the load balancer generates a header with a `Root` field and forwards the request\. For example:

```
X-Amzn-Trace-Id: Root=1-67891233-abcdef012345678912345678
```

If the **X\-Amzn\-Trace\-Id** header is present and has a `Root` field, the load balancer inserts a `Self` field and forwards the request\. For example:

```
X-Amzn-Trace-Id: Self=1-67891234-12456789abcdef012345678;Root=1-67891233-abcdef012345678912345678
```

If an application adds a header with a `Root` field and a custom field, the load balancer preserves both fields, inserts a `Self` field, and forwards the request:

```
X-Amzn-Trace-Id: Self=1-67891234-12456789abcdef012345678;Root=1-67891233-abcdef012345678912345678;CalledFrom=app
```

If the **X\-Amzn\-Trace\-Id** header is present and has a `Self` field, the load balancer updates the value of the `Self` field\.

## Limitations<a name="request-tracing-limits"></a>
+ The load balancer updates the header when it receives an incoming request, not when it receives a response\.
+ If the HTTP headers are greater than 7 KB, the load balancer rewrites the **X\-Amzn\-Trace\-Id** header with a `Root` field\.
+ With WebSockets, you can trace only until the upgrade request is successful\.