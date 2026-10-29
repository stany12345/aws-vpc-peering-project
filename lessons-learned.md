# Lessons Learned — AWS VPC Peering

## 1. Overview

This project provided practical experience configuring and validating private network connectivity between two Amazon Virtual Private Clouds (VPCs) using AWS VPC Peering.

The implementation involved VPC creation, subnet configuration, route-table management, security groups, a bastion host, and connectivity testing between EC2 instances in separate VPCs.

This document summarizes the main technical concepts, troubleshooting lessons, security considerations, and best practices learned during the project.

---

## 2. VPC Design and IP Address Planning

### What I Learned

- Each VPC requires an appropriate IPv4 CIDR block.
- VPCs must use non-overlapping CIDR ranges for straightforward private routing through VPC Peering.
- Subnets divide a VPC's address space into smaller network segments.
- Public and private subnets serve different purposes depending on their routing and resource configuration.
- Proper IP address planning helps avoid routing conflicts and simplifies troubleshooting.

### Applied in This Project

| Network | CIDR Block | Purpose |
|---|---|---|
| VPC A | `10.10.0.0/16` | Bastion host and App-A |
| VPC A public subnet | `10.10.1.0/24` | Bastion host |
| VPC A private subnet | `10.10.2.0/24` | App-A |
| VPC B | `10.20.0.0/16` | App-B network |
| VPC B private subnet | `10.20.1.0/24` | App-B |

**Key takeaway:** Plan CIDR ranges and subnet allocations before creating AWS networking resources.

---

## 3. Understanding VPC Peering

### What I Learned

- VPC Peering enables private communication between two VPCs using private IP addresses.
- The peering connection must be established and active before it can carry traffic.
- Creating a peering connection does not automatically configure the required routes.
- Both VPCs need appropriate route-table entries for bidirectional communication.
- VPC Peering is not transitive. A peered VPC does not automatically provide onward routing to other networks.

### Applied in This Project

The project uses the peering connection:

`pcx-01934796beac9a6c4`

The route tables direct traffic for the remote VPC CIDR through this peering connection.

**Key takeaway:** A peering connection alone is insufficient; correct routing on both sides is essential.

---

## 4. Route Tables and Traffic Flow

### What I Learned

Route tables determine where network traffic is sent based on its destination IP address.

The important routes in this project include:

| Route Table | Destination | Target |
|---|---|---|
| VPC A public | `10.10.0.0/16` | Local |
| VPC A public | `10.20.0.0/16` | VPC Peering |
| VPC A public | `0.0.0.0/0` | Internet Gateway |
| VPC A private | `10.10.0.0/16` | Local |
| VPC A private | `10.20.0.0/16` | VPC Peering |
| VPC B private | `10.20.0.0/16` | Local |
| VPC B private | `10.10.0.0/16` | VPC Peering |

### Troubleshooting Lessons

When connectivity fails, check:

1. Whether the peering connection is active.
2. Whether the source subnet uses the expected route table.
3. Whether the source route table contains a route to the destination VPC.
4. Whether the destination VPC has a route back to the source VPC.
5. Whether security groups and network ACLs permit the traffic.

**Key takeaway:** Successful communication requires a valid forward path and a valid return path.

---

## 5. Security Groups and Network ACLs

### What I Learned

- Security groups act as stateful virtual firewalls associated with network interfaces.
- Network ACLs operate at the subnet level and are stateless.
- Inbound and outbound rules must be considered when troubleshooting network access.
- ICMP rules are needed when ping is used to test connectivity.
- Restricting SSH access to known source addresses reduces unnecessary exposure.

### Applied in This Project

The intended security group configuration permits:

- SSH access to the bastion from the administrator's public IPv4 address.
- SSH access to private instances from the bastion's private IP.
- ICMP Echo Requests between the designated application IP addresses.

### Security Best Practices

- Avoid allowing SSH from `0.0.0.0/0` unless explicitly required and appropriately justified.
- Prefer narrow source CIDRs over unrestricted inbound rules.
- Review security group rules after troubleshooting.
- Verify Network ACL rules when traffic remains blocked despite correct security groups.
- Remove temporary rules that are no longer required.

**Key takeaway:** Use least-privilege network rules and allow only the traffic required by the project.

---

## 6. Bastion Host and Secure SSH Access

### What I Learned

A bastion host provides a controlled entry point for accessing instances in private subnets.

Instead of exposing App-A and App-B directly to the internet, an administrator can connect to the bastion and use it as an SSH jump host.

The intended access path is:

```text
Kali Linux
    |
    | SSH / TCP 22
    v
Bastion Host
    |
    +---- SSH ----> App-A
    |
    +---- SSH ----> App-B through VPC Peering
```

SSH `ProxyJump` can simplify access by allowing the SSH client to connect through the bastion automatically.

### Security Considerations

- Keep private SSH keys secure.
- Restrict bastion SSH access to the administrator's current public IP address where practical.
- Use the correct operating-system username and private key.
- Verify that the bastion can reach the destination instance on TCP port 22.
- Avoid assigning public IP addresses to private application instances when direct internet access is unnecessary.

