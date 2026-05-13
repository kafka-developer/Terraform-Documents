# Terraform Associate (004) - COMPLETE Study Guide
## All 37 Topics with Theory, Exam Tricks, Questions & Labs

> **Your complete exam preparation in one document**

---

## 📋 Quick Navigation

### [Section 1: IaC with Terraform](#section-1-iac-with-terraform) (3 topics)
- [1a: What is Infrastructure as Code?](#1a-what-is-infrastructure-as-code)
- [1b: Advantages of IaC](#1b-advantages-of-iac)
- [1c: Multi-cloud and Hybrid Cloud](#1c-multi-cloud-and-hybrid-cloud)

### [Section 2: Fundamentals](#section-2-terraform-fundamentals) (4 topics)
- [2a: Providers - Installation and Versioning](#2a-providers---installation-and-versioning)
- [2b: How Providers Work](#2b-how-providers-work)
- [2c: Using Multiple Providers](#2c-using-multiple-providers)
- [2d: Terraform State](#2d-terraform-state)

### [Section 3: Core Workflow](#section-3-core-terraform-workflow) (7 topics)
- [3a: The Terraform Workflow](#3a-the-terraform-workflow)
- [3b: terraform init](#3b-terraform-init)
- [3c: terraform validate](#3c-terraform-validate)
- [3d: terraform plan](#3d-terraform-plan)
- [3e: terraform apply](#3e-terraform-apply)
- [3f: terraform destroy](#3f-terraform-destroy)
- [3g: terraform fmt](#3g-terraform-fmt)

### [Section 4: Configuration](#section-4-terraform-configuration) (8 topics)
- [4a: Resources vs Data Sources](#4a-resources-vs-data-sources)
- [4b: Resource References](#4b-resource-references)
- [4c: Variables and Outputs](#4c-variables-and-outputs)
- [4d: Complex Types](#4d-complex-types)
- [4e: Expressions and Functions](#4e-expressions-and-functions)
- [4f: Resource Dependencies](#4f-resource-dependencies)
- [4g: Custom Validation](#4g-custom-validation)
- [4h: Managing Sensitive Data](#4h-managing-sensitive-data)

### [Section 5: Modules](#section-5-terraform-modules) (4 topics)
- [5a: Module Sources](#5a-module-sources)
- [5b: Variable Scope in Modules](#5b-variable-scope-in-modules)
- [5c: Using Modules](#5c-using-modules)
- [5d: Module Versioning](#5d-module-versioning)

### [Section 6: State Management](#section-6-state-management) (4 topics)
- [6a: Local Backend](#6a-local-backend)
- [6b: State Locking](#6b-state-locking)
- [6c: Remote State](#6c-remote-state)
- [6d: Drift and State Management](#6d-drift-and-state-management)

### [Section 7: Maintenance](#section-7-maintaining-infrastructure) (3 topics)
- [7a: Importing Resources](#7a-importing-resources)
- [7b: State Inspection](#7b-state-inspection)
- [7c: Logging and Debugging](#7c-logging-and-debugging)

### [Section 8: HCP Terraform](#section-8-hcp-terraform) (4 topics)
- [8a: HCP Terraform Basics](#8a-hcp-terraform-basics)
- [8b: Collaboration Features](#8b-collaboration-features)
- [8c: Workspaces and Projects](#8c-workspaces-and-projects)
- [8d: CLI Integration](#8d-cli-integration)

---

# Section 1: IaC with Terraform

## 1a: What is Infrastructure as Code?

**📖 Theory:** IaC = managing infrastructure through version-controlled code instead of manual processes. Enables: version control, repeatability, automation, and treating infrastructure like software.

**🎯 Key Concepts:**
- Infrastructure defined in text files
- Changes tracked in Git
- Automated deployment
- Declarative (desired state) vs Imperative (steps)

**💻 Code:**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}
```

**⚠️ Exam Caveats:**
1. IaC ≠ just files without version control
2. Code documents WHAT, still need docs for WHY
3. Can make manual changes, should update code after

**🤔 Confusions:**
1. "IaC = Cloud only" - No, works on-prem too
2. "IaC = Terraform" - Terraform is ONE IaC tool
3. "Eliminates all documentation" - No, changes type needed

**❓ Questions:**
Q: What describes IaC?
A: Infrastructure definitions in version-controlled files

Q: True/False: IaC eliminates documentation needs
A: False - still need architecture docs and WHY decisions

**🔬 Lab:**
```bash
# Create config
echo 'resource "random_pet" "server" { length = 2 }' > main.tf
terraform init && terraform apply

# Version control
git init && git add . && git commit -m "Initial"

# Make change
echo 'resource "random_pet" "server" { length = 3 }' > main.tf
git diff  # See exact change
terraform apply

# Rollback
git revert HEAD
terraform apply
```

---

## 1b: Advantages of IaC

**📖 Theory:** IaC solves: configuration drift (env becomes unique), slow provisioning (days→minutes), knowledge silos (Bob knows everything), manual errors (typos/forgotten steps).

**🎯 Key Concepts:**
- Automation: deploy without humans
- Version control: track all changes
- Consistency: dev = staging = prod (structure)
- Reusability: write once, use everywhere
- Testing: validate before prod

**💻 Code:**
```hcl
variable "environment" { type = string }
variable "count" { type = number }

resource "random_pet" "servers" {
  count  = var.count
  prefix = var.environment
}
```

```bash
terraform apply -var="environment=dev" -var="count=1"
terraform apply -var="environment=prod" -var="count=5"
# Same structure, different scale
```

**⚠️ Exam Caveats:**
1. "Eliminates errors" - Reduces, doesn't eliminate
2. "Always faster" - First time slower, subsequent faster
3. "Prevents drift" - Only if no manual changes
4. "Version control optional" - Technically yes, loses benefits

**🤔 Confusions:**
1. "IaC = no testing" - Makes testing easier, still needed
2. "Dev = Prod identical" - Structure same, scale different
3. "No manual intervention ever" - Can, but should update code

**❓ Questions:**
Q: Company uses Terraform but devs make console changes. Problem?
A: Configuration drift

Q: True/False: IaC makes first deployment faster
A: False - writing code takes time, subsequent faster

Q: Which are IaC advantages? (Pick 3)
A: Changes reviewable in PRs, Consistent environments, Versioned changes

**🔬 Lab:**
```hcl
# dev.tfvars
environment = "dev"
servers = 1

# prod.tfvars
environment = "prod"
servers = 5
```

```bash
terraform apply -var-file=dev.tfvars
terraform apply -var-file=prod.tfvars
# Notice: same code, different scale
```

---

## 1c: Multi-cloud and Hybrid Cloud

**📖 Theory:** Multi-cloud = AWS+Azure+GCP together. Hybrid = on-prem+cloud. Traditional problem: each cloud has own tool (CloudFormation/ARM/Deployment Manager). Terraform solution: one tool, providers translate to APIs.

**🎯 Key Concepts:**
- Providers = plugins that know cloud APIs
- 3000+ providers available
- Same workflow everywhere (init/plan/apply)
- Cross-provider integration in one config

**💻 Code:**
```hcl
terraform {
  required_providers {
    aws   = { source = "hashicorp/aws" }
    azure = { source = "hashicorp/azurerm" }
  }
}

resource "aws_instance" "app" {
  ami = "ami-12345"
}

resource "azurerm_virtual_machine" "db" {
  name = "db-vm"
}
```

**⚠️ Exam Caveats:**
1. "Terraform creates resources" - No, providers do
2. "Multi-cloud = cloud-agnostic" - No, aws_instance ≠ azurerm_vm
3. "All providers equal" - No, official vs community
4. "Manage anything" - Need network access

**🤔 Confusions:**
1. "Terraform knows all APIs" - No, providers know APIs
2. "Write once, run anywhere" - No, resources differ per cloud
3. "3000 providers built-in" - No, downloaded separately

**❓ Questions:**
Q: How does Terraform support multiple clouds?
A: Providers are plugins that translate to APIs

Q: True/False: AWS Terraform code works on Azure
A: False - different resource types

Q: What downloads during init?
A: Provider plugins

**🔬 Lab:**
```hcl
terraform {
  required_providers {
    random = { source = "hashicorp/random" }
    time   = { source = "hashicorp/time" }
  }
}

resource "random_pet" "name" { length = 2 }
data "time_static" "now" {}

output "result" {
  value = "${random_pet.name.id} at ${data.time_static.now.rfc3339}"
}
```

---

# Section 2: Terraform Fundamentals

## 2a: Providers - Installation and Versioning

**📖 Theory:** Providers = plugins enabling Terraform to talk to APIs. Lifecycle: specify in code → init downloads → stored in .terraform/ → lock file records versions → used in plan/apply.

**🎯 Key Concepts:**
- Version constraints control allowed versions
- `~> 5.0` means >= 5.0.0 and < 6.0.0
- Lock file (.terraform.lock.hcl) must commit to Git
- init -upgrade to update providers

**💻 Code:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.6"
}
```

**⚠️ Exam Caveats:**
1. init installs providers, NOT Terraform itself
2. Lock file goes in Git, .terraform/ doesn't
3. Changed constraint? Need init -upgrade
4. Provider version ≠ Terraform version

**🤔 Confusions:**
1. "~> means latest" - No, specific range
2. "Lock file prevents upgrades" - No, prevents accidental
3. "All providers from HashiCorp" - No, many third-party

**❓ Questions:**
Q: What does init do? (Pick 3)
A: Downloads providers, Creates lock file, Downloads modules

Q: Lock shows 3.5.1, constraint ~> 3.5. What's allowed?
A: 3.5.x only

Q: Changed ~> 4.0 to ~> 5.0, plan still uses 4.5. Why?
A: Need init -upgrade

**🔬 Lab:**
```bash
# Exact version
terraform { required_providers { random = { version = "= 3.5.1" }}}
terraform init
cat .terraform.lock.hcl | grep version

# Pessimistic
terraform { required_providers { random = { version = "~> 3.5" }}}
terraform init -upgrade
```

---

## 2b: How Providers Work

**📖 Theory:** Architecture: Terraform Core → Provider Plugin → Service API. Provider responsibilities: define schemas, CRUD operations, translate HCL to API, handle errors, manage state.

**🎯 Key Concepts:**
- Provider = separate process from Terraform
- RPC communication
- Each provider exposes resource types
- Authentication handled by provider, not Terraform

**💻 Code:**
```hcl
provider "aws" {
  region = "us-west-2"
}

provider "aws" {
  alias  = "dr"
  region = "us-east-1"
}

resource "aws_instance" "primary" {
  ami = "ami-west"
}

resource "aws_instance" "backup" {
  provider = aws.dr
  ami      = "ami-east"
}
```

**⚠️ Exam Caveats:**
1. Defining ≠ using (no resources = no API calls)
2. Multiple default providers = error
3. Credentials in provider = works but insecure
4. Provider makes API calls, not Terraform

**🤔 Confusions:**
1. "First provider is default" - No, default = no alias
2. "Version in provider block" - Wrong, goes in required_providers
3. "One provider per file" - No, can have multiple

**❓ Questions:**
Q: Defined provider, no resources. What happens?
A: Provider initialized, no API calls

Q: Where store credentials?
A: Environment variables

Q: Two provider blocks without alias?
A: Error - multiple defaults

**🔬 Lab:**
```hcl
provider "random" {}
provider "random" { alias = "special" }

resource "random_pet" "normal" { length = 2 }
resource "random_pet" "special" {
  provider = random.special
  length   = 3
}
```

---

## 2c: Using Multiple Providers

**📖 Theory:** Real infrastructure uses multiple services: AWS for compute, Cloudflare for DNS, Datadog for monitoring. Single terraform apply manages all. Cross-provider references enable integration.

**🎯 Key Concepts:**
- Multiple different providers in one config
- Multiple instances of same provider (aliases)
- Cross-provider references work naturally
- One apply manages all

**💻 Code:**
```hcl
resource "aws_instance" "web" {
  ami = "ami-12345"
}

resource "cloudflare_record" "web" {
  name  = "www"
  value = aws_instance.web.public_ip
  type  = "A"
}

resource "datadog_monitor" "web" {
  name = "Web down"
  query = "host:${aws_instance.web.id}"
}
```

**⚠️ Exam Caveats:**
1. Can use different providers together
2. Can use same provider multiple times (alias)
3. Resources reference across providers
4. One apply for everything

**🤔 Confusions:**
1. "Must apply per provider" - No, single apply
2. "Can't mix providers" - Yes you can
3. "Alias required for all" - Only for multiple instances

**❓ Questions:**
Q: AWS + Datadog + GitHub in config. One apply manages?
A: All three at once

Q: How deploy to us-west-2 and us-east-1?
A: Two AWS provider blocks with different aliases

**🔬 Lab:**
```hcl
terraform {
  required_providers {
    random = { source = "hashicorp/random" }
    time   = { source = "hashicorp/time" }
  }
}

resource "random_pet" "name" { length = 2 }
resource "time_sleep" "wait" { create_duration = "5s" }
resource "random_password" "secret" {
  length = 16
  depends_on = [time_sleep.wait]
}
```

---

## 2d: Terraform State

**📖 Theory:** State = JSON file mapping config to real infrastructure. Tracks: resource IDs, attributes, dependencies. Used to: detect changes (plan), determine what to modify (apply). Contains secrets in plaintext!

**🎯 Key Concepts:**
- State = terraform.tfstate file
- Maps config (desired) to reality (current)
- Plan compares state vs config
- Updated after every apply
- **DO NOT edit manually**

**💻 Code:**
State file structure:
```json
{
  "resources": [{
    "type": "aws_instance",
    "name": "web",
    "instances": [{
      "attributes": {
        "id": "i-1234567",
        "public_ip": "54.1.2.3"
      }
    }]
  }]
}
```

**⚠️ Exam Caveats:**
1. State is source of truth for what Terraform manages
2. Contains sensitive data (passwords, keys)
3. Performance: caches attributes, no API query every plan
4. State enables dependency tracking

**🤔 Confusions:**
1. "State = backup" - No, state = current mapping
2. "Can delete state to start over" - Loses track of resources
3. "State auto-syncs" - No, updated via Terraform only

**❓ Questions:**
Q: What is state and why needed?
A: Maps config to infrastructure, determines changes

Q: Where stored by default?
A: terraform.tfstate in current directory

Q: Does state contain secrets?
A: Yes, in plaintext

**🔬 Lab:**
```bash
terraform apply
cat terraform.tfstate  # See JSON
terraform state list   # List resources
terraform state show random_pet.example  # Show one

# Change config
terraform plan  # Shows difference state vs config
terraform apply  # State updated
```

---

# Section 3: Core Terraform Workflow

## 3a: The Terraform Workflow

**📖 Theory:** Core workflow = Write → Plan → Apply. Write: author .tf files. Plan: preview changes. Apply: execute. Optional: Destroy when done.

**🎯 Key Concepts:**
- Write: design infrastructure in HCL
- Plan: review before executing
- Apply: create/modify/destroy
- Iterative: write → plan → review → apply → repeat

**💻 Code:**
```hcl
resource "random_pet" "server" {
  length = 2
}
```

```bash
terraform init   # Setup
terraform plan   # Preview
terraform apply  # Execute
terraform destroy  # Cleanup
```

**⚠️ Exam Caveats:**
1. Plan shows +create, ~modify, -destroy
2. Apply requires confirmation (type "yes")
3. Each step has purpose: write=design, plan=review, apply=execute
4. Workflow is iterative

**🤔 Confusions:**
1. "Plan makes changes" - No, preview only
2. "Apply auto-approves" - No, asks for "yes" (unless -auto-approve)
3. "Must destroy to change" - No, modify in place when possible

**❓ Questions:**
Q: Core Terraform workflow steps?
A: Write, Plan, Apply

Q: What does plan do?
A: Shows what will change, no execution

Q: Must type what to confirm apply?
A: yes

**🔬 Lab:**
```bash
# Write
echo 'resource "random_pet" "test" { length = 2 }' > main.tf

# Plan
terraform init
terraform plan  # See: will create

# Apply
terraform apply  # Type "yes"

# Modify
echo 'resource "random_pet" "test" { length = 3 }' > main.tf
terraform plan  # See: will replace
terraform apply
```

---

## 3b: terraform init

**📖 Theory:** init = first command in new directory. Downloads providers, downloads modules, initializes backend, creates lock file. Safe to run multiple times.

**🎯 Key Concepts:**
- Downloads providers to .terraform/
- Creates .terraform.lock.hcl
- Re-run when: add provider, change backend, add module
- init -upgrade updates providers

**💻 Code:**
```bash
terraform init

# Output:
# Initializing provider plugins...
# - Installing hashicorp/random v3.5.1...
# Terraform has been successfully initialized!
```

**⚠️ Exam Caveats:**
1. First command in new directory
2. Re-run when providers/modules/backend change
3. init -upgrade to update provider versions
4. Safe to run multiple times

**🤔 Confusions:**
1. "Init installs Terraform" - No, installs providers
2. "Only run once" - No, run when config changes
3. "Init makes API calls" - No, just setup

**❓ Questions:**
Q: What does init do?
A: Downloads providers, initializes backend, downloads modules

Q: When re-run init?
A: Add provider, change backend, add module

Q: init -upgrade does what?
A: Updates providers to latest matching constraints

**🔬 Lab:**
```bash
terraform init
ls .terraform/  # See providers directory
cat .terraform.lock.hcl  # See versions

# Add provider
# Edit config: add new provider

terraform init  # Downloads new provider
```

---

## 3c: terraform validate

**📖 Theory:** validate = check syntax and logic without accessing remote services. Fast, no credentials needed. Checks: HCL syntax, resource references, required arguments.

**🎯 Key Concepts:**
- Validates syntax/structure
- No cloud API calls needed
- Runs automatically during plan
- Useful in CI/CD for fast failure

**💻 Code:**
```bash
terraform validate

# Success:
# Success! The configuration is valid.

# Failure:
# Error: Missing required argument
#   on main.tf line 5:
#   resource "random_password" "test" {}
```

**⚠️ Exam Caveats:**
1. Checks syntax, not if resources will work
2. Doesn't verify AMI exists or permissions
3. No credentials needed
4. Faster than plan

**🤔 Confusions:**
1. "Validates resources exist" - No, just config syntax
2. "Needs credentials" - No
3. "Same as plan" - No, plan also checks infrastructure

**❓ Questions:**
Q: What does validate check?
A: Syntax, invalid references, required arguments

Q: Does validate need credentials?
A: No

Q: Does validate check if AMI exists?
A: No, only config structure

**🔬 Lab:**
```hcl
# Bad config
resource "random_password" "bad" {
  # Missing required 'length'
}
```

```bash
terraform validate
# Error: Missing required argument

# Fix
resource "random_password" "good" { length = 16 }
terraform validate
# Success!
```

---

## 3d: terraform plan

**📖 Theory:** plan = preview changes. Compares state (current) vs config (desired). Shows +create, ~modify, -destroy, -/+replace. No changes made. Can save to file with -out.

**🎯 Key Concepts:**
- Reads state file
- Reads .tf files
- Calculates difference
- Shows execution plan
- No changes to infrastructure

**💻 Code:**
```bash
terraform plan

# Output:
#   + resource will be created
#   ~ resource will be updated
#   - resource will be destroyed
#  -/+ resource will be replaced

# Plan: 1 to add, 2 to change, 0 to destroy
```

**⚠️ Exam Caveats:**
1. Plan does NOT make changes
2. + = create, ~ = modify, - = destroy, -/+ = replace
3. Can save: terraform plan -out=file
4. Saved plan applies without confirmation

**🤔 Confusions:**
1. "Plan makes changes" - No, preview only
2. "Must save plan" - No, optional
3. "-/+ means modify" - No, means replace (destroy+create)

**❓ Questions:**
Q: What does plan do?
A: Shows what will change, no execution

Q: What does -/+ mean?
A: Destroy and recreate (replace)

Q: How save plan?
A: terraform plan -out=filename

**🔬 Lab:**
```hcl
resource "random_pet" "test" { length = 2 }
```

```bash
terraform plan  # Shows: +create

terraform apply

# Change length
resource "random_pet" "test" { length = 3 }

terraform plan  # Shows: -/+ replace

# Save plan
terraform plan -out=myplan
terraform apply myplan  # No confirmation needed
```

---

## 3e: terraform apply

**📖 Theory:** apply = execute changes. Shows plan, asks confirmation, makes API calls, updates state. Can use saved plan (no confirmation). Use -auto-approve to skip confirmation (dangerous).

**🎯 Key Concepts:**
- Shows plan first
- Requires "yes" to proceed
- Makes real changes
- Updates state after
- -auto-approve skips confirmation

**💻 Code:**
```bash
terraform apply

# Shows plan
# Asks: Do you want to perform these actions?
# Type: yes

# Executes changes
# Updates state

terraform apply -auto-approve  # Dangerous!
```

**⚠️ Exam Caveats:**
1. Shows plan, asks confirmation
2. Makes real infrastructure changes
3. Updates state file
4. -auto-approve skips confirmation

**🤔 Confusions:**
1. "Apply always asks" - No, -auto-approve or saved plan
2. "Can't undo apply" - Can revert code and re-apply
3. "Apply = deploy" - Yes, but be careful

**❓ Questions:**
Q: What does apply do?
A: Shows plan, asks confirmation, executes, updates state

Q: How skip confirmation?
A: -auto-approve flag

Q: Apply from saved plan needs confirmation?
A: No

**🔬 Lab:**
```bash
terraform apply  # Type "yes"

terraform apply -auto-approve  # No confirmation

terraform plan -out=plan
terraform apply plan  # No confirmation (pre-approved)
```

---

## 3f: terraform destroy

**📖 Theory:** destroy = remove all managed infrastructure. Shows destruction plan, requires confirmation, respects dependencies (correct order), updates state.

**🎯 Key Concepts:**
- Destroys all resources
- Shows plan first
- Requires "yes"
- Respects dependencies
- Updates state

**💻 Code:**
```bash
terraform destroy

# Shows: Plan: 0 to add, 0 to change, 5 to destroy
# Asks: Do you really want to destroy?
# Type: yes
```

**⚠️ Exam Caveats:**
1. Destroys ALL resources
2. Requires confirmation
3. Respects dependencies
4. Can target specific: destroy -target=resource

**🤔 Confusions:**
1. "Can't undo destroy" - True, permanent
2. "Destroy = remove from state only" - No, actually deletes
3. "Destroy removes state file" - No, state remains (empty)

**❓ Questions:**
Q: What does destroy do?
A: Removes all infrastructure, updates state

Q: Does destroy respect dependencies?
A: Yes, correct order

Q: Can destroy specific resource?
A: Yes, destroy -target=resource

**🔬 Lab:**
```bash
# Create resources
terraform apply

# Destroy all
terraform destroy  # Type "yes"

# Destroy specific
terraform destroy -target=random_pet.test
```

---

## 3g: terraform fmt

**📖 Theory:** fmt = auto-format .tf files to standard style. Updates files in place. Useful before committing. fmt -check to verify without modifying.

**🎯 Key Concepts:**
- Formats all .tf files
- Standard style conventions
- Updates in place
- fmt -check to verify only
- fmt -recursive for subdirectories

**💻 Code:**
```bash
# Before:
resource "random_pet"    "test"    {
length=2
}

terraform fmt

# After:
resource "random_pet" "test" {
  length = 2
}
```

**⚠️ Exam Caveats:**
1. Formats files automatically
2. Updates in place (no backup)
3. -check verifies without changing
4. Useful before commit

**🤔 Confusions:**
1. "fmt validates code" - No, just formats
2. "fmt creates backup" - No
3. "fmt changes logic" - No, only formatting

**❓ Questions:**
Q: What does fmt do?
A: Formats .tf files to standard style

Q: Does fmt change logic?
A: No, only formatting/whitespace

Q: How check without modifying?
A: fmt -check

**🔬 Lab:**
```hcl
# Create messy file
resource "random_pet"    "test"    {
length=2
}
```

```bash
terraform fmt -check  # Shows if needs formatting
terraform fmt  # Formats
cat main.tf  # See clean formatting
```

---

# Section 4: Terraform Configuration

## 4a: Resources vs Data Sources

**📖 Theory:** Resources CREATE infrastructure (managed by Terraform). Data sources READ existing infrastructure (query only, not managed).

**🎯 Key Concepts:**
- resource = creates/manages
- data = queries existing
- Resources: create/update/destroy lifecycle
- Data sources: read-only, query during plan

**💻 Code:**
```hcl
# RESOURCE - creates
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}

# DATA SOURCE - reads
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
}

# Use together
resource "aws_instance" "web2" {
  ami = data.aws_ami.ubuntu.id
}
```

**⚠️ Exam Caveats:**
1. Resources managed, data sources queried
2. Data sources run during PLAN
3. destroy doesn't affect data sources
4. Data sources read-only

**🤔 Confusions:**
1. "Data sources create" - No, read only
2. "Can destroy data source" - No, not managed
3. "Data sources in state" - Yes, but not managed resources

**❓ Questions:**
Q: Difference between resource and data source?
A: Resources create, data sources read

Q: What happens to data source on destroy?
A: Nothing, not managed

Q: When do data sources run?
A: During plan

**🔬 Lab:**
```hcl
# Query current time
data "time_static" "now" {}

# Create resource using it
resource "random_pet" "stamped" {
  prefix = "created"
  keepers = {
    time = data.time_static.now.rfc3339
  }
}
```

---

## 4b: Resource References

**📖 Theory:** Reference syntax: resource_type.name.attribute. References create implicit dependencies. Terraform builds dependency graph, ensures correct order.

**🎯 Key Concepts:**
- Syntax: aws_instance.web.id
- References = implicit dependencies
- Dependency graph determines order
- self for within provisioners

**💻 Code:**
```hcl
resource "random_pet" "name" {
  length = 2
}

resource "random_password" "secret" {
  length = 16
  keepers = {
    name = random_pet.name.id  # Reference!
  }
}
# Terraform knows: create name before secret
```

**⚠️ Exam Caveats:**
1. References create dependencies automatically
2. Dependency graph ensures order
3. Circular dependencies = error
4. Can reference across modules

**🤔 Confusions:**
1. "Must use depends_on" - No, references auto-create
2. "References slow" - No, Terraform optimizes
3. "Can't reference data sources" - Yes you can

**❓ Questions:**
Q: How reference another resource?
A: resource_type.name.attribute

Q: What do references create?
A: Implicit dependencies

Q: Syntax for referencing?
A: random_pet.web.id

**🔬 Lab:**
```hcl
resource "random_pet" "first" { length = 1 }
resource "random_pet" "second" {
  prefix = random_pet.first.id
  length = 2
}
resource "random_password" "third" {
  length = 16
  keepers = {
    combined = "${random_pet.first.id}-${random_pet.second.id}"
  }
}
# Order: first → second → third
```

---

## 4c: Variables and Outputs

**📖 Theory:** Variables = inputs. Outputs = return values. Variables: type, default, description. Pass via: -var, -var-file, terraform.tfvars, env vars, default. Precedence: CLI > var-file > terraform.tfvars > env > default.

**🎯 Key Concepts:**
- Variables: input parameters
- Outputs: expose values
- Types: string, number, bool, list, map
- Reference: var.name
- Sensitive flag hides display

**💻 Code:**
```hcl
variable "environment" {
  type        = string
  description = "Environment name"
  default     = "dev"
}

variable "count" {
  type    = number
  default = 1
}

resource "random_pet" "servers" {
  count  = var.count
  prefix = var.environment
}

output "server_names" {
  value = random_pet.servers[*].id
}
```

**⚠️ Exam Caveats:**
1. Precedence: -var > -var-file > terraform.tfvars > env > default
2. sensitive = true hides display (not state!)
3. terraform.tfvars auto-loaded
4. TF_VAR_name for env vars

**🤔 Confusions:**
1. "sensitive removes from state" - No, just hides display
2. "Variables optional" - Yes if default provided
3. "Can't use variables in variables" - Correct, no interpolation

**❓ Questions:**
Q: How reference variable?
A: var.name

Q: Precedence order?
A: -var highest, default lowest

Q: Does sensitive hide from state?
A: No, only from CLI display

**🔬 Lab:**
```hcl
variable "name" { type = string }
variable "count" { type = number, default = 1 }

resource "random_pet" "test" {
  count  = var.count
  prefix = var.name
}

output "results" {
  value = random_pet.test[*].id
}
```

```bash
terraform apply -var="name=prod" -var="count=3"
terraform output
```

---

## 4d: Complex Types

**📖 Theory:** Complex types: list (ordered), map (key-value), object (structured), set (unique). Access: list[0], map["key"], object.attribute.

**🎯 Key Concepts:**
- list: ordered collection
- map: key-value pairs
- object: structured with named attributes
- set: unordered unique

**💻 Code:**
```hcl
variable "zones" {
  type    = list(string)
  default = ["us-west-2a", "us-west-2b"]
}

variable "tags" {
  type = map(string)
  default = {
    Environment = "prod"
    Team        = "platform"
  }
}

variable "config" {
  type = object({
    name  = string
    count = number
  })
}
```

**⚠️ Exam Caveats:**
1. Lists ordered, sets unordered
2. Access: list[index], map["key"]
3. Objects have defined structure
4. Type constraints validated

**🤔 Confusions:**
1. "List = array" - Similar concept
2. "Map = object" - Different, object has schema
3. "Set = list" - No, sets unique and unordered

**❓ Questions:**
Q: Difference list vs map?
A: List ordered by index, map by key

Q: How access list element?
A: var.list[0]

Q: How access map value?
A: var.map["key"]

**🔬 Lab:**
```hcl
variable "servers" {
  type = list(string)
  default = ["web", "app", "db"]
}

variable "sizes" {
  type = map(number)
  default = {
    web = 1
    app = 2
    db  = 3
  }
}

resource "random_pet" "servers" {
  count  = length(var.servers)
  prefix = var.servers[count.index]
  length = var.sizes[var.servers[count.index]]
}
```

---

## 4e: Expressions and Functions

**📖 Theory:** Expressions: interpolation "${ }", conditionals "?:", for loops. Functions: file(), join(), concat(), lookup(), length(), merge(). Built-in only, can't create custom.

**🎯 Key Concepts:**
- String interpolation: "${var.name}"
- Conditionals: condition ? true_val : false_val
- For: [for x in list : transform]
- Common functions: file, join, concat, length

**💻 Code:**
```hcl
# Interpolation
name = "server-${var.environment}"

# Conditional
type = var.env == "prod" ? "t2.large" : "t2.micro"

# For expression
names = [for s in var.servers : upper(s)]

# Functions
data = file("config.txt")
combined = concat(var.list1, var.list2)
count = length(var.items)
```

**⚠️ Exam Caveats:**
1. No custom functions
2. For expressions powerful
3. Conditionals: condition ? true : false
4. Functions take arguments

**🤔 Confusions:**
1. "Can create functions" - No, built-in only
2. "For = for_each" - Different concepts
3. "Interpolation always needed" - No, direct reference ok

**❓ Questions:**
Q: How conditional in Terraform?
A: condition ? true_val : false_val

Q: How read file?
A: file("path")

Q: Can create custom functions?
A: No, built-in only

**🔬 Lab:**
```hcl
variable "env" { default = "dev" }
variable "items" {
  default = ["a", "b", "c"]
}

locals {
  size = var.env == "prod" ? "large" : "small"
  upper_items = [for i in var.items : upper(i)]
  count = length(var.items)
}

output "results" {
  value = {
    size  = local.size
    items = local.upper_items
    count = local.count
  }
}
```

---

## 4f: Resource Dependencies

**📖 Theory:** Dependencies: implicit (from references) vs explicit (depends_on). Implicit preferred. Terraform builds dependency graph, determines order. Use depends_on for hidden dependencies.

**🎯 Key Concepts:**
- Implicit: created by references
- Explicit: depends_on meta-argument
- Dependency graph determines order
- Prefer implicit over explicit

**💻 Code:**
```hcl
# Implicit - reference creates dependency
resource "random_pet" "name" { length = 2 }
resource "random_password" "secret" {
  length = 16
  keepers = {
    name = random_pet.name.id  # Implicit!
  }
}

# Explicit - hidden dependency
resource "random_integer" "priority" {
  min = 1
  max = 100
  depends_on = [random_pet.name]  # Explicit
}
```

**⚠️ Exam Caveats:**
1. References auto-create dependencies
2. depends_on for hidden dependencies
3. Circular dependencies = error
4. Prefer implicit (clearer)

**🤔 Confusions:**
1. "Must use depends_on" - No, references usually enough
2. "depends_on = string" - No, list of resources
3. "Order of resources matters" - No, dependencies do

**❓ Questions:**
Q: Implicit vs explicit dependencies?
A: Implicit from references, explicit from depends_on

Q: When use depends_on?
A: Hidden dependencies not represented by references

Q: Syntax for depends_on?
A: depends_on = [resource.type.name]

**🔬 Lab:**
```hcl
resource "random_pet" "first" { length = 1 }

# Implicit
resource "random_pet" "second" {
  prefix = random_pet.first.id
}

# Explicit
resource "random_integer" "third" {
  min = 1
  max = 100
  depends_on = [random_pet.first]
}
```

---

## 4g: Custom Validation

**📖 Theory:** Validation blocks in variables enforce rules. Condition = boolean expression, error_message shown on failure. Validates during plan.

**🎯 Key Concepts:**
- validation block in variable
- condition: boolean expression
- error_message: shown on failure
- Validates during plan

**💻 Code:**
```hcl
variable "environment" {
  type = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod"
  }
}

variable "count" {
  type = number
  
  validation {
    condition     = var.count >= 1 && var.count <= 10
    error_message = "Must be 1-10"
  }
}
```

**⚠️ Exam Caveats:**
1. Validates during plan
2. condition = must be true
3. Can have multiple validations
4. Fails with error_message

**🤔 Confusions:**
1. "Validation = type checking" - No, additional rules
2. "Validation optional" - Yes, but useful
3. "Can validate resources" - No, variables only

**❓ Questions:**
Q: How validate variable?
A: validation block with condition and error_message

Q: When does validation run?
A: During plan

Q: Can have multiple validations?
A: Yes, per variable

**🔬 Lab:**
```hcl
variable "name" {
  type = string
  
  validation {
    condition     = can(regex("^[a-z]{3,10}$", var.name))
    error_message = "Must be 3-10 lowercase letters"
  }
}

variable "count" {
  type = number
  
  validation {
    condition     = var.count % 2 == 0 || var.count == 1
    error_message = "Must be even or 1"
  }
}
```

```bash
terraform apply -var="name=ABC"  # Fails validation
terraform apply -var="count=3"   # Fails validation
terraform apply -var="name=test" -var="count=2"  # Works
```

---

## 4h: Managing Sensitive Data

**📖 Theory:** State contains ALL data including secrets. sensitive flag hides from display (not state!). Protect state: encryption, access control, never commit. External secrets: Vault, env vars.

**🎯 Key Concepts:**
- State has secrets in plaintext
- sensitive = hides display only
- Never commit state to Git
- Use remote state with encryption
- External secret stores: Vault

**💻 Code:**
```hcl
variable "password" {
  type      = string
  sensitive = true
}

resource "random_password" "secret" {
  length  = 16
  special = true
}

output "password" {
  value     = random_password.secret.result
  sensitive = true  # Hides from CLI
}
```

**⚠️ Exam Caveats:**
1. State has secrets in plaintext
2. sensitive flag only hides display
3. Must protect state file access
4. Never commit state to Git

**🤔 Confusions:**
1. "sensitive removes from state" - No, just hides display
2. "State file safe" - No, contains secrets
3. "Vault replaces state" - No, Vault for secrets, state still needed

**❓ Questions:**
Q: Why protect state files?
A: Contains all data including secrets in plaintext

Q: Does sensitive hide from state?
A: No, only from CLI display

Q: Should commit state to Git?
A: No, use remote state

**🔬 Lab:**
```hcl
variable "secret" {
  type      = string
  sensitive = true
}

resource "random_password" "pass" {
  length  = 16
  special = true
}

output "password" {
  value     = random_password.pass.result
  sensitive = true
}
```

```bash
terraform apply
terraform output  # Shows <sensitive>
terraform output password  # Shows actual value
cat terraform.tfstate | grep result  # See plaintext!
```

---

# Section 5: Terraform Modules

## 5a: Module Sources

**📖 Theory:** Module = container for resources. Sources: local (./path), Terraform Registry (namespace/name/provider), Git (git::url), HTTP, S3. Downloaded during init.

**🎯 Key Concepts:**
- Local: ./modules/vpc
- Registry: terraform-aws-modules/vpc/aws
- Git: git::https://github.com/org/repo
- HTTP: https://example.com/module.zip
- S3: s3::https://bucket/module.zip

**💻 Code:**
```hcl
# Local
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
}

# Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
}

# Git
module "vpc" {
  source = "git::https://github.com/org/modules.git//vpc"
}
```

**⚠️ Exam Caveats:**
1. version only works with Registry
2. Git uses ref= for branch/tag
3. //subdir for subdirectories in Git
4. init downloads modules

**🤔 Confusions:**
1. "All modules in Registry" - No, can be local/Git
2. "version everywhere" - No, Registry only
3. "Modules = providers" - Different concepts

**❓ Questions:**
Q: Valid module sources? (Pick 3)
A: Local path, Terraform Registry, Git

Q: How specify subdirectory in Git?
A: //subdir after URL

Q: Where does version work?
A: Terraform Registry only

**🔬 Lab:**
```bash
mkdir -p modules/greeter
cat > modules/greeter/main.tf << EOF
variable "name" {}
output "greeting" {
  value = "Hello, \${var.name}!"
}
EOF

# Use it
module "greet" {
  source = "./modules/greeter"
  name   = "World"
}

output "message" {
  value = module.greet.greeting
}
```

---

## 5b: Variable Scope in Modules

**📖 Theory:** Each module has isolated namespace. Parent passes data via module block arguments. Child defines variables to accept. No automatic inheritance.

**🎯 Key Concepts:**
- Modules isolated
- Parent passes explicitly
- Child declares variables
- No automatic variable flow

**💻 Code:**
```hcl
# Parent
variable "environment" { default = "prod" }

module "server" {
  source = "./modules/server"
  
  environment = var.environment  # Explicit pass
}

# Child (modules/server/variables.tf)
variable "environment" {
  type = string
}
```

**⚠️ Exam Caveats:**
1. Variables don't flow automatically
2. Must explicitly pass
3. Child must declare variables
4. Complete isolation

**🤔 Confusions:**
1. "Child sees parent vars" - No, must pass
2. "Implicit inheritance" - No, explicit only
3. "Variables global" - No, scoped to module

**❓ Questions:**
Q: Can child module access parent variables?
A: No, must explicitly pass

Q: How pass data to module?
A: Module block arguments

Q: Do variables inherit?
A: No, explicit passing required

**🔬 Lab:**
```hcl
# Parent
variable "name" { default = "app" }

# Child CAN'T see var.name automatically
module "server" {
  source = "./modules/server"
  name   = var.name  # Must pass explicitly
}

# modules/server/variables.tf
variable "name" {}  # Must declare
```

---

## 5c: Using Modules

**📖 Theory:** module block calls module. Arguments = inputs. Access outputs: module.name.output_name. Can use count/for_each with modules.

**🎯 Key Concepts:**
- module "name" { source = "..." }
- Pass inputs via arguments
- Access: module.name.output
- count/for_each supported

**💻 Code:**
```hcl
module "server" {
  source = "./modules/server"
  
  name  = "web"
  count = 3
}

# Access output
output "server_names" {
  value = module.server.names
}

# With count
module "servers" {
  count  = 3
  source = "./modules/server"
  name   = "server-${count.index}"
}

# Access with count
output "first" {
  value = module.servers[0].name
}
```

**⚠️ Exam Caveats:**
1. Module outputs must be declared
2. count creates array
3. for_each creates map
4. depends_on works with modules

**🤔 Confusions:**
1. "Outputs automatic" - No, must declare
2. "Module = resource" - Different, module contains resources
3. "Can't use count" - Yes you can

**❓ Questions:**
Q: How pass value to module?
A: Module block arguments

Q: How access module output?
A: module.name.output_name

Q: Can use count with modules?
A: Yes

**🔬 Lab:**
```hcl
# Create module
mkdir -p modules/pet
cat > modules/pet/main.tf << 'EOF'
variable "prefix" {}
resource "random_pet" "pet" {
  prefix = var.prefix
}
output "name" {
  value = random_pet.pet.id
}
EOF

# Use it
module "pets" {
  count  = 3
  source = "./modules/pet"
  prefix = "pet${count.index}"
}

output "all_pets" {
  value = module.pets[*].name
}
```

---

## 5d: Module Versioning

**📖 Theory:** version argument for Registry modules. Constraints: =, >=, ~>. Git uses ref= for tags. Lock file tracks versions. init -upgrade updates.

**🎯 Key Concepts:**
- version for Registry only
- ~> 1.0 = >= 1.0, < 2.0
- Git: ref=v1.0.0
- init -upgrade updates

**💻 Code:**
```hcl
# Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
}

# Git with tag
module "vpc" {
  source = "git::https://github.com/org/modules.git//vpc?ref=v1.0.0"
}
```

**⚠️ Exam Caveats:**
1. version only Registry
2. Git uses ref=
3. ~> for flexible versions
4. Lock file tracks

**🤔 Confusions:**
1. "version everywhere" - No, Registry only
2. "Git version automatic" - No, use ref=
3. "Can't upgrade" - init -upgrade

**❓ Questions:**
Q: How ensure module stays 1.x?
A: version = "~> 1.0"

Q: Where does version argument work?
A: Terraform Registry only

Q: How specify Git version?
A: ref=v1.0.0

**🔬 Lab:**
```hcl
# Simulate versioning with local
mkdir -p modules/v1 modules/v2

cat > modules/v1/main.tf << 'EOF'
resource "random_pet" "pet" { length = 2 }
output "name" { value = random_pet.pet.id }
EOF

cat > modules/v2/main.tf << 'EOF'
resource "random_pet" "pet" { length = 3 }
output "name" { value = random_pet.pet.id }
EOF

# Use v1
module "app" { source = "./modules/v1" }

# Upgrade to v2
module "app" { source = "./modules/v2" }
```

---

# Section 6: State Management

## 6a: Local Backend

**📖 Theory:** Local backend = default. State in terraform.tfstate on disk. Good for: learning, solo work. Bad for: teams, CI/CD, production. No locking, no sharing.

**🎯 Key Concepts:**
- Default backend
- State on disk
- terraform.tfstate file
- No locking
- Not for teams

**💻 Code:**
```hcl
# No backend block = local
resource "random_pet" "example" {
  length = 2
}
```

```bash
terraform init
terraform apply
ls  # See terraform.tfstate
```

**⚠️ Exam Caveats:**
1. Default when no backend specified
2. State in current directory
3. No locking
4. Not suitable for teams

**🤔 Confusions:**
1. "Local = bad" - No, fine for solo/learning
2. "Must configure backend" - No, local is default
3. "Local has locking" - No

**❓ Questions:**
Q: What is local backend?
A: Default, state on disk, no locking

Q: When use local backend?
A: Solo work, learning, testing

Q: Does local backend have locking?
A: No

**🔬 Lab:**
```bash
terraform apply
cat terraform.tfstate  # See state
# No locking - can run multiple applies (dangerous)
```

---

## 6b: State Locking

**📖 Theory:** Locking prevents concurrent modifications. Protects from corruption. Automatic with supported backends. S3 needs DynamoDB for locking.

**🎯 Key Concepts:**
- Prevents concurrent writes
- Automatic with backends
- S3 + DynamoDB
- force-unlock if stuck

**💻 Code:**
```hcl
# S3 with locking
terraform {
  backend "s3" {
    bucket         = "my-state"
    key            = "terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-locks"  # For locking!
  }
}
```

**⚠️ Exam Caveats:**
1. Not all backends support locking
2. S3 needs DynamoDB
3. Automatic when supported
4. force-unlock if stuck (dangerous)

**🤔 Confusions:**
1. "All backends lock" - No, check docs
2. "Manual locking" - No, automatic
3. "S3 auto-locks" - No, needs DynamoDB

**❓ Questions:**
Q: What is state locking?
A: Prevents concurrent state modifications

Q: Does S3 backend support locking?
A: Yes, with DynamoDB table

Q: Is locking automatic?
A: Yes, if backend supports it

**🔬 Lab:**
```bash
# Simulate locking issue
terraform apply &  # Background
terraform apply    # Immediate

# If backend supports locking:
# Error: state locked

# If no locking (local):
# Both run - dangerous!
```

---

## 6c: Remote State

**📖 Theory:** Remote state = state stored remotely (S3, Terraform Cloud, Azure, GCS). Benefits: sharing, locking, encryption, versioning. Configure with backend block.

**🎯 Key Concepts:**
- State stored remotely
- Enables team collaboration
- Locking and encryption
- backend block configures

**💻 Code:**
```hcl
terraform {
  backend "s3" {
    bucket  = "my-terraform-state"
    key     = "prod/terraform.tfstate"
    region  = "us-west-2"
    encrypt = true
  }
}
```

**⚠️ Exam Caveats:**
1. init after adding backend
2. init -migrate-state to move
3. Backend config can't use variables
4. One backend per config

**🤔 Confusions:**
1. "Can use variables in backend" - No
2. "Multiple backends" - No, one only
3. "Backend = provider" - Different concepts

**❓ Questions:**
Q: How configure remote state?
A: backend block in terraform {}

Q: Can use variables in backend?
A: No

Q: Benefits of remote state?
A: Sharing, locking, encryption, versioning

**🔬 Lab:**
```hcl
# Start local
terraform apply

# Add backend
terraform {
  backend "local" {
    path = "custom/terraform.tfstate"
  }
}

terraform init -migrate-state
# State moved to custom location
```

---

## 6d: Drift and State Management

**📖 Theory:** Drift = real infrastructure ≠ state. plan detects drift. refresh-only updates state without changes. moved block refactors without recreating. removed block removes from state, keeps infrastructure.

**🎯 Key Concepts:**
- Drift = reality ≠ state
- plan detects automatically
- refresh-only updates state
- moved refactors without recreate
- removed keeps infrastructure

**💻 Code:**
```bash
# Detect drift
terraform plan
# Shows changes outside Terraform

# Update state
terraform apply -refresh-only
```

```hcl
# Refactor
moved {
  from = random_pet.old_name
  to   = random_pet.new_name
}

# Remove from management
removed {
  from = random_pet.example
  lifecycle { destroy = false }
}
```

**⚠️ Exam Caveats:**
1. plan refreshes automatically
2. refresh-only doesn't change infrastructure
3. moved updates references
4. removed keeps infrastructure

**🤔 Confusions:**
1. "Drift = error" - No, detectable and fixable
2. "refresh makes changes" - No, updates state only
3. "moved destroys" - No, updates state

**❓ Questions:**
Q: How detect drift?
A: terraform plan (refreshes automatically)

Q: What does refresh-only do?
A: Updates state to match reality, no infrastructure changes

Q: What does moved do?
A: Refactors without recreating

**🔬 Lab:**
```hcl
resource "random_pet" "test" { length = 2 }
```

```bash
terraform apply

# Simulate drift (modify config)
# length = 3

terraform plan
# Shows: will replace

# Or refresh state
terraform apply -refresh-only
```

---

# Section 7: Maintaining Infrastructure

## 7a: Importing Resources

**📖 Theory:** import brings existing resources under Terraform management. Process: write resource block → import command → check plan → add missing attributes.

**🎯 Key Concepts:**
- Write resource block first
- terraform import <address> <id>
- Updates state only
- Then add attributes to match

**💻 Code:**
```hcl
# 1. Write block
resource "random_pet" "imported" {}
```

```bash
# 2. Import
terraform import random_pet.imported "existing-pet-name"

# 3. Check
terraform plan
# Shows missing attributes

# 4. Add attributes
resource "random_pet" "imported" {
  length = 2
}

# 5. Verify
terraform plan
# Should show no changes
```

**⚠️ Exam Caveats:**
1. Write resource block FIRST
2. Import updates state, not config
3. Must add attributes manually
4. Each resource type has ID format

**🤔 Confusions:**
1. "Import generates config" - No (except 1.5+ with generate-config)
2. "Import modifies resource" - No, just state
3. "Can import anything" - No, must be supported

**❓ Questions:**
Q: What must do before import?
A: Write resource block in config

Q: Does import create config?
A: No, only updates state

Q: After import, plan shows changes. Why?
A: Missing attributes in config

**🔬 Lab:**
```bash
# Create resource
resource "random_pet" "original" { length = 2 }
terraform apply

# Remove from state
terraform state rm random_pet.original

# Import it back
resource "random_pet" "imported" {}
terraform import random_pet.imported $(terraform output -raw original_name)

# Add attributes
terraform plan  # Shows needed attributes
```

---

## 7b: State Inspection

**📖 Theory:** state commands inspect/modify state. list = show all. show = details. mv = rename. rm = remove from state (keeps infrastructure).

**🎯 Key Concepts:**
- state list: show all resources
- state show: resource details
- state mv: rename
- state rm: remove from state
- output: show outputs

**💻 Code:**
```bash
# List all
terraform state list

# Show details
terraform state show random_pet.example

# Rename
terraform state mv random_pet.old random_pet.new

# Remove from state
terraform state rm random_pet.example
# Resource still exists, Terraform forgets

# Show outputs
terraform output
terraform output name
```

**⚠️ Exam Caveats:**
1. state commands modify state
2. rm removes from state, not cloud
3. Changes immediate
4. Backup first for safety

**🤔 Confusions:**
1. "state rm destroys" - No, removes from management
2. "state show changes" - No, read-only
3. "Can't undo state mv" - Can mv back

**❓ Questions:**
Q: How see resource details?
A: terraform state show <address>

Q: What does state rm do?
A: Removes from state, keeps infrastructure

Q: How rename resource?
A: terraform state mv old new

**🔬 Lab:**
```bash
terraform apply

# Inspect
terraform state list
terraform state show random_pet.example

# Rename
terraform state mv random_pet.example random_pet.renamed
terraform state list  # See new name

# Remove
terraform state rm random_pet.renamed
terraform state list  # Gone from state
```

---

## 7c: Logging and Debugging

**📖 Theory:** TF_LOG enables logging. Levels: TRACE (most), DEBUG, INFO, WARN, ERROR (least). TF_LOG_PATH writes to file. Use for troubleshooting.

**🎯 Key Concepts:**
- TF_LOG environment variable
- Levels: TRACE, DEBUG, INFO, WARN, ERROR
- TF_LOG_PATH for file
- Shows API calls

**💻 Code:**
```bash
# Enable logging
export TF_LOG=DEBUG
terraform apply

# Log to file
export TF_LOG=TRACE
export TF_LOG_PATH=./terraform.log
terraform apply
cat terraform.log

# Disable
unset TF_LOG
unset TF_LOG_PATH
```

**⚠️ Exam Caveats:**
1. TRACE most verbose
2. Logs can contain sensitive data
3. Use for debugging
4. Disable after troubleshooting

**🤔 Confusions:**
1. "INFO most detailed" - No, TRACE is
2. "Logging always on" - No, opt-in
3. "Logs safe to share" - No, may have secrets

**❓ Questions:**
Q: How enable detailed logging?
A: Set TF_LOG environment variable

Q: Most verbose log level?
A: TRACE

Q: Where write logs to file?
A: TF_LOG_PATH

**🔬 Lab:**
```bash
# Enable
export TF_LOG=DEBUG
export TF_LOG_PATH=debug.log

terraform plan 2>&1 | tee output.txt

# Check logs
cat debug.log | grep -i "api"

# Disable
unset TF_LOG TF_LOG_PATH
```

---

# Section 8: HCP Terraform

## 8a: HCP Terraform Basics

**📖 Theory:** HCP Terraform (formerly Cloud) = SaaS platform. Features: remote state, remote execution, VCS integration, collaboration. Workspace ≠ CLI workspace (different concepts).

**🎯 Key Concepts:**
- SaaS platform
- Remote execution
- VCS integration
- Workspace = complete environment
- Free tier available

**💻 Code:**
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

**⚠️ Exam Caveats:**
1. HCP workspace ≠ CLI workspace
2. HCP workspace = project/environment
3. Remote execution in HCP
4. Free tier available

**🤔 Confusions:**
1. "Workspace = CLI workspace" - No, different
2. "HCP = enterprise only" - No, free tier
3. "Runs locally" - No, remote execution

**❓ Questions:**
Q: Difference HCP vs CLI workspaces?
A: CLI = state variants, HCP = complete environments

Q: Where do runs execute?
A: In HCP infrastructure

Q: Is HCP free?
A: Free tier available

**🔬 Lab:**
```hcl
# Simulate HCP workflow
locals {
  workspace = "prod"
  organization = "mycompany"
}

resource "random_pet" "app" {
  prefix = local.workspace
}

output "info" {
  value = {
    org = local.organization
    workspace = local.workspace
    app = random_pet.app.id
  }
}
```

---

## 8b: Collaboration Features

**📖 Theory:** Features: Policy as Code (Sentinel/OPA), cost estimation, drift detection, private registry, RBAC, change requests.

**🎯 Key Concepts:**
- Policy as Code (Sentinel)
- Cost estimation before apply
- Drift detection (health checks)
- Private module registry
- RBAC for access
- Change requests (workflow)

**💻 Code:**
```hcl
# Policy (pseudocode)
policy "require-tags" {
  rule = all resources have tags
  enforcement = "hard-mandatory"
}

# Cost shown automatically:
# New: +$100/mo
# Delta: +$50/mo
```

**⚠️ Exam Caveats:**
1. Policies: advisory vs mandatory
2. Cost estimation automatic
3. Drift detection scheduled
4. Private registry for modules

**🤔 Confusions:**
1. "Policy = validation" - Similar but different
2. "Cost = billing" - No, estimation
3. "Drift = error" - No, detection/alert

**❓ Questions:**
Q: What are HCP governance features?
A: Policy as Code, cost estimation, drift detection, RBAC

Q: Difference advisory vs mandatory policy?
A: Advisory = warning, mandatory = hard fail

Q: What is drift detection?
A: Scheduled checks for infrastructure changes

**🔬 Lab:**
```hcl
# Simulate policy checking
variable "require_tags" { default = true }

resource "random_pet" "server" {
  length = 2
}

# Pretend policy check
output "policy_check" {
  value = var.require_tags ? "Tags required!" : "No tags needed"
}
```

---

## 8c: Workspaces and Projects

**📖 Theory:** Projects group workspaces. Workspaces = complete environment. Variable sets share across workspaces. Run triggers chain workspaces.

**🎯 Key Concepts:**
- Projects group related workspaces
- Workspace = environment/component
- Variable sets share variables
- Run triggers chain deployments

**💻 Code:**
```hcl
# Project structure:
Project: "MyApp"
├── Workspace: network
├── Workspace: compute
└── Workspace: database

# Variable Set: "AWS Creds"
# Applied to all workspaces in project
```

**⚠️ Exam Caveats:**
1. Projects group workspaces
2. Workspaces isolated by default
3. Variable sets share config
4. Run triggers create dependencies

**🤔 Confusions:**
1. "Workspace = folder" - No, complete environment
2. "Projects = modules" - No, organizational
3. "Variables auto-share" - No, use variable sets

**❓ Questions:**
Q: How organize infrastructure?
A: Projects group workspaces

Q: How share variables across workspaces?
A: Variable sets

Q: What are run triggers?
A: Chain workspace deployments

**🔬 Lab:**
```hcl
# Simulate workspace isolation
locals {
  workspace = "prod-network"
  project = "myapp"
}

# Workspace variables
variable "vpc_cidr" { default = "10.0.0.0/16" }

# Shared variables (simulated)
variable "aws_region" { default = "us-west-2" }
```

---

## 8d: CLI Integration

**📖 Theory:** cloud {} block configures CLI. terraform login authenticates. Remote operations execute in HCP. Can force local execution.

**🎯 Key Concepts:**
- cloud {} in terraform block
- terraform login for auth
- Remote execution by default
- CLI-driven workflow possible

**💻 Code:**
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

```bash
# Authenticate
terraform login

# Initialize
terraform init

# Plan (runs remotely)
terraform plan

# Apply (runs remotely)
terraform apply
```

**⚠️ Exam Caveats:**
1. cloud {} configures integration
2. login saves token
3. Operations execute remotely
4. Can force local with settings

**🤔 Confusions:**
1. "Runs locally" - No, remote by default
2. "Must use web UI" - No, CLI works
3. "Can't run local" - Can force local

**❓ Questions:**
Q: How configure CLI for HCP?
A: cloud {} block in terraform settings

Q: How authenticate?
A: terraform login

Q: Where do operations run?
A: Remotely in HCP

**🔬 Lab:**
```hcl
# Simulate HCP config
terraform {
  # cloud {} would go here
  required_providers {
    random = { source = "hashicorp/random" }
  }
}

resource "random_pet" "app" {
  length = 2
}

# In real HCP:
# - Runs execute remotely
# - Logs visible in UI
# - State stored in HCP
```

---

## 🎯 Study Summary

**You now have:**
- ✅ All 37 exam topics covered
- ✅ Theory and concepts
- ✅ Exam tricks and caveats
- ✅ Common confusions addressed
- ✅ Practice questions with answers
- ✅ Hands-on labs for each topic

**Exam Strategy:**
1. Read questions carefully - they twist concepts
2. Eliminate obvious wrong answers
3. Watch for "always", "never", "must" - usually wrong
4. Know differences: provider vs Terraform, resource vs data, HCP vs CLI workspaces
5. Understand state management thoroughly
6. Practice the workflow until automatic

**Time Management:**
- 60 minutes, ~57 questions = ~1 min each
- Flag hard questions, return later
- Don't spend >2 min on any question
- Answer everything (no penalty for wrong)

**Good luck! 🚀**

You're ready when you can:
- Explain all 37 topics
- Avoid the exam tricks
- Answer practice questions correctly
- Complete labs without reference

---

*Document created for Terraform Associate (004) exam preparation*
*All 37 official exam objectives covered*
*Study + Practice + Pass*
