# Daily Command Drill — Setup & Routine
**Terraform Associate 004 | Run every morning through June 19th**

---

## PART 1: One-Time Setup (~5 minutes)

### Step 1 — Create the scratch directory

```bash
mkdir ~/tf-drill
cd ~/tf-drill
```

### Step 2 — Create the configuration

Create `main.tf` with exactly this content:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    random = {
      source  = "hashicorp/random"
      version = "~> 3.6"
    }
  }
}

variable "name_length" {
  type    = number
  default = 2

  validation {
    condition     = var.name_length >= 1 && var.name_length <= 5
    error_message = "Name length must be between 1 and 5."
  }
}

resource "random_pet" "server" {
  length = var.name_length
}

resource "random_id" "suffix" {
  byte_length = 4

  keepers = {
    pet_name = random_pet.server.id
  }
}

resource "random_integer" "port" {
  count = 3
  min   = 8000
  max   = 9000
}

output "server_name" {
  value       = random_pet.server.id
  description = "Generated server name"
}

output "bucket_suffix" {
  value = random_id.suffix.hex
}

output "ports" {
  value = random_integer.port[*].result
}
```

This config deliberately includes: a variable with validation, an implicit dependency
(`random_id.suffix` references `random_pet.server.id` via keepers), a `count` resource,
and multiple outputs — so the drill exercises real exam concepts, not just commands.

### Step 3 — Initialize once

```bash
terraform init
```

**Predict before running:** What gets created? → `.terraform/` directory (provider binary)
and `.terraform.lock.hcl` (version pins).

Verify:

```bash
ls -la
cat .terraform.lock.hcl
```

Setup done.

---

## PART 2: The Daily Drill (~10 minutes)

Run this exact sequence every morning. **The rule: say your predicted output OUT LOUD
before pressing enter.** Wrong prediction = gap found = note it.

### Round 1 — Hygiene

```bash
terraform fmt -check
```
*Predict: exit silently (clean) or list files needing format?*

```bash
terraform validate
```
*Predict: success. Why does this need init to have run? (provider schemas)*

### Round 2 — Plan & Apply

```bash
terraform plan -out=tfplan
```
*Predict: how many resources to add? (Count them in the config — should be 5:
1 pet + 1 id + 3 integers)*

```bash
terraform show tfplan
```
*Predict: same plan, human-readable. What does the saved plan guarantee that
a plain apply doesn't? (determinism — what was reviewed is what applies)*

```bash
terraform apply tfplan
```
*Predict: no confirmation prompt. Why? (saved plan file = pre-approved)*

### Round 3 — State Inspection

```bash
terraform state list
```
*Predict the exact addresses: random_pet.server, random_id.suffix,
random_integer.port[0], [1], [2]*

```bash
terraform state show random_pet.server
```
*Predict: attributes including id (the generated name) and length*

```bash
terraform output
```
*Predict: all three outputs with values*

```bash
terraform output -raw server_name
```
*Predict: just the name, no quotes — what's -raw for? (scripting)*

### Round 4 — Console

```bash
terraform console
```

Inside, evaluate (predict each first):

```
length(["a","b","c"])
cidrsubnet("10.0.0.0/16", 8, 2)
merge({a=1}, {b=2})
random_pet.server.id
lookup({x="1"}, "y", "default")
toset(["a","a","b"])
exit
```

### Round 5 — Replace & Idempotency

```bash
terraform plan
```
*Predict: NO CHANGES. Why doesn't random re-randomize? (value stored in state,
state matches config)*

```bash
terraform apply -replace="random_pet.server" -auto-approve
```
*Predict: pet replaced — AND what else? (random_id.suffix also replaced, because
its keepers reference the pet's id — implicit dependency cascade. This is -replace
evaluating the full graph.)*

```bash
terraform output server_name
```
*Predict: a NEW name*

### Round 6 — Targeted State Ops (every 2-3 days, not daily)

```bash
terraform state rm random_integer.port[2]
terraform plan
```
*Predict: 1 to add. Why? (config still declares 3, state now tracks 2 —
the three-way model)*

```bash
terraform apply -auto-approve
```

### Round 7 — Teardown

```bash
terraform destroy -auto-approve
```
*Predict: 5 destroyed (or 6 if you did Round 6 — why?)*

```bash
terraform state list
```
*Predict: empty*

---

## PART 3: Rotating Extras (one per day)

Pick ONE each day on top of the core drill:

**Day A — Exit codes:**
```bash
terraform plan -detailed-exitcode; echo "Exit: $?"
# Predict with no changes pending: 0
terraform apply -replace="random_pet.server" -auto-approve >/dev/null 2>&1 || true
terraform plan -detailed-exitcode; echo "Exit: $?"
# After making a pending change, predict: 2
```

**Day B — Logging:**
```bash
TF_LOG=DEBUG terraform plan 2>debug.log; wc -l debug.log
TF_LOG_PATH=./never.log terraform plan; ls never.log
# Predict: never.log does NOT exist — TF_LOG_PATH alone does nothing
TF_LOG=DEBUG TF_LOG_PATH=./yes.log terraform plan; ls yes.log
# Predict: yes.log exists
rm -f debug.log yes.log never.log
```

**Day C — Workspaces:**
```bash
terraform workspace list
terraform workspace new staging
terraform workspace show
terraform plan        # Predict: 5 to add — fresh empty state per workspace
terraform workspace select default
terraform workspace delete staging
```

**Day D — Variables & precedence:**
```bash
echo 'name_length = 3' > terraform.tfvars
TF_VAR_name_length=4 terraform plan -var="name_length=1"
# Predict which value wins: 1 (the -var flag). Check the plan's pet length.
rm terraform.tfvars
```

**Day E — Graph & lock:**
```bash
terraform graph | head -20
# DOT format — note the suffix → pet dependency edge
terraform providers
cat .terraform.lock.hcl
rm -rf .terraform && terraform plan
# Predict: init-required error. Lock file survives, .terraform/ is the artifact.
terraform init
```

---

## What Each Round Maps To (your actual misses)

| Drill element | Exam gap it closes |
|---------------|--------------------|
| Round 4 console | Mixed run Q5 |
| Day A exit codes | Mixed run Q6 |
| Day E rm .terraform | Mixed run Q14 |
| Round 5 replace cascade | -replace vs -target graph behavior |
| Round 6 state rm | Three-way model |
| Day B logging | TF_LOG / TF_LOG_PATH dependency |
| Day D precedence | Local variable precedence order |

---

*Ten minutes a morning. Predict out loud, then press enter. Wrong prediction = found gap = exam point saved.*
