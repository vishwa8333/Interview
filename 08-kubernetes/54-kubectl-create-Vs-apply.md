# kubectl create vs kubectl apply

This document explains the **difference between `kubectl create` and `kubectl apply`**, how they work internally, when to use each, and practical examples.

---

## 1. What is `kubectl create`?

`kubectl create` is used to **create a new Kubernetes resource for the first time**.

If the resource **already exists**, the command **fails**.

### Characteristics

* Used for **initial creation only**
* Does **not track future changes**
* Fails if the object already exists
* Mostly **imperative-style** workflow

### Example 1: Create from YAML

```bash
kubectl create -f deployment.yaml
```

If `deployment.yaml` defines a Deployment named `nginx-deploy`:

* Kubernetes creates the Deployment
* A ReplicaSet is created
* Pods are created

Running the same command again:

```bash
kubectl create -f deployment.yaml
```

Result:

```
Error from server (AlreadyExists): deployments.apps "nginx-deploy" already exists
```

---

### Example 2: Create directly from CLI

```bash
kubectl create deployment nginx --image=nginx
```

This:

* Creates a Deployment
* Auto-generates a basic spec
* Does **not store desired state locally**

---

## 2. What is `kubectl apply`?

`kubectl apply` is used to **create OR update resources declaratively**.

Kubernetes **stores the desired state** and continuously reconciles it.

### Characteristics

* Used for **create + update**
* Tracks changes using **last-applied-configuration**
* Supports **declarative workflows**
* Safe to run multiple times

### Example 3: Apply from YAML

```bash
kubectl apply -f deployment.yaml
```

Behavior:

* If resource does not exist → **created**
* If resource exists → **updated**

Running it multiple times:

```bash
kubectl apply -f deployment.yaml
```

Result:

```
deployment.apps/nginx-deploy configured
```

---

## 3. Internal Difference (Important)

### `kubectl create`

* Sends a **POST** request to API Server
* No comparison with existing state
* No patching logic

### `kubectl apply`

* Uses **three-way merge**:

  1. Last-applied configuration (stored in annotation)
  2. Current live object
  3. New desired YAML

Annotation used:

```yaml
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: "..."
```

This allows Kubernetes to:

* Update only changed fields
* Avoid overwriting unrelated changes

---

## 4. Update Behavior Example

### Initial YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 2
```

```bash
kubectl apply -f deployment.yaml
```

### Modify replicas to 5

```yaml
spec:
  replicas: 5
```

```bash
kubectl apply -f deployment.yaml
```

Result:

* Old ReplicaSet scaled down
* New ReplicaSet scaled up
* Pods adjusted automatically

With `kubectl create` → this update is **not possible**

---

## 5. Can `apply` replace `create`?

Yes, in most real-world scenarios.

You can safely use:

```bash
kubectl apply -f resource.yaml
```

For:

* First-time creation
* Repeated updates
* GitOps workflows

---

## 6. When to Use What?

| Use Case                 | Recommended Command |
| ------------------------ | ------------------- |
| One-time quick resource  | kubectl create      |
| Production deployments   | kubectl apply       |
| CI/CD pipelines          | kubectl apply       |
| GitOps / version control | kubectl apply       |
| Learning / demos         | kubectl create      |

---

## 7. Common Interview Summary

* `create` → **create once, fail if exists**
* `apply` → **create or update declaratively**
* `apply` stores desired state using annotations
* `apply` supports reconciliation and GitOps

---

## 8. Simple Rule to Remember

> **If you want Kubernetes to manage state → use `kubectl apply`**
> **If you just want to create something once → use `kubectl create`**

---

## 9. Bonus Tip

To convert imperative `create` to declarative YAML:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

Then manage it using:

```bash
kubectl apply -f deployment.yaml
```

---

End of document.
