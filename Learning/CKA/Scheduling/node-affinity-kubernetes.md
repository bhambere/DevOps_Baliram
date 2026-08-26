# Kubernetes Node Affinity, Made Easy

## 1. What is node affinity?

**Node affinity tells Kubernetes which nodes are suitable for running a Pod.**

Think of a hotel:

- A Pod is a guest.
- A node is a hotel room.
- Node labels describe each room, such as `location=india`, `disk=ssd`, or `team=payments`.
- Node affinity is the guest's room preference or requirement.
- The Kubernetes scheduler chooses a matching room.

Without node affinity, Kubernetes can place a Pod on any available node. With node affinity, you can say:

> "Run this Pod only on nodes with SSD disks."

or:

> "Prefer nodes in India, but use another location if necessary."

---

## 2. Real-world example

Suppose a company has three types of servers:

| Node | Labels | Purpose |
|---|---|---|
| `node-1` | `disk=ssd`, `zone=india-west` | Fast application workloads |
| `node-2` | `disk=hdd`, `zone=india-west` | Cheap batch workloads |
| `node-3` | `disk=ssd`, `zone=india-south` | Fast application workloads |

A database needs fast storage. We can require the Pod to run only on SSD nodes:

````yaml
apiVersion: v1
kind: Pod
metadata:
  name: database
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk
            operator: In
            values:
            - ssd
  containers:
  - name: database
    image: postgres:16
````

The database can run on `node-1` or `node-3`, but not `node-2`.

---

## 3. First, label the nodes

Node affinity works with node labels. Add a label like this:

````bash
kubectl label node node-1 disk=ssd
kubectl label node node-2 disk=hdd
kubectl label node node-3 disk=ssd
````

View labels:

````bash
kubectl get nodes --show-labels
````

For production, use stable labels such as:

````text
kubernetes.io/arch=amd64
kubernetes.io/os=linux
topology.kubernetes.io/zone=india-west
node-pool=compute
workload=database
````

Avoid relying on labels that users can change casually. Protect important labels with cluster permissions and policy controls.

---

## 4. Two important types of node affinity

### A. Required affinity: a hard rule

Field:

````yaml
requiredDuringSchedulingIgnoredDuringExecution
````

Meaning:

> The Pod must run on a matching node. If no node matches, the Pod stays `Pending`.

Example:

````yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: workload
          operator: In
          values: [database]
````

Use this when placement is necessary, for example a workload needs a GPU, a special device, or a particular operating system.

### B. Preferred affinity: a soft rule

Field:

````yaml
preferredDuringSchedulingIgnoredDuringExecution
````

Meaning:

> Prefer a matching node, but use another node if required.

Example:

````yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 80
      preference:
        matchExpressions:
        - key: topology.kubernetes.io/zone
          operator: In
          values: [india-west]
````

`weight` is from `1` to `100`. A higher weight gives the preference more influence when Kubernetes scores nodes.

---

## 5. Why does the name contain `IgnoredDuringExecution`?

The full names are intentionally precise:

- `requiredDuringSchedulingIgnoredDuringExecution`
- `preferredDuringSchedulingIgnoredDuringExecution`

They mean:

- During **scheduling**, Kubernetes checks the rule.
- During **execution**, Kubernetes normally does not evict the already-running Pod if the node label later changes.

Example:

1. A Pod is scheduled to a node labeled `disk=ssd`.
2. The label is removed later.
3. The Pod usually continues running.
4. A newly scheduled Pod will not select that node if the rule is required.

Kubernetes does not currently provide a standard `requiredDuringSchedulingRequiredDuringExecution` node affinity mode. Eviction, rescheduling, or a separate controller may be needed if continuous enforcement is required.

---

## 6. Node affinity operators

The most commonly used operators are:

| Operator | Meaning |
|---|---|
| `In` | Label value must be one of the listed values |
| `NotIn` | Label value must not be one of the listed values |
| `Exists` | Label key must exist, values are not needed |
| `DoesNotExist` | Label key must not exist |
| `Gt` | Numeric label value must be greater than the specified value |
| `Lt` | Numeric label value must be less than the specified value |

Examples:

````yaml
# disk is SSD or NVMe
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disk
          operator: In
          values: [ssd, nvme]
````

````yaml
# GPU label must exist
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: accelerator
          operator: Exists
````

---

## 7. `nodeSelectorTerms` and `matchExpressions` logic

This is a common source of mistakes.

### Different terms use OR

````yaml
nodeSelectorTerms:
- matchExpressions:
  - key: zone
    operator: In
    values: [india-west]
- matchExpressions:
  - key: zone
    operator: In
    values: [india-south]
````

Meaning:

> The node can be in `india-west` OR `india-south`.

### Expressions inside one term use AND

