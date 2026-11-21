
## 1. Deployment Overview

The CloudTech Solutions infrastructure follows a three-tier AWS architecture deployed in the `eu-west-3` Region.

The infrastructure includes:

- Amazon VPC and six subnets.
- Internet Gateway and NAT Gateways.
- Route Tables and Security Groups.
- Application Load Balancer.
- Amazon EC2 and Auto Scaling Group.
- Amazon RDS Multi-AZ.
- Amazon S3.

### Prerequisites

- AWS CLI installed.
- AWS credentials configured.
- An AWS account with appropriate IAM permissions.
- An EC2 key pair if SSH access is required.
- An existing application image or application deployment package.

Configure the AWS CLI:

```bash
aws configure
```

Set the deployment Region:

```bash
aws configure set region eu-west-3
```

Verify the AWS identity:

```bash
aws sts get-caller-identity
```

## 2. Environment Variables

Define reusable variables for the deployment:

```bash
export AWS_REGION="eu-west-3"
export VPC_CIDR="10.0.0.0/16"
```

The following subnet allocation is based on the project architecture:

| Subnet | CIDR | Availability Zone | Tier |
|---|---|---|---|
| Public 1 | 10.0.1.0/24 | eu-west-3a | Public |
| Public 2 | 10.0.2.0/24 | eu-west-3b | Public |
| Application 1 | 10.0.3.0/24 | eu-west-3a | Private |
| Application 2 | 10.0.4.0/24 | eu-west-3b | Private |
| Database 1 | 10.0.5.0/24 | eu-west-3a | Private |
| Database 2 | 10.0.6.0/24 | eu-west-3b | Private |

## 3. VPC Creation

Create the virtual private cloud:

```bash
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications \
  'ResourceType=vpc,Tags=[{Key=Name,Value=CloudTech-VPC}]'
```

Enable DNS support and DNS hostnames:

```bash
aws ec2 modify-vpc-attribute \
  --vpc-id <VPC_ID> \
  --enable-dns-support '{"Value":true}'

aws ec2 modify-vpc-attribute \
  --vpc-id <VPC_ID> \
  --enable-dns-hostnames '{"Value":true}'
```

Replace `<VPC_ID>` with the ID returned by the VPC creation command.

## 4. Subnet Creation

Create the six subnets in their respective Availability Zones.

Example:

```bash
aws ec2 create-subnet \
  --vpc-id <VPC_ID> \
  --cidr-block 10.0.1.0/24 \
  --availability-zone eu-west-3a \
  --tag-specifications \
  'ResourceType=subnet,Tags=[{Key=Name,Value=CloudTech-Public-AZ1}]'
```

Repeat the command for the remaining five subnets using the CIDR allocations in Section 2.

Enable public IPv4 address assignment on the public subnets:

```bash
aws ec2 modify-subnet-attribute \
  --subnet-id <PUBLIC_SUBNET_ID> \
  --map-public-ip-on-launch
```

## 5. Internet Gateway

Create an Internet Gateway:

```bash
aws ec2 create-internet-gateway
```

Attach it to the VPC:

```bash
aws ec2 attach-internet-gateway \
  --internet-gateway-id <IGW_ID> \
  --vpc-id <VPC_ID>
```

The Internet Gateway provides internet connectivity for resources with appropriate public routing and security configuration.

## 6. NAT Gateway Configuration

Allocate Elastic IP addresses:

```bash
aws ec2 allocate-address \
  --domain vpc
```

Create a NAT Gateway in each public subnet:

```bash
aws ec2 create-nat-gateway \
  --subnet-id <PUBLIC_SUBNET_AZ1> \
  --allocation-id <EIP_ALLOCATION_ID_AZ1>
```

Repeat for the second Availability Zone.

Wait for the NAT Gateways to become available before configuring private route tables.

## 7. Route Tables

Create a public route table:

```bash
aws ec2 create-route-table \
  --vpc-id <VPC_ID>
```

Add a default route through the Internet Gateway:

```bash
aws ec2 create-route \
  --route-table-id <PUBLIC_ROUTE_TABLE_ID> \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id <IGW_ID>
```

Associate the public route table with both public subnets.

Create private application route tables and configure their default routes through the corresponding NAT Gateways:

```bash
aws ec2 create-route \
  --route-table-id <PRIVATE_APP_ROUTE_TABLE_ID> \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id <NAT_GATEWAY_ID>
```

Associate each application route table with its intended private application subnet.

The database subnet route tables should not provide a default internet route unless the actual design explicitly requires one.

## 8. Security Groups

Create separate Security Groups for the load balancer, application tier, and database tier.

Example — ALB Security Group:

