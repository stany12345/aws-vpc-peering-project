# AWS VPC Peering Project

## Project Overview

This project demonstrates private communication between two Amazon Virtual Private Clouds (VPCs) using VPC Peering. It uses Amazon EC2 instances, custom route tables, security groups, and a bastion host for secure administration.

## Objectives

- Create two VPCs with non-overlapping IPv4 CIDR blocks.
- Configure public and private subnets.
- Establish a VPC peering connection.
- Configure routes in both VPCs.
- Restrict inbound access using security groups.
- Access private EC2 instances through a bastion host.
- Test connectivity and document troubleshooting results.

## Architecture

- **AWS Region:** Asia Pacific (Mumbai) — `ap-south-1`
- **VPC A:** `10.10.0.0/16`
- **VPC B:** `10.20.0.0/16`
- **App-A:** Intended private IP `10.10.2.10`
- **App-B:** Intended private IP `10.20.1.10`
- **Bastion:** Publicly accessible administration host in VPC A

See `documentation/architecture-diagram.png` for the network diagram.

## Technologies Used

- Amazon VPC
- VPC Peering
- Amazon EC2
- Route Tables
- Internet Gateway
- Security Groups
- Network ACLs
- Linux
- SSH
- ICMP

## Validation Results

| Test                                 | Result                       |
| ------------------------------------ | ---------------------------- |
| App-A to App-B ping                  | Passed                       |
| App-B to App-A ping                  | Passed                       |
| SSH access through bastion           | Document the verified result |
| Controlled ICMP failure and recovery | Document the verified result |

Both ping tests succeeded using the intended secondary private IP addresses, with zero packet loss.

## Repository Structure

- `documentation/` — Project report, architecture diagram, and resource inventory
- `screenshots/` — AWS configuration and testing evidence
- `learning/` — Lessons learned and troubleshooting notes

## Security Notes

- Application instances have no public IPv4 addresses.
- SSH access to private instances is routed through the bastion host.
- Security group rules are restricted to required sources.
- Private keys, passwords, and access tokens are not included in this repository.

## Author

**Name:** Stany Joseph
**Project:** EWB AWS VPC Peering
**Completion Date:** [Enter completion date]

## Report

See `documentation/project-report.pdf` for the complete project report.

