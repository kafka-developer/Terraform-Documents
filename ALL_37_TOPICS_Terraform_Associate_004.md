# Terraform Associate 004 — All 37 Topics Revision Guide

> **Exam:** HashiCorp Certified: Terraform Associate (004)
> **Topics:** 37 across 8 sections | **Duration:** 60 min | **Cost:** $70.50 | **Validity:** 2 years

---

## 📊 Section Overview

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

## SECTION 1 — Infrastructure as Code (IaC) with Terraform

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

## SECTION 2 — Terraform Fundamentals

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
| `~>` | Pessimistic / allows rightmost increment only | `~> 5.0` allows 5.x, not 6.x |

> ⚠️ **Exam Trap — `~>` Pessimistic Constraint:**
> `~> 3.2.0` allows `>= 3.2.0, < 3.3.0` — only the **patch** version can increment.
> Available: 3.2.0, 3.2.5, 3.3.0, 4.0.0 → Terraform downloads **3.2.5** (highest allowed).
> `~> 3.2` (no patch) allows `>= 3.2, < 4.0` → would allow 3.3.0.

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
- Used to: plan changes (diff current vs desired), store metadata, improve performance (avoid API calls)
- **Never edit state manually** — use `terraform state` commands

> ⚠️ **Exam Trap — Deleted State File:**
> **Local backend:** If `terraform.tfstate` is deleted and you run `terraform plan`, Terraform sees **no existing state** and plans to **CREATE all resources** (even though they exist in reality). It does NOT error — it just thinks nothing exists.
> **Remote backend:** Same behavior — if the remote tfstate is deleted, `terraform plan` plans to create everything. Terraform has no memory of the existing infra.
> **Fix:** Use `terraform import` to re-import existing resources back into state.

---

## SECTION 3 — Core Terraform Workflow

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
> `-upgrade` does NOT just install providers — it **upgrades already-installed providers** to the latest version that still satisfies the version constraints in your config. It updates `.terraform.lock.hcl` with the new provider hashes.

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

## SECTION 4 — Terraform Configuration

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
> Example: `data.aws_vpc.production.id`
> Resources do NOT use the `data.` prefix → `aws_instance.web.id`

---

### Topic 16: Refer to resource attributes and create cross-resource references

```hcl
resource "azurerm_sql_server" "main" {
  # ...
}

resource "azurerm_app_service" "web" {
  connection_string {
    value = azurerm_sql_server.main.fully_qualified_domain_name
  }
}
```

- Terraform automatically infers dependency from the reference
- The referenced resource is created first
- Use `depends_on` only when the dependency is not expressed by an attribute reference

> ⚠️ **Exam Trap — Cross-resource connection strings:**
> To pass a database connection string to a web app, reference the database resource's output attribute directly. Terraform resolves it after the DB is created. No manual ordering needed.

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
> `for_each` requires a **map or set**, NOT a list.
> ```hcl
> variable "namespaces" {
>   type    = list(string)
>   default = ["platform", "apps", "observability"]
> }
>
> resource "kubernetes_namespace" "ns" {
>   for_each = toset(var.namespaces)   # ← must convert list to set
>   metadata { name = each.key }
> }
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
[for s in var.names : upper(s)]                    # list
{for k, v in var.map : k => upper(v)}              # map
[for s in var.names : upper(s) if s != ""]         # with filter
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
aws_instance.web[*].id   # all IDs from count-based resource
```

---

### Topic 21: Use meta-arguments in resource blocks

#### `depends_on`
```hcl
resource "aws_instance" "app" {
  depends_on = [aws_db_instance.main]
}
```
Use only when dependency isn't inferred from attribute references.

#### `count`
```hcl
resource "aws_instance" "web" {
  count         = 3
  instance_type = "t2.micro"
  tags = { Name = "web-${count.index}" }
}
# Reference: aws_instance.web[0].id
# All IDs:   aws_instance.web[*].id
```

#### `for_each`
```hcl
resource "aws_s3_bucket" "b" {
  for_each = toset(["logs", "backups"])
  bucket   = each.key
}
# Reference: aws_s3_bucket.b["logs"].id
```

**`count` vs `for_each`:**
- `count` → indexed; removing middle element causes downstream destroy/recreate
- `for_each` → keyed by identity; removing one key only destroys that one ✅ preferred

