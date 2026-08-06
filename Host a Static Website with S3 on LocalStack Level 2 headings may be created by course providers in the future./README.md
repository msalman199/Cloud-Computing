<div align="center">

# 🌐 Host a Static Website with S3 on LocalStack

### Deploying and Serving a Static Site on an Emulated S3 Bucket

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Amazon S3](https://img.shields.io/badge/Amazon_S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
![LocalStack](https://img.shields.io/badge/LocalStack-6C2EB5?style=for-the-badge&logo=localstack&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)

</div>

---

## 📑 Table of Contents

- [🎯 Learning Objectives](#-learning-objectives)
- [📋 Prerequisites](#-prerequisites)
- [🖥️ Environment Setup](#️-environment-setup)
- [🪣 Task 1: Create the S3 Bucket and Enable Static Hosting](#-task-1-create-the-s3-bucket-and-enable-static-hosting)
- [📝 Task 2: Create the HTML Files](#-task-2-create-the-html-files)
- [📤 Task 3: Upload Files to the Bucket](#-task-3-upload-files-to-the-bucket)
- [🔓 Task 4: Apply a Public Read Bucket Policy](#-task-4-apply-a-public-read-bucket-policy)
- [🌐 Task 5: Access the Website](#-task-5-access-the-website)
- [🔄 Task 6: Update and Redeploy Content](#-task-6-update-and-redeploy-content)
- [✅ Verification](#-verification)
- [🛠️ Troubleshooting](#️-troubleshooting)
- [🔑 Key Concepts](#-key-concepts)
- [🏁 Conclusion](#-conclusion)

---

## 🎯 Learning Objectives

By the end of this lab, you will be able to:

| # | Objective |
|---|-----------|
| 1 | 🌐 Deploy a static website using S3 bucket hosting emulated on LocalStack |
| 2 | 🔓 Configure bucket policies to grant public read access |
| 3 | 📤 Upload, verify, and update website content served from a local S3 endpoint |

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| 💻 Terminal familiarity | Comfortable with Linux terminal commands and basic AWS S3 concepts (buckets, objects) |
| 🐳 Docker | Docker and Docker Compose installed |
| 🐍 Python | Python 3 and pip installed |
| 🧠 Conceptual grounding | Basic understanding of JSON (for bucket policies) and HTML |

## 🖥️ Environment Setup

> **📍 Setup Overview**
> Al Nafi provides a single Linux machine. Start your lab and open a terminal.

![pip](https://img.shields.io/badge/Tool-pip-3776AB?style=flat-square&logo=python&logoColor=white)

### 1️⃣ Install AWS CLI and LocalStack

```bash
pip install awscli-local localstack
```

![LocalStack](https://img.shields.io/badge/Service-LocalStack-6C2EB5?style=flat-square&logo=localstack&logoColor=white)

### 2️⃣ Start LocalStack (S3 service)

```bash
docker run -d --name localstack -p 4566:4566 \
  -e SERVICES=s3 \
  localstack/localstack
```

![Health Check](https://img.shields.io/badge/Check-Health_Endpoint-brightgreen?style=flat-square&logo=curl&logoColor=white)

### 3️⃣ Confirm the service is healthy

```bash
curl http://localhost:4566/_localstack/health
```

> **📝 TODO:** Check the JSON output — the `s3` key should show `"available"` or `"running"`.

---

## 🪣 Task 1: Create the S3 Bucket and Enable Static Hosting

![S3](https://img.shields.io/badge/S3-Static_Website-569A31?style=flat-square&logo=amazons3&logoColor=white)

Use `awslocal` (a wrapper for `aws --endpoint-url=http://localhost:4566`).

```bash
awslocal s3 mb s3://my-static-site
```

> **📝 TODO:** Enable static website hosting on the bucket with `index.html` as the index document and `error.html` as the error document. Reference the AWS CLI command `s3 website`:

```bash
awslocal s3 website s3://my-static-site/ \
  --index-document _____ \
  --error-document _____
```

---

## 📝 Task 2: Create the HTML Files

![HTML5](https://img.shields.io/badge/File-HTML-E34F26?style=flat-square&logo=html5&logoColor=white)

### 1️⃣ Create a working directory

```bash
mkdir ~/static-site && cd ~/static-site
```

### 2️⃣ Create `index.html`

```html
<!DOCTYPE html>
<html>
<head><title>My Static Site</title></head>
<body>
  <h1>Welcome to My LocalStack Static Site</h1>
  <p>Version 1</p>
</body>
</html>
```

> **📝 TODO:** Create `error.html` with a simple "404 - Page Not Found" message following the same structure.

---

## 📤 Task 3: Upload Files to the Bucket

![S3](https://img.shields.io/badge/S3-Upload-569A31?style=flat-square&logo=amazons3&logoColor=white)

```bash
awslocal s3 cp index.html s3://my-static-site/
awslocal s3 cp error.html s3://my-static-site/
```

> **📝 TODO:** Verify both objects exist using `awslocal s3 ls s3://my-static-site/`

---

## 🔓 Task 4: Apply a Public Read Bucket Policy

![S3](https://img.shields.io/badge/S3-Bucket_Policy-569A31?style=flat-square&logo=amazons3&logoColor=white) ![JSON](https://img.shields.io/badge/Format-JSON-000000?style=flat-square&logo=json&logoColor=white)

Create a file named `policy.json`. Complete the missing fields:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "_____",
      "Principal": "_____",
      "Action": "_____",
      "Resource": "arn:aws:s3:::my-static-site/*"
    }
  ]
}
```

**Hints:**

- 💡 `Effect` should allow access
- 💡 `Principal` should be `"*"` for public access
- 💡 `Action` should permit reading objects (`s3:GetObject`)

**Apply the policy:**

```bash
awslocal s3api put-bucket-policy \
  --bucket my-static-site \
  --policy file://policy.json
```

---

## 🌐 Task 5: Access the Website

![S3](https://img.shields.io/badge/S3-Website_Access-569A31?style=flat-square&logo=amazons3&logoColor=white)

LocalStack serves static websites via a path-style or virtual-host-style URL. Try:

```bash
curl http://localhost:4566/my-static-site/index.html
```

> **📝 TODO:** Confirm the HTML content of `index.html` is returned in the terminal output.

💡 **Optional:** Open the same URL in a browser if the lab machine has a GUI.

---

## 🔄 Task 6: Update and Redeploy Content

![S3](https://img.shields.io/badge/S3-Redeploy-569A31?style=flat-square&logo=amazons3&logoColor=white)

### 1️⃣ Edit `index.html`

Change `"Version 1"` to `"Version 2"` and add one new line of content.

### 2️⃣ Re-upload the file

```bash
awslocal s3 cp index.html s3://my-static-site/
```

### 3️⃣ Re-verify

Re-run the `curl` command from Task 5 and confirm the updated content appears.

---

## ✅ Verification

Run the following checks to confirm successful completion:

```bash
# ✅ Check bucket exists
awslocal s3 ls

# ✅ Check website configuration
awslocal s3api get-bucket-website --bucket my-static-site

# ✅ Check bucket policy applied
awslocal s3api get-bucket-policy --bucket my-static-site

# ✅ Confirm updated content is live
curl http://localhost:4566/my-static-site/index.html | grep "Version 2"
```

**Expected outcomes:**

- 🪣 Bucket `my-static-site` listed and configured for website hosting
- 🔓 Policy shows `"Effect": "Allow"` and `"Principal": "*"`
- 🔄 `curl` output contains the updated `"Version 2"` text

---

## 🛠️ Troubleshooting

<details>
<summary><strong>🔌 LocalStack container not responding</strong></summary>

Run `docker ps` to confirm it's running; check logs with `docker logs localstack`.

</details>

<details>
<summary><strong>❓ awslocal: command not found</strong></summary>

Reinstall with `pip install awscli-local` and ensure your `PATH` includes pip's bin directory.

</details>

<details>
<summary><strong>🚫 Policy rejected</strong></summary>

Validate JSON syntax with `cat policy.json | python3 -m json.tool`.

</details>

<details>
<summary><strong>🔍 404 on curl</strong></summary>

Confirm object key names match exactly (`index.html`, case-sensitive) and the bucket website config was applied.

</details>

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| 🚀 **LocalStack** | A local AWS cloud emulator — running just the S3 service here means no real AWS account or billing is required |
| 🌐 **S3 Static Website Hosting** | An S3 bucket setting (`s3 website`) that designates an index document and an error document, letting the bucket serve HTML directly |
| 🔓 **Bucket Policy** | A JSON document attached to a bucket (not a user or group) that can grant public, unauthenticated read access via `Principal: "*"` |
| 📤 **Object Upload** | Website files (HTML, CSS, JS) are uploaded as regular S3 objects with `s3 cp`, keyed by their file path |
| 🔄 **Update-and-Redeploy** | Re-uploading a file to the same key overwrites the live content — the core workflow behind static site updates |
| 🌐 **`awslocal`** | A wrapper command that pre-fills `--endpoint-url=http://localhost:4566`, routing AWS CLI calls to LocalStack instead of real AWS |

---

## 🏁 Conclusion

In this lab, you deployed a fully functional static website using an S3 bucket emulated locally through LocalStack.

### 🏆 Key Accomplishments

- 🌐 Configured website hosting settings on an S3 bucket
- 📝 Wrote and uploaded HTML files (`index.html`, `error.html`)
- 🔓 Applied a public-read bucket policy using JSON IAM syntax
- ✅ Verified content delivery via `curl`
- 🔄 Practiced the update-and-redeploy workflow common in real-world static site management

### 🌍 Real-World Applications

These skills mirror actual AWS S3 static hosting workflows used by **Cloud** and **Web Developers**, while using a cost-free, local-only environment suitable for practice and experimentation.

---

<div align="center">

![Al Nafi](https://img.shields.io/badge/Al%20Nafi-Cybersecurity%20Training-1a1a2e?style=for-the-badge)

</div>
