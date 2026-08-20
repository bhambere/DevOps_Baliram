# Kubernetes Scheduler: Easy Explanation

## 1. What is the Kubernetes Scheduler?

The **Kubernetes Scheduler** decides **which worker node should run a new Pod**.

Think of it as a **dispatcher assigning a delivery to the best driver**. It checks each driver's capacity, location, and restrictions before assigning the delivery.

> **The Scheduler decides where a Pod runs. It does not start the containers.**

## 2. Real-world example

Imagine three delivery drivers:

- **Driver A** has no space left.
- **Driver B** has enough space and is allowed to deliver to the destination.
- **Driver C** has space but is already overloaded with other deliveries.

The dispatcher filters out unsuitable drivers, compares the remaining drivers, and assigns the delivery to Driver B.

Kubernetes works the same way:

- **Pod** = delivery
- **Worker node** = driver
- **Scheduler** = dispatcher
- **CPU, memory, labels, taints, and policies** = assignment rules

## 3. Simple scheduling flow

```text
User creates a Pod or Deployment
              |
              v
API Server stores the Pod
              |
              v
Pod has no nodeName yet
              |
              v
Scheduler finds the unscheduled Pod
              |
              v
Filter: remove nodes that cannot run it
              |
              v
Score: compare the valid nodes
              |
              v
Select the best node
              |
              v
Bind the Pod to that node through the API Server
              |
              v
Kubelet on that node notices the Pod
              |
              v
Container runtime starts the containers
```

## 4. How it works, step by step

### Step 1: A Pod is created

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
        requests:
          cpu: "500m"
          memory: "512Mi"
```

Initially, the Pod has no assigned node.

### Step 2: The Scheduler watches the API Server

It looks for Pods where `spec.nodeName` is empty. These are called **unscheduled Pods**.

### Step 3: Filtering

The Scheduler rejects nodes that do not satisfy the Pod's requirements, for example:

- Not enough requested CPU or memory
- Required node label is missing
- Pod does not tolerate a node taint
- Volume or storage requirement is not satisfied
- Affinity or anti-affinity rule does not match
- Node is unhealthy or unavailable

### Step 4: Scoring

Several nodes may pass filtering. The Scheduler scores them and selects the best one. It may prefer a node with more available resources, fewer Pods, better label matches, or better distribution across zones.

### Step 5: Binding

The Scheduler writes the selected node into the Pod assignment through the API Server. The Pod now has a `nodeName`.

### Step 6: Execution

The kubelet on the selected node:

1. Detects the assigned Pod.
2. Pulls the container image if necessary.
3. Creates and starts the containers.
4. Reports the Pod status back to the API Server.

## 5. Scheduler versus kubelet

| Component | Job |
|---|---|
| Scheduler | Decides which node should run the Pod |
| Kubelet | Makes the Pod run on that node |
| API Server | Stores and coordinates cluster state |
| Controller | Creates and maintains Pods |
| Container runtime | Starts and manages containers |

> **Scheduler chooses. Kubelet executes.**

## 6. Important scheduling concepts

### Resource requests and limits

```yaml
resources:
  requests:
    cpu: "1"
    memory: "1Gi"
  limits:
    cpu: "2"
    memory: "2Gi"
```

- **Request**: Amount used by the Scheduler when deciding whether the Pod fits.
- **Limit**: Maximum amount the container may use at runtime.

The Scheduler mainly uses requests, not real-time CPU usage.

### Labels and `nodeSelector`

Labels describe nodes. A Pod can request a matching label:

```yaml
nodeSelector:
  disktype: ssd
