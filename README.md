 **complete, professional “Microsoft Entra ID (formerly Azure Active Directory) – Full Setup & Practical Guide”**.

It includes:

* 📖 Full **theory** section
* 💻 All **CLI / PowerShell / Terraform codes**
* 🧩 Step-by-step **hands-on practicals**
* 🧠 Quick revision points & interview matrix

Ready for **GitHub README**, **Azure Lab**, or **student assignment** under Cloudnautic 💼.

---

# 📦 **Microsoft Entra ID (Azure AD) – Full Setup, Theory & Practical Guide**

**Author:** Atul Kamble
**Role:** Cloud Solutions Architect | DevOps Trainer
**Brand:** Cloudnautic
**Tools:** Azure Portal • Azure CLI • PowerShell • Terraform

---

## 🧠 **1. Theory Points to Remember**

### 🔹 Identity Fundamentals

| Concept                                  | Description                                                                |
| ---------------------------------------- | -------------------------------------------------------------------------- |
| **Tenant**                               | The top-level Entra ID container that represents your organization.        |
| **User**                                 | Represents a person or application identity that can authenticate.         |
| **Group**                                | Logical container for users; simplifies permission management.             |
| **Service Principal**                    | Application identity used for automation or API-based authentication.      |
| **RBAC**                                 | Role-Based Access Control – assign least-privilege roles to users or apps. |
| **Conditional Access**                   | Zero-Trust enforcement — requires MFA, device compliance, etc.             |
| **Identity Protection**                  | Uses AI to detect risky sign-ins and compromised identities.               |
| **Privileged Identity Management (PIM)** | Provides time-bound admin role activation.                                 |

---

## 🧩 **2. Core Administrative Practical Tasks**

Each lab below includes **commands, steps, and verification**.
You can perform these in **Azure Cloud Shell** or local CLI with authentication.

---

### 🧭 **Lab 1: Login and List Tenant Information**

#### 🔧 Azure CLI

```bash
az login
az account tenant list --output table
```

#### 🧩 Expected Output

| TenantId                             | DisplayName     | DefaultDomain              |
| ------------------------------------ | --------------- | -------------------------- |
| xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | Atul-DemoTenant | yourtenant.onmicrosoft.com |

---

### 👤 **Lab 2: Create a User**

#### 🔧 Azure CLI

```bash
az ad user create \
  --display-name "Atul Kamble" \
  --user-principal-name "atul.kamble@yourtenant.onmicrosoft.com" \
  --password "YourSecureP@ssw0rd!"
```

#### 🔧 PowerShell

```powershell
Connect-AzureAD
New-AzureADUser -DisplayName "Atul Kamble" `
  -UserPrincipalName "atul.kamble@yourtenant.onmicrosoft.com" `
  -PasswordProfile @{Password="YourSecureP@ssw0rd!"} `
  -AccountEnabled $true
```

#### ✅ Verify

```bash
az ad user list --output table
```

---

### 👥 **Lab 3: Create a Group**

#### 🔧 Azure CLI

```bash
az ad group create \
  --display-name "DevOpsTeam" \
  --mail-nickname "devopsteam"
```

#### 🔧 PowerShell

```powershell
New-AzureADGroup -DisplayName "DevOpsTeam" `
  -MailEnabled $false -SecurityEnabled $true `
  -MailNickname "devopsteam"
```

#### ✅ Verify

```bash
az ad group list --output table
```

---

### ➕ **Lab 4: Add User to Group**

#### Step 1: Get User Object ID

```bash
az ad user list --output table
```

#### Step 2: Add Member

```bash
az ad group member add \
  --group "DevOpsTeam" \
  --member-id <UserObjectId>
```

#### ✅ Verify

```bash
az ad group member list --group "DevOpsTeam" --output table
```

---

### 🧰 **Lab 5: Create a Service Principal (App Registration)**

#### 🔧 Azure CLI

```bash
az ad sp create-for-rbac \
  --name "tf-service-principal" \
  --role Contributor \
  --scopes /subscriptions/<your-subscription-id>
```

#### ✅ Output

```bash
{
  "appId": "xxxx-xxxx-xxxx-xxxx",
  "displayName": "tf-service-principal",
  "password": "xxxxxxxxxxxx",
  "tenant": "xxxx-xxxx-xxxx-xxxx"
}
```

Save these credentials for Terraform:

* appId → `client_id`
* password → `client_secret`
* tenant → `tenant_id`

---

### 🧩 **Lab 6: List & Manage Service Principals**

```bash
az ad sp list --output table
```

Optional – assign role:

```bash
az role assignment create --assignee <appId> --role "Reader"
```

---

### 🔐 **Lab 7: Configure Conditional Access (Portal)**

#### GUI Steps

1. Go to **Microsoft Entra ID → Security → Conditional Access**
2. Click **+ New Policy**
3. Name: “Require MFA for all users”
4. Assignments:

   * **Users:** All Users
   * **Cloud Apps:** All
   * **Grant:** Require MFA
5. Enable Policy ✅
6. Save and verify by testing user login with MFA prompt.

---

### 🧩 **Lab 8: Identity Protection Dashboard**

Portal → Microsoft Entra ID → **Protection**

* View **Risky Sign-ins**
* **Risky Users**
* **Leaked Credentials**
* Take remediation actions → block / reset password.

---

## ☁️ **3. Infrastructure as Code (Terraform) Setup**

### 📁 Folder Structure

```bash
entra-terraform/
├── provider.tf
├── user.tf
├── group.tf
├── group_member.tf
└── outputs.tf
```

---

### 🧩 `provider.tf`

```hcl
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 3.0"
    }
  }
}

