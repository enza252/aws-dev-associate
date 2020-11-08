# Load balancing and auto scaling groups

## Scalability and High Availability

Scalability means that an application can handle greater loads by adapting

Two kinds of scalability
- Vertical scalability
- Horizontal scalability (elasticity)

Scalability is linked but different to high availability

### Vertical Scalability

Increases the size of the instance running the application, adding more resources and computational power.

Example: application runs on t2.micro and want to run on a t2.large.

Standard for non-distributed systems like databases. RDS and ElastiCache services can be scaled vertically by changing instance type.

### Horizontal Scalability

Increase the number of instances for the application, this implies you have distributed systems (web app with back end etc)

AWS enables easy horizontal scaling of EC2s.

This is known as scaling out (increase number of instances) or scaling in (decrease number of instances) on AWS.

Achieved using an Auto Scaling Group or Load Balancer

### High Availability

Running across at least two data centres (availability zones)

The goal of high availability is to survive a data centre loss, therefore providing redundancy for the application.

Highly available systems can be passive or active.

Auto scaling group should be multi AZ
Load balancer should point to instances across multiple AZ


## Load Balancers

A load balancer is a server that will forward traffic to multiple downstream instances.

A load balancer can serve many different instances to spread load. Only a single point of access (DNS) is exposed to the user, this is the hostname of the load balancer.

Seamlessly handles failure of downstream instances through regular health checks to these instances.

Provides SSL termination for websites

Enforce cookie stickiness

High availability across zones

Separate public and private traffic

ELBs are managed, therefore AWS guarantees it will work. AWS perform upgrades, maintenance and so on. AWS provides only a few configuration options.

It costs less to set up your own instance, but it's a lot more work.

### Health Checks

Crucial for load balancers.

Enable load balancer to know if instances it forwards traffic to are available to reply to requests.

The health check is done on a port and a route (/health is common)

If the response is not 200 OK, the instance is marked as 'unhealthy'.

### AWS Load Balancers (ELBs)

Must know these for the exam

Classic Load Balancer (V1)

- Supports HTTP, HTTPS, TCP

Application Load Balancer (V2, 2016)

- Supports HTTP, HTTPS, WebSockets

Network Load Balancer (v2, 2017)

- Supports TCP, TLS (secure TCP) & UDP

Recommended to use the new gen load balancers.

Can set up internal (private) or external (public) ELBs

### Security Groups

Security Groups can be attached to load balancers so that you can receive traffic from the internet. You can create an Application Security Group for your EC2 that _only_ accepts traffic from the Load Balancer.

### Good to know for the exam

- Load balancers can scale but not instantaneously, contact AWS to 'warm up' load balancers in anticipation of high volumes of traffic

Application Load Balancer (V2, 2016)

- Supports HTTP, HTTPS, WebSockets

Network Load Balancer (v2, 2017)

- Supports TCP, TLS (secure TCP) & UDP

Troubleshooting:

- 4xx are client induced errors
- 5xx errors are application induced errors
- Load Balancer Errors 503 means at capacity or no registered target
- If the load balancer cannot connect to the application, check security groups.

Monitoring

ELB access logs will log all access requests (can debug per request)
CloudWatch Metrics will give you aggregate statistics (e.g. connection count)

### Classic Load Balancers

Support TCP (Layer 4), HTTP and HTTPS (Layer 7)

Health checks are TCP or HTTP based

Fixed hostname e.g. `XXX.region.elb.amazonaws.com`

### Application Load Balancer

Support HTTP/HTTPS (Layer 7 only)

Load balancing to multiple http applications across machines (target groups)

Load balancing to multiple applications on the same machine (e.g. containers)

Support for HTTP/2 and WebSocket

Support redirects (e.g. HTTP -> HTTPS)

ALB supports Routing, via routing tables to point to target groups
- Routing can be based on a path in URL e.g. example.com/users
- Based on hostname in URL e.g. one.example.com
- Based on query strings and headers, e.g. example.com/users?id=2131321

ALBs are a great fit for micro-service architectures and container based applications (e.g. Docker, Amazon ECS)

ALBs have a port mapping feature to redirect to a dynamic port in ECS

In comparison, we'd need multiple CLBs per application

#### Target Groups

