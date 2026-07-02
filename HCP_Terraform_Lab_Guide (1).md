# HCP Terraform — Hands-On Lab Guide
**Terraform Associate 004 Retake Prep | July 30, 2026**

> **Prerequisites before starting any lab:**
> - HCP Terraform account at app.terraform.io
> - Terraform CLI installed locally (1.5+)
> - Git installed locally
> - `terraform login` completed successfully (Lab 1.0 covers this)

---

## Table of Contents

- [Lab 1.0 — Authentication: `terraform login` and the `cloud` Block](#lab-10--authentication-terraform-login-and-the-cloud-block)
- [Lab 1.1 — Execution Modes (Remote vs Local)](#lab-11--execution-modes-remote-vs-local)
- [Lab 1.2 — Variable Precedence in HCP Terraform](#lab-12--variable-precedence-in-hcp-terraform)
- [Lab 1.3 — Run Pipeline (Remote Execution)](#lab-13--run-pipeline-remote-execution)
- [Lab 1.4 — Sentinel Policy Enforcement](#lab-14--sentinel-policy-enforcement)
- [Lab 1.5 — Variable Sets (Global vs Scoped)](#lab-15--variable-sets-global-vs-scoped)
- [Lab 1.6 — Projects and Workspace Organization](#lab-16--projects-and-workspace-organization)
- [Lab 1.7 — Run Triggers (Cross-Workspace Dependencies)](#lab-17--run-triggers-cross-workspace-dependencies)
- [Lab 1.8 — VCS-Driven Workflow](#lab-18--vcs-driven-workflow)
- [Lab 1.9 — State Migration to HCP Terraform](#lab-19--state-migration-to-hcp-terraform)
- [Lab 1.10 — Private Registry](#lab-110--private-registry)
- [Lab 1.11 — Teams and Permissions](#lab-111--teams-and-permissions)
- [Lab 1.12 — Health Assessments (Drift Detection & Continuous Validation)](#lab-112--health-assessments-drift-detection--continuous-validation)
- [Syllabus Cross-Check](#syllabus-cross-check)
- [Lab Summary](#lab-summary)

---

## Lab 1.0 — Authentication: `terraform login` and the `cloud` Block

**Objective:** Authenticate the Terraform CLI to HCP Terraform and connect a local config to an HCP Terraform workspace.

**Syllabus:** 8d

---

### Step 1 — Run `terraform login`

```bash
terraform login
```

**What happens:**
- Terraform opens your browser to app.terraform.io
- You generate a user API token
- Token is stored locally at `~/.terraform.d/credentials.tfrc.json` (Unix/Mac) or `%APPDATA%\terraform.d\credentials.tfrc.json` (Windows)
- CLI confirms: `Retrieved token for user <your-username>`

To remove stored credentials:
```bash
terraform logout
```

> ⚠️ **EXAM NOTE:** `terraform login` stores credentials locally. Default hostname is `app.terraform.io`. `terraform logout` removes them. The token file location is `~/.terraform.d/credentials.tfrc.json` by default — this can be overridden with the `TF_CLI_CONFIG_FILE` environment variable.

---

### Step 2 — The `cloud` block vs `backend "remote"` block

The `cloud` block is the **current way** to connect to HCP Terraform (Terraform 1.1+). The `backend "remote"` block is the **legacy way**. The exam may show you both — know which is current.

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

**Backend remote block (legacy):**

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

| Feature | `cloud` block | `backend "remote"` |
|---------|--------------|-------------------|
| Introduced | Terraform 1.1+ | Earlier versions |
| Status | Current recommended | Legacy |
| Workspace tags support | Yes | Limited |

---

### Step 3 — Workspace targeting: name vs tags

**Single workspace by name:**

```hcl
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "production"
    }
  }
}
```

**Multiple workspaces by tag:**

```hcl
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      tags = ["app:web", "env:prod"]
    }
  }
}
```

> ⚠️ **EXAM NOTE:** When using tags, you must run `terraform workspace select <name>` to choose which matching workspace to use. When using a specific name, no workspace selection is needed.

---

## Lab 1.1 — Execution Modes (Remote vs Local)

**Objective:** Observe the difference between Remote and Local execution modes in HCP Terraform.

**Syllabus:** 8a, 8d

---

### Step 1 — Create a CLI-driven workspace

1. Log in to app.terraform.io
2. Go to your organization → **New Workspace**
3. Choose **CLI-driven workflow**
4. Name it `lab-execution-modes`
5. Leave execution mode as default
6. Click **Create workspace**

---

### Step 2 — Create a minimal configuration

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

```bash
terraform init
```

---

### Step 3 — Run in Remote execution mode

In workspace **Settings** → scroll to **Execution mode** → confirm **Remote (custom)** is selected → Save.

```bash
terraform plan
```

**What you observe:** Terminal shows:
```
Running plan in HCP Terraform. Output will stream here...
```

The plan is executing on HCP Terraform's infrastructure. Your terminal is only streaming the output.

---

### Step 4 — Switch to Local execution mode

1. Workspace **Settings** → scroll to **Execution mode**
2. Select **Local (custom)**
3. Click **Save settings**

```bash
terraform plan
```

**What you observe:** No streaming message. Plan runs immediately on your machine.

---

### Step 5 — Key comparison

| Aspect | Remote (custom) | Local (custom) |
|--------|----------------|----------------|
| Where plan/apply runs | HCP Terraform infrastructure | Your local machine |
| Where state is stored | HCP Terraform | HCP Terraform |
| Provider credentials | Must be in workspace variables | Local environment |
| Terminal output | Streamed from remote | Direct local output |

> ⚠️ **EXAM NOTE:** Execution mode controls WHERE Terraform runs — not where state is stored. State always stays in HCP Terraform when the `cloud` block is configured, regardless of execution mode.

Switch execution mode back to **Remote (custom)** before proceeding to Lab 1.2.

---

## Lab 1.2 — Variable Precedence in HCP Terraform

**Objective:** Prove that workspace variables override variable sets, and that `-var` flags are ignored in remote execution mode.

**Syllabus:** 8c

---

### Step 1 — Update configuration to use a variable

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

### Step 2 — Create a Variable Set

1. HCP Terraform → **Organization Settings → Variable Sets**
2. Click **Create variable set** → name it `lab-varset`
3. Add variable:
   - Key: `environment`
   - Value: `from_variable_set`
   - Type: **Terraform variable**
4. Scope: apply to workspace `lab-execution-modes`
5. Save

---

### Step 3 — Run plan and observe Variable Set value

```bash
terraform plan
```

**Observe output:**
```
env_output = "from_variable_set"
```

Variable set value is being used since no workspace variable is set.

---

### Step 4 — Add a Workspace-Specific Variable

1. Workspace `lab-execution-modes` → left sidebar → **Variables**
2. Under **Terraform Variables** → **Add variable**:
   - Key: `environment`
   - Value: `from_workspace_variable`
3. Save

---

### Step 5 — Run plan and observe Workspace Variable wins

```bash
terraform plan
```

**Observe output:**
```
env_output = "from_workspace_variable"
```

Workspace variable overrides the variable set.

---

### Step 6 — Attempt to override with `-var` flag

```bash
terraform plan -var="environment=from_local_flag"
```

**Observe:** Output still shows `from_workspace_variable`. HCP Terraform may display:
```
Warning: Ignored -var flag in remote operations
```

The `-var` flag is completely ignored — it does not participate in the precedence chain in remote execution mode.

---

### Step 7 — Variable precedence summary

**Remote execution mode — HCP Terraform rules (the only ones that matter here):**

| Order | Source |
|-------|--------|
| 1 (lowest) | Default values in config |
| 2 | Variable sets (scoped to workspace) |
| 3 (highest) | Workspace-specific variables |
| **IGNORED** | `-var`, `-var-file`, `TF_VAR_`, local `.tfvars` files |

> ⚠️ **EXAM NOTE:** In remote execution mode, `-var` is not just lowest priority — it does not participate at all. It is completely ignored and never sent to HCP Terraform. Local flags and files are not part of remote execution variable resolution.

---

## Lab 1.3 — Run Pipeline (Remote Execution)

**Objective:** Trigger runs from the UI and CLI, observe pipeline stages, queuing behavior, and the difference between auto-apply and manual apply.

**Syllabus:** 8a, 8b

---

### Step 1 — Run pipeline order (burn this in before touching anything)

```
Pending
  ↓
Planning
  ↓
Cost Estimation  ← after planning, before policy check
  ↓
Policy Check     ← only if Sentinel/OPA configured
  ↓
Planned and finished  ← plan-only run stops here
  ↓
Apply (if plan and apply run)
  ↓
Applied
```

> ⚠️ **EXAM NOTE:** Cost estimation comes BEFORE policy check — do not invert these. The exam tests this order directly.
>
> Cost estimation has been removed from the HCP Terraform product as of 2026. However the 004 exam content was written before this change and still tests cost estimation as part of the run pipeline. Answer what the exam expects, not what the product does today. For the exam: cost estimation exists and sits between planning and policy check.

---

### Step 2 — Verify workspace settings

1. Left sidebar → **Settings**
2. Scroll to **Execution mode** → confirm **Remote (custom)** is selected
3. Scroll to **Auto-apply** → confirm BOTH checkboxes are **unchecked**:
   - **Auto-apply API, UI, & VCS runs**
   - **Auto-apply run triggers**
4. Click **Save settings** if you changed anything

---

### Step 3 — Trigger a plan-only run from the CLI

```bash
cd hcp-lab-execution
terraform plan
```

**What happens:** `terraform plan` in CLI-driven workflow triggers a **speculative plan** — plan-only, runs on HCP Terraform infrastructure, streams output to your terminal. Cannot be applied.

**Immediately open the browser** → workspace `lab-execution-modes` → **Runs** (left sidebar)

**What you see in the Run List:**
- New entry at the top: **Triggered via CLI**
- Type: **plan-only run**
- Status badge: starts as **Running** → changes to **Planned and finished**
- Filter tabs at the top: All | Needs Attention | Errored | Running | On Hold | Success

Click on the run to open the detail page.

**What you see on the run detail page:**
- **Plan duration** — how long the plan took
- **Resources to be changed** — +0 -0 ~0 breakdown
- **Run details** section — who triggered it and how
- **Plan finished** section — full plan output
- **Outputs** section — output values
- **Download Sentinel mocks** button
- Banner at the bottom: **"This run was started from a speculative plan and cannot be applied"**
- **Retry run** button and **Add comment** box

> ⚠️ **EXAM NOTE:** `terraform plan` from CLI = speculative plan = cannot be applied = does NOT lock the workspace. To trigger a full plan+apply run use `terraform apply` from CLI or **+ New run** from the UI.

---

### Step 4 — Trigger a full Plan and Apply run from the UI

1. In the Run List page click the blue **+ New run** button (top right)
2. Modal appears: **Start a new run**
3. **Run name** — leave as default or type a name
4. **Run Type** dropdown — leave as **Plan and apply (standard)**
5. Click **Additional planning options** to see extra flags available
6. Click **Start**

**What you see in the Run List:**
- New run at the top with status **Running**

Click on the run and watch it progress through stages in real time.

---

### Step 5 — Observe the run waiting for confirmation

Since Auto-apply is disabled, the run completes planning and stops.

**What you see on the run detail page:**
- Plan output showing what will change
- Confirmation section at the bottom with:
  - **Confirm & Apply** button
  - **Discard run** button
  - Optional comment box

**Do NOT click Confirm & Apply yet.**

---

### Step 6 — Trigger a second run and observe queuing

Click **+ New run** again → **Plan and apply (standard)** → **Start**

**What you see in the Run List:**
- First run: **On Hold** — waiting for confirmation
- Second run: **Running** — plan-only and planning stages do NOT block the queue

Go back to the first run → click **Discard run** at the bottom.

**What you observe:**
- First run moves to discarded state — remains in run history permanently
- Infrastructure NOT changed
- Second run continues independently

> ⚠️ **EXAM NOTE:** Plan-only runs and the planning stage of saved plan runs do NOT block the run queue. Full plan+apply runs awaiting confirmation hold their place but do not block other runs from planning.

---

### Step 7 — Confirm and apply the second run

1. Click on the second run in the Run List
2. Wait for planning to complete
3. At the bottom click **Confirm & Apply**
4. Optionally add a comment
5. Click **Confirm Plan**

**What you observe:**
- Run progresses through Apply
- Apply output streams in the UI
- Run status changes to **Success** in the Run List

---

### Step 8 — Enable Auto-apply and observe the difference

1. Left sidebar → **Settings**
2. Scroll to **Auto-apply**
3. Check **Auto-apply API, UI, & VCS runs**
4. Click **Save settings**

Trigger: **+ New run** → **Plan and apply (standard)** → **Start**

**What you observe:**
- Run goes through planning and applies automatically — no confirmation step
- No **Confirm & Apply** button appears
- Run completes end to end without human action

Go back to Settings → uncheck **Auto-apply API, UI, & VCS runs** → **Save settings** when done.

> ⚠️ **EXAM NOTE:** There are TWO separate auto-apply checkboxes:
> - **Auto-apply API, UI, & VCS runs** — covers CLI, UI, API, and VCS-triggered runs
> - **Auto-apply run triggers** — covers runs triggered by upstream workspace run triggers
> These can be enabled independently.

---

### Step 9 — Run types reference

| Run type | How to trigger | Can apply? | Blocks queue? |
|----------|---------------|-----------|--------------|
| Plan and apply | `terraform apply` from CLI or + New run → Plan and apply | Yes | Yes |
| Plan only (speculative) | `terraform plan` from CLI | No | No |
| Plan only (UI) | + New run → Plan only | No | No |
| Destroy | + New run → Additional planning options → Destroy | Yes | Yes |
| Refresh only | + New run → Additional planning options → Refresh only | Yes (state only) | Yes |

---

### Step 10 — Workspace destroy protection

1. Workspace **Settings** → scroll to find **Destroy Protection** toggle
2. Enable it → **Save settings**

```bash
terraform destroy
```

**What you observe:** HCP Terraform blocks the destroy at the workspace level. You must explicitly disable destroy protection in settings before a destroy can proceed.

> ⚠️ **EXAM NOTE:** Workspace destroy protection is an HCP Terraform UI setting that blocks destroy runs for the entire workspace. It has nothing to do with the Terraform `prevent_destroy` lifecycle argument — that is a Terraform configuration language feature, not an HCP Terraform feature.

---

## Lab 1.4 — Sentinel Policy Enforcement

**Objective:** Understand Sentinel enforcement levels and where policy checks sit in the run pipeline.

**Syllabus:** 8b

> **Note:** Sentinel requires HCP Terraform Plus tier or trial. If unavailable, study the concepts — you will still be tested on them.

---

### Step 1 — Sentinel enforcement levels (memorize these)

| Enforcement Level | Policy fails | What happens | Can override? |
|-----------------|-------------|-------------|--------------|
| Advisory | Yes | Warning logged, run **proceeds** to apply | N/A — does not block |
| Soft Mandatory | Yes | Run **blocked**, authorized user can override | Yes |
| Hard Mandatory | Yes | Run **blocked**, NO override possible | No |

---

### Step 2 — Create a policy set in HCP Terraform

1. Organization Settings → **Policy Sets**
2. Click **Connect a new policy set**
3. Choose **No VCS connection**
4. Name it `lab-enforce-tags`
5. Scope to workspace `lab-execution-modes`

---

### Step 3 — Write a Sentinel policy

Create file `enforce-tags.sentinel` locally:

```hcl
# Sentinel is its own language — NOT Python, NOT HCL
import "tfplan/v2" as tfplan

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

### Step 5 — Observe policy check in the run pipeline

Trigger a run: **+ New run** → **Plan and apply (standard)** → **Start**

**What you see in the run detail page:**
- Plan completes ✓
- Cost Estimation (if enabled) ✓
- Policy Check — **FAILED** ✗
- Run is blocked — Confirm & Apply is unavailable
- Since Soft Mandatory: an admin sees an **Override and Continue** option

---

### Step 6 — Key facts

- Sentinel sits AFTER cost estimation and BEFORE apply in the run pipeline
- Sentinel is a Plus/Enterprise feature
- OPA (Open Policy Agent) is the alternative policy framework also supported
- Policy sets can be scoped to specific workspaces, projects, or the entire organization

> ⚠️ **EXAM NOTE:** Enforcement level order matters — Advisory lets the run proceed with a warning, Soft Mandatory blocks but allows override, Hard Mandatory blocks with no override possible under any circumstance.

---

## Lab 1.5 — Variable Sets (Global vs Scoped)

**Objective:** Understand variable set scoping — global vs project vs workspace — and the correct pattern for managing provider credentials.

**Syllabus:** 8c

---

### Step 1 — Create a second workspace

1. New workspace → CLI-driven → name it `lab-varset-test`
2. Remote execution mode

---

### Step 2 — Create a global variable set for credentials

1. Organization Settings → Variable Sets → **Create variable set**
2. Name: `global-aws-credentials`
3. Add as **environment variables**:
   - Key: `AWS_ACCESS_KEY_ID` — Value: your access key
   - Key: `AWS_SECRET_ACCESS_KEY` — Value: your secret key → mark as **Sensitive**
4. Scope: **Apply to all workspaces in the organization**
5. Save

**Observe:** All workspaces now have credentials available without configuring per workspace.

---

### Step 3 — Create a scoped variable set

1. New variable set → name `environment-config`
2. Add Terraform variable: `environment = from_variable_set`
3. Scope: apply only to `lab-execution-modes` workspace
4. Save

---

### Step 4 — Verify scoping behavior

In `lab-varset-test`:
- The scoped `environment-config` variable set does NOT apply — not in scope
- Only the global credentials set applies

In `lab-execution-modes`:
- Both the global credentials set AND the scoped environment-config set apply

> ⚠️ **EXAM NOTE:** Variable sets can be scoped globally (all workspaces), to a project (all workspaces in the project), or to specific workspaces. A workspace-specific variable always overrides a variable set value for the same key. Sensitive variable values are write-only — they cannot be read back after being set.

---

## Lab 1.6 — Projects and Workspace Organization

**Objective:** Create a project, assign workspaces to it, and apply a variable set at the project level.

**Syllabus:** 8b, 8c

---

### Step 1 — Create a project

1. HCP Terraform → **Projects** (top nav or sidebar)
2. Click **New Project**
3. Name it `lab-project`
4. Save

---

### Step 2 — Assign workspaces to the project

1. Go to workspace `lab-execution-modes` → **Settings**
2. Under **Project** → select `lab-project`
3. Save
4. Repeat for `lab-varset-test`

---

### Step 3 — Apply a variable set at the project level

1. Organization Settings → Variable Sets → `environment-config`
2. Edit scope → change from individual workspace to **Project: lab-project**
3. Save

**Observe:** The variable set now automatically applies to ALL workspaces inside `lab-project` — including future ones added to the project.

---

### Step 4 — Key concepts

| Level | Scope | Use for |
|-------|-------|---------|
| Organization | All workspaces | Org-wide credentials, global policies |
| Project | All workspaces in project | Team or environment-specific governance |
| Workspace | Single workspace | Config-specific overrides |

> ⚠️ **EXAM NOTE:** Projects are a governance and organizational layer above workspaces. Variable sets and policy sets can be applied at the project level, affecting all workspaces within it. This is a 004-specific exam area — not tested in 003.

---

## Lab 1.7 — Run Triggers (Cross-Workspace Dependencies)

**Objective:** Configure a run trigger so one workspace automatically triggers another after a successful apply.

**Syllabus:** 8c

---

### Step 1 — What run triggers solve

Without run triggers: Workspace B depends on outputs from Workspace A. You manually trigger B every time A applies.

With run triggers: When A applies successfully, B automatically queues a new run.

---

### Step 2 — Configure a run trigger

1. Go to workspace `lab-varset-test` (downstream workspace)
2. Left sidebar → **Settings**
3. Look for **Run Triggers** in the settings menu
4. Add `lab-execution-modes` as the source workspace
5. Save

---

### Step 3 — Observe the trigger fire

1. Go to workspace `lab-execution-modes`
2. Trigger a **+ New run** → **Plan and apply (standard)** → **Start** → **Confirm & Apply**
3. After successful apply, go to workspace `lab-varset-test` → **Runs**

**What you observe:** `lab-varset-test` has automatically queued a new run triggered by the upstream workspace completing.

---

### Step 4 — Key facts

| Fact | Detail |
|------|--------|
| What triggers a downstream run? | Successful apply on the source workspace only |
| Does a failed apply trigger? | No |
| Does a plan-only run trigger? | No |
| Can multiple sources trigger one workspace? | Yes |
| Auto-apply run triggers setting | Controls whether triggered runs auto-apply or wait for confirmation |

> ⚠️ **EXAM NOTE:** Run triggers fire on **successful applies only** — not on failed runs, not on plan-only runs. The **Auto-apply run triggers** checkbox in workspace settings is separate from the general auto-apply setting.

---

## Lab 1.8 — VCS-Driven Workflow

**Objective:** Connect a workspace to a GitHub repository and observe VCS-triggered runs and speculative plans.

**Syllabus:** 8a, 8d

---

### Step 1 — Connect HCP Terraform to GitHub

1. Organization Settings → **Version Control → Providers**
2. Click **Add a VCS provider** → GitHub
3. Follow OAuth flow to authorize HCP Terraform
4. Save

---

### Step 2 — Create a VCS-driven workspace

1. New workspace → **Version Control Workflow**
2. Select your GitHub connection
3. Select your repository
4. Name workspace `lab-vcs-driven`
5. Leave working directory blank if `main.tf` is at root
6. Save

---

### Step 3 — Trigger a run via commit

1. Make a change to `main.tf` in your GitHub repo
2. Commit and push to the default branch

**What you observe in HCP Terraform:**
- A new run is automatically queued in `lab-vcs-driven`
- No CLI command was run — the VCS push triggered it
- Run is labeled as triggered via VCS

---

### Step 4 — Observe a speculative plan on a Pull Request

1. Create a branch: `git checkout -b test-branch`
2. Make a small change to `main.tf`
3. Push and open a Pull Request in GitHub

**What you observe:**
- HCP Terraform runs a **speculative plan** automatically
- Plan result appears as a status check on the PR in GitHub
- This plan **cannot be applied** — read-only and informational only
- It does **NOT lock the workspace**

---

### Step 5 — VCS workflow key facts

| Trigger | What happens |
|---------|-------------|
| Push to default branch | Full plan + apply (or awaiting confirmation) |
| Pull request opened/updated | Speculative plan only — cannot apply |
| Manual trigger in UI | Full plan + apply |
| `terraform apply` from CLI | Possible but discouraged — VCS is source of truth |

> ⚠️ **EXAM NOTE:** In VCS-driven workflow the VCS repo is the source of truth. Speculative plans triggered by PRs are read-only, cannot be applied, and do not lock the workspace.

---

## Lab 1.9 — State Migration to HCP Terraform

**Objective:** Migrate an existing local state file to HCP Terraform using the `cloud` block.

**Syllabus:** 8d

---

### Step 1 — Start with a local backend config

In a new directory:

```bash
mkdir hcp-migration-lab && cd hcp-migration-lab
```

Create `main.tf` with no backend block (defaults to local):

```hcl
output "migration_test" {
  value = "running locally"
}
```

```bash
terraform init
terraform apply -auto-approve
```

`terraform.tfstate` is now created locally.

---

### Step 2 — Create destination workspace in HCP Terraform

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
  value = "now running in HCP Terraform"
}
```

```bash
terraform init
```

**What you observe:** Terraform detects existing local state and prompts:
```
Do you wish to proceed? (yes/no)
```

Type `yes`.

---

### Step 4 — Verify state migration

```bash
terraform state list
```

Go to HCP Terraform UI → workspace `lab-migration-target` → **States** tab — the state is there.

The local `terraform.tfstate` file is now empty (original backed up as `terraform.tfstate.backup`).

> ⚠️ **EXAM NOTE:** Adding the `cloud` block to an existing config with local state and running `terraform init` prompts for state migration. It does not happen silently — you must confirm. Local state is NOT deleted — it becomes a backup.

---

## Lab 1.10 — Private Registry

**Objective:** Understand the HCP Terraform Private Registry for modules and providers.

**Syllabus:** 8b

---

### Step 1 — What the Private Registry does

The HCP Terraform Private Registry lets your organization:
- Publish and version internal Terraform modules for reuse across teams
- Host private providers not on the public registry
- Browse and document internal modules the same way the public registry does

---

### Step 2 — Module naming convention (required for VCS-connected modules)

```
terraform-<PROVIDER>-<MODULE_NAME>
```

Examples:
- `terraform-aws-networking`
- `terraform-gcp-database`
- `terraform-azurerm-compute`

---

### Step 3 — Consuming a private registry module

```hcl
module "networking" {
  source  = "app.terraform.io/<ORG_NAME>/networking/aws"
  version = "1.0.0"
}
```

| Registry | Source format |
|----------|--------------|
| Public Terraform Registry | `hashicorp/consul/aws` |
| HCP Terraform Private Registry | `app.terraform.io/<ORG>/module/provider` |

---

### Step 4 — Key facts

- Private registry modules support versioning via Git tags
- Modules are scoped to the organization
- `terraform login` must be completed before pulling private registry modules
- Private providers can also be hosted — not just modules

> ⚠️ **EXAM NOTE:** Private registry source format uses `app.terraform.io/<org>/<module>/<provider>`. The naming convention `terraform-<provider>-<name>` is required for VCS-connected modules. Know the source format difference between public and private registry.

---

## Lab 1.11 — Teams and Permissions

**Objective:** Understand HCP Terraform team structure and workspace permission levels.

**Syllabus:** 8b

---

### Step 1 — Team structure

| Team | Behavior |
|------|---------|
| owners | Full organization access — cannot be deleted or restricted |
| Custom teams | You define name and workspace permissions |

---

### Step 2 — Workspace permission levels

| Level | What they can do |
|-------|-----------------|
| Read | View runs, state, variables |
| Plan | Trigger plans, cannot apply |
| Write | Trigger and apply runs, manage variables |
| Admin | Full workspace control including settings and deletion |

---

### Step 3 — Create a team and assign workspace access

1. Organization Settings → **Teams → Create team**
2. Name it `lab-developers`
3. Go to workspace `lab-execution-modes` → **Settings**
4. Look for **Team Access** section
5. Add team `lab-developers` with **Write** permission
6. Save

---

### Step 4 — Key facts

| Fact | Detail |
|------|--------|
| Where are teams created? | Organization level |
| Where are permissions assigned? | Per workspace, per team |
| Can owners be restricted? | No — implicit admin access everywhere |
| API tokens for CI/CD? | Yes — team tokens can be scoped for pipeline use |
| Teams on Free tier? | Only the owners team — additional teams require paid tier |

> ⚠️ **EXAM NOTE:** Four workspace permission levels: Read, Plan, Write, Admin. Teams are organization-level constructs — permissions are assigned at the workspace level. The `owners` team cannot be restricted. Additional teams beyond `owners` require a paid tier.

---

## Lab 1.12 — Health Assessments (Drift Detection & Continuous Validation)

**Objective:** Understand what health assessments do, which tier they require, and what they cover.

**Syllabus:** 8b

> **Note:** Health assessments require **HCP Terraform Standard or Premium tier**. NOT available on Free or Essentials. If you are on Free tier, study the concepts — you will still be tested on them.

---

### Step 1 — What health assessments cover

| Check | What it does |
|-------|-------------|
| **Drift detection** | Compares real infrastructure against Terraform state — flags if someone changed infra outside Terraform |
| **Continuous validation** | Runs custom `check` blocks and `precondition`/`postcondition` assertions — verifies they still pass after provisioning |

Health assessments run **periodically and automatically** in the background. They are read-only — they do NOT trigger a plan or apply.

---

### Step 2 — Enable health assessments (Standard/Premium only)

1. Go to workspace `lab-execution-modes`
2. Left sidebar → **Settings**
3. Click **Health** in the settings sub-menu
4. Under **Health Assessments** → click **Enable**
5. Click **Save settings**

If on Free/Essentials — this option will not be available.

---

### Step 3 — Requirements for health assessments to run

All of the following must be true:

- Workspace has at least one **successfully applied run**
- Most recent run did NOT end in errored, cancelled, or discarded state
- Workspace is on **Standard or Premium tier**
- Remote execution mode is enabled

If the latest run errored, HCP Terraform **pauses** health assessments until a successful run completes.

---

### Step 4 — Trigger an on-demand assessment (Standard/Premium only)

1. Workspace → **Health** page
2. Click **Start health assessment**

Triggering on-demand resets the scheduling timer.

---

### Step 5 — Key facts table

| Fact | Answer |
|------|--------|
| Which tiers? | **Standard and Premium only** — NOT Free, NOT Essentials |
| Drift detection checks | Real infrastructure vs Terraform configuration |
| Continuous validation checks | Custom `check` blocks, `precondition`/`postcondition` |
| Triggers a plan/apply? | No — read-only |
| What pauses assessments? | Latest run ending in errored, cancelled, or discarded state |
| Where to enable? | Workspace Settings → Health |
| Org-wide enforcement? | Yes — org settings can enforce across all eligible workspaces, overriding workspace-level settings |

> ⚠️ **EXAM NOTE:** Health assessments = **Standard and Premium only**. Two components: drift detection and continuous validation. Automatic and periodic — not triggered by runs. A failed run pauses them until a successful run completes.

---

## Syllabus Cross-Check

| Objective | Labs | Status |
|-----------|------|--------|
| 8a — Use HCP Terraform to create infrastructure | 1.1, 1.3, 1.8 | ✓ |
| 8b — Collaboration and governance features | 1.4, 1.5, 1.10, 1.11, 1.12 | ✓ |
| 8c — Workspaces, projects, run triggers, variable sets | 1.2, 1.5, 1.6, 1.7 | ✓ |
| 8d — CLI integration, cloud block, migration | 1.0, 1.1, 1.8, 1.9 | ✓ |

---

## Lab Summary

| Lab | Key concept |
|-----|------------|
| 1.0 | `terraform login`, `cloud` block vs `backend "remote"`, name vs tags |
| 1.1 | Remote vs local execution mode — where plan runs, state location unchanged |
| 1.2 | Variable precedence — workspace var > variable set > `-var` flag ignored entirely |
| 1.3 | Run pipeline — Plan → Cost Estimation → Policy Check → Apply, speculative plans, queuing, auto-apply, destroy protection |
| 1.4 | Sentinel — advisory / soft mandatory / hard mandatory, placement in pipeline |
| 1.5 | Variable sets — global vs scoped, sensitive variables, workspace var wins |
| 1.6 | Projects — governance layer above workspaces, project-scoped variable sets |
| 1.7 | Run triggers — cross-workspace orchestration, fires on successful apply only |
| 1.8 | VCS workflow — push triggers run, PR triggers speculative plan, read-only |
| 1.9 | State migration — `terraform init` prompts migration, local state becomes backup |
| 1.10 | Private registry — naming convention, source format vs public registry |
| 1.11 | Teams and permissions — Read / Plan / Write / Admin levels |
| 1.12 | Health assessments — Standard/Premium only, drift detection + continuous validation |

---

## Cleanup

```bash
terraform destroy
```

Delete workspaces in HCP Terraform UI when done.

---

*Lab guide built for Terraform Associate 004 retake — July 30, 2026*
*Pair with: Terraform_004_CLI_Reference.md*
