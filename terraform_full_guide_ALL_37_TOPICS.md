# HashiCorp Certified: Terraform Associate (004)
## Complete Study Guide — Exam-Essential Topics Only

**Exam Version:** Terraform 1.12
**Format:** 60 minutes · Multiple choice / True-False / Multi-select · ~57 questions
**Pass Score:** Not published (typically ~70%)
**Validity:** 2 years

> ⭐ **004 NEW topics**: Ephemeral values & write-only arguments · Custom conditions (preconditions, postconditions, checks) · depends_on & create_before_destroy lifecycle emphasis · HCP Terraform Projects

---

## HOW TO USE THIS GUIDE

Each topic follows: Concept → Why it matters → HCL/CLI syntax → Exam traps → Practice questions

The exam tests understanding of behaviour, not syntax memorisation. Questions are scenario-based: "what happens when…", "which command…", "which statement is true about…". Study the why, not just the what.

## TABLE OF CONTENTS

| # | Section | Weight | Topics |
|---|---------|--------|--------|
| 1 | [Infrastructure as Code with Terraform](#section-1) | ~8% | 1a, 1b, 1c |
| 2 | [Terraform Fundamentals](#section-2) | ~11% | 2a, 2b, 2c, 2d |
| 3 | [Core Terraform Workflow](#section-3) | ~19% | 3a-3g |
| 4 | [Terraform Configuration](#section-4) | ~22% | 4a-4h |
| 5 | [Terraform Modules](#section-5) | ~11% | 5a-5d |
| 6 | [Terraform State Management](#section-6) | ~11% | 6a-6d |
| 7 | [Maintain Infrastructure with Terraform](#section-7) | ~8% | 7a-7c |
| 8 | [HCP Terraform](#section-8) | ~11% | 8a-8d |

---

<a id="section-1"></a>
# SECTION 1 — Infrastructure as Code with Terraform (~8%)

---

## 1a. What is Infrastructure as Code (IaC)?

### Concept

**Infrastructure as Code (IaC)** is the practice of defining and managing infrastructure resources — servers, networks, databases, DNS records, load balancers — using human-readable configuration files instead of manual processes (clicking through a console, running ad-hoc scripts).

The core insight behind IaC is that infrastructure should be treated with the same discipline as application code: version-controlled, reviewed, tested, and deployed through automated pipelines. This eliminates "snowflake servers" — unique, hand-configured machines that no one fully understands and can never be safely reproduced.

### Why Terraform specifically?

Terraform is a **declarative** IaC tool. You describe the *desired end state* of your infrastructure ("I want 3 EC2 instances with these properties") and Terraform figures out the operations needed to achieve it. This is different from *imperative* tools (like Ansible in procedural mode) where you describe the *steps* ("run this script, install this package").

Terraform is also **provider-agnostic**: the same workflow and language (HCL) applies whether you're provisioning AWS, Azure, GCP, Kubernetes, GitHub, Datadog, or hundreds of other providers. This is Terraform's key differentiator.

### Benefits of IaC (exam frequently tests these)

| Benefit | Explanation |
|---|---|
| **Reproducibility** | The same configuration always produces identical infrastructure. |
| **Version control** | Infrastructure changes are tracked as Git commits — who changed what, when, why. |
| **Collaboration** | Teams review infrastructure changes via pull requests before production. |
| **Automation** | Infrastructure provisioning integrates into CI/CD pipelines. |
| **Self-documentation** | The config is the documentation. |
| **Idempotency** | Running Terraform multiple times against unchanged config produces no changes. Terraform only acts when there is drift. |
| **Disaster recovery** | Lost environment? Re-run terraform apply. Infrastructure is recreated from config in minutes. |

### Exam Traps

- The exam distinguishes **declarative** (describe desired state — tool figures out steps) from **imperative** (describe exact steps). Terraform is declarative.
- **Idempotent** means running the same operation multiple times produces the same result. If infra matches config, terraform apply makes zero changes. Frequently tested term.
- Terraform is **not a configuration management tool**. It provisions infrastructure. For configuring software on that infrastructure, use Ansible, Chef, or Puppet.

### Practice Questions

**Q1:** Which statement best describes Terraform's approach to IaC?
A) Terraform executes shell scripts in sequence to build infrastructure
B) Terraform describes desired infrastructure state and determines the steps to reach it
C) Terraform copies an existing environment to create a new one
D) Terraform requires you to specify the exact API calls for each provider
Answer: B — Terraform is declarative.

**Q2:** True or False: Running terraform apply twice with an unchanged configuration will provision duplicate resources.
Answer: False — Terraform is idempotent. The second apply detects no drift and makes no changes.

---

## 1b. Terraform vs Other IaC Tools

### Concept

| Tool | Type | Scope | Approach |
|---|---|---|---|
| **Terraform** | Declarative | Multi-cloud, multi-provider | State-driven; plans before acting |
| **CloudFormation** | Declarative | AWS only | AWS-native; no state file |
| **Ansible** | Primarily Imperative | Config management + some IaC | Agentless; procedural playbooks |
| **Pulumi** | Declarative | Multi-cloud | Uses real programming languages |
| **Chef/Puppet** | Declarative | Config management | Agent-based; manages running systems |

### Key Terraform Differentiators

**Execution plans:** Before making any change, Terraform generates a plan showing exactly what will be created, modified, or destroyed. Operators review and approve before anything touches real infrastructure. This "plan-then-apply" safety model is one of Terraform's defining characteristics.

**State file:** Terraform maintains a mapping between your configuration and the real-world resources it manages. State enables drift detection and calculating the minimum changes needed for each apply.

**Provider ecosystem:** The Terraform Registry hosts thousands of providers — official (maintained by HashiCorp or the cloud vendor) and community. Providers translate HCL resource definitions into the target platform's API calls.

---

## 1c. Terraform Use Cases

### Concept

Terraform is appropriate for:
- **Multi-tier applications:** Provision network, compute, database, and DNS layers together with dependency ordering.
- **Disposable environments:** Spin up a complete testing environment, run tests, destroy everything.
- **Self-service clusters:** Platform teams write modules; product teams consume them via simple variable inputs.
- **Multi-cloud:** Manage AWS, Azure, and GCP resources in a single plan.

Terraform is **not** the right tool for:
- Installing software packages on existing VMs (use Ansible/Chef)
- Application deployment (use Kubernetes/Helm)

---

<a id="section-2"></a>
# SECTION 2 — Terraform Fundamentals (~11%)

---

## 2a. Terraform Providers

### Concept

A **provider** is a plugin that enables Terraform to interact with a specific platform or service. Providers translate HCL resource definitions into the target platform's API calls.

Without a provider, Terraform has no knowledge of any cloud or service. The provider is the translation layer: you write resource "aws_instance" "web" { ... } and the AWS provider knows how to call the EC2 API to create it.

Providers are downloaded from the Terraform Registry during terraform init and stored in .terraform/providers/.

### Provider Configuration

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"    # registry.terraform.io/hashicorp/aws
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.12.0"
}

provider "aws" {
  region = "us-east-1"
}
```

### Provider Source Address Format

Format: HOSTNAME/NAMESPACE/TYPE

- **HOSTNAME** — defaults to registry.terraform.io if omitted
- **NAMESPACE** — organisation; hashicorp for official providers
- **TYPE** — provider name (aws, google, azurerm)

So hashicorp/aws is short for registry.terraform.io/hashicorp/aws.

### Multiple Provider Instances (Alias)

You can configure the same provider multiple times — for example, to manage resources in two AWS regions:

```hcl
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

resource "aws_instance" "west" {
  provider      = aws.west
  ami           = "ami-87654321"
  instance_type = "t3.micro"
}
```

### Exam Traps

- required_providers goes inside the terraform {} block, not at the top level.
- terraform init downloads providers. Re-running init after adding a new provider is required.
- The version constraint in required_providers is your constraint. The actual installed version is recorded in the lock file.

### Practice Questions

**Q1:** Where do you declare provider requirements and version constraints?
A) In a separate providers.tf file
B) Inside a required_providers block within the terraform {} block
C) Directly in each resource block
D) In the provider {} block
Answer: B

**Q2:** You need to manage AWS resources in us-east-1 and eu-west-1. What must you add to the second provider block?
Answer: An alias argument. Then reference provider = aws.alias_name in resources using the second region.

---

## 2b. The Dependency Lock File

### Concept

The **dependency lock file** (.terraform.lock.hcl) records the exact provider versions and their cryptographic checksums selected when you last ran terraform init. It ensures every person on the team — and every CI/CD run — uses the same provider versions, even if the version constraints in required_providers allow a range.

Think of it as the infrastructure equivalent of package-lock.json (npm). The constraint says "I accept any version matching ~> 5.0." The lock file records "we selected 5.31.0 specifically, and here is its hash."

### Key Properties

- **Committed to version control:** The lock file should always be committed to Git. This guarantees reproducibility across machines and CI runs.
- **Provider-only:** The lock file records provider dependencies only — not module dependencies.
- **Checksums:** Each entry includes h1: hashes (SHA-256) for the provider binary. Terraform verifies the provider has not been tampered with.
- **Upgrading:** Running terraform init -upgrade re-evaluates constraints and potentially updates provider versions. The lock file is updated to reflect the new selection.

```hcl
# .terraform.lock.hcl (auto-generated — never edit manually)
provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.31.0"
  constraints = "~> 5.0"
  hashes = [
    "h1:abc123...",
  ]
}
```

### Exam Traps

- The lock file is auto-generated by terraform init. Never edit it manually.
- Commit it to version control — correct and recommended practice.
- Running terraform init without -upgrade will NOT change provider versions already in the lock file, even if newer matching versions exist.

### Practice Questions

**Q1:** True or False: The .terraform.lock.hcl file should be committed to version control.
Answer: True — it ensures team members use identical provider versions.

**Q2:** Which command updates provider versions within allowed constraints and updates the lock file?
A) terraform refresh  B) terraform init -upgrade  C) terraform providers update  D) terraform apply -refresh-only
Answer: B

---

## 2c. Resources and Data Sources

### Concept

**Resource blocks** (resource) define infrastructure objects that Terraform creates, updates, and destroys. They are the primary building block of Terraform configuration.

**Data source blocks** (data) allow Terraform to read information from external sources and use it in configuration. Data sources do not create or manage anything — they are read-only queries.

```hcl
# Resource — Terraform CREATES and MANAGES this
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
}

# Data source — Terraform READS this, does not manage it
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]
  }
}