provider "azuread" {
  tenant_id = "<your-tenant-id>"
  client_id = "<appId>"
  client_secret = "<password>"
}
```

---

### 👤 `user.tf`

```hcl
resource "azuread_user" "atul_user" {
  user_principal_name = "atul.kamble@yourtenant.onmicrosoft.com"
  display_name        = "Atul Kamble"
  password            = "YourSecureP@ssw0rd!"
  force_password_change = false
}
```

---

### 👥 `group.tf`

```hcl
resource "azuread_group" "devops_team" {
  display_name     = "DevOpsTeam"
  security_enabled = true
}
```

---

### 👥➕ `group_member.tf`

```hcl
resource "azuread_group_member" "devops_member" {
  group_object_id  = azuread_group.devops_team.id
  member_object_id = azuread_user.atul_user.id
}
```

---

### 📤 `outputs.tf`

```hcl
output "user_id" {
  value = azuread_user.atul_user.id
}

output "group_id" {
  value = azuread_group.devops_team.id
}
```

---

### 🚀 Terraform Commands

```bash
terraform init
terraform plan
terraform apply -auto-approve
terraform show
terraform destroy
```

✅ Verify from Azure Portal → Entra ID → Users → Groups.

---

## 🧠 **4. Quick Interview & Revision Notes**

| Concept                               | Key Point                                                    |
| ------------------------------------- | ------------------------------------------------------------ |
| **Tenant vs Subscription**            | Tenant = identity container; Subscription = billing unit.    |
| **SPN vs Managed Identity**           | SPN = manual credential, Managed Identity = auto-managed.    |
| **RBAC Scope Levels**                 | Management Group → Subscription → Resource Group → Resource. |
| **Conditional Access Policy Example** | Require MFA for users outside corporate IP range.            |
| **MFA Risk Reduction**                | Reduces breach likelihood by 99.22%.                         |
| **PIM Advantage**                     | Time-based, approver-based elevation for admin roles.        |
| **AzureAD vs Microsoft Graph**        | Graph is the modern API replacing AzureAD.                   |

---

## 🧰 **5. Troubleshooting Tips**

| Issue                | Cause                       | Fix                        |
| -------------------- | --------------------------- | -------------------------- |
| Login fails in CLI   | Token expired               | Run `az login` again       |
| Terraform auth error | Wrong SPN secret or expired | Rotate SPN credentials     |
| User not found       | Sync delay                  | Wait few minutes / refresh |
| Cannot delete user   | Assigned to group/app       | Remove dependencies first  |

---

## 📊 **6. Summary Matrix**

| Operation                 | Azure CLI | PowerShell | Terraform      | Portal |
| ------------------------- | --------- | ---------- | -------------- | ------ |
| Create User               | ✅         | ✅          | ✅              | ✅      |
| Create Group              | ✅         | ✅          | ✅              | ✅      |
| Add User to Group         | ✅         | ✅          | ✅              | ✅      |
| Create Service Principal  | ✅         | ✅          | ✅              | ✅      |
| Conditional Access Policy | ❌         | ❌          | 🟡 (Graph API) | ✅      |
| PIM & Identity Protection | ❌         | ❌          | ❌              | ✅      |

---

## 📚 **7. References & Learning Resources**

* [🔗 Microsoft Entra ID Docs](https://learn.microsoft.com/entra/identity/)
* [🔗 Azure CLI AD Reference](https://learn.microsoft.com/cli/azure/ad)
* [🔗 Terraform AzureAD Provider](https://registry.terraform.io/providers/hashicorp/azuread/latest/docs)
* [🔗 Microsoft Learn – Identity Fundamentals](https://learn.microsoft.com/training/modules/azure-ad-overview/)
* [🔗 Security Blog – Entra ID](https://techcommunity.microsoft.com/t5/azure-active-directory-blog/bg-p/AzureActiveDirectoryBlog)
* [🔗 Entra ID Pricing – India 2025](https://www.microsoft.com/en-in/security/business/microsoft-entra-id)

---

## 🧾 **8. Practice Assignment Ideas (For Students)**

| Task        | Description                                                      |
| ----------- | ---------------------------------------------------------------- |
| **Task 1:** | Create users and groups for DevOps Batch and assign permissions. |
| **Task 2:** | Automate creation of users & groups via Terraform.               |
| **Task 3:** | Setup Conditional Access → Block login from non-India IP.        |
| **Task 4:** | Create SPN for Jenkins & integrate in Azure Pipeline.            |
| **Task 5:** | Review Audit Logs & identify risky sign-ins.                     |

---

## 🚀 **End Result Snapshot**

✅ Users & Groups Created
✅ MFA & Conditional Access Enforced
✅ Terraform SPN Created
✅ Role-based Access Controlled
✅ Identity Protection Configured

---


