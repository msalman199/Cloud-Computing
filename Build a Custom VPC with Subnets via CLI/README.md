<div align="center">

# 🌐 Build a Custom VPC with Subnets via CLI

### Provisioning Network Topology on LocalStack with AWS CLI

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![AWS VPC](https://img.shields.io/badge/AWS_VPC-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![LocalStack](https://img.shields.io/badge/LocalStack-6C2EB5?style=for-the-badge&logo=localstack&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)

</div>

---

## 📑 Table of Contents

- [🎯 Learning Objectives](#-learning-objectives)
- [📋 Prerequisites](#-prerequisites)
- [🖥️ Environment Setup](#️-environment-setup)
- [🏗️ Task 1: Create the VPC](#️-task-1-create-the-vpc)
- [🧩 Task 2: Create Public and Private Subnets](#-task-2-create-public-and-private-subnets)
- [🚪 Task 3: Create and Attach an Internet Gateway](#-task-3-create-and-attach-an-internet-gateway)
- [🗺️ Task 4: Create a Route Table with a Default Route](#️-task-4-create-a-route-table-with-a-default-route)
- [🔗 Task 5: Associate Route Table with the Public Subnet](#-task-5-associate-route-table-with-the-public-subnet)
- [✅ Verification](#-verification)
- [🛠️ Troubleshooting](#️-troubleshooting)
- [🔑 Key Concepts](#-key-concepts)
- [🏁 Conclusion](#-conclusion)

---

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

| # | Objective |
|---|-----------|
| 1 | 🏗️ Deploy a VPC with custom CIDR blocks using AWS CLI on LocalStack |
| 2 | 🧩 Configure public and private subnets within a VPC |
| 3 | 🚪 Attach an internet gateway and configure route tables for internet access |
| 4 | ✅ Verify network topology using AWS CLI describe commands |

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| 🌐 Networking concepts | Basic understanding of CIDR notation and IP addressing |
| 🧩 AWS VPC familiarity | Comfort with VPC components (subnets, route tables, gateways) |
| 💻 Terminal familiarity | Comfort using a Linux terminal |
| ☁️ AWS CLI knowledge | Basic AWS CLI command structure knowledge |

## 🖥️ Environment Setup

> **📍 Setup Overview**
> Al Nafi provides a single Linux machine via Start Lab. Use it for all steps below.

![Docker](https://img.shields.io/badge/Tool-Docker-2496ED?style=flat-square&logo=docker&logoColor=white)

### Step 1: Install Docker and LocalStack

```bash
sudo apt update && sudo apt install -y docker.io python3-pip
sudo systemctl start docker
pip3 install localstack awscli-local
```

![LocalStack](https://img.shields.io/badge/Service-LocalStack-6C2EB5?style=flat-square&logo=localstack&logoColor=white)

### Step 2: Start LocalStack with VPC/EC2 services

```bash
# 🧩 EC2 service covers VPC, subnets, route tables, and gateways
localstack start -d
```

Wait ~10 seconds, then verify it's running:

```bash
localstack status services
```

Confirm `ec2` shows status `running`.

![AWS CLI](https://img.shields.io/badge/Config-Dummy_Credentials-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

### Step 3: Configure AWS CLI for LocalStack

```bash
aws configure set aws_access_key_id test
aws configure set aws_secret_access_key test
aws configure set region us-east-1
```

> ℹ️ Use `awslocal` (wraps `aws --endpoint-url=http://localhost:4566`) for all commands below, or manually append `--endpoint-url=http://localhost:4566` to each AWS CLI call.

---

## 🏗️ Task 1: Create the VPC

![VPC](https://img.shields.io/badge/VPC-Create-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

Create a VPC with CIDR block `10.0.0.0/16`.

```bash
awslocal ec2 create-vpc --cidr-block 10.0.0.0/16
```

> **📝 TODO:** Capture the `VpcId` from the output JSON and export it as an environment variable:

```bash
export VPC_ID=<TODO: paste VpcId here>
```

Tag the VPC for clarity:

```bash
awslocal ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=CustomLabVPC
```

---

## 🧩 Task 2: Create Public and Private Subnets

![Subnets](https://img.shields.io/badge/VPC-Subnets-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

Create two subnets inside the VPC:

```bash
# 🌍 Public subnet
awslocal ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24

# 🔒 Private subnet
awslocal ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24
```

> **📝 TODO:** Capture each `SubnetId` and export them:

```bash
export PUBLIC_SUBNET_ID=<TODO>
export PRIVATE_SUBNET_ID=<TODO>
```

Tag both subnets appropriately (`Name=PublicSubnet` / `Name=PrivateSubnet`) using `create-tags` as in Task 1.

---

## 🚪 Task 3: Create and Attach an Internet Gateway

![Internet Gateway](https://img.shields.io/badge/VPC-Internet_Gateway-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

```bash
awslocal ec2 create-internet-gateway
```

> **📝 TODO:** Capture `InternetGatewayId`:

```bash
export IGW_ID=<TODO>
```

Attach the gateway to your VPC:

```bash
awslocal ec2 attach-internet-gateway \
  --vpc-id $VPC_ID \
  --internet-gateway-id $IGW_ID
```

---

## 🗺️ Task 4: Create a Route Table with a Default Route

![Route Table](https://img.shields.io/badge/VPC-Route_Table-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

Create a new route table in the VPC:

```bash
awslocal ec2 create-route-table --vpc-id $VPC_ID
```

> **📝 TODO:** Capture `RouteTableId`:

```bash
export RT_ID=<TODO>
```

Add a default route (`0.0.0.0/0`) pointing to the internet gateway:

```bash
awslocal ec2 create-route \
  --route-table-id $RT_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID
```

> 💡 **Troubleshooting tip:** If `create-route` fails with an invalid gateway error, confirm `$IGW_ID` is exported correctly and that the gateway attachment in Task 3 succeeded (check with `describe-internet-gateways`).

---

## 🔗 Task 5: Associate Route Table with the Public Subnet

![Route Table](https://img.shields.io/badge/VPC-Route_Association-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

```bash
awslocal ec2 associate-route-table \
  --route-table-id $RT_ID \
  --subnet-id $PUBLIC_SUBNET_ID
```

> ℹ️ **Note:** The private subnet remains associated with the VPC's implicit main route table (no internet route), simulating a private network.

---

## ✅ Verification

Run the following commands to confirm your topology:

```bash
# 🏗️ Verify VPC
awslocal ec2 describe-vpcs --vpc-ids $VPC_ID

# 🧩 Verify subnets
awslocal ec2 describe-subnets --filters Name=vpc-id,Values=$VPC_ID

# 🚪 Verify internet gateway attachment
awslocal ec2 describe-internet-gateways --internet-gateway-ids $IGW_ID

# 🗺️ Verify route table and associations
awslocal ec2 describe-route-tables --route-table-ids $RT_ID
```

**Expected outcomes:**

- 🏗️ VPC shows `CidrBlock: 10.0.0.0/16` and state `available`
- 🧩 Two subnets listed with CIDRs `10.0.1.0/24` and `10.0.2.0/24`
- 🚪 Internet gateway `Attachments` shows your `VPC_ID` with state `available`
- 🗺️ Route table shows a route to `0.0.0.0/0` via `$IGW_ID` and an association with `$PUBLIC_SUBNET_ID`

---

## 🛠️ Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| `VpcID` malformed error | Environment variable not exported correctly | Re-run `export VPC_ID=...` with correct ID |
| Subnet CIDR overlaps error | CIDR outside VPC range or duplicate | Confirm subnet CIDR falls within `10.0.0.0/16` |
| Route creation fails | IGW not attached | Re-check Task 3 attach step |
| `awslocal: command not found` | pip install path not in `PATH` | Use `python3 -m pip install --user awscli-local` and restart shell |

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| 🚀 **LocalStack** | A local AWS cloud emulator — running the EC2 service here emulates VPC, subnets, route tables, and gateways with no real AWS account required |
| 🏗️ **VPC (Virtual Private Cloud)** | An isolated network defined by a CIDR block (`10.0.0.0/16`) that all subnets and resources live inside |
| 🧩 **Public vs. Private Subnet** | A subnet becomes "public" only when its route table sends `0.0.0.0/0` traffic to an internet gateway — the private subnet stays on the VPC's default (internet-less) main route table |
| 🚪 **Internet Gateway (IGW)** | A VPC component that must be created and explicitly attached before any subnet can reach the internet |
| 🗺️ **Route Table** | A set of rules (routes) that determine where subnet traffic is sent; associating a route table with a subnet is what actually applies those rules |
| 🌐 **`awslocal`** | A wrapper command that pre-fills `--endpoint-url=http://localhost:4566`, routing AWS CLI calls to LocalStack instead of real AWS |

---

## 🏁 Conclusion

In this lab, you built a custom VPC network topology entirely through the AWS CLI on LocalStack.

### 🏆 Key Accomplishments

- 🏗️ Created a VPC with a `/16` CIDR block
- 🧩 Provisioned public and private subnets
- 🚪 Attached an internet gateway
- 🗺️ Configured a route table to enable internet access for the public subnet only
- ✅ Verified the complete topology using `describe-*` commands

### 🌍 Real-World Applications

These skills — CIDR planning, gateway attachment, and route table configuration — are foundational for the **AWS Certified Solutions Architect - Associate** exam and directly applicable to **Cloud Network Engineer** roles managing production VPC architectures.

---

<div align="center">

![Al Nafi](https://img.shields.io/badge/Al%20Nafi-Cybersecurity%20Training-1a1a2e?style=for-the-badge)

</div>