# Using the data source result in a resource
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}
```

### Reference Syntax

- Resource: RESOURCE_TYPE.NAME.ATTRIBUTE — e.g., aws_instance.web.id
- Data source: data.DATA_SOURCE_TYPE.NAME.ATTRIBUTE — e.g., data.aws_ami.ubuntu.id

### Key Differences

| Aspect | Resource | Data Source |
|---|---|---|
| Purpose | Create/manage infrastructure | Read existing data |
| Tracked in state | Yes | No |
| Destroyed by terraform destroy | Yes | No |
| Block keyword | resource | data |

### Exam Traps

- Data sources are not managed by Terraform. If the underlying object changes, the data source reflects the new value on the next plan.
- Data sources are evaluated during the plan phase.
- Most common use case: look up resources created outside Terraform.

---

## 2d. Implicit vs Explicit Dependencies

### Concept

Every resource has a unique address Terraform uses to identify it in state and the dependency graph.

Format: RESOURCE_TYPE.NAME — e.g., aws_instance.web, module.network.aws_vpc.main

### Implicit Dependencies

Terraform automatically infers dependencies by analysing resource references. If resource B references an attribute of resource A, Terraform knows A must be created before B.

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id    # implicit dependency — reference IS the dependency
  cidr_block = "10.0.1.0/24"
}
```

Terraform always creates aws_vpc.main before aws_subnet.public because of the reference. No depends_on needed.

### Explicit Dependencies (depends_on) — NEW 004 EMPHASIS

Sometimes a resource depends on another's side effects — not any specific attribute, so there's no reference for Terraform to detect. Use depends_on:

```hcl
resource "aws_iam_role_policy" "example" {
  role   = aws_iam_role.example.id
  policy = jsonencode({...})
}

resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"

  depends_on = [aws_iam_role_policy.example]
}
```

**Rule:** Only use depends_on when a dependency cannot be expressed through a direct attribute reference. Overusing it reduces parallelism.

### Exam Traps

- depends_on is a last resort. A direct attribute reference creates the dependency automatically.
- depends_on accepts a list of resource references — not strings.
- depends_on can be added to resources, modules, and data sources.

---

<a id="section-3"></a>
# SECTION 3 — Core Terraform Workflow (~19%)

---

## 3a. terraform init

### Concept

terraform init is the first command you run in any Terraform workspace. It:

1. Downloads providers into .terraform/providers/
2. Downloads modules into .terraform/modules/
3. Initialises the backend (where state is stored)
4. Creates or validates the lock file (.terraform.lock.hcl)

Until you run init, no other Terraform commands will work.

