# Scalable Web Application with ALB and Auto Scaling

<br>

## Table of Contents

1. [Project Overview](#project-overview)
2. [Problem Statement](#problem-statement)
3. [Objectives](#objectives)
4. [Solution Architecture](#solution-architecture)
   - [Architecture Diagram](#architecture-diagram)
5. [AWS Services Used](#aws-services-used)
6. [Architecture Components](#architecture-components)
   - [1. Amazon VPC](#1-amazon-vpc)
   - [2. Internet Gateway](#2-internet-gateway)
   - [3. Public Subnets](#3-public-subnets)
   - [4. Private Application Subnets](#4-private-application-subnets)
   - [5. NAT Gateways](#5-nat-gateways)
   - [6. Application Load Balancer](#6-application-load-balancer)
   - [7. Amazon EC2](#7-amazon-ec2)
   - [8. Auto Scaling Group](#8-auto-scaling-group)
   - [9. Amazon RDS Multi-AZ](#9-amazon-rds-multi-az)
   - [10. Amazon CloudFront](#10-amazon-cloudfront)
   - [11. AWS WAF](#11-aws-waf)
   - [12. Amazon Route 53](#12-amazon-route-53)
   - [13. AWS Identity and Access Management](#13-aws-identity-and-access-management)
   - [14. AWS Systems Manager](#14-aws-systems-manager)
   - [15. Amazon CloudWatch](#15-amazon-cloudwatch)
   - [16. Amazon SNS](#16-amazon-sns)
7. [Route Tables](#route-tables)
   - [Public Subnet Route Table](#public-subnet-route-table)
   - [Private Application Subnet Route Table](#private-application-subnet-route-table)
8. [Traffic Flow](#traffic-flow)
   - [Inbound Application Traffic](#inbound-application-traffic)
   - [Outbound EC2 Traffic](#outbound-ec2-traffic)
9. [High Availability](#high-availability)
10. [Scalability](#scalability)
11. [Security](#security)
    - [Network Security](#network-security)
    - [Security Groups](#security-groups)
    - [AWS WAF](#aws-waf-1)
    - [Private Subnets](#private-subnets)
    - [Administrative Access](#administrative-access)
    - [IAM](#iam)
12. [Monitoring and Notifications](#monitoring-and-notifications)
13. [Conclusion](#conclusion)
14. [Author](#author)

<br>

---

<br>

## Project Overview

<br>

This project demonstrates the design of a secure, highly available, and scalable web application on AWS.

<br>

The application runs on Amazon EC2 instances deployed in private application subnets across two Availability Zones. A single internet-facing Application Load Balancer (ALB) spans the two public subnets and distributes incoming traffic across healthy EC2 instances. The EC2 instances are managed by an Auto Scaling Group (ASG), which adjusts application capacity based on demand.

<br>

Amazon RDS Multi-AZ provides a highly available relational database across private database subnets. Amazon CloudFront improves content delivery by caching content closer to users, while AWS WAF provides protection against common web attacks. Amazon Route 53 provides DNS services for the application.

<br>

The architecture also uses an Internet Gateway to provide internet connectivity to the VPC and NAT Gateways to allow EC2 instances in private subnets to initiate outbound connections without being directly exposed to the internet.

<br>

Security Groups provide resource-level traffic control, while Network Access Control Lists (NACLs) can provide an additional layer of subnet-level protection.

<br>

The overall goal is to build a production-style AWS architecture that remains available, secure, and scalable during changes in traffic and infrastructure failures.

<br>

---

<br>

## Problem Statement

<br>

A company wants to host a web application on AWS that can support changing levels of traffic while remaining highly available and secure.

<br>

The solution should:

<br>

- Avoid a single point of failure.
- Automatically scale when application demand changes.
- Distribute traffic across multiple application servers.
- Keep EC2 application servers and databases in private subnets.
- Provide controlled outbound internet access for private EC2 instances.
- Provide a highly available relational database.
- Protect the web application from common attacks.
- Provide monitoring and notifications.
- Allow administrators to securely manage EC2 instances.

<br>

---

<br>

## Objectives

<br>

The objectives of this project are to:

<br>

- Design a VPC across two Availability Zones.
- Deploy public, private application, and private database subnets.
- Configure an Internet Gateway for VPC internet connectivity.
- Configure NAT Gateways for outbound internet access from private application subnets.
- Deploy EC2 instances inside private application subnets.
- Distribute application traffic using an Application Load Balancer.
- Automatically scale EC2 instances using an Auto Scaling Group.
- Deploy an Amazon RDS Multi-AZ database.
- Improve application security using AWS WAF, Security Groups, and Network ACLs.
- Improve content delivery using Amazon CloudFront.
- Configure DNS using Amazon Route 53.
- Control access using AWS Identity and Access Management (IAM).
- Monitor the environment using Amazon CloudWatch.
- Send alerts using Amazon SNS.
- Provide secure EC2 access using AWS Systems Manager Session Manager.

<br>

---

<br>

## Solution Architecture

<br>

The main inbound application traffic follows this path:

<br>

**Users → Route 53 → CloudFront → AWS WAF → Internet Gateway → Application Load Balancer → EC2 Auto Scaling Group → Amazon RDS**

<br>

The Application Load Balancer is internet-facing and spans the two public subnets. The EC2 application instances are deployed across two private application subnets.

<br>

Amazon RDS is deployed using Multi-AZ across private database subnets.

<br>

For outbound internet connectivity, EC2 instances in the private application subnets use NAT Gateways located in the public subnets:

<br>

**EC2 → NAT Gateway → Internet Gateway → Internet**

<br>

The NAT Gateways are used for outbound connectivity and are not part of the inbound user traffic path.

<br>

### Architecture Diagram

<br>

![AWS Scalable Web Application Architecture](architecture/architecture-diagram.png)

<br>

---

<br>

## AWS Services Used

<br>

| Service | Purpose |
|---|---|
| Amazon VPC | Provides an isolated network for the application |
| Internet Gateway | Provides internet connectivity for resources using public routes |
| Public Subnets | Contain the internet-facing ALB and NAT Gateways |
| Private App Subnets | Contain EC2 application instances |
| Private DB Subnets | Contain the Amazon RDS database |
| Route Tables | Control how traffic is routed within and outside the VPC |
| Network ACLs | Provide subnet-level network traffic control |
| Security Groups | Provide resource-level traffic control |
| NAT Gateway | Provides outbound internet connectivity for EC2 instances in private subnets |
| Amazon EC2 | Hosts the web application |
| Auto Scaling Group | Automatically adjusts EC2 capacity |
| Application Load Balancer | Distributes incoming traffic across healthy EC2 instances |
| AWS WAF | Protects the application from common web attacks |
| Amazon CloudFront | Caches content and improves content delivery |
| Amazon RDS Multi-AZ | Provides a highly available relational database |
| Amazon Route 53 | Provides DNS services |
| AWS IAM | Controls access to AWS resources |
| AWS Systems Manager | Provides secure administrative access to EC2 instances |
| Amazon CloudWatch | Provides monitoring, metrics, dashboards, and alarms |
| Amazon SNS | Sends notifications when alarms are triggered |

<br>

---

<br>

## Architecture Components

<br>

### 1. Amazon VPC

<br>

The application infrastructure is deployed inside an Amazon Virtual Private Cloud (VPC).

<br>

The VPC provides network isolation and allows resources to be separated into public, private application, and private database subnets.

<br>

The architecture spans two Availability Zones to improve availability and reduce dependency on a single Availability Zone.

<br>

---

<br>

### 2. Internet Gateway

<br>

An Internet Gateway is attached to the VPC to provide connectivity between the VPC and the internet.

<br>

The public subnet route tables contain a default route that directs internet-bound traffic to the Internet Gateway.

<br>

```text
0.0.0.0/0 → Internet Gateway
```

<br>

The internet-facing Application Load Balancer and NAT Gateways use the public subnet infrastructure.

<br>

---

<br>

### 3. Public Subnets

<br>

A public subnet is deployed in each Availability Zone.

<br>

The public subnets contain:

<br>

- Application Load Balancer connectivity
- NAT Gateways

<br>

The public subnet route tables have a route to the Internet Gateway.

<br>

The Application Load Balancer spans both public subnets, allowing it to distribute incoming requests to EC2 instances in both Availability Zones.

<br>

---

<br>

### 4. Private Application Subnets

<br>

The EC2 instances are deployed in private application subnets across the two Availability Zones.

<br>

The EC2 instances do not require public IP addresses because users do not connect directly to them.

<br>

Incoming application requests reach the EC2 instances through the Application Load Balancer.

<br>

For outbound internet access, the private application subnet route tables direct internet-bound traffic to a NAT Gateway.

<br>

```text
0.0.0.0/0 → NAT Gateway
```

<br>

---

<br>

### 5. NAT Gateways

<br>

A NAT Gateway is deployed in each public subnet.

<br>

The NAT Gateways allow EC2 instances in private application subnets to initiate outbound internet connections without making those EC2 instances publicly accessible.

<br>

For example, an EC2 instance might need outbound connectivity to download software updates or communicate with an external API.

<br>

The outbound path is:

<br>

```text
EC2 Instance
      ↓
NAT Gateway
      ↓
Internet Gateway
      ↓
Internet
```

<br>

The NAT Gateway is not used for inbound user traffic.

<br>

---

<br>

### 6. Application Load Balancer

<br>

A single internet-facing Application Load Balancer spans the public subnets in both Availability Zones.

<br>

The ALB receives incoming application requests and forwards them to healthy EC2 instances registered in its target group.

<br>

The ALB uses health checks to determine which EC2 instances are healthy and available to receive traffic.

<br>

This prevents the application from depending on a single EC2 instance.

<br>

---

<br>

### 7. Amazon EC2

<br>

Amazon EC2 instances host the web application.

<br>

The EC2 instances are deployed inside private application subnets across two Availability Zones.

<br>

Users do not communicate directly with individual EC2 instances. Application requests pass through the Application Load Balancer.

<br>

---

<br>

### 8. Auto Scaling Group

<br>

The EC2 instances are managed by an Auto Scaling Group.

<br>

Auto Scaling allows application capacity to adjust according to workload demand.

<br>

When demand increases, additional EC2 instances can be launched.

<br>

When demand decreases, unnecessary EC2 instances can be terminated.

<br>

The Auto Scaling Group spans the two private application subnets, helping maintain application availability across both Availability Zones.

<br>

---

<br>

### 9. Amazon RDS Multi-AZ

<br>

Amazon RDS provides the relational database for the application.

<br>

The database is deployed in private database subnets and is not directly accessible from the internet.

<br>

The Multi-AZ configuration maintains database availability by providing a primary database instance and a standby instance in another Availability Zone.

<br>

If the primary database used to grant AWS services and administrators only the permissions required to perform their tasks.

<br>

---

<br>

### 14. AWS Systems Manager

<br>

AWS Systems Manager Session Manager provides secure administrative access to EC2 instances without exposing them directly to the internet or maintaining a traditional bastion host.

<br>

---

<br>

### 15. Amazon CloudWatch

<br>

Amazon CloudWatch provides monitoring for the application and its AWS infrastructure.

<br>

CloudWatch can monitor resources such as:

<br>

- Amazon EC2
- Application Load Balancer
- Auto Scaling Group
- Amazon RDS

<br>

CloudWatch dashboards provide visibility into application and infrastructure health.

<br>

CloudWatch alarms can identify configured conditions that require attention.

<br>

---

<br>

### 16. Amazon SNS

<br>

Amazon Simple Notification Service (SNS) provides notifications.

<br>

CloudWatch alarms can trigger SNS notifications when configured conditions occur, allowing administrators to receive alerts about application or infrastructure issues.

<br>

---

<br>

## Route Tables

<br>

### Public Subnet Route Table

<br>

The public subnets use the Internet Gateway as the destination for internet-bound traffic.

<br>

| Destination | Target |
|---|---|
| VPC CIDR | Local |
| 0.0.0.0/0 | Internet Gateway |

<br>

### Private Application Subnet Route Table

<br>

Private application subnets use a NAT Gateway for outbound internet traffic.

<br>

| Destination | Target |
|---|---|
| VPC CIDR | Local |
| 0.0.0.0/0 | NAT Gateway |

<br>

The architecture uses a NAT Gateway in each Availability Zone so that private application resources can use a NAT Gateway in the corresponding Availability Zone.

<br>

---

<br>

## Traffic Flow

<br>

### Inbound Application Traffic

<br>

A normal user request follows this path:

<br>

1. A user accesses the web application's domain.
2. Amazon Route 53 resolves the application's domain.
3. The request is delivered through Amazon CloudFront.
4. AWS WAF checks the request against configured security rules.
5. The request reaches the internet-facing Application Load Balancer.
6. The Application Load Balancer receives the request.
7. The ALB selects a healthy EC2 instance from its target group.
8. The EC2 instance processes the application request.
9. If database access is required, the EC2 application communicates with Amazon RDS.
10. The response is returned to the user.

<br>

### Outbound EC2 Traffic

<br>

When an EC2 instance in a private application subnet needs internet access:

<br>

1. The EC2 instance initiates the outbound request.
2. The private subnet route table directs the request to the NAT Gateway in the corresponding Availability Zone.
3. The NAT Gateway sends the traffic toward the Internet Gateway.
4. The traffic leaves the VPC for the external destination.
5. Response traffic returns through the established connection.

<br>

---

<br>

## High Availability

<br>

The architecture is designed to avoid dependency on a single EC2 instance or Availability Zone.

<br>

High availability is achieved through:

<br>

- Two Availability Zones.
- Public subnets in both Availability Zones.
- Private application subnets in both Availability Zones.
- Private database subnets in both Availability Zones.
- Multiple EC2 instances.
- Auto Scaling.
- Application Load Balancer health checks.
- NAT Gateways across both Availability Zones.
- Amazon RDS Multi-AZ.

<br>

If an EC2 instance becomes unhealthy, the Application Load Balancer stops routing new application requests to the unhealthy target.

<br>

The Auto Scaling Group maintains the configured application capacity across the private application subnets.

<br>

Amazon RDS Multi-AZ provides database redundancy across Availability Zones.

<br>

---

<br>

## Scalability

<br>

The main scaling mechanism in the architecture is the Auto Scaling Group.

<br>

Scaling policies can increase or decrease EC2 capacity according to application demand.

<br>

For example:

<br>

```text
Traffic increases
       ↓
Application load increases
       ↓
Auto Scaling adds EC2 capacity
       ↓
ALB distributes traffic across healthy instances
```

<br>

When demand decreases, Auto Scaling can reduce unnecessary EC2 capacity.

<br>

CloudFront also reduces the amount of traffic reaching the application infrastructure by serving cached content closer to users.

<br>

---

<br>

## Security

<br>

The architecture applies security at multiple layers.

<br>

### Network Security

<br>

EC2 instances and Amazon RDS are deployed inside private subnets, preventing them from being directly exposed to the internet.

<br>

Network ACLs (NACLs) can provide additional subnet-level protection by controlling inbound and outbound traffic for the public and private subnets.

<br>

### Security Groups

<br>

Security Groups provide resource-level traffic control. The ALB Security Group controls traffic reaching the load balancer, the EC2 Security Group restricts application traffic to the ALB, and the RDS Security Group restricts database access to the application layer.

<br>

Security Groups are stateful, while Network ACLs are stateless and operate at the subnet level. Both can be used together to provide layered network security.

<br>

### AWS WAF

<br>

AWS WAF inspects web requests and provides protection against configured web threats.

<br>

### Private Subnets

<br>

EC2 and database resources remain in private subnets and are not directly exposed to the internet.

<br>

### Administrative Access

<br>

AWS Systems Manager Session Manager provides secure administrative access to EC2 instances without requiring direct inbound administrative access from the internet.

<br>

### IAM

<br>

IAM roles and policies control permissions to AWS resources and services.

<br>

---

<br>

## Monitoring and Notifications

<br>

Amazon CloudWatch monitors the application infrastructure.

<br>

Important resources that can be monitored include:

<br>

- EC2 instances
- Application Load Balancer
- Auto Scaling Group
- Amazon RDS

<br>

CloudWatch dashboards provide visibility into infrastructure health and performance.

<br>

CloudWatch alarms can trigger Amazon SNS notifications when configured conditions are met.

<br>

The monitoring flow is:

<br>

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

<br>

---

<br>

## Conclusion

<br>

This project demonstrates how a web application can be deployed on AWS using a secure, highly available, and scalable architecture.

<br>

The Application Load Balancer distributes incoming application requests across EC2 instances running in private application subnets, while the Auto Scaling Group manages application capacity across two Availability Zones.

<br>

Amazon RDS Multi-AZ provides a highly available database layer in private database subnets. The Internet Gateway provides internet connectivity for resources using public routes, while NAT Gateways provide controlled outbound internet connectivity for EC2 instances located in private application subnets.

<br>

Security Groups control traffic at the resource level, while Network ACLs can provide additional subnet-level network protection.

<br>

CloudFront, AWS WAF, Route 53, IAM, Systems Manager, CloudWatch, and SNS provide supporting content delivery, security, DNS, access control, administration, monitoring, and notification capabilities.

<br>

Overall, the solution demonstrates the core AWS networking, compute, database, security, scalability, and high-availability concepts required to design a production-style web application architecture.

<br>

---

<br>

## Author

<br>

**Johnson Kalibbala**

<br>

AWS Solutions Architect Associate Graduation Project

<br>
