# Terraform Associate (004) — Complete CLI Command Reference

Every command, subcommand, and flag tested on the 004 exam — organized by exam domain, with the gotchas that cost points.

*Prepared for Prateek Shukla — Retake: July 5, 2026*

---

## 1. Core Workflow Commands

These four commands are the backbone of every Terraform operation and appear constantly across Fundamentals, Core Workflow, and Basics domains.

### `terraform init`

Initializes a working directory: downloads providers, installs modules, configures the backend.

| Flag / Subcommand | Effect |
|---|---|
| `-upgrade` | Re-evaluates version constraints and upgrades providers/modules to the latest allowed version. Also rewrites `.terraform.lock.hcl`. |
| `-reconfigure` | Reconfigures the backend, ignoring any saved configuration. Does NOT migrate existing state. |
| `-migrate-state` | Reconfigures the backend AND migrates existing state to the new backend. |
| `-backend-config=<path\|key=value>` | Supplies backend configuration partially or fully at init time — common for CI/CD where backend secrets aren't hardcoded. |
| `-input=false` | Disables interactive prompts — required for non-interactive CI pipelines. |
| `-get=false` | Skips downloading modules during init. |

> ⚠️ **EXAM NOTE:** `init` must be re-run after adding/removing providers, modules, or changing backend config. Forgetting this is a common exam trap question.

### `terraform plan`

Creates an execution plan, showing what actions Terraform will take without making any changes.

| Flag / Subcommand | Effect |
|---|---|
| `-out=<path>` | Saves the plan to a file for later use with apply. Without this, the plan is not guaranteed to apply identically (target infra may drift between plan and apply). |
| `-detailed-exitcode` | Returns 0 (no changes), 1 (error), or 2 (changes present, no errors). Used in CI/CD to detect drift programmatically. |
| `-refresh=false` | Skips querying real infrastructure state before planning — faster but may miss drift. |
| `-refresh-only` | Plan mode that ONLY reconciles state with real infrastructure; proposes no resource changes. |
| `-replace=<resource>` | Forces a resource to be destroyed and recreated in the next apply (replaces deprecated taint workflow). |
| `-target=<resource>` | Limits the plan to only the specified resource and its dependencies. NOT recommended for routine use — can cause state drift. |
| `-var` / `-var-file` | Supplies variable values at the CLI, overriding `.tfvars` and environment variables. |
| `-destroy` | Creates a plan to destroy all resources, equivalent to previewing `terraform destroy`. |

> ⚠️ **EXAM NOTE:** `-detailed-exitcode` and `-replace` are very frequently tested — know the exact exit code meanings (0/1/2) cold.

### `terraform apply`

Executes the actions in a plan to reach the desired state.

| Flag / Subcommand | Effect |
|---|---|
| `terraform apply` | Generates a fresh plan, prompts for approval, then applies. |
| `terraform apply <planfile>` | Applies an exact, previously saved plan — no re-planning, no approval prompt. |
| `-auto-approve` | Skips the interactive yes/no confirmation prompt. Only valid on apply, never on plan. |
| `-replace=<resource>` | Same as plan — forces destroy/recreate of a specific resource within this apply. |
| `-target=<resource>` | Limits apply scope to a specific resource. |
| `-parallelism=<n>` | Controls the number of concurrent resource operations (default 10). |

> ⚠️ **EXAM NOTE:** Terraform apply is NOT transactional/atomic. If it fails partway through, successfully created resources remain in state — failed ones do not. Always re-run plan after a failed apply.

### `terraform destroy`

Destroys all resources managed by the current configuration/workspace.

| Flag / Subcommand | Effect |
|---|---|
| `-target=<resource>` | Destroys only the specified resource (and dependents). |
| `-auto-approve` | Skips confirmation prompt. |

> ⚠️ **EXAM NOTE:** `prevent_destroy` lifecycle argument will block this command (and apply) entirely if set to `true` on a resource.

---

## 2. Configuration & Validation Commands

This domain (Terraform Configuration) covers syntax checking, formatting, and configuration introspection.

### `terraform validate`

Checks whether a configuration is syntactically valid and internally consistent — does NOT check against real infrastructure or provider credentials.

| Flag / Subcommand | Effect |
|---|---|
| `-json` | Outputs validation results in machine-readable JSON, useful in CI pipelines. |
| No flags needed normally | Run after init — providers must be installed first or validate will fail on provider-specific schema checks. |

> ⚠️ **EXAM NOTE:** `validate` catches syntax & type errors only. It will NOT catch a wrong AMI ID or an IAM permission issue — that's only caught at plan/apply when the provider API is called.

