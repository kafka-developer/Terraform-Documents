# HCP Terraform — Hands-On Lab Guide
**Terraform Associate 004 Retake Prep | June 21st**

> **Prerequisites before starting any lab:**
> - HCP Terraform account (free tier at app.terraform.io)
> - Terraform CLI installed locally (1.5+)
> - AWS account with IAM credentials (free tier sufficient)
> - Git installed locally
> - `terraform login` completed successfully

---

## Lab 1.1 — Execution Modes (Remote vs Local)

**Objective:** Observe the difference between remote and local execution mode behavior firsthand.

**What you will prove:** Remote execution runs on HCP Terraform infrastructure. Local execution runs on your machine.

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

Create a new directory on your machine:

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
The plan is executing on HCP Terraform's servers, not your machine.

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

**Observe:** Plan now runs locally on your machine. No streaming message. Output is immediate.

---

### Step 5 — Key observation

Note what changed and what didn't:
- State is still stored remotely in HCP Terraform in both modes
- Only WHERE the plan/apply executes changed
- In remote mode: HCP Terraform needs your provider credentials as workspace variables
- In local mode: your local environment variables/credentials are used

**Exam checkpoint:** Remote execution = HCP infra runs it. Local execution = your machine runs it. State storage location does not change.

---

## Lab 1.2 — Variable Precedence in HCP Terraform (Remote Execution)

**Objective:** Prove that `-var` flags are ignored in remote execution mode and workspace variables take precedence over variable sets.

---

### Step 1 — Reset workspace to Remote execution mode

1. Go to workspace Settings → General
2. Set Execution Mode back to **Remote**
3. Save

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

1. In HCP Terraform → go to **Organization Settings → Variable Sets**
2. Click **Create variable set**
3. Name it `lab-varset`
4. Under Variables, add:
   - Key: `environment`
   - Value: `from_variable_set`
   - Type: Terraform variable
5. Under **Scope**, apply it to workspace `lab-execution-modes`
6. Save

---

### Step 4 — Run plan and observe Variable Set value

```bash
terraform plan
```

**Observe output:**
```
env_output = "from_variable_set"
```

The variable set value is being used.

---

### Step 5 — Add a Workspace-Specific Variable

1. Go to workspace `lab-execution-modes` → **Variables**
2. Add a Terraform variable:
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

Workspace variable overrides the variable set. Variable set value is ignored.

---

### Step 7 — Attempt to override with `-var` flag

```bash
terraform plan -var="environment=from_local_var_flag"
```

**Observe output:**
```
env_output = "from_workspace_variable"
```

The `-var` flag value `from_local_var_flag` is completely ignored. Workspace variable still wins.

> If you see a warning like `Warning: Ignored -var flag in remote operations`, that confirms the behavior.

---

### Step 8 — Key observations

Write these down:

| Source | Value set | Result |
|--------|-----------|--------|
| Variable set | `from_variable_set` | Overridden |
| Workspace variable | `from_workspace_variable` | **Used** |
| `-var` flag | `from_local_var_flag` | **Ignored** |

**Exam checkpoint:** In remote execution mode, workspace variable > variable set > `-var` flag (ignored).

---

## Lab 1.3 — Run Pipeline (Remote Execution)

**Objective:** Walk through each stage of the HCP Terraform run pipeline and observe the run states.

---

### Step 1 — Enable Auto-Apply (optional for faster observation)

1. Workspace Settings → General
2. Toggle **Auto Apply** ON
3. Save

> Leave this OFF if you want to manually confirm each apply — gives you more time to observe pipeline stages.

---

### Step 2 — Trigger a plan and observe pipeline stages

```bash
terraform plan
```

Or trigger via UI: workspace → **Actions → Start new run → Plan only**

**Observe in HCP Terraform UI the run moving through stages:**

```
Pending
  ↓
Planning
  ↓
Cost Estimation (if enabled)
  ↓
Policy Check (if Sentinel configured)
  ↓
Planned (awaiting confirmation)
  ↓
Applying
  ↓
Applied
```

Click on each stage in the UI to see its output and duration.

---

### Step 3 — Observe a queued run

1. Trigger a plan from the CLI:
```bash
terraform plan
```

2. While it is still running, immediately trigger another run from the HCP Terraform UI:
   - Workspace → **Actions → Start new run**

3. **Observe:** The second run shows status **Pending** while the first run completes. It does not fail. It waits in queue.

