# AWS VPC Configuration Lab: Complete Walkthrough

## 1. Project Overview

This project demonstrates how to build a Virtual Private Cloud (VPC) in Amazon Web Services (AWS), with separate public and private subnets.

The lab covers network configuration, routing, internet connectivity, EC2 instance deployment, secure administration through a bastion host, and connectivity testing.

The public subnet contains a Bastion Server and a NAT Gateway. The private subnet contains a Private Instance that does not have a public IPv4 address. The Bastion Server provides administrative access to the Private Instance, while the NAT Gateway allows the Private Instance to initiate outbound internet connections.

**Platform:** AWS Academy / Vocareum  
**Operating system:** Amazon Linux 2023  
**Instance type:** t3.micro  
**Purpose:** Learn VPC networking, subnetting, routing, and secure access to private EC2 instances.

## 2. Lab Objectives

The objectives of this lab were to:

1. Create a custom VPC.
2. Configure DNS hostnames.
3. Create public and private subnets.
4. Create and attach an Internet Gateway.
5. Configure public and private route tables.
6. Launch a Bastion Server in the public subnet.
7. Create a NAT Gateway with an Elastic IP address.
8. Configure outbound internet routing for the private subnet.
9. Launch a Private Instance without a public IPv4 address.
10. Configure security groups for SSH connectivity.
11. Connect to the Private Instance through the Bastion Server.
12. Test outbound internet connectivity from the Private Instance.

## 3. Network Architecture and IP Addressing

### 3.1 VPC configuration

The VPC was created with the following configuration:

| Setting | Value |
|---|---|
| VPC name | Lab VPC |
| IPv4 CIDR block | 10.0.0.0/16 |
| DNS hostnames | Enabled |

A VPC provides an isolated virtual network in which AWS resources can communicate using private IP addresses and configured network routes.

The CIDR block `10.0.0.0/16` defines the IPv4 address range available to the VPC.

### 3.2 Subnet configuration

Two subnets were created inside the VPC.

| Setting | Public Subnet | Private Subnet |
|---|---|---|
| Subnet name | Public Subnet | Private Subnet |
| IPv4 CIDR | 10.0.0.0/24 | 10.0.2.0/23 |
| Availability Zone | Same selected AZ | Same selected AZ |
| Purpose | Public-facing resources | Internal workloads |

The public subnet was configured to automatically assign public IPv4 addresses to eligible newly launched instances. The private subnet was used for the Private Instance, which was launched without a public IPv4 address.

A `/24` IPv4 network contains 256 total addresses, while a `/23` network contains 512 total addresses. AWS reserves five IPv4 addresses in each IPv4 subnet, leaving 251 and 507 assignable addresses respectively for these subnet sizes.

### Why use two subnets?

Separating resources into public and private subnets improves network organization and security.

- The public subnet contains resources that need a route to the Internet Gateway.
- The private subnet contains resources that should not be directly reachable from the public internet.

A subnet is not automatically public or private based only on its name or IP address settings. Its route table and the resources' addressing and security configuration determine its connectivity.

## 4. Create the VPC

### Step 1: Open the VPC console

1. Sign in to AWS Academy/Vocareum.
2. Open the AWS Management Console.
3. Search for **VPC** in the AWS console search bar.
4. Open the VPC service.
5. Select **Your VPCs**.
6. Choose **Create VPC**.

### Step 2: Configure the VPC

1. Enter `Lab VPC` as the name.
2. Choose IPv4 CIDR manual input if prompted.
3. Enter `10.0.0.0/16`.
4. Complete the required settings.
5. Create the VPC.

### Step 3: Enable DNS hostnames

1. Select `Lab VPC`.
2. Open the VPC actions or settings menu.
3. Select **Edit VPC settings**.
4. Enable **DNS hostnames**.
5. Save the changes.

DNS hostnames help instances use DNS names where supported and configured.

**Result:** The custom VPC was created successfully.

## 5. Create the Public Subnet

### Step 1: Open the subnet creation page

1. In the VPC console, select **Subnets**.
2. Choose **Create subnet**.
3. Select `Lab VPC`.

### Step 2: Configure the subnet

Use these settings:

