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

> **💡 IaC vs Configuration Management — the mental model (Exam Q56):**
>
> | | Terraform (IaC) | Chef / Ansible / Puppet |
> |---|---|---|
> | Purpose | Provision infrastructure | Configure software on servers |
> | Manages | VMs, networks, DNS, cloud resources | Packages, files, services on a server |
> | Approach | Immutable — replace resources | Mutable — configure existing servers |
> | State | Tracks infra state | No infra state |
>
> Terraform **provisions** the server. Chef **configures** what runs on it. They are complementary — Terraform creates the VM, Ansible installs Nginx on it.

> **💡 Single config capabilities (Exam Q38 — select three):**
> A single Terraform configuration CAN:
> 1. Deploy resources across **multiple cloud providers** in one config using multiple provider blocks
> 2. Be used as a **reusable module** called by other configurations
> 3. Manage **multiple environments** using CLI workspaces with separate state per workspace

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

> **💡 Pessimistic constraint `~>` — the mental model:**
> Count the digits. `~> 5.0` has two digits — only the rightmost (minor) can grow. So `5.x` is allowed, `6.0` is not.
> `~> 5.0.0` has three digits — only the rightmost (patch) can grow. So `5.0.x` is allowed, `5.1.0` is not.
> Think of it as: everything to the left is frozen, only the rightmost number can increment.
>
> **Exam Q30:** `~> 3.2.0` with available 3.2.0, 3.2.5, 3.3.0, 4.0.0 → Terraform downloads **3.2.5** (3.3.0 and 4.0.0 are blocked)
> **Exam Q50:** `~> 6.0.0` with `terraform init -upgrade`, available 6.0.5 and 6.1.0 → upgrades to **6.0.5** (6.1.0 is blocked by the constraint)

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

> **💡 Exam Q47 — What does `terraform init -upgrade` actually do?**
> It upgrades already-installed provider plugins to the **latest version that still satisfies your version constraints**. It does NOT ignore constraints — it works within them and updates `.terraform.lock.hcl` with the new version and checksums. Use it when a newer patch or minor version of a provider is available and you want to pull it in without editing the lock file manually.

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

> **💡 Exam Q13 — Data source reference syntax trap:**
> Always prefix data source references with `data.` → `data.<type>.<name>.<attribute>`
> Example: `data.aws_vpc.production.id`
> Resource references do NOT use the `data.` prefix → `aws_instance.web.id`
> This distinction is a frequent exam trap — one extra word changes everything.

> **💡 Exam Q19 — `terraform init` does NOT create state:**
> State is created only when `terraform apply` runs successfully for the first time. If you run `terraform init` and then `terraform apply` with no existing state file, Terraform applies normally, creates all resources fresh, and writes a new `terraform.tfstate`. No error — this is normal first-run behaviour.

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
terraform init -reconfigure     # reinitialise backend; discard existing state location
terraform init -migrate-state   # reinitialise backend; copy existing state to new backend
```

### Backend Change Flags — Exam Critical

When you change the `backend` block in your configuration — for example switching from S3 to HCP Terraform, or from local to S3 — running `terraform init` alone will error. Terraform detects that the backend has changed and requires you to explicitly tell it what to do with the existing state. You must use one of two flags:

**`-migrate-state`** — Copies the existing state from the old backend to the new backend during reinitialisation. Use this when you want to preserve and continue managing the same infrastructure from the new backend. After this command, the new backend holds your state and Terraform is configured to use it going forward. The old state file is not automatically deleted — you should verify the migration succeeded before removing it.

**`-reconfigure`** — Reinitialises the backend without touching the existing state at all. It does not copy or move state — it simply reconfigures Terraform to use the new backend as if starting fresh. Use this when you deliberately do not want the old state moved — for example when setting up a new environment from scratch, or when the old state is irrelevant.

The key distinction: `-migrate-state` preserves your existing state by moving it. `-reconfigure` ignores the existing state entirely.

```
Changing backend config + terraform init
        |
        ├── terraform init -migrate-state
        |       Copies existing state → new backend
        |       Use when: switching backends for existing infrastructure
        |
        └── terraform init -reconfigure
                Ignores existing state; starts fresh in new backend
                Use when: new environment, or existing state is irrelevant
