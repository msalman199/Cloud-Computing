<div align="center">

# 🔐 Manage IAM Users and Policies Locally

### Practicing AWS Identity and Access Management with LocalStack

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![AWS IAM](https://img.shields.io/badge/AWS_IAM-232F3E?style=for-the-badge&logo=amazoniam&logoColor=white)
![LocalStack](https://img.shields.io/badge/LocalStack-6C2EB5?style=for-the-badge&logo=localstack&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)

</div>

---

## 📑 Table of Contents

- [🎯 Objectives](#-objectives)
- [📋 Prerequisites](#-prerequisites)
- [🖥️ Environment Setup](#️-environment-setup)
- [🚀 Task 1: Start LocalStack via Docker Compose](#-task-1-start-localstack-via-docker-compose)
- [⚙️ Task 2: Configure AWS CLI for LocalStack](#️-task-2-configure-aws-cli-for-localstack)
- [👤 Task 3: Create the cloud-admin IAM User](#-task-3-create-the-cloud-admin-iam-user)
- [👥 Task 4: Create the developers Group with an Inline S3 Read-Only Policy](#-task-4-create-the-developers-group-with-an-inline-s3-read-only-policy)
- [➕ Task 5: Add cloud-admin to the developers Group](#-task-5-add-cloud-admin-to-the-developers-group)
- [🔑 Task 6: Create app-user with Programmatic Access Keys](#-task-6-create-app-user-with-programmatic-access-keys)
- [📋 Task 7: List All Users and Groups](#-task-7-list-all-users-and-groups)
- [🗑️ Task 8: Delete app-user and Confirm Removal](#️-task-8-delete-app-user-and-confirm-removal)
- [✅ Verification](#-verification)
- [🛠️ Troubleshooting Tips](#️-troubleshooting-tips)
- [🔑 Key Concepts](#-key-concepts-1)
- [🏁 Conclusion](#-conclusion)

---

## 🎯 Objectives

By the end of this lab, you will be able to:

| # | Objective |
|---|-----------|
| 1 | 🚀 Start LocalStack, an open-source AWS emulator, using Docker Compose |
| 2 | 👤 Create IAM users and groups using the AWS CLI against a local endpoint |
| 3 | 📄 Attach an inline policy to control S3 access permissions |
| 4 | 👥 Add users to groups and verify group membership |
| 5 | 📋 List and delete IAM resources |
| 6 | 🧩 Understand the basic building blocks of AWS IAM (Users, Groups, Policies) |

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| 💻 Terminal familiarity | Basic comfort navigating folders and running commands |
| ☁️ AWS/IAM experience | Not required — this lab starts from zero |
| 🖥️ Machine | Al Nafi Linux machine (via Start Lab) with internet access |
| 🧠 Conceptual grounding | Basic understanding that IAM = Identity and Access Management (controls who can do what in AWS) |

## 🖥️ Environment Setup

> **📍 Setup Overview**
> You will use a single Linux machine provided by Al Nafi. All tools will be installed locally — no real AWS account or billing is needed, since LocalStack emulates AWS services on your machine.

![Docker](https://img.shields.io/badge/Tool-Docker-2496ED?style=flat-square&logo=docker&logoColor=white)

### Step 1: Install Docker and Docker Compose

```bash
# 📦 Update package lists
sudo apt update

# 🐳 Install Docker
sudo apt install -y docker.io docker-compose

# ▶️ Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# ✅ Verify installation
docker --version
docker-compose --version
```

![AWS CLI](https://img.shields.io/badge/Tool-AWS_CLI-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

### Step 2: Install AWS CLI

```bash
# ⬇️ Download the AWS CLI installer
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# 🔧 Unzip the installer
sudo apt install -y unzip
unzip awscliv2.zip

# 🚀 Run the installer
sudo ./aws/install

# ✅ Verify installation
aws --version
```

---

## 🚀 Task 1: Start LocalStack via Docker Compose

![LocalStack](https://img.shields.io/badge/Service-LocalStack-6C2EB5?style=flat-square&logo=localstack&logoColor=white) ![Compose](https://img.shields.io/badge/Tool-Docker_Compose-2496ED?style=flat-square&logo=docker&logoColor=white)

Create a folder and a `docker-compose.yml` file to define LocalStack with the IAM service enabled.

### 1️⃣ Create the project folder and compose file

```bash
# 📁 Create a project folder
mkdir ~/iam-lab && cd ~/iam-lab

# 📝 Create the docker-compose.yml file
nano docker-compose.yml
```

Paste the following content into the file:

```yaml
version: "3.8"
services:
  localstack:
    image: localstack/localstack:latest
    ports:
      - "4566:4566"      # 🌐 Main LocalStack edge port
    environment:
      - SERVICES=iam,s3  # 🔐 Enable IAM and S3 services
      - DEBUG=1
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
```

### 2️⃣ Start the container

```bash
# ▶️ Launch LocalStack in the background
docker-compose up -d

# ✅ Check the container is running
docker ps
```

---

## ⚙️ Task 2: Configure AWS CLI for LocalStack

![AWS CLI](https://img.shields.io/badge/Config-Dummy_Credentials-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

LocalStack does not check real credentials, but the CLI still requires dummy values.

### 1️⃣ Configure fake credentials

```bash
# 🧾 Configure fake credentials (required by AWS CLI, not validated by LocalStack)
aws configure set aws_access_key_id test
aws configure set aws_secret_access_key test
aws configure set region us-east-1
```

### 2️⃣ Create a shell alias for LocalStack

```bash
# 🔗 Add alias to your current shell session
alias awslocal="aws --endpoint-url=http://localhost:4566"

# 🧪 Test the alias
awslocal iam list-users
```

**Expected output:** an empty `Users` list (no users created yet).

---

## 👤 Task 3: Create the cloud-admin IAM User

![IAM](https://img.shields.io/badge/IAM-Create_User-232F3E?style=flat-square&logo=amazoniam&logoColor=white)

```bash
# TODO: Create a user named cloud-admin
awslocal iam create-user --user-name cloud-admin
```

**Verify:**

```bash
awslocal iam get-user --user-name cloud-admin
```

---

## 👥 Task 4: Create the developers Group with an Inline S3 Read-Only Policy

![IAM](https://img.shields.io/badge/IAM-Groups_%26_Policies-232F3E?style=flat-square&logo=amazoniam&logoColor=white) ![S3](https://img.shields.io/badge/S3-Read_Only-569A31?style=flat-square&logo=amazons3&logoColor=white)

### 1️⃣ Create the group

```bash
# 👥 Step 1: Create the group
awslocal iam create-group --group-name developers
```

### 2️⃣ Create the S3 read-only policy document

```bash
# 📝 Step 2: Create a policy JSON file for S3 read-only access
nano s3-readonly-policy.json
```

Paste this policy document:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

### 3️⃣ Attach the inline policy to the group

```bash
# TODO: Attach the inline policy to the developers group
awslocal iam put-group-policy \
  --group-name developers \
  --policy-name S3ReadOnlyPolicy \
  --policy-document file://s3-readonly-policy.json
```

**Verify the policy was attached:**

```bash
awslocal iam get-group-policy \
  --group-name developers \
  --policy-name S3ReadOnlyPolicy
```

---

## ➕ Task 5: Add cloud-admin to the developers Group

![IAM](https://img.shields.io/badge/IAM-Group_Membership-232F3E?style=flat-square&logo=amazoniam&logoColor=white)

```bash
# TODO: Add the cloud-admin user to the developers group
awslocal iam add-user-to-group \
  --user-name cloud-admin \
  --group-name developers
```

**Verify membership:**

```bash
awslocal iam get-group --group-name developers
```

---

## 🔑 Task 6: Create app-user with Programmatic Access Keys

![IAM](https://img.shields.io/badge/IAM-Access_Keys-232F3E?style=flat-square&logo=amazoniam&logoColor=white)

```bash
# 👤 Step 1: Create the second user
awslocal iam create-user --user-name app-user

# 🔑 Step 2: Generate access keys for programmatic access
awslocal iam create-access-key --user-name app-user
```

> ℹ️ **Note:** Save the `AccessKeyId` and `SecretAccessKey` values printed in the output — in a real environment these would be used to configure a CLI profile or SDK.

---

## 📋 Task 7: List All Users and Groups

![IAM](https://img.shields.io/badge/IAM-Inventory-232F3E?style=flat-square&logo=amazoniam&logoColor=white)

```bash
# 📋 List all IAM users
awslocal iam list-users

# 📋 List all IAM groups
awslocal iam list-groups

# 👥 List members of the developers group
awslocal iam get-group --group-name developers
```

---

## 🗑️ Task 8: Delete app-user and Confirm Removal

![IAM](https://img.shields.io/badge/IAM-Cleanup-232F3E?style=flat-square&logo=amazoniam&logoColor=white)

IAM requires access keys to be deleted before the user itself.

### 1️⃣ List access keys for app-user

```bash
# 🔍 Step 1: List access keys for app-user to get the Access Key ID
awslocal iam list-access-keys --user-name app-user
```

### 2️⃣ Delete the access key

```bash
# 🗑️ Step 2: Delete the access key (replace <ACCESS_KEY_ID> with actual value)
awslocal iam delete-access-key \
  --user-name app-user \
  --access-key-id <ACCESS_KEY_ID>
```

### 3️⃣ Delete the user

```bash
# 🗑️ Step 3: Delete the user
awslocal iam delete-user --user-name app-user
```

---

## ✅ Verification

Run these commands to confirm your setup is correct:

```bash
# ✅ Should show only cloud-admin (app-user deleted)
awslocal iam list-users

# ✅ Should show developers group with cloud-admin as a member
awslocal iam get-group --group-name developers

# ✅ Should confirm the S3 read-only policy still exists
awslocal iam get-group-policy --group-name developers --policy-name S3ReadOnlyPolicy

# ❌ Should return an error confirming app-user no longer exists
awslocal iam get-user --user-name app-user
```

**Expected results:**

| Check | Expected Outcome |
|---|---|
| `list-users` | Only `cloud-admin` appears |
| `get-group` | `cloud-admin` listed as group member |
| `get-group-policy` | Policy document returned successfully |
| `get-user app-user` | Error: `"NoSuchEntity"` |

---

## 🛠️ Troubleshooting Tips

<details>
<summary><strong>🐳 Docker container not starting</strong></summary>

Run `docker-compose logs` to view error details.

</details>

<details>
<summary><strong>🔌 "Connection refused" errors</strong></summary>

Wait 10-15 seconds after `docker-compose up -d` for LocalStack to fully initialize.

</details>

<details>
<summary><strong>🚫 awslocal alias not working</strong></summary>

Aliases only persist in the current terminal session; re-run the alias command if you open a new terminal, or add it to `~/.bashrc`.

</details>

<details>
<summary><strong>🔓 Permission denied on Docker commands</strong></summary>

Run `sudo usermod -aG docker $USER` then log out and back in.

</details>

<details>
<summary><strong>📄 Policy JSON errors</strong></summary>

Validate your JSON syntax using `cat s3-readonly-policy.json | python3 -m json.tool`.

</details>

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| 🚀 **LocalStack** | A local AWS cloud emulator — running the IAM and S3 services here means no real AWS account or billing is required |
| 👤 **IAM User** | An identity representing a person or application that can be granted permissions (e.g. `cloud-admin`, `app-user`) |
| 👥 **IAM Group** | A named collection of users that share the same attached permissions (e.g. `developers`) |
| 📄 **Inline Policy** | A JSON permissions document attached directly to a single group (or user/role), scoping exactly what actions are allowed |
| 🔑 **Access Keys** | Programmatic credentials (`AccessKeyId` + `SecretAccessKey`) used by applications and SDKs instead of interactive login |
| 🔗 **`awslocal`-style Alias** | A shell shortcut (`--endpoint-url=http://localhost:4566`) that routes AWS CLI commands to LocalStack instead of real AWS |

---

## 🏁 Conclusion

In this lab, you set up LocalStack to emulate AWS IAM on a single Linux machine without needing a real AWS account.

### 🏆 Key Accomplishments

- 🚀 Started LocalStack via Docker Compose with the IAM and S3 services enabled
- 👤 Created IAM users (`cloud-admin` and `app-user`)
- 👥 Built a `developers` group and attached an inline policy granting S3 read-only access
- ➕ Added a user to a group and verified group membership
- 🔑 Generated programmatic access keys for an application user
- 🗑️ Safely deleted a user along with its access keys, and confirmed removal

### 🌍 Real-World Applications

These skills directly translate to real-world AWS environments and form a foundational competency for **Cloud Security Engineer** and **IAM Administrator** roles, as well as the **AWS Certified Cloud Practitioner** certification.

---

<div align="center">

![Al Nafi](https://img.shields.io/badge/Al%20Nafi-Cybersecurity%20Training-1a1a2e?style=for-the-badge)

</div>