#### `lifecycle`
```hcl
lifecycle {
  create_before_destroy = true
  prevent_destroy       = true
  ignore_changes        = [tags, ami]
  replace_triggered_by  = [aws_ami.new]
}
```

> ⚠️ **Exam Trap — `ignore_changes` for external tag bots:**
> If a tag management bot updates tags after deployment and you want Terraform to ignore those future changes:
> ```hcl
> lifecycle {
>   ignore_changes = [tags]
> }
> ```
> This applies tags at creation time but ignores any subsequent external changes.

#### `provider`
```hcl
resource "aws_instance" "west" {
  provider = aws.west
}
```

---

### Topic 22: Use sensitive data and manage secrets in state

> ⚠️ **Critical Exam Trap — `sensitive = true` does NOT protect state:**
> `sensitive = true` on a variable or output ONLY suppresses display in CLI output.
> The value is **still stored in plaintext in `terraform.tfstate`**.

**Methods to keep sensitive data OUT of state:**

| Method | How |
|--------|-----|
| **Ephemeral resources** (TF 1.10+) | Use `ephemeral` block — value is never written to state |
| **External secret manager** | Read secrets via `data` source at runtime (value still in state) |
| **Encrypt state backend** | S3+KMS, Azure Blob encryption, etc. |
| **`nonsensitive()` carefully** | Only unwraps for display — doesn't remove from state |

**Ephemeral resources (exam-relevant new feature):**
```hcl
ephemeral "aws_secretsmanager_secret_version" "api_key" {
  secret_id = "my-api-key"
}
# Value is available in config but NEVER written to state
```

> ⚠️ **Exam Trap — One-time API key not in state:**
> If a resource requires a randomly generated key that must NOT be stored in state, use an `ephemeral` resource or ephemeral values. This is the correct answer when the question says "explicitly forbidden from being written to state."

---

### Topic 23: Use data sources with lifecycle validation

```hcl
data "aws_resourcegroups_group" "rg" {
  name = "my-resource-group"

  lifecycle {
    postcondition {
      condition     = self.arn != ""
      error_message = "Resource group not found or returned empty."
    }
  }
}
```

> ⚠️ **Exam Trap — Validating data source results:**
> To ensure a data source returns a valid result (e.g., verify an Azure resource group exists before proceeding), use a `lifecycle` block with a `postcondition` on the data source. This is different from variable `validation` blocks (which only check input values).

---

## SECTION 5 — Terraform Modules

### Topic 24: Contrast module source options

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"  # Public Terraform Registry
  version = "~> 5.0"
}

module "vpc" {
  source = "./modules/vpc"                   # Local path
}

module "vpc" {
  source = "git::https://github.com/org/repo.git?ref=v1.2.0"  # Git + tag
}

module "vpc" {
  source = "github.com/org/repo//modules/vpc"  # GitHub shorthand
}
```

> ⚠️ **Exam Trap — Module source formats:**
> Public Terraform Registry format: `"<namespace>/<module>/<provider>"` — no URL, no `https://`.
> Git repo at a specific tag: `"git::https://example.com/repo.git?ref=v1.0"` — uses `?ref=` for the tag.
> Both work **without additional setup** (no credentials needed for public registry or public Git repos).

---

### Topic 25: Interact with module inputs and outputs

```hcl
module "network" {
  source      = "./modules/network"
  vpc_cidr    = "10.0.0.0/16"       # input variable
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
- A parent module **cannot access** a child module's internal resources directly — only its `output` values
- Variables in child module are completely separate from parent's variables (no automatic inheritance except via explicit passing)

---

### Topic 27: Use a module from the public Terraform Registry

```hcl
terraform {
  required_providers {
    aws = { source = "hashicorp/aws" }
  }
}