```bash
terraform init                  # standard init
terraform init -upgrade         # re-evaluate constraints; update providers + lock file
terraform init -reconfigure     # reinitialise backend even if already configured
terraform init -migrate-state   # migrate state from old backend to new backend
```

### Exam Traps

- terraform init is idempotent — safe to re-run at any time.
- It does not read, modify, or apply any changes to infrastructure.
- Required after: adding a new provider, changing the backend, adding a module source.
- The .terraform/ directory is NOT committed to version control. The lock file IS.

---

## 3b. terraform fmt

### Concept

terraform fmt automatically reformats Terraform configuration files to the canonical HCL style — consistent 2-space indentation, aligned = signs. It modifies files in-place. This is purely cosmetic — it changes formatting, not logic.

```bash
terraform fmt             # format .tf files in current directory
terraform fmt -recursive  # format subdirectories too
terraform fmt -check      # exit non-zero if files need formatting (CI gate)
terraform fmt -diff       # show diff without applying
```

### Exam Traps

- fmt does NOT validate syntax or logic.
- fmt -check returns exit code 0 if no changes needed, 1 if changes would be made. This is the CI-friendly mode.

---

## 3c. terraform validate

### Concept

terraform validate checks configuration for syntactic and semantic correctness — invalid syntax, references to undefined variables, incorrect argument types, missing required arguments. It does not access any remote APIs or state.

```bash
terraform validate
# Success! The configuration is valid.
# Or: Error: An argument named "instanc_type" is not expected here.
```

### fmt vs validate vs plan

| Command | What it checks | Modifies files? | Requires network? |
|---|---|---|---|
| terraform fmt | Formatting/style only | Yes | No |
| terraform validate | Syntax and logic | No | No (needs init) |
| terraform plan | Everything + real infra | No | Yes |

### Exam Traps

- validate catches typos in argument names, wrong types, missing required arguments.
- It does NOT check if referenced cloud resources exist — that is plan.
- Requires terraform init to have run first (needs provider schemas).

---

## 3d. terraform plan

### Concept

terraform plan generates an execution plan — a preview of changes Terraform would make if you ran apply. It compares configuration against current state and produces a diff.

```bash
terraform plan                      # standard plan
terraform plan -out=tfplan          # save plan to file
terraform plan -refresh-only        # only reconcile state, no config changes
terraform plan -destroy             # preview a full destroy
terraform plan -var="env=prod"
terraform plan -var-file=prod.tfvars
```

### Reading Plan Output

```
+ resource "aws_instance" "web" { ... }   # will be CREATED
- resource "aws_instance" "web" { ... }   # will be DESTROYED
~ resource "aws_instance" "web" { ... }   # will be UPDATED in-place
-/+ resource "aws_instance" "web" { ... } # will be DESTROYED and RECREATED
<= data "aws_ami" "ubuntu" { ... }        # data source will be READ
```

### Saving and Using Plan Files

```bash
terraform plan -out=tfplan    # save plan
terraform apply tfplan         # apply exactly that plan (no approval prompt)
```

Saved plan files guarantee the applied changes match exactly what was reviewed — critical for separating who reviews from who applies.

### Exam Traps

- plan does NOT make any changes to infrastructure.
- -refresh-only plans update the state file to match reality without applying config changes.
- -target is a last-resort escape hatch, not a routine practice.
- Saved plan files are binary — not human-readable text.

---

## 3e. terraform apply

### Concept

terraform apply executes changes — creating, updating, or destroying resources to bring infrastructure into alignment with configuration.

```bash
terraform apply               # generate plan, prompt for approval, apply
terraform apply tfplan        # apply a saved plan (no approval prompt)
terraform apply -auto-approve # skip interactive approval (CI use)
```

### What Apply Does

1. Refreshes state (reads real-world resource values)
2. Generates a plan
3. Shows the plan and prompts for confirmation (type yes)
4. Executes changes in dependency order, in parallel where possible
5. Updates the state file

### Exam Traps

- When you type yes, Terraform is committed. There is no undo.
- Passing a saved plan file skips the approval prompt — the plan IS the approval.
- apply updates state even if some resources fail — partial state is real.

---

## 3f. terraform destroy

### Concept

terraform destroy destroys all resources managed by the current workspace's state. Equivalent to terraform apply -destroy.

```bash
terraform destroy               # destroy all resources; prompts for approval
terraform destroy -auto-approve # skip approval
terraform plan -destroy         # preview what would be destroyed
```

### Exam Traps

- Resources are destroyed in reverse dependency order.
- Resources with prevent_destroy = true in their lifecycle block will cause destroy to error.
- Destroying does not delete configuration files — only real-world resources and state entries.

---

## 3g. Dependency Graph and Parallelism

### Concept

Terraform builds a directed acyclic graph (DAG) of all resources. Resources with no dependencies are provisioned in parallel (default: up to 10 concurrent operations). Resources that depend on others are sequenced.

This parallelism is why adding unnecessary depends_on can slow down applies — it introduces false sequencing constraints that prevent parallel execution.

---

<a id="section-4"></a>
# SECTION 4 — Terraform Configuration (~22%)

---

## 4a. Resource Meta-Arguments

### Concept

Meta-arguments are special arguments available to all resource types, regardless of provider. They modify Terraform's default behaviour.

| Meta-Argument | Purpose |
|---|---|
| depends_on | Explicit dependency declaration |
| count | Create multiple instances using an integer |
| for_each | Create multiple instances using a map or set |
| provider | Specify a non-default provider instance |
| lifecycle | Customise create/update/destroy behaviour |

### count

count creates N copies of a resource. Each instance is addressed as TYPE.NAME[INDEX] (0-based).

```hcl
resource "aws_instance" "server" {
  count         = 3
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  tags = {
    Name = "server-${count.index}"   # server-0, server-1, server-2
  }
}
# Reference: aws_instance.server[0].id
# All:       aws_instance.server[*].id
```

**The count index problem:** If you remove an item from the middle of a count-based list, Terraform re-indexes all items after it — triggering unnecessary destroy-and-recreate operations. This is count's key weakness vs for_each.

### for_each

for_each creates one instance per element in a map or set. Each instance is addressed by its key, not a numeric index. Removing one item does not affect others.

```hcl
# Using a set of strings
resource "aws_iam_user" "team" {
  for_each = toset(["alice", "bob", "carol"])
  name     = each.key
}
# Addresses: aws_iam_user.team["alice"], aws_iam_user.team["bob"]

# Using a map
resource "aws_instance" "servers" {
  for_each = {
    web = "t3.micro"
    api = "t3.small"
  }
  ami           = "ami-12345678"
  instance_type = each.value
  tags = { Name = each.key }
}
# Addresses: aws_instance.servers["web"], aws_instance.servers["api"]
```

