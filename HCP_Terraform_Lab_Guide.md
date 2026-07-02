# HCP Terraform — Hands-On Lab Guide
**Terraform Associate 004 Retake Prep | July 30, 2026**

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Lab 1.0 — Authentication and the `cloud` Block](#lab-10--authentication-and-the-cloud-block)
- [Lab 1.1 — Execution Modes: Remote vs Local](#lab-11--execution-modes-remote-vs-local)
- [Lab 1.2 — Variable Precedence in Remote Execution](#lab-12--variable-precedence-in-remote-execution)
- [Lab 1.3 — The Run Pipeline](#lab-13--the-run-pipeline)
- [Lab 1.4 — Sentinel Policy Enforcement](#lab-14--sentinel-policy-enforcement)
- [Lab 1.5 — Variable Sets](#lab-15--variable-sets)
- [Lab 1.6 — Projects](#lab-16--projects)
- [Lab 1.7 — Run Triggers](#lab-17--run-triggers)
- [Lab 1.8 — VCS-Driven Workflow](#lab-18--vcs-driven-workflow)
- [Lab 1.9 — State Migration to HCP Terraform](#lab-19--state-migration-to-hcp-terraform)
- [Lab 1.10 — Private Registry](#lab-110--private-registry)
- [Lab 1.11 — Teams and Permissions](#lab-111--teams-and-permissions)
- [Lab 1.12 — Health Assessments](#lab-112--health-assessments)
- [Syllabus Cross-Check](#syllabus-cross-check)

---

## Prerequisites

Before starting Lab 1.0, complete the following:

- Create a free account at [app.terraform.io](https://app.terraform.io)
- Install Terraform CLI 1.5+ — verify with `terraform version`
- Install AWS CLI — verify with `aws --version`
- Configure AWS credentials locally — verify with `aws sts get-caller-identity`
- Install Git — verify with `git --version`

---

## Lab 1.0 — Authentication and the `cloud` Block

**What you will build:** Authenticate the Terraform CLI to HCP Terraform, create an organization and workspace, and connect a local Terraform configuration to HCP Terraform using the `cloud` block.

**Syllabus:** 8d

**Time:** ~15 minutes

---

### Step 1 — Create an organization in HCP Terraform

1. Log in to [app.terraform.io](https://app.terraform.io)
2. If prompted to create an organization, enter a name (e.g. `prateek-lab`) and click **Create organization**
3. Note your organization name — you will need it in every lab

---

### Step 2 — Authenticate the CLI

Run:

```bash
terraform login
```

Expected output:

```
Terraform will request an API token for app.terraform.io using your browser.

If login is successful, Terraform will store the token in plain text in
the following file for use by subsequent commands:
    /Users/YOU/.terraform.d/credentials.tfrc.json

Do you want to proceed?
  Only 'yes' will be accepted to confirm.

  Enter a value:
```

Type `yes` and press Enter.

Terraform opens your browser to `app.terraform.io/app/settings/tokens`. Generate a token, copy it, paste it back into the terminal prompt. Terraform will NOT display the token as you type — that is expected.

Expected output after pasting token:

```
Retrieved token for user <your-username>

Welcome to HCP Terraform!
```

The token is now stored at `~/.terraform.d/credentials.tfrc.json` on Mac/Linux, or `%APPDATA%\terraform.d\credentials.tfrc.json` on Windows.

> ⚠️ **EXAM NOTE:** `terraform login` stores credentials locally at `~/.terraform.d/credentials.tfrc.json` by default. Default hostname is `app.terraform.io`. `terraform logout` removes stored credentials.

---

### Step 3 — Create a working directory and configuration

```bash
mkdir ~/hcp-lab && cd ~/hcp-lab
```

Create `main.tf`:

```hcl
terraform {
  cloud {
    organization = "YOUR_ORG_NAME"   # replace with your org name from Step 1

    workspaces {
      name = "lab-hcp-intro"
    }
  }
}

output "hello" {
  value = "Connected to HCP Terraform"
}
```

---

### Step 4 — Initialize and connect to HCP Terraform

```bash
terraform init
```

Expected output:

```
Initializing HCP Terraform...

HCP Terraform has been successfully initialized!

You may now begin working with HCP Terraform. Try running "terraform plan" to
see any changes that are required for your infrastructure.
```

What just happened:
- Terraform read the `cloud` block
- It found (or created) the workspace `lab-hcp-intro` in your organization
- It connected your local working directory to that workspace

Go to `app.terraform.io` → your organization → **Workspaces**. You will see `lab-hcp-intro` listed there.

---

### Step 5 — Run a plan

```bash
terraform plan
```

Expected output:

```
Running plan in HCP Terraform. Output will stream here. Pressing Ctrl-C
will cancel the remote plan if it's still pending.

Preparing the remote plan...

To view this run in a browser, visit:
https://app.terraform.io/app/YOUR_ORG/lab-hcp-intro/runs/run-XXXXXXXX

Waiting for the plan to start...

Terraform v1.x.x
on linux_amd64

Changes to Outputs:
  + hello = "Connected to HCP Terraform"

Plan: 0 to add, 0 to change, 0 to destroy.
```

The plan ran on HCP Terraform's infrastructure — not your machine. Your terminal only received the streamed output.

Click the URL in the output to open the run in HCP Terraform UI. You will see:
- **Run details** — triggered via CLI, speculative plan
- **Plan finished** — shows the output diff
- Banner: **"This run was started from a speculative plan and cannot be applied"**

> ⚠️ **EXAM NOTE:** `terraform plan` from CLI creates a speculative plan — plan only, cannot be applied, does not lock the workspace. To trigger a full plan+apply you use `terraform apply`.

---

### Step 6 — Understand `cloud` block vs `backend "remote"` block

The `cloud` block (Terraform 1.1+) is the current way to connect to HCP Terraform. The `backend "remote"` block is the legacy way. Both work but the exam expects you to know the difference.

| | `cloud` block | `backend "remote"` |
|---|---|---|
| Introduced | Terraform 1.1+ | Earlier |
| Status | Current, recommended | Legacy |
| Workspace tags | Supported | Limited |

**Targeting multiple workspaces with tags instead of a name:**

```hcl
terraform {
  cloud {
    organization = "YOUR_ORG_NAME"
    workspaces {
      tags = ["env:staging", "app:web"]
    }
  }
}
```

When using tags, you must run `terraform workspace select <name>` to pick which matching workspace to use. When using a specific name, no selection is needed.

---

## Lab 1.1 — Execution Modes: Remote vs Local

**What you will prove:** Remote execution runs Terraform on HCP Terraform's infrastructure. Local execution runs Terraform on your machine. State stays in HCP Terraform in both cases.

**Syllabus:** 8a, 8d

**Time:** ~10 minutes

---

### Step 1 — Confirm current execution mode

1. Go to `app.terraform.io` → your org → workspace `lab-hcp-intro`
2. Left sidebar → **Settings**
3. Scroll to **Execution mode**
4. Confirm **Remote (custom)** is selected — your org and project may show **Project default** which defaults to Remote

You will see:
- **Project default** — inherits from the project setting
- **Remote (custom)** — runs on HCP Terraform infrastructure
- **Local (custom)** — runs on your machine, HCP Terraform only stores state

Leave it on Remote for now.

---

### Step 2 — Run a plan in Remote mode and observe

```bash
cd ~/hcp-lab
terraform plan
```

Expected output (key line):

```
Running plan in HCP Terraform. Output will stream here.
```

The plan executes on HCP Terraform's workers. Your terminal is a streaming viewer.

---

### Step 3 — Switch to Local execution mode

1. Workspace → **Settings**
2. Scroll to **Execution mode**
3. Select **Local (custom)**

   You will see the description: *"Your plans and applies run on your own machines. HCP Terraform only stores and synchronizes state."*

4. Click **Save settings**

---

### Step 4 — Run a plan in Local mode and observe

```bash
terraform plan
```

Expected output:

```
Terraform used the selected providers to generate the following execution plan.

Changes to Outputs:
  + hello = "Connected to HCP Terraform"

Plan: 0 to add, 0 to change, 0 to destroy.
```

No "Running plan in HCP Terraform" message. The plan ran locally on your machine.

---

### Step 5 — Verify state is still stored in HCP Terraform

Despite running locally, go to HCP Terraform UI → workspace `lab-hcp-intro` → left sidebar → **States**.

The state is stored in HCP Terraform regardless of execution mode.

---

### Step 6 — Key comparison

| | Remote (custom) | Local (custom) |
|---|---|---|
| Where plan/apply runs | HCP Terraform infrastructure | Your local machine |
| Where state is stored | HCP Terraform | HCP Terraform |
| Provider credentials | Must be in workspace variables | Your local environment |
| Terminal output | Streamed from remote | Direct |

> ⚠️ **EXAM NOTE:** Execution mode controls WHERE Terraform runs — NOT where state is stored. State always remains in HCP Terraform when the `cloud` block is configured, regardless of execution mode.

Switch execution mode back to **Remote (custom)** before proceeding.

---

## Lab 1.2 — Variable Precedence in Remote Execution

**What you will prove:** In remote execution, workspace variables override variable sets, and the `-var` flag is completely ignored — not just deprioritized.

**Syllabus:** 8c

**Time:** ~15 minutes

---

### Step 1 — Add a variable to your configuration

Update `~/hcp-lab/main.tf`:

```hcl
terraform {
  cloud {
    organization = "YOUR_ORG_NAME"
    workspaces {
      name = "lab-hcp-intro"
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

Run a plan to confirm the default value is used:

```bash
terraform plan
```

Expected in output:

```
Changes to Outputs:
  + env_output = "default_value"
```

---

### Step 2 — Create a Variable Set and observe it override the default

1. HCP Terraform → left sidebar → **Settings** (organization level, not workspace)
2. Click **Variable Sets**
3. Click **Create variable set**
4. Name: `lab-varset`
5. Under **Variables**, click **Add variable**:
   - Variable category: **Terraform variable**
   - Key: `environment`
   - Value: `from_variable_set`
   - Leave HCL and Sensitive unchecked
6. Under **Variable set scope**, select **Apply to specific workspaces**
7. Select workspace `lab-hcp-intro`
8. Click **Create variable set**

Now run:

```bash
terraform plan
```

Expected in output:

```
Changes to Outputs:
  + env_output = "from_variable_set"
```

The variable set value overrode the default.

---

### Step 3 — Add a workspace variable and observe it override the variable set

1. HCP Terraform → workspace `lab-hcp-intro` → left sidebar → **Variables**
2. Under **Workspace Variables** → **Terraform Variables** → click **+ Add variable**
3. Key: `environment`
4. Value: `from_workspace_variable`
5. Click **Save variable**

Run:

```bash
terraform plan
```

Expected in output:

```
Changes to Outputs:
  + env_output = "from_workspace_variable"
```

The workspace variable overrode the variable set.

---

### Step 4 — Attempt to override with `-var` flag

```bash
terraform plan -var="environment=from_local_flag"
```

Expected in output:

```
Warning: Ignored -var flag in remote operations
│ 
│ The configured remote backend does not support setting remote variables
│ through the "-var" option.

Changes to Outputs:
  + env_output = "from_workspace_variable"
```

The `-var` flag is completely ignored. Workspace variable still wins.

> ⚠️ **EXAM NOTE:** In remote execution, `-var` does not participate in the precedence chain at all — it is ignored entirely and never sent to HCP Terraform. This is NOT a precedence battle — local flags simply do not exist in remote execution variable resolution.

---

### Step 5 — Variable precedence summary

**Remote execution — the only rules that apply:**

| Order | Source | Notes |
|-------|--------|-------|
| 1 (lowest) | Default values in config | Overridden by everything |
| 2 | Variable sets | Scoped to workspace/project/org |
| 3 (highest) | Workspace variables | Always wins |
| IGNORED | `-var`, `-var-file`, `TF_VAR_`, local `.tfvars` | Not sent to HCP Terraform |

---

## Lab 1.3 — The Run Pipeline

**What you will prove:** The exact stages in the HCP Terraform run pipeline, how CLI-driven vs UI-triggered runs differ, queuing behavior, and auto-apply vs manual apply.

**Syllabus:** 8a, 8b

**Time:** ~20 minutes

---

### Step 1 — Run pipeline order — burn this in

```
Pending
  ↓
Planning
  ↓
Cost Estimation   ← exam expects this to exist between Planning and Policy Check
  ↓
Policy Check      ← only if Sentinel/OPA policies are configured
  ↓
Planned and finished   ← plan-only runs stop here
  ↓
Applying          ← only for plan+apply runs after confirmation (or auto-apply)
  ↓
Applied
```

> ⚠️ **EXAM NOTE:** Cost Estimation comes BEFORE Policy Check. Do not invert these. The exam tests this order directly. Note: Cost estimation was removed from the HCP Terraform product in 2026, but the 004 exam was written before this change — answer what the exam expects.

---

### Step 2 — Understand the three workspace workflow types

Before running anything, understand which workflow type you are in. This determines how runs are triggered and where configuration comes from.

| Workflow | Config comes from | Runs triggered by |
|----------|------------------|------------------|
| **CLI-driven** | Your local machine, uploaded at run time | `terraform plan` / `terraform apply` from CLI |
| **VCS-driven** | A connected Git repository | Git push, pull request, or manual UI trigger |
| **API-driven** | Uploaded via API call | API call |

Your workspace `lab-hcp-intro` is CLI-driven. This has a critical implication for the next step.

---

### Step 3 — Trigger a run from CLI and observe the run list

Ensure execution mode is **Remote (custom)**. Then run:

```bash
terraform plan
```

While the plan is running, immediately open the HCP Terraform UI → workspace `lab-hcp-intro` → left sidebar → **Runs**.

**What you see in the Run List:**
- A new entry at the top labelled **Triggered via CLI**
- Type shows: **plan-only run**
- Status badge on the right: **Running** → transitions to **Planned and finished**
- Filter tabs across the top: **All | Needs Attention | Errored | Running | On Hold | Success**

Click on the run to open the run detail page.

**What you see on the run detail page:**
- **Plan duration** — how long the plan took
- **Resources to be changed** — green additions, red destructions, yellow changes
- **Run details** (expandable) — who triggered it, timestamp, run ID
- **Plan finished** section — full plan output with resource diffs
- **Outputs** section — output values from the plan
- **Download Sentinel mocks** button — for testing Sentinel policies offline
- Banner at bottom: **"This run was started from a speculative plan and cannot be applied"**
- **Retry run** button
- **Add comment** box

---

### Step 4 — Why you CANNOT use the + New run button for CLI-driven workspaces

In the Run List, click the blue **+ New run** button (top right). A modal appears with Run name and Run Type fields. Click **Start**.

**You will see this error:**

```
Error starting plan
Configuration version is missing
```

**Why this happens:** For a CLI-driven workspace, HCP Terraform has no configuration stored. The configuration only exists when you upload it via `terraform plan` or `terraform apply` from your local machine. The UI **+ New run** button requires a configuration version to already exist — which only VCS-driven and API-driven workspaces have permanently attached.

> ⚠️ **EXAM NOTE:** This is a fundamental distinction between the three workflow types. In CLI-driven workspaces, configuration is uploaded on demand by the CLI. In VCS-driven workspaces, configuration is permanently attached from a Git repo — which is why the UI button works there.

Close the error. You will trigger full runs via `terraform apply` instead.

---

### Step 5 — Trigger a full plan+apply run from CLI

```bash
terraform apply
```

Expected output:

```
Running apply in HCP Terraform. Output will stream here. Pressing Ctrl-C
will cancel the remote apply if it's still pending. If the apply started it
will stop streaming the logs, but will not stop the apply running remotely.

Preparing the remote apply...

To view this run in a browser, visit:
https://app.terraform.io/app/YOUR_ORG/lab-hcp-intro/runs/run-XXXXXXXX

Waiting for the plan to start...

Terraform v1.x.x
on linux_amd64

Changes to Outputs:
  + env_output = "from_workspace_variable"

Plan: 0 to add, 0 to change, 0 to destroy.

Do you want to perform these actions in workspace "lab-hcp-intro"?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

**Do NOT type yes yet.** Go to the HCP Terraform UI → workspace → **Runs**.

**What you see:** The run is listed with status **On Hold** — it is waiting for your confirmation. This is manual apply mode (auto-apply is disabled by default).

Now go back to your terminal and type `yes`.

The run proceeds to apply and completes. In the UI the run moves from **On Hold** → **Applying** → **Applied** (shown as **Success** in the run list).

---

### Step 6 — Enable Auto-apply and observe the difference

1. Workspace **Settings** → scroll to **Auto-apply**
2. You will see two checkboxes:
   - **Auto-apply API, UI, & VCS runs** — covers CLI, UI, API, VCS-triggered runs
   - **Auto-apply run triggers** — covers runs triggered by upstream workspaces
3. Check **Auto-apply API, UI, & VCS runs**
4. Click **Save settings**

Run:

```bash
terraform apply
```

Expected output: plan runs, then applies immediately without prompting for `yes`. No confirmation step.

Go to the Run List in the UI — the run completed end to end without any **On Hold** pause.

> ⚠️ **EXAM NOTE:** Two separate auto-apply settings exist and can be enabled independently. Unconfirmed plans in manual apply mode are discarded after **30 days** — not 24 hours.

Uncheck **Auto-apply API, UI, & VCS runs** → **Save settings** before continuing.

---

### Step 7 — Observe workspace destroy protection

1. Workspace **Settings** → scroll down to find **Destroy Protection** (may be near the bottom)
2. Enable it → **Save settings**

Run:

```bash
terraform destroy
```

**What you observe:** HCP Terraform blocks the destroy. You must disable destroy protection in Settings before a destroy can proceed.

Disable destroy protection → **Save settings** before continuing.

> ⚠️ **EXAM NOTE:** Workspace destroy protection is an HCP Terraform UI-level guard that blocks destroy runs for the entire workspace. It is not the same as the Terraform `prevent_destroy` lifecycle argument, which is a Terraform configuration language feature.

---

### Step 8 — Run types reference

| Run type | How to trigger | Can apply? | Blocks queue? | Config required in HCP? |
|----------|---------------|-----------|--------------|------------------------|
| `terraform plan` (CLI) | CLI | No — speculative | No | No — uploaded at run time |
| `terraform apply` (CLI) | CLI | Yes | Yes | No — uploaded at run time |
| + New run → Plan and apply | UI | Yes | Yes | Yes — VCS/API workspaces only |
| + New run → Plan only | UI | No | No | Yes — VCS/API workspaces only |
| VCS push to default branch | Git push | Yes (or manual apply) | Yes | Yes — from repo |
| Pull request | Git PR | No — speculative | No | Yes — from repo |

---

## Lab 1.4 — Sentinel Policy Enforcement

**What you will prove:** Sentinel policy enforcement levels, where policy checks sit in the run pipeline, and what happens when a policy fails.

**Syllabus:** 8b

**Time:** ~20 minutes

> **Tier note:** Sentinel requires HCP Terraform Plus tier or a free trial. If unavailable on your current tier, complete Steps 1–3 conceptually and note the behavior for the exam.

---

### Step 1 — Sentinel enforcement levels

| Level | Policy fails | What happens | Override possible? |
|-------|-------------|-------------|-------------------|
| Advisory | Yes | Warning logged, run **proceeds** to apply | N/A — does not block |
| Soft Mandatory | Yes | Run **blocked**, authorized user can override | Yes |
| Hard Mandatory | Yes | Run **blocked**, NO override, ever | No |

> ⚠️ **EXAM NOTE:** Advisory does NOT block the run — it only logs a warning. Hard Mandatory cannot be overridden by anyone including organization owners.

---

### Step 2 — Create a policy set

1. HCP Terraform → organization **Settings** (not workspace settings) → **Policy Sets**
2. Click **Connect a new policy set**
3. Select **No VCS connection** (manual upload for this lab)
4. Name: `lab-tag-policy`
5. Under **Scope** → **Apply to specific workspaces** → select `lab-hcp-intro`
6. Click **Connect policy set**

---

### Step 3 — Write a Sentinel policy

Create a file locally named `enforce-tags.sentinel`:

```hcl
# Sentinel is its own language — not HCL, not Python
# This policy requires all EC2 instances to have an Environment tag

import "tfplan/v2" as tfplan

main = rule {
  all tfplan.resource_changes as _, rc {
    rc.type is not "aws_instance" or
    rc.change.after.tags contains "Environment"
  }
}
```

Upload this file to the policy set you just created in Step 2.

Set enforcement level to **Soft Mandatory**.

---

### Step 4 — Add a resource that violates the policy

Add to `~/hcp-lab/main.tf`:

```hcl
resource "aws_instance" "test" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  # No tags — this will fail the Sentinel policy
}
```

Run:

```bash
terraform apply
```

**What you see in the CLI output:**

```
Running apply in HCP Terraform. Output will stream here.

...

Plan: 1 to add, 0 to change, 0 to destroy.

Sentinel Result: false

Sentinel policies evaluated:

│ lab-tag-policy/enforce-tags.sentinel
│   Result: false
│   FAIL - enforce-tags.sentinel (soft-mandatory)

Policy check failed (soft-mandatory). To override, go to:
https://app.terraform.io/app/YOUR_ORG/lab-hcp-intro/runs/run-XXXXXXXX
```

**What you see in the HCP Terraform UI on the run detail page:**
- Plan stage: ✓ complete
- Policy Check stage: ✗ FAILED
- Apply stage: not reached — blocked
- Since enforcement is Soft Mandatory: an **Override and Continue** button is available to authorized users

---

### Step 5 — Fix the violation and confirm policy passes

Update `main.tf` to add the required tag:

```hcl
resource "aws_instance" "test" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Environment = "lab"
  }
}
```

Run:

```bash
terraform apply
```

**What you see:** Policy check passes, run proceeds to confirmation/apply.

> ⚠️ **EXAM NOTE:** Sentinel sits AFTER cost estimation and BEFORE apply in the run pipeline. It is a Plus/Enterprise feature. OPA (Open Policy Agent) is also supported as an alternative policy framework.

---

## Lab 1.5 — Variable Sets

**What you will prove:** How variable sets are scoped, how global vs project vs workspace scoping works, and how to manage provider credentials correctly using variable sets.

**Syllabus:** 8c

**Time:** ~15 minutes

---

### Step 1 — Create a second workspace

1. HCP Terraform → your org → **New Workspace**
2. Choose **CLI-driven workflow**
3. Name: `lab-varset-test`
4. Leave execution mode as default
5. Click **Create workspace**

---

### Step 2 — Create a global variable set for AWS credentials

1. Organization **Settings** → **Variable Sets** → **Create variable set**
2. Name: `global-aws-credentials`
3. Under **Variables** → **Add variable**:
   - Category: **Environment variable**
   - Key: `AWS_ACCESS_KEY_ID`
   - Value: your AWS access key ID
4. Add another variable:
   - Category: **Environment variable**
   - Key: `AWS_SECRET_ACCESS_KEY`
   - Value: your AWS secret access key
   - Check **Sensitive** — this hides the value after saving
5. Under **Variable set scope** → select **Apply to all workspaces in the organization**
6. Click **Create variable set**

**What you observe:** Both `lab-hcp-intro` and `lab-varset-test` now have AWS credentials available without configuring them per workspace. This is the correct pattern for managing provider credentials in HCP Terraform.

---

### Step 3 — Verify a sensitive variable cannot be read back

Go to Organization Settings → Variable Sets → `global-aws-credentials` → click to open it.

**What you observe:** The `AWS_SECRET_ACCESS_KEY` value shows as `[redacted]` — it cannot be read back after being saved as sensitive. It can only be overwritten or deleted.

> ⚠️ **EXAM NOTE:** Sensitive variable values in HCP Terraform are write-only — they cannot be read after being set. They can only be updated or deleted.

---

### Step 4 — Verify scoping: scoped variable set only applies to its target

Your `lab-varset` from Lab 1.2 is scoped only to `lab-hcp-intro`.

In workspace `lab-varset-test` → left sidebar → **Variables** → scroll to **Variable Sets Applied**.

**What you observe:** Only the global `global-aws-credentials` variable set is applied here — not `lab-varset`. The `environment` variable from `lab-varset` is not available in `lab-varset-test`.

---

### Step 5 — Variable set scope summary

| Scope | How to configure | Applies to |
|-------|-----------------|------------|
| All workspaces in org | "Apply to all workspaces" | Every workspace in the organization |
| Specific project | Select a project | All workspaces inside that project |
| Specific workspaces | Select individual workspaces | Only those workspaces |

> ⚠️ **EXAM NOTE:** A workspace-specific variable always overrides a variable set value for the same key. Variable sets applied at a broader scope (org) are overridden by narrower scope (workspace variable).

---

## Lab 1.6 — Projects

**What you will prove:** Projects organize workspaces into governance boundaries. Variable sets and policy sets applied to a project affect all workspaces in it.

**Syllabus:** 8b, 8c

**Time:** ~10 minutes

---

### Step 1 — Create a project

1. HCP Terraform → top navigation or left sidebar → **Projects**
2. Click **New project**
3. Name: `lab-project`
4. Click **Create**

---

### Step 2 — Move workspaces into the project

1. Go to workspace `lab-hcp-intro` → **Settings**
2. Scroll to **Project** at the top of the settings page
3. Change from your current project to **lab-project**
4. Click **Save settings**

Repeat for `lab-varset-test`.

---

### Step 3 — Apply a variable set at the project level

1. Organization **Settings** → **Variable Sets** → click `lab-varset`
2. Click **Edit**
3. Under **Variable set scope** → change from specific workspace to **Apply to specific projects**
4. Select **lab-project**
5. Save

**What you observe:** The `environment = from_variable_set` value now automatically applies to ALL workspaces inside `lab-project` — including any new ones added to the project in the future.

Go to workspace `lab-varset-test` → left sidebar → **Variables** → scroll to **Variable Sets Applied**.

**What you observe:** `lab-varset` now appears here even though you never scoped it directly to this workspace — it came through the project.

> ⚠️ **EXAM NOTE:** Projects are a governance and organizational layer above workspaces. Variable sets and policy sets applied at the project level affect all workspaces within it automatically. This is a 004-specific topic — not tested in the 003 exam.

---

## Lab 1.7 — Run Triggers

**What you will prove:** Run triggers fire automatically when an upstream workspace successfully applies. They fire on successful apply only — not on failed runs, not on plan-only runs.

**Syllabus:** 8c

**Time:** ~15 minutes

---

### Step 1 — What run triggers solve

Without run triggers: Workspace B depends on outputs from Workspace A. Every time A applies, you manually go trigger B.

With run triggers: When A successfully applies, B automatically queues a new run.

---

### Step 2 — Configure workspace `lab-varset-test` as the downstream workspace

1. Go to workspace `lab-varset-test`
2. Left sidebar → **Settings**
3. In the settings menu look for **Run Triggers**
4. Click **Add another workspace**
5. Select `lab-hcp-intro` as the source workspace
6. Save

---

### Step 3 — Create a minimal configuration for `lab-varset-test`

On your local machine:

```bash
mkdir ~/hcp-lab-b && cd ~/hcp-lab-b
```

Create `main.tf`:

```hcl
terraform {
  cloud {
    organization = "YOUR_ORG_NAME"
    workspaces {
      name = "lab-varset-test"
    }
  }
}

output "downstream" {
  value = "I was triggered by upstream"
}
```

```bash
terraform init
terraform apply
```

Type `yes` to apply. This gives `lab-varset-test` its initial state and a successful run — required for run triggers to work.

---

### Step 4 — Trigger the upstream workspace and observe the downstream fire

Go back to your original directory:

```bash
cd ~/hcp-lab
terraform apply
```

Type `yes`.

After the apply completes in `lab-hcp-intro`, go to HCP Terraform UI → workspace `lab-varset-test` → **Runs**.

**What you observe:** A new run has automatically queued in `lab-varset-test` triggered by the upstream workspace completing its apply. The run detail shows it was triggered by a run trigger, not by a CLI command.

---

### Step 5 — Verify run triggers fire on successful apply only

Go back to workspace `lab-hcp-intro`. Trigger a plan-only run:

```bash
terraform plan
```

After it completes, check `lab-varset-test` → **Runs**.

**What you observe:** No new run was triggered. Plan-only runs do NOT fire run triggers.

> ⚠️ **EXAM NOTE:** Run triggers fire on **successful applies only**. Failed applies, discarded runs, and plan-only runs do NOT trigger downstream workspaces. The **Auto-apply run triggers** checkbox in workspace settings controls whether triggered runs auto-apply or wait for manual confirmation.

---

## Lab 1.8 — VCS-Driven Workflow

**What you will prove:** VCS-driven workspaces get config from a Git repo. Git pushes trigger full runs. Pull requests trigger speculative plans only. The + New run UI button works here (unlike CLI-driven workspaces).

**Syllabus:** 8a, 8d

**Time:** ~20 minutes

---

### Step 1 — Create a GitHub repository

1. Go to [github.com](https://github.com) → **New repository**
2. Name: `terraform-hcp-vcs-lab`
3. Initialize with a README
4. Create the repository

---

### Step 2 — Connect HCP Terraform to GitHub

1. HCP Terraform → organization **Settings** → **Version Control** → **Providers**
2. Click **Add a VCS provider**
3. Select **GitHub**
4. Follow the OAuth authorization flow — authorize HCP Terraform to access your GitHub
5. After authorization you are redirected back to HCP Terraform with GitHub connected

---

### Step 3 — Create a VCS-driven workspace

1. HCP Terraform → **New Workspace**
2. Select **Version Control Workflow** (not CLI-driven)
3. Select your GitHub connection
4. Select repository `terraform-hcp-vcs-lab`
5. Name workspace: `lab-vcs-driven`
6. Leave working directory blank
7. Click **Create workspace**

---

### Step 4 — Push a Terraform configuration and observe an automatic run

On your local machine:

```bash
git clone https://github.com/YOUR_USERNAME/terraform-hcp-vcs-lab.git
cd terraform-hcp-vcs-lab
```

Create `main.tf`:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

output "vcs_test" {
  value = "Triggered by VCS push"
}
```

Commit and push:

```bash
git add main.tf
git commit -m "Add initial Terraform configuration"
git push origin main
```

Go to HCP Terraform → workspace `lab-vcs-driven` → **Runs**.

**What you observe:**
- A new run appears automatically — triggered by your Git push
- Source shows: **Triggered via VCS**
- No CLI command was run — the push triggered it

---

### Step 5 — Use the + New run button (works here unlike CLI-driven)

In workspace `lab-vcs-driven` → **Runs** → click **+ New run**.

Select **Plan and apply (standard)** → click **Start**.

**What you observe:** This works — no "Configuration version is missing" error. Because this is a VCS-driven workspace, HCP Terraform has the configuration permanently available from the Git repo.

---

### Step 6 — Create a Pull Request and observe a speculative plan

```bash
git checkout -b test-feature
```

Edit `main.tf` — change the output value:

```hcl
output "vcs_test" {
  value = "Testing a new feature"
}
```

```bash
git add main.tf
git commit -m "Test output change"
git push origin test-feature
```

Open a Pull Request in GitHub from `test-feature` → `main`.

**What you observe in GitHub:** A status check appears on the PR from HCP Terraform showing the plan result.

**What you observe in HCP Terraform → workspace `lab-vcs-driven` → Runs:**
- A new run appears labelled as a speculative plan
- It is triggered by the pull request
- The run detail page shows the banner: **"This run was started from a speculative plan and cannot be applied"**
- No **Confirm & Apply** button is available

> ⚠️ **EXAM NOTE:** Speculative plans triggered by pull requests are read-only — they cannot be applied and do NOT lock the workspace. They exist to show reviewers what would change if the PR is merged.

---

## Lab 1.9 — State Migration to HCP Terraform

**What you will prove:** Adding a `cloud` block to an existing config with local state and running `terraform init` triggers a migration prompt. The migration is not automatic or silent.

**Syllabus:** 8d

**Time:** ~10 minutes

---

### Step 1 — Create a local Terraform workspace

```bash
mkdir ~/hcp-migration-lab && cd ~/hcp-migration-lab
```

Create `main.tf` with no backend block (defaults to local):

```hcl
output "migration_test" {
  value = "currently running locally"
}
```

Initialize and apply locally:

```bash
terraform init
terraform apply -auto-approve
```

Expected output:

```
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

migration_test = "currently running locally"
```

Verify local state file exists:

```bash
ls -la terraform.tfstate
```

Expected output: `terraform.tfstate` is present with file size > 0.

---

### Step 2 — Create a destination workspace in HCP Terraform

1. HCP Terraform → **New Workspace** → CLI-driven workflow
2. Name: `lab-migration-target`
3. Remote execution mode
4. Create workspace

---

### Step 3 — Add the `cloud` block and run init

Update `main.tf`:

```hcl
terraform {
  cloud {
    organization = "YOUR_ORG_NAME"
    workspaces {
      name = "lab-migration-target"
    }
  }
}

output "migration_test" {
  value = "now running in HCP Terraform"
}
```

Run:

```bash
terraform init
```

Expected output:

```
Initializing HCP Terraform...
Do you wish to proceed?
  As part of migrating to HCP Terraform, Terraform can optionally copy
  your current workspace state to the configured HCP Terraform workspace.
  
  Answer "yes" to copy the latest state snapshot to the configured
  HCP Terraform workspace.
  
  Answer "no" to ignore the existing state and just activate the configured
  HCP Terraform workspace with its existing state, if any.
  
  Should Terraform migrate your existing state?

  Enter a value:
```

Type `yes`.

```
HCP Terraform has been successfully initialized!
```

---

### Step 4 — Verify migration

Check local state:

```bash
cat terraform.tfstate
```

Expected output: `{}` — the local state is now empty (original backed up as `terraform.tfstate.backup`).

Go to HCP Terraform → workspace `lab-migration-target` → left sidebar → **States**.

**What you observe:** The state is now in HCP Terraform — the output value from before migration is visible in the state.

> ⚠️ **EXAM NOTE:** State migration requires explicit confirmation — it does NOT happen silently. Local state becomes a backup (`terraform.tfstate.backup`) and is NOT deleted. You cannot migrate back to local without manually reconfiguring the backend and running `terraform init -migrate-state`.

---

## Lab 1.10 — Private Registry

**What you will prove:** The Private Registry stores and versions internal modules. It has a different source format from the public Terraform Registry. Modules must follow a specific naming convention.

**Syllabus:** 8b

**Time:** ~15 minutes (conceptual + UI exploration)

---

### Step 1 — Explore the Private Registry in the UI

1. HCP Terraform → left sidebar → **Registry**

You will see two tabs:
- **Modules** — internal modules published by your organization
- **Providers** — private providers hosted by your organization

The registry is currently empty for a new organization.

---

### Step 2 — Module naming convention

To publish a module to the Private Registry from a VCS repository, the repo name MUST follow this exact format:

```
terraform-<PROVIDER>-<MODULE_NAME>
```

Examples:
- `terraform-aws-networking` → module name `networking`, provider `aws`
- `terraform-gcp-database` → module name `database`, provider `gcp`
- `terraform-azurerm-compute` → module name `compute`, provider `azurerm`

If the repository name does not match this format, HCP Terraform will reject it.

---

### Step 3 — Understand the source format difference

When consuming a module from the **public** Terraform Registry:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
}
```

When consuming a module from your **HCP Terraform Private Registry**:

```hcl
module "networking" {
  source  = "app.terraform.io/YOUR_ORG_NAME/networking/aws"
  version = "1.0.0"
}
```

| | Public Registry | Private Registry |
|---|---|---|
| Source format | `<namespace>/<module>/<provider>` | `app.terraform.io/<org>/<module>/<provider>` |
| Authentication | None required | `terraform login` must be completed |
| Visibility | Public | Organization-only |
| Versioning | Git tags | Git tags |

---

### Step 4 — Key facts

- `terraform login` must be completed before Terraform can pull modules from the Private Registry
- Private providers can also be hosted — not just modules
- No-code modules (Plus/Enterprise feature) allow users to deploy resources without writing HCL

> ⚠️ **EXAM NOTE:** Private Registry source format is `app.terraform.io/<org>/<module>/<provider>`. The naming convention `terraform-<provider>-<name>` is required for VCS-connected modules. Know both the source format AND the naming convention — both are tested.

---

## Lab 1.11 — Teams and Permissions

**What you will prove:** Teams are an organization-level construct. Workspace permissions are assigned per team per workspace. There are four permission levels.

**Syllabus:** 8b

**Time:** ~10 minutes

---

### Step 1 — Understand the team structure

Every HCP Terraform organization has one built-in team:

- **owners** — has full access to everything in the organization, cannot be deleted, cannot be restricted

Additional teams can be created and given specific workspace-level permissions.

---

### Step 2 — Create a team

1. Organization **Settings** → **Teams**
2. Click **Create a team**
3. Name: `lab-developers`
4. Leave organization access permissions unchecked for now
5. Click **Create team**

---

### Step 3 — Assign workspace access to the team

1. Go to workspace `lab-hcp-intro` → **Settings**
2. Scroll to find the **Team Access** section (may be under a sub-menu)
3. Click **Add team and permissions**
4. Select `lab-developers`
5. Select permission level: **Write**
6. Click **Add team**

**What you observe:** The `lab-developers` team now has Write access to `lab-hcp-intro` but not to other workspaces.

---

### Step 4 — Permission levels

| Level | Can do |
|-------|--------|
| **Read** | View runs, state, variables |
| **Plan** | Trigger plans, cannot apply |
| **Write** | Trigger and apply runs, manage variables |
| **Admin** | Full workspace control including settings and deletion |

---

### Step 5 — Key facts

| Fact | Answer |
|------|--------|
| Where are teams created? | Organization level |
| Where are permissions assigned? | Per workspace, per team |
| Can owners be restricted? | No — implicit full access everywhere |
| API tokens for CI/CD? | Yes — team tokens can be scoped for pipeline automation |
| Teams on Free tier? | Only owners team — additional teams require paid tier |

> ⚠️ **EXAM NOTE:** Four workspace permission levels: Read, Plan, Write, Admin. Teams are org-level — permissions are workspace-level. The owners team cannot be restricted under any circumstances. Additional teams beyond owners require a paid tier.

---

## Lab 1.12 — Health Assessments

**What you will prove:** Health assessments automatically check for drift and validate custom conditions. They are available on Standard and Premium tiers only.

**Syllabus:** 8b

**Time:** ~10 minutes (conceptual if on Free tier)

> **Tier note:** Health assessments require **HCP Terraform Standard or Premium tier**. NOT available on Free or Essentials. The exam tests this tier restriction.

---

### Step 1 — What health assessments check

Health assessments run two types of checks automatically and periodically:

| Check | What it does |
|-------|-------------|
| **Drift detection** | Compares real infrastructure against the Terraform state — flags if someone changed infra outside Terraform |
| **Continuous validation** | Runs custom `check` blocks and `precondition`/`postcondition` assertions — verifies they still pass after provisioning |

Health assessments are **read-only** — they never trigger a plan or apply.

---

### Step 2 — Enable health assessments (Standard/Premium only)

1. Go to workspace `lab-hcp-intro`
2. Left sidebar → **Settings**
3. Look for **Health** in the settings sub-navigation
4. Under **Health Assessments** → click **Enable**
5. Click **Save settings**

If on Free/Essentials tier: the Health option will not appear in the settings menu. This confirms the tier restriction.

---

### Step 3 — Requirements for health assessments to run

All of the following must be true before HCP Terraform will run a health assessment:

- Workspace has at least one **successfully applied run**
- Most recent run did NOT end in errored, cancelled, or discarded state
- Workspace is on **Standard or Premium tier**
- Remote execution mode enabled

If the latest run errored, HCP Terraform **pauses** all health assessments until a new successful run completes.

---

### Step 4 — Trigger an on-demand assessment

1. Workspace → **Health** page (accessible via Settings → Health or direct sidebar link)
2. Click **Start health assessment**

HCP Terraform runs an assessment immediately. Triggering on-demand also resets the automated scheduling timer.

---

### Step 5 — Key exam facts

| Fact | Answer |
|------|--------|
| Which tiers? | **Standard and Premium only** — NOT Free, NOT Essentials |
| Two components | Drift detection + continuous validation |
| Triggers a plan/apply? | No — read-only |
| What pauses assessments? | Latest run ending in errored, cancelled, or discarded state |
| Where to enable? | Workspace Settings → Health |
| Org-wide enforcement? | Yes — org settings can enforce across all eligible workspaces, overriding workspace-level settings |

> ⚠️ **EXAM NOTE:** Health assessments = **Standard and Premium only**. Two components: drift detection and continuous validation. They are automatic and periodic — never triggered by a run. A failed run pauses them until a successful run completes.

---

## Syllabus Cross-Check

Use this before your exam to confirm every 8x objective is covered:

| Objective | Labs | ✓ |
|-----------|------|---|
| 8a — Use HCP Terraform to create infrastructure | 1.1, 1.3, 1.8 | ✓ |
| 8b — Collaboration and governance features | 1.4, 1.5, 1.6, 1.10, 1.11, 1.12 | ✓ |
| 8c — Workspaces, projects, run triggers, variable sets | 1.2, 1.5, 1.6, 1.7 | ✓ |
| 8d — CLI integration, cloud block, migration | 1.0, 1.1, 1.8, 1.9 | ✓ |

---

## Cleanup

When all labs are complete:

```bash
# Destroy resources in each workspace
cd ~/hcp-lab && terraform destroy
cd ~/hcp-lab-b && terraform destroy
cd ~/hcp-migration-lab && terraform destroy
cd ~/terraform-hcp-vcs-lab && terraform destroy
```

In HCP Terraform UI:
- Delete each workspace: workspace **Settings** → scroll to bottom → **Delete workspace**
- Delete the `lab-project` project: Projects → lab-project → Settings → Delete
- Delete variable sets: Organization Settings → Variable Sets → delete each lab variable set

---

*Lab guide built for Terraform Associate 004 retake — July 30, 2026*
*Pair with: Terraform_004_CLI_Reference.md*
