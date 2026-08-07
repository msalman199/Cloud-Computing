<div align="center">

# 🧩 Deploy a Full-Stack App with IaC on LocalStack

![CloudFormation](https://img.shields.io/badge/AWS_CloudFormation-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white)
![Lambda](https://img.shields.io/badge/AWS_Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![DynamoDB](https://img.shields.io/badge/DynamoDB-4053D6?style=for-the-badge&logo=amazondynamodb&logoColor=white)
![S3](https://img.shields.io/badge/Amazon_S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
![LocalStack](https://img.shields.io/badge/LocalStack-2E2ADB?style=for-the-badge&logo=localstack&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

*Independently architect, deploy, validate, and tear down a multi-service serverless stack — entirely as code*

</div>

---

## 📚 Table of Contents

- [🎯 Objectives](#-objectives)
- [📋 Prerequisites](#-prerequisites)
- [🖥️ Environment Setup](#️-environment-setup)
- [📝 Tasks](#-tasks)
  - [Phase 1: Design and Provision Infrastructure (IaC)](#phase-1-design-and-provision-infrastructure-iac)
  - [Phase 2: Application Logic and Data Flow Validation](#phase-2-application-logic-and-data-flow-validation)
  - [Phase 3: Teardown and Verification](#phase-3-teardown-and-verification)
- [🏆 Success Criteria](#-success-criteria)
- [✅ Verification](#-verification)
- [🔑 Key Concepts](#-key-concepts)
- [🏁 Conclusion](#-conclusion)

---

## 🎯 Objectives

By completing this challenge, you will:

| # | Objective |
|---|-----------|
| 1 | 🏗️ Architect and deploy a multi-service serverless stack (S3, DynamoDB, Lambda) using CloudFormation on LocalStack |
| 2 | 📄 Design an IaC template from scratch that satisfies functional and operational requirements |
| 3 | 🔄 Validate end-to-end data flow across storage, compute, and database layers |
| 4 | 🗑️ Implement clean teardown and verify complete resource removal |

---

## 📋 Prerequisites

| Requirement | Details |
|-------------|---------|
| ☁️ AWS Services | Expert-level familiarity with S3, DynamoDB, Lambda, IAM, CloudFormation |
| ⌨️ CLI & Shell | Strong command of AWS CLI, Docker, and Linux shell scripting |
| 🐍 Lambda Authoring | Working knowledge of Python or Node.js for Lambda function authoring |
| 📄 Template Anatomy | Understanding of CloudFormation template anatomy (Resources, Outputs, Parameters) |
| 🧭 LocalStack | Prior experience with LocalStack or willingness to consult its documentation independently |

---

## 🖥️ Environment Setup

> 💻 **Single Linux machine (provided via Start Lab) with Docker and Docker Compose installed.**
> ⚠️ Install required tooling yourself — **no pre-baked scripts are provided.**

**You are responsible for:**

- 🔌 `awslocal`, or configure the standard AWS CLI with a `localstack` profile pointing to `http://localhost:4566`
- 🐳 LocalStack Community Edition (Docker image `localstack/localstack:latest`)
- ⚙️ Starting LocalStack with the correct `SERVICES` environment variable (`s3`, `dynamodb`, `lambda`, `cloudformation`, `iam`, `sts`)
- 🩺 Confirming service health via LocalStack's health-check endpoint before proceeding
- 📂 No dataset or template files are provided — **you build everything from scratch**

---

## 📝 Tasks

### Phase 1: Design and Provision Infrastructure (IaC)

**Requirements:**
- 🎯 Launch LocalStack with only the services needed for this stack enabled — justify your service list
- 📄 Author a single CloudFormation YAML template that provisions:
  - 🪣 An S3 bucket configured for static website hosting (public read policy or website configuration block)
  - 🗄️ A DynamoDB table with a partition key of your choice (e.g., `id`), on-demand billing mode
  - ⚡ A Lambda function (runtime of your choice: Python 3.12 or Node.js 20.x) with an IAM execution role granting **least-privilege** access to only the DynamoDB table it needs
- 🤔 The Lambda function's code must be inlined or referenced from an S3 code package you upload yourself — decide which approach is more production-realistic and defend it
- 📤 Template must expose `Outputs` for the bucket name, table name, and Lambda function ARN
- 🚀 Deploy using `aws cloudformation create-stack` (or `awslocal cloudformation create-stack`) with an appropriate `--capabilities` flag
- ⏳ Poll stack status programmatically until `CREATE_COMPLETE`; handle `ROLLBACK_COMPLETE` failure scenarios gracefully in your workflow

**Constraints:**
- 🚫 No manual resource creation via CLI outside of CloudFormation — everything must be declared as IaC
- 🔁 Template must be idempotent: a second `create-stack` attempt with the same name should not silently succeed with duplicate resources
- 📝 Justify your choice of DynamoDB billing mode and Lambda memory/timeout settings based on assumed production traffic patterns

---

### Phase 2: Application Logic and Data Flow Validation

**Requirements:**
- 🌐 Design a minimal static front-end HTML file describing the app's purpose (content is your choice) and upload it to the deployed S3 bucket using the CLI
- ⚡ Implement Lambda function logic that:
  - 📨 Accepts an event payload (define your own schema)
  - ✍️ Writes a structured item to the DynamoDB table (must include a timestamp and a generated unique ID)
  - ✅ Returns a JSON response confirming success, including the written item's key
- ▶️ Invoke the Lambda function via CLI (`aws lambda invoke` or `awslocal lambda invoke`) with a custom test payload; capture and inspect the response
- 🔍 Scan the DynamoDB table via CLI to confirm the item was persisted correctly — validate attribute types and values match what Lambda wrote
- 📝 Document any IAM permission errors encountered and how you resolved them (a common real-world debugging task)

**Constraints:**
- 🚫 Lambda must not have overly permissive IAM policies (no wildcard `dynamodb:*` on `*` resources) — scope precisely
- ⚠️ Handle and log at least one failure case in your Lambda code (e.g., malformed event payload)

---

### Phase 3: Teardown and Verification

**Requirements:**
- 🗑️ Delete the entire stack using `delete-stack`
- ⏳ Poll for stack deletion completion and handle the case where deletion stalls (e.g., non-empty S3 bucket blocking removal — you must solve this yourself)
- 🔍 Confirm all three resources (S3 bucket, DynamoDB table, Lambda function) no longer exist via individual describe/list/head CLI calls
- 📝 Produce a brief post-mortem: what would break this stack in a real AWS account that didn't break it in LocalStack (e.g., S3 bucket deletion policies, IAM propagation delays)?

---

## 🏆 Success Criteria

- ✅ Stack reaches `CREATE_COMPLETE` on first clean attempt (no manual patching after failure)
- ✅ Lambda invocation returns HTTP 200-equivalent success with a valid item reference
- ✅ DynamoDB scan output contains exactly one item matching the Lambda-written data
- ✅ Stack deletion completes with zero orphaned resources
- ✅ You can articulate, without notes, why each IAM permission you granted was necessary

---

## ✅ Verification

Run independently and capture output as evidence:

```bash
awslocal cloudformation describe-stacks --stack-name <your-stack-name>
awslocal s3 ls s3://<bucket-name>
awslocal dynamodb scan --table-name <table-name>
awslocal lambda list-functions

# After deletion:
awslocal cloudformation describe-stacks --stack-name <your-stack-name>   # should error/not found
```

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| 🏗️ Multi-Service Stack | A single CloudFormation template orchestrating S3, DynamoDB, and Lambda as one deployable unit |
| 🔐 Least-Privilege IAM | Scoping an execution role's policy to only the specific actions and resource ARNs a function needs |
| 📦 Code Packaging | Inline `ZipFile` vs. S3-referenced deployment packages — a production trade-off between simplicity and scale |
| 💳 On-Demand Billing | DynamoDB's `PAY_PER_REQUEST` mode — no capacity planning, ideal for unpredictable or bursty traffic |
| 🔁 Stack Idempotency | Relying on CloudFormation's own `AlreadyExistsException` behavior rather than ad-hoc existence checks |
| 🧹 Clean Teardown | Emptying dependent resources (e.g., S3 objects) before deletion so a stack doesn't stall in `DELETE_FAILED` |

---

## 🏁 Conclusion

### 🎉 Key Accomplishments

- ✅ Independently architected, deployed, and tore down a full-stack serverless application entirely through Infrastructure as Code on LocalStack
- ✅ Made autonomous decisions about IAM scoping, billing modes, Lambda packaging strategy, and failure handling
- ✅ Validated data flow from S3 through Lambda into DynamoDB and confirmed clean resource teardown

### 🌍 Real-World Applications

These decisions mirror real-world **Cloud Solutions Architect** responsibilities, demonstrating production-grade IaC discipline directly relevant to the **AWS Certified Solutions Architect – Associate** exam and to full-stack cloud engineering roles in GCC and global markets.

---

<div align="center">

### 🎓 Al Nafi Cybersecurity Training Labs

*Empowering the Next Generation of Cloud & Security Professionals*

</div>
