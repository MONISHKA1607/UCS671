# Embedded Vision EST Notes -- Kubernetes

# 1. Kubernetes

Kubernetes (K8s) is a container orchestration platform used to deploy,
manage, scale and recover containerized applications.

## Advantages

-   Automatic scaling
-   Load balancing
-   Self-healing
-   Automatic restart of failed containers
-   Pod rescheduling on node failure
-   Rolling updates and rollback
-   Desired state management
-   Resource isolation
-   Secure configuration using Secrets/ConfigMaps
-   Portability across platforms

------------------------------------------------------------------------

# 2. Kubernetes Architecture

``` text
Client (kubectl)
      |
      v
 Master/Server Node
      |
 -------------------------
 |           |           |
Worker1    Worker2    Worker3
(Pods)     (Pods)     (Pods)
```

-   **Server/Master**: Controls the cluster.
-   **Agent/Worker**: Runs Pods.
-   **Client**: Uses kubectl to manage the cluster.

------------------------------------------------------------------------

# 3. Jetson Cluster Setup

## Step 1 -- Configure Swap

``` bash
git clone https://github.com/JetsonHacksNano/resizeSwapMemory.git
cd resizeSwapMemory
./setSwapMemorySize.sh -g 8
```

## Step 2 -- Disable IPv6

``` bash
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1
```

## Step 3 -- Install K3s Server

``` bash
mkdir $HOME/.kube/
curl -sfL https://get.k3s.io | sh -s - --docker --write-kubeconfig-mode 644 --write-kubeconfig $HOME/.kube/config
```

## Step 4 -- Verify Nodes

``` bash
kubectl get nodes
```

## Step 5 -- Join Agent Node

``` bash
curl -sfL https://get.k3s.io | K3S_URL=https://<SERVER_IP>:6443 K3S_TOKEN=<NODE_TOKEN> sh -
```

Get token:

``` bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

## Step 6 -- Configure Client

Copy kubeconfig and install kubectl.

------------------------------------------------------------------------

# 4. Pod

Smallest deployable Kubernetes unit.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: torch
spec:
  containers:
  - name: torchtest
    image: nvcr.io/nvidia/l4t-pytorch:r32.7.1-pth1.10-py3
```

Commands:

``` bash
kubectl apply -f test_k3s.yaml
kubectl exec -it torch -- python3
kubectl delete pod torch
```

Use Pod YAML only for: - Testing - Debugging - Single temporary
container

------------------------------------------------------------------------

# 5. Deployment

Deployment manages Pods automatically.

``` text
Deployment
    ↓
Creates Pods
    ↓
Runs Containers
```

Important fields: - replicas - selector - template - resources - image -
ports

``` yaml
replicas: 5
```

Guarantees five Pods.

## requests vs limits

requests: - Minimum guaranteed resources.

limits: - Maximum allowed resources.

------------------------------------------------------------------------

# 6. Service

Purpose: - Stable networking - External access - Load balancing

``` text
User
 ↓
Service
 ↓
Deployment
 ↓
Pods
```

Example:

``` yaml
kind: Service
spec:
  selector:
    app: tensorflow
  ports:
  - port: 80
    targetPort: 5000
  type: NodePort
```

NodePort exposes application externally.

------------------------------------------------------------------------

# 7. Horizontal Pod Autoscaler (HPA)

Purpose: - Automatic scaling

``` yaml
minReplicas: 5
maxReplicas: 10
averageUtilization: 50
```

If CPU \> 50%, Kubernetes creates more Pods.

Flow:

``` text
High CPU
   ↓
HPA
   ↓
Deployment
   ↓
More Pods
```

------------------------------------------------------------------------

# 8. Relationship

``` text
Deployment
     ↓
Creates Pods
     ↓
Service exposes Pods
     ↓
HPA scales Deployment
```

Remember:

-   Deployment creates Pods.
-   Service exposes Pods.
-   HPA scales Deployment.

------------------------------------------------------------------------

# 9. Which YAML to Use?

## Pod

Use when: - Single container - Testing - Debugging

## Deployment

Use when: - Production application - Multiple replicas - Self-healing -
Scaling

## Service

Use when: - Users need network access - Stable IP - Load balancing

## HPA

Use when: - Dynamic workload - CPU-based scaling - Multi-user
applications

------------------------------------------------------------------------

# 10. Exam Approach

For application questions:

1.  Prepare Docker image.
2.  Create Deployment.
3.  Set replicas/resources.
4.  Create Service (NodePort).
5.  Configure HPA if scaling required.
6.  Apply YAMLs.
7.  Verify using:

``` bash
kubectl get pods
kubectl get svc
kubectl get hpa
```

------------------------------------------------------------------------

# 11. Common Exam Scenarios

## "Support 10 users simultaneously"

-   Deployment
-   Service
-   HPA

## "Limit CPU per user"

Use resource limits.

## "Application accessible externally"

Use NodePort Service.

## "At least 5 instances always running"

-   Deployment with `replicas: 5`
-   HPA with `minReplicas: 5`

------------------------------------------------------------------------

# 12. One-Line Memory Tricks

``` text
Pod = Runs Container

Deployment = Creates & Manages Pods

Service = Exposes Pods

HPA = Scales Deployment
```

``` text
Docker
   ↓
Deployment
   ↓
Pods
   ↓
Service
   ↓
Users
```

