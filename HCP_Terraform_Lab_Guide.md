# HCP Terraform — Hands-On Lab Guide
**Terraform Associate 004 Retake Prep | July 30, 2026**

> **Prerequisites before starting any lab:**
> - HCP Terraform account (free tier at app.terraform.io)
> - Terraform CLI installed locally (1.5+)
> - AWS account with IAM credentials (free tier sufficient)
> - Git installed locally
> - `terraform login` completed successfully (Lab 1.0 covers this)

---

## Lab 1.0 — Authentication: `terraform login` and the `cloud` Block

**Objective:** Authenticate the Terraform CLI to HCP Terraform and understand the `cloud` block vs `backend` block.

**Syllabus:** 8d

---

### Step 1 — Run `terraform login`

```bash
terraform login
```

**Observe:**
- Terraform opens your browser to app.terraform.io
- You generate a user API token
- Token is stored locally at `~/.terraform.d/credentials.tfrc.json`
- CLI confirms: `Retrieved token for user <your-username>`

**Exam checkpoint:** `terraform login` stores credentials locally. The default hostname is `app.terraform.io`. You can run `terraform logout` to remove stored credentials.

---

### Step 2 — Understand the `cloud` block vs `backend "remote"` block

The `cloud` block is the **modern way** to connect to HCP Terraform (Terraform 1.1+). The `backend "remote"` block is the **legacy way**. Both work but the exam expects you to know the `cloud` block syntax.

**Cloud block (current — use this):**

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

