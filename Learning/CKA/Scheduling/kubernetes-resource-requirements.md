# Kubernetes Resource Requirements

## 1. The simple idea

Kubernetes runs containers on worker nodes. Every node has a limited amount of:

- **CPU**: processing power
- **Memory**: working space, or RAM

For each container, you can define:

- **Request**: the minimum amount the container needs and the amount Kubernetes uses when choosing a node.
- **Limit**: the maximum amount the container is allowed to use.

A simple analogy:

> A restaurant has 10 tables. A reservation is a **request**. The maximum number of guests allowed at that table is the **limit**.

## 2. Basic example

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-app
          image: nginx:1.27
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
````

This means:

- Kubernetes reserves **250 millicores of CPU** and **256 MiB of memory** for each Pod replica.
- The container may use up to **500 millicores of CPU** and **512 MiB of memory**.
- With 2 replicas, the scheduler needs at least **500m CPU** and **512Mi memory** available across a suitable node or nodes.

### CPU units

- `1` CPU = one full logical CPU or core
- `500m` = 500 millicores = half a CPU
- `100m` = one tenth of a CPU

### Memory units

- `Mi` means mebibytes, commonly used for RAM
- `Gi` means gibibytes
- Example: `512Mi`, `2Gi`

## 3. Request versus limit

| Setting | Meaning | Main effect |
|---|---|---|
| CPU request | Guaranteed scheduling amount | Helps Kubernetes select a node |
| CPU limit | Maximum CPU usage | CPU is usually throttled above this value |
| Memory request | Expected or reserved memory | Helps Kubernetes select a node |
| Memory limit | Maximum memory usage | Container can be terminated if it exceeds it |

The most important difference is this:

> CPU is compressible. Memory is not.

If a container exceeds its CPU limit, it is normally slowed down. If it exceeds its memory limit, it can be killed with an **OOMKilled** status.

## 4. Real-world example: online shop

Imagine an online shop with three workloads:

1. **Frontend**: receives customer requests
2. **Checkout API**: processes payments and orders
3. **Background worker**: sends emails and generates reports

A reasonable starting point could be:

````yaml
resources:
  requests:
    cpu: "200m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "512Mi"
````

For the checkout API, you might choose a higher request because it is important for customer transactions:

````yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "2"
    memory: "1Gi"
````

For a low-priority report worker, you might use a smaller request:

````yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
````

The request protects the checkout API during scheduling. The limit prevents one broken process from consuming all available node resources.

## 5. How scheduling works

````text
Developer creates Deployment
          |
          v
Pod template contains requests and limits
          |
          v
API Server stores the Pod specification
          |
          v
Scheduler checks nodes
  - Is enough requested CPU available?
  - Is enough requested memory available?
  - Do labels, taints, affinity, and policies match?
          |
     +----+----+
     |         |
   Yes        No
     |         |
     v         v
Pod assigned   Pod remains Pending
 to a node     and Kubernetes reports why
     |
     v
Kubelet starts the container
     |
     v
Linux cgroups enforce CPU and memory limits
````

Important: the scheduler primarily uses **requests**, not current real-time usage. A node can look lightly used but still reject a Pod if existing requests already consume its allocatable capacity.

## 6. What happens inside the node

````text
Pod starts
   |
   v
Kubelet asks the container runtime to run it
   |
   v
Runtime creates Linux cgroups
   |
   +--> CPU request influences scheduling weight/shares
   |
   +--> CPU limit sets a CPU quota
   |
   +--> Memory limit sets a memory ceiling
   |
   v
The Linux kernel tracks and controls usage
   |
   +--> CPU over limit: throttling, slower application
   |
   +--> Memory over limit: OOM kill may occur
````

The exact implementation depends on the operating system, container runtime, and cgroup version, but the core idea is that Kubernetes translates resource settings into Linux kernel controls.

## 7. Kubernetes QoS classes