module "s3_bucket" {
  source  = "terraform-aws-modules/s3-bucket/aws"
  version = "4.1.0"
  bucket  = "my-bucket"
}
```

- Registry URL: `registry.terraform.io`
- Format: `<NAMESPACE>/<MODULE>/<PROVIDER>`
- `version` argument is **optional** but strongly recommended
- Downloaded to `.terraform/modules/` during `terraform init`

---

## SECTION 6 — Terraform State Management

### Topic 28: Describe the default local backend

- Default backend stores state in `terraform.tfstate` in current working directory
- `terraform.tfstate.backup` keeps the previous state
- **Not suitable for teams** — no locking, no shared access
- Never commit `terraform.tfstate` to Git (contains sensitive data)

---

### Topic 29: Describe state locking

- Prevents concurrent modifications to state
- Supported by most remote backends (S3+DynamoDB, HCP Terraform, Azure Blob, GCS)
- Local backend has OS-level file locking

> ⚠️ **Exam Trap — Concurrent `terraform apply`:**
> If two engineers run `terraform apply` simultaneously against the same remote backend workspace, **state locking prevents this**. The second apply gets a **lock acquisition error** and is blocked until the first completes. This is the correct and expected behavior — not data corruption, not silent failure.

---

### Topic 30: Handle backend authentication methods

- Backend credentials configured in `backend` block or via environment variables
- **Never hardcode credentials in backend config** — use environment variables or IAM roles
- HCP Terraform: authenticate via `TF_TOKEN_app_terraform_io` or `terraform login`

```hcl
terraform {
  backend "s3" {
    bucket         = "my-tf-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
    # Credentials via AWS env vars or IAM role — NOT hardcoded here
  }
}
```

---

### Topic 31: Describe remote state storage benefits

| Benefit | Detail |
|---------|--------|
| **Shared access** | Team can all work against same state |
| **Locking** | Prevents concurrent apply conflicts |
| **Encryption** | State encrypted at rest |
| **Versioning** | S3 versioning = state history |
| **Remote operations** | HCP Terraform runs plans/applies remotely |

**Remote state data source** — read outputs from another Terraform workspace:
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

## SECTION 7 — Maintain Infrastructure with Terraform

### Topic 32: Manage resource drift

- **Drift** = real-world infra differs from Terraform state
- Detect with `terraform plan` — shows unexpected changes
- `terraform refresh` (deprecated) or `terraform apply -refresh-only` — updates state to match reality without changing infra
- `ignore_changes` in `lifecycle` — tell Terraform to accept certain external changes

---

### Topic 33: Manage Terraform state with CLI

| Command | Purpose |
|---------|---------|
| `terraform state list` | List all resources in state |
| `terraform state show <addr>` | Show details of one resource |
| `terraform state mv <src> <dst>` | Rename/move resource in state |
| `terraform state rm <addr>` | Remove resource from state (doesn't destroy real resource) |
| `terraform state pull` | Download remote state |
| `terraform state push` | Upload local state to remote |

---

### Topic 34: Use `terraform import`

```bash
terraform import aws_instance.web i-1234567890abcdef0
```

- Imports existing real-world resource into Terraform state
- **Does NOT generate configuration** — you must write the `resource` block first
- After import, run `terraform plan` to verify config matches actual state

> ⚠️ **Exam Trap:** `terraform import` only updates **state**, not config. If you import without a matching resource block, next `plan` will queue a deletion.
> TF 1.5+ `import` block with `-generate-config-out` can generate config, but this is a separate feature.

---

## SECTION 8 — HCP Terraform

### Topic 35: Describe HCP Terraform capabilities

- Cloud-based Terraform execution platform (formerly Terraform Cloud)
- Provides: remote state, remote execution, team collaboration, policy as code (Sentinel), private module registry
- Free tier available for small teams

---

### Topic 36: Describe workspaces in HCP Terraform

- Each workspace = one Terraform working directory + state + variables + run history
- Workspaces are isolated — separate state, separate variable sets
- Different from CLI workspaces (`terraform workspace`) — HCP workspaces are more full-featured

> **CLI workspaces vs HCP workspaces:**
> CLI workspaces: multiple state files in same config directory (lightweight env separation)
> HCP workspaces: full separation with their own state, variables, access controls, run history

---

### Topic 37: Describe managed resources in HCP Terraform

**Variable sets** — reusable variables applied across multiple workspaces (e.g., AWS credentials)

**Run triggers** — automatically trigger a downstream workspace run when an upstream workspace applies successfully

**Remote execution modes:**
- **Remote** — plan and apply run on HCP Terraform infrastructure
- **Local** — plan and apply run locally, state stored remotely
- **Agent** — runs on self-hosted Terraform agents (for private networks)

**Sentinel policies** — policy-as-code framework that enforces governance rules before apply

**Private module registry** — host internal modules, versioned and accessible across the organization

---

## ❌ PRACTICE TEST WEAK AREAS — Targeted Revision

*These are the specific topics where errors occurred in practice testing. Study these carefully.*

---

### 🔴 STATE MANAGEMENT (3 wrong out of 8)

**Q7 — Local backend, tfstate deleted, run `terraform plan`:**
Terraform treats the directory as having no state. It plans to **CREATE all resources** as if they're new. No error is thrown — Terraform simply doesn't know the resources exist. This will cause duplicate resource creation if applied. Fix: use `terraform import` to restore state.

**Q10 — Remote backend, tfstate deleted, run `terraform plan`:**
Same behavior as local. Terraform plans to CREATE everything. The remote backend just stores state — if the file is gone, Terraform has no memory of existing infrastructure. The plan looks identical to a fresh deployment.

**Q49 — Two engineers run `terraform apply` simultaneously (remote backend):**
The remote backend **locks state** during apply. The second engineer's apply will **fail with a lock error** — it cannot proceed until the first apply completes and releases the lock. This is the intended behavior. The infrastructure is NOT corrupted.

---

### 🔴 CORE WORKFLOW (1 wrong out of 11)

**Q47 — `terraform init -upgrade`:**
Upgrades **already-installed** provider plugins and modules to the latest version that still satisfies version constraints. It also updates `.terraform.lock.hcl` with new hashes. It does NOT ignore constraints — it works within them. Use when a newer patch/minor version of a provider is available and you want to pull it in.

---

### 🔴 MODULES (2 wrong out of 5)

**Q30 — Version constraint `~> 3.2.0`:**
The pessimistic constraint `~>` only allows the **rightmost version component** to increment.
- `~> 3.2.0` → allows `>= 3.2.0, < 3.3.0` → **3.2.5 is downloaded** (not 3.3.0 or 4.0.0)
- `~> 3.2` (no patch digit) → allows `>= 3.2, < 4.0` → would allow 3.3.0

**Q46 — Module source without additional setup:**
Two sources that work without extra credentials or configuration:
1. **Public Terraform Registry:** `source = "hashicorp/consul/aws"` — just namespace/module/provider, no URL
2. **Git repo at a specific tag:** `source = "git::https://github.com/org/repo.git?ref=v1.2.0"` — uses `?ref=` for tag pinning

Both work out of the box. A private registry or SSH Git repo would need additional setup.

---

### 🔴 TERRAFORM CONFIGURATION (7 wrong out of 13) ← PRIORITY FOCUS

**Q11 — Randomly generated API key must NOT be written to state:**
Use an **ephemeral resource** (introduced in Terraform 1.10). Ephemeral resources are evaluated at runtime but their values are **never persisted to state**. This is the correct answer when the requirement is that a generated secret is explicitly forbidden from state.
```hcl
ephemeral "random_password" "api_key" {
  length = 32
}
```

**Q13 — Reference `id` of a data source:**
Data source reference syntax: `data.<type>.<name>.<attribute>`
```hcl
data "aws_vpc" "production" {
  tags = { Name = "prod" }
}
# Reference the id:
subnet_vpc_id = data.aws_vpc.production.id
```
Never forget the `data.` prefix for data sources.

**Q20 — Validate data source returns a result:**
Use a `postcondition` in the data source's `lifecycle` block:
```hcl
data "azurerm_resource_group" "rg" {
  name = var.resource_group_name

  lifecycle {
    postcondition {
      condition     = self.id != null && self.id != ""
      error_message = "Resource group '${var.resource_group_name}' does not exist."
    }
  }
}
```
This is distinct from variable `validation` blocks — postconditions run after the data is fetched.

**Q28 — Sensitive information NOT stored in state:**
`sensitive = true` does NOT prevent state storage. Options that actually keep data out of state:
- **Ephemeral resources** — never written to state
- **Encrypt the backend** — state is encrypted at rest (data still there, but protected)
- **Use external secret manager at runtime** — e.g., read from Vault only when needed
The only way to truly prevent a value from appearing in state is to use ephemeral resources.

**Q36 — Pass connection string from DB to web app:**
Use a direct attribute reference — Terraform resolves it after the DB resource is created:
```hcl
resource "azurerm_app_service" "web" {
  app_settings = {
    DB_CONNECTION = azurerm_sql_database.main.connection_string
  }
}
```
This creates an implicit `depends_on`. No explicit `depends_on` needed because the reference is direct.

**Q51 — Tag management bot updates tags externally; ignore future tag changes:**
```hcl
resource "aws_instance" "fleet" {
  tags = {
    ship  = "millennium-falcon"
    pilot = "han"
  }

  lifecycle {
    ignore_changes = [tags]
  }
}
```
Tags are applied at create time. After that, any external changes to tags (by the bot) are ignored by Terraform. If you used `ignore_changes = all`, ALL attribute changes would be ignored — too broad. `[tags]` is the precise answer.

**Q53 — `for_each` with `list(string)` variable:**
`for_each` requires a **map or set**, not a list. Convert with `toset()`:
```hcl
variable "namespaces" {
  type    = list(string)
  default = ["platform", "apps", "observability"]
}

