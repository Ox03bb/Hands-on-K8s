<img src="./assets/secret/icons.drawio.svg" alt="LoadBalancer Service" width="60"/>

# Kubernetes Secrets

A **Secret** is a Kubernetes object used to store **sensitive information** such as:

- Database passwords
- API keys
- OAuth tokens
- SSH keys
- TLS certificates
- Docker registry credentials

Instead of embedding credentials inside a Pod or Deployment, Kubernetes allows them to be stored separately and injected into containers when needed.


# Why Secrets Exist

Imagine a Deployment like this:

```yaml
env:
- name: DB_PASSWORD
  value: password123
```

Problems:

- Password is visible in Git.
- Everyone with repository access can read it.
- Changing the password requires editing Deployment files.
- Multiple applications cannot easily reuse it.

Instead:

```
Deployment
        │
        ▼
   Kubernetes Secret
        │
        ▼
Container Environment Variables
```

The Deployment references the Secret rather than containing the password directly.



# Secret vs ConfigMap

| ConfigMap | Secret |
|------------|---------|
| Non-sensitive data | Sensitive data |
| Plain text | Base64 encoded |
| Application configuration | Passwords, certificates, tokens |
| Public configuration | Restricted access |

Example:

ConfigMap

```yaml
APP_NAME=myapp
APP_PORT=8080
```

Secret

```yaml
DB_PASSWORD=password123
API_KEY=xxxxx
```


# How Secrets Work

```
              kubectl apply
                    │
                    ▼
              Kubernetes API
                    │
                    ▼
                 Secret Object
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
 Environment Variables      Mounted Volume
        ▼                       ▼
     Application           Filesystem
```

Secrets are stored in the Kubernetes API and can later be injected into Pods.



# Secret Types

## 1. Opaque (Default)

Generic key-value Secret.

```yaml
type: Opaque
```

Example:

```yaml
stringData:
  username: admin
  password: secret123
```

Most commonly used.


## 2. kubernetes.io/tls

Stores:

- TLS certificate
- Private key

Required keys:

```text
tls.crt
tls.key
```

Example:

```yaml
type: kubernetes.io/tls
```

Used by:

- Ingress
- HTTPS servers
- Web applications


## 3. kubernetes.io/dockerconfigjson

Stores Docker registry credentials.

Required key:

```
.dockerconfigjson
```

Example:

```bash
kubectl create secret docker-registry registry-secret \
    --docker-server=docker.io \
    --docker-username=user \
    --docker-password=password
```

Used when pulling private images.


## 4. kubernetes.io/basic-auth

Stores

```
username
password
```


## 5. kubernetes.io/ssh-auth

Stores SSH private keys.

Required key:

```
ssh-privatekey
```


## 6. kubernetes.io/service-account-token

Automatically created by Kubernetes for ServiceAccounts.

Used to authenticate Pods with the Kubernetes API.


## 7. bootstrap.kubernetes.io/token

Used during kubeadm cluster bootstrapping.


# Creating Secrets

## Method 1 — YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque

stringData:
  username: admin
  password: mypassword
```

Apply

```bash
kubectl apply -f secret.yaml
```


## Method 2 — kubectl

```bash
kubectl create secret generic mongo-secret \
    --from-literal=username=admin \
    --from-literal=password=mypassword
```


## Method 3 — From Files

```bash
kubectl create secret generic app-secret \
    --from-file=config.txt
```


## Method 4 — Directory

```bash
kubectl create secret generic app-secret \
    --from-file=./certificates
```

Each file becomes a key.


# Secret Manifest

Example

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: mongo-secret

type: Opaque

data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```


# data vs stringData

## data

Must be Base64 encoded.

```yaml
data:
  username: YWRtaW4=
```

Encoding:

```bash
echo -n admin | base64
```


## stringData

Accepts plain text.

```yaml
stringData:
  username: admin
```

Kubernetes automatically converts it into Base64.

Recommendation:

Always use `stringData` when writing YAML manually.


# Why Base64?

Many beginners believe Base64 provides security.

It does not.

Base64 only converts binary data into text.

```
password

↓

cGFzc3dvcmQ=
```

Anyone can decode it.

Example:

```bash
echo "cGFzc3dvcmQ=" | base64 -d
```

