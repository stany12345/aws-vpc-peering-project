# AWS VPC Peering — Resource Inventory

## 1. Project Overview

| Property | Details |
|---|---|
| Project Name | AWS VPC Peering |
| Cloud Provider | Amazon Web Services (AWS) |
| AWS Region | Asia Pacific (Mumbai) — `ap-south-1` |
| VPC A | `ewb-vpc-a` |
| VPC B | `ewb-vpc-b` |
| Peering Connection | `ewb-a-b-peer` |
| Connectivity | Private IPv4 communication between VPCs |
| Administrative Access | SSH through a bastion host |

This document provides an inventory of the AWS resources used in the VPC peering project. Resource IDs and configuration details should be verified against the AWS Management Console before final submission.

---

## 2. VPC Inventory

| Resource | Name | Resource ID | IPv4 CIDR |
|---|---|---|---|
| VPC A | `ewb-vpc-a` | `vpc-08092d12e8b63a296` | `10.10.0.0/16` |
| VPC B | `ewb-vpc-b` | `vpc-0cabf8113fbdd7d46` | `10.20.0.0/16` |

### VPC Configuration

- Both VPCs use non-overlapping IPv4 CIDR ranges.
- VPC A contains a public subnet and a private subnet.
- VPC B contains a private subnet for the application instance.
- A VPC peering connection provides private connectivity between the two VPCs.
- VPC B does not require its own Internet Gateway for communication with VPC A through the peering connection.

---

## 3. Subnet Inventory

| VPC | Subnet Name | IPv4 CIDR | Purpose |
|---|---|---|---|
| VPC A | `ewb-a-public` | `10.10.1.0/24` | Bastion host |
| VPC A | `ewb-a-private` | `10.10.2.0/24` | App-A |
| VPC B | `ewb-b-private` | `10.20.1.0/24` | App-B |

### Subnet Resource IDs

| Subnet | Resource ID |
|---|---|
| VPC A private subnet | `subnet-0558eab73a3d67ba1` |
| VPC B private subnet | `subnet-0b1529b9710d77133` |
| VPC A public subnet | Verify in AWS Console |

---

## 4. EC2 Instance Inventory

| Instance | Instance ID | Private IP | Additional Details |
|---|---|---|---|
| Bastion Host | Verify in AWS Console | `10.10.1.26` | Public IP previously reported as `13.207.38.241` |
| App-A | `i-0c275e8611a22903a` | `10.10.2.53` | Secondary IP used for testing: `10.10.2.10` |
| App-B | `i-02eb907d7b30eaac6` | `10.20.1.97` | Secondary IP used for testing: `10.20.1.10` |

### App-A Network Interface

| Property | Value |
|---|---|
| Network Interface ID | `eni-0a7048efe76a299c5` |
| Subnet ID | `subnet-0558eab73a3d67ba1` |
| Primary Private IP | `10.10.2.53` |
| Secondary Private IP used for testing | `10.10.2.10` |

### App-B Network Interface

| Property | Value |
|---|---|
| Network Interface ID | `eni-0427654500ae809fc` |
| Subnet ID | `subnet-0b1529b9710d77133` |
| Primary Private IP | `10.20.1.97` |
| Secondary Private IP used for testing | `10.20.1.10` |

**Note:** The bastion public IP may change if it is not associated with an Elastic IP. Verify the current address before connecting.

---

## 5. VPC Peering Connection

| Property | Value |
|---|---|
| Name | `ewb-a-b-peer` |
| Peering Connection ID | `pcx-01934796beac9a6c4` |
| Requester VPC | `ewb-vpc-a` |
| Accepter VPC | `ewb-vpc-b` |
| VPC A CIDR | `10.10.0.0/16` |
| VPC B CIDR | `10.20.0.0/16` |
| AWS Region | `ap-south-1` |
| Expected Status | `Active` |

The peering connection allows instances in the two VPCs to communicate using private IPv4 addresses when routing tables and security rules permit the traffic.

