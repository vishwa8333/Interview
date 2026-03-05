# NodeSelector in Kubernetes

This document explains **nodeSelector** in detail: what it is, how it works internally, why it exists, and real-world use cases with examples.

---

## 1. What is nodeSelector?

`nodeSelector` is a **simple Pod scheduling mechanism** in Kubernetes.

It allows you to constrain Pods so that they are **scheduled only on nodes with specific labels**.

> nodeSelector works by **matching Pod requirements with node labels**.

---

## 2. Why do we need nodeSelector?

By default, the Kubernetes scheduler can place a Pod on **any available node**.

In real clusters, nodes are **not identical**:

* Some nodes have SSDs
* Some have GPUs
* Some belong to a specific zone or environment
* Some are dedicated to certain workloads

`nodeSelector` gives **basic control** over *where* Pods run.

---

## 3. How nodeSelector works (Internals)

Scheduling flow:

1. Scheduler watches for **unscheduled Pods**
2. It reads Pod `nodeSelector`
3. It filters nodes that **do not match labels**
4. Pod is scheduled only on **matching nodes**
5. If no node matches → Pod remains **Pending**

Important:

* Matching is **AND-based**
* All labels must match exactly

---

## 4. Node Labels (Foundation of nodeSelector)

nodeSelector works only with **node labels**.

### View node labels

```bash
kubectl get nodes --show-labels
```

### Add a label to a node

```bash
kubectl label node node-1 disktype=ssd
```

### Remove a label

```bash
kubectl label node node-1 disktype-
```

---

## 5. Basic nodeSelector Example

### Step 1: Label the nodes

```bash
kubectl label node worker-1 disktype=ssd
kubectl label node worker-2 disktype=hdd
```

---

### Step 2: Pod using nodeSelector

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-ssd
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: nginx
    image: nginx
```

Result:

* Pod runs **only on worker-1**
* Scheduler will ignore other nodes

---

## 6. Deployment Example (Most Common Use)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      nodeSelector:
        nodegroup: backend
      containers:
      - name: app
        image: nginx
```

All Pods created by this Deployment:

* Will be scheduled **only on nodes labeled** `nodegroup=backend`

---

## 7. Real-World Use Cases

### Use Case 1: SSD vs HDD workloads

Problem:

* Databases need fast disks

Solution:

* Label SSD nodes
* Schedule DB Pods only on them

```yaml
nodeSelector:
  disktype: ssd
```

---

### Use Case 2: GPU workloads

Problem:

* ML workloads require GPUs

Solution:

* Label GPU nodes

```bash
kubectl label node gpu-node accelerator=nvidia
```

```yaml
nodeSelector:
  accelerator: nvidia
```

---

### Use Case 3: Environment separation

Problem:

* Keep prod and non-prod workloads separate

Solution:

* Label nodes by environment

```yaml
nodeSelector:
  env: prod
```

---

### Use Case 4: Zone-based scheduling

Problem:

* Latency-sensitive apps

Solution:

* Pin workloads to a specific zone

```yaml
nodeSelector:
  topology.kubernetes.io/zone: ap-south-1a
```

---

## 8. What happens if no node matches?

If no node satisfies the nodeSelector:

* Pod stays in **Pending** state
* Scheduler events show:

```text
0/3 nodes are available: node(s) didn't match node selector
```

---

## 9. Limitations of nodeSelector

nodeSelector is **intentionally simple**.

Limitations:

* Only **exact match**
* No OR conditions
* No priorities or weights
* Cannot express complex rules

---

## 10. nodeSelector vs nodeAffinity

| Feature           | nodeSelector | nodeAffinity |
| ----------------- | ------------ | ------------ |
| Match type        | Exact        | Expressions  |
| OR / AND          | AND only     | AND / OR     |
| Soft rules        | No           | Yes          |
| Weight / priority | No           | Yes          |

nodeSelector is a **subset of nodeAffinity**.

---

## 11. Best Practices

* Use nodeSelector for **simple, strict placement**
* Combine with **taints & tolerations** for isolation
* Prefer **nodeAffinity** for complex scheduling
* Always monitor Pods stuck in Pending

---

## 12. Interview One-Liner

> nodeSelector is a basic scheduling constraint that binds Pods to nodes with matching labels; if no node matches, the Pod is never scheduled.

---

End of document.