Output

```
password
```

Base64 is used because Kubernetes stores objects as JSON/YAML, which cannot safely represent arbitrary binary data.


# Using Secrets

Secrets can be consumed in three primary ways.


## Method 1 — Environment Variables

```yaml
env:
- name: DB_USERNAME
  valueFrom:
    secretKeyRef:
      name: mongo-secret
      key: username

- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mongo-secret
      key: password
```

Application:

```
DB_USERNAME=admin
DB_PASSWORD=password
```


## Method 2 — Mount as Volume

```yaml
volumes:
- name: secret-volume

  secret:
    secretName: mongo-secret
```

Mount

```yaml
volumeMounts:
- name: secret-volume
  mountPath: /etc/secrets
```

Result

```
/etc/secrets

├── username
└── password
```

Reading

```bash
cat /etc/secrets/password
```


## Method 3 — Image Pull Secret

```yaml
spec:

  imagePullSecrets:

  - name: registry-secret
```

Used for private container registries.


# Updating Secrets

Edit

```bash
kubectl edit secret mongo-secret
```

Or

```bash
kubectl apply -f secret.yaml
```

Verify

```bash
kubectl get secret
```

Describe

```bash
kubectl describe secret mongo-secret
```

View

```bash
kubectl get secret mongo-secret -o yaml
```

Decode

```bash
kubectl get secret mongo-secret \
-o jsonpath='{.data.password}' \
| base64 -d
```


# Security Considerations

Secrets are **NOT encrypted by default.**

Base64

❌ Encryption

Base64

❌ Hashing

Base64

✅ Encoding

Security comes from:

- RBAC
- TLS
- Secret mounting
- Encryption at Rest


# Encryption at Rest

Default

```
Secret

↓

API Server

↓

etcd (plain text)
```

With encryption

```
Secret

↓

API Server

↓

AES Encryption

↓

etcd
```

Enable encryption using an EncryptionConfiguration.


# Best Practices

## Never commit Secrets to Git

Bad

```yaml
password: admin123
```

Good

Use:

- External Secrets
- Vault
- Sealed Secrets


**Use RBAC**
Only authorized users should read Secrets.


**Enable Encryption at Rest**
Always encrypt etcd in production.


**Rotate Secrets**
Do not use the same credentials forever.


**Avoid Hardcoding**
Never write passwords inside Deployments.


**Use stringData**
It is easier to maintain than Base64.


**Limit Secret Scope**
Each application should only receive the Secrets it needs.


# Common Commands

Create

```bash
kubectl create secret generic app-secret \
--from-literal=username=admin
```

List

```bash
kubectl get secrets
```

Describe

```bash
kubectl describe secret app-secret
```

View YAML

```bash
kubectl get secret app-secret -o yaml
```

Delete

```bash
kubectl delete secret app-secret
```

Decode

```bash
kubectl get secret app-secret \
-o jsonpath='{.data.password}' \
| base64 -d
```


# Complete Example

## Secret

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: mongo-secret

type: Opaque

stringData:
  username: admin
  password: password123
```


## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mongo-client

spec:

  replicas: 1

  selector:
    matchLabels:
      app: mongo-client

  template:

    metadata:
      labels:
        app: mongo-client

    spec:

      containers:

      - name: app

        image: busybox

        command:
        - sleep
        - "3600"

        env:

        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: username

        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: password
```

Verify

```bash
kubectl exec -it <pod-name> -- env
```

Output

```
DB_USERNAME=admin
DB_PASSWORD=password123
```


# Common Mistakes

❌ Assuming Base64 is encryption.

❌ Storing Secrets inside Git.

❌ Hardcoding passwords in Deployments.

❌ Giving every Pod access to every Secret.

❌ Forgetting RBAC.

❌ Forgetting Encryption at Rest.


# Summary

- A Secret stores sensitive information.
- Secrets are Kubernetes API objects.
- Base64 is **encoding**, not encryption.
- `Opaque` is the default and most commonly used Secret type.
- Secrets can be consumed through environment variables, mounted files, or image pull credentials.
- Use `stringData` when writing YAML manually.
- Secure Secrets with RBAC, TLS, and Encryption at Rest.
- Avoid storing Secrets in version control, and rotate credentials regularly.