```

### Exam Traps

- terraform init is idempotent — safe to re-run at any time.
- It does not read, modify, or apply any changes to infrastructure.
- Required after: adding a new provider, changing the backend, adding a module source.
- The .terraform/ directory is NOT committed to version control. The lock file IS.
- Changing the backend block without `-migrate-state` or `-reconfigure` causes init to error — Terraform will not silently pick one for you.

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

> **💡 Exam Q53 — `for_each` with a `list(string)` variable:**
> `for_each` requires a **map or set**, NOT a list. A list has numeric indexes — not stable string keys — so Terraform cannot use it as a `for_each` source directly. Convert with `toset()`:
> ```hcl
> variable "namespaces" {
>   type    = list(string)
>   default = ["platform", "apps", "observability"]
> }
> resource "kubernetes_namespace" "ns" {
>   for_each = toset(var.namespaces)   # required — list → set
>   metadata { name = each.key }
> }
> ```
> After `toset()`, `each.key` and `each.value` are both the string value (e.g. "platform"). Removing one entry only destroys that one resource — unlike `count` which would cascade.

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

> **💡 Exam Q20 — Validate that a data source returns a result:**
> Use `lifecycle { postcondition {} }` on the data source block. It runs after the data is fetched and fails with a clear error if the condition is not met:
> ```hcl
> data "azurerm_resource_group" "rg" {
>   name = var.resource_group_name
>   lifecycle {
>     postcondition {
>       condition     = self.id != null && self.id != ""
>       error_message = "Resource group '${var.resource_group_name}' does not exist."
>     }
>   }
> }
> ```
> Key distinction: variable `validation` runs before plan and checks user input. `postcondition` runs after the data is fetched and checks what the cloud returned.

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

> **💡 Exam Q28 — Critical trap: `sensitive = true` does NOT protect state:**
> `sensitive = true` on a variable or output **only suppresses the value in CLI terminal output** (shows `(sensitive value)` instead). The actual value is still stored in **plaintext JSON** in `terraform.tfstate`. Anyone with access to the state file can read it. The flag is purely cosmetic for terminal display.
>
> **Exam Q11 — API key that must NOT be written to state:**
> The question says the key is "explicitly forbidden from being written to state." The answer is always **ephemeral resource**:
> ```hcl
> ephemeral "aws_secretsmanager_secret_version" "api_key" {
>   secret_id = "prod/api/key"
> }
> resource "aws_instance" "web" {
>   user_data = ephemeral.aws_secretsmanager_secret_version.api_key.secret_string
>   # This value is NEVER written to state
> }
> ```
> Ephemeral values are computed in memory at runtime and discarded — they never touch state.

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

> **💡 Exam Q46 — Module source formats that work without additional setup:**
> - **Public Terraform Registry:** `"hashicorp/consul/aws"` — just `namespace/module/provider`, no URL prefix. No credentials needed for public modules.
> - **Git repo at a specific tag:** `"git::https://github.com/org/repo.git?ref=v1.2.0"` — `?ref=` pins the tag. Works for public repos without extra config.
> Private registries or SSH Git repos need credentials — those require additional setup.
>
> **Exam Q30 — `~> 3.2.0` version constraint:**
> Available versions: 3.2.0, 3.2.5, 3.3.0, 4.0.0 → Terraform downloads **3.2.5**
> `~> 3.2.0` = `>= 3.2.0, < 3.3.0`. Only the patch digit can increment.

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

> **💡 Exam Q7 (local backend) and Q10 (remote backend) — Deleted state file + `terraform plan`:**
> Whether local or remote backend — if the state file is deleted and you run `terraform plan`, Terraform sees **no existing state** and plans to **CREATE all resources** as if starting from scratch. It does not error. It does not detect the existing infrastructure. It simply thinks nothing exists.
> The infrastructure is not affected — only Terraform's knowledge of it is lost.
> **Fix:** Use `terraform import` to bring existing resources back into state one by one.

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

> **💡 Exam Q49 — Two engineers run `terraform apply` simultaneously against the same remote backend:**
> State locking kicks in immediately. The **second apply fails with a lock acquisition error** and cannot proceed until the first completes and releases the lock. Infrastructure is NOT corrupted — this is exactly the behaviour state locking is designed to produce.
> The first apply proceeds normally. The second engineer must wait and re-run after the lock is released.
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

> **💡 Exam Q36 — Pass a DB connection string to a web app resource:**
> Reference the database attribute directly — Terraform resolves it after the DB is created:
> ```hcl
> resource "azurerm_app_service" "web" {
>   app_settings = {
>     DB_CONNECTION = azurerm_sql_database.main.connection_string
>   }
> }
> ```
> The attribute reference creates an **implicit dependency** — no `depends_on` needed. Terraform knows to create the DB first because of the reference.

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

> **💡 The conceptual trap — CLI workspaces vs HCP Terraform workspaces:**
>
> | | CLI Workspaces | HCP Terraform Workspaces |
> |---|---|---|
> | State isolation | Yes — separate state per workspace | Yes — each workspace has own state |
> | Variable management | None — share same config and .tfvars | Full per-workspace variable store |
> | Access control | None | Team-based permissions |
> | Run history | None | Full audit log |
> | VCS integration | None | Direct GitHub/GitLab connection |
>
> CLI workspaces are lightweight — same config, different state file. HCP workspaces are full isolation units with their own everything. The exam may test that you know these are fundamentally different despite sharing the word "workspace".
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

<a id="section-8"></a>
# SECTION 8 — HCP Terraform (~11%)

---

## 8a. What is HCP Terraform? Organizations and Authentication

### Concept

**HCP Terraform** (HashiCorp Cloud Platform Terraform, formerly Terraform Cloud) is HashiCorp's SaaS platform that wraps open-source Terraform with collaboration, security, and governance features that the CLI alone cannot provide. It is the production-grade way to use Terraform in a team.

The fundamental unit of HCP Terraform is the **organization** — a shared space where teams, workspaces, policies, and modules live. Every user account belongs to one or more organizations. All workspaces, variable sets, and policies are scoped to an organization.

### Tiers

| Tier | Key capabilities |
|---|---|
| **Free** | Up to 500 resources managed; 1 concurrent run; state storage; VCS integration; private module registry |
| **Plus** | Audit logging; SSO; policy enforcement (Sentinel + OPA); team management; concurrent runs |
| **Enterprise** | Self-hosted option; advanced audit; clustering |

The exam tests that you know HCP Terraform has a **free tier with meaningful features** — state storage, VCS integration, and the private module registry are all free.

### Authentication

```bash
# terraform login stores credentials in:
# ~/.terraform.d/credentials.tfrc.json

