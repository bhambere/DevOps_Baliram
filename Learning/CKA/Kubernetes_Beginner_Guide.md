# Kubernetes Architecture, Very Easy Explanation

The correct names are:

- **Master** is now usually called the **Control Plane**.
- **Control Manager** is called the **Controller Manager**.
- **Worker nodes** run your applications.
- **Kubelet** runs on every worker node.

## Simple Architecture Diagram

```text
                 YOU / DEVELOPER
                        |
                        | kubectl create deployment
                        v
              +----------------------+
              |    KUBE-APISERVER    |
              | Main entry point     |
              +----------+-----------+
                         |
          +--------------+--------------+
          |                             |
          v                             v
     +----------+              +-------------------+
     |   ETCD   |              | CONTROLLER        |
     | Cluster  |              | MANAGER           |
     | database |              | Keeps desired     |
     +----------+              | state running     |
                                +---------+---------+
                                          |
                                          v
                                +-------------------+
                                | KUBE SCHEDULER   |
                                | Selects a worker  |
                                | node for the Pod  |
                                +---------+---------+
                                          |
              ------------------------------------------------
              |                                              |
              v                                              v
       +---------------+                              +---------------+
       | WORKER NODE 1 |                              | WORKER NODE 2 |
       +---------------+                              +---------------+
       | Kubelet       |                              | Kubelet       |
       | Kube-proxy    |                              | Kube-proxy    |
       | Runtime       |                              | Runtime       |
       | Application   |                              | Application   |
       | Pod           |                              | Pod           |
       +---------------+                              +---------------+
```

## What Does Each Component Do?

### 1. Control Plane

The **Control Plane** is the brain of the Kubernetes cluster.

It decides:

- What applications should run.
- How many copies should run.
- Which worker node should run each application.
- What to do when something fails.

The Control Plane usually contains:

```text
Control Plane
├── kube-apiserver
├── etcd
├── controller manager
└── kube scheduler
```

### 2. kube-apiserver

The **kube-apiserver** is the main communication door of Kubernetes.

You send commands to it using `kubectl`:

```text
kubectl create deployment nginx --image=nginx
```

The request goes here:

```text
kubectl → kube-apiserver
```

The API server communicates with the other Kubernetes components.

Think of the API server as a **receptionist**. Everyone talks through the receptionist instead of talking directly to each other.

### 3. etcd

**etcd** is Kubernetes' database. It stores information such as:

- Which applications should be running.
- How many replicas are required.
- Which Pods exist.
- Which nodes are available.
- Cluster configuration and secrets.

Example:

```text
Desired state:
Run 3 copies of the nginx application
```

This information is stored in etcd.

Think of etcd as the **Kubernetes notebook or memory**.

### 4. Controller Manager

The **Controller Manager** continuously checks whether the actual situation matches the desired situation.

Example:

```text
You want: 3 nginx Pods
Current situation: 2 nginx Pods
```

The Controller Manager notices the problem and asks Kubernetes to create another Pod.

```text
Desired: 3 Pods
Actual:  2 Pods

Controller Manager:
"One Pod is missing. Create one more."
```

Think of it as a **supervisor** who continuously checks the work.

### 5. kube-scheduler

The **kube-scheduler** decides which worker node should run a new Pod.

It checks things such as:

- Available CPU.
- Available memory.
- Node status.
- Special rules.
- Whether the node can run the Pod.

Example:

```text
New nginx Pod needs a home.

Worker Node 1: Not enough memory
Worker Node 2: Enough memory

Scheduler chooses Worker Node 2.
```

Think of the scheduler as a **hotel receptionist assigning a room** to a guest.

The scheduler chooses the node, but it does not run the container itself.

# Worker Nodes

A **worker node** is a machine that runs your actual applications.

A cluster can have one or many worker nodes.

```text
Worker Node
├── kubelet
├── kube-proxy
├── container runtime
└── Pods
```

## 6. kubelet

The **kubelet** is an agent running on every worker node.

It receives instructions from the API server and makes sure the required Pods are running.

Example:

```text
API Server:
"Run an nginx Pod on Worker Node 1."

Kubelet:
"Okay, I will create and monitor that Pod."
```