VPC peering does not automatically configure routes. The appropriate route tables in both VPCs must contain routes to the remote VPC CIDR.

---

## 6. Internet Gateway

| Property | Value |
|---|---|
| Name | `ewb-igw-a` |
| Internet Gateway ID | `igw-030431e12118d6dad` |
| Attached VPC | `ewb-vpc-a` |
| Purpose | Internet connectivity for resources configured to use the public subnet |

The Internet Gateway is attached to VPC A. The bastion host uses the public access path to provide administrative SSH connectivity to the private instances.

**VPC B does not require an Internet Gateway for this project's VPC-to-VPC communication.** Traffic between App-A and App-B uses the VPC peering connection and the corresponding route-table entries.

---

## 7. Route Table Inventory

### Route Table IDs

| Route Table | Name | Resource ID |
|---|---|---|
| Public route table | `ewb-rt-a-public` | `rtb-01b8e85216ac7c4d2` |
| VPC A private route table | `ewb-rt-a-private` | `rtb-07be1132cea4a051d` |
| VPC B private route table | `ewb-rt-b-private` | `rtb-0e06e4260a368e14d` |

### 7.1 VPC A Public Route Table

**Route table:** `ewb-rt-a-public`

| Destination | Target | Purpose |
|---|---|---|
| `10.10.0.0/16` | `local` | Communication within VPC A |
| `10.20.0.0/16` | `pcx-01934796beac9a6c4` | Route to VPC B |
| `0.0.0.0/0` | `igw-030431e12118d6dad` | Internet-bound traffic |

### 7.2 VPC A Private Route Table

**Route table:** `ewb-rt-a-private`

| Destination | Target | Purpose |
|---|---|---|
| `10.10.0.0/16` | `local` | Communication within VPC A |
| `10.20.0.0/16` | `pcx-01934796beac9a6c4` | Route to VPC B |

**Subnet association:** `ewb-a-private`

### 7.3 VPC B Private Route Table

**Route table:** `ewb-rt-b-private`

| Destination | Target | Purpose |
|---|---|---|
| `10.20.0.0/16` | `local` | Communication within VPC B |
| `10.10.0.0/16` | `pcx-01934796beac9a6c4` | Route to VPC A |

**Subnet association:** `ewb-b-private`

---

## 8. Security Group Inventory

### 8.1 Bastion Security Group

| Property | Value |
|---|---|
| Name | `ewb-sg-bastion` |
| Security Group ID | Verify in AWS Console |

**Intended inbound rule:**

| Protocol | Port | Source | Purpose |
|---|---|---|---|
| TCP | 22 | Administrator's current public IPv4 address `/32` | SSH access to bastion |

SSH should not be exposed to `0.0.0.0/0` unless there is a specific, justified requirement.

### 8.2 App-A Security Group

| Property | Value |
|---|---|
| Name | `ewb-sg-app-a` |
| Security Group ID | Verify in AWS Console |

**Intended inbound rules:**

| Protocol | Port | Source | Purpose |
|---|---|---|---|
| TCP | 22 | `10.10.1.26/32` | SSH from bastion |
| ICMP Echo Request | N/A | `10.20.1.10/32` | Allow ping requests from App-B |

### 8.3 App-B Security Group

| Property | Value |
|---|---|
| Name | `ewb-sg-app-b` |
| Security Group ID | `sg-09d1174a8b11ca1d9` |

**Intended inbound rules:**

| Protocol | Port | Source | Purpose |
|---|---|---|---|
| TCP | 22 | `10.10.1.26/32` | SSH from bastion |
| ICMP Echo Request | N/A | `10.10.2.10/32` | Allow ping requests from App-A |

Security groups are stateful. Verify the live inbound and outbound rules in AWS before final submission.

---

## 9. Network ACL Inventory

| Subnet | Network ACL ID | Recorded Configuration |
|---|---|---|
| App-A private subnet | `acl-0c2be1c848b953608` | Inbound and outbound rules recorded as allowing all traffic |
| App-B private subnet | `acl-035c8991c4f760e19` | Inbound and outbound rules recorded as allowing all traffic |

