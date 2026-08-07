<div align="center">

# 🏗️ Automate Infrastructure with CloudFormation

![CloudFormation](https://img.shields.io/badge/AWS_CloudFormation-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white)
![LocalStack](https://img.shields.io/badge/LocalStack-2E2ADB?style=for-the-badge&logo=localstack&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![AWS CLI](https://img.shields.io/badge/AWS_CLI-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![YAML](https://img.shields.io/badge/YAML-CB171E?style=for-the-badge&logo=yaml&logoColor=white)

*Author, deploy, and manage a full CloudFormation stack lifecycle entirely against a local AWS emulator*

</div>

---

## 📚 Table of Contents

- [🎯 Objectives](#-objectives)
- [📋 Prerequisites](#-prerequisites)
- [🖥️ Environment Setup](#️-environment-setup)
- [📝 Tasks](#-tasks)
  - [Task 1: Launch LocalStack with Required Services](#task-1-launch-localstack-with-required-services)
  - [Task 2: Author the Initial CloudFormation Template](#task-2-author-the-initial-cloudformation-template)
  - [Task 3: Deploy and Validate the Stack](#task-3-deploy-and-validate-the-stack)
  - [Task 4: Update the Stack — Add a Second Bucket](#task-4-update-the-stack--add-a-second-bucket)
  - [Task 5: Inspect Resources and Tear Down](#task-5-inspect-resources-and-tear-down)
- [✅ Verification](#-verification)
- [🔧 Troubleshooting Tips](#-troubleshooting-tips)
- [🔑 Key Concepts](#-key-concepts)
- [🏁 Conclusion](#-conclusion)

---

## 🎯 Objectives

By completing this lab, you will:

| # | Objective |
|---|-----------|
| 1 | 🐳 Deploy a local AWS emulation environment using LocalStack |
| 2 | 📄 Author CloudFormation YAML templates implementing IaC principles |
| 3 | 🔄 Manage full stack lifecycle: create, update, inspect, delete |
| 4 | 🔍 Validate resource provisioning and drift using AWS CLI against a local endpoint |

---

## 📋 Prerequisites

| Requirement | Details |
|-------------|---------|
| ☁️ AWS Core Services | Solid understanding of S3, IAM basics, and CloudFormation concepts (stacks, templates, logical IDs, intrinsic functions) |
| ⌨️ AWS CLI | Working knowledge of AWS CLI syntax and JSON/YAML structure |
| 🐳 Docker & Linux | Familiarity with Docker and Linux shell operations |
| 🛠️ Troubleshooting | Comfortable troubleshooting using CLI output and logs independently |

---

## 🖥️ Environment Setup

> 💻 **Single Linux machine (provided via Start Lab).** Install dependencies:

```bash
# Verify Docker is available; install if missing
docker --version || sudo apt-get update && sudo apt-get install -y docker.io   # 🐳 ensure Docker is present

# Install Python tooling for LocalStack and AWS CLI
sudo apt-get install -y python3-pip     # 🐍 pip for Python packages
pip3 install localstack awscli-local    # 🔌 LocalStack + awslocal wrapper

# Confirm installations
localstack --version   # ✅ verify LocalStack CLI
aws --version            # ✅ verify AWS CLI
```

**Configure a dummy AWS CLI profile** (LocalStack does not validate credentials):

```bash
aws configure set aws_access_key_id test      # 🔑 dummy key
aws configure set aws_secret_access_key test  # 🔑 dummy secret
aws configure set region us-east-1            # 🌎 default region
```

---

## 📝 Tasks

### Task 1: Launch LocalStack with Required Services

- 🚀 Start LocalStack in Docker, enabling only `cloudformation` and `s3` services (minimize resource footprint)
- 🩺 Confirm the container is healthy and the CloudFormation/S3 endpoints respond before proceeding

**Requirements:**
- ⚙️ Use `SERVICES` environment variable to restrict active services
- 🌐 Expose the edge port (default `4566`)
- ✅ Verify readiness via the LocalStack health check endpoint

```bash
# TODO: Run LocalStack container with correct SERVICES and port mapping
docker run -d --name localstack -p 4566:4566 -e SERVICES=cloudformation,s3 localstack/localstack

# TODO: Poll health endpoint until cloudformation/s3 status = "available"
curl -s localhost:4566/_localstack/health | jq
```

---

### Task 2: Author the Initial CloudFormation Template

- 📄 Design `infra.yaml` defining a single S3 bucket resource

**Requirements:**
- 🧾 Use `AWSTemplateFormatVersion` and `Description`
- 🪣 Logical ID must be descriptive (e.g., `PrimaryDataBucket`)
- 🔐 Add a `BucketName` using an intrinsic function to guarantee uniqueness (e.g., `!Sub` with `AWS::AccountId`/`AWS::Region`, or a stack-parameterized name)
- 📤 Include an `Outputs` section exporting the bucket name

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: <TODO: describe stack purpose>

Resources:
  # TODO: Define logical resource for S3 bucket
  # Type: AWS::S3::Bucket
  # Properties: BucketName using !Sub

Outputs:
  # TODO: Export bucket name for reference
```

---

### Task 3: Deploy and Validate the Stack

- 🚀 Deploy `infra.yaml` against LocalStack using `aws cloudformation create-stack` with `--endpoint-url` pointed at LocalStack
- ⏳ Poll stack status until `CREATE_COMPLETE`; investigate and resolve any `ROLLBACK` states independently using `describe-stack-events`

```bash
# TODO: Create stack (choose a descriptive stack name)
aws --endpoint-url=http://localhost:4566 cloudformation create-stack \
  --stack-name <TODO> --template-body file://infra.yaml

# TODO: Describe stack and confirm StackStatus
aws --endpoint-url=http://localhost:4566 cloudformation describe-stacks \
  --stack-name <TODO> --query "Stacks[0].StackStatus"

# TODO: Confirm bucket exists via S3 API
aws --endpoint-url=http://localhost:4566 s3 ls
```

---

### Task 4: Update the Stack — Add a Second Bucket

- ✏️ Modify `infra.yaml` to add a second, distinct `AWS::S3::Bucket` resource with its own logical ID and output
- 🔄 Apply changes using `update-stack`. Consider: should you use a **Change Set** instead for review before applying? Justify your choice in a short comment in your notes
- 🔍 Confirm both resources are tracked post-update

```bash
# TODO: Update the template file with second bucket resource

# TODO: Apply update
aws --endpoint-url=http://localhost:4566 cloudformation update-stack \
  --stack-name <TODO> --template-body file://infra.yaml

# TODO: Wait/poll for UPDATE_COMPLETE
```

---

### Task 5: Inspect Resources and Tear Down

- 📋 List all stack resources and confirm both buckets appear with `CREATE_COMPLETE`/`UPDATE_COMPLETE` status
- 🗑️ Delete the stack and verify complete removal of both S3 buckets

```bash
# TODO: List stack resources
aws --endpoint-url=http://localhost:4566 cloudformation list-stack-resources \
  --stack-name <TODO>

# TODO: Delete stack
aws --endpoint-url=http://localhost:4566 cloudformation delete-stack \
  --stack-name <TODO>

# TODO: Confirm stack no longer exists (describe-stacks should fail/return empty)
# TODO: Confirm buckets removed via s3 ls
```

---

## ✅ Verification

Confirm the following on your machine:

- `localstack status services` shows `cloudformation` and `s3` as running
- `describe-stacks` output showed `CREATE_COMPLETE` after Task 3
- `list-stack-resources` after Task 4 returns exactly two `AWS::S3::Bucket` entries, both in a completed state
- Post-deletion, `aws s3 ls` no longer lists either bucket, and `describe-stacks` returns a `ValidationError` or empty result for the deleted stack name

---

## 🔧 Troubleshooting Tips

<details>
<summary><b>🔁 ROLLBACK_COMPLETE</b></summary>

Inspect `describe-stack-events` for the first `CREATE_FAILED` event; a rolled-back stack must be deleted before redeploying with the same name.

</details>

<details>
<summary><b>🪣 Bucket name collisions</b></summary>

S3 bucket names are global in real AWS but LocalStack enforces uniqueness per-region state; use `!Sub` with a random or account/region-based suffix.

</details>

<details>
<summary><b>⏳ CLI calls hang</b></summary>

Verify `--endpoint-url` is set on every command — LocalStack does not intercept default AWS endpoints.

</details>

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| 📄 CloudFormation Template | A declarative YAML/JSON document describing the desired AWS resource state |
| 🆔 Logical ID | A template-local name identifying a resource, used by intrinsic functions like `!Ref` |
| 🔗 Intrinsic Functions | Built-ins (`!Sub`, `!Ref`, `!GetAtt`) that resolve values at deploy time, e.g. for unique naming |
| 🔄 Stack Lifecycle | Create → Update → Inspect → Delete — the full managed lifecycle of a CloudFormation stack |
| 📝 Change Sets | A preview of the changes an update would make, reviewable before being applied |
| 🖥️ LocalStack | Emulates AWS services (CloudFormation, S3) locally for cloud-cost-free IaC development |

---

## 🏁 Conclusion

### 🎉 Key Accomplishments

- ✅ Provisioned and managed AWS infrastructure entirely through code, using LocalStack as a zero-cost, local AWS emulator
- ✅ Authored a CloudFormation template from scratch and executed the full stack lifecycle (create, update, list, delete)
- ✅ Validated state transitions via the CLI rather than a console

### 🌍 Real-World Applications

These skills — declarative infrastructure design, stack lifecycle management, and CLI-driven verification — map directly to real-world DevOps workflows and the **AWS Certified DevOps Engineer – Professional** exam's IaC domain.

---

<div align="center">

### 🎓 Al Nafi Cybersecurity Training Labs

*Empowering the Next Generation of Cloud & Security Professionals*

</div>
