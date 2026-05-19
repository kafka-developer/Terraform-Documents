# HashiCorp Certified: Terraform Associate (004)
## Complete Study Guide — All 37 Topics

**Exam Version:** Terraform 1.12  
**Format:** 60 minutes · Multiple choice / True-False / Multi-select · ~57 questions  
**Pass Score:** Not published (typically ~70%)  
**Validity:** 2 years  
**Proctoring:** Certiverse (online, remote proctored)

> ⭐ **004 NEW topics** (added over 003): Ephemeral values & write-only arguments · Custom conditions (preconditions, postconditions, checks) · `depends_on` & `create_before_destroy` lifecycle emphasis · HCP Terraform Projects

---

## HOW TO USE THIS GUIDE

Each topic follows: **Concept** → **Why it matters** → **HCL/CLI syntax** → **Exam traps** → **Practice questions**

The exam tests *understanding of behaviour*, not syntax memorisation. The questions are scenario-based: "what happens when…", "which command…", "which statement is true about…". Study the *why*, not just the *what*.

---

## TABLE OF CONTENTS

| # | Section | Weight | Topics |
|---|---------|--------|--------|
| 1 | [Infrastructure as Code with Terraform](#section-1) | ~8% | 1a, 1b, 1c |
| 2 | [Terraform Fundamentals](#section-2) | ~11% | 2a, 2b, 2c, 2d |
| 3 | [Core Terraform Workflow](#section-3) | ~19% | 3a–3g |
| 4 | [Terraform Configuration](#section-4) | ~22% | 4a–4h |
| 5 | [Terraform Modules](#section-5) | ~11% | 5a–5d |
| 6 | [Terraform State Management](#section-6) | ~11% | 6a–6d |
| 7 | [Maintain Infrastructure with Terraform](#section-7) | ~8% | 7a–7c |
| 8 | [HCP Terraform](#section-8) | ~11% | 8a–8d |

---

---

<a id="section-1"></a>
# SECTION 1 — Infrastructure as Code with Terraform (~8%)

---

## 1a. What is Infrastructure as Code (IaC)?

### Concept

**Infrastructure as Code (IaC)** is the practice of defining and managing infrastructure resources — servers, networks, databases, DNS records, load balancers — using human-readable configuration files instead of manual processes (clicking through a console, running ad-hoc scripts).

The core insight behind IaC is that infrastructure should be treated with the same discipline as application code: it should be version-controlled, reviewed, tested, and deployed through automated pipelines. This eliminates the problem of "snowflake servers" — unique, hand-configured machines that no one fully understands and that can never be safely reproduced.

### Why Terraform specifically?

Terraform is a **declarative** IaC tool. You describe the *desired end state* of your infrastructure ("I want 3 EC2 instances with these properties") and Terraform figures out the operations needed to achieve it. This is different from *imperative* tools (like Ansible in procedural mode) where you describe the *steps* ("run this script, install this package, create this file").

Terraform is also **provider-agnostic**: the same workflow and language (HCL) applies whether you're provisioning AWS, Azure, GCP, Kubernetes, GitHub, Datadog, or hundreds of other providers. This is Terraform's key differentiator — a single tool for multi-cloud and multi-service infrastructure.

### Benefits of IaC (exam frequently tests these)

| Benefit | Explanation |
|---|---|
| **Reproducibility** | The same configuration always produces identical infrastructure. No more "it works on Bob's account" problems. |
| **Version control** | Infrastructure changes are tracked as Git commits. You can see who changed what, when, and why. |
| **Collaboration** | Teams review infrastructure changes via pull requests before they hit production. |
| **Automation** | Infrastructure provisioning integrates into CI/CD pipelines — no manual steps. |
| **Self-documentation** | The config *is* the documentation. Reading the HCL tells you exactly what exists. |
| **Idempotency** | Running Terraform multiple times against unchanged config produces no changes. Terraform only acts when there is drift. |
| **Disaster recovery** | Lost environment? Re-run `terraform apply`. Infrastructure is recreated from config in minutes. |

### Exam Traps

- The exam may ask you to distinguish between **declarative** (describe desired state → tool figures out steps) and **imperative** (describe exact steps) IaC approaches. Terraform is declarative.
- "Idempotent" means running the same operation multiple times produces the same result. If your infra matches the config, `terraform apply` makes zero changes. This is a frequently tested term.
- Terraform is **not a configuration management tool**. It provisions infrastructure. For configuring software *on* that infrastructure, you use tools like Ansible, Chef, Puppet, or cloud-init.

### Practice Questions

**Q1:** Which statement best describes Terraform's approach to IaC?  
A) Terraform executes shell scripts in sequence to build infrastructure  
B) Terraform describes desired infrastructure state and determines the steps to reach it  
C) Terraform copies an existing environment to create a new one  
D) Terraform requires you to specify the exact API calls for each provider  
✅ **Answer: B** — Terraform is declarative.

**Q2:** True or False: Running `terraform apply` twice with an unchanged configuration will provision duplicate resources.  
✅ **Answer: False** — Terraform is idempotent. The second apply detects no drift and makes no changes.

---

## 1b. Terraform vs Other IaC Tools

### Concept

The exam expects you to understand Terraform's position relative to other tools and *why you would choose it*.

| Tool | Type | Scope | Approach |
|---|---|---|---|
| **Terraform** | Declarative | Multi-cloud, multi-provider | State-driven; plans before acting |
| **CloudFormation** | Declarative | AWS only | AWS-native; no state file |
| **Ansible** | Primarily Imperative | Config management + some IaC | Agentless; procedural playbooks |
| **Pulumi** | Declarative | Multi-cloud | Uses real programming languages (Python, JS) |
| **Chef/Puppet** | Declarative | Config management | Agent-based; manages running systems |

### Key Terraform Differentiators

**Execution plans:** Before making any change, Terraform generates a plan showing exactly what will be created, modified, or destroyed. Operators can review and approve the plan before anything touches real infrastructure. This "plan-then-apply" safety model is one of Terraform's defining characteristics.

**State file:** Terraform maintains a mapping between your configuration and the real-world resources it manages. This state enables Terraform to know what already exists, detect drift, and calculate the minimum changes needed for each apply.

**Provider ecosystem:** The Terraform Registry hosts thousands of providers — official (maintained by HashiCorp or the cloud vendor) and community (maintained by the open-source community). Providers translate HCL resource definitions into the target platform's API calls.

---

## 1c. Terraform Use Cases

### Concept

Terraform is appropriate for:
- **Multi-tier applications:** Provision the network, compute, database, and DNS layers together with dependency ordering.
- **Disposable environments:** Spin up a complete testing environment, run tests, destroy everything. Each developer gets identical environments.
- **Self-service clusters:** Platform teams write modules; product teams consume them via simple variable inputs.
- **Multi-cloud:** Manage AWS, Azure, and GCP resources in a single plan.
- **Software-defined networking:** Configure Cloudflare, Datadog, PagerDuty, GitHub as code.

Terraform is **not** the right tool for:
- Installing software packages on existing VMs (use Ansible/Chef)
- Application deployment (use Kubernetes/Helm)
- Real-time policy enforcement on running systems (use OPA/Sentinel)

---

---

<a id="section-2"></a>
# SECTION 2 — Terraform Fundamentals (~11%)

---

## 2a. Terraform Providers

### Concept

A **provider** is a plugin that enables Terraform to interact with a specific platform or service — AWS, Azure, GCP, Kubernetes, GitHub, MySQL, Vault, and hundreds more. Providers translate HCL resource definitions into the target platform's API calls.

Without a provider, Terraform has no knowledge of any cloud or service. The provider acts as a translation layer: you write `resource "aws_instance" "web" { ... }` and the AWS provider knows how to call the EC2 API to create that instance.

Providers are distributed as binaries and are **downloaded from the Terraform Registry** during `terraform init`. They are stored in the `.terraform/` directory of your workspace.

### Provider Configuration

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"    # registry.terraform.io/hashicorp/aws
      version = "~> 5.0"           # version constraint
    }
    random = {
      source  = "hashicorp/random"
      version = ">= 3.0"
    }
  }
  required_version = ">= 1.12.0"  # minimum Terraform version
}

provider "aws" {
  region  = "us-east-1"
  profile = "default"             # AWS CLI profile
}
```

### Provider Source Address Format

`<HOSTNAME>/<NAMESPACE>/<TYPE>`

- **HOSTNAME** — registry where the provider is hosted. Defaults to `registry.terraform.io` if omitted.
- **NAMESPACE** — organisation. `hashicorp` for official providers.
- **TYPE** — provider name (e.g., `aws`, `google`, `azurerm`).

So `hashicorp/aws` is short for `registry.terraform.io/hashicorp/aws`.

### Multiple Provider Instances (Alias)

You can configure the same provider multiple times — for example, to manage resources in two different AWS regions:

```hcl
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

resource "aws_instance" "east" {
  # uses default provider (us-east-1)
  ami           = "ami-12345678"
  instance_type = "t3.micro"
}

resource "aws_instance" "west" {
  provider      = aws.west          # explicit reference to aliased provider
  ami           = "ami-87654321"
  instance_type = "t3.micro"
}
```

### Exam Traps

- `required_providers` goes inside the `terraform {}` block, not at the top level.
- The `source` argument was introduced in Terraform 0.13+. In older configs you may see providers without `source`, but the exam tests the modern approach.
- `terraform init` downloads providers. Re-running `init` after adding a new provider is required.
- The `version` constraint in `required_providers` is *your* constraint. The actual installed version is recorded in the **lock file** (see 2b).

### Practice Questions

**Q1:** Where in a Terraform configuration do you declare provider requirements and version constraints?  
A) In a separate `providers.tf` file at root level  
B) Inside a `required_providers` block within the `terraform {}` block  
C) Directly in each `resource` block  
D) In the `provider {}` configuration block  
✅ **Answer: B**

**Q2:** You need to manage AWS resources in both `us-east-1` and `eu-west-1` in the same configuration. What must you add to the second provider block?  
✅ **Answer:** An `alias` argument. Then reference `provider = aws.<alias>` in resources that use the second region.

---

## 2b. The Dependency Lock File

### Concept

The **dependency lock file** (`.terraform.lock.hcl`) records the exact provider versions and their cryptographic checksums that were selected when you last ran `terraform init`. It exists to ensure that every person on the team — and every CI/CD run — uses the *same* provider versions, even if the version constraints in `required_providers` allow a range.

Think of it as the infrastructure equivalent of `package-lock.json` (npm) or `Gemfile.lock` (Ruby). The constraint says "I accept any version matching `~> 5.0`." The lock file records "we selected `5.31.0` specifically, and here is its hash so we know it hasn't been tampered with."

### Key Properties

- **Committed to version control:** The lock file should always be committed to Git. This guarantees reproducibility across machines and CI runs.
- **Provider-only:** The lock file records provider dependencies, not module dependencies. Module versions are handled differently (registry modules use a `version` argument; local modules have no versioning).
- **Checksums:** Each provider entry includes `h1:` hashes (SHA-256 of the zip file) for multiple platforms. This allows Terraform to verify the provider binary hasn't been tampered with.
- **Upgrading:** Running `terraform init -upgrade` will re-evaluate constraints and potentially update provider versions. The lock file is updated to reflect the new selection.

```hcl
# .terraform.lock.hcl (auto-generated, do not edit manually)
provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.31.0"
  constraints = "~> 5.0"
  hashes = [
    "h1:abc123...",
    "zh:def456...",
  ]
}
```

### Exam Traps

- The lock file is **auto-generated** by `terraform init`. You never edit it manually.
- Committing it to version control is the **correct and recommended** practice.
- Running `terraform init` without `-upgrade` will NOT change provider versions if they're already in the lock file — even if newer versions matching the constraint exist.
- If the lock file and the installed providers disagree, `terraform init` will error.

### Practice Questions

**Q1:** True or False: The `.terraform.lock.hcl` file should be committed to version control.  
✅ **Answer: True** — It ensures team members use identical provider versions.

**Q2:** Which command updates provider versions within the allowed constraints and updates the lock file?  
A) `terraform refresh`  
B) `terraform init -upgrade`  
C) `terraform providers update`  
D) `terraform apply -refresh-only`  
✅ **Answer: B**

---

## 2c. Resources and Data Sources

### Concept

**Resource blocks** (`resource`) define infrastructure objects that Terraform manages. They are the primary building block of Terraform configuration. When you apply a configuration containing a resource, Terraform calls the provider's API to create, update, or delete the corresponding object in the real world.

**Data source blocks** (`data`) allow Terraform to *read* information from external sources — either from the provider's API or from other Terraform states — and use that information in your configuration. Data sources do not create or manage anything. They are read-only queries.

```hcl
# Resource — Terraform CREATES and MANAGES this
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
}