The network ACLs were recorded as allowing all inbound and outbound traffic. Verify the current rules in the AWS Console before relying on this configuration.

Network ACLs are stateless, unlike security groups.

---

## 10. Connectivity Test Reference

The following tests were recorded using the secondary private IP addresses configured for the project.

### App-A to App-B

```bash
ping -I 10.10.2.10 -c 4 -W 2 10.20.1.10
```

| Metric | Result |
|---|---|
| Packets transmitted | 4 |
| Packets received | 4 |
| Packet loss | 0% |
| Average round-trip time | 0.177 ms |

### App-B to App-A

```bash
ping -I 10.20.1.10 -c 4 -W 2 10.10.2.10
```

| Metric | Result |
|---|---|
| Packets transmitted | 4 |
| Packets received | 4 |
| Packet loss | 0% |
| Average round-trip time | 0.167 ms |

These results document bidirectional ICMP connectivity using the specified source and destination addresses. They do not independently verify every security rule or SSH connection.

---

## 11. SSH Access Reference

The administrative access path is:

```text
Kali Linux
    |
    | SSH / TCP 22
    v
Bastion Host — VPC A public subnet
    |
    +---- SSH ----> App-A — VPC A private subnet
    |
    +---- SSH ----> App-B — VPC B private subnet via VPC peering
```

The application instances use private IP addresses and are accessed through the bastion. Verify the current bastion public IP, login username, SSH key, and target IPs before connecting.

---

## 12. Screenshot Evidence

Store project screenshots in the `screenshots/` directory.

| Filename | Evidence |
|---|---|
| `01-vpc-a-created.png` | VPC A configuration |
| `02-vpc-b-created.png` | VPC B configuration |
| `03-subnets-created.png` | Subnet configuration |
| `04-internet-gateway.png` | Internet Gateway attachment |
| `05-route-tables.png` | Route tables and subnet associations |
| `06-ec2-instances.png` | EC2 instances and private IP addresses |
| `07-vpc-peering-active.png` | Active VPC peering connection |
| `08-security-groups.png` | Security group rules |
| `09-app-a-to-app-b-ping.png` | App-A to App-B connectivity test |
| `10-app-b-to-app-a-ping.png` | App-B to App-A connectivity test |
| `11-ssh-through-bastion-app-a.png` | SSH connection to App-A |
| `12-ssh-through-bastion-app-b.png` | SSH connection to App-B |
| `13-baseline-failure.png` | Baseline failure, if required |
| `14-controlled-icmp-failure.png` | Controlled ICMP failure, if required |
| `15-icmp-recovery.png` | Connectivity recovery, if required |
| `16-cleanup-evidence.png` | Cleanup evidence, if required |

Only include genuine screenshots of the actual AWS environment. Update the evidence status as screenshots are captured.

---

## 13. Resource Verification Checklist

Before submitting the project:

- [ ] Confirm VPC A and VPC B IDs and CIDR ranges.
- [ ] Confirm subnet IDs, CIDRs, and route-table associations.
- [ ] Confirm the peering connection is `Active`.
- [ ] Verify reciprocal routes to the remote VPC CIDRs.
- [ ] Verify the current bastion public and private IP addresses.
- [ ] Confirm the primary and secondary private IPs of App-A and App-B.
- [ ] Confirm security group IDs and inbound/outbound rules.
- [ ] Confirm the current Network ACL rules.
- [ ] Capture genuine screenshots for the project evidence.
- [ ] Remove or correct any inventory values that have changed.

---

## 14. Summary

The AWS VPC peering project connects two non-overlapping VPC networks in the Mumbai region. VPC A hosts the bastion and App-A, while VPC B hosts App-B in a private subnet. Reciprocal route-table entries direct private traffic through the peering connection, and security groups control access to SSH and ICMP services.

The recorded connectivity tests show successful ICMP communication in both directions between the specified secondary private IP addresses. Final submission should include verified AWS resource details and actual screenshots of the configuration and test results.