terraform login              # opens browser; generates API token; stores locally
terraform logout             # removes stored credentials
```

The API token is used by Terraform CLI to communicate with HCP Terraform for state operations and remote runs. Without login, CLI-driven remote runs will fail.

### Exam Traps

- HCP Terraform is the new name for Terraform Cloud — the exam may use either name.
- The free tier includes state storage and the private module registry — these are not paid features.
- `terraform login` stores the token locally; it does not embed it in config files.

---

## 8b. HCP Terraform Workspaces — Deep Dive

### Concept

An **HCP Terraform workspace** is the fundamental operational unit. It is not the same as a CLI workspace. Understanding this distinction is exam-critical.

| Aspect | CLI Workspace | HCP Terraform Workspace |
|---|---|---|
| Purpose | Isolated state for the same config | Complete environment — state + variables + runs + access control |
| Variable management | None — use .tfvars files | Built-in per-workspace variable store |
| Access control | None | Team-based permissions |
| Run history | None | Full audit log of every plan and apply |
| VCS integration | None | Direct connection to GitHub/GitLab/Bitbucket |
| Locking | Via backend | Built-in |

Each HCP Terraform workspace has:
- Its own state file (with version history)
- Its own set of Terraform and environment variables
- Its own run queue and history
- Its own team access settings
- One optional VCS repository connection

### Three Workflow Types

**VCS-Driven Workflow (most common for teams):**

A workspace is connected to a specific VCS repository and branch. Terraform runs are triggered automatically by code changes.

1. Developer pushes a commit or opens a pull request
2. HCP Terraform detects the change and automatically queues a plan
3. The plan output appears as a PR status check in GitHub/GitLab
4. On merge to the target branch, HCP Terraform queues an apply
5. An operator reviews and confirms the apply (if auto-apply is off)

This is the GitOps model for infrastructure. Every infrastructure change is traceable to a Git commit.

**CLI-Driven Workflow:**

You run `terraform plan` and `terraform apply` from your local terminal, but the actual execution happens on HCP Terraform's infrastructure — not your machine. State is stored remotely. Useful when you want remote execution without full VCS integration.

```hcl
# Configure the workspace in your config
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "prod-network"      # single workspace by name
      # tags = ["prod", "network"]  # or select by tags
    }
  }
}
```

```bash
terraform login
terraform init     # connects to HCP Terraform workspace
terraform plan     # runs remotely on HCP Terraform
terraform apply    # runs remotely; shows output streaming to your terminal
```

**API-Driven Workflow:**

Runs are triggered programmatically via the HCP Terraform API — no CLI or VCS required. Used by CI/CD pipelines (Jenkins, GitHub Actions, CircleCI) that call the API directly to queue runs.

### Execution Modes

The execution mode is a per-workspace setting that determines **where the Terraform process actually runs** — meaning where `terraform plan` and `terraform apply` execute. This is one of the most tested concepts in this section because it is counterintuitive: even when you type `terraform plan` into your terminal, the actual computation may not be happening on your machine.

**Remote (default):**

When a workspace is in Remote mode, the Terraform CLI on your machine is acting as a thin client. When you run `terraform plan` or `terraform apply`, the CLI packages your configuration, sends it to HCP Terraform over HTTPS, and HCP Terraform executes the operation on its own managed compute infrastructure. Your terminal just streams the output back to you in real time.

The consequence of this is critical for the exam: **environment variables set on your local machine are not available to the Terraform process**, because that process is running on HCP Terraform's machines, not yours. Cloud provider credentials (`AWS_ACCESS_KEY_ID`, `GOOGLE_CREDENTIALS`, etc.) must be stored as Environment Variables in the HCP Terraform workspace itself — not on the developer's laptop.

**Local:**

When a workspace is in Local mode, `terraform plan` and `terraform apply` execute on your local machine — the same as open-source Terraform. The only thing HCP Terraform provides is remote state storage and locking. In this mode, environment variables on your machine ARE available to Terraform because it is running locally. Variables stored in the HCP Terraform workspace are NOT automatically used — you manage variables yourself via `.tfvars` files or local env vars.

Use Local mode when you want the safety of remote state without changing where execution happens — common during migrations from local state.

**Agent:**

When a workspace is in Agent mode, Terraform runs on **your own self-hosted agent** — a machine you control that is registered with HCP Terraform. HCP Terraform sends the run to the agent, which executes it locally within your network. This is required when your infrastructure is inside a private network (VPC, on-premises datacenter) that HCP Terraform's servers cannot reach directly.

```
Remote mode:  Your terminal → HCP Terraform servers → Cloud provider API
Local mode:   Your terminal → Your machine → Cloud provider API (state stored in HCP)
Agent mode:   Your terminal → HCP Terraform → Your agent machine → Cloud provider API
```

### Run Pipeline (phases in order)

Every run in HCP Terraform goes through a defined pipeline. Understanding the order is exam-tested:

```
1. Plan           — terraform plan executes; generates the execution plan
2. Cost Estimation — (paid tier) estimates cost of proposed changes
3. Policy Check    — (Sentinel/OPA) evaluates policies against the plan
4. Apply           — terraform apply executes (requires confirmation unless auto-apply enabled)
```

**Key point:** Policy checks happen AFTER the plan but BEFORE the apply. If a policy check fails, the apply is blocked. An operator can override a soft-mandatory failure; a hard-mandatory failure cannot be overridden.

### Workspace States

| State | Meaning |
|---|---|
| **Active** | A run is currently in progress |
| **Errored** | The last run encountered an error |
| **Needs Attention** | Awaiting manual confirmation (e.g., a soft-mandatory policy failure) |
| **Locked** | Workspace is locked; no new runs can start |

A workspace can be manually locked to prevent runs while maintenance is in progress.

### Exam Traps

- The `terraform cloud` block replaces the `backend "remote"` block — both work but `cloud` is the modern approach.
- In Remote execution mode, env vars like `AWS_ACCESS_KEY_ID` must be set as Environment Variables in the HCP Terraform workspace — not on your local machine.
- API-driven workflow is the third workflow type — the exam sometimes frames questions around all three.
- Cost estimation runs only on the paid tier and only for supported providers.

---

## 8c. HCP Terraform Projects ⭐ NEW 004

### Concept

**Projects** are an organisational layer that sits above workspaces. As organisations scale to dozens or hundreds of workspaces, managing access and shared configuration at the individual workspace level becomes unworkable. Projects solve this.

A project is a named container that groups related workspaces. Every workspace belongs to exactly one project. A default project exists in every organisation and captures workspaces not explicitly assigned.

### What Projects Enable

**Project-level access control:** Instead of granting a team access to 20 individual workspaces one by one, you assign that team to the project — and they automatically get access to all 20 workspaces in it. When a new workspace is added to the project, the team gains access automatically.

**Project-level variable sets:** Variable sets can be scoped to a project, making their variables available to every workspace in that project — without manually attaching the variable set to each workspace.

**Logical organisation:** The HCP Terraform UI groups workspaces by project. A "payments" project might contain workspaces for payments-dev, payments-staging, payments-prod — making it clear which workspaces belong together.

### Project vs Workspace vs Organisation

```
Organisation
  └── Project A (e.g., "Payments Platform")
        ├── Workspace: payments-dev
        ├── Workspace: payments-staging
        └── Workspace: payments-prod
  └── Project B (e.g., "Shared Infrastructure")
        ├── Workspace: network-prod
        └── Workspace: dns-prod
