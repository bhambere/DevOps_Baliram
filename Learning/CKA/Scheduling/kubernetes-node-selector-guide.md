# Kubernetes `nodeSelector`, Made Easy

## 1. What is `nodeSelector`?

`nodeSelector` tells Kubernetes:

> "Run this Pod only on a node that has these labels."

It is the simplest way to place a Pod on a specific type of worker node.

### Simple example

Suppose your cluster has two types of nodes:

- General-purpose nodes
- SSD nodes

You label the SSD node:

````bash
kubectl label node worker-ssd-1 disk=ssd
````

Then add this to the Pod:

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: fast-app
spec:
  nodeSelector:
    disk: ssd
  containers:
    - name: app
      image: nginx
````

Kubernetes will schedule `fast-app` only on a node with this exact label:

````text
disk=ssd
````

---

## 2. Real-world example

Imagine a company running an online shopping application:

| Node type | Purpose | Label |
|---|---|---|
| Standard node | Web servers | `workload=web` |
| Memory-optimized node | Product search and caching | `workload=memory` |
| GPU node | Machine learning recommendations | `workload=gpu` |

Label the nodes:

````bash
kubectl label node node-web-1 workload=web
kubectl label node node-memory-1 workload=memory
kubectl label node node-gpu-1 workload=gpu
````

Schedule the recommendation service on a GPU node:

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recommendation-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: recommendation-service
  template:
    metadata:
      labels:
        app: recommendation-service
    spec:
      nodeSelector:
        workload: gpu
      containers:
        - name: recommendation-service
          image: example/recommendation-service:v1
````

The Deployment creates Pods, and every Pod is restricted to nodes labeled `workload=gpu`.

---

## 3. Flow chart

````mermaid
flowchart TD
    A[Create or update a Pod] --> B{Does the Pod have nodeSelector?}
    B -- No --> C[Consider all suitable nodes]
    B -- Yes --> D[Read required labels]
    D --> E{Does a node contain every exact label?}
    E -- No --> F[Node is rejected]
    E -- Yes --> G[Check resources and other rules]
    C --> G
    G --> H{Is at least one node suitable?}
    H -- No --> I[Pod remains Pending]
    H -- Yes --> J[Scheduler selects a node]
    J --> K[ kubelet on that node starts the Pod ]
````

### In one sentence

`nodeSelector` filters nodes first, then Kubernetes checks CPU, memory, taints, affinity, and other scheduling rules before assigning the Pod to a node.

---

## 4. Important syntax

````yaml
spec:
  nodeSelector:
    key: value
````

Multiple labels mean **AND**, not OR:

````yaml
spec:
  nodeSelector:
    environment: production
    disk: ssd
````

The node must have both labels:

````text
environment=production
disk=ssd
````

`nodeSelector` supports exact key-value matching only. It does not support expressions such as `In`, `NotIn`, or `Exists`.

---

## 5. Useful commands

List nodes and their labels:

````bash
kubectl get nodes --show-labels
````

Describe one node:

````bash
kubectl describe node worker-ssd-1
````

Add a label:

````bash
kubectl label node worker-ssd-1 disk=ssd
````

Change an existing label:

````bash
kubectl label node worker-ssd-1 disk=nvme --overwrite
````

Remove a label:

````bash
kubectl label node worker-ssd-1 disk-
````

Check the Pod and scheduling events:

````bash
kubectl get pod fast-app -o wide
kubectl describe pod fast-app
````

---

## 6. Basic questions

### Q1. Is `nodeSelector` mandatory?

No. If it is not specified, Kubernetes can consider any node that satisfies the other scheduling rules.

### Q2. Does the label go on the Pod or the node?

The label used by `nodeSelector` must be on the **node**. The selector is written in the Pod specification.

### Q3. What happens if no node has the required label?

The Pod stays in `Pending` state. It is not automatically moved to another kind of node.

### Q4. Is matching case-sensitive?

Yes. These are different values:

````text
SSD
ssd
````

### Q5. Can I use multiple labels?

Yes. All labels must match.

### Q6. Does `nodeSelector` move an already-running Pod?

No. It affects scheduling. Changing the selector in a Deployment normally causes new Pods to be created, but an existing Pod is not live-migrated.

### Q7. Can I use `nodeSelector` in a Deployment?

Yes. Put it under:

````yaml
spec:
  template:
    spec:
      nodeSelector:
````

It must be part of the Pod template, not directly beside `replicas`.

### Q8. Can users change node labels?

Only users with sufficient Kubernetes permissions should be allowed to label nodes. Node labels can influence where workloads run, so they should be controlled carefully.

---

## 7. Intermediate questions

### Q9. What is the difference between `nodeSelector` and `nodeAffinity`?

| Feature | `nodeSelector` | `nodeAffinity` |
|---|---|---|
| Matching | Exact labels | Rules and expressions |
| OR conditions | No | Yes |
| Required or preferred placement | Required only | Required or preferred |
| Complexity | Very simple | More flexible |

Example of a more flexible rule using node affinity:

````yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disk
              operator: In
              values:
                - ssd
                - nvme
````

### Q10. Does `nodeSelector` override resource requests?

