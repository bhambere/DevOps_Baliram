# Kubernetes Scheduling: Node Selector vs Node Affinity vs Taints and Tolerations

Kubernetes uses these features to decide **which nodes can run a Pod**.

A simple way to remember them:

| Feature | Applied to | Main purpose | Easy meaning |
|---|---|---|---|
| `nodeSelector` | Pod | Choose nodes with specific labels | “Run this Pod only on nodes matching these labels.” |
| `nodeAffinity` | Pod | Choose nodes using more flexible rules | “Prefer or require nodes matching these advanced conditions.” |
| `taints` | Node | Repel ordinary Pods | “Do not run Pods here unless they are allowed.” |
| `tolerations` | Pod | Allow Pods onto tainted nodes | “This Pod is allowed to run on that restricted node.” |

## 1. `nodeSelector`

`nodeSelector` is the simplest option. It tells Kubernetes to schedule a Pod only on nodes having specific labels.

### Example

Label a node:

````bash
kubectl label node worker-1 workload=production
````

Pod configuration:

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: production-app
spec:
  nodeSelector:
    workload: production
  containers:
    - name: app
      image: nginx
````

This Pod can run only on nodes with `workload=production`.

### Real-world example

You have separate nodes for production and development workloads:

- Production nodes: `workload=production`
- Development nodes: `workload=development`

You can ensure that production applications do not accidentally run on development nodes.

### Limitations

`nodeSelector` supports only simple exact matches. It cannot express rules such as:

- Use either `region=us-east` or `region=us-west`
- Prefer one type of node, but allow another
- Require a particular label, but only if another label also exists

For those cases, use node affinity.

---

## 2. `nodeAffinity`

Node affinity is a more powerful and flexible version of `nodeSelector`.

It supports required rules, preferred rules, and operators such as `In`, `NotIn`, `Exists`, and `Gt`.

### Required node affinity

The Pod **must** run on a matching node.

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-memory-app
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: instance-type
                operator: In
                values:
                  - memory-optimized
  containers:
    - name: app
      image: nginx
````

This Pod will not be scheduled unless a node has `instance-type=memory-optimized`.

The phrase `IgnoredDuringExecution` means that if the node label changes after the Pod is already running, Kubernetes does not automatically evict the Pod.

### Preferred node affinity

The scheduler **prefers** matching nodes, but can use another node if necessary.

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 80
          preference:
            matchExpressions:
              - key: zone
                operator: In
                values:
                  - us-east-1a
  containers:
    - name: app
      image: nginx
````

The scheduler tries to place the Pod in `us-east-1a`, but it can schedule the Pod elsewhere if that zone has no available capacity.

### Real-world example

Suppose you run a machine-learning workload:

- GPU nodes have `accelerator=gpu`
- CPU nodes do not have that label

You could require GPU nodes using `requiredDuringSchedulingIgnoredDuringExecution`.

For a normal web application, you might prefer SSD-backed nodes but allow standard nodes as a fallback using `preferredDuringSchedulingIgnoredDuringExecution`.

---

## 3. Taints

A taint is applied to a **node**. It prevents Pods from being scheduled there unless they have a matching toleration.

### Add a taint

````bash
kubectl taint nodes gpu-node-1 accelerator=gpu:NoSchedule
````

The taint means:

> Do not schedule ordinary Pods on this node.

The taint consists of `key=value:effect`. For example, `accelerator=gpu:NoSchedule`.

### Taint effects

| Effect | Behavior |
|---|---|
| `NoSchedule` | New Pods are not scheduled unless they tolerate the taint |
| `PreferNoSchedule` | Kubernetes tries to avoid placing Pods there, but may do so |
| `NoExecute` | Existing non-tolerating Pods are evicted, and new ones are rejected |

### Real-world example

You have expensive GPU nodes that should be used only by machine-learning workloads.

Without a taint, regular applications might consume the GPU nodes. Add this taint:

````bash
kubectl taint nodes gpu-node-1 accelerator=gpu:NoSchedule
````

Now ordinary Pods avoid the node.

However, a taint alone does not identify which Pods should use the node. You normally combine it with a toleration and node affinity.

