# Terraform Associate (004) Exam Study Guide

> **Complete study guide for the HashiCorp Certified: Terraform Associate certification exam**
> 
> This guide covers all 8 sections and 37 topics from the official exam objectives, with depth guidance to help you study efficiently.

---

## 📚 How to Use This Guide

- **✅ MUST KNOW**: Study deeply - core exam content
- **⚠️ SHOULD KNOW**: Understand the concept, don't memorize details  
- **❌ SKIP**: Too detailed for the exam - ignore for now
- **⏱️ Time Budget**: Recommended study time per topic
- **🎯 Key Test Question**: If you can answer this, you're ready to move on

**Study Strategy**: The exam rewards breadth (knowing all 37 topics) over depth (mastering a few). Focus on Must Know items, understand Should Know concepts, and skip the rest until after you pass.

---

## 📑 Table of Contents

### [Section 1: Infrastructure as Code (IaC) with Terraform](#section-1-infrastructure-as-code-iac-with-terraform-1)
- [1a: Explain what IaC is](#1a-explain-what-iac-is)
- [1b: Describe the advantages of IaC patterns](#1b-describe-the-advantages-of-iac-patterns)
- [1c: Explain how Terraform manages multi-cloud, hybrid cloud, and service-agnostic workflows](#1c-explain-how-terraform-manages-multi-cloud-hybrid-cloud-and-service-agnostic-workflows)

### [Section 2: Terraform fundamentals](#section-2-terraform-fundamentals-1)
- [2a: Install and version Terraform providers](#2a-install-and-version-terraform-providers)
- [2b: Describe how Terraform uses providers](#2b-describe-how-terraform-uses-providers)
- [2c: Write Terraform configuration using multiple providers](#2c-write-terraform-configuration-using-multiple-providers)
- [2d: Explain how Terraform uses and manages state](#2d-explain-how-terraform-uses-and-manages-state)

### [Section 3: Core Terraform workflow](#section-3-core-terraform-workflow-1)
- [3a: Describe the Terraform workflow](#3a-describe-the-terraform-workflow)
- [3b: Initialize a Terraform working directory](#3b-initialize-a-terraform-working-directory)
- [3c: Validate a Terraform configuration](#3c-validate-a-terraform-configuration)
- [3d: Generate and review an execution plan for Terraform](#3d-generate-and-review-an-execution-plan-for-terraform)
- [3e: Apply changes to infrastructure with Terraform](#3e-apply-changes-to-infrastructure-with-terraform)
- [3f: Destroy Terraform-managed infrastructure](#3f-destroy-terraform-managed-infrastructure)
- [3g: Apply formatting and style adjustments to a configuration](#3g-apply-formatting-and-style-adjustments-to-a-configuration)

### [Section 4: Terraform configuration](#section-4-terraform-configuration-1)
- [4a: Use and differentiate resource and data blocks](#4a-use-and-differentiate-resource-and-data-blocks)
- [4b: Refer to resource attributes and create cross-resource references](#4b-refer-to-resource-attributes-and-create-cross-resource-references)
- [4c: Use variables and outputs](#4c-use-variables-and-outputs)
- [4d: Understand and use complex types](#4d-understand-and-use-complex-types)
- [4e: Write dynamic configuration using expressions and functions](#4e-write-dynamic-configuration-using-expressions-and-functions)
- [4f: Define resource dependencies in configuration](#4f-define-resource-dependencies-in-configuration)
- [4g: Validate configuration using custom conditions](#4g-validate-configuration-using-custom-conditions)
- [4h: Understand best practices for managing sensitive data, including secrets management with Vault](#4h-understand-best-practices-for-managing-sensitive-data-including-secrets-management-with-vault)

### [Section 5: Terraform modules](#section-5-terraform-modules-1)
- [5a: Explain how Terraform sources modules](#5a-explain-how-terraform-sources-modules)
- [5b: Describe variable scope within modules](#5b-describe-variable-scope-within-modules)
- [5c: Use modules in configuration](#5c-use-modules-in-configuration)
- [5d: Manage module versions](#5d-manage-module-versions)

### [Section 6: Terraform state management](#section-6-terraform-state-management-1)
- [6a: Describe the local backend](#6a-describe-the-local-backend)
- [6b: Describe state locking](#6b-describe-state-locking)
- [6c: Configure remote state using the backend block](#6c-configure-remote-state-using-the-backend-block)
- [6d: Manage resource drift and Terraform state](#6d-manage-resource-drift-and-terraform-state)

### [Section 7: Maintain infrastructure with Terraform](#section-7-maintain-infrastructure-with-terraform-1)
- [7a: Import existing infrastructure into your Terraform workspace](#7a-import-existing-infrastructure-into-your-terraform-workspace)
- [7b: Use the CLI to inspect state](#7b-use-the-cli-to-inspect-state)
- [7c: Describe when and how to use verbose logging](#7c-describe-when-and-how-to-use-verbose-logging)

### [Section 8: HCP Terraform](#section-8-hcp-terraform-1)
- [8a: Use HCP Terraform to create infrastructure](#8a-use-hcp-terraform-to-create-infrastructure)
- [8b: Describe HCP Terraform collaboration and governance features](#8b-describe-hcp-terraform-collaboration-and-governance-features)
- [8c: Describe how to organize and use HCP Terraform workspaces and projects](#8c-describe-how-to-organize-and-use-hcp-terraform-workspaces-and-projects)
- [8d: Configure and use HCP Terraform integration](#8d-configure-and-use-hcp-terraform-integration)

---

## Section 1: Infrastructure as Code (IaC) with Terraform

### 1a: Explain what IaC is

**⏱️ Time Budget: 10 minutes**

#### ✅ MUST KNOW

- **Infrastructure as Code = managing infrastructure through code/config files**
  - Instead of clicking in consoles or running manual commands, you write code that describes your infrastructure
  - This code is then executed to create, modify, or destroy infrastructure
  
- **Infrastructure definitions are versioned and repeatable**
  - Store your infrastructure code in Git alongside application code
  - Same code produces same infrastructure every time
  - Can track who changed what and when
  
- **IaC replaces manual configuration through UIs or scripts**
  - Traditional: log into AWS console, click to create EC2 instance, configure settings manually
  - IaC: write `resource "aws_instance" "web" {...}`, run terraform apply

#### ⚠️ SHOULD KNOW

- **IaC enables automation and consistency**
  - Deploy infrastructure automatically as part of CI/CD
  - Every environment (dev/staging/prod) built the same way
  
- **Infrastructure changes are trackable in version control**
  - Pull requests for infrastructure changes
  - Code reviews before applying changes
  - Rollback by reverting commits

#### ❌ SKIP FOR EXAM

- History of infrastructure management evolution
- Detailed comparison with traditional approaches
- Non-Terraform IaC tools' specifics

#### 🎯 Key Test Question

**What does Infrastructure as Code mean?**

If you can explain that IaC is managing infrastructure through code files (rather than manual processes) and that this enables version control and repeatability, you're ready.

---

### 1b: Describe the advantages of IaC patterns

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **Automation: deploy infrastructure consistently without manual steps**
  - One command (`terraform apply`) creates entire infrastructure
  - No forgetting steps or making typos in console
  - Can deploy 10 environments as easily as 1
  
- **Version control: track changes, rollback, collaboration**
  - Every infrastructure change is committed to Git
  - See who changed what and why (commit messages)
  - Roll back bad changes by reverting commits
  - Multiple team members can propose changes via pull requests
  
- **Consistency: same config produces same infrastructure**
  - Dev environment matches production exactly
  - No "configuration drift" between environments
  - New team members spin up identical environments
  
- **Reusability: templates/modules can be shared and reused**
  - Write a VPC module once, use it everywhere
  - Share modules across teams
  - Don't reinvent the wheel for common patterns

#### ⚠️ SHOULD KNOW

- **Documentation benefit - code IS the documentation**
  - Want to know how production is configured? Read the code
  - No separate docs that get outdated
  
- **Faster provisioning vs manual approaches**
  - Minutes instead of days/weeks
  - Parallel resource creation
  
- **Reduced human error**
  - Computer executes code the same way every time
  - No typos, no skipped steps

#### ❌ SKIP FOR EXAM

- ROI calculations and cost analyses
- Organizational change management details
- Specific case studies or company examples

#### 🎯 Key Test Question

**Name 3 key advantages of using IaC over manual infrastructure management**

Good answers: automation, version control, consistency, reusability, speed, reduced errors. If you can explain why any 3 of these matter, you're ready.

---

### 1c: Explain how Terraform manages multi-cloud, hybrid cloud, and service-agnostic workflows

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **Multi-cloud: same tool works across AWS, Azure, GCP, etc.**
  - Write Terraform code for AWS, Azure, GCP with same workflow
  - One language (HCL) for all providers
  - Can manage resources across multiple clouds in same config
  
- **Provider-agnostic: uses providers to abstract different APIs**
  - Terraform doesn't talk directly to cloud APIs
  - Providers are plugins that translate Terraform → cloud API
  - AWS provider, Azure provider, GCP provider, etc.
  
- **Single workflow for all infrastructure (cloud + on-prem + SaaS)**
  - Same commands work everywhere: init, plan, apply
  - Manage AWS + GitHub + Datadog + PagerDuty in one config
  
- **Terraform Registry has 3000+ providers**
  - Cloud providers (AWS, Azure, GCP)
  - SaaS services (GitHub, Datadog, Cloudflare)
  - Infrastructure tools (Kubernetes, Docker)
  - Can even manage physical hardware via custom providers

#### ⚠️ SHOULD KNOW

- **Hybrid cloud means mixing on-prem and cloud**
  - Manage VMware VMs and AWS EC2 with same tool
  - Connect on-prem network to cloud VPC
  
- **Service-agnostic means not locked to one vendor**
  - Switch from AWS to Azure by changing provider
  - Same concepts (resources, variables, state) across all providers
  
- **Same HCL syntax regardless of provider**
  - `resource "aws_instance"` and `resource "azurerm_virtual_machine"` use same structure
  - Learn Terraform once, use everywhere

#### ❌ SKIP FOR EXAM

- Specific multi-cloud architecture patterns
- Provider development details
- Deep comparison with cloud-specific tools

#### 🎯 Key Test Question

**How does Terraform support managing infrastructure across AWS, Azure, and on-premises?**

Good answer: Terraform uses providers (plugins) that translate its declarative config into API calls for each platform. Same workflow (init/plan/apply) works everywhere, with 3000+ providers for different clouds, services, and tools.

---

## Section 2: Terraform fundamentals

### 2a: Install and version Terraform providers

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **Providers installed via terraform init**
  ```hcl
  # After writing config with providers, run:
  terraform init
  # This downloads necessary provider plugins
  ```
  
- **required_providers block specifies provider source and version**
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
  
- **Dependency lock file (.terraform.lock.hcl) pins exact versions**
  - Created automatically during `terraform init`
  - Records exact provider versions used
  - Ensures team uses same provider versions
  - **Commit this file to version control**
  
- **Version constraints: ~>, >=, =, <**
  - `~> 5.0` means >= 5.0 and < 6.0 (allows 5.1, 5.2, but not 6.0)
  - `>= 5.0` means 5.0 or higher
  - `= 5.0.0` means exactly 5.0.0
  - `< 6.0` means anything below 6.0

#### ⚠️ SHOULD KNOW

- **terraform init -upgrade to update providers**
  - Respects version constraints in required_providers
  - Updates lock file with new versions
  
- **Lock file should be committed to version control**
  - Prevents "works on my machine" issues
  - Everyone gets same provider versions
  
- **Provider versions affect available resources**
  - New provider version = new resources and features
  - Breaking changes between major versions

#### ❌ SKIP FOR EXAM

- Manual provider installation methods
- Provider plugin cache configuration
- Custom provider installation paths
- Provider signing and verification details

#### 🎯 Key Test Question

**How do you specify and install a specific version of the AWS provider?**

Good answer: Use `required_providers` block with version constraint, then run `terraform init`. The lock file records the exact version. Example: `version = "~> 5.0"` allows 5.x releases.

---

### 2b: Describe how Terraform uses providers

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **Providers are plugins that interact with APIs**
  - Provider = bridge between Terraform and service API
  - AWS provider talks to AWS API
  - GitHub provider talks to GitHub API
  
- **Each provider exposes resources and data sources**
  ```hcl
  # AWS provider gives you:
  resource "aws_instance" "web" { ... }
  data "aws_ami" "ubuntu" { ... }
  ```
  
- **Provider block configures authentication/settings**
  ```hcl
  provider "aws" {
    region = "us-west-2"
    # Authentication via env vars or credentials file
  }
  ```
  
- **Terraform downloads providers during init**
  - Reads required_providers
  - Downloads from Terraform Registry
  - Stores in .terraform directory

#### ⚠️ SHOULD KNOW

- **Providers translate HCL to API calls**
  - You write: `resource "aws_instance" "web" { ... }`
  - Provider makes: AWS EC2 API call to create instance
  
- **Multiple provider instances with alias**
  ```hcl
  provider "aws" {
    alias  = "west"
    region = "us-west-1"
  }
  
  provider "aws" {
    alias  = "east"
    region = "us-east-1"
  }
  ```
  
- **Default provider selection rules**
  - Resource uses default provider unless specified
  - `provider = aws.west` to use aliased provider

#### ❌ SKIP FOR EXAM

- Provider plugin protocol details
- Provider development internals
- How providers are compiled

#### 🎯 Key Test Question

**What is a provider in Terraform and what does it do?**

Good answer: A provider is a plugin that translates Terraform configuration into API calls for a specific service (AWS, Azure, GitHub, etc.). Each provider exposes resources and data sources you can use in your config.

---

### 2c: Write Terraform configuration using multiple providers

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **Can use multiple different providers in one config (AWS + Azure)**
  ```hcl
  terraform {
    required_providers {
      aws = { source = "hashicorp/aws", version = "~> 5.0" }
      azurerm = { source = "hashicorp/azurerm", version = "~> 3.0" }
    }
  }
  
  resource "aws_instance" "web" { ... }
  resource "azurerm_virtual_machine" "db" { ... }
  ```
  
- **Can use multiple instances of same provider with alias**
  ```hcl
  provider "aws" {
    alias  = "west"
    region = "us-west-2"
  }
  
  provider "aws" {
    alias  = "east"
    region = "us-east-1"
  }
  
  resource "aws_instance" "west_server" {
    provider = aws.west
    # ...
  }
  
  resource "aws_instance" "east_server" {
    provider = aws.east
    # ...
  }
  ```
  
- **Each resource references its provider explicitly or by default**
  - No `provider` argument = uses default provider
  - `provider = aws.west` = uses that specific provider
  
- **Basic syntax: provider "aws" { alias = "west" }**

#### ⚠️ SHOULD KNOW

- **Use case: multi-region deployments**
  - Create resources in multiple AWS regions
  - Use alias to distinguish regions
  
- **Use case: different cloud providers together**
  - AWS for compute + Cloudflare for DNS
  - Azure for storage + Datadog for monitoring
  
- **provider argument in resource blocks**
  ```hcl
  resource "aws_instance" "example" {
    provider = aws.west  # Use specific provider instance
    # ...
  }
  ```

#### ❌ SKIP FOR EXAM

- Complex multi-provider patterns
- Provider meta-arguments details
- Cross-provider data sharing strategies

#### 🎯 Key Test Question

**How would you configure resources in both us-east-1 and us-west-2 regions?**

Good answer: Create two AWS provider blocks with different aliases and regions. Then use the `provider` argument in resources to specify which one to use.

---

### 2d: Explain how Terraform uses and manages state

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **State maps config to real infrastructure**
  - Config says: "I want an EC2 instance"
  - State says: "Instance i-1234567 exists and matches this config"
  - Terraform compares them to decide what to change
  
- **Stored in terraform.tfstate (JSON file)**
  - Created automatically after first `terraform apply`
  - Updated after every apply
  - **DO NOT edit manually**
  
- **Tracks resource attributes and metadata**
  ```json
  {
    "resources": [{
      "type": "aws_instance",
      "name": "web",
      "provider": "...",
      "instances": [{
        "attributes": {
          "id": "i-1234567",
          "ami": "ami-abc123",
          "public_ip": "54.1.2.3"
        }
      }]
    }]
  }
  ```
  
- **Used to determine what changes to make**
  - Plan: compares state (current) vs config (desired)
  - Determines: create new, modify existing, delete removed
  
- **State is source of truth for what Terraform manages**
  - Resource in state = Terraform manages it
  - Resource not in state = Terraform doesn't know about it

#### ⚠️ SHOULD KNOW

- **Performance benefit - caches resource attributes**
  - Don't have to query API for every resource on every plan
  - Large infrastructure plans faster
  
- **State enables dependency tracking**
  - Records relationships between resources
  - Ensures correct creation/deletion order
  
- **State contains sensitive data**
  - Passwords, keys, secrets stored in plaintext
  - Must protect state file access

#### ❌ SKIP FOR EXAM

- State file JSON structure details
- State format versioning
- State storage backends (covered in Section 6)

#### 🎯 Key Test Question

**What is Terraform state and why is it needed?**

Good answer: State is a file that maps your configuration to real infrastructure. It tracks which resources exist, their attributes, and relationships. Terraform uses it to determine what changes to make (create/modify/destroy) when you run plan or apply.

---

## Section 3: Core Terraform workflow

### 3a: Describe the Terraform workflow

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **Write → Plan → Apply core workflow**
  1. **Write:** Author `.tf` files describing desired infrastructure
  2. **Plan:** Preview changes before applying (`terraform plan`)
  3. **Apply:** Execute changes to create/modify infrastructure (`terraform apply`)
  
- **Write: author infrastructure config in .tf files**
  ```hcl
  # main.tf
  resource "aws_instance" "web" {
    ami           = "ami-123456"
    instance_type = "t2.micro"
  }
  ```
  
- **Plan: preview changes before applying**
  - Shows what will be created (+), modified (~), or destroyed (-)
  - No changes made to infrastructure
  - Catch mistakes before they happen
  
- **Apply: execute changes to match desired state**
  - Actually creates/modifies/destroys resources
  - Updates state file
  - Requires confirmation (unless -auto-approve)
  
- **Optional: Destroy when infrastructure no longer needed**
  - `terraform destroy` removes all managed infrastructure
  - Also requires confirmation

#### ⚠️ SHOULD KNOW

- **Workflow is iterative - make changes and replan**
  - Make code change → run plan → review → apply
  - Repeat as infrastructure evolves
  
- **Plan shows +create, ~modify, -destroy**
  - `+` = new resource will be created
  - `~` = existing resource will be modified
  - `-` = resource will be destroyed
  - `-/+` = resource will be replaced (destroy then create)
  
- **Each step serves distinct purpose**
  - Write = design
  - Plan = review
  - Apply = execute

#### ❌ SKIP FOR EXAM

- Advanced workflow variations
- CI/CD pipeline integration details
- Team collaboration workflows

#### 🎯 Key Test Question

**What are the core steps in the Terraform workflow?**

Good answer: Write configuration, plan changes to see what will happen, apply changes to create/modify infrastructure. Optionally destroy when done.

---

### 3b: Initialize a Terraform working directory

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **terraform init - first command in new directory**
  ```bash
  cd my-terraform-project/
  terraform init
  ```
  
- **Downloads provider plugins**
  - Reads `required_providers` block
  - Downloads plugins from Terraform Registry
  - Stores in `.terraform/` directory
  
- **Initializes backend for state storage**
  - Sets up where state will be stored
  - Default = local file
  - Can configure remote backends
  
- **Downloads modules referenced in config**
  ```hcl
  module "vpc" {
    source = "terraform-aws-modules/vpc/aws"
  }
  # init downloads this module
  ```
  
- **Creates .terraform directory and lock file**
  - `.terraform/` = cached providers and modules
  - `.terraform.lock.hcl` = provider version lock file

#### ⚠️ SHOULD KNOW

- **Re-run init when: adding providers, changing backend, adding modules**
  - New provider in config → run init
  - Changed backend configuration → run init with -migrate-state
  - Added new module → run init
  
- **terraform init -upgrade updates providers**
  - Updates to latest version matching constraints
  - Updates lock file
  
- **Safe to run init multiple times**
  - Won't break anything
  - Use it when unsure

#### ❌ SKIP FOR EXAM

- Plugin cache mechanics and configuration
- Backend migration detailed process
- Custom plugin installation

#### 🎯 Key Test Question

**What does terraform init do and when should you run it?**

Good answer: init downloads provider plugins, initializes the backend, and downloads modules. Run it first in a new directory, and again whenever you add providers, change backends, or add modules.

---

### 3c: Validate a Terraform configuration

**⏱️ Time Budget: 10 minutes**

#### ✅ MUST KNOW

- **terraform validate checks syntax and logic**
  ```bash
  terraform validate
  # Output: Success! or specific errors
  ```
  
- **Validates without accessing remote services**
  - Doesn't need cloud credentials
  - Doesn't need network access
  - Fast check before plan
  
- **Checks: syntax errors, invalid references, required arguments**
  - HCL syntax correct?
  - Resource references valid?
  - Required arguments present?
  
- **Run after writing config to catch errors early**
  - Faster than waiting for plan to fail
  - Catch typos immediately

#### ⚠️ SHOULD KNOW

- **validate runs automatically during plan**
  - plan validates first, then builds execution plan
  - Explicit validate still useful for quick checks
  
- **Useful in CI/CD pipelines**
  - Fail fast on syntax errors
  - Before even trying to authenticate to cloud
  
- **Does NOT check if resources will actually work**
  - Doesn't verify AMI ID exists
  - Doesn't check if you have permissions
  - Only checks config structure

#### ❌ SKIP FOR EXAM

- Validation rule internals
- Custom validation plugins
- Deep validation logic

#### 🎯 Key Test Question

**What does terraform validate check for?**

Good answer: validate checks syntax errors, invalid resource references, and missing required arguments. It's a quick local check that doesn't require cloud credentials or network access.

---

### 3d: Generate and review an execution plan for Terraform

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **terraform plan shows what will change**
  ```bash
  terraform plan
  # Shows: X to add, Y to change, Z to destroy
  ```
  
- **Compares current state with desired config**
  - Reads state file (what exists)
  - Reads .tf files (what you want)
  - Calculates difference
  
- **+ means create, ~ means modify, - means destroy**
  ```
  + resource will be created
  ~ resource will be updated in-place
  - resource will be destroyed
  -/+ resource will be replaced (destroyed and recreated)
  ```
  
- **Can save plan to file with -out flag**
  ```bash
  terraform plan -out=tfplan
  terraform apply tfplan  # Execute saved plan
  ```
  
- **Plan does NOT make changes**
  - Safe to run anytime
  - Review before every apply

#### ⚠️ SHOULD KNOW

- **Plan shows dependency graph execution order**
  - Resources created in correct order
  - Dependencies respected
  
- **-target to plan specific resources**
  ```bash
  terraform plan -target=aws_instance.web
  ```
  
- **Plan includes resource attribute changes**
  - Shows old value → new value
  - (known after apply) for computed attributes

#### ❌ SKIP FOR EXAM

- Plan file binary format
- Graph visualization details
- Plan file encryption

#### 🎯 Key Test Question

**What does terraform plan do? What do +, ~, and - symbols mean?**

Good answer: plan shows what changes will be made without actually making them. + means create, ~ means modify in-place, - means destroy. It compares current state with desired configuration.

---

### 3e: Apply changes to infrastructure with Terraform

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **terraform apply executes planned changes**
  ```bash
  terraform apply
  # Shows plan, asks for confirmation
  # Type 'yes' to proceed
  ```
  
- **Creates/modifies/destroys real infrastructure**
  - Makes actual API calls to cloud providers
  - Changes are permanent
  
- **Shows plan and requires approval (yes/no)**
  - Displays execution plan first
  - Must type "yes" to confirm
  - Protects against accidental changes
  
- **Updates state file after completion**
  - Records what was created
  - Tracks resource attributes
  - State now matches reality
  
- **-auto-approve skips confirmation (use carefully)**
  ```bash
  terraform apply -auto-approve
  # No confirmation prompt - dangerous!
  ```

#### ⚠️ SHOULD KNOW

- **Can apply from saved plan file**
  ```bash
  terraform plan -out=tfplan
  terraform apply tfplan  # No confirmation needed with saved plan
  ```
  
- **Without plan file, generates new plan first**
  - Shows plan
  - Asks for confirmation
  - Then executes
  
- **Rollback on error depends on operation**
  - Some resources created before error → left in place
  - State updated with what succeeded
  - Re-run apply to continue

#### ❌ SKIP FOR EXAM

- Parallelism tuning (-parallelism flag)
- Apply internals and recovery mechanisms
- Partial state updates

#### 🎯 Key Test Question

**What happens when you run terraform apply?**

Good answer: apply shows an execution plan, asks for confirmation, then creates/modifies/destroys infrastructure as planned. After completion, it updates the state file to reflect the changes made.

---

### 3f: Destroy Terraform-managed infrastructure

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **terraform destroy removes all managed infrastructure**
  ```bash
  terraform destroy
  # Shows destruction plan
  # Type 'yes' to confirm
  ```
  
- **Shows destruction plan first**
  - Lists all resources that will be destroyed
  - Shows `-` next to each one
  
- **Requires confirmation**
  - Must type "yes" to proceed
  - Protects against accidental deletion
  
- **Updates state to reflect destruction**
  - State file cleared of destroyed resources
  - State remains (empty or with remaining resources)
  
- **Respects dependencies (destroys in correct order)**
  - Dependent resources destroyed first
  - No errors from destroying in wrong order

#### ⚠️ SHOULD KNOW

- **-target to destroy specific resources**
  ```bash
  terraform destroy -target=aws_instance.web
  # Only destroys specified resource
  ```
  
- **Equivalent to removing all resources and applying**
  - `destroy` = remove all resources from config + apply
  - Can also: remove resources from .tf files, run apply
  
- **Cannot be undone without reapplying**
  - Destruction is permanent
  - Must run apply again to recreate

#### ❌ SKIP FOR EXAM

- Destroy provisioners
- Partial destruction strategies
- Destroy optimization

#### 🎯 Key Test Question

**What does terraform destroy do?**

Good answer: destroy removes all Terraform-managed infrastructure. It shows a plan, requires confirmation, then destroys resources in the correct order (respecting dependencies) and updates the state file.

---

### 3g: Apply formatting and style adjustments to a configuration

**⏱️ Time Budget: 10 minutes**

#### ✅ MUST KNOW

- **terraform fmt auto-formats .tf files**
  ```bash
  terraform fmt
  # Formats all .tf files in current directory
  ```
  
- **Applies standard style conventions**
  - Consistent indentation
  - Aligned equals signs
  - Proper spacing
  
- **Updates files in place**
  - Modifies your .tf files directly
  - No backup created
  
- **Useful before committing code**
  - Ensures consistent style
  - Prevents formatting discussions in PR reviews

#### ⚠️ SHOULD KNOW

- **-check flag checks without modifying**
  ```bash
  terraform fmt -check
  # Exit code 0 = properly formatted
  # Exit code non-zero = needs formatting
  ```
  
- **-recursive for subdirectories**
  ```bash
  terraform fmt -recursive
  # Formats all .tf files in subdirectories too
  ```
  
- **Consistent formatting aids collaboration**
  - Team uses same style
  - Git diffs show real changes, not formatting

#### ❌ SKIP FOR EXAM

- Detailed formatting rules
- Custom style guides
- Formatter configuration options

#### 🎯 Key Test Question

**What does terraform fmt do?**

Good answer: fmt automatically formats Terraform configuration files according to standard style conventions. It updates files in place and is useful before committing code to ensure consistent formatting.

---

## Section 4: Terraform configuration

### 4a: Use and differentiate resource and data blocks

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **resource blocks CREATE infrastructure**
  ```hcl
  resource "aws_instance" "web" {
    ami           = "ami-123456"
    instance_type = "t2.micro"
  }
  # Creates an EC2 instance
  ```
  
- **data blocks READ existing infrastructure**
  ```hcl
  data "aws_ami" "ubuntu" {
    most_recent = true
    owners      = ["099720109477"]  # Canonical
    
    filter {
      name   = "name"
      values = ["ubuntu/images/hvm-ssd/ubuntu-focal-*"]
    }
  }
  # Queries for an existing AMI
  ```
  
- **resource "type" "name" { } syntax**
  - `type` = resource type from provider (e.g., aws_instance)
  - `name` = local name you choose (e.g., web)
  
- **data "type" "name" { } syntax**
  - Similar structure to resource
  - Used for querying, not creating
  
- **Reference: resource.type.name.attr vs data.type.name.attr**
  ```hcl
  # Reference resource
  resource.aws_instance.web.id
  
  # Reference data source
  data.aws_ami.ubuntu.id
  ```

#### ⚠️ SHOULD KNOW

- **Data sources query during plan**
  - Read at plan time
  - Used to fetch information for use in config
  
- **Resources have lifecycle (create/update/destroy)**
  - Managed by Terraform
  - State tracks them
  
- **Data sources are read-only**
  - Never create or modify infrastructure
  - Just query and return information

#### ❌ SKIP FOR EXAM

- Data source caching details
- Resource lifecycle meta-arguments deep dive
- Local-only data sources

#### 🎯 Key Test Question

**What's the difference between a resource and a data source?**

Good answer: Resources CREATE infrastructure (like EC2 instances). Data sources READ existing infrastructure (like querying for an AMI). Resources are managed by Terraform; data sources just fetch information.

---

### 4b: Refer to resource attributes and create cross-resource references

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **Reference syntax: resource_type.resource_name.attribute**
  ```hcl
  resource "aws_instance" "web" {
    ami           = "ami-123456"
    instance_type = "t2.micro"
  }
  
  resource "aws_eip" "web_ip" {
    instance = aws_instance.web.id  # Reference!
  }
  ```
  
- **References create implicit dependencies**
  - Terraform knows web must be created before web_ip
  - Builds dependency graph automatically
  - Ensures correct creation order
  
- **self for within provisioners/connections**
  ```hcl
  resource "aws_instance" "web" {
    # ...
    
    provisioner "local-exec" {
      command = "echo ${self.private_ip}"
      # self refers to this instance
    }
  }
  ```
  
- **Cross-resource references enable resource composition**
  - Pass VPC ID to subnet
  - Pass subnet ID to instance
  - Build complex infrastructure from simple pieces

#### ⚠️ SHOULD KNOW

- **Terraform builds dependency graph from references**
  - Analyzes all references
  - Determines creation order
  - Parallelizes when possible
  
- **count/for_each with indexing**
  ```hcl
  resource "aws_instance" "web" {
    count = 3
    # ...
  }
  
  # Reference with index
  aws_instance.web[0].id
  aws_instance.web[1].id
  ```
  
- **Module output references**
  ```hcl
  module.vpc.vpc_id
  ```

#### ❌ SKIP FOR EXAM

- Dependency graph algorithm
- Complex reference patterns
- Graph cycle detection

#### 🎯 Key Test Question

**How do you reference another resource's attribute? What effect does this have?**

Good answer: Use `resource_type.resource_name.attribute` syntax. For example, `aws_instance.web.id`. This creates an implicit dependency, so Terraform knows to create the referenced resource first.

---

### 4c: Use variables and outputs

**⏱️ Time Budget: 30 minutes**

#### ✅ MUST KNOW

- **variable block defines inputs**
  ```hcl
  variable "instance_type" {
    type        = string
    description = "EC2 instance type"
    default     = "t2.micro"
  }
  ```
  
- **output block exposes values**
  ```hcl
  output "instance_ip" {
    value       = aws_instance.web.public_ip
    description = "Public IP of the web server"
  }
  ```
  
- **Reference variables with var.name**
  ```hcl
  resource "aws_instance" "web" {
    instance_type = var.instance_type
  }
  ```
  
- **Variable types: string, number, bool, list, map, object**
  ```hcl
  variable "name" {
    type = string
  }
  
  variable "count" {
    type = number
  }
  
  variable "enabled" {
    type = bool
  }
  
  variable "azs" {
    type = list(string)
  }
  
  variable "tags" {
    type = map(string)
  }
  ```
  
- **Passing values: -var, -var-file, .tfvars, env vars, defaults**
  ```bash
  # Command line
  terraform apply -var="instance_type=t2.large"
  
  # Variable file
  terraform apply -var-file="prod.tfvars"
  
  # terraform.tfvars (auto-loaded)
  instance_type = "t2.large"
  
  # Environment variable
  export TF_VAR_instance_type="t2.large"
  
  # Default in variable definition
  variable "instance_type" {
    default = "t2.micro"
  }
  ```
  
- **Precedence order for variable values**
  1. Command line `-var` flags (highest)
  2. `-var-file` flags
  3. `terraform.tfvars` or `terraform.tfvars.json`
  4. `*.auto.tfvars` or `*.auto.tfvars.json` (alphabetical)
  5. Environment variables `TF_VAR_name`
  6. Default value in variable block (lowest)

#### ⚠️ SHOULD KNOW

- **description and default in variable blocks**
  - description: documents purpose
  - default: value if none provided
  
- **sensitive flag to hide values**
  ```hcl
  variable "db_password" {
    type      = string
    sensitive = true
  }
  
  output "password" {
    value     = var.db_password
    sensitive = true  # Won't display in output
  }
  ```
  
- **validation blocks for variables**
  ```hcl
  variable "instance_type" {
    type = string
    
    validation {
      condition     = can(regex("^t2\\.", var.instance_type))
      error_message = "Instance type must be t2 family."
    }
  }
  ```
  
- **terraform.tfvars auto-loaded**
  - No need for -var-file flag
  - Convenient for common values

#### ❌ SKIP FOR EXAM

- Complex variable validation rules
- Variable definition precedence edge cases
- Sensitive value handling internals

#### 🎯 Key Test Question

**How do you define and use a variable? What's the variable precedence order?**

Good answer: Define with `variable "name" { type, default, description }`, use with `var.name`. Precedence from highest to lowest: command line, var-file, terraform.tfvars, auto.tfvars, env vars, default value.

---

### 4d: Understand and use complex types

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **list: ordered collection ["a", "b", "c"]**
  ```hcl
  variable "availability_zones" {
    type    = list(string)
    default = ["us-west-2a", "us-west-2b"]
  }
  ```
  
- **map: key-value pairs {key1 = "value1"}**
  ```hcl
  variable "tags" {
    type = map(string)
    default = {
      Environment = "dev"
      Project     = "demo"
    }
  }
  ```
  
- **object: structured with named attributes**
  ```hcl
  variable "instance_config" {
    type = object({
      instance_type = string
      ami           = string
      count         = number
    })
  }
  ```
  
- **set: unordered unique collection**
  ```hcl
  variable "allowed_ips" {
    type = set(string)
    default = ["10.0.0.1", "10.0.0.2"]
  }
  ```

#### ⚠️ SHOULD KNOW

- **Type constraints in variable blocks**
  - Terraform validates input matches type
  - Fails early with clear error
  
- **Accessing list elements: var.list[0]**
  ```hcl
  var.availability_zones[0]  # First element
  ```
  
- **Accessing map values: var.map["key"]**
  ```hcl
  var.tags["Environment"]  # Gets "dev"
  ```
  
- **Object type definitions**
  - Specify structure upfront
  - Each attribute has its own type

#### ❌ SKIP FOR EXAM

- Deep type theory
- Complex nested type definitions
- Type conversion edge cases
- Tuple types

#### 🎯 Key Test Question

**What's the difference between a list and a map? How do you access their elements?**

Good answer: A list is an ordered collection accessed by index (`var.list[0]`). A map is key-value pairs accessed by key (`var.map["key"]`). Lists use brackets with numbers, maps use brackets with strings.

---

### 4e: Write dynamic configuration using expressions and functions

**⏱️ Time Budget: 25 minutes**

#### ✅ MUST KNOW

- **String interpolation: "${var.name}"**
  ```hcl
  resource "aws_instance" "web" {
    tags = {
      Name = "web-server-${var.environment}"
    }
  }
  ```
  
- **Conditionals: condition ? true_val : false_val**
  ```hcl
  instance_type = var.environment == "prod" ? "t2.large" : "t2.micro"
  ```
  
- **Common functions: file(), lookup(), concat(), join()**
  ```hcl
  # Read file
  user_data = file("script.sh")
  
  # Lookup with default
  instance_type = lookup(var.instance_types, var.environment, "t2.micro")
  
  # Concatenate lists
  all_ips = concat(var.private_ips, var.public_ips)
  
  # Join strings
  name = join("-", ["web", var.environment, "server"])
  ```
  
- **Functions called in expressions: function(arg1, arg2)**
  - Not like resources or variables
  - Called directly in expressions
  - Return values used inline

#### ⚠️ SHOULD KNOW

- **for expressions: [for item in list : item.value]**
  ```hcl
  # Transform list
  instance_names = [for i in var.instances : i.name]
  
  # Transform to map
  instance_map = {for i in var.instances : i.id => i.name}
  ```
  
- **Splat expressions: resource.*.attribute**
  ```hcl
  # Get IDs of all instances
  instance_ids = aws_instance.web[*].id
  ```
  
- **String functions: format, replace**
  ```hcl
  formatted = format("Server %02d", count.index)
  cleaned = replace(var.name, "_", "-")
  ```
  
- **Collection functions: merge, keys, values**
  ```hcl
  all_tags = merge(var.common_tags, var.specific_tags)
  tag_keys = keys(var.tags)
  ```
  
- **Cannot create custom functions**
  - Use locals for reusable expressions
  - Functions are built-in only

#### ❌ SKIP FOR EXAM

- Every built-in function
- Complex expression patterns
- Function implementation details
- Template expressions

#### 🎯 Key Test Question

**How do you conditionally set a value? How do you read a file into a variable?**

Good answer: Conditional: `condition ? true_val : false_val`. For example: `var.env == "prod" ? "t2.large" : "t2.small"`. Read file: `file("path/to/file")`.

---

### 4f: Define resource dependencies in configuration

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **Implicit dependencies: created by references**
  ```hcl
  resource "aws_instance" "web" {
    subnet_id = aws_subnet.main.id  # Implicit dependency
  }
  # Terraform knows to create subnet before instance
  ```
  
- **Explicit dependencies: depends_on meta-argument**
  ```hcl
  resource "aws_instance" "web" {
    # ...
    depends_on = [aws_iam_role_policy.example]
  }
  ```
  
- **depends_on = [resource.type.name]**
  - Takes list of resources
  - Creates explicit dependency
  
- **Prefer implicit over explicit**
  - Implicit is clearer (you see the reference)
  - Explicit is for hidden dependencies
  
- **Dependencies control creation/destruction order**
  - Creation: dependencies first
  - Destruction: dependents first

#### ⚠️ SHOULD KNOW

- **When depends_on is needed (hidden dependencies)**
  - Permission must exist before using it
  - Resource needed but not referenced
  - Example: IAM policy must exist before using the permissions
  
- **Module-level depends_on**
  ```hcl
  module "vpc" {
    # ...
    depends_on = [aws_iam_role.example]
  }
  ```
  
- **Terraform builds dependency graph**
  - Analyzes all dependencies
  - Creates execution plan
  - Parallelizes independent resources

#### ❌ SKIP FOR EXAM

- Graph algorithm details
- Circular dependency resolution
- Dependency optimization

#### 🎯 Key Test Question

**What's the difference between implicit and explicit dependencies? When would you use depends_on?**

Good answer: Implicit dependencies are created automatically when you reference one resource in another. Explicit dependencies use `depends_on` for cases where a dependency exists but isn't represented by a reference (like needing IAM permissions before using them).

---

### 4g: Validate configuration using custom conditions

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **validation blocks in variable definitions**
  ```hcl
  variable "ami_id" {
    type = string
    
    validation {
      condition     = length(var.ami_id) > 4 && substr(var.ami_id, 0, 4) == "ami-"
      error_message = "AMI ID must start with 'ami-'."
    }
  }
  ```
  
- **check blocks for infrastructure validation**
  ```hcl
  check "health_check" {
    data "http" "example" {
      url = "https://${aws_instance.web.public_ip}"
    }
    
    assert {
      condition     = data.http.example.status_code == 200
      error_message = "Website is not responding."
    }
  }
  ```
  
- **precondition/postcondition in resources**
  ```hcl
  resource "aws_instance" "web" {
    # ...
    
    lifecycle {
      precondition {
        condition     = var.ami_id != ""
        error_message = "AMI ID cannot be empty."
      }
    }
  }
  ```
  
- **Validation happens during plan**
  - Catches errors before apply
  - Fails fast with clear message

#### ⚠️ SHOULD KNOW

- **condition = expression that must be true**
  - Boolean expression
  - True = validation passes
  - False = shows error_message
  
- **error_message shown when validation fails**
  - Custom message explaining the problem
  - Helps users fix the issue
  
- **Checks vs validation blocks**
  - validation = for variable inputs
  - checks = for infrastructure state

#### ❌ SKIP FOR EXAM

- Complex validation logic
- Check block advanced features
- Custom validation functions

#### 🎯 Key Test Question

**How do you validate that a variable meets certain requirements?**

Good answer: Use a `validation` block inside the variable definition with a `condition` (boolean expression) and `error_message`. If the condition is false, Terraform fails with the error message.

---

### 4h: Understand best practices for managing sensitive data, including secrets management with Vault

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **State files contain ALL resource data (including secrets)**
  - Passwords, keys, connection strings stored in plaintext
  - Anyone with state file access can see secrets
  - This is by design - Terraform needs full resource info
  
- **sensitive = true in outputs/variables hides display**
  ```hcl
  variable "db_password" {
    type      = string
    sensitive = true
  }
  
  output "db_password" {
    value     = var.db_password
    sensitive = true  # Not shown in CLI output
  }
  ```
  
- **Never commit state files to version control**
  - Add terraform.tfstate to .gitignore
  - Add terraform.tfstate.backup too
  - Use remote state instead
  
- **Use remote state with encryption**
  - S3 with encryption at rest
  - Terraform Cloud with encrypted storage
  - Azure Storage with encryption
  
- **Vault provider can inject secrets dynamically**
  ```hcl
  data "vault_generic_secret" "db" {
    path = "secret/database"
  }
  
  resource "aws_db_instance" "example" {
    password = data.vault_generic_secret.db.data["password"]
  }
  ```

#### ⚠️ SHOULD KNOW

- **Environment variables for sensitive provider config**
  ```bash
  export AWS_ACCESS_KEY_ID="..."
  export AWS_SECRET_ACCESS_KEY="..."
  # Terraform reads these automatically
  ```
  
- **Vault as external secret store**
  - Secrets stored in Vault, not in code
  - Terraform fetches at runtime
  - Secrets never in state... wait, yes they are in state
  
- **Sensitive values still in state file**
  - sensitive flag only hides from display
  - Values still written to state
  - Protect state file access
  
- **State access control is critical**
  - Limit who can read state
  - Use IAM policies for S3 backend
  - Use RBAC in Terraform Cloud

#### ❌ SKIP FOR EXAM

- Vault installation and configuration
- Deep Vault provider usage
- Alternative secret management tools
- State encryption mechanisms

#### 🎯 Key Test Question

**Why should state files be protected? How do you prevent secrets from displaying in output?**

Good answer: State files contain ALL resource data including secrets in plaintext. Protect them with encryption, access controls, and never commit to Git. Use `sensitive = true` in outputs to hide values from CLI display (but they're still in state).

---

## Section 5: Terraform modules

### 5a: Explain how Terraform sources modules

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **Modules from Terraform Registry: namespace/name/provider**
  ```hcl
  module "vpc" {
    source = "terraform-aws-modules/vpc/aws"
    version = "5.0.0"
  }
  ```
  
- **Local modules: ./modules/vpc or ../shared/network**
  ```hcl
  module "vpc" {
    source = "./modules/vpc"
  }
  ```
  
- **Git repositories: git::https://... or github.com/...**
  ```hcl
  module "vpc" {
    source = "git::https://github.com/org/repo.git"
  }
  
  module "vpc" {
    source = "github.com/org/repo"
  }
  ```
  
- **HTTP URLs, S3 buckets**
  ```hcl
  module "vpc" {
    source = "https://example.com/vpc-module.zip"
  }
  
  module "vpc" {
    source = "s3::https://s3-us-west-2.amazonaws.com/bucket/vpc-module.zip"
  }
  ```
  
- **source argument in module block specifies location**

#### ⚠️ SHOULD KNOW

- **Registry modules: simple namespace/name syntax**
  - Format: namespace/name/provider
  - Example: hashicorp/consul/aws
  - Terraform knows to fetch from registry
  
- **Git sources can use ref= for specific branch/tag**
  ```hcl
  module "vpc" {
    source = "git::https://github.com/org/repo.git?ref=v1.0.0"
  }
  ```
  
- **Subdirectories within repos: //subdir**
  ```hcl
  module "vpc" {
    source = "git::https://github.com/org/repo.git//modules/vpc"
  }
  ```
  
- **Module sources downloaded during init**
  - terraform init downloads modules
  - Cached in .terraform/modules/

#### ❌ SKIP FOR EXAM

- Every module source type
- Module authentication details
- Registry API internals
- S3 module sources configuration

#### 🎯 Key Test Question

**What are 3 valid module source types?**

Good answer: 1) Terraform Registry (namespace/name/provider), 2) Local path (./modules/vpc), 3) Git repository (git::https://... or github.com/...). Also HTTP URLs and S3 buckets.

---

### 5b: Describe variable scope within modules

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **Each module has its own variable namespace**
  - Variables defined in child not visible in parent
  - Variables in parent not visible in child
  - Complete isolation
  
- **Parent passes data via module block arguments**
  ```hcl
  # Parent
  module "vpc" {
    source = "./modules/vpc"
    
    vpc_cidr = "10.0.0.0/16"  # Passing data to child
    azs      = ["us-west-2a", "us-west-2b"]
  }
  ```
  
- **Child defines what inputs it accepts via variable blocks**
  ```hcl
  # Child module (modules/vpc/variables.tf)
  variable "vpc_cidr" {
    type = string
  }
  
  variable "azs" {
    type = list(string)
  }
  ```
  
- **Variables don't automatically flow down to child modules**
  - Must explicitly pass them
  - No implicit inheritance
  
- **Modules are isolated - can't directly access parent vars**
  - Isolation is intentional
  - Makes modules reusable
  - Prevents hidden dependencies

#### ⚠️ SHOULD KNOW

- **Explicit passing required for data sharing**
  - Parent: `module "x" { foo = var.bar }`
  - Child: `variable "foo" {}`
  - Two separate pieces
  
- **Module inputs should be well-defined interface**
  - Document required inputs
  - Provide defaults where sensible
  - Clear variable names
  
- **Default values in child module variables**
  ```hcl
  variable "instance_type" {
    type    = string
    default = "t2.micro"
  }
  ```

#### ❌ SKIP FOR EXAM

- Variable precedence in modules
- Complex module composition patterns
- Module variable inheritance strategies

#### 🎯 Key Test Question

**Can a child module access variables defined in the parent module?**

Good answer: No. Modules are isolated. The parent must explicitly pass values via the module block arguments, and the child must define variables to accept them. Variables don't automatically flow between modules.

---

### 5c: Use modules in configuration

**⏱️ Time Budget: 25 minutes**

#### ✅ MUST KNOW

- **module block calls a module**
  ```hcl
  module "vpc" {
    source = "terraform-aws-modules/vpc/aws"
    
    name = "my-vpc"
    cidr = "10.0.0.0/16"
  }
  ```
  
- **Arguments in module block = inputs to child**
  - Each argument passes value to child variable
  - `name = "my-vpc"` → `var.name` in child
  
- **Child outputs accessed via module.name.output_name**
  ```hcl
  # Child module outputs
  output "vpc_id" {
    value = aws_vpc.main.id
  }
  
  # Parent references it
  resource "aws_subnet" "main" {
    vpc_id = module.vpc.vpc_id
  }
  ```
  
- **Module block syntax: module "name" { source = "..." }**
  - name = local reference name (your choice)
  - source = where to get module
  
- **Module creates reusable infrastructure patterns**
  - VPC module = VPC + subnets + route tables
  - Use same module for dev, staging, prod
  - Different inputs, same structure

#### ⚠️ SHOULD KNOW

- **count/for_each with modules**
  ```hcl
  module "vpc" {
    count  = 3
    source = "./modules/vpc"
    
    name = "vpc-${count.index}"
  }
  
  # Reference
  module.vpc[0].vpc_id
  ```
  
- **depends_on with modules**
  ```hcl
  module "vpc" {
    source = "./modules/vpc"
    
    depends_on = [aws_iam_role.example]
  }
  ```
  
- **Module outputs must be declared in child**
  - Not automatic
  - Child chooses what to expose

#### ❌ SKIP FOR EXAM

- Complex module composition
- Module testing strategies
- Module versioning strategies

#### 🎯 Key Test Question

**How do you pass a value to a module and retrieve an output from it?**

Good answer: Pass via module block arguments (`module "vpc" { cidr = "10.0.0.0/16" }`), retrieve via `module.name.output_name` (like `module.vpc.vpc_id`). The child must have corresponding variable and output blocks.

---

### 5d: Manage module versions

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **version argument in module block**
  ```hcl
  module "vpc" {
    source  = "terraform-aws-modules/vpc/aws"
    version = "5.0.0"
  }
  ```
  
- **Version constraints: ~>, >=, =, <**
  - `= 5.0.0` = exactly 5.0.0
  - `>= 5.0.0` = 5.0.0 or higher
  - `~> 5.0` = >= 5.0.0 and < 6.0.0
  - `~> 5.1.0` = >= 5.1.0 and < 5.2.0
  
- **~> 1.0 means >= 1.0 and < 2.0**
  - Pessimistic constraint
  - Allows patches and minor updates
  - Not major updates
  
- **Only works with Terraform Registry modules**
  - version argument ignored for other sources
  - Git modules use ref= instead
  
- **Pin versions to avoid unexpected changes**
  - Module updates might break your code
  - Version constraint gives control
  - Lock file records exact version used

#### ⚠️ SHOULD KNOW

- **Git sources use ref= instead of version**
  ```hcl
  module "vpc" {
    source = "git::https://github.com/org/repo.git?ref=v1.0.0"
  }
  ```
  
- **terraform init -upgrade updates modules**
  - Respects version constraints
  - Gets latest matching version
  - Updates lock file
  
- **Lock file tracks exact module versions**
  - .terraform.lock.hcl
  - Records what was actually downloaded
  - Commit to version control

#### ❌ SKIP FOR EXAM

- Semantic versioning deep dive
- Module publishing workflows
- Module version management strategies

#### 🎯 Key Test Question

**How do you ensure a module stays on version 1.x but gets patches?**

Good answer: Use the `~>` constraint: `version = "~> 1.0"`. This allows any 1.x version (like 1.1, 1.2.5) but not 2.0 or higher.

---

## Section 6: Terraform state management

### 6a: Describe the local backend

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **Local backend = default state storage**
  - When you don't configure a backend
  - Terraform uses local backend automatically
  
- **Stores state in terraform.tfstate file on disk**
  - JSON file in current directory
  - Updated after every apply
  - Plain text (including secrets)
  
- **Good for individual work, testing, learning**
  - Simple - just works
  - No setup needed
  - Fine for solo development
  
- **Not suitable for teams (no locking, sharing issues)**
  - Each person has their own state file
  - No way to share state
  - No locking = risk of conflicts
  - No encryption, backup, or versioning
  
- **No backend block = local backend automatically**
  ```hcl
  # No backend block?
  # Local backend is used
  ```

#### ⚠️ SHOULD KNOW

- **Local backend has no locking**
  - Two people running apply = disaster
  - State file corruption possible
  
- **State file in current directory**
  - terraform.tfstate
  - terraform.tfstate.backup (previous state)
  
- **Can specify path for state file**
  ```hcl
  terraform {
    backend "local" {
      path = "relative/path/to/terraform.tfstate"
    }
  }
  ```

#### ❌ SKIP FOR EXAM

- Local backend configuration options
- State file migration details
- Manual state file handling

#### 🎯 Key Test Question

**What is the local backend and when would you use it?**

Good answer: The local backend stores state in a terraform.tfstate file on disk. It's the default when no backend is configured. Good for individual work and learning, but not suitable for teams (no locking, no sharing).

---

### 6b: Describe state locking

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **Prevents concurrent modifications to state**
  - Two people/processes can't modify state simultaneously
  - Lock acquired before operations
  - Released after completion
  
- **Locks state during operations (plan, apply)**
  - terraform plan = read lock
  - terraform apply = write lock
  - terraform destroy = write lock
  
- **Prevents corruption from simultaneous runs**
  - Without locking: two applies at once = broken state
  - With locking: second apply waits or fails
  
- **Automatic with supported backends**
  - Most remote backends support locking
  - Terraform handles it automatically
  - You don't need to do anything
  
- **Can force-unlock if lock stuck (dangerous)**
  ```bash
  terraform force-unlock <lock-id>
  # Only use if lock is stuck
  ```

#### ⚠️ SHOULD KNOW

- **Not all backends support locking**
  - Local backend = no locking
  - S3 backend = needs DynamoDB table for locking
  - Terraform Cloud = has locking
  - Check docs for your backend
  
- **S3 backend needs DynamoDB for locking**
  ```hcl
  terraform {
    backend "s3" {
      bucket         = "my-terraform-state"
      key            = "prod/terraform.tfstate"
      region         = "us-west-2"
      dynamodb_table = "terraform-locks"  # For locking!
    }
  }
  ```
  
- **-lock=false disables (not recommended)**
  ```bash
  terraform apply -lock=false
  # Dangerous! Only for debugging
  ```
  
- **Lock errors prevent accidental conflicts**
  - Better to fail than corrupt state
  - Error message shows who has lock

#### ❌ SKIP FOR EXAM

- Lock implementation mechanics
- Lock timeout configuration
- Custom locking mechanisms

#### 🎯 Key Test Question

**What is state locking and why is it important?**

Good answer: State locking prevents multiple people or processes from modifying state at the same time. It prevents state file corruption and conflicts. Most remote backends support it automatically.

---

### 6c: Configure remote state using the backend block

**⏱️ Time Budget: 25 minutes**

#### ✅ MUST KNOW

- **backend block in terraform {} settings**
  ```hcl
  terraform {
    backend "s3" {
      bucket = "my-terraform-state"
      key    = "prod/terraform.tfstate"
      region = "us-west-2"
    }
  }
  ```
  
- **Common backends: s3, azurerm, gcs, remote (HCP Terraform)**
  - s3 = AWS S3 + DynamoDB
  - azurerm = Azure Blob Storage
  - gcs = Google Cloud Storage
  - remote = Terraform Cloud/HCP Terraform
  
- **Remote state enables team collaboration**
  - Everyone reads/writes same state
  - Locking prevents conflicts
  - Encryption protects secrets
  
- **Must run terraform init after backend changes**
  ```bash
  # Change backend configuration
  # Then:
  terraform init
  # Or to migrate state:
  terraform init -migrate-state
  ```
  
- **Partial backend configuration with -backend-config**
  ```bash
  # In config - leave out sensitive values
  terraform {
    backend "s3" {
      bucket = "my-terraform-state"
      key    = "prod/terraform.tfstate"
      region = "us-west-2"
      # No credentials in code!
    }
  }
  
  # At command line
  terraform init -backend-config="access_key=..." -backend-config="secret_key=..."
  ```

#### ⚠️ SHOULD KNOW

- **Backend migration with terraform init -migrate-state**
  - Switches from one backend to another
  - Copies state to new backend
  - Interactive prompts for confirmation
  
- **Only one backend per configuration**
  - Can't have multiple backends
  - If you need different backends, use workspaces or separate configs
  
- **Backend config cannot use variables**
  ```hcl
  # This DOESN'T work
  terraform {
    backend "s3" {
      bucket = var.bucket_name  # ❌ Not allowed
    }
  }
  
  # Use partial config + CLI flags instead
  ```
  
- **Remote state provides locking and encryption**
  - S3: encryption at rest, versioning, locking via DynamoDB
  - Terraform Cloud: encryption, locking, versioning, audit logs

#### ❌ SKIP FOR EXAM

- Every backend type configuration
- Backend authentication details
- Custom backend development

#### 🎯 Key Test Question

**How do you configure S3 as a backend for remote state?**

Good answer: Use a `backend "s3"` block in terraform {} settings with bucket, key, and region. Add a DynamoDB table for locking. Run `terraform init` to initialize the backend.

---

### 6d: Manage resource drift and Terraform state

**⏱️ Time Budget: 25 minutes**

#### ✅ MUST KNOW

- **Drift = real infrastructure differs from state**
  - Someone manually changed resource in console
  - External process modified infrastructure
  - State no longer matches reality
  
- **terraform plan detects drift automatically**
  ```bash
  terraform plan
  # Shows changes needed to match config
  # Including drift from manual changes
  ```
  
- **Refresh-only mode: update state without changes**
  ```bash
  terraform apply -refresh-only
  # Updates state to match reality
  # Doesn't modify infrastructure
  ```
  
- **moved block: refactor without recreating resources**
  ```hcl
  moved {
    from = aws_instance.old_name
    to   = aws_instance.new_name
  }
  # Terraform updates state references
  # Resource not destroyed/recreated
  ```
  
- **removed block: remove from state, keep infrastructure**
  ```hcl
  removed {
    from = aws_instance.example
    
    lifecycle {
      destroy = false
    }
  }
  # Removes from state
  # Infrastructure stays
  ```

#### ⚠️ SHOULD KNOW

- **terraform refresh updates state (deprecated)**
  ```bash
  terraform refresh
  # Old way to update state
  # Now: use plan/apply with refresh behavior
  ```
  
- **plan/apply refresh by default**
  - Every plan refreshes state first
  - Sees current reality
  - Then calculates needed changes
  
- **Import for resources created outside Terraform**
  ```bash
  terraform import aws_instance.example i-1234567
  # Brings existing resource under management
  ```
  
- **State manipulation via terraform state commands**
  - `terraform state rm` = remove from state
  - `terraform state mv` = rename in state
  - Use for refactoring

#### ❌ SKIP FOR EXAM

- Complex refactoring patterns
- State file surgery
- Advanced drift resolution
- Drift detection tools

#### 🎯 Key Test Question

**How does Terraform detect if infrastructure has changed outside of Terraform?**

Good answer: terraform plan automatically refreshes state (queries real infrastructure) before planning changes. It compares reality with desired config and shows drift as proposed changes.

---

## Section 7: Maintain infrastructure with Terraform

### 7a: Import existing infrastructure into your Terraform workspace

**⏱️ Time Budget: 25 minutes**

#### ✅ MUST KNOW

- **terraform import brings existing infrastructure under Terraform management**
  - For resources created manually or by other tools
  - Adds them to Terraform state
  - Terraform can then manage them
  
- **Syntax: terraform import <resource_address> <id>**
  ```bash
  # 1. Write the resource block first
  resource "aws_instance" "imported" {
    # Leave empty or add some attributes
  }
  
  # 2. Then import
  terraform import aws_instance.imported i-1234567890abcdef0
  ```
  
- **Must write resource block FIRST, then import**
  - Import doesn't generate configuration
  - You must write the resource block manually
  - Import only updates state
  
- **Import updates state only, not configuration**
  - After import, state knows about resource
  - But config still needs attributes
  - Run `terraform plan` to see missing attributes
  - Add them to your config
  
- **Each resource type has specific ID format**
  - EC2 instance: instance ID (i-...)
  - S3 bucket: bucket name
  - IAM role: role name
  - Check provider docs for format

#### ⚠️ SHOULD KNOW

- **import block (Terraform 1.5+) for config generation**
  ```hcl
  import {
    to = aws_instance.example
    id = "i-1234567890abcdef0"
  }
  ```
  - Newer way to import
  - Can generate config automatically
  
- **You must manually write the resource config**
  - terraform import doesn't create .tf files
  - You write the resource block
  - Then add attributes based on plan output
  
- **Resource ID varies by type (consult provider docs)**
  - Not always obvious what ID to use
  - Provider docs show import format
  - Example: AWS docs show "Import" section for each resource
  
- **Useful for migrating to Terraform**
  - Existing infrastructure → Terraform-managed
  - Gradual adoption strategy
  - Import resources one by one

#### ❌ SKIP FOR EXAM

- Import for every resource type
- Third-party import tools (terraformer, etc.)
- Bulk import strategies

#### 🎯 Key Test Question

**What must you do BEFORE running terraform import?**

Good answer: Write the resource block in your configuration first. Import only updates state - it doesn't generate configuration. You define the resource, then import the real infrastructure into it.

---

### 7b: Use the CLI to inspect state

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **terraform state list - lists all resources**
  ```bash
  terraform state list
  # Output:
  # aws_instance.web
  # aws_vpc.main
  # module.vpc.aws_subnet.private[0]
  ```
  
- **terraform state show <address> - shows resource details**
  ```bash
  terraform state show aws_instance.web
  # Shows all attributes of that resource
  ```
  
- **terraform state mv - rename/move resources**
  ```bash
  # Rename resource
  terraform state mv aws_instance.old aws_instance.new
  
  # Move to module
  terraform state mv aws_instance.web module.web.aws_instance.server
  ```
  
- **terraform state rm - remove from state (keeps infrastructure)**
  ```bash
  terraform state rm aws_instance.web
  # Removed from state
  # Resource still exists in cloud
  # Terraform no longer manages it
  ```
  
- **terraform output - shows output values**
  ```bash
  terraform output
  # Shows all outputs
  
  terraform output vpc_id
  # Shows specific output
  ```

#### ⚠️ SHOULD KNOW

- **state pull/push for remote state backup/restore**
  ```bash
  # Download state
  terraform state pull > backup.tfstate
  
  # Upload state (dangerous!)
  terraform state push backup.tfstate
  ```
  
- **State commands modify state file**
  - Changes are immediate
  - Use carefully
  - Backup first for safety
  
- **Use for refactoring without recreating resources**
  - Rename resources: use `mv`
  - Move to modules: use `mv`
  - Stop managing: use `rm`

#### ❌ SKIP FOR EXAM

- Every state subcommand option
- State file manual editing
- Complex state manipulation

#### 🎯 Key Test Question

**How do you see details of a specific resource in state?**

Good answer: Use `terraform state show <resource_address>`. For example: `terraform state show aws_instance.web`. This displays all attributes of that resource from the state file.

---

### 7c: Describe when and how to use verbose logging

**⏱️ Time Budget: 15 minutes**

#### ✅ MUST KNOW

- **TF_LOG environment variable enables logging**
  ```bash
  export TF_LOG=DEBUG
  terraform apply
  # Shows detailed debug output
  ```
  
- **Levels: TRACE, DEBUG, INFO, WARN, ERROR**
  - TRACE = most verbose (everything)
  - DEBUG = debugging information
  - INFO = general informational messages
  - WARN = warnings
  - ERROR = errors only
  
- **TF_LOG_PATH specifies log file location**
  ```bash
  export TF_LOG=TRACE
  export TF_LOG_PATH=./terraform.log
  terraform apply
  # Logs written to file instead of console
  ```
  
- **Use for troubleshooting complex issues**
  - API call failures
  - Provider bugs
  - Unexpected behavior
  - Support requests

#### ⚠️ SHOULD KNOW

- **TRACE is most verbose**
  - Shows every API call
  - Request/response bodies
  - Internal Terraform operations
  - Use when nothing else helps
  
- **Logs can contain sensitive data**
  - API responses include resource data
  - May include secrets, passwords
  - Don't share logs without sanitizing
  
- **Useful for provider debugging**
  - See exact API calls being made
  - Identify where failures occur
  - Provider maintainers may request logs
  
- **Disable after troubleshooting**
  ```bash
  unset TF_LOG
  unset TF_LOG_PATH
  ```

#### ❌ SKIP FOR EXAM

- Log format details
- Log parsing tools
- Provider-specific logging
- Custom log levels

#### 🎯 Key Test Question

**How do you enable detailed logging for debugging?**

Good answer: Set the TF_LOG environment variable to a log level (TRACE, DEBUG, INFO, WARN, or ERROR). Optionally set TF_LOG_PATH to write logs to a file. TRACE is the most verbose.

---

## Section 8: HCP Terraform

### 8a: Use HCP Terraform to create infrastructure

**⏱️ Time Budget: 25 minutes**

#### ✅ MUST KNOW

- **HCP Terraform (formerly Terraform Cloud) is SaaS platform**
  - Managed by HashiCorp
  - Web-based UI
  - No infrastructure to manage
  
- **Workspaces in HCP != CLI workspaces (different concept)**
  - CLI workspace = state file variant (same config)
  - HCP workspace = complete environment (config + state + variables + settings)
  - **These are NOT the same thing**
  
- **Remote execution runs in HCP infrastructure**
  - terraform plan/apply run in HCP's machines
  - Not on your laptop
  - Consistent environment
  
- **VCS integration triggers runs on commits**
  - Connect GitHub/GitLab/Bitbucket
  - Push code → auto-runs plan
  - Pull request → speculative plan
  
- **Stores state remotely with encryption and locking**
  - State stored in HCP
  - Encrypted at rest and in transit
  - Automatic locking
  - Version history

#### ⚠️ SHOULD KNOW

- **Free tier available**
  - Up to 5 users
  - 500 resources per month
  - Good for small teams and learning
  
- **HCP workspace = project/environment**
  - Not just a state file
  - Includes variables, permissions, settings
  - Like a complete Terraform project
  
- **CLI-driven vs VCS-driven workflows**
  - VCS-driven: code in Git, HCP pulls and runs
  - CLI-driven: run terraform commands locally, execution happens in HCP
  
- **Runs show logs in web UI**
  - See plan/apply output in browser
  - Persist logs for audit
  - Share with team

#### ❌ SKIP FOR EXAM

- HCP Terraform pricing tiers
- Enterprise-specific features
- Custom execution environments
- Agents and self-hosted runners

#### 🎯 Key Test Question

**What's the difference between HCP Terraform workspaces and CLI workspaces?**

Good answer: CLI workspaces are multiple state files from the same config. HCP workspaces are complete environments with their own config, state, variables, and settings. They're different concepts with the same name.

---

### 8b: Describe HCP Terraform collaboration and governance features

**⏱️ Time Budget: 25 minutes**

#### ✅ MUST KNOW

- **Policy as Code with Sentinel/OPA**
  - Write policies to enforce rules
  - "All S3 buckets must be encrypted"
  - "Only t2 instances in dev"
  - Policies checked automatically
  
- **Cost estimation shows infrastructure costs**
  - Before apply, see estimated monthly cost
  - Compares new cost vs current cost
  - Supports AWS, Azure, GCP
  
- **Health checks monitor workspace drift**
  - Automatically runs plan periodically
  - Detects if infrastructure changed
  - Alerts on drift
  
- **Private module registry for sharing modules**
  - Host internal modules
  - Version control
  - Share across organization
  
- **Teams and RBAC for access control**
  - Define teams
  - Assign permissions per workspace
  - Read, Plan, Write, Admin roles
  
- **Change requests for review workflows**
  - Like pull requests for infrastructure
  - Propose changes, team reviews
  - Approve before apply

#### ⚠️ SHOULD KNOW

- **Policies: advisory vs mandatory**
  - Advisory = warning, can override
  - Mandatory = hard fail, cannot proceed
  
- **Run triggers connect workspaces**
  - Workspace A completes → triggers Workspace B
  - Chain infrastructure deployments
  
- **Drift detection automatic checks**
  - Scheduled plans
  - Email/Slack notifications
  - Catch manual changes
  
- **Variable sets share across workspaces**
  - Define variables once
  - Apply to multiple workspaces
  - Good for common settings (region, tags)

#### ❌ SKIP FOR EXAM

- Writing Sentinel policies
- Complex governance patterns
- SSO configuration
- Policy set management

#### 🎯 Key Test Question

**What governance features does HCP Terraform provide?**

Good answer: Policy as Code (Sentinel/OPA) to enforce rules, cost estimation before apply, drift detection with health checks, private module registry, teams/RBAC for access control, and change requests for review workflows.

---

### 8c: Describe how to organize and use HCP Terraform workspaces and projects

**⏱️ Time Budget: 20 minutes**

#### ✅ MUST KNOW

- **Projects group related workspaces**
  - Organizational unit above workspace
  - Example: "Production" project with prod-vpc, prod-app, prod-db workspaces
  
- **Workspaces map to environments/components**
  - dev-vpc, staging-vpc, prod-vpc
  - Or: network, compute, database
  - Each workspace = one state file
  
- **Variable sets apply to multiple workspaces**
  ```
  Variable Set: "AWS Credentials"
  Applied to: All workspaces in Production project
  Variables: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
  ```
  
- **Run triggers create workspace dependencies**
  ```
  Workspace "network" completes
    ↓ Triggers
  Workspace "compute"
  ```
  
- **Each workspace has own state and variables**
  - Isolated from each other
  - Can't directly reference another workspace's resources
  - Use data source terraform_remote_state if needed

#### ⚠️ SHOULD KNOW

- **Projects enable organization**
  - Group by application, team, or environment
  - Apply permissions at project level
  
- **Project-level permissions**
  - Grant team access to entire project
  - Easier than per-workspace permissions
  
- **Workspace naming conventions matter**
  - Clear names help organization
  - Examples: prod-us-west-2-vpc, staging-app
  
- **Tags for workspace organization**
  - Add tags to workspaces
  - Filter by tag
  - Organize beyond projects

#### ❌ SKIP FOR EXAM

- Complex multi-project patterns
- Migration strategies between organizations
- Advanced workspace automation

#### 🎯 Key Test Question

**How do projects and workspaces help organize infrastructure?**

Good answer: Projects group related workspaces together. Workspaces map to environments or components, each with their own state and variables. Variable sets and permissions can be applied at the project level for easier management.

---

### 8d: Configure and use HCP Terraform integration

**⏱️ Time Budget: 25 minutes**

#### ✅ MUST KNOW

- **cloud {} block in terraform settings**
  ```hcl
  terraform {
    cloud {
      organization = "my-org"
      
      workspaces {
        name = "my-workspace"
      }
    }
  }
  ```
  
- **terraform login authenticates CLI to HCP**
  ```bash
  terraform login
  # Opens browser for authentication
  # Saves token locally
  ```
  
- **Remote operations execute in HCP**
  - When you run terraform plan locally
  - Plan actually runs in HCP
  - Results streamed to your terminal
  
- **Local operations with remote state possible**
  ```hcl
  terraform {
    cloud {
      organization = "my-org"
      
      workspaces {
        name = "my-workspace"
      }
      
      hostname = "app.terraform.io"
      
      # Force local execution
      ## (Not recommended for remote state only)
    }
  }
  ```
  
- **Migrate state with terraform init**
  ```bash
  # Add cloud block to config
  # Then:
  terraform init
  # Prompts to migrate state to HCP
  ```

#### ⚠️ SHOULD KNOW

- **Execution mode: remote vs local**
  - Remote = runs in HCP (recommended)
  - Local = runs on your machine, state in HCP
  
- **CLI-driven workflow from local machine**
  - Run terraform commands locally
  - Execution happens remotely
  - Best of both worlds
  
- **VCS integration auto-runs on commits**
  - Alternative to CLI-driven
  - Code in Git
  - HCP watches repo and runs plans
  
- **Workspace variables vs Terraform variables**
  - Workspace variables = environment variables (AWS_*)
  - Terraform variables = input variables (var.*)

#### ❌ SKIP FOR EXAM

- Agent pools configuration
- Custom execution images
- Advanced VCS patterns
- Webhook configuration

#### 🎯 Key Test Question

**How do you configure Terraform CLI to use HCP Terraform?**

Good answer: Add a `cloud {}` block in terraform settings with your organization and workspace. Run `terraform login` to authenticate. Then `terraform init` to migrate state if needed. Remote operations will then execute in HCP.

---

## 📊 Study Progress Checklist

Track your progress through all 37 topics:

**Section 1: IaC with Terraform** (3 topics)
- [ ] 1a: Explain what IaC is
- [ ] 1b: Advantages of IaC patterns  
- [ ] 1c: Multi-cloud workflows

**Section 2: Terraform fundamentals** (4 topics)
- [ ] 2a: Install and version providers
- [ ] 2b: How Terraform uses providers
- [ ] 2c: Multiple providers
- [ ] 2d: State management

**Section 3: Core workflow** (7 topics)
- [ ] 3a: Terraform workflow
- [ ] 3b: Initialize directory (init)
- [ ] 3c: Validate configuration
- [ ] 3d: Execution plan (plan)
- [ ] 3e: Apply changes
- [ ] 3f: Destroy infrastructure
- [ ] 3g: Format configuration (fmt)

**Section 4: Configuration** (8 topics)
- [ ] 4a: Resources vs data sources
- [ ] 4b: Resource references
- [ ] 4c: Variables and outputs
- [ ] 4d: Complex types
- [ ] 4e: Expressions and functions
- [ ] 4f: Resource dependencies
- [ ] 4g: Custom validation
- [ ] 4h: Sensitive data management

**Section 5: Modules** (4 topics)
- [ ] 5a: Module sources
- [ ] 5b: Variable scope in modules
- [ ] 5c: Using modules
- [ ] 5d: Module versions

**Section 6: State management** (4 topics)
- [ ] 6a: Local backend
- [ ] 6b: State locking
- [ ] 6c: Remote state backends
- [ ] 6d: Resource drift

**Section 7: Maintain infrastructure** (3 topics)
- [ ] 7a: Import existing infrastructure
- [ ] 7b: Inspect state (CLI)
- [ ] 7c: Verbose logging

**Section 8: HCP Terraform** (4 topics)
- [ ] 8a: Create infrastructure with HCP
- [ ] 8b: Collaboration features
- [ ] 8c: Workspaces and projects
- [ ] 8d: HCP integration

---

## 🎯 Final Exam Tips

1. **Focus on breadth over depth** - Know all 37 topics at working level rather than mastering a few
2. **Practice the workflow** - Write → Init → Plan → Apply until it's second nature
3. **Understand state** - Many questions test state management concepts
4. **Know module basics** - How to source, use, and version modules
5. **HCP Terraform naming** - It's "HCP Terraform" now, not "Terraform Cloud"
6. **Time management** - 60 minutes, ~57 questions = ~1 minute per question
7. **Read carefully** - Questions often have "select ALL that apply"
8. **Flag and return** - Skip hard questions, come back later
9. **No guessing penalty** - Answer everything, even if unsure

---

## 📚 Additional Resources

- **Official Exam Objectives**: [HashiCorp Exam Review 004](https://developer.hashicorp.com/terraform/tutorials/certification-004/associate-review-004)
- **Sample Questions**: [Official Sample Questions](https://developer.hashicorp.com/terraform/tutorials/certification-004/associate-questions-004)
- **Terraform Registry**: [registry.terraform.io](https://registry.terraform.io)
- **Terraform Documentation**: [developer.hashicorp.com/terraform](https://developer.hashicorp.com/terraform)

---

**Good luck with your exam! 🚀**

*Remember: The exam tests practical knowledge, not memorization. Focus on understanding concepts and how to apply them.*
