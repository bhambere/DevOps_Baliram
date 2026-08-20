# Kubernetes Pod Commands Explained

This document explains the Kubernetes commands used while checking and troubleshooting `myapp-pod`.

## 1. Command Summary

| Command | Purpose |
|---|---|
| `kubectl describe pod myapp-pod` | Shows detailed information about the pod, including containers, events, IP address, status, and errors. |
| `kubectl get pods` | Lists pods and shows their current status. |
| `kubectl logs myapp-pod -c nginx2` | Displays the current logs from the `nginx2` container. |
| `kubectl logs myapp-pod -c nginx2 --previous` | Displays logs from the previous crashed or restarted container instance. |
| `kubectl apply -f Myapp-Pod.yml` | Creates or updates the pod configuration from a YAML file. |
| `kubectl delete pod myapp-pod` | Deletes the pod. |
| `kubectl create -f Myapp-Pod.yml` | Creates a new Kubernetes object from a YAML file. It fails if the object already exists. |

## 2. Understanding the Pod

A **Pod** is the smallest deployable unit in Kubernetes. It can contain one or more containers.

In this example, the pod is named `myapp-pod`, and one of its containers is named `nginx2`.

Example YAML file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx2
      image: nginx:latest
      ports:
        - containerPort: 80
```

The names must match exactly:

- Pod name: `myapp-pod`
- Container name: `nginx2`
- YAML file name: `Myapp-Pod.yml`

Linux file names are case-sensitive. `Myapp-Pod.yml` and `myapp-pod.yml` are different file names.

## 3. Checking Pod Status

### `kubectl get pods`

This command gives a quick view of pods in the current namespace.

```bash
kubectl get pods
```

Example output:

```text
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   1/1     Running   0          2m
```

Important columns:

- `NAME`: Name of the pod.
- `READY`: Number of ready containers compared with the total number of containers.
- `STATUS`: Current pod status, such as `Running`, `Pending`, `Completed`, or `CrashLoopBackOff`.
- `RESTARTS`: Number of times a container has restarted.
- `AGE`: How long the pod has existed.

To see pods in all namespaces:

```bash
kubectl get pods --all-namespaces
```

To monitor the pod continuously:

```bash
kubectl get pods -w
```

Press `Ctrl+C` to stop watching.

## 4. Inspecting Detailed Pod Information

### `kubectl describe pod myapp-pod`

Use this command when the pod is not starting, keeps restarting, or is not ready.

```bash
kubectl describe pod myapp-pod
```

The output contains sections such as:

- `Name`: Pod name.
- `Namespace`: Namespace where the pod exists.
- `Status`: Current pod status.
- `IP`: IP address assigned to the pod.
- `Containers`: Container names, images, states, and restart information.
- `Conditions`: Whether the pod is initialized, scheduled, and ready.
- `Events`: Scheduling, image-pull, mounting, and startup events.

The **Events** section is especially useful. For example:

```text
Events:
  Type     Reason     Message
  ----     ------     -------
  Normal   Scheduled  Successfully assigned default/myapp-pod to worker-node
  Normal   Pulled     Container image "nginx:latest" already present
  Normal   Started    Started container nginx2