# Data source — Terraform READS this (doesn't create/manage it)
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]   # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]
  }
}

# Use data source result in a resource
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id    # referencing the data source
  instance_type = "t3.micro"
}
```

### Reference Syntax

- Resource: `<RESOURCE_TYPE>.<NAME>.<ATTRIBUTE>` — e.g., `aws_instance.web.id`
- Data source: `data.<DATA_SOURCE_TYPE>.<NAME>.<ATTRIBUTE>` — e.g., `data.aws_ami.ubuntu.id`

### Key Differences

| Aspect | Resource | Data Source |
|---|---|---|
| Purpose | Create/manage infrastructure | Read existing data |
| Terraform ownership | Yes — tracked in state | No — just a query result |
| Destroyed by `terraform destroy` | Yes | No |
| Can create side effects | Yes | No |
| Block keyword | `resource` | `data` |

### Exam Traps

- Data sources are **not** managed by Terraform. If the underlying object changes, the data source reflects the new value on the next plan. Terraform does not prevent that change.
- Data sources are evaluated during the **plan phase**, not just at apply time.
- You can use a data source to look up resources that were created outside Terraform — the most common use case.

---

## 2d. Resource Addressing and Dependency Graph

### Concept

Every resource in a Terraform configuration has a unique **address** that Terraform uses to identify it in the state, CLI output, and dependency graph.

**Address format:** `<RESOURCE_TYPE>.<NAME>`

Examples:
- `aws_instance.web`
- `aws_security_group.allow_http`
- `module.network.aws_vpc.main` (resource inside a module)

### Implicit Dependencies

Terraform automatically infers dependencies by analyzing resource references. If `resource B` references an attribute of `resource A`, Terraform knows A must be created before B. This is the dependency graph.

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id    # implicit dependency on aws_vpc.main
  cidr_block = "10.0.1.0/24"
}
```

Terraform will always create `aws_vpc.main` before `aws_subnet.public` because `aws_subnet.public` references `aws_vpc.main.id`. No `depends_on` needed here — the reference alone is sufficient.

### Explicit Dependencies (`depends_on`) — ⭐ NEW 004 EMPHASIS

Sometimes a resource depends on another resource's side effects — not any specific attribute, so there's no reference for Terraform to detect. In these cases you use `depends_on` to declare the dependency explicitly.

```hcl
resource "aws_iam_role_policy" "example" {
  name = "example"
  role = aws_iam_role.example.id
  policy = jsonencode({...})
}

resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"

  depends_on = [aws_iam_role_policy.example]  # wait for policy to exist
}
```

**Rule:** Only use `depends_on` when a dependency cannot be expressed through a direct attribute reference. Overusing `depends_on` makes configuration harder to reason about and can reduce parallelism (Terraform parallelises independent resources).

### Exam Traps

- `depends_on` should be a last resort. If you can reference an attribute, do that instead.
- `depends_on` accepts a list of resource references, **not strings**.
- `depends_on` can be added to resources, modules, and data sources.

---

---

<a id="section-3"></a>
# SECTION 3 — Core Terraform Workflow (~19%)

---

## 3a. `terraform init`

### Concept

`terraform init` is the **first command** you run in any Terraform workspace. It initialises the working directory by:

