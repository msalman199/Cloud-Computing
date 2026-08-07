<div align="center">

# ⚡ Deploy a Lambda Function Locally

![AWS Lambda](https://img.shields.io/badge/AWS_Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![LocalStack](https://img.shields.io/badge/LocalStack-2E2ADB?style=for-the-badge&logo=localstack&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![AWS CLI](https://img.shields.io/badge/AWS_CLI-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)

*Emulate a serverless AWS Lambda workflow entirely on your local machine using LocalStack and Docker*

</div>

---

## 📚 Table of Contents

- [🎯 Learning Objectives](#-learning-objectives)
- [📋 Prerequisites](#-prerequisites)
- [🖥️ Lab Environment](#️-lab-environment)
- [🏗️ Architecture Overview](#️-architecture-overview)
- [📝 Tasks](#-tasks)
  - [Task 1: Provision LocalStack with Lambda + IAM](#task-1-provision-localstack-with-lambda--iam)
  - [Task 2: Author the Lambda Handler](#task-2-author-the-lambda-handler)
  - [Task 3: Create, Invoke, Update, and Redeploy](#task-3-create-invoke-update-and-redeploy)
- [✅ Verification](#-verification)
- [🔧 Troubleshooting Tips](#-troubleshooting-tips)
- [🔑 Key Concepts](#-key-concepts)
- [🏁 Conclusion](#-conclusion)

---

## 🎯 Learning Objectives

| # | Objective |
|---|-----------|
| 1 | 🐳 Deploy and manage LocalStack as a Lambda emulation environment using Docker |
| 2 | 🐍 Design a Python Lambda handler conforming to the AWS Lambda execution contract |
| 3 | 📦 Package, deploy, and version Lambda functions using AWS CLI against a local endpoint |
| 4 | 🔄 Perform function updates and validate behavioral changes through synchronous invocation |
| 5 | 🔍 Analyze invocation payloads, logs, and response structures for debugging serverless workloads |

---

## 📋 Prerequisites

| Requirement | Details |
|-------------|---------|
| ⚡ AWS Lambda Concepts | Solid understanding of handlers, event/context objects, IAM execution roles |
| 🐳 Docker Knowledge | Working knowledge of Docker, Docker Compose, and container networking |
| ☁️ AWS CLI | Proficiency with AWS CLI configuration and command structure |
| 🐍 Python | Python 3.x scripting experience |
| 📄 JSON/IAM | Familiarity with JSON and IAM trust policy documents |

---

## 🖥️ Lab Environment

> 💻 **Al Nafi provides a single Linux machine (Start Lab) with sudo access.**

**Verify/install dependencies:**

```bash
docker --version         # 🐳 confirm Docker engine
docker compose version   # 🐙 confirm Compose plugin
python3 --version        # 🐍 confirm Python 3.x
aws --version             # ☁️ confirm AWS CLI
pip3 install awscli-local # 🔌 install awslocal wrapper
```

- 🛠️ If any tool is missing, install via your distro's package manager (`apt`, `yum`) or `pip`
- ✅ Confirm the Docker daemon is active:

```bash
sudo systemctl status docker   # 🟢 daemon must be running before LocalStack starts
```

---

## 🏗️ Architecture Overview

```
[Host Machine]
   |
   +-- Docker Engine (socket mounted into LocalStack container)
   |
   +-- LocalStack Container
   |      - Lambda service (executes handler in sub-containers)
   |      - IAM service (execution role validation)
   |
   +-- AWS CLI (--endpoint-url=http://localhost:4566)
   |
   +-- Local filesystem: handler.py -> function.zip
```

> ⚠️ **Design decisions you must make**
>
> - 🐳 **Lambda executor mode** (Docker vs Docker-reuse) — evaluate trade-offs for cold-start latency
> - 🔐 **IAM role strategy** — LocalStack accepts dummy ARNs, but structure them realistically
> - 📦 **Payload structure** — define the input/output contract between invoker and handler

---

## 📝 Tasks

### Task 1: Provision LocalStack with Lambda + IAM

- 📄 Create a `docker-compose.yml` defining a LocalStack service
- ⚙️ Required env vars: `SERVICES=lambda,iam`, `DOCKER_HOST=unix:///var/run/docker.sock`, `LAMBDA_EXECUTOR` (research valid values for your LocalStack version)
- 🔌 Mount `/var/run/docker.sock` into the container — this is **mandatory** for Lambda's Docker-based execution
- 🌐 Expose port `4566`

```yaml
# docker-compose.yml — fill in service definition
services:
  localstack:
    image: localstack/localstack:latest
    ports:
      - "4566:4566"          # 🌐 LocalStack gateway port
    environment:
      - SERVICES=# TODO      # 🧩 which local AWS services to enable
      - DOCKER_HOST=# TODO   # 🐳 socket path for Lambda's Docker executor
    volumes:
      - # TODO: mount docker socket
      - # TODO: mount a temp data dir if needed
```

- ▶️ Start the stack:

```bash
docker compose up -d   # 🚀 launch LocalStack in the background
```

- 🩺 Confirm health:

```bash
curl http://localhost:4566/_localstack/health   # ✅ check service status
```

---

### Task 2: Author the Lambda Handler

- 🐍 Write `handler.py` implementing the standard Lambda contract
- 📤 Function must accept `event` and `context`, return a dict with `statusCode` and JSON-encoded `body` containing a greeting derived from an input field (e.g., `name`)

```python
def lambda_handler(event: dict, context) -> dict:
    """
    AWS Lambda entrypoint.

    Args:
        event: Invocation payload (expects a 'name' key)
        context: Lambda context object (runtime metadata)

    Returns:
        dict with 'statusCode' and JSON string 'body'
    """
    # TODO: extract 'name' from event, default if missing
    # TODO: build greeting message
    # TODO: return properly structured response dict
    pass
```

- 📦 Package it — create a ZIP containing `handler.py` at the archive root:

```bash
zip function.zip handler.py   # 🗜️ package the handler
unzip -l function.zip          # 🔍 verify handler.py sits at archive root
```

---

### Task 3: Create, Invoke, Update, and Redeploy

- 🏗️ Create the function using `awslocal` or `aws --endpoint-url=http://localhost:4566`
- 🧾 Required parameters: `--function-name`, `--runtime` (choose an appropriate Python runtime), `--role` (dummy ARN, e.g. `arn:aws:iam::000000000000:role/lambda-role`), `--handler` (`module.function`), `--zip-file`
- ▶️ Invoke synchronously, passing a JSON payload via `--payload`, capture output to a file
- 🔎 Inspect the response file and decode the base64/JSON body
- ✏️ Modify `handler.py` (e.g., add a timestamp or change message format), rezip, and use `update-function-code` to redeploy
- 🔁 Re-invoke and diff the output against the first invocation

```bash
# Reference commands — supply correct flags/values yourself

awslocal lambda create-function \
  --function-name greeting-fn \
  --runtime <choose-runtime> \          # 🐍 pick a supported Python runtime
  --role <dummy-role-arn> \             # 🔐 dummy IAM execution role
  --handler <module>.<function> \       # 🎯 entrypoint reference
  --zip-file fileb://function.zip       # 📦 deployment package

awslocal lambda invoke \
  --function-name greeting-fn \
  --payload '<json-payload>' \          # 📨 test event
  response.json                          # 📥 capture invocation output

awslocal lambda update-function-code \
  --function-name greeting-fn \
  --zip-file fileb://function.zip        # 🔄 push updated handler code
```

---

## ✅ Verification

- `awslocal lambda list-functions` shows `greeting-fn` with expected runtime and last-modified timestamp
- `response.json` from the first invocation contains the original greeting logic output
- After update, `awslocal lambda get-function --function-name greeting-fn` shows a new `CodeSha256` value
- Second invocation's response reflects the modified handler logic (verify via diff or manual inspection)
- `docker logs <localstack-container-id>` shows successful invocation entries with no unhandled exceptions

---

## 🔧 Troubleshooting Tips

<details>
<summary><b>🐳 Invocation hangs or fails with Docker errors</b></summary>

Confirm socket mount permissions and the `DOCKER_HOST` value in your `docker-compose.yml`.

</details>

<details>
<summary><b>📦 Handler import errors</b></summary>

Usually indicates a ZIP structure issue — `handler.py` must be at the archive root, not nested in a subfolder.

</details>

<details>
<summary><b>🔄 Stale code after update</b></summary>

Confirm you rezipped `handler.py` before calling `update-function-code`.

</details>

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| 🖥️ LocalStack | Emulates AWS services (Lambda, IAM) locally for cloud-cost-free development |
| 🐳 Docker Executor | Runs each Lambda invocation inside a sub-container, mirroring AWS's isolation model |
| 📇 Lambda Contract | `handler(event, context)` signature every AWS Lambda function must implement |
| 🔐 Dummy IAM Roles | LocalStack accepts placeholder ARNs while still enforcing realistic role structure |
| 🔁 Function Lifecycle | Create → invoke → update → redeploy → re-invoke, mirroring real AWS deployment cycles |
| 🔍 CodeSha256 | Hash used to verify that a function's deployed code actually changed after an update |

---

## 🏁 Conclusion

### 🎉 Key Accomplishments

- ✅ Provisioned a local serverless environment using LocalStack with Docker-based Lambda execution
- ✅ Authored and packaged a Python handler conforming to AWS's invocation contract
- ✅ Managed the full function lifecycle — creation, invocation, code update, and redeployment — entirely through AWS CLI against a local endpoint

### 🌍 Real-World Applications

This workflow mirrors real-world serverless development practices, enabling rapid iteration without cloud costs, and reinforces core competencies aligned with the **AWS Certified Developer – Associate** certification and **Cloud Developer / Serverless Engineer** roles.

---

<div align="center">

### 🎓 Al Nafi Cybersecurity Training Labs

*Empowering the Next Generation of Cloud & Security Professionals*

</div>
