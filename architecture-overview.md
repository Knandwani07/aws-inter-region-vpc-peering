## 🏛️ Architecture Overview
<img width="1774" height="799" alt="Inter-Region VPC Peering Diagram" src="https://github.com/user-attachments/assets/b9f8f44e-8bdd-4219-8144-e03c253cf72b" />

This project establishes private network connectivity between two Amazon Virtual Private Clouds (VPCs) deployed in different AWS Regions using **Inter-Region VPC Peering**.

The architecture consists of two isolated VPCs: one in **N. Virginia (`us-east-1`)** and another in **Mumbai (`ap-south-1`)**. Each VPC contains a subnet, an Internet Gateway, a custom route table, a security group, and an EC2 instance.

A VPC peering connection (`peer-a-to-b`) connects the two VPCs through the AWS global network infrastructure, allowing resources to communicate using their private IP addresses without routing traffic through the public internet.

### Region A – N. Virginia (us-east-1)

- **VPC (`vpc-a`):** Uses CIDR block `10.0.0.0/24`.
- **Subnet (`subnet-a`):** Uses CIDR block `10.0.0.0/25` and hosts the EC2 instance.
- **Internet Gateway (`igw-a`):** Provides internet connectivity for management access.
- **Route Table (`rt-a`):** Routes internet-bound traffic through the Internet Gateway and traffic destined for `10.1.0.0/24` through the VPC peering connection.
- **Security Group (`security-group-a`):** Allows SSH access from My IP and ICMP traffic from `10.1.0.0/24`.
- **EC2 Instance (`ec2-a`):** Amazon Linux 2023 instance used to initiate connectivity tests.

### Region B – Mumbai (ap-south-1)

- **VPC (`vpc-b`):** Uses CIDR block `10.1.0.0/24`.
- **Subnet (`subnet-b`):** Uses CIDR block `10.1.0.0/25` and hosts the EC2 instance.
- **Internet Gateway (`igw-b`):** Provides internet connectivity for management access.
- **Route Table (`rt-b`):** Routes internet-bound traffic through the Internet Gateway and traffic destined for `10.0.0.0/24` through the VPC peering connection.
- **Security Group (`security-group-b`):** Allows SSH access from My IP and ICMP traffic from `10.0.0.0/24`.
- **EC2 Instance (`ec2-b`):** Amazon Linux 2023 instance used to receive and initiate connectivity tests.

### VPC Peering Connection

The VPC peering connection (`peer-a-to-b`) establishes private connectivity between `vpc-a` and `vpc-b`.

- Requester: `vpc-a` in `us-east-1`.
- Accepter: `vpc-b` in `ap-south-1`.
- Connection status: Active.
- Traffic destination: Private IP addresses within the peer VPC CIDR ranges.
- Routing: Both VPC route tables contain routes pointing to the peering connection.

### Communication Flow

1. An administrator connects to each EC2 instance using SSH or EC2 Instance Connect.
2. `ec2-a` sends an ICMP request to the private IP address of `ec2-b`.
3. The route table in Region A forwards the traffic to the VPC peering connection.
4. The request travels through the AWS global network to Region B.
5. The security group in Region B permits ICMP traffic from `10.0.0.0/24`.
6. `ec2-b` responds through the peering connection using its private IP address.
7. The same process is repeated in the reverse direction to verify bidirectional connectivity.

The Internet Gateways are used for management access only. The private communication between the EC2 instances takes place through the VPC peering connection.

### Architecture Design Considerations

- Both VPCs use non-overlapping CIDR ranges.
- Each VPC requires a route to the other VPC's CIDR block.
- Security groups restrict ICMP traffic to the peer VPC CIDR.
- SSH access is restricted to the administrator's public IP.
- VPC peering is non-transitive.
- Inter-Region data transfer charges apply.
