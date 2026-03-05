## Multiple Schedulers in Kubernetes

### What is a Scheduler?

The **Kubernetes Scheduler** is responsible for assigning Pods to Nodes. It watches for Pods without a `nodeName` and selects the most suitable Node based on:

* Resource requirements (CPU, memory)
* Node constraints (taints, tolerations, affinity)
* Scheduling policies and plugins

By default, Kubernetes runs **kube-scheduler**, but Kubernetes allows **multiple schedulers** to run in the same cluster.

---

## Why Multiple Schedulers?

Multiple schedulers are useful when **different workloads require different scheduling logic**.

### Common Use Cases

1. **Workload Isolation**

   * Run critical workloads with a custom scheduler
   * Prevent them from competing with general workloads

2. **Custom Scheduling Logic**

   * Special hardware (GPU, FPGA)
   * Latency-aware or cost-aware scheduling
   * Regulatory or compliance-based placement

3. **Multi-Tenant Clusters**

   * Each team/project uses its own scheduler
   * Independent scheduling rules

4. **Experimentation & Research**

   * Test new scheduling algorithms without touching kube-scheduler

5. **Batch vs Service Workloads**

   * Batch jobs prefer packing
   * Services prefer spreading

---

## How Multiple Schedulers Work

### Leader Election in Schedulers

When **multiple replicas of the same scheduler** are running, Kubernetes uses **leader election** to ensure that **only one scheduler is actively scheduling Pods at a time**.

This prevents:

* Double scheduling
* Race conditions
* Conflicting Pod-to-Node bindings

Leader election is **per scheduler name**, not cluster-wide.

---

### How Leader Election Works

* Scheduler instances participate in leader election
* Uses a coordination lock stored in **etcd**
* Implemented via the **Lease** object in the `kube-system` namespace

Only the **leader**:

* Watches unscheduled Pods
* Makes scheduling decisions
* Performs the bind operation

Followers stay idle but ready to take over.

---

### What Happens If Leader Fails

* Leader stops responding
* Lease expires
* One follower becomes the new leader
* Scheduling resumes automatically

No Pod resubmission is required.

---

## How Multiple Schedulers Work

* Each scheduler runs as a Pod (usually in `kube-system`)
* A Pod specifies which scheduler to use via:

```yaml
spec:
  schedulerName: <scheduler-name>
```

* If `schedulerName` is not specified → **default kube-scheduler** is used

---

## Creating a Custom Scheduler

### Step 1: Scheduler Configuration

Create a scheduler config file:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-custom-scheduler
```

---

### Step 2: Run the Scheduler as a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - name: scheduler
    image: k8s.gcr.io/kube-scheduler:v1.29.0
    command:
    - kube-scheduler
    - --config=/etc/kubernetes/scheduler-config.yaml
    volumeMounts:
    - name: config
      mountPath: /etc/kubernetes
  volumes:
  - name: config
    configMap:
      name: my-scheduler-config
```

---

### Step 3: ConfigMap for Scheduler

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-scheduler-config
  namespace: kube-system
data:
  scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    profiles:
    - schedulerName: my-custom-scheduler
```

---

## Creating a Pod Using the Custom Scheduler

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-custom-scheduled
spec:
  schedulerName: my-custom-scheduler
  containers:
  - name: nginx
    image: nginx
```

This Pod will **only be scheduled** by `my-custom-scheduler`.

---

## How Default and Custom Schedulers Coexist

| Pod Type   | schedulerName       | Scheduler Used   |
| ---------- | ------------------- | ---------------- |
| Normal Pod | not set             | kube-scheduler   |
| Custom Pod | my-custom-scheduler | Custom Scheduler |

Schedulers **ignore Pods** that don’t match their `schedulerName`.

---

## Failure Behavior

* If a custom scheduler is **down**:

  * Pods requesting it stay in `Pending`
* Default scheduler is unaffected

---

## Real-World Example

* kube-scheduler → microservices
* custom-scheduler → ML workloads on GPU nodes

```yaml
spec:
  schedulerName: gpu-scheduler
  nodeSelector:
    accelerator: nvidia
```

---

## Key Points to Remember

* Kubernetes supports **multiple active schedulers**
* Scheduler selection is **per Pod**
* Custom schedulers run as normal Pods
* Useful for isolation, customization, and experimentation

---

## When NOT to Use Multiple Schedulers

* Simple clusters
* No special scheduling requirements
* Added operational complexity is not justified
