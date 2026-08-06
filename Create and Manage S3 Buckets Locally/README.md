<div align="center">

# 🪣 Create and Manage S3 Buckets Locally

### Practicing Amazon S3 Object Storage with LocalStack

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Amazon S3](https://img.shields.io/badge/Amazon_S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
![LocalStack](https://img.shields.io/badge/LocalStack-6C2EB5?style=for-the-badge&logo=localstack&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)

</div>

---

## 📑 Table of Contents

- [🎯 Learning Objectives](#-learning-objectives)
- [📋 Prerequisites](#-prerequisites)
- [🖥️ Environment Setup](#️-environment-setup)
- [🚀 Task 1: Start LocalStack with S3 Enabled](#-task-1-start-localstack-with-s3-enabled)
- [🪣 Task 2: Create an S3 Bucket](#-task-2-create-an-s3-bucket)
- [📤 Task 3: Upload a Sample File](#-task-3-upload-a-sample-file)
- [📥 Task 4: Download and Compare the File](#-task-4-download-and-compare-the-file)
- [🕒 Task 5: Enable Versioning and Upload a Modified File](#-task-5-enable-versioning-and-upload-a-modified-file)
- [📜 Task 6: List Object Versions](#-task-6-list-object-versions)
- [🗑️ Task 7: Delete Object, Delete Bucket, and Confirm Cleanup](#️-task-7-delete-object-delete-bucket-and-confirm-cleanup)
- [🧹 Optional Cleanup: Stop LocalStack](#-optional-cleanup-stop-localstack)
- [✅ Verification](#-verification)
- [🛠️ Troubleshooting Tips](#️-troubleshooting-tips)
- [🔑 Key Concepts](#-key-concepts)
- [🏁 Conclusion](#-conclusion)

---

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

| # | Objective |
|---|-----------|
| 1 | 🧠 Explain the basics of Amazon S3 object storage concepts (buckets, objects, versioning) |
| 2 | 🚀 Start a local S3-compatible service using LocalStack and Docker |
| 3 | 🪣 Create and manage S3 buckets using the AWS CLI |
| 4 | 📤 Upload, download, and version objects in an S3 bucket |
| 5 | 🧹 Clean up cloud storage resources properly |

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| 🖥️ Machine | A single Linux machine (provided via Al Nafi Start Lab) |
| 💻 Terminal familiarity | Basic comfort running commands and navigating folders |
| ☁️ AWS experience | Not required — this lab starts from zero |
| 🌐 Internet access | Needed to download Docker images and packages |

## 🖥️ Environment Setup

> **📍 Setup Overview**
> Run these commands on your Al Nafi Linux machine to prepare the lab environment.

![Docker](https://img.shields.io/badge/Tool-Docker-2496ED?style=flat-square&logo=docker&logoColor=white)

### Step 1: Install Docker

```bash
# 📦 Update package lists
sudo apt-get update

# 🐳 Install Docker
sudo apt-get install -y docker.io

# ▶️ Start Docker service
sudo systemctl start docker

# ✅ Verify Docker is running
sudo docker --version
```

![AWS CLI](https://img.shields.io/badge/Tool-AWS_CLI-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

### Step 2: Install AWS CLI

```bash
# 🐍 Install pip (Python package manager) if not present
sudo apt-get install -y python3-pip

# ☁️ Install AWS CLI using pip
pip3 install awscli --upgrade --user

# ✅ Verify installation
aws --version
```

![Credentials](https://img.shields.io/badge/Config-Dummy_Credentials-232F3E?style=flat-square&logo=amazonaws&logoColor=white)

### Step 3: Configure Dummy AWS Credentials

LocalStack does not check real AWS credentials, but the CLI requires some values to be set.

```bash
# 🧾 Set dummy credentials (any values work for LocalStack)
aws configure set aws_access_key_id test
aws configure set aws_secret_access_key test
aws configure set region us-east-1
```

---

## 🚀 Task 1: Start LocalStack with S3 Enabled

![LocalStack](https://img.shields.io/badge/Service-LocalStack-6C2EB5?style=flat-square&logo=localstack&logoColor=white) ![S3](https://img.shields.io/badge/Service-S3-569A31?style=flat-square&logo=amazons3&logoColor=white)

```bash
# ▶️ Run LocalStack container with S3 service enabled
sudo docker run -d \
  --name localstack \
  -p 4566:4566 \
  -e SERVICES=s3 \
  localstack/localstack

# ✅ Check the container is running
sudo docker ps
```

- `-p 4566:4566` maps LocalStack's main port to your machine
- `-e SERVICES=s3` tells LocalStack to only start the S3 service
- ⏳ Wait about 10-15 seconds for LocalStack to fully initialize

**Verify LocalStack is ready:**

```bash
# 🩺 Check LocalStack health status
curl http://localhost:4566/_localstack/health
```

You should see `"s3": "available"` in the output.

---

## 🪣 Task 2: Create an S3 Bucket

![S3](https://img.shields.io/badge/S3-Bucket_Creation-569A31?style=flat-square&logo=amazons3&logoColor=white)

```bash
# 🪣 Create a bucket named my-project-data
aws --endpoint-url=http://localhost:4566 s3 mb s3://my-project-data

# 📋 List all buckets to confirm creation
aws --endpoint-url=http://localhost:4566 s3 ls
```

- `--endpoint-url` redirects the AWS CLI to talk to LocalStack instead of real AWS
- `s3 mb` means "make bucket"

---

## 📤 Task 3: Upload a Sample File

![S3](https://img.shields.io/badge/S3-Upload-569A31?style=flat-square&logo=amazons3&logoColor=white)

```bash
# 📝 Create a sample text file
echo "Hello from my first S3 bucket!" > sample.txt

# 📤 Upload the file to the bucket
aws --endpoint-url=http://localhost:4566 s3 cp sample.txt s3://my-project-data/

# 📋 List objects in the bucket to verify upload
aws --endpoint-url=http://localhost:4566 s3 ls s3://my-project-data/
```

---

## 📥 Task 4: Download and Compare the File

![S3](https://img.shields.io/badge/S3-Download-569A31?style=flat-square&logo=amazons3&logoColor=white)

```bash
# 📁 Create a separate directory for download
mkdir ~/downloaded-files

# 📥 Download the file into the new directory
aws --endpoint-url=http://localhost:4566 s3 cp \
  s3://my-project-data/sample.txt \
  ~/downloaded-files/sample.txt

# 🔍 Compare original and downloaded file contents
diff sample.txt ~/downloaded-files/sample.txt

# TODO: If diff produces no output, the files are identical
echo "Comparison complete - no output above means files match"
```

---

## 🕒 Task 5: Enable Versioning and Upload a Modified File

![S3](https://img.shields.io/badge/S3-Versioning-569A31?style=flat-square&logo=amazons3&logoColor=white)

```bash
# 🕒 Enable versioning on the bucket
aws --endpoint-url=http://localhost:4566 s3api put-bucket-versioning \
  --bucket my-project-data \
  --versioning-configuration Status=Enabled

# ✏️ Modify the sample file
echo "This is version 2 of the file." >> sample.txt

# 📤 Upload the modified file (overwrites the same key)
aws --endpoint-url=http://localhost:4566 s3 cp sample.txt s3://my-project-data/
```

---

## 📜 Task 6: List Object Versions

![S3](https://img.shields.io/badge/S3-Object_Versions-569A31?style=flat-square&logo=amazons3&logoColor=white)

```bash
# 📜 List all versions of objects in the bucket
aws --endpoint-url=http://localhost:4566 s3api list-object-versions \
  --bucket my-project-data
```

- 🔎 Look for the `Versions` array in the JSON output
- 🆔 Each version has a unique `VersionId`
- ⭐ The most recent version has `"IsLatest": true`

---

## 🗑️ Task 7: Delete Object, Delete Bucket, and Confirm Cleanup

![S3](https://img.shields.io/badge/S3-Cleanup-569A31?style=flat-square&logo=amazons3&logoColor=white)

```bash
# 🗑️ Delete the object from the bucket
aws --endpoint-url=http://localhost:4566 s3 rm s3://my-project-data/sample.txt

# ✅ Confirm the bucket is now empty
aws --endpoint-url=http://localhost:4566 s3 ls s3://my-project-data/

# 🗑️ Delete the empty bucket
aws --endpoint-url=http://localhost:4566 s3 rb s3://my-project-data

# ✅ Confirm the bucket no longer exists
aws --endpoint-url=http://localhost:4566 s3 ls
```

- `s3 rm` deletes an object
- `s3 rb` means "remove bucket" (bucket must be empty first)

---

## 🧹 Optional Cleanup: Stop LocalStack

![Docker](https://img.shields.io/badge/Cleanup-Stop_Container-2496ED?style=flat-square&logo=docker&logoColor=white)

```bash
# 🛑 Stop and remove the LocalStack container
sudo docker stop localstack
sudo docker rm localstack
```

---

## ✅ Verification

Confirm your lab work is complete by checking:

- [ ] 🐳 `docker ps` showed the `localstack` container running during the lab
- [ ] 🪣 `aws s3 ls` (with endpoint URL) showed `my-project-data` after creation
- [ ] 📤 `sample.txt` appeared in the bucket listing after upload
- [ ] 🔍 `diff` command produced no output (downloaded file matched original)
- [ ] 📜 `list-object-versions` output showed at least 2 versions of `sample.txt`
- [ ] 🗑️ Final `aws s3 ls` command showed no buckets remaining

---

## 🛠️ Troubleshooting Tips

<details>
<summary><strong>🔌 "Could not connect to endpoint" error</strong></summary>

Wait 10-15 seconds after starting LocalStack, then retry.

</details>

<details>
<summary><strong>❓ "command not found: aws"</strong></summary>

Run `export PATH=$PATH:~/.local/bin` and try again.

</details>

<details>
<summary><strong>🔓 Docker permission denied</strong></summary>

Use `sudo` before docker commands, or add your user to the docker group with `sudo usermod -aG docker $USER` (requires logout/login).

</details>

<details>
<summary><strong>🪣 Bucket already exists error</strong></summary>

Someone else may have created it; try a different bucket name or delete the existing one first.

</details>

<details>
<summary><strong>🔌 Port 4566 already in use</strong></summary>

Stop any existing LocalStack container with `sudo docker stop localstack` before starting a new one.

</details>

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| 🚀 **LocalStack** | A local AWS cloud emulator — running just the S3 service here means no real AWS account or billing is required |
| 🪣 **Bucket** | A top-level S3 container for objects, created with `s3 mb` and identified by a globally-unique name |
| 📦 **Object** | A file stored inside a bucket, uploaded and retrieved with `s3 cp` |
| 🕒 **Versioning** | An S3 feature (`put-bucket-versioning`) that keeps every past copy of an object, each with its own `VersionId` |
| 🌐 **`--endpoint-url`** | The flag that redirects AWS CLI traffic to LocalStack (`http://localhost:4566`) instead of real AWS |
| 🧹 **Resource Cleanup** | Objects must be deleted (`s3 rm`) before an empty bucket can be removed (`s3 rb`) — mirrors real AWS behavior |

---

## 🏁 Conclusion

In this lab, you learned the fundamentals of Amazon S3 object storage using LocalStack as a free, local emulator.

### 🏆 Key Accomplishments

- 🚀 Started a containerized S3 service with Docker
- 🪣 Created and managed a bucket using the AWS CLI
- 📤 Practiced core storage operations including uploading, downloading, and verifying file integrity
- 🕒 Enabled versioning to understand how S3 tracks multiple copies of an object over time
- 🗑️ Practiced proper resource cleanup by deleting objects and buckets

### 🌍 Real-World Applications

These skills directly translate to real-world AWS S3 usage and form a foundational building block for cloud storage administration — a key competency for the **AWS Certified Cloud Practitioner** certification and **Cloud Engineer** roles in the GCC IT industry.

---

<div align="center">

![Al Nafi](https://img.shields.io/badge/Al%20Nafi-Cybersecurity%20Training-1a1a2e?style=for-the-badge)

</div>
