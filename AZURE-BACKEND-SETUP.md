# Azure Backend Setup for Terraform - Beginner's Guide

This guide will walk you through setting up Azure storage for Terraform state files using the Azure Portal. No prior Azure experience needed!

## What is a Backend?

A **backend** is where Terraform stores its "state file" - a file that keeps track of what resources Terraform has created. Storing this in Azure allows teams to collaborate and keeps your infrastructure safe.

---

## Step 1: Log into Azure Portal

1. Go to [https://portal.azure.com](https://portal.azure.com)
2. Sign in with your Azure account
3. You should see the Azure Portal dashboard

---

## Step 2: Create a Resource Group

A resource group is like a folder that holds related Azure resources.

1. In the search bar at the top, type **Resource groups** and click it
2. Click the **+ Create** button
3. Fill in:
   - **Subscription**: Select your subscription
   - **Resource group name**: `terraform-state-rg` (or any name you prefer)
   - **Region**: Choose a region close to you (e.g., `East US`)
4. Click **Review + create**
5. Click **Create**

---

## Step 3: Create a Storage Account

This is where your Terraform state file will live.

1. In the search bar at the top, type **Storage accounts** and click it
2. Click the **+ Create** button
3. Fill in the **Basics** tab:
   - **Subscription**: Select your subscription
   - **Resource group**: Select `terraform-state-rg` (the one you just created)
   - **Storage account name**: `tfstate12345` (must be unique across all of Azure, use numbers to make it unique)
   - **Region**: Same as your resource group (e.g., `East US`)
   - **Performance**: Standard
   - **Redundancy**: Locally-redundant storage (LRS)
4. Click **Review + create**
5. Click **Create**
6. Wait for deployment to complete (about 1 minute)
7. Click **Go to resource**

---

## Step 4: Create a Container

A container is like a folder inside your storage account.

1. You should be on your storage account page
2. In the left menu, scroll down and click **Containers** (under "Data storage")
3. Click **+ Container** at the top
4. Fill in:
   - **Name**: `tfstate`
   - **Public access level**: Private (no anonymous access)
5. Click **Create**

---

## Step 5: Create a Service Principal

A service principal is like a "robot user" that Terraform will use to log into Azure automatically.

1. In the search bar at the top, type **App registrations** and click it
2. Click **+ New registration**
3. Fill in:
   - **Name**: `terraform-sp`
   - **Supported account types**: Leave default (Accounts in this organizational directory only)
4. Click **Register**
5. You'll see the app registration page. **Copy and save these values somewhere safe:**
   - **Application (client) ID** - you'll need this
   - **Directory (tenant) ID** - you'll need this

### Create a Secret for the Service Principal

1. Still on the app registration page, click **Certificates & secrets** in the left menu
2. Click **+ New client secret**
3. Fill in:
   - **Description**: `terraform-secret`
   - **Expires**: 12 months (or your preference)
4. Click **Add**
5. **IMPORTANT**: Copy the **Value** immediately - you can't see it again! Save it somewhere safe.

---

## Step 6: Grant Permissions to Service Principal

Now we need to give the service principal permission to manage resources and access the storage account.

### 6.1: Grant Contributor Access on Subscription

1. In the search bar, type **Subscriptions** and click it
2. Click on your subscription name
3. Click **Access control (IAM)** in the left menu
4. Click **+ Add** → **Add role assignment**
5. On the **Role** tab:
   - Search for and select **Contributor**
   - Click **Next**
6. On the **Members** tab:
   - Click **+ Select members**
   - Search for `terraform-sp` (your service principal name)
   - Click on it to select it
   - Click **Select**
   - Click **Next**
7. Click **Review + assign**
8. Click **Review + assign** again

### 6.2: Grant Storage Access on Storage Account

1. In the search bar, type **Storage accounts** and click it
2. Click on your storage account (`tfstate12345` or whatever you named it)
3. Click **Access control (IAM)** in the left menu
4. Click **+ Add** → **Add role assignment**
5. On the **Role** tab:
   - Search for and select **Storage Blob Data Contributor**
   - Click **Next**
6. On the **Members** tab:
   - Click **+ Select members**
   - Search for `terraform-sp`
   - Click on it to select it
   - Click **Select**
   - Click **Next**
7. Click **Review + assign**
8. Click **Review + assign** again

---

## Step 7: Get Your Subscription ID

1. In the search bar, type **Subscriptions** and click it
2. You'll see your subscription listed
3. Copy the **Subscription ID** (it looks like: `12345678-1234-1234-1234-123456789abc`)

---

## Step 8: Configure Terraform

Now you have everything you need! Here's what you collected:

- **Application (client) ID** - from Step 5
- **Client secret value** - from Step 5
- **Directory (tenant) ID** - from Step 5
- **Subscription ID** - from Step 7
- **Resource group name** - `terraform-state-rg`
- **Storage account name** - `tfstate12345` (or your name)
- **Container name** - `tfstate`

### Create Your Terraform Backend Configuration

Create a new file called `backend.tf` in your Terraform project folder:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstate12345"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }
}
```

**Replace** `tfstate12345` with your actual storage account name.

### Set Environment Variables

Terraform needs your credentials. Create a file called `.env` (add this to `.gitignore`!):

```bash
export ARM_CLIENT_ID="your-client-id-here"
export ARM_CLIENT_SECRET="your-client-secret-here"
export ARM_SUBSCRIPTION_ID="your-subscription-id-here"
export ARM_TENANT_ID="your-tenant-id-here"
```

Replace the placeholder values with the actual values you copied earlier.

Before running Terraform, load these variables:

```bash
source .env
```

---

## Step 9: Initialize Terraform

Now you're ready to use Terraform with your backend!

```bash
# Load your credentials
source .env

# Initialize Terraform (this connects to your Azure backend)
terraform init

# Your state file is now stored in Azure!
```

---

## What Just Happened?

1. You created a place in Azure (storage account) to store Terraform's state file
2. You created a "robot user" (service principal) for Terraform to use
3. You gave that robot user permission to manage Azure resources and access the storage
4. You told Terraform where to store its state file and how to log in

Now when you run `terraform apply`, the state file will be safely stored in Azure instead of on your local computer!

---

## Troubleshooting

**"Storage account name is already taken"**
- Storage account names must be unique across ALL of Azure
- Try adding more numbers: `tfstate98765`

**"Permission denied" when running Terraform**
- Make sure you ran `source .env` to load your credentials
- Double-check that you copied the client secret correctly (it's only shown once!)

**"Backend initialization required"**
- This is normal! Run `terraform init` to connect to your Azure backend

**Need to share with your team?**
- Share the storage account details and backend configuration
- Each team member needs to set their own environment variables OR use the same service principal credentials (store them securely!)
