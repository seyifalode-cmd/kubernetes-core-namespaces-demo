**Kubernetes Core Namespaces Demo** — isolating workloads across development and production environments within a single cluster using Kubernetes Namespaces.

---

## Project at a Glance

| | |
|---|---|
| **Tools Used** | Kubernetes (kubectl), YAML manifests |
| **Platform** | Kubernetes cluster (local or cloud-hosted) |
| **Languages** | YAML |
| **What It Does** | Deploys the same React application into three different namespaces (default, development, production) with differing replica counts, demonstrating how namespaces enforce workload isolation and context-scoping within a shared cluster |

---

## The Problem This Project Solves

In a shared Kubernetes cluster, multiple teams or environments running side by side quickly create operational problems. Without logical separation, a developer applying changes in their environment can accidentally affect production workloads. Listing pods or services without a namespace filter returns noise from every tenant in the cluster. Access controls become coarse-grained and difficult to reason about.

Kubernetes Namespaces solve this by creating virtual sub-clusters within a single physical cluster. Each namespace has its own scope for resources, names, and eventually — when combined with RBAC and NetworkPolicies — its own security boundary. A pod in the `development` namespace cannot inadvertently reference a service in `production` by the same name, because each namespace maintains its own DNS scope for cluster-internal resolution.

This project walks through the practical mechanics of that isolation: creating a namespace declaratively, deploying the same application into three different namespaces with intentionally different replica counts, and using kubectl context switching to observe how the active namespace changes what is visible to an operator. These are day-one skills for any engineer working with multi-team or multi-environment Kubernetes deployments.

---

## Architecture

```
Kubernetes Cluster
│
├── Namespace: default
│     └── react-app-deployment (1 replica)
│           Service: react-app-service  NodePort 30001
│
├── Namespace: development
│     └── react-app-deployment (2 replicas)
│           Service: react-app-service  NodePort 30002
│
└── Namespace: production
      └── react-app-deployment (4 replicas)
            Service: react-app-service  NodePort 30003
```

The same container image (`wessamabdelwahab/react-app:latest`) is deployed across all three namespaces. Each namespace is scoped independently — services in `development` and `production` share the same resource names without conflict because the namespace acts as a prefix in the cluster's internal DNS.

---

## Repository Structure

```
kubernetes-core-namespaces-demo/
├── README.md
├── namespace-dev.yml          # Declarative Namespace definition for development
├── react-app-pod.yml          # Deployment + Service in the default namespace
├── react-app-pod-DEV.yml      # Deployment + Service in the development namespace (2 replicas)
├── react-app-pod-PROD.yml     # Deployment + Service in the production namespace (4 replicas)
├── simple-pod.yml             # Basic nginx pod manifest (reference example)
├── simple-pod2.yml            # Alternate basic pod manifest (reference example)
└── simple-replicaset.yml      # Basic nginx ReplicaSet manifest (reference example)
```

---

## Walkthrough

### Step 1: Review existing namespaces

```bash
kubectl get namespaces
```

This returns the default namespaces every cluster ships with: `default`, `kube-system`, `kube-public`, and `kube-node-lease`.

### Step 2: Create the development namespace

```bash
kubectl create -f namespace-dev.yml
```

The manifest declares a namespace named `development`. The production namespace is created from the upstream Kubernetes example:

```bash
kubectl create -f https://k8s.io/examples/admin/namespace-prod.json
```

### Step 3: Deploy the application across namespaces

```bash
# Deploy to default namespace (1 replica, NodePort 30001)
kubectl create -f react-app-pod.yml

# Deploy to development namespace (2 replicas, NodePort 30002)
kubectl create -f react-app-pod-DEV.yml

# Deploy to production namespace (4 replicas, NodePort 30003)
kubectl create -f react-app-pod-PROD.yml
```

### Step 4: Observe namespace-scoped visibility

```bash
# Pods visible only in development
kubectl get pods -l app=react-app --namespace=development

# Pods visible only in production
kubectl get pods --namespace=production

# Pods visible only in default
kubectl get pods -l app=react-app --namespace=default
```

### Step 5: Switch the active context namespace

```bash
# Set active namespace to production
kubectl config set-context --current --namespace=production

# Now 'kubectl get pods' returns production pods without a --namespace flag
kubectl get pods

# Default namespace pods are still accessible but require explicit flag
kubectl get pods -l app=react-app --namespace=default
```

### Step 6: Inspect the active context

```bash
kubectl config get-contexts
```

### Step 7: Review pods across all namespaces

```bash
kubectl get pods --all-namespaces
```

---

## How to Reproduce

Prerequisites: A running Kubernetes cluster with `kubectl` configured.

```bash
# Clone the repository
git clone <repo-url>
cd kubernetes-core-namespaces-demo

# Create namespaces
kubectl create -f namespace-dev.yml
kubectl create -f https://k8s.io/examples/admin/namespace-prod.json

# Deploy application into all three namespaces
kubectl create -f react-app-pod.yml
kubectl create -f react-app-pod-DEV.yml
kubectl create -f react-app-pod-PROD.yml

# Verify deployments
kubectl get pods --all-namespaces -l app=react-app

# Verify services and NodePorts
kubectl get services --all-namespaces -l app=react-app

# Clean up
kubectl delete -f react-app-pod.yml
kubectl delete -f react-app-pod-DEV.yml
kubectl delete -f react-app-pod-PROD.yml
kubectl delete namespace development production
```

---

## Key Observations

- The `default` namespace deploys 1 replica; `development` deploys 2; `production` deploys 4. The deployment manifests are structurally identical aside from the `namespace` field and replica count.
- Context switching with `kubectl config set-context` changes what `kubectl get pods` returns without requiring a `--namespace` flag every time — a workflow convenience in long-running operational sessions.
- Namespaces do not provide network isolation by themselves. NetworkPolicies must be applied separately to restrict cross-namespace traffic.

---

*Oluwaseyi Michael Falode · Cybersecurity & Cloud Security Engineer · Toronto, ON*
