# ⚖️ HPA & VPA Demo in Kubernetes

This demo explains **Horizontal Pod Autoscaler (HPA)** and **Vertical Pod Autoscaler (VPA)** with:

* real-world use cases 🧠
* step-by-step setup 🪜
* working YAML examples 📄

---

# 🚀 Part 1: Horizontal Pod Autoscaler (HPA)

## 🧠 What is HPA?

HPA automatically **scales the number of Pods** based on resource usage (CPU / memory) or custom metrics.

> 📈 More traffic → more Pods

---

## 🎯 HPA Use Case (Real World)

👉 A web application where traffic spikes during peak hours.

* Low traffic → 1–2 Pods 🌙
* High traffic → 10+ Pods 🌞

---

## ⚙️ Prerequisites for HPA

✅ Metrics Server must be running

```bash
kubectl get deployment metrics-server -n kube-system
```

If not installed:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

## 🧩 Step 1: Create Deployment for HPA

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa-demo
  template:
    metadata:
      labels:
        app: hpa-demo
    spec:
      containers:
      - name: app
        image: k8s.gcr.io/hpa-example
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "500m"
```

---

## 🧩 Step 2: Create HPA Object

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-demo
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

## 🧪 Step 3: Generate Load

```bash
kubectl run -i --tty load-generator --image=busybox /bin/sh
```

Inside the pod:

```bash
while true; do wget -q -O- http://hpa-demo; done
```

---

## 📊 Step 4: Observe HPA Behavior

```bash
kubectl get hpa -w
kubectl get pods
```

🔍 Result:

* CPU ↑ → Pods scale out ➕
* CPU ↓ → Pods scale in ➖

---

## 🔄 HPA Internal Flow

1️⃣ Metrics Server collects CPU data 📊
2️⃣ HPA Controller evaluates metrics 🧠
3️⃣ Desired replicas calculated ➗
4️⃣ Deployment updated 📄
5️⃣ New Pods created or removed ⚙️

---

# 🧠 Part 2: Vertical Pod Autoscaler (VPA)

## 🧠 What is VPA?

VPA automatically **adjusts CPU & memory requests/limits** of Pods.

> 📦 Same Pods, better sizing

---

## 🎯 VPA Use Case (Real World)

👉 Backend service with unpredictable memory usage.

* Over-allocated resources → wasted money 💸
* Under-allocated resources → OOMKills 💥

VPA solves this by right-sizing Pods 🎯

---

## ⚙️ Prerequisites for VPA

VPA is **not installed by default**.

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
```

Verify:

```bash
kubectl get pods -n kube-system | grep vpa
```

---

## 🧩 Step 1: Create Deployment for VPA

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vpa-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: vpa-demo
  template:
    metadata:
      labels:
        app: vpa-demo
    spec:
      containers:
      - name: app
        image: nginx
        resources:
          requests:
            cpu: "50m"
            memory: "50Mi"
```

---

## 🧩 Step 2: Create VPA Object

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: vpa-demo
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vpa-demo
  updatePolicy:
    updateMode: Auto
```

---

## 🔄 Step 3: Observe VPA in Action

```bash
kubectl describe vpa vpa-demo
```

🔍 What happens:

* VPA recommends new CPU/memory values 📈
* Pods are restarted 🔄
* New requests applied automatically

---

## 🧠 VPA Internal Flow

1️⃣ Metrics collected over time 📊
2️⃣ VPA Recommender calculates ideal resources 🧮
3️⃣ Admission Controller mutates Pod spec ✍️
4️⃣ Pod recreated with new requests 🔁

---

# ⚠️ HPA vs VPA (Important Rule)

❌ Do NOT use HPA and VPA on **CPU at the same time**

| Feature            | HPA            | VPA             |
| ------------------ | -------------- | --------------- |
| Scales Pods        | ✅              | ❌               |
| Changes CPU/Memory | ❌              | ✅               |
| Pod Restart        | ❌              | ✅               |
| Best For           | Traffic spikes | Resource tuning |

---

## ⭐ Best Practices

✅ Use **HPA for stateless apps** 🌐
✅ Use **VPA for backend & batch workloads** 🧮
✅ Combine HPA (CPU) + VPA (Memory) carefully 🧠
✅ Monitor before enabling Auto mode 👀

---

## 🧩 One-Line Summary

> **HPA scales Pods horizontally 📈, VPA scales resources vertically 📦 — both solve different scaling problems in Kubernetes 🚀**

---

---

# 🧩 Why `apiVersion: apps/v1` Is Used

