<!--
 Copyright 2025 lesongvi

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     https://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->

# High Availability and Scalability: ELB & ASG

## High Availability and Scalability

- Scalability means that an application / system can handle greater loads by adapting its resources.
- There are two kinds of scalability:
  - **Vertical Scalability**: Adding more power (CPU, RAM) to an existing machine.
  - **Horizontal Scalability**: Adding more machines to the pool of resources.
- **Scalability is linked but different to High Availability**.

### Let's deep dive into the distinction, using a call center as an example

### Vertical Scalability

- Vertical scalability means increasing the size of the instance.
- For example, your application runs on a t2.micro instance, and you need more CPU and RAM.
- Scaling that application vertically means changing the instance type to t2.medium or t2.large.
- Vertical scaling is very common for non distributed systems, such as databases.
- RDS, ElastiCache and Redshift are examples of services that can be scaled vertically.
- There's usually a limit to how much you can vertically scale (hardware limit).
  ![Example junior operator vs. senior operator](/assets/vertical-scalability.png)

### Horizontal Scalability

- Horizontal Scalability means increasing the number of instances / systems for your application.

- Horizontal scaling implies distributed systems.
- This is very common for web applications / modern applications.

- It's easy to horizontally scale thanks the cloud offerings such as Amazon EC2.

![Example call center with multiple operators](/assets/horizontal-scalability.png)

### High Availability

- High Availability usually goes hand in hand with horizontal scaling.
- High availability means running your application / system in at least 2 data centers (== Availability Zones).
- The goal of high availability is to survive a data center failure.

- The high availability can be passive (for RDS Multi AZ for example)
- The high availability can be active (for example, running your application in 2 data centers, horizontally scaled).

![Example call center with multiple operators in different locations](/assets/high-availability.png)

### High Availability & Scalability For EC2

- Vertical Scaling: Increase instance size (= scale up / down)

  - From: t2.nano - 0.5G of RAM, 1 vCPU
  - To: u-12tb1.metal - 12.3TB of RAM, 448 vCPUs

- Horizontal Scaling: Increase number of instances (= scale out / in)

  - Auto Scaling Group
  - Elastic Load Balancer

- High Availability: Run in multiple AZs
  - Auto Scaling Group multi-AZ
  - Elastic Load Balancer multi-AZ

## Elastic Load Balancer (ELB) Overview

### What is load balancing?

- Load Balancers are servers that forward traffic to multiple servers (e.g. EC2 instances) downstream.

![Example of load balancer](/assets/load-balancer.png)

### Why use load balancing?

- Spread load across multiple downstream instances
- Expose a single point of access (DNS name) to your application
- Seamlessly handle failures of downstream instances
- Do regular health checks to your instances
- Provide SSL termination (HTTPS) for your web applications
- Enfore stickiness with cookies
- High availability across zones
- Separate public traffic from private traffic

### Why use an ELB?

- An Elastic Load Balancer is a **managed load balancer**.

  - AWS guarantess that it will be working
  - AWS takes care of upgrades, maintenance, high availability, etc.
  - AWS provides only a few configuration knobs

- It costs less to setup your own load balancer but it will be a lot more effort on your end.

- It is integrated with many AWS offerings / services
  - EC2, EC2 Auto Scaling Groups, Amazon ECS
  - AWS Certificate Manager (ACM), CloudWatch
  - Route 53, AWS WAF, AWS Global Accelerator

### Health Checks

- Health Checks are crucial for Load Balancers.
- They anable the load balancer to know if instances it forwards traffic to are available to reply to requests.
- The health check is done on a port and a route (/health is common)
- If the response is not 200 (OK), then the instance is unhealthy.

### Type of load balancer on AWS

- AWS has **4 kinds of Load Balancers**:

  - <strike>**Classic Load Balancer** (v1 - old generation) - 2009 - CLB
    - HTTP, HTTPS, TCP, SSL (secure TCP)</strike> -> deprecated and will not have in the exam.
  - **Application Load Balancer** (v2 - new generation) - 2016 - ALB
    - HTTP, HTTPS, WebSocket
  - **Network Load Balancer** (v2 - new generation) - 2017 - NLB
    - TCP, TLS (secure TCP), UDP
  - **Gateway Load Balancer** (v2 - new generation) - 2020 - GWLB
    - Operates at layer 3 (Network Layer) - IP Protocol

- Overall, it is recommended to use the newer generation load balancers as they provide more features.
- Some load balancers can be setup as **internal** (private) or **external** (public) ELBs.

### Load Balancer Security Groups

![Load balancer security groups](/assets/load-balancer-security-groups.png)

## Application Load Balancer (ALB)

### Application Load Balancer (v2)

- Application Load Balancer is Layer 7 (HTTP).

- Load balancing to multiple HTTP applications across machines (target groups).
- Load balancing to multiple applications on the same machine (ex. containers)
- Support for HTTP/2 and WebSocket
- Support redirects (from HTTP to HTTPS for example)

- Routing tables to different target groups:

  - Routing based on path in URL (example.com/users & example.com/products)
  - Routing based on hostname in URL (one.example.com & two.example.com)
  - Routing based on Query String, Headers (example.com?user=123)

- ALB are a great fit for microservices & container-based architectures (example. Docker & Amazon ECS)
- Has a port mapping feature to redirect to a dynamic port in ECS
- In comparison, we'd need multiple Classic Load Balancers per application to achieve the same thing.

### Application Load Balancer (ALB) - HTTP Based Traffic

![Application Load Balancer (ALB) - HTTP Based Traffic](/assets/application-load-balancer-http-based-traffic.png)

