# Create Deployment and Export YAML in Kubernetes

This document explains **how to create a Deployment and export its YAML as a reusable template**, why this approach is preferred, and common real-world patterns.

---

## 1. Why export Deployment YAML?

In Kubernetes, **YAML is the source of truth** for deployments.

Exporting YAML allows you to:

* Store manifests in Git
* Use CI/CD pipelines
* Apply changes declaratively
* Avoid manual YAML writing errors

---

## 2. Recommended Way: `--dry-run=client` + `-o yaml`

This is the **cleanest and safest method**.

### Command

```bash
kubectl create deployment nginx-deploy \
  --image=nginx \
  --replicas=3 \
  --dry-run=client \
  -o yaml > deployment.yaml
```

### What happens

* Nothing is created in the cluster
* Kubernetes prints the Deployment manifest
* Output is saved to `deployment.yaml`

---

## 3. Example Generated Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deploy
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - name: nginx
        image: nginx
```

This file can now be treated as a **deployment template**.

---

## 4. Apply the Generated YAML

```bash
kubectl apply -f deployment.yaml
```

Result:

* Deployment is created
* ReplicaSet is created
* Pods are created

---

## 5. Export YAML from an Existing Deployment (Not for templates)

```bash
kubectl get deployment nginx-deploy -o yaml > deployed.yaml
```

Why this is **not recommended for templates**:

* Contains cluster-specific fields
* Includes `status`, `resourceVersion`, `uid`

Use this only for **debugging or inspection**.

---

## 6. Clean-Up Checklist for Production Templates

When using exported YAML in Git:

* Remove `status:` section
* Remove `creationTimestamp`
* Remove `resourceVersion`, `uid`

---

## 7. Common Variations

### Add namespace

```bash
kubectl create deployment nginx \
  --image=nginx \
  -n web \
  --dry-run=client -o yaml > deployment.yaml
```

---

### Add container port

Edit YAML:

```yaml
ports:
- containerPort: 80
```

---

## 8. Best Practice Workflow (Real World)

1. Generate YAML using `dry-run`
2. Review and clean the file
3. Commit to Git
4. Deploy using `kubectl apply`

---

## 9. Interview One-Liner

> Use `kubectl create deployment --dry-run=client -o yaml` to generate a Deployment manifest without creating resources, then apply it declaratively using `kubectl apply`.

---

End of document.
