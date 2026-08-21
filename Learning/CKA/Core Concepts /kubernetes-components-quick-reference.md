# Kubernetes Components, Side-by-Side Comparison

## 1. Core Kubernetes components

| Component | Runs where | Main use | Simple comparison |
|---|---|---|---|
| **Control plane** | Dedicated control-plane node(s) | Manages the entire cluster | The management team |
| **Worker node** | Worker machine or VM | Runs application workloads | The production computer |
| **Pod** | On a worker node | Smallest deployable Kubernetes unit | A package containing the application |
| **Container** | Inside a Pod | Runs the actual application process | The application itself |
| **Cluster** | Control plane plus worker nodes | Complete Kubernetes environment | The whole company |

## 2. Control plane components

| Component | Main use | Easy comparison |
|---|---|---|
| **kube-apiserver** | Receives commands from users and components | Front desk |
| **etcd** | Stores cluster configuration and current state | Database or filing cabinet |
| **kube-scheduler** | Selects a worker node for new Pods | Job assignment manager |
| **kube-controller-manager** | Ensures the actual cluster matches the desired state | Supervisor |
| **cloud-controller-manager** | Connects Kubernetes to cloud services | Cloud specialist |

### Example flow

```text
You create a Deployment
        ↓
kube-apiserver receives the request
        ↓
etcd stores the desired configuration
        ↓
kube-scheduler selects a worker node
        ↓
kubelet starts the Pod
        ↓
controller-manager checks that the Pod remains healthy
```

## 3. Worker node components

| Component | Main use | Easy comparison |
|---|---|---|
| **kubelet** | Makes sure assigned Pods are running correctly | Local supervisor |
| **Container runtime** | Downloads images and starts or stops containers | Container engine |
| **kube-proxy** | Helps route Service traffic to the correct Pods | Network traffic assistant |
| **Pod** | Provides the environment in which containers run | Application package |

Common container runtimes include `containerd` and `CRI-O`.

## 4. Application workload objects

| Object | Main use | Creates or manages | Use it when |
|---|---|---|---|
| **Deployment** | Runs and updates stateless applications | ReplicaSets and Pods | You run web servers or APIs |
| **ReplicaSet** | Keeps the required number of Pod copies running | Pods | Usually indirectly through a Deployment |
| **StatefulSet** | Runs applications needing stable names or storage | Stateful Pods | You run databases or clustered systems |
| **DaemonSet** | Runs one Pod on each selected node | One Pod per node | You need logging or monitoring agents |
| **Job** | Runs a task until it completes | Temporary Pods | You need a one-time task |
| **CronJob** | Runs Jobs on a schedule | Scheduled Jobs | You need backups or reports |
| **Pod** | Runs one or more containers | Containers | You need the actual workload unit |

### Workload relationship

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Containers
```

## 5. Networking components

| Component | Main use | Scope | Easy comparison |
|---|---|---|---|
| **Service** | Provides a stable address for Pods | Internal or external | Reception phone number |
| **ClusterIP Service** | Provides internal access to Pods | Inside the cluster | Internal extension |
| **NodePort Service** | Exposes a Service through a node port | Basic external access | Open building door |
| **LoadBalancer Service** | Creates or connects to a cloud load balancer | External access | Public reception |
| **Ingress** | Routes HTTP and HTTPS requests to Services | External web traffic | Traffic director |
| **NetworkPolicy** | Controls which Pods may communicate | Pod-to-Pod traffic | Security checkpoint |
| **kube-proxy** | Helps implement Service networking | Worker node | Local traffic router |

### Network relationship

```text
Internet
   ↓
Ingress or LoadBalancer
   ↓
Service
   ↓
Pods
   ↓
Containers
```

## 6. Configuration and security objects

| Object | Main use | Stores or provides | Important difference |
|---|---|---|---|
| **ConfigMap** | Provides normal application configuration | URLs, settings, feature flags | Not intended for secrets |
| **Secret** | Provides sensitive configuration | Passwords, tokens, certificates | Designed for sensitive values |
| **ServiceAccount** | Gives a Pod an identity | Application identity | Used by workloads to access Kubernetes |
| **Role** | Defines permissions | Namespace-level permissions | Permission definition only |
| **RoleBinding** | Assigns a Role to a user or ServiceAccount | Permission assignment | Connects identity to permissions |
| **ClusterRole** | Defines cluster-wide permissions | Cluster or reusable permissions | Broader than a Role |
| **ClusterRoleBinding** | Assigns a ClusterRole | Cluster-level access | Grants permissions across the cluster |
| **Namespace** | Separates resources and teams | Logical environment boundary | Similar to a project or department |

## 7. Storage components

| Object | Main use | Easy comparison |
|---|---|---|
| **Volume** | Provides storage to a Pod | Disk attached to an application |
| **PersistentVolume, PV** | Represents available storage | Storage resource |
| **PersistentVolumeClaim, PVC** | Requests storage for a workload | Storage request |
| **StorageClass** | Defines how storage should be created | Storage template |
| **StatefulSet** | Connects stable Pods with persistent storage | Database manager |

### Storage relationship

```text
StorageClass
      ↓
PersistentVolume
      ↓
PersistentVolumeClaim
      ↓
Pod
```

## 8. Most common confusion

| Easy to confuse | Difference |
|---|---|
| **Control plane vs. worker node** | Control plane makes decisions, worker nodes run applications |
| **Pod vs. container** | A Pod hosts containers, a container runs the application |
| **Deployment vs. Pod** | Deployment manages Pods, Pod runs the workload |
| **Service vs. Ingress** | Service exposes Pods, Ingress routes web traffic to Services |
| **ConfigMap vs. Secret** | ConfigMap stores normal settings, Secret stores sensitive values |
| **PV vs. PVC** | PV is available storage, PVC is a request for storage |
| **Role vs. RoleBinding** | Role defines permissions, RoleBinding assigns them |
| **kubelet vs. kube-proxy** | kubelet manages Pods, kube-proxy helps route network traffic |
| **Scheduler vs. Controller Manager** | Scheduler chooses a node, Controller Manager keeps the desired state |
| **Master vs. Control plane** | Master is the old term, Control plane is the preferred term |
| **Control Manager vs. Controller Manager** | The correct Kubernetes term is **Controller Manager** |

## Very short memory guide

```text
API server       = receives requests
etcd             = stores cluster state
Scheduler        = chooses the worker node
Controller       = keeps the desired state
Kubelet          = runs and monitors Pods
Runtime          = runs containers
Kube-proxy       = handles Service networking
Pod              = runs containers
Deployment       = manages Pods
Service          = provides stable access
Ingress          = routes web traffic
ConfigMap        = normal configuration
Secret            = sensitive configuration
PVC              = requests storage
Namespace         = separates resources
```
