# Terraform Associate 004 — All 37 Topics Revision Guide

> **Exam:** HashiCorp Certified: Terraform Associate (004)
> **Topics:** 37 across 8 sections | **Duration:** 60 min | **Cost:** $70.50 | **Validity:** 2 years

---

## Table of Contents

- [Section Overview](#overview)
- [Section 1 — IaC with Terraform](#section-1)
- [Section 2 — Terraform Fundamentals](#section-2)
- [Section 3 — Core Terraform Workflow](#section-3)
- [Section 4 — Terraform Configuration](#section-4)
- [Section 5 — Terraform Modules](#section-5)
- [Section 6 — Terraform State Management](#section-6)
- [Section 7 — Maintain Infrastructure](#section-7)
- [Section 8 — HCP Terraform](#section-8)
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

> 💡 **Conceptual twist:** Idempotency is the most important property. If you run `terraform apply` on an already-applied config, Terraform detects no changes and does nothing. It doesn't re-create resources — it compares desired state (config) against current state (state file) and only acts on the diff.

---

### Topic 2: Describe the advantages of IaC patterns

| Advantage | Detail |
|-----------|--------|
| Consistency | Same config = same infra, no config drift |
| Speed | Automate provisioning at scale |
| Auditability | Git history = full change log |
| Collaboration | Code reviews for infra changes |
| Disaster recovery | Rebuild entire infra from code |

> 💡 **Thing to keep in mind:** "Config drift" is when someone manually changes a resource outside Terraform (via console, CLI, etc.) and the real world no longer matches what Terraform thinks it manages. IaC solves this by making the config the single source of truth. Terraform detects drift on the next `plan`.

---

### Topic 3: Explain how Terraform manages multi-cloud, hybrid cloud, and service-agnostic workflows

- Terraform uses **providers** to interface with any API — AWS, Azure, GCP, Kubernetes, GitHub, Datadog, etc.
- A single Terraform config can provision resources across **multiple clouds simultaneously**
- The core workflow (`init → plan → apply`) is identical regardless of provider
- This makes Terraform **cloud-agnostic** — no vendor lock-in at the tooling level

> 💡 **Conceptual twist:** Terraform itself has no knowledge of AWS, Azure, or any cloud. All cloud-specific logic lives in the **provider plugin**. Terraform's job is to build a dependency graph and call provider plugins in the right order. This is why the same workflow works for every provider.

> ⚠️ **Exam Q38 — Single config: which statements can be true? (select three)**
> 1. Can deploy resources across **multiple cloud providers** in one config using multiple provider blocks
> 2. Can be used as a **reusable module** by other configurations
> 3. Can manage **multiple environments** using CLI workspaces with separate state

> ⚠️ **Exam Q56 — How does Terraform differ from Chef/Ansible/Puppet?**
>
> | | Terraform (IaC) | Chef / Ansible / Puppet |
> |--|----------------|------------------------|
> | Purpose | Provision infrastructure | Configure software on servers |
> | Manages | VMs, networks, DNS, cloud resources | Packages, files, services on a server |
> | Approach | Immutable — replace resources | Mutable — configure existing servers |
> | State | Tracks infra state | No infra state |
>
> Key answer: Terraform **provisions** the server. Chef **configures** what runs on it. They are **complementary**, not competing. Terraform provisions the VM; Chef installs Nginx on it.

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
| `~>` | Pessimistic — allows only rightmost component to increment | `~> 5.0` allows 5.x, not 6.x |

> 💡 **How `~>` really works — the mental model:**
> Count the digits. `~> 5.0` has two digits — the last one (minor) can grow, major cannot. So `5.x` is fine, `6.0` is not.
> `~> 5.0.0` has three digits — only the last one (patch) can grow. So `5.0.x` is fine, `5.1.0` is not.
> Think of it as: "everything to the left is frozen, only the rightmost number can count up."

> ⚠️ **Exam Q30:** `~> 3.2.0` available: 3.2.0, 3.2.5, 3.3.0, 4.0.0 → downloads **3.2.5**
> ⚠️ **Exam Q50:** `~> 6.0.0` with `init -upgrade`, available 6.0.5 and 6.1.0 → upgrades to **6.0.5** (6.1.0 blocked)

---

### Topic 5: Describe how Terraform uses providers

- Providers are **plugins** downloaded during `terraform init` into `.terraform/providers/`
- They translate HCL resource declarations into actual API calls to cloud platforms
- Provider config goes in a `provider` block:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

- `.terraform.lock.hcl` locks exact provider versions and checksums — **commit this to Git**

> 💡 **Thing to keep in mind:** The lock file exists so every team member and CI pipeline uses the exact same provider version. Without it, `terraform init` could silently pull a different provider version on a different machine. Never gitignore `.terraform.lock.hcl` — but always gitignore the `.terraform/` directory itself (it's large and regenerated by init).

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
  # uses default provider — no provider argument needed
}

resource "aws_instance" "west" {
  provider = aws.west   # references the aliased provider
}
```

> 💡 **Conceptual twist:** You can have multiple configurations of the same provider using `alias`. Without an alias, there can only be one default config per provider. The `provider` meta-argument on a resource uses dot notation: `provider = <provider_name>.<alias>`. This is how you deploy to multiple regions in one config.

---

### Topic 7: Explain how Terraform uses and manages state

- State is stored in `terraform.tfstate` (plain JSON)
- Maps your config resources to real-world resource IDs (e.g. `aws_instance.web` → `i-0abc123`)
- Used to: compute diffs, track metadata, improve plan performance (cache API responses)
- **Never edit state manually** — use `terraform state` subcommands

> 💡 **Why state exists — the core reason:** Terraform needs to know which real resource corresponds to which config block. Without state, `terraform plan` would have to query every provider API for every resource every time. State acts as a local cache of reality.

> 💡 **The dangerous implication:** State is the ground truth Terraform operates from — NOT the actual cloud. If state and reality diverge, Terraform acts on what state says, not what actually exists. This is why deleted/corrupted state is so dangerous.

> ⚠️ **Exam Trap — State is created on `apply`, NOT on `init`:**
> `terraform init` downloads providers and sets up backends. It does NOT create state.
> State only gets created/updated when `terraform apply` runs successfully.
>
> **Exam Q19:** No state file, `terraform init` already run, run `terraform apply`:
> Terraform applies normally, creates all resources fresh, and writes a new `terraform.tfstate`. No error.

> ⚠️ **Exam Q7 (local) and Q10 (remote) — Deleted state file, run `terraform plan`:**
> Terraform sees no existing state → plans to **CREATE all resources** as if starting fresh.
> It does not error. It does not detect the existing infra. It just thinks nothing exists.
> **Fix:** Use `terraform import` to bring existing resources back into state.

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

> 💡 **Thing to keep in mind:** `plan` is always a dry run — it never touches real infrastructure. `apply` shows you the plan again and asks for confirmation before acting (unless `-auto-approve` is passed). The workflow is designed so you always see what will happen before it happens.

---

### Topic 9: Initialize a Terraform working directory

```bash
terraform init
```

**What `init` does:**
- Downloads provider plugins into `.terraform/`
- Initializes and configures the backend
- Downloads any referenced modules into `.terraform/modules/`
- Creates or updates `.terraform.lock.hcl`

**Key flags:**

| Flag | Purpose |
|------|---------|
| `-upgrade` | Upgrade providers to newest allowed version; update lock file |
| `-reconfigure` | Reconfigure the backend, ignoring existing state |
| `-migrate-state` | Migrate existing state to a new backend |
| `-backend=false` | Skip backend initialization entirely |

> 💡 **Conceptual twist — `init` is idempotent and safe to re-run:** Running `terraform init` again doesn't harm anything. It's required whenever you add a new provider, change backend config, or add a module. Think of it as "sync your dependencies" — like `npm install` or `pip install`.

> ⚠️ **Exam Q47 — `terraform init -upgrade`:**
> `-upgrade` upgrades already-installed providers to the **latest version that still satisfies version constraints**. It does NOT ignore constraints. It updates `.terraform.lock.hcl` with the new version and checksums. Use it when you want to pull in a newer patch or minor version without manually editing the lock file.

---

### Topic 10: Validate a Terraform configuration

```bash
terraform validate
```

- Checks configuration syntax and internal consistency (e.g. correct argument names, valid references)
- Does NOT access remote state, provider APIs, or check if resources exist
- Requires `init` to have been run — needs provider schemas to validate against
- Returns zero exit code if valid, non-zero if errors found — useful in CI pipelines

> 💡 **Thing to keep in mind:** `validate` catches mistakes like misspelled argument names or referencing a variable that doesn't exist. It does NOT catch things like "this AMI ID doesn't exist in AWS" — that only surfaces during `plan` or `apply` when the provider actually calls the cloud API.

---

### Topic 11: Generate and review an execution plan

```bash
terraform plan
terraform plan -out=tfplan       # save plan to file
terraform plan -destroy          # preview destroy
terraform plan -var="key=value"  # pass variable inline
```

- Shows: resources to add (+), change (~), destroy (-)
- Does NOT make any changes to real infrastructure
- Saved plan file guarantees what `apply` will do — no re-evaluation

> 💡 **Why save a plan file?** In automated pipelines (CI/CD), you `plan` in one step, get approval, then `apply tfplan` in another step. Without a saved plan, a second `apply` re-evaluates the config, which could differ if anything changed in between. Saved plans = deterministic applies.

> 💡 **Twist on `~` (in-place change) vs `-`/`+` (destroy/recreate):** Some attribute changes force a destroy and recreate (e.g. changing an EC2 `ami`). Terraform shows this as `-/+`. Only immutable attributes force this. Mutable attributes show `~` (in-place update). Knowing which attributes are immutable is where `lifecycle { create_before_destroy }` becomes important.

---

### Topic 12: Apply changes to infrastructure

```bash
terraform apply
terraform apply tfplan           # apply a saved plan exactly
terraform apply -auto-approve    # skip the yes/no confirmation
terraform apply -replace=<addr>  # force-replace a specific resource
```

> 💡 **Conceptual twist — apply refreshes state first:** Before computing the diff, `apply` (and `plan`) refreshes the state by querying the provider APIs for current resource attributes. This is how it detects drift. You can skip this with `-refresh=false` for speed, but then drift won't be detected.

---

### Topic 13: Destroy Terraform-managed infrastructure

```bash
terraform destroy
terraform destroy -target=aws_instance.web  # destroy one specific resource
```

- Equivalent to `terraform apply -destroy`
- Reads state to determine what to destroy — only touches resources Terraform manages
- Destruction order is the reverse of creation order (respects the dependency graph)

> 💡 **Thing to keep in mind:** `destroy` only destroys what is in state. Resources that exist in the cloud but were never imported into state will NOT be touched. This is a common source of confusion — Terraform manages what it knows about, not everything in your account.

---

### Topic 14: Apply formatting and style adjustments

```bash
terraform fmt               # format current directory
terraform fmt -recursive    # format all subdirectories
terraform fmt -check        # check only, exit non-zero if needed (CI use)
terraform fmt -diff         # show what would change
```

- Enforces canonical HCL style: 2-space indentation, aligned `=` signs in blocks
- Safe to run at any time — only changes whitespace/formatting, never logic

---

<a id="section-4"></a>
## Section 4 — Terraform Configuration

### Topic 15: Use and differentiate resource and data blocks

**`resource` block** — declares infrastructure Terraform will CREATE and MANAGE:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}
```
Reference anywhere else in config: `aws_instance.web.id`

**`data` block** — READ-ONLY query of existing infrastructure Terraform does NOT manage:
```hcl
data "aws_vpc" "production" {
  tags = { Name = "prod" }
}
```
Reference: `data.aws_vpc.production.id`

> 💡 **The core conceptual difference:** A `resource` block tells Terraform "make this exist." A `data` block tells Terraform "go find this thing that already exists and let me use its attributes." Data sources don't create, modify, or destroy anything — they just read. If the thing doesn't exist, the data source fails at plan/apply time.

> ⚠️ **Exam Q13 — Data Block Reference Syntax:**
> Always prefix with `data.` → `data.<type>.<name>.<attribute>`
> Resources do NOT use the `data.` prefix → `aws_instance.web.id`
> This distinction trips people up constantly.

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

> 💡 **How implicit dependency works:** When resource B references an attribute of resource A, Terraform automatically knows A must be created before B. You don't need `depends_on`. The dependency graph is built from these attribute references. This is the preferred way to express ordering.

> 💡 **When to use explicit `depends_on`:** Only when there's a hidden dependency that can't be expressed through attribute references. For example, if resource B relies on a side effect of resource A (like a file written to disk, or an IAM policy that needs to propagate) but doesn't directly reference any of A's attributes. Overusing `depends_on` slows down Terraform because it forces sequential execution unnecessarily.

> ⚠️ **Exam Q36 — Pass DB connection string to web app:**
> Reference the DB attribute directly: `azurerm_sql_database.main.connection_string`
> Terraform resolves it after the DB is created. No explicit `depends_on` needed — the reference creates the dependency automatically.

---

### Topic 17: Use variables and outputs

**Variables** — parameterize your config so callers can pass custom values:
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

> 💡 **Twist on precedence:** Higher in the list = easier to override. Environment variables (`TF_VAR_`) win over everything except `-var` flags. This matters in CI/CD — if you set a variable in a `.tfvars` file AND as an env var, the env var wins. If a variable has no `default` and no value is provided by any method, Terraform prompts interactively (which breaks CI pipelines).

> ⚠️ **Critical trap:** Only `terraform.tfvars` and `*.auto.tfvars` are loaded automatically. A file named `prod.tfvars` or `secrets.tfvars` is silently ignored unless you explicitly pass `-var-file="prod.tfvars"`. This is a frequent exam question.

**Outputs** — expose values from your config to the CLI, parent modules, or remote state:
```hcl
output "instance_ip" {
  description = "Private IP"
  value       = aws_instance.web.private_ip
  sensitive   = true
}
```

> 💡 **Thing to keep in mind about outputs:** Outputs serve two purposes. (1) Display info after apply (e.g. the IP of your server). (2) Expose values from a child module so the parent can use them. You cannot access a child module's internal resources — only its declared outputs. In HCP Terraform, outputs from one workspace can be consumed by another via `terraform_remote_state`.

**Locals** — intermediate values scoped to the current module, not exposed or overridable:
```hcl
locals {
  app_name = "${var.project}-${var.env}"
}
# Reference: local.app_name
```

> 💡 **Variables vs Locals:** Variables are inputs — they come from outside the module and can be overridden by the caller. Locals are internal computed values — they're defined inside the module and cannot be passed in or overridden. Use locals to avoid repeating the same expression multiple times.

---

### Topic 18: Understand and use complex types

| Type | Example | Notes |
|------|---------|-------|
| `string` | `"hello"` | UTF-8 text |
| `number` | `42`, `3.14` | Int or decimal |
| `bool` | `true`/`false` | |
| `list(type)` | `["a","b"]` | Ordered, allows duplicates, accessed by index |
| `set(type)` | `toset(["a","b"])` | Unordered, no duplicates, no index access |
| `map(type)` | `{key = "val"}` | String keys, all values same type |
| `object({})` | `{name=string, age=number}` | Named attrs, each can be a different type |
| `tuple([])` | `["a", 1, true]` | Fixed length, positional, mixed types |
| `any` | — | Disables type checking — Terraform infers |

> 💡 **list vs set mental model:** A list is like an array — ordered and indexed. A set is like a mathematical set — unordered, unique values only. You can't do `set[0]` because sets have no order. This is why `for_each` requires a set or map — it needs stable, unique keys to identify each resource instance.

> 💡 **map vs object mental model:** A `map` enforces the same type for all values (`map(string)` means every value is a string). An `object` lets each named attribute have its own type — more like a struct. Use `object` when different attributes have different types.

> ⚠️ **Exam Q53 — `for_each` with `list(string)` variable:**
> `for_each` requires a **map or set**, NOT a list. A list has numeric indexes, not stable string keys. Convert with `toset()`:
> ```hcl
> variable "namespaces" {
>   type    = list(string)
>   default = ["platform", "apps", "observability"]
> }
>
> resource "kubernetes_namespace" "ns" {
>   for_each = toset(var.namespaces)   # required conversion
>   metadata { name = each.key }
> }
> ```
> After `toset()`, `each.key` and `each.value` are the same string (the namespace name). Each resource instance is identified by name, not index — so removing "apps" only destroys that one namespace, not everything after it.

---

### Topic 19: Use built-in functions

Terraform functions are **built-in only** — you cannot define custom functions. This is a hard exam fact.

**String:** `upper()`, `lower()`, `trimspace()`, `format()`, `join(",", list)`, `split(",", str)`, `replace()`
**Collection:** `length()`, `flatten()`, `merge()`, `keys()`, `values()`, `contains()`, `lookup(map, key, default)`, `zipmap()`
**Type conversion:** `tostring()`, `tonumber()`, `tobool()`, `tolist()`, `toset()`, `tomap()`
**Filesystem:** `file(path)` reads file as string, `templatefile(path, vars)` renders a template
**Encoding:** `jsonencode()`, `jsondecode()`, `base64encode()`, `base64decode()`
**Numeric:** `min()`, `max()`, `abs()`, `ceil()`, `floor()`

> 💡 **`lookup()` vs direct map access:** `lookup(map, key, default)` is safe — returns the default if the key doesn't exist. Direct access `map["key"]` throws an error if the key is missing. Use `lookup` when you're not sure a key will be present.

> 💡 **`flatten()` use case:** When you have a list of lists (e.g. from a `for` expression that produces multiple items per iteration), `flatten()` collapses it into a single list. Very common when generating dynamic resources from nested data structures.

---

### Topic 20: Use expressions and dynamic blocks

**Conditional (ternary):**
```hcl
instance_type = var.env == "prod" ? "t3.large" : "t2.micro"
```

**For expression (list output):**
```hcl
[for s in var.names : upper(s)]
[for s in var.names : upper(s) if s != ""]   # with filter
```

**For expression (map output):**
```hcl
{for k, v in var.tags : k => upper(v)}
```

**Dynamic block — generates repeated nested blocks:**
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

**Splat expression:**
```hcl
aws_instance.web[*].id   # collect all IDs from a count-based resource
```

> 💡 **Dynamic blocks — when to use them:** Some resources have nested blocks that need to repeat (like `ingress` rules in a security group). You can't use `for_each` on a nested block directly — that's what `dynamic` is for. The iterator name defaults to the block label (`ingress` in the example above) and gives you `.value` and `.key` to access the current item.

> 💡 **Splat vs for expression:** `resource[*].attr` is shorthand for `[for r in resource : r.attr]`. Splat is cleaner for simple cases. Use a full `for` expression when you need filtering or transformation.

---

### Topic 21: Use meta-arguments in resource blocks

#### depends_on
Explicit dependency when no attribute reference exists:
```hcl
resource "aws_instance" "app" {
  depends_on = [aws_iam_role_policy.this]
}
```

#### count
Create N identical resources:
```hcl
resource "aws_instance" "web" {
  count         = 3
  instance_type = "t2.micro"
  tags = { Name = "web-${count.index}" }   # index starts at 0
}
# Single instance:  aws_instance.web[0]
# All IDs (splat):  aws_instance.web[*].id
```

#### for_each
Create resources keyed by a map or set:
```hcl
resource "aws_s3_bucket" "b" {
  for_each = toset(["logs", "backups", "assets"])
  bucket   = each.key
}
# Reference: aws_s3_bucket.b["logs"].id
```

> 💡 **count vs for_each — the critical conceptual difference:**
> `count` identifies instances by **index** (0, 1, 2...). If you remove the middle item from your list, Terraform re-indexes everything after it and destroys/recreates those resources. This is destructive and dangerous in production.
> `for_each` identifies instances by **key** (the map key or set value). Removing one item only affects that one resource — others are untouched. Always prefer `for_each` for real infrastructure.

#### lifecycle
```hcl
lifecycle {
  create_before_destroy = true   # new resource before old is destroyed
  prevent_destroy       = true   # block any plan that would destroy this
  ignore_changes        = [tags, ami]   # ignore these attribute changes
  replace_triggered_by  = [aws_ami.new] # replace when this other resource changes
}
```

> 💡 **`create_before_destroy` — why it matters:** By default Terraform destroys old before creating new. For resources that can't have a gap in availability (load balancers, DNS records), this causes downtime. `create_before_destroy = true` reverses the order. But be careful — both resources exist simultaneously for a moment, which can cause naming conflicts if the resource has a unique name constraint.

> 💡 **`prevent_destroy` — what it does and doesn't do:** It only blocks a `terraform apply` that would destroy the resource. It does NOT stop `terraform destroy`. It also does NOT stop destroying through the cloud console or CLI. Think of it as a guard rail against accidental Terraform-driven deletion, not a security control.

> ⚠️ **Exam Q51 — Tag management bot updates tags externally:**
> Use `ignore_changes = [tags]` — Terraform applies the declared tags at creation, then permanently ignores any future external tag changes. Do NOT use `ignore_changes = all` — that ignores every attribute on the resource, meaning Terraform would never update it for any reason.

#### provider (resource-level)
```hcl
resource "aws_instance" "west" {
  provider = aws.west   # references an aliased provider block
}
```

---

### Topic 22: Use sensitive data and manage secrets in state

```hcl
variable "db_password" {
  type      = string
  sensitive = true   # hides value in plan/apply output
}

output "db_pass_out" {
  value     = var.db_password
  sensitive = true   # hides value in output display
}
```

> ⚠️ **Exam Q28 — Critical conceptual trap — `sensitive = true` does NOT protect state:**
> `sensitive = true` only suppresses the value in CLI terminal output (shows `(sensitive value)` instead).
> The actual value is **still stored in plaintext JSON in `terraform.tfstate`**.
> Anyone with access to the state file can read it. The flag is purely cosmetic for terminal display.

> 💡 **What actually protects sensitive data:**
> - **Ephemeral resources** (TF 1.10+) — value is NEVER written to state at all
> - **Encrypted remote backend** (S3+KMS, Azure Blob encryption) — state is stored encrypted at rest
> - **External secret manager** (Vault, AWS Secrets Manager) — retrieve secrets at runtime, but if read via a `data` source the value still lands in state

> ⚠️ **Exam Q11 — API key must NOT be written to state:**
> Use an **ephemeral resource**. Ephemeral values are computed at runtime and used in config, but Terraform never writes them to state:
> ```hcl
> ephemeral "random_password" "api_key" {
>   length = 32
> }
> ```
> When a question says a secret is "explicitly forbidden from being written to state," the answer is always ephemeral resource.

---

### Topic 23: Use data sources with lifecycle validation

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

> 💡 **postcondition vs variable validation — the key difference:**
> `variable validation` runs BEFORE Terraform does anything — it validates the value the user passed in.
> `postcondition` runs AFTER the data source fetches its result — it validates what was returned from the cloud API.
> Use validation to catch bad inputs. Use postcondition to catch bad infrastructure state (e.g. a resource group that should exist but doesn't).

> ⚠️ **Exam Q20 — Validate that a data source actually returns a result:**
> Add `lifecycle { postcondition {} }` to the data block. This is the correct pattern. It fires at plan/apply time after the data is fetched and fails with a clear error message if the condition isn't met.

---

<a id="section-5"></a>
## Section 5 — Terraform Modules

### Topic 24: Contrast module source options

```hcl
# Public Terraform Registry — no URL, just namespace/module/provider
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
}

# Local path — relative to current config directory
module "vpc" {
  source = "./modules/vpc"
}

# Git repo pinned to a specific tag
module "vpc" {
  source = "git::https://github.com/org/repo.git?ref=v1.2.0"
}

# GitHub shorthand (double slash separates repo from subdirectory)
module "vpc" {
  source = "github.com/org/repo//modules/vpc"
}
```

> 💡 **Why pin module versions?** Unpinned modules (`source = "terraform-aws-modules/vpc/aws"` with no version) will pull the latest version on every `terraform init`. A new major version could introduce breaking changes silently. Always pin with `version` for registry modules or `?ref=` for Git modules in production configs.

> 💡 **The `//` in Git sources:** The double slash is Terraform's way of pointing to a subdirectory within a Git repo. `github.com/org/repo//modules/vpc` means: repo is `org/repo`, subdirectory is `modules/vpc`. Single slash is just part of the URL.

> ⚠️ **Exam Q46 — Module source formats that work without additional setup:**
> - Public Terraform Registry: `"hashicorp/consul/aws"` — just `namespace/module/provider`, no URL prefix
> - Git repo at a tag: `"git::https://github.com/org/repo.git?ref=v1.2.0"` — `?ref=` pins the tag
> Private registries or SSH Git repos need credentials — those require additional setup.

---

### Topic 25: Interact with module inputs and outputs

```hcl
module "network" {
  source      = "./modules/network"
  vpc_cidr    = "10.0.0.0/16"      # passing input variable
  environment = var.env
}

# Consuming an output from the module:
resource "aws_instance" "app" {
  subnet_id = module.network.public_subnet_id
}
```

> 💡 **Module outputs are the only interface:** The calling module cannot reach inside a child module and reference its internal resources directly (e.g. `module.network.aws_subnet.public.id` is NOT valid). The child module must explicitly expose values via `output` blocks. This enforces clean module interfaces and encapsulation.

---

### Topic 26: Describe variable scope within modules

- Variables in a child module are completely separate from variables in the parent module
- No automatic inheritance — everything must be passed explicitly through the `module` block
- A child module's outputs are accessed as `module.<name>.<output_name>`
- Locals inside a module are completely private — not visible to the caller

> 💡 **Encapsulation is intentional:** Module variables and locals are scoped so modules stay self-contained. This is what makes them reusable — a module shouldn't depend on what the parent happens to have in its namespace. All dependencies must flow through explicit inputs and outputs.

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
- Downloaded to `.terraform/modules/` on `terraform init`
- `version` argument only works with registry modules — not local paths or Git sources

> ⚠️ **Exam Q30 — Version constraint `~> 3.2.0`:**
> Available: 3.2.0, 3.2.5, 3.3.0, 4.0.0 → Terraform downloads **3.2.5**
> `~> 3.2.0` allows `>= 3.2.0, < 3.3.0`. The patch digit is the only one that can grow.

---

<a id="section-6"></a>
## Section 6 — Terraform State Management

### Topic 28: Describe the default local backend

- Default backend stores state as `terraform.tfstate` in the working directory
- `terraform.tfstate.backup` is the automatically created backup of the previous state
- Not suitable for teams — no locking mechanism, no shared access, anyone could corrupt it
- Never commit `terraform.tfstate` to Git — it often contains plaintext secrets

> 💡 **Why local state is dangerous for teams:** Two engineers running `terraform apply` at the same time against local state means two independent state files. One's changes overwrite the other's. The second apply has no idea what the first one did. Remote backends with locking exist specifically to solve this.

---

### Topic 29: Describe state locking

- Locks state during `plan` and `apply` operations to prevent concurrent writes
- Supported by: S3+DynamoDB, HCP Terraform, Azure Blob Storage, GCS, Consul
- Local backend uses OS-level file locking (works for single user, not network shares)
- Lock can be force-released with `terraform force-unlock <lock-id>` — use with caution

> 💡 **How S3 state locking works:** S3 stores the state file but S3 itself has no locking mechanism. You pair it with a **DynamoDB table** that holds a lock record. When Terraform starts an operation, it writes a lock entry to DynamoDB. When done, it deletes it. If the process crashes, the lock stays — use `force-unlock` to clear it.

> ⚠️ **Exam Q49 — Two engineers run `terraform apply` simultaneously (remote backend):**
> The second engineer's apply **fails with a lock acquisition error** and cannot proceed.
> The first apply completes normally. Infrastructure is NOT corrupted.
> This is the correct, intended behavior — state locking working as designed.

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
    # Credentials come from env vars or IAM role — never hardcode here
  }
}
```

> 💡 **Backend config is static — no variables allowed:** You cannot use `var.`, `local.`, or any interpolation inside a `backend` block. Backend is initialized before variables are resolved. This is why credentials must come from environment variables or IAM roles, not from Terraform variables. If you need dynamic backend config, use `terraform init -backend-config="key=value"`.

---

### Topic 31: Describe remote state storage benefits

| Benefit | Detail |
|---------|--------|
| Shared access | Entire team works against the same state |
| Locking | Prevents concurrent apply conflicts |
| Encryption | State encrypted at rest |
| Versioning | S3 versioning gives you full state history |
| Remote operations | HCP Terraform runs plans/applies on managed infrastructure |

**Reading outputs from another workspace with `terraform_remote_state`:**
```hcl
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "my-tf-state"
    key    = "network/terraform.tfstate"
    region = "us-east-1"
  }
}
# Access outputs: data.terraform_remote_state.network.outputs.vpc_id
```

> 💡 **`terraform_remote_state` reads outputs only:** You cannot access internal resource attributes from another state — only explicitly declared `output` values. This enforces loose coupling between workspaces. The source workspace controls exactly what it exposes. If the output doesn't exist, the data source fails.

---

<a id="section-7"></a>
## Section 7 — Maintain Infrastructure with Terraform

### Topic 32: Manage resource drift

- **Drift** = real-world infra has changed outside Terraform (console, CLI, another tool, manual fix)
- `terraform plan` detects drift by comparing state to live infrastructure
- `terraform apply -refresh-only` — updates state to match reality **without changing infrastructure**
- `ignore_changes` in `lifecycle` — permanently tell Terraform to accept certain external changes

> 💡 **The drift detection flow:** On every `plan` and `apply`, Terraform calls the provider API to get the current attribute values of each managed resource. It compares those against what's in state. If they differ, that's drift — and Terraform will try to correct it back to what the config declares. This is the "desired state" model in action.

> 💡 **`-refresh-only` vs normal apply:** `-refresh-only` says "update my state to match reality, but don't push any changes back." This is useful when you made manual changes that you want to keep — you're telling Terraform "accept this as the new truth." After `-refresh-only`, the next plan will no longer show those changes as drift.

---

### Topic 33: Manage Terraform state with CLI

| Command | Purpose |
|---------|---------|
| `terraform state list` | List all resources tracked in state |
| `terraform state show <addr>` | Show all attributes of one resource in state |
| `terraform state mv <src> <dst>` | Rename or move a resource in state (e.g. after refactoring) |
| `terraform state rm <addr>` | Remove from state — does NOT destroy real resource |
| `terraform state pull` | Download and print the remote state |
| `terraform state push` | Upload a local state file to the remote backend |

> 💡 **`state mv` use case:** You renamed a resource block in your config (e.g. `aws_instance.web` → `aws_instance.webserver`). Without `state mv`, Terraform sees this as destroy old + create new. With `state mv`, you tell Terraform "this is the same resource, just renamed" — no destroy/recreate happens. Essential for safe refactoring.

> 💡 **`state rm` use case:** You want to stop managing a resource with Terraform (hand it off manually) without destroying it. `state rm` removes it from Terraform's knowledge. The real resource keeps running — Terraform just stops tracking it.

---

### Topic 34: Use `terraform import`

```bash
terraform import aws_instance.web i-1234567890abcdef0
```

- Brings an existing real-world resource under Terraform management by writing it to state
- **Does NOT generate configuration** — you must write the `resource` block first
- After import, always run `terraform plan` — if config doesn't match actual state, plan will show changes (or deletions)

> 💡 **The import workflow in practice:**
> 1. Write the `resource` block in your config
> 2. Run `terraform import` with the resource address and real-world ID
> 3. Run `terraform plan` — ideally it shows "No changes" meaning your config matches reality
> 4. If plan shows changes, update your config to match what was imported
>
> This is the manual workflow. TF 1.5+ introduced an `import` block + `-generate-config-out` flag that can draft the config for you, but you still need to review and clean it up.

---

<a id="section-8"></a>
## Section 8 — HCP Terraform

### Topic 35: Describe HCP Terraform capabilities

- Cloud-based Terraform execution platform (formerly Terraform Cloud)
- Provides: remote state storage, remote plan/apply execution, team collaboration, Sentinel policy enforcement, private module registry, audit logs
- Free tier available for small teams

> 💡 **Why remote execution matters:** When Terraform runs locally, it needs cloud credentials on the local machine. Remote execution means HCP Terraform runs the plan/apply on its own infrastructure — credentials are stored securely in the workspace, not on developer machines. This is important for compliance and audit trails.

---

### Topic 36: Describe workspaces in HCP Terraform

- Each workspace = isolated unit with its own state, variables, run history, and access controls
- You can have many workspaces per organization (one per environment, one per service, etc.)

**CLI workspaces vs HCP workspaces — a common confusion:**

| | CLI Workspaces (`terraform workspace`) | HCP Workspaces |
|--|---------------------------------------|----------------|
| What they are | Multiple state files within one config dir | Fully separate configs, state, vars, and access |
| Isolation level | State only | Complete — separate everything |
| Variable management | Shared config, different state | Each workspace has its own variable set |
| Team features | None | Access controls, run history, notifications |
| Use case | Simple env separation (dev/staging) | Full production multi-team workflows |

> 💡 **The conceptual trap:** CLI workspaces and HCP workspaces share the name "workspace" but are very different things. CLI workspaces are lightweight — same config, different state file. HCP workspaces are full isolation units. The exam may test whether you know this distinction.

---

### Topic 37: Describe managed resources in HCP Terraform

**Variable sets** — define a group of variables once, apply them to multiple workspaces (e.g. AWS credentials applied to all prod workspaces)

**Run triggers** — when workspace A applies successfully, automatically queue a run in workspace B. Used for dependency chains between infrastructure layers.

**Remote execution modes:**
- `remote` — plan and apply run on HCP Terraform's infrastructure
- `local` — plan and apply run on your machine, but state is stored remotely in HCP
- `agent` — runs on a self-hosted Terraform agent in your private network (for air-gapped environments)

**Sentinel** — policy-as-code framework that evaluates policies between `plan` and `apply`. If a policy fails, the apply is blocked. Used for governance (e.g. "no public S3 buckets", "all resources must have cost tags").

**Private module registry** — host your own internal modules with versioning, access controls, and documentation. Teams publish modules once; others consume them just like public registry modules.

> ⚠️ **Exam Q45 — `prod-dns` needs the public IP from `prod-webserver` automatically:**
> Use `terraform_remote_state` data source in `prod-dns`:
> ```hcl
> data "terraform_remote_state" "webserver" {
>   backend = "remote"
>   config = {
>     organization = "my-org"
>     workspaces = { name = "prod-webserver" }
>   }
> }
> # data.terraform_remote_state.webserver.outputs.public_ip
> ```
> `prod-webserver` must declare the IP as an `output`. Every time `prod-webserver` applies and the IP changes, `prod-dns` picks it up on its next plan/apply. No manual variable updates.
>
> 💡 **Why this pattern exists:** Different infrastructure layers (networking, compute, DNS) are managed in separate workspaces for isolation. But they need to share data. `terraform_remote_state` is the official coupling mechanism — it reads outputs from another workspace's state without the two configs needing to know anything about each other's internals.

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
