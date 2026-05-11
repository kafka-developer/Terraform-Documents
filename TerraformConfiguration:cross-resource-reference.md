In Terraform, **cross-resource reference** means one resource uses an attribute from another resource instead of hardcoding the value.

### Basic Syntax

```hcl
resource_type.resource_name.attribute
```

Example:

```hcl
confluent_kafka_cluster.basic.id
```

That means:

```text
resource type  = confluent_kafka_cluster
resource name  = basic
attribute      = id
```

### Example: Topic References Kafka Cluster

```hcl
resource "confluent_kafka_cluster" "basic" {
  display_name = "dev-kafka-cluster"
  availability = "SINGLE_ZONE"
  cloud        = "GCP"
  region       = "us-central1"
}

resource "confluent_kafka_topic" "orders" {
  kafka_cluster {
    id = confluent_kafka_cluster.basic.id
  }

  topic_name    = "orders"
  partitions_count = 6
}
```

Here:

```hcl
id = confluent_kafka_cluster.basic.id
```

means:

> Use the Kafka cluster ID created by the `confluent_kafka_cluster.basic` resource.

Terraform automatically understands:

```text
Topic depends on Kafka Cluster
```

So it creates the cluster first, then the topic.

### Another Example: API Key References Service Account and Cluster

```hcl
resource "confluent_service_account" "app_sa" {
  display_name = "orders-app-sa"
  description  = "Service account for orders application"
}

resource "confluent_api_key" "app_key" {
  display_name = "orders-app-api-key"

  owner {
    id          = confluent_service_account.app_sa.id
    api_version = confluent_service_account.app_sa.api_version
    kind        = confluent_service_account.app_sa.kind
  }

  managed_resource {
    id          = confluent_kafka_cluster.basic.id
    api_version = confluent_kafka_cluster.basic.api_version
    kind        = confluent_kafka_cluster.basic.kind
  }
}
```

### Key Point

Do **not** hardcode values like this:

```hcl
id = "lkc-abc123"
```

Use references instead:

```hcl
id = confluent_kafka_cluster.basic.id
```

### Why This Matters

Terraform uses these references to build the dependency graph:

```text
Service Account → API Key
Kafka Cluster → Topic
Kafka Cluster → API Key
Service Account → ACL
```

So the rule is simple:

> If one Terraform resource needs information from another resource, reference its attribute directly.
