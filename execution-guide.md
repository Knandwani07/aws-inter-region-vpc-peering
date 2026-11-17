## 🛠️ Execution Guide: Inter-Region VPC Peering

### Step 1: Create VPC-A (Region A – us-east-1)

1. Open the AWS Management Console.
2. Switch the Region to **US East (N. Virginia) – us-east-1**.
3. Navigate to **VPC → Your VPCs → Create VPC**.
4. Select `VPC only`.
5. Configure:
   - Name: `vpc-a`
   - IPv4 CIDR: `10.0.0.0/24`
6. Click **Create VPC**.

### Step 2: Create VPC-B (Region B – ap-south-1)

1. Switch the Region to **Asia Pacific (Mumbai) – ap-south-1**.
2. Navigate to **VPC → Your VPCs → Create VPC**.
3. Select `VPC only`.
4. Configure:
   - Name: `vpc-b`
   - IPv4 CIDR: `10.1.0.0/24`
5. Click **Create VPC**.

### Step 3: Create Subnets

**Region A – us-east-1**

1. Navigate to **VPC → Subnets → Create subnet**.
2. Select `vpc-a`.
3. Configure:
   - Subnet name: `subnet-a`
   - Availability Zone: Select one.
   - IPv4 CIDR: `10.0.0.0/25`
4. Click **Create subnet**.

**Region B – ap-south-1**

1. Switch to Mumbai.
2. Navigate to **VPC → Subnets → Create subnet**.
3. Select `vpc-b`.
4. Configure:
   - Subnet name: `subnet-b`
   - Availability Zone: Select one.
   - IPv4 CIDR: `10.1.0.0/25`
5. Click **Create subnet**.

### Step 4: Create and Attach Internet Gateways

Repeat the following steps in both Regions.

**Region A:**

1. Navigate to **VPC → Internet Gateways**.
2. Click **Create internet gateway**.
3. Name it `igw-a`.
4. Click **Create internet gateway**.
5. Select the gateway and choose **Actions → Attach to a VPC**.
6. Select `vpc-a` and attach it.

**Region B:**

Repeat the same steps using:
- Internet Gateway: `igw-b`
- VPC: `vpc-b`

Verify that both Internet Gateways show the `Attached` state.

### Step 5: Configure Route Tables

Create a custom route table in each Region.

**Region A:**

1. Navigate to **VPC → Route Tables**.
2. Click **Create route table**.
3. Configure:
   - Name: `rt-a`
   - VPC: `vpc-a`
4. Click **Create route table**.
5. Open **Subnet associations → Edit subnet associations**.
6. Select `subnet-a` and save.
7. Open **Routes → Edit routes**.
8. Add the following route:

| Destination | Target |
|---|---|
| 0.0.0.0/0 | igw-a |

9. Click **Save changes**.

**Region B:**

Repeat the same steps using:

- Route table: `rt-b`
- VPC: `vpc-b`
- Subnet association: `subnet-b`
- Destination: `0.0.0.0/0`
- Target: `igw-b`

### Step 6: Create Security Groups

Create one security group in each Region.

**Region A – security-group-a**

1. Navigate to **EC2 → Security Groups**.
2. Click **Create security group**.
3. Configure:
   - Name: `security-group-a`
   - VPC: `vpc-a`
4. Add the following inbound rules:

| Type | Port | Source |
|---|---|---|
| SSH | 22 | My IP |
| All ICMP - IPv4 | All | 10.1.0.0/24 |

5. Click **Create security group**.

**Region B – security-group-b**

1. Switch to Mumbai.
2. Create another security group.
3. Configure:
   - Name: `security-group-b`
   - VPC: `vpc-b`
4. Add the following inbound rules:

| Type | Port | Source |
|---|---|---|
| SSH | 22 | My IP |
| All ICMP - IPv4 | All | 10.0.0.0/24 |

5. Click **Create security group**.

### Step 7: Launch EC2 Instances

Launch one EC2 instance in each Region.

**Region A – ec2-a**