### `terraform fmt`

Rewrites configuration files to a canonical style (indentation, alignment, spacing).

| Flag / Subcommand | Effect |
|---|---|
| `-recursive` | Formats files in subdirectories too (default is current directory only). |
| `-check` | Returns a non-zero exit code if files are not already formatted — does not rewrite. Used in CI to enforce formatting. |
| `-diff` | Displays the diff of formatting changes without necessarily writing them (combine with `-check`). |

### `terraform console`

Opens an interactive REPL for evaluating expressions: variables, locals, resource attributes, and built-in functions.

| Flag / Subcommand | Effect |
|---|---|
| Read-only | Never modifies state or real infrastructure — purely for testing expressions like `cidrsubnet()`, `merge()`, `lookup()`. |
| Loads `.tfvars` automatically | Pulls in variable defaults and any `.tfvars`/`.auto.tfvars` present in the directory. |
| Common error | Referencing `var.x` with no default and no supplied value throws an error — same precedence rules as plan/apply apply here. |

### `terraform output`

Reads output values from the current state file.

| Flag / Subcommand | Effect |
|---|---|
| `terraform output <name>` | Prints a single named output value. |
| `-json` | Returns all outputs as JSON — common for chaining into scripts or other tooling. |
| `-raw` | Prints a single string output without quotes — useful for shell scripting. |

### `terraform graph`

Generates a visual representation (DOT format) of the resource dependency graph — used with Graphviz to render an image.

---

## 3. State Management Commands

Highest-weight domain alongside HCP Terraform. The `terraform state` subcommand family is tested extensively.

### `terraform state list`

Lists all resources currently tracked in state.

> ⚠️ **EXAM NOTE:** Empty output usually means the backend is misconfigured/unreachable, not that no resources exist — check backend config first.

### `terraform state show <resource>`

Displays detailed attributes of a single resource as recorded in state.

### `terraform state mv <src> <dst>`

Moves a resource within state (e.g., renaming) or between state files using `-state-out`. Does NOT touch real infrastructure — purely a state-bookkeeping operation.

| Flag / Subcommand | Effect |
|---|---|
| `-state=<path>` | Specify the source state file (defaults to current). |
| `-state-out=<path>` | Specify a destination state file, enabling moves across state files/workspaces. |

### `terraform state rm <resource>`

Removes a resource from state without destroying the real infrastructure. Terraform "forgets" the resource entirely — use when handing management to another tool or config.

> ⚠️ **EXAM NOTE:** Frequently confused with `terraform destroy -target`, which DOES delete real infrastructure. `state rm` never touches real infra.

### `terraform state pull` / `push`

| Flag / Subcommand | Effect |
|---|---|
| `terraform state pull` | Downloads and outputs the current remote state in JSON format (read-only, prints to stdout). |
| `terraform state push <file>` | Uploads a local state file to overwrite the remote state. Dangerous — used rarely, mainly for manual recovery. |

### `terraform state replace-provider`

Updates the provider source address recorded against resources in state, e.g., after a provider is renamed or forked.

### `terraform import <address> <id>`

Brings an existing, externally-created resource under Terraform management by writing it into state. Does NOT generate configuration — you must write the matching resource block yourself (or use `generate-config-out` in newer versions).

> ⚠️ **EXAM NOTE:** Not all providers/resources support import. This is a TRUE statement frequently tested as a true/false trap.

### `terraform refresh` (and `plan -refresh-only`)

| Flag / Subcommand | Effect |
|---|---|
| `terraform refresh` | Legacy/standalone command — directly updates state to match real infra. Largely superseded. |
| `terraform apply -refresh-only` | Modern recommended approach — shows drift as a plan and requires approval before updating state. Safer and auditable. |

### `terraform taint` / `terraform apply -replace`

| Flag / Subcommand | Effect |
|---|---|
| `terraform taint <resource>` | Marks a resource for destroy+recreate on the NEXT plan/apply. Deprecated since 1.2 but still functional. |
| `terraform untaint <resource>` | Removes the taint marker. |
| `terraform apply -replace=<resource>` | Modern equivalent — destroys and recreates in a SINGLE apply operation, visible directly in plan output. |

---

## 4. Workspace Commands

CLI-level workspaces (distinct from HCP Terraform workspaces) allow multiple state files from one configuration.

