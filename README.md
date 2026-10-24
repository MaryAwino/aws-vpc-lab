# AWS VPC Configuration: Public and Private Subnets

## 1. Project Overview

This project demonstrates how to create and configure an Amazon Virtual Private Cloud (VPC) with public and private subnets, an Internet Gateway, a NAT Gateway, route tables, and Amazon EC2 instances.

The lab also demonstrates how to connect to an EC2 instance in a private subnet through a bastion server and verify outbound internet connectivity through a NAT Gateway.

## 2. Objectives

- Create a custom VPC with an IPv4 CIDR block.
- Create public and private subnets.
- Attach an Internet Gateway to the VPC.
- Configure public and private route tables.
- Launch a bastion server in the public subnet.
- Configure a NAT Gateway for private-subnet outbound connectivity.
- Launch an EC2 instance in the private subnet.
- Connect to the private instance through the bastion server.
- Test internet connectivity from the private instance.

## 3. Network Configuration

| Resource | Configuration |
|---|---|
| VPC name | Lab VPC |
| VPC IPv4 CIDR | 10.0.0.0/16 |
| Public subnet | 10.0.0.0/24 |
| Private subnet | 10.0.2.0/23 |
| Internet Gateway | Lab IGW |
| Public route table | Public Route Table |
| Private route table | Private Route Table |
| NAT Gateway | Lab NAT gateway |
| Bastion instance | Bastion Server |
| Private instance | Private Instance |
| Instance operating system | Amazon Linux 2023 |
| Instance type | t3.micro |

Both subnets were configured in the same Availability Zone.

## 4. Architecture

The VPC contains two subnets.

### Public subnet

The public subnet contains the bastion server and NAT Gateway. Its route table directs internet-bound IPv4 traffic to the Internet Gateway.

### Private subnet

The private subnet contains an EC2 instance without a public IPv4 address. It uses the NAT Gateway for outbound internet connectivity and the bastion server for administrative SSH access.

### Traffic routing

- VPC-local traffic uses the local route.
- Internet-bound traffic from the public subnet is routed through the Internet Gateway.
- Internet-bound traffic from the private subnet is routed through the NAT Gateway.
- Administrative access to the private instance is performed through the bastion server.

## 5. Implementation Steps

1. Created the Lab VPC with CIDR 10.0.0.0/16.
2. Enabled DNS hostnames for the VPC.
3. Created the public subnet using 10.0.0.0/24.
4. Enabled automatic public IPv4 address assignment for the public subnet.
5. Created the private subnet using 10.0.2.0/23.
6. Created and attached Lab IGW to Lab VPC.
7. Configured Public Route Table with a default route to the Internet Gateway.
8. Associated Public Route Table with Public Subnet.
9. Configured Private Route Table with a default route to the NAT Gateway.
10. Launched Bastion Server in Public Subnet.
11. Created Lab NAT gateway in Public Subnet and allocated an Elastic IP.
12. Launched Private Instance in Private Subnet.
13. Connected to Private Instance through Bastion Server using SSH.
14. Tested outbound internet connectivity from the private instance.

## 6. Route Table Configuration

### Public Route Table

| Destination | Target |
|---|---|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |

Associated subnet: Public Subnet.

### Private Route Table

| Destination | Target |
|---|---|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | NAT Gateway |

Associated subnet: Private Subnet.

## 7. Validation and Testing

The following tests were completed:

- Verified the VPC and subnet configuration.
- Verified the Internet Gateway attachment.
- Configured the public and private routes.
- Launched the bastion and private EC2 instances.
- Established SSH access to the private instance through the bastion server.
- Successfully pinged amazon.com from the private instance.

The successful connectivity test demonstrated that the private instance could reach an internet destination through the configured network path.

## 8. Troubleshooting

During NAT Gateway creation, the AWS account initially displayed an authorization error related to creating a service-linked role for regional NAT Gateways. The NAT Gateway was subsequently created.

An SSH connection to the private instance initially timed out. Connectivity was subsequently established, and the internet connectivity test succeeded.

## 9. Security Considerations

- Keep private workloads in private subnets when public access is unnecessary.
- Restrict SSH access to trusted sources or the bastion security group in production.
- Avoid opening SSH to all IP addresses in production environments.
- Prefer AWS Systems Manager Session Manager or other approved secure access methods where appropriate.
- Avoid password-based SSH authentication in production.
- Monitor and remove unused AWS resources to control costs.

## 10. Key Learnings

This lab provided practical experience with AWS networking, IPv4 CIDR planning, subnet design, route tables, Internet Gateways, NAT Gateways, EC2 security groups, bastion-host access, SSH troubleshooting, and connectivity testing.

## 11. Screenshots

Screenshots documenting each implementation stage will be stored in `docs/screenshots/`.

Planned evidence:

1. VPC configuration
2. Public subnet
3. Private subnet
4. Internet Gateway
5. Public route table
6. Private route table
7. Bastion server
8. NAT Gateway
9. Private EC2 instance
10. Successful SSH connection
11. Successful internet connectivity test

## 12. Conclusion

The lab demonstrated how to build a VPC with public and private subnets, configure internet routing, and access a private EC2 instance through a bastion server. The private instance successfully reached an internet destination through the NAT Gateway, validating the core network configuration.