1. **Downloading providers** declared in `required_providers` and storing them in `.terraform/providers/`
2. **Downloading modules** referenced with `source` paths (from Registry or Git) and storing them in `.terraform/modules/`
3. **Initialising the backend** — configuring where state is stored (local or remote)
4. **Creating or validating the lock file** (`.terraform.lock.hcl`)

Until you run `init`, you cannot run `plan`, `apply`, or any other Terraform command.

### Key Flags

```bash
terraform init                    # standard init
terraform init -upgrade           # re-evaluate constraints; update providers + lock file
terraform init -reconfigure       # reinitialise backend even if already initialised
terraform init -migrate-state     # migrate state from old backend to new backend
terraform init -backend=false     # skip backend initialisation (local only)
```

### Exam Traps

- `terraform init` is **idempotent** — safe to re-run at any time.
- It does **not** read, modify, or apply any configuration changes to infrastructure.
- Running `init` is required after: adding a new provider, changing the backend, adding a module source.
- The `.terraform/` directory is **not** committed to version control (add to `.gitignore`). The lock file is committed.

---

## 3b. `terraform fmt`

### Concept

`terraform fmt` **automatically reformats** Terraform configuration files to the canonical HCL style — consistent indentation (2 spaces), aligned `=` signs, and standardised spacing. It modifies files in-place.

This is a purely cosmetic tool — it changes formatting, not logic. It does not validate correctness.

```bash
terraform fmt             # format all .tf files in current directory
terraform fmt -recursive  # format all .tf files in subdirectories too
terraform fmt -check      # check if files need formatting; exit non-zero if yes (CI use)
terraform fmt -diff       # show diff of changes without applying them
```

### Why it matters

In CI pipelines, `terraform fmt -check` is commonly run as a gate: if config files aren't properly formatted, the pipeline fails. This enforces consistent style across the team.

### Exam Traps

- `fmt` does **not** validate syntax or logic.
- `fmt -check` returns exit code `0` if no changes needed, `1` if files would be changed. This is the CI-friendly mode.

---

## 3c. `terraform validate`

### Concept

`terraform validate` checks your configuration for **syntactic and semantic correctness** — invalid syntax, references to undefined variables, incorrect argument types, or missing required arguments. It does not access any remote APIs or state.

`validate` can be run after `init` (it needs providers to be downloaded to validate resource schemas).

```bash
terraform validate
# Output: Success! The configuration is valid.
# Or: Error: An argument named "instanc_type" is not expected here.
```

### Difference from `fmt`

| Command | What it checks | Modifies files? | Requires network/state? |
|---|---|---|---|
| `terraform fmt` | Formatting/style only | Yes | No |
| `terraform validate` | Syntax and logic | No | No (but needs `init`) |
| `terraform plan` | Everything + real infra | No | Yes |

### Exam Traps

