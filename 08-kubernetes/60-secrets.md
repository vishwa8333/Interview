# 🔐 Kubernetes Secrets – Explained with Examples

Kubernetes **Secrets** are used to store **sensitive data** such as passwords, tokens, API keys, and certificates.

> ⚠️ Why Secrets?

* Avoid hard‑coding credentials in Pod YAML 🧾
* Keep sensitive data separate from application code 🔒
* Control access using RBAC 👮

---

## 🧠 What is a Secret?

A **Secret** is an object that stores data in **base64‑encoded** form.

Typical use cases:

* Database username & password 🗄️
* TLS certificates 🔐
* Docker registry credentials 📦

---

## 📦 Types of Kubernetes Secrets

| Type                             | Purpose                      |
| -------------------------------- | ---------------------------- |
| `Opaque`                         | Generic key‑value secrets 🔑 |
| `kubernetes.io/tls`              | TLS certs & keys 🔐          |
| `kubernetes.io/dockerconfigjson` | Image pull secrets 🐳        |
| `service-account-token`          | API access tokens 🤖         |

---

## ✍️ Example 1: Creating a Secret (Opaque)

### Step 1️⃣: Create Secret YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQxMjM=
```

> 🔍 Note:

* `admin` → `YWRtaW4=`
* `password123` → `cGFzc3dvcmQxMjM=`

Encode using:

```bash
echo -n admin | base64
```

---

## 🚀 How Secrets Are Injected into Pods

Secrets can be injected in **two main ways**:

1️⃣ As **Environment Variables** 🌱
2️⃣ As **Mounted Volumes** 📂

---

## 🌱 Method 1: Inject Secret as Environment Variables

### Pod YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

### 🔄 What Happens Internally?

* kubelet fetches secret from API server 📡
* Decodes base64 🔓
* Injects values as environment variables 🌱

Inside container:

```bash
echo $DB_USERNAME
echo $DB_PASSWORD
```

---

## 📂 Method 2: Inject Secret as a Volume (Recommended ✅)

### Pod YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod-volume
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/secrets"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

### 📁 Inside the Container

```bash
/etc/secrets/username
/etc/secrets/password
```

Each key becomes a **file** 📄 containing the decoded value.

---

## 🔄 Secret Injection – End‑to‑End Flow 🧭

1️⃣ User creates Secret → stored in etcd 🗄️
2️⃣ Pod definition references Secret 🧾
3️⃣ API Server validates request ✅
4️⃣ kubelet pulls Secret for the Pod 🤝
5️⃣ Secret injected as env vars or files 🎯
6️⃣ Application consumes secret securely 🔐

---

## ⚠️ Important Security Notes

* Secrets are **base64 encoded, not encrypted** ❗
* Enable **encryption at rest** for etcd 🔐
* Avoid env vars for highly sensitive secrets 🚫
* Use RBAC to restrict secret access 👮

---

## ⭐ Best Practices

✅ Prefer **volume mount** over env vars
✅ Use **external secret managers** (Vault, AWS Secrets Manager) 🌩️
✅ Rotate secrets regularly 🔄
✅ Never commit secrets to GitHub 🚫🐙

---

## 🧩 Quick One‑Line Summary

> **Secrets = secure config + controlled access + safer Pods 🔐🚀**

---

🎉 You now understand **what Secrets are, how they’re created, and how they reach Pods** in Kubernetes!