resource "kubernetes_namespace" "ns" {
  for_each = toset(var.namespaces)   # ← required conversion
  metadata {
    name = each.key
  }
}
```
Without `toset()`, Terraform throws: `The given "for_each" argument value is unsuitable: the "for_each" map includes keys derived from resource attributes that cannot be determined until apply.`

---

## ⚡ Master Cheat Sheet — All Key Exam Traps

| Topic | Trap | Correct Answer |
|-------|------|----------------|
| `sensitive = true` | Protects data in state? | ❌ Only hides in CLI output. Still in state. |
| Deleted local tfstate | `terraform plan` errors? | ❌ Plans to CREATE everything |
| Deleted remote tfstate | `terraform plan` errors? | ❌ Same — plans to create everything |
| Concurrent apply (remote) | Both succeed? | ❌ Second gets lock error |
| `terraform init -upgrade` | Does what? | Upgrades providers within version constraints |
| `~> 3.2.0` | Allows 3.3.0? | ❌ Only `>= 3.2.0, < 3.3.0` → max is 3.2.x |
| `for_each` with list | Works directly? | ❌ Must use `toset()` first |
| Data source reference | Syntax? | `data.<type>.<name>.<attr>` |
| Validate data source result | Use what? | `lifecycle { postcondition {} }` |
| Key must NOT be in state | Use what? | Ephemeral resource |
| External tag bot, ignore changes | lifecycle rule? | `ignore_changes = [tags]` |
| `terraform import` | Generates config? | ❌ State only. Write config manually. |
| `terraform.tfvars` | Auto-loaded? | ✅ Yes. Other .tfvars files need `-var-file`. |
| Custom functions | Can you define them? | ❌ Built-in only |
| `prevent_destroy` | Stops `terraform destroy`? | ❌ Only stops plans that include that resource |
| Module source (public registry) | Format? | `"namespace/module/provider"` no URL |
| Module source (git + tag) | Format? | `"git::https://...git?ref=v1.0"` |

---

### 🔴 TERRAFORM FUNDAMENTALS (2 wrong out of 7)

**Q19 — No state file exists, `terraform init` already run, run `terraform apply`:**
Terraform state is created **during `terraform apply`**, not during `terraform init`. If no state file exists:
- `terraform init` only downloads providers/modules — it does NOT create state
- When `terraform apply` runs with no existing state, Terraform treats all resources as **new** and creates them from scratch
- A fresh `terraform.tfstate` is created after the apply completes
- This is normal first-run behavior — no error, just creates everything

> Key distinction: `init` ≠ state creation. State is only created/updated on `apply`.

**Q50 — Provider at `6.0.2`, constraint `~> 6.0.0`, newer versions `6.0.5` and `6.1.0`, run `terraform init -upgrade`:**
- `~> 6.0.0` → pessimistic constraint → allows `>= 6.0.0, < 6.1.0` (only patch can increment)
- Available: `6.0.5` (within range ✅) and `6.1.0` (outside range ❌)
- `terraform init -upgrade` upgrades to **`6.0.5`** — the highest version within the constraint
- `6.1.0` is excluded because minor version increment is not allowed by `~> 6.0.0`
- `.terraform.lock.hcl` is updated to reflect the new version `6.0.5`

---

### 🔴 HCP TERRAFORM (1 wrong out of 5)

**Q45 — `prod-dns` workspace needs public IP from `prod-webserver` workspace automatically:**
Use a **`terraform_remote_state` data source** in the `prod-dns` workspace:
```hcl
data "terraform_remote_state" "webserver" {
  backend = "remote"
  config = {
    organization = "my-org"
    workspaces = {
      name = "prod-webserver"
    }
  }
}

