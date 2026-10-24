# Scalable Web Application with ALB and Auto Scaling

## Project Overview

This project demonstrates the design of a secure, highly available, and scalable web application on AWS.

The application runs on Amazon EC2 instances deployed across two Availability Zones. An Application Load Balancer (ALB) distributes incoming traffic across the EC2 instances, while an Auto Scaling Group (ASG) automatically adjusts the number of instances based on application demand.

Amazon RDS Multi-AZ provides a highly available relational database. Amazon CloudFront improves content delivery, AWS WAF protects the application from common web attacks, and Amazon Route 53 provides DNS services.

The overall goal is to build a production-style AWS architecture that can remain available during increased traffic or infrastructure failures.

---

## Problem Statement

A company wants to host a web application on AWS that can support changing levels of traffic while remaining highly available.

The solution should:

- Avoid a single point of failure.
- Automatically scale when application demand changes.
- Distribute traffic across multiple application servers.
- Keep application servers and databases away from direct internet access.
- Provide a highly available database.
- Protect the web application from common attacks.
- Provide monitoring and notifications.
- Allow administrators to securely manage EC2 instances.

---

## Objectives

The objectives of this project are to:

- Design a VPC across multiple Availability Zones.
- Deploy public and private subnets.
- Deploy EC2 instances inside private subnets.
- Distribute application traffic using an Application Load Balancer.
- Automatically scale EC2 instances using an Auto Scaling Group.
- Deploy an Amazon RDS Multi-AZ database.
- Improve application security using AWS WAF and Security Groups.
- Improve content delivery using Amazon CloudFront.
- Configure DNS using Amazon Route 53.
- Monitor the environment using Amazon CloudWatch.
- Send alerts using Amazon SNS.
- Provide secure EC2 access using AWS Systems Manager Session Manager.

---

## Solution Architecture

The architecture follows this general traffic flow:

**Users → Route 53 → CloudFront → AWS WAF → Application Load Balancer → EC2 Auto Scaling Group → Amazon RDS**

The EC2 instances and database are placed inside private subnets, while internet-facing components are placed in public subnets.

### Architecture Diagram

![AWS Scalable Web Application Architecture](architecture/architecture-diagram.png)

---

## AWS Services Used

| Service | Purpose |
|---|---|
| Amazon VPC | Provides an isolated network for the application |
| Public Subnets | Host internet-facing resources |
| Private Subnets | Host application and database resources |
| NAT Gateway | Provides outbound internet connectivity for private resources |
| Amazon EC2 | Hosts the web application |
| Auto Scaling Group | Automatically adjusts EC2 capacity |
| Application Load Balancer | Distributes traffic across healthy EC2 instances |
| AWS WAF | Protects the application from common web attacks |
| Amazon CloudFront | Provides content caching and improves content delivery |
| Amazon RDS Multi-AZ | Provides the relational database and automated failover |
| Amazon Route 53 | Provides DNS services and health checks |
| AWS Systems Manager | Provides secure administrative access to EC2 instances |
| Amazon CloudWatch | Provides monitoring, metrics, dashboards, and alarms |
| Amazon SNS | Sends notifications when alarms are triggered |

---

## Architecture Components

### 1. Amazon VPC

The application is deployed inside an Amazon Virtual Private Cloud (VPC).

The VPC provides network isolation and allows the application infrastructure to be divided into public and private subnets.

The architecture spans two Availability Zones to improve availability.

---

### 2. Public and Private Subnets

The architecture separates resources according to whether they require direct internet access.

Public subnets are used for internet-facing infrastructure.

Private subnets contain resources that should not be directly accessible from the internet, including:

- EC2 application instances
- Amazon RDS database

This separation reduces unnecessary exposure of the application infrastructure.

---

### 3. Application Load Balancer

The Application Load Balancer receives incoming application requests and distributes them across healthy EC2 instances.

The ALB uses target groups and health checks to determine which EC2 instances are healthy and available to receive traffic.

This prevents users from depending on a single application server.

---

### 4. Amazon EC2

Amazon EC2 instances host the web application.

The EC2 instances are deployed in private subnets across multiple Availability Zones.

Users do not communicate directly with individual EC2 instances. Requests are sent through the Application Load Balancer.

---

### 5. Auto Scaling Group

The EC2 instances are managed by an Auto Scaling Group.

Auto Scaling allows the application infrastructure to adjust according to workload demand.

When demand increases, additional EC2 capacity can be added.

When demand decreases, unnecessary capacity can be removed.

This helps the application maintain performance while avoiding unnecessary infrastructure usage.

---

### 6. Amazon RDS Multi-AZ

Amazon RDS provides the relational database for the application.

A Multi-AZ deployment is used to improve database availability.

The database is deployed inside private subnets so that it is not directly accessible from the internet.

If the primary database becomes unavailable, the Multi-AZ configuration provides automated failover.

---

### 7. Amazon CloudFront

Amazon CloudFront provides content delivery for the application.

CloudFront can cache static content closer to users, reducing the amount of traffic that needs to travel directly to the application infrastructure.

This can improve application performance for geographically distributed users.

---

### 8. AWS WAF

AWS WAF provides an additional security layer for the web application.

WAF rules can be used to protect the application against common web attacks, including attacks covered by OWASP security protections.

---

### 9. Amazon Route 53

Amazon Route 53 provides DNS services for the application.

