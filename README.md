# AWS VPC Series

A recap of the **9-project NextWork AWS networking series** I completed between September and October 2026. Each project has its own repo with a write-up (README) and a cheat sheet of lesson notes with a self-quiz.

**Also in this repo:**
- [COMPARISONS.md](COMPARISONS.md): side-by-side tables for the AWS networking pieces that are easiest to mix up
- [QUIZ.md](QUIZ.md): a mixed self-quiz covering the whole series

## Contents

1. [The projects](#the-projects)
2. [What I built, end to end](#what-i-built-end-to-end)
3. [How it fits together](#how-it-fits-together)
4. [Skills demonstrated](#skills-demonstrated)
5. [Troubleshooting stories](#troubleshooting-stories)

---

## The projects

| # | Project | What I learned | Links |
|---|---|---|---|
| 1 | **Build a Virtual Private Cloud** | VPCs, IPv4 and CIDR blocks, subnets, auto-assign public IPs, internet gateways | [README](https://github.com/jasahorro/Build-A-Virtual-Private-Cloud) · [Cheat sheet](https://github.com/jasahorro/Build-A-Virtual-Private-Cloud/blob/main/CHEATSHEET.md) |
| 2 | **VPC Traffic Flow and Security** | Route tables, security groups, network ACLs, ports and protocols, inbound vs outbound | [README](https://github.com/jasahorro/VPC-Traffic-Flow-and-Security) · [Cheat sheet](https://github.com/jasahorro/VPC-Traffic-Flow-and-Security/blob/main/CHEATSHEET.md) |
| 3 | **Creating a Private Subnet** | Non-overlapping CIDRs, a dedicated private route table, a dedicated network ACL | [README](https://github.com/jasahorro/Creating-a-Private-Subnet) · [Cheat sheet](https://github.com/jasahorro/Creating-a-Private-Subnet/blob/main/CHEATSHEET.md) |
| 4 | **Launching VPC Resources** | Public and private EC2 servers, key pairs and SSH, the "VPC and more" wizard, NAT gateways | [README](https://github.com/jasahorro/Launching-VPC-Resources) · [Cheat sheet](https://github.com/jasahorro/Launching-VPC-Resources/blob/main/CHEATSHEET.md) |
| 5 | **Testing VPC Connectivity** | Testing every layer with EC2 Instance Connect, `ping` and `curl`, and fixing what failed | [README](https://github.com/jasahorro/Testing-VPC-Connectivity) · [Cheat sheet](https://github.com/jasahorro/Testing-VPC-Connectivity/blob/main/CHEATSHEET.md) |
| 6 | **VPC Peering** | Connecting two VPCs privately, peering routes, Elastic IPs | [README](https://github.com/jasahorro/VPC-Peering) · [Cheat sheet](https://github.com/jasahorro/VPC-Peering/blob/main/CHEATSHEET.md) |
| 7 | **VPC Monitoring with Flow Logs** | VPC Flow Logs, CloudWatch log groups, IAM roles and trust policies, Logs Insights queries | [README](https://github.com/jasahorro/VPC-Monitoring-with-Flow-Logs) · [Cheat sheet](https://github.com/jasahorro/VPC-Monitoring-with-Flow-Logs/blob/main/CHEATSHEET.md) |
| 8 | **Access S3 from a VPC** | The AWS CLI on EC2, IAM access keys, `aws configure`, `aws s3` commands | [README](https://github.com/jasahorro/Access-S3-from-a-VPC) · [Cheat sheet](https://github.com/jasahorro/Access-S3-from-a-VPC/blob/main/CHEATSHEET.md) |
| 9 | **VPC Endpoints** | S3 gateway endpoints, bucket policies locked to an endpoint, endpoint policies | [README](https://github.com/jasahorro/VPC-Endpoints) · [Cheat sheet](https://github.com/jasahorro/VPC-Endpoints/blob/main/CHEATSHEET.md) |

## What I built, end to end

Each project added a piece to the same idea: a private network in AWS that only lets in the traffic I choose.

```
                                   Internet 🌐
                                       │
                            ┌──────────┴──────────┐
                            │  Internet gateway   │ ①
                            └──────────┬──────────┘
 ┌──────────────── VPC 10.0.0.0/16 ────┼──────────────────────────────┐        ┌──── 2nd VPC ────┐
 │                                     │                              │        │                 │
 │  ┌─ Public subnet ──────────────────┼───┐  ┌─ Private subnet ────┐ │  ⑥     │   EC2 instance  │
 │  │ route: 0.0.0.0/0 → IGW ②         │   │  │ route: local only ③ │ │◄──────►│                 │
 │  │ NACL ② (subnet firewall)         │   │  │ NACL ③ (deny all)   │ │ peering│                 │
 │  │  ┌────────────────────┐          │   │  │ ┌─────────────────┐ │ │        └─────────────────┘
 │  │  │ Public server  ④   │──ping⑤──────────►│ Private server ④│ │ │
 │  │  │ SG ② (instance fw) │          │   │  │ │ SG: from public │ │ │
 │  │  └─────────┬──────────┘          │   │  │ └─────────────────┘ │ │
 │  └────────────┼──────────────────────────┘  └─────────────────────┘ │
 │               │ AWS CLI + access keys ⑧         Flow Logs ⑦ ──► CloudWatch Logs
 │               │ route: S3 prefix list → vpce ⑨                      │
 └───────────────┼─────────────────────────────────────────────────────┘
                 ▼
     ┌────────────────────────┐      ┌───────────────────────────────────────┐
     │ S3 gateway endpoint ⑨  │ ───► │ S3 bucket (bucket policy: only via    │
     └────────────────────────┘      │ this endpoint) ⑧ ⑨                    │
                                     └───────────────────────────────────────┘
```

The circled numbers are the project where each piece was built.

## How it fits together

Traffic in AWS has to pass several layers. If any one of them says no, the traffic doesn't get through. Most of this series came down to working out which layer was blocking.

| Layer | Job | Controls | Projects |
|---|---|---|---|
| **Internet gateway** | The VPC's door to the internet | Whether the VPC can reach the internet at all | 1, 2 |
| **Route table** | Decides **where** traffic goes | `0.0.0.0/0 → IGW`, `local`, `→ pcx-…` (peering), `pl-… → vpce-…` (endpoint) | 2, 3, 6, 9 |
| **Network ACL** | Subnet firewall, **stateless** | Numbered allow and deny rules, inbound **and** outbound | 2, 3, 5 |
| **Security group** | Instance firewall, **stateful** | Allow rules only. Replies are allowed back automatically | 2, 4, 5, 6 |
| **Public IP** | Lets an instance be reached from the internet | Auto-assign public IPv4, Elastic IPs | 1, 6 |
| **IAM** | Decides **who** can call AWS | Access keys, roles, trust policies | 7, 8 |
| **Resource policies** | Rules on the resource itself | Bucket policies, endpoint policies | 9 |
| **Monitoring** | Shows what actually happened | Flow logs show ACCEPT and REJECT per connection | 7 |

**The checklist I ended up with** for "why can't A reach B?":
1. Is there a **route** from A to B, and a route back?
2. Does the **NACL** allow it **both ways**?
3. Does B's **security group** allow it in?
4. Does it need a **public IP**, an **internet gateway**, a **peering connection** or an **endpoint**?
5. For AWS services, do the **credentials and policies** allow it?
6. Still stuck? **Flow logs** show whether the traffic arrived, and whether it was accepted or rejected.

## Skills demonstrated

**Networking**
- VPC design and **CIDR planning** (non-overlapping ranges, subnet sizing)
- Public and private **subnetting** across Availability Zones
- **Route tables** (internet gateway, local, peering and endpoint routes)
- **Internet gateways**, NAT gateway concepts, **Elastic IPs**
- **VPC peering** between two VPCs
- **VPC gateway endpoints** for private access to S3

**Security**
- **Security groups** vs **network ACLs** (stateful vs stateless, allow vs deny)
- Least-privilege inbound rules (SSH, ICMP, HTTP; source set to a CIDR or another security group)
- **IAM** access keys, roles and **trust policies**
- **S3 bucket policies** and **VPC endpoint policies** with conditions (`aws:sourceVpce`)

**Operations and troubleshooting**
- **Testing connectivity** layer by layer with EC2 Instance Connect (SSH), `ping` (ICMP) and `curl` (HTTP)
- **Monitoring** with VPC Flow Logs and **Amazon CloudWatch** log groups
- **CloudWatch Logs Insights** queries for traffic analysis
- **AWS CLI** for EC2, VPC and S3 (`aws configure`, `aws s3 ls / cp / rm / rb`, `aws ec2 create-vpc`)
- Cleaning up resources to avoid charges (NAT gateways, Elastic IPs, access keys)

## Troubleshooting stories

Real problems I hit during the series, and how I fixed them.

**1. "I can't connect to my own server"** · [Testing VPC Connectivity](https://github.com/jasahorro/Testing-VPC-Connectivity)
- **Problem:** EC2 Instance Connect failed on my public server.
- **Cause:** its security group only allowed inbound **HTTP**, but Instance Connect uses **SSH (port 22)**.
- **Fix:** added an inbound SSH rule on port 22.
- **Lesson:** match the rule to the protocol the tool actually uses.

**2. "Ping gets 100% packet loss"** · [Testing VPC Connectivity](https://github.com/jasahorro/Testing-VPC-Connectivity)
- **Problem:** pinging the private server from the public server got no replies.
- **Cause:** **two layers** were blocking ICMP: the private subnet's NACL, which denies everything by default, and the private server's security group.
- **Fix:** allowed ICMP in the NACL (inbound **and** outbound, because NACLs are stateless) and in the security group.
- **Lesson:** fixing one layer isn't enough when two layers are blocking.

**3. "Instance Connect fails, but the security group is fine"** · [VPC Peering](https://github.com/jasahorro/VPC-Peering)
- **Problem:** Instance Connect still failed after SSH was allowed.
- **Cause:** the instance had **no public IPv4 address**, because auto-assign was off.
- **Fix:** allocated an **Elastic IP** and associated it with the instance.
- **Lesson:** Instance Connect needs a public subnet *and* a public IP.

**4. "Ping works by public IP but not by private IP"** · [VPC Monitoring with Flow Logs](https://github.com/jasahorro/VPC-Monitoring-with-Flow-Logs)
- **Problem:** Instance 1 couldn't ping Instance 2's private IP, but pinging its public IP worked.
- **Cause:** the public ping went over the internet. Privately, there was **no route** for VPC 2's CIDR (`10.2.0.0/16`), because no peering connection existed yet.
- **Fix:** created a peering connection and added routes for each other's CIDR **in both VPCs**.
- **Lesson:** a response from the public IP proves the instance is up. The private path then needs its own route.

**5. "The endpoint exists, but access is still denied"** · [VPC Endpoints](https://github.com/jasahorro/VPC-Endpoints)
- **Problem:** after locking the bucket to my VPC endpoint, my EC2 instance was denied too.
- **Cause:** the subnet's **route table didn't use the endpoint**, so S3 traffic still went out through the internet gateway and arrived without the endpoint ID.
- **Fix:** Endpoints → **Manage route tables** → added the public route table. That created a route from S3's prefix list to `vpce-…`.
- **Lesson:** a gateway endpoint does nothing until a route table points to it.

---

*Built with [NextWork](https://nextwork.ai). Each project is in its own repo, linked above.*
