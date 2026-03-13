### Kubernetes Architecture:

#### Control Plane Components:

- **kube-apiserver**: The entry point for all REST API requests.
- **etcd**: A distributed key-value store for storing all cluster data.
- **kube-scheduler**: Watches for new Pods and assigns them to Nodes.
- **kube-controller-manager**: Runs various controllers (Node Controller, Replication Controller, etc.).

#### Worker Node Components:

- **kubelet**: An agent that runs on each Node and ensures containers are running in a Pod. Receives intructions from API server.
- **kube-proxy**: A network proxy that maintains network rules on Nodes. It is responsible for network connectivity between Pods.
- **Container Runtime**: Docker, containerd, or CRI-O.



#### To create a kind K8S Cluster:
``` bash 
kind create cluster --image kindest/node:v1.35.0@sha256:452d707d4862f52530247495d180205e029056831160e22870e37e3f6c1ac31f --name cka-cluster1
```

#### To get cluster info:
``` bash 
kubectl cluster-info --context kind-cka-cluster1
```

#### Replication Controller:

A **Replication Controller (RC)** is a Kubernetes object that ensures a specified number of Pod replicas are running at any given time. It is the **older** mechanism for maintaining Pod availability.

**Key Characteristics:**
- Ensures the desired number of Pods are always running.
- If a Pod crashes or is deleted, the RC automatically creates a new one.
- If there are too many Pods, it terminates the extras.
- Uses **equality-based selectors** only (e.g., `app=nginx`) — cannot use `matchExpressions`.
- Considered **legacy** — superseded by ReplicaSet.

**How it works:**
1. The RC watches the cluster state via the API server.
2. It compares the actual number of running Pods against the desired `replicas` count.
3. It creates or deletes Pods to match the desired state.

**Example Manifest** (`replicationcontroller.yml`):
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
  labels:
    env: demo
spec:
  replicas: 3
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

**Useful Commands:**
```bash
# Create a Replication Controller
kubectl apply -f replicationcontroller.yml

# List Replication Controllers
kubectl get rc

# Describe a Replication Controller
kubectl describe rc nginx-rc

# Scale a Replication Controller
kubectl scale rc nginx-rc --replicas=5

# Delete a Replication Controller (also deletes its Pods)
kubectl delete rc nginx-rc
```

---

#### ReplicaSet:

A **ReplicaSet (RS)** is the **next-generation** replacement for the Replication Controller. It has the same purpose — maintaining a stable set of replica Pods — but with more expressive selector capabilities.

**Key Characteristics:**
- Supports both **equality-based** (`matchLabels`) and **set-based** (`matchExpressions`) selectors.
- Recommended to use via a **Deployment** (which manages ReplicaSets under the hood).
- Can adopt existing Pods if their labels match the selector.
- Provides fine-grained control over Pod management.

**Difference from Replication Controller:**

| Feature | ReplicationController | ReplicaSet |
|---|---|---|
| API Version | `v1` | `apps/v1` |
| Selector Type | Equality-based only | Equality + Set-based |
| Status | Legacy (deprecated) | Current standard |
| Used by Deployment | ❌ No | ✅ Yes |
| `matchExpressions` support | ❌ No | ✅ Yes |

**Example Manifest** (`replicaset.yml`):
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    env: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      tier: frontend
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

> **Note:** The `selector` field is **mandatory** in ReplicaSet (unlike ReplicationController where it was optional and inferred from the template labels).

**Using `matchExpressions` (advanced selector):**
```yaml
selector:
  matchExpressions:
    - key: app
      operator: In
      values:
        - nginx
        - web
    - key: tier
      operator: NotIn
      values:
        - backend
```
Supported operators: `In`, `NotIn`, `Exists`, `DoesNotExist`.

**Useful Commands:**
```bash
# Create a ReplicaSet
kubectl apply -f replicaset.yml

# List ReplicaSets
kubectl get rs

# Describe a ReplicaSet
kubectl describe rs nginx-rs

# Scale a ReplicaSet
kubectl scale rs nginx-rs --replicas=5

# Delete a ReplicaSet (also deletes its Pods by default)
kubectl delete rs nginx-rs

# Delete ReplicaSet but keep the Pods running
kubectl delete rs nginx-rs --cascade=orphan
```

---

> **Best Practice:** In production, prefer using a **Deployment** over a bare ReplicaSet. Deployments wrap ReplicaSets and additionally provide rolling updates, rollback capabilities, and revision history.

---

#### Deployment:

A **Deployment** is the most commonly used higher-level Kubernetes object for managing stateless applications. It wraps a **ReplicaSet** and adds powerful capabilities like **rolling updates**, **rollbacks**, and **revision history**.

**Key Characteristics:**
- Declaratively manages a ReplicaSet and the Pods it controls.
- Supports **RollingUpdate** (default) and **Recreate** update strategies.
- Keeps a **revision history** so you can roll back to a previous state at any time.
- Automatically replaces unhealthy or crashed Pods.
- Scaling, updating, and rolling back can all be done without downtime.

**How it works internally:**
```
Deployment  →  manages  →  ReplicaSet  →  manages  →  Pods
```
When you update a Deployment (e.g., change the container image), Kubernetes creates a **new ReplicaSet** for the new version while gradually scaling down the old one — this is called a **Rolling Update**.

**Example Manifest** (`deployments.yml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    env: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      tier: frontend
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

**Rolling Update Strategy (default):**

From `kubectl describe deploy/nginx-deployment`:
```
StrategyType:           RollingUpdate
RollingUpdateStrategy:  25% max unavailable, 25% max surge
```

| Parameter | Meaning |
|---|---|
| `maxUnavailable: 25%` | At most 25% of desired Pods can be unavailable during the update |
| `maxSurge: 25%` | At most 25% extra Pods can be created above the desired count during the update |

This means for `replicas: 3`, at most **1 Pod** can be unavailable and **1 extra Pod** can be created at a time during a rolling update — ensuring near-zero downtime.

---

**Useful Commands:**

```bash
# Apply / Create a Deployment
kubectl apply -f deployments.yml

# List all Deployments
kubectl get deployments
kubectl get deploy

# List all resources (Deployment + ReplicaSet + Pods)
kubectl get all

# Describe a Deployment (shows strategy, replicas, events, etc.)
kubectl describe deploy/nginx-deployment

# Scale a Deployment
kubectl scale deployment/nginx-deployment --replicas=5

# Update container image (triggers a rolling update → new revision)
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1

# Watch rollout status in real time
kubectl rollout status deployment/nginx-deployment

# View rollout revision history
kubectl rollout history deployment/nginx-deployment

# Roll back to the previous revision
kubectl rollout undo deployment/nginx-deployment

# Roll back to a specific revision number
kubectl rollout undo deployment/nginx-deployment --to-revision=1

# Pause a rollout (to batch multiple changes)
kubectl rollout pause deployment/nginx-deployment

# Resume a paused rollout
kubectl rollout resume deployment/nginx-deployment

# Delete a Deployment (also deletes its ReplicaSets and Pods)
kubectl delete deploy/nginx-deployment
kubectl delete -f deployments.yml
```

---

**What you did in the terminal — explained:**

| Command | What happened |
|---|---|
| `kubectl apply -f deployments.yml` | Created `nginx-deployment` with `nginx:latest`, Revision **1** |
| `kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1` | Triggered rolling update → new ReplicaSet created, Revision **2** |
| `kubectl describe deploy/nginx-deployment` | Showed old RS (`nginx-deployment-c4485c9b9`, 3 replicas) and new RS (`nginx-deployment-6cd5dfcb9c`, 1 replica) mid-update |
| `kubectl rollout history deployment/nginx-deployment` | Listed Revision 1 (`nginx:latest`) and Revision 2 (`nginx:1.9.1`) |
| `kubectl rollout undo deployment/nginx-deployment` | Rolled back to Revision 1 (`nginx:latest`), which was recorded as Revision **3** |
| `kubectl describe deploy/nginx-deployment` (after undo) | Confirmed all 3 Pods running `nginx:latest`, old RS scaled to 0 |

> **Note:** After a `rollout undo`, the previous revision is re-applied and recorded as a **new** revision number. Revision 1 becomes Revision 3 — Kubernetes always appends to the history rather than overwriting it.

---

**Recreate Strategy (alternative):**
```yaml
spec:
  strategy:
    type: Recreate
```
> All existing Pods are **killed first**, then new ones are created. This causes **downtime** but avoids running two versions simultaneously. Use this only if your app cannot tolerate two versions running at once (e.g., database schema changes).

---

**Annotating Rollout Cause (best practice):**
```bash
# Add a change-cause annotation so rollout history is meaningful
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="Updated nginx to 1.9.1"

# Now history shows:
# REVISION  CHANGE-CAUSE
# 1         <none>
# 2         Updated nginx to 1.9.1
```