```

### Exam Traps

- Projects are an HCP Terraform concept — they do not exist in open-source Terraform.
- A workspace belongs to **exactly one** project — it cannot be in multiple projects.
- Project-level permissions are additive — a team with project access gets that access on top of any org-level access.
- The default project always exists and cannot be deleted.

---

## 8d. Variable Sets and Sensitive Variables

### Variable Sets

A **variable set** is a reusable, named collection of variables that can be applied to multiple workspaces without duplicating configuration. It is HCP Terraform's solution to the "credentials in every workspace" problem.

**Without variable sets:** Every one of your 50 workspaces stores `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` separately. Rotating credentials means updating 50 workspaces.

**With variable sets:** One variable set called "AWS Prod Credentials" holds both variables. It is applied to all 50 workspaces. Rotating credentials means updating one variable set.

### Variable Set Scope

Variable sets can be applied at three levels:

**Organisation-wide:** Applies to every workspace in the organisation automatically. Used for global credentials or org-level defaults.

**Project-level:** Applies to every workspace in a specific project. Used for team- or product-specific configuration.

**Workspace-level:** Applied to specific individual workspaces. Most granular scope.

When the same variable key exists in multiple variable sets that apply to the same workspace, the most specific scope wins: workspace-level overrides project-level, which overrides org-level.

### Variable Types

Every variable in HCP Terraform is one of two types:

**Terraform variable:** Equivalent to setting `-var="key=value"` on the CLI or setting `TF_VAR_key` in the environment. The value is available in your HCL as `var.key`. Must match a declared `variable` block.

**Environment variable:** Set as a shell environment variable during the run. The Terraform process and any external programs it calls can read these. Used for provider credentials (`AWS_ACCESS_KEY_ID`, `GOOGLE_CREDENTIALS`), and any flags that programs check via env vars (`TF_LOG`, `HELM_DRIVER`).

### Sensitive Variables

Any variable in HCP Terraform can be marked **sensitive**. Sensitive variables:
- Are write-once after the first save — you can overwrite but never read the current value back through the UI or API
- Are never shown in run logs
- Are masked in plan output

This is how cloud provider credentials should be stored. Even admins cannot read them back after saving — only overwrite them.

### Exam Traps

- Variable set scope priority: workspace-level overrides project-level, which overrides org-level.
- Terraform variables in HCP must match a declared variable block in the config — otherwise Terraform ignores them.
- Environment variables are for credentials and process-level flags — not HCL input values.
- Sensitive in HCP Terraform means unreadable after saving — this is stronger than `sensitive = true` in open-source Terraform (which just hides CLI output).

---

## 8e. Sentinel Policies and Policy as Code

### Concept

**Sentinel** is HashiCorp's policy-as-code framework. It allows organisations to write rules that are automatically evaluated against every Terraform plan — before the apply happens. This is governance automation: instead of relying on humans to manually review every plan for compliance, Sentinel codifies the rules and enforces them programmatically.

Sentinel policies are available on the **Plus tier and above** — not the free tier.

### What Sentinel Enforces

Sentinel can inspect the full Terraform plan and enforce rules like:

- All S3 buckets must have versioning enabled
- EC2 instances must not use instance types larger than t3.medium in non-prod workspaces
- All resources must have required tags (cost_centre, owner, environment)
- RDS instances must have deletion_protection enabled
- No security groups may allow 0.0.0.0/0 on port 22

### Policy Sets

Policies are grouped into **policy sets** — collections of related policies that are applied to workspaces. A policy set can be applied to:
- All workspaces in the organisation
- Specific projects
- Specific individual workspaces

A single workspace can have multiple policy sets applied. All policies in all applied sets are evaluated on every run.

### Enforcement Levels

This is exam-critical. Sentinel has three enforcement levels that determine what happens when a policy fails:

**Advisory:** The policy failure is logged as a warning but does NOT block the apply. The run continues normally. Used for informational policies or when rolling out a new policy gradually.

**Soft Mandatory:** The policy failure BLOCKS the apply by default. However, an authorised user (with "override policy checks" permission) can manually override the failure and allow the apply to proceed. Used for policies where exceptions are legitimately needed but must be reviewed.

**Hard Mandatory:** The policy failure BLOCKS the apply with no override possible — not even by org admins. The only way to proceed is to fix the violation in the code. Used for absolute compliance requirements (e.g., regulatory mandates).

```
Plan
  ↓
