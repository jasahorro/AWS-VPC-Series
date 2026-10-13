# AWS Networking: Easily Confused Pairs

Side-by-side tables for the pieces of AWS networking that are easiest to mix up. Back to the [series overview](README.md).

## Contents

1. [Security groups vs network ACLs](#1-security-groups-vs-network-acls)
2. [Internet gateway vs NAT gateway vs VPC endpoint](#2-internet-gateway-vs-nat-gateway-vs-vpc-endpoint)
3. [VPC peering vs VPC endpoints](#3-vpc-peering-vs-vpc-endpoints)
4. [Gateway endpoints vs interface endpoints](#4-gateway-endpoints-vs-interface-endpoints)
5. [Public vs private subnets](#5-public-vs-private-subnets)
6. [Public IP vs Elastic IP vs private IP](#6-public-ip-vs-elastic-ip-vs-private-ip)
7. [Key pairs vs access keys vs IAM roles](#7-key-pairs-vs-access-keys-vs-iam-roles)
8. [Bucket policies vs endpoint policies vs IAM policies](#8-bucket-policies-vs-endpoint-policies-vs-iam-policies)
9. [Ping vs curl vs SSH](#9-ping-vs-curl-vs-ssh)
10. [Default vs custom: what's allowed out of the box](#10-default-vs-custom-whats-allowed-out-of-the-box)

---

## 1. Security groups vs network ACLs

| | Security group | Network ACL |
|---|---|---|
| Protects | One **instance** (its network interface) | A whole **subnet** |
| State | **Stateful**: if a request is allowed in, the reply goes out automatically | **Stateless**: you need rules for both directions, including replies |
| Rule types | **Allow only** | **Allow and deny** |
| Rule order | All rules checked together | Checked **in number order**, and the first match wins |
| Default (custom one) | Inbound: nothing allowed. Outbound: everything allowed | **Everything denied**, both ways |
| Default (the VPC's default one) | Allows traffic from members of the same group | **Everything allowed**, both ways |
| Analogy | The lock on each house's door | The guard at the street entrance checking everyone in and out |
| Projects | 2, 4, 5, 6 | 2, 3, 5 |

> **Memory trick:** **S**ecurity groups are **S**tateful. NACLs can say **N**o (deny).

## 2. Internet gateway vs NAT gateway vs VPC endpoint

| | Internet gateway | NAT gateway | VPC endpoint |
|---|---|---|---|
| Purpose | Two-way internet access for public subnets | **Outbound-only** internet for private subnets | **Private** access to AWS services (like S3) |
| Who can start a connection | Both the inside and the internet | Only the inside (replies come back) | Only the inside, to that one service |
| Route in the route table | `0.0.0.0/0 → igw-…` | `0.0.0.0/0 → nat-…` (in the **private** route table) | `pl-… (service prefix list) → vpce-…` |
| Lives in | Attached to the VPC | A **public** subnet | Attached to the VPC (gateway type) |
| Traffic uses the public internet? | Yes | Yes | **No**, it stays on the AWS network |
| Cost | Free | 💲 Hourly + per GB | Gateway endpoint: free. Interface endpoint: 💲 |
| Analogy | The city gate | A one-way exit with a mail-forwarding service | A private tunnel to one building |
| Projects | 1, 2 | 4 | 9 |

## 3. VPC peering vs VPC endpoints

| | VPC peering | VPC endpoint |
|---|---|---|
| Connects | **Your VPC ↔ another VPC** | **Your VPC → an AWS service** |
| Route | `other VPC's CIDR → pcx-…` in **both** VPCs | `service prefix list → vpce-…` |
| Requirement | The CIDR blocks **must not overlap** | The route table must be associated with the endpoint |
| Transitive? | **No**: if A↔B and B↔C, A still can't reach C | Not applicable |
| Projects | 6, 7 | 9 |

## 4. Gateway endpoints vs interface endpoints

| | Gateway endpoint | Interface endpoint |
|---|---|---|
| Services | **S3 and DynamoDB only** | Most AWS services |
| How it works | Adds a **route** (prefix list → `vpce-…`) | Adds a **network interface** with a private IP in your subnet (AWS PrivateLink) |
| Security | Endpoint policy | Endpoint policy + **security groups** |
| Cost | **Free** | Hourly + per GB |
| Project | 9 | — |

## 5. Public vs private subnets

| | Public subnet | Private subnet |
|---|---|---|
| What makes it so | Its route table has **`0.0.0.0/0 → internet gateway`** | **No** route to an internet gateway |
| Instances need | A public IP to be reachable | Nothing public. A NAT gateway if they need outbound internet |
| Typical contents | Web servers, load balancers, NAT gateways | Databases, app servers |
| Common mistake | Thinking the **name** "Public" makes it public | Leaving it on the main route table, which may have an IGW route |
| Projects | 1, 2 | 3, 4 |

## 6. Public IP vs Elastic IP vs private IP

| | Private IP | Auto-assigned public IP | Elastic IP |
|---|---|---|---|
| Reachable from | **Inside** the VPC (and peered VPCs) | The internet | The internet |
| Changes when the instance stops and starts? | No | **Yes**, you get a new one | **No, it's static** |
| How you get it | Automatic | Turn on "auto-assign public IPv4" on the subnet | Allocate it to your account, then associate it |
| Cost | Free | 💲 (public IPv4 addresses are charged hourly) | 💲 Charged hourly, **including when it isn't attached** |
| Projects | All | 1, 4 | 6 |

## 7. Key pairs vs access keys vs IAM roles

| | Key pair | Access key | IAM role |
|---|---|---|---|
| Lets you | **Log in to an EC2 instance** over SSH | Have an app or server **call AWS services** | Give an AWS service or instance **temporary** permissions |
| Made of | A public key (on AWS) + a private `.pem` (you keep it) | Access key ID + secret access key | A permissions policy + a **trust policy** saying who can use the role |
| Lifetime | Until you delete it | **Long-term**, until you delete it | Temporary credentials that **rotate automatically** |
| Best practice | Or use EC2 Instance Connect instead | Avoid on EC2. Delete when you're done | ✅ The recommended way for EC2 to call AWS |
| Never | Push the `.pem` to GitHub | Put it in code or GitHub | — |
| Projects | 4 | 8, 9 | 7 (flow logs role) |

## 8. Bucket policies vs endpoint policies vs IAM policies

| | IAM policy | Bucket policy | Endpoint policy |
|---|---|---|---|
| Attached to | A user, group or role (**identity-based**) | An S3 bucket (**resource-based**) | A VPC endpoint |
| Answers | "What can **this identity** do?" | "Who can do what **to this bucket**?" | "What can travel **through this endpoint**?" |
| Example from the series | The flow logs role's permissions (create log streams, put log events) | Deny everything unless `aws:sourceVpce` = my endpoint | Switched Allow → Deny as a kill switch |
| Projects | 7 | 9 | 9 |

> A request has to pass **every** policy that applies, and an **explicit Deny always wins** over any Allow.

## 9. Ping vs curl vs SSH

| | `ping` | `curl` | SSH / EC2 Instance Connect |
|---|---|---|---|
| Protocol | **ICMP** (no port) | **HTTP / HTTPS** (TCP 80 / 443) | **SSH** (TCP 22) |
| Network layer | Network layer | Application layer | Application layer |
| Tests | "Can I reach that IP at all?" | "Can I fetch a web page or API?" | "Can I log in to the machine?" |
| Security group rule needed | All ICMP - IPv4 | HTTP / HTTPS | SSH, port 22 |
| Projects | 5, 6, 7 | 5 | 5, 6, 8, 9 |

## 10. Default vs custom: what's allowed out of the box

| Resource | Default (comes with the VPC) | Custom (one you create) |
|---|---|---|
| **Network ACL** | Allows **all** traffic in and out | **Denies all** traffic until you add rules |
| **Security group** | Allows traffic from **other members of the same group**. All outbound allowed | **No inbound** rules. All outbound allowed |
| **Route table** | The **main** route table. New subnets use it automatically | Only has the `local` route until you add more |
| **VPC** | Comes with a subnet per AZ and an internet gateway already attached | Nothing until you build it |

> **The lesson from project 3:** a new subnet is automatically associated with the **main** route table. If that table has an IGW route, your "private" subnet is actually public. Give private subnets their own route table.