each.key — the map key (or set value)
each.value — the map value (for sets, same as each.key)

### count vs for_each — Exam Critical

| Aspect | count | for_each |
|---|---|---|
| Input type | Integer | Map or Set |
| Instance address | [0], [1], [2] | ["alice"], ["bob"] |
| Removing one item | Re-indexes — side effects | Removes only that key |
| Best for | Identical resources | Resources that differ by key |
| With a list input | Supported | Must use toset() first |

### lifecycle Block — NEW 004 EMPHASIS

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"

  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags]
    replace_triggered_by  = [aws_security_group.main]
  }
}
```

**create_before_destroy** — By default, Terraform destroys the old resource first, then creates the replacement. With create_before_destroy = true, it creates the replacement first, then destroys the old. This minimises downtime for load balancers, DNS records, and other resources where a gap is unacceptable.

**prevent_destroy** — Acts as a safety catch. Terraform will error if a plan would destroy this resource. Useful for databases and other critical resources. Important caveat: this only protects while the lifecycle block exists in config. Remove the block then apply destroy — and the resource is gone.

**ignore_changes** — Tells Terraform to ignore changes to specific attributes when calculating diffs. Use when an attribute is managed externally (e.g., auto-scaling updates tags) and you don't want Terraform to revert those changes.

**replace_triggered_by** — Forces replacement of this resource whenever the referenced resource changes, even if this resource's own attributes have not changed.

### Exam Traps

- for_each does not accept a plain list. Convert with toset() if your source is a list.
- create_before_destroy on a resource propagates to all resources that depend on it.
- prevent_destroy = true only works while the block exists in config. Remove it, then you can destroy.

---

## 4b. Variables, Outputs, and Locals

### Input Variables

An **input variable** is a parameter for a Terraform configuration. It allows the same module or configuration to behave differently based on values supplied at runtime — the mechanism for making configurations reusable and environment-agnostic.

```hcl
variable "instance_type" {
  type        = string
  description = "EC2 instance type"
  default     = "t3.micro"
}
```

### Variable Assignment — Precedence (Lowest to Highest)

```
default value in variable block
  (lowest)
Terraform Cloud workspace variables
terraform.tfvars and *.auto.tfvars  (auto-loaded)
-var-file=custom.tfvars  (explicit CLI flag)
-var="key=value"  (CLI flag)
TF_VAR_NAME environment variables
  (highest)