```

This Pod can run only on nodes labeled `disktype=ssd`.

### Taints and tolerations

A **taint** says, “Do not place Pods here unless they tolerate this rule.”

A **toleration** allows a Pod to run on a tainted node. It does not force the Pod to use that node.

### Affinity and anti-affinity

- **Affinity**: Prefer or require placement near certain nodes or Pods.
- **Anti-affinity**: Prefer or require keeping Pods apart.

These rules are useful for spreading replicas across nodes or availability zones.

### Priority and preemption

If a high-priority Pod cannot fit anywhere, Kubernetes may remove lower-priority Pods to make room. This is called **preemption**.

## 7. What if scheduling fails?

The Pod remains in the `Pending` state.

Use:

```bash
kubectl get pods
kubectl describe pod <pod-name>
```

Look at the **Events** section. Common messages include:

```text
0/3 nodes are available: 3 Insufficient memory.
```

Other causes include an untolerated taint, a node selector mismatch, an unsatisfied affinity rule, or a volume restriction. The Scheduler keeps trying when conditions change, such as when a node is added or resources become available.

## 8. Under the hood, briefly

The Scheduler uses a scheduling loop and a plugin framework:

```text
Scheduling queue
      |
      v
Pick an unscheduled Pod
      |
      v
Pre-filter and filter plugins
      |
      v
Score plugins
      |
      v
Choose the highest-scoring node
      |
      v
Reserve and permit checks
      |
      v
Bind Pod to the node
```

Important extension points are:

- **QueueSort**: Orders Pods waiting in the queue.
- **PreFilter**: Prepares information before node checks.
- **Filter**: Rejects unsuitable nodes.
- **Score**: Ranks suitable nodes.
- **Reserve**: Temporarily reserves the node during scheduling.
- **Permit**: Allows a plugin to approve or delay binding.
- **Bind**: Assigns the Pod to the selected node.

The Scheduler makes the placement decision. It does not directly start containers or control the operating system's CPU scheduler.

## 9. Basic questions and answers

### What is the main job of the Scheduler?

To select a suitable worker node for an unscheduled Pod.

### Does the Scheduler start containers?

No. The kubelet starts containers after assignment.

### Where does the Scheduler get information?

From the API Server, including Pods, nodes, resources, labels, taints, volumes, and scheduling rules.

### What does `Pending` mean?

The Pod is not running successfully yet. It may be waiting for scheduling or another dependency.

### What is a worker node?

A physical or virtual machine that runs Pods.

### What is `nodeName`?

The field identifying the node assigned to the Pod. The Scheduler normally sets it.

## 10. Intermediate questions and answers

### Filtering versus scoring?

**Filtering** asks: “Can this Pod run on this node?”

**Scoring** asks: “Which valid node is the best choice?”

### Does a toleration force placement on a tainted node?

No. It only permits placement there. Other nodes may still score higher.

### How can I keep replicas on different nodes?

Use topology spread constraints or Pod anti-affinity.

### What is preemption?

Evicting lower-priority Pods so that a higher-priority Pod can be scheduled.

### Can Kubernetes use more than one scheduler?

Yes. A Pod can select a scheduler using `schedulerName`.

### Is a Deployment the Scheduler?

No. A Deployment creates and manages Pods. The Scheduler chooses nodes for those Pods.

## 11. Advanced questions and answers

### Does the Scheduler use real-time CPU usage?

Usually, it relies mainly on declared resource requests and node status. Accurate requests are important.

### What is a scheduling profile?

A configuration that controls which Scheduler plugins and behaviors are used for a group of Pods.

### What is `schedulerName`?

The name of the scheduler responsible for a Pod. The usual default is `default-scheduler`.

### What is a custom scheduler?

A separate scheduler that applies special placement logic when normal rules are not enough.

### Can a Pod fail after it is scheduled?

Yes. Scheduling only selects a node. Image pull failures, volume errors, crashes, node failures, or application configuration can still prevent the Pod from running.

## 12. Useful troubleshooting commands

```bash
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl get nodes
kubectl describe node <node-name>
kubectl get events --sort-by=.lastTimestamp
```

For a `Pending` Pod, start with:

```bash
kubectl describe pod <pod-name>
```

Then read the Events section for the Scheduler's reason.

## Final summary

```text
Pod created
   -> API Server stores it
   -> Scheduler finds it
   -> Bad nodes are filtered out
   -> Good nodes are scored
   -> Best node is selected
   -> Pod is bound to that node
   -> Kubelet starts the containers
```

> **Kubernetes Scheduler = the decision-maker that places Pods on suitable worker nodes.**