- Subnet name: `Public Subnet`
- IPv4 CIDR block: `10.0.0.0/24`
- Availability Zone: Select the first available AZ used for the lab.

Create the subnet.

### Step 3: Enable automatic public IPv4 assignment

1. Select `Public Subnet`.
2. Open **Actions**.
3. Choose **Edit subnet settings**.
4. Enable **Auto-assign public IPv4 address**.
5. Save the settings.

This setting allows eligible instances launched into the subnet to receive public IPv4 addresses automatically, unless the launch configuration overrides it.

**Result:** The public subnet was created and configured.

## 6. Create the Private Subnet

### Step 1: Start subnet creation

1. Select **Subnets**.
2. Choose **Create subnet**.
3. Select `Lab VPC`.

### Step 2: Configure the subnet

Use these settings:

- Subnet name: `Private Subnet`
- IPv4 CIDR block: `10.0.2.0/23`
- Availability Zone: Use the same selected AZ as the Public Subnet.

Create the subnet.

The Private Instance was later launched in this subnet without a public IPv4 address.

**Result:** The private subnet was created inside the VPC.

## 7. Create and Attach the Internet Gateway

An Internet Gateway (IGW) provides the internet-routing attachment for a VPC.

### Step 1: Create the Internet Gateway

1. In the VPC console, select **Internet gateways**.
2. Choose **Create internet gateway**.
3. Enter the name `Lab IGW`.
4. Create the Internet Gateway.

### Step 2: Attach the Internet Gateway

1. Select `Lab IGW`.
2. Open **Actions**.
3. Choose **Attach to a VPC**.
4. Select `Lab VPC`.
5. Attach the gateway.

An attached Internet Gateway does not automatically make every subnet public. A suitable route table, public IP addressing, and security rules are also required.

**Result:** `Lab IGW` was attached to `Lab VPC`.

## 8. Configure the Public Route Table

A route table determines where traffic from associated subnets is sent.

### Step 1: Create the public route table

1. Open **Route tables** in the VPC console.
2. Choose **Create route table**.
3. Enter `Public Route Table`.
4. Select `Lab VPC`.
5. Create the route table.

### Step 2: Add the internet route

1. Select `Public Route Table`.
2. Open the **Routes** tab.
3. Choose **Edit routes**.
4. Add a route with these values:

| Destination | Target |
|---|---|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | Lab IGW |

5. Select `Lab IGW` as the target for `0.0.0.0/0`.
6. Save the changes.

The `local` route is automatically created for communication within the VPC. The `0.0.0.0/0` route directs other IPv4 destinations toward the Internet Gateway.

### Step 3: Associate the public subnet

1. Open the **Subnet associations** tab.
2. Choose **Edit subnet associations**.
3. Select `Public Subnet`.
4. Save the association.

**Result:** The public subnet was associated with a route table that provides a route to the Internet Gateway.

## 9. Launch the Bastion Server

A bastion host is an administrative entry point used to access resources inside a private network.

### Step 1: Open EC2

1. Search for **EC2** in the AWS console.
2. Open the EC2 service.
3. Select **Instances**.
4. Choose **Launch instances**.

### Step 2: Configure the instance

The following settings were used:

| Setting | Value |
|---|---|
| Name | Bastion Server |
| Operating system | Amazon Linux 2023 |
| Instance type | t3.micro |
| VPC | Lab VPC |
| Subnet | Public Subnet |
| Public IPv4 address | Enabled |
| Security group | Bastion Security Group |

Select an appropriate key pair or the authentication method provided by the lab.

### Step 3: Configure the security group

The `Bastion Security Group` was configured to allow SSH traffic on TCP port `22`, using the source specified by the lab instructions.

SSH enables remote command-line administration.

For production deployments, SSH access should normally be restricted to trusted source IP addresses or replaced with a managed access method such as AWS Systems Manager Session Manager where appropriate.

### Step 4: Launch the instance

1. Review the configuration.
2. Choose **Launch instance**.
3. Wait for the instance to reach the Running state.
4. Confirm that the instance has a public IPv4 address.

**Result:** The Bastion Server was launched in the public subnet.

## 10. Create the NAT Gateway

