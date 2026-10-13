# AWS VPC Series — Self-Quiz

A mixed quiz covering all 9 projects. Try to answer before opening each one. The link under each answer goes to the cheat sheet with the full notes. Back to the [series overview](README.md).

**Sections:** [Foundations](#foundations) · [Routing](#routing) · [Security layers](#security-layers) · [Connectivity and troubleshooting](#connectivity-and-troubleshooting) · [Peering and monitoring](#peering-and-monitoring) · [Credentials and S3](#credentials-and-s3) · [Scenario questions](#scenario-questions)

---

## Foundations

<details><summary>1. In the city analogy, what are a Region, a VPC, a subnet and an internet gateway?</summary>

The Region is the country, the VPC is your private city, a subnet is a neighbourhood, and the internet gateway is the city gate.

📘 [Build a Virtual Private Cloud](https://github.com/jasahorro/Build-A-Virtual-Private-Cloud/blob/main/CHEATSHEET.md)
</details>

<details><summary>2. How many IP addresses are in a /16, a /24 and a /28? How many can you actually use in a /24 subnet?</summary>

65,536, 256 and 16, using 2^(32 − the number after the slash). A /24 subnet has **251** usable addresses, because AWS reserves 5 in every subnet.

📘 [Build a Virtual Private Cloud](https://github.com/jasahorro/Build-A-Virtual-Private-Cloud/blob/main/CHEATSHEET.md)
</details>

<details><summary>3. Can a subnet span two Availability Zones? Can two subnets in one VPC overlap?</summary>

No to both. A subnet lives in exactly one AZ, and subnet ranges in the same VPC must be unique.

📘 [Build a Virtual Private Cloud](https://github.com/jasahorro/Build-A-Virtual-Private-Cloud/blob/main/CHEATSHEET.md) · [Creating a Private Subnet](https://github.com/jasahorro/Creating-a-Private-Subnet/blob/main/CHEATSHEET.md)
</details>

<details><summary>4. Your VPC is 10.0.0.0/16 and your public subnet is 10.0.0.0/24. Give a valid CIDR for a private subnet.</summary>

Anything inside `10.0.0.0/16` that doesn't overlap `10.0.0.0/24`, for example `10.0.1.0/24`.

📘 [Creating a Private Subnet](https://github.com/jasahorro/Creating-a-Private-Subnet/blob/main/CHEATSHEET.md)
</details>

## Routing

<details><summary>5. What single route makes a subnet public?</summary>

`0.0.0.0/0 → internet gateway` in the subnet's route table. The subnet's name has nothing to do with it.

📘 [VPC Traffic Flow and Security](https://github.com/jasahorro/VPC-Traffic-Flow-and-Security/blob/main/CHEATSHEET.md)
</details>

<details><summary>6. What does the <code>local</code> route do, and can you delete it?</summary>

It lets every subnet in the VPC reach every other subnet inside the VPC's CIDR. You can't delete it.

📘 [VPC Traffic Flow and Security](https://github.com/jasahorro/VPC-Traffic-Flow-and-Security/blob/main/CHEATSHEET.md)
</details>

<details><summary>7. Why did the private subnet need its own route table?</summary>

New subnets use the VPC's **main** route table by default, and that one had a route to the internet gateway. A dedicated route table with only the `local` route keeps the subnet private.

📘 [Creating a Private Subnet](https://github.com/jasahorro/Creating-a-Private-Subnet/blob/main/CHEATSHEET.md)
</details>

<details><summary>8. Match each route target to its project: <code>igw-…</code>, <code>pcx-…</code>, <code>vpce-…</code>.</summary>

- `igw-…`: an internet gateway (projects 1–2)
- `pcx-…`: a peering connection (projects 6–7)
- `vpce-…`: a VPC endpoint, with an S3 prefix list as the destination (project 9)

📘 [Series overview](README.md#how-it-fits-together)
</details>

## Security layers

<details><summary>9. Give three differences between security groups and network ACLs.</summary>

Any three of these:
- Security groups protect an **instance**. NACLs protect a **subnet**.
- Security groups are **stateful**. NACLs are **stateless**.
- Security groups **only allow**. NACLs can **allow and deny**.
- NACL rules are checked **in number order**.

📘 [COMPARISONS.md](COMPARISONS.md#1-security-groups-vs-network-acls)
</details>

<details><summary>10. What does a brand-new custom NACL allow? What about the VPC's default NACL?</summary>

A custom NACL **denies everything** until you add rules. The default NACL **allows everything**.

📘 [VPC Traffic Flow and Security](https://github.com/jasahorro/VPC-Traffic-Flow-and-Security/blob/main/CHEATSHEET.md)
</details>

<details><summary>11. Why does a NACL need an outbound rule for replies when a security group doesn't?</summary>

NACLs are **stateless**: they don't remember that a request was allowed in, so the reply has to be allowed out separately. Security groups are **stateful** and let replies through automatically.

📘 [VPC Traffic Flow and Security](https://github.com/jasahorro/VPC-Traffic-Flow-and-Security/blob/main/CHEATSHEET.md)
</details>

<details><summary>12. How did the private server's security group allow SSH only from the public server?</summary>

Its inbound SSH rule used the **public server's security group** as the source, instead of `0.0.0.0/0`.

📘 [Launching VPC Resources](https://github.com/jasahorro/Launching-VPC-Resources/blob/main/CHEATSHEET.md)
</details>

<details><summary>13. Which ports or protocols do SSH, HTTP, HTTPS and ping use?</summary>

SSH is **TCP 22**, HTTP is **TCP 80**, HTTPS is **TCP 443**, and ping uses **ICMP**, which has no port.

📘 [VPC Traffic Flow and Security](https://github.com/jasahorro/VPC-Traffic-Flow-and-Security/blob/main/CHEATSHEET.md)
</details>

## Connectivity and troubleshooting

<details><summary>14. What two things does EC2 Instance Connect need from an instance (besides an SSH rule)?</summary>

The instance has to be in a **public subnet** and have a **public IPv4 address**.

📘 [Testing VPC Connectivity](https://github.com/jasahorro/Testing-VPC-Connectivity/blob/main/CHEATSHEET.md) · [VPC Peering](https://github.com/jasahorro/VPC-Peering/blob/main/CHEATSHEET.md)
</details>

<details><summary>15. What's the difference between what <code>ping</code> and <code>curl</code> test?</summary>

`ping` (ICMP) tests whether you can **reach an IP at all**. `curl` (HTTP/HTTPS) tests whether you can **fetch real content** from a web server.

📘 [Testing VPC Connectivity](https://github.com/jasahorro/Testing-VPC-Connectivity/blob/main/CHEATSHEET.md)
</details>

<details><summary>16. What does a NAT gateway do, and where does it go?</summary>

It gives **private** subnets **outbound-only** internet access, for things like software updates. It sits in a **public** subnet, and the private route table sends `0.0.0.0/0` to it. It costs money, so delete it after a project.

📘 [Launching VPC Resources](https://github.com/jasahorro/Launching-VPC-Resources/blob/main/CHEATSHEET.md)
</details>

<details><summary>17. What's an Elastic IP, and why is it better than an auto-assigned public IP?</summary>

A **static** public IPv4 address allocated to your account. An auto-assigned public IP changes when the instance stops and starts. An Elastic IP stays the same. It's charged hourly even when it isn't attached, so release it after use.

📘 [VPC Peering](https://github.com/jasahorro/VPC-Peering/blob/main/CHEATSHEET.md)
</details>

## Peering and monitoring

<details><summary>18. Why must peered VPCs have non-overlapping CIDR blocks?</summary>

If the ranges overlap, the route table can't tell whether an address is local or on the other side of the peering connection.

📘 [VPC Peering](https://github.com/jasahorro/VPC-Peering/blob/main/CHEATSHEET.md)
</details>

<details><summary>19. After accepting a peering connection, what else must you do?</summary>

Add a route in **both** VPCs' route tables: the other VPC's CIDR → `pcx-…`. You also need security group rules that allow the traffic.

📘 [VPC Peering](https://github.com/jasahorro/VPC-Peering/blob/main/CHEATSHEET.md)
</details>

<details><summary>20. What do VPC Flow Logs record, and what do they not record?</summary>

They record **metadata** about each connection: source and destination IPs and ports, protocol, packets, bytes, start and end times, and **ACCEPT or REJECT**. They **don't** record the contents of the packets.

📘 [VPC Monitoring with Flow Logs](https://github.com/jasahorro/VPC-Monitoring-with-Flow-Logs/blob/main/CHEATSHEET.md)
</details>

<details><summary>21. Why did VPC Flow Logs need an IAM role, and what is a trust policy?</summary>

Flow Logs needs permission to write to CloudWatch Logs. The **role** holds those permissions. The **trust policy** says who is allowed to use the role, here `vpc-flow-logs.amazonaws.com`.

📘 [VPC Monitoring with Flow Logs](https://github.com/jasahorro/VPC-Monitoring-with-Flow-Logs/blob/main/CHEATSHEET.md)
</details>

## Credentials and S3

<details><summary>22. What's the difference between a key pair and an access key?</summary>

A key pair logs you in **to an EC2 instance** over SSH. An access key (ID + secret) lets apps and servers **call AWS services** through the CLI, SDKs or APIs.

📘 [Access S3 from a VPC](https://github.com/jasahorro/Access-S3-from-a-VPC/blob/main/CHEATSHEET.md)
</details>

<details><summary>23. What's the best-practice alternative to putting access keys on an EC2 instance?</summary>

Attach an **IAM role**. The instance gets temporary credentials that rotate automatically, with no long-term secret stored on it.

📘 [Access S3 from a VPC](https://github.com/jasahorro/Access-S3-from-a-VPC/blob/main/CHEATSHEET.md)
</details>

<details><summary>24. Which two services can use a gateway endpoint, and how does it send traffic there?</summary>

**S3 and DynamoDB**. It adds a route to your route table: the service's **prefix list** (`pl-…`) → the endpoint (`vpce-…`).

📘 [VPC Endpoints](https://github.com/jasahorro/VPC-Endpoints/blob/main/CHEATSHEET.md)
</details>

<details><summary>25. What's the difference between a bucket policy and an endpoint policy?</summary>

A **bucket policy** sits on the bucket and controls who can reach it from anywhere. An **endpoint policy** sits on the endpoint and controls what can travel through it. A request must pass both.

📘 [VPC Endpoints](https://github.com/jasahorro/VPC-Endpoints/blob/main/CHEATSHEET.md)
</details>

## Scenario questions

<details><summary>26. Instance Connect to your public server fails. The subnet has an internet gateway route. What do you check next?</summary>

1. Does the **security group** allow **SSH on port 22**?
2. Does the instance have a **public IPv4 address**? If not, associate an Elastic IP.
3. Does the subnet's **NACL** allow port 22 in **and** the reply ports out?

📘 [Testing VPC Connectivity](https://github.com/jasahorro/Testing-VPC-Connectivity/blob/main/CHEATSHEET.md) · [VPC Peering](https://github.com/jasahorro/VPC-Peering/blob/main/CHEATSHEET.md)
</details>

<details><summary>27. Ping from a public server to a private server gets 100% packet loss. Name the two layers to check.</summary>

The **private subnet's NACL** (allow ICMP inbound **and** outbound) and the **private server's security group** (allow ICMP from the public server).

📘 [Testing VPC Connectivity](https://github.com/jasahorro/Testing-VPC-Connectivity/blob/main/CHEATSHEET.md)
</details>

<details><summary>28. Pinging another VPC's instance works by its public IP but not its private IP. Why?</summary>

The public ping goes over the internet. The private path needs a **peering connection plus routes** in both VPCs, and they're missing.

📘 [VPC Monitoring with Flow Logs](https://github.com/jasahorro/VPC-Monitoring-with-Flow-Logs/blob/main/CHEATSHEET.md)
</details>

<details><summary>29. You locked an S3 bucket to your VPC endpoint, and EC2 still gets Access Denied. What's the most likely cause?</summary>

The subnet's **route table isn't associated with the endpoint**, so traffic still goes over the internet, without the endpoint ID the bucket policy requires.

📘 [VPC Endpoints](https://github.com/jasahorro/VPC-Endpoints/blob/main/CHEATSHEET.md)
</details>

<details><summary>30. You want to know whether traffic is even reaching your instance and being rejected. What tool do you use?</summary>

**VPC Flow Logs**. A **REJECT** record shows that the traffic arrived but a security group or NACL blocked it. No record at all points to a routing problem further upstream.

📘 [VPC Monitoring with Flow Logs](https://github.com/jasahorro/VPC-Monitoring-with-Flow-Logs/blob/main/CHEATSHEET.md)
</details>