No. A node must match the selector **and** have enough CPU and memory, among other requirements.

### Q11. What if the selected nodes are full?

The Pod remains `Pending` until a matching node has enough available resources, or another suitable node becomes available.

### Q12. What if a matching node has a taint?

The Pod also needs a matching toleration. `nodeSelector` does not bypass taints.

Example:

````yaml
tolerations:
  - key: dedicated
    operator: Equal
    value: gpu
    effect: NoSchedule
````

### Q13. Can `nodeSelector` guarantee that only my application uses a node?

No. It only asks for placement on matching nodes. Other Pods can also run there unless you use taints and tolerations, resource controls, or other isolation methods.

### Q14. What labels should I use?

Use stable, meaningful labels such as:

````text
workload=database
disk=ssd
tier=backend
zone=us-east-1a
````

Avoid labels that change frequently or depend on temporary node names.

---

## 8. Advanced questions

### Q15. What does `nodeSelector` match internally?

Conceptually, Kubernetes evaluates each selector entry as an exact requirement:

````text
node.labels[key] == requested.value
````

For multiple entries, all comparisons must be true.

### Q16. What does the scheduler do under the hood?

1. The API server stores the Pod specification.
2. The scheduler notices that the Pod has no assigned node.
3. The scheduler filters out nodes whose labels do not match `nodeSelector`.
4. It applies other filters, such as resource availability, taints, volumes, and affinity rules.
5. It scores the remaining nodes.
6. It binds the Pod to the selected node.
7. The kubelet on that node pulls the image and starts the containers.

### Q17. What does `IgnoredDuringExecution` mean in node affinity?

It means the Pod can continue running if a node label later changes. `nodeSelector` is also mainly a scheduling-time constraint. It does not continuously evict a running Pod just because a label changes.

### Q18. Are Kubernetes built-in labels safe to use?

Prefer well-known, stable labels such as:

````text
kubernetes.io/hostname
topology.kubernetes.io/zone
kubernetes.io/os
kubernetes.io/arch
````

Cloud providers and Kubernetes versions may manage some labels. Do not manually overwrite provider-managed labels unless you understand the impact.

### Q19. How can I require a workload to run on dedicated nodes?

Use both a node label and a taint:

````bash
kubectl label node node-db-1 workload=database
kubectl taint node node-db-1 dedicated=database:NoSchedule
````

Then the workload needs both:

````yaml
spec:
  nodeSelector:
    workload: database
  tolerations:
    - key: dedicated
      operator: Equal
      value: database
      effect: NoSchedule
````

The selector targets the nodes. The taint prevents unrelated Pods from using them.

### Q20. How can I troubleshoot a Pending Pod?

Run:

````bash
kubectl describe pod <pod-name>
````

Look at the **Events** section. Common causes include:

- No node has the requested label.
- The label key or value is misspelled.
- Matching nodes lack CPU or memory.
- A taint has no matching toleration.
- A required volume cannot be used in the node's zone.
- The node is unschedulable or cordoned.

---

## 9. Common mistakes

### Mistake 1: Labeling the Pod instead of the node

This does not help `nodeSelector`:

````yaml
metadata:
  labels:
    disk: ssd
````

The label must be added to the node:

````bash
kubectl label node <node-name> disk=ssd
````

### Mistake 2: Wrong YAML indentation

Correct:

````yaml
spec:
  template:
    spec:
      nodeSelector:
        disk: ssd
````

### Mistake 3: Expecting OR behavior

This means AND:

````yaml
nodeSelector:
  disk: ssd
  zone: east
````

For SSD **or** NVMe, use node affinity instead.

### Mistake 4: Using a node name as a long-term strategy

A node can be replaced, autoscaled, or recreated. Prefer capability labels such as `disk=ssd` over a specific node name.

### Mistake 5: Forgetting that labels are trusted scheduling input

Restrict who can label nodes. Incorrect or unauthorized labels can place sensitive workloads on the wrong machines.

---

## 10. Quick mental model

Think of `nodeSelector` as a simple gate:

````text
Pod says:       disk=ssd
Node says:      disk=ssd       -> eligible
Another node:   disk=hdd       -> rejected
````

Then Kubernetes asks additional questions:

````text
Does the node have enough resources?
Does the Pod tolerate the node's taints?
Can its volumes be mounted there?
Are affinity and topology rules satisfied?
````

Only after all required checks pass can the Pod be scheduled.

---

## 11. When should you use it?

Use `nodeSelector` when the requirement is simple and mandatory, for example:

- Run a GPU workload on GPU nodes.
- Run a database on SSD nodes.
- Run Linux workloads on Linux nodes.
- Keep a workload in a specific environment or hardware class.

Use `nodeAffinity` when you need preferences, OR conditions, multiple alternatives, or more advanced matching. Use taints and tolerations when you need dedicated nodes that should repel unrelated workloads.

## Final summary

`nodeSelector` is an exact label match used by a Pod to select eligible nodes. It is easy to configure, but intentionally limited. The node must match every requested label, and the Pod must still pass resource, taint, volume, and other scheduler checks. If nothing matches, the Pod remains `Pending`.
