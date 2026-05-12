# 4c: Use Variables and Outputs

1. **Variables** make Terraform configurations reusable and environment-independent.

2. Instead of hardcoding values such as:

   * environment names
   * Kafka topic names
   * partition counts
   * retention settings
   * connector configurations
   * service account names

Terraform allows you to define variables and assign values later.

---

# Variables = Inputs to Terraform

Think of variables like input fields.

```text id="cgq04j"
variables.tf = defines placeholders
.tfvars      = provides actual values
```

Or:

```text id="g5w2iq"
Variable = empty input field
.tfvars  = filled-in form
```

---

# Step 1: Define Variables

`variables.tf`

```hcl id="1l3l7g"
variable "environment" {
  description = "Deployment environment"
  type        = string
}

variable "topic_name" {
  description = "Kafka topic name"
  type        = string
}

variable "partitions_count" {
  description = "Number of partitions"
  type        = number
}

variable "retention_ms" {
  description = "Topic retention period"
  type        = string
}
```

---

# Step 2: Use Variables Inside Resources

`main.tf`

```hcl id="l7hwhs"
resource "confluent_kafka_topic" "topic" {
  topic_name       = "${var.topic_name}-${var.environment}"
  partitions_count = var.partitions_count

  config = {
    "retention.ms" = var.retention_ms
  }
}
```

Terraform now expects values for:

```text id="o2m96s"
environment
topic_name
partitions_count
retention_ms
```

---

# Step 3: Assign Values Using `.tfvars`

`dev.tfvars`

```hcl id="kwq1np"
environment      = "dev"
topic_name       = "orders"
partitions_count = 6
retention_ms     = "604800000"
```

This file supplies actual values to the variables.

---

# Step 4: Run Terraform with `.tfvars`

```bash id="8j4jjm"
terraform plan  -var-file="dev.tfvars"

terraform apply -var-file="dev.tfvars"
```

---

# Step 5: What Terraform Builds

Terraform replaces:

```hcl id="1d9m5v"
var.topic_name
var.environment
```

with values from `.tfvars`.

So:

```hcl id="x4g76l"
"${var.topic_name}-${var.environment}"
```

becomes:

```text id="lfglqe"
orders-dev
```

Final result:

```text id="ch1iy2"
Topic Name : orders-dev
Partitions : 6
Retention  : 604800000 ms
```

---

# Why `.tfvars` Matters

You keep the same Terraform code across all environments:

```text id="vw1xj8"
DEV
QA
PROD
```

Only the values change.

Example:

```text id="l5jzb4"
dev.tfvars
qa.tfvars
prod.tfvars
```

This prevents duplicating Terraform code for every environment.

---

# Outputs

1. **Outputs** expose values created by Terraform.

2. Outputs can display:

   * topic names
   * cluster identifiers
   * service account identifiers
   * connector names
   * Schema Registry identifiers
   * generated resource references

---

# Outputs = Exports from Terraform

Think of outputs like exported values.

```text id="7swcxv"
Variables = values going INTO Terraform
Outputs   = values coming OUT of Terraform
```

---

# Example Output

`outputs.tf`

```hcl id="w3q1qw"
output "topic_name" {
  description = "Kafka topic created by Terraform"
  value       = confluent_kafka_topic.topic.topic_name
}
```

---

# View Outputs

After:

```bash id="d2t9br"
terraform apply
```

Run:

```bash id="0n0j9m"
terraform output
```

Example:

```text id="kz2clq"
topic_name = orders-dev
```

Or retrieve one output:

```bash id="r2mjlwm"
terraform output topic_name
```

---

# Outputs Between Modules

The real power of outputs is not displaying values.

The real power is:

```text id="mkhqrz"
module-to-module communication
```

---

# Child Module Example

`modules/kafka-topic/main.tf`

```hcl id="cjlwmw"
resource "confluent_kafka_topic" "topic" {
  topic_name       = var.topic_name
  partitions_count = var.partitions_count
}
```

---

# Child Module Output

`modules/kafka-topic/outputs.tf`

```hcl id="ry3zti"
output "topic_name" {
  value = confluent_kafka_topic.topic.topic_name
}
```

This exports:

```text id="k6r1r5"
orders-dev
```

from the child module.

---

# Parent Module Calls Child Module

`main.tf`

```hcl id="bjlwmx"
module "orders_topic" {
  source = "./modules/kafka-topic"

  topic_name       = "orders-dev"
  partitions_count = 6
}
```

---

# Parent Consumes Child Output

```hcl id="njlwmy"
output "created_topic" {
  value = module.orders_topic.topic_name
}
```

Notice:

```hcl id="0tmymj"
module.orders_topic.topic_name
```

means:

```text id="4c2x1h"
module.<module-name>.<output-name>
```

---

# Real Kafka Terraform Workflow

One module creates:

```text id="2xcrp0"
Kafka Topic
```

Another module consumes that output for:

```text id="j0xv6q"
ACLs
Connectors
MirrorMaker
Schema bindings
Service account permissions
Monitoring
```

Example flow:

```text id="n0g6h6"
Topic Module
   ↓ exports topic name

ACL Module
   ↓ consumes topic name

Connector Module
   ↓ consumes topic name
```

---

# Core Architectural Idea

```text id="zhb4i9"
Variables = module inputs
Outputs   = module exports
```

This becomes the foundation for:

```text id="x26tga"
Reusable modules
Decoupled infrastructure
Environment portability
Scalable Terraform architecture
Application onboarding workflows
```

Especially in large Confluent Kafka environments where many application teams share the same platform infrastructure 🚀
