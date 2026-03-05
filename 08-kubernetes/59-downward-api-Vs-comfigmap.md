# 🧪 Downward API (via env) vs ConfigMap — Complete Guide

This canvas gives a **complete, end‑to‑end understanding** of:

* ✅ What **Downward API (via env)** is
* ✅ How it works internally
* ✅ YAML examples
* ✅ How it compares with **ConfigMap**
* ✅ When to use which (real‑world + interviews)

---

## 🧠 What is Downward API (via env)?

**Downward API** allows a container to **consume Pod and container metadata** as **environment variables**, injected by Kubernetes **at container startup time** — without calling the Kubernetes API server.

📌 In simple words:

> Kubernetes tells the container **who it is and where it runs**.

---

## ⬇️ Why is it called *Downward* API?

Information flows **downward**:

```
Kubernetes (Pod / Node / Runtime)
        ↓
   Container (Environment Variables)
```

* No API calls
* No RBAC
* No ServiceAccount token

---

## 📦 What Data Can Downward API Expose?

### 🔹 Pod Metadata

* `metadata.name`
* `metadata.namespace`
* `metadata.uid`

### 🔹 Runtime Info

* `spec.nodeName`
* `status.podIP`

### 🔹 Resource Info

* `resources.requests.cpu`
* `resources.limits.memory`

---

## ✅ Example: Downward API via env (Pod Metadata)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downward-env-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name

    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace

    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName

    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
```

---

## 🔍 Verify Inside the Container

```bash
kubectl exec -it downward-env-demo -- sh
```

```bash
echo $POD_NAME
echo $POD_NAMESPACE
echo $NODE_NAME
echo $POD_IP
```

📌 Values are available **before the container process starts**.

---

## ⚙️ Example: Resource Values via Downward API

```yaml
env:
- name: CPU_LIMIT
  valueFrom:
    resourceFieldRef:
      containerName: app
      resource: limits.cpu

- name: MEMORY_REQUEST
  valueFrom:
    resourceFieldRef:
      containerName: app
      resource: requests.memory
```

📌 Commonly used for:

* JVM heap sizing
* App throttling
* Metrics tagging

---

## ⏱ When Are Downward API env Values Resolved?

| Stage                    | Description       |
| ------------------------ | ----------------- |
| Pod scheduled            | Node assigned     |
| Kubelet starts container | Metadata resolved |
| Container starts         | Env vars injected |

⚠️ Env values are **static** — require container restart to change.

---

# 🧾 What is a ConfigMap?

A **ConfigMap** is a Kubernetes object used to **externalize application configuration** as:

* Environment variables
* Files (via volume)

📌 ConfigMaps are **user‑defined**, not system‑generated.

---

## ✅ Example: ConfigMap via env

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "prod"
  LOG_LEVEL: "debug"
```

```yaml
envFrom:
- configMapRef:
    name: app-config
```

---

## 🧪 Downward API vs ConfigMap (Deep Comparison)

| Aspect           | Downward API (env)     | ConfigMap (env)    |
| ---------------- | ---------------------- | ------------------ |
| Source of data   | Kubernetes system      | User / Ops         |
| Type of data     | Pod identity & runtime | App configuration  |
| Who manages it   | Kubernetes             | Humans             |
| Scope            | Pod‑specific           | Shared across Pods |
| Dynamic updates  | ❌                      | ❌                  |
| Triggers rollout | ❌                      | ❌                  |
| Needs API access | ❌                      | ❌                  |

---

## 🎯 When to Use What?

### ✅ Use Downward API when:

* App needs Pod name, IP, namespace
* Logging & metrics enrichment
* App needs runtime awareness
* JVM/resource‑based tuning

### ✅ Use ConfigMap when:

* Feature flags
* URLs / endpoints
* App behavior configuration
* Environment‑specific values

---

## 🚫 What NOT to Use Them For

❌ Downward API:

* Application configuration
* Secrets
* Cross‑Pod communication

❌ ConfigMap:

* Pod identity
* Runtime metadata
* Sensitive data (use Secret)

---

## 🧠 Mental Model (Very Important)

* **Downward API** → "Who am I & where do I run?" 🪞
* **ConfigMap** → "How should my app behave?" 🧾
* **Secret** → "Sensitive config" 🔐

---

## 🎤 Interview One‑Liners

**Downward API:**

> Allows containers to consume Pod metadata and resource information as environment variables without calling the Kubernetes API server.

**ConfigMap:**

> Used to externalize application configuration and inject it into Pods via env variables or volumes.

---

## 🧩 Key Takeaways

* Downward API is resolved by **kubelet**, not API server
* Env‑based Downward API is **static**
* ConfigMaps are **shared**, Downward API is **Pod‑scoped**
* They solve **different problems** — never interchangeable

---