Kubernetes assigns every Pod a Quality of Service class.

### Guaranteed

Every container has CPU and memory requests, and each request equals its limit.

````yaml
requests:
  cpu: "500m"
  memory: "512Mi"
limits:
  cpu: "500m"
  memory: "512Mi"
````

These Pods are the last candidates for eviction during node memory pressure.

### Burstable

The Pod has resource settings, but at least one request is lower than its limit.

````yaml
requests:
  cpu: "250m"
  memory: "256Mi"
limits:
  cpu: "1"
  memory: "512Mi"
````

This is a common choice for web applications.

### BestEffort

No container in the Pod has resource requests or limits.

BestEffort Pods are the first candidates for eviction during resource pressure. In production, avoid this class unless the workload is intentionally low priority.

## 8. Why resource settings matter

Without requests and limits:

- A noisy container can consume most of a node's CPU.
- A memory leak can cause other applications to be killed.
- The scheduler cannot make good placement decisions.
- Autoscaling signals can become misleading.
- Production behavior becomes unpredictable.

With poorly chosen values:

- Requests that are too high waste cluster capacity and may leave Pods Pending.
- CPU limits that are too low can cause latency and throttling.
- Memory limits that are too low can cause OOMKilled restarts.
- Memory limits that are too high can allow one Pod to destabilize a node if requests are also inaccurate.

## 9. Requests, limits, and autoscaling

### Horizontal Pod Autoscaler, HPA

HPA often uses CPU or memory utilization relative to the **request**.

For example, if a Pod requests `500m` CPU and currently uses `250m`, its CPU utilization is approximately 50%.

If requests are missing or unrealistic, HPA may not scale as expected.

### Vertical Pod Autoscaler, VPA

VPA observes usage and recommends or updates requests and limits. It is useful for workloads whose resource needs are difficult to estimate, but it may restart Pods when changing their resources.

### Cluster Autoscaler

Cluster Autoscaler adds nodes when Pods cannot be scheduled because their **requests** do not fit. It does not add nodes simply because current CPU usage is high.

## 10. How to choose values

Start with measurements, not guesses:

1. Deploy with a conservative request and safe memory limit.
2. Observe normal and peak usage.
3. Test realistic traffic.
4. Set the request near normal sustained usage, with room for variation.
5. Set the memory limit above normal peak, but below an unsafe runaway value.
6. Revisit the values after application changes.

Useful commands:

````bash
kubectl top pod
kubectl top node
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o yaml
kubectl get events --sort-by=.lastTimestamp
````

Look for:

- `Pending` Pods with scheduling events
- `OOMKilled` container states
- CPU throttling metrics
- Restart counts
- Node memory pressure
- Difference between requested and actual usage

## 11. Intermediate concepts

### Pod resources are the sum of container resources

If a Pod has two containers:

- Container A requests `200m` CPU
- Container B requests `300m` CPU

The Pod's CPU request is `500m`.

### Init containers

Init container requests can affect scheduling. Kubernetes considers the largest relevant init-container requirement along with the total of regular containers. A short-lived initialization step can therefore influence where the Pod is placed.

### Ephemeral storage

CPU and memory are not the only resources. Containers also use local disk space for writable layers, logs, and temporary files.

````yaml
resources:
  requests:
    ephemeral-storage: "1Gi"
  limits:
    ephemeral-storage: "2Gi"
````

### Extended resources

Special hardware such as GPUs can be advertised as a node resource and requested by a Pod.

````yaml
resources:
  limits:
    nvidia.com/gpu: 1
````

### Namespace quotas

A `ResourceQuota` can limit the total CPU and memory requests or limits allowed in a namespace. This prevents one team from consuming the whole cluster.

### LimitRange

A `LimitRange` can provide default requests and limits or enforce minimum and maximum values when a Pod is created.

## 12. Advanced concepts and common traps

### Overcommitment