- `validate` catches typos in argument names, wrong types, missing required arguments.
- It does NOT check if the referenced cloud resources exist (that's `plan`).
- It requires `terraform init` to have been run first (needs provider schemas).

---

## 3d. `terraform plan`

### Concept

`terraform plan` generates an **execution plan** — a preview of the changes Terraform would make if you ran `apply`. It compares your configuration against the current state (and optionally against real infra via a refresh) and produces a diff.

This is the "plan" in Terraform's "plan-then-apply" safety model. You review what will happen before anything changes.

```bash
terraform plan                    # standard plan
terraform plan -out=tfplan        # save plan to file (use with apply)
terraform plan -refresh-only      # plan that only reconciles state (no config changes)
terraform plan -destroy           # plan a full destroy
terraform plan -target=aws_instance.web  # plan for one specific resource
terraform plan -var="env=prod"    # pass variable
terraform plan -var-file=prod.tfvars
```

### Reading Plan Output

```
# aws_instance.web will be created
+ resource "aws_instance" "web" {
    + ami           = "ami-12345678"
    + instance_type = "t3.micro"
    + id            = (known after apply)
  }

Plan: 1 to add, 0 to change, 0 to destroy.
```

Symbols:
- `+` — resource will be **created**
- `-` — resource will be **destroyed**
- `~` — resource will be **updated in-place**
- `-/+` — resource will be **destroyed and recreated** (replacement)
- `<=` — data source will be **read**

### Saving and Using Plan Files

```bash
# Save a plan
terraform plan -out=tfplan

# Apply exactly that saved plan (no interactive approval needed)
terraform apply tfplan
```

Saved plan files are important for: (a) separating the "who reviews" from the "who applies" step, (b) guaranteeing the applied changes match exactly what was reviewed.

### Exam Traps

- `plan` does NOT make any changes to infrastructure.
- By default, `plan` refreshes state (reads current real-world values). Use `-refresh=false` to skip this.
- `-refresh-only` plans do not apply configuration changes — they only update the state to match reality (drift reconciliation).
- `-target` is a last-resort escape hatch, not a routine practice. It can leave state inconsistent.
- Saved plan files are **binary** — not human-readable text files.

---

## 3e. `terraform apply`

### Concept

`terraform apply` executes the changes described in a plan — creating, updating, or destroying resources to bring real infrastructure into alignment with the configuration.

```bash
terraform apply                   # generates a plan, asks for approval, then applies
terraform apply tfplan            # applies a saved plan file (no approval prompt)
terraform apply -auto-approve     # skip interactive approval (CI use)
terraform apply -var="env=prod"
terraform apply -target=aws_instance.web  # apply only one resource
```

### What Apply Does

1. Refreshes state (reads real-world resource values)
2. Generates a plan (same as `terraform plan`)
3. Shows the plan and prompts for confirmation (type `yes`)
4. Executes the changes in dependency order, in parallel where possible
5. Updates the state file with the results

### Exam Traps

- When you type `yes` at the approval prompt, Terraform is committed. There is no undo.
- If you pass a saved plan file (`terraform apply tfplan`), Terraform skips the approval prompt — the plan IS the approval.
- `apply` updates the state file even if some resources fail — partial state is real and must be managed carefully.

---

## 3f. `terraform destroy`

### Concept

`terraform destroy` destroys all resources managed by the current workspace's state. It is equivalent to `terraform apply -destroy`.

```bash
terraform destroy                  # destroy all resources; prompts for approval
terraform destroy -auto-approve    # skip approval
terraform destroy -target=aws_instance.web  # destroy one resource
terraform plan -destroy            # preview what would be destroyed
```

### Exam Traps

- `terraform destroy` destroys resources in **reverse dependency order** — it's the opposite of create.
- Resources with `prevent_destroy = true` in their `lifecycle` block will cause `destroy` to fail with an error.
- Destroying does not delete the configuration files. Only the real-world resources and their state entries are removed.

---

## 3g. Dependency Graph and Parallelism

### Concept

Terraform builds a **directed acyclic graph (DAG)** of all resources. Resources with no dependencies on each other are provisioned in **parallel** (default: up to 10 concurrent operations). Resources that depend on others are sequenced accordingly.

```bash
terraform graph                    # outputs DOT graph representation
terraform graph | dot -Tsvg > graph.svg  # visualise (requires graphviz)
```

This parallelism is why adding unnecessary `depends_on` can slow down your applies — it introduces false sequencing constraints.

---

---

<a id="section-4"></a>
# SECTION 4 — Terraform Configuration (~22%)

---

## 4a. Resources and Data Sources (Deep Dive)

### Resource Meta-Arguments

Meta-arguments are special arguments available to *all* resource types, regardless of provider. They modify Terraform's default behaviour for the resource.

| Meta-Argument | Purpose |
|---|---|
| `depends_on` | Explicit dependency declaration |
| `count` | Create multiple instances using an integer |
| `for_each` | Create multiple instances using a map or set |
| `provider` | Specify a non-default provider instance |
| `lifecycle` | Customise create/update/destroy behaviour |

### `count`

`count` creates N identical (or nearly identical) copies of a resource. Each instance is addressed as `<TYPE>.<NAME>[<INDEX>]` where index is 0-based.

```hcl
resource "aws_instance" "server" {
  count         = 3
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  tags = {
    Name = "server-${count.index}"   # server-0, server-1, server-2
  }
}

# Reference: aws_instance.server[0].id, aws_instance.server[1].id, etc.
# All instances:  aws_instance.server[*].id
```

**The count index problem:** If you remove an item from the *middle* of a count-based list, Terraform re-indexes all items after the removed one. This triggers unnecessary destroy-and-recreate operations. This is the key weakness of `count` vs `for_each`.

### `for_each`

`for_each` creates one instance per element in a **map** or **set**. Each instance is addressed by its key, not a numeric index. Removing one item does not affect others.

```hcl
# Using a set of strings
resource "aws_iam_user" "team" {
  for_each = toset(["alice", "bob", "carol"])
  name     = each.key
}
# Addresses: aws_iam_user.team["alice"], aws_iam_user.team["bob"], etc.

# Using a map (key → object)
resource "aws_instance" "servers" {
  for_each = {
    web = "t3.micro"
    api = "t3.small"
    db  = "t3.medium"
  }
  ami           = "ami-12345678"
  instance_type = each.value
  tags = {
    Name = each.key
  }
}
# Addresses: aws_instance.servers["web"], aws_instance.servers["api"], etc.
```

**`each.key`** — the map key (or set value, since sets have no separate keys)  
**`each.value`** — the map value (for sets, same as `each.key`)

### `count` vs `for_each` — Exam Critical

| Aspect | `count` | `for_each` |
|---|---|---|
| Input type | Integer | Map or Set |
| Instance address | `[0]`, `[1]`, `[2]` | `["alice"]`, `["bob"]` |
| Removing one item | Re-indexes → side effects | Removes only that key |
| Best for | Identical resources | Resources that differ by key |
| `for_each` with a list | ❌ Not supported | Use `toset()` to convert |

### `lifecycle` Block — ⭐ NEW 004 EMPHASIS

The `lifecycle` block customises how Terraform handles a resource's creation, update, and destruction:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"

  lifecycle {
    create_before_destroy = true    # create replacement BEFORE destroying old
    prevent_destroy       = true    # block any attempt to destroy this resource
    ignore_changes        = [tags]  # don't act on changes to these attributes
    replace_triggered_by  = [aws_security_group.main]  # force replacement if this changes
  }
}
```

**`create_before_destroy`** — By default, Terraform destroys the old resource first, then creates the new one. With `create_before_destroy = true`, it creates the replacement first, then destroys the old. This minimises downtime for resources like load balancers and DNS records where a brief gap is unacceptable.

**`prevent_destroy`** — Acts as a safety catch. Terraform will **error** if a plan would destroy this resource. Useful for databases and other critical, irreplaceable resources. Note: this only protects during `terraform apply`. The resource can still be destroyed if the block is removed from config and then applied.

**`ignore_changes`** — Tells Terraform to ignore changes to specific attributes when calculating diffs. Use when an attribute is managed by an external process (e.g., AWS auto-scaling updates tags) and you don't want Terraform to revert those changes.

### Exam Traps

- `for_each` does not accept a plain `list`. You must convert with `toset()` if your source is a list.
- `create_before_destroy` on a resource propagates to all resources that depend on it — they also get `create_before_destroy` implicitly.
- `prevent_destroy = true` only prevents destruction via `terraform apply`. It does not protect against `terraform destroy` if the lifecycle block is removed before running destroy.

---

## 4b. Variables, Outputs, and Complex Types

### Input Variables

An **input variable** is a parameter for a Terraform configuration. It allows the same module or configuration to behave differently based on the values supplied at runtime. Variables are the mechanism for making configurations reusable and environment-agnostic.

```hcl
variable "instance_type" {
  type        = string
  description = "EC2 instance type to use"
  default     = "t3.micro"

  validation {
    condition     = contains(["t3.micro", "t3.small", "t3.medium"], var.instance_type)
    error_message = "Must be t3.micro, t3.small, or t3.medium."
  }
}
```

### Variable Assignment — Precedence (Lowest to Highest)

1. Default value in the variable block
2. Terraform Cloud / Enterprise workspace variables
3. `terraform.tfvars` file (auto-loaded if present)
4. `*.auto.tfvars` files (auto-loaded alphabetically)
5. `-var-file=filename.tfvars` CLI flag
6. `-var="key=value"` CLI flag
7. `TF_VAR_<NAME>` environment variables

**Auto-loaded files:** `terraform.tfvars` and any file matching `*.auto.tfvars` are loaded automatically without any CLI flags. Any other `.tfvars` file must be explicitly specified with `-var-file`.

### Variable Types — Primitives

**`string`** — A sequence of Unicode characters. Used for names, ARNs, regions, tags.
```hcl
variable "region" { type = string }
```

**`number`** — A numeric value. Can be integer or floating point. Terraform automatically converts between number and string in some contexts.
```hcl
variable "replica_count" { type = number; default = 2 }
```

**`bool`** — A boolean: `true` or `false`. Used for feature flags.
```hcl
variable "enable_monitoring" { type = bool; default = true }
```

### Variable Types — Collections

**`list(<TYPE>)`** — An **ordered** sequence of values of the same type. Supports indexing: `var.subnets[0]`. Allows duplicates. Use when order matters and items are identified by position.

```hcl
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b", "us-east-1c"]
}
# Access: var.availability_zones[0]  →  "us-east-1a"
```

**`set(<TYPE>)`** — A collection of **unique**, **unordered** values of the same type. A `set` is a collection type that stores an unordered group of unique values of the same data type. Unlike a `list`, a `set` automatically discards duplicate values and does not support indexing by position (you cannot call `set[0]`). Sets are primarily useful when you need uniqueness guarantees and will iterate over items without caring about order — most commonly to feed `for_each`.

```hcl
variable "allowed_cidrs" {
  type    = set(string)
  default = ["10.0.0.0/8", "172.16.0.0/12"]
}
# Convert list → set to eliminate duplicates and use with for_each:
# toset(["a", "b", "a"])  →  {"a", "b"}
```

**`map(<TYPE>)`** — A collection of **key-value pairs** where all values are the same type. Keys are strings. Unlike `object`, a `map` enforces that all values share the same type — you cannot mix strings and numbers. Use when you have a dynamic set of named values all of the same type (e.g., a map of environment-to-CIDR mappings).

```hcl
variable "env_cidrs" {
  type = map(string)
  default = {
    dev  = "10.0.0.0/16"
    prod = "10.1.0.0/16"
  }
}
# Access: var.env_cidrs["prod"]  →  "10.1.0.0/16"
```

### Variable Types — Structural

**`object({...})`** — A structured type where each attribute can have a different type. Unlike `map`, an `object` specifies the exact schema: attribute names and their individual types. Use for complex, structured configuration where different fields have different types.

```hcl
variable "db_config" {
  type = object({
    engine        = string
    version       = string
    instance_class = string
    multi_az      = bool
    storage_gb    = number
  })
  default = {
    engine         = "mysql"
    version        = "8.0"
    instance_class = "db.t3.micro"
    multi_az       = false
    storage_gb     = 20
  }
}
# Access: var.db_config.engine, var.db_config.multi_az
```

**`tuple([<TYPE>, ...])`** — A fixed-length sequence where each element can have a *different* type. Unlike `list` (same type, any length), a `tuple` has a fixed structure — the first element is one type, the second another. Rarely used directly but useful for function return values.

```hcl
variable "server_spec" {
  type    = tuple([string, number, bool])
  default = ["t3.micro", 2, true]
}
# var.server_spec[0] = "t3.micro", var.server_spec[1] = 2, var.server_spec[2] = true
```

### Type Comparison Table

| Type | Order | Duplicates | Mixed types | Index access |
|---|---|---|---|---|
| `list` | ✅ Yes | ✅ Yes | ❌ No | ✅ `[0]`, `[1]` |
| `set` | ❌ No | ❌ No (auto-removed) | ❌ No | ❌ No |
| `map` | ❌ No | N/A (keys unique) | ❌ No (values same type) | ✅ `["key"]` |
| `object` | N/A | N/A | ✅ Yes (per attribute) | ✅ `.attr` |
| `tuple` | ✅ Yes | ✅ Yes | ✅ Yes | ✅ `[0]`, `[1]` |

### Output Values

An **output value** exposes data from a Terraform configuration to the CLI, to HCP Terraform, or to a calling module. Outputs are declared with the `output` block.

```hcl
output "instance_ip" {
  description = "Public IP of the web server"
  value       = aws_instance.web.public_ip
  sensitive   = true   # hides value in CLI output; still stored in state
}

