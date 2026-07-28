# 🚀 Kubernetes Deployment Rollout & Rollback (Rollout / Rollout Undo)

This guide explains **how Kubernetes rollouts work**, **how to update a Deployment**, **how to track versions**, and **how to rollback safely** — using **`set image`**, **YAML**, and **revision history**.

---

## 🧠 What is a Rollout?

A **rollout** is the process by which Kubernetes **updates Pods of a Deployment** to a new version.

Each rollout:

* Creates a **new ReplicaSet** 📦
* Assigns a **revision number** (v1, v2, v3…)
* Gradually replaces old Pods (based on strategy)

---

## 🏗 Initial Deployment (Revision 1)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
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
        image: nginx:1.21
```

```bash
kubectl apply -f nginx.yaml
```

✅ Result:

* Deployment created
* ReplicaSet created → **revision 1**
* 3 Pods running nginx:1.21

---

## 🔍 Check Rollout Status

```bash
kubectl rollout status deployment nginx-deploy
```

🟢 Shows:

* Progress of Pods being created
* Whether rollout succeeded or failed

---

## 🧾 Check Rollout History (Versions)

```bash
kubectl rollout history deployment nginx-deploy
```

Example output:

```
REVISION  CHANGE-CAUSE
1         <none>
```

📌 At this stage:

* Only **revision 1** exists

---

## 🔁 Update Deployment using `set image` (Revision 2)

### Command

```bash
kubectl set image deployment nginx-deploy nginx=nginx:1.23
```

### What happens internally 🧩

* New ReplicaSet created
* Old Pods gradually terminated
* New Pods created with nginx:1.23
* Revision increments → **revision 2**

---

## 📝 Record Change Cause (IMPORTANT ⭐)

```bash
kubectl set image deployment nginx-deploy nginx=nginx:1.23 --record
```

Now check history 👇

```bash
kubectl rollout history deployment nginx-deploy
```

Output:

```
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deploy nginx=nginx:1.23 --record
```

🧠 **Change-cause helps during rollback debugging**

---

## 🔄 Update Deployment using YAML (Revision 3)

### Modify YAML

```yaml
containers:
- name: nginx
  image: nginx:1.25
```

Apply:

```bash
kubectl apply -f nginx.yaml
```

📌 Result:

* New ReplicaSet created
* Revision increments → **revision 3**

---

## 🔎 Describe Deployment (See Events + Strategy)

```bash
kubectl describe deployment nginx-deploy
```

Shows:

* Rollout strategy (RollingUpdate)
* MaxSurge / MaxUnavailable
* Events for each rollout

---

## ⏪ Rollback (Undo Rollout)

### Rollback to Previous Version (Revision 2)

```bash
kubectl rollout undo deployment nginx-deploy
```

📌 Kubernetes rolls back to **last successful revision**

---

## 🎯 Rollback to a Specific Revision

```bash
kubectl rollout undo deployment nginx-deploy --to-revision=1
```

Result:

* nginx image goes back to **1.21**
* New ReplicaSet created again
* Revision increases (yes, rollback also creates a new revision!)

---

## 📜 Check History After Rollback

```bash
kubectl rollout history deployment nginx-deploy
```

You’ll see:

```
REVISION  CHANGE-CAUSE
1         initial deploy
2         nginx 1.23 update
3         nginx 1.25 update
4         rollback to revision 1
```

🧠 Rollbacks are **not deletions**, they are **new rollouts**

---

## ⚙️ What Changes Trigger a New Rollout?

Any change in **Pod Template (`spec.template`)** 🚨

### Examples that DO trigger rollout ✅

* Image change 🐳
* Env variable change 🌱
* Resource limits/requests 💾
* Labels/annotations in Pod template 🏷️

### Examples that DO NOT trigger rollout ❌

* Replicas count
* Deployment labels
* Service changes

---

## 🧪 Simulate a Bad Rollout (CrashLoop)

```bash
kubectl set image deployment nginx-deploy nginx=nginx:wrongtag
```

Then:

```bash
kubectl rollout status deployment nginx-deploy
```

❌ Rollout will hang or fail

### Immediate Fix 🔥

```bash
kubectl rollout undo deployment nginx-deploy
```

---

## 🛡 Best Practices

✅ Always use `--record`
✅ Check rollout status after every change
✅ Keep revisionHistoryLimit (default 10)
✅ Rollback instead of hot-fixing Pods

---

## 🧩 Summary Table

| Action              | Command                                                 |
| ------------------- | ------------------------------------------------------- |
| Check rollout       | kubectl rollout status deployment <name>                |
| History             | kubectl rollout history deployment <name>               |
| Update image        | kubectl set image deployment <name> <container>=<image> |
| Apply YAML          | kubectl apply -f file.yaml                              |
| Rollback            | kubectl rollout undo deployment <name>                  |
| Rollback to version | kubectl rollout undo deployment <name> --to-revision=N  |

---

✨ If you want next:

* 🔄 RollingUpdate strategy deep dive
* 🧪 Canary vs Blue-Green with Deployment
* 🖼 Visual rollout flow diagram

Just say the word 😄