resource "azurerm_dns_a_record" "web" {
  records = [data.terraform_remote_state.webserver.outputs.public_ip]
}
```
- `prod-webserver` must have an `output` block exposing the IP
- Every time `prod-webserver` applies and the IP changes, `prod-dns` automatically gets the updated value on its next plan/apply
- No manual variable updates needed — this is the correct pattern for cross-workspace data sharing in HCP Terraform

---

### 🔴 INFRASTRUCTURE AS CODE — IaC vs CONFIG MANAGEMENT (2 wrong out of 4)

**Q38 — Single Terraform configuration: which statements can be true? (select three)**

A single Terraform configuration CAN:
1. **Deploy resources across multiple cloud providers** — you can have AWS, Azure, and GCP resources in one config using multiple provider blocks
2. **Be used as a reusable module** — any directory with `.tf` files can be called as a module by other configs
3. **Manage multiple environments using workspaces** — CLI workspaces allow the same config to manage dev/staging/prod with separate state

A single Terraform configuration CANNOT:
- Automatically create separate state per environment without workspaces or separate directories
- Run applies in parallel for different providers (sequential by dependency graph)

**Q56 — How do IaC tools (Terraform) differ from configuration management tools (Chef, Ansible, Puppet)?**

| Dimension | IaC Tools (Terraform) | Config Management (Chef/Ansible/Puppet) |
|-----------|----------------------|----------------------------------------|
| **Primary purpose** | Provision and manage infrastructure resources | Configure software on existing servers |
| **What it manages** | VMs, networks, databases, DNS, cloud resources | Packages, files, services, users on a server |
| **State model** | Declarative + tracks state of infra | Declarative or procedural, no infra state |
| **Typical use** | Create the VM, the VPC, the load balancer | Install Nginx on the VM, configure `/etc/nginx.conf` |
| **Immutability** | Terraform favors replacing resources (immutable) | Config management mutates existing servers (mutable) |

> **The key exam answer:** Terraform is an **infrastructure provisioning** tool — it creates and destroys infrastructure. Chef/Ansible/Puppet are **configuration management** tools — they configure what runs ON that infrastructure. They are **complementary**, not competing. Terraform provisions the server; Chef configures it.

---

## 📊 Updated Practice Test Score Summary

| Section | Total Qs | Correct | Incorrect | Score |
|---------|----------|---------|-----------|-------|
| Section 1: IaC with Terraform | 4 | 2 | 2 | 50% ⚠️ |
| Section 2: Terraform Fundamentals | 7 | 5 | 2 | 71% ⚠️ |
| Section 3: Core Workflow | 11 | 10 | 1 | 91% ✅ |
| Section 4: Terraform Configuration | 13 | 6 | 7 | 46% 🔴 |
| Section 5: Terraform Modules | 5 | 3 | 2 | 60% ⚠️ |
| Section 6: State Management | 8 | 5 | 3 | 63% ⚠️ |
| Section 7: Maintain Infrastructure | — | — | — | — |
| Section 8: HCP Terraform | 5 | 4 | 1 | 80% ✅ |

**Priority study order based on scores:**
1. 🔴 Section 4 (Configuration) — 46% — ephemeral, data refs, lifecycle, for_each
2. ⚠️ Section 1 (IaC concepts) — 50% — IaC vs config management, single config capabilities
3. ⚠️ Section 5 (Modules) — 60% — version constraints, source formats
4. ⚠️ Section 6 (State) — 63% — deleted state behavior, locking
5. ⚠️ Section 2 (Fundamentals) — 71% — no-state apply, init -upgrade

---

*Last updated: June 2026 | Based on official HashiCorp Terraform Associate 004 exam objectives*