output "bucket_arn" {
  value = aws_s3_bucket.data.arn
}
```

**`sensitive = true`** on an output does **not** encrypt the value in state. It only prevents Terraform from printing the value in CLI output. The value is still stored in plaintext in `terraform.tfstate`. Anyone with read access to the state file can see it.

**Viewing outputs:**
```bash
terraform output                   # show all outputs
terraform output instance_ip       # show specific output
terraform output -json             # machine-readable JSON
terraform output -raw instance_ip  # raw value, no quotes (useful in scripts)
```

### Local Values

A **local value** assigns a name to an expression within a module. It's like a constant or computed intermediate value — you define it once, reference it many times. Unlike variables, locals cannot be set from outside the module.

```hcl
locals {
  env     = "production"
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

- `terraform.tfvars` and `*.auto.tfvars` are loaded **automatically**. Any other `.tfvars` file must be explicitly passed with `-var-file`.
- `sensitive = true` on an output hides the value in CLI output but does NOT protect it in the state file.
- `for_each` requires a **map or set**. Pass a `list` → error. Use `toset()` to convert.
- `local` values are referenced as `local.<name>` (singular), not `locals.<name>`.

---

## 4c. Expressions, Built-in Functions, and Dynamic Blocks

### String Interpolation and Directives

```hcl
# Interpolation
name = "server-${var.environment}-${count.index}"

# Heredoc (multi-line string)
user_data = <<-EOT
  #!/bin/bash
  echo "Hello ${var.name}"
  apt-get update
EOT
```

### Conditional Expression (Ternary)

```hcl
# condition ? true_value : false_value
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
min_size      = var.high_availability ? 3 : 1
```

### `for` Expressions

`for` expressions transform one collection into another:

```hcl
# Transform a list
variable "names" { default = ["alice", "bob", "carol"] }
locals {
  upper_names = [for n in var.names : upper(n)]
  # Result: ["ALICE", "BOB", "CAROL"]
}

# Filter with if clause
locals {
  prod_instances = [for k, v in var.instances : k if v.environment == "prod"]
}

# List → Map
locals {
  name_to_upper = {for n in var.names : n => upper(n)}
  # Result: {"alice" = "ALICE", "bob" = "BOB", ...}
}
```

### Splat Expressions

```hcl
# Old way (explicit for)
instance_ids = [for i in aws_instance.web : i.id]

# Splat operator (shorthand for same thing)
instance_ids = aws_instance.web[*].id
```

### Key Built-in Functions

**String functions:**
```hcl
upper("hello")          # "HELLO"
lower("HELLO")          # "hello"
trim("  hello  ", " ")  # "hello"
trimspace("  hello  ")  # "hello"
replace("hello", "l", "r")  # "herro"
split(",", "a,b,c")     # ["a", "b", "c"]
join("-", ["a","b","c"]) # "a-b-c"
format("Hello, %s!", var.name)
substr("hello world", 0, 5)  # "hello"
```

**Collection functions:**
```hcl
length(["a","b","c"])   # 3
toset(["a","a","b"])    # {"a","b"}  (removes duplicates)
tolist(toset(["b","a"])) # a list (order not guaranteed)
flatten([["a","b"],["c"]]) # ["a","b","c"]
distinct(["a","b","a"])  # ["a","b"]
concat(["a"],["b","c"])  # ["a","b","c"]
contains(["a","b"],"b")  # true
keys({a=1, b=2})        # ["a","b"]
values({a=1, b=2})      # [1, 2]
lookup({a=1}, "a", 0)   # 1 (default 0 if key missing)
merge({a=1},{b=2})      # {a=1, b=2}
```

**Numeric functions:**
```hcl
min(1, 2, 3)   # 1
max(1, 2, 3)   # 3
floor(1.7)     # 1
ceil(1.2)      # 2
abs(-5)        # 5
```

**Encoding/JSON:**
```hcl
jsonencode({name = "alice"})     # '{"name":"alice"}'
jsondecode('{"name":"alice"}')   # {name = "alice"}
base64encode("hello")            # "aGVsbG8="
base64decode("aGVsbG8=")         # "hello"
```

**Type conversion:**
```hcl
tostring(42)       # "42"
tonumber("42")     # 42
tobool("true")     # true
toset([...])       # converts list to set (removes duplicates)
```

**File functions:**
```hcl
file("script.sh")            # reads file content as string
filebase64("image.png")      # reads and base64-encodes
templatefile("tmpl.tpl", {name = var.name})  # renders template
```

### Dynamic Blocks

A **dynamic block** generates repeated nested blocks within a resource based on a collection. Without dynamic blocks, you'd have to hard-code every nested block; with dynamic, you iterate.

```hcl
# Without dynamic (repetitive):
resource "aws_security_group" "web" {
  ingress {
    from_port = 80; to_port = 80; protocol = "tcp"; cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port = 443; to_port = 443; protocol = "tcp"; cidr_blocks = ["0.0.0.0/0"]
  }
}

# With dynamic (elegant):
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

Inside the `content` block, the iterator variable is named after the dynamic block label (`ingress` in this case). You access items with `<iterator>.value` and `<iterator>.key`.

---

## 4d. Input Validation

### Concept

**Input validation** allows you to add custom rules to variable blocks that Terraform checks *before* attempting to apply. If the condition is false, Terraform reports the error immediately, without making any API calls. This prevents bad configurations from reaching your infrastructure.

```hcl
variable "environment" {
  type        = string
  description = "Deployment environment"

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

The `condition` expression must evaluate to `true` or `false`. You can use `can()` to test whether an expression succeeds without an error — useful for validating formats.

The `error_message` must be a plain string (no interpolation of resource attributes). It should clearly explain what valid values look like.

### Multiple Validation Blocks

A single variable can have multiple `validation` blocks. Each is evaluated independently.

```hcl
variable "port" {
  type = number

  validation {
    condition     = var.port >= 1
    error_message = "Port must be at least 1."
  }

  validation {
    condition     = var.port <= 65535
    error_message = "Port must be at most 65535."
  }
}
```

---

## 4e. Expressions — `can()` and `try()`

**`can(expression)`** — Returns `true` if the expression succeeds, `false` if it would produce an error. Used inside `validation` blocks to test whether a value has a valid format.

```hcl
can(cidrhost("10.0.0.0/16", 0))  # true — valid CIDR
can(cidrhost("not-a-cidr", 0))   # false — invalid format
```

**`try(expr1, expr2, ...)`** — Evaluates expressions left to right and returns the value of the first one that doesn't produce an error. Like a try-catch for expressions.

```hcl
try(var.config.optional_field, "default_value")  # returns optional_field if it exists, else default
```

---

## 4f. `terraform console`

### Concept

`terraform console` opens an interactive REPL (Read-Eval-Print Loop) where you can evaluate Terraform expressions, test functions, and inspect values from your state. Useful for debugging and learning.

```bash
terraform console

> length(["a", "b", "c"])
3
> upper("hello")
"HELLO"
> toset(["x", "x", "y"])
toset(["x", "y"])
> var.region
"us-east-1"
> aws_instance.web.public_ip
"54.1.2.3"
```

---

## 4g. Custom Conditions — Preconditions, Postconditions, Checks ⭐ NEW 004

### Concept

Custom conditions extend Terraform's validation capabilities beyond input variables. They allow you to assert expectations about your configuration and infrastructure at different points in the lifecycle.

### Preconditions

A **precondition** is a check that runs *before* Terraform makes changes to a resource. It validates inputs and configuration assumptions before any API calls occur.

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

A **postcondition** is a check that runs *after* Terraform creates or updates a resource. It validates that the resulting resource has the expected properties — catching cases where the provider created the resource but with unexpected attributes.

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

Note: postconditions use `self` to reference the resource's own attributes.

### Check Blocks (Terraform 1.5+)

A **`check` block** is a standalone assertion that runs after every plan and apply. Unlike preconditions/postconditions (which fail the apply on error), check blocks produce **warnings**, not errors. They're used for asserting ambient conditions (e.g., a URL is reachable) without blocking the apply.

```hcl
check "health_check" {
  data "http" "app" {
    url = "https://api.example.com/health"
  }

  assert {
    condition     = data.http.app.status_code == 200
    error_message = "Health check endpoint returned ${data.http.app.status_code}."
  }
}
```

### When to Use Which

| Mechanism | When it runs | Failure effect | Use case |
|---|---|---|---|
| Variable `validation` | Before plan | Blocks plan | Validate user inputs |
| `precondition` | Before resource create/update | Blocks apply | Validate config assumptions |
| `postcondition` | After resource create/update | Blocks apply | Validate resource outcome |
| `check` block | After apply | Warning only | Assert ambient health |

---

## 4h. Ephemeral Values and Write-Only Arguments ⭐ NEW 004 (Terraform 1.12)

### Concept

**Ephemeral values** are a Terraform 1.12 feature for handling sensitive data — like passwords, tokens, and API keys — that should **never be persisted to the state file**. Traditional Terraform secrets (variables with `sensitive = true`) are hidden in CLI output but still written to state. Ephemeral values exist only in memory during a plan/apply and are never stored.

### Ephemeral Resources

An **ephemeral resource** is like a data source, but its result is never stored in state. The value is fetched fresh on every run and discarded after.

```hcl
ephemeral "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  # This password is never written to state
  password = ephemeral.aws_secretsmanager_secret_version.db_password.secret_string
}
```

### Write-Only Arguments

A **write-only argument** is a resource attribute that Terraform accepts as input but **never reads back from the provider** and **never stores in state**. Introduced in Terraform 1.12 for password-type fields.

```hcl
resource "aws_db_instance" "main" {
  engine         = "mysql"
  instance_class = "db.t3.micro"
  password       = var.db_password    # if password is write-only, it won't appear in state
}
```

Write-only arguments are declared in the provider schema. Terraform sets them during create/update but never reads them back during refresh. This means the provider's API response for that attribute is ignored — preventing the secret from ever landing in state.

### Why This Matters for the Exam

The 004 exam specifically tests the distinction between:
- `sensitive = true` on a variable/output — hides in CLI, **still stored in state**
- Ephemeral values — never stored in state, only in memory during execution
- Write-only arguments — sent to provider but never read back or stored in state

---

---

<a id="section-5"></a>
# SECTION 5 — Terraform Modules (~11%)

---

## 5a. What is a Module?

### Concept

A **module** is a container for multiple related Terraform resources. Every Terraform configuration has at least one module: the **root module** (the directory where you run Terraform). Any other directory of `.tf` files that is called from the root module is a **child module**.

Modules are Terraform's primary mechanism for code reuse. Instead of copy-pasting the same VPC configuration across 10 projects, you write it once as a module and call it 10 times with different variables.

```hcl
# Calling a module from the Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "prod-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
}

# Referencing module outputs
resource "aws_instance" "app" {
  subnet_id = module.vpc.private_subnets[0]
}
```

---

## 5b. Module Sources

### Concept

The `source` argument tells Terraform where to find the module's code. Terraform supports several source types:

| Source Type | Example | Notes |
|---|---|---|
| **Local path** | `source = "./modules/vpc"` | Must start with `./` or `../` |
| **Terraform Registry** | `source = "hashicorp/consul/aws"` | Public registry; requires `version` |
| **GitHub** | `source = "github.com/org/repo//modules/vpc"` | `//` separates repo from subdirectory |
| **Git (generic)** | `source = "git::https://example.com/repo.git"` | |
| **HTTP URL** | `source = "https://example.com/module.zip"` | |
| **S3 bucket** | `source = "s3::https://s3.amazonaws.com/bucket/module.zip"` | |

### Registry Module Address Format

`<NAMESPACE>/<MODULE_NAME>/<PROVIDER>`

Example: `hashicorp/consul/aws` → the `consul` module for AWS, published by HashiCorp.

### `version` Constraint

For registry and remote modules, always pin a version:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"     # accept 5.x but not 6.x
}
```

Version constraints:
- `= 1.2.0` — exact version
- `>= 1.2.0` — any version 1.2.0 or newer
- `~> 1.2.0` — pessimistic constraint: `>= 1.2.0, < 1.3.0` (patch updates only)
- `~> 1.2` — `>= 1.2, < 2.0` (minor updates)
- `>= 1.0.0, < 2.0.0` — range

### Exam Traps

- Local paths use `./` or `../`. A bare `modules/vpc` (no `./`) is NOT treated as a local path — it would be interpreted as a registry address.
- `version` is only valid for registry and some remote sources. Local paths ignore `version`.
- After changing a module `source` or `version`, run `terraform init` to download the new module code.

---

## 5c. Module Inputs and Outputs

### Concept

Modules communicate through **input variables** (caller passes values in) and **output values** (module exposes values out).

```hcl
# modules/vpc/variables.tf
variable "name" {
  type        = string
  description = "VPC name"
}
variable "cidr" {
  type = string
}

# modules/vpc/outputs.tf
output "vpc_id" {
  value = aws_vpc.main.id
}
output "private_subnets" {
  value = aws_subnet.private[*].id
}

# Root module calling the child module
module "vpc" {
  source = "./modules/vpc"
  name   = "my-vpc"          # passed as input variable
  cidr   = "10.0.0.0/16"
}

# Consuming child module outputs
resource "aws_instance" "app" {
  subnet_id = module.vpc.private_subnets[0]   # module.<NAME>.<OUTPUT>
}
```

### Variable Scope

Variables declared inside a module are **scoped to that module**. They are invisible to the root module and other child modules. The only way data crosses module boundaries is through explicit `variable` inputs (in) and `output` values (out).

---

## 5d. Module Best Practices and Structure

### Standard Module Structure

```
terraform-aws-vpc/
├── main.tf           # primary resource definitions
├── variables.tf      # input variable declarations
├── outputs.tf        # output value declarations
├── README.md         # documentation
├── versions.tf       # terraform {} and required_providers
└── examples/         # example usage
    └── basic/
        ├── main.tf
        └── README.md
```

### Exam Traps

- Modules do not inherit provider configurations from the calling module — unless you pass them explicitly. You may need to use `providers` argument in the module call.
- Module outputs must be explicitly declared with `output` blocks. Resources inside a module are not directly accessible from outside.
- `terraform init` must be run after adding or changing a module source.

---

---

<a id="section-6"></a>
# SECTION 6 — Terraform State Management (~11%)

---

## 6a. Purpose of State

### Concept

Terraform's **state file** (`terraform.tfstate`) is the cornerstone of how Terraform works. It is a JSON file that maps the resources defined in your configuration to the real-world objects they represent. Without state, Terraform would have no memory of what it previously created.

The state serves three primary purposes:

**1. Mapping configuration to real world:** Your config says `resource "aws_instance" "web"`. The state records that this corresponds to the EC2 instance with ID `i-0abc1234def56789`. On the next plan, Terraform knows to compare `i-0abc1234def56789` against the config — not create a new one.

**2. Metadata tracking:** State stores metadata that can't be queried from the provider — like resource dependencies and the sequence in which resources were created. This enables correct destroy ordering.

**3. Performance:** For large configurations with hundreds of resources, querying every resource's real-world state on every plan would be slow. Terraform uses the cached state to calculate plans without making API calls for every resource (controlled by `-refresh` flag).

### State File Location

By default, Terraform stores state locally as `terraform.tfstate` in the working directory. For teams, local state is unworkable — there's no locking, no sharing, no history. Remote backends solve this.

### Exam Traps

- The state file may contain sensitive values (passwords, secrets) in plaintext. Protect it accordingly.
- **Never edit the state file manually** (except in specific cases using `terraform state` commands).
- Deleting the state file does not delete your infrastructure. But Terraform will think none of it exists and will try to create everything from scratch on the next apply.

---

## 6b. Remote Backends and State Locking

### Concept

A **backend** determines how Terraform stores state and where operations run. The default backend is `local` — state lives in `terraform.tfstate` on your machine. Remote backends store state in a shared, persistent location.

### Common Remote Backends

| Backend | State Storage | Locking |
|---|---|---|
| `s3` (with DynamoDB) | AWS S3 bucket | DynamoDB table |
| `azurerm` | Azure Blob Storage | Blob leases |
| `gcs` | Google Cloud Storage | GCS object locking |
| `http` | Any HTTP endpoint | If endpoint supports it |
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

**State locking** prevents multiple users or CI runs from applying changes simultaneously — which would corrupt the state file. When one operation acquires the lock, others must wait. Most remote backends support locking automatically.

If Terraform crashes while holding a lock, the lock may need to be manually released:
```bash
terraform force-unlock <LOCK_ID>
```

### Partial Backend Configuration

You can leave some backend config values empty in the configuration and pass them at init time. This is useful for keeping secrets out of config files:

```bash
terraform init -backend-config="bucket=my-state-bucket" \
               -backend-config="region=us-east-1"
```

### Exam Traps

- The `s3` backend does not natively lock state. You must configure a **DynamoDB table** separately for locking.
- Migrating from one backend to another: use `terraform init -migrate-state`.
- Backend configuration cannot use variables, `locals`, or resource references — only literal values (or `-backend-config` overrides).
- HCP Terraform has locking built in — no separate DynamoDB table needed.

---

## 6c. Drift Detection and `terraform refresh`

### Concept

**Drift** occurs when the real-world state of infrastructure diverges from what Terraform's state file records. This can happen when someone manually changes a resource through the console, or when an external process modifies configuration.

**Detecting drift:**
```bash
terraform plan              # implicitly refreshes; shows drift as proposed changes
terraform plan -refresh-only   # plan that ONLY reconciles state (no config changes)
```

**`-refresh-only` mode** (added in Terraform 1.1) generates a plan that, if applied, updates the state file to match reality — without changing the configuration or applying config changes. This is the correct way to reconcile drift when you want to accept the manual changes.

```bash
# See drift without changing anything
terraform plan -refresh-only

# Accept the drift into state
terraform apply -refresh-only
```

**`-refresh=false`** — Skip the refresh phase entirely. Terraform uses only the cached state, not real-world values. Use only when you're certain nothing has changed and you want a faster plan.

### `terraform state` Commands

```bash
terraform state list                           # list all resources in state
terraform state show aws_instance.web          # show state for one resource
terraform state mv aws_instance.web aws_instance.app  # rename resource in state
terraform state rm aws_instance.web            # remove resource from state (does NOT destroy it)
terraform state pull                           # download remote state to stdout
terraform state push terraform.tfstate         # upload a state file to backend
```

**`state rm`** is useful when you want to stop managing a resource with Terraform without destroying it. The real resource remains; only the state entry is removed.

### Exam Traps

- `terraform refresh` (standalone command) is **deprecated** in recent Terraform versions. Use `terraform apply -refresh-only` instead.
- `terraform plan` automatically refreshes state by default. No separate refresh step needed.
- `state rm` removes from Terraform management — the cloud resource still exists.

---

## 6d. `terraform import` and `moved` / `removed` Blocks

### `terraform import`

`terraform import` brings an **existing real-world resource** under Terraform management without destroying and recreating it. It reads the current state of the resource and adds it to the Terraform state file.

```bash
terraform import aws_instance.web i-0abc1234def56789
```

**Critical limitation:** `import` only updates the **state** file. It does **not** generate the corresponding HCL configuration. You must write the `resource` block yourself, then run `import` to link the state to it. Running `plan` after import will show if your config matches the real resource.

Terraform 1.5+ introduced **`import` blocks** — a declarative, configuration-based approach:

```hcl
import {
  to = aws_instance.web
  id = "i-0abc1234def56789"
}
```

This can be combined with `terraform plan -generate-config-out=generated.tf` to auto-generate the HCL configuration.

### `moved` Block

The `moved` block tells Terraform that a resource has been renamed in the configuration — without destroying and recreating it. Terraform updates the state reference to point to the new address.

```hcl
# You renamed aws_instance.web to aws_instance.app in your config
moved {
  from = aws_instance.web
  to   = aws_instance.app
}
```

Without the `moved` block, Terraform would destroy `aws_instance.web` and create `aws_instance.app` — identical resources but needlessly replaced.

### `removed` Block (Terraform 1.7+)

The `removed` block gracefully removes a resource from Terraform state without destroying the real-world object. This is the declarative equivalent of `terraform state rm`.

```hcl
removed {
  from = aws_instance.legacy

  lifecycle {
    destroy = false    # don't destroy the real resource; just remove from state
  }
}
```

### Exam Traps

- `terraform import` adds to state — does NOT generate config.
- After import, `terraform plan` should show no changes if your config correctly describes the resource.
- `moved` blocks are for refactoring — renaming resources or moving them between modules.
- `moved` blocks can be left in configs permanently (they're idempotent after the first apply) or cleaned up later.

---

---

<a id="section-7"></a>
# SECTION 7 — Maintain Infrastructure with Terraform (~8%)

---

## 7a. Terraform Workspaces

### Concept

A **workspace** is a named, isolated instance of state. Each workspace has its own independent state file. The `default` workspace exists in every configuration and is where Terraform operates unless you switch.

Workspaces allow you to use the same configuration to manage multiple environments (dev, staging, prod) from the same directory, with separate state for each.

```bash
terraform workspace list          # list all workspaces
terraform workspace show          # show current workspace
terraform workspace new staging   # create and switch to a new workspace
terraform workspace select prod   # switch to an existing workspace
terraform workspace delete staging  # delete a workspace (must be empty)
```

**Referencing workspace in config:**
```hcl
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"
  tags = {
    Environment = terraform.workspace
  }
}
```

### Workspace Limitations

Workspaces are useful for lightweight environment separation but have limitations:
- All workspaces share the same backend and configuration.
- There's no per-workspace variable management built in.
- Access controls cannot be applied per workspace in open-source Terraform.

For more isolation, separate root modules per environment (with different state files) is often preferred.

### Exam Traps

- The `default` workspace cannot be deleted.
- Workspaces in open-source Terraform are **different from HCP Terraform workspaces** — HCP Terraform workspaces are heavier and include VCS integration, variable management, and RBAC.
- Switching workspace changes which state Terraform reads. The configuration files stay the same.

---

## 7b. Debugging and Logging

```bash
# Enable verbose logging
TF_LOG=DEBUG terraform plan      # DEBUG, INFO, WARN, ERROR, TRACE