Policy Check
  ├── Advisory: warning shown, apply continues
  ├── Soft Mandatory: apply blocked; authorised user CAN override
  └── Hard Mandatory: apply blocked; NO override possible
  ↓
Apply
```

### OPA (Open Policy Agent)

HCP Terraform also supports **OPA (Open Policy Agent)** as an alternative policy engine. OPA uses the Rego language for writing policies. Like Sentinel, OPA policies are evaluated after the plan and before the apply. OPA is open-source; Sentinel is HashiCorp's proprietary language.

The exam may ask: both Sentinel AND OPA are supported as policy engines in HCP Terraform.

### Exam Traps

- Sentinel policies run **after** plan and **before** apply — this placement in the run pipeline is frequently tested.
- Hard Mandatory cannot be overridden by anyone — not even admins.
- Soft Mandatory CAN be overridden but only by users with the "override policy checks" permission.
- Advisory policies never block a run — they only add warnings.
- Sentinel is a paid feature (Plus tier). The free tier does not include policy enforcement.

---

## 8f. Private Module Registry

### Concept

The **Private Module Registry** is HCP Terraform's internal module store for your organisation. It works like the public Terraform Registry but for modules you want to keep internal — proprietary infrastructure patterns, company-standard configurations, or modules that embed organisation-specific defaults.

The private registry is available on the **free tier**.

### Why it exists

Without a private registry, sharing internal modules across teams requires either copy-pasting HCL, maintaining a shared Git repository that everyone references by URL, or publishing to the public registry (exposing internal infrastructure). The private registry provides a proper, versioned, documented home for internal modules that integrates natively with HCP Terraform workspaces.

### How it works

**Publishing:** Modules are published to the private registry by connecting a VCS repository that follows the standard naming convention: `terraform-<PROVIDER>-<MODULE_NAME>`. When you push a Git tag (e.g., `v1.2.0`), HCP Terraform publishes that version to the private registry automatically.

**Consuming:** Modules from the private registry are referenced with the private registry address:

```hcl
module "vpc" {
  # Private registry: <HOSTNAME>/<ORGANIZATION>/<MODULE_NAME>/<PROVIDER>
  source  = "app.terraform.io/my-org/vpc/aws"
  version = "~> 2.0"
}
```

The `app.terraform.io` hostname tells Terraform to fetch the module from HCP Terraform's registry rather than the public registry.

**Browsing:** The HCP Terraform UI shows all published modules with their documentation (auto-generated from README.md), input variables, outputs, and version history — identical to the public registry experience.

### Private Registry vs Public Registry

| Aspect | Public Registry | Private Registry |
|---|---|---|
| Access | Anyone | Your organisation only |
| Publishing | HashiCorp-curated or community | Your VCS repos |
| Source address | `hashicorp/vpc/aws` | `app.terraform.io/my-org/vpc/aws` |
| Cost | Free | Free (HCP Terraform free tier) |

### Exam Traps

- The private module registry is a **free** HCP Terraform feature — not a paid one.
- The naming convention for VCS repos is `terraform-<PROVIDER>-<MODULE_NAME>` — this triggers auto-publishing.
- Modules from the private registry require `terraform login` to authenticate during `terraform init`.
- The private registry hosts modules only — not providers.

---

## 8g. Teams, Permissions, and Access Control

### Concept

HCP Terraform uses a team-based access control model. Permissions are granted to **teams**, and users belong to teams. This is the standard enterprise IAM pattern: manage team membership rather than individual user permissions.

### Organisation-Level Roles

Every user in an organisation has one of these roles:

| Role | What they can do |
|---|---|
| **Owner** | Full control — manage members, teams, billing, settings, delete org |
| **Member** | Default role; can be granted access to workspaces via teams |

Owners are rare — typically only the platform team leads who manage HCP Terraform itself.

### Teams

**Teams** are named groups of users within an organisation. You create teams like "backend-engineers", "security-reviewers", "ops-team" and assign users to them. Permissions are then granted to the team, not individual users.

Teams can be granted permissions at two scopes:

**Organisation-level team permissions:**
- Manage workspaces
- Manage policies
- Manage policy overrides
- Manage VCS settings
- Manage modules (private registry)
- Manage providers

**Workspace-level team permissions:**
Each team can be assigned a permission level for a specific workspace:

| Permission | What it allows |
|---|---|
| **Read** | View runs, state, variables (non-sensitive) |
| **Plan** | Read + queue plans (but not apply) |
| **Write** | Plan + confirm and apply runs, lock/unlock workspace, manage variables |
| **Admin** | Write + manage team access, delete workspace, force-unlock |

### Implicit vs Explicit Workspace Access

- **Owners** always have admin-level access to all workspaces — no explicit grant needed.
- All other users need explicit team-level grants to access workspaces.
- A user with no team assignments cannot see or access any workspaces.

### Exam Traps

- Permissions are assigned to **teams**, not individual users.
- A user inherits the union of permissions from all teams they belong to.
- "Plan" permission lets a user queue a plan but NOT apply it — apply requires "Write" or higher.
- Workspace admin permission does NOT make the user an org owner — those are separate scopes.

---

## 8h. Run Triggers and Cross-Workspace Data Sharing

### Run Triggers

A **run trigger** creates an automatic dependency between two workspaces. When a **source workspace** completes a successful apply, the **downstream workspace** is automatically queued for a new plan and apply.

This is used to chain infrastructure layers that must be applied in sequence:

```
Workspace: shared-network (VPC, subnets, route tables)
     |
     | run trigger
     v
