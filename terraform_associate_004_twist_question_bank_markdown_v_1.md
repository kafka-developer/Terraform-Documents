# Terraform Associate 004 — Twist / Trick Question Bank

## Table of Contents

1. Objective 1: Understand Infrastructure as Code (IaC)
2. Objective 2: Understand Terraform Purpose
3. Objective 3: Terraform Basics
4. Objective 4: Use Terraform Outside Core Workflow
5. Objective 5: Interact with Terraform Modules
6. Objective 6: Navigate Terraform Workflow
7. Objective 7: Variables and Outputs
8. Objective 8: Terraform State
9. Objective 9: Terraform Cloud & Enterprise

---

# Objective 1: Understand Infrastructure as Code (IaC)

## Common Exam Traps

- Terraform vs Configuration Management
- Declarative vs Imperative
- Mutable vs Immutable
- State misunderstandings
- Drift confusion

---

### Question 1

Which statement BEST describes Terraform?

A. Terraform continuously monitors infrastructure and repairs drift.

B. Terraform uses imperative instructions.

C. Terraform uses declarative configuration to describe desired state.

D. Terraform replaces operating system configuration tools.

✅ Answer: C

Why wrong:

A ❌ Not continuous

B ❌ Declarative not imperative

D ❌ Doesn't replace configuration tools

---

### Question 2

An engineer manually modifies an EC2 instance after Terraform deployment.

What occurred?

A. Terraform updated state automatically

B. Terraform instantly detects drift

C. Infrastructure drift occurred

D. Terraform destroys instance

✅ Answer: C

Trap:

Terraform does not instantly know changes happened.

---

### Question 3

What is Terraform state primarily used for?

A. Store provider binaries

B. Store infrastructure configuration

C. Map configuration to real resources

D. Store Terraform modules

✅ Answer: C

---

### Question 4

A resource block is removed from configuration.

terraform apply is executed.

What happens?

A. Resource removed only from state

B. Resource ignored

C. Resource destroyed

D. Terraform errors

✅ Answer: C

Trap:

Configuration is the boss.

---

### Question 5

Two engineers execute Terraform simultaneously without state locking.

Likely result?

A. Terraform blocks automatically

B. State corruption risk

C. Terraform merges state

D. No impact

✅ Answer: B

---

# Objective 2: Understand Terraform Purpose

## Common Traps

- Orchestration vs configuration management
- Multi-cloud wording tricks
- Agent misconceptions

---

### Question 6

Terraform primarily focuses on:

A. Software deployment

B. Infrastructure orchestration

C. Operating system patching

D. Runtime monitoring

✅ Answer: B

---

### Question 7

Which is TRUE regarding Terraform providers?

A. Terraform supports one cloud only

B. Providers communicate with APIs

C. Providers are optional

D. Providers create state files

✅ Answer: B

---

### Question 8

Terraform can provision:

A. AWS only

B. Cloud resources only

C. Infrastructure through provider APIs

D. Virtual machines only

✅ Answer: C

Trap:

Provider ecosystem extends beyond cloud.

---

# Objective 3: Terraform Basics

## Common Traps

- init vs plan vs apply
- validate vs fmt
- backend reinitialization

---

### Question 9

Which command downloads providers and modules?

A. terraform plan

B. terraform apply

C. terraform init

D. terraform validate

✅ Answer: C

---

### Question 10

What occurs after backend configuration changes?

A. terraform refresh

B. terraform apply

C. terraform init

D. terraform destroy

✅ Answer: C

Trap:

Backend changes require reinitialization.

---

### Question 11

Which command checks syntax only?

A. terraform apply

B. terraform validate

C. terraform plan

D. terraform output

✅ Answer: B

---

### Question 12

Which command changes infrastructure?

A. terraform plan

B. terraform fmt

C. terraform validate

D. terraform apply

✅ Answer: D

---

# Objective 4: Use Terraform Outside Core Workflow

### Question 13

terraform fmt primarily:

A. changes state

B. changes infrastructure

C. formats code

D. validates providers

✅ Answer: C

---

### Question 14

terraform taint marks a resource:

A. for deletion only

B. for recreation

C. as read-only

D. as drifted

✅ Answer: B

Trap:

Many confuse taint with destroy.

---

### Question 15

terraform import:

A. creates infrastructure

B. updates provider

C. adds existing resource into state

D. modifies configuration

✅ Answer: C

Trap:

Import changes state, not configuration.

---

# Objective 5: Interact with Terraform Modules

### Question 16

Child modules should:

A. hardcode providers

B. avoid inputs

C. maximize reusability

D. define backends

✅ Answer: C

---

### Question 17

Modules communicate with parent through:

A. providers

B. variables and outputs

C. backend files

D. state locking

✅ Answer: B

---

### Question 18

Which source is valid?

A. Git

B. Registry

C. Local path

D. All

✅ Answer: D

---

# Objective 6: Navigate Terraform Workflow

### Question 19

terraform plan:

A. changes infrastructure

B. previews changes

C. updates state only

D. destroys resources

✅ Answer: B

---

### Question 20

Correct order:

A. apply → init → plan

B. plan → init → apply

C. init → plan → apply

D. validate → apply → init

✅ Answer: C

---

# Objective 7: Variables and Outputs

## Biggest Exam Trap = Variable Precedence

CLI > auto.tfvars > terraform.tfvars > env vars > defaults

---

### Question 21

Variable default:

us-east-1

Environment variable:

TF_VAR_region=us-west-2

CLI:

terraform apply -var region=eu-west-1

Final value?

A us-east-1

B us-west-2

C eu-west-1

D Error

✅ Answer: C

---

### Question 22

Which file automatically loads?

A variables.txt

B custom.vars

C terraform.tfvars

D inputs.tf

✅ Answer: C

---

### Question 23

Outputs primarily:

A. create variables

B. expose values

C. store state

D. define resources

✅ Answer: B

---

# Objective 8: Terraform State

### Question 24

Resource deleted manually outside Terraform.

terraform plan executed.

Result?

A ignored

B state updated

C resource recreated

D provider crash

✅ Answer: C

---

### Question 25

terraform state rm:

A destroys resource

B removes from cloud

C removes from state only

D deletes module

✅ Answer: C

Trap:

Very common exam trick.

---

### Question 26

State locking exists to:

A speed provider downloads

B prevent concurrent modification

C validate modules

D prevent drift

✅ Answer: B

---

# Objective 9: Terraform Cloud & Enterprise

### Question 27

Remote execution means:

A runs on your laptop

B runs inside Terraform Cloud

C runs in providers

D runs in modules

✅ Answer: B

---

### Question 28

Terraform workspace primarily:

A creates separate state instances

B creates provider versions

C creates child modules

D creates backends

✅ Answer: A

---

### Question 29

Sentinel policies are used for:

A monitoring

B governance

C provider installation

D state storage

✅ Answer: B

---

# Final Exam Trap Summary

Watch for:

❌ always

❌ automatically

❌ continuously

❌ instantly

Terraform Associate loves absolute wording.

Read every question twice.

