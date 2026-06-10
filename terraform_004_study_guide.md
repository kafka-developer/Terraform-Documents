# Terraform Associate 004 — Retake Study Guide
**Exam Date: June 21st | Focus: HCP Terraform, Core Workflow, IaC Concepts, Fundamentals**

---

## SECTION 1: HCP Terraform

### 1.1 Execution Modes

| Mode | Where plan/apply runs | `-var` flag behavior |
|------|----------------------|---------------------|
| Remote | HCP Terraform infrastructure | **Ignored** |
| Local | Developer's machine | Honored — highest precedence |
| Agent | Self-hosted agent pool | Ignored (same as remote) |

**Critical rule:** In remote execution mode, `-var` flags passed from local CLI are NOT transmitted to the remote runner. They are silently ignored.

---

### 1.2 Variable Precedence in HCP Terraform (Remote Execution)

Highest to lowest:
1. Workspace-specific variables (set directly on the workspace)
2. Variable sets applied to the workspace
3. Default values in configuration
4. `-var` flag — **IGNORED in remote execution mode**

**Exam trap:** Don't apply local CLI variable precedence rules to remote execution mode questions.

---

### 1.3 Run Pipeline (Remote Execution)

```
Code push / manual trigger
        ↓
    Plan queued
        ↓
    Plan runs
        ↓
  Sentinel policy check (if configured)
        ↓
  Cost estimation (if enabled)
        ↓
  Awaiting confirmation (if not auto-apply)
        ↓
    Apply runs
        ↓
  State saved to HCP Terraform backend
```

**Key behaviors:**
- Runs within a workspace execute **serially** — never concurrently
- A second triggered apply while one is running is **queued automatically**, not rejected
- Compare: S3/DynamoDB backend → second apply **fails with lock error**

---

### 1.4 Sentinel Policy Enforcement

Sentinel is HCP Terraform's policy-as-code framework. It is **not available in open source Terraform**.

**Enforcement levels:**

| Level | Behavior |
|-------|----------|
| Advisory | Warning logged, run proceeds |
| Soft Mandatory | Run blocked, authorized user can override |
| Hard Mandatory | Run blocked, no override possible |

**When Sentinel runs:** After plan, before apply — in the run pipeline.

**Use cases the exam tests:**
- Enforce instance size limits (e.g. no larger than m5.4xlarge)
- Enforce encryption requirements on storage resources
- Restrict allowed regions
- Enforce tagging standards

**`terraform validate` does NOT enforce policy** — it only checks syntax and internal config consistency.

---

### 1.5 Workspace Features

**Workspace-level settings:**
- Execution mode (remote / local / agent)
- Auto-apply toggle
- Terraform version pinning
- Variable sets assignment
- Destroy protection

**Destroy protection:** When enabled, `terraform destroy` requires explicit confirmation in the HCP Terraform UI. Adds a safeguard against accidental destruction.

**Variable sets:** Reusable collections of variables that can be applied to multiple workspaces or globally across an organization. Workspace-specific variables override variable set values.

---

### 1.6 HCP Terraform vs Open Source — Key Differences

| Feature | Open Source | HCP Terraform |
|---------|-------------|---------------|
| Remote state storage | Manual (S3, GCS, etc.) | Built-in |
| State locking | Manual (DynamoDB, etc.) | Built-in |
| Sentinel policies | ❌ | ✅ |
| Run queue management | ❌ | ✅ |
| Audit logging | ❌ | ✅ |
| SSO / SAML | ❌ | ✅ (Plus/Enterprise) |
| Private module registry | ❌ | ✅ |
| Cost estimation | ❌ | ✅ |

---

### 1.7 Private Module Registry

- HCP Terraform hosts a private registry for internal modules
- Modules are versioned and referenced like public registry modules
- Reference syntax: `<HOSTNAME>/<ORGANIZATION>/<MODULE_NAME>/<PROVIDER>`
- Supports VCS-backed module publishing

---

### 1.8 Import Block in HCP Terraform (Terraform 1.5+)

The `import` block is the correct way to import resources in remote execution mode:

```hcl
import {
  to = aws_db_instance.legacy
  id = "my-rds-instance-id"
}
```

- Lives in the codebase — version controlled, reviewable
- Executes within the normal HCP Terraform run pipeline
- Pre-1.5 workaround: switch workspace to local execution mode, run `terraform import` CLI, switch back

