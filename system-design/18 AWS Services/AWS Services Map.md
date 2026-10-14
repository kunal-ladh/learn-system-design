---
title: AWS Services Map
aliases: [AWS Map, Which AWS Service, AWS Cheat Sheet]
tags: [system-design, aws]
type: note
status: evergreen
created: 2026-10-05
updated: 2026-10-14
summary: One-page lookup from each AWS service to the vault note that teaches the underlying concept and the note that teaches the AWS specifics, plus a reverse table from interview phrasing to the AWS answer.
related: ["[[AWS Foundations]]", "[[AWS Cost Shape]]", "[[Choosing a Message Broker]]", "[[Choosing a Database]]"]
---

# AWS Services Map

A reference page, not a lesson. Use it two ways. **Forward:** you meet an AWS name (in a design review, a bill, a job description) and want the one-line meaning, the generic concept it instantiates, and where the vault teaches the AWS specifics. **Reverse:** an interviewer states a requirement in plain words and you want the AWS service to name once you have said the concept. In an interview the AWS name is the *last* word of the answer, not the first: say the concept, say the service, then give one limit or one cost shape ([[AWS Cost Shape]]).

**Bold** services are in your stack. "None yet" in the last column means the vault has no AWS-specific note for that service; the generic note still teaches the concept. Scope reminders (global, regional, zonal) live in [[AWS Foundations]].

## Service to concept to note

