# AWS VPC Peering Lab (EWB Courses)

Two private VPCs connected with a VPC peering connection, with a bastion host for administration.
Built in AWS region **ap-southeast-2 (Sydney)**.

## Architecture

| Resource | Name | CIDR / IP |
|---|---|---|
| VPC A | ewb-vpc-a | 10.10.0.0/16 |
| VPC B | ewb-vpc-b | 10.20.0.0/16 (no internet gateway) |
| Public subnet A | ewb-public-subnet-a | 10.10.1.0/24, bastion |
| Private subnet A | ewb-private-subnet-a | 10.10.2.0/24, App-A (10.10.2.10) |
| Private subnet B | ewb-b-private | 10.20.1.0/24, App-B (10.20.1.10) |
| Peering | ewb-a-b-peer | VPC A (requester) to VPC B (accepter) |

## Routes
- `ewb-rt-a-public`: 0.0.0.0/0 to the internet gateway; 10.20.0.0/16 to the peering connection
- `ewb-rt-a-private`: 10.20.0.0/16 to the peering connection
- `ewb-rt-b-private`: 10.10.0.0/16 to the peering connection (return path)

## Security groups
Every rule allows one specific host (/32); nothing is open to 0.0.0.0/0.
- ewb-sg-bastion: SSH from my public IP
- ewb-sg-app-a: SSH from the bastion's private IP; ICMP from App-B
- ewb-sg-app-b: SSH from the bastion's private IP; ICMP echo request from App-A

## Tests
| Test | Result |
|---|---|
| Ping App-A to App-B after peering and routes | 4/4 replies (screenshot in report) |
| Ping App-B to App-A | 4/4 replies (screenshot in report) |
| App-A and App-B have no public IPv4 | Confirmed (screenshot in report) |
| Baseline before peering, ProxyJump SSH, controlled ICMP failure | Not captured |

See the report for what was and was not captured, and for one deviation: I reached the private hosts by hopping from the bastion because ProxyJump from Windows failed.

## Terraform
The `terraform/` folder is an alternative way to build the same lab (staged with `enable_peering` and `break_icmp`). It was not used for my console build and has not been validated against AWS.

## Cleanup
Delete in this order: instances, peering connection, security groups, subnets, internet gateway, VPCs, key pair.

## What I learned
Active peering is not enough. Both route tables and the destination security group must allow the traffic, and peering does not give VPC B internet access or transit through VPC A.

## License
MIT