**Key takeaway:** A bastion host provides a practical administrative access pattern for private cloud workloads.

---

## 7. Private IP Addresses and Connectivity Testing

### What I Learned

EC2 instances can have primary and secondary private IP addresses. The source IP used by a connectivity test can affect whether a security group rule matches the traffic.

In this project, the recorded primary and secondary addresses differ:

| Instance | Primary Private IP | Secondary IP Used for Testing |
|---|---|---|
| App-A | `10.10.2.53` | `10.10.2.10` |
| App-B | `10.20.1.97` | `10.20.1.10` |

The tests explicitly selected the secondary source IP addresses.

### App-A to App-B

```bash
ping -I 10.10.2.10 -c 4 -W 2 10.20.1.10
```

Recorded result:

- Packets transmitted: 4
- Packets received: 4
- Packet loss: 0%
- Average round-trip time: 0.177 ms

### App-B to App-A

```bash
ping -I 10.20.1.10 -c 4 -W 2 10.10.2.10
```

Recorded result:

- Packets transmitted: 4
- Packets received: 4
- Packet loss: 0%
- Average round-trip time: 0.167 ms

### Troubleshooting Lessons

- Confirm that the intended private IP addresses are assigned to the network interfaces.
- Check the source address used by the test.
- Verify that security group rules match the actual source IP.
- Use explicit source addresses when testing a particular secondary IP configuration.
- Remember that a successful ping verifies ICMP connectivity, not every application protocol or security requirement.

**Key takeaway:** Understand the actual source and destination IP addresses before troubleshooting connectivity.

---

## 8. Why VPC B Does Not Need an Internet Gateway

### What I Learned

An Internet Gateway is not required for private communication between two VPCs connected through VPC Peering.

In this project, App-A and App-B communicate through:

1. A route to the destination VPC CIDR.
2. The VPC peering connection.
3. A corresponding route back to the source VPC.
4. Security rules that allow the required traffic.

VPC B can therefore communicate privately with VPC A without requiring its own Internet Gateway for this use case.

**Key takeaway:** Private VPC-to-VPC communication and internet connectivity are separate networking requirements.

---

## 9. Troubleshooting Methodology

This project reinforced the importance of checking network connectivity systematically rather than changing multiple settings at once.

When a connection fails, use the following sequence:

1. Confirm that both EC2 instances are running.
2. Verify their VPCs, subnets, network interfaces, and private IP addresses.
3. Confirm that the VPC peering connection is active.
4. Inspect route-table associations and destination routes.
5. Review security group inbound and outbound rules.
6. Review Network ACL rules.
7. Test connectivity using the intended source and destination addresses.
8. Check SSH configuration, usernames, key permissions, and bastion reachability when troubleshooting SSH.
9. Repeat the test after making one controlled change.
10. Record the result and capture supporting evidence.

**Key takeaway:** A structured troubleshooting process makes network issues easier to isolate and resolve.

---

## 10. Documentation and Evidence Management

### What I Learned

Cloud projects should include clear documentation of both the intended architecture and the implemented configuration.

Useful evidence includes:

- VPC names, CIDR ranges, and resource IDs.
- Subnet configurations and route-table associations.
- VPC peering status.
- Security group rules.
- EC2 instance details.
- Successful connectivity tests.
- SSH access through the bastion.
- Any required failure, recovery, or cleanup demonstrations.

Screenshots should show the actual AWS console or terminal output and should use consistent filenames.

**Key takeaway:** Good documentation makes cloud projects easier to review, reproduce, troubleshoot, and present in a technical portfolio.

---

## 11. Improvements for Future Projects

The following improvements can make future AWS networking projects more secure and maintainable:

- Use infrastructure as code, such as AWS CloudFormation or Terraform, to make deployments reproducible.
- Maintain a centralized IP address and resource inventory.
- Use narrowly scoped security group rules.
- Consider AWS Systems Manager Session Manager where appropriate to reduce dependence on publicly reachable SSH bastions.
- Enable suitable logging and monitoring for operational visibility.
- Validate both connectivity and access restrictions.
- Capture evidence during each implementation stage rather than waiting until the end.
- Review resource usage and remove unused lab resources to control costs.

These are potential improvements for future iterations; they are not claims that every item was implemented in this project.

---

## 12. Final Summary

Completing this project strengthened my practical understanding of AWS networking, VPC Peering, subnet design, route tables, security groups, Network ACLs, private IP addressing, and bastion-based SSH access.

The recorded tests demonstrated successful bidirectional ICMP connectivity between App-A and App-B using their designated secondary private IP addresses.

The most important lesson was that reliable private connectivity depends on several components working together: correct network addressing, active peering, reciprocal routes, and appropriate security rules.

This project provides a foundation for further learning in AWS infrastructure administration, cloud networking, Linux system administration, and cloud security.

---

**Project:** AWS VPC Peering  
**AWS Region:** `ap-south-1` (Mumbai)  
**Documentation:** `learning/lessons-learned.md`