A Network Address Translation (NAT) Gateway allows resources in a private subnet to initiate outbound internet connections without requiring their own public IPv4 addresses.

### Step 1: Open NAT Gateways

1. Return to the VPC console.
2. Select **NAT gateways**.
3. Choose **Create NAT gateway**.

### Step 2: Configure the NAT Gateway

Use these settings:

| Setting | Value |
|---|---|
| Name | Lab NAT gateway |
| VPC | Lab VPC |
| Subnet | Public Subnet |
| Connectivity type | Public |
| Elastic IP | Allocate an Elastic IP address |

Create the NAT Gateway and wait for it to become Available.

### Step 3: Troubleshoot the authorization error

During the lab, the initial NAT Gateway creation attempt returned an authorization error relating to permission to create a service-linked role for regional NAT Gateways.

A service-linked role allows an AWS service to perform the operations it needs on your behalf. In a managed training environment, permissions may be restricted by the lab's IAM configuration or service control policies.

The issue was resolved sufficiently for the NAT Gateway to be created, and the lab proceeded.

**Troubleshooting lesson:** If an AWS Academy lab blocks a required operation, check the resource status, the lab instructions, and the permissions available in the training environment. Do not attempt to bypass an organizational restriction.

**Result:** `Lab NAT gateway` was created in the public subnet with an Elastic IP address.

## 11. Configure the Private Route Table

The private route table directs private-subnet traffic to the NAT Gateway for outbound internet access.

### Step 1: Identify the existing route table

The existing or main route table was renamed `Private Route Table` for clarity.

The main route table is the VPC's default route table for subnets without an explicit association. Renaming it does not change its routing behavior.

### Step 2: Add the NAT route

1. Select `Private Route Table`.
2. Open the **Routes** tab.
3. Choose **Edit routes**.
4. Add the following route:

| Destination | Target |
|---|---|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | Lab NAT gateway |

5. Select `Lab NAT gateway` as the target.
6. Save the changes.

### Step 3: Associate the private subnet

1. Open **Subnet associations**.
2. Choose **Edit subnet associations**.
3. Select `Private Subnet`.
4. Save the association.

**Result:** The private subnet was configured to send internet-bound IPv4 traffic to the NAT Gateway.

## 12. Launch the Private Instance

The Private Instance runs in the private subnet and does not need a public IPv4 address.

### Step 1: Launch a new EC2 instance

1. Open the EC2 console.
2. Select **Instances**.
3. Choose **Launch instances**.

### Step 2: Configure the instance

| Setting | Value |
|---|---|
| Name | Private Instance |
| Operating system | Amazon Linux 2023 |
| Instance type | t3.micro |
| VPC | Lab VPC |
| Subnet | Private Subnet |
| Public IPv4 address | Disabled |
| Security group | Private Instance SG |

### Step 3: Configure the security group

The `Private Instance SG` was configured to allow inbound SSH traffic on TCP port `22` from `10.0.0.0/16`, as specified by the lab.

This permits SSH traffic from sources in the VPC address range, subject to the other network and instance controls.

In production, security-group sources should be narrowed to the actual trusted administration path wherever possible, such as the Bastion Server's security group.

### Step 4: Configure the lab user data

The following script was used as part of the lab setup:

```bash
#!/bin/bash
echo 'lab-password' | passwd ec2-user --stdin
sed -i 's|[#]*PasswordAuthentication no|PasswordAuthentication yes|g' /etc/ssh/sshd_config
systemctl restart sshd.service
```

The script configures a lab password for `ec2-user`, changes the SSH password-authentication setting, and restarts the SSH service.

**Security warning:** This configuration is intended only for the exercise. Password-based SSH access and hard-coded passwords are not recommended for production. Use key-based authentication or an appropriate managed access solution, and never reuse lab credentials.

### Step 5: Launch the instance

1. Review the configuration.
2. Launch the instance.
3. Wait until the instance reaches the Running state.
4. Verify that it has a private IP address.
5. Confirm that it does not have a public IPv4 address.

**Result:** The Private Instance was launched successfully in the private subnet.

## 13. Connect to the Bastion Server

The Bastion Server provides a route for administrators to access the Private Instance without exposing the latter directly to the internet.

