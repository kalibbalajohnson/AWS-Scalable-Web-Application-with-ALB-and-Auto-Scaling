# Scalable Web Application with ALB and Auto Scaling

A secure, highly available, and scalable web application on AWS. EC2 instances run in private application subnets across two Availability Zones (AZs) behind a single internet-facing Application Load Balancer (ALB). An Auto Scaling Group (ASG) adjusts capacity to demand, and Amazon RDS Multi-AZ provides the database. CloudFront, AWS WAF, and Route 53 handle content delivery, protection, and DNS.

## Goals

The solution must:

- Avoid single points of failure and scale automatically with demand.
- Distribute traffic across multiple application servers.
- Keep EC2 and the database in private subnets, with controlled outbound internet access.
- Protect the application from common web attacks.
- Provide monitoring, alerting, and secure administrative access.

## Architecture

![AWS Scalable Web Application Architecture](architecture/architecture-diagram.png)

**Inbound traffic**

```text
Users → Route 53 → CloudFront → AWS WAF → Internet Gateway → ALB → EC2 Auto Scaling Group → RDS
```

**Outbound traffic** (not part of the user request path)

```text
EC2 → NAT Gateway → Internet Gateway → Internet
```

1. Route 53 resolves the domain, and the request is delivered through CloudFront.
2. AWS WAF checks the request against its security rules.
3. The request enters the VPC through the Internet Gateway and reaches the ALB.
4. The ALB selects a healthy EC2 instance, which queries RDS if needed.
5. The response returns to the user.

## AWS Services

| Service | Role |
|---|---|
| **VPC** | Isolated network across two AZs, split into public, private app, and private DB subnets |
| **Internet Gateway** | Connects the VPC to the internet for resources with public routes |
| **Public subnets** | One per AZ. Hold the ALB and NAT Gateways |
| **Private app subnets** | One per AZ. Hold EC2 instances, which have no public IPs |
| **Private DB subnets** | One per AZ. Hold RDS, with no direct internet access |
| **NAT Gateways** | One per AZ. Give private EC2 instances outbound-only internet access (updates, external APIs) |
| **Application Load Balancer** | One internet-facing ALB spanning both public subnets. Uses health checks to route only to healthy targets |
| **EC2 + Auto Scaling Group** | Hosts the app. The ASG spans both private app subnets and adds or removes instances with demand |
| **RDS Multi-AZ** | Primary and standby in different AZs, with automatic failover |
| **CloudFront** | Caches content near users, reducing latency and load on the app |
| **AWS WAF** | Inspects web requests and blocks common attacks |
| **Route 53** | DNS for the application's domain |
| **IAM** | Grants services and administrators only the permissions they need |
| **Systems Manager** | Session Manager gives secure EC2 access with no bastion host or open inbound ports |
| **CloudWatch** | Metrics, dashboards, and alarms for EC2, ALB, ASG, and RDS |
| **SNS** | Sends notifications when CloudWatch alarms trigger |

## Route Tables

| Subnet | Destination | Target |
|---|---|---|
| Public | VPC CIDR | Local |
| Public | 0.0.0.0/0 | Internet Gateway |
| Private app | VPC CIDR | Local |
| Private app | 0.0.0.0/0 | NAT Gateway (same AZ) |

## High Availability and Scalability

- **Two AZs:** public, private app, and private DB subnets exist in both, with a NAT Gateway in each.
- **Health checks:** the ALB stops sending traffic to unhealthy instances.
- **Auto Scaling:** the ASG maintains the configured capacity across both AZs, adding instances as load rises and removing them as it falls.
- **Database failover:** RDS Multi-AZ switches to the standby if the primary fails.
- **Caching:** CloudFront serves cached content, reducing traffic to the application tier.

## Security

- **Private subnets:** EC2 and RDS are not reachable directly from the internet.
- **Security groups:** EC2 accepts application traffic only from the ALB, and RDS accepts database traffic only from the application layer.
- **AWS WAF:** filters malicious web requests.
- **Session Manager:** administrators reach instances without exposing them to the internet.
- **IAM:** roles and policies limit permissions.

## Monitoring

```text
AWS Resources → CloudWatch → Alarm → SNS → Notification
```

## Outcomes

This project demonstrates a highly available, scalable, secure, fault-tolerant, and monitored architecture, with separate public, application, and database layers across two AZs. It shows practical skills in:

- VPC design, subnets, route tables, Internet Gateways, and NAT Gateways
- ALB configuration and health checks
- EC2 Auto Scaling and RDS Multi-AZ
- CloudFront, AWS WAF, Route 53, and IAM
- Session Manager, CloudWatch, and SNS

## Author

**Johnson Kalibbala**
AWS Solutions Architect Associate Graduation Project
