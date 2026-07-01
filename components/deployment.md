<img src="./assets/deployment/icons.drawio.svg" alt="deployment" width="60"/>

# Kubernetes Deployments

## What is a Deployment?

A **Deployment** is a Kubernetes resource responsible for managing the lifecycle of Pods.

It ensures that:

- The desired number of Pods are running.
- Failed Pods are recreated.
- Updates happen safely.
- Rollbacks are possible.
- Scaling is easy.

Think of a Deployment as the **manager** of your application.

Instead of creating Pods directly, you create a Deployment and let Kubernetes manage everything.

```
     You
      │
      │ create
      ▼
Deployment
      │
      ▼
 ReplicaSet
      │
      ▼
     Pods
```


## Why Do We Need Deployments?

Imagine you create a Pod directly.

```
Pod
```

Problems:

- If the Pod dies → it stays dead.
- No automatic updates.
- No rollback.
- No scaling.
- No version history.

Deployments solve all of these.


## Deployment Architecture

A Deployment does **not** create Pods directly.

It creates a ReplicaSet.

The ReplicaSet creates Pods.

```
Deployment
      │
      ▼
 ReplicaSet
      │
      ▼
     Pod
     Pod
     Pod
```

Hierarchy:

```
Deployment
    │
    └── ReplicaSet
              │
              ├── Pod
              ├── Pod
              └── Pod
```


## Deployment Responsibilities

A Deployment is responsible for:

- Creating ReplicaSets
- Updating ReplicaSets
- Deleting old ReplicaSets
- Scaling Pods
- Rolling updates
- Rollbacks
- Version history

A Deployment **never manages Pods directly.**


## Deployment Lifecycle

```
kubectl apply

      │

      ▼

Deployment created

      │

      ▼

ReplicaSet created

      │

      ▼

Pods created

      │

      ▼

Scheduler assigns Pods

      │

      ▼

Containers start

      │

      ▼

Application becomes Ready
```


## Basic Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:

  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:

    metadata:
      labels:
        app: nginx

    spec:

      containers:

      - name: nginx

        image: nginx:latest

        ports:
        - containerPort: 80
```


## YAML Explained

### apiVersion

```yaml
apiVersion: apps/v1
```

The API group responsible for Deployments.


### kind

```yaml
kind: Deployment
```

Tells Kubernetes which resource to create.


### metadata

```yaml
metadata:
  name: nginx-deployment
```

Resource information.

Example:

```yaml
metadata:
  name: backend
  namespace: production
```


### spec

Contains the desired state.


## replicas

```yaml
replicas: 3
```

Means:

```
Desired Pods = 3
```

If one Pod crashes:

```
Current = 2

Desired = 3

ReplicaSet creates another Pod.
```


## selector

```yaml
selector:
  matchLabels:
    app: nginx
```

The Deployment looks for Pods matching:

```
app=nginx
```

This selector **must** match the labels defined in the Pod template.


## template

This is the Pod template.

Every Pod created by this Deployment comes from here.

```
Deployment

     │

Creates Pods from

     │

template
```


## Pod Labels

```yaml
metadata:
  labels:
    app: nginx
```

These labels identify the Pods.

Example:

```
app=nginx

tier=backend

version=v1
```


## Containers

```yaml
containers:

- name: nginx

  image: nginx:latest
```

Defines what container runs.


## Creating a Deployment

```bash
kubectl apply -f deployment.yaml
```

Result:

```
Deployment

↓

ReplicaSet

↓

Pods
```


## Viewing Deployments

```bash
kubectl get deployments
```

Example:

```
NAME          READY   UP-TO-DATE   AVAILABLE

nginx         3/3     3            3
```


## Describe Deployment

```bash
kubectl describe deployment nginx
```

Shows:

- Events
- ReplicaSets
- Conditions
- Images
- Strategy
- Labels


## Viewing ReplicaSets

```bash
kubectl get rs
```

Example:

```
NAME

nginx-6489c8f5d
```


## Viewing Pods

```bash
kubectl get pods
```

Example:

```
nginx-6489c8f5d-abc12

nginx-6489c8f5d-def45

nginx-6489c8f5d-ghi89
```

Notice the Pod names contain the ReplicaSet hash.


## Scaling a Deployment

Scale manually:

```bash
kubectl scale deployment nginx --replicas=5
```

Result:

```
Deployment

↓

ReplicaSet

↓

5 Pods
```

Scale down:

```bash
kubectl scale deployment nginx --replicas=2
```


## Updating a Deployment

Example:

Old image:

```yaml
image: nginx:1.25
```

New image:

```yaml
image: nginx:1.26
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Deployment notices the change and starts a rolling update.


## Rolling Updates

Without Deployments:

```
Delete old Pods

↓

Create new Pods

↓

Downtime
```

With Deployments:

```
Old Pod

↓

New Pod created

↓

Ready

↓

Old Pod removed
```

No downtime.


## Rolling Update Process

Suppose:

```
Replicas = 4
```