### Step 1: Open the Bastion Server connection

1. In the EC2 console, select `Bastion Server`.
2. Choose **Connect**.
3. Open **EC2 Instance Connect**, if available for the lab.
4. Connect to the instance.

A terminal session opens on the Bastion Server.

### Step 2: Confirm the private instance's private IP address

In the EC2 console, select `Private Instance` and note its private IPv4 address.

Use that private IP address for the next connection.

## 14. Connect to the Private Instance Using SSH

From the Bastion Server terminal, connect to the Private Instance with:

```bash
ssh PRIVATE-IP
```

Replace `PRIVATE-IP` with the actual private IPv4 address of the Private Instance.

For example, if the private instance's IP address were `10.0.2.25`, the command would be:

```bash
ssh 10.0.2.25
```

The example IP is illustrative; use the actual address shown in your EC2 console.

### Troubleshooting the initial SSH timeout

The first SSH connection attempt timed out.

The configuration was reviewed, including the SSH rule in `Private Instance SG`. After troubleshooting, the connection succeeded.

A timeout can occur for several reasons:

- The security group does not allow SSH from the source.
- The instances cannot communicate through their network paths.
- A network ACL blocks traffic.
- The SSH service is not ready or is not listening.
- The wrong private IP address was used.

The route table's local route normally enables communication between subnets within the same VPC. The private instance's security group and other applicable network controls must still permit the connection.

### Successful result

The SSH connection was eventually established from the Bastion Server to the Private Instance.

This demonstrated that the private instance could be administered through the bastion host using its private IP address, without assigning it a public IPv4 address.

## 15. Test Internet Connectivity from the Private Instance

After connecting to the Private Instance, the next step was to test outbound connectivity.

Run:

```bash
ping -c 3 amazon.com
```

### What the command does

- `ping` sends ICMP echo requests to test network reachability.
- `-c 3` sends three requests and then stops.
- `amazon.com` is the hostname being tested.

The test returned successful replies in the lab.

This demonstrated that the private instance could resolve the hostname and exchange ICMP traffic with the destination through the configured network path.

### Expected traffic path

The outbound traffic path was:

1. Private Instance
2. Private Route Table
3. NAT Gateway
4. Public subnet's route table
5. Internet Gateway
6. Internet destination

Reply traffic returns through the established network path.

A successful ping is useful evidence of connectivity, although some internet destinations may block ICMP. A failed ping alone does not always mean internet access is unavailable.

**Result:** The Private Instance successfully reached the internet without having its own public IPv4 address.

## 16. Final Configuration Summary

| Resource | Configuration |
|---|---|
| VPC | `Lab VPC` |
| VPC CIDR | `10.0.0.0/16` |
| Public subnet | `10.0.0.0/24` |
| Private subnet | `10.0.2.0/23` |
| Internet Gateway | `Lab IGW` |
| Public route table | `Public Route Table` |
| Private route table | `Private Route Table` |
| NAT Gateway | `Lab NAT gateway` |
| Bastion instance | `Bastion Server` |
| Bastion security group | `Bastion Security Group` |
| Private instance | `Private Instance` |
| Private instance security group | `Private Instance SG` |
| Instance operating system | Amazon Linux 2023 |
| Instance type | `t3.micro` |
| SSH validation | Successful through the Bastion Server |
| Internet validation | Successful `ping -c 3 amazon.com` |

## 17. Troubleshooting Summary

### Issue 1: NAT Gateway authorization error

**Problem:** AWS returned an authorization error involving creation of a service-linked role.

**Resolution:** The issue was resolved sufficiently to create the NAT Gateway and continue with the lab.

**Lesson learned:** Managed AWS training environments may restrict certain permissions. Check the lab instructions and allowed permissions before troubleshooting further.

### Issue 2: SSH connection timeout

**Problem:** The initial SSH connection from the Bastion Server to the Private Instance timed out.

**Resolution:** We reviewed the private instance's SSH security-group rule and continued troubleshooting until the connection succeeded.

**Lesson learned:** SSH connectivity depends on the correct private IP address, permitted security-group traffic, network ACLs, routes, and a working SSH service.