- Target groups can be EC2 instances (managed by Auto Scaling Group) - HTTP
- ECS tasks (managed by ECS) - HTTP
- Lambda functions - HTTP request is translated in to a JSON event
- IP Addresses - must be private IPs only

ALBs can route to multiple target groups
Health checks are at the target group level

#### Exam Tips

ALBs have a fixed hostname (static DNS)

Application servers don't see the IP of the client directly
- The true IP of the client is inserted in the header `X-Forwarded-For`
- We also get the port `X-Forwarded-Port` 
- and Protocol `X-Forwarded-Proto` - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto

Network Load Balancers expose a public, static IP.



### Network Load Balancers

Layer 4 (TCP/UDP) 
- Forward TCP/UDP to instances
- Can handle millions of requests per second
- Ultra low latency

NLB has one static IP per AZ and supports assigning Elastic IP (useful for whitelisting a specific IP)

NLBs are for extreme performance, TCP or UDP traffic

Not included in the AWS Free Tier.

Target groups can be EC2 instances.

To enable traffic to pass from an NLB to an instance, you must create those rules in the security group attached to the EC2, instead of the load balancer.

You can assign elastic IPs to load balancers, or allow AWS to automatically assign an IPv4 address.

### Load Balancer Stickiness

It's possible to implement stickiness, so that the same client is directed to the same instance behind a load balancer.

This works for classic and application load balancers.

The cookie is used for stickiness and has an expiration date you control

Use case: make sure the user doesn't lose session data

Enabling stickiness may bring imbalance to the load over the backend ec2 instances.

Stickiness is not a load balancer specific configuration - stickiness is handled at the target group level.

### Cross Zone Load Balancing - Good to know for exam

Each load balancer instance evenly distributes traffic across all instances in all AZs

For example, if you have 3 availability zones, each containing 2 ec2s and an application load balancer:

The first load balancer in the first AZ will distribute traffic across all 6 instances (across AZs).

The other load balancers will do the same, across all AZs, therefore evenly distributing traffic.

- Classic Load Balancer

Disabled by default no charges for inter-AZ data if enabled. Usually in AWS, transfer of data across AZ costs.

- ALB

Always on. No charges for inter AZ data.

- NLB

Disabled by default. You pay for inter-AZ data transfer.

### SSL and TLS certificates

SSL certs allow traffic between clients and load balancers to be encrypted whilst in transit (in-flight encryption). SSL refers to Secure Sockets Layer, used to encrypt connections. TLS, Tansport Layer Security, is a newer version. TLS certificates are mainly used, but people still refer to it as SSL.

Public SSL certs are issued by Certificate Authorities (LetsEncrypt, GoDaddy, Digicert etc)

SSL certs have an expiration date and should be renewed regularly.

Load balancers use X.509 certificates (SSL/TLS). They can be managed in AWS using AWS Certificate Manager.

You can upload your own certificates if you want to.

HTTPS Listener:
- Specify a default certificate
- Can add an optional list of certs to support multiple domains.
- Clients can use SNI (Server Name Indication) to specify the hostname they reach.
- Ability to specify a security policy to support SSL (legacy) / TLS 

### Server Name Indication

SNI solves the problem of loading multiple SSL certificates onto one web server, to serve multiple websites.

SNI is a newer protocol that requires the client to indicate the hostname of the target server in the initial SSL handshake.

The server will then find the correct certificate or return the default cert.

Note: only works for ALB & NLB, CloudFront.

DOES NOT WORK FOR CLB (older gen)

#### Good to know for the exam

Classic Load Balancers
- Only support one SSL certificate
- Must use multiple CLB for multiple hostnames with multiple SSL certificates

Application Load Balancers
- Support multiple listeners with multiple SSL certificates
- Uses SNI to make it work

Network Load Balancers
- Supports multiple Listeners with multiple SSL certificates
- Uses SNI to make it work

SNI (Server Name Indication) is a feature allowing you to expose multiple SSL certs if the client supports it. Read more here: https://aws.amazon.com/blogs/aws/new-application-load-balancer-sni/

### ELB Connection Draining - exam tip

This can come up in the exam!

Feature naming differs depending on your Load Balancer
 
Classic Load Balancer
- **connection draining**