**Exam checkpoint:** HCP Terraform queues concurrent runs. It does NOT fail with a lock error like S3/DynamoDB backends do.

---

### Step 4 — Observe a discarded run

1. Trigger a plan with auto-apply OFF
2. When the plan completes and shows **Planned — Awaiting Confirmation**
3. Click **Discard run** instead of confirming

**Observe:** Run moves to **Discarded** state. No apply happens. Infrastructure unchanged.

---

### Step 5 — Review run history

In the workspace → **Runs** tab:

- Every run is logged with timestamp, triggered by, status, and duration
- Click any run to see full plan output, policy check results, apply logs
- This is the audit trail HCP Terraform maintains automatically

**Exam checkpoint:** Run history and audit logging are built-in HCP Terraform features not available in open source Terraform.

---

## Lab 1.4 — Sentinel Policy Enforcement

**Objective:** Create and enforce a Sentinel policy that blocks a non-compliant resource.

> **Note:** Sentinel requires HCP Terraform Plus tier or trial. If unavailable, study the concepts and skip to step 6 for the mock observation.

---

### Step 1 — Create a policy set in HCP Terraform

1. Organization Settings → **Policy Sets**
2. Click **Connect a new policy set**
3. Choose **No VCS connection** (manual upload for lab)
4. Name it `lab-enforce-tags`
5. Scope to workspace `lab-execution-modes`

---

### Step 2 — Write a Sentinel policy

Create file `enforce-tags.sentinel` locally:

```python
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

### Step 3 — Upload policy and set enforcement level

1. Upload `enforce-tags.sentinel` to the policy set
2. Set enforcement level to **Soft Mandatory**
3. Save

---

### Step 4 — Add a non-compliant resource to your config

Add to `main.tf`:

```hcl
resource "aws_instance" "test" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  # No tags — this will fail the Sentinel policy
}
```

---

### Step 5 — Trigger a plan and observe policy check failure

```bash
terraform plan
```

**Observe in HCP Terraform UI:**
- Plan completes
- Policy Check stage shows **FAILED**
- Run is blocked at the policy check stage
- Since enforcement is Soft Mandatory, an authorized user can override

---

### Step 6 — Observe enforcement level behavior (conceptual if no Plus tier)

| Enforcement Level | Policy fails | What happens |
|------------------|-------------|--------------|
| Advisory | Yes | Warning logged, run proceeds to apply |
| Soft Mandatory | Yes | Run blocked, admin can override and proceed |
| Hard Mandatory | Yes | Run blocked, NO override possible, period |

**Fix the policy violation** by adding tags:

```hcl
resource "aws_instance" "test" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Environment = "lab"
  }
}
```

Run plan again — policy check should pass.

---

## Lab 1.5 — Workspace Variables and Variable Sets (Deep Dive)

**Objective:** Understand the full variable hierarchy and variable set scoping.

---

### Step 1 — Create a second workspace

1. New workspace → CLI-driven → name it `lab-varset-test`
2. Remote execution mode

---

### Step 2 — Apply your variable set to both workspaces

1. Organization Settings → Variable Sets → `lab-varset`
2. Edit scope → add `lab-varset-test` workspace
3. Save

---

### Step 3 — Create a global variable set

1. New variable set → name `global-credentials`
2. Add AWS credentials as **environment variables**:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
3. Set scope to **Apply to all workspaces**
4. Save

**Observe:** Both workspaces now have AWS credentials available without configuring them per workspace. This is the correct pattern for managing provider credentials in HCP Terraform.

---

### Step 4 — Verify variable set vs workspace variable override

In `lab-varset-test`:
- Variable set has `environment = from_variable_set`
- Do NOT add a workspace-specific variable

Run plan:
```bash
# Switch workspace context
terraform workspace select lab-varset-test
terraform plan
```

**Observe:** `environment = "from_variable_set"` — variable set value is used since no workspace variable overrides it.

---

## Lab 1.6 — Destroy Protection

**Objective:** Enable destroy protection and observe it blocking a `terraform destroy`.

---

### Step 1 — Enable destroy protection

1. Workspace `lab-execution-modes` → Settings → General
2. Enable **Destroy Protection**
3. Save

---

### Step 2 — Attempt terraform destroy

```bash
terraform destroy
```

**Observe:** The destroy is blocked. HCP Terraform UI shows the run requires explicit confirmation to proceed. A warning is displayed that destroy protection is enabled.

---

### Step 3 — Compare with `prevent_destroy` lifecycle

Add to a resource in `main.tf`:

```hcl
lifecycle {
  prevent_destroy = true
}
```

Run:
```bash
terraform destroy
```

**Observe:** Plan fails with error:
```
Error: Instance cannot be destroyed
```

**Key difference:**

| Mechanism | Scope | Where enforced |
|-----------|-------|----------------|
| `prevent_destroy` | Single resource | Terraform plan/apply |
| Workspace destroy protection | Entire workspace | HCP Terraform UI |

---

## Lab 1.7 — `import` Block (Terraform 1.5+)

**Objective:** Import a manually created resource into Terraform state using the declarative `import` block in remote execution mode.

---

### Step 1 — Manually create a resource outside Terraform

In AWS console (or CLI), create an S3 bucket manually:

```bash
aws s3 mb s3://my-lab-import-bucket-<unique-suffix>
```

Note the bucket name.

---

### Step 2 — Write the resource block and import block

Add to `main.tf`:

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

### Step 3 — Run plan and observe import in pipeline

```bash
terraform plan
```

**Observe in HCP Terraform UI:**
- Plan shows: `aws_s3_bucket.imported will be imported`
- No destroy/create planned
- The import executes as part of the remote run pipeline

---

### Step 4 — Apply to complete the import

```bash
terraform apply
```

**Observe:**
- Resource is now tracked in HCP Terraform state
- Run in the UI shows import completed successfully

---

### Step 5 — Remove the import block

After a successful apply, remove the `import` block from `main.tf`:

```hcl
# Remove this block after successful import
# import {
#   to = aws_s3_bucket.imported
#   id = "my-lab-import-bucket-<unique-suffix>"
# }
```

Run plan — should show no changes. Resource is now fully managed by Terraform.

**Exam checkpoint:** `import` block is consumed after apply. Remove it from config afterward. It is not a permanent fixture.

---

## Lab 1.8 — State Lock Behavior: HCP Terraform vs S3/DynamoDB

**Objective:** Observe and contrast concurrent run behavior between HCP Terraform and S3/DynamoDB backends.

---

### Part A — HCP Terraform (Remote Execution)

#### Step 1 — Trigger two concurrent runs

1. From CLI: `terraform plan` (let it run)
2. While running, go to HCP Terraform UI → Actions → Start new run

**Observe:** Second run shows **Pending** status. It is queued. It does not fail.

---

### Part B — S3/DynamoDB Backend (Conceptual + Optional CLI test)

If you have an S3 backend configured separately:

#### Step 1 — Simulate a stale lock

```bash
# Terminal 1 — start an apply
terraform apply