````yaml
nodeSelectorTerms:
- matchExpressions:
  - key: disk
    operator: In
    values: [ssd]
  - key: workload
    operator: In
    values: [database]
````

Meaning:

> The node must have `disk=ssd` AND `workload=database`.

A simple memory trick:

> **Terms are OR. Expressions in a term are AND.**

---

## 8. Node affinity versus `nodeSelector`

`nodeSelector` is the simpler form:

````yaml
spec:
  nodeSelector:
    disk: ssd
````

It means the Pod must run on a node with `disk=ssd`.

Use `nodeSelector` when you need one simple exact-match rule. Use node affinity when you need:

- Multiple possible values
- OR conditions
- Soft preferences
- Operators such as `Exists`, `NotIn`, `Gt`, or `Lt`
- More expressive placement logic

Both can be used together. If both are present, **both conditions must be satisfied**.

---

## 9. How scheduling works under the hood

The Kubernetes scheduler generally works in two major stages:

1. **Filter:** Remove nodes that cannot run the Pod.
2. **Score:** Rank the remaining nodes and choose the best one.

Node affinity participates in both stages:

- Required affinity is mainly a filter.
- Preferred affinity contributes to scoring.

### Flow chart

````mermaid
flowchart TD
    A[Pod is created] --> B[Scheduler watches for unscheduled Pod]
    B --> C[Read Pod node affinity rules]
    C --> D[Read node labels and cluster state]
    D --> E{Do required rules match?}
    E -- No --> F[Filter out the node]
    E -- Yes --> G[Keep node as a candidate]
    F --> H{Any candidate nodes left?}
    G --> H
    H -- No --> I[Pod remains Pending]
    H -- Yes --> J[Score candidates using preferences and other rules]
    J --> K[Select the highest-scoring suitable node]
    K --> L[Bind Pod to node]
    L --> M[Kubelet pulls image and starts containers]
````

### Simplified internal view

````text
Pod with affinity
       |
       v
Scheduler framework
       |
       +--> PreFilter: prepare affinity information
       |
       +--> Filter: reject nodes that fail required affinity
       |
       +--> Score: give points to nodes matching preferred affinity
       |
       +--> Reserve / Permit: optional scheduling checks
       |
       +--> Bind: assign Pod.spec.nodeName
````

The scheduler does not move a running Pod just because another node becomes a better match. Scheduling happens when the Pod needs a node.

---

## 10. Complete Deployment example

This example requires a node labeled `workload=api` and prefers the `india-west` zone.

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payments-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payments-api
  template:
    metadata:
      labels:
        app: payments-api
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: workload
                operator: In
                values: [api]
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 70
            preference:
              matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values: [india-west]
      containers:
      - name: api
        image: nginx:1.27
        ports:
        - containerPort: 80
````

Apply and inspect it:

````bash
kubectl apply -f payments-api.yaml
kubectl get pods -o wide
kubectl describe pod <pod-name>
````

If placement fails, look near the Events section of `kubectl describe pod`. You may see messages such as:

````text
0/3 nodes are available: 3 node(s) didn't match Pod's node affinity/selector.
````

---

## 11. Node affinity versus related features

### Node affinity versus taints and tolerations

- **Node affinity:** Pod says, "I want to run on these nodes."
- **Taint:** Node says, "Do not run Pods here unless they tolerate me."
- **Toleration:** Pod says, "I am allowed to run on a tainted node."

For dedicated nodes, use both:

1. Label the node, for example `workload=database`.
2. Add a taint, for example `workload=database:NoSchedule`.
3. Add required node affinity to select the node.
4. Add a toleration to permit the Pod onto the node.

Affinity attracts. Taints repel.

### Node affinity versus Pod affinity

- **Node affinity** places a Pod based on node labels.
- **Pod affinity** places a Pod near other Pods.
- **Pod anti-affinity** keeps a Pod away from other Pods.

Example: node affinity selects SSD nodes. Pod anti-affinity spreads replicas across zones or nodes.

---

## 12. Troubleshooting checklist

When a Pod is stuck in `Pending`:

````bash
kubectl get nodes --show-labels
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o yaml
````

Check the following:

1. Does the required label exist on any node?
2. Is the label key spelled correctly?
3. Is the value spelled correctly and case-sensitive?
4. Did you use `AND` conditions accidentally when you wanted `OR`?
5. Are there enough CPU, memory, or other resources?
6. Are nodes cordoned or marked unschedulable?
7. Are taints blocking the Pod?
8. Are other constraints, such as topology spread or inter-Pod affinity, conflicting?
9. Is the affinity placed under `spec.template.spec` for a Deployment, not only under the Deployment's top-level `spec`?

A useful test is to temporarily inspect labels and compare them directly with the affinity rule.

---

# Basic questions and answers

### 1. What problem does node affinity solve?

It controls which nodes are eligible or preferred for a Pod.

### 2. Does node affinity create a node?

