# Terraform: Resource Blocks vs Data Blocks

## MUST KNOW ✅

### 1. Resource Blocks → CREATE / MANAGE Infrastructure

A `resource` block tells Terraform:

> “Create and manage this infrastructure object.”

Examples:

* Kafka topic
* VM
* VPC
* DNS record
* IAM user
* S3 bucket

### Syntax

```hcl
resource "type" "name" {
}
```

Example:

```hcl
resource "aws_s3_bucket" "logs_bucket" {
  bucket = "prod-logs-bucket"
}
```

Breakdown:

* `resource` → Terraform keyword
* `aws_s3_bucket` → resource type from provider
* `logs_bucket` → local Terraform name

---

### 2. Data Blocks → READ Existing Infrastructure

A `data` block tells Terraform:

> “Look up existing infrastructure and use its information.”

Terraform does NOT create it.

Examples:

* Existing VPC
* Existing Kafka cluster
* Existing subnet
* Existing AMI image
* Existing service account

### Syntax

```hcl
data "type" "name" {
}
```

Example:

```hcl
data "aws_vpc" "existing_vpc" {
  id = "vpc-12345"
}
```

This only reads the VPC details.

---

# Core Difference

| Feature         | Resource Block               | Data Block                   |
| --------------- | ---------------------------- | ---------------------------- |
| Purpose         | Create/manage infrastructure | Read existing infrastructure |
| Lifecycle       | Create/Update/Destroy        | Read-only                    |
| Stored in state | Yes                          | Only lookup metadata         |
| Changes infra?  | Yes                          | No                           |

---

# Real Terraform Workflow Example

## Resource Example

Create Kafka Topic:

```hcl
resource "confluent_kafka_topic" "orders" {
  topic_name = "orders"
}
```

Terraform will:

1. Plan creation
2. Create topic
3. Track it in state
4. Update/delete later if config changes

---

## Data Example

Read Existing Kafka Cluster:

```hcl
data "confluent_kafka_cluster" "prod" {
  display_name = "prod-cluster"
}
```

Terraform:

1. Queries Confluent Cloud/API
2. Fetches cluster info
3. Uses returned attributes
4. Does NOT create cluster

---

# Referencing Syntax

## Resource Reference

```hcl
resource.type.name.attribute
```

Example:

```hcl
aws_s3_bucket.logs_bucket.id
```

---

## Data Reference

```hcl
data.type.name.attribute
```

Example:

```hcl
data.aws_vpc.existing_vpc.id
```

---

# SHOULD KNOW ⚠️

## Data Sources Query During PLAN

Data blocks are evaluated during:

```bash
terraform plan
```

Terraform contacts provider APIs during planning to fetch live information.

Example:

* Query AWS for latest AMI
* Query Confluent for existing cluster
* Query GCP for subnet

---

# Resources Have Lifecycle

Resources go through lifecycle stages:

```text
Create → Update → Destroy
```

Terraform compares:

* Desired configuration
  vs
* Actual infrastructure

Then decides:

* create
* modify
* replace
* destroy

---

# Data Sources Are Read-Only

Terraform cannot:

❌ update data sources
❌ destroy data sources
❌ manage lifecycle of data sources

They are only:

* lookup objects
* references
* external information fetchers

---

# VERY IMPORTANT INTERVIEW POINT ⭐

## Typical Design Pattern

### Create New Infrastructure with Resource

```hcl
resource "aws_instance" "app" {
}
```

### Read Existing Shared Infrastructure with Data

```hcl
data "aws_vpc" "shared" {
}
```

This is extremely common in enterprises.

Because:

* Networking team already owns VPC
* App team only deploys inside it

This directly matches the architecture discussions we had earlier about:

* shared stable infrastructure
* app-specific deployments
* avoiding unnecessary centralization 👍

---

# Quick Memory Trick

## Resource = CREATE

```text
resource = Terraform owns it
```

## Data = READ

```text
data = Terraform looks it up
```

---

# Common Interview Question

## Q:

Why use a data block instead of resource block?

## A:

Because the infrastructure already exists and Terraform should only read/reference it instead of creating and managing it.

---

# Common Real Enterprise Example

```hcl
data "aws_vpc" "shared_vpc" {
  tags = {
    Name = "shared-prod-vpc"
  }
}

resource "aws_subnet" "app_subnet" {
  vpc_id = data.aws_vpc.shared_vpc.id
}
```

Meaning:

* VPC already exists
* App team creates subnet inside it

Very common enterprise pattern 🚀
