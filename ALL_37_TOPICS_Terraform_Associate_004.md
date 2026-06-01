# Terraform Associate 004 — All 37 Topics Revision Guide

> **Exam:** HashiCorp Certified: Terraform Associate (004)
> **Topics:** 37 across 8 sections | **Duration:** 60 min | **Cost:** $70.50 | **Validity:** 2 years

---

## Table of Contents

**Sections**
- [Section Overview](#overview)
- [Section 1 — IaC with Terraform](#section-1)
- [Section 2 — Terraform Fundamentals](#section-2)
- [Section 3 — Core Terraform Workflow](#section-3)
- [Section 4 — Terraform Configuration](#section-4)
- [Section 5 — Terraform Modules](#section-5)
- [Section 6 — Terraform State Management](#section-6)
- [Section 7 — Maintain Infrastructure](#section-7)
- [Section 8 — HCP Terraform](#section-8)

**Revision Extras**
- [Practice Test Weak Areas](#weak-areas)
  - [State Management (3 wrong)](#weak-state)
  - [Core Workflow (1 wrong)](#weak-workflow)
  - [Modules (2 wrong)](#weak-modules)
  - [Configuration (7 wrong)](#weak-config)
  - [Fundamentals (2 wrong)](#weak-fundamentals)
  - [HCP Terraform (1 wrong)](#weak-hcp)
  - [IaC Concepts (2 wrong)](#weak-iac)
- [Master Cheat Sheet — All Exam Traps](#cheatsheet)
- [Practice Test Score Summary](#scores)

---

<a id="overview"></a>
## Section Overview

| # | Section | Topics | Exam Weight |
|---|---------|--------|-------------|
| 1 | Infrastructure as Code (IaC) with Terraform | 3 | ~8% |
| 2 | Terraform Fundamentals | 4 | ~11% |
| 3 | Core Terraform Workflow | 7 | ~19% |
| 4 | Terraform Configuration | 8 | ~22% |
| 5 | Terraform Modules | 4 | ~11% |
| 6 | Terraform State Management | 4 | ~11% |
| 7 | Maintain Infrastructure with Terraform | 3 | ~8% |
| 8 | HCP Terraform | 4 | ~11% |

---

<a id="section-1"></a>
## Section 1 — Infrastructure as Code (IaC) with Terraform

### Topic 1: Explain what IaC is

IaC means defining and managing infrastructure using configuration files (code) rather than manual processes or GUIs.

**Key benefits:**
- **Idempotency** — applying the same config multiple times produces the same result
- **Version control** — infra tracked in Git like application code
- **Reproducibility** — identical environments every time
- **Automation** — no manual clicks, no human error

---

### Topic 2: Describe the advantages of IaC patterns

| Advantage | Detail |
|-----------|--------|
| Consistency | Same config = same infra, no config drift |
| Speed | Automate provisioning at scale |
| Auditability | Git history = full change log |
| Collaboration | Code reviews for infra changes |
| Disaster recovery | Rebuild entire infra from code |

---

### Topic 3: Explain how Terraform manages multi-cloud, hybrid cloud, and service-agnostic workflows

- Terraform uses **providers** to interface with any API — AWS, Azure, GCP, Kubernetes, GitHub, Datadog, etc.
- A single Terraform config can provision resources across **multiple clouds simultaneously**
- The core workflow (`init → plan → apply`) is identical regardless of provider
- This makes Terraform **cloud-agnostic** — no vendor lock-in at the tooling level

---

<a id="section-2"></a>
## Section 2 — Terraform Fundamentals

### Topic 4: Install and version Terraform providers

Providers are declared in a `required_providers` block:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.5.0"
}
```

**Version constraint operators:**

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Exact version | `= 5.0.0` |
| `!=` | Exclude version | `!= 4.0.0` |
| `>`, `>=`, `<`, `<=` | Comparison | `>= 4.0` |
| `~>` | Pessimistic — allows rightmost component to increment only | `~> 5.0` allows 5.x, not 6.x |

> ⚠️ **Exam Trap — Pessimistic Constraint `~>`:**
> `~> 3.2.0` allows `>= 3.2.0, < 3.3.0` — only the **patch** can increment.
> Available: 3.2.0, 3.2.5, 3.3.0, 4.0.0 → Terraform downloads **3.2.5**.
> `~> 3.2` (no patch digit) allows `>= 3.2, < 4.0` → would allow 3.3.0.
>
> `~> 6.0.0` with available 6.0.5 and 6.1.0 → downloads **6.0.5** (6.1.0 blocked).

---

### Topic 5: Describe how Terraform uses providers

- Providers are **plugins** that translate Terraform resource declarations into API calls
- Each provider is downloaded during `terraform init` into `.terraform/providers/`
- Provider config goes in a `provider` block:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

- Providers are versioned independently from Terraform itself
- `.terraform.lock.hcl` locks provider versions for reproducibility — **commit this file to Git**

---

### Topic 6: Write Terraform configuration using multiple providers

```hcl
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

resource "aws_instance" "east" {
  # uses default aws provider
}

resource "aws_instance" "west" {
  provider = aws.west
}
```

---

### Topic 7: Explain how Terraform uses and manages state

- State is stored in `terraform.tfstate` (JSON format)
- Tracks what real-world resources Terraform manages
- Used to: plan changes (diff current vs desired), store metadata, improve performance
- **Never edit state manually** — use `terraform state` commands

> ⚠️ **Exam Trap — Deleted State File:**
> **Local backend:** If `terraform.tfstate` is deleted and you run `terraform plan`, Terraform sees **no existing state** and plans to **CREATE all resources** — no error.
> **Remote backend:** Same behavior — plans to create everything.
> **Fix:** Use `terraform import` to re-import existing resources back into state.

> ⚠️ **Exam Trap — State is created on `apply`, NOT on `init`:**
> `terraform init` only downloads providers/modules. State is only created/updated when `terraform apply` runs. If no state file exists and you run apply, Terraform creates all resources fresh and writes a new state file.

---

<a id="section-3"></a>
## Section 3 — Core Terraform Workflow

### Topic 8: Describe the Terraform workflow

```
Write → Init → Plan → Apply → Destroy
```

| Step | Command | What happens |
|------|---------|--------------|
| Initialize | `terraform init` | Downloads providers, sets up backend |
| Validate | `terraform validate` | Checks syntax and config validity |
| Format | `terraform fmt` | Auto-formats to canonical style |
| Plan | `terraform plan` | Shows what will change (dry run) |
| Apply | `terraform apply` | Makes changes to real infrastructure |
| Destroy | `terraform destroy` | Removes all managed infrastructure |

---

### Topic 9: Initialize a Terraform working directory

```bash
terraform init
```

**What `init` does:**
- Downloads provider plugins into `.terraform/`
- Initializes the backend
- Downloads modules
- Creates `.terraform.lock.hcl`

**Key flags:**

| Flag | Purpose |
|------|---------|
| `-upgrade` | Upgrade provider plugins to newest version allowed by constraints |
| `-reconfigure` | Reconfigure the backend, ignoring existing state |
| `-migrate-state` | Migrate state to a new backend |
| `-backend=false` | Skip backend initialization |

> ⚠️ **Exam Trap — `terraform init -upgrade`:**
> Upgrades **already-installed** providers to the latest version that still satisfies version constraints. Updates `.terraform.lock.hcl` with new hashes. It does NOT ignore constraints.

---

### Topic 10: Validate a Terraform configuration

```bash
terraform validate
```

- Checks configuration syntax and internal consistency
- Does NOT access remote state or provider APIs
- Run after `init` — requires providers to be initialized
- Returns zero exit code if valid, non-zero if errors found

---

### Topic 11: Generate and review an execution plan

```bash
terraform plan
terraform plan -out=tfplan       # save plan to file
terraform plan -destroy          # preview destroy
terraform plan -var="key=value"  # pass variable
```

- Shows: resources to add (+), change (~), destroy (-)
- Does NOT make any changes
- Saved plan file can be passed to `terraform apply tfplan` for exact execution

---

### Topic 12: Apply changes to infrastructure

```bash
terraform apply
terraform apply tfplan           # apply saved plan
terraform apply -auto-approve    # skip confirmation prompt
terraform apply -replace=<addr>  # force replace a specific resource
```

---

### Topic 13: Destroy Terraform-managed infrastructure

```bash
terraform destroy
terraform destroy -target=aws_instance.web  # destroy specific resource
```

- Equivalent to `terraform apply -destroy`
- Only destroys resources tracked in state

---

### Topic 14: Apply formatting and style adjustments

```bash
terraform fmt               # format current directory
terraform fmt -recursive    # format all subdirectories
terraform fmt -check        # exit non-zero if formatting needed (CI use)
terraform fmt -diff         # show diff of changes
```

- Uses canonical HCL formatting (2-space indentation, aligned `=` signs)

---

<a id="section-4"></a>
## Section 4 — Terraform Configuration

### Topic 15: Use and differentiate resource and data blocks

**`resource` block** — creates/manages infrastructure:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}
```
Reference: `aws_instance.web.id`

**`data` block** — reads existing infrastructure (read-only):
```hcl
data "aws_vpc" "production" {
  tags = { Name = "prod" }
}
```
Reference: `data.aws_vpc.production.id`

> ⚠️ **Exam Trap — Data Block Reference Syntax:**
> Always prefix with `data.` → `data.<type>.<name>.<attribute>`
> Resources do NOT use the `data.` prefix → `aws_instance.web.id`

---

### Topic 16: Refer to resource attributes and create cross-resource references

```hcl
resource "azurerm_sql_server" "main" { }

resource "azurerm_app_service" "web" {
  app_settings = {
    DB_CONNECTION = azurerm_sql_server.main.fully_qualified_domain_name
  }
}
```

- Terraform automatically infers dependency from the reference — DB is created first
- Use `depends_on` only when the dependency is NOT expressed by an attribute reference

---

### Topic 17: Use variables and outputs

**Variables:**
```hcl
variable "instance_type" {
  type        = string
  description = "EC2 instance type"
  default     = "t2.micro"
  sensitive   = false

  validation {
    condition     = contains(["t2.micro", "t3.small"], var.instance_type)
    error_message = "Must be t2.micro or t3.small."
  }
}
```

**Variable assignment precedence (lowest → highest):**
1. `default` in variable block
2. `terraform.tfvars` / `*.auto.tfvars` (auto-loaded)
3. `-var-file="file.tfvars"` flag
4. `-var="key=value"` flag
5. `TF_VAR_<name>` environment variables

> ⚠️ Only `terraform.tfvars` and `*.auto.tfvars` are **automatically loaded**. Any other `.tfvars` file requires `-var-file` flag.

**Outputs:**
```hcl
output "instance_ip" {
  description = "Private IP"
  value       = aws_instance.web.private_ip
  sensitive   = true
}
```

**Locals:**
```hcl
locals {
  app_name = "${var.project}-${var.env}"
}
# Reference: local.app_name
```

---

### Topic 18: Understand and use complex types

| Type | Example | Notes |
|------|---------|-------|
| `string` | `"hello"` | UTF-8 text |
| `number` | `42`, `3.14` | Int or decimal |
| `bool` | `true`/`false` | |
| `list(type)` | `["a","b"]` | Ordered, allows duplicates |
| `set(type)` | `toset(["a","b"])` | Unordered, no duplicates |
| `map(type)` | `{key = "val"}` | String keys, same value type |
| `object({})` | `{name=string, age=number}` | Named attrs, mixed types |
| `tuple([])` | `["a", 1, true]` | Fixed length, mixed types |
| `any` | — | Disables type checking |

> ⚠️ **Exam Trap — `for_each` with list:**
> `for_each` requires a **map or set**, NOT a list. Convert with `toset()`:
> ```hcl
> for_each = toset(var.namespaces)
> ```

---

### Topic 19: Use built-in functions

Terraform functions are **built-in only** — you cannot define custom functions.

**String:** `upper()`, `lower()`, `trimspace()`, `format()`, `join()`, `split()`, `replace()`
**Collection:** `length()`, `flatten()`, `merge()`, `keys()`, `values()`, `contains()`, `lookup()`, `zipmap()`
**Type conversion:** `tostring()`, `tonumber()`, `tobool()`, `tolist()`, `toset()`, `tomap()`
**Filesystem:** `file(path)`, `templatefile(path, vars)`
**Encoding:** `jsonencode()`, `jsondecode()`, `base64encode()`, `base64decode()`
**Numeric:** `min()`, `max()`, `abs()`, `ceil()`, `floor()`

---

### Topic 20: Use expressions and dynamic blocks

**Conditional:**
```hcl
instance_type = var.env == "prod" ? "t3.large" : "t2.micro"
```

**For expression:**
```hcl
[for s in var.names : upper(s)]
{for k, v in var.map : k => upper(v)}
[for s in var.names : upper(s) if s != ""]
```

**Dynamic block:**
```hcl
dynamic "ingress" {
  for_each = var.ingress_rules
  content {
    from_port = ingress.value.from_port
    to_port   = ingress.value.to_port
    protocol  = ingress.value.protocol
  }
}
```

**Splat:**
```hcl
aws_instance.web[*].id
```

---

### Topic 21: Use meta-arguments in resource blocks

#### depends_on
```hcl
resource "aws_instance" "app" {
  depends_on = [aws_db_instance.main]
}
```

#### count
```hcl
resource "aws_instance" "web" {
  count         = 3
  instance_type = "t2.micro"
  tags = { Name = "web-${count.index}" }
}
# Single: aws_instance.web[0].id
# All:    aws_instance.web[*].id
```

#### for_each
```hcl
resource "aws_s3_bucket" "b" {
  for_each = toset(["logs", "backups"])
  bucket   = each.key
}
# Reference: aws_s3_bucket.b["logs"].id
```

**count vs for_each:**
- `count` → indexed; removing middle element cascades destroy/recreate
- `for_each` → keyed; removing one key only destroys that one ✅ preferred

#### lifecycle
```hcl
lifecycle {
  create_before_destroy = true
  prevent_destroy       = true
  ignore_changes        = [tags, ami]
  replace_triggered_by  = [aws_ami.new]
}
```

> ⚠️ **Exam Trap — ignore_changes for external tag bots:**
> If a bot updates tags externally and you want Terraform to ignore future tag changes:
> `ignore_changes = [tags]` — applies tags at create time, ignores subsequent external changes.
> Do NOT use `ignore_changes = all` — too broad, ignores everything.

#### provider (resource-level)
```hcl
resource "aws_instance" "west" {
  provider = aws.west
}
```

---

### Topic 22: Use sensitive data and manage secrets in state

> ⚠️ **Critical Exam Trap — `sensitive = true` does NOT protect state:**
> `sensitive = true` ONLY suppresses display in CLI output.
> The value is **still stored in plaintext in `terraform.tfstate`**.

**Methods to keep sensitive data OUT of state:**

| Method | Protection level |
|--------|-----------------|
| **Ephemeral resources** (TF 1.10+) | Value NEVER written to state |
| **Encrypt state backend** (S3+KMS etc.) | Data in state but encrypted at rest |
| **External secret manager** | Value still in state when read via data source |

**Ephemeral resources:**
```hcl
ephemeral "aws_secretsmanager_secret_version" "api_key" {
  secret_id = "my-api-key"
}
# Value available in config but NEVER written to state
```

> ⚠️ When the exam says a generated key is "explicitly forbidden from being written to state" — the answer is **ephemeral resource**.

---

### Topic 23: Use data sources with lifecycle validation

```hcl
data "azurerm_resource_group" "rg" {
  name = var.resource_group_name

  lifecycle {
    postcondition {
      condition     = self.id != null && self.id != ""
      error_message = "Resource group not found."
    }
  }
}
```

> ⚠️ **Exam Trap — Validating data source results:**
> Use `lifecycle { postcondition {} }` on a data source to validate it returned a result.
> This is different from variable `validation` blocks (which only check input values before fetching).

---

<a id="section-5"></a>
## Section 5 — Terraform Modules

### Topic 24: Contrast module source options

```hcl
# Public Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
}

# Local path
module "vpc" {
  source = "./modules/vpc"
}

# Git repo + specific tag
module "vpc" {
  source = "git::https://github.com/org/repo.git?ref=v1.2.0"
}

# GitHub shorthand
module "vpc" {
  source = "github.com/org/repo//modules/vpc"
}
```

> ⚠️ **Exam Trap — Module source formats:**
> Public Registry format: `"namespace/module/provider"` — no URL, no `https://`
> Git at a tag: `"git::https://...git?ref=v1.0"` — uses `?ref=` for tag
> Both work without additional setup for public sources.

---

### Topic 25: Interact with module inputs and outputs

```hcl
module "network" {
  source      = "./modules/network"
  vpc_cidr    = "10.0.0.0/16"
  environment = var.env
}

# Access module output:
resource "aws_instance" "app" {
  subnet_id = module.network.public_subnet_id
}
```

---

### Topic 26: Describe variable scope within modules

- Module inputs are defined with `variable` blocks inside the module
- Module outputs are defined with `output` blocks inside the module
- Parent module can ONLY access child module's `output` values — not internal resources directly
- Variables do not inherit automatically — must be passed explicitly

---

### Topic 27: Use a module from the public Terraform Registry

```hcl
module "s3_bucket" {
  source  = "terraform-aws-modules/s3-bucket/aws"
  version = "4.1.0"
  bucket  = "my-bucket"
}
```

- Format: `<NAMESPACE>/<MODULE>/<PROVIDER>`
- `version` is optional but strongly recommended
- Downloaded to `.terraform/modules/` during `terraform init`

---

<a id="section-6"></a>
## Section 6 — Terraform State Management

### Topic 28: Describe the default local backend

- Default backend stores state in `terraform.tfstate` in working directory
- `terraform.tfstate.backup` keeps the previous state
- Not suitable for teams — no locking, no shared access
- Never commit `terraform.tfstate` to Git

---

### Topic 29: Describe state locking

- Prevents concurrent modifications to state
- Supported by most remote backends (S3+DynamoDB, HCP Terraform, Azure Blob, GCS)

> ⚠️ **Exam Trap — Concurrent `terraform apply`:**
> Two engineers applying simultaneously against the same remote backend → state locking kicks in.
> The **second apply fails with a lock error**. Infrastructure is NOT corrupted. First apply completes normally.

---

### Topic 30: Handle backend authentication methods

```hcl
terraform {
  backend "s3" {
    bucket         = "my-tf-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
    # Credentials via AWS env vars or IAM role — never hardcoded
  }
}
```

---

### Topic 31: Describe remote state storage benefits

| Benefit | Detail |
|---------|--------|
| Shared access | Team works against same state |
| Locking | Prevents concurrent apply conflicts |
| Encryption | State encrypted at rest |
| Versioning | S3 versioning = state history |
| Remote operations | HCP Terraform runs plans/applies remotely |

**Remote state data source:**
```hcl
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "my-tf-state"
    key    = "network/terraform.tfstate"
    region = "us-east-1"
  }
}
# Access: data.terraform_remote_state.network.outputs.vpc_id
```

---

<a id="section-7"></a>
## Section 7 — Maintain Infrastructure with Terraform

### Topic 32: Manage resource drift

- **Drift** = real-world infra differs from Terraform state
- Detect with `terraform plan`
- `terraform apply -refresh-only` — updates state to match reality without changing infra
- `ignore_changes` in `lifecycle` — accept certain external changes

---

### Topic 33: Manage Terraform state with CLI

| Command | Purpose |
|---------|---------|
| `terraform state list` | List all resources in state |
| `terraform state show <addr>` | Show details of one resource |
| `terraform state mv <src> <dst>` | Rename/move resource in state |
| `terraform state rm <addr>` | Remove from state (does NOT destroy real resource) |
| `terraform state pull` | Download remote state |
| `terraform state push` | Upload local state to remote |

---

### Topic 34: Use `terraform import`

```bash
terraform import aws_instance.web i-1234567890abcdef0
```

> ⚠️ **Exam Trap:**
> `terraform import` only updates **state** — it does NOT generate config.
> You must write the `resource` block first. After import, run `terraform plan` to verify alignment.
> TF 1.5+ `import` block with `-generate-config-out` can generate config, but is a separate feature.

---

<a id="section-8"></a>
## Section 8 — HCP Terraform

### Topic 35: Describe HCP Terraform capabilities

- Cloud-based Terraform execution platform (formerly Terraform Cloud)
- Provides: remote state, remote execution, team collaboration, Sentinel policies, private module registry
- Free tier available for small teams

---

### Topic 36: Describe workspaces in HCP Terraform

- Each workspace = one working directory + state + variables + run history
- Workspaces are isolated — separate state, variables, access controls

**CLI workspaces vs HCP workspaces:**

| | CLI Workspaces | HCP Workspaces |
|--|---------------|----------------|
| Scope | Multiple state files, same config dir | Full isolation with own state, vars, access |
| Use case | Lightweight env separation | Full team collaboration |

---

### Topic 37: Describe managed resources in HCP Terraform

**Variable sets** — reusable variables applied across multiple workspaces

**Run triggers** — automatically trigger downstream workspace run when upstream applies

**Remote execution modes:**
- `remote` — plan and apply run on HCP Terraform
- `local` — plan and apply run locally, state stored remotely
- `agent` — runs on self-hosted agents (for private networks)

**Sentinel** — policy-as-code that enforces governance rules before apply

**Private module registry** — internal modules, versioned, accessible org-wide

> ⚠️ **Exam Trap — Cross-workspace data sharing:**
> To automatically read an output from another HCP workspace (e.g. get a VM's public IP), use `terraform_remote_state` data source:
> ```hcl
> data "terraform_remote_state" "webserver" {
>   backend = "remote"
>   config = {
>     organization = "my-org"
>     workspaces = { name = "prod-webserver" }
>   }
> }
> # Access: data.terraform_remote_state.webserver.outputs.public_ip
> ```
> The source workspace must have an `output` block exposing the value. No manual variable updates needed.

---

<a id="weak-areas"></a>
## Practice Test Weak Areas — Targeted Revision

*Every question answered incorrectly in practice testing, with correct explanation.*

---

<a id="weak-state"></a>
### State Management — 3 wrong out of 8

**Q7 — Local backend, tfstate deleted, run `terraform plan`:**
Terraform sees no existing state and plans to **CREATE all resources** as if they're new. No error. If applied, duplicate resources get created. Fix with `terraform import`.

**Q10 — Remote backend, tfstate deleted, run `terraform plan`:**
Identical behavior. Terraform plans to create everything. The remote backend just stores state — if the file is gone, Terraform has zero memory of existing infrastructure.

**Q49 — Two engineers run `terraform apply` simultaneously (remote backend):**
State locking kicks in. The **second apply fails with a lock error**. Infrastructure is NOT corrupted. This is expected, correct behavior.

---

<a id="weak-workflow"></a>
### Core Workflow — 1 wrong out of 11

**Q47 — `terraform init -upgrade`:**
Upgrades already-installed provider plugins to the **latest version within version constraints**. Updates `.terraform.lock.hcl`. Does NOT ignore constraints — works within them.

---

<a id="weak-modules"></a>
### Modules — 2 wrong out of 5

**Q30 — Version constraint `~> 3.2.0`, available: 3.2.0, 3.2.5, 3.3.0, 4.0.0:**
`~> 3.2.0` = `>= 3.2.0, < 3.3.0`. Answer: **3.2.5**. (3.3.0 and 4.0.0 are blocked.)

**Q46 — Module source that works without additional setup:**
- Public Terraform Registry: `"hashicorp/consul/aws"` — no URL needed
- Git repo at tag: `"git::https://github.com/org/repo.git?ref=v1.2.0"` — `?ref=` pins the tag

---

<a id="weak-config"></a>
### Terraform Configuration — 7 wrong out of 13 (PRIORITY)

**Q11 — API key must NOT be written to state:**
Use an **ephemeral resource**. Ephemeral values are available in config at runtime but never persisted to state.
```hcl
ephemeral "random_password" "api_key" { length = 32 }
```

**Q13 — Reference `id` from a data block:**
`data.aws_vpc.production.id` — always prefix with `data.`

**Q20 — Validate data source returns a result:**
Use `lifecycle { postcondition {} }` on the data block. Runs after data is fetched.

**Q28 — Sensitive information NOT stored in state:**
`sensitive = true` only hides CLI output — value still in state. Only **ephemeral resources** truly prevent state storage.

**Q36 — Pass DB connection string to web app:**
Reference the DB attribute directly: `azurerm_sql_database.main.connection_string`. Terraform resolves after DB is created.

**Q51 — Ignore tag changes made by external bot:**
```hcl
lifecycle {
  ignore_changes = [tags]
}
```
Tags applied at create time; external changes ignored after that. Do not use `ignore_changes = all`.

**Q53 — `for_each` with `list(string)` variable:**
`for_each` requires map or set — must convert: `for_each = toset(var.namespaces)`. Use `each.key` to reference each value.

---

<a id="weak-fundamentals"></a>
### Terraform Fundamentals — 2 wrong out of 7

**Q19 — No state file, `terraform init` already run, run `terraform apply`:**
`terraform init` does NOT create state. State is created on first `apply`. Terraform applies normally and creates a fresh `terraform.tfstate`.

**Q50 — Provider at `6.0.2`, constraint `~> 6.0.0`, available `6.0.5` and `6.1.0`, run `terraform init -upgrade`:**
`~> 6.0.0` allows `>= 6.0.0, < 6.1.0`. Upgrades to **6.0.5**. `6.1.0` is blocked by the constraint.

---

<a id="weak-hcp"></a>
### HCP Terraform — 1 wrong out of 5

**Q45 — `prod-dns` needs public IP from `prod-webserver` automatically:**
Use `terraform_remote_state` data source in `prod-dns`. Automatically picks up the latest output value from `prod-webserver` on every plan/apply. No manual variable updates.

---

<a id="weak-iac"></a>
### IaC Concepts — 2 wrong out of 4

**Q38 — Single Terraform config: which statements can be true? (select three)**
1. Can deploy resources across multiple cloud providers (multiple provider blocks)
2. Can be used as a reusable module by other configs
3. Can manage multiple environments using CLI workspaces

**Q56 — How does Terraform differ from Chef/Ansible/Puppet?**

| | Terraform (IaC) | Chef / Ansible / Puppet |
|--|----------------|------------------------|
| Purpose | Provision infrastructure | Configure software on servers |
| Manages | VMs, networks, DNS, cloud resources | Packages, files, services on a server |
| Approach | Immutable — replace resources | Mutable — configure existing servers |
| State | Tracks infra state | No infra state |

Key answer: Terraform **provisions** the server. Chef **configures** what runs on it. They are **complementary**.

---

<a id="cheatsheet"></a>
## Master Cheat Sheet — All Key Exam Traps

| Topic | Trap | Correct Answer |
|-------|------|----------------|
| `sensitive = true` | Protects data in state? | No — only hides in CLI output |
| Deleted local tfstate | `terraform plan` errors? | No — plans to CREATE everything |
| Deleted remote tfstate | Same as local? | Yes — plans to create everything |
| Concurrent apply (remote) | Both succeed? | No — second gets lock error |
| `terraform init` | Creates state? | No — state created on first `apply` |
| `terraform init -upgrade` | Does what? | Upgrades providers within constraints |
| `~> 3.2.0` | Allows 3.3.0? | No — only `>= 3.2.0, < 3.3.0` |
| `~> 6.0.0` with 6.0.5 and 6.1.0 | Which is installed? | 6.0.5 |
| `for_each` with list | Works directly? | No — must use `toset()` first |
| Data source reference | Syntax? | `data.<type>.<name>.<attr>` |
| Validate data source result | How? | `lifecycle { postcondition {} }` |
| Key must NOT be in state | Use what? | Ephemeral resource |
| External tag bot changes tags | lifecycle rule? | `ignore_changes = [tags]` |
| `terraform import` | Generates config? | No — state only |
| `terraform.tfvars` | Auto-loaded? | Yes — other .tfvars need `-var-file` |
| Custom functions | Can you define them? | No — built-in only |
| `prevent_destroy` | Stops `terraform destroy`? | No — only stops plans containing that resource |
| Module source (public registry) | Format? | `"namespace/module/provider"` no URL |
| Module source (git + tag) | Format? | `"git::https://...git?ref=v1.0"` |
| Cross-workspace data | How to read? | `terraform_remote_state` data source |
| Terraform vs Chef | Difference? | Terraform provisions; Chef configures |

---

<a id="scores"></a>
## Practice Test Score Summary

| Section | Total | Correct | Incorrect | Score |
|---------|-------|---------|-----------|-------|
| Section 1: IaC with Terraform | 4 | 2 | 2 | 50% ⚠️ |
| Section 2: Terraform Fundamentals | 7 | 5 | 2 | 71% ⚠️ |
| Section 3: Core Workflow | 11 | 10 | 1 | 91% ✅ |
| Section 4: Terraform Configuration | 13 | 6 | 7 | 46% 🔴 |
| Section 5: Terraform Modules | 5 | 3 | 2 | 60% ⚠️ |
| Section 6: State Management | 8 | 5 | 3 | 63% ⚠️ |
| Section 7: Maintain Infrastructure | — | — | — | — |
| Section 8: HCP Terraform | 5 | 4 | 1 | 80% ✅ |

**Priority study order:**
1. 🔴 Section 4 (Configuration) — 46%
2. ⚠️ Section 1 (IaC concepts) — 50%
3. ⚠️ Section 5 (Modules) — 60%
4. ⚠️ Section 6 (State) — 63%
5. ⚠️ Section 2 (Fundamentals) — 71%

---

*Last updated: June 2026 | HashiCorp Certified Terraform Associate (004)*
