# Kubernetes Services

This document explains **Kubernetes Services** in a **slow, layered, and intuitive way**.
We build understanding in **four small steps**, without overload.

---

# 1️⃣ – What is a Service?

## Problem without a Service

* Pods have IPs
* Pod IPs change when Pods restart
* Multiple Pods exist for the same app

❌ You cannot safely connect to Pods directly.

---

## What a Service Actually Is

A **Service** is a **stable network endpoint** that represents a group of Pods.

It provides:

* One **stable IP** (virtual)
* One **stable DNS name**
* Automatic traffic distribution to Pods

Think like this:

```
Service = Fixed front door
Pods    = People behind the door (they may change)
```

---

## Very Simple Example

```
Frontend Pod  ───►  Service  ───►  Backend Pods
```

Frontend never talks to Pods directly.
It always talks to the **Service**.

---

# 2️⃣ – Types of Services

---

## 1. ClusterIP (Default)

### What it does

* Exposes Pods **inside the cluster only**

### Flow Diagram

```
Client Pod
   |
   | Service DNS / ClusterIP
   v
ClusterIP Service
   |
   v
Pod A   Pod B   Pod C
```

### When to use

* Internal microservice communication

---

## 2. NodePort

### What it does

* Exposes Service on **every Node IP** and a port

### Flow Diagram

```
External User
   |
   | NodeIP:30080
   v
Node
   |
   v
ClusterIP Service
   |
   v
Pods
```

### When to use

* Testing
* Learning
* Temporary access

---

## 3. LoadBalancer

### What it does

* Uses **cloud provider load balancer**

### Flow Diagram

```
Internet
   |
   v
Cloud Load Balancer
   |
   v
NodePort
   |
   v
ClusterIP Service
   |
   v
Pods
```

### When to use

* Production external traffic

---

## 4. Headless Service

### What it does

* No load balancing
* DNS returns **Pod IPs directly**

### Flow Diagram

```
Client Pod
   |
   | DNS Query
   v
Pod IP A   Pod IP B   Pod IP C
```

### When to use

* Databases
* StatefulSets (Kafka, Cassandra)

---

## 5. ExternalName

### What it does

* Points Service name to **external DNS**

### Flow Diagram

```
Client Pod
   |
   v
ExternalName Service
   |
   v
external.api.com
```

### When to use

* External dependencies

---

# 3️⃣ – Role of kube-proxy (Very Important)

## Simple Definition

**kube-proxy makes Services actually work.**

Service is only a definition.
kube-proxy turns it into **real networking rules**.

---

## What kube-proxy Does

* Runs on **every node**
* Watches Services and Pod IPs
* Creates **iptables rules**

❗ kube-proxy does NOT handle packets itself.

---

## Mental Model

```
kube-proxy = rule writer
iptables   = rule enforcer
Linux kernel = packet mover
```

---

# 4️⃣ – kube-proxy Traffic Flow

## Example Setup

```
Service IP: 10.96.0.10
Pods:
- 10.244.1.5
- 10.244.1.6
```

---

## Traffic Flow Diagram (ClusterIP)

```
Client Pod
   |
   | Request to 10.96.0.10
   v
Node Network Stack
   |
   | iptables (written by kube-proxy)
   v
Choose one Pod
   |
   | DNAT
   v
Pod IP (10.244.1.5 or 10.244.1.6)
```

---

## How Load Distribution Happens

iptables rules look like:

```
50% → Pod A
50% → Pod B
```

Each new connection is randomly matched.

---

## Return Traffic (Important)

For cross-node traffic:

* Source IP is rewritten (SNAT)
* Ensures response goes back correctly

```
Client ←── Pod
```

---

# FINAL SUMMARY (Lock This In)

* Service = stable entry point
* Pods = dynamic backends
* kube-proxy = writes routing rules
* iptables = executes routing
* Kernel = moves packets

---

## One-Line Interview Answer

> A Kubernetes Service provides a stable virtual IP and DNS name, and kube-proxy programs iptables rules so traffic sent to the Service is distributed to backend Pods by the Linux kernel.
