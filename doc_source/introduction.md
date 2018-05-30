# What Is an Application Load Balancer?<a name="introduction"></a>

Elastic Load Balancing supports three types of load balancers: Application Load Balancers, Network Load Balancers, and Classic Load Balancers\. This guide discusses Application Load Balancers\. For more information about Network Load Balancers, see the [User Guide for Network Load Balancers](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/)\. For more information about Classic Load Balancers, see the [User Guide for Classic Load Balancers](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/)\.

## Application Load Balancer Components<a name="application-load-balancer-components"></a>

A *load balancer* serves as the single point of contact for clients\. The load balancer distributes incoming application traffic across multiple targets, such as EC2 instances, in multiple Availability Zones\. This increases the availability of your application\. You add one or more listeners to your load balancer\.

A *listener* checks for connection requests from clients, using the protocol and port that you configure, and forwards requests to one or more target groups, based on the rules that you define\. Each rule specifies a target group, condition, and priority\. When the condition is met, the traffic is forwarded to the target group\. You must define a default rule for each listener, and you can add rules that specify different target groups based on the content of the request \(also known as *content\-based routing*\)\.

Each *target group* routes requests to one or more registered targets, such as EC2 instances, using the protocol and port number that you specify\. You can register a target with multiple target groups\. You can configure health checks on a per target group basis\. Health checks are performed on all targets registered to a target group that is specified in a listener rule for your load balancer\.

The following diagram illustrates the basic components\. Notice that each listener contains a default rule, and one listener contains another rule that routes requests to a different target group\. One target is registered with two target groups\.

![\[The components of a basic Application Load Balancer\]](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/images/component_architecture.png)

For more information, see the following documentation:
+ [Load Balancers](application-load-balancers.md)
+ [Listeners](load-balancer-listeners.md)
+ [Target Groups](load-balancer-target-groups.md)

## Application Load Balancer Overview<a name="application-load-balancer-overview"></a>

An Application Load Balancer functions at the application layer, the seventh layer of the Open Systems Interconnection \(OSI\) model\. After the load balancer receives a request, it evaluates the listener rules in priority order to determine which rule to apply, and then selects a target from the target group for the rule action using the round robin routing algorithm\. Note that you can configure listener rules to route requests to different target groups based on the content of the application traffic\. Routing is performed independently for each target group, even when a target is registered with multiple target groups\.

You can add and remove targets from your load balancer as your needs change, without disrupting the overall flow of requests to your application\. Elastic Load Balancing scales your load balancer as traffic to your application changes over time\. Elastic Load Balancing can scale to the vast majority of workloads automatically\.

You can configure health checks, which are used to monitor the health of the registered targets so that the load balancer can send requests only to the healthy targets\.

For more information, see [How Elastic Load Balancing Works](http://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html) in the *Elastic Load Balancing User Guide*\.

## Benefits of Migrating from a Classic Load Balancer<a name="application-load-balancer-benefits"></a>

Using an Application Load Balancer instead of a Classic Load Balancer has the following benefits:
+ Support for path\-based routing\. You can configure rules for your listener that forward requests based on the URL in the request\. This enables you to structure your application as smaller services, and route requests to the correct service based on the content of the URL\.
+ Support for host\-based routing\. You can configure rules for your listener that forward requests based on the host field in the HTTP header\. This enables you to route requests to multiple domains using a single load balancer\.
+ Support for routing requests to multiple applications on a single EC2 instance\. You can register each instance or IP address with the same target group using multiple ports\.
+ Support for registering targets by IP address, including targets outside the VPC for the load balancer\.
+ Support for containerized applications\. Amazon Elastic Container Service \(Amazon ECS\) can select an unused port when scheduling a task and register the task with a target group using this port\. This enables you to make efficient use of your clusters\.
+ Support for monitoring the health of each service independently, as health checks are defined at the target group level and many CloudWatch metrics are reported at the target group level\. Attaching a target group to an Auto Scaling group enables you to scale each service dynamically based on demand\.
+ Access logs contain additional information and are stored in compressed format\.
+ Improved load balancer performance\.

For more information about the features supported by each load balancer type, see [Comparison of Elastic Load Balancing Products](https://aws.amazon.com/elasticloadbalancing/details/#compare)\.

## How to Get Started<a name="application-load-balancer-getting-started"></a>

To create an Application Load Balancer, try one of the following tutorials:
+ [Getting Started with Elastic Load Balancing](http://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/load-balancer-getting-started.html) in the *Elastic Load Balancing User Guide*\.
+ [Tutorial: Use Path\-Based Routing with Your Application Load Balancer](tutorial-load-balancer-routing.md)
+ [Tutorial: Use Microservices as Targets with Your Application Load Balancer](tutorial-target-ecs-containers.md)

## Pricing<a name="application-load-balancer-pricing"></a>

With your load balancer, you pay only for what you use\. For more information, see [Elastic Load Balancing Pricing](https://aws.amazon.com/elasticloadbalancing/pricing/)\.