# Write logs to file
TF_LOG=DEBUG TF_LOG_PATH=./terraform.log terraform apply

# Provider-specific logging
TF_LOG_PROVIDER=DEBUG terraform plan
```

Logging levels (increasing verbosity): `ERROR` → `WARN` → `INFO` → `DEBUG` → `TRACE`

---

## 7c. Terraform Taint (Deprecated) and `replace`

`terraform taint` marked a resource for forced replacement on next apply. It is **deprecated** in Terraform 1.x. Use `-replace` flag instead:

```bash
terraform apply -replace=aws_instance.web
```

This forces Terraform to destroy and recreate the specified resource, even if the config hasn't changed — useful when a resource is in a bad state that Terraform can't detect.

---

---

<a id="section-8"></a>
# SECTION 8 — HCP Terraform (~11%)

---

## 8a. What is HCP Terraform?

### Concept

**HCP Terraform** (HashiCorp Cloud Platform Terraform, formerly Terraform Cloud) is HashiCorp's hosted service for Terraform. It extends the open-source Terraform CLI with:

- **Remote state storage** — state stored securely in HCP Terraform, not on local machines
- **State locking** — built-in, no external DynamoDB needed
- **Remote execution** — runs happen on HCP Terraform's infrastructure, not your laptop
- **VCS integration** — plans triggered automatically by Git push
- **Team collaboration** — role-based access control, plan review, approval workflows
- **Variable management** — store sensitive variables securely, outside of config files
- **Policy enforcement** — Sentinel policies for governance

---

## 8b. HCP Terraform Workspaces

### Concept

An **HCP Terraform workspace** is the fundamental unit of operation. Each workspace:
- Has its own state file
- Has its own variable set
- Connects to one VCS repository (or uses CLI/API)
- Has its own run history
- Has its own access controls

This is much richer than CLI workspaces. In HCP Terraform, a "workspace" is closer to "one environment for one configuration."

### Execution Modes

| Mode | Where Terraform runs | State stored |
|---|---|---|
| **Remote** (default) | HCP Terraform's infrastructure | HCP Terraform |
| **Local** | Your machine | HCP Terraform (remote state only) |
| **Agent** | Your own agents (for private networks) | HCP Terraform |

### VCS-Driven Workflow

With VCS integration (GitHub, GitLab, Bitbucket):
1. Developer opens a pull request → HCP Terraform runs `terraform plan`
2. Plan is shown in the PR
3. PR is merged → HCP Terraform runs `terraform apply` (with optional approval gate)

This creates a fully auditable, automated GitOps workflow for infrastructure.

### CLI-Driven Workflow

You run `terraform plan` and `terraform apply` locally, but Terraform is actually executing on HCP Terraform's infrastructure (remote execution). State is stored in HCP Terraform.

```hcl
# Configure HCP Terraform backend
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "prod-network"
    }
  }
}
```

Then authenticate:
```bash
terraform login    # opens browser to get API token; stores in ~/.terraform.d/credentials.tfrc.json
```

---

## 8c. HCP Terraform Projects ⭐ NEW 004

### Concept

**Projects** are a way to group and organise multiple HCP Terraform workspaces. As organisations scale to dozens or hundreds of workspaces, projects provide:
- **Logical grouping** — e.g., all workspaces for the "payments" team in one project
- **Project-level access control** — grant a team access to all workspaces in a project without configuring each workspace separately
- **Project-level variable sets** — share variables across all workspaces in a project

Every workspace belongs to exactly one project. A default project exists in every organisation.

### Project vs Workspace

| Concept | Purpose |
|---|---|
| **Project** | Organisational container for workspaces; governs access and shared config |
| **Workspace** | Operational unit; one environment, one state, one VCS connection |

### Exam Traps

- Projects are an HCP Terraform concept — they do not exist in open-source Terraform.
- A workspace belongs to exactly one project.
- Project-level permissions cascade to all workspaces in the project.

---

## 8d. Variable Sets and Run Triggers

### Variable Sets

A **variable set** is a reusable collection of variables that can be applied to multiple workspaces (or an entire project or organisation). Instead of configuring the same AWS credentials in 20 workspaces, you create one variable set and apply it to all 20.

Variable sets can be:
- **Globally applied** — applied to every workspace in the organisation
- **Project-scoped** — applied to every workspace in a project
- **Workspace-scoped** — applied to specific individual workspaces

Variables in HCP Terraform can be:
- **Terraform variables** — equivalent to `TF_VAR_<name>` or `-var` (HCL values)
- **Environment variables** — set as environment variables during execution (e.g., `AWS_ACCESS_KEY_ID`)
- **Sensitive** — value is write-once, never displayed after saving

### Run Triggers

A **run trigger** creates a dependency between workspaces — when workspace A successfully applies, workspace B is automatically queued for a plan/apply. Used to chain dependent infrastructure:

- Workspace A: shared networking (VPC, subnets)
- Workspace B: application infrastructure — triggered when workspace A changes

### `terraform_remote_state` Data Source

Allows one workspace to read outputs from another workspace's state:

```hcl
data "terraform_remote_state" "network" {
  backend = "remote"
  config = {
    organization = "my-org"
    workspaces = {
      name = "prod-network"
    }
  }
}

resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.network.outputs.private_subnet_id
}
```

---

---

# QUICK REFERENCE — EXAM CHEATSHEET

## The 8 Sections and Weight

| Section | Weight | Key Topics |
|---|---|---|
| 1. IaC with Terraform | ~8% | Declarative, idempotent, multi-cloud |
| 2. Terraform Fundamentals | ~11% | Providers, lock file, resources vs data sources |
| 3. Core Workflow | ~19% | init → fmt → validate → plan → apply → destroy |
| 4. Configuration | ~22% | Variables, types, lifecycle, validation, ephemeral ⭐ |
| 5. Modules | ~11% | Source, version, inputs/outputs |
| 6. State | ~11% | Purpose, backends, locking, drift, import/moved ⭐ |
| 7. Maintenance | ~8% | Workspaces, logging, replace |
| 8. HCP Terraform | ~11% | Workspaces, projects ⭐, variable sets, VCS |

⭐ = new or expanded in 004

---

## Critical Variable Precedence (Low → High)

```
default in variable block
  ↓
Terraform Cloud workspace variables
  ↓
terraform.tfvars  /  *.auto.tfvars  (auto-loaded)
  ↓
-var-file=custom.tfvars  (explicit)
  ↓
-var="key=value"  (CLI flag)
  ↓
