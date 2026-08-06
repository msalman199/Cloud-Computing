<div align="center">

# ☁️ Install AWS CLI and LocalStack on Linux

### Building a Free, Local AWS Emulation Environment with Docker

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![AWS CLI](https://img.shields.io/badge/AWS_CLI-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![LocalStack](https://img.shields.io/badge/LocalStack-6C2EB5?style=for-the-badge&logo=localstack&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)

**Domain:** Domain 3: Cloud Computing | **Complexity:** 🟢 Basic | **Duration:** ⏱️ 30-60 minutes

*Source Course: Introduction to Cloud Computing on AWS for Beginners [2026]*

</div>

---

## 📑 Table of Contents

- [🎯 Learning Objectives](#-learning-objectives)
- [📋 Prerequisites](#-prerequisites)
- [🖥️ Lab Environment](#️-lab-environment)
- [🐳 Task 1: Install Docker and Docker Compose](#-task-1-install-docker-and-docker-compose)
- [☁️ Task 2: Install AWS CLI v2](#️-task-2-install-aws-cli-v2)
- [⚙️ Task 3: Configure AWS CLI with a LocalStack Profile](#️-task-3-configure-aws-cli-with-a-localstack-profile)
- [🚀 Task 4: Pull and Run LocalStack](#-task-4-pull-and-run-localstack)
- [🩺 Task 5: Verify LocalStack Health](#-task-5-verify-localstack-health)
- [🔗 Task 6: Create the `awslocal` Alias](#-task-6-create-the-awslocal-alias)
- [✅ Verification Checklist](#-verification-checklist)
- [🛠️ Troubleshooting Tips](#️-troubleshooting-tips)
- [🔑 Key Concepts](#-key-concepts)
- [🏁 Conclusion](#-conclusion)

---

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

| # | Objective |
|---|-----------|
| 1 | 🐳 Install Docker and Docker Compose on a Linux machine |
| 2 | ☁️ Install and verify AWS CLI v2 |
| 3 | ⚙️ Configure a local AWS profile for use with LocalStack |
| 4 | 🚀 Run LocalStack in a Docker container to emulate AWS services locally |
| 5 | 🩺 Verify LocalStack is running correctly |
| 6 | 🔗 Create a shortcut command (`awslocal`) for easier local testing |

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| 💻 Terminal familiarity | Basic comfort running commands and using `sudo` |
| ☁️ AWS/Docker experience | Not required — this lab starts from zero |
| 🖥️ Machine | A single Ubuntu-based Linux machine (provided via Start Lab) |

## 🖥️ Lab Environment

> **📍 Environment Setup**
> Al Nafi provides a single Linux virtual machine accessible via **Start Lab**. Ensure you have terminal access and an active internet connection — all commands in this lab are run directly on this machine.

---

## 🐳 Task 1: Install Docker and Docker Compose

![Docker](https://img.shields.io/badge/Tool-Docker_Engine-2496ED?style=flat-square&logo=docker&logoColor=white) ![APT](https://img.shields.io/badge/Package_Manager-APT-E95420?style=flat-square&logo=ubuntu&logoColor=white)

Docker lets us run LocalStack in an isolated container without installing extra software directly on our machine.

### 1️⃣ Update package lists and install dependencies

```bash
# 📦 Update package lists
sudo apt-get update

# 🔧 Install required dependencies
sudo apt-get install -y ca-certificates curl gnupg
```

### 2️⃣ Add Docker's official GPG key and repository

```bash
# 🔑 Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 📝 Add Docker repository to sources list
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 🔄 Update package lists again to include Docker repo
sudo apt-get update
```

### 3️⃣ Install Docker Engine, CLI, and Compose plugin

```bash
# 🐳 Install Docker Engine, CLI, and Compose plugin
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 👤 Allow current user to run docker without sudo (log out/in may be required)
sudo usermod -aG docker $USER

# TODO: If this is a shared lab machine, confirm with your instructor before
# adding your user to the docker group — it grants root-equivalent access.
```

### ✅ Verify installation

```bash
docker --version
docker compose version
```

**Expected output:** version numbers for both Docker and Docker Compose (e.g., `Docker version 25.x.x`).

> ℹ️ **Note:** If `docker` commands still require `sudo`, either continue using `sudo docker ...` or log out and log back in.

---

## ☁️ Task 2: Install AWS CLI v2

![AWS CLI](https://img.shields.io/badge/Tool-AWS_CLI_v2-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

### 1️⃣ Download and unzip the installer

```bash
# ⬇️ Download the AWS CLI v2 installer
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# 🔧 Install unzip if not already available
sudo apt-get install -y unzip

# 📂 Unzip the installer package
unzip awscliv2.zip
```

### 2️⃣ Run the install script

```bash
# 🚀 Run the install script
sudo ./aws/install
```

### ✅ Verify installation

```bash
aws --version
```

**Expected output similar to:** `aws-cli/2.x.x Python/3.x.x Linux/x.x.x`

---

## ⚙️ Task 3: Configure AWS CLI with a LocalStack Profile

![AWS CLI](https://img.shields.io/badge/Config-Named_Profile-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

LocalStack does not check real AWS credentials, but the AWS CLI still requires values to be set — we use dummy values.

### 1️⃣ Configure a named profile

```bash
# 🧾 Configure a named profile called "localstack"
aws configure --profile localstack
```

When prompted, enter:

| Prompt | Value |
|---|---|
| AWS Access Key ID | `test` |
| AWS Secret Access Key | `test` |
| Default region name | `us-east-1` |
| Default output format | `json` |

### ✅ Verify the profile was created

```bash
cat ~/.aws/credentials
cat ~/.aws/config
```

**Expected result:** you should see a `[localstack]` section with the values you entered.

---

## 🚀 Task 4: Pull and Run LocalStack

![LocalStack](https://img.shields.io/badge/Service-LocalStack-6C2EB5?style=flat-square&logo=localstack&logoColor=white) ![Docker](https://img.shields.io/badge/Runtime-Docker_Container-2496ED?style=flat-square&logo=docker&logoColor=white)

### 1️⃣ Pull the LocalStack image

```bash
# 📥 Pull the official LocalStack image from Docker Hub
docker pull localstack/localstack
```

### 2️⃣ Run the LocalStack container

```bash
# ▶️ Run LocalStack, mapping the main port 4566 to your machine
docker run -d \
  --name localstack \
  -p 4566:4566 \
  localstack/localstack
```

**Command breakdown:**

| Flag | Purpose |
|---|---|
| `-d` | Runs the container in the background (detached mode) |
| `--name localstack` | Gives the container a friendly name |
| `-p 4566:4566` | Maps container port `4566` to host port `4566` (LocalStack's main gateway port) |

### ✅ Verify the container is running

```bash
docker ps
```

**Expected result:** a row with `localstack/localstack` and status `Up`.

---

## 🩺 Task 5: Verify LocalStack Health

![Health Check](https://img.shields.io/badge/Check-Health_Endpoint-brightgreen?style=flat-square&logo=curl&logoColor=white)

### 1️⃣ Install curl (if needed) and query the health endpoint

```bash
# 🔧 Install curl if not already present
sudo apt-get install -y curl

# 🩺 Query the LocalStack health endpoint
curl http://localhost:4566/_localstack/health
```

**Expected output (formatted example):**

```json
{
  "services": {
    "s3": "available",
    "dynamodb": "available",
    "lambda": "available"
  },
  "edition": "community"
}
```

✅ If you see JSON output listing services, LocalStack is running correctly.

---

## 🔗 Task 6: Create the `awslocal` Alias

![Bash](https://img.shields.io/badge/Shortcut-Shell_Alias-4EAA25?style=flat-square&logo=gnubash&logoColor=white)

This alias saves time by automatically pointing the AWS CLI to LocalStack's endpoint.

### 1️⃣ Add the alias to your shell configuration

```bash
# ➕ Add the alias to your shell configuration file
echo 'alias awslocal="aws --endpoint-url=http://localhost:4566 --profile localstack"' >> ~/.bashrc

# 🔄 Reload the shell configuration
source ~/.bashrc

# TODO: Add any additional awslocal-style shortcuts here as you progress
# through future labs (e.g. an alias scoped to a different local endpoint).
```

### 2️⃣ Test the alias

```bash
# 📋 List S3 buckets (should return an empty list, no errors)
awslocal s3 ls
```

**Expected output:** empty result (no buckets yet) with no connection errors. This confirms `awslocal` is correctly routing requests to LocalStack.

---

## ✅ Verification Checklist

Run through these checks to confirm the lab was completed successfully:

| Check | Command | Expected Result |
|---|---|---|
| 🐳 Docker installed | `docker --version` | Version number displayed |
| 🧩 Docker Compose installed | `docker compose version` | Version number displayed |
| ☁️ AWS CLI installed | `aws --version` | `aws-cli/2.x.x` displayed |
| ⚙️ Profile configured | `cat ~/.aws/config` | `[profile localstack]` section present |
| 🚀 LocalStack running | `docker ps` | Container `localstack` status `Up` |
| 🩺 Health check passes | `curl http://localhost:4566/_localstack/health` | JSON with `"services"` key |
| 🔗 Alias works | `awslocal s3 ls` | Runs without connection error |

---

## 🛠️ Troubleshooting Tips

<details>
<summary><strong>🔓 "Permission denied" on docker commands</strong></summary>

Run `newgrp docker` or log out and back in after adding your user to the docker group.

</details>

<details>
<summary><strong>🔌 Port 4566 already in use</strong></summary>

Stop the conflicting process or remove old containers with `docker rm -f localstack`.

</details>

<details>
<summary><strong>❓ aws: command not found</strong></summary>

Ensure `/usr/local/bin` is in your `PATH`, or re-run `sudo ./aws/install`.

</details>

<details>
<summary><strong>🔗 curl: (7) Failed to connect</strong></summary>

Wait 10-15 seconds after starting LocalStack; it needs time to initialize.

</details>

<details>
<summary><strong>🚫 Alias not recognized</strong></summary>

Make sure you ran `source ~/.bashrc`, or open a new terminal session.

</details>

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| 🐳 **Docker & Docker Compose** | Container runtime and orchestration tooling that lets LocalStack run in an isolated environment without polluting the host machine |
| ☁️ **AWS CLI v2** | Command-line interface for interacting with AWS services — also used to talk to LocalStack once pointed at a local endpoint |
| 🗂️ **Named Profiles** | AWS CLI configuration profiles (`--profile localstack`) that isolate dummy local credentials from real AWS credentials |
| 🚀 **LocalStack** | A local AWS cloud emulator that spins up services like S3, DynamoDB, and Lambda inside a single container — no real AWS account or cost required |
| 🩺 **Health Endpoint** | `/_localstack/health` reports which emulated AWS services are currently available |
| 🔗 **`awslocal` Alias** | A shell shortcut that pre-fills `--endpoint-url` and `--profile`, so every command automatically targets LocalStack instead of real AWS |

---

## 🏁 Conclusion

In this lab, you successfully set up a local cloud development environment on a single Linux machine.

### 🏆 Key Accomplishments

- 🐳 Installed Docker and Docker Compose to run containerized applications
- ☁️ Installed and verified AWS CLI v2
- ⚙️ Created a `localstack` profile with dummy credentials for local testing
- 🚀 Pulled and launched the LocalStack Docker container, exposing port 4566
- 🩺 Verified LocalStack's health using `curl`
- 🔗 Configured an `awslocal` alias to simplify future AWS CLI commands against LocalStack

### 🌍 Real-World Applications

You now have a fully functional, cost-free local AWS emulation environment. This setup forms the foundation for future labs where you will create and interact with services like S3, DynamoDB, and Lambda entirely on your local machine, without incurring any real AWS costs.

---

<div align="center">

![Al Nafi](https://img.shields.io/badge/Al%20Nafi-Cybersecurity%20Training-1a1a2e?style=for-the-badge)

</div>
