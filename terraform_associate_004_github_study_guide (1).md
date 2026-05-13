# Terraform Associate 004 Study Guide

> Official syllabus-aligned preparation guide for the Terraform Associate 004 certification exam.

---

# Table of Contents

- [1. Infrastructure as Code with Terraform](#1-infrastructure-as-code-with-terraform)
  - [1a. Explain What Infrastructure as Code Is](#1a-explain-what-infrastructure-as-code-is)
  - [1b. Describe the Advantages of Infrastructure as Code Patterns](#1b-describe-the-advantages-of-infrastructure-as-code-patterns)
  - [1c. Explain How Terraform Manages Multi-Cloud Hybrid Cloud and Service-Agnostic Workflows](#1c-explain-how-terraform-manages-multi-cloud-hybrid-cloud-and-service-agnostic-workflows)

- [2. Terraform Fundamentals](#2-terraform-fundamentals)
  - [2a. Install and Version Terraform Providers](#2a-install-and-version-terraform-providers)
  - [2b. Describe How Terraform Uses Providers](#2b-describe-how-terraform-uses-providers)
  - [2c. Write Terraform Configuration Using Multiple Providers](#2c-write-terraform-configuration-using-multiple-providers)
  - [2d. Explain How Terraform Uses and Manages State](#2d-explain-how-terraform-uses-and-manages-state)

- [3. Core Terraform Workflow](#3-core-terraform-workflow)
  - [3a. Describe the Terraform Workflow](#3a-describe-the-terraform-workflow)
  - [3b. Initialize a Terraform Working Directory](#3b-initialize-a-terraform-working-directory)
  - [3c. Validate a Terraform Configuration](#3c-validate-a-terraform-configuration)
  - [3d. Generate and Review an Execution Plan for Terraform](#3d-generate-and-review-an-execution-plan-for-terraform)
  - [3e. Apply Changes to Infrastructure with Terraform](#3e-apply-changes-to-infrastructure-with-terraform)
  - [3f. Destroy Terraform-Managed Infrastructure](#3f-destroy-terraform-managed-infrastructure)
  - [3g. Apply Formatting and Style Adjustments to a Configuration](#3g-apply-formatting-and-style-adjustments-to-a-configuration)

- [4. Terraform Configuration](#4-terraform-configuration)
  - [4a. Use and Differentiate resource and data Blocks](#4a-use-and-differentiate-resource-and-data-blocks)
  - [4b. Refer to Resource Attributes and Create Cross-Resource References](#4b-refer-to-resource-attributes-and-create-cross-resource-references)
  - [4c. Use Variables and Outputs](#4c-use-variables-and-outputs)
  - [4d. Understand and Use Complex Types](#4d-understand-and-use-complex-types)
  - [4e. Write Dynamic Configuration Using Expressions and Functions](#4e-write-dynamic-configuration-using-expressions-and-functions)
  - [4f. Define Resource Dependencies in Configuration](#4f-define-resource-dependencies-in-configuration)
  - [4g. Validate Configuration Using Custom Conditions](#4g-validate-configuration-using-custom-conditions)
  - [4h. Understand Best Practices for Managing Sensitive Data Including Secrets Management with Vault](#4h-understand-best-practices-for-managing-sensitive-data-including-secrets-management-with-vault)

- [5. Terraform Modules](#5-terraform-modules)
  - [5a. Explain How Terraform Sources Modules](#5a-explain-how-terraform-sources-modules)
  - [5b. Describe Variable Scope Within Modules](#5b-describe-variable-scope-within-modules)
  - [5c. Use Modules in Configuration](#5c-use-modules-in-configuration)
  - [5d. Manage Module Versions](#5d-manage-module-versions)

- [6. Terraform State Management](#6-terraform-state-management)
  - [6a. Describe the Local Backend](#6a-describe-the-local-backend)
  - [6b. Describe State Locking](#6b-describe-state-locking)
  - [6c. Configure Remote State Using the Backend Block](#6c-configure-remote-state-using-the-backend-block)
  - [6d. Manage Resource Drift and Terraform State](#6d-manage-resource-drift-and-terraform-state)

- [7. Maintain Infrastructure with Terraform](#7-maintain-infrastructure-with-terraform)
  - [7a. Import Existing Infrastructure into Your Terraform Workspace](#7a-import-existing-infrastructure-into-your-terraform-workspace)
  - [7b. Use the CLI to Inspect State](#7b-use-the-cli-to-inspect-state)
  - [7c. Describe When and How to Use Verbose Logging](#7c-describe-when-and-how-to-use-verbose-logging)

- [8. HCP Terraform](#8-hcp-terraform)
  - [8a. Use HCP Terraform to Create Infrastructure](#8a-use-hcp-terraform-to-create-infrastructure)
  - [8b. Describe HCP Terraform Collaboration and Governance Features](#8b-describe-hcp-terraform-collaboration-and-governance-features)
  - [8c. Describe How to Organize and Use HCP Terraform Workspaces and Projects](#8c-describe-how-to-organize-and-use-hcp-terraform-workspaces-and-projects)
  - [8d. Configure and Use HCP Terraform Integration](#8d-configure-and-use-hcp-terraform-integration)

---

# 1. Infrastructure as Code with Terraform

# 1a. Explain What Infrastructure as Code Is

## Theory

Infrastructure as Code (IaC) means managing infrastructure using code instead of manually creating resources through a GUI.

Instead of:
- clicking buttons in AWS,
- manually creating VMs,
- manually configuring networking,

You define infrastructure in configuration files.

Terraform then creates and manages the infrastructure.

---

## Why It Exists

Manual infrastructure creation causes:
- inconsistency,
- human errors,
- configuration drift,
- poor scalability.

IaC solves this by making infrastructure:
- repeatable,
- version-controlled,
- automated,
- consistent.

---

## Important Concepts

| Concept | Meaning |
|---|---|
| Declarative | Define desired end state |
| Imperative | Define step-by-step instructions |
| Idempotent | Same result even if run multiple times |
| Version Controlled | Infrastructure tracked in Git |

Terraform is declarative.

---

## Code Example

```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

Terraform understands:
- what should exist,
- compares against state,
- and creates infrastructure.

---

## What Terraform Is Actually Doing

Terraform compares:
1. Configuration
2. Terraform State
3. Real Infrastructure

Then builds an execution plan.

---

## Important Exam Caveats

- Terraform is declarative, not imperative.
- Terraform does NOT continuously monitor infrastructure.
- Terraform uses state to track resources.
- Terraform is NOT a configuration management tool like Ansible.

---

## Common Mistakes

### Wrong

"Terraform continuously watches infrastructure."

### Correct

Terraform only evaluates infrastructure during Terraform operations.

---

## Quick Revision Notes

- IaC = infrastructure managed through code
- Terraform is declarative
- Terraform uses state
- IaC improves consistency and automation
- Terraform is idempotent

---

## Quiz

### Q1
What does Infrastructure as Code mean?

### Q2
Is Terraform declarative or imperative?

### Q3
What does idempotent mean?

### Q4
Does Terraform continuously monitor infrastructure?

### Q5
What are the 3 things Terraform compares?

---

# 1b. Describe the Advantages of Infrastructure as Code Patterns

## Theory

IaC provides operational consistency and automation.

Infrastructure becomes:
- reusable,
- scalable,
- auditable,
- predictable.

---

## Advantages

| Advantage | Description |
|---|---|
| Automation | Reduces manual work |
| Consistency | Same infrastructure every time |
| Version Control | Track infrastructure changes |
| Reusability | Reuse modules and templates |
| Faster Deployments | Infrastructure can be recreated quickly |
| Disaster Recovery | Infrastructure can be rebuilt from code |

---

## Real-World Example

Without IaC:
- Dev and Prod environments differ.

With IaC:
- Same configuration builds both environments.

---

## Important Exam Caveats

Terraform improves:
- reproducibility,
- auditability,
- collaboration.

Terraform does NOT eliminate all operational risks.

---

## Common Mistakes

### Wrong

"IaC automatically fixes all drift."

### Correct

Terraform detects drift during operations.

---

## Quick Revision Notes

- IaC improves consistency
- IaC enables version control
- IaC improves disaster recovery
- IaC improves automation

---

## Quiz

### Q1
Why is version control important in IaC?

### Q2
How does IaC improve disaster recovery?

### Q3
What problem does IaC reduce?

### Q4
Can Terraform completely prevent infrastructure drift?

---

# 1c. Explain How Terraform Manages Multi-Cloud Hybrid Cloud and Service-Agnostic Workflows

## Theory

Terraform supports multiple providers.

Examples:
- AWS
- Azure
- Google Cloud
- Kubernetes
- GitHub
- Datadog
- Confluent

Terraform can manage all of them from one workflow.

---

## Multi-Cloud Example

```hcl
provider "aws" {
  region = "us-east-1"
}

provider "azurerm" {
  features {}
}
```

Terraform can deploy infrastructure across both clouds.

---

## Hybrid Cloud

Hybrid cloud means:
- on-prem + cloud.

Terraform can manage:
- VMware,
- Kubernetes,
- cloud providers,
- SaaS platforms.

---

## What Terraform Is Actually Doing

Terraform uses provider plugins.

Each provider translates Terraform configuration into API calls.

---

## Important Exam Caveats

- Terraform itself does NOT create infrastructure.
- Providers interact with APIs.
- Terraform is service-agnostic because providers abstract APIs.

---

## Quick Revision Notes

- Providers enable multi-cloud support
- Terraform uses provider plugins
- Terraform supports hybrid cloud
- Providers make API calls

---

## Quiz

### Q1
What enables Terraform to support multiple clouds?

### Q2
Does Terraform directly create infrastructure?

### Q3
What is hybrid cloud?

### Q4
What translates Terraform configuration into API calls?

---

# 2. Terraform Fundamentals

# 2a. Install and Version Terraform Providers

## Theory

Providers are plugins used by Terraform to communicate with APIs.

Terraform downloads providers during:

```bash
terraform init
```

---

## Provider Configuration

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

---

## Version Constraints

| Constraint | Meaning |
|---|---|
| = 5.0.0 | Exact version |
| >= 5.0 | Minimum version |
| ~> 5.0 | Allow minor updates |

---

## Important Exam Caveats

- `terraform init` downloads providers.
- Providers are plugins.
- Provider versions should be pinned.
- `.terraform.lock.hcl` stores dependency versions.

---

## Common Mistakes

### Wrong

"Providers are built into Terraform."

### Correct

Providers are external plugins.

---

## Quick Revision Notes

- Providers = plugins
- init downloads providers
- required_providers defines versions
- lock file stores provider versions

---

## Quiz

### Q1
What command downloads providers?

### Q2
What file stores provider version locks?

### Q3
Are providers built into Terraform?

### Q4
What does `~>` mean?

---

# 2b. Describe How Terraform Uses Providers

## Theory

Providers allow Terraform to interact with external APIs.

Terraform itself does not know how to:
- create EC2 instances,
- create Kubernetes deployments,
- create GitHub repositories.

Providers contain that logic.

---

## Example

```hcl
provider "aws" {
  region = "us-east-1"
}
```

Terraform loads the AWS provider.

The provider then communicates with AWS APIs.

---

## What Terraform Is Actually Doing

Terraform:
1. Reads configuration
2. Loads provider plugin
3. Authenticates to API
4. Makes API calls
5. Stores results in state

---

## Important Exam Caveats

- Providers manage resources.
- Multiple providers can exist.
- Providers require authentication.

---

## Quick Revision Notes

- Providers interact with APIs
- Terraform uses plugins
- Providers manage resources
- Providers authenticate to services

---

## Quiz

### Q1
What is the role of a provider?

### Q2
Does Terraform directly communicate with cloud APIs?

### Q3
Can Terraform use multiple providers?

---

# 2c. Write Terraform Configuration Using Multiple Providers

## Theory

Terraform supports multiple providers in the same configuration.

---

## Example

```hcl
provider "aws" {
  region = "us-east-1"
}

provider "google" {
  project = "demo-project"
  region  = "us-central1"
}
```

---

## Provider Alias Example

```hcl
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}
```

---

## Important Exam Caveats

- Aliases allow multiple configurations of same provider.
- Resources can specify providers.

Example:

```hcl
provider = aws.west
```

---

## Quick Revision Notes

- Multiple providers supported
- Aliases support multiple configs
- Resources can target providers

---

## Quiz

### Q1
What is provider alias used for?

### Q2
Can multiple AWS regions be used in one configuration?

### Q3
How does a resource select a provider alias?

---

# 2d. Explain How Terraform Uses and Manages State

## Theory

Terraform state stores Terraform's understanding of infrastructure.

State allows Terraform to:
- track resources,
- compare infrastructure,
- build execution plans.

---

## State File

Default local state file:

```text
terraform.tfstate
```

---

## What Terraform Is Actually Doing

Terraform compares:
1. Configuration
2. State
3. Real Infrastructure

Then determines:
- create,
- update,
- delete,
- no-op.

---

## Why State Is Important

Without state:
- Terraform would not know what exists.

---

## Important Exam Caveats

- State may contain sensitive data.
- Remote state is preferred for teams.
- State locking prevents concurrent modifications.

---

## Common Mistakes

### Wrong

"Terraform only compares configuration to infrastructure."

### Correct

Terraform also uses state.

---

## Quick Revision Notes

- State tracks resources
- Default state file = terraform.tfstate
- State may contain secrets
- Remote state preferred for teams

---

## Quiz

### Q1
What is the default Terraform state file called?

### Q2
Why is state important?

### Q3
Can state contain sensitive data?

### Q4
Why is remote state preferred?

---

# 3. Core Terraform Workflow

# 3a. Describe the Terraform Workflow

## Theory

Core workflow:

1. Write configuration
2. Initialize
3. Plan
4. Apply
5. Manage state

---

## Workflow Commands

```bash
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy
```

---

## Important Exam Caveats

- `plan` does not change infrastructure.
- `apply` changes infrastructure.
- `destroy` removes infrastructure.

---

## Quick Revision Notes

- init initializes working directory
- validate checks syntax
- plan previews changes
- apply executes changes
- destroy removes resources

---

## Quiz

### Q1
Which command previews changes?

### Q2
Which command modifies infrastructure?

### Q3
Does validate create infrastructure?

---

# 3b. Initialize a Terraform Working Directory

## Theory

`terraform init` initializes a Terraform working directory.

---

## What init Does

- Downloads providers
- Downloads modules
- Configures backend
- Creates `.terraform` directory

---

## Example

```bash
terraform init
```

---

## Important Exam Caveats

- init must run before plan/apply.
- init is safe to run multiple times.
- init does NOT create infrastructure.

---

## Quick Revision Notes

- init downloads providers/modules
- init configures backend
- init creates .terraform directory

---

## Quiz

### Q1
What directory does init create?

### Q2
Does init create infrastructure?

### Q3
What does init download?

---

# 3c. Validate a Terraform Configuration

## Theory

`terraform validate` checks configuration syntax and internal consistency.

---

## Example

```bash
terraform validate
```

---

## Important Exam Caveats

- validate checks syntax only.
- validate does NOT contact providers.
- validate does NOT create infrastructure.

---

## Quick Revision Notes

- validate checks syntax
- no infrastructure changes
- no API calls

---

## Quiz

### Q1
Does validate contact cloud APIs?

### Q2
Does validate create resources?

### Q3
What does validate check?

---

# 3d. Generate and Review an Execution Plan for Terraform

## Theory

`terraform plan` previews infrastructure changes.

---

## Example

```bash
terraform plan
```

---

## Plan Symbols

| Symbol | Meaning |
|---|---|
| + | Create |
| ~ | Update |
| - | Destroy |

---

## Important Exam Caveats

- plan is read-only.
- plan compares configuration/state/infrastructure.
- plan does NOT change infrastructure.

---

## Quick Revision Notes

- plan previews changes
- no infrastructure modifications
- plan uses state

---

## Quiz

### Q1
What does `+` mean in plan output?

### Q2
Does plan change infrastructure?

### Q3
What does `~` mean?

---

# 3e. Apply Changes to Infrastructure with Terraform

## Theory

`terraform apply` executes infrastructure changes.

---

## Example

```bash
terraform apply
```

---

## What Terraform Is Actually Doing

Terraform:
1. Builds dependency graph
2. Calls provider APIs
3. Creates/updates infrastructure
4. Updates state

---

## Important Exam Caveats

- apply modifies infrastructure.
- apply updates state.
- apply may partially succeed.

---

## Quick Revision Notes

- apply changes infrastructure
- apply updates state
- apply uses providers

---

## Quiz

### Q1
Does apply update state?

### Q2
Can apply partially succeed?

### Q3
What executes API calls?

---

# 3f. Destroy Terraform-Managed Infrastructure

## Theory

`terraform destroy` removes Terraform-managed infrastructure.

---

## Example

```bash
terraform destroy
```

---

## Important Exam Caveats

- destroy removes managed resources.
- destroy updates state.
- resources outside state are unaffected.

---

## Quick Revision Notes

- destroy removes infrastructure
- destroy updates state
- unmanaged resources unaffected

---

## Quiz

### Q1
Does destroy affect unmanaged resources?

### Q2
Does destroy update state?

---

# 3g. Apply Formatting and Style Adjustments to a Configuration

## Theory

`terraform fmt` formats Terraform code.

---

## Example

```bash
terraform fmt
```

---

## Important Exam Caveats

- fmt improves readability.
- fmt does NOT validate logic.
- fmt does NOT modify infrastructure.

---

## Quick Revision Notes

- fmt formats code
- no infrastructure changes
- improves consistency

---

## Quiz

### Q1
Does fmt validate Terraform logic?

### Q2
Does fmt change infrastructure?

---

# 4. Terraform Configuration

# 4a. Use and Differentiate resource and data Blocks

## Theory

`resource` blocks create/manage infrastructure.

`data` blocks read existing infrastructure.

---

## Resource Example

```hcl
resource "aws_s3_bucket" "demo" {
  bucket = "my-demo-bucket"
}
```

---

## Data Example

```hcl
data "aws_ami" "latest" {
  most_recent = true
}
```

---

## Important Exam Caveats

- resource = managed infrastructure
- data = read-only lookup
- data blocks do NOT create resources

---

## Common Mistakes

### Wrong

Using data block expecting infrastructure creation.

---

## Quick Revision Notes

- resource creates/manages
- data reads existing resources
- data blocks are read-only

---

## Quiz

### Q1
Which block creates infrastructure?

### Q2
Are data blocks read-only?

### Q3
Can data blocks modify infrastructure?

---

# 4b. Refer to Resource Attributes and Create Cross-Resource References

## Theory

Terraform resources can reference attributes from other resources.

---

## Example

```hcl
resource "aws_instance" "web" {
  subnet_id = aws_subnet.main.id
}
```

---

## Important Exam Caveats

- References create implicit dependencies.
- Terraform dependency graph uses references.

---

## Quick Revision Notes

- references create dependencies
- use dot notation
- Terraform builds dependency graph

---

## Quiz

### Q1
Do references create implicit dependencies?

### Q2
What notation is used for references?

---

# 4c. Use Variables and Outputs

## Theory

Variables define reusable inputs.

Outputs expose values after apply.

---

## Variable Example

```hcl
variable "region" {
  type = string
}
```

---

## Output Example

```hcl
output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

---

## Variable Precedence

Highest to Lowest:

1. CLI `-var`
2. `.auto.tfvars`
3. `terraform.tfvars`
4. Environment variables
5. Default values

---

## Important Exam Caveats

- variables = inputs
- outputs = exposed values
- locals are NOT variables

---

## Quick Revision Notes

- variables define inputs
- outputs expose values
- var.name syntax
- outputs support module sharing

---

## Quiz

### Q1
What is variable precedence order?

### Q2
What is purpose of outputs?

### Q3
Are locals external inputs?

---

# 4d. Understand and Use Complex Types

## Theory

Terraform supports:
- list
- map
- set
- object
- tuple

---

## Example

```hcl
variable "regions" {
  type = list(string)
}
```

---

## Important Exam Caveats

- list maintains order
- set does not guarantee order
- map uses key/value pairs

---

## Quick Revision Notes

- list ordered
- set unordered
- map key/value
- object structured data

---

## Quiz

### Q1
Which type does not guarantee order?

### Q2
What type uses key/value pairs?

---

# 4e. Write Dynamic Configuration Using Expressions and Functions

## Theory

Expressions and functions create dynamic Terraform configuration.

---

## Example

```hcl
name = upper("terraform")
```

---

## Common Functions

| Function | Purpose |
|---|---|
| upper() | uppercase |
| lower() | lowercase |
| length() | count length |
| concat() | merge lists |

---

## Important Exam Caveats

- expressions create dynamic values
- functions manipulate data

---

## Quick Revision Notes

- functions transform data
- expressions create dynamic config

---

## Quiz

### Q1
What does length() do?

### Q2
Why use expressions?

---

# 4f. Define Resource Dependencies in Configuration

## Theory

Terraform automatically builds dependencies.

Sometimes explicit dependencies are needed.

---

## Example

```hcl
depends_on = [aws_vpc.main]
```

---

## Important Exam Caveats

- references create implicit dependencies
- depends_on creates explicit dependencies

---

## Quick Revision Notes

- implicit via references
- explicit via depends_on

---

## Quiz

### Q1
What creates implicit dependency?

### Q2
What creates explicit dependency?

---

# 4g. Validate Configuration Using Custom Conditions

## Theory

Terraform supports validation rules.

---

## Example

```hcl
variable "instance_count" {
  type = number

  validation {
    condition     = var.instance_count > 0
    error_message = "Value must be greater than zero"
  }
}
```

---

## Important Exam Caveats

- validation improves input safety
- validation prevents invalid values

---

## Quick Revision Notes

- validation checks inputs
- condition defines rule
- error_message displays failure

---

## Quiz

### Q1
What block validates variables?

### Q2
What displays validation failure?

---

# 4h. Understand Best Practices for Managing Sensitive Data Including Secrets Management with Vault

## Theory

Sensitive data should not be hardcoded.

Examples:
- passwords
- API keys
- tokens
- secrets

---

## Sensitive Variable Example

```hcl
variable "password" {
  type      = string
  sensitive = true
}
```

---

## Vault

Vault securely stores secrets.

Terraform can retrieve secrets dynamically.

---

## Important Exam Caveats

- state may contain secrets
- sensitive values are masked in output
- sensitive values may still exist in state

---

## Quick Revision Notes

- avoid hardcoding secrets
- sensitive masks output
- Vault manages secrets securely

---

## Quiz

### Q1
Does sensitive prevent secrets from entering state?

### Q2
What tool securely stores secrets?

---

# 5. Terraform Modules

# 5a. Explain How Terraform Sources Modules

## Theory

Terraform modules can be sourced from:
- local paths,
- Terraform Registry,
- Git repositories,
- remote URLs.

Modules allow reusable infrastructure code.

---

## Local Module Example

```hcl
module "network" {
  source = "./modules/network"
}
```

---

## Registry Module Example

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
}
```

---

## Git Module Example

```hcl
module "app" {
  source = "git::https://github.com/company/modules.git"
}
```

---

## What Terraform Is Actually Doing

During `terraform init`:
- Terraform downloads modules,
- stores them locally,
- and loads them into dependency graph evaluation.

---

## Important Exam Caveats

- Modules are downloaded during `terraform init`.
- Registry modules should use version constraints.
- `source` defines module location.

---

## Quick Revision Notes

- modules support reuse
- source defines module location
- init downloads modules
- registry modules should use versions

---

## Quiz

### Q1
What argument defines module location?

### Q2
When are modules downloaded?

### Q3
Can modules come from Git repositories?

---

# 5b. Describe Variable Scope Within Modules

## Theory

Variables inside modules are scoped to that module.

Parent modules pass values into child modules.

---

## Example

### Child Module Variable

```hcl
variable "region" {
  type = string
}
```

### Parent Module

```hcl
module "network" {
  source = "./modules/network"
  region = "us-east-1"
}
```

---

## Important Exam Caveats

- Child modules cannot directly access parent variables.
- Variables must be explicitly passed.
- Outputs expose values from child modules.

---

## Common Mistakes

### Wrong

Assuming variables are globally shared.

### Correct

Variables are module-scoped.

---

## Quick Revision Notes

- variables are module-scoped
- parent passes variables to child
- outputs expose child values

---

## Quiz

### Q1
Are module variables globally shared?

### Q2
How are values passed into child modules?

### Q3
How are child module values exposed?

---

# 5c. Use Modules in Configuration

## Theory

Modules organize reusable infrastructure components.

Examples:
- VPC module
- EC2 module
- Kubernetes module
- Kafka module

---

## Example

```hcl
module "storage" {
  source = "./modules/storage"

  bucket_name = "demo-bucket"
}
```

---

## What Terraform Is Actually Doing

Terraform treats modules as reusable configuration blocks.

Resources inside modules become part of Terraform dependency graph.

---

## Important Exam Caveats

- Modules improve reuse and standardization.
- Root module = current working directory.
- Child modules = referenced modules.

---

## Quick Revision Notes

- modules improve reuse
- root module is current directory
- child modules are referenced modules

---

## Quiz

### Q1
What is the root module?

### Q2
Why use modules?

### Q3
Are module resources included in dependency graph?

---

# 5d. Manage Module Versions

## Theory

Module versions ensure predictable deployments.

Without version pinning:
- unexpected updates may occur.

---

## Example

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
}
```

---

## Important Exam Caveats

- Version pinning improves stability.
- Registry modules support versions.
- Local modules do not use version argument.

---

## Common Mistakes

### Wrong

Using latest versions without testing.

---

## Quick Revision Notes

- pin module versions
- registry modules support versions
- local modules do not use versions

---

## Quiz

### Q1
Why should module versions be pinned?

### Q2
Do local modules support version argument?

### Q3
Which modules commonly use version constraints?

---

# 6. Terraform State Management

# 6a. Describe the Local Backend

## Theory

The local backend stores Terraform state locally on disk.

Default state file:

```text
terraform.tfstate
```

---

## Example

```hcl
terraform {
  backend "local" {}
}
```

---

## Important Exam Caveats

- Local backend is default backend.
- Local state is not ideal for teams.
- Local state increases risk of state conflicts.

---

## Quick Revision Notes

- local backend stores state locally
- default backend is local
- not ideal for collaboration

---

## Quiz

### Q1
What is the default backend?

### Q2
Why is local backend problematic for teams?

### Q3
What file stores local state?

---

# 6b. Describe State Locking

## Theory

State locking prevents multiple Terraform operations from modifying state simultaneously.

---

## Why Locking Matters

Without locking:
- concurrent applies may corrupt state.

---

## What Terraform Is Actually Doing

Terraform locks state before:
- apply,
- destroy,
- state modifications.

Lock released after operation completes.

---

## Important Exam Caveats

- State locking prevents concurrent modifications.
- Some remote backends support locking.
- Local backend does not support distributed locking.

---

## Quick Revision Notes

- locking prevents concurrent state changes
- remote backends commonly support locking
- protects state integrity

---

## Quiz

### Q1
Why is state locking important?

### Q2
Does local backend support distributed locking?

### Q3
What problem does locking prevent?

---

# 6c. Configure Remote State Using the Backend Block

## Theory

Remote backends store Terraform state remotely.

Examples:
- S3
- HCP Terraform
- Azure Storage
- GCS

---

## Example

```hcl
terraform {
  backend "s3" {
    bucket = "terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}
```

---

## Advantages

| Advantage | Description |
|---|---|
| Collaboration | Shared state |
| Locking | Prevents corruption |
| Centralization | Single source of truth |

---

## Important Exam Caveats

- Backend configured inside terraform block.
- Backend configuration requires reinitialization.
- Remote state preferred for teams.

---

## Quick Revision Notes

- remote backend supports collaboration
- backend block configures state storage
- init required after backend changes

---

## Quiz

### Q1
Where is backend configured?

### Q2
Why is remote state preferred?

### Q3
What command required after backend changes?

---

# 6d. Manage Resource Drift and Terraform State

## Theory

Drift occurs when infrastructure changes outside Terraform.

Example:
- engineer manually changes cloud resource.

Terraform state becomes inconsistent.

---

## Example Scenario

Terraform expects:

```text
instance_type = t2.micro
```

Engineer manually changes to:

```text
t2.large
```

Terraform detects drift during plan/apply.

---

## What Terraform Is Actually Doing

Terraform refreshes infrastructure information and compares against state.

---

## Important Exam Caveats

- Drift is detected during operations.
- Terraform does not continuously monitor infrastructure.
- Refresh updates state information.

---

## Quick Revision Notes

- drift = manual infrastructure changes
- Terraform detects drift during operations
- refresh updates state data

---

## Quiz

### Q1
What is resource drift?

### Q2
When does Terraform detect drift?

### Q3
Does Terraform continuously monitor infrastructure?

---

# 7. Maintain Infrastructure with Terraform

# 7a. Import Existing Infrastructure into Your Terraform Workspace

## Theory

`terraform import` brings existing infrastructure into Terraform state.

---

## Example

```bash
terraform import aws_instance.web i-123456789
```

---

## Important Exam Caveats

- Import updates state only.
- Import does not generate configuration automatically.
- Matching configuration should exist.

---

## Common Mistakes

### Wrong

Assuming import creates Terraform code.

### Correct

Import only maps infrastructure into state.

---

## Quick Revision Notes

- import adds resources to state
- import does not auto-create config
- configuration should match infrastructure

---

## Quiz

### Q1
Does import generate Terraform configuration?

### Q2
What does import modify?

### Q3
Why should matching configuration exist?

---

# 7b. Use the CLI to Inspect State

## Theory

Terraform provides CLI commands for state inspection.

---

## Common Commands

```bash
terraform state list
terraform state show aws_instance.web
```

---

## Purpose

| Command | Purpose |
|---|---|
| state list | List resources |
| state show | Show resource details |

---

## Important Exam Caveats

- State commands inspect Terraform state.
- State commands do not modify infrastructure.

---

## Quick Revision Notes

- state list shows tracked resources
- state show displays resource details
- state commands inspect state

---

## Quiz

### Q1
What does state list do?

### Q2
What does state show do?

### Q3
Do state inspection commands modify infrastructure?

---

# 7c. Describe When and How to Use Verbose Logging

## Theory

Terraform supports verbose logging for troubleshooting.

---

## Example

```bash
export TF_LOG=DEBUG
```

---

## Common Log Levels

| Level | Purpose |
|---|---|
| TRACE | Most detailed |
| DEBUG | Debugging |
| INFO | General information |
| ERROR | Errors only |

---

## Important Exam Caveats

- TF_LOG enables logging.
- TRACE is most verbose.
- Logging helps troubleshoot providers and API calls.

---

## Quick Revision Notes

- TF_LOG enables logs
- TRACE most verbose
- useful for troubleshooting

---

## Quiz

### Q1
What variable enables Terraform logging?

### Q2
Which log level is most verbose?

### Q3
Why use verbose logging?

---

# 8. HCP Terraform

# 8a. Use HCP Terraform to Create Infrastructure

## Theory

HCP Terraform provides managed Terraform execution.

Features include:
- remote runs,
- state management,
- collaboration.

---

## Workflow

1. Push code to VCS
2. HCP Terraform runs plan
3. Review changes
4. Apply infrastructure

---

## Important Exam Caveats

- HCP Terraform supports remote execution.
- State stored remotely.
- Integrates with VCS systems.

---

## Quick Revision Notes

- HCP Terraform enables remote runs
- integrates with Git repositories
- manages remote state

---

## Quiz

### Q1
What does HCP Terraform provide?

### Q2
Does HCP Terraform support remote execution?

### Q3
Can HCP Terraform integrate with Git repositories?

---

# 8b. Describe HCP Terraform Collaboration and Governance Features

## Theory

HCP Terraform supports team collaboration and governance.

---

## Features

| Feature | Purpose |
|---|---|
| RBAC | Access control |
| Policy Enforcement | Governance |
| Run Approvals | Controlled deployments |
| Remote State | Shared collaboration |

---

## Important Exam Caveats

- Governance improves operational control.
- Collaboration features support teams.
- Policies help standardization.

---

## Quick Revision Notes

- RBAC controls access
- policies enforce governance
- approvals control deployments

---

## Quiz

### Q1
What does RBAC control?

### Q2
Why are governance policies useful?

### Q3
What feature supports controlled deployments?

---

# 8c. Describe How to Organize and Use HCP Terraform Workspaces and Projects

## Theory

Workspaces isolate Terraform deployments.

Projects organize related workspaces.

---

## Example Structure

```text
Project: Production
  ├── networking-workspace
  ├── compute-workspace
  └── database-workspace
```

---

## Important Exam Caveats

- Workspaces separate state.
- Projects group related workspaces.
- Each workspace has independent state.

---

## Quick Revision Notes

- workspaces isolate deployments
- projects organize workspaces
- each workspace has separate state

---

## Quiz

### Q1
What do workspaces isolate?

### Q2
What do projects organize?

### Q3
Does each workspace maintain separate state?

---

# 8d. Configure and Use HCP Terraform Integration

## Theory

HCP Terraform integrates with:
- GitHub,
- GitLab,
- Bitbucket,
- cloud providers,
- policy systems.

---

## Common Integrations

| Integration | Purpose |
|---|---|
| GitHub | VCS workflows |
| AWS | Cloud infrastructure |
| Vault | Secrets management |

---

## Important Exam Caveats

- VCS integration enables automated runs.
- HCP Terraform supports CI/CD workflows.
- Integrations improve automation.

---

## Quick Revision Notes

- HCP Terraform integrates with VCS
- supports automation pipelines
- Vault integration supports secrets management

---

## Quiz

### Q1
What enables automated Terraform runs?

### Q2
Can HCP Terraform integrate with Vault?

### Q3
Why are integrations important?

---

# Final Rapid Revision Sheet

## Most Important Commands

```bash
terraform init
terraform validate
terraform fmt
terraform plan
terraform apply
terraform destroy
terraform state list
terraform state show
terraform import
```

---

## Most Important Concepts

- Terraform is declarative
- Providers are plugins
- State tracks infrastructure
- plan previews changes
- apply changes infrastructure
- data blocks are read-only
- references create implicit dependencies
- depends_on creates explicit dependencies
- modules improve reusability
- remote state preferred for teams

---

# End of Guide