```

Auto-loaded files: terraform.tfvars and any file matching *.auto.tfvars are loaded automatically without CLI flags. Any other .tfvars file must be explicitly specified with -var-file.

### Variable Types — Primitives

**string** — A sequence of Unicode characters. Used for names, ARNs, regions, tags.

**number** — A numeric value. Can be integer or floating point.

**bool** — A boolean: true or false. Used for feature flags.

### Variable Types — Collections

**list(TYPE)** — An ordered sequence of values of the same type. Supports indexing: var.subnets[0]. Allows duplicates. Use when order matters and items are identified by position.

```hcl
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b", "us-east-1c"]
}
# Access: var.availability_zones[0]  =>  "us-east-1a"
```

**set(TYPE)** — A collection of unique, unordered values of the same type. A set automatically discards duplicate values and does not support indexing by position (you cannot call set[0]). Sets are primarily useful when you need uniqueness guarantees and will iterate over items without caring about order — most commonly to feed for_each.

```hcl
variable "allowed_cidrs" {
  type    = set(string)
  default = ["10.0.0.0/8", "172.16.0.0/12"]
}
# Convert list to set to eliminate duplicates and use with for_each:
# toset(["a", "b", "a"])  =>  {"a", "b"}
```

**map(TYPE)** — A collection of key-value pairs where all values are the same type. Keys are strings. Unlike object, a map enforces that all values share the same type — you cannot mix strings and numbers. Use when you have a dynamic set of named values all of the same type.

```hcl
variable "env_cidrs" {
  type = map(string)
  default = {
    dev  = "10.0.0.0/16"
    prod = "10.1.0.0/16"
  }
}
# Access: var.env_cidrs["prod"]  =>  "10.1.0.0/16"
```

### Variable Types — Structural

**object({...})** — A structured type where each attribute can have a different type. Unlike map, an object specifies an exact schema: attribute names and their individual types. Use for complex, structured configuration where different fields have different types.

```hcl
variable "db_config" {
  type = object({
    engine     = string
    multi_az   = bool
    storage_gb = number
  })
}
# Access: var.db_config.engine, var.db_config.multi_az
```

### Type Comparison Table

| Type | Order | Duplicates | Mixed types | Index access |
|---|---|---|---|---|
| list | Yes | Yes | No (same type) | [0], [1] |
| set | No | No (auto-removed) | No (same type) | None |
| map | No | N/A (keys unique) | No (values same type) | ["key"] |
| object | N/A | N/A | Yes (per attribute) | .attr |

### Output Values

An output value exposes data from a Terraform configuration to the CLI, to HCP Terraform, or to a calling module.

```hcl
output "instance_ip" {
  description = "Public IP of the web server"
  value       = aws_instance.web.public_ip
  sensitive   = true   # hides value in CLI output; still stored in state
}
```

**sensitive = true** on an output does NOT encrypt the value in state. It only prevents Terraform from printing the value in CLI output. The value is still stored in plaintext in terraform.tfstate.

```bash
terraform output                   # show all outputs
terraform output instance_ip       # show specific output
terraform output -json             # machine-readable JSON
terraform output -raw instance_ip  # raw value, no quotes (useful in scripts)
```

### Local Values

A **local value** assigns a name to an expression within a module. Like a constant — defined once, referenced many times. Unlike variables, locals cannot be set from outside the module.

```hcl
locals {
  name_prefix = "${var.project}-${var.environment}"
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

resource "aws_instance" "web" {
  tags = local.common_tags    # reference with local.<name>
}
```

### Exam Traps

- terraform.tfvars and *.auto.tfvars are loaded automatically. Other .tfvars files need -var-file.
- sensitive = true hides value in CLI but does NOT protect it in the state file — still stored in plaintext.
- for_each requires a map or set — passing a list causes an error. Use toset() to convert.
- local values are referenced as local.name (singular), not locals.name.

---

## 4c. Expressions, Key Functions, and Dynamic Blocks

### Conditional Expression (Ternary)

The conditional expression is heavily tested. It returns one of two values based on a boolean condition:

```hcl
# condition ? value_if_true : value_if_false
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
min_size      = var.high_availability ? 3 : 1
```

### for Expressions

for expressions transform one collection into another:

```hcl
# Transform a list
locals {
  upper_names = [for n in var.names : upper(n)]
  # ["alice", "bob"]  =>  ["ALICE", "BOB"]
}

# Filter with if clause
locals {
  prod_instances = [for k, v in var.instances : k if v.env == "prod"]
}

# List to Map
locals {
  name_map = {for n in var.names : n => upper(n)}
  # {"alice" = "ALICE", "bob" = "BOB"}
}
```

### Splat Expressions

```hcl
# Shorthand for iterating all instances of a count-based resource
instance_ids = aws_instance.web[*].id
# Equivalent to: [for i in aws_instance.web : i.id]
```

### Exam-Relevant Built-in Functions

The exam does not test function syntax memorisation. These functions appear in scenario questions because they solve real configuration problems:

**toset(list)** — Converts a list to a set, removing duplicates. Critical because for_each requires a map or set — this is the bridge when your input is a list.

```hcl
toset(["alice", "bob", "alice"])  # => {"alice", "bob"}
```

Exam scenario: you have a list(string) variable of usernames and want to use for_each to create IAM users. You cannot pass the list directly — wrap it in toset().

**lookup(map, key, default)** — Returns the value for a key in a map, or a default if the key does not exist. Safe map access without errors.

```hcl
lookup(var.env_cidrs, "staging", "10.99.0.0/16")
```

**merge(map1, map2, ...)** — Merges multiple maps into one. Later maps override earlier ones for duplicate keys. Commonly used to merge common_tags with resource-specific tags.

```hcl
tags = merge(local.common_tags, { Name = "web-server" })
```

**jsonencode(value)** — Converts an HCL value to a JSON string. Appears in IAM policy resource questions — AWS policy documents must be JSON strings.

```hcl
policy = jsonencode({
  Version = "2012-10-17"
  Statement = [{
    Effect   = "Allow"
    Action   = ["s3:GetObject"]
    Resource = "*"
  }]
})
```

**file(path)** — Reads the contents of a file and returns it as a string. Used to load scripts or certificates.

```hcl
user_data = file("${path.module}/scripts/init.sh")
```

**templatefile(path, vars)** — Reads a template file and renders it with provided variables. More powerful than file() when the file needs dynamic values injected.

```hcl
user_data = templatefile("${path.module}/init.tpl", {
  db_host = aws_db_instance.main.endpoint
  env     = var.environment
})
```

**flatten(list_of_lists)** — Collapses a list of lists into a single flat list. Useful when module outputs return nested lists.

```hcl
flatten([["a", "b"], ["c"]])  # => ["a", "b", "c"]
```

### Dynamic Blocks

A **dynamic block** generates repeated nested blocks within a resource based on a collection. Without dynamic blocks, every nested block must be hard-coded; with dynamic, you iterate.

```hcl
variable "ingress_rules" {
  default = [
    { port = 80,  cidr = "0.0.0.0/0" },
    { port = 443, cidr = "0.0.0.0/0" },
  ]
}

resource "aws_security_group" "web" {
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = [ingress.value.cidr]
    }
  }
}
```

Inside the content block, the iterator variable is named after the dynamic block label (ingress). Access items with iterator.value and iterator.key.

### Exam Traps

- The exam tests toset() frequently in the context of for_each — know why it is needed and what it does.
- jsonencode() appears in IAM/policy resource questions — AWS policies must be JSON strings.
- Dynamic blocks generate nested blocks, not top-level resources. They are for things like ingress, egress, tag, setting blocks inside a resource.

---

## 4d. Input Validation

### Concept

**Input validation** adds custom rules to variable blocks. Terraform checks them before attempting to apply — catching bad configurations without making any API calls.

```hcl
variable "environment" {
  type = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "cidr_block" {
  type = string

  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be a valid IPv4 CIDR block (e.g., 10.0.0.0/16)."
  }
}
```

The condition must evaluate to true or false. The error_message must be a plain string — no interpolation of resource attributes.

A single variable can have multiple validation blocks, each evaluated independently.

### can() and try()

**can(expression)** — Returns true if the expression succeeds, false if it would error. Used inside validation blocks to test whether a value has a valid format without crashing.

```hcl
can(cidrhost("10.0.0.0/16", 0))  # true — valid CIDR
can(cidrhost("not-a-cidr", 0))   # false — invalid
```

**try(expr1, expr2, ...)** — Evaluates expressions left to right and returns the value of the first one that does not produce an error. Like a try-catch for expressions.

```hcl
try(var.config.optional_field, "default_value")
```

---

## 4e. terraform console

### Concept

terraform console opens an interactive REPL where you can evaluate Terraform expressions, test functions, and inspect values from your state. Useful for debugging variable logic and function behaviour before embedding it in config.

```bash
terraform console

> toset(["x", "x", "y"])
toset(["x", "y"])
> var.region
"us-east-1"
> aws_instance.web.public_ip
"54.1.2.3"
```

---

## 4f. Custom Conditions — Preconditions, Postconditions, Checks — NEW 004

### Concept

Custom conditions extend Terraform's validation capabilities beyond input variables. They assert expectations about configuration and infrastructure at different lifecycle points.

### Preconditions

A **precondition** runs before Terraform makes changes to a resource. It validates inputs and configuration assumptions before any API calls occur.

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  lifecycle {
    precondition {
      condition     = data.aws_ami.ubuntu.architecture == "x86_64"
      error_message = "The selected AMI must be x86_64 architecture."
    }
  }
}
```

### Postconditions