---

## 4. Tolerations

A toleration is added to a **Pod**. It allows the Pod to run on a node with a matching taint.

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: ml-training
spec:
  tolerations:
    - key: accelerator
      operator: Equal
      value: gpu
      effect: NoSchedule
  containers:
    - name: trainer
      image: ml-training:latest
````

This Pod tolerates `accelerator=gpu:NoSchedule`, so it is allowed to run on the tainted GPU node.

### Important point

A toleration does **not** force a Pod to run on the tainted node. It only says:

> This Pod is allowed to run there.

The Pod might still be scheduled on another node unless you also add node affinity or `nodeSelector`.

---

# The most important difference

## `nodeSelector` and `nodeAffinity` select nodes

They are configured on the Pod and select nodes based on labels.

```text
Pod says: “I want a node with these labels.”
```

## Taints repel Pods from nodes

They are configured on the node.

```text
Node says: “Do not run Pods here unless they are allowed.”
```

## Tolerations permit Pods onto tainted nodes

They are configured on the Pod.

```text
Pod says: “I am allowed to run on this restricted node.”
```

---

# Combined real-world example: GPU nodes

Assume you have a GPU node:

````bash
kubectl label node gpu-node-1 accelerator=gpu
kubectl taint node gpu-node-1 accelerator=gpu:NoSchedule
````

The label attracts the correct workloads, while the taint keeps unrelated workloads away.

A GPU workload could use both toleration and affinity:

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-training
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ml-training
  template:
    metadata:
      labels:
        app: ml-training
    spec:
      tolerations:
        - key: accelerator
          operator: Equal
          value: gpu
          effect: NoSchedule

      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: accelerator
                    operator: In
                    values:
                      - gpu

      containers:
        - name: trainer
          image: ml-training:latest
````

Here:

- The **taint** prevents normal Pods from using the GPU node.
- The **toleration** allows this Pod to enter.
- The **node affinity** ensures this Pod actually chooses a GPU node.

This is a common production pattern.

---

# Another real-world example: dedicated database nodes

Suppose database nodes should run only database workloads.

Label and taint the nodes:

````bash
kubectl label node db-node-1 workload=database
kubectl taint node db-node-1 workload=database:NoSchedule
````

Database Pod:

````yaml
spec:
  tolerations:
    - key: workload
      operator: Equal
      value: database
      effect: NoSchedule

  nodeSelector:
    workload: database
````

The result is:

- Non-database Pods are blocked by the taint.
- Database Pods are allowed by the toleration.
- The `nodeSelector` directs database Pods to the database node.

---

# Comparison by use case

| Requirement | Recommended feature |
|---|---|
| Run a Pod only on nodes with `disk=ssd` | `nodeSelector` |
| Use complex matching rules | `nodeAffinity` |
| Prefer one node type but allow fallback | Preferred node affinity |
| Keep general workloads away from special nodes | Taints |
| Allow a specific Pod onto a restricted node | Tolerations |
| Reserve GPU nodes for GPU workloads | Taint plus toleration plus affinity |
| Reserve database nodes for database workloads | Taint plus toleration plus selector or affinity |
| Evict Pods when a node becomes unhealthy | `NoExecute` taint |

---

# Simple analogy

Imagine a parking lot:

- **Node labels** are signs on parking spaces, such as “Electric Vehicle” or “VIP.”
- **`nodeSelector`** says, “Park me only in a space marked VIP.”
- **Node affinity** says, “Prefer VIP spaces, but use premium spaces if needed.”
- **A taint** says, “This space is restricted. Do not park here.”
- **A toleration** is a permit that allows a vehicle to park in the restricted space.

Having a permit does not automatically direct the vehicle to that space. The vehicle still needs a selection rule if it specifically wants that location.

---

# Quick memory trick

````text
nodeSelector = simple selection
nodeAffinity = advanced selection
taint = node restriction
toleration = Pod permission
````

In production, the usual pattern for dedicated nodes is:

````text
Node label + Node taint
        +
Pod node affinity or nodeSelector + Pod toleration
````

That combination both directs the intended workload to the node and prevents unrelated workloads from using it.