| Flag / Subcommand | Effect |
|---|---|
| `terraform workspace list` | Lists all workspaces; current one marked with an asterisk. |
| `terraform workspace new <name>` | Creates and switches to a new workspace. |
| `terraform workspace select <name>` | Switches to an existing workspace. |
| `terraform workspace show` | Prints the name of the current workspace. |
| `terraform workspace delete <name>` | Deletes a workspace (must not be the current one, and ideally should be empty). |

> ⚠️ **EXAM NOTE:** `default` workspace can never be deleted and always exists, even after you create others. Local backend stores non-default workspace state in `terraform.tfstate.d/<name>/terraform.tfstate` — the default workspace stays at the root `terraform.tfstate`.

---

## 5. Module Commands

| Flag / Subcommand | Effect |
|---|---|
| `terraform get` | Downloads/updates modules referenced in configuration without running a full init. |
| `terraform get -update` | Forces re-downloading of modules even if already present. |
| `terraform init` | Also downloads modules as part of its normal flow — `get` is rarely needed standalone. |

> ⚠️ **EXAM NOTE:** Module source can be local path, Terraform Registry, GitHub, Git, HTTP URL, or S3/GCS bucket — know the syntax differences (e.g. local paths must start with `./` or `../`).

---

## 6. Provider & Dependency Lock Commands

### `terraform providers`

| Flag / Subcommand | Effect |
|---|---|
| `terraform providers` | Prints a tree of all providers required by the configuration, including those from modules. |
| `terraform providers schema -json` | Outputs full schema (resources, data sources, attributes) for all providers, as JSON. |
| `terraform providers mirror <dir>` | Downloads provider plugins into a local directory — used for air-gapped/offline installs. |
| `terraform providers lock` | Generates/updates entries in `.terraform.lock.hcl` for specified platforms without running a full init. |

> ⚠️ **EXAM NOTE:** `.terraform.lock.hcl` SHOULD be committed to version control (unlike the `.terraform/` directory, which should NOT be). This is a frequently inverted exam trap.

### `.terraform/` directory (not a command, but tested)

Created by `init`. Contains downloaded provider plugins (`providers/`) and module source code (`modules/`). Regenerated automatically — never commit this directory to version control.

---

## 7. Misc Workflow & Debug Commands

| Flag / Subcommand | Effect |
|---|---|
| `terraform show` | Displays the current state or a saved plan file in human-readable form. |
| `terraform show -json` | Same, but machine-readable JSON — common for piping into external tooling/policy checks. |
| `terraform force-unlock <lock-id>` | Manually removes a stuck state lock. Use with caution — only when certain no other operation is running. |
| `terraform version` | Prints Terraform CLI version and installed provider versions. |
| `terraform login` / `logout` | Authenticates the CLI against Terraform Cloud/HCP Terraform or another host. |
| CLI workspace vs HCP Terraform workspace | CLI workspaces = multiple state files, same backend config. HCP Terraform workspaces = full remote-execution environments with their own variables, VCS triggers, and run history. |

### `TF_LOG` environment variable

| Flag / Subcommand | Effect |
|---|---|
| `TF_LOG=<level>` | Enables detailed logging. Levels: TRACE, DEBUG, INFO, WARN, ERROR (TRACE = most verbose). |
| `TF_LOG_PATH=<path>` | Redirects log output to a file instead of the default destination. |
| Default output stream | Logs go to **stderr** by default, NOT stdout — a frequently tested distinction. |

---

## 8. Quick-Reference Tables

### `plan -detailed-exitcode`

| Exit Code | Meaning |
|---|---|
| `0` | Succeeded, no diff — infrastructure matches configuration. |
| `1` | Error occurred during plan. |
| `2` | Succeeded, with a non-empty diff — changes are pending apply. |

### Variable precedence (low → high)

| Order | Source |
|---|---|
| 1 | Environment variables — `TF_VAR_<name>` |
| 2 | `terraform.tfvars` — automatically loaded if present in the working directory |
| 3 | `*.auto.tfvars` — automatically loaded, processed in alphabetical order by filename |
| 4 | `-var-file=<file>` — explicitly passed on the CLI |
| 5 | `-var '<key>=<value>'` — highest precedence, overrides everything else |

### HCP Terraform run lifecycle order

| Step | Stage |
|---|---|
| 1 | **Plan** — generated on the HCP Terraform managed infrastructure (remote execution) or locally (local execution mode) |
| 2 | **Sentinel policy check** — evaluated after plan, before apply. Enforcement levels: advisory, soft-mandatory, hard-mandatory |
| 3 | **Cost estimation** — optional, shown alongside the plan (Plus/Enterprise tiers) |
| 4 | **Apply** — runs only if policies pass and the workspace's apply method (auto or manual) allows it |