A **postcondition** runs after Terraform creates or updates a resource. It validates that the resulting resource has the expected properties — catching cases where the provider created the resource with unexpected attributes.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"

  lifecycle {
    postcondition {
      condition     = self.public_ip != ""
      error_message = "EC2 instance must have a public IP address."
    }
  }
}
```

Note: postconditions use self to reference the resource's own attributes.

### Check Blocks (Terraform 1.5+)

A **check block** is a standalone assertion that runs after every plan and apply. Unlike preconditions/postconditions (which fail the apply), check blocks produce warnings — not errors. Used for asserting ambient conditions without blocking the apply.

```hcl
check "health_check" {
  data "http" "app" {
    url = "https://api.example.com/health"
  }

  assert {
    condition     = data.http.app.status_code == 200
    error_message = "Health check returned ${data.http.app.status_code}."
  }
}
```

### When to Use Which

| Mechanism | When it runs | Failure effect | Use case |
|---|---|---|---|
| Variable validation | Before plan | Blocks plan | Validate user inputs |
| precondition | Before resource change | Blocks apply | Validate config assumptions |
| postcondition | After resource change | Blocks apply | Validate resource outcome |
| check block | After apply | Warning only | Assert ambient health |

---

## 4g. Ephemeral Values and Write-Only Arguments — NEW 004 (Terraform 1.12)

### Concept

**Ephemeral values** are a Terraform 1.12 feature for handling sensitive data — passwords, tokens, API keys — that should never be persisted to the state file. Traditional Terraform secrets (sensitive = true) are hidden in CLI output but still written to state. Ephemeral values exist only in memory during plan/apply and are never stored anywhere.

### Ephemeral Resources

An **ephemeral resource** is like a data source, but its result is never stored in state. The value is fetched fresh on every run and discarded after.

```hcl
ephemeral "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = ephemeral.aws_secretsmanager_secret_version.db_password.secret_string
  # This password is NEVER written to state
}
```

### Write-Only Arguments

A **write-only argument** is a resource attribute that Terraform accepts as input but never reads back from the provider and never stores in state. Terraform sets it during create/update but ignores the provider's response — preventing the secret from landing in state.

### Why This Matters for the Exam

The 004 exam specifically tests the distinction between these mechanisms:

| Mechanism | Hidden in CLI | Stored in State | Notes |
|---|---|---|---|
| sensitive = true (variable/output) | Yes | Yes (plaintext) | Cosmetic only |
| TF_VAR_ env var | Yes (not in config) | Yes | Still in state |
| Ephemeral values | Yes | Never | Memory only |
| Write-only arguments | Yes | Never | Provider support required |

---

<a id="section-5"></a>
# SECTION 5 — Terraform Modules (~11%)

---

## 5a. What is a Module?

### Concept

A **module** is a container for multiple related Terraform resources. Every Terraform configuration has at least one module: the **root module** (the directory where you run Terraform). Any other directory of .tf files called from the root is a **child module**.

Modules are Terraform's primary mechanism for code reuse. Instead of copy-pasting the same VPC configuration across 10 projects, you write it once and call it 10 times with different variables.

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "prod-vpc"
  cidr = "10.0.0.0/16"
  azs  = ["us-east-1a", "us-east-1b"]
}

# Referencing module outputs
resource "aws_instance" "app" {
  subnet_id = module.vpc.private_subnets[0]
}
```

---

## 5b. Module Sources

The source argument tells Terraform where to find the module's code.

| Source Type | Example | Notes |
|---|---|---|
| Local path | source = "./modules/vpc" | Must start with ./ or ../ |
| Terraform Registry | source = "hashicorp/consul/aws" | Public registry; requires version |
| GitHub | source = "github.com/org/repo//modules/vpc" | // separates repo from subdirectory |
| Git (generic) | source = "git::https://example.com/repo.git" | |

### Registry Module Address Format

Format: NAMESPACE/MODULE_NAME/PROVIDER

Example: hashicorp/consul/aws — the consul module for AWS, published by HashiCorp.

### Version Constraints

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
}
```

| Constraint | Meaning |
|---|---|
| = 1.2.0 | Exact version only |
| >= 1.2.0 | Any version 1.2.0 or newer |
| ~> 1.2.0 | >= 1.2.0, < 1.3.0 (patch updates only) |
| ~> 1.2 | >= 1.2, < 2.0 (minor updates) |

### Exam Traps

- Local paths must start with ./ or ../. A bare modules/vpc is interpreted as a registry address.
- version is only valid for registry sources. Local paths ignore version.
- After changing a module source or version, run terraform init.

---

## 5c. Module Inputs and Outputs

### Concept

Modules communicate through input variables (caller passes values in) and output values (module exposes values out). Variable scope is isolated — variables inside a module are invisible outside it. The only way data crosses module boundaries is through explicit variable inputs and output values.

```hcl
# modules/vpc/variables.tf
variable "name" { type = string }
variable "cidr" { type = string }

# modules/vpc/outputs.tf
output "vpc_id" {
  value = aws_vpc.main.id
}

# Root module
module "vpc" {
  source = "./modules/vpc"
  name   = "my-vpc"
  cidr   = "10.0.0.0/16"
}

# Consuming child module output: module.MODULE_NAME.OUTPUT_NAME
resource "aws_instance" "app" {
  subnet_id = module.vpc.private_subnets[0]
}
```

---

## 5d. Module Best Practices

### Standard Module Structure

```
terraform-module/
  main.tf        primary resource definitions
  variables.tf   input variable declarations
  outputs.tf     output value declarations
  versions.tf    terraform {} and required_providers
  README.md      documentation
```

### Exam Traps

- Module outputs must be explicitly declared with output blocks. Resources inside a module are not directly accessible from outside.
- terraform init must be run after adding or changing a module source.
- Modules do not inherit provider configurations from the calling module unless passed explicitly via the providers argument.

---

<a id="section-6"></a>
# SECTION 6 — Terraform State Management (~11%)

---

## 6a. Purpose of State

### Concept

Terraform's **state file** (terraform.tfstate) is a JSON file that maps the resources in your configuration to the real-world objects they represent. Without state, Terraform would have no memory of what it previously created.

State serves three primary purposes:

**1. Mapping configuration to real world:** Your config says resource "aws_instance" "web". The state records that this corresponds to EC2 instance i-0abc1234def56789. On the next plan, Terraform knows to compare that specific instance against config — not create a new one.

**2. Metadata tracking:** State stores metadata that cannot be queried from the provider — like resource dependencies and creation sequence. This enables correct destroy ordering.

**3. Performance:** For large configurations, Terraform uses the cached state to calculate plans without making API calls for every resource on every run.

### Exam Traps

- The state file may contain sensitive values in plaintext. Protect it accordingly.
- Never edit the state file manually — use terraform state commands instead.
- Deleting the state file does not delete your infrastructure. But Terraform will think nothing exists and try to recreate everything on the next apply.

---

## 6b. Remote Backends and State Locking

### Concept

A **backend** determines how Terraform stores state. The default is local — state lives in terraform.tfstate on your machine. Remote backends store state in a shared, persistent location — required for team use.

### Common Remote Backends

| Backend | State Storage | Locking mechanism |
|---|---|---|
| s3 (with DynamoDB) | AWS S3 bucket | DynamoDB table |
| azurerm | Azure Blob Storage | Blob leases |
| gcs | Google Cloud Storage | GCS object locking |
| HCP Terraform | HCP Terraform | Built-in |

### Backend Configuration

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/network/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"    # enables state locking
    encrypt        = true
  }
}
```

