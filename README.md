# AWS Authentication with GitHub Actions (OIDC)

## What is This Project?

This project shows how to **connect GitHub Actions to AWS securely** **without using AWS Access Keys**.

Instead of storing secrets, we use **OIDC (OpenID Connect)** which gives **temporary AWS access** only when the workflow runs.

---

## Why Use OIDC?

1. No AWS Access Key
2. No Secret Key stored in GitHub

3. More secure
4. AWS recommended approach
5. Temporary credentials (auto-expire)

---

## Simple Flow (How It Works)

1. GitHub Actions starts a workflow
2. GitHub creates a temporary OIDC token
3. AWS verifies the token
4. AWS gives temporary permissions
5. Workflow accesses AWS safely

```
GitHub Actions → OIDC Token → AWS IAM Role → AWS Services
```

---

## Prerequisites

* GitHub account
* AWS account
* Basic GitHub Actions knowledge
* Permission to create IAM roles in AWS

---

## Step 1: Create GitHub Repository

1. Login to GitHub
2. Create a new repository
3. Example name:

   ```
   github-aws-oidc
   ```
4. Initialize with a README file

---

## Step 2: Create OIDC Provider in AWS

1. Login to **AWS Console**
2. Go to **IAM → Identity Providers**
3. Click **Add provider**
4. Select:

   * Provider type: **OpenID Connect**
   * Provider URL:

     ```
     https://token.actions.githubusercontent.com
     ```
   * Audience:

     ```
     sts.amazonaws.com
     ```
5. Click **Add provider**

---

## Step 3: Create IAM Role in AWS

1. Go to **Assign Roles → Create role**
2. Select **Web identity**
3. Choose:

   * Identity provider: `token.actions.githubusercontent.com`
   * Audience: `sts.amazonaws.com`
4. Attach policy

   * (For learning): `AdministratorAccess`
5. Role name example:

   ```
   github-oidc-role
   ```
6. Create the role

 Copy the **Role ARN** (you’ll need it later)

---

## Step 4: Create GitHub Actions Workflow

1. In your repository, create this file:

   ```
   .github/workflows/aws-oidc.yml
   ```
2. Paste the following code 

---

## Step 5: GitHub Actions Workflow (`aws-oidc.yml`)

```yaml
name: AWS OIDC Test

on:
  push:
    branches: [ "main" ]

jobs:
  aws-oidc-job:
    runs-on: ubuntu-latest

    permissions:
      id-token: write
      contents: read

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials using OIDC
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::ACCOUNT_ID:role/github-oidc-role
          aws-region: us-east-1

      - name: Verify AWS Identity
        run: aws sts get-caller-identity
```

### Replace:

* `ACCOUNT_ID` → your AWS account ID
* `github-oidc-role` → your IAM role name

---

## Step 6: Push Code to GitHub

```bash
git add .
git commit -m "Add AWS OIDC workflow"
git push origin main
```

---

## Step 7: Run & Verify Workflow

1. Go to **GitHub → Repository → Actions**
2. Click the workflow run
3. Open the logs

If successful, you will see:

```json
{
  "Account": "123456789012",
  "Arn": "arn:aws:sts::123456789012:assumed-role/github-oidc-role/GitHubActions"
}
```

**Success!**

* GitHub connected to AWS
* No access keys used
* Secure authentication completed

---

## Common Beginner Mistakes

1. Wrong IAM Role ARN
2. Missing `id-token: write` permission
3. OIDC provider not created in AWS

1. Double-check role ARN
2. Ensure permissions are correct
3. Check workflow logs

---
