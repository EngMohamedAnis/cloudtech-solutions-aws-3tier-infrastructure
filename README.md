# CloudTech Solutions — AWS 3-Tier Infrastructure

## Project Overview

CloudTech Solutions is an academic team project focused on designing and provisioning a scalable, highly available, and secure cloud infrastructure on Amazon Web Services (AWS).

The project implements a three-tier infrastructure architecture using AWS services and custom AWS CloudFormation templates to organize and provision the required resources.

## Objectives

- Design a structured AWS cloud infrastructure.
- Implement network isolation using public and private subnets.
- Configure secure communication between infrastructure components.
- Improve application availability through load balancing and Auto Scaling.
- Deploy a relational database using Amazon RDS across multiple Availability Zones.
- Automate infrastructure provisioning using AWS CloudFormation.

## Architecture

The infrastructure is based on a three-tier architecture and includes:

- **Networking:** Amazon VPC, public and private subnets, routing, and security groups.
- **Compute:** Amazon EC2 instances and Auto Scaling.
- **Load Balancing:** Application Load Balancer for traffic distribution.
- **Database:** Amazon RDS deployed across Availability Zones.

An architecture diagram will be added to illustrate the infrastructure and its components.

## Technologies & AWS Services

| Category | Technologies |
|---|---|
| Cloud Provider | Amazon Web Services (AWS) |
| Infrastructure as Code | AWS CloudFormation |
| Networking | Amazon VPC, Subnets, Route Tables, Security Groups |
| Compute | Amazon EC2, Auto Scaling |
| Load Balancing | Application Load Balancer |
| Database | Amazon RDS |

## Implementation

The infrastructure was provisioned using custom CloudFormation templates.

The implementation covered:

1. VPC and network infrastructure setup.
2. Public and private subnet configuration.
3. Routing and security group configuration.
4. EC2 and load balancing deployment.
5. Auto Scaling configuration.
6. Amazon RDS deployment across Availability Zones.

## Repository Structure

The repository is organized into the following directories:

- `architecture/` — Infrastructure architecture diagrams.
- `cloudformation/` — Custom CloudFormation templates.
- `documentation/` — Project documentation and deployment details.
- `screenshots/` — Implementation and deployment evidence.

## Project Status

**Completed — Academic Team Project (2025–2026)**

## Learning Outcomes

This project provided practical experience in:

- AWS infrastructure design and provisioning.
- Cloud networking and security configuration.
- Infrastructure as Code using CloudFormation.
- Compute, load balancing, and database integration.
- Designing scalable and highly available infrastructure.

## Disclaimer

This project was developed for academic purposes. The repository documents the architecture and implementation work carried out during the project.