The total of Pod limits can be greater than node capacity. This is called overcommitment. It can improve utilization, but it increases the risk of contention and eviction.

### Allocatable versus capacity

A node's total capacity is not fully available to Pods. Kubernetes reserves some resources for the operating system and Kubernetes system components. Scheduling uses the node's **allocatable** resources.

### Memory pressure and eviction

When a node runs low on memory, the kubelet may evict Pods. QoS class, priority, and usage compared with requests influence eviction decisions. A high-priority Pod is not automatically immune to eviction.

### CPU throttling

A CPU limit can protect a node, but a limit that is too low may cause periodic throttling, increased response time, or poor performance even when the node has spare CPU. For latency-sensitive workloads, carefully evaluate whether a CPU limit is appropriate.

### OOMKilled versus eviction

- **OOMKilled**: the container exceeded a memory boundary and was killed by the kernel or runtime.
- **Evicted**: the kubelet removed the Pod because the node was under pressure.

They are related to memory, but they are not the same event.

### Resource units are not performance guarantees

`500m` means a CPU quantity, not a fixed application speed. Performance varies with CPU model, virtualization, runtime, workload type, and contention.

## 13. Basic interview and troubleshooting questions

### Basic

**Q: What is a resource request?**  
A: The amount Kubernetes uses for scheduling and capacity planning.

**Q: What is a resource limit?**  
A: The maximum amount a container may use for that resource.

**Q: What does `500m` mean?**  
A: Half of one CPU.

**Q: What happens when CPU exceeds its limit?**  
A: The container is generally throttled rather than killed.

**Q: What happens when memory exceeds its limit?**  
A: The container may be killed and reported as `OOMKilled`.

### Intermediate

**Q: Why is a Pod Pending even though a node appears idle?**  
A: Its requests may not fit the node's allocatable resources, or another scheduling rule may block placement.

**Q: Does the scheduler use actual current usage?**  
A: Primarily no. It uses declared requests.

**Q: What is the difference between Burstable and Guaranteed?**  
A: Guaranteed Pods have equal CPU and memory requests and limits for every container. Burstable Pods have resource settings that do not all match.

**Q: Why can HPA behave badly with incorrect requests?**  
A: Utilization is commonly calculated as usage relative to the request, so an unrealistic request changes the percentage.

**Q: How do you investigate OOMKilled?**  
A: Check `kubectl describe pod`, container termination state, restart history, application logs, memory metrics, and whether the limit is below the real peak requirement.

### Advanced

**Q: Can the sum of limits exceed node capacity?**  
A: Yes. Kubernetes permits overcommitment, but it increases contention and eviction risk.

**Q: Why might a CPU limit hurt a low-latency service?**  
A: CPU quota enforcement can throttle bursts and create latency, even when unused CPU exists elsewhere on the node.

**Q: How are multiple containers in one Pod scheduled?**  
A: Their resource requests are combined for scheduling. The Pod is placed as one unit.

**Q: What is the purpose of ResourceQuota?**  
A: To control aggregate resource consumption in a namespace.

**Q: What is the difference between request-based scheduling and runtime enforcement?**  
A: The scheduler uses requests to choose a node. The kubelet and Linux cgroups enforce limits after the Pod starts.

**Q: Why can a node experience memory pressure when requests appear to fit?**  
A: Actual usage can exceed requests, system processes also need memory, and limits can be overcommitted. Requests are planning values, not a hard runtime reservation of physical RAM.

## 14. One-minute summary

````text
Request = “Please reserve this much for scheduling.”
Limit   = “Do not let me use more than this.”

CPU over limit    -> usually throttled
Memory over limit -> may be OOMKilled
No settings       -> BestEffort and risky for production
Scheduler         -> checks requests
Kubelet/cgroups   -> enforce limits
Good values       -> come from measurements and load testing
````

A practical production baseline is to define requests and limits for every important container, monitor actual usage, and adjust the values as the application and traffic change.
