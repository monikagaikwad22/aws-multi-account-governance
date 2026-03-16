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

---

## 3. Create Member Accounts

| Environment | Account Name |
| ----------- | ------------ |
| Development | dev-account  |
| Testing     | test-account |
| Production  | prod-account |

Move accounts into their respective Organizational Units.

---

# Service Control Policies (SCP)

## 1. Deny EC2 Instance Creation in Dev Account

This policy prevents developers from launching EC2 instances in the development environment to control infrastructure cost.

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

# Validation

Policy enforcement was tested in the **dev-account**.

When attempting to perform restricted actions such as launching EC2 instances, AWS returned the following error:

```
Access Denied
explicit deny in service control policy
```

This confirms that the governance policies are successfully enforced.

---

# Risk Mitigation Strategy

| Risk                                      | Governance Solution       |
| ----------------------------------------- | ------------------------- |
| Developers launching expensive resources  | EC2 restriction policy    |
| Security logs being disabled              | CloudTrail protection     |
| Resources created in unauthorized regions | Region restriction policy |

---

# Governance Benefits

* Centralized governance across AWS accounts
* Improved security enforcement
* Better cost management
* Environment isolation
* Strong compliance controls

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
