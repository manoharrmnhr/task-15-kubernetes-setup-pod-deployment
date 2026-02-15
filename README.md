# Task 15: Kubernetes Setup + First Pod Deployment

## Overview
This task covers setting up a local Kubernetes cluster using Minikube and deploying your first nginx Pod using kubectl.

---

## Prerequisites
- Linux/macOS/Windows (WSL2 recommended on Windows)
- Docker installed and running
- At least 2 CPU cores and 2GB RAM free

---

## Step-by-Step Guide

---

### STEP 1: Install kubectl

**Linux:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

**macOS:**
```bash
brew install kubectl
```

**Verify installation:**
```bash
kubectl version --client
```

**Expected output:**
```
Client Version: v1.29.x
Kustomize Version: v5.x.x
```

---

### STEP 2: Install Minikube

**Linux:**
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**macOS:**
```bash
brew install minikube
```

**Verify installation:**
```bash
minikube version
```

**Expected output:**
```
minikube version: v1.32.x
commit: abc123...
```

---

### STEP 3: Start Minikube Cluster

```bash
minikube start --driver=docker
```

> If Docker is your container runtime, `--driver=docker` is recommended. On macOS with Apple Silicon, you can also use `--driver=qemu2`.

**Expected output:**
```
😄  minikube v1.32.0 on Ubuntu 22.04
✨  Using the docker driver based on user configuration
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster
```

**Check Minikube status:**
```bash
minikube status
```

**Expected output:**
```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

---

### STEP 4: Explore Cluster Details

**Get cluster info:**
```bash
kubectl cluster-info
```

**Expected output:**
```
Kubernetes control plane is running at https://127.0.0.1:49154
CoreDNS is running at https://127.0.0.1:49154/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

**Verify nodes:**
```bash
kubectl get nodes
```

**Expected output:**
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   2m    v1.28.3
```

**Get more node details:**
```bash
kubectl get nodes -o wide
```

---

### STEP 5: Create the Pod YAML File

Create a file named `pod.yml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: dev
    tier: frontend
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
```

**YAML Field Explanation:**

| Field | Description |
|-------|-------------|
| `apiVersion: v1` | Uses the core Kubernetes API version |
| `kind: Pod` | Declares this as a Pod resource |
| `metadata.name` | Unique name of the pod: `nginx-pod` |
| `metadata.labels` | Key-value tags for grouping/selecting pods |
| `spec.containers` | List of containers to run inside the pod |
| `image: nginx:latest` | Docker image to use |
| `containerPort: 80` | Port the container listens on |
| `resources.requests` | Minimum resources guaranteed |
| `resources.limits` | Maximum resources allowed |

---

### STEP 6: Deploy the Pod

```bash
kubectl apply -f pod.yml
```

**Expected output:**
```
pod/nginx-pod created
```

> **Why `apply` instead of `create`?**
> `kubectl apply` is declarative — it creates the resource if it doesn't exist, or updates it if it does. `kubectl create` is imperative and fails if the resource already exists. `apply` is preferred for reproducibility and GitOps workflows.

---

### STEP 7: Check Pod Status

**Basic pod list:**
```bash
kubectl get pods
```

**Expected output:**
```
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          30s
```

**Wide output (shows IP and Node):**
```bash
kubectl get pods -o wide
```

**Expected output:**
```
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          1m    10.244.0.3   minikube   <none>           <none>
```

**Status Lifecycle explanation:**
- `Pending` → Pod is scheduled but container not yet started
- `ContainerCreating` → Image is being pulled
- `Running` → Container is up and healthy
- `CrashLoopBackOff` → Container keeps crashing
- `Completed` → Container exited successfully

---

### STEP 8: Describe the Pod

```bash
kubectl describe pod nginx-pod
```

**Expected output (abbreviated):**
```
Name:             nginx-pod
Namespace:        default
Priority:         0
Node:             minikube/192.168.49.2
Start Time:       Sat, 14 Feb 2026 10:00:00 +0530
Labels:           app=nginx
                  environment=dev
                  tier=frontend
Status:           Running
IP:               10.244.0.3
Containers:
  nginx-container:
    Image:          nginx:latest
    Port:           80/TCP
    State:          Running
      Started:      Sat, 14 Feb 2026 10:00:05 +0530
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        250m
      memory:     64Mi
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m    default-scheduler  Successfully assigned default/nginx-pod to minikube
  Normal  Pulling    2m    kubelet            Pulling image "nginx:latest"
  Normal  Pulled     1m    kubelet            Successfully pulled image "nginx:latest"
  Normal  Created    1m    kubelet            Created container nginx-container
  Normal  Started    1m    kubelet            Started container nginx-container
```

> The **Events** section at the bottom is crucial for debugging. It shows the complete lifecycle from scheduling → pulling image → starting container.

---

### STEP 9: Exec Into the Pod

```bash
kubectl exec -it nginx-pod -- /bin/sh
```

Once inside the container shell, run these commands to verify nginx:

```sh
# Check nginx process is running
ps aux | grep nginx

# View nginx default web page
cat /usr/share/nginx/html/index.html

# Check nginx config
cat /etc/nginx/nginx.conf

# Test local connectivity (curl should be available in nginx image)
curl http://localhost:80

# Check listening ports
netstat -tlnp   # or: ss -tlnp

# Exit the shell
exit
```

**Expected output from `curl localhost`:**
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>
```

---

### STEP 10: Access nginx from Your Host (Optional)

```bash
# Port-forward pod port 80 to localhost:8080
kubectl port-forward pod/nginx-pod 8080:80
```

Now open your browser and visit: `http://localhost:8080`

You should see the **"Welcome to nginx!"** page.

---

### STEP 11: Delete and Recreate the Pod

**Delete the pod:**
```bash
kubectl delete pod nginx-pod
```

**Expected output:**
```
pod "nginx-pod" deleted
```

**Verify it's gone:**
```bash
kubectl get pods
```

**Expected output:**
```
No resources found in default namespace.
```

**Recreate instantly from YAML:**
```bash
kubectl apply -f pod.yml
```

**Expected output:**
```
pod/nginx-pod created
```

> This demonstrates the power of **declarative configuration** — your pod.yml is the single source of truth. You can always recreate the exact same environment by reapplying the YAML.

---

### STEP 12: Clean Up

```bash
# Delete the pod
kubectl delete pod nginx-pod

# Stop Minikube (preserves cluster state)
minikube stop

# Delete Minikube cluster entirely (if done)
minikube delete
```

---


## Project Structure

```
task-15-kubernetes-setup-pod-deployment
├── pod.yml
├── README.md
└── screenshots
    ├── Delete and recreate the Pod to observe.png
    ├── Exec into the Pod using kubectl exec -it.png
    ├── kubectl apply -f pod.yml and confirm that the Pod is created.png
    ├── kubectl get pods -o wide and kubectl describe pod.png
    ├── kubectl get pods -o wide and use kubectl describe pod.png
    ├── live nginx .png
    ├── minikube kubectl installation and verification.png
    ├── pod creation.png
    ├── Screenshot From 2026-02-15 15-36-44.png
    ├── Screenshot From 2026-02-15 15-37-42.png
    ├── Screenshot From 2026-02-15 15-46-48.png
    ├── Screenshot From 2026-02-15 15-47-07.png
    └── verify nginx files and\012connectivity internally.png
```

---

## Tools Used
- **Minikube** v1.32+ — Local Kubernetes cluster
- **kubectl** v1.29+ — Kubernetes CLI
- **Docker** — Container runtime / Minikube driver
- **nginx:latest** — Container image used in the Pod