```

An error may look like this:

```text
Warning  Failed     Failed to pull image "nginx:wrong-tag"
```

This indicates that Kubernetes could not download the specified container image.

## 5. Reading Container Logs

### Current container logs

```bash
kubectl logs myapp-pod -c nginx2
```

Explanation:

- `myapp-pod` is the pod name.
- `-c nginx2` selects the container named `nginx2`.
- The command prints logs produced by the current container process.

Example output:

```text
/docker-entrypoint.sh: Configuration complete
nginx: ready to handle connections
```

To follow new log entries as they appear:

```bash
kubectl logs -f myapp-pod -c nginx2
```

To show only the last 100 lines:

```bash
kubectl logs --tail=100 myapp-pod -c nginx2
```

To show logs from the last 10 minutes:

```bash
kubectl logs --since=10m myapp-pod -c nginx2
```

### Logs from a previous container instance

```bash
kubectl logs myapp-pod -c nginx2 --previous
```

Use `--previous` when the container crashed or restarted. It shows the logs from the previous container instance, not the currently running instance.

Example:

```text
2026/08/11 17:30:22 [emerg] 1#1: invalid number of arguments in "listen" directive
```

This message can explain why the previous container stopped.

The command may fail if the container has never restarted, because there is no previous instance:

```text
Error from server: previous terminated container "nginx2" in pod "myapp-pod" not found
```

## 6. The Incorrect Log Command

The following command was used:

```bash
kubectl logs myapp-pod -c nginx2 --p
```

`--p` is not the correct option for normal Kubernetes log retrieval. Use one of these commands instead:

```bash
kubectl logs myapp-pod -c nginx2
```

For the previous container instance:

```bash
kubectl logs myapp-pod -c nginx2 --previous
```

## 7. Applying a YAML Configuration

### `kubectl apply -f Myapp-Pod.yml`

```bash
kubectl apply -f Myapp-Pod.yml
```

This command reads the YAML file and creates or updates the Kubernetes object described in it.

- If the pod does not exist, Kubernetes creates it.
- If the pod already exists, Kubernetes tries to update it.
- Running the same `apply` command several times is normally safe and idempotent.

Example output for a new pod:

```text
pod/myapp-pod created
```

Example output when there is no change:

```text
pod/myapp-pod unchanged
```

Example output after a change:

```text
pod/myapp-pod configured
```

Before applying a file, you can validate what Kubernetes would change without actually changing the cluster:

```bash
kubectl apply --dry-run=client -f Myapp-Pod.yml
```

To inspect the YAML file locally:

```bash
cat Myapp-Pod.yml
```

## 8. Applying the Same File Multiple Times

The command was executed several times:

```bash
kubectl apply -f Myapp-Pod.yml
```

This is usually not a problem. Kubernetes compares the desired configuration in the YAML file with the current object.

If nothing changed, the result is normally:

```text
pod/myapp-pod unchanged
```

Repeated `apply` commands are useful when testing changes. However, if the YAML contains fields that cannot be changed for an existing Pod, Kubernetes may return an error. Some Pod fields are effectively immutable after creation, such as certain parts of the container specification.

If a change cannot be applied to the existing Pod, delete and recreate it, or use a Deployment instead of managing a single Pod directly.

## 9. Deleting the Pod

### `kubectl delete pod myapp-pod`

```bash
kubectl delete pod myapp-pod
```

This removes the pod from the current namespace.

Example output:

```text
pod "myapp-pod" deleted
```

After deletion, verify the result:

```bash
kubectl get pods
```

If the pod was created directly using `kubectl apply` or `kubectl create`, it normally will not come back automatically after deletion. A controller such as a Deployment is needed for automatic recreation.

## 10. Creating a Pod from YAML

### `kubectl create -f Myapp-Pod.yml`

```bash
kubectl create -f Myapp-Pod.yml
```

This creates the object described in the YAML file.

Example output:

```text
pod/myapp-pod created
```

If the pod already exists, the command returns an error similar to:

```text
Error from server (AlreadyExists): pods "myapp-pod" already exists
```

Use `create` when you specifically want to create a new object. Use `apply` when you want to create or update an object declaratively.

## 11. Difference Between `create` and `apply`

| Feature | `kubectl create` | `kubectl apply` |
|---|---|---|
| Creates a new object | Yes | Yes |
| Updates an existing object | No | Yes |
| Safe to run repeatedly | No, it fails if the object exists | Yes, usually |
| Best use | One-time creation | Managing configuration over time |

Recommended approach for configuration files:

```bash
kubectl apply -f Myapp-Pod.yml
```

## 12. Recommended Troubleshooting Workflow

Use this sequence when `myapp-pod` is not working:

### Step 1: Check the pod status

```bash
kubectl get pods
```

### Step 2: Get detailed information and events

```bash
kubectl describe pod myapp-pod
```

### Step 3: Read current container logs

```bash
kubectl logs myapp-pod -c nginx2
```

### Step 4: Read logs from a crashed container

```bash
kubectl logs myapp-pod -c nginx2 --previous
```

### Step 5: Correct the YAML file and apply it

```bash
kubectl apply -f Myapp-Pod.yml
```

### Step 6: Check the result again

```bash
kubectl get pods
kubectl describe pod myapp-pod
```

### Step 7: Delete and recreate only when necessary

```bash
kubectl delete pod myapp-pod
kubectl apply -f Myapp-Pod.yml
```

## 13. Complete Example

Assume the YAML file contains a Pod named `myapp-pod` with a container named `nginx2`.

```bash
# Create or update the pod
kubectl apply -f Myapp-Pod.yml

# Check whether the pod is running
kubectl get pods

# Inspect detailed status and events
kubectl describe pod myapp-pod

# Read logs from the nginx2 container
kubectl logs myapp-pod -c nginx2

# If the container restarted, read the previous logs
kubectl logs myapp-pod -c nginx2 --previous

# Delete the pod if it must be recreated
kubectl delete pod myapp-pod

# Create it again from the YAML file
kubectl apply -f Myapp-Pod.yml
```

## 14. Quick Reference

```bash
# List pods
kubectl get pods

# List pods with more details
kubectl get pods -o wide

# Describe a pod
kubectl describe pod myapp-pod

# View current logs
kubectl logs myapp-pod -c nginx2

# Follow logs
kubectl logs -f myapp-pod -c nginx2

# View previous logs after a restart
kubectl logs myapp-pod -c nginx2 --previous

# Create or update from YAML
kubectl apply -f Myapp-Pod.yml

# Validate without changing the cluster
kubectl apply --dry-run=client -f Myapp-Pod.yml

# Delete the pod
kubectl delete pod myapp-pod
```

## 15. Important Notes

1. Run the commands in the namespace where the pod exists. For another namespace, add `-n`, for example:

   ```bash
   kubectl get pods -n dev
   kubectl logs -n dev myapp-pod -c nginx2
   ```

2. If the pod has multiple containers, `-c nginx2` is required to select the correct container.

3. Use `kubectl describe` for Kubernetes events and resource details. Use `kubectl logs` for application output.

4. Use `--previous` only when the container has restarted and you need the logs from the older container instance.

5. For production workloads, a `Deployment` is usually preferred over creating a standalone Pod because a Deployment can maintain the desired number of replicas and recreate failed Pods automatically.