## ❓ What does `apps/v1` mean?

In Kubernetes, every object is identified by:

```
API Group + Version
```

```yaml
apiVersion: apps/v1
kind: Deployment
```

* `apps` → API group for **application workloads**
* `v1` → **stable, production-ready** version

---

## 📦 Resources That Use `apps/v1`

| Resource    | Reason                     |
| ----------- | -------------------------- |
| Deployment  | Manages ReplicaSets & Pods |
| ReplicaSet  | Ensures desired Pod count  |
| StatefulSet | Stateful workloads         |
| DaemonSet   | One Pod per Node           |

All these control Pods at scale, so they live in the **apps API group**.

---

## 🏗️ Why Not Just `apiVersion: v1`?

`v1` (without a group) is the **core API group**, used for basic building blocks:

```yaml
kind: Pod
kind: Service
kind: ConfigMap
kind: Secret
```

Workload controllers evolved separately, so Kubernetes moved them to `apps` for:

* better separation of concerns
* safer evolution
* backward compatibility

---

## 🕰️ History (Interview Favorite ⭐)

Older, deprecated versions:

```yaml
extensions/v1beta1
apps/v1beta1
apps/v1beta2
```

🚫 Removed since Kubernetes **1.16**

✅ Today, **`apps/v1` is mandatory** for Deployments.

---

## ⚙️ What `apps/v1` Enforces (Very Important)

In `apps/v1`, some fields are **mandatory**.

Example: `spec.selector`

```yaml
spec:
  selector:
    matchLabels:
      app: my-app
```

This prevents orphaned ReplicaSets and broken rollouts.

---

## 🔄 How API Server Uses `apps/v1`

1️⃣ YAML applied by user
2️⃣ API Server reads `apps/v1`
3️⃣ Routes to Apps API handler
4️⃣ Validates against version schema
5️⃣ Stores object in etcd
6️⃣ Controller acts on it

---

## 🧠 One-Line Interview Answer

> **Deployments use `apps/v1` because they belong to the Apps API group, and `v1` is the stable, production-safe version supported by modern Kubernetes clusters.**

---

---

# 🔥 Can HPA and VPA Be Used Together?

## ❓ Short Answer

✅ **Yes — but only in a controlled and safe configuration**

❌ **No — if both try to control the same resource (CPU)**

---

## 🚫 Golden Rule (Very Important)

> **Never allow HPA and VPA to control the same resource metric at the same time.**

Why?

* HPA scales Pods based on resource utilization 📈
* VPA changes resource requests 📦
* Both acting on CPU causes feedback loops 🔄

---

## ❌ Unsafe Combination (Do NOT Use)

```text
HPA → scales Pods based on CPU utilization
VPA → modifies CPU requests
```

### What Happens Internally?

1️⃣ VPA increases CPU request 📦
2️⃣ CPU utilization percentage drops 📉
3️⃣ HPA scales Pods down ➖
4️⃣ Load increases again 📈
5️⃣ HPA scales Pods up ➕

⚠️ Result: unstable scaling loop

---

## ✅ Safe & Recommended Pattern (Production Ready)

### ✔️ HPA on CPU + VPA on Memory

| Component | Controls      | Benefit                |
| --------- | ------------- | ---------------------- |
| HPA       | CPU / traffic | Handles load spikes 📈 |
| VPA       | Memory only   | Prevents OOMKills 💥   |

Both autoscalers work on **different dimensions**, so no conflict occurs.

---

## 🧩 Safe Configuration Example

### VPA – Memory Only

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: vpa-demo
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  resourcePolicy:
    containerPolicies:
    - containerName: "*"
      controlledResources: ["memory"]
  updatePolicy:
    updateMode: Auto
```

### HPA – CPU Based

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```

---

## 🧠 Alternative Safe Patterns

### 🔹 VPA in Recommendation Mode

```yaml
updatePolicy:
  updateMode: Off
```

* No Pod restarts 🔕
* Only recommendations shown 📊
* Safe with any HPA setup

---

## 🎯 Real-World Use Case

👉 E-commerce backend service

* Traffic spikes → handled by **HPA** 🛒
* Memory-heavy workloads → handled by **VPA** 🧠
* Result → stable scaling + cost efficiency 💰

---

## 🧠 One-Line Interview Answer

> **HPA and VPA can be used together only if they do not act on the same resource. The recommended approach is HPA on CPU or traffic metrics and VPA on memory.**

---

🎉 You now have a complete **hands-on demo for HPA & VPA**, ready for labs, interviews, and production discussions!
