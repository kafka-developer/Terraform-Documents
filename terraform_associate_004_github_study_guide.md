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
- [6. Terraform State Management](#6-terraform-state-management)
- [7. Maintain Infrastructure with Terraform](#7-maintain-infrastructure-with-terraform)
- [8. HCP Terraform](#8-hcp-terraform)

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

## Key Topics

- Root modules
- Child modules
- Module sources
- Module versioning
- Variable scope

---

## Quick Revision Notes

- modules improve reusability
- root module is current working directory
- child modules are called modules
- modules support versioning

---

# 6. Terraform State Management

## Key Topics

- local backend
- remote backend
- state locking
- drift
- backend block

---

## Quick Revision Notes

- remote state preferred for teams
- locking prevents concurrent changes
- drift = infrastructure changed outside Terraform

---

# 7. Maintain Infrastructure with Terraform

## Key Topics

- terraform import
- terraform state list
- terraform state show
- TF_LOG

---

## Quick Revision Notes

- import adds existing infrastructure to state
- state commands inspect Terraform state
- TF_LOG enables verbose logging

---

# 8. HCP Terraform

## Key Topics

- workspaces
- remote execution
- governance
- VCS integration
- projects

---

## Quick Revision Notes

- HCP Terraform supports collaboration
- workspaces isolate deployments
- VCS integration enables automation

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

