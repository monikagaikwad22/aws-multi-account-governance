# Multi-Account AWS Governance Setup using AWS Organizations & SCP

<img width="1536" height="1024" alt="ChatGPT Image Mar 16, 2026, 09_46_53 PM" src="https://github.com/user-attachments/assets/7cbf9f8f-3b61-43ec-9cb2-60e7af54485e" />


## Project Overview

This project demonstrates how to implement **centralized governance across multiple AWS accounts** using AWS Organizations and Service Control Policies (SCPs).

The objective is to enforce **security, compliance, and cost control policies** across Development, Testing, and Production environments.

---

# Architecture

```
AWS Organization (Root)
│
├── Dev-OU
│     └── dev-account
│
├── Test-OU
│     └── test-account
│
└── Prod-OU
      └── prod-account
```

Each environment is separated using **Organizational Units (OUs)** to maintain security and governance.

---

# Technologies Used

* AWS Organizations
* Service Control Policies (SCP)
* IAM
* Amazon EC2
* AWS CloudTrail

---

# Implementation Steps

## 1. Create AWS Organization

1. Login to AWS Management Account
2. Open **AWS Organizations**
3. Enable **All Features**

---

## 2. Create Organizational Units

Create the following OUs:

* Dev-OU
* Test-OU
* Prod-OU

   ![WhatsApp Image 2026-03-15 at 4 01 54 PM](https://github.com/user-attachments/assets/4701e452-1b3c-4149-9cb1-52c28cd98f38)

---

## 3. Create Member Accounts

| Environment | Account Name |
| ----------- | ------------ |
| Development | dev-account  |
| Testing     | test-account |
| Production  | prod-account |

• Move accounts DEV-OU into dev-account Organizational Units
• Move accounts TEST-OU into test-account Organizational Units
• Move accounts PROD-OU into prod-account Organizational Units

---

# Service Control Policies (SCP)

## 1. Deny EC2 Instance Creation in Dev Account

This policy prevents developers from launching EC2 instances in the development environment to control infrastructure cost.

![WhatsApp Image 2026-03-15 at 4 02 40 PM](https://github.com/user-attachments/assets/e447032f-9cc6-459f-9feb-a57102fcc697)


### Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyEC2RunInstances",
      "Effect": "Deny",
      "Action": [
        "ec2:RunInstances",
        "ec2:CreateKeyPair"
      ],
      "Resource": "*"
    }
  ]
}
```

Attach this policy to:

```
Dev-OU
```

---

# 2. Prevent Disabling CloudTrail

This policy ensures security logs cannot be disabled.

<img width="1920" height="1008" alt="image" src="https://github.com/user-attachments/assets/88540f0d-e390-4d4e-9968-85e6e37b419e" />

### Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyCloudTrailDisable",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail"
      ],
      "Resource": "*"
    }
  ]
}
```

Attach this policy to:

```
Root
```

---

# 3. Restrict AWS Regions

This policy restricts resource creation to an approved region.

![WhatsApp Image 2026-03-15 at 4 02 40 PM](https://github.com/user-attachments/assets/aeed7ca1-689b-4b4f-95f6-c57ab65c4775)

Approved Region Example:

```
ap-south-1
```

### Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnapprovedRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": "ap-south-1"
        }
      }
    }
  ]
}
```

Attach this policy to:

```
Root
```

---
# **Test 1 – EC2 Restriction**
# 1. Open Swich role
# 2. Login to the Dev Account.

<img width="1920" height="1008" alt="Amazon Web Services Switch Role - Google Chrome 3_15_2026 10_23_47 PM" src="https://github.com/user-attachments/assets/198b3012-0ac5-4eec-b00e-f1ff3d8d413a" />


# Validation

Policy enforcement was tested in the **dev-account**.

When attempting to perform restricted actions such as launching EC2 instances, AWS returned the following error:

![WhatsApp Image 2026-03-15 at 10 34 33 PM](https://github.com/user-attachments/assets/638054df-45eb-4b5d-830e-6befa2afde39)

```
Access Denied
explicit deny in service control policy
```
# **Test 2 – CloudTrail Protection**

<img width="1920" height="1008" alt="Instances _ EC2 _ eu-north-1 - Google Chrome 3_16_2026 10_24_33 PM" src="https://github.com/user-attachments/assets/3e4a6940-66b2-4148-b061-f1681ac543bb" />

This confirms that the governance policies are successfully enforced.

---


# Risk Mitigation Strategy

| Risk                                      | Governance Solution       |
| ----------------------------------------- | ------------------------- |
| Developers launching expensive resources  | EC2 restriction policy    |
| Security logs being disabled              | CloudTrail protection     |
| Resources created in unauthorized regions | Region restriction policy |

---



---

# Project Folder Structure

```
aws-governance-project
│
├── README.md
├── architecture-diagram.png
│
└── policies
     ├── deny-ec2.json
     ├── cloudtrail-protection.json
     └── region-restriction.json
```

---

# Author

Monika Sharad Gaikwad
AWS Cloud & DevOps Enthusiast