Workspace: application-infra (EC2, RDS — depends on VPC IDs)
     |
     | run trigger
     v
Workspace: dns-config (DNS records pointing to app load balancers)
```

When `shared-network` successfully applies, `application-infra` is automatically queued. When `application-infra` successfully applies, `dns-config` is automatically queued. Each workspace manages its own state independently.

Run triggers replace the need for monolithic configurations that provision everything in one workspace.

### `terraform_remote_state` Data Source

**`terraform_remote_state`** allows one workspace to read the **outputs** of another workspace's state file. It is how you pass data between independent workspaces without coupling their configurations.

```hcl
# In the application-infra workspace config:
data "terraform_remote_state" "network" {
  backend = "remote"
  config = {
    organization = "my-org"
    workspaces = {
      name = "shared-network"
    }
  }
}

resource "aws_instance" "app" {
  # Use the VPC ID from the network workspace's output
  subnet_id              = data.terraform_remote_state.network.outputs.private_subnet_id
  vpc_security_group_ids = [data.terraform_remote_state.network.outputs.app_sg_id]
}
```

The workspace reading the remote state must have **at least Read access** to the source workspace.

### Run Triggers vs terraform_remote_state

These are complementary, not alternatives:

| Mechanism | Purpose |
|---|---|
| **Run trigger** | Automates *when* downstream workspace runs — triggers the run |
| **terraform_remote_state** | Passes *data* between workspaces — reads outputs from another state |

You typically use both together: run triggers ensure the downstream workspace re-plans when upstream changes, and `terraform_remote_state` passes the updated output values into the downstream configuration.

### Exam Traps

- `terraform_remote_state` reads outputs only — not the full state. The source workspace must declare `output` blocks for data to be readable.
- The workspace reading remote state needs Read permission on the source workspace.
- Run triggers are one-directional — A triggers B, but B does not trigger A.

> **💡 Exam Q45 — `prod-dns` workspace needs the public IP from `prod-webserver` automatically:**
> Use `terraform_remote_state` in the `prod-dns` workspace to read the output of `prod-webserver`:
> ```hcl
> data "terraform_remote_state" "webserver" {
>   backend = "remote"
>   config = {
>     organization = "my-org"
>     workspaces = { name = "prod-webserver" }
>   }
> }
> # Use in DNS record:
> records = [data.terraform_remote_state.webserver.outputs.public_ip]
> ```
> `prod-webserver` must declare the IP as an `output` block. Every time `prod-webserver` applies and the IP changes, `prod-dns` picks up the new value on its next plan/apply — no manual variable updates needed.
> Use a **run trigger** alongside this so `prod-dns` automatically re-plans when `prod-webserver` applies.

---

*Verify current objectives at:*
*https://developer.hashicorp.com/terraform/tutorials/certification-004/associate-review-004*

---

---