The kubelet:

- Creates Pods.
- Starts containers.
- Checks whether containers are healthy.
- Restarts failed containers.
- Reports node and Pod status to the API server.

Think of kubelet as a **worker manager** on each machine.

## 7. Container Runtime Engine

The **container runtime** actually starts and stops containers.

Common examples include:

- containerd
- CRI-O

The kubelet does not directly create the container. It asks the container runtime to do it.

```text
Kubelet:
"Please start an nginx container."

Container Runtime:
"Container started."
```

Think of the container runtime as the **engine of a car**. It performs the actual work of running containers.

## 8. kube-proxy

**kube-proxy** manages network communication on each worker node.

It helps traffic reach the correct Pod.

Example:

```text
User requests:
http://my-nginx-service

kube-proxy:
"Send this request to one of the nginx Pods."
```

If you have three nginx Pods, kube-proxy helps distribute network traffic between them.

```text
                 Service
                    |
          +---------+---------+
          |         |         |
          v         v         v
       Pod 1      Pod 2      Pod 3
```

Think of kube-proxy as a **traffic police officer** directing vehicles to the correct destination.

# Complete Example: Running an Nginx Application

Suppose you run:

```text
kubectl create deployment nginx --image=nginx
```

Here is what happens:

```text
1. You use kubectl
        |
        v
2. kubectl sends the request to kube-apiserver
        |
        v
3. kube-apiserver saves the desired state in etcd
   Desired state: Run one nginx Pod
        |
        v
4. Controller Manager notices that nginx needs to run
        |
        v
5. Scheduler selects a suitable worker node
        |
        v
6. kubelet on that worker node receives the instruction
        |
        v
7. kubelet asks the container runtime to start nginx
        |
        v
8. Container runtime starts the nginx container
        |
        v
9. kubelet reports the status back to kube-apiserver
        |
        v
10. kube-proxy helps network traffic reach the nginx Pod
```

## Complete Flow Diagram

```text
Developer
   |
   | kubectl command
   v
kube-apiserver
   |
   +--> etcd
   |    Stores cluster information
   |
   +--> Controller Manager
   |    Checks what should be running
   |
   +--> Scheduler
        Chooses a worker node
              |
              v
        Worker Node
              |
              +--> kubelet
              |    Creates and monitors the Pod
              |
              +--> container runtime
              |    Starts the container
              |
              +--> kube-proxy
                   Manages network traffic
```

# Very Simple Real-Life Example

Imagine a company kitchen:

| Kubernetes Component | Real-Life Example |
|---|---|
| Control Plane | Head office |
| kube-apiserver | Reception desk |
| etcd | Company record book |
| Controller Manager | Supervisor |
| Scheduler | Person assigning work to employees |
| Worker Node | Kitchen |
| kubelet | Kitchen manager |
| Container Runtime | Cooking equipment |
| Pod | Prepared dish |
| kube-proxy | Waiter delivering dishes |

Example:

```text
Customer orders 3 pizzas
        |
        v
Reception records the order
        |
        v
Supervisor checks that 3 pizzas are prepared
        |
        v
Scheduler assigns the order to a kitchen
        |
        v
Kitchen manager tells the cooking equipment to prepare pizzas
        |
        v
Waiter delivers the pizzas to the customer
```

# What Happens If a Pod Fails?

Suppose you want three Pods:

```text
Pod 1: Running
Pod 2: Running
Pod 3: Failed
```

Kubernetes automatically tries to fix the problem:

```text
Controller Manager notices the failed Pod
        |
        v
API Server records the new request
        |
        v
Scheduler selects a node if needed
        |
        v
Kubelet starts a replacement Pod
        |
        v
Container Runtime runs the new container
```

Final result:

```text
Pod 1: Running
Pod 2: Running
Pod 3: Running
```

This self-healing ability is one of the most important features of Kubernetes.

## One-Line Summary

```text
Control Plane decides what should happen,
worker nodes run the applications,
kubelet manages the Pods,
container runtime runs the containers,
and kube-proxy manages network traffic.
```
