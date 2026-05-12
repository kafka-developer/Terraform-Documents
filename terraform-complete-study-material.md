# Terraform Associate 004 - Complete Study Material

> **Everything you need to pass the exam in one document**
> 
> Actual explanations, working code examples, and hands-on labs for all 37 exam topics.

---

## How to Use This Document

1. Read each topic in order
2. Type out the code examples (don't just read)
3. Try the labs in your own environment
4. Move to next topic when you understand the current one

**You don't need anything else.** This document contains all the explanations and examples you need.

---

## Table of Contents

### Section 1: Infrastructure as Code (IaC) with Terraform
- [1a: What is Infrastructure as Code?](#1a-what-is-infrastructure-as-code)
- [1b: Advantages of IaC](#1b-advantages-of-iac)
- [1c: Multi-cloud and Hybrid Cloud](#1c-multi-cloud-and-hybrid-cloud)

### Section 2: Terraform Fundamentals
- [2a: Providers - Installation and Versioning](#2a-providers---installation-and-versioning)
- [2b: How Providers Work](#2b-how-providers-work)
- [2c: Using Multiple Providers](#2c-using-multiple-providers)
- [2d: Terraform State](#2d-terraform-state)

### Section 3: Core Terraform Workflow
- [3a: The Terraform Workflow](#3a-the-terraform-workflow)
- [3b: terraform init](#3b-terraform-init)
- [3c: terraform validate](#3c-terraform-validate)
- [3d: terraform plan](#3d-terraform-plan)
- [3e: terraform apply](#3e-terraform-apply)
- [3f: terraform destroy](#3f-terraform-destroy)
- [3g: terraform fmt](#3g-terraform-fmt)

### Section 4: Terraform Configuration
- [4a: Resources vs Data Sources](#4a-resources-vs-data-sources)
- [4b: Resource References](#4b-resource-references)
- [4c: Variables and Outputs](#4c-variables-and-outputs)
- [4d: Complex Types](#4d-complex-types)
- [4e: Expressions and Functions](#4e-expressions-and-functions)
- [4f: Resource Dependencies](#4f-resource-dependencies)
- [4g: Custom Validation](#4g-custom-validation)
- [4h: Managing Sensitive Data](#4h-managing-sensitive-data)

### Section 5: Terraform Modules
- [5a: Module Sources](#5a-module-sources)
- [5b: Variable Scope in Modules](#5b-variable-scope-in-modules)
- [5c: Using Modules](#5c-using-modules)
- [5d: Module Versioning](#5d-module-versioning)

### Section 6: State Management
- [6a: Local Backend](#6a-local-backend)
- [6b: State Locking](#6b-state-locking)
- [6c: Remote State](#6c-remote-state)
- [6d: Drift and State Management](#6d-drift-and-state-management)

### Section 7: Maintaining Infrastructure
- [7a: Importing Resources](#7a-importing-resources)
- [7b: State Inspection](#7b-state-inspection)
- [7c: Logging and Debugging](#7c-logging-and-debugging)

### Section 8: HCP Terraform
- [8a: HCP Terraform Basics](#8a-hcp-terraform-basics)
- [8b: Collaboration Features](#8b-collaboration-features)
- [8c: Workspaces and Projects](#8c-workspaces-and-projects)
- [8d: CLI Integration](#8d-cli-integration)

---

# Section 1: Infrastructure as Code (IaC) with Terraform

## 1a: What is Infrastructure as Code?

### The Concept

Infrastructure as Code (IaC) means managing your infrastructure through code files instead of clicking through web consoles or running manual commands.

**Traditional approach:**
1. Log into AWS console
2. Click "Launch Instance"
3. Fill out form: ami, instance type, network settings
4. Click "Launch"

**IaC approach:**
1. Write in a file: "I want an EC2 instance with these settings"
2. Run one command
3. Infrastructure is created

### Why This Matters

When you write infrastructure as code:
- You can track changes in Git (who changed what, when, why)
- You can recreate the exact same infrastructure anywhere
- You can review infrastructure changes like code changes (pull requests)
- You can't forget steps - the code is the checklist

### Simple Example

```hcl
# main.tf
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "MyWebServer"
  }
}
```

Save this file, run `terraform apply`, and you have an EC2 instance. Run it again tomorrow, you get the same instance configuration.

---

## 1b: Advantages of IaC

### 1. Version Control

Your infrastructure changes are tracked just like code:

```bash
git log main.tf
# See: who created the VPC, who added the subnet, who changed security groups
```

If something breaks, you can revert:
```bash
git revert abc123
terraform apply
# Infrastructure rolled back to previous version
```

### 2. Repeatability

Create identical environments:

```hcl
# dev.tfvars
environment = "dev"
instance_type = "t2.micro"

# prod.tfvars  
environment = "prod"
instance_type = "t2.large"
```

```bash
terraform apply -var-file=dev.tfvars   # Creates dev environment
terraform apply -var-file=prod.tfvars  # Creates prod environment (identical structure)
```

### 3. Automation

No human needed:

```bash
# CI/CD pipeline
git push → terraform plan → review → terraform apply
```

Deploy infrastructure on every merge, just like deploying code.

### 4. Documentation

The code IS the documentation:

```hcl
# Instead of documentation that says:
# "Production uses 3 availability zones with private subnets..."

# The code shows it:
resource "aws_subnet" "private" {
  count             = 3
  availability_zone = var.azs[count.index]
  # ...
}
```

---

## 1c: Multi-cloud and Hybrid Cloud

### The Problem Terraform Solves

Each cloud has its own tools:
- AWS → CloudFormation
- Azure → ARM Templates  
- GCP → Deployment Manager

Learning all three is painful. Managing infrastructure across them is worse.

### Terraform's Solution: Providers

Terraform uses **providers** - plugins that know how to talk to different services:

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
    azure = {
      source = "hashicorp/azurerm"
    }
  }
}

# Same syntax, different clouds
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}

resource "azurerm_virtual_machine" "web" {
  name     = "webvm"
  vm_size  = "Standard_B1s"
}
```

Same commands (`init`, `plan`, `apply`) work for both.

### Multi-cloud Example

```hcl
# AWS for compute
resource "aws_instance" "app" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}

# Cloudflare for DNS
resource "cloudflare_record" "app" {
  zone_id = var.cloudflare_zone
  name    = "app"
  value   = aws_instance.app.public_ip
  type    = "A"
}

# Datadog for monitoring
resource "datadog_monitor" "app" {
  name    = "App Health"
  type    = "metric alert"
  query   = "avg(last_5m):avg:system.load.1{host:${aws_instance.app.id}} > 2"
  message = "App server overloaded"
}
```

One tool, three different services, all working together.

### Hybrid Cloud Example

```hcl
# On-premises VMware
resource "vsphere_virtual_machine" "db" {
  name = "database"
  # ... 
}

# AWS public cloud
resource "aws_instance" "web" {
  ami = "ami-12345"
  # ...
}

# Connect them
resource "aws_vpn_connection" "onprem" {
  vpn_gateway_id      = aws_vpn_gateway.main.id
  customer_gateway_id = aws_customer_gateway.onprem.id
}
```

Same workflow manages both on-prem and cloud infrastructure.

---

# Section 2: Terraform Fundamentals

## 2a: Providers - Installation and Versioning

### What Providers Are

Providers are plugins that let Terraform talk to APIs. Without a provider, Terraform can't do anything.

Think of it like:
- Terraform = your brain
- Provider = your hands
- API = the thing you want to manipulate

### Specifying Providers

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

**What this means:**
- `source = "hashicorp/aws"` - Get the AWS provider from HashiCorp
- `version = "~> 5.0"` - Use version 5.x (5.0, 5.1, 5.2, etc. but not 6.0)

### Version Constraint Operators

```hcl
version = "= 5.0.0"   # Exactly 5.0.0
version = ">= 5.0"    # 5.0 or higher
version = "< 6.0"     # Anything below 6.0
version = "~> 5.0"    # >= 5.0.0 and < 6.0.0
version = "~> 5.1.0"  # >= 5.1.0 and < 5.2.0
```

The `~>` operator is most common - it means "this major/minor version, but I'll take patches."

### Installing Providers

When you run `terraform init`:

```bash
$ terraform init

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.31.0...
- Installed hashicorp/aws v5.31.0
```

Terraform:
1. Reads your `required_providers` block
2. Downloads the provider plugin
3. Stores it in `.terraform/providers/`
4. Creates `.terraform.lock.hcl` with exact version

### The Lock File

`.terraform.lock.hcl` records exactly which version was installed:

```hcl
provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.31.0"
  constraints = "~> 5.0"
  hashes = [
    "h1:abc123...",
  ]
}
```

**Commit this file to Git.** It ensures your team uses the same provider version.

### Lab: Install and Version a Provider

**Step 1:** Create `main.tf`:

```hcl
terraform {
  required_providers {
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
  }
}

resource "random_pet" "name" {
  length = 2
}

output "pet_name" {
  value = random_pet.name.id
}
```

**Step 2:** Initialize:
```bash
terraform init
# Watch it download the random provider
```

**Step 3:** Apply:
```bash
terraform apply
# Creates a random pet name
```

**Step 4:** Check the lock file:
```bash
cat .terraform.lock.hcl
# See the exact version that was installed
```

**Step 5:** Try upgrading:
```hcl
# Change version constraint
version = "~> 3.6"
```

```bash
terraform init -upgrade
# Downloads new version within constraint
```

---

## 2b: How Providers Work

### The Provider's Job

A provider translates your HCL code into API calls.

**You write:**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}
```

**Provider does:**
```
1. terraform plan called
2. Provider reads your config
3. Provider calls AWS API: "GET /instances" to check current state
4. Provider compares desired (t2.micro) vs actual (nothing exists)
5. Provider tells Terraform: "Need to create an instance"

6. terraform apply called
7. Provider calls AWS API: "POST /instances" with ami + instance_type
8. AWS creates the instance
9. Provider returns the instance ID to Terraform
10. Terraform stores it in state
```

### Provider Configuration

Providers need credentials to access APIs:

```hcl
provider "aws" {
  region = "us-west-2"
  # Credentials from environment variables or ~/.aws/credentials
}
```

You don't typically put credentials in code:

```bash
# Instead, use environment variables
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."

# Or AWS credentials file
~/.aws/credentials
```

### Resources and Data Sources

Each provider gives you:

**Resources** (things you create):
```hcl
resource "aws_instance" "web" { }
resource "aws_vpc" "main" { }
resource "aws_s3_bucket" "data" { }
```

**Data sources** (things you query):
```hcl
data "aws_ami" "ubuntu" { }
data "aws_availability_zones" "available" { }
data "aws_caller_identity" "current" { }
```

### Multiple Provider Configurations

You can configure the same provider multiple times:

```hcl
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

provider "aws" {
  alias  = "east"
  region = "us-east-1"
}

# Use specific provider
resource "aws_instance" "west_server" {
  provider = aws.west
  # ...
}

resource "aws_instance" "east_server" {
  provider = aws.east
  # ...
}
```

### Lab: Provider Configuration

**Step 1:** Create multi-region setup:

```hcl
provider "aws" {
  alias  = "california"
  region = "us-west-1"
}

provider "aws" {
  alias  = "virginia"
  region = "us-east-1"
}

# Using random provider since it doesn't need credentials
provider "random" {}

resource "random_pet" "west" {
  # Uses default random provider
  prefix = "west"
}

resource "random_pet" "east" {
  prefix = "east"
}

output "west_pet" {
  value = random_pet.west.id
}

output "east_pet" {
  value = random_pet.east.id
}
```

**Step 2:** Run it:
```bash
terraform init
terraform apply
# See two different pets created
```

---

## 2c: Using Multiple Providers

### Why Multiple Providers?

Real infrastructure uses multiple services:

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
    cloudflare = {
      source = "cloudflare/cloudflare"
    }
    datadog = {
      source = "datadog/datadog"
    }
  }
}

# AWS for servers
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}

# Cloudflare for DNS
resource "cloudflare_record" "web" {
  zone_id = var.zone_id
  name    = "www"
  value   = aws_instance.web.public_ip
  type    = "A"
}

# Datadog for monitoring
resource "datadog_monitor" "web" {
  name = "Web server down"
  type = "service check"
  query = "\"http.can_connect\".over(\"instance:${aws_instance.web.id}\").last(2).count_by_status()"
}
```

One `terraform apply` manages all three services.

### Cross-Provider References

Providers can reference each other:

```hcl
# Create S3 bucket in AWS
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket"
}

# Point domain to bucket
resource "cloudflare_record" "bucket" {
  zone_id = var.zone_id
  name    = "data"
  value   = aws_s3_bucket.data.bucket_regional_domain_name
  type    = "CNAME"
}
```

Cloudflare resource uses output from AWS resource.

### Lab: Multi-Provider Setup

Since we need providers that don't require credentials, we'll use `random` and `time`:

```hcl
terraform {
  required_providers {
    random = {
      source = "hashicorp/random"
    }
    time = {
      source = "hashicorp/time"
    }
  }
}

# Generate a random name
resource "random_pet" "server" {
  length = 2
}

# Wait 5 seconds
resource "time_sleep" "wait" {
  create_duration = "5s"
}

# Generate another name after waiting
resource "random_pet" "delayed_server" {
  length = 2
  
  depends_on = [time_sleep.wait]
}

output "first_server" {
  value = random_pet.server.id
}

output "delayed_server" {
  value = random_pet.delayed_server.id
}

output "wait_completed" {
  value = "Waited 5 seconds between creations"
}
```

Run this:
```bash
terraform init
terraform apply
# Watch it create first pet, wait, then create second pet
```

---

## 2d: Terraform State

### What is State?

State is how Terraform knows what infrastructure exists.

**Without state:**
```bash
terraform apply  # Creates instance i-123
terraform apply  # Creates ANOTHER instance i-456 (doesn't know about first!)
```

**With state:**
```bash
terraform apply  # Creates instance i-123, records in state
terraform apply  # Checks state, sees i-123 exists, does nothing
```

### State File Structure

`terraform.tfstate` is JSON:

```json
{
  "version": 4,
  "terraform_version": "1.6.0",
  "resources": [
    {
      "type": "aws_instance",
      "name": "web",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "attributes": {
            "id": "i-1234567890abcdef0",
            "ami": "ami-0c55b159cbfafe1f0",
            "instance_type": "t2.micro",
            "public_ip": "54.123.45.67",
            "private_ip": "10.0.1.5"
          }
        }
      ]
    }
  ]
}
```

State tracks:
- Resource IDs (i-123)
- All attributes (public_ip, private_ip, etc.)
- Dependencies between resources
- Provider configuration

### How Terraform Uses State

**During plan:**
```
1. Read state file
2. Read .tf config files
3. Compare: "State says instance is t2.micro, config says t2.large"
4. Plan: "Will modify instance"
```

**During apply:**
```
1. Execute changes
2. Update state with new values
```

### State Contains Secrets

**Important:** State files contain everything about your resources, including:
- Database passwords
- API keys
- Private keys
- Any "sensitive" values

```hcl
resource "aws_db_instance" "db" {
  password = "super-secret-password"
}
```

State will contain:
```json
{
  "attributes": {
    "password": "super-secret-password"
  }
}
```

**Never commit state to Git!**

### Lab: Examining State

**Step 1:** Create simple infrastructure:

```hcl
resource "random_pet" "first" {
  length = 2
}

resource "random_pet" "second" {
  length = 3
}

output "first_pet" {
  value = random_pet.first.id
}
```

**Step 2:** Apply:
```bash
terraform apply
```

**Step 3:** Look at state:
```bash
cat terraform.tfstate
# See JSON with both random_pet resources
```

**Step 4:** Inspect specific resource:
```bash
terraform state show random_pet.first
# Shows just that resource's state
```

**Step 5:** List all resources:
```bash
terraform state list
# Lists all resources in state
```

**Step 6:** Make a change:
```hcl
resource "random_pet" "first" {
  length = 4  # Changed from 2 to 4
}
```

**Step 7:** Plan:
```bash
terraform plan
# See: Terraform detects difference between state (length=2) and config (length=4)
# Plans to replace the resource
```

**Step 8:** Look at state again - it hasn't changed yet:
```bash
terraform state show random_pet.first
# Still shows length = 2 (old value)
```

**Step 9:** Apply:
```bash
terraform apply
```

**Step 10:** Check state now:
```bash
terraform state show random_pet.first
# Now shows length = 4 (updated)
```

This demonstrates how state tracks current infrastructure and enables Terraform to detect changes.

---

# Section 3: Core Terraform Workflow

## 3a: The Terraform Workflow

The standard workflow is:

```
Write → Init → Plan → Apply → (repeat)
```

### 1. Write

Create `.tf` files:

```hcl
# main.tf
resource "random_pet" "my_pet" {
  length = 2
}
```

### 2. Init

Download providers:

```bash
terraform init
```

### 3. Plan

Preview changes:

```bash
terraform plan
```

Output shows:
```
Terraform will perform the following actions:

  # random_pet.my_pet will be created
  + resource "random_pet" "my_pet" {
      + id        = (known after apply)
      + length    = 2
      + separator = "-"
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

### 4. Apply

Execute changes:

```bash
terraform apply
```

### Full Lab: Complete Workflow

**Step 1 - Write:**

```hcl
# main.tf
terraform {
  required_providers {
    random = {
      source = "hashicorp/random"
    }
  }
}

resource "random_pet" "server_name" {
  length    = 2
  separator = "-"
  prefix    = "server"
}

resource "random_integer" "port" {
  min = 8000
  max = 9000
}

output "server_name" {
  value = random_pet.server_name.id
}

output "port" {
  value = random_integer.port.result
}
```

**Step 2 - Init:**
```bash
terraform init
# Downloads random provider
```

**Step 3 - Plan:**
```bash
terraform plan
# Shows: will create 2 resources
```

**Step 4 - Apply:**
```bash
terraform apply
# Type "yes"
# Creates resources
# Shows outputs: server_name and port
```

**Step 5 - Modify:**
```hcl
# Change length
resource "random_pet" "server_name" {
  length    = 3  # Changed from 2
  separator = "-"
  prefix    = "server"
}
```

**Step 6 - Plan again:**
```bash
terraform plan
# Shows: will replace random_pet (because length changed)
```

**Step 7 - Apply changes:**
```bash
terraform apply
# Destroys old pet, creates new one
```

**Step 8 - Destroy everything:**
```bash
terraform destroy
# Type "yes"
# Removes all resources
```

---

## 3b: terraform init

### What Init Does

1. **Downloads providers**
2. **Downloads modules**
3. **Initializes backend**
4. **Creates lock file**

### First Time Init

```bash
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/random versions matching "~> 3.5"...
- Installing hashicorp/random v3.5.1...
- Installed hashicorp/random v3.5.1

Terraform has been successfully initialized!
```

Creates:
- `.terraform/` directory (cached providers/modules)
- `.terraform.lock.hcl` (provider versions)

### When to Run Init

Run `terraform init` when you:
- Start a new project
- Add a new provider
- Add a new module  
- Change backend configuration
- Clone a repo with Terraform code

### Init with Upgrade

Update providers to latest version matching constraints:

```bash
terraform init -upgrade
```

Before:
```
Using hashicorp/random v3.5.1
```

After:
```
Upgrading hashicorp/random from v3.5.1 to v3.6.0
```

### Lab: Understanding Init

**Step 1:** Create config:
```hcl
terraform {
  required_providers {
    random = {
      source  = "hashicorp/random"
      version = "= 3.5.1"  # Exact version
    }
  }
}

resource "random_pet" "name" {}
```

**Step 2:** Init:
```bash
terraform init
# Installs exactly 3.5.1
```

**Step 3:** Check what was created:
```bash
ls -la .terraform/
# See providers directory

cat .terraform.lock.hcl
# See exact version locked
```

**Step 4:** Change version constraint:
```hcl
version = "~> 3.5"  # Allow 3.5.x
```

**Step 5:** Try to apply without re-init:
```bash
terraform apply
# Error! Provider version changed, need init
```

**Step 6:** Re-init with upgrade:
```bash
terraform init -upgrade
# Downloads newer 3.5.x version
```

**Step 7:** Check lock file:
```bash
cat .terraform.lock.hcl
# See new version
```

---

## 3c: terraform validate

### What Validate Checks

Syntax and logic errors:

```hcl
# Syntax error - missing quote
resource "random_pet" "bad {
  length = 2
}

# Logic error - invalid resource reference
resource "random_pet" "pet" {
  length = random_integer.doesnt_exist.result
}

# Missing required argument
resource "random_password" "pass" {
  # Missing 'length' - required argument
}
```

### Running Validate

```bash
$ terraform validate
```

**Success:**
```
Success! The configuration is valid.
```

**Failure:**
```
Error: Argument or block definition required

  on main.tf line 5:
   5: resource "random_pet" "bad {

An argument or block definition is required here.
```

### Validate vs Plan

**Validate:**
- Checks syntax/structure
- No cloud API calls
- Fast
- Works without credentials

**Plan:**
- Validates first
- Then checks against actual infrastructure
- Makes API calls
- Needs credentials

### Lab: Validation Errors

**Step 1:** Create bad config:
```hcl
# main.tf with multiple errors
terraform {
  required_providers {
    random = {
      source = "hashicorp/random"
    }
  }
}

# Error 1: Missing closing brace
resource "random_pet" "bad1" {
  length = 2

# Error 2: Invalid reference
resource "random_pet" "bad2" {
  length = random_integer.nonexistent.result
}

# Error 3: Missing required argument
resource "random_password" "bad3" {
  special = true
  # Missing 'length' argument
}
```

**Step 2:** Try to validate:
```bash
terraform init
terraform validate
# See errors listed
```

**Step 3:** Fix one error at a time:

Fix error 1:
```hcl
resource "random_pet" "bad1" {
  length = 2
}  # Added closing brace
```

```bash
terraform validate
# Now shows error 2
```

Fix error 2:
```hcl
# Remove the invalid reference
resource "random_pet" "bad2" {
  length = 2
}
```

Fix error 3:
```hcl
resource "random_password" "bad3" {
  length  = 16
  special = true
}
```

**Step 4:** Validate again:
```bash
terraform validate
# Success!
```

---

## 3d: terraform plan

### What Plan Shows

Plan compares desired state (config) vs current state (infrastructure):

```
Current state: no resources
Config says: create a random_pet
Plan: + create random_pet
```

### Plan Output Symbols

```
+ create
~ update in-place
-/+ destroy and recreate
- destroy
```

### Reading a Plan

```bash
$ terraform plan

Terraform will perform the following actions:

  # random_pet.server will be created
  + resource "random_pet" "server" {
      + id        = (known after apply)
      + length    = 2
      + separator = "-"
    }

  # random_integer.port will be updated in-place
  ~ resource "random_integer" "port" {
        id     = "8543"
      ~ max    = 9000 -> 10000
        min    = 8000
        # (2 unchanged attributes hidden)
    }

Plan: 1 to add, 1 to change, 0 to destroy.
```

**+ random_pet.server will be created** - New resource  
**~ random_integer.port** - Existing resource modified  
**~ max = 9000 -> 10000** - Attribute changing from 9000 to 10000

### Saving Plans

```bash
# Save plan to file
terraform plan -out=tfplan

# Apply saved plan (no approval needed)
terraform apply tfplan
```

###  Lab: Understanding Plan Output

**Step 1:** Initial config:
```hcl
resource "random_pet" "servers" {
  count  = 2
  length = 2
}

resource "random_password" "db" {
  length  = 16
  special = true
}

output "server_names" {
  value = random_pet.servers[*].id
}
```

**Step 2:** Plan and apply:
```bash
terraform init
terraform plan
# See: 3 resources to create (2 pets + 1 password)

terraform apply
```

**Step 3:** Modify - add a server:
```hcl
resource "random_pet" "servers" {
  count  = 3  # Changed from 2 to 3
  length = 2
}
```

```bash
terraform plan
```

Output:
```
  # random_pet.servers[2] will be created
  + resource "random_pet" "servers" {
      + id        = (known after apply)
      + length    = 2
      + separator = "-"
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

**Step 4:** Modify - change password length:
```hcl
resource "random_password" "db" {
  length  = 20  # Changed from 16
  special = true
}
```

```bash
terraform plan
```

Output:
```
  # random_password.db must be replaced
-/+ resource "random_password" "db" {
      ~ id          = "xxx" -> (known after apply)
      ~ length      = 16 -> 20
      ~ result      = (sensitive value)
        # (6 unchanged attributes hidden)
    }

Plan: 0 to add, 0 to change, 1 to replace.
```

Notice `-/+` means destroy and recreate (password can't be modified in-place).

**Step 5:** Save and apply plan:
```bash
terraform plan -out=myplan
terraform apply myplan
# Applies without asking for confirmation
```

---

## 3e: terraform apply

### What Apply Does

1. Shows execution plan
2. Asks for confirmation
3. Executes changes
4. Updates state file

### Apply with Auto-Approve

Skip confirmation (dangerous!):

```bash
terraform apply -auto-approve
```

### Apply from Saved Plan

```bash
terraform plan -out=tfplan
terraform apply tfplan
# No confirmation needed - plan already approved
```

### Apply Output

```bash
$ terraform apply

random_pet.server: Creating...
random_pet.server: Creation complete after 0s [id=exact-seahorse]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

server_name = "exact-seahorse"
```

### Lab: Apply Workflow

**Step 1:** Create infrastructure:
```hcl
resource "random_pet" "app" {
  length = 2
  prefix = "app"
}

resource "random_integer" "count" {
  min = 1
  max = 10
}

output "app_name" {
  value = "${random_pet.app.id}-${random_integer.count.result}"
}
```

**Step 2:** Apply:
```bash
terraform apply
```

Output:
```
Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

random_integer.count: Creating...
random_pet.app: Creating...
random_integer.count: Creation complete [id=7]
random_pet.app: Creation complete [id=app-trusty-mole]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

app_name = "app-trusty-mole-7"
```

**Step 3:** Apply again (no changes):
```bash
terraform apply
```

Output:
```
No changes. Your infrastructure matches the configuration.
```

**Step 4:** Modify and apply:
```hcl
resource "random_integer" "count" {
  min = 1
  max = 100  # Changed from 10
}
```

```bash
terraform apply
# Shows change, asks for confirmation
# Type "yes"
# Replaces the random_integer
```

**Step 5:** Check outputs:
```bash
terraform output
# See new value
```

---

## 3f: terraform destroy

### What Destroy Does

Removes all resources managed by Terraform.

```bash
$ terraform destroy

Plan: 0 to add, 0 to change, 3 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure.
  
  Enter a value: yes

random_pet.server: Destroying...
random_pet.server: Destruction complete
```

### Destroy Specific Resources

```bash
terraform destroy -target=random_pet.server
# Only destroys that one resource
```

### Destroy Without Confirmation

```bash
terraform destroy -auto-approve
# Dangerous! No confirmation
```

### Lab: Destroy Operations

**Step 1:** Create some resources:
```hcl
resource "random_pet" "pets" {
  count  = 3
  length = 2
}

resource "random_password" "passwords" {
  count   = 2
  length  = 16
  special = true
}

output "all_pets" {
  value = random_pet.pets[*].id
}
```

```bash
terraform apply
# Creates 5 resources total
```

**Step 2:** Destroy one resource:
```bash
terraform destroy -target=random_pet.pets[0]
# Only destroys first pet
```

**Step 3:** Check state:
```bash
terraform state list
# See 4 resources remain
```

**Step 4:** Destroy all:
```bash
terraform destroy
# Shows plan to destroy 4 remaining resources
# Type "yes"
# All resources destroyed
```

**Step 5:** Check state file:
```bash
cat terraform.tfstate
# See empty resources array
```

---

## 3g: terraform fmt

### What Fmt Does

Automatically formats `.tf` files to standard style:

**Before fmt:**
```hcl
resource "random_pet" "server"{
length=2
separator="_"
}
```

**After fmt:**
```hcl
resource "random_pet" "server" {
  length    = 2
  separator = "_"
}
```

### Running Fmt

```bash
# Format all .tf files in current directory
terraform fmt

# Format recursively
terraform fmt -recursive

# Check if formatting is needed (doesn't modify)
terraform fmt -check
```

### Lab: Formatting

**Step 1:** Create messy file:
```hcl
# badly_formatted.tf
resource "random_pet"    "server"    {
length=2
separator   =   "-"
}

resource    "random_integer" "port"   {
min=8000
    max=9000
}

output "result"   {
value="${random_pet.server.id}:${random_integer.port.result}"
}
```

**Step 2:** Check if formatting needed:
```bash
terraform fmt -check
# Returns non-zero exit code if formatting needed
# Lists files that need formatting
```

**Step 3:** Format:
```bash
terraform fmt
```

**Step 4:** Look at result:
```hcl
# badly_formatted.tf (now nicely formatted)
resource "random_pet" "server" {
  length    = 2
  separator = "-"
}

resource "random_integer" "port" {
  min = 8000
  max = 9000
}

output "result" {
  value = "${random_pet.server.id}:${random_integer.port.result}"
}
```

---

# Section 4: Terraform Configuration

## 4a: Resources vs Data Sources

### Resources Create Infrastructure

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}
```

This creates a real EC2 instance. Run `terraform destroy` and it's deleted.

### Data Sources Read Existing Infrastructure

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-*"]
  }
}
```

This queries AWS for an AMI that already exists. You didn't create it. Terraform just reads it.

### Using Them Together

```hcl
# Find an existing AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-*"]
  }
}

# Create an instance using that AMI
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id  # Reference data source
  instance_type = "t2.micro"
}
```

### Reference Syntax

**Resource:**
```hcl
resource.TYPE.NAME.ATTRIBUTE

aws_instance.web.id
random_pet.server.id
```

**Data source:**
```hcl
data.TYPE.NAME.ATTRIBUTE

data.aws_ami.ubuntu.id
data.aws_availability_zones.available.names
```

### Lab: Resources and Data Sources

We'll use `random` and `time` providers since they don't need AWS credentials:

```hcl
terraform {
  required_providers {
    random = { source = "hashicorp/random" }
    time   = { source = "hashicorp/time" }
  }
}

# DATA SOURCE - reads current time
data "time_static" "creation_time" {}

# RESOURCE - creates random pet name
resource "random_pet" "server" {
  length    = 2
  separator = "-"
}

# RESOURCE - creates password
resource "random_password" "db_password" {
  length  = 16
  special = true
}

# Use data source in output
output "created_at" {
  value = data.time_static.creation_time.rfc3339
}

# Use resource in output
output "server_name" {
  value = random_pet.server.id
}

output "db_password" {
  value     = random_password.db_password.result
  sensitive = true  # Won't show in output
}
```

Apply this:
```bash
terraform init
terraform apply
```

Notice:
- Data source (`time_static`) = read-only, returns current time
- Resources (`random_pet`, `random_password`) = created by Terraform

To see sensitive output:
```bash
terraform output db_password
```

---

## 4b: Resource References

### Basic Reference

```hcl
resource "random_pet" "app_name" {
  length = 2
}

resource "random_integer" "port" {
  min = 8000
  max = 9000
}

# Reference other resources
output "full_name" {
  value = "${random_pet.app_name.id}:${random_integer.port.result}"
}
```

### References Create Dependencies

Terraform sees the reference and knows: "port depends on app_name, so create app_name first."

```hcl
resource "random_pet" "database" {
  length = 2
}

# This DEPENDS ON database
resource "random_password" "db_password" {
  length = 16
  # Use database name as seed for consistent passwords
  keepers = {
    db_name = random_pet.database.id
  }
}
```

Terraform automatically creates database before db_password.

### Chaining References

```hcl
resource "random_pet" "project" {
  length = 1
}

resource "random_pet" "environment" {
  length = 1
}

resource "random_integer" "version" {
  min = 1
  max = 100
}

# Chain them all together
output "full_identifier" {
  value = "${random_pet.project.id}-${random_pet.environment.id}-v${random_integer.version.result}"
}
```

### Lab: Dependency Chain

```hcl
# Step 1: Generate project name
resource "random_pet" "project" {
  length = 2
}

# Step 2: Generate environment ID (depends on project)
resource "random_integer" "env_id" {
  min = 100
  max = 999
  
  # This creates an explicit dependency
  keepers = {
    project = random_pet.project.id
  }
}

# Step 3: Create server name using both
resource "random_pet" "server" {
  length = 2
  prefix = "${random_pet.project.id}-${random_integer.env_id.result}"
}

# Step 4: Create password using server name
resource "random_password" "server_password" {
  length = 16
  
  keepers = {
    server = random_pet.server.id
  }
}

output "deployment_info" {
  value = {
    project  = random_pet.project.id
    env_id   = random_integer.env_id.result
    server   = random_pet.server.id
    password = random_password.server_password.result
  }
  sensitive = true
}
```

Apply and watch the order:
```bash
terraform apply
```

Terraform creates in order:
1. project (no dependencies)
2. env_id (needs project)
3. server (needs project and env_id)
4. server_password (needs server)

Check the graph:
```bash
terraform graph | dot -Tpng > graph.png
# Opens dependency graph visualization
```

---

## 4c: Variables and Outputs

### Variable Basics

Variables are inputs to your configuration:

```hcl
variable "environment" {
  type        = string
  description = "Deployment environment"
  default     = "dev"
}

variable "instance_count" {
  type        = number
  description = "Number of instances"
  default     = 1
}

variable "enable_monitoring" {
  type    = bool
  default = false
}

# Use variables
resource "random_pet" "servers" {
  count  = var.instance_count
  length = var.environment == "prod" ? 3 : 2
}
```

### Passing Variable Values

**Method 1: Command line**
```bash
terraform apply -var="environment=prod" -var="instance_count=3"
```

**Method 2: Variable file**
```hcl
# dev.tfvars
environment      = "dev"
instance_count   = 1
enable_monitoring = false

# prod.tfvars
environment      = "prod"
instance_count   = 5
enable_monitoring = true
```

```bash
terraform apply -var-file="prod.tfvars"
```

**Method 3: terraform.tfvars (auto-loaded)**
```hcl
# terraform.tfvars
environment = "staging"
instance_count = 2
```

```bash
terraform apply  # Automatically uses terraform.tfvars
```

**Method 4: Environment variables**
```bash
export TF_VAR_environment="prod"
export TF_VAR_instance_count=5
terraform apply
```

### Variable Precedence (highest to lowest)

1. `-var` command line flags
2. `-var-file` files
3. `terraform.tfvars`
4. `*.auto.tfvars` (alphabetical order)
5. Environment variables (`TF_VAR_*`)
6. Default value in variable block

### Output Values

Outputs display values after apply:

```hcl
output "server_names" {
  value       = random_pet.servers[*].id
  description = "Names of all servers"
}

output "database_password" {
  value     = random_password.db.result
  sensitive = true  # Won't display automatically
}
```

View outputs:
```bash
terraform output
# Shows all non-sensitive outputs

terraform output database_password
# Shows specific output (even if sensitive)
```

### Variable Types

```hcl
# String
variable "name" {
  type = string
}

# Number
variable "count" {
  type = number
}

# Bool
variable "enabled" {
  type = bool
}

# List
variable "availability_zones" {
  type = list(string)
}

# Map
variable "tags" {
  type = map(string)
}

# Object
variable "server_config" {
  type = object({
    name = string
    size = number
    enabled = bool
  })
}
```

### Lab: Complete Variables Example

**variables.tf:**
```hcl
variable "project_name" {
  type        = string
  description = "Name of the project"
}

variable "environment" {
  type        = string
  description = "Environment (dev/staging/prod)"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "server_count" {
  type        = number
  description = "Number of servers"
  default     = 1
}

variable "tags" {
  type        = map(string)
  description = "Tags to apply"
  default     = {}
}

variable "password_length" {
  type      = number
  default   = 16
  sensitive = true
}
```

**main.tf:**
```hcl
terraform {
  required_providers {
    random = { source = "hashicorp/random" }
  }
}

resource "random_pet" "servers" {
  count  = var.server_count
  length = 2
  prefix = "${var.project_name}-${var.environment}"
}

resource "random_password" "admin" {
  length  = var.password_length
  special = true
}

output "server_names" {
  value = [for server in random_pet.servers : server.id]
}

output "admin_password" {
  value     = random_password.admin.result
  sensitive = true
}

output "deployment_info" {
  value = {
    project     = var.project_name
    environment = var.environment
    servers     = length(random_pet.servers)
    tags        = var.tags
  }
}
```

**dev.tfvars:**
```hcl
project_name = "myapp"
environment  = "dev"
server_count = 1
tags = {
  Team = "platform"
  Cost = "dev"
}
```

**prod.tfvars:**
```hcl
project_name = "myapp"
environment  = "prod"
server_count = 5
tags = {
  Team = "platform"
  Cost = "production"
}
password_length = 32
```

**Apply for dev:**
```bash
terraform apply -var-file="dev.tfvars"
```

**Apply for prod:**
```bash
terraform apply -var-file="prod.tfvars"
```

**View outputs:**
```bash
terraform output
terraform output admin_password
```

---

## 4d: Complex Types

### Lists

Ordered collections:

```hcl
variable "availability_zones" {
  type = list(string)
  default = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

# Access by index
resource "random_pet" "zone_servers" {
  count  = length(var.availability_zones)
  prefix = var.availability_zones[count.index]
}
```

### Maps

Key-value pairs:

```hcl
variable "instance_types" {
  type = map(string)
  default = {
    dev     = "t2.micro"
    staging = "t2.small"
    prod    = "t2.large"
  }
}

# Access by key
resource "random_pet" "server" {
  keepers = {
    instance_type = var.instance_types[var.environment]
  }
}
```

### Objects

Structured types:

```hcl
variable "server_config" {
  type = object({
    name        = string
    count       = number
    enabled     = bool
    tags        = map(string)
  })
  
  default = {
    name    = "web-server"
    count   = 3
    enabled = true
    tags    = {
      Environment = "prod"
      Team        = "platform"
    }
  }
}

# Access object properties
resource "random_pet" "servers" {
  count  = var.server_config.enabled ? var.server_config.count : 0
  prefix = var.server_config.name
}
```

### Sets

Unordered, unique collections:

```hcl
variable "allowed_cidrs" {
  type = set(string)
  default = ["10.0.0.0/16", "172.16.0.0/12"]
}

# Iterate over set
output "cidrs" {
  value = [for cidr in var.allowed_cidrs : cidr]
}
```

### Lab: Complex Types

```hcl
variable "environments" {
  type = map(object({
    server_count    = number
    instance_prefix = string
    monitoring      = bool
    allowed_ips     = list(string)
  }))
  
  default = {
    dev = {
      server_count    = 1
      instance_prefix = "dev-server"
      monitoring      = false
      allowed_ips     = ["10.0.1.0/24"]
    }
    prod = {
      server_count    = 5
      instance_prefix = "prod-server"
      monitoring      = true
      allowed_ips     = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
    }
  }
}

variable "current_env" {
  type    = string
  default = "dev"
}

# Use the complex type
locals {
  env_config = var.environments[var.current_env]
}

resource "random_pet" "servers" {
  count  = local.env_config.server_count
  length = 2
  prefix = local.env_config.instance_prefix
}

output "environment_summary" {
  value = {
    environment = var.current_env
    servers     = random_pet.servers[*].id
    monitoring  = local.env_config.monitoring
    allowed_ips = local.env_config.allowed_ips
  }
}
```

Apply with different environments:
```bash
terraform apply -var="current_env=dev"
terraform apply -var="current_env=prod"
```

---

## 4e: Expressions and Functions

### String Interpolation

```hcl
variable "project" {
  default = "webapp"
}

variable "environment" {
  default = "prod"
}

resource "random_pet" "server" {
  prefix = "${var.project}-${var.environment}"
  # Results in: webapp-prod-xxxxx
}
```

### Conditional Expressions

```hcl
variable "environment" {
  default = "dev"
}

resource "random_integer" "server_count" {
  min = var.environment == "prod" ? 5 : 1
  max = var.environment == "prod" ? 10 : 3
  # prod: between 5-10, others: between 1-3
}

resource "random_password" "password" {
  length  = var.environment == "prod" ? 32 : 16
  special = var.environment == "prod" ? true : false
}
```

### Common Functions

**file()** - Read file contents:
```hcl
resource "random_pet" "from_file" {
  prefix = file("${path.module}/prefix.txt")
}
```

**join()** - Concatenate strings:
```hcl
output "formatted_name" {
  value = join("-", ["app", "server", "01"])
  # Results in: app-server-01
}
```

**split()** - Split string into list:
```hcl
locals {
  ip_address = "192.168.1.100"
  octets = split(".", local.ip_address)
  # Results in: ["192", "168", "1", "100"]
}
```

**length()** - Get length of list/map/string:
```hcl
variable "zones" {
  default = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

output "zone_count" {
  value = length(var.zones)
  # Results in: 3
}
```

**lookup()** - Get value from map with default:
```hcl
variable "instance_types" {
  default = {
    dev  = "t2.micro"
    prod = "t2.large"
  }
}

locals {
  instance_type = lookup(var.instance_types, var.environment, "t2.small")
  # Returns value for environment, or t2.small if not found
}
```

**concat()** - Combine lists:
```hcl
locals {
  west_zones = ["us-west-2a", "us-west-2b"]
  east_zones = ["us-east-1a", "us-east-1b"]
  all_zones  = concat(local.west_zones, local.east_zones)
  # Results in: ["us-west-2a", "us-west-2b", "us-east-1a", "us-east-1b"]
}
```

**merge()** - Combine maps:
```hcl
variable "common_tags" {
  default = {
    Project = "webapp"
    Team    = "platform"
  }
}

variable "specific_tags" {
  default = {
    Environment = "prod"
  }
}

locals {
  all_tags = merge(var.common_tags, var.specific_tags)
  # Results in: {Project = "webapp", Team = "platform", Environment = "prod"}
}
```

### For Expressions

Transform lists:
```hcl
variable "names" {
  default = ["alice", "bob", "charlie"]
}

output "uppercase_names" {
  value = [for name in var.names : upper(name)]
  # Results in: ["ALICE", "BOB", "CHARLIE"]
}

output "name_map" {
  value = {for name in var.names : name => upper(name)}
  # Results in: {alice = "ALICE", bob = "BOB", charlie = "CHARLIE"}
}
```

### Splat Expressions

Shorthand for iterating:
```hcl
resource "random_pet" "servers" {
  count  = 3
  length = 2
}

output "all_ids" {
  value = random_pet.servers[*].id
  # Equivalent to: [for s in random_pet.servers : s.id]
}
```

### Lab: Functions and Expressions

```hcl
variable "project_name" {
  default = "myapp"
}

variable "environments" {
  default = ["dev", "staging", "prod"]
}

variable "team_members" {
  default = ["alice", "bob", "charlie"]
}

# Create servers for each environment
resource "random_pet" "servers" {
  count  = length(var.environments)
  length = 2
  prefix = "${var.project_name}-${var.environments[count.index]}"
}

# Create passwords for each team member
resource "random_password" "user_passwords" {
  for_each = toset(var.team_members)
  
  length  = 16
  special = true
  
  keepers = {
    user = each.key
  }
}

# Use various functions
output "summary" {
  value = {
    # String manipulation
    project_upper = upper(var.project_name)
    project_title = title(var.project_name)
    
    # List functions
    environment_count = length(var.environments)
    first_environment = var.environments[0]
    last_environment  = var.environments[length(var.environments) - 1]
    
    # For expression
    all_servers = [for s in random_pet.servers : s.id]
    
    # Map transformation
    user_list = keys(random_password.user_passwords)
    
    # Conditional
    is_production = contains(var.environments, "prod") ? "yes" : "no"
    
    # Join
    env_string = join(", ", var.environments)
  }
}

output "passwords" {
  value     = {for user, pass in random_password.user_passwords : user => pass.result}
  sensitive = true
}
```

Apply and check outputs:
```bash
terraform apply
terraform output summary
terraform output passwords
```

---

## 4f: Resource Dependencies

### Implicit Dependencies

Created automatically when you reference one resource in another:

```hcl
resource "random_pet" "database_name" {
  length = 2
}

resource "random_password" "db_password" {
  length = 16
  
  # This reference creates implicit dependency
  keepers = {
    db_name = random_pet.database_name.id
  }
}
```

Terraform knows: create database_name before db_password.

### Explicit Dependencies with depends_on

For dependencies that aren't obvious from references:

```hcl
resource "random_pet" "app" {
  length = 2
}

resource "random_password" "secret" {
  length = 16
}

# This resource doesn't reference the others,
# but needs them to exist first
resource "random_integer" "priority" {
  min = 1
  max = 100
  
  # Explicit dependency
  depends_on = [
    random_pet.app,
    random_password.secret
  ]
}
```

### When to Use depends_on

Use it when:
- Resource B needs Resource A to exist, but doesn't reference it
- A creates permissions that B will use
- Order matters but there's no direct reference

Don't use it when:
- You can create an implicit dependency with a reference
- There's no real dependency (implicit is clearer)

### Lab: Dependencies

```hcl
# 1. Foundation resources (no dependencies)
resource "random_pet" "project" {
  length = 1
}

resource "random_pet" "team" {
  length = 1
}

# 2. Config depends on both (implicit via references)
resource "random_integer" "config_version" {
  min = 1
  max = 100
  
  keepers = {
    project = random_pet.project.id
    team    = random_pet.team.id
  }
}

# 3. Database name uses config (implicit)
resource "random_pet" "database" {
  length = 2
  prefix = "db-v${random_integer.config_version.result}"
}

# 4. Password needs database (implicit)
resource "random_password" "db_password" {
  length = 16
  
  keepers = {
    database = random_pet.database.id
  }
}

# 5. Monitoring config needs everything (explicit)
resource "random_integer" "monitor_id" {
  min = 1000
  max = 9999
  
  # Explicit dependency - monitoring should be set up last
  depends_on = [
    random_pet.database,
    random_password.db_password
  ]
}

output "setup_order" {
  value = {
    "1_project"         = random_pet.project.id
    "1_team"            = random_pet.team.id
    "2_config_version"  = random_integer.config_version.result
    "3_database"        = random_pet.database.id
    "4_password"        = random_password.db_password.result
    "5_monitor_id"      = random_integer.monitor_id.result
  }
  sensitive = true
}
```

Apply and watch creation order:
```bash
terraform apply
# Notice the order resources are created
```

View dependency graph:
```bash
terraform graph
# Shows arrows between dependent resources
```

---

## 4g: Custom Validation

### Variable Validation

Ensure variable values meet requirements:

```hcl
variable "environment" {
  type = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "server_count" {
  type = number
  
  validation {
    condition     = var.server_count >= 1 && var.server_count <= 10
    error_message = "Server count must be between 1 and 10."
  }
}

variable "prefix" {
  type = string
  
  validation {
    condition     = can(regex("^[a-z][a-z0-9-]{2,20}$", var.prefix))
    error_message = "Prefix must start with lowercase letter, be 3-21 chars, and contain only lowercase letters, numbers, and hyphens."
  }
}
```

### Lab: Validation

```hcl
variable "project_name" {
  type        = string
  description = "Project name (lowercase, 3-20 chars)"
  
  validation {
    condition     = can(regex("^[a-z]{3,20}$", var.project_name))
    error_message = "Project name must be 3-20 lowercase letters only."
  }
}

variable "environment" {
  type = string
  
  validation {
    condition     = contains(["dev", "test", "staging", "prod"], var.environment)
    error_message = "Environment must be one of: dev, test, staging, prod."
  }
}

variable "replicas" {
  type = number
  
  validation {
    condition     = var.replicas >= 1 && var.replicas <= 100
    error_message = "Replicas must be between 1 and 100."
  }
  
  validation {
    condition     = var.replicas % 2 == 0 || var.replicas == 1
    error_message = "Replicas must be even number (for HA) or 1 (for dev)."
  }
}

resource "random_pet" "validated" {
  length = 2
  prefix = "${var.project_name}-${var.environment}"
}

output "config" {
  value = {
    project  = var.project_name
    env      = var.environment
    replicas = var.replicas
    name     = random_pet.validated.id
  }
}
```

Test validation:
```bash
# This will fail
terraform apply -var="project_name=MyApp" -var="environment=dev" -var="replicas=1"
# Error: uppercase in project_name

# This will fail
terraform apply -var="project_name=app" -var="environment=development" -var="replicas=1"
# Error: environment not in list

# This will fail
terraform apply -var="project_name=myapp" -var="environment=prod" -var="replicas=3"
# Error: replicas must be even (except 1)

# This works
terraform apply -var="project_name=myapp" -var="environment=prod" -var="replicas=4"
```

---

## 4h: Managing Sensitive Data

### The Problem

State files contain everything:

```hcl
resource "random_password" "database" {
  length  = 16
  special = true
}
```

State file will have:
```json
{
  "resources": [{
    "type": "random_password",
    "instances": [{
      "attributes": {
        "result": "V3ryS3cr3tP@ssw0rd!"
      }
    }]
  }]
}
```

**Anyone with state file access sees all secrets.**

### Sensitive Flag

Hide values from console output:

```hcl
variable "api_key" {
  type      = string
  sensitive = true
}

resource "random_password" "secret" {
  length  = 32
  special = true
}

output "password" {
  value     = random_password.secret.result
  sensitive = true
}
```

When you apply:
```bash
terraform apply
# password = <sensitive>
```

But it's still in state file!

### Protecting State

**1. Use remote state with encryption:**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true  # Encrypt at rest
    dynamodb_table = "terraform-locks"
  }
}
```

**2. Control state file access:**
```hcl
# S3 bucket policy
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::my-terraform-state/*",
  "Principal": {
    "AWS": ["arn:aws:iam::123456789:user/terraform"]
  }
}
```

**3. Never commit state to Git:**
```bash
# .gitignore
terraform.tfstate
terraform.tfstate.backup
*.tfstate
*.tfstate.*
```

### External Secret Management

Instead of hardcoding secrets:

```hcl
# BAD - secret in code
resource "random_password" "bad_example" {
  length = 16
}

# BETTER - from environment variable
variable "api_key" {
  type      = string
  sensitive = true
}
# Set via: export TF_VAR_api_key="secret"

# BEST - from vault/secrets manager
data "vault_generic_secret" "api" {
  path = "secret/data/api"
}

resource "some_service" "app" {
  api_key = data.vault_generic_secret.api.data["key"]
}
```

### Lab: Sensitive Data

```hcl
variable "db_admin_password" {
  type        = string
  description = "Database admin password"
  sensitive   = true
}

# Simulate database with secrets
resource "random_pet" "database" {
  length = 2
}

resource "random_password" "app_secret" {
  length  = 32
  special = true
}

resource "random_password" "api_key" {
  length  = 64
  special = false
}

# Sensitive output
output "database_connection" {
  value = {
    host     = "db.example.com"
    database = random_pet.database.id
    password = var.db_admin_password
  }
  sensitive = true
}

output "api_credentials" {
  value = {
    key    = random_password.api_key.result
    secret = random_password.app_secret.result
  }
  sensitive = true
}

# Non-sensitive output
output "database_name" {
  value = random_pet.database.id
}
```

Apply:
```bash
export TF_VAR_db_admin_password="SuperSecret123!"
terraform apply

# Outputs show:
# database_name = "clever-shark"
# database_connection = <sensitive>
# api_credentials = <sensitive>
```

View sensitive outputs:
```bash
terraform output database_connection
terraform output api_credentials
```

Check state file:
```bash
cat terraform.tfstate | grep -A5 "random_password"
# See all passwords in plaintext!
```

This shows why protecting state files is critical.

---

# Section 5: Terraform Modules

## 5a: Module Sources

### What are Modules?

A module is a container for multiple resources. Every Terraform configuration has at least one module (the root module).

### Local Modules

**Directory structure:**
```
.
├── main.tf
└── modules/
    └── server/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

**Calling local module:**
```hcl
module "web_server" {
  source = "./modules/server"
  
  name  = "web"
  count = 3
}
```

### Terraform Registry Modules

**Public modules:**
```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
}
```

Format: `namespace/module-name/provider`

### Git Repositories

```hcl
# HTTPS
module "server" {
  source = "git::https://github.com/org/terraform-modules.git//server"
}

# SSH
module "server" {
  source = "git@github.com:org/terraform-modules.git//server"
}

# Specific branch
module "server" {
  source = "git::https://github.com/org/terraform-modules.git//server?ref=development"
}

# Specific tag
module "server" {
  source = "git::https://github.com/org/terraform-modules.git//server?ref=v1.0.0"
}
```

### HTTP URLs

```hcl
module "server" {
  source = "https://example.com/terraform-modules/server.zip"
}
```

### Lab: Local Modules

**Step 1:** Create module structure:

```bash
mkdir -p modules/name-generator
```

**modules/name-generator/variables.tf:**
```hcl
variable "prefix" {
  type        = string
  description = "Prefix for generated names"
}

variable "length" {
  type    = number
  default = 2
}

variable "count" {
  type    = number
  default = 1
}
```

**modules/name-generator/main.tf:**
```hcl
resource "random_pet" "names" {
  count  = var.count
  length = var.length
  prefix = var.prefix
}
```

**modules/name-generator/outputs.tf:**
```hcl
output "names" {
  value = random_pet.names[*].id
}

output "first_name" {
  value = random_pet.names[0].id
}
```

**Step 2:** Use the module (root main.tf):

```hcl
terraform {
  required_providers {
    random = { source = "hashicorp/random" }
  }
}

# Use module for servers
module "servers" {
  source = "./modules/name-generator"
  
  prefix = "server"
  length = 2
  count  = 3
}

# Use module for databases
module "databases" {
  source = "./modules/name-generator"
  
  prefix = "db"
  length = 3
  count  = 2
}

output "all_servers" {
  value = module.servers.names
}

output "all_databases" {
  value = module.databases.names
}

output "first_server" {
  value = module.servers.first_name
}
```

**Step 3:** Apply:
```bash
terraform init  # Downloads module
terraform apply
```

---

## 5b: Variable Scope in Modules

### Module Isolation

Each module has its own variable namespace:

```
Root Module
├── var.environment = "prod"
└── module "server"
    └── var.environment = ???  # NOT automatically "prod"
```

Variables don't flow down automatically.

### Passing Data to Modules

**Parent must explicitly pass:**
```hcl
# Root module
variable "environment" {
  default = "prod"
}

module "server" {
  source = "./modules/server"
  
  environment = var.environment  # Explicit pass
}
```

**Child must declare:**
```hcl
# modules/server/variables.tf
variable "environment" {
  type = string
}
```

### Lab: Variable Scope

**modules/app/variables.tf:**
```hcl
variable "app_name" {
  type = string
}

variable "environment" {
  type = string
}

variable "replica_count" {
  type = number
}
```

**modules/app/main.tf:**
```hcl
resource "random_pet" "instances" {
  count  = var.replica_count
  length = 2
  prefix = "${var.app_name}-${var.environment}"
}
```

**modules/app/outputs.tf:**
```hcl
output "instance_names" {
  value = random_pet.instances[*].id
}
```

**Root main.tf:**
```hcl
variable "environment" {
  default = "production"
}

variable "region" {
  default = "us-west-2"
}

# Deploy app to production
module "prod_app" {
  source = "./modules/app"
  
  app_name      = "webapp"
  environment   = var.environment  # Pass root var
  replica_count = 5
}

# Deploy app to staging
module "staging_app" {
  source = "./modules/app"
  
  app_name      = "webapp"
  environment   = "staging"  # Hardcoded different value
  replica_count = 2
}

output "prod_instances" {
  value = module.prod_app.instance_names
}

output "staging_instances" {
  value = module.staging_app.instance_names
}
```

Notice:
- `var.region` exists in root but modules can't see it
- Must explicitly pass `var.environment` to modules
- Each module can receive different values

---

## 5c: Using Modules

### Module Block

```hcl
module "name" {
  source = "..."
  
  # Input variables
  argument1 = value1
  argument2 = value2
}
```

### Accessing Module Outputs

```hcl
module.module_name.output_name
```

**Example:**
```hcl
module "network" {
  source = "./modules/network"
  cidr   = "10.0.0.0/16"
}

# Use module output
resource "random_pet" "server" {
  prefix = "server-${module.network.vpc_id}"
}
```

### Count and For_Each with Modules

```hcl
# Create 3 instances of module
module "servers" {
  count  = 3
  source = "./modules/server"
  
  name = "server-${count.index}"
}

# Access with index
output "first_server" {
  value = module.servers[0].instance_id
}
```

```hcl
# Create module for each environment
module "env" {
  for_each = toset(["dev", "staging", "prod"])
  source   = "./modules/environment"
  
  name = each.key
}

# Access with key
output "prod_vpc" {
  value = module.env["prod"].vpc_id
}
```

### Lab: Complete Module Usage

**modules/stack/variables.tf:**
```hcl
variable "stack_name" {
  type = string
}

variable "server_count" {
  type = number
}
```

**modules/stack/main.tf:**
```hcl
resource "random_pet" "servers" {
  count  = var.server_count
  length = 2
  prefix = var.stack_name
}

resource "random_password" "admin" {
  length = 16
}
```

**modules/stack/outputs.tf:**
```hcl
output "server_names" {
  value = random_pet.servers[*].id
}

output "admin_password" {
  value     = random_password.admin.result
  sensitive = true
}
```

**Root main.tf:**
```hcl
# Multiple environments using count
module "environments" {
  count  = 3
  source = "./modules/stack"
  
  stack_name   = ["dev", "staging", "prod"][count.index]
  server_count = [1, 2, 5][count.index]
}

# Multiple regions using for_each
module "regions" {
  for_each = {
    us_west   = { name = "us-west-stack", servers = 3 }
    us_east   = { name = "us-east-stack", servers = 3 }
    eu_central = { name = "eu-stack", servers = 2 }
  }
  
  source = "./modules/stack"
  
  stack_name   = each.value.name
  server_count = each.value.servers
}

# Outputs from count-based modules
output "dev_servers" {
  value = module.environments[0].server_names
}

output "prod_servers" {
  value = module.environments[2].server_names
}

# Outputs from for_each-based modules
output "us_west_servers" {
  value = module.regions["us_west"].server_names
}

output "all_regions" {
  value = {
    for region, module in module.regions : 
      region => module.server_names
  }
}
```

Apply:
```bash
terraform init
terraform apply
```

---

## 5d: Module Versioning

### Why Version Modules?

Without versioning:
```hcl
module "server" {
  source = "git::https://github.com/org/modules.git//server"
}
```

Problem: Module author pushes breaking changes, your code breaks.

### Terraform Registry Versioning

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"  # Exact version
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.1"  # 5.1.x (allows patches)
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = ">= 5.0, < 6.0"  # Range
}
```

### Git Versioning with Tags

```hcl
module "server" {
  source = "git::https://github.com/org/modules.git//server?ref=v1.0.0"
}

module "server" {
  source = "git::https://github.com/org/modules.git//server?ref=development"
}
```

### Upgrading Modules

```bash
# Upgrade to latest version matching constraints
terraform init -upgrade
```

### Lab: Module Versioning

Since we can't use actual Terraform Registry or Git without external dependencies, we'll simulate versioning:

**Create versioned module directories:**

```bash
mkdir -p modules/generator/v1
mkdir -p modules/generator/v2
```

**modules/generator/v1/main.tf:**
```hcl
variable "prefix" {
  type = string
}

resource "random_pet" "name" {
  length = 2
  prefix = var.prefix
}

output "name" {
  value = random_pet.name.id
}
```

**modules/generator/v2/main.tf:**
```hcl
variable "prefix" {
  type = string
}

variable "length" {
  type    = number
  default = 3  # Changed default
}

resource "random_pet" "name" {
  length    = var.length
  prefix    = var.prefix
  separator = "_"  # Changed separator
}

resource "random_integer" "version" {
  min = 1
  max = 100
}

output "name" {
  value = random_pet.name.id
}

output "version_info" {
  value = "v2.${random_integer.version.result}"
}
```

**Root main.tf:**
```hcl
# Using v1
module "old_style" {
  source = "./modules/generator/v1"
  prefix = "app"
}

# Using v2
module "new_style" {
  source = "./modules/generator/v2"
  prefix = "app"
  length = 4
}

output "v1_name" {
  value = module.old_style.name
}

output "v2_name" {
  value = module.new_style.name
}

output "v2_version" {
  value = module.new_style.version_info
}
```

Apply:
```bash
terraform init
terraform apply
```

See differences between v1 and v2 outputs.

---

# Section 6: State Management

## 6a: Local Backend

### Default State Storage

By default, Terraform stores state locally:

```
terraform.tfstate          # Current state
terraform.tfstate.backup   # Previous state
```

### When Local Backend Works

Good for:
- Learning Terraform
- Personal projects
- Local testing
- Solo development

### When Local Backend Fails

Bad for:
- Teams (no sharing)
- CI/CD (state on each runner)
- Multiple machines
- Any production use

### Lab: Local Backend

```hcl
# No backend block = local backend
resource "random_pet" "local_example" {
  length = 2
}

output "pet_name" {
  value = random_pet.local_example.id
}
```

```bash
terraform init
# Notice: "Initializing the backend..."

terraform apply

ls -la
# See: terraform.tfstate
# See: terraform.tfstate.backup (after second apply)

cat terraform.tfstate
# See all state data in JSON

terraform apply
# Make no changes

ls -la
# See: terraform.tfstate.backup now contains previous state
```

---

## 6b: State Locking

### The Problem

**Without locking:**
```
Person A: terraform apply (starts)
Person B: terraform apply (starts)
Both try to write state simultaneously
State file corrupted
```

### How Locking Works

```
Person A: terraform apply
  → Acquires lock on state
  → Makes changes
  → Updates state
  → Releases lock

Person B: terraform apply
  → Tries to acquire lock
  → Waits (lock held by A)
  → Lock released
  → Acquires lock
  → Makes changes
```

### Backends with Locking

- **Terraform Cloud/HCP**: Built-in locking
- **S3 + DynamoDB**: Locking via DynamoDB table
- **Azure Blob**: Built-in locking
- **GCS**: Built-in locking
- **Local**: NO locking

### Lab: Observing Lock Behavior

We'll simulate what happens with local backend (no locking):

```hcl
resource "random_pet" "example" {
  length = 2
}

resource "time_sleep" "long_operation" {
  create_duration = "30s"
}
```

**Terminal 1:**
```bash
terraform apply
# This will take 30 seconds
```

**Terminal 2 (immediately while first is running):**
```bash
terraform apply
# With local backend, this will also start!
# Both can run simultaneously - dangerous!
```

With a locking backend, Terminal 2 would show:
```
Error: Error acquiring the state lock

Terraform acquires a state lock to protect the state from being written
by multiple users at the same time. Please resolve the issue above and try
again. For most commands, you can disable locking with the "-lock=false"
flag, but this is not recommended.

Lock Info:
  ID:        abc-123
  Path:      terraform.tfstate
  Operation: OperationTypeApply
  Who:       user@hostname
  Version:   1.6.0
  Created:   2024-01-15 10:30:00
```

---

## 6c: Remote State

### S3 Backend Example

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

### Benefits of Remote State

1. **Shared access** - Team uses same state
2. **Locking** - Prevents conflicts
3. **Encryption** - Protects secrets
4. **Versioning** - Keep history (S3)
5. **Backup** - Disaster recovery

### Backend Migration

**Step 1:** Start with local:
```hcl
# No backend block
resource "random_pet" "example" {
  length = 2
}
```

```bash
terraform init
terraform apply
# State in terraform.tfstate
```

**Step 2:** Add backend:
```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "example/terraform.tfstate"
    region = "us-west-2"
  }
}
```

**Step 3:** Migrate:
```bash
terraform init -migrate-state
```

Output:
```
Initializing the backend...
Terraform detected that the backend type changed from "local" to "s3".

Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "local" backend to the
  newly configured "s3" backend. No existing state was found in the newly
  configured "s3" backend. Do you want to copy this state to the new "s3"
  backend? Enter "yes" to copy and "no" to start with an empty state.

  Enter a value: yes

Successfully configured the backend "s3"!
```

### Lab: Simulated Remote Backend

Since we can't create real S3 buckets, we'll use the `local` backend explicitly to understand configuration:

```hcl
terraform {
  backend "local" {
    path = "custom-state-location/terraform.tfstate"
  }
}

resource "random_pet" "backend_example" {
  length = 2
}
```

```bash
terraform init
terraform apply

# State is now in custom location
cat custom-state-location/terraform.tfstate
```

Change backend path:
```hcl
terraform {
  backend "local" {
    path = "another-location/terraform.tfstate"
  }
}
```

```bash
terraform init -migrate-state
# Moves state to new location

ls custom-state-location/
# Old location still has file

ls another-location/
# New location has state
```

---

## 6d: Drift and State Management

### What is Drift?

Drift = when real infrastructure doesn't match state:

```
State says: instance type = t2.micro
Reality:     instance type = t2.large (someone changed it manually)
```

### Detecting Drift

```bash
terraform plan
```

Output:
```
Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since the
last "terraform apply":

  # random_pet.example has changed
  ~ resource "random_pet" "example" {
        id        = "cool-horse"
      ~ length    = 2 -> 3
    }

This is a refresh-only plan, so Terraform will not take any actions to undo
these. If you'd like to update the Terraform state to match, run:
  terraform apply -refresh-only
```

### Refresh-Only Mode

Update state without changing infrastructure:

```bash
terraform apply -refresh-only
```

This updates state to match reality.

### Moved Blocks

Refactor without recreating:

```hcl
# Old name
resource "random_pet" "old_name" {
  length = 2
}

# Add moved block
moved {
  from = random_pet.old_name
  to   = random_pet.new_name
}

# New name
resource "random_pet" "new_name" {
  length = 2
}
```

Terraform updates state references without recreating the resource.

### Removed Blocks

Remove from Terraform without destroying:

```hcl
removed {
  from = random_pet.example
  
  lifecycle {
    destroy = false
  }
}
```

Resource removed from state but remains in cloud.

### Lab: Drift Management

**Step 1:** Create resource:
```hcl
resource "random_pet" "drift_example" {
  length = 2
}

output "current_name" {
  value = random_pet.drift_example.id
}
```

```bash
terraform apply
```

**Step 2:** Check state:
```bash
terraform state show random_pet.drift_example
# Shows length = 2
```

**Step 3:** Manually modify state (simulating drift):
```bash
# DON'T DO THIS IN REAL LIFE
# This is just to simulate external changes
terraform state rm random_pet.drift_example
terraform import random_pet.drift_example $(terraform output -raw current_name)
```

**Step 4:** Modify config (simulate external change):
```hcl
resource "random_pet" "drift_example" {
  length = 3  # Changed!
}
```

**Step 5:** Detect drift:
```bash
terraform plan
# Shows: will replace resource (length changed)
```

**Step 6:** Use moved block:
```hcl
moved {
  from = random_pet.drift_example
  to   = random_pet.renamed_example
}

resource "random_pet" "renamed_example" {
  length = 3
}
```

```bash
terraform plan
# Shows: will move, not recreate
```

---

# Section 7: Maintaining Infrastructure

## 7a: Importing Resources

### The Import Process

1. Write resource block (empty or partial)
2. Run terraform import
3. State updated
4. Run terraform plan to see missing attributes
5. Add those attributes to config
6. Run plan again - should show no changes

### Basic Import

```bash
terraform import <resource_address> <resource_id>
```

**Example:**
```hcl
# 1. Write resource block first
resource "random_pet" "imported" {
  # Leave empty for now
}
```

```bash
# 2. Import (using existing pet name)
terraform import random_pet.imported "existing-happy-shark"

# 3. Check what's needed
terraform plan
```

Output shows missing attributes:
```
  ~ resource "random_pet" "imported" {
        id        = "existing-happy-shark"
      + length    = 2
      + separator = "-"
    }
```

```hcl
# 4. Add missing attributes
resource "random_pet" "imported" {
  length    = 2
  separator = "-"
}
```

```bash
# 5. Verify
terraform plan
# Should show: No changes
```

### Import Block (Terraform 1.5+)

Newer syntax that can generate config:

```hcl
import {
  to = random_pet.new_import
  id = "brave-lion"
}

resource "random_pet" "new_import" {
  # Terraform can generate this
}
```

```bash
terraform plan -generate-config-out=generated.tf
# Creates generated.tf with full resource config
```

### Lab: Import Workflow

**Step 1:** Create a resource outside Terraform:
```hcl
# temp.tf - we'll delete this
resource "random_pet" "to_import_later" {
  length    = 3
  separator = "_"
  prefix    = "import"
}
```

```bash
terraform apply
terraform output
# Note the pet name, e.g., "import_brave_lion_shark"
```

**Step 2:** Remove from Terraform (but keep resource):
```bash
# Remove from state
terraform state rm random_pet.to_import_later

# Delete temp.tf
rm temp.tf

terraform state list
# Resource gone from state, but still exists
```

**Step 3:** Import it back:
```hcl
# main.tf
resource "random_pet" "imported_pet" {
  # Empty for now
}
```

```bash
# Import using the name from Step 1
terraform import random_pet.imported_pet "import_brave_lion_shark"
```

**Step 4:** Fill in attributes:
```bash
terraform plan
# Shows what attributes are missing/different
```

```hcl
resource "random_pet" "imported_pet" {
  length    = 3
  separator = "_"
  prefix    = "import"
}
```

```bash
terraform plan
# No changes - import complete!
```

---

## 7b: State Inspection

### List Resources

```bash
terraform state list
```

Output:
```
random_integer.port
random_password.secret
random_pet.server
```

### Show Resource Details

```bash
terraform state show random_pet.server
```

Output:
```
# random_pet.server:
resource "random_pet" "server" {
    id        = "happy-shark"
    length    = 2
    separator = "-"
}
```

### Move Resources

Rename in state:
```bash
terraform state mv random_pet.old_name random_pet.new_name
```

Move to module:
```bash
terraform state mv random_pet.server module.web.random_pet.server
```

### Remove from State

```bash
terraform state rm random_pet.server
# Removes from state, resource still exists
```

### Pull/Push State

Backup:
```bash
terraform state pull > backup.tfstate
```

Restore (dangerous!):
```bash
terraform state push backup.tfstate
```

### Lab: State Manipulation

```hcl
resource "random_pet" "pets" {
  count  = 3
  length = 2
}

resource "random_password" "password" {
  length = 16
}

resource "random_integer" "number" {
  min = 1
  max = 100
}
```

```bash
terraform apply
```

**List everything:**
```bash
terraform state list
# random_integer.number
# random_password.password
# random_pet.pets[0]
# random_pet.pets[1]
# random_pet.pets[2]
```

**Show specific resource:**
```bash
terraform state show random_pet.pets[0]
```

**Move resource:**
```bash
terraform state mv random_pet.pets[0] random_pet.first_pet
terraform state list
# random_pet.first_pet (no longer [0])
# random_pet.pets[1]
# random_pet.pets[2]
```

**Remove resource:**
```bash
terraform state rm random_integer.number
terraform state list
# random_integer.number is gone
```

**Verify resource still exists:**
```bash
# It's not in state, but if we added it back to config:
resource "random_integer" "number" {
  min = 1
  max = 100
}

terraform plan
# Shows: will create (Terraform doesn't know it exists)
```

**Pull state for backup:**
```bash
terraform state pull > my-backup.json
cat my-backup.json
# Full state in JSON
```

---

## 7c: Logging and Debugging

### Enable Logging

```bash
export TF_LOG=TRACE
terraform apply
```

Log levels (least to most verbose):
- ERROR
- WARN
- INFO
- DEBUG
- TRACE

### Log to File

```bash
export TF_LOG=DEBUG
export TF_LOG_PATH=./terraform-debug.log
terraform apply

cat terraform-debug.log
```

### What Logging Shows

**TRACE level shows:**
- Every API call
- Request/response bodies
- Internal operations
- Provider communication

**Example output:**
```
2024/01/15 10:30:45 [TRACE] plugin.terraform-provider-random_v3.5.1_x5: 
HTTP Request: POST /create
2024/01/15 10:30:45 [TRACE] plugin.terraform-provider-random_v3.5.1_x5: 
HTTP Response: 200 OK
```

### Disable Logging

```bash
unset TF_LOG
unset TF_LOG_PATH
```

### Lab: Debug a Problem

Create a problematic config:
```hcl
variable "length" {
  type = number
}

resource "random_pet" "test" {
  length = var.length
}

# Intentional reference to non-existent resource
resource "random_password" "broken" {
  length = random_integer.doesnt_exist.result
}
```

**Enable debugging:**
```bash
export TF_LOG=DEBUG
export TF_LOG_PATH=debug.log

terraform init
terraform plan 2>&1 | tee plan-output.txt
# Error: Reference to undeclared resource
```

**Check logs:**
```bash
cat debug.log | grep -A5 -B5 "doesnt_exist"
# See detailed information about the error
```

**Fix and re-run:**
```hcl
resource "random_integer" "size" {
  min = 12
  max = 20
}

resource "random_password" "fixed" {
  length = random_integer.size.result
}
```

```bash
terraform plan
# Success
```

**Disable logging:**
```bash
unset TF_LOG
unset TF_LOG_PATH
```

---

# Section 8: HCP Terraform

## 8a: HCP Terraform Basics

### What is HCP Terraform?

HCP Terraform (formerly Terraform Cloud) is HashiCorp's SaaS platform for:
- Remote state storage
- Remote execution
- Team collaboration
- VCS integration
- Policy enforcement

### Key Concept: Workspaces

**CLI workspaces** (local):
- Multiple state files from same config
- Switch between them: `terraform workspace select prod`

**HCP workspaces:**
- Complete environment
- Has: config, state, variables, settings, permissions
- Like a project

**They are NOT the same!**

### Remote Execution

**Without HCP:**
```
Your laptop → API → Cloud Provider
```

**With HCP:**
```
Your laptop → HCP Terraform → API → Cloud Provider
```

Benefits:
- Consistent environment
- No credential management on laptops
- Centralized logs
- Team visibility

### VCS-Driven Workflow

```
Git push → HCP Terraform sees change → Runs plan → Team reviews → Apply
```

### Lab: Understanding HCP Concepts

We'll simulate the workflow:

```hcl
# Simulating HCP workspace structure
locals {
  workspace_name = "my-app-prod"
  organization   = "my-company"
  
  # Variables that would be in HCP workspace
  workspace_vars = {
    environment = "production"
    region      = "us-west-2"
  }
}

resource "random_pet" "app" {
  length = 2
  prefix = "${local.workspace_vars["environment"]}-server"
}

output "deployment" {
  value = {
    workspace = local.workspace_name
    org       = local.organization
    server    = random_pet.app.id
  }
}
```

This simulates how HCP workspaces inject variables.

---

## 8b: Collaboration Features

### Policy as Code (Sentinel)

Enforce rules before apply:

```hcl
# Sentinel policy (pseudocode)
policy "require-encryption" {
  rule = all s3_buckets have encryption enabled
  enforcement = "hard-mandatory"  # Cannot override
}

policy "cost-limit" {
  rule = estimated_cost < 1000
  enforcement = "advisory"  # Can override
}
```

### Cost Estimation

Before apply, HCP shows:

```
Plan: 5 to add, 2 to change, 1 to destroy

Cost Estimate:
  +$234.56/mo      New resources
  ~$  45.23/mo     Changed resources
  -$ 178.00/mo     Destroyed resources
  ───────────────
  $101.79/mo       New total
  +$ 45.56/mo      Delta
```

### Drift Detection

Scheduled runs check for changes:

```
Last successful run: 2 days ago
Drift detected: Yes
  - Resource modified outside Terraform
  - 3 resources have changed
```

### Private Module Registry

Share modules within organization:

```hcl
module "vpc" {
  source  = "app.terraform.io/my-org/vpc/aws"
  version = "1.0.0"
  
  cidr = "10.0.0.0/16"
}
```

### Run Triggers

Chain workspaces:

```
Workspace "network" completes
  ↓ Triggers
Workspace "compute"
  ↓ Triggers
Workspace "monitoring"
```

---

## 8c: Workspaces and Projects

### Projects

Group related workspaces:

```
Project: "MyApp Production"
├── Workspace: prod-network
├── Workspace: prod-compute
├── Workspace: prod-database
└── Workspace: prod-monitoring
```

### Workspace Organization

By environment:
```
- dev-myapp
- staging-myapp
- prod-myapp
```

By component:
```
- myapp-network
- myapp-compute
- myapp-database
```

By region:
```
- myapp-us-west
- myapp-us-east
- myapp-eu-central
```

### Variable Sets

Share variables across workspaces:

```
Variable Set: "AWS Production Credentials"
Applied to: All workspaces in "Production" project

Variables:
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- AWS_DEFAULT_REGION=us-west-2
```

### Run Triggers Configuration

```
Source Workspace: network
↓ Triggers
Destination Workspace: compute

Condition: On successful apply
```

---

## 8d: CLI Integration

### Cloud Block

Configure HCP in your code:

```hcl
terraform {
  cloud {
    organization = "my-company"
    
    workspaces {
      name = "my-app-prod"
    }
  }
}

resource "random_pet" "server" {
  length = 2
}
```

### Authentication

```bash
terraform login
# Opens browser for authentication
# Saves token to ~/.terraform.d/credentials.tfrc.json
```

### Remote Operations

After configuration:

```bash
terraform init
# Initializes HCP backend

terraform plan
# Runs in HCP, streams output to terminal

terraform apply
# Runs in HCP, asks for confirmation locally
```

### Migrate to HCP

**Step 1:** Local state:
```hcl
resource "random_pet" "example" {
  length = 2
}
```

```bash
terraform init
terraform apply
# State in local file
```

**Step 2:** Add cloud block:
```hcl
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "my-workspace"
    }
  }
}

resource "random_pet" "example" {
  length = 2
}
```

**Step 3:** Migrate:
```bash
terraform init
```

Output:
```
Initializing Terraform Cloud...

Terraform Cloud has been successfully initialized!

Do you wish to proceed?
  As part of migrating to Terraform Cloud, Terraform can optionally copy your
  current workspace state to the configured Terraform Cloud workspace.
  
  Enter a value: yes

Terraform Cloud has been successfully initialized!
```

State now in HCP instead of local file.

---

## Final Summary

You now have complete study material covering all 37 exam topics:

**Section 1-3:** Core concepts, fundamentals, and workflow
**Section 4:** Configuration (resources, variables, expressions)
**Section 5:** Modules (sources, scope, usage, versioning)
**Section 6:** State management (backends, locking, drift)
**Section 7:** Maintenance (import, inspection, debugging)
**Section 8:** HCP Terraform (workspaces, collaboration, integration)

Each topic includes:
- Clear explanations
- Working code examples
- Hands-on labs you can run

**Next steps:**
1. Work through each lab
2. Type out the code (don't copy-paste)
3. Experiment with modifications
4. Move to next topic when comfortable

Good luck on your exam! 🎯