| AWS service | What it is in one line | The generic concept note | The AWS note |
|---|---|---|---|
| **EKS** | Managed Kubernetes control plane; you bring the worker nodes (managed node groups, self-managed, or Fargate), or let EKS Auto Mode manage them. | [[Kubernetes Architecture]] | [[EKS vs ECS]] |
| **ECR** | Private container image registry with IAM-based access, image scanning and lifecycle rules. | [[Container Registries and ECR]] | [[Container Registries and ECR]] |
| ECS | AWS's own container orchestrator (tasks and services): simpler than Kubernetes, AWS-only. | [[Kubernetes Architecture]] | [[EKS vs ECS]] |
| Fargate | Serverless compute for containers: you size the task or pod, AWS supplies the machine; works under ECS and EKS. | [[Kubernetes Architecture]] | [[EKS vs ECS]] |
| **EC2** | Virtual machines, billed per second for Linux, Windows and a few other OSes with a 60-second minimum (as of 2026); the substrate under your EKS nodes. | [[Vertical vs Horizontal Scaling]] | [[EC2]] |
| Lambda | Functions as a service: event-triggered code billed per request and per GB-second of duration, metered in milliseconds (shape as of 2026). | [[Autoscaling]] | [[Lambda]] |
| **S3** | Object storage with a flat key namespace and a design durability of 11 nines. | [[How S3 Works]] | [[S3 Deep Dive]] |
| DynamoDB | Fully managed key-value and document database, partitioned by key, with on-demand or provisioned capacity. | [[NoSQL Taxonomy]] | [[DynamoDB Design]], [[DynamoDB in Production]] |
| RDS and Aurora | Managed relational databases (MySQL, PostgreSQL and others); Aurora is AWS's cloud-native engine with a storage layer spread across AZs. | [[Replication]], [[ACID Transactions]] | None yet |
| **ElastiCache** | Managed Valkey, Redis OSS or Memcached, as a serverless cache or a node-based cluster; on node-based clusters you still choose node types and shard layout. | [[Redis Cluster and Sentinel]] | [[ElastiCache for Redis]] |
| **SQS** | Managed queue: at-least-once delivery, visibility timeout, DLQs, no replay. | [[Message Queues vs Event Logs]] | [[SQS and SNS]] |
| **SNS** | Managed pub-sub topic that pushes each message to many subscribers, usually one SQS queue per consumer. | [[Why Async Messaging Exists]] | [[SQS and SNS]] |
| **MSK** | Managed Apache Kafka: AWS runs the brokers, you still own topics, partitions and consumers. | [[Kafka Architecture]] | [[Kafka Operations]] |
| Kinesis | AWS-native managed event stream: a shard-based ordered log with replay inside a retention window. | [[Message Queues vs Event Logs]] | None yet |
| EventBridge | Serverless event bus: rules match events by content and route them from AWS services, SaaS and your apps to targets. | [[Communication Patterns Between Services]] | None yet |
| Step Functions | Managed state-machine service that orchestrates multi-step workflows with retries, timeouts and error handling built in. | [[Orchestration vs Choreography]], [[Saga Pattern]] | None yet |
| **ALB** | Layer 7 HTTP and HTTPS load balancer with host and path routing, target groups and WAF integration. | [[Load Balancers]] | [[ALB and NLB]] |
| NLB | Layer 4 TCP, UDP and TLS (plus QUIC passthrough since November 2025) load balancer with a static IP per AZ, optionally an Elastic IP per subnet, built for very high throughput and low latency. | [[Load Balancers]] | [[ALB and NLB]] |
| **CloudFront** | Global CDN: edge caches, regional caches, Origin Shield, signed URLs, WAF attachment. | [[CDN]] | [[CloudFront]] |
| **Route 53** | Authoritative DNS with health checks and routing policies (weighted, latency, failover); also domain registration. | [[How DNS Works]] | [[Route 53]] |
| **VPC** | Your private virtual network inside one Region: CIDR ranges, per-AZ subnets, route tables, security groups, gateways. | [[Kubernetes Networking]], [[Anatomy of a Web Request]] | [[VPC and Networking]] |
| **IAM** | Identity and permissions for every AWS API call: users, roles, policies; IRSA lets pods assume roles. | [[Designing an IAM Service]], [[RBAC vs ABAC]] | [[IAM]] |
| **Secrets Manager** | Stores, encrypts and rotates secrets such as database credentials; billed per secret per month plus API calls. | [[Kubernetes Configuration and Resources]] | [[Secrets Manager Parameter Store and KMS]] |
| **Parameter Store** | Hierarchical configuration store in Systems Manager, plain or KMS-encrypted `SecureString` values; standard tier free, advanced tier billed, no native rotation (as of 2026). | [[Kubernetes Configuration and Resources]] | [[Secrets Manager Parameter Store and KMS]] |
| **KMS** | Managed encryption keys and envelope encryption; every use is an IAM-checked, CloudTrail-logged API call. | [[API Security Essentials]] | [[Secrets Manager Parameter Store and KMS]] |
| **ACM** | Issues public TLS certificates and auto-renews the DNS-validated ones in use by integrated services such as ALB and CloudFront. | [[TLS and HTTPS]] | [[WAF and ACM]] |
| **WAF** | Layer 7 firewall with managed rule groups and rate-based rules, attached to CloudFront, ALB, API Gateway REST APIs and a few others such as AppSync, Cognito, App Runner and Amplify (list as of 2026). | [[Rate Limiting]], [[API Security Essentials]] | [[WAF and ACM]] |
| **CloudWatch** | Metrics, logs, alarms and dashboards for every AWS service and your own workloads. | [[Logs Metrics and Traces]], [[Alerting and On-Call Design]] | [[CloudWatch]] |
| X-Ray | AWS's distributed-tracing service: records the path of a request across services. | [[Distributed Tracing]] | None yet |
| CloudTrail | Audit log of AWS API calls — who did what, when, from where; management events stay in a free 90-day event history by default (as of 2026); longer needs a trail. | [[Logs Metrics and Traces]] | None yet |
| **Organizations** | Multi-account management: an OU tree, consolidated billing and SCP guardrails; your uat and prod split lives here. | [[Bulkheads and Isolation]] | [[AWS Foundations]] |

## If the interviewer says X, the AWS answer is Y

