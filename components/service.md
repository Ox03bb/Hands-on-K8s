
<img src="./assets/service/service.drawio.svg" alt="LoadBalancer Service" width="60"/>

# Kubernetes Service

A **Service** is a Kubernetes object that provides a **stable network endpoint** for a group of Pods. Since Pods are ephemeral—they can be created, terminated, or rescheduled at any time—their IP addresses are not reliable for communication. A Service solves this problem by assigning a permanent virtual IP address and DNS name that clients can use regardless of the underlying Pods.

A Service also performs **load balancing** by distributing incoming requests across all healthy Pods that match its selector.


## Why Do We Need a Service?

Consider the following Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
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
```

After creating the Deployment:

```bash
kubectl get pods -o wide
```

Example output:

```text
NAME                     IP
nginx-6c8fd              10.244.0.15
nginx-7d92a              10.244.0.16
nginx-9ab13              10.244.0.17
```

Problems:

- Pod IPs change when Pods are recreated.
- Clients must know every Pod IP.
- There is no automatic load balancing.
- Applications cannot reliably communicate with the Pods.

A Service solves these problems by exposing the Pods through a single stable endpoint.


# How a Service Works

```
                Client
                   │
                   ▼
        +-------------------+
        |    Kubernetes     |
        |      Service      |
        +-------------------+
          │      │      │
          ▼      ▼      ▼
       Pod A   Pod B   Pod C
```

The Service:

- Has one stable virtual IP (Cluster IP).
- Has one DNS name.
- Automatically discovers matching Pods.
- Load balances requests across the Pods.


# Service Components

A typical Service consists of the following fields.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```


**apiVersion**

Specifies the Kubernetes API version.

```yaml
apiVersion: v1
```



**kind**

Defines the object type.

```yaml
kind: Service
```



## metadata

Contains object metadata.

```yaml
metadata:
  name: nginx-service
```

The Service will be accessible through:

```
nginx-service
```

or

```
nginx-service.default.svc.cluster.local
```



**spec**

Contains the Service configuration.



**selector**

The selector determines which Pods belong to the Service.

```yaml
selector:
  app: nginx
```

Any Pod with:

```yaml
labels:
  app: nginx
```

will automatically become part of the Service.



**ports**

Defines how traffic is forwarded.

```yaml
ports:
  - protocol: TCP
    port: 80 # the port exposed by the Service
    targetPort: 8080 # the port on the container that receives the traffic
```

**port**

The port exposed by the Service.

```yaml
port: 80
```

Clients connect to:

```
http://service-name:80
```



**targetPort**

The container port that receives the traffic.

```yaml
targetPort: 8080
```

Traffic flow:

```
Client
   │
Port 80 (Service)
   │
Target Port 8080 (Container)
```



**protocol**

Network protocol.

```yaml
protocol: TCP
```

Supported protocols:

- TCP
- UDP
- SCTP



# Service Discovery

Every Service automatically receives a DNS entry.

Example:

```
http://nginx-service
```

instead of

```
http://10.96.0.15
```

DNS is automatically managed by CoreDNS.



# Service Types

Kubernetes supports four Service types.

| Type | Internal | External | Use Case |
|-------|----------|----------|----------|
| ClusterIP | ✅ | ❌ | Internal communication |
| NodePort | ✅ | ✅ | Development and testing |
| LoadBalancer | ✅ | ✅ | Production |
| ExternalName | ❌ | External service | DNS alias |

---

# 1. ClusterIP

The default Service type.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

Architecture:

```
Pod
 │
 ▼
ClusterIP
 │
 ├── Pod A
 ├── Pod B
 └── Pod C
```

Accessible only inside the cluster.

Example:

```
curl http://backend
```

Use cases:

- Database
- Backend API
- Internal microservices

---

# 2. NodePort

Exposes the Service on every cluster node.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: NodePort
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

Traffic flow:

```
Internet
     │
NodeIP:30080
     │
     ▼
Service
     │
 ├── Pod A
 ├── Pod B
 └── Pod C
```

Access:

```
http://192.168.1.100:30080
```

Characteristics:

- Opens the same port on every node.
- Kubernetes still load balances across Pods.
- Suitable for development.
- Rarely used directly in production.

---

# 3. LoadBalancer

Creates an external cloud load balancer.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

**Architecture**:

<div style="text-align:center;">
  <img src="./assets/service/service01.drawio.svg" alt="LoadBalancer Service" width="400"/>
</div>

Characteristics:

- Provides one public IP.
- High availability.
- Automatically distributes traffic across nodes.
- The Service continues to load balance across Pods.

---

# 4. ExternalName

Does not proxy traffic.

Instead, it creates a DNS alias.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: google
spec:
  type: ExternalName
  externalName: google.com
```

Now Pods can access:

```
http://google
```

which resolves to

```
google.com
```

Useful for:

- External databases
- External APIs
- Legacy systems

---

# Load Balancing

Every Service except `ExternalName` performs load balancing.

Example:

```
    Client
       │
       ▼
    Service
       │
 ┌─────┼───────┐
 ▼     ▼       ▼
Pod1   Pod2   Pod3
```

The Service automatically chooses one healthy Pod for each request.

---

# Labels and Selectors

Pods:

```yaml
labels:
  app: nginx
```

Service:

```yaml
selector:
  app: nginx
```

Matching process:

```
Service Selector
      │
      ▼
app=nginx
      │
 ┌────┼────┐
 ▼    ▼    ▼
Pod1 Pod2 Pod3
```

If a Pod loses the matching label, it is automatically removed from the Service.

---

# Port Mapping

```
Client
   │
Port 80
   │
Service
   │
TargetPort 8080
   │
Container
```

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Meaning:

```
Client → Service:80 → Container:8080
```

---

# Service vs Deployment

| Deployment | Service |
|------------|----------|
| Creates Pods | Exposes Pods |
| Manages replicas | Provides networking |
| Handles updates | Provides stable endpoint |
| Controls lifecycle | Performs load balancing |

---

# Summary

- A Service provides a stable network identity for Pods.
- Pods are discovered using label selectors.
- Services automatically load balance traffic across healthy Pods.
- Pods can be recreated without affecting clients.
- Kubernetes automatically creates DNS entries for every Service.
- `ClusterIP` is used for internal communication.
- `NodePort` exposes the Service on every node.
- `LoadBalancer` creates an external cloud load balancer.
- `ExternalName` maps a Service name to an external DNS name.