### Application Load Balancer (ALB) - Target Groups

- EC2 instances (can be managed by an Auto Scaling Group) - HTTP
- ECS tasks (managed by ECS itself) - HTTP
- Lambda functions (serverless) - HTTP request is translated into a JSON event
- IP addresses (on-premises) - must be private IPs

- ALB can route multiple target groups
- Health checks are at the target group level

### Application Load Balancer (ALB) - Query Strings/Parameters Routing

![Application Load Balancer (ALB) - Query Strings/Parameters Routing](/assets/application-load-balancer-query-strings-parameters-routing.png)

### Application Load Balancer (ALB) - Good To Know

- Fixed hostname (XXX.region.elb.amazonaws.com)
- The application servers don't see the IP of the client directly
  - The true IP of the client is inserted in the header X-Forwarded-For
  - We can also get Port (X-Forwarded-Port) and Protocol (X-Forwarded-Proto)

![Application Load Balancer (ALB) - Good To Know](/assets/application-load-balancer-good-to-know.png)

## Network Load Balancer (NLB)

- Network load balancer (Layer 4) allow to:

  - **Forward TCP & UDP traffic to your EC2 instances**
  - Handle millions of requests per second
  - Ultra-low latency

- **NLB has _one static IP address_, and supports assigning Elastic IP** (helpful for whitelisting specific IP)

- NLB are used for extreme performance, TCP or UDP traffic
- Not included in the AWS Free Tier

### Network Load Balancer (NLB) - TCP (Layer 4) Based Traffic

![Network Load Balancer (NLB) - TCP (Layer 4) Based Traffic](/assets/network-load-balancer-tcp-based-traffic.png)

### Network Load Balancer (NLB) - Target Groups

![Network Load Balancer (NLB) - Target Groups](/assets/network-load-balancer-target-groups.png)

## Gateway Load Balancer (GWLB)

![Gateway Load Balancer (GWLB)](/assets/gateway-load-balancer.png)

### Gateway Load Balancer (GWLB) - Target Groups

![Gateway Load Balancer (GWLB) - Target Groups](/assets/gateway-load-balancer-target-groups.png)

## Elastic Load Balancer (ELB) - Sticky Sessions

- It is possible to implement stickiness so that the same client is always redirected to the same instance behind the load balancer.
- This works for **Classic Load Balancer**, **Application Load Balancer** and **Network Load Balancer**.
- The "cookie" used for stickiness has an expiration date you control.
- Use case: make sure the user doesn't lose his session data
- Enabling stickiness may bring imbalance to the load over the backend EC2 instances.
  ![Sticky Sessions](/assets/sticky-sessions.png)

### Sticky Sessions - Cookie Names

- **Application-based Cookies**

  - Custom cookie:
    - Generated by the target
    - Can include any custom attributes required by the application
    - Cookie name must be specified individually for each target group
    - Don't use **AWSALB**, **AWSALBAPP**, or **AWSALBTG** (reserved for ELB)
  - Application cookie
    - Generated by the load balancer
    - Cookie name is **AWSALBAPP**

- **Duration-based Cookies**
  - Cookie generated by the load balancer
  - Cookie name is **AWSALB** for ALB and **AWSELB** for CLB

## Elastic Load Balancer (ELB) - Cross-Zone Load Balancing

![Elastic Load Balancer (ELB) - Cross-Zone Load Balancing](/assets/elastic-load-balancer-cross-zone-load-balancing.png)

### Cross-Zone Load Balancing

- **Application Load Balancer**:
  - Enabled by default (can be disabled at the Target Group level)
  - No charges for inter AZ data transfer
- **Network Load Balancer** & **Gateway Load Balancer**:
  - Disabled by default
  - You pay charges ($) for inter AZ data transfer

## Elastic Load Balancer (ELB) - SSL Certificates

### SSL/TLS - Basics

- An SSL Certificate allows traffic between your clients and your load balancer to be encrypted in transit (in-flight encryption).

- **SSL** refers to Secure Sockets Layer, used to encrypt connections
- **TLS** refers to Transport Layer Security, the successor to SSL
- Nowadays, **TLS certificates are mainly used**, but people still refer to them as SSL certificates.

- Public SSL certificates are issued by a Certificate Authority (CA)
- Comodo, Symantec, GoDaddy, GlobalSign, DigiCert, Let's Encrypt, etc.

- SSL certificates have an expiration date (1 year, 2 years, etc.) and must be renewed

### Load Balancer - SSL Certificates

![Load Balancer - SSL Certificates](/assets/load-balancer-ssl-certificates.png)

- The load balancer uses an X.509 certificate (SSL/TLS server certificate)
- You can manage certificates using ACM (AWS Certificate Manager)
- You can create upload your own certificates alternatively
- HTTPS listener:
  - You must specify a default certificate
  - You can add an optional list of certs to support multiple domains
  - **Clients can use SNI (Server Name Indication) to specify the domain name they are trying to reach**
  - Ability to specify a security policy to support older versions of SSL / TLS (legacy clients)

### SSL - Server Name Indication (SNI)

![SSL - Server Name Indication (SNI)](/assets/ssl-server-name-indication.png)

### Elastic Load Balancer - What Supports SSL?

- **Application Load Balancer** & **Network Load Balancer**:
  - Supports multiple listeners with multiple SSL certificates
  - Uses Server Name Indication (SNI) to support multiple domains

## Elastic Load Balancer - Connection Draining

![Elastic Load Balancer - Connection Draining](/assets/elastic-load-balancer-connection-draining.png)
