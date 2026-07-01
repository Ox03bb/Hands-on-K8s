<img src="./assets/namespace/icons.drawio.svg" alt="configmap icon" width="60"/>

# Kubernetes Namespaces

## What is a Namespace?

A **Namespace** is a logical partition inside a Kubernetes cluster. you can think of it as a virtual cluster within the physical cluster.

It allows you to divide a single cluster into multiple isolated environments.

Think of a Namespace as a folder inside the cluster.

```
Cluster
├── Namespace: default
│      ├── Pods
│      ├── Services
│      └── Deployments
│
├── Namespace: development
│      ├── Pods
│      ├── Services
│      └── ConfigMaps
│
└── Namespace: production
       ├── Pods
       ├── Services
       └── Secrets
```

Namespaces provide **organizational isolation**, not virtualization. All namespaces still share the same Kubernetes cluster.


## Why Do We Need Namespaces?

Without namespaces:

```
Cluster

├── backend
├── frontend
├── database
├── api
├── monitoring
├── testing
├── development
└── production
```

Everything exists together, making resource management difficult.

With namespaces:

```
Cluster

├── dev
│     ├── backend
│     └── frontend
│
├── staging
│     ├── backend
│     └── frontend
│
└── production
      ├── backend
      └── frontend
```

This provides:

- Better organization
- Isolation between teams
- Easier resource management
- Separate environments
- Independent quotas


## Default Namespaces

Every Kubernetes cluster includes several built-in namespaces.

| Namespace | Purpose |
|-----------|---------|
| `default` | Default namespace for user workloads |
| `kube-system` | Kubernetes system components |
| `kube-public` | Publicly readable resources |
| `kube-node-lease` | Node heartbeat leases |

View them:

```bash
kubectl get namespaces
```

Example:

```
NAME              STATUS   AGE

default           Active   10d
kube-system       Active   10d
kube-public       Active   10d
kube-node-lease   Active   10d
```


## Namespace Architecture

```
Cluster

│
├── Namespace A
│      ├── Pods
│      ├── Services
│      └── Deployments
│
├── Namespace B
│      ├── Pods
│      ├── Services
│      └── Secrets
│
└── Namespace C
       ├── Pods
       ├── ConfigMaps
       └── Jobs
```

Each namespace has its own collection of resources.


## How Namespaces Work

Suppose two teams deploy an application.

Team A:

```
Deployment

Name: backend

Namespace: dev
```

Team B:

```
Deployment

Name: backend

Namespace: production
```

Even though both Deployments have the same name, there is no conflict because they belong to different namespaces.


## Resources That Are Namespaced

Most application resources belong to a namespace.

Examples:

- Pods
- Deployments
- ReplicaSets
- StatefulSets
- DaemonSets
- Services
- ConfigMaps
- Secrets
- Jobs
- CronJobs
- PersistentVolumeClaims
- NetworkPolicies
- ServiceAccounts
- Roles
- RoleBindings
- Ingress

Example:

```
development
├── backend Pod
├── api Service
└── ConfigMap
```


## Cluster-Wide Resources

Some resources are shared across the entire cluster and do **not** belong to a namespace.

Examples:

- Nodes
- Namespaces
- PersistentVolumes
- StorageClasses
- ClusterRoles
- ClusterRoleBindings
- CustomResourceDefinitions (CRDs)
- ValidatingWebhookConfigurations
- MutatingWebhookConfigurations

Example:

```
Cluster
├── Node 1
├── Node 2
├── Namespace dev
└── Namespace production
```


## Creating a Namespace

Using kubectl:

```bash
kubectl create namespace development
```

Verify:

```bash
kubectl get namespaces
```


## Namespace YAML

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: development
```

Apply it:

```bash
kubectl apply -f namespace.yaml
```


## YAML Explained

### apiVersion

```yaml
apiVersion: v1
```

Namespaces belong to the Kubernetes core API.


### kind

```yaml
kind: Namespace
```

Defines the resource type.


### metadata

```yaml
metadata:
  name: development
```

Defines the namespace name.


## Creating Resources in a Namespace

Specify the namespace in the resource manifest.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend
  namespace: development
```

The Deployment is created inside the `development` namespace.


## Creating Resources with kubectl

Specify the namespace:

```bash
kubectl apply -f deployment.yaml -n development
```

or

```bash
kubectl create deployment nginx \
--image=nginx \
-n development
```


## Listing Resources

Pods in the default namespace:

```bash
kubectl get pods
```

Pods in a specific namespace:

```bash
kubectl get pods -n development
```

Pods in all namespaces:

```bash
kubectl get pods --all-namespaces
```

or

```bash
kubectl get pods -A
```


## Switching the Default Namespace

Instead of writing `-n` every time:

Current namespace:

```bash
kubectl config view --minify
```

Change it:

```bash
kubectl config set-context --current --namespace=development
```

Now:

```bash
kubectl get pods
```

automatically uses the `development` namespace.


## Deleting a Namespace

```bash
kubectl delete namespace development
```

