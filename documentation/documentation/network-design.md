# Network Design — CloudTech Solutions

## 1. VPC Configuration

The infrastructure uses an Amazon Virtual Private Cloud (VPC) to provide an isolated network environment.

| Parameter | Configuration |
|---|---|
| VPC CIDR | `10.0.0.0/16` |
| AWS Region | `eu-west-3` |
| Availability Zones | `eu-west-3a`, `eu-west-3b` |

## 2. Subnet Allocation

The VPC is divided into six subnets distributed across two Availability Zones.

| Subnet | Availability Zone | CIDR | Purpose |
|---|---|---|---|
| Public Subnet 1 | eu-west-3a | `10.0.1.0/24` | Internet-facing resources and NAT Gateway |
| Public Subnet 2 | eu-west-3b | `10.0.2.0/24` | Internet-facing resources and NAT Gateway |
| Private Application Subnet 1 | eu-west-3a | `10.0.3.0/24` | Application EC2 instances |
| Private Application Subnet 2 | eu-west-3b | `10.0.4.0/24` | Application EC2 instances |
| Private Database Subnet 1 | eu-west-3a | `10.0.5.0/24` | Database infrastructure |
| Private Database Subnet 2 | eu-west-3b | `10.0.6.0/24` | Database infrastructure |

## 3. Network Components

- **Internet Gateway (IGW):** Provides internet connectivity to the VPC.
- **NAT Gateways:** Enable outbound internet access for resources in private application subnets.
- **Application Load Balancer:** Receives HTTP/HTTPS traffic and distributes requests to application instances.
- **Route Tables:** Define traffic routing between subnets and network gateways.
- **Security Groups:** Control inbound and outbound traffic for infrastructure resources.

## 4. Network Traffic Flow

The architecture follows this general traffic flow:

1. Users access the application through the Internet.
2. Incoming traffic enters the VPC through the Internet Gateway.
3. The Application Load Balancer distributes requests to EC2 instances in private application subnets.
4. Application instances communicate with the database hosted in private database subnets.
5. NAT Gateways provide outbound internet connectivity for private application resources.

## 5. Network Design Considerations

- Subnet separation between public, application, and database tiers.
- Distribution across two Availability Zones.
- Restricted database network access.
- Controlled inbound and outbound traffic through Security Groups.
- Private placement of application and database resources.

## 6. Design Reference

The network design is based on the CloudTech Solutions AWS 3-Tier Infrastructure architecture diagram located in the `architecture/` directory.
