<div align="center">

# 🗄️ Create a DynamoDB Table and Perform CRUD

### Practicing Serverless NoSQL Operations with LocalStack

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![DynamoDB](https://img.shields.io/badge/DynamoDB-4053D6?style=for-the-badge&logo=amazondynamodb&logoColor=white)
![LocalStack](https://img.shields.io/badge/LocalStack-6C2EB5?style=for-the-badge&logo=localstack&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)

</div>

---

## 📑 Table of Contents

- [🎯 Objectives](#-objectives)
- [📋 Prerequisites](#-prerequisites)
- [🖥️ Environment Setup](#️-environment-setup)
- [🚀 Task 1: Start LocalStack with DynamoDB Enabled](#-task-1-start-localstack-with-dynamodb-enabled)
- [🗄️ Task 2: Create the Products Table](#️-task-2-create-the-products-table)
- [📝 Task 3: Insert Sample Items (Create)](#-task-3-insert-sample-items-create)
- [🔍 Task 4: Read an Item (Query)](#-task-4-read-an-item-query)
- [✏️ Task 5: Update an Item](#️-task-5-update-an-item)
- [📡 Task 6: Scan the Entire Table](#-task-6-scan-the-entire-table)
- [🗑️ Task 7: Delete an Item and the Table](#️-task-7-delete-an-item-and-the-table)
- [✅ Verification](#-verification)
- [🛠️ Troubleshooting](#️-troubleshooting)
- [🔑 Key Concepts](#-key-concepts)
- [🏁 Conclusion](#-conclusion)

---

## 🎯 Objectives

By the end of this lab, you will be able to:

| # | Objective |
|---|-----------|
| 1 | 🚀 Run LocalStack (open-source AWS emulator) via Docker Compose |
| 2 | 🗄️ Create a DynamoDB table with a partition key and provisioned throughput |
| 3 | 📝 Perform Create, Read, Update, Delete (CRUD) operations using AWS CLI |
| 4 | 🔍 Query and scan a DynamoDB table |
| 5 | 🗑️ Clean up resources by deleting items and tables |

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| 💻 Terminal familiarity | Basic Linux command-line familiarity |
| 🧩 NoSQL concepts | Understanding of tables, keys, items vs. relational rows |
| 🧾 JSON syntax | Familiarity with JSON |
| 🐳 Docker | Docker and Docker Compose installed (or install steps below) |
| ☁️ AWS CLI | AWS CLI v2 installed (used only to talk to LocalStack, no real AWS account needed) |

## 🖥️ Environment Setup

> **📍 Setup Overview**
> You are working on a single Linux machine (provided via Start Lab).

![Docker](https://img.shields.io/badge/Tool-Docker-2496ED?style=flat-square&logo=docker&logoColor=white)

### 1️⃣ Install Docker and Docker Compose (if not present)

```bash
sudo apt-get update -y
sudo apt-get install -y docker.io docker-compose-plugin
sudo systemctl start docker
sudo usermod -aG docker $USER && newgrp docker
```

![AWS CLI](https://img.shields.io/badge/Tool-AWS_CLI_v2-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

### 2️⃣ Install AWS CLI v2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

![Credentials](https://img.shields.io/badge/Config-Dummy_Credentials-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

### 3️⃣ Configure dummy AWS credentials

Required by the CLI, not validated by LocalStack.

```bash
aws configure set aws_access_key_id test
aws configure set aws_secret_access_key test
aws configure set region us-east-1
```

---

## 🚀 Task 1: Start LocalStack with DynamoDB Enabled

![LocalStack](https://img.shields.io/badge/Service-LocalStack-6C2EB5?style=flat-square&logo=localstack&logoColor=white) ![DynamoDB](https://img.shields.io/badge/Service-DynamoDB-4053D6?style=flat-square&logo=amazondynamodb&logoColor=white)

Create a working directory and a `docker-compose.yml` file.

```bash
mkdir ~/dynamodb-lab && cd ~/dynamodb-lab
nano docker-compose.yml
```

Complete the file below (`TODO` items must be filled in):

```yaml
version: "3.8"
services:
  localstack:
    image: localstack/localstack:latest
    container_name: localstack
    ports:
      - "4566:4566"
    environment:
      # TODO: enable only the dynamodb service
      - SERVICES=___
      - DEBUG=1
    volumes:
      - "./volume:/var/lib/localstack"
```

### ▶️ Start the container

```bash
docker compose up -d
docker ps
```

### 🩺 Verify DynamoDB endpoint is live

```bash
curl http://localhost:4566/_localstack/health
```

> ℹ️ **Note:** All subsequent CLI commands need `--endpoint-url http://localhost:4566`. To simplify, define an alias:

```bash
alias ddb="aws --endpoint-url=http://localhost:4566 dynamodb"
```

---

## 🗄️ Task 2: Create the Products Table

![DynamoDB](https://img.shields.io/badge/DynamoDB-Create_Table-4053D6?style=flat-square&logo=amazondynamodb&logoColor=white)

Fill in the missing key schema and throughput values.

```bash
ddb create-table \
  --table-name Products \
  --attribute-definitions AttributeName=ProductID,AttributeType=S \
  --key-schema AttributeName=ProductID,KeyType=___ \
  --provisioned-throughput ReadCapacityUnits=___,WriteCapacityUnits=___
```

**Confirm creation:**

```bash
ddb list-tables
ddb describe-table --table-name Products
```

---

## 📝 Task 3: Insert Sample Items (Create)

![DynamoDB](https://img.shields.io/badge/DynamoDB-Put_Item-4053D6?style=flat-square&logo=amazondynamodb&logoColor=white)

Insert three products. Complete the second and third `put-item` commands based on the pattern of the first.

```bash
ddb put-item --table-name Products --item '{
  "ProductID": {"S": "P001"},
  "Name": {"S": "Laptop"},
  "Price": {"N": "899.99"},
  "Stock": {"N": "15"}
}'

# TODO: Insert P002 - "Wireless Mouse", Price 19.99, Stock 100
ddb put-item --table-name Products --item '{ ___ }'

# TODO: Insert P003 - "USB-C Hub", Price 29.50, Stock 40
ddb put-item --table-name Products --item '{ ___ }'
```

---

## 🔍 Task 4: Read an Item (Query)

![DynamoDB](https://img.shields.io/badge/DynamoDB-Get_%26_Query-4053D6?style=flat-square&logo=amazondynamodb&logoColor=white)

Retrieve a specific product by key using `get-item`:

```bash
ddb get-item --table-name Products --key '{"ProductID": {"S": "P001"}}'
```

💡 **Challenge:** Use `query` (not `get-item`) to fetch P002, using `--key-condition-expression` and `--expression-attribute-values`. Reference the AWS CLI docs for the exact syntax.

```bash
ddb query --table-name Products \
  --key-condition-expression "ProductID = :id" \
  --expression-attribute-values '{":id": {"S": "___"}}'
```

---

## ✏️ Task 5: Update an Item

![DynamoDB](https://img.shields.io/badge/DynamoDB-Update_Item-4053D6?style=flat-square&logo=amazondynamodb&logoColor=white)

Update the price of P001 to `849.99` using `update-item`. Complete the expression syntax:

```bash
ddb update-item --table-name Products \
  --key '{"ProductID": {"S": "P001"}}' \
  --update-expression "SET Price = :newPrice" \
  --expression-attribute-values '{":newPrice": {"N": "___"}}' \
  --return-values ALL_NEW
```

Verify the update with `get-item` on P001.

---

## 📡 Task 6: Scan the Entire Table

![DynamoDB](https://img.shields.io/badge/DynamoDB-Scan-4053D6?style=flat-square&logo=amazondynamodb&logoColor=white)

```bash
ddb scan --table-name Products
```

💡 **Optional filter challenge:** Scan only items where `Stock` is greater than 20. Look up `--filter-expression` syntax and complete:

```bash
ddb scan --table-name Products \
  --filter-expression "Stock > :minStock" \
  --expression-attribute-values '{":minStock": {"N": "___"}}'
```

---

## 🗑️ Task 7: Delete an Item and the Table

![DynamoDB](https://img.shields.io/badge/DynamoDB-Delete-4053D6?style=flat-square&logo=amazondynamodb&logoColor=white)

Delete P003:

```bash
ddb delete-item --table-name Products --key '{"ProductID": {"S": "___"}}'
```

Confirm it's gone (scan or `get-item`), then delete the table:

```bash
ddb delete-table --table-name Products
ddb list-tables
```

---

## ✅ Verification

Confirm lab completion on the same machine:

- 🐳 `docker ps` shows the `localstack` container running (before cleanup)
- 🗄️ `ddb list-tables` initially returned `Products`, later returns an empty list after Task 7
- 📡 `ddb scan --table-name Products` (before deletion) showed 3 items, then 2 after delete
- ✏️ `get-item` for P001 after Task 5 shows updated price `849.99`
- ✅ No errors returned from any CLI command (check for `ResourceNotFoundException` if table name is mistyped)

---

## 🛠️ Troubleshooting

<details>
<summary><strong>🔌 curl health check fails</strong></summary>

Run `docker logs localstack` to check startup errors.

</details>

<details>
<summary><strong>⏳ CLI hangs</strong></summary>

Confirm `--endpoint-url` is included or the `ddb` alias is active in the current shell.

</details>

<details>
<summary><strong>🚫 "Table not found" errors</strong></summary>

Usually means the table was created in a different region than configured.

</details>

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| 🚀 **LocalStack** | A local AWS cloud emulator — running just the DynamoDB service here means no real AWS account or billing is required |
| 🗄️ **Table & Partition Key** | DynamoDB's core structure — `Products` is keyed on `ProductID`, which uniquely identifies each item |
| 📝 **Item** | DynamoDB's equivalent of a row — a JSON document with typed attributes (`S` for string, `N` for number) |
| 🔍 **Query vs. Scan** | `query` retrieves items efficiently by key condition; `scan` reads the entire table and can be filtered client-side with `--filter-expression` |
| ✏️ **Update Expression** | The `SET`-based syntax (`--update-expression`) used to modify specific attributes of an existing item without replacing it |
| 🗑️ **Cleanup Order** | Items should be deleted (or verified) before the table itself is dropped — mirrors safe cleanup practice across AWS services |

---

## 🏁 Conclusion

In this lab, you deployed a local DynamoDB environment using LocalStack and Docker Compose, avoiding real AWS costs while learning core serverless database skills.

### 🏆 Key Accomplishments

- 🚀 Ran LocalStack via Docker Compose with the DynamoDB service enabled
- 🗄️ Created a `Products` table with a partition key
- 📝 Performed full CRUD operations (`put-item`, `get-item`, `query`, `update-item`, `scan`, `delete-item`)
- 🗑️ Cleaned up by deleting the table

### 🌍 Real-World Applications

These skills map directly to real-world DynamoDB usage on AWS and are foundational for the **AWS Certified Developer - Associate** certification and **Cloud Developer/DBA** roles in the GCC and global cloud market.

---

<div align="center">

![Al Nafi](https://img.shields.io/badge/Al%20Nafi-Cybersecurity%20Training-1a1a2e?style=for-the-badge)

</div>
