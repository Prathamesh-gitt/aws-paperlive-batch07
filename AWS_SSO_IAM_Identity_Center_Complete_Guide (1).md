# AWS SSO (IAM Identity Center) – Complete Teaching Guide
### Learn With KASTRO | Region: ap-south-1 (Mumbai)

---

## 📚 Table of Contents

1. [What is AWS SSO? – The Problem It Solves](#1-what-is-aws-sso--the-problem-it-solves)
2. [AWS SSO vs IAM Users – Key Differences](#2-aws-sso-vs-iam-users--key-differences)
3. [Core Concepts You Must Know](#3-core-concepts-you-must-know)
4. [Real-Time Scenarios – When SSO is Used](#4-real-time-scenarios--when-sso-is-used)
5. [How IAM Users Work – The Old Way](#5-how-iam-users-work--the-old-way)
6. [How SSO Users Work – The New Way](#6-how-sso-users-work--the-new-way)
7. [Architecture – How SSO Works Under the Hood](#7-architecture--how-sso-works-under-the-hood)
8. [LAB 1 – Enable IAM Identity Center (SSO)](#8-lab-1--enable-iam-identity-center-sso)
9. [LAB 2 – Create Users and Groups in SSO](#9-lab-2--create-users-and-groups-in-sso)
10. [LAB 3 – Create Permission Sets (What Users Can Do)](#10-lab-3--create-permission-sets-what-users-can-do)
11. [LAB 4 – Assign Users to AWS Accounts](#11-lab-4--assign-users-to-aws-accounts)
12. [LAB 5 – Login as SSO User and Verify Access](#12-lab-5--login-as-sso-user-and-verify-access)
13. [LAB 6 – Multi-Account Setup (SSO Across AWS Accounts)](#13-lab-6--multi-account-setup-sso-across-aws-accounts)
14. [LAB 7 – Connect External Identity Provider (Google Workspace)](#14-lab-7--connect-external-identity-provider-google-workspace)
15. [SSO for IAM Users – Does SSO Help IAM Users?](#15-sso-for-iam-users--does-sso-help-iam-users)
16. [Real-World Company Setup Example](#16-real-world-company-setup-example)
17. [Troubleshooting Common Issues](#17-troubleshooting-common-issues)
18. [SSO vs IAM Users – When to Use What](#18-sso-vs-iam-users--when-to-use-what)
19. [Quick Reference Cheatsheet](#19-quick-reference-cheatsheet)

---

## 1. What is AWS SSO? – The Problem It Solves

### The Problem (Without SSO)

Imagine you join a company called **KastroTech** as a DevOps engineer. The company has:
- 5 AWS accounts (Dev, QA, Staging, Production, Shared Services)
- 50 engineers in the team

**Without SSO**, this is what happens:

```
WITHOUT SSO – The Painful Reality
───────────────────────────────────────────────────────────────────

Admin creates 50 IAM users × 5 accounts = 250 IAM user accounts to manage

Each engineer gets:
  - 5 different usernames
  - 5 different passwords
  - 5 different AWS Console URLs
  - 5 different MFA devices (or reused, which is a security risk)

When an engineer leaves the company:
  Admin must delete IAM user in ALL 5 accounts = 5 manual deletions
  One missed deletion = Security vulnerability (ex-employee still has access!)

When engineer forgets password:
  Admin must reset in each account separately

When you need to audit who did what:
  Check CloudTrail in 5 separate accounts
  5 separate IAM user IDs to correlate

Result: Chaos, security risk, operational nightmare
```

### The Solution (With SSO)

```
WITH SSO – One Identity, Everything Accessible
───────────────────────────────────────────────────────────────────

Admin creates 1 SSO user per engineer = 50 users total (not 250!)

Each engineer gets:
  - 1 username (e.g., kastro@kastrotech.com)
  - 1 password (or Google/Microsoft login)
  - 1 SSO portal URL (e.g., kastrotech.awsapps.com/start)
  - 1 MFA device

From the SSO portal, they see ALL accounts they have access to:
  ┌─────────────────────────────────┐
  │  kastrotech.awsapps.com/start  │
  │                                 │
  │  👤 Kastro (kastro@kastro.com) │
  │                                 │
  │  AWS Accounts:                  │
  │  ▸ Dev Account     [Access]     │
  │  ▸ QA Account      [Access]     │
  │  ▸ Staging Account [Access]     │
  │  ▸ Production      [Read Only]  │
  └─────────────────────────────────┘

When engineer leaves:
  Admin disables 1 SSO user → Access to ALL 5 accounts revoked instantly

When engineer forgets password:
  Reset in 1 place → Works everywhere

Audit:
  All activity under one identity across all accounts
```

### What is "Single Sign-On"?

**Single Sign-On (SSO)** means: **Login once, access everything you are authorized for.**

Just like how you sign into Google once and can use Gmail, Google Drive, Google Meet, YouTube — all without logging in separately to each one. AWS SSO does the same for AWS accounts and applications.

---

## 2. AWS SSO vs IAM Users – Key Differences

| Feature | IAM Users | AWS SSO Users |
|---|---|---|
| **Where managed** | Inside each AWS account | Centralized in IAM Identity Center |
| **Login URL** | `console.aws.amazon.com` (per account) | `your-org.awsapps.com/start` (one URL) |
| **Credentials** | Username + Password per account | Username + Password (one set) |
| **Multi-account** | Separate user in each account | One user, access multiple accounts |
| **Password reset** | Per account | Centralized (one reset) |
| **MFA** | Per account setup | Once, applies everywhere |
| **Access keys** | Long-lived (security risk) | Temporary STS credentials (secure) |
| **Offboarding** | Delete in every account | Disable one SSO user |
| **External IdP** | ❌ Not natively | ✅ Google, Microsoft AD, Okta |
| **Audit trail** | Per account CloudTrail | Unified identity in CloudTrail |
| **Best for** | Legacy, single account, automation | Multi-account, teams, enterprises |

---

## 3. Core Concepts You Must Know

### 3.1 IAM Identity Center (Formerly AWS SSO)

AWS renamed "AWS SSO" to **IAM Identity Center** in 2022. They are the same service. You may hear both names — they refer to the same thing.

```
Old Name:  AWS SSO (Single Sign-On)
New Name:  AWS IAM Identity Center
Console:   "IAM Identity Center" in AWS Console search
```

### 3.2 Identity Source

Where user accounts are stored and authenticated:

```
Option 1: IAM Identity Center Built-in Directory
├── You create users manually inside IAM Identity Center
├── AWS manages the passwords
└── Best for: Small teams, no existing directory

Option 2: Active Directory (AD) – Self-managed or AWS Managed
├── Connect your company's Windows Active Directory
├── Users log in with their Windows domain credentials
└── Best for: Companies already using AD

Option 3: External Identity Provider (IdP)
├── Connect Google Workspace, Okta, Azure AD, OneLogin, etc.
├── Users log in with their Google/Okta/Microsoft accounts
└── Best for: Companies using Google Workspace or Okta
```

### 3.3 AWS Organization

AWS Organizations is the service that lets you manage **multiple AWS accounts** under one umbrella. SSO works with AWS Organizations to give your users access to any account in your organization.

```
AWS Organization (KastroTech)
├── Management Account (Root/Payer account)
│   └── All billing consolidated here
│
├── Dev Account (123456789001)
├── QA Account (123456789002)
├── Staging Account (123456789003)
└── Production Account (123456789004)

IAM Identity Center is enabled in the Management Account
and can control access to all member accounts.
```

### 3.4 Permission Sets

A Permission Set is like an **IAM Role template** that defines what a user can DO in an AWS account. It is created once in SSO and reused across all accounts.

```
Permission Set Examples:
┌─────────────────────────────────────────────────────────────┐
│  Permission Set Name    │  What it allows                   │
├─────────────────────────┼───────────────────────────────────│
│  AdministratorAccess    │  Full access to all AWS services   │
│  ReadOnly               │  Can view, cannot modify anything  │
│  S3FullAccess           │  Only S3 — nothing else            │
│  DevOpsEngineer         │  EC2, ECS, EKS, CodePipeline, ECR │
│  DataEngineer           │  Glue, Athena, S3, Redshift, EMR  │
└─────────────────────────┴───────────────────────────────────┘

You assign: User + Account + Permission Set = Access
```

### 3.5 SSO Portal URL

Every organization gets a unique SSO portal URL:
```
Format:  https://<your-alias>.awsapps.com/start
Example: https://kastrotech.awsapps.com/start
```

All users in your organization bookmark this ONE URL to access all their AWS accounts.

### 3.6 Temporary Credentials (How SSO Credentials Work)

When an SSO user logs in and clicks on an account, AWS:
1. Checks their permissions
2. Creates a **temporary IAM role** in the target account
3. Issues **temporary STS credentials** (Access Key + Secret Key + Session Token)
4. These expire in 1–12 hours (configurable in Permission Set)

```
SSO Login Flow:
User logs in → SSO validates identity → User selects account
→ AWS creates temp credentials via STS → User gets console/CLI access
→ Credentials expire after session duration → Auto-renewed on next login

NO long-lived access keys = much more secure than IAM users
```

---

## 4. Real-Time Scenarios – When SSO is Used

### Scenario 1 – IT Company with Multiple AWS Accounts

```
Company: KastroTech (200 employees, 8 AWS accounts)

Without SSO:
  200 employees × 8 accounts = 1,600 IAM users to manage
  HR offboarding: Admin manually deletes 8 accounts per person

With SSO (connected to Google Workspace):
  200 SSO users (mapped from Google accounts)
  Employees log in with their company Google account
  HR disables Google account → All AWS access revoked instantly
```

### Scenario 2 – Consultant/Freelancer Accessing Client Accounts

```
You (Kastro) are a consultant working with 3 client AWS accounts.

Without SSO:
  Client 1 creates IAM user for you → Username, Password, MFA
  Client 2 creates IAM user for you → Different Username, Password, MFA
  Client 3 creates IAM user for you → Different Username, Password, MFA
  You manage 3 separate accounts, 3 passwords, 3 MFA devices

With SSO:
  Each client adds your email as a guest in their SSO
  You get one SSO portal per client
  (True SSO: They share one portal where you appear as a federated user)
```

### Scenario 3 – Dev Team with Different Access Levels

```
Team: 10 developers, 2 DevOps, 1 DB admin

Without SSO:
  Create custom IAM policies in every account × every person

With SSO:
  Create 3 Permission Sets:
    - Developer: EC2 read, S3 write, CodeCommit full
    - DevOps: EC2 full, ECS full, CodePipeline full, CloudFormation full
    - DBA: RDS full, CloudWatch, Secrets Manager

  Assign:
    - 10 developers → Developer permission set in Dev + QA accounts
    - 2 DevOps → DevOps permission set in all accounts
    - 1 DBA → DBA permission set in Production only
```

### Scenario 4 – New Employee Onboarding

```
New joiner: Priya (Cloud Engineer)

Admin actions (2 minutes):
  1. Create SSO user: priya@kastrotech.com
  2. Add to group "CloudEngineers"
  3. (Done – group already has permissions assigned)

Priya gets:
  - Welcome email with SSO portal URL
  - One-time password link
  - Sets up MFA
  - Logs in → sees Dev, QA, Staging accounts with correct permissions
  - No manual IAM user creation in any account
```

---

## 5. How IAM Users Work – The Old Way

### The IAM User Login Flow

```
Step 1: Admin creates IAM user in AWS Account (e.g., Dev account)
Step 2: Admin sets username, password, assigns policies
Step 3: User gets account ID or alias + username + password
Step 4: User visits: https://123456789012.signin.aws.amazon.com/console
         OR: https://kastro-dev.signin.aws.amazon.com/console (if alias set)
Step 5: User enters username + password → Logged in to ONLY that account
Step 6: To access another account → Completely separate login
```

### Problems with IAM Users in Multi-Account Setup

```
Problem 1: Credential sprawl
  User has 5 different passwords → writes them in a notebook (security risk!)

Problem 2: Access key management
  Each IAM user can have up to 2 access keys (for CLI)
  Long-lived keys → If leaked, attacker has permanent access
  Must manually rotate every 90 days × all users × all accounts

Problem 3: Offboarding
  Employee leaves on Friday evening
  Admin must remember to delete IAM users in all 8 accounts
  If they forget even one → That ex-employee still has access on Monday

Problem 4: No unified audit
  CloudTrail logs show "IAM user: kastro-dev-account" in Dev account
  And "IAM user: kastro-prod-account" in Prod account
  These look like different people — hard to correlate

Problem 5: MFA fatigue
  5 accounts × 1 MFA = 5 MFA setups, or reuse same MFA across accounts
  AWS discourages MFA reuse across accounts
```

---

## 6. How SSO Users Work – The New Way

### The SSO Login Flow (Step by Step)

```
Step 1: User visits SSO portal: https://kastrotech.awsapps.com/start
Step 2: Enters email + password (or clicks "Sign in with Google")
Step 3: Completes MFA (once)
Step 4: Sees the AWS Access Portal — a dashboard of all accessible accounts

┌──────────────────────────────────────────────────────────────────┐
│            AWS Access Portal – kastrotech.awsapps.com            │
├──────────────────────────────────────────────────────────────────┤
│  Hello, Kastro!                                                  │
│                                                                  │
│  AWS Accounts                                                    │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │ KastroTech Dev Account (123456789001)                     │   │
│  │ Permission: DevOpsEngineer                                │   │
│  │ [Management Console]  [Command line or programmatic]      │   │
│  ├───────────────────────────────────────────────────────────┤   │
│  │ KastroTech QA Account (123456789002)                      │   │
│  │ Permission: DevOpsEngineer                                │   │
│  │ [Management Console]  [Command line or programmatic]      │   │
│  ├───────────────────────────────────────────────────────────┤   │
│  │ KastroTech Production (123456789004)                      │   │
│  │ Permission: ReadOnlyAccess                                │   │
│  │ [Management Console]  [Command line or programmatic]      │   │
│  └───────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘

Step 5: User clicks "Management Console" for Dev Account
Step 6: AWS issues temporary credentials → User is logged into Dev account
Step 7: Session expires after configured time (e.g., 8 hours)
Step 8: User refreshes SSO portal → Re-authenticates → New temp credentials
```

---

## 7. Architecture – How SSO Works Under the Hood

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        AWS Organization                                  │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │              Management Account (Root)                          │   │
│   │                                                                 │   │
│   │   ┌───────────────────────┐    ┌──────────────────────────┐    │   │
│   │   │  IAM Identity Center  │    │   AWS Organizations      │    │   │
│   │   │  (SSO Service)        │◄───│   (Account Management)   │    │   │
│   │   │                       │    │                          │    │   │
│   │   │  • Users & Groups     │    │   • Dev Account          │    │   │
│   │   │  • Permission Sets    │    │   • QA Account           │    │   │
│   │   │  • Account Assignments│    │   • Production Account   │    │   │
│   │   └───────────┬───────────┘    └──────────────────────────┘    │   │
│   │               │                                                 │   │
│   └───────────────┼─────────────────────────────────────────────────┘   │
│                   │                                                     │
│   Identity        │  SAML 2.0 / OIDC                                   │
│   Provider        │  (Authentication Protocol)                          │
│                   │                                                     │
│   ┌───────────────▼─────────────────────────────────────────────────┐   │
│   │                    SSO Portal                                   │   │
│   │            kastrotech.awsapps.com/start                        │   │
│   └───────────────┬─────────────────────────────────────────────────┘   │
│                   │                                                     │
│         User logs in and selects account                                │
│                   │                                                     │
│   ┌───────────────▼──────────────────┐                                  │
│   │     AWS STS (Security Token      │                                  │
│   │     Service)                     │                                  │
│   │     Issues Temporary Credentials │                                  │
│   │     (Valid 1–12 hours)           │                                  │
│   └───────────────┬──────────────────┘                                  │
│                   │                                                     │
│        ┌──────────┼──────────┐                                          │
│        ▼          ▼          ▼                                          │
│   Dev Account  QA Account  Prod Account                                 │
│   (IAM Role    (IAM Role   (IAM Role                                    │
│    assumed)    assumed)     assumed)                                     │
└─────────────────────────────────────────────────────────────────────────┘
```

### What Happens in Each Member Account?

When SSO assigns a user to an account with a permission set, AWS **automatically creates an IAM Role** in that account called:

```
Role Name: AWSReservedSSO_<PermissionSetName>_<RandomSuffix>
Example:   AWSReservedSSO_AdministratorAccess_a1b2c3d4e5f6

You do NOT create this role manually.
SSO creates and manages it automatically.
The user assumes this role when they access that account.
```

---

## 8. LAB 1 – Enable IAM Identity Center (SSO)

### Prerequisites

- You must have an **AWS Account** (the Management account — ideally within AWS Organizations)
- You need **Administrator** access to the management account

> 📌 If you have a single AWS account (not in an organization), you can still enable IAM Identity Center — it works for single accounts too. AWS Organizations is required only for multi-account setup.

---

### STEP 1 – Open IAM Identity Center

1. Log into AWS Console as root user or an IAM user with admin privileges
2. In the search bar at the top, type: **`IAM Identity Center`**
3. Click on **IAM Identity Center** from the results
4. You will land on the IAM Identity Center homepage

---

### STEP 2 – Enable IAM Identity Center

1. Click the orange button: **Enable**
2. A dialog appears with two options:
   ```
   Option A: Enable with AWS Organizations (Recommended)
   Option B: Enable for this account only
   ```
3. For multi-account setup: Select **"Enable with AWS Organizations"**
   - If prompted to create an organization, click **Create AWS Organization** → AWS creates it automatically
4. For single-account learning: Select **"Enable for this account only"**
5. Click **Enable**
6. Wait a few seconds — IAM Identity Center is now enabled

---

### STEP 3 – Choose Identity Source

1. You land on the IAM Identity Center dashboard
2. In the left sidebar, click **Settings**
3. Under **Identity source** tab, you see the current source: **"IAM Identity Center directory"** (default)
4. This means users are stored inside IAM Identity Center itself — you create them manually
5. Click **Actions** → **Change identity source** if you want to change to Google/AD/Okta (we cover this in LAB 7)
6. For now, keep the default **"IAM Identity Center directory"**

---

### STEP 4 – Configure the SSO Portal URL

1. In **Settings** → **Identity source** tab
2. Find **AWS access portal URL**
3. It shows a default URL like: `https://d-xxxxxxxxxx.awsapps.com/start`
4. Click **Edit** to customize it:
   - **Custom alias**: `kastrotech` (or your company name)
   - New URL becomes: `https://kastrotech.awsapps.com/start`
5. Click **Save**

> 📌 The alias must be unique globally across all AWS customers. If `kastrotech` is taken, try `kastrotech-lab` or `learnwithkastro`.

---

### STEP 5 – Verify IAM Identity Center is Active

In the IAM Identity Center dashboard, verify:
```
Status:           Enabled ✅
Identity source:  IAM Identity Center directory
AWS access portal: https://kastrotech.awsapps.com/start
Region:           ap-south-1 (Mumbai) — SSO is a global service but has a home region
```

> ⚠️ IAM Identity Center must be enabled in ONE region — it cannot be changed after. Choose your primary region carefully. For this lab: **ap-south-1**.

---

## 9. LAB 2 – Create Users and Groups in SSO

### STEP 1 – Create a Group First

Groups make it easy to manage access for many users. Instead of assigning permissions to each user, assign to a group.

1. IAM Identity Center → Left sidebar → **Groups** → **Create Group**
2. Fill in:
   - **Group name**: `DevOpsEngineers`
   - **Description**: `DevOps team - has access to Dev and QA accounts`
3. Click **Create Group**

Create two more groups:
- **Group name**: `Developers` | Description: `App developers - Dev account access only`
- **Group name**: `ReadOnlyTeam` | Description: `Managers and auditors - read only access`

---

### STEP 2 – Create SSO Users

#### Create User 1 (DevOps Engineer):

1. IAM Identity Center → Left sidebar → **Users** → **Add User**
2. Fill in:

   | Field | Value |
   |---|---|
   | **Username** | `kastro.devops` |
   | **Email address** | `kastro.devops@yourdomain.com` |
   | **First name** | `Kastro` |
   | **Last name** | `DevOps` |
   | **Display name** | `Kastro DevOps` |

3. **Send an email invitation**: ✅ Enable (user gets a welcome email with login instructions)
4. Click **Next**
5. **Add user to groups**: Select `DevOpsEngineers`
6. Click **Next** → Review → **Add User**

#### Create User 2 (Developer):

1. **Users** → **Add User**
2. Fill in:
   - **Username**: `priya.developer`
   - **Email**: `priya.developer@yourdomain.com`
   - **First name**: `Priya`, **Last name**: `Developer`
3. Add to group: `Developers`
4. Click **Add User**

#### Create User 3 (Read-Only – Manager):

1. **Users** → **Add User**
2. Fill in:
   - **Username**: `manager.readonly`
   - **Email**: `manager@yourdomain.com`
   - **First name**: `Manager`, **Last name**: `ReadOnly`
3. Add to group: `ReadOnlyTeam`
4. Click **Add User**

---

### STEP 3 – Verify Users Were Created

1. IAM Identity Center → **Users**
2. You should see all 3 users listed
3. Status: **Active** (if email confirmed) or **Pending** (if invitation not yet accepted)
4. Click on any user to see their details:
   - **Groups**: Shows which groups they belong to
   - **AWS accounts**: Shows which accounts they can access (empty for now — we add this next)
   - **MFA devices**: Shows any registered MFA devices

---

## 10. LAB 3 – Create Permission Sets (What Users Can Do)

### What is a Permission Set?

A Permission Set = the set of IAM policies that define what an SSO user can do when they access an AWS account. Think of it as a job role template.

---

### STEP 1 – Create "AdministratorAccess" Permission Set

1. IAM Identity Center → **Permission Sets** → **Create Permission Set**
2. **Permission set type**: Select **"Predefined permission set"**
3. **Policy for predefined permission set**: Select `AdministratorAccess`
4. Click **Next**
5. **Permission set details**:
   - **Name**: `AdministratorAccess` (auto-filled)
   - **Description**: `Full administrator access to all AWS services`
   - **Session duration**: `8 hours` (how long before the session expires)
   - **Relay state**: Leave blank (used for deep-linking to a specific AWS service)
6. Click **Next** → **Create**

---

### STEP 2 – Create "DevOpsEngineer" Custom Permission Set

1. **Permission Sets** → **Create Permission Set**
2. **Permission set type**: Select **"Custom permission set"**
3. Click **Next**
4. **Add policies**:
   - Click **AWS managed policies** tab
   - Search and select these policies one by one:
     - `AmazonEC2FullAccess`
     - `AmazonECS_FullAccess`
     - `AmazonS3FullAccess`
     - `CloudWatchFullAccess`
     - `AWSCodePipelineFullAccess`
     - `IAMReadOnlyAccess`
5. Click **Next**
6. **Permission set details**:
   - **Name**: `DevOpsEngineer`
   - **Description**: `DevOps team - EC2, ECS, S3, CodePipeline, CloudWatch`
   - **Session duration**: `8 hours`
7. Click **Next** → **Create**

---

### STEP 3 – Create "ReadOnlyAccess" Permission Set

1. **Permission Sets** → **Create Permission Set**
2. **Type**: Predefined
3. **Policy**: Select `ReadOnlyAccess`
4. **Name**: `ReadOnlyAccess`
5. **Description**: `Read-only access - for managers and auditors`
6. **Session duration**: `4 hours`
7. Click **Next** → **Create**

---

### STEP 4 – Create "S3AndEC2Only" Developer Permission Set

1. **Permission Sets** → **Create Permission Set**
2. **Type**: Custom permission set
3. **AWS managed policies**: Add `AmazonEC2ReadOnlyAccess` + `AmazonS3FullAccess` + `AWSCodeCommitFullAccess`
4. **Name**: `DeveloperAccess`
5. **Description**: `Developer team - S3, EC2 read, CodeCommit`
6. **Session duration**: `8 hours`
7. Click **Next** → **Create**

---

### STEP 5 – Verify All Permission Sets

1. IAM Identity Center → **Permission Sets**
2. You should see all 4 permission sets listed:
   ```
   AdministratorAccess    – AWS managed policy
   DevOpsEngineer         – Custom
   ReadOnlyAccess         – AWS managed policy
   DeveloperAccess        – Custom
   ```

---

## 11. LAB 4 – Assign Users to AWS Accounts

This is where you connect: **User/Group + AWS Account + Permission Set = Access**

---

### STEP 1 – Go to AWS Accounts in SSO

1. IAM Identity Center → Left sidebar → **AWS Accounts**
2. You see a list of accounts in your organization
3. For single-account labs, you see only your current account

---

### STEP 2 – Assign DevOpsEngineers Group to Dev Account

1. Click on your **AWS account name** (e.g., `KastroTech-Dev` or your account name)
2. Click **Assign users or groups**
3. Select **Groups** tab
4. Select `DevOpsEngineers` group → Click **Next**
5. Select permission set: `DevOpsEngineer`
6. Click **Next** → **Submit**
7. Wait a moment — AWS creates the IAM role in the target account automatically

---

### STEP 3 – Assign ReadOnlyTeam Group to Production Account

1. Click on your **Production account** (if you have multiple accounts in org)
   - For single-account lab: Same account, different assignment
2. Click **Assign users or groups**
3. Select **Groups** tab → Select `ReadOnlyTeam` → Click **Next**
4. Select permission set: `ReadOnlyAccess` → Click **Next** → **Submit**

---

### STEP 4 – Assign Individual User (Manager) with Admin Access

1. Click on your account
2. Click **Assign users or groups**
3. Select **Users** tab → Select `kastro.devops` → Click **Next**
4. Select permission set: `AdministratorAccess` → Click **Next** → **Submit**

---

### STEP 5 – Verify Assignments

1. IAM Identity Center → **AWS Accounts** → Click on your account
2. Click the **Users and Groups** tab
3. You should see:
   ```
   DevOpsEngineers (group)  → DevOpsEngineer permission set
   ReadOnlyTeam (group)     → ReadOnlyAccess permission set
   kastro.devops (user)     → AdministratorAccess permission set
   ```

---

### STEP 6 – Verify IAM Role Was Created in Target Account

This is a great thing to show students — SSO automatically creates IAM roles:

1. Go to **IAM Console** (regular IAM, not IAM Identity Center)
2. Click **Roles** in the left sidebar
3. Search for: `AWSReservedSSO`
4. You will see auto-created roles like:
   ```
   AWSReservedSSO_AdministratorAccess_a1b2c3d4e5f6
   AWSReservedSSO_DevOpsEngineer_b2c3d4e5f6a7
   AWSReservedSSO_ReadOnlyAccess_c3d4e5f6a7b8
   ```
5. Click on one of these roles → See the **Trust policy**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Federated": "arn:aws:iam::123456789012:saml-provider/AWSSSO_..."
         },
         "Action": "sts:AssumeRoleWithSAML",
         "Condition": {
           "StringEquals": {
             "SAML:aud": "https://signin.aws.amazon.com/saml"
           }
         }
       }
     ]
   }
   ```
   > This trust policy allows SSO to assume this role on behalf of authenticated users.

---

## 12. LAB 5 – Login as SSO User and Verify Access

### STEP 1 – Open the SSO Portal in a New Browser (Incognito Mode)

> Use **Incognito/Private browsing** so your admin session does not interfere.

1. Open a new Incognito window
2. Navigate to: `https://kastrotech.awsapps.com/start`
   (Replace `kastrotech` with your actual alias)

---

### STEP 2 – Accept the Invitation (First-Time User Setup)

If the SSO user received an invitation email:
1. Open the invitation email (to `kastro.devops@yourdomain.com`)
2. Click **Accept Invitation**
3. Set a new password (must meet complexity requirements)
4. Click **Set new password**

If testing with an email you control:
1. Check the inbox for the invitation email from `no-reply@login.awsapps.com`
2. Subject: "Invitation to join the AWS IAM Identity Center"
3. Click **Accept invitation** → Set password

---

### STEP 3 – Set Up MFA

1. After setting password, you are prompted to register MFA
2. Choose MFA type:
   - **Authenticator app** (recommended): Use Google Authenticator or Microsoft Authenticator
   - **Security key**: Physical hardware key (YubiKey)
   - **Built-in authenticators**: Fingerprint/Face ID (on supported devices)
3. For **Authenticator app**:
   - Click **Authenticator app** → **Show QR code**
   - Open Google Authenticator on your phone → Tap + → Scan QR code
   - Enter the 6-digit OTP from the app → Click **Assign MFA**

---

### STEP 4 – Explore the SSO Access Portal

After login, you land on the **AWS Access Portal**:

```
URL: https://kastrotech.awsapps.com/start

You should see:
┌─────────────────────────────────────────────────────────────────┐
│                AWS Access Portal                                │
│  👤 kastro.devops                                              │
│                                                                │
│  AWS Accounts (1)                                              │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ KastroTech Account (123456789012)                        │  │
│  │                                                          │  │
│  │ DevOpsEngineer                                           │  │
│  │ [Management Console ↗]  [Command line or programmatic ↗] │  │
│  │                                                          │  │
│  │ AdministratorAccess (if you assigned this too)           │  │
│  │ [Management Console ↗]  [Command line or programmatic ↗] │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

### STEP 5 – Access AWS Console as SSO User

1. Click **"Management Console"** next to `DevOpsEngineer`
2. You are logged into the AWS Console
3. Notice the top-right corner — it does NOT show an IAM username
4. Instead it shows:
   ```
   kastro.devops @ kastrotech
   (Role: AWSReservedSSO_DevOpsEngineer_xxxxx)
   ```

---

### STEP 6 – Verify the Permission Set is Working

Test that DevOpsEngineer can do allowed things and is blocked from restricted things:

#### Test 1 – Can access EC2 (Allowed):
1. Go to **EC2 Console** → **Instances**
2. Try to launch an instance
3. ✅ Should work — EC2 access is allowed in DevOpsEngineer permission set

#### Test 2 – Cannot access IAM (Restricted):
1. Go to **IAM Console**
2. Try to create an IAM user
3. ❌ You will see: `You are not authorized to perform this action`
4. Because `IAMReadOnlyAccess` only allows reading IAM, not modifying

#### Test 3 – Cannot access RDS (Not in permission set):
1. Go to **RDS Console**
2. Try to create a database
3. ❌ Access Denied — RDS is not in the DevOpsEngineer permission set

---

### STEP 7 – Get CLI Credentials from SSO Portal

The SSO portal also provides CLI credentials (temporary):

1. Go back to SSO portal: `https://kastrotech.awsapps.com/start`
2. Click **"Command line or programmatic"** next to the account
3. You see three options:
   ```
   Option 1: AWS IAM Identity Center credentials (Recommended)
             → Configure AWS CLI with SSO profile

   Option 2: Environment variables
             → Copy-paste temporary env vars into your terminal

   Option 3: AWS credentials file
             → Copy-paste to ~/.aws/credentials file
   ```

4. For **Environment variables** (quick testing), copy these into your terminal:
   ```bash
   export AWS_ACCESS_KEY_ID="ASIA..."
   export AWS_SECRET_ACCESS_KEY="..."
   export AWS_SESSION_TOKEN="..."
   ```

5. These expire after the session duration (1–8 hours)

---

### STEP 8 – Verify SSO Session in CloudTrail

1. Go back to your admin account
2. Open **CloudTrail** → **Event History**
3. Filter by **Event name**: `AssumeRoleWithSAML`
4. You will see entries showing the SSO user assuming the IAM role
5. The **userIdentity** section shows:
   ```json
   {
     "type": "AssumedRole",
     "principalId": "AROA...:kastro.devops",
     "arn": "arn:aws:sts::123456789012:assumed-role/AWSReservedSSO_DevOpsEngineer_xxx/kastro.devops"
   }
   ```

> This shows who logged in, what role they assumed, and when. Full audit trail.

---

## 13. LAB 6 – Multi-Account Setup (SSO Across AWS Accounts)

### Prerequisites
- AWS Organizations must be active
- At least 2 AWS accounts in the organization

### STEP 1 – Add Member Accounts to the Organization

If you have only one account, skip to the concept explanation.

1. **AWS Organizations Console** → **AWS Accounts**
2. Click **Add an AWS Account**
3. Select **Create an AWS account**:
   - **Account name**: `KastroTech-Dev`
   - **Email**: `kastro+dev@yourdomain.com` (use email aliases)
   - **IAM role name**: Leave default
4. Click **Create AWS account** → Wait a few minutes

Repeat for QA, Staging accounts.

---

### STEP 2 – Organize Accounts into OUs (Organizational Units)

Organizational Units (OUs) are folders for grouping accounts:

1. **AWS Organizations** → **AWS Accounts**
2. Click **Actions** → **Create new organizational unit**
3. Create these OUs:
   - `Development` (contains Dev, QA accounts)
   - `Production` (contains Prod account)
   - `SharedServices` (contains logging, security accounts)

4. Move accounts into OUs by selecting an account → **Actions** → **Move**

---

### STEP 3 – Assign SSO Access Across Multiple Accounts

1. IAM Identity Center → **AWS Accounts**
2. Select ALL accounts you want to assign (use checkboxes)
3. Click **Assign Users or Groups**
4. Select group: `DevOpsEngineers`
5. Select permission set: `DevOpsEngineer`
6. Click **Submit**

AWS now assigns the permission set to ALL selected accounts simultaneously. This creates the IAM role in each account automatically.

---

### STEP 4 – Login and Verify Multi-Account Access

1. Open SSO portal (as the DevOps user)
2. Now you see ALL accounts:
   ```
   AWS Accounts (3)
   ┌─────────────────────────────────┐
   │ KastroTech Dev (123456789001)   │
   │ DevOpsEngineer [Console] [CLI]  │
   ├─────────────────────────────────┤
   │ KastroTech QA (123456789002)    │
   │ DevOpsEngineer [Console] [CLI]  │
   ├─────────────────────────────────┤
   │ KastroTech Prod (123456789004)  │
   │ ReadOnlyAccess [Console] [CLI]  │
   └─────────────────────────────────┘
   ```

3. Click "Console" for Dev → Logged in to Dev ✅
4. Go back to portal → Click "Console" for QA → Logged in to QA ✅
5. Go back to portal → Click "Console" for Prod → Logged in to Prod (read-only) ✅

---

## 14. LAB 7 – Connect External Identity Provider (Google Workspace)

### 🎯 Goal
Allow users to log into the SSO portal using their **company Google account** — no separate SSO password needed.

### Architecture

```
User clicks "Sign in with Google"
         │
         ▼
Google Workspace authenticates the user
         │
         ▼  SAML 2.0 Assertion (tells AWS "this user is verified")
         ▼
IAM Identity Center receives SAML response
         │
         ▼
User is logged into SSO portal with their permissions
```

### STEP 1 – Configure Google Workspace as an External IdP

> This requires admin access to a Google Workspace account.
> If you only have personal Gmail, you cannot do this lab (Google Workspace is for business domains).

#### In Google Admin Console (admin.google.com):
1. Go to **Apps** → **Web and mobile apps** → **Add App** → **Add custom SAML app**
2. **App name**: `AWS IAM Identity Center`
3. Download the **IdP metadata XML** file (you will upload this to AWS)
4. Configure ACS URL and Entity ID (these come from AWS — we get them in the next step)

#### In AWS IAM Identity Center:
1. **Settings** → **Identity source** tab
2. Click **Actions** → **Change identity source**
3. Select **External identity provider**
4. Click **Next**
5. Under **Service provider metadata**:
   - Copy the **IAM Identity Center SAML metadata URL**
   - Copy the **IAM Identity Center ACS URL**
   - Copy the **IAM Identity Center Issuer URL**
6. Go back to Google Admin Console → Paste these values into your SAML app configuration
7. Back in AWS → Upload the **IdP metadata XML** you downloaded from Google
8. Click **Next** → Check the confirmation box → Click **Change identity source**

### STEP 2 – Provision Users from Google to AWS (SCIM)

SCIM (System for Cross-domain Identity Management) automatically syncs users and groups from Google to AWS SSO.

1. In IAM Identity Center → **Settings** → **Identity source**
2. Click **Enable** under **Automatic provisioning**
3. Copy the **SCIM endpoint** and **Access token**
4. In Google Admin Console → Your SAML app → **Provisioning**
5. Paste the SCIM endpoint and access token
6. Enable provisioning → Google now automatically creates SSO users when you add them in Google Workspace

### STEP 3 – Test Google Login

1. Open SSO portal: `https://kastrotech.awsapps.com/start`
2. Click **Sign in with your corporate credentials** (or the Google button)
3. Enter your `@kastrotech.com` Google credentials
4. You are redirected back to the SSO portal — logged in without a separate AWS password

---

## 15. SSO for IAM Users – Does SSO Help IAM Users?

This is a common question from students.

### Short Answer

**SSO does not directly "upgrade" existing IAM users.** SSO is a separate authentication system. IAM users and SSO users are completely independent.

### Long Answer

```
IAM User:
  - Exists inside a specific AWS account
  - Has their own username, password, access keys in that account
  - Authenticates directly against that account's IAM service
  - Is NOT an SSO user

SSO User:
  - Exists in IAM Identity Center (centralized)
  - Has credentials managed by IAM Identity Center
  - Gets temporary access to accounts via role assumption
  - Is NOT an IAM user in any specific account

They do NOT interact with each other.
A person can be BOTH:
  - An IAM user in Account A (old way)
  - An SSO user with access to Account B (new way)
  ...but that's usually done during a migration, not long-term.
```

### Migration Path: IAM Users → SSO

```
BEFORE (IAM User setup):
  Account: KastroTech-Dev
  IAM User: kastro-devops-iam (password, access keys, MFA configured)

MIGRATION:
  Step 1: Create SSO user: kastro.devops@kastrotech.com
  Step 2: Assign SSO user to KastroTech-Dev account with DevOpsEngineer permission
  Step 3: Give SSO user time to set up and verify access
  Step 4: Disable IAM user access keys and console login
  Step 5: Eventually delete the IAM user

AFTER (SSO setup):
  IAM User: deleted (or disabled)
  SSO User: kastro.devops@kastrotech.com → accesses via SSO portal
```

### What About IAM Users for Automation/CI-CD?

IAM users with access keys are still used for **service accounts and automation** where a human is not logging in:
```
Human users       → Use SSO (temporary credentials, MFA, portal login)
Automation/CI-CD  → Use IAM Roles (not users) wherever possible
                    Or IAM Users with access keys if roles cannot be used
                    (e.g., external CI systems like Jenkins not on AWS)
```

---

## 16. Real-World Company Setup Example

### KastroTech – 50 Employees, 4 AWS Accounts

#### Account Structure:
```
Management Account (Billing + SSO + Organizations)
├── Development Account
├── QA Account
├── Staging Account
└── Production Account
```

#### Team Structure:
```
Team               Count  Permission Set          Accounts
─────────────────────────────────────────────────────────────
Backend Devs         15   DeveloperAccess         Dev, QA
Frontend Devs        10   DeveloperAccess         Dev, QA
DevOps Engineers      5   DevOpsEngineer          Dev, QA, Staging, Prod
QA Engineers          8   QAEngineer              Dev, QA
Data Engineers        5   DataEngineer            Dev, QA, Prod (data)
Security Team         2   SecurityAudit           All accounts (read)
Management            3   ReadOnlyAccess          All accounts
Cloud Admin           2   AdministratorAccess     All accounts
```

#### SSO Setup:
```
Identity Source: Google Workspace (kastrotech.com domain)
SSO URL: https://kastrotech.awsapps.com/start
MFA: Required for all users (enforced in IAM Identity Center settings)

Groups in IAM Identity Center:
  BackendDevs      → DeveloperAccess on Dev + QA
  FrontendDevs     → DeveloperAccess on Dev + QA
  DevOpsTeam       → DevOpsEngineer on all accounts
  QATeam           → QAEngineer on Dev + QA
  DataTeam         → DataEngineer on Dev + Prod
  SecurityTeam     → SecurityAudit on all accounts
  Management       → ReadOnlyAccess on all accounts
  CloudAdmins      → AdministratorAccess on all accounts
```

#### What Happens When a New Employee Joins:
```
1. IT creates Google Workspace account: newemployee@kastrotech.com
2. SCIM auto-creates SSO user in IAM Identity Center
3. IT adds user to the correct Google group (e.g., BackendDevs)
4. SCIM syncs group membership to IAM Identity Center
5. User automatically gets correct AWS access
6. Total admin time: 2 minutes
7. No manual IAM user creation in any AWS account
```

#### What Happens When an Employee Leaves:
```
1. IT disables Google Workspace account
2. SSO user is deprovisioned via SCIM automatically
3. All AWS access is revoked within minutes
4. Total admin time: 30 seconds
5. Zero chance of orphaned IAM users in any account
```

---

## 17. Troubleshooting Common Issues

### Issue 1 – User Does Not See Any AWS Accounts in Portal

**Symptom**: User logs into SSO portal but sees "No AWS accounts available"

**Cause**: Account assignment was not completed

**Fix**:
1. IAM Identity Center → **AWS Accounts**
2. Select the account → **Assign Users or Groups**
3. Assign the user or their group with a permission set
4. User refreshes the SSO portal

---

### Issue 2 – User Sees "Access Denied" After Clicking Console

**Symptom**: User clicks "Management Console" but gets AccessDenied error

**Cause**: Permission set does not allow what the user is trying to do

**Fix**:
1. IAM Identity Center → **Permission Sets** → Select the permission set
2. Review the policies attached
3. If the action needs a new policy, click **Actions** → **Edit** → Add the required policy
4. After editing, the permission set must be **re-provisioned** — go to **AWS Accounts** → Select account → Select the assignment → Click **Reprovision**

---

### Issue 3 – MFA Not Prompted After Password

**Symptom**: User can log in with just password, MFA is not required

**Fix** (Enforce MFA for all users):
1. IAM Identity Center → **Settings** → **Authentication** tab
2. Under **Multi-factor authentication** → Click **Configure**
3. Set:
   - **Prompt users for MFA**: `Every time they sign in`
   - **Who can manage MFA devices**: `Users and administrators can add and manage their own MFA devices`
   - **If a user does not yet have an MFA device**: `Require them to register an MFA device at sign in`
4. Click **Save**

---

### Issue 4 – Cannot Delete SSO User (Has Active Assignments)

**Symptom**: Delete button for user is greyed out or gives error

**Fix**:
1. Click on the user → **AWS Accounts** tab
2. Remove all account assignments first
3. Remove from all groups
4. Then delete the user

---

### Issue 5 – Session Expired, Cannot Access Console

**Symptom**: User gets "Session expired" or "Token expired" error in console

**Fix**: 
1. User goes back to SSO portal: `https://kastrotech.awsapps.com/start`
2. Click the account again → New temporary credentials are issued
3. This is expected behavior — sessions expire for security (configured by session duration in permission set)
4. If it expires too frequently → Edit permission set → Increase session duration (max 12 hours)

---

## 18. SSO vs IAM Users – When to Use What

| Situation | Recommendation |
|---|---|
| Human logging into AWS Console | ✅ SSO |
| DevOps engineer accessing multiple accounts | ✅ SSO |
| New company/startup setting up AWS | ✅ SSO (start the right way) |
| Company already using Google Workspace or Azure AD | ✅ SSO (external IdP) |
| Compliance requirements for centralized access control | ✅ SSO |
| Automated CI/CD pipeline (GitHub Actions, Jenkins) | IAM Role or IAM User with access keys |
| EC2 instance accessing S3 | IAM Role (attached to EC2 — not SSO, not IAM user) |
| Lambda function accessing DynamoDB | IAM Role (attached to Lambda) |
| Legacy single-account, no plans to scale | IAM User (acceptable but not recommended) |
| Emergency/break-glass access | IAM User with MFA + strict policy (backup to SSO) |

---

## 19. Quick Reference Cheatsheet

### Console Navigation

| Task | Console Path |
|---|---|
| Enable SSO | Search "IAM Identity Center" → Enable |
| Create SSO User | IAM Identity Center → Users → Add User |
| Create Group | IAM Identity Center → Groups → Create Group |
| Create Permission Set | IAM Identity Center → Permission Sets → Create |
| Assign to Account | IAM Identity Center → AWS Accounts → Select → Assign Users or Groups |
| Change Identity Source | IAM Identity Center → Settings → Identity source → Actions → Change |
| Enforce MFA | IAM Identity Center → Settings → Authentication → Configure |
| View SSO portal URL | IAM Identity Center → Settings → Identity source → AWS access portal URL |
| View auto-created IAM Roles | IAM Console → Roles → Search "AWSReservedSSO" |
| Audit SSO logins | CloudTrail → Event History → Filter: AssumeRoleWithSAML |

### Key Concepts Summary

```
IAM Identity Center (SSO)
│
├── Users → Created manually or synced from Google/AD/Okta
├── Groups → Logical grouping of users
├── Permission Sets → Define what users can do (IAM policies)
├── AWS Accounts → Assigned with user/group + permission set
└── SSO Portal → Single URL for all access (awsapps.com/start)

Login Flow:
User → SSO Portal → Authenticate (password + MFA) → Select Account
→ STS issues temp credentials → Console/CLI access → Session expires → Repeat

Security Benefits:
✅ No long-lived credentials
✅ Temporary credentials (1–12 hour sessions)
✅ Centralized MFA enforcement
✅ Single offboarding point
✅ Full audit trail in CloudTrail
✅ Cannot be made public (unlike IAM access keys which can be shared)
```

---

*Teaching Guide by Learn With KASTRO | YouTube: @LearnWithKASTRO | learnwithkastro.com*
*AWS Region: ap-south-1 (Mumbai) | Service: AWS IAM Identity Center (SSO)*