### State Locking

**State locking** prevents multiple users or CI runs from applying changes simultaneously — which would corrupt the state file. If Terraform crashes while holding a lock:

```bash
terraform force-unlock LOCK_ID
```

### Exam Traps

- The s3 backend does NOT natively lock state. You must configure a DynamoDB table separately.
- Migrating between backends: terraform init -migrate-state.
- Backend configuration cannot use variables, locals, or resource references — only literal values.
- HCP Terraform has locking built in — no DynamoDB needed.

---

## 6c. Drift Detection and State Reconciliation

### Concept

**Drift** occurs when real-world infrastructure diverges from what Terraform's state file records — for example, when someone manually changes a resource through the cloud console.

```bash
terraform plan               # implicitly refreshes; shows drift as proposed changes
terraform plan -refresh-only # plan that only reconciles state, no config changes
terraform apply -refresh-only # accept the drift into state
```

**-refresh-only mode** generates a plan that, if applied, updates the state file to match reality without changing configuration. This is the correct way to accept manual changes and bring state in sync.

### terraform state Commands

```bash
terraform state list                  # list all resources in state
terraform state show aws_instance.web # full details for one resource
terraform state mv OLD_ADDR NEW_ADDR  # rename resource in state (without destroying)
terraform state rm aws_instance.web   # remove from state (resource still exists in cloud)
terraform state pull                  # download remote state to stdout
```

**state rm** is useful when you want Terraform to stop managing a resource without destroying it. The real cloud resource remains; only the state entry is removed.

### Exam Traps

- terraform refresh (standalone command) is deprecated. Use terraform apply -refresh-only instead.
- terraform plan automatically refreshes state by default — no separate refresh step needed.
- state rm removes from Terraform management — the cloud resource still exists.

---

## 6d. terraform import and moved / removed Blocks

### terraform import

terraform import brings an existing real-world resource under Terraform management without destroying and recreating it.

```bash
terraform import aws_instance.web i-0abc1234def56789
```

**Critical limitation:** import only updates the state file. It does NOT generate HCL configuration. You must write the resource block yourself, then run import to link state to it.

Terraform 1.5+ added declarative import blocks:

```hcl
import {
  to = aws_instance.web
  id = "i-0abc1234def56789"
}
```

Combined with: terraform plan -generate-config-out=generated.tf to auto-generate HCL.

### moved Block

The moved block tells Terraform a resource has been renamed in configuration — without destroying and recreating it. Terraform updates the state reference to the new address.

```hcl
# You renamed aws_instance.web to aws_instance.app in config
moved {
  from = aws_instance.web
  to   = aws_instance.app
}
```

Without moved, Terraform would destroy aws_instance.web and create aws_instance.app — needlessly replacing an identical resource.

### removed Block (Terraform 1.7+)

The removed block gracefully removes a resource from state without destroying the real-world object. Declarative equivalent of terraform state rm.

```hcl
removed {
  from = aws_instance.legacy

  lifecycle {
    destroy = false    # don't destroy the real resource; just remove from state
  }
}
```

### Exam Traps

- terraform import adds to state — does NOT generate config.
- After import, terraform plan should show no changes if your config correctly describes the resource.
- moved blocks are for refactoring — renaming or moving resources between modules — without infrastructure disruption.

---

<a id="section-7"></a>
# SECTION 7 — Maintain Infrastructure with Terraform (~8%)

---

## 7a. Terraform Workspaces

### Concept

A **workspace** is a named, isolated instance of state. Each workspace has its own independent state file. The default workspace exists in every configuration and is where Terraform operates unless you switch.

Workspaces allow the same configuration to manage multiple environments (dev, staging, prod) from the same directory, with separate state for each.

```bash
terraform workspace list          # list all workspaces
terraform workspace show          # show current workspace
terraform workspace new staging   # create and switch to new workspace
terraform workspace select prod   # switch to existing workspace
terraform workspace delete staging
```

Referencing workspace in config:

```hcl
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"
  tags = {
    Environment = terraform.workspace
  }
}
```

### Workspace Limitations

All workspaces share the same backend and configuration. There is no per-workspace variable management or access control in open-source Terraform. For stronger isolation, separate root modules per environment is often preferred.

### Exam Traps

- The default workspace cannot be deleted.
- CLI workspaces are different from HCP Terraform workspaces — HCP Terraform workspaces are heavier (VCS integration, variable management, RBAC).
- Switching workspace changes which state Terraform reads. Configuration files stay the same.

---

## 7b. Debugging and Logging

Terraform uses the TF_LOG environment variable to enable verbose logging.

```bash
TF_LOG=DEBUG terraform plan             # enable debug logging to stderr
TF_LOG_PATH=./terraform.log terraform apply  # write logs to file
TF_LOG_PROVIDER=DEBUG terraform plan    # debug the provider specifically
```

Logging levels (increasing verbosity): ERROR, WARN, INFO, DEBUG, TRACE

The exam may ask which environment variable enables logging (TF_LOG) and where logs can be directed (TF_LOG_PATH).

---

## 7c. terraform apply -replace (formerly taint)

### Concept

terraform taint marked a resource for forced replacement on the next apply. It is **deprecated** in Terraform 1.x. The modern approach is the -replace flag:

```bash
terraform apply -replace=aws_instance.web
```

This forces Terraform to destroy and recreate the specified resource, even if the config has not changed — useful when a resource is in a bad state that Terraform cannot detect through a normal refresh.

### Exam Traps

- terraform taint is deprecated — the exam may test that you know to use -replace instead.
- -replace generates a plan showing -/+ (destroy then create) for the targeted resource.

---

<a id="section-8"></a>
# SECTION 8 — HCP Terraform (~11%)

---

## 8a. What is HCP Terraform?

### Concept

**HCP Terraform** (HashiCorp Cloud Platform Terraform, formerly Terraform Cloud) is HashiCorp's hosted service that extends the open-source Terraform CLI with:

- Remote state storage — state stored securely, not on local machines
- State locking — built-in, no external DynamoDB needed
- Remote execution — runs happen on HCP Terraform's infrastructure
- VCS integration — plans triggered automatically by Git push
- Team collaboration — RBAC, plan review, approval workflows
- Variable management — store sensitive variables securely outside config files
- Policy enforcement — Sentinel policies for governance

---

## 8b. HCP Terraform Workspaces

### Concept

An **HCP Terraform workspace** is the fundamental unit of operation. Each workspace has its own state file, variable set, VCS repository connection, run history, and access controls.

