# AWS Infrastructure Components — CloudTech Solutions

## 1. Overview

The CloudTech Solutions infrastructure is built on AWS using a three-tier architecture distributed across two Availability Zones in the `eu-west-3` Region.

The infrastructure combines networking, compute, load balancing, database, and storage services to provide a structured cloud environment.

## 2. Networking Components

### Amazon VPC
Provides an isolated virtual network for hosting the project infrastructure.

- CIDR: `10.0.0.0/16`
- Contains six subnets across two Availability Zones.

### Public and Private Subnets
The VPC uses three subnet layers:

- **Public Subnets:** Host internet-facing infrastructure components.
- **Private Application Subnets:** Host application EC2 instances.
- **Private Database Subnets:** Provide isolated network placement for the database tier.

### Internet Gateway
Enables communication between the VPC and the public internet for resources with appropriate routing and security configuration.

### NAT Gateways
Provide outbound internet connectivity to resources in private application subnets without requiring those resources to have public IP addresses.

### Route Tables
Control traffic routing between subnets, the Internet Gateway, and NAT Gateways.

### Security Groups
Act as virtual firewalls controlling inbound and outbound traffic for AWS resources.

## 3. Load Balancing

### Application Load Balancer (ALB)

The Application Load Balancer provides an entry point for application traffic and distributes incoming requests across registered EC2 instances.

Its role includes:

- Distributing application traffic.
- Supporting application availability.
- Integrating with the EC2 Auto Scaling environment.

## 4. Compute Infrastructure

### Amazon EC2

EC2 instances provide the compute capacity for the application tier.

They are deployed in private application subnets across the two Availability Zones.

### EC2 Auto Scaling

Auto Scaling manages the number of EC2 instances according to the configured scaling policies and capacity requirements.

It supports application availability and workload scalability.

## 5. Database Infrastructure

### Amazon RDS Multi-AZ

Amazon RDS provides managed relational database services for the application.

The project architecture uses a Multi-AZ deployment design to improve database availability.

Database resources are placed in private database subnets.

## 6. Storage

### Amazon S3

Amazon S3 provides object storage for application-related assets and data.

It is included as part of the overall cloud infrastructure design.

## 7. Infrastructure Integration

The components work together as follows:

1. Internet traffic reaches the application through the Internet Gateway and Application Load Balancer.
2. The ALB distributes requests to EC2 instances in private application subnets.
3. EC2 instances communicate with the RDS database tier through permitted network paths.
4. NAT Gateways provide outbound internet connectivity to private application resources.
5. Amazon S3 provides object storage for application assets.

## 8. Design Goals

The infrastructure design focuses on:

- **Scalability:** EC2 Auto Scaling and load balancing.
- **Availability:** Multi-AZ architecture and RDS Multi-AZ.
- **Security:** Network segmentation and Security Groups.
- **Isolation:** Separate public, application, and database subnet layers.
- **Manageability:** AWS managed infrastructure services.