No. It only selects from existing nodes. A cluster autoscaler may add nodes separately if configured and if the node group can satisfy the requirement.

### 3. What happens if no node matches required affinity?

The Pod remains `Pending`. Kubernetes does not ignore a required rule.

### 4. Is label matching case-sensitive?

Yes. `SSD` and `ssd` are different values.

### 5. Where is affinity configured?

For a Pod, use `spec.affinity`. For a Deployment, StatefulSet, or Job, use `spec.template.spec.affinity`.

### 6. Can one Pod use more than one affinity rule?

Yes. You can define multiple expressions and terms, but make sure their AND and OR logic is what you intend.

### 7. Can node affinity select a specific node name?

It can, but selecting a node by name is usually less flexible. Prefer stable labels such as node pool, zone, hardware type, or workload class.

---

# Intermediate questions and answers

### 1. What is the difference between required and preferred affinity?

Required affinity is a hard filter. Preferred affinity is a scoring preference and can be ignored if necessary.

### 2. How does `weight` work?

Each preferred rule has a weight from `1` to `100`. A matching node receives points for that preference. The scheduler combines these points with other scheduling scores, so weight is a preference, not an absolute guarantee.

### 3. What does `IgnoredDuringExecution` mean?

The rule is checked while scheduling. If labels later change, Kubernetes normally does not evict the running Pod automatically.

### 4. What is the difference between `In` and `Exists`?

`In` checks both a key and specific values. `Exists` only checks that the key is present.

### 5. What happens when `nodeSelector` and node affinity are both defined?

Both must match. This can unintentionally make scheduling impossible.

### 6. Why is a Pod still pending even though affinity matches?

Affinity is only one requirement. Resources, taints, volumes, topology constraints, readiness, and other scheduler plugins can also prevent placement.

### 7. Can preferred affinity guarantee placement?

No. Use required affinity for a guarantee, but use it carefully because strict rules can reduce availability.

### 8. How can I spread replicas across zones?

Use topology spread constraints or Pod anti-affinity, often together with node affinity. Node affinity alone usually selects allowed zones but does not guarantee even distribution.

---

# Advanced questions and answers

### 1. How does the scheduler combine required and preferred affinity?

Required rules are evaluated during filtering. Nodes that fail are removed. Preferred rules are evaluated during scoring, and matching nodes receive additional points. The final choice also depends on resource fit, topology, taints, volumes, and other scheduler plugins.

### 2. Does the scheduler scan every node for every Pod?

Conceptually it evaluates candidate nodes, but implementation details such as parallelism, caching, percentage of nodes considered, profiles, and framework plugins affect the exact behavior. Do not assume a simple sequential scan.

### 3. What is the effect of multiple `nodeSelectorTerms`?

They are OR alternatives. A node needs to match at least one term. Within a term, all expressions must match.

### 4. Can affinity rules become invalid after a label change?

Yes. A running Pod may continue operating because of `IgnoredDuringExecution`, while new Pods may fail to schedule. This can create inconsistent placement across replicas.

### 5. How should dedicated node pools be implemented?

Use a combination of:

- A stable node label for selection
- A taint to keep unrelated Pods away
- Required node affinity on the intended workload
- A matching toleration
- Resource requests and limits
- Monitoring for pending Pods and capacity

### 6. How does node affinity interact with cluster autoscaling?

A pending Pod with required affinity may trigger scale-up only when the autoscaler knows that an available node group can create a matching node. Incorrect or nonstandard labels can prevent scale-up or cause it to choose the wrong node group.

### 7. Can node affinity be changed after a Pod is created?

For controllers, changing the Pod template usually creates replacement Pods according to the controller's rollout behavior. Changing a node label does not automatically relocate existing Pods. A Pod generally must be recreated to be scheduled again.

### 8. What are the risks of overusing required affinity?

It can cause pending Pods, poor utilization, fragile deployments, and dependency on one node pool or zone. Use hard rules only for real requirements. Use preferences when placement is an optimization.

### 9. How can labels be protected?

Use restricted permissions, admission policies, and trusted label namespaces where appropriate. In production, prevent ordinary users from changing labels that influence security-sensitive scheduling decisions.

### 10. What should be tested before production use?

Test node failure, zone failure, node draining, autoscaler behavior, rollout behavior, missing labels, taints, resource exhaustion, and label changes. Confirm that the failure mode is acceptable when no matching node is available.

---

## Quick summary

Remember these five points:

1. **Labels describe nodes.**
2. **Node affinity selects or prefers nodes using those labels.**
3. **Required means must match.**
4. **Preferred means try to match.**
5. **Terms are OR, expressions inside a term are AND.**

A practical rule is:

> Use required node affinity for hardware, operating system, compliance, or isolation requirements. Use preferred node affinity for performance, locality, or optimization preferences.
