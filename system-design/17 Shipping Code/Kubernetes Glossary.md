---
title: Kubernetes Glossary
aliases: [K8s Terms, Kubernetes Terms, K8s Vocabulary]
tags: [system-design, shipping]
type: note
status: evergreen
created: 2026-10-05
updated: 2026-10-05
summary: Alphabetical reference of the Kubernetes and EKS vocabulary this module uses, each term in one honest line with a pointer to the note that teaches it in depth.
related: ["[[Kubernetes Architecture]]", "[[Kubernetes Workloads]]", "[[Kubernetes Networking]]", "[[Kubernetes Configuration and Resources]]", "[[The Deployment Pipeline]]", "[[EKS vs ECS]]"]
---

# Kubernetes Glossary

A lookup table, not a lesson. When a word in another note is unfamiliar, find it here, read the one-line definition, and follow the link in the last column if you need the *why* — the definitions are deliberately short and the linked note carries the depth. Case is ignored in the ordering, and the three probes, the Service types and the RBAC pieces each get their own row. If you are reading the module in order, start at [[The Deployment Pipeline]] and [[Docker and Container Images]], then [[Kubernetes Architecture]] (the one idea: reconciliation loops over desired state), then [[Kubernetes Workloads]], [[Kubernetes Networking]] and [[Kubernetes Configuration and Resources]]; this glossary is the companion you keep open alongside.

| Term | One-line definition | Note that teaches it |
|---|---|---|
| Access entry | EKS object that maps an IAM principal to a Kubernetes user and groups (a self-managed Linux node's role uses type `EC2_LINUX`, mapped to `system:node:`; EKS creates managed node groups' entries itself); the successor to the `aws-auth` ConfigMap. | [[Kubernetes Architecture]] |
| Admission controller | API-server plugin that runs after authentication and authorization and can mutate or reject an object before it is stored; webhooks extend it. | [[Kubernetes Architecture]] |
| Affinity and anti-affinity, pod | Rules that place a pod near, or keep it away from, other pods matching a label selector within a topology such as node or zone. | [[Kubernetes Architecture]] |
| Affinity, node | A pod rule that attracts it to nodes whose labels match, as a hard requirement or a soft preference. | [[Kubernetes Architecture]] |
| Allocatable | A node's capacity minus what the OS and kubelet reserve; the budget the scheduler spends requests against. | [[Kubernetes Architecture]] |
| Annotation | Non-identifying key-value metadata that tools and controllers read; unlike a label it is never used for selection, and it is most often met as controller hints on Ingress and Service objects. | [[Kubernetes Networking]] |
| API Priority and Fairness | API-server overload protection that queues request classes separately so one runaway client cannot starve the rest (stable since 1.29, 2023). | [[Kubernetes Architecture]] |
| AWS Load Balancer Controller | The AWS controller that satisfies an Ingress with an ALB and an opted-in LoadBalancer Service with an NLB. | [[Kubernetes Networking]] |
| BestEffort | QoS class for a pod with no requests or limits at all; first in line for eviction under node resource pressure. | [[Kubernetes Configuration and Resources]] |
| Binding | The object the scheduler writes to assign a pod to a node; it sets `spec.nodeName` and starts nothing. | [[Kubernetes Architecture]] |
| Burstable | QoS class for a pod that sets some requests or limits but does not meet the Guaranteed rule. | [[Kubernetes Configuration and Resources]] |
| chart | A Helm package: templated Kubernetes manifests plus default values. | [[The Deployment Pipeline]] |
| cloud-controller-manager | Control-plane component that hosts the loops calling the cloud API: nodes, routes and cloud load balancers. | [[Kubernetes Architecture]] |
| Cluster | One control plane (one API, one etcd) plus the nodes registered to it. | [[Kubernetes Architecture]] |
| Cluster Autoscaler | Adds nodes when pods cannot be scheduled and removes nodes that are underused. | [[Autoscaling]] |
| ClusterIP | Default Service type: a stable virtual IP reachable only from inside the cluster. | [[Kubernetes Networking]] |
| ClusterRole | Like a Role but cluster-wide, or for resource kinds that are not namespaced. | [[Kubernetes Architecture]] |
| CNI | Container Network Interface: the plugin standard that gives a pod its network interface and IP; on EKS usually the VPC CNI. | [[Kubernetes Networking]] |
| ConfigMap | Object holding non-secret configuration, consumed by pods as environment variables or mounted files. | [[Kubernetes Configuration and Resources]] |
| Container | An isolated process (kernel namespaces and cgroups) started from an image, with its own view of the filesystem. | [[Docker and Container Images]] |
| Container runtime | Software that pulls images and runs containers on a node; the kubelet drives it through the CRI. | [[Kubernetes Architecture]] |
| containerd | A widely used container runtime and the one the kubelet's default CRI endpoint (`unix:///run/containerd/containerd.sock`) points at; it launches containers through an OCI runtime such as runc. | [[Kubernetes Architecture]] |
| Context | A named kubeconfig entry combining a cluster, a user and a default namespace; the current context decides where kubectl talks. | [[Kubernetes Architecture]] |
| Control plane | The components that store and reconcile cluster state: API server, etcd, scheduler and controller managers. | [[Kubernetes Architecture]] |
| Controller | A loop that watches objects of one kind and acts to make the observed state match their spec. | [[Kubernetes Architecture]] |
| Cordon and drain | Cordon marks a node unschedulable; drain also evicts its pods, honouring PodDisruptionBudgets. | [[Graceful Shutdown and Deployment Safety]] |
| CrashLoopBackOff | A container keeps exiting and the kubelet restarts it with growing delays between attempts. | [[Kubernetes Workloads]] |
| CRD | CustomResourceDefinition: teaches the API server a new resource kind, stored in etcd like the built-in ones. | [[Kubernetes Architecture]] |
| CRI | Container Runtime Interface: the gRPC API between the kubelet and a container runtime. | [[Kubernetes Architecture]] |
| CronJob | Creates a Job on a cron schedule. | [[Kubernetes Workloads]] |
| CSI | Container Storage Interface: the plugin standard through which storage drivers provision, attach and mount volumes. | [[Kubernetes Configuration and Resources]] |
| DaemonSet | Runs one pod on every node, or on every node matching a selector. | [[Kubernetes Workloads]] |
| DefaultTolerationSeconds | Admission plugin that gives every pod a 300 s toleration for the `not-ready` and `unreachable` taints unless the pod sets its own. | [[Kubernetes Architecture]] |
| Deployment | Declares a replica count and a pod template for interchangeable pods and manages ReplicaSets to roll out changes. | [[Kubernetes Workloads]] |
| ECR | AWS's managed container registry. | [[Container Registries and ECR]] |
| EKS | AWS's managed Kubernetes: AWS runs the control plane, and you run or delegate the nodes. | [[EKS vs ECS]] |
| EKS Auto Mode | EKS option in which AWS also runs and scales the node layer, beyond the control plane. | [[Kubernetes Architecture]] |
| EKS Pod Identity | AWS's newer way to give a ServiceAccount an IAM role: no OIDC provider, a per-node agent serves the credentials. | [[Kubernetes Architecture]] |
| EndpointSlice | Object listing the pod IPs, ports and ready state behind a Service; kube-proxy reads it to build its rules. | [[Kubernetes Networking]] |
| etcd | The strongly consistent key-value store (Raft) that holds all cluster state; only the API server talks to it. | [[Kubernetes Architecture]] |
| Evicted | Status of a pod the kubelet failed under node pressure (memory, disk, process IDs), without consulting PodDisruptionBudgets; the verb also covers taint-based eviction after a node goes unreachable and API-initiated eviction such as a drain, which delete the pod. | [[Kubernetes Architecture]] |
| Extended support | EKS's 12 months after a version's 14 months of standard support, billed at a higher per-cluster hourly rate. | [[Kubernetes Architecture]] |
| ExternalName | Service type that returns a DNS CNAME to a name outside the cluster; no virtual IP and no proxying. | [[Kubernetes Networking]] |
| Fargate | AWS-managed serverless compute that runs each pod on its own AWS-run node, so there are no nodes for you to patch; AWS's own patching can evict pods, and pods must be recycled to pick up a new kubelet after an upgrade. | [[EKS vs ECS]] |
| Gateway API | The newer, role-oriented routing API (GatewayClass, Gateway, routes); the 2026 docs recommend it over Ingress, whose API is frozen. | [[Kubernetes Networking]] |
| Guaranteed | QoS class where every container's CPU and memory requests equal its limits; last in line for eviction. | [[Kubernetes Configuration and Resources]] |
| Headless Service | A Service with clusterIP set to None: DNS returns the pod IPs directly and there is no virtual IP. | [[Kubernetes Networking]] |
| Helm | Packaging and templating tool that renders charts into manifests and tracks installed releases. | [[The Deployment Pipeline]] |
| HPA | HorizontalPodAutoscaler: changes a workload's replica count from a metric such as CPU. | [[Autoscaling]] |
| Image | An immutable, layered filesystem bundle plus metadata from which containers are started. | [[Docker and Container Images]] |
| ImagePullBackOff | The kubelet cannot pull the image (wrong name or tag, missing permission, unreachable registry) and is retrying with growing delays. | [[Container Registries and ECR]] |
| imagePullPolicy | Per-container rule for when the kubelet pulls an image: Always, IfNotPresent or Never. | [[Container Registries and ECR]] |
| imagePullSecret | Reference to a Secret of registry credentials the kubelet uses to pull from a private registry. | [[Container Registries and ECR]] |
| Ingress | API object of host-and-path rules for routing HTTP traffic from outside to Services; inert without a controller. | [[Kubernetes Networking]] |
| Ingress controller | The component that implements Ingress rules, for example by programming a load balancer or proxy. | [[Kubernetes Networking]] |
| Init container | A container that runs to completion, in order, before the app containers start. | [[Kubernetes Workloads]] |
| IRSA | IAM Roles for Service Accounts: an annotated ServiceAccount whose projected token is exchanged for IAM role credentials; EKS Pod Identity is the newer alternative. | [[Kubernetes Architecture]] |
| Jib | Build tool that produces container images from a Java project without a Dockerfile or a Docker daemon. | [[Jib]] |
| Job | Runs pods until a set number complete successfully, then stops. | [[Kubernetes Workloads]] |
| Karpenter | Node autoscaler that skips node groups and launches right-sized instances directly in response to pending pods. | [[Kubernetes Architecture]] |
| KEDA | Kubernetes Event-Driven Autoscaling: drives the HPA from event sources such as queue depth or consumer lag, and can scale to zero. | [[Autoscaling]] |
| kube-apiserver | The cluster's front door: authenticates, authorizes, admits and persists requests, and is the only component that talks to etcd. | [[Kubernetes Architecture]] |
| kube-controller-manager | One binary running the built-in controllers: Deployment, ReplicaSet, Node, Job, EndpointSlice and more. | [[Kubernetes Architecture]] |
| kube-dns / CoreDNS | Cluster DNS: CoreDNS answers names such as service.namespace.svc.cluster.local, and its Service is still conventionally called kube-dns. | [[Kubernetes Networking]] |
| kube-proxy | Per-node agent that programs kernel rules (iptables by default as of 1.37 in 2026, nftables GA since 1.33 in 2025, or ipvs, deprecated since 1.35 in 2025) so Service IPs reach ready pods. | [[Kubernetes Networking]] |
| kube-scheduler | Chooses a node for each unscheduled pod: filter out nodes that cannot fit it, score the rest, bind. | [[Kubernetes Architecture]] |
| kubeconfig | File (by default ~/.kube/config) listing the clusters, users and contexts kubectl uses. | [[Kubernetes Architecture]] |
| kubectl | The command-line client for the Kubernetes API. | [[Kubernetes Architecture]] |
| kubelet | The agent on every node that makes pods real through the CRI, runs probes and reports status and heartbeats. | [[Kubernetes Architecture]] |
| Label | Key-value pair on an object used to group and select it; a ReplicaSet finds its pods and a Service finds its endpoints by label. | [[Kubernetes Workloads]] |
| Lease | Small API object used as a node heartbeat and as the lock for leader election. | [[Kubernetes Architecture]] |
| Level-triggered | Acting on current observed state rather than on a stream of events, so a restarted controller rediscovers what it missed. | [[Kubernetes Architecture]] |
| Limits | Per-container ceiling: CPU above it is throttled, memory above it gets the container OOMKilled. | [[Kubernetes Configuration and Resources]] |
| liveness probe | Check that asks whether a process is beyond saving; failure restarts the container. | [[Health Checks and Service Discovery]] |
| LoadBalancer | Service type that asks the cloud to provision an external load balancer in front of the Service. | [[Kubernetes Networking]] |
| Managed node group | An EKS-managed set of EC2 nodes whose provisioning and lifecycle AWS automates; an update or upgrade is one operation, but you initiate it. | [[EKS vs ECS]] |
| maxSurge | How many extra pods a rolling update may create above the desired count. | [[Graceful Shutdown and Deployment Safety]] |
| maxUnavailable | How many pods a rolling update may take out of service at once. | [[Graceful Shutdown and Deployment Safety]] |
| Namespace | A partition of names, RBAC and quotas inside a cluster; an administrative boundary, not by itself a network or security one. | [[Kubernetes Architecture]] |
| Node | A machine, usually a VM, running a kubelet and registered with the API to run pods. | [[Kubernetes Architecture]] |
| node-monitor-grace-period | How long the node controller waits without a heartbeat before marking a node's Ready condition Unknown (50 s by default since 1.32, 2024). | [[Kubernetes Architecture]] |
| NodePort | Service type that opens the same port on every node and forwards it to the Service. | [[Kubernetes Networking]] |
| NotReady | kubectl's status for a node whose Ready condition is False (kubelet unhealthy) or Unknown (kubelet silent); the node controller adds a `not-ready` or `unreachable` NoExecute taint respectively, and pods are evicted once their toleration (300 s by default) runs out. | [[Kubernetes Architecture]] |
| OOMKilled | A container was killed by the kernel for exceeding its memory limit (exit code 137). | [[Kubernetes Workloads]] |
| Operator | A controller for a CRD that encodes operational knowledge of an application. | [[Kubernetes Architecture]] |
| Owner reference | A pointer from a dependent object to its owner, so deleting a Deployment cascades to its ReplicaSets and pods. | [[Kubernetes Architecture]] |
| Pending | Pod accepted by the API but not running yet: either not scheduled (no node passes the filters) or scheduled with containers not started, image pull included. | [[Kubernetes Architecture]] |
| PersistentVolume | A piece of storage in the cluster, provisioned by an admin or dynamically, that outlives any one pod. | [[Kubernetes Configuration and Resources]] |
| PersistentVolumeClaim | A pod's request for storage of a given size and class, bound to a matching PersistentVolume. | [[Kubernetes Configuration and Resources]] |
| Pod | One or more containers placed together on a node, sharing a network namespace and optionally volumes; the unit Kubernetes schedules. | [[Kubernetes Workloads]] |
| Pod sandbox | The per-pod environment, chiefly the shared network namespace, that the runtime creates through the CRI before any container starts. | [[Kubernetes Architecture]] |
| pod-template-hash | Label a Deployment adds to each ReplicaSet and its pods, hashed from the pod template, so the ReplicaSets of different revisions never select each other's pods. | [[Kubernetes Workloads]] |
| PodDisruptionBudget | Limit on how many pods of a workload may be voluntarily disrupted at once, enforced when evicting. | [[Graceful Shutdown and Deployment Safety]] |
| preStop | Lifecycle hook that runs before SIGTERM is sent to the container. | [[Graceful Shutdown and Deployment Safety]] |
| progressDeadlineSeconds | How long a Deployment rollout may go without progress before it reports `ProgressDeadlineExceeded`; it does not roll back by itself. | [[Kubernetes Workloads]] |
| QoS class | Guaranteed, Burstable or BestEffort, derived from requests and limits; decides eviction order under node pressure. | [[Kubernetes Configuration and Resources]] |
| RBAC | Role-based authorization in the API server: Roles list verbs on resource kinds and bindings attach them to subjects. | [[Kubernetes Architecture]] |
| readiness probe | Check that asks whether a pod can take traffic now; failure marks its endpoint not-ready so Services stop routing to it, without restarting it. | [[Health Checks and Service Discovery]] |
| Reconciliation loop | The control loop at the heart of Kubernetes: compare desired state with observed state, act on the difference, repeat. | [[Kubernetes Architecture]] |
| Registry | Server that stores and serves container images by name and tag or digest. | [[Container Registries and ECR]] |
| release | An installed instance of a Helm chart, with a name and a revision history. | [[The Deployment Pipeline]] |
| ReplicaSet | Keeps a stated number of pods matching a label selector running; Deployments create and swap them. | [[Kubernetes Workloads]] |
| Requests | What a container reserves: the scheduler sums them against a node's allocatable resources to place pods. | [[Kubernetes Configuration and Resources]] |
| resourceVersion | The version every object carries; an update built on a stale one is rejected, so the writer re-reads and retries (optimistic concurrency). | [[Kubernetes Architecture]] |
| Role | A set of permitted verbs on resource kinds within one namespace. | [[Kubernetes Architecture]] |
| RoleBinding | Attaches a Role or ClusterRole to users, groups or ServiceAccounts within a namespace; a ClusterRoleBinding does it cluster-wide. | [[Kubernetes Architecture]] |
| Rollback | Reverting a Deployment to an earlier ReplicaSet revision; in effect a new rollout that reuses the old template. | [[Kubernetes Workloads]] |
| Rollout | The gradual replacement of old pods by new ones after a Deployment's pod template changes. | [[Kubernetes Workloads]] |
| Secret | Object for sensitive values; base64 only obscures them. Upstream Kubernetes stores them unencrypted in etcd unless encryption at rest is configured; EKS envelope-encrypts all API data by default on 1.28 and later (announced 2025), but anyone with API read access still reads the values. | [[Kubernetes Configuration and Resources]] |
| Selector | A query over labels, used by Services, ReplicaSets and Deployments to find their pods. | [[Kubernetes Workloads]] |
| Service | A stable virtual IP and DNS name in front of a changing set of pods chosen by a label selector. | [[Kubernetes Networking]] |
| ServiceAccount | An in-cluster identity for pods, with a projected token, used as a subject in RBAC. | [[Kubernetes Architecture]] |
| Sidecar container | A helper container in the same pod as the app; natively declared as an init container with restartPolicy Always (stable since 1.33, 2025). | [[Sidecar Pattern]] |
| Spec and status | The two halves of an API object: what you want, and what the responsible component observes. | [[Kubernetes Architecture]] |
| startup probe | Check that holds liveness and readiness off while a slow process boots; kills the container if boot overruns its budget. | [[Health Checks and Service Discovery]] |
| StatefulSet | Like a Deployment but each pod keeps a stable name and its own storage across rescheduling. | [[Kubernetes Workloads]] |
| Static pod | A pod run by the kubelet directly from a manifest file on the node, with no API server needed to start it. | [[Kubernetes Architecture]] |
| StorageClass | Names a kind of storage and its provisioner, so a claim can request storage dynamically. | [[Kubernetes Configuration and Resources]] |
| Taint | A mark on a node that repels pods unless they tolerate it; the NoExecute effect also evicts running pods. | [[Kubernetes Architecture]] |
| Terminating | Pod with a deletion timestamp that is shutting down; it lingers on an unreachable node until the node returns, its Node object is deleted, or it is force-deleted. | [[Graceful Shutdown and Deployment Safety]] |
| terminationGracePeriodSeconds | How long the kubelet waits between SIGTERM and SIGKILL, with preStop time counted inside it. | [[Graceful Shutdown and Deployment Safety]] |
| Toleration | A pod's permission to land on, or stay on, a node carrying a matching taint; it permits but never attracts. | [[Kubernetes Architecture]] |
| Topology spread | Constraint that spreads a workload's pods across nodes or zones within a maximum skew. | [[Kubernetes Architecture]] |
| Version skew | The allowed version gap between components: a kubelet must never be newer than the API server and may lag it by up to three minor versions. | [[Kubernetes Architecture]] |
| Volume | A directory available to a pod's containers; ephemeral ones die with the pod, persistent ones are backed by a PersistentVolume. | [[Kubernetes Configuration and Resources]] |
| VPA | VerticalPodAutoscaler: adjusts a workload's CPU and memory requests instead of its replica count. | [[Autoscaling]] |
| VPC CNI | The AWS CNI plugin for EKS that gives each pod a real VPC IP address. | [[Kubernetes Networking]] |
| Watch | A long-lived streaming request by which components learn of object changes without polling. | [[Kubernetes Architecture]] |

## Pairs that get confused

- **Requests versus limits.** A request is what the scheduler *reserves* for placement; a limit is the ceiling the kernel *enforces* at runtime. A node can be full by requests while nearly idle in reality.
- **Taint versus affinity.** A taint repels from the node side and a toleration only permits; affinity attracts from the pod side. Dedicating a node pool to a workload needs both a taint and an affinity.
- **Service versus Ingress.** A Service gives a stable address to a set of pods inside the cluster; an Ingress or Gateway routes HTTP from outside to Services, and does nothing until a controller implements it.
- **Liveness versus readiness.** Liveness failure restarts the container; readiness failure only removes the pod from traffic. Mixing them up is the root of most probe bugs ([[Health Checks and Service Discovery]]).
- **Role versus ServiceAccount.** A ServiceAccount is *who* a pod is; a Role is *what* may be done; a RoleBinding connects the two.