**Backend remote block (legacy — know it exists but don't use it):**

```hcl
terraform {
  backend "remote" {
    organization = "my-org"
    workspaces {
      name = "my-workspace"
    }
  }
}
```

**Key differences:**

| Feature | `cloud` block | `backend "remote"` |
|---------|--------------|-------------------|
| Introduced | Terraform 1.1+ | Earlier versions |
| Status | Current recommended | Legacy |
| Workspace tags support | Yes | Limited |
| `terraform init` behavior | Same | Same |

**Exam checkpoint:** The `cloud` block replaced `backend "remote"` for HCP Terraform integration. Both connect your local config to HCP Terraform. The exam may show you both — know which is current.

---

### Step 3 — Workspace name vs tags in the `cloud` block

You can target a workspace by **name** (single workspace) or by **tags** (multiple workspaces):

```hcl
# Single workspace by name
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "production"
    }
  }
}
```

```hcl
# Multiple workspaces by tag
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      tags = ["app:web", "env:prod"]
    }
  }
}
```

**Exam checkpoint:** When using tags in the `cloud` block, you must run `terraform workspace select <name>` to choose which matching workspace to use. When using a specific name, no workspace selection is needed.

---

## Lab 1.1 — Execution Modes (Remote vs Local)

**Objective:** Observe the difference between remote and local execution mode behavior firsthand.

**Syllabus:** 8a, 8d

---

### Step 1 — Create a workspace in HCP Terraform

1. Log in to app.terraform.io
2. Go to your organization → **New Workspace**
3. Choose **CLI-driven workflow**
4. Name it `lab-execution-modes`
5. Leave execution mode as **Remote** (default)
6. Click **Create workspace**

---

### Step 2 — Create a minimal Terraform configuration

```bash
mkdir hcp-lab-execution && cd hcp-lab-execution
```

Create `main.tf`:

```hcl
terraform {
  cloud {
    organization = "<YOUR_ORG_NAME>"
    workspaces {
      name = "lab-execution-modes"
    }
  }
}

output "hello" {
  value = "Execution mode test"
}
```

---

### Step 3 — Initialize and run in Remote execution mode

```bash
terraform init
terraform plan
```

**Observe:** The plan output shows:
```
Running plan in HCP Terraform. Output will stream here...
```

The plan is executing on HCP Terraform's infrastructure, not your machine. Your terminal is just streaming the output.

---

### Step 4 — Switch to Local execution mode

1. In HCP Terraform UI → workspace `lab-execution-modes`
2. Go to **Settings → General**
3. Change **Execution Mode** to **Local**
4. Save settings

Run plan again:

```bash
terraform plan
```

**Observe:** Plan now runs locally on your machine. No "Running plan in HCP Terraform" message.

---

### Step 5 — Key observations

| Aspect | Remote Execution | Local Execution |
|--------|-----------------|-----------------|
| Where plan runs | HCP Terraform infrastructure | Your local machine |
| Where state is stored | HCP Terraform (remote) | HCP Terraform (remote) |
| Provider credentials needed | In workspace variables | In local environment |
| CLI output | Streamed from remote | Direct local output |

**Exam checkpoint:** Execution mode controls WHERE Terraform runs. State storage location does NOT change between execution modes — state always stays in HCP Terraform when the `cloud` block is configured.

---

## Lab 1.2 — Variable Precedence in HCP Terraform (Remote Execution)

**Objective:** Prove that `-var` flags are ignored in remote execution mode and workspace variables override variable sets.

**Syllabus:** 8c

---

### Step 1 — Reset workspace to Remote execution mode

1. Workspace Settings → General → Execution Mode → **Remote**
2. Save

---

### Step 2 — Update configuration to use a variable

Update `main.tf`:

```hcl
terraform {
  cloud {
    organization = "<YOUR_ORG_NAME>"
    workspaces {
      name = "lab-execution-modes"
    }
  }
}

variable "environment" {
  type    = string
  default = "default_value"
}

output "env_output" {
  value = var.environment
}
```

---

### Step 3 — Create a Variable Set

1. HCP Terraform → **Organization Settings → Variable Sets**
2. **Create variable set** → name it `lab-varset`
3. Add variable:
   - Key: `environment`
   - Value: `from_variable_set`
   - Type: Terraform variable
4. Scope: apply to workspace `lab-execution-modes`
5. Save

---

### Step 4 — Run plan and observe Variable Set value

```bash
terraform plan
```

**Observe output:**
```
env_output = "from_variable_set"
```

---

### Step 5 — Add a Workspace-Specific Variable

1. Workspace `lab-execution-modes` → **Variables tab**
2. Add Terraform variable:
   - Key: `environment`
   - Value: `from_workspace_variable`
3. Save

---

### Step 6 — Run plan and observe Workspace Variable wins

```bash
terraform plan
```

**Observe output:**
```
env_output = "from_workspace_variable"
```

Workspace variable overrides the variable set.

---

### Step 7 — Attempt to override with `-var` flag

```bash
terraform plan -var="environment=from_local_var_flag"
```

**Observe:** Output still shows `from_workspace_variable`. The `-var` flag is ignored in remote execution mode. HCP Terraform may display:
```
Warning: Ignored -var flag in remote operations
```

---

### Step 8 — Variable precedence summary

**This is NOT a simple precedence chain — it depends on execution mode.**

**Local execution mode — standard precedence (low → high):**

| Order | Source |
|-------|--------|
| 1 (lowest) | Default values in config |
| 2 | `TF_VAR_` environment variables |
| 3 | `terraform.tfvars` |
| 4 | `*.auto.tfvars` (alphabetical order) |
| 5 | `-var-file` flag |
| 6 (highest) | `-var` flag |

**Remote execution mode — completely different rules:**

| Order | Source |
|-------|--------|
| 1 (lowest) | Default values in config |
| 2 | Variable sets (scoped to workspace) |
| 3 (highest) | Workspace-specific variables |
| **IGNORED** | `-var`, `-var-file`, `TF_VAR_`, local `.tfvars` files |

> ⚠️ **EXAM NOTE:** In remote execution mode, `-var` is not just lowest priority — it does not participate in the precedence chain at all. It is completely ignored and never sent to HCP Terraform. This is not a tie-breaker situation — local flags are simply not part of remote execution variable resolution. The exam tests this distinction.

---

## Lab 1.3 — Run Pipeline (Remote Execution)

**Objective:** Walk through each stage of the HCP Terraform run pipeline in the correct order.

**Syllabus:** 8a, 8b

---

### Step 1 — Run pipeline order (memorize this)

The correct HCP Terraform run pipeline order is:

```
Pending
  ↓
Planning
  ↓
Cost Estimation  ← only if enabled (paid tiers)
  ↓
Policy Check     ← only if Sentinel/OPA policies configured
  ↓
Planned (awaiting confirmation if manual apply)
  ↓
Applying
  ↓
Applied
```

> ⚠️ **EXAM NOTE:** Cost estimation comes BEFORE policy check, not after. Many candidates get this order wrong. The exam tests this sequence directly.

---

### Step 2 — Trigger a plan and observe pipeline stages

```bash
terraform plan
```

Or in HCP Terraform UI: workspace → **Actions → Start new run → Plan only**

Click through each stage in the UI to see output and duration.

---

### Step 3 — Observe Auto Apply vs Manual Apply

**Manual apply (default):**
- Run reaches **Planned — Awaiting Confirmation**
- Waits indefinitely for a human to confirm or discard
- Unconfirmed plans are **discarded after 30 days** (not 24 hours)

**Auto apply:**
- Run proceeds from planned directly to applying without human confirmation
- Enable in: Workspace Settings → General → Auto Apply

**Exam checkpoint:** The exam may test the timeout on unconfirmed plans. It is **30 days**, not 24 hours.

---

### Step 4 — Observe a queued run

1. Trigger a plan from CLI: `terraform plan` (let it run)
2. While running, go to HCP Terraform UI → Actions → Start new run

**Observe:** Second run shows **Pending** status. It queues, it does not fail with a lock error.

**Exam checkpoint:** HCP Terraform queues concurrent runs automatically. It does NOT return a lock error like S3/DynamoDB backends do. The workspace is locked during a run but subsequent runs queue rather than fail.

---

### Step 5 — Discard a run

1. Trigger a plan with Auto Apply OFF
2. When plan completes: **Planned — Awaiting Confirmation**
3. Click **Discard run**

**Observe:** Run moves to **Discarded** state. Infrastructure unchanged. This run is logged in run history permanently.

---

### Step 6 — Run types

| Run type | When triggered | What it does |
|----------|---------------|-------------|
| Plan and Apply | Normal `terraform apply` | Full plan + apply |
| Plan only | `terraform plan` or speculative | Plan only, no apply |
| Destroy | `terraform destroy` | Destroys all resources |
| Refresh only | `terraform apply -refresh-only` | Updates state only |

**Exam checkpoint:** A **speculative plan** is a plan-only run triggered by a VCS pull request. It shows what would change but cannot be applied. It is read-only and does not lock the workspace.

---

## Lab 1.4 — Sentinel Policy Enforcement

**Objective:** Understand Sentinel enforcement levels and where policy checks sit in the run pipeline.

**Syllabus:** 8b

> **Note:** Sentinel requires HCP Terraform Plus tier. If unavailable on free tier, study the concepts in Steps 1-5 and observe the UI behavior conceptually.

---

### Step 1 — Sentinel enforcement levels (memorize these)

| Enforcement Level | Policy fails | What happens | Can override? |
|-----------------|-------------|-------------|--------------|
| Advisory | Yes | Warning logged, run **proceeds** to apply | N/A — doesn't block |
| Soft Mandatory | Yes | Run **blocked**, authorized user can override | Yes |
| Hard Mandatory | Yes | Run **blocked**, NO override possible ever | No |

---

### Step 2 — Create a policy set in HCP Terraform

1. Organization Settings → **Policy Sets**
2. **Connect a new policy set**
3. Choose **No VCS connection** (manual for lab)
4. Name it `lab-enforce-tags`
5. Scope to workspace `lab-execution-modes`

---

### Step 3 — Write a Sentinel policy

Create file `enforce-tags.sentinel` locally:

```hcl
# Sentinel uses its own language — NOT Python, NOT HCL
# Comments use # but syntax is Sentinel-specific

import "tfplan/v2" as tfplan

# Require all EC2 instances to have an Environment tag
main = rule {
  all tfplan.resource_changes as _, rc {
    rc.type is not "aws_instance" or
    rc.change.after.tags contains "Environment"
  }
}
```

---

### Step 4 — Upload policy and set enforcement level

1. Upload `enforce-tags.sentinel` to the policy set
2. Set enforcement level to **Soft Mandatory**
3. Save

---

### Step 5 — Add a non-compliant resource and observe failure

Add to `main.tf`:

```hcl
resource "aws_instance" "test" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  # No tags — violates the policy
}
```

```bash
terraform plan
```

**Observe in HCP Terraform UI:**
- Plan completes ✓
- Cost Estimation (if enabled) completes ✓
- Policy Check **FAILED** ✗
- Run is blocked — apply button is greyed out
- Since Soft Mandatory: an admin can click **Override and Continue**

---

### Step 6 — Fix the violation

```hcl
resource "aws_instance" "test" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Environment = "lab"
  }
}
```

Run plan again — policy check passes, run proceeds.

**Exam checkpoint:** Sentinel sits AFTER cost estimation and BEFORE apply. It is a Plus/Enterprise feature — not available on Free tier. OPA (Open Policy Agent) is the alternative policy framework also supported by HCP Terraform.

---

## Lab 1.5 — Variable Sets (Global vs Scoped)

**Objective:** Understand variable set scoping and the correct pattern for managing provider credentials.

**Syllabus:** 8c

---

### Step 1 — Create a second workspace

1. New workspace → CLI-driven → name it `lab-varset-test`
2. Remote execution mode

---

### Step 2 — Create a global variable set for AWS credentials

1. Organization Settings → Variable Sets → **Create variable set**
2. Name: `global-aws-credentials`
3. Add as **environment variables** (not Terraform variables):
   - Key: `AWS_ACCESS_KEY_ID` — Value: your access key
   - Key: `AWS_SECRET_ACCESS_KEY` — Value: your secret key — mark as **Sensitive**
4. Scope: **Apply to all workspaces in the organization**
5. Save

**Observe:** All workspaces now have AWS credentials available without configuring them per workspace. This is the correct pattern.

---

### Step 3 — Create a scoped variable set

1. New variable set → name `environment-config`
2. Add Terraform variable: `environment = from_variable_set`
3. Scope: apply only to `lab-execution-modes` workspace
4. Save

---

### Step 4 — Verify scoping

In `lab-varset-test` — the `environment` variable from the scoped set is NOT available (not in scope). Only the global AWS credentials variable set applies.

In `lab-execution-modes` — both the global credentials set AND the scoped environment-config set apply.

**Exam checkpoint:** Variable sets can be scoped globally (all workspaces) or to specific workspaces/projects. A workspace-specific variable always overrides a variable set value for the same key.

---

## Lab 1.6 — Projects and Workspace Organization

**Objective:** Create a project, assign workspaces to it, and understand how projects organize governance.

**Syllabus:** 8b, 8c

---

### Step 1 — Create a project

1. HCP Terraform → **Projects** (top nav or sidebar)
2. **New Project**
3. Name it `lab-project`
4. Save

---

### Step 2 — Assign workspaces to the project

1. Go to workspace `lab-execution-modes` → Settings → General
2. Under **Project**, select `lab-project`
3. Save
4. Repeat for `lab-varset-test`

---

### Step 3 — Apply a variable set to a project

1. Organization Settings → Variable Sets → `environment-config`
2. Edit scope → change from individual workspace to **project: lab-project**
3. Save

**Observe:** The variable set now automatically applies to ALL workspaces inside `lab-project` — both existing and future ones.

---

### Step 4 — Key concepts

| Concept | Scope | Purpose |
|---------|-------|---------|
| Organization | Top level | All workspaces, all projects |
| Project | Groups workspaces | Governance boundary, apply policies/varsets at project level |
| Workspace | Single config | Individual Terraform state and run history |

**Exam checkpoint:** Projects are a governance and organizational layer above workspaces. You can apply variable sets and policy sets at the project level, affecting all workspaces within it. This is new in the 004 exam compared to 003.

---

## Lab 1.7 — Run Triggers (Cross-Workspace Dependencies)

**Objective:** Configure a run trigger so one workspace automatically triggers another workspace's run after a successful apply.

**Syllabus:** 8c

---

### Step 1 — Understand run triggers

Run triggers solve this problem: Workspace B depends on outputs from Workspace A. When Workspace A applies successfully, Workspace B should automatically plan/apply to consume the new outputs.

Without run triggers: you manually trigger Workspace B every time Workspace A changes.
With run triggers: it's automatic.

---

### Step 2 — Configure a run trigger

1. Go to workspace `lab-varset-test` (this is the downstream workspace)
2. Settings → **Run Triggers**
3. **Add another workspace**
4. Select `lab-execution-modes` as the source workspace
5. Save

---

### Step 3 — Observe the trigger

1. Go to workspace `lab-execution-modes`
2. Trigger a plan and apply
3. After successful apply, go to `lab-varset-test`

**Observe:** `lab-varset-test` automatically queues a new run triggered by the upstream workspace completing.

---

### Step 4 — Key facts

- Run triggers are workspace-to-workspace, not resource-to-resource
- The downstream workspace must be in **VCS or CLI-driven** workflow
- Multiple source workspaces can trigger one downstream workspace
- Run triggers only fire on **successful applies**, not on plan-only runs or failed runs

**Exam checkpoint:** Run triggers are for cross-workspace orchestration. They replace manual coordination between dependent workspaces. Know that they fire on successful apply only.

---

## Lab 1.8 — VCS-Driven Workflow

**Objective:** Connect a workspace to a GitHub repository and observe VCS-triggered runs.

**Syllabus:** 8a, 8d

---

### Step 1 — Create a GitHub repository

1. Create a new GitHub repo: `terraform-hcp-lab`
2. Push your current `main.tf` to it

---

### Step 2 — Connect HCP Terraform to GitHub

1. Organization Settings → **Version Control → Providers**
2. **Add a VCS provider** → GitHub
3. Follow OAuth flow to authorize HCP Terraform
4. Save

---

### Step 3 — Create a VCS-driven workspace

1. New workspace → **Version Control Workflow**
2. Select your GitHub connection
3. Select repo `terraform-hcp-lab`
4. Name workspace `lab-vcs-driven`
5. Set Terraform working directory if needed (leave blank if `main.tf` is at root)
6. Save

---

### Step 4 — Trigger a run via commit

1. Make a change to `main.tf` in your GitHub repo (e.g., change the output value)
2. Commit and push to the default branch

**Observe in HCP Terraform UI:**
- A new run is automatically queued in `lab-vcs-driven`
- No CLI command was run — the VCS push triggered it

---

### Step 5 — Observe a speculative plan on a Pull Request

1. Create a branch: `git checkout -b test-branch`
2. Make a small change to `main.tf`
3. Push the branch and open a Pull Request in GitHub

**Observe:**
- HCP Terraform runs a **speculative plan** automatically
- The plan result appears as a status check on the PR in GitHub
- This plan cannot be applied — it is read-only and informational only
- It does NOT lock the workspace

---

### Step 6 — VCS workflow key facts

| Trigger | What happens |
|---------|-------------|
| Push to default branch | Full plan + apply (or awaiting confirmation if manual apply) |
| Pull request opened/updated | Speculative plan only — cannot apply |
| Manual trigger in UI | Full plan + apply |
| `terraform apply` from CLI | Works but discouraged — VCS is the source of truth |

**Exam checkpoint:** In VCS-driven workflow, the VCS repo is the source of truth. Direct CLI applies are possible but go against the intended workflow. Speculative plans on PRs are read-only and informational.

---

## Lab 1.9 — State Migration to HCP Terraform

**Objective:** Migrate a local backend state file to HCP Terraform.

**Syllabus:** 8d

---

### Step 1 — Create a local backend configuration

In a new directory:

```bash
mkdir hcp-migration-lab && cd hcp-migration-lab
```

Create `main.tf` with a local backend:

```hcl
terraform {
  # No backend block = local backend by default
}

output "migration_test" {
  value = "this is running locally"
}
```

```bash
terraform init
terraform apply -auto-approve
```

**Observe:** `terraform.tfstate` is created locally.

---

### Step 2 — Create a destination workspace in HCP Terraform

1. HCP Terraform → New workspace → CLI-driven
2. Name it `lab-migration-target`
3. Remote execution mode

---

### Step 3 — Add the `cloud` block and re-init

Update `main.tf`:

```hcl
terraform {
  cloud {
    organization = "<YOUR_ORG_NAME>"
    workspaces {
      name = "lab-migration-target"
    }
  }
}

output "migration_test" {
  value = "this is now running in HCP Terraform"
}
```

```bash
terraform init
```

**Observe:** Terraform detects the existing local state and asks:
```
Do you wish to proceed? (yes/no)
```

Type `yes`.

---

### Step 4 — Observe state migration

```bash
terraform state list
```

**Observe:** The outputs are now managed in HCP Terraform. The local `terraform.tfstate` file is now empty (or backed up as `terraform.tfstate.backup`).

Go to HCP Terraform UI → workspace `lab-migration-target` → **States tab** — the state is there.

---

### Step 5 — Key facts

- `terraform init` handles state migration automatically when you add the `cloud` block
- Local state is NOT deleted — it becomes a backup
- You cannot migrate back to local without manually editing backend config and running `terraform init -migrate-state`

**Exam checkpoint:** Adding the `cloud` block to an existing config with local state and running `terraform init` prompts for state migration. It does not happen silently.

---

## Lab 1.10 — Private Registry

**Objective:** Understand the HCP Terraform Private Registry for modules and providers.

**Syllabus:** 8b

> This lab is conceptual — publishing to the private registry requires a GitHub connection and a correctly structured module repo.

---

### Step 1 — What the Private Registry does

The HCP Terraform Private Registry lets your organization:
- Publish internal Terraform modules for reuse across teams
- Host private providers not available on the public registry
- Version and document internal modules the same way the public registry does

---

### Step 2 — Module requirements for the Private Registry

To publish a module to the private registry your repo must follow this naming convention:

```
terraform-<PROVIDER>-<MODULE_NAME>
```

Examples:
- `terraform-aws-networking`
- `terraform-gcp-database`
- `terraform-azurerm-compute`

The repo must have this structure:

```
terraform-aws-networking/
├── main.tf
├── variables.tf
├── outputs.tf
└── README.md
```

---

### Step 3 — Consuming a private registry module

```hcl
module "networking" {
  source  = "app.terraform.io/<ORG_NAME>/networking/aws"
  version = "1.0.0"
}
```

**Key difference from public registry:**
- Public registry source: `hashicorp/consul/aws`
- Private registry source: `app.terraform.io/<ORG>/module/provider`

---

### Step 4 — Key facts

- Private registry modules support versioning via Git tags
- Modules are scoped to the organization — not visible outside it
- `terraform login` must be completed before you can pull private registry modules
- Private providers can also be hosted — not just modules

**Exam checkpoint:** Private registry module source format uses `app.terraform.io/<org>/<module>/<provider>`. This is different from the public Terraform Registry format. The naming convention `terraform-<provider>-<name>` is required for VCS-connected modules.

---

## Lab 1.11 — Teams and Permissions

**Objective:** Understand HCP Terraform team structure and workspace permission levels.

**Syllabus:** 8b

---

### Step 1 — Team structure in HCP Terraform

HCP Terraform uses Teams for access control:

| Team | Default behavior |
|------|-----------------|
| owners | Full organization access — cannot be deleted |
| (custom teams) | You define name and permissions |

---

### Step 2 — Workspace permission levels

| Permission Level | What they can do |
|----------------|-----------------|
| Read | View runs, state, variables |
| Plan | Trigger plans, cannot apply |
| Write | Trigger and apply runs, manage variables |
| Admin | Full workspace control including settings and deletion |

---

### Step 3 — Create a team and assign workspace access

1. Organization Settings → **Teams → Create team**
2. Name it `lab-developers`
3. Go to workspace `lab-execution-modes` → Settings → **Team Access**
4. Add team `lab-developers` with **Write** permission
5. Save

---

### Step 4 — Key facts

- Teams are an **organization-level** construct
- Workspace permissions are assigned **per workspace per team**
- The `owners` team has implicit admin access everywhere and cannot be restricted
- API tokens can be scoped to a team — useful for CI/CD pipelines

**Exam checkpoint:** Know the four workspace permission levels: Read, Plan, Write, Admin. Teams vs individual users — HCP Terraform recommends team-based access over individual user access for scalability. Teams are a paid tier feature (not available on Free beyond the owners team).

---

## Lab 1.12 — Destroy Protection and `prevent_destroy`

**Objective:** Distinguish between workspace-level destroy protection and resource-level `prevent_destroy`.

**Syllabus:** 8a

---

### Step 1 — Enable workspace destroy protection

1. Workspace `lab-execution-modes` → Settings → General
2. Enable **Destroy Protection**
3. Save

---

### Step 2 — Attempt terraform destroy

```bash
terraform destroy
```

**Observe:** HCP Terraform blocks the destroy at the workspace level. You must explicitly disable destroy protection in settings before a destroy can run.

---

### Step 3 — Resource-level `prevent_destroy`

Add to a resource in `main.tf`:

```hcl
resource "aws_instance" "test" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  lifecycle {
    prevent_destroy = true
  }
}
```

```bash
terraform destroy
```

**Observe:** Plan fails with:
```
Error: Instance cannot be destroyed
  on main.tf line X:
  Resource aws_instance.test has lifecycle.prevent_destroy set...
```

---

### Step 4 — Key difference

| Mechanism | Scope | Where enforced | Who can bypass |
|-----------|-------|---------------|----------------|
| `prevent_destroy` | Single resource | Terraform plan/apply | Remove from config |
| Workspace destroy protection | Entire workspace | HCP Terraform UI | Disable in workspace settings |

**Exam checkpoint:** `prevent_destroy = true` blocks destroy at the plan level — Terraform refuses to create a plan that would destroy that resource. Workspace destroy protection is a UI-level guard in HCP Terraform, not a Terraform language feature.

---

## Lab 1.13 — `import` Block in Remote Execution

**Objective:** Import a manually created resource using the declarative `import` block in remote execution mode.

**Syllabus:** 8a, 8d

---

### Step 1 — Manually create a resource outside Terraform

```bash
aws s3 mb s3://my-lab-import-bucket-<unique-suffix>
```

---

### Step 2 — Write the resource block and import block

```hcl
resource "aws_s3_bucket" "imported" {
  bucket = "my-lab-import-bucket-<unique-suffix>"
}

import {
  to = aws_s3_bucket.imported
  id = "my-lab-import-bucket-<unique-suffix>"
}
```

---

### Step 3 — Run plan in remote execution mode

```bash
terraform plan
```

**Observe in HCP Terraform UI:**
- Plan shows: `aws_s3_bucket.imported will be imported`
- No destroy or create actions — import only
- The plan executes remotely on HCP Terraform infrastructure

---

### Step 4 — Apply to complete the import

```bash
terraform apply
```

Resource is now tracked in HCP Terraform state.

---

### Step 5 — Remove the import block after successful apply

```hcl
# REMOVE this block after a successful apply — it is consumed
# Leaving it in causes a no-op on next plan but is unnecessary
# import {
#   to = aws_s3_bucket.imported
#   id = "my-lab-import-bucket-<unique-suffix>"
# }
```

Run `terraform plan` — should show no changes.

**Exam checkpoint:** The `import` block is NOT a permanent fixture. After a successful apply it has done its job. Remove it. Leaving it in is harmless but incorrect practice. The `import` block was introduced in Terraform 1.5. Prior to 1.5, only the `terraform import` CLI command existed.

---

## Lab 1.14 — State Lock Behavior: HCP Terraform vs S3/DynamoDB

**Objective:** Contrast concurrent run handling between HCP Terraform and S3/DynamoDB backends.

**Syllabus:** 8a

---

### Part A — HCP Terraform queues concurrent runs

1. Trigger `terraform plan` from CLI
2. While running, trigger another run from UI: Actions → Start new run

**Observe:** Second run shows **Pending**. It queues. It does NOT error.

---

### Part B — S3/DynamoDB fails with lock error

If you have a separate S3 backend configured:

```bash
# Terminal 1
terraform apply

# Terminal 2 — while apply is running
terraform plan
```

**Observe in Terminal 2:**
```
Error: Error acquiring the state lock
Lock Info:
  ID: <lock-id>
  ...
```

To release:

```bash
terraform force-unlock <LOCK_ID>
```

---

### Key contrast

| Aspect | HCP Terraform | S3 + DynamoDB |
|--------|--------------|--------------|
| Concurrent run behavior | Queued automatically | Fails with lock error |
| Lock release | Automatic after run completes | Manual `force-unlock` if stuck |
| Lock visibility | Run history in UI | DynamoDB table |

**Exam checkpoint:** HCP Terraform queues runs — it does not fail on concurrent access. S3/DynamoDB uses DynamoDB for locking and returns an error if a lock cannot be acquired. `terraform force-unlock` is the manual escape for stuck S3 locks.

---

## Syllabus Cross-Check

Use this to verify every 8x objective is covered before your exam:

| Objective | Lab | Status |
|-----------|-----|--------|
| 8a — Use HCP Terraform to create infrastructure | 1.0, 1.1, 1.3, 1.12, 1.13, 1.14 | ✓ |
| 8b — Collaboration and governance features | 1.4, 1.5, 1.10, 1.11 | ✓ |
| 8c — Workspaces, projects, run triggers | 1.2, 1.5, 1.6, 1.7 | ✓ |
| 8d — CLI integration, cloud block, migration | 1.0, 1.1, 1.8, 1.9, 1.13 | ✓ |

---

## Lab Summary

| Lab | Key concept |
|-----|------------|
| 1.0 | `terraform login`, `cloud` block vs `backend "remote"`, workspace name vs tags |
| 1.1 | Remote vs local execution — where plan runs, state location unchanged |
| 1.2 | Variable precedence — workspace var > variable set > `-var` flag ignored |
| 1.3 | Run pipeline order — Plan → Cost Estimation → Policy Check → Apply |
| 1.4 | Sentinel — advisory / soft mandatory / hard mandatory enforcement levels |
| 1.5 | Variable sets — global vs scoped, workspace var always wins |
| 1.6 | Projects — governance layer above workspaces, project-scoped varsets |
| 1.7 | Run triggers — cross-workspace orchestration, fires on successful apply only |
| 1.8 | VCS workflow — push triggers run, PR triggers speculative plan only |
| 1.9 | State migration — `terraform init` prompts migration when `cloud` block added |
| 1.10 | Private registry — module naming convention, source format |
| 1.11 | Teams and permissions — Read / Plan / Write / Admin levels |
| 1.12 | Destroy protection vs `prevent_destroy` — workspace vs resource scope |
| 1.13 | `import` block — declarative import in remote pipeline, remove after apply |
| 1.14 | Concurrent runs — HCP queues, S3/DynamoDB returns lock error |

---

## Cleanup

```bash
terraform destroy
aws s3 rb s3://my-lab-import-bucket-<unique-suffix> --force
```

Delete workspaces in HCP Terraform UI when done.

---

*Lab guide built for Terraform Associate 004 retake — July 30, 2026*
*Pair with: Terraform_004_CLI_Reference.md*