```bash
aws ec2 create-security-group \
  --group-name CloudTech-ALB-SG \
  --description "Security group for application load balancer" \
  --vpc-id <VPC_ID>
```

Example — Application Security Group:

```bash
aws ec2 create-security-group \
  --group-name CloudTech-App-SG \
  --description "Security group for application instances" \
  --vpc-id <VPC_ID>
```

Example — Database Security Group:

```bash
aws ec2 create-security-group \
  --group-name CloudTech-DB-SG \
  --description "Security group for database tier" \
  --vpc-id <VPC_ID>
```

Example inbound rule for HTTPS traffic to the ALB:

```bash
aws ec2 authorize-security-group-ingress \
  --group-id <ALB_SG_ID> \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

Application instances should accept application traffic only from the ALB Security Group.

The database Security Group should allow the database port only from the application Security Group.

The exact application and database ports depend on the selected application and database engines.

## 9. Application Load Balancer

Create an internet-facing Application Load Balancer:

```bash
aws elbv2 create-load-balancer \
  --name CloudTech-ALB \
  --subnets <PUBLIC_SUBNET_AZ1> <PUBLIC_SUBNET_AZ2> \
  --security-groups <ALB_SG_ID> \
  --scheme internet-facing \
  --type application
```

Create a target group:

```bash
aws elbv2 create-target-group \
  --name CloudTech-TargetGroup \
  --protocol HTTP \
  --port <APPLICATION_PORT> \
  --vpc-id <VPC_ID> \
  --target-type instance
```

Configure a listener and associate it with the target group.

For HTTPS, an appropriate ACM certificate must also be configured.

## 10. EC2 and Auto Scaling

Create a Launch Template containing the application instance configuration:

```bash
aws ec2 create-launch-template \
  --launch-template-name CloudTech-App-Template \
  --launch-template-data file://launch-template.json
```

The launch template should define the selected:

- AMI.
- Instance type.
- IAM instance profile, if required.
- Security Group.
- User data or application initialization configuration.

Create an Auto Scaling Group:

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name CloudTech-App-ASG \
  --launch-template LaunchTemplateName=CloudTech-App-Template,Version='$Latest' \
  --min-size 2 \
  --max-size 4 \
  --desired-capacity 2 \
  --vpc-zone-identifier "<APP_SUBNET_AZ1>,<APP_SUBNET_AZ2>" \
  --target-group-arns <TARGET_GROUP_ARN>
```

The capacity values are illustrative and must be adjusted to the intended workload.

## 11. Amazon RDS Multi-AZ

Create a DB subnet group using the two private database subnets:

```bash
aws rds create-db-subnet-group \
  --db-subnet-group-name cloudtech-db-subnets \
  --db-subnet-group-description "Private database subnets" \
  --subnet-ids <DB_SUBNET_AZ1> <DB_SUBNET_AZ2>
```

Create a Multi-AZ database instance:

```bash
aws rds create-db-instance \
  --db-instance-identifier cloudtech-database \
  --engine <DATABASE_ENGINE> \
  --db-instance-class <DB_INSTANCE_CLASS> \
  --allocated-storage <STORAGE_GB> \
  --master-username <MASTER_USERNAME> \
  --manage-master-user-password \
  --db-subnet-group-name cloudtech-db-subnets \
  --vpc-security-group-ids <DB_SG_ID> \
  --multi-az \
  --no-publicly-accessible
```

The database engine, instance class, and storage configuration must be selected according to the application requirements.

## 12. Amazon S3

Create an S3 bucket using a globally unique name:

```bash
aws s3api create-bucket \
  --bucket <UNIQUE_BUCKET_NAME> \
  --region eu-west-3 \
  --create-bucket-configuration LocationConstraint=eu-west-3
```

Keep public access blocked unless a specific, justified requirement exists.

Configure appropriate IAM permissions for application access.

## 13. Deployment Validation

The following commands illustrate how to inspect the provisioned infrastructure.

### VPC and Subnets

```bash
aws ec2 describe-vpcs
aws ec2 describe-subnets
```

### Internet and NAT Gateways

```bash
aws ec2 describe-internet-gateways
aws ec2 describe-nat-gateways
```

### Route Tables and Security Groups

```bash
aws ec2 describe-route-tables
aws ec2 describe-security-groups
```

### Load Balancer and Target Health

```bash
aws elbv2 describe-load-balancers
aws elbv2 describe-target-health \
  --target-group-arn <TARGET_GROUP_ARN>
```

### Auto Scaling

```bash
aws autoscaling describe-auto-scaling-groups
```

### Database

```bash
aws rds describe-db-instances
```

### S3

```bash
aws s3api list-buckets
```