ALB and NLB
Target Group: Deregistration Delay

'Connection Draining' is the time to complete "in-flight" requests while the instance is de-registering or unhealthy.

The ELB stops sending new requests to the instance which is deregistering. 

New connections in to the ELB will be directed to OTHER instances. Existing connections will complete and no new connections will be served to the deregistering instance.

Between 1 - 3600 seconds (1 hour), default is 300 seconds.

Can be disabled by setting the value to 0.

If requests are served quickly, set a low value because your requests are short.

If requests are long, set the value higher so that requests are not interrupted and are able to complete.

## Auto Scaling Group

In real life, the load on a website or application can change. More users = greater load.

Auto-scaling groups allow you to scale out to match increased load, as well as to scale in to match a decreased load.

Ensure we have a minimum or maximum number of machines running - good for budget management.

Automatically Register new instances to a load balancer.

ASGs have the following attributes
- A launch configuration
    - AMI + Instance Type
    - EC2 User Data
    - EBS Volumes
    - Security Groups
    - SSH Key Pair
- Min Size / Max Size & initial Capacity (as well as desired capacity)
- Networks and Subnet Information
- Load Balancer + Target Group Information
- Scaling Policies (what triggers scale out/in)

### Auto Scaling Alarms

Can scale an ASG based on CloudWatch Alarms

Alarms monitor a metric (such as average CPU)

Metrics are computed for the OVERALL ASG INSTANCES
   
It's possible to define better auto-scaling rules that are directly managed by EC2
- Target Average CPU usage
- Number of requests on the ELB per instance
- Average Network In
- Average Network Out

Can auto scale based on a custom metric, e.g. number of connected users
1. Send a custom metric from application on EC2 to CloudWatch (PutMetric API)
2. Create CloudWatch alarm to react to low/high values
3. Use the CloudWatch alarm as the scaling policy for ASG

#### ASG Exam Tips

Scaling policies can be on CPU, Network and even based on custom metrics or a schedule (if you know visitor patterns)

ASGs use Launch Configurations or Launch Templates (newer version of launch config)

To update an ASG, you must provide a new launch configuration / launch template. Underlying EC2s can be replaced over time.

Attaching an IAM role to an ASG will automatically assign the IAM role to the EC2.

ASGs are free. You pay for the underlying resources that are launched.

For instances in an ASG, if an instance is terminated, the ASG will realise that this instance has been removed and launch a new instance to replace this, maintaining your desired capacity.

ASGs can terminate instances marked as unhealthy by a load balancer.

### ASG Scaling Policies

3 Kinds of policies:

Target Tracking Scaling
- With target tracking scaling policies, you select a scaling metric and set a target value.
- Example: I want average ASG CPU to stay around 40%. If CPU usage exceeds 40% then we scale out, if below 40%, scale in.

Simple/Step Scaling
- With step scaling and simple scaling, you choose scaling metrics and threshold values for the CloudWatch alarms that trigger the scaling process.
- Based on CloudWatch alarm triggers
- More granular control of what to do when the limit is reached
- When a CloudWatch alarm is triggered (e.g. average CPU of the group exceeds 70%), then specify number of units to add
- Example: if an alarm is triggered and CPU usage is low, specify units to remove.
 
Scheduled Actions
- Anticipate a scaling based on known usage patterns, e.g. increase min capacity between a certain time frame

### Scaling Cooldowns - Exam Tip

The cooldown period helps to ensure that your Auto Scaling group doesn't launch or terminate additional instances before the previous scaling activity takes effect.

In addition to default cooldown for ASG, we can create cooldowns that apply to a specific simple scaling policy.
- A scaling specific cooldown period overrides the default cooldown period

One common use for scaling-specific cooldowns is with a scale-in policy - a policy that terminates instances based on a specific criteria or metric.

Since this policy terminates instances, EC2 Auto Scaling needs less time to determine whether to terminate additional instances.

If the default cooldown period of 300 seconds is too long, you can reduce costs by applying a scaling-specific cooldown period of 180 seconds to the scale in policy.

If your application is scaling up and down multiple times each hour, modify the Auto Scaling Groups cool-down timers and the CloudWatch Alarm Period that triggers the scale in. This is a big sign that your policies may not be appropriate