| The interviewer says | The AWS answer | Then say |
|---|---|---|
| "Serve static content to users worldwide" | CloudFront in front of S3 | Edge caches cut latency and origin load; versioned URLs avoid invalidations ([[CDN]]) |
| "Decouple producer and consumer with a queue" | SQS | Visibility timeout, DLQ, at-least-once, so consumers must be idempotent ([[SQS and SNS]]) |
| "One event, many independent consumers" | SNS fanning out to one SQS queue per consumer | Filter policies route in infrastructure; each consumer gets its own retries and DLQ |
| "Durable event log with replay and per-key ordering" | MSK (Kafka semantics) or Kinesis (AWS-native shards) | Replay and consumer groups versus zero-ops; name the cost shape ([[Choosing a Message Broker]]) |
| "Route events between services by content" | EventBridge | Rules on event patterns; more routing, less throughput than a log |
| "Store images, PDFs, backups" | S3 | Presigned URLs, lifecycle rules, versioning; not a database ([[How S3 Works]]) |
| "Key-value store at massive scale" | DynamoDB | The partition key is the design; hot partitions are the failure ([[DynamoDB Design]]) |
| "Relational data with transactions" | RDS or Aurora | Multi-AZ standby for failover, read replicas for read scale ([[Database Scaling Patterns]]) |
| "Cache hot reads" | ElastiCache | Cache-aside with TTLs; the cache must never be the source of truth ([[Caching Strategies]]) |
| "Balance HTTP traffic across services" | ALB | Layer 7 routing and health checks; NLB when you need layer 4 or static IPs ([[Load Balancers]]) |
| "Fail over DNS to another site" | Route 53 health checks with failover records | Pre-provision the records; do not edit DNS mid-incident ([[Route 53]], [[AWS Foundations]]) |
| "Put an API gateway in front" | Amazon API Gateway as the managed option; Kong in your stack | Authn, throttling and routing at the edge ([[API Gateway]]) |
| "Run code without managing servers" | Lambda | Per-request billing, scale to zero, cold starts, concurrency limits |
| "Run containers at scale" | EKS (or ECS for a simpler AWS-only model) | Managed control plane, you own the data plane ([[EKS vs ECS]]) |
| "Orchestrate a multi-step workflow or saga" | Step Functions | Durable state machine with retries instead of hand-rolled glue ([[Orchestration vs Choreography]]) |
| "Where do secrets live?" | Secrets Manager; Parameter Store for plain config | Rotation and per-secret cost versus simplicity; never in the image or the repo |
| "Encrypt data at rest" | KMS keys behind S3, EBS, Secrets Manager and others | Envelope encryption; key policy plus IAM decide who can decrypt |
| "HTTPS certificates" | ACM on ALB and CloudFront | Auto-renewal; the viewer-facing CloudFront certificate must live in `us-east-1`, the ALB's in its own Region ([[TLS and HTTPS]]) |
| "Block bots, injection, abusive clients at the edge" | WAF on CloudFront or ALB | Managed rule groups plus rate-based rules ([[Rate Limiting]]) |
| "Metrics, logs and alerts" | CloudWatch | Alarm on symptoms, mind log-ingestion cost ([[Logs Metrics and Traces]]) |
| "Trace a request across services" | X-Ray, or OpenTelemetry into a tracing backend | Propagate a correlation ID everywhere ([[Distributed Tracing]]) |
| "Who changed this, and when?" | CloudTrail | Immutable audit trail of API calls; central to regulated evidence |
| "Separate environments and limit blast radius" | Separate accounts under Organizations, with SCPs | Accounts are the hard wall for IAM, quotas and billing ([[AWS Foundations]]) |
| "Give a pod AWS permissions without keys" | IAM roles for service accounts (IRSA) | Projected token exchanged at STS for temporary credentials ([[IAM]]) |
| "Isolate tiers on a private network" | VPC with private subnets, security groups | One subnet per AZ per tier; mind NAT and cross-AZ cost ([[VPC and Networking]]) |
