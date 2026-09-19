---
tags: [cloud, containerisation, kubernetes]
type: cheatsheet
source: personal Notion — "Kubernetes: Complete Guide from Zero to Production"
last-verified: 2026-08-25
---

# Kubernetes

## Up
- [[Containerisation]]

## Subtopics
- [[Karpenter]] — node autoscaler (right-sizes & consolidates EC2 nodes)
- [[K9s]] — terminal UI for managing clusters
- [[Helm]] — package manager for Kubernetes (charts & releases)
- [[Service Mesh]] — service-to-service traffic, mTLS & observability (Istio, Linkerd)
- [[Autoscaling]] — HPA, VPA & KEDA (pod/event-driven scaling)
- [[Kustomize]] — template-free config management (overlays/patches)
- [[Ingress Controllers]] — HTTP routing & TLS (NGINX, Traefik, Gateway API)
- [[cert-manager]] — automated TLS certificates (Let's Encrypt/ACME)
- [[Kyverno & OPA Gatekeeper]] — admission-control policy enforcement
- [[CNI]] — pod networking & NetworkPolicy (Calico, Cilium)
- [[Velero]] — backup, restore & migration
- [[Operators & CRDs]] — extend the API + automate day-2 ops
- [[Falco]] — runtime threat detection (in [[Container & K8s Security]])

Complete Kubernetes guide/cheatsheet imported from my Notion — architecture, kubectl, YAML manifests, deployment & scaling, networking, storage, monitoring, Helm, and Docker Compose→K8s migration.

**For developers with container knowledge but zero Kubernetes experience**
---
### 📚 Table of Contents
1. Introduction to Kubernetes
2. Kubernetes Architecture
3. Core Concepts Explained
4. Installation Methods
5. kubectl - Command-Line Tool
6. Working with YAML Manifests
7. Application Deployment & Management
8. Networking in Kubernetes
9. Storage in Kubernetes
10. Monitoring and Logging
11. Package Management with Helm
12. Docker Compose vs Kubernetes
---
## 1. Introduction to Kubernetes
### What is Kubernetes?
Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.
**Think of it this way:** If Docker is like having a single shipping container, Kubernetes is the entire shipping port with cranes, trucks, and logistics systems that manage thousands of containers efficiently.
### Why Kubernetes?
**Key Benefits:**
- **Automatic Scaling:** Scale applications up or down based on demand
- **Self-Healing:** Automatically restarts failed containers
- **Load Balancing:** Distributes traffic across containers
- **Rolling Updates:** Deploy new versions without downtime
- **Declarative Configuration:** Define desired state, K8s makes it happen
- **Portability:** Run anywhere - local, cloud, hybrid
**When You Need Kubernetes:**
- Running microservices architecture
- Need high availability and fault tolerance
- Managing multiple containers across multiple hosts
- Require automated deployment and scaling
- Production workloads requiring orchestration
---
## 2. Kubernetes Architecture
### High-Level Overview
Kubernetes uses a **master-worker architecture** (now called Control Plane and Nodes).
```javascript
┌─────────────────────────────────────┐
│       Control Plane (Master)        │
│  ┌──────────┬──────────┬─────────┐ │
│  │ API      │ Scheduler│ Controller│
│  │ Server   │          │ Manager  │ │
│  └──────────┴──────────┴─────────┘ │
│  ┌──────────────────────────────┐  │
│  │          etcd                 │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘
             │
             │ (manages)
             ▼
┌─────────────────────────────────────┐
│              Nodes                   │
│  ┌────────────────────────────────┐ │
│  │ Node 1  │ Node 2  │ Node 3     │ │
│  │ Kubelet │ Kubelet │ Kubelet    │ │
│  │ Pods    │ Pods    │ Pods       │ │
│  └────────────────────────────────┘ │
└─────────────────────────────────────┘
```
### Control Plane Components
#### 1. API Server
- **What:** Front-end for the Kubernetes control plane
- **Function:** Exposes Kubernetes API, handles all REST operations
- **Analogy:** The reception desk at a hotel - all requests go through here
#### 2. etcd
- **What:** Distributed key-value store
- **Function:** Stores all cluster data (configuration, state, metadata)
- **Analogy:** The hotel's database with all room bookings and guest info
#### 3. Scheduler
- **What:** Component that assigns Pods to Nodes
- **Function:** Watches for newly created Pods and selects optimal Node
- **Analogy:** The hotel manager deciding which room to assign guests
#### 4. Controller Manager
- **What:** Runs controller processes
- **Function:** Monitors cluster state and makes changes to match desired state
- **Analogy:** The maintenance team ensuring everything runs as expected
### Node Components
#### 1. Kubelet
- **What:** Agent running on each node
- **Function:** Ensures containers are running in Pods
- **Analogy:** Room service staff ensuring your room has everything
#### 2. Kube-proxy
- **What:** Network proxy running on each node
- **Function:** Maintains network rules, enables communication
- **Analogy:** The hotel's phone system routing calls
#### 3. Container Runtime
- **What:** Software responsible for running containers
- **Function:** Docker, containerd, or CRI-O
- **Analogy:** The actual room where guests stay
---
## 3. Core Concepts Explained
### Pods
**The smallest deployable unit in Kubernetes.**
- A Pod wraps one or more containers
- Containers in a Pod share network and storage
- Pods are ephemeral (temporary)
**Analogy:** A Pod is like a hotel room - it can have one bed (container) or multiple beds, sharing the same bathroom and amenities.
**When to use:**
- Single container: Most common pattern
- Multiple containers: When containers must work together (sidecar pattern)
### Deployments
**Manages stateless applications.**
- Declares desired number of Pod replicas
- Handles rolling updates and rollbacks
- Ensures specified number of Pods are always running
**Key Features:**
- Automatic replacement of failed Pods
- Easy scaling (increase/decrease replicas)
- Zero-downtime updates
### ReplicaSets
**Ensures a specified number of Pod replicas are running.**
- Usually managed by Deployments
- Maintains stable set of Pods
- Rarely created directly (use Deployments instead)
### Services
**Network abstraction that exposes Pods.**
**Problem it solves:** Pods have dynamic IPs and can be replaced. Services provide a stable endpoint.
**Types:**
1. **ClusterIP** (default)
\t- Internal access only
\t- Use for: Internal microservices communication
2. **NodePort**
\t- Exposes service on each Node's IP at a static port
\t- Use for: Development/testing external access
3. **LoadBalancer**
\t- Exposes service externally using cloud provider's load balancer
\t- Use for: Production external access
4. **ExternalName**
\t- Maps service to external DNS name
\t- Use for: Accessing external services
### Namespaces
**Logical separation of resources.**
- Divide cluster resources between multiple users/teams
- Provide scope for names
- Enable resource quotas and access control
**Default Namespaces:**
- `default`: Default namespace for resources
- `kube-system`: Kubernetes system components
- `kube-public`: Public resources
- `kube-node-lease`: Node heartbeat data
**When to use:**
- Multiple teams/projects sharing a cluster
- Environment separation (dev, staging, prod)
- Resource isolation and quotas
### ConfigMaps
**Store non-sensitive configuration data.**
- Key-value pairs
- Environment variables
- Configuration files
- Command-line arguments
**Use cases:**
- Application configuration
- Environment-specific settings
- Feature flags
### Secrets
**Store sensitive data (passwords, tokens, keys).**
- Base64 encoded (not encrypted by default)
- Can be encrypted at rest with additional configuration
- Mounted as files or environment variables
**Types:**
- `Opaque`: Arbitrary user-defined data
- `kubernetes.io/dockerconfigjson`: Docker registry credentials
- `kubernetes.io/tls`: TLS certificate and key
### Volumes
**Persistent storage for Pods.**
**Problem:** Container storage is ephemeral (lost when container stops)
**Solution:** Volumes persist data beyond Pod lifecycle
**Types:**
- `emptyDir`: Temporary, exists as long as Pod exists
- `hostPath`: Mounts file/directory from host node
- `persistentVolumeClaim`: Claims storage from PersistentVolume
- Cloud provider volumes: AWS EBS, GCE PD, Azure Disk
### Ingress
**Manages external HTTP/HTTPS access to services.**
- Layer 7 load balancing
- SSL/TLS termination
- Name-based virtual hosting
- Path-based routing
**Requires:** Ingress Controller (nginx, traefik, etc.)
**Use cases:**
- Single entry point for multiple services
- SSL certificate management
- Path-based routing (`/api` → API service, `/web` → Web service)
### StatefulSets
**Manages stateful applications.**
**Difference from Deployments:**
- Stable, unique network identifiers
- Stable, persistent storage
- Ordered deployment and scaling
- Ordered, automated rolling updates
**Use cases:**
- Databases (MySQL, PostgreSQL, MongoDB)
- Message queues (Kafka, RabbitMQ)
- Distributed systems requiring stable identities
### DaemonSets
**Ensures a copy of a Pod runs on all (or some) Nodes.**
- Automatically adds Pods to new Nodes
- Removes Pods when Nodes are removed
**Use cases:**
- Logging agents (Fluentd, Logstash)
- Monitoring agents (Prometheus Node Exporter)
- Network plugins
- Storage daemons
---
## 4. Kubernetes Installation
### Local Development Options
#### Minikube
**Single-node cluster for local development.**
**Prerequisites:**
- 2+ CPUs
- 2GB+ RAM
- Container runtime (Docker, containerd)
- Hypervisor (VirtualBox, HyperKit, KVM)
**Installation:**
```bash
# macOS
brew install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Windows
choco install minikube
```
**Start cluster:**
```bash
minikube start
minikube status
minikube dashboard  # Web UI
```
#### Kind (Kubernetes in Docker)
**Runs Kubernetes clusters in Docker containers.**
**Installation:**
```bash
# macOS/Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create cluster
kind create cluster --name my-cluster
```
#### Docker Desktop
**Built-in Kubernetes support.**
**Steps:**
1. Install Docker Desktop
2. Settings → Kubernetes → Enable Kubernetes
3. Apply & Restart
**Pros:**
- Easy setup
- Integrated with Docker
**Cons:**
- Resource intensive
- Single-node only
### Production Deployments
#### kubeadm
**Official tool for bootstrapping Kubernetes clusters.**
**Prerequisites:**
- Linux hosts (Ubuntu, CentOS, etc.)
- 2GB+ RAM per machine
- Network connectivity between machines
- Unique hostname, MAC, product_uuid
**Basic Setup:**
```bash
# 1. Install container runtime (containerd)
sudo apt-get update
sudo apt-get install -y containerd

# 2. Install kubeadm, kubelet, kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# 3. Initialize control plane (on master node)
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# 4. Configure kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 5. Install network plugin (Calico example)
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# 6. Join worker nodes (run on worker nodes)
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
#### Managed Kubernetes Services
**Cloud providers handle control plane management.**
##### Google Kubernetes Engine (GKE)
```bash
# Install gcloud CLI
# Create cluster
gcloud container clusters create my-cluster --num-nodes=3

# Get credentials
gcloud container clusters get-credentials my-cluster
```
##### Amazon Elastic Kubernetes Service (EKS)
```bash
# Install eksctl
# Create cluster
eksctl create cluster --name my-cluster --region us-west-2 --nodes 3
```
##### Azure Kubernetes Service (AKS)
```bash
# Install Azure CLI
# Create cluster
az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 3

# Get credentials
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```
**Benefits of Managed Services:**
- No control plane management
- Automatic upgrades
- Integrated monitoring
- Enterprise support
- High availability built-in
---
## 5. kubectl - The Command-Line Tool
### Installation
```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Windows
choco install kubernetes-cli
```
### Essential Commands
#### Cluster Information
```bash
# Check cluster info
kubectl cluster-info

# View nodes
kubectl get nodes

# Detailed node info
kubectl describe node <node-name>
```
#### Working with Resources
```bash
# Get resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get all  # All resource types

# Get resources in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A  # Short form

# Get resources in specific namespace
kubectl get pods -n kube-system

# Watch resources (auto-refresh)
kubectl get pods --watch
kubectl get pods -w  # Short form
```
#### Creating Resources
```bash
# Create from YAML file
kubectl apply -f deployment.yaml

# Create from directory
kubectl apply -f ./configs/

# Create from URL
kubectl apply -f https://example.com/deployment.yaml

# Imperative creation (quick testing)
kubectl create deployment nginx --image=nginx
kubectl run nginx --image=nginx
```
#### Updating Resources
```bash
# Apply changes from YAML
kubectl apply -f deployment.yaml

# Edit resource directly
kubectl edit deployment nginx

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Set image (rolling update)
kubectl set image deployment/nginx nginx=nginx:1.21
```
#### Deleting Resources
```bash
# Delete from YAML
kubectl delete -f deployment.yaml

# Delete by name
kubectl delete deployment nginx
kubectl delete pod nginx-abc123

# Delete all pods in namespace
kubectl delete pods --all

# Force delete (use with caution)
kubectl delete pod nginx --force --grace-period=0
```
#### Inspecting Resources
```bash
# Detailed information
kubectl describe pod nginx-abc123
kubectl describe deployment nginx

# Get YAML/JSON output
kubectl get pod nginx-abc123 -o yaml
kubectl get pod nginx-abc123 -o json

# Custom output columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
```
#### Logs and Debugging
```bash
# View logs
kubectl logs nginx-abc123

# Follow logs (stream)
kubectl logs -f nginx-abc123

# Logs from specific container in multi-container pod
kubectl logs nginx-abc123 -c nginx-container

# Previous container logs (after crash)
kubectl logs nginx-abc123 --previous

# Logs from all pods with label
kubectl logs -l app=nginx
```
#### Executing Commands in Containers
```bash
# Execute command
kubectl exec nginx-abc123 -- ls /app

# Interactive shell
kubectl exec -it nginx-abc123 -- /bin/bash

# Specific container in multi-container pod
kubectl exec -it nginx-abc123 -c nginx-container -- /bin/bash
```
#### Port Forwarding
```bash
# Forward local port to pod
kubectl port-forward pod/nginx-abc123 8080:80

# Forward to service
kubectl port-forward service/nginx 8080:80

# Forward to deployment
kubectl port-forward deployment/nginx 8080:80
```
#### Context and Namespaces
```bash
# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context minikube

# Set default namespace
kubectl config set-context --current --namespace=dev

# Create namespace
kubectl create namespace dev
```
#### Useful Shortcuts
```bash
# Abbreviations
kubectl get po     # pods
kubectl get svc    # services
kubectl get deploy # deployments
kubectl get ns     # namespaces
kubectl get no     # nodes

# Alias (add to ~/.bashrc or ~/.zshrc)
alias k=kubectl
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
```
---
## 6. Working with YAML Manifests
### YAML Structure Basics
All Kubernetes YAML files share this structure:
```yaml
apiVersion: <api-version>  # API group and version
kind: <resource-type>      # Type of resource
metadata:                  # Resource metadata
  name: <name>
  labels:
    key: value
spec:                      # Resource specification
  # Resource-specific configuration
```
### Best Practices
1. **Use declarative configuration** - Define desired state, not imperative commands
2. **Version control** - Store YAML in Git
3. **Validate before applying** - Use `kubectl apply --dry-run=client`
4. **Use labels consistently** - For organization and selection
5. **Add comments** - Explain complex configurations
6. **Separate concerns** - One file per resource type or logical grouping
7. **Use namespaces** - Isolate environments
8. **Resource limits** - Always specify CPU/memory limits
### Example: Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: production
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
      protocol: TCP
    resources:
      requests:        # Minimum resources required
        memory: "64Mi"
        cpu: "250m"
      limits:          # Maximum resources allowed
        memory: "128Mi"
        cpu: "500m"
    env:               # Environment variables
    - name: ENV_VAR_NAME
      value: "value"
    volumeMounts:      # Mount volumes
    - name: config-volume
      mountPath: /etc/config
  volumes:             # Define volumes
  - name: config-volume
    configMap:
      name: app-config
```
### Example: Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                    # Number of pod replicas
  selector:
    matchLabels:
      app: nginx                 # Must match template labels
  template:                      # Pod template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        livenessProbe:           # Health check - restart if fails
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:          # Traffic readiness - remove from service if fails
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
  strategy:                      # Update strategy
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1                # Max pods above desired during update
      maxUnavailable: 0          # Max pods unavailable during update
```
### Example: Service (ClusterIP)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP              # Internal cluster access only
  selector:
    app: nginx                 # Selects pods with this label
  ports:
  - protocol: TCP
    port: 80                   # Service port (cluster-internal)
    targetPort: 80             # Container port
  sessionAffinity: None        # or ClientIP for sticky sessions
```
### Example: Service (NodePort)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort               # Exposes on each node's IP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80                   # Service port
    targetPort: 80             # Container port
    nodePort: 30080            # External port on nodes (30000-32767)
```
### Example: Service (LoadBalancer)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer           # Cloud provider load balancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  loadBalancerIP: 1.2.3.4      # Optional: Request specific IP
```
### Example: Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"  # For SSL
spec:
  ingressClassName: nginx      # Ingress controller class
  tls:                         # SSL/TLS configuration
  - hosts:
    - example.com
    secretName: example-tls    # Secret containing certificate
  rules:
  - host: example.com          # Domain name
    http:
      paths:
      - path: /                # URL path
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
      - path: /api             # Different path to different service
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```
### Example: ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Key-value pairs
  database_url: "postgres://localhost:5432/mydb"
  log_level: "info"
  
  # Configuration file
  app.properties: |
    server.port=8080
    server.timeout=30
    cache.enabled=true
  
  # JSON configuration
  config.json: |
    {
      "apiKey": "abc123",
      "timeout": 30
    }
```
**Using ConfigMap in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    # Use specific key as environment variable
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    # Load all keys as environment variables
    envFrom:
    - configMapRef:
        name: app-config
    # Mount as file
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```
### Example: Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque                   # Generic secret
data:
  # Values must be base64 encoded
  username: YWRtaW4=           # "admin" in base64
  password: cGFzc3dvcmQxMjM=   # "password123" in base64

---
# Alternatively, use stringData (auto-encodes)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials-plain
type: Opaque
stringData:                    # Will be auto-encoded
  username: admin
  password: password123
```
**Create secret from command line:**
```bash
# From literal values
kubectl create secret generic db-credentials \\
  --from-literal=username=admin \\
  --from-literal=password=password123

# From files
kubectl create secret generic tls-secret \\
  --from-file=tls.crt=path/to/cert \\
  --from-file=tls.key=path/to/key

# Docker registry secret
kubectl create secret docker-registry regcred \\
  --docker-server=https://index.docker.io/v1/ \\
  --docker-username=myuser \\
  --docker-password=mypass \\
  --docker-email=myemail@example.com
```
**Using Secret in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    # Use specific secret key
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
    # Mount as files
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
```
### Example: PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
  - ReadWriteOnce              # Single node read-write
  # - ReadOnlyMany             # Multiple nodes read-only
  # - ReadWriteMany            # Multiple nodes read-write
  resources:
    requests:
      storage: 10Gi            # Requested storage size
  storageClassName: standard   # Storage class (optional)
```
**Using PVC in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
  - name: mysql
    image: mysql:8.0
    volumeMounts:
    - name: mysql-storage
      mountPath: /var/lib/mysql
  volumes:
  - name: mysql-storage
    persistentVolumeClaim:
      claimName: mysql-pvc
```
### Complete Application Example
**File: app-deployment.yaml**
```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: myapp

---
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: myapp
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"

---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: myapp
type: Opaque
stringData:
  DB_PASSWORD: "secure-password"
  API_KEY: "secret-api-key"

---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DB_PASSWORD
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: myapp
spec:
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```
**Apply the configuration:**
```bash
kubectl apply -f app-deployment.yaml
```
---
## 7. Application Deployment & Management
### Deploying an Application
#### Method 1: Using kubectl create (Imperative)
```bash
# Create deployment
kubectl create deployment nginx --image=nginx:1.21 --replicas=3

# Expose as service
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Check status
kubectl get deployments
kubectl get pods
kubectl get services
```
#### Method 2: Using YAML (Declarative - Recommended)
**deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```
**Apply:**
```bash
kubectl apply -f deployment.yaml
```
### Scaling Applications
#### Manual Scaling
```bash
# Scale using kubectl
kubectl scale deployment nginx --replicas=5

# Update YAML and apply
# Edit replicas: 5 in deployment.yaml
kubectl apply -f deployment.yaml

# Check scaling progress
kubectl get deployments
kubectl get pods --watch
```
#### Autoscaling (Horizontal Pod Autoscaler)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Scale when CPU > 70%
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80  # Scale when memory > 80%
```
**Create using kubectl:**
```bash
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=70

# Check HPA status
kubectl get hpa
kubectl describe hpa nginx
```
### Rolling Updates
**How it works:**
1. Creates new pods with updated version
2. Waits for new pods to be ready
3. Terminates old pods
4. Repeats until all pods updated
#### Perform Rolling Update
```bash
# Update image
kubectl set image deployment/nginx nginx=nginx:1.22

# Or update YAML and apply
kubectl apply -f deployment.yaml

# Watch rollout
kubectl rollout status deployment/nginx

# View rollout history
kubectl rollout history deployment/nginx
```
#### Update Strategy Configuration
```yaml
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # Max 2 extra pods during update
      maxUnavailable: 1  # Max 1 pod can be unavailable
```
**Strategy Types:**
- `RollingUpdate`: Gradual replacement (default)
- `Recreate`: Delete all old pods, then create new ones (downtime)
### Rollbacks
```bash
# Rollback to previous version
kubectl rollout undo deployment/nginx

# Rollback to specific revision
kubectl rollout undo deployment/nginx --to-revision=2

# View revision history
kubectl rollout history deployment/nginx

# View specific revision details
kubectl rollout history deployment/nginx --revision=2

# Pause rollout (useful for canary deployments)
kubectl rollout pause deployment/nginx

# Resume rollout
kubectl rollout resume deployment/nginx
```
### Health Checks
#### Liveness Probe
**Purpose:** Detect if container is alive. Restarts container if check fails.
```yaml
livenessProbe:
  httpGet:                  # HTTP check
    path: /health
    port: 8080
  initialDelaySeconds: 30   # Wait before first check
  periodSeconds: 10         # Check every 10 seconds
  timeoutSeconds: 5         # Timeout for each check
  failureThreshold: 3       # Restart after 3 failures
  successThreshold: 1       # Consider healthy after 1 success
```
**Probe Types:**
```yaml
# HTTP GET
livenessProbe:
  httpGet:
    path: /health
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: value

# TCP Socket
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20

# Command Execution
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```
#### Readiness Probe
**Purpose:** Detect if container is ready to serve traffic. Removes from service endpoints if check fails.
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
  successThreshold: 1
```
**Use Cases:**
- Liveness: Restart crashed/deadlocked containers
- Readiness: Remove slow-starting containers from load balancer
#### Startup Probe
**Purpose:** Give slow-starting containers time to start before liveness checks begin.
```yaml
startupProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 30      # 30 * 10 = 300 seconds (5 minutes) to start
  periodSeconds: 10
```
### Resource Management
#### Resource Requests and Limits
```yaml
resources:
  requests:                 # Guaranteed resources
    memory: "64Mi"          # Minimum memory
    cpu: "250m"             # Minimum CPU (0.25 cores)
  limits:                   # Maximum allowed
    memory: "128Mi"         # Max memory (OOMKilled if exceeded)
    cpu: "500m"             # Max CPU (throttled if exceeded)
```
**CPU Units:**
- `1` = 1 CPU core
- `1000m` = 1 CPU core (millicores)
- `500m` = 0.5 CPU cores
- `100m` = 0.1 CPU cores
**Memory Units:**
- `Ki` = Kibibytes (1024 bytes)
- `Mi` = Mebibytes (1024 Ki)
- `Gi` = Gibibytes (1024 Mi)
- `Ti` = Tebibytes (1024 Gi)
**Best Practices:**
- Always set requests (for scheduling)
- Set limits to prevent resource hogging
- requests ≤ limits
- Monitor actual usage and adjust
---
## 8. Networking in Kubernetes
### Cluster Networking Model
**Fundamental Rules:**
1. All Pods can communicate with all other Pods without NAT
2. All Nodes can communicate with all Pods without NAT
3. The IP a Pod sees itself as is the same IP others see it as
**Network Types:**
- **Pod Network:** IP range for Pod IPs
- **Service Network:** IP range for Service IPs (virtual)
- **Node Network:** IP range for Node IPs
### Service Discovery
#### DNS-Based Discovery
Kubernetes provides automatic DNS for services.
**Service DNS Format:**
```javascript
<service-name>.<namespace>.svc.cluster.local
```
**Examples:**
```bash
# Within same namespace
curl http://nginx-service

# From different namespace
curl http://nginx-service.default.svc.cluster.local

# Short form (different namespace)
curl http://nginx-service.default
```
#### Environment Variables
Kubernetes injects service info as environment variables.
```bash
# For service "nginx-service" on port 80
NGINX_SERVICE_SERVICE_HOST=10.0.0.1
NGINX_SERVICE_SERVICE_PORT=80
NGINX_SERVICE_PORT=tcp://10.0.0.1:80
NGINX_SERVICE_PORT_80_TCP=tcp://10.0.0.1:80
NGINX_SERVICE_PORT_80_TCP_PROTO=tcp
NGINX_SERVICE_PORT_80_TCP_PORT=80
NGINX_SERVICE_PORT_80_TCP_ADDR=10.0.0.1
```
### Network Policies
**Control traffic flow between Pods.**
**Default Behavior:**
- All Pods can communicate with all Pods (no restrictions)
**With Network Policies:**
- Whitelist allowed traffic
- Deny by default (after policy is applied)
**Example: Allow Only from Frontend to Backend**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend           # Apply to pods with app=backend
  policyTypes:
  - Ingress                  # Control incoming traffic
  - Egress                   # Control outgoing traffic
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend      # Only allow from pods with app=frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database      # Only allow to database
    ports:
    - protocol: TCP
      port: 5432
  - to:                      # Allow DNS
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```
**Example: Deny All Ingress**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}            # Empty selector = all pods
  policyTypes:
  - Ingress
```
**Example: Allow All Egress**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - {}                       # Empty rule = allow all
```
**Requirements:**
- Network plugin must support NetworkPolicy (Calico, Weave, Cilium)
- Default plugins (kubenet, Flannel) don't support NetworkPolicy
---
## 9. Storage in Kubernetes
### Volumes Overview
**Problem:** Container storage is ephemeral (deleted with container).
**Solution:** Kubernetes Volumes persist data.
### Volume Types
#### emptyDir
**Temporary storage, exists as long as Pod exists.**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: container-1
    image: busybox
    volumeMounts:
    - name: shared-data
      mountPath: /data
  - name: container-2
    image: busybox
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}              # Empty directory created on disk
```
**Use Cases:**
- Scratch space
- Sharing data between containers in same Pod
- Cache that doesn't need to persist
#### hostPath
**Mounts file/directory from host node.**
```yaml
volumes:
- name: host-volume
  hostPath:
    path: /data              # Path on host
    type: DirectoryOrCreate  # Create if doesn't exist
```
**Use Cases:**
- Access node's filesystem
- Development/testing
- Running cAdvisor (needs access to /sys)
**Warning:** Not portable across nodes. Pod may fail if scheduled on different node.
### Persistent Volumes (PV)
**Cluster-level storage resource.**
**Lifecycle independent of Pods.**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce            # Single node read-write
  persistentVolumeReclaimPolicy: Retain  # Keep data after PVC deleted
  storageClassName: manual
  hostPath:
    path: /mnt/data
```
**Access Modes:**
- `ReadWriteOnce` (RWO): Single node read-write
- `ReadOnlyMany` (ROX): Multiple nodes read-only
- `ReadWriteMany` (RWX): Multiple nodes read-write
**Reclaim Policies:**
- `Retain`: Keep PV and data (manual cleanup)
- `Delete`: Delete PV and underlying storage
- `Recycle`: Delete data, reuse PV (deprecated)
### Persistent Volume Claims (PVC)
**Request for storage by user.**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi           # Request 5Gi
  storageClassName: manual
```
**How it works:**
1. User creates PVC requesting storage
2. Kubernetes finds suitable PV
3. Binds PVC to PV
4. Pod uses PVC
**Using PVC in Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pvc
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-data
```
### Storage Classes
**Enables dynamic provisioning of PVs.**
**Instead of:** Admin pre-creates PVs → User creates PVC → Bind
**With StorageClass:** User creates PVC → Kubernetes auto-creates PV → Bind
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs  # Cloud provider
parameters:
  type: gp3                  # SSD type
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer  # Delay binding until Pod scheduled
```
**Using StorageClass:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd  # Reference StorageClass
  resources:
    requests:
      storage: 10Gi
```
**Common Provisioners:**
- AWS: `kubernetes.io/aws-ebs`
- GCP: `kubernetes.io/gce-pd`
- Azure: `kubernetes.io/azure-disk`
- NFS: `kubernetes.io/nfs`
- Local: `kubernetes.io/no-provisioner`
### StatefulSet Storage
**Automatic PVC creation for each Pod.**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:      # Auto-create PVC for each Pod
  - metadata:
      name: data
    spec:
      accessModes:
      - ReadWriteOnce
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 10Gi
```
**Result:**
- Pod `mysql-0` gets PVC `data-mysql-0`
- Pod `mysql-1` gets PVC `data-mysql-1`
- Pod `mysql-2` gets PVC `data-mysql-2`
---
## 10. Monitoring and Logging
### Monitoring with Prometheus & Grafana
#### Prometheus
**Open-source monitoring and alerting system.**
**Features:**
- Time-series database
- Metrics collection
- PromQL query language
- Alerting
**Install using Helm:**
```bash
# Add Prometheus Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/prometheus

# Access Prometheus UI
kubectl port-forward -n default deploy/prometheus-server 9090
# Visit http://localhost:9090
```
**What Prometheus Monitors:**
- Node metrics (CPU, memory, disk)
- Pod/container metrics
- Kubernetes API metrics
- Application metrics (via exporters)
**Example PromQL Queries:**
```javascript
# CPU usage per pod
rate(container_cpu_usage_seconds_total[5m])

# Memory usage per pod
container_memory_usage_bytes

# Pod restarts
kube_pod_container_status_restarts_total

# HTTP request rate
rate(http_requests_total[5m])
```
#### Grafana
**Visualization and dashboards for Prometheus data.**
**Install using Helm:**
```bash
# Add Grafana Helm repo
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Grafana
helm install grafana grafana/grafana

# Get admin password
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Access Grafana UI
kubectl port-forward -n default deploy/grafana 3000
# Visit http://localhost:3000
# Username: admin, Password: (from above command)
```
**Add Prometheus Data Source:**
1. Configuration → Data Sources → Add data source
2. Select Prometheus
3. URL: `http://prometheus-server:80`
4. Save & Test
**Import Pre-built Dashboards:**
- Kubernetes Cluster Monitoring: Dashboard ID 6417
- Node Exporter: Dashboard ID 1860
- Pod Monitoring: Dashboard ID 6879
### Logging
#### Viewing Logs
```bash
# Single pod logs
kubectl logs pod-name

# Stream logs
kubectl logs -f pod-name

# Previous container logs (after crash)
kubectl logs pod-name --previous

# Specific container in multi-container pod
kubectl logs pod-name -c container-name

# All pods with label
kubectl logs -l app=nginx

# Last 100 lines
kubectl logs pod-name --tail=100

# Since timestamp
kubectl logs pod-name --since-time=2024-01-01T00:00:00Z

# Since duration
kubectl logs pod-name --since=1h
```
#### Centralized Logging (EFK Stack)
**Components:**
- **Elasticsearch**: Store and index logs
- **Fluentd**: Collect and forward logs
- **Kibana**: Visualize and search logs
**Architecture:**
```javascript
Pods → Fluentd (DaemonSet) → Elasticsearch → Kibana
```
**Install Elasticsearch:**
```bash
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch
```
**Install Kibana:**
```bash
helm install kibana elastic/kibana
```
**Install Fluentd:**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch-master"
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```
**Alternative: Loki + Promtail + Grafana**
**Advantages:**
- Lighter than EFK
- Integrated with Grafana
- Label-based indexing
```bash
# Install Loki stack
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack \\
  --set grafana.enabled=true \\
  --set prometheus.enabled=true
```
---
## 11. Package Management with Helm
### What is Helm?
**Kubernetes package manager.**
**Analogy:** Helm is to Kubernetes what apt/yum is to Linux or npm is to Node.js.
**Benefits:**
- Package complex applications
- Manage dependencies
- Version applications
- Templating for YAML files
- Easy rollbacks
- Share applications via charts
### Installation
```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows
choco install kubernetes-helm

# Verify installation
helm version
```
### Key Concepts
**Chart:**
- Package of Kubernetes resources
- Contains templates, values, metadata
- Like a .deb or .rpm package
**Repository:**
- Collection of charts
- Like apt repository or npm registry
**Release:**
- Instance of a chart running in cluster
- Can install same chart multiple times with different releases
### Basic Commands
#### Managing Repositories
```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable

# List repositories
helm repo list

# Update repository index
helm repo update

# Search for charts
helm search repo nginx
helm search repo wordpress

# Search Artifact Hub (public charts)
helm search hub wordpress
```
#### Installing Charts
```bash
# Install chart
helm install my-release bitnami/nginx

# Install with custom values
helm install my-release bitnami/nginx --set service.type=NodePort

# Install with values file
helm install my-release bitnami/nginx -f values.yaml

# Install in specific namespace
helm install my-release bitnami/nginx --namespace production --create-namespace

# Dry run (see what would be installed)
helm install my-release bitnami/nginx --dry-run --debug
```
#### Managing Releases
```bash
# List releases
helm list
helm list --all-namespaces

# Get release status
helm status my-release

# Get release values
helm get values my-release

# Get release manifest
helm get manifest my-release

# Upgrade release
helm upgrade my-release bitnami/nginx --set replicaCount=3

# Rollback release
helm rollback my-release
helm rollback my-release 1  # Rollback to revision 1

# View release history
helm history my-release

# Uninstall release
helm uninstall my-release

# Uninstall and keep history
helm uninstall my-release --keep-history
```
### Creating Charts
#### Create New Chart
```bash
# Create chart scaffold
helm create myapp

# Chart structure:
myapp/
  Chart.yaml          # Chart metadata
  values.yaml         # Default configuration values
  charts/             # Chart dependencies
  templates/          # Kubernetes manifest templates
    deployment.yaml
    service.yaml
    ingress.yaml
    _helpers.tpl      # Template helpers
```
#### Chart.yaml
```yaml
apiVersion: v2
name: myapp
description: A Helm chart for my application
type: application
version: 0.1.0          # Chart version
appVersion: "1.0"       # Application version
maintainers:
- name: Your Name
  email: you@example.com
```
#### values.yaml
```yaml
replicaCount: 2

image:
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: nginx
  hosts:
  - host: myapp.example.com
    paths:
    - path: /
      pathType: Prefix

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```
#### Template Example (deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 80
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
```
#### Install Custom Chart
```bash
# Package chart
helm package myapp/

# Install from directory
helm install my-release ./myapp

# Install from package
helm install my-release myapp-0.1.0.tgz
```
### Popular Charts
```bash
# MySQL
helm install mysql bitnami/mysql

# PostgreSQL
helm install postgres bitnami/postgresql

# Redis
helm install redis bitnami/redis

# MongoDB
helm install mongodb bitnami/mongodb

# Nginx Ingress Controller
helm install nginx-ingress ingress-nginx/ingress-nginx

# Prometheus + Grafana
helm install prometheus prometheus-community/kube-prometheus-stack

# WordPress
helm install wordpress bitnami/wordpress
```
---
## 12. Docker Compose vs Kubernetes
### Key Differences
| Aspect | Docker Compose | Kubernetes |
|---|---|---|
| **Purpose** | Local development, simple deployments | Production orchestration at scale |
| **Scope** | Single host | Multiple hosts (cluster) |
| **Configuration** | docker-compose.yml | Multiple YAML files (Deployments, Services, etc.) |
| **Scaling** | Manual, limited | Automatic, horizontal |
| **Load Balancing** | Basic | Advanced with Services, Ingress |
| **Self-Healing** | No | Yes (restarts failed containers) |
| **Rolling Updates** | No | Yes |
| **Service Discovery** | DNS in networks | Built-in DNS, Services |
| **Secrets Management** | Environment variables, files | Secrets API with encryption |
| **Storage** | Bind mounts, volumes | PersistentVolumes, StorageClasses |
| **Learning Curve** | Easy | Steep |
| **Complexity** | Simple | Complex |
| **Best For** | Development, testing | Production, enterprise |
### Architecture Comparison
#### Docker Compose Architecture
```javascript
┌─────────────────────────────────┐
│        Single Host              │
│                                 │
│  ┌──────────┐  ┌──────────┐   │
│  │Container1│  │Container2│   │
│  └──────────┘  └──────────┘   │
│  ┌──────────┐  ┌──────────┐   │
│  │Container3│  │Container4│   │
│  └──────────┘  └──────────┘   │
│                                 │
│     Docker Engine               │
└─────────────────────────────────┘
```
#### Kubernetes Architecture
```javascript
┌──────────────────────────────────────┐
│         Control Plane                │
│   (API, Scheduler, Controllers)      │
└──────────────────────────────────────┘
              │
              │ manages
              ▼
┌──────────────────────────────────────┐
│            Worker Nodes              │
│  ┌──────────┐  ┌──────────┐         │
│  │  Node 1  │  │  Node 2  │         │
│  │  Pods    │  │  Pods    │  ...    │
│  └──────────┘  └──────────┘         │
└──────────────────────────────────────┘
```
### Configuration Comparison
#### Docker Compose Example
**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:1.21
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    environment:
      - ENV=production
    depends_on:
      - db
    deploy:
      replicas: 3
    networks:
      - app-network

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secretpassword
      MYSQL_DATABASE: myapp
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
```
#### Kubernetes Equivalent
**web-deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        env:
        - name: ENV
          value: production
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html
        hostPath:
          path: /path/to/html
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```
**db-statefulset.yaml:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: db
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        - name: MYSQL_DATABASE
          value: myapp
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  clusterIP: None  # Headless service for StatefulSet
  selector:
    app: db
  ports:
  - port: 3306
---
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  password: secretpassword
```
### Use Cases
#### When to Use Docker Compose
✅ **Good for:**
1. **Local Development**
\t- Quick setup
\t- Easy to iterate
\t- Matches production architecture
2. **Simple Applications**
\t- Few services
\t- Single server deployment
\t- Low traffic
3. **CI/CD Testing**
\t- Integration tests
\t- End-to-end tests
\t- Reproducible environments
4. **Personal Projects**
\t- Blogs
\t- Small websites
\t- Hobby projects
5. **Prototyping**
\t- Proof of concept
\t- MVP development
\t- Demo applications
❌ **Not suitable for:**
- High availability requirements
- Auto-scaling needs
- Multi-host deployments
- Complex networking
- Enterprise production
#### When to Use Kubernetes
✅ **Good for:**
1. **Production Workloads**
\t- Mission-critical applications
\t- High availability required
\t- Zero-downtime deployments
2. **Microservices**
\t- Many services
\t- Service mesh
\t- Complex dependencies
3. **Scalable Applications**
\t- Traffic spikes
\t- Horizontal scaling
\t- Auto-scaling requirements
4. **Multi-Environment**
\t- Dev, staging, production
\t- Different configurations
\t- Resource quotas
5. **Enterprise**
\t- Compliance requirements
\t- Advanced monitoring
\t- RBAC and security
❌ **Overkill for:**
- Simple websites
- Single-server apps
- Personal projects
- Learning basic containerization
### Migration Path: Docker Compose → Kubernetes
#### Step 1: Understand Current Setup
```bash
# List services
docker-compose ps

# View configuration
docker-compose config
```
#### Step 2: Convert Services to Deployments
**For each service in docker-compose.yml:**
1. Create Deployment for stateless services
2. Create StatefulSet for stateful services (databases)
3. Map environment variables to ConfigMaps/Secrets
4. Convert volumes to PersistentVolumeClaims
#### Step 3: Create Services
**For each exposed port:**
1. Create ClusterIP Service for internal communication
2. Create LoadBalancer Service for external access
3. Or create Ingress for HTTP/HTTPS routing
#### Step 4: Handle Dependencies
**Docker Compose ****`depends_on`****:**
```yaml
web:
  depends_on:
    - db
```
**Kubernetes equivalent:**
Use `initContainers` to wait for dependencies:
```yaml
initContainers:
- name: wait-for-db
  image: busybox
  command: ['sh', '-c', 'until nc -z db 3306; do sleep 1; done']
```
#### Step 5: Test and Iterate
```bash
# Apply configurations
kubectl apply -f k8s/

# Monitor deployment
kubectl get pods -w

# Check logs
kubectl logs -f deployment/web

# Test connectivity
kubectl exec -it deployment/web -- curl http://db:3306
```
#### Step 6: Automate with Kompose
**Kompose** converts Docker Compose files to Kubernetes manifests.
```bash
# Install Kompose
curl -L https://github.com/kubernetes/kompose/releases/download/v1.31.2/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv kompose /usr/local/bin/

# Convert docker-compose.yml
kompose convert

# Review generated files
ls -l
# deployment.yaml, service.yaml, etc.

# Apply to cluster
kubectl apply -f .
```
**Note:** Kompose provides a starting point but may need manual adjustments.
### Best Practices for Both
#### Docker Compose
1. Use environment-specific files
\t```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
\t```
2. Don't store secrets in compose files
\t```yaml
environment:
  - DB_PASSWORD=${DB_PASSWORD}  # From .env file
\t```
3. Use health checks
\t```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 30s
  timeout: 10s
  retries: 3
\t```
#### Kubernetes
1. Use namespaces for separation
2. Always set resource limits
3. Implement health checks (liveness/readiness)
4. Use ConfigMaps/Secrets for configuration
5. Version control all YAML files
6. Use Helm for complex applications
7. Implement monitoring and logging
8. Use RBAC for security
9. Regular backups of etcd
10. Keep cluster updated
---
## 🎯 Quick Reference
### Essential kubectl Commands
```bash
# Cluster
kubectl cluster-info
kubectl get nodes

# Resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get all

# Create/Update
kubectl apply -f file.yaml
kubectl create deployment nginx --image=nginx

# Delete
kubectl delete -f file.yaml
kubectl delete pod nginx

# Inspect
kubectl describe pod nginx
kubectl logs nginx
kubectl logs -f nginx
kubectl exec -it nginx -- /bin/bash

# Scale
kubectl scale deployment nginx --replicas=3

# Rollout
kubectl rollout status deployment/nginx
kubectl rollout undo deployment/nginx

# Port Forward
kubectl port-forward pod/nginx 8080:80
```
### Common YAML Patterns
```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: app:1.0
        ports:
        - containerPort: 8080

# Service
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  selector:
    app: app
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```
### Troubleshooting Checklist
1. **Pod not starting**
\t```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
\t```
2. **Service not accessible**
\t```bash
kubectl get endpoints <service-name>
kubectl describe service <service-name>
\t```
3. **Image pull errors**
\t- Check image name
\t- Verify registry credentials
\t- Check imagePullSecrets
4. **Resource issues**
\t```bash
kubectl top nodes
kubectl top pods
\t```
5. **Network issues**
\t```bash
kubectl exec -it <pod> -- nc -zv <service> <port>
\t```
---
## 📚 Additional Resources
**Official Documentation:**
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [kubectl Reference](https://kubernetes.io/docs/reference/kubectl/)
- [Helm Documentation](https://helm.sh/docs/)
**Interactive Learning:**
- [Katacoda Kubernetes](https://www.katacoda.com/courses/kubernetes)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [Kubernetes Tutorials](https://kubernetes.io/docs/tutorials/)
**Books:**
- "Kubernetes Up & Running" by Kelsey Hightower
- "The Kubernetes Book" by Nigel Poulton
- "Kubernetes in Action" by Marko Lukša
**Community:**
- [Kubernetes Slack](https://slack.k8s.io/)
- [Stack Overflow - Kubernetes](https://stackoverflow.com/questions/tagged/kubernetes)
- [r/kubernetes](https://reddit.com/r/kubernetes)
---
**Congratulations!** You now have a comprehensive understanding of Kubernetes. Start with local development using Minikube or Kind, practice with the examples, and gradually move to production deployments. Remember: Kubernetes has a steep learning curve, but the benefits for production workloads are immense.
**Next Steps:**
1. Set up a local cluster (Minikube/Kind)
2. Deploy your first application
3. Practice kubectl commands
4. Explore Helm charts
5. Learn about advanced topics (service mesh, operators, GitOps)
Good luck on your Kubernetes journey! 🚀