1. Navigate to **EC2 → Instances → Launch instance**.
2. Configure:
   - Name: `ec2-a`
   - AMI: Amazon Linux 2023
   - Instance type: `t2.micro` or `t3.micro`
   - Key pair: Create a new key pair or select an existing one.
3. Under Network settings:
   - VPC: `vpc-a`
   - Subnet: `subnet-a`
   - Auto-assign public IP: Enable
   - Security group: `security-group-a`
4. Click **Launch instance**.

**Region B – ec2-b**

Repeat the process in Mumbai using:

- Name: `ec2-b`
- AMI: Amazon Linux 2023
- Instance type: `t2.micro` or `t3.micro`
- VPC: `vpc-b`
- Subnet: `subnet-b`
- Auto-assign public IP: Enable
- Security group: `security-group-b`

Wait until both instances pass their status checks.

Record the private IP addresses of both EC2 instances.

### Step 8: Create the VPC Peering Connection

1. Switch to **us-east-1**.
2. Navigate to **VPC → Peering Connections**.
3. Click **Create peering connection**.
4. Configure:
   - Name: `peer-a-to-b`
   - Requester VPC: `vpc-a`
   - Account: My account
   - Region: Another Region
   - Region: `ap-south-1`
   - Accepter VPC ID: Enter the VPC ID of `vpc-b`.
5. Click **Create peering connection**.

The connection status will initially show `Pending acceptance`.

### Step 9: Accept the Peering Request

1. Switch to **ap-south-1**.
2. Navigate to **VPC → Peering Connections**.
3. Select the pending peering request.
4. Choose **Actions → Accept request**.
5. Confirm acceptance.

Wait until the peering connection status changes to `Active`.

### Step 10: Update Route Tables for VPC Peering

Both VPC route tables must contain routes pointing to the peer VPC through the peering connection.

**Region A – rt-a**

1. Switch to `us-east-1`.
2. Navigate to **VPC → Route Tables**.
3. Select `rt-a`.
4. Open **Routes → Edit routes**.
5. Click **Add route**.
6. Configure:

| Destination | Target |
|---|---|
| 10.1.0.0/24 | VPC Peering Connection (peer-a-to-b) |

7. Click **Save changes**.

**Region B – rt-b**

1. Switch to `ap-south-1`.
2. Navigate to **VPC → Route Tables**.
3. Select `rt-b`.
4. Open **Routes → Edit routes**.
5. Click **Add route**.
6. Configure:

| Destination | Target |
|---|---|
| 10.0.0.0/24 | VPC Peering Connection (peer-a-to-b) |

7. Click **Save changes**.

### Step 11: Test Private Connectivity

After the peering connection becomes active and the routes and security groups are configured, test communication between the EC2 instances.

**Test 1: Region A to Region B**

1. Connect to `ec2-a` using SSH or EC2 Instance Connect.
2. Execute the following command using the private IP of `ec2-b`:

```bash
ping <private-ip-of-ec2-b>
```

3. Verify that ICMP replies are received.

**Test 2: Region B to Region A**

1. Connect to `ec2-b`.
2. Execute the following command using the private IP of `ec2-a`:

```bash
ping <private-ip-of-ec2-a>
```

3. Verify that ICMP replies are received.

Successful responses in both directions confirm private connectivity between the two VPCs through the inter-Region peering connection.

### Step 12: Cleanup AWS Resources

After completing the connectivity test, delete the resources to avoid unnecessary AWS charges.

Perform the cleanup in both Regions.

1. Terminate both EC2 instances.
2. Delete the VPC peering connection.
3. Delete custom route tables (`rt-a` and `rt-b`).
4. Delete security groups (`security-group-a` and `security-group-b`).
5. Detach and delete Internet Gateways (`igw-a` and `igw-b`).
6. Delete subnets (`subnet-a` and `subnet-b`).
7. Delete VPCs (`vpc-a` and `vpc-b`).

Ensure all dependent resources have been removed before deleting the VPCs.