This is much richer than CLI workspaces. In HCP Terraform, a workspace is closer to "one environment for one configuration."

### Execution Modes

| Mode | Where Terraform runs | State stored |
|---|---|---|
| Remote (default) | HCP Terraform's infrastructure | HCP Terraform |
| Local | Your machine | HCP Terraform (remote state only) |
| Agent | Your own agents (for private networks) | HCP Terraform |

### VCS-Driven Workflow

With VCS integration (GitHub, GitLab, Bitbucket):
1. Developer opens a PR — HCP Terraform runs terraform plan
2. Plan is shown in the PR for review
3. PR is merged — HCP Terraform runs terraform apply (with optional approval gate)

### CLI-Driven Workflow

You run terraform plan and apply locally, but execution happens on HCP Terraform's infrastructure:

```hcl
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "prod-network"
    }
  }
}
```

```bash
terraform login    # opens browser for API token
```

---

## 8c. HCP Terraform Projects — NEW 004

### Concept

**Projects** group and organise multiple HCP Terraform workspaces. As organisations scale to dozens or hundreds of workspaces, projects provide:

- Logical grouping — all workspaces for the "payments" team in one project
- Project-level access control — grant a team access to all workspaces in a project without configuring each individually
- Project-level variable sets — share variables across all workspaces in a project

Every workspace belongs to exactly one project. A default project exists in every organisation.

### Project vs Workspace

| Concept | Purpose |
|---|---|
| Project | Organisational container; governs access and shared config across workspaces |
| Workspace | Operational unit; one environment, one state, one VCS connection |

### Exam Traps

- Projects are an HCP Terraform concept — they do not exist in open-source Terraform.
- A workspace belongs to exactly one project.
- Project-level permissions cascade to all workspaces in the project.

---

## 8d. Variable Sets and Run Triggers

### Variable Sets

A **variable set** is a reusable collection of variables applied to multiple workspaces. Instead of configuring the same AWS credentials in 20 workspaces, create one variable set and apply it to all 20.

Variable sets can be scoped to:
- Organisation — every workspace in the organisation
- Project — every workspace in a project
- Workspace — specific individual workspaces

Variable types in HCP Terraform:
- Terraform variables — equivalent to TF_VAR_name or -var (HCL values)
- Environment variables — set as env vars during execution (e.g., AWS_ACCESS_KEY_ID)
- Sensitive — write-once, never displayed after saving

### Run Triggers

A **run trigger** creates a dependency between workspaces — when workspace A successfully applies, workspace B is automatically queued. Used to chain dependent infrastructure:

- Workspace A: shared networking (VPC, subnets)
- Workspace B: application infra — automatically triggered when workspace A changes

### terraform_remote_state Data Source

Allows one workspace to read outputs from another workspace's state:

```hcl
data "terraform_remote_state" "network" {
  backend = "remote"
  config = {
    organization = "my-org"
    workspaces = { name = "prod-network" }
  }
}

resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.network.outputs.private_subnet_id
}
```

---

# QUICK REFERENCE — EXAM CHEATSHEET

## The 8 Sections and Weight

| Section | Weight | Key Focus |
|---|---|---|
| 1. IaC with Terraform | ~8% | Declarative, idempotent, multi-cloud, Terraform vs other tools |
| 2. Fundamentals | ~11% | Providers, lock file, resources vs data sources, depends_on |
| 3. Core Workflow | ~19% | init — fmt — validate — plan — apply — destroy |
| 4. Configuration | ~22% | Variables/types, lifecycle, count/for_each, validation, ephemeral |
| 5. Modules | ~11% | Source types, version constraints, inputs/outputs |
| 6. State | ~11% | Purpose, backends, locking, drift, import/moved |
| 7. Maintenance | ~8% | Workspaces, TF_LOG, -replace |
| 8. HCP Terraform | ~11% | Workspaces, projects, variable sets, VCS workflow |

---

## Variable Precedence (Low to High)

```
default value in variable block          (lowest)
Terraform Cloud workspace variables
terraform.tfvars and *.auto.tfvars       (auto-loaded)
-var-file=custom.tfvars                  (explicit CLI flag)
-var="key=value"                         (CLI flag)
TF_VAR_NAME environment variable         (highest)
```

---

## Type Quick Reference

```
string  = "hello"                  sequence of characters
number  = 42                       integer or float
bool    = true                     true or false
list    = ["a", "b", "c"]         ordered, duplicates OK, index with [0]
set     = toset(["a", "b"])       unordered, no duplicates, no index
map     = {a = 1, b = 2}         key-value, all values same type
object  = {name = "x", age = 30} key-value, mixed types per attribute
```

---

## Lifecycle Arguments

```hcl
lifecycle {
  create_before_destroy = true              # create replacement first, then destroy old
  prevent_destroy       = true              # error if plan would destroy this resource
  ignore_changes        = [tags]            # ignore changes to these attributes
  replace_triggered_by  = [aws_sg.main]    # force replacement if this changes
}
```

---

## State Commands

```bash
terraform state list              # all resources in state
terraform state show ADDR         # full details for one resource
terraform state mv OLD NEW        # rename in state — no destroy
terraform state rm ADDR           # remove from state — resource still exists
terraform import ADDR ID          # import existing resource to state
terraform force-unlock LOCK_ID    # release stuck lock
```

---

## Sensitive Data — What Each Mechanism Actually Does

| Mechanism | Hidden in CLI | Stored in State | Notes |
|---|---|---|---|
| sensitive = true (variable/output) | Yes | Yes (plaintext) | Cosmetic only |
| TF_VAR_ env var | Yes (not in config) | Yes | Still in state |
| Ephemeral values | Yes | Never | Memory only — NEW 004 |
| Write-only arguments | Yes | Never | Provider support required — NEW 004 |

---

## Top 10 Exam Traps

1. terraform import adds to state — does not generate config
2. sensitive = true hides CLI output — value is still in state file in plaintext
3. for_each requires map or set — not a list; use toset()
4. terraform.tfvars is auto-loaded — other .tfvars files need -var-file
5. .terraform.lock.hcl records exact provider versions — commit to Git, never edit manually
6. terraform init -upgrade is required to update provider versions within constraints
7. local values are referenced as local.name — not locals.name
8. prevent_destroy only protects while the lifecycle block exists in config
9. count index shifts when you remove a middle item — for_each keys are stable
10. depends_on is a last resort — a direct attribute reference creates the dependency automatically

---

Verify current objectives at:
https://developer.hashicorp.com/terraform/tutorials/certification-004/associate-review-004