Initial:

```
v1
v1
v1
v1
```

Step 1

```
v1
v1
v1
v2
```

Step 2

```
v1
v1
v2
v2
```

Step 3

```
v1
v2
v2
v2
```

Final

```
v2
v2
v2
v2
```

Users always have running Pods.


## Update Strategy

Default:

```yaml
strategy:
  type: RollingUpdate
```

Options:

- RollingUpdate
- Recreate


## RollingUpdate Options

```yaml
strategy:

  rollingUpdate:

    maxUnavailable: 1

    maxSurge: 1
```


### maxUnavailable

Maximum Pods that may be unavailable.

Example:

```
Replicas = 5

maxUnavailable = 1
```

Minimum running Pods:

```
4
```


### maxSurge

Maximum extra Pods created temporarily.

Example:

```
Replicas = 5

maxSurge = 2
```

Deployment may temporarily create:

```
7 Pods
```


## Recreate Strategy

```yaml
strategy:
  type: Recreate
```

Process:

```
Delete ALL Pods

↓

Create NEW Pods
```

Causes downtime.

Useful for applications that cannot run multiple versions simultaneously.


## Rollback

Deployment stores revision history.

See history:

```bash
kubectl rollout history deployment nginx
```

Undo:

```bash
kubectl rollout undo deployment nginx
```

Rollback to revision:

```bash
kubectl rollout undo deployment nginx --to-revision=2
```


## Check Rollout Status

```bash
kubectl rollout status deployment nginx
```

Example:

```
deployment "nginx" successfully rolled out
```


## Restart Deployment

```bash
kubectl rollout restart deployment nginx
```

Useful when ConfigMaps or Secrets change.


## Pause Deployment

```bash
kubectl rollout pause deployment nginx
```

Resume:

```bash
kubectl rollout resume deployment nginx
```


## Deployment Conditions

```
Available

Progressing

ReplicaFailure
```

Check:

```bash
kubectl describe deployment nginx
```


## Labels

Example:

```yaml
labels:

  app: nginx

  env: production

  version: v1
```


## Selectors

```yaml
selector:

  matchLabels:

    app: nginx
```

Never change the selector after creation.

It is immutable in most cases.


## Complete Example

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend

spec:

  replicas: 3

  strategy:
    type: RollingUpdate

    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1

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

        image: my-app:v1

        ports:
        - containerPort: 8080

        resources:

          requests:
            cpu: "250m"
            memory: "256Mi"

          limits:
            cpu: "500m"
            memory: "512Mi"

        readinessProbe:

          httpGet:
            path: /
            port: 8080

          initialDelaySeconds: 5

        livenessProbe:

          httpGet:
            path: /
            port: 8080

          initialDelaySeconds: 10
```


## Deployment vs ReplicaSet vs Pod

| Feature | Deployment | ReplicaSet | Pod |
|----------|------------|------------|-----|
| Creates Pods | ✅ | ✅ | ❌ |
| Scaling | ✅ | ✅ | ❌ |
| Rolling Update | ✅ | ❌ | ❌ |
| Rollback | ✅ | ❌ | ❌ |
| Version History | ✅ | ❌ | ❌ |
| Self-Healing | Through ReplicaSet | ✅ | ❌ |
| Recommended | ✅ | Rarely | Only for testing |


## Best Practices

### Never create Pods directly

Use Deployments instead.


### Always define resource requests and limits

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```


### Use readiness probes

Avoid sending traffic to Pods that are not ready.


### Use liveness probes

Automatically restart unhealthy containers.


### Avoid using `latest`

Instead of:

```yaml
image: nginx:latest
```

Use:

```yaml
image: nginx:1.28.0
```

This makes deployments reproducible.


### Keep Deployments declarative

Prefer:

```bash
kubectl apply -f deployment.yaml
```

over imperative commands for production environments.


### Use RollingUpdate

This is the safest deployment strategy for most applications.


### Keep replica count greater than one

```yaml
replicas: 2
```

or more to improve availability during updates and failures.


## Common Mistakes

❌ Selector does not match template labels

```yaml
selector:
  matchLabels:
    app: backend

template:
  metadata:
    labels:
      app: frontend
```

No Pods will be managed by the Deployment.


❌ Using `latest` image tag

Harder to reproduce deployments and roll back.


❌ Forgetting resource limits

Can lead to noisy-neighbor problems and unstable scheduling.


❌ Creating Pods directly

Pods created manually are not managed or recreated if they fail.


## Summary

A Deployment is the recommended way to run stateless applications in Kubernetes.

It provides:

- Declarative application management
- Automatic Pod replacement
- Scaling
- Rolling updates
- Rollbacks
- Version history
- High availability
- Integration with ReplicaSets

A Deployment manages ReplicaSets, and ReplicaSets manage Pods:

```
Deployment
      │
      ▼
 ReplicaSet
      │
      ▼
     Pods
```

For almost all production workloads, create a **Deployment** instead of creating Pods directly.