**Exam rule:** If question mentions Terraform 1.5+ and HCP Terraform remote execution → `import` block is the answer.

---

### 1.9 Teams and API Tokens

**HCP Terraform API token types:**

| Token type | Tied to | Scope | Use case |
|------------|---------|-------|---------|
| User token | Individual user | That user's permissions | Personal CLI use (`terraform login`) |
| Team token | A team | Team's workspace permissions | **CI/CD, automation, machine auth** |
| Organization token | The org | Org-wide management | Org administration only |

**Rules to lock in:**
- CI/CD integration (Jenkins, GitHub Actions, etc.) → **Team API token**. Scoped to team permissions, revocable independently of any user account.
- Creating a "service account user" for a machine = anti-pattern. Team tokens exist to avoid it.
- User tokens create a human dependency — engineer leaves, token must be rotated everywhere.
- Organization tokens are over-privileged for workspace operations. Never the answer for CI/CD.

**Exam pattern:** Machine authentication + scoped permissions + revocable independently of users = Team API token.

---

### 1.10 Projects

Projects are HCP Terraform's grouping layer between organization and workspaces:

```
Organization
    ↓
  Projects (group workspaces by team/domain)
    ↓
  Workspaces
```

**Key behavior:** Variable sets can be scoped to a project. All workspaces in the project — including ones created later — automatically inherit the variable set. Zero manual intervention per workspace.

**Variable set scoping options (complete list):**

| Scope | Auto-applies to |
|-------|----------------|
| Global | All workspaces in org |
| Project-scoped | All workspaces in that project, including future ones |
| Workspace-scoped | Only explicitly selected workspaces |

**Exam pattern:** "New workspaces for a team must automatically inherit X" → Projects + project-scoped variable set.

**Reminder from a costly miss:** Variable sets CAN be scoped — they are NOT always global. Any answer claiming variable sets "cannot be scoped" or "are always global" is false.

---

### 1.11 Speculative Plans

Triggered automatically when a pull request is opened/updated against a VCS-connected workspace.

**Defining properties:**
- Read-only — does NOT acquire a state lock
- Can NEVER be confirmed and applied — this is the defining characteristic
- Results posted to the PR as a status check
- Shows full plan output for review before merge

**Exam pattern:** "See what Terraform would do for a PR without affecting state" → speculative plan.

---

### 1.12 Run Queue — Re-Plan Behavior

When concurrent runs are triggered on the same workspace:
- Runs execute **serially**, never concurrently
- The second run is **queued**, not rejected (contrast: S3/DynamoDB → lock error)
- **Critical detail:** when a queued run reaches the front of the queue, HCP Terraform generates a **fresh plan against current state**. Queued runs never apply stale plans generated at trigger time.

This protects state consistency between queued runs. An answer saying a queued run applies "the plan it generated at trigger time, without re-planning" is wrong.

---

### 1.13 Built-In Audit Logging

HCP Terraform records all run activity automatically — no prior configuration or external integration required.

**Captured per run:** triggering user/VCS event, trigger source (UI/API/CLI/VCS), timestamp, full plan output, policy check results, auto-applied vs manually confirmed, who confirmed, final status.

**Access:** Workspace → Runs tab (per workspace); organization-level audit log stream on Plus/Enterprise tier.