# Terminal 2 — while apply is running, attempt another plan
terraform plan
```

**Observe in Terminal 2:**
```
Error: Error acquiring the state lock
Lock Info:
  ID: <lock-id>
  ...
```

#### Step 2 — Force unlock

```bash
terraform force-unlock <LOCK_ID>
```

Confirm the prompt. Lock is released.

**Key contrast:**

| Backend | Concurrent run behavior |
|---------|------------------------|
| HCP Terraform | Queued automatically |
| S3 + DynamoDB | Fails with lock error — must `force-unlock` |

---

## Lab Summary — What You Proved

| Lab | Key concept validated |
|-----|----------------------|
| 1.1 | Remote vs local execution mode — where plan runs |
| 1.2 | Variable precedence — workspace var > variable set > `-var` flag ignored |
| 1.3 | Run pipeline stages — pending, planning, policy check, apply, queued |
| 1.4 | Sentinel enforcement levels — advisory, soft mandatory, hard mandatory |
| 1.5 | Variable sets — global vs scoped, workspace override behavior |
| 1.6 | Destroy protection — workspace level vs `prevent_destroy` resource level |
| 1.7 | `import` block — declarative import in remote execution pipeline |
| 1.8 | Concurrent runs — HCP Terraform queues vs S3/DynamoDB lock error |

---

## Cleanup

When labs are complete, destroy all provisioned resources:

```bash
terraform destroy
```

Delete manually created S3 bucket:

```bash
aws s3 rb s3://my-lab-import-bucket-<unique-suffix> --force
```

Delete workspaces in HCP Terraform UI if no longer needed.

---

*Lab guide built for Terraform Associate 004 retake — June 21st*
*Pair with: terraform_004_study_guide.md*
