# AWS Secure Healthcare Infrastructure

A production-inspired AWS infrastructure project designed to demonstrate secure cloud networking, Linux administration, database deployment, IAM, monitoring, and infrastructure troubleshooting.

The environment supports a simulated healthcare web application while applying principles such as network segmentation, least-privilege access, private database connectivity, and infrastructure monitoring.

> **Note:** This is a portfolio/lab environment. It does not contain real patient information, PHI, or production healthcare data.

---

## Architecture

![AWS Secure Healthcare Infrastructure Architecture](architecture/architecture.png)

### Architecture Flow

```text
Internet
    ↓
Internet Gateway
    ↓
Public Subnets
    ↓
EC2 Web Server
    ↓
MySQL / TCP 3306
    ↓
Private Amazon RDS
```

The environment uses a custom VPC spanning two Availability Zones with separate public and private database subnets.

The EC2 web server runs Amazon Linux and Apache in a public subnet, while the MySQL database is hosted on Amazon RDS in the private database tier.

---

## Project Objectives

The goal was to design and manually deploy an AWS environment that demonstrates practical cloud infrastructure skills beyond certification-level knowledge.

The project focuses on:

* Designing a custom VPC and subnet architecture
* Separating public-facing and private resources
* Deploying and administering a Linux EC2 web server
* Deploying a private MySQL database using Amazon RDS
* Restricting communication through Security Groups
* Using IAM roles instead of static AWS credentials
* Implementing infrastructure monitoring with Amazon CloudWatch
* Validating application-to-database connectivity
* Troubleshooting real configuration and authentication failures

---

## AWS Services & Technologies

| Technology         | Purpose                                                       |
| ------------------ | ------------------------------------------------------------- |
| Amazon VPC         | Provides the isolated network environment                     |
| Public Subnets     | Hosts resources requiring direct internet connectivity        |
| Private Subnets    | Provides isolated network placement for the database tier     |
| Internet Gateway   | Provides internet connectivity to the VPC                     |
| Route Tables       | Controls network routing for the public subnet tier           |
| Amazon EC2         | Hosts the Linux/Apache web server                             |
| Amazon RDS         | Hosts the managed MySQL database                              |
| Security Groups    | Provides stateful network access control                      |
| AWS IAM            | Provides role-based AWS permissions to EC2                    |
| Amazon CloudWatch  | Provides infrastructure monitoring and alarms                 |
| CloudWatch Agent   | Collects guest OS metrics such as memory and disk utilization |
| Amazon Linux 2023  | Operating system for the EC2 instance                         |
| Apache HTTP Server | Provides the web service                                      |
| MySQL              | Database engine used by the application tier                  |

---

## Network Design

The infrastructure uses a custom VPC:

```text
VPC
10.0.0.0/16
```

The VPC spans two Availability Zones and contains four subnets:

```text
Availability Zone A
├── Public Subnet 1
└── Private DB Subnet 1

Availability Zone B
├── Public Subnet 2
└── Private DB Subnet 2
```

Public and private resources are separated to reduce unnecessary exposure to the internet.

The private database subnets also provide the multi-Availability-Zone subnet coverage required by the RDS DB subnet group.

---

## Security Design

### EC2 Web Tier

The EC2 instance uses a dedicated Security Group.

Inbound access is restricted to:

```text
HTTP / TCP 80
Source: Internet

SSH / TCP 22
Source: Administrator IP only
```

SSH is not exposed to `0.0.0.0/0`, reducing unnecessary administrative exposure.

### RDS Database Tier

Amazon RDS is configured without public accessibility.

The RDS Security Group permits:

```text
MySQL / TCP 3306
Source: EC2 web-server Security Group
```

The database therefore accepts MySQL traffic from the application tier rather than from arbitrary public IP addresses.

### IAM

The EC2 instance uses an IAM role to obtain the AWS permissions required for CloudWatch.

Static AWS access keys are not stored on the EC2 instance.

---

## Monitoring

Amazon CloudWatch provides infrastructure monitoring for the environment.

A CloudWatch alarm monitors EC2 CPU utilization, while the CloudWatch Agent collects additional guest operating-system metrics including:

* Memory utilization
* Disk utilization

The EC2 IAM role provides the permissions required for the CloudWatch Agent to publish metrics.

---

## Infrastructure Validation

The environment was validated from the EC2 instance by connecting directly to the private RDS MySQL endpoint.

A test database and table were created and queried successfully, confirming:

* DNS resolution
* VPC connectivity
* Security Group configuration
* TCP 3306 connectivity
* MySQL authentication
* Database read/write functionality

This verified the complete EC2-to-RDS communication path rather than relying only on AWS resource status indicators.

---

## Troubleshooting Experience

Several issues encountered during deployment were intentionally documented rather than omitted.

### RDS Authentication Failure

**Symptom**

```text
Access denied for user 'ec2-user'
```

**Cause**

The Linux `ec2-user` account was mistakenly used as the MySQL username.

**Resolution**

The connection was retried using the RDS master database account, successfully authenticating to MySQL.

**Lesson**

Operating-system identities and database identities are independent authentication domains.

---

### CloudWatch Agent Configuration Failure

**Symptom**

The CloudWatch Agent failed configuration validation.

An initial generated configuration also referenced a missing `collectd` dependency. During subsequent configuration, JSON validation returned:

```text
Expecting ':' delimiter
```

**Resolution**

The unnecessary collectd configuration was removed and a minimal CloudWatch Agent configuration was created for memory and disk metrics.

The JSON configuration was independently validated with:

```bash
python3 -m json.tool /opt/aws/amazon-cloudwatch-agent/etc/cloudwatch-agent.json
```

After correcting the configuration, the CloudWatch Agent started successfully.

**Lesson**

Configuration errors should be isolated using logs and validation tools before changing unrelated infrastructure.

---

## Key Skills Demonstrated

* AWS networking
* VPC and subnet design
* CIDR planning
* Routing
* Linux administration
* EC2 administration
* Security Groups
* IAM roles
* RDS administration
* MySQL connectivity
* CloudWatch monitoring
* Infrastructure troubleshooting
* Security and least-privilege design
* Technical documentation

---

## Future Improvements

The current architecture represents Version 1 of the environment.

Future iterations could include:

* Application Load Balancer
* Auto Scaling Group
* Private EC2 application instances
* HTTPS using AWS Certificate Manager
* Route 53 DNS
* AWS Systems Manager for administrative access
* AWS Secrets Manager for database credentials
* Infrastructure as Code using Terraform
* CI/CD deployment pipeline
* Expanded CloudWatch dashboards and alerting

These improvements are intentionally separated from Version 1 so the repository accurately represents the infrastructure that has actually been deployed.

---

## Repository Structure

```text
aws-secure-healthcare-infrastructure/
│
├── README.md
├── architecture/
│   ├── architecture.drawio
│   └── architecture.png
├── screenshots/
├── docs/
│   ├── deployment-guide.md
│   ├── architecture.md
│   ├── security-decisions.md
│   ├── troubleshooting.md
│   └── lessons-learned.md
└── LICENSE
```

---

## Project Status

**Version 1.0 — Infrastructure deployed and validated**

Core AWS networking, compute, database, IAM, security, and monitoring components have been successfully deployed and tested.

The next phase focuses on documentation and future infrastructure automation.