**Exam pattern:** "Who applied what, when" → built-in HCP Terraform run history. Not state files (they don't record run metadata), not CloudTrail, not VCS webhook logs. External logging integrations are optional add-ons, not prerequisites for having an audit trail.

---

### 1.14 Workspace Deletion — No Recovery

**There is no workspace trash, no undo, no recovery window.** A deleted workspace and its state are permanently gone. (Resources in the cloud survive — only Terraform's tracking is lost.)

**Recovery path after accidental deletion:**
1. Create a new workspace with the same configuration
2. Write `import` blocks for all orphaned resources
3. `terraform plan` to verify imports
4. `terraform plan -refresh-only` to confirm state matches live infrastructure

**This is why destroy protection exists.** Enable it on every production workspace.

---

## SECTION 2: Core Workflow

### 2.1 Standard Workflow

```
terraform init
    ↓
terraform plan
    ↓
terraform apply
    ↓
terraform destroy (when needed)
```

---

### 2.2 `terraform init` Flags

| Flag | Purpose |
|------|---------|
| `-backend=false` | Skip backend init, use local state for session |
| `-migrate-state` | Migrate existing state to newly configured backend |
| `-reconfigure` | Reinitialize backend, ignore existing state (no migration) |
| `-upgrade` | Upgrade provider and module versions per version constraints |
| `-backend-config` | Pass partial backend config at init time (for sensitive values) |

**Critical distinction:**
- `-migrate-state` = move state to new backend
- `-reconfigure` = reinitialize backend, do NOT migrate state
- `-backend=false` = skip backend entirely, use local state

---

### 2.3 `terraform plan` Flags

| Flag | Purpose |
|------|---------|
| `-refresh-only` | Plan that only reconciles state with live infrastructure |
| `-refresh=false` | Skip refresh phase entirely (dangerous, use sparingly) |
| `-target=<resource>` | Scope plan to specific resource and dependencies |
| `-var="key=value"` | Pass variable value |
| `-var-file="file.tfvars"` | Pass variable file |
| `-out=<file>` | Save plan to file for deterministic apply |
| `-detailed-exitcode` | Exit code 2 = changes present, 0 = no changes, 1 = error |

---

### 2.4 `-refresh-only` Workflow (Drift Remediation)

Use when state has drifted from real infrastructure:

```
terraform plan -refresh-only     → inspect what drift exists
terraform apply -refresh-only    → update state to match live infra
terraform apply                  → apply any remaining config changes
```

**When state shows resources that don't exist in reality:**
- State recorded creation before API confirmed it (failed apply)
- Manual deletion outside Terraform
- Use `-refresh-only` to detect and reconcile

**Legacy command:** `terraform refresh` — deprecated in 0.15.4. Replaced by `-refresh-only` workflow. Still appears as a wrong answer option on the exam.

---

### 2.5 `-target` Flag

**Purpose:** Scope plan/apply/destroy to a specific resource.

**Risks:**
- Bypasses full dependency graph evaluation
- Does not refresh or reconcile unrelated resources
- Leaves state partially evaluated
- Subsequent full plans may surface unexpected diffs

**HashiCorp's official stance:** Use only for exceptional circumstances, not routine operations.

**`-lock=false`:** Skips acquiring a state lock for the operation. Does NOT remove existing locks.

---

### 2.6 `terraform destroy`

- Destroys all resources managed by the current configuration
- `terraform destroy -target=<resource>` destroys a specific resource
- `prevent_destroy = true` blocks explicit destroy operations
- `prevent_destroy` does **NOT** block resource replacement during apply

---

### 2.7 `terraform fmt` and `terraform validate`

| Command | Purpose |
|---------|---------|
| `terraform fmt` | Formats configuration files to canonical style |
| `terraform validate` | Checks syntax and internal configuration consistency |

**`terraform validate` does NOT:**
- Check against live infrastructure
- Enforce policy rules
- Validate provider credentials
- Require initialized backend

---

### 2.8 `terraform output`

- Displays output values from state
- `terraform output -json` outputs in JSON format
- Sensitive outputs are redacted in CLI display
- `sensitive = true` on outputs suppresses display only — value still in state

---

### 2.9 Saving and Using Plan Files

```bash
terraform plan -out=tfplan      # Save plan
terraform apply tfplan           # Apply saved plan deterministically
terraform show tfplan            # Inspect saved plan
terraform show -json tfplan      # JSON representation of plan
```

Saved plan files guarantee that exactly what was reviewed gets applied — no drift between plan and apply.

**Without a saved plan file:** `terraform apply` generates a **fresh plan at apply time** before prompting for confirmation. It never reuses previous plan output. If infrastructure changed between the reviewed plan and the apply, the apply reflects the new reality — not what was reviewed. Plans do not expire and are not cached; there is simply no determinism without `-out`.

---

### 2.10 Forcing Resource Replacement — `-replace` vs `taint`

**Current command (Terraform 0.15.2+):**
```bash
terraform apply -replace="aws_instance.web"
```
Plans and applies a destroy-and-recreate of the specified resource in one reviewed operation.

**`terraform taint` is deprecated.** If it appears as an answer option, it is almost always the trap.

**Critical distinction — `-replace` evaluates the FULL dependency graph:**

| | `-replace` | `-target` |
|---|-----------|----------|
| Dependency graph | Fully evaluated | Restricted to target + deps |
| Other resources | Refreshed, included in plan | Skipped |
| Downstream ripple effects | Surfaced in plan | Hidden |
| State consistency after | Clean | Potentially partial |

`-replace` changes *what happens to one resource* within a full plan. `-target` changes *the scope of the plan itself*. Only `-target` compromises the dependency graph.

**Also wrong for forced replacement:**
- `terraform state rm` + apply → creates a duplicate; the original keeps running unmanaged
- Manual console deletion + apply → works mechanically but is the out-of-band anti-pattern; the exam penalizes console answers when a native Terraform mechanism exists

---

## SECTION 3: IaC Concepts

### 3.1 Infrastructure as Code Benefits

| Benefit | Description |
|---------|-------------|
| Consistency | Same config produces same infrastructure every time |
| Repeatability | Environments can be reproduced exactly |
| Version control | Infrastructure changes tracked like code |
| Automation | Reduces manual, error-prone operations |
| Self-documenting | Config describes the infrastructure |
| Collaboration | Teams can review, approve, and audit changes |

---

### 3.2 Declarative vs Imperative

| Approach | Description | Example |
|----------|-------------|---------|
| Declarative | Define desired end state; tool figures out how | Terraform, CloudFormation |
| Imperative | Define step-by-step instructions | Ansible scripts, shell scripts |

**Terraform is declarative.** You describe what you want, not how to get there.

---

### 3.3 Mutable vs Immutable Infrastructure

| Type | Description | Risk |
|------|-------------|------|
| Mutable | Infrastructure modified in place | Configuration drift over time |
Immutable | Infrastructure replaced entirely on change | No drift, predictable state |

**Terraform favors immutable infrastructure** — resources are destroyed and recreated when attributes change, rather than modified in place (unless the provider supports in-place updates).

---

### 3.4 Terraform's Place in the IaC Ecosystem

- **Provisioning tool** — provisions infrastructure resources
- **Not a configuration management tool** — does not manage software inside servers (that's Ansible, Chef, Puppet)
- **Not a deployment tool** — does not deploy application code (that's Kubernetes, Helm, etc.)
- Can be combined with configuration management tools via provisioners (use sparingly)

---

### 3.5 Terraform vs Other IaC Tools

| Tool | Type | Cloud |
|------|------|-------|
| Terraform | Declarative, multi-cloud | Any |
| CloudFormation | Declarative | AWS only |
| Pulumi | Imperative (real code) | Any |
| Ansible | Imperative/procedural | Any |
| ARM Templates | Declarative | Azure only |

---

## SECTION 4: Terraform Fundamentals

### 4.1 Terraform Architecture

```
Configuration files (.tf)
        ↓
    Terraform Core
        ↓
    Provider plugins
        ↓
    Cloud/service APIs
```

**Providers:** Plugins that interact with cloud provider APIs. Each provider is downloaded during `terraform init`.

---

### 4.2 Provider Version Constraints

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

**Version constraint operators:**

| Operator | Meaning |
|----------|---------|
| `= 5.0.0` | Exactly 5.0.0 |
| `!= 5.0.0` | Not 5.0.0 |
| `> 5.0.0` | Greater than 5.0.0 |
| `>= 5.0.0` | 5.0.0 or higher |
| `~> 5.0` | 5.x but not 6.0 (pessimistic constraint) |
| `~> 5.0.0` | 5.0.x but not 5.1.0 |

**`.terraform.lock.hcl`:** Records exact provider versions used. Should be committed to version control. Ensures all team members use identical provider versions.

---

### 4.3 Module Sources

| Source type | Example |
|-------------|---------|
| Public registry | `hashicorp/consul/aws` |
| Private registry | `app.terraform.io/org/module/aws` |
| GitHub | `github.com/org/repo` |
| Local path | `./modules/vpc` |
| S3 bucket | `s3::https://bucket/module.zip` |
| Git URL | `git::https://github.com/org/repo.git` |

---

### 4.4 Variable Precedence (Local Execution — Highest to Lowest)

1. `-var` command line flag
2. `-var-file` command line flag
3. `*.auto.tfvars` files (alphabetical order)
4. `terraform.tfvars.json`
5. `terraform.tfvars`
6. Environment variables (`TF_VAR_<name>`)
7. Default values in variable declarations

**Exam trap:** `TF_VAR_` environment variables are near the **bottom** of precedence — not the top. `-var` flag is highest.

---

### 4.5 Resource Lifecycle Meta-Arguments

```hcl
lifecycle {
  create_before_destroy = true   # Create replacement before destroying original
  prevent_destroy       = true   # Block explicit destroy operations (NOT replacement)
  ignore_changes        = [attr] # Ignore changes to specific attributes
  replace_triggered_by  = [res]  # Force replacement when another resource changes
}
```

**`prevent_destroy` scope:**
- Blocks: `terraform destroy`, plans containing explicit resource destroy
- Does NOT block: resource replacement during apply, `terraform state rm`

**`ignore_changes = all`:** Terraform will never update the resource after initial creation regardless of config drift.

---

### 4.6 State Commands Reference

| Command | Purpose |
|---------|---------|
| `terraform state list` | List all resources in state |
| `terraform state show <resource>` | Show attributes of a resource in state |
| `terraform state mv <src> <dst>` | Move/rename resource in state |
| `terraform state rm <resource>` | Remove resource from state tracking |
| `terraform state pull` | Download and display current state |
| `terraform state push` | Upload a local state file to remote backend |
| `terraform force-unlock <ID>` | Release a stale state lock |

**`state rm` does NOT destroy infrastructure** — it only removes Terraform's tracking of the resource.

---

### 4.7 Backend Types

| Backend | State storage | Locking mechanism |
|---------|--------------|-------------------|
| local | Local filesystem | Local file lock |
| s3 | AWS S3 | DynamoDB table |
| gcs | Google Cloud Storage | Built-in GCS locking |
| azurerm | Azure Blob Storage | Built-in blob leasing |
| http | HTTP endpoint | Optional via lock/unlock endpoints |
| HCP Terraform | HCP Terraform | Built-in |

---

### 4.8 `terraform_remote_state` Data Source

Used to read outputs from another Terraform state file:

```hcl
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = "company-tfstate"
    key    = "vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

# Reference outputs
resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.vpc.outputs.subnet_id
}
```

**Requirement:** The source module must explicitly declare `output` blocks for values to be accessible.

---

### 4.9 `moved` Block

Introduced in Terraform 1.1. Declarative way to rename resources in state without destroying and recreating.

```hcl
moved {
  from = aws_instance.web
  to   = aws_instance.web_server
}
```

**Behavior:**
- `terraform plan` shows move operation, no destroy/create
- `terraform apply` updates state with new resource address
- Can be removed after all workspaces have applied the change
- Risk: removing before all team members apply causes destroy/create on stale workspaces

**vs `terraform state mv`:**
- `moved` block = declarative, version controlled, part of plan/apply
- `terraform state mv` = imperative CLI, immediate, no code record

---

### 4.10 `import` Block (Terraform 1.5+)

Declarative resource import integrated into plan/apply workflow:

```hcl
import {
  to = aws_instance.legacy
  id = "i-1234567890abcdef0"
}
```

**vs `terraform import` CLI:**
- `import` block = declarative, version controlled, works in remote execution
- `terraform import` CLI = imperative, requires local execution mode in HCP Terraform

---

### 4.11 The Reference Trio — `data` vs `terraform_remote_state` vs `import`

Three mechanisms, three different jobs. The exam swaps these scenarios deliberately:

| Need | Mechanism | Ownership |
|------|-----------|-----------|
| Read attributes of a resource created **outside Terraform** | `data` source | None — read-only lookup |
| Read **outputs** from another Terraform configuration's state | `terraform_remote_state` | None — read-only, requires source to declare `output` blocks |
| **Take over management** of an existing resource | `import` block | Full — Terraform now owns its lifecycle |

**How to pick on the exam — check the provenance:**
- "Created manually / by another team outside Terraform, must not manage" → `data` source
- "Managed by another Terraform configuration / workspace" → `terraform_remote_state`
- "Bring under Terraform management" → `import`

**Never:** writing a `resource` block matching an existing resource hoping Terraform will "detect and skip" — Terraform never detects-and-skips; a resource block always means *manage this*, and apply will attempt creation.

---

### 4.12 The Three-Way Model

Every plan question resolves by comparing three things:

```
Configuration (.tf files)  +  State (.tfstate)  +  Live Infrastructure
```

| Config | State | Live | Plan shows |
|--------|-------|------|-----------|
| Defines it | Has it | Exists | No changes |
| Defines it | Missing | Missing | Create |
| Defines it | Has it | Missing | Create (after refresh detects) |
| Removed | Has it | Exists | Destroy |
| Defines it (changed) | Has it (old) | Exists (old) | Update or Replace |

**Key principles:**
- State updates **incrementally** as each operation succeeds — never as an atomic batch, never with rollback. Partial applies and partial destroys leave state reflecting exactly what completed.
- After a partial destroy: destroyed resources are gone from state AND infra, but config still defines them → a plain `terraform plan` shows them as **creates**, not pending destroys.

---

## QUICK REFERENCE — Common Exam Traps

| Trap | Correct answer |
|------|---------------|
| `sensitive = true` encrypts state | FALSE — display only, state still plaintext |
| `terraform refresh` is current best practice | FALSE — deprecated, use `-refresh-only` |
| `-var` flag works in HCP Terraform remote execution | FALSE — ignored in remote execution |
| `prevent_destroy` blocks resource replacement | FALSE — only blocks explicit destroy |
| `terraform validate` enforces policy | FALSE — syntax/consistency only |
| `state rm` permanently purges sensitive data | FALSE — old state versions persist in S3 |
| Variable sets take highest precedence in HCP Terraform | FALSE — workspace variables win |
| `-target` evaluates full dependency graph | FALSE — scoped evaluation only |
| HCP Terraform queues concurrent applies | TRUE |
| S3/DynamoDB backend fails on concurrent applies | TRUE — lock error |
| `TF_VAR_` env vars are highest precedence | FALSE — near lowest, `-var` flag is highest |
| `moved` block must stay permanently | FALSE — removable after all workspaces apply |
| Override files can override backend config | FALSE — backend config cannot be overridden |
| Variable sets are always global / cannot be scoped | FALSE — global, project, or workspace scope |
| Admins can override hard mandatory Sentinel policies | FALSE — nobody can, fix the code |
| Remote execution forwards local env vars to the runner | FALSE — runner has no visibility into local machine |
| Queued runs apply the plan from trigger time | FALSE — re-plan against current state before applying |
| Deleted workspaces can be restored within 24 hours | FALSE — no recovery, ever |
| `terraform taint` is the current way to force replacement | FALSE — deprecated, use `apply -replace` |
| `-replace` skips dependency evaluation like `-target` | FALSE — full graph evaluated |
| `terraform apply` reuses the last plan output | FALSE — fresh plan unless a saved plan file is given |
| Plan outputs expire after a time limit | FALSE — no expiry, but no caching either |
| Terraform rolls back partial applies/destroys | FALSE — state updates incrementally, no rollback |
| A resource block "detects and skips" existing resources | FALSE — resource block always means manage/create |
| `terraform fmt` validates configuration | FALSE — formatting only, no logic checks |
| `terraform validate` works without init | FALSE — needs provider schemas from init |
| Init installs newest provider version within constraints | FALSE — lock file pins it until `-upgrade` |
| Lock files are machine-specific, can't be shared | FALSE — designed to be committed and shared |
| `depends_on` should be declared on every dependency | FALSE — references create implicit deps; depends_on is for hidden deps only |

---

## EXAM TECHNIQUE — The Two Habits

**1. The absolute-language scan.** Before selecting, scan options for: *always, never, cannot, only, all, entirely, eliminates, impossible, regardless*. When found, hunt for ONE counterexample — one is enough to kill the option. Correct answers usually use qualified language: *commonly, typically, can be, recommended*. Caution: some absolutes ARE true (hard mandatory cannot be overridden; no workspace recovery). The rule is not "absolute = wrong" — it's "absolute = stop and verify."

**2. The full read.** Read all four options completely, every question — especially when an early option looks immediately right. The "I know this one" impulse mid-read is the signal to slow down, not speed up. ~1 minute per question is enough; the time pressure is perceived, not real.

---

*Study guide built for Terraform Associate 004 retake — June 21st*
*Focus sequence: HCP Terraform → Core Workflow → IaC Concepts → Fundamentals*