Users access the application through its domain name, and Route 53 directs requests toward the AWS application infrastructure.

Health checks can also be used as part of monitoring application availability.

---

### 10. AWS Systems Manager

AWS Systems Manager Session Manager provides administrators with secure access to EC2 instances.

This removes the need to expose the EC2 instances directly to the internet for administrative access and provides a bastion-free method of managing the servers.

---

### 11. Amazon CloudWatch

Amazon CloudWatch provides monitoring for the AWS environment.

CloudWatch can monitor metrics from resources such as:

- EC2
- Application Load Balancer
- Auto Scaling
- Amazon RDS

CloudWatch dashboards provide visibility into application and infrastructure health.

CloudWatch alarms can also identify conditions that require attention.

---

### 12. Amazon SNS

Amazon Simple Notification Service (SNS) provides notifications.

CloudWatch alarms can send notifications through SNS when configured conditions occur.

This allows administrators to receive alerts when there are application or infrastructure problems.

---

## Traffic Flow

A normal user request follows the process below:

1. A user accesses the web application's domain.
2. Amazon Route 53 resolves the domain.
3. The request is delivered through Amazon CloudFront.
4. AWS WAF checks the request against configured security rules.
5. The request is forwarded to the Application Load Balancer.
6. The ALB selects a healthy EC2 instance.
7. The EC2 instance processes the application request.
8. If data is required, the application communicates with Amazon RDS.
9. Amazon RDS processes the database request.
10. The result is returned through the application to the user.

---

## High Availability

The n a single server or Availability Zone.

High availability is achieved through:

- Multiple Availability Zones.
- Multiple EC2 instances.
- Application Load Balancer health checks.
- Auto Scaling.
- Amazon RDS Multi-AZ.
- Route 53 health checks.

If one EC2 instance becomes unhealthy, the load balancer can stop sending requests to that instance.

The Auto Scaling Group can maintain the required application capacity by managing EC2 instances across the configured Availability Zones.

---

## Scalability

The main scaling mechanism in the architecture is the Auto Scaling Group.

Scaling policies can be configured to increase or decrease EC2 capacity according to application demand.

For example:

```text
Traffic increases
       ↓
Application demand increases
       ↓
Auto Scaling adds EC2 capacity
       ↓
ALB distributes traffic across the available instances
```

When application demand decreases, Auto Scaling can reduce unnecessary EC2 capacity.

---

## Security

The architecture uses several security layers.

### Network Security

EC2 instances and Amazon RDS are deployed inside private subnets.

This prevents these resources from being directly exposed to the internet.

### Security Groups

Security Groups control communication between resources.

For example:

```text
Internet
   ↓
CloudFront / WAF
   ↓
Application Load Balancer
   ↓
EC2 Instances
   ↓
Amazon RDS
```

Each layer should only allow the communication required by the application.

### AWS WAF

AWS WAF protects the application's web-facing layer from common web attacks.

### Administrative Access

AWS Systems Manager Session Manager provides secure access to EC2 instances without requiring direct internet access to those instances.

---

## Monitoring and Notifications

Amazon CloudWatch is used to monitor the application infrastructure.

Important resources that can be monitored include:

- EC2 instances
- Application Load Balancer
- Auto Scaling Group
- Amazon RDS

CloudWatch dashboards provide visibility into the infrastructure.

CloudWatch alarms can detect configured conditions and use Amazon SNS to send notifications.

The monitoring flow is:

```text
AWS Resources
      ↓
Amazon CloudWatch
      ↓
CloudWatch Alarm
      ↓
Amazon SNS
      ↓
Notification
```

---

## Expected Result

After completing this project, the solution should demonstrate a web application architecture that is:

- Highly available
- Scalable
- Secure
- Fault tolerant
- Monitored
- Distributed across multiple Availability Zones

The completed architecture should demonstrate practical understanding of:

- VPC design
- Public and private subnets
- Route tables
- NAT Gateway
- Application Load Balancer
- ALB health checks
- EC2
- Auto Scaling
- Amazon RDS Multi-AZ
- Amazon CloudFront
- AWS WAF
- Amazon Route 53
- AWS Systems Manager
- Amazon CloudWatch
- Amazon SNS

---

## Key Learning Outcomes

By completing this project, I gained practical understanding of how AWS services can be combined to build a production-style web application architecture.

The project covers:

- Designing a VPC with public and private subnets.
- Building a multi-AZ architecture.
- Configuring load balancing and health checks.
- Understanding EC2 Auto Scaling.
- Designing a highly available database layer.
- Protecting an application using AWS WAF and Security Groups.
- Using private subnets to reduce direct internet exposure.
- Using Systems Manager Session Manager for secure server access.
- Monitoring AWS resources using CloudWatch.
- Using SNS for notifications.

---

## Conclusion

This project demonstrates how a traditional web application can be deployed on AWS using a highly available and scalable architecture.

The Application Load Balancer distributes user requests across EC2 instances, while the Auto Scaling Group manages application capacity across multiple Availability Zones.

Amazon RDS Multi-AZ provides a highly available database layer, while CloudFront, AWS WAF, Route 53, CloudWatch, SNS, and Systems Manager provide supporting content delivery, security, DNS, monitoring, notification, and administration capabilities.

Overall, the solution demonstrates several important concepts required when designing secure, scalable, and highly available applications on AWS.

---

## Author

**Johnson Kalibbala**

AWS Solutions Architect Associate Graduation Project