TF_VAR_<name>  (environment variable)   ← HIGHEST
```

---

## Type Quick Reference

```hcl
string     = "hello"
number     = 42
bool       = true
list       = ["a", "b", "c"]        # ordered, duplicates OK, [0] index
set        = toset(["a", "b"])       # unordered, no duplicates, no index
map        = {a = 1, b = 2}          # key-value, all same type
object     = {name = "x", age = 30}  # key-value, mixed types
tuple      = ["x", 2, true]          # fixed-length, mixed types
```

---

## Lifecycle Arguments

```hcl
lifecycle {
  create_before_destroy = true   # replace → create new first, then destroy old
  prevent_destroy       = true   # errors if plan would destroy this resource
  ignore_changes        = [tags] # ignore changes to these attributes
  replace_triggered_by  = [other_resource]  # force replacement if this changes
}
```

---

## State Commands

```bash
terraform state list              # all resources in state
terraform state show ADDR         # full details for one resource
terraform state mv OLD NEW        # rename/move resource in state
terraform state rm ADDR           # remove from state (resource still exists)
terraform import ADDR ID          # import existing resource to state
terraform force-unlock LOCK_ID    # release stuck lock
```

---

## `count` vs `for_each` Decision Rule

```
Use count  → number of identical resources is known, order doesn't matter
Use for_each → resources differ by a key; you need stable addresses; you may remove individual items
for_each input must be a map or set → use toset() if you have a list
```

---

## Sensitive Data — What Each Mechanism Actually Does

| Mechanism | Hidden in CLI | Stored in State | Notes |
|---|---|---|---|
| `sensitive = true` (variable) | ✅ Yes | ✅ Yes (plaintext) | Just cosmetic |
| `sensitive = true` (output) | ✅ Yes | ✅ Yes (plaintext) | Just cosmetic |
| `TF_VAR_` env var | ✅ (not in config) | ✅ Yes | Still in state |
| Ephemeral values | ✅ Yes | ❌ **Never** | True secret handling |
| Write-only arguments | ✅ Yes | ❌ **Never** | Provider support required |

---

## Top 10 Exam Traps

1. `terraform import` adds to state — **does not generate config**
2. `sensitive = true` hides output in CLI — **value is still in state file in plaintext**
3. `for_each` requires **map or set** — not a list (use `toset()`)
4. `terraform.tfvars` is auto-loaded — other `.tfvars` files need `-var-file`
5. `.terraform.lock.hcl` records **exact provider versions** — commit it to Git, never edit manually
6. `terraform init -upgrade` is needed to update provider versions within constraints
7. `local` values are referenced as `local.<name>` — not `locals.<name>`
8. `prevent_destroy` only protects while the lifecycle block exists — remove the block, then destroy is possible
9. `count` index shifts when you remove a middle item — `for_each` keys are stable
10. `depends_on` is a last resort — if you can reference an attribute, that creates an implicit dependency automatically

---

*Guide covers all 37 Terraform Associate 004 objectives. Verify current objectives at:*
*https://developer.hashicorp.com/terraform/tutorials/certification-004/associate-review-004*