### Issue 3: Private subnet internet connectivity

**Problem to validate:** Confirm that an instance without a public IPv4 address can reach the internet.

**Resolution:** The private route table was configured to direct `0.0.0.0/0` to the NAT Gateway, and the ping test succeeded.

**Lesson learned:** A private instance can initiate outbound IPv4 connections through a NAT Gateway without being directly addressed by unsolicited inbound internet traffic.

## 18. Screenshots and Evidence

Save screenshots inside the `docs/screenshots/` folder in this project.

Use the following filenames to keep the evidence organized.

| Filename | Evidence to capture |
|---|---|
| `01-lab-vpc.png` | Lab VPC details and CIDR |
| `02-public-subnet.png` | Public Subnet and CIDR |
| `03-private-subnet.png` | Private Subnet and CIDR |
| `04-internet-gateway.png` | Lab IGW attached to Lab VPC |
| `05-public-route-table.png` | Public route to the Internet Gateway |
| `06-private-route-table.png` | Private route to the NAT Gateway |
| `07-bastion-server.png` | Bastion Server running in the public subnet |
| `08-nat-gateway.png` | Lab NAT gateway and its status |
| `09-private-instance.png` | Private Instance in the private subnet |
| `10-private-ssh.png` | Successful SSH connection to the private instance |
| `11-nat-connectivity-test.png` | Successful `ping -c 3 amazon.com` output |

Only include screenshots after capturing the actual evidence. Avoid exposing access keys, passwords, private credentials, or other sensitive account information.

## 19. Security Considerations

This lab demonstrates basic network separation and controlled administration. A production deployment should consider the following improvements:

- Restrict SSH access to known trusted sources.
- Prefer Session Manager or another approved managed access method where appropriate.
- Avoid hard-coded passwords and password-based SSH authentication.
- Use least-privilege security-group rules.
- Review network ACLs and route tables.
- Enable suitable logging and monitoring.
- Keep software and operating systems updated.
- Remove unused lab resources to avoid unnecessary charges.

A NAT Gateway provides outbound IPv4 connectivity for private-subnet resources, but it is not a substitute for security groups, network ACLs, host security, or outbound traffic controls.

## 20. Cleanup and Cost Management

When the lab is no longer required, remove resources carefully to avoid ongoing charges.

Before deleting anything, confirm that you have saved the required screenshots and documentation.

A typical cleanup sequence is:

1. Terminate `Private Instance`.
2. Terminate `Bastion Server`.
3. Delete `Lab NAT gateway` and wait for deletion to complete.
4. Release the Elastic IP address allocated to the NAT Gateway if it is no longer associated or needed.
5. Detach and delete `Lab IGW`.
6. Delete the custom route tables after removing their subnet associations, where required.
7. Delete the public and private subnets.
8. Delete `Lab VPC`.
9. Remove unused security groups if they are not already removed with the associated resources.

AWS may require resources to be disassociated or deleted in a particular order. Follow the console prompts and verify that you are deleting the intended lab resources.

NAT Gateways and public IPv4 addresses can incur charges while provisioned, so do not leave them running unnecessarily.

## 21. Key Lessons Learned

Through this lab, I learned how to:

- Build a custom VPC with a defined IPv4 CIDR range.
- Divide a network into public and private subnets.
- Configure an Internet Gateway and route tables.
- Understand the difference between public and private routing.
- Deploy a Bastion Server for administrative access.
- Configure a NAT Gateway for outbound internet access.
- Launch an EC2 instance without a public IPv4 address.
- Troubleshoot SSH connectivity and security-group rules.
- Validate outbound network connectivity from a private instance.
- Document AWS infrastructure configuration and troubleshooting.

## 22. Conclusion

This lab demonstrated how to configure a VPC with separate public and private subnets, route traffic through an Internet Gateway and a NAT Gateway, and administer a private EC2 instance through a Bastion Server.

The final connectivity tests confirmed that SSH access to the private instance was successful and that the instance could reach the internet through the NAT Gateway without requiring a public IPv4 address.

This project strengthened my understanding of AWS networking, EC2 deployment, route tables, security groups, troubleshooting, and foundational cloud infrastructure security.