This deletes:

- Deployments
- Pods
- Services
- ConfigMaps
- Secrets

Everything inside the namespace.

Be careful.


## Resource Quotas

A namespace can limit resource usage.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota

metadata:
  name: quota

spec:

  hard:

    pods: "20"

    requests.cpu: "8"

    requests.memory: 16Gi

    limits.cpu: "16"

    limits.memory: 32Gi
```

Benefits:

- Prevent resource abuse
- Fair resource sharing
- Better cluster stability


## Limit Ranges

LimitRanges define default resource requests and limits.

Example:

```yaml
apiVersion: v1
kind: LimitRange

metadata:
  name: default-limits

spec:

- type: Container

  default:

    cpu: "500m"

    memory: "512Mi"

  defaultRequest:

    cpu: "250m"

    memory: "256Mi"
```

If a container doesn't specify resources, Kubernetes applies these defaults.


## DNS in Namespaces

Within the same namespace:

```
backend

↓

frontend
```

The frontend can simply use:

```
http://backend
```

Across namespaces:

```
frontend.dev

↓

backend.production
```

Use the fully qualified service name:

```
backend.production.svc.cluster.local
```

DNS format:

```
service-name.namespace.svc.cluster.local
```

Example:

```
redis.database.svc.cluster.local
```


## Network Policies

Namespaces work well with NetworkPolicies.

Example:

```
Production Namespace

↓

Accept traffic only from

↓

Frontend Namespace
```

This improves security by controlling communication between namespaces.


## Labels for Namespaces

Namespaces can have labels.

Example:

```yaml
metadata:

  name: development

  labels:

    environment: dev

    team: backend
```

Useful for automation and policy management.


## Useful kubectl Commands

Create

```bash
kubectl create namespace development
```

List

```bash
kubectl get namespaces
```

Describe

```bash
kubectl describe namespace development
```

Delete

```bash
kubectl delete namespace development
```

Resources in namespace

```bash
kubectl get all -n development
```

All resources in cluster

```bash
kubectl get all -A
```

Switch namespace

```bash
kubectl config set-context --current --namespace=development
```


## Complete Example

Create the namespace:

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: development
```

Deployment inside the namespace:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:

  name: backend

  namespace: development

spec:

  replicas: 2

  selector:

    matchLabels:

      app: backend

  template:

    metadata:

      labels:

        app: backend

    spec:

      containers:

      - name: backend

        image: nginx:1.28
```

Service inside the same namespace:

```yaml
apiVersion: v1
kind: Service

metadata:

  name: backend

  namespace: development

spec:

  selector:

    app: backend

  ports:

  - port: 80

    targetPort: 80
```


## Namespace vs Cluster

| Feature | Namespace | Cluster |
|----------|-----------|----------|
| Scope | Logical partition | Entire Kubernetes environment |
| Contains Pods | ✅ | Through namespaces |
| Contains Services | ✅ | Through namespaces |
| Contains Deployments | ✅ | Through namespaces |
| Contains Nodes | ❌ | ✅ |
| Isolation | Logical | Physical/administrative |
| Resource Quotas | ✅ | ❌ |


## Best Practices

### Use Namespaces for Environments

Separate:

- Development
- Testing
- Staging
- Production


### Use Resource Quotas

Prevent one namespace from consuming all cluster resources.


### Use LimitRanges

Ensure containers receive default CPU and memory requests and limits.


### Use Meaningful Names

Examples:

```
development

testing

staging

production

monitoring

logging
```


### Apply RBAC Per Namespace

Grant users access only to the namespaces they need.


### Use Labels

Labels make namespaces easier to organize and automate.


## Common Mistakes

### Using the Default Namespace for Everything

Keeping all workloads in `default` makes management difficult.


### Deleting a Namespace Accidentally

Deleting a namespace removes nearly everything inside it.

Always verify before running:

```bash
kubectl delete namespace <name>
```


### Forgetting the Namespace

Running:

```bash
kubectl get pods
```

may show no Pods simply because you're looking in the wrong namespace.


### Assuming Namespaces Provide Complete Isolation

Namespaces isolate Kubernetes resources, but they do not provide complete security isolation. For stronger isolation, combine namespaces with:

- RBAC
- NetworkPolicies
- ResourceQuotas
- Pod Security Admission


### Creating Too Many Namespaces

Namespaces should represent logical boundaries (teams, projects, environments), not individual applications.


## Summary

A Namespace is a logical partition of a Kubernetes cluster that organizes and isolates resources.

It provides:

- Resource organization
- Name isolation
- Team separation
- Environment separation
- Resource quotas
- RBAC boundaries
- Policy management

Architecture:

```
Cluster
├── development
│      ├── Pods
│      ├── Services
│      └── Deployments
├── staging
│      ├── Pods
│      └── Services
└── production
       ├── Pods
       ├── Services
       └── Deployments
```

Namespaces are one of the fundamental organizational tools in Kubernetes and are essential for managing multi-team, multi-environment, and production clusters effectively.