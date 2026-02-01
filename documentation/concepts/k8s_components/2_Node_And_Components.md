
# 🖥️ Worker Nodes and Their Components


## 🧩 Components of a Worker Node

A worker node consists of several essential components:

| Component       | Description                                                                 |
|-----------------|-----------------------------------------------------------------------------|
| **Kubelet**     | An agent that runs on every node. It ensures containers are running in a pod as specified by the Control Plane. |
| **Container Runtime** | Software (Docker, containerd, CRI-O) that pulls and runs container images. |
| **Kube-proxy**  | A network proxy that maintains network rules and enables communication between pods and external networks. |


---
## 🧩 Kubelet

The **Kubelet** is an agent that runs on every node in cluster. Basically, it makes a server as K8s node.
It is responsible for managing the lifecycle of pods and containers, running on the node. Basically it is incharge of the node.
It acts as the bridge between the Kubernetes control plane and the container runtime on the node.


### 🔗 Means it is some kind of local manager, which
- Listens for instructions from the control plane (via the API server),
  - Takes appropriate actions on the instructions recieved,
- Ensures the node runs the correct containers,  
- Reports back the status of the node and its workloads  


### 🛠️ Key Responsibilities

#### 📦 Pod Lifecycle Management
- Watches for Pod specs assigned to its node  
- Starts containers using the container runtime  
- Monitors container health and **restarts them if needed**  

#### 📊 Node Status Reporting
- Periodically reports the node's status (CPU, memory, disk, etc.) to the API server  

#### 📁 Volume Mounting
- Mounts volumes specified in the Pod spec  

#### 📈 Logging and Metrics
- Collects logs and metrics from containers  
- Exposes them to monitoring tools  

#### ❤️ Health Checks
- Kubelet monitors health of a pod.
- It executes liveness and readiness probes defined in Pod specs  


### 🚀 What Happens When We Deploy a Pod
1. The API server will make entry of the Pod spec in **etcd**  
2. The scheduler assigns the Pod to a node  
   - Here, I think API-server wont ask kube-schedule to find a node
   - Rather, as soon as entry is made in etcd, then i think kube-controller-manager will ask kube-scheduler (NOt sure, my theory)
3. Now the API-Server with connect the kubelet of that node.
4. So the instructions or pod spec will be given to kubelet.
   - Kubelet will see that it needs to create a pod.
4. So, It uses the container runtime to start the container  
5. If a container crashes, kubelet attempts to restart it based on the restart policy.
6. It keeps checking the container's health and reports back to the API server  
7. If a container crashes, kubelet attempts to restart it based on the restart policy.


---

### 🔄 Node Controller and Kubelet Relationship

- **Node Controller** (on Control Plane): Monitors node health, marks nodes as Ready/NotReady.
- **Kubelet** (on Worker Node): Sends heartbeats, reports status, manages pods.

This partnership ensures the Control Plane always knows the state of worker nodes and can respond if a node becomes unhealthy.

---

### 🧠 Can the Control Plane Act as a Worker Node?

In Kubernetes, the **kubelet** is an agent that runs on every node — and it is also avaialble on control plane.  
This means the control plane can also function as a **worker node** if kubelet is active there.


##### 📦 Why This Matters

All core control plane components — like ETCD, API server, kube scheduler, and kube controller — are applicatons.
And all are deployed as **containers** on a control plane.
Since kubelet manages containers, the control plane also needs kubelet. This means the control plane can also function as a worker node and deploy workloads alongside its primary control plane responsibilities. But it is not advisable.

So if kubelet is running on the control plane, it can:
- Deploy regular application containers
- Schedule system-level components
- Host GUI tools like the Kubernetes Dashboard


#### 🖥️ GUI Access via Kubernetes Dashboard

If you prefer GUI over CLI, you can install the **Kubernetes Dashboard** application on the control plane.  
This dashboard is a containerized application that gets scheduled by kubelet.

Once installed:
- You can manage your cluster visually
- The dashboard runs directly on the control plane
- Kubelet ensures its lifecycle and health


#### ✅ Summary

- Kubelet can run on the control plane
- This enables the control plane to act as a worker node
- You can deploy workloads and dashboards directly on the control plane

---

## 🐳 Container Runtime

The **Container Runtime** is the software that actually runs containers on the node.

### Common Container Runtimes:
- **Docker**: The most popular container runtime (though Docker itself is being phased out in favor of containerd).
- **containerd**: A lightweight, open-source container runtime increasingly used in Kubernetes.
- **CRI-O**: An alternative container runtime specifically designed for Kubernetes.

### Container Runtime Responsibilities:
- **Pulling container images**: Downloads images from registries (Docker Hub, ECR, etc.).
- **Running containers**: Starts, stops, and manages container processes.
- **Resource isolation**: Uses Linux cgroups and namespaces to isolate container resources.

---

## 🕸️ Kube-proxy

- It is a network component in k8s that runs on each node.
- It handles networking tasks.
- It maintains network rules on nodes. means:
  - **Server IP** to **Pod IP** mapping
  
## � Let's unerstand Kube-proxy with help of k8s component **service**

---

### What is ***service**, **Cluster IP**, or **Service IP**

When you create a Kubernetes Service, Kubernetes automatically assigns it a **virtual IP address** inside the cluster. This is known as the **ClusterIP** or **Service IP**.

**Example: Creating a service as per below yaml**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hr-app-service  ---> host-name or the name of service
spec:
  selector:
    app: hr-app
  ports:
    - port: 80
      targetPort: 8080
```

#### ⚙️ How It Works 

- To this service, K8s will assign some IP from **ClusterIP range** which is defined when the cluster is set up.
- Let's say it is `10.96.0.15`. So it will be called as: **ClusterIP** or **Service IP**
- Any pod in the cluster can access the app using: 
  - `10.96.0.15` 
  - Or `http://hr-app-service` 
- This IP remains **constant** as long as the Service exists 
- **Kube-proxy** sets up routing rules so that traffic to this IP is **load-balanced** to the correct pods

#### 🔍 How to Find the Service IP

- using `kubectl get service` command
- Output will be like
```code
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
hr-app-service    ClusterIP   10.96.0.15     <none>        80/TCP    5m
```

---

### 🧩 Why Do We Need a Service?

In k8s **service** component is used to expose endpoints. You create a Service, and behind it you have an application running inside a container on a node. The end-user connects to the Service, and the Service redirects the request to your application.

> But "Why can't the client reach the application directly using the pod's IP and port?"

For example, suppose your pod has IP `10.5.9.11` (or let's say its host name is: "my-pod") and your app is running in one of the containers of this pod, listening on port `80`. Yes, the end-user **can** reach it via `10.5.9.11:80` or `my-pod:80` — **but**:

- Pods are **ephemeral** — they can be destroyed and recreated frequently.
- When recreated, the pod gets a **new IP address**.
- This makes direct access **unstable and unreliable**.
- So there is chance that IP:Port of the application will change next time.

---

### 🛡️ Solution: Stable Service Object

To avoid dependency on changing pod IPs, Kubernetes introduces a stable component: **Service** object, which sits between end-user and application. What makes it stable?

- When you create a service, it gets a **stable IP** (e.g., `12.5.11.9`)
- Since a Service doesn't contain any process, there is not chance it will get terminate automatically
- It only can be deleted if you delete it manually

So clients can reliably send traffic to this IP. The service will redirect all traffic towards your applicaition pods.

This is where **kube-proxy** comes in — it manages the traffic routing between the service and the application (actual pods).

---
### ⚙️ Role of Kube-Proxy 

This is where **kube-proxy** comes in: 

- It manages the traffic routing between the Service and the actual pods. 
- It ensures that requests sent to the Service IP are forwarded to the correct pod endpoints. 
- It provides load balancing if multiple pod replicas exist.

### ⚙️ Kube-proxy Responsibilities:

- Kube-proxy watches the API Server for changes to **Services** and **Endpoints**
  - My explanations: 
    - If services added or deleted.
    - If pod created or distroyed, it will monitor the labels and if it matches with services's selector that it will store the enpoints and setup rules for routing.
- **Maintains routing rules on each node** — Sets up iptables or IPVS rules for routing traffic between pods.
- **Ensures traffic routing** — Traffic sent to a service IP is forwarded to the correct pod(s).
- **Supports load balancing** — Distributes traffic across multiple pod replicas.
- **Dynamic updates** — When pods are added or removed, kube-proxy updates rules in real-time.
- **DNS resolution**: Works with CoreDNS to resolve service names to IPs.


### 🔄 How Kube-proxy Works:
1. Kube-proxy watches the API Server for Service and Endpoint updates.
2. When the kube-apiserver writes the **service** object to **etcd**.
3. **kube-proxy** sees the new service
4. It set up network rules on the node, such that traffic to **Service IP** is forwarded to pods labeled `app : hr-app`
5. If pods are added or removed, kube-proxy updates the routing rules dynamically.

---

## 🧭 How Kubernetes Services Discover Pods  

### Powered by Labels, Selectors, and Kube-Proxy

In Kubernetes, a **Service** can route traffic to application pods **without knowing their IP addresses or hostnames**.  
This is made possible by using **labels and selectors**.

---

### 🏷️ Labels and Selectors: The Core Mechanism

- Each pod is assigned a **label**, e.g., `app=backend`
- The Service uses a **selector** like `app=backend` to match those pods
- The Service then automatically discovers and routes traffic to all matching pods

This dynamic discovery removes the need to hardcode IPs or hostnames.

**Example:**
```yaml
# Pod definition with label
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  labels:
    app: backend    # ← This is the label

# Service definition with selector
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend    # ← This selector matches pods with app=backend
  ports:
  - port: 80
    targetPort: 8080
```

---

![alt text](kube-proxy-routing.png)

### 🔁 What If Multiple Pods Match?

If multiple pods share the same label:
- The Service acts as a **load balancer**
- It distributes traffic across the pods using a **round-robin** strategy

**🍽️ Analogy: Panipuri Distribution**

Just like a Panipuri vendor serves puris in a rotating order — one to each person in sequence — Kubernetes Services forward traffic like this:

```
Request 1 → Pod A
Request 2 → Pod B
Request 3 → Pod C
Request 4 → Pod A
... and so on
```

This ensures balanced traffic distribution across all healthy pods.

---

### ⚙️ Who Manages This Service Discovery?

The traffic routing between the Service and the matched pods is managed by **kube-proxy**:

- It watches for **Service definitions** and **pod defintions**
- It tracks labels and selectors
- Identifies the IP address ofmatching pod
- It **maintains routing rules** on each node using iptables or IPVS
- It **ensures traffic reaches** the correct pod endpoints
- It **updates dynamically** when pods are added, removed, or scaled

---

### 📦 How to View Service IP and Endpoints

After creating a Service using a YAML file, you can inspect its details using:

```bash
kubectl get svc
```

This command shows:
- The Service name
- Its stable **ClusterIP**
- Port mappings
- **External IP** (if applicable)
- **Endpoints** (the pods it's routing to)

**Example output:**
```
NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)
backend-service   ClusterIP   10.96.0.15    <none>        80/TCP
```

---

### ✅ Summary: Service Discovery

- Services use **labels and selectors** to find pods dynamically
- **No need to hardcode IPs or hostnames**
- **Load balancing is automatic** via round-robin
- **Kube-proxy handles all routing logic**
- Use `kubectl get svc` to inspect Service IPs and ports

---

## �🔗 Node Interaction with Control Plane

Here's how worker nodes and the Control Plane work together:

```
Control Plane (API Server)
       ↓ (Assignments)
Kubelet (on Worker Node)
       ↓ (Instructions)
Container Runtime (Pulls images & runs containers)
       ↓ (Pod created)
Application Pod (Your app running inside a container)
       ↑ (Status & Heartbeat)
Kubelet reports back to API Server
```


---

## 🏥 Node Health and Status

The Control Plane continuously monitors worker node health through kubelet heartbeats.

### Node Status Conditions:

| Status     | Meaning                                                    |
|------------|------------------------------------------------------------|
| **Ready**  | Node is healthy and ready to accept pods.                 |
| **Not Ready** | Node is not ready (kubelet is down, disk is full, etc.).  |
| **Unknown** | Control Plane hasn't heard from the node recently.        |

### What Happens When a Node Fails:
1. **Node Controller detects**: The Control Plane's node controller notices the node is unresponsive.
2. **Status changes**: The node status is marked as **"Not Ready"** or **"Unknown"**.
3. **Pods are evicted**: Existing pods on the unhealthy node are evicted and rescheduled.
4. **Rescheduling**: The scheduler assigns the pods to other healthy nodes.

---

## 📊 Node Capacity and Resources

Each worker node has limited resources (CPU, memory, disk). The scheduler considers these when assigning pods.

### Node Resource Tracking:
- **Allocatable resources**: The amount of resources available for pods after reserving system resources.
- **Requested resources**: The CPU and memory each pod requests.
- **Limit resources**: The maximum CPU and memory each pod can use.

### Example:
```
Node Capacity:
- CPU: 4 cores
- Memory: 8 GB

Reserved (for system):
- CPU: 0.5 cores
- Memory: 1 GB

Allocatable (for pods):
- CPU: 3.5 cores
- Memory: 7 GB

Running Pods:
- Pod A: 1 CPU, 2 GB memory
- Pod B: 1.5 CPU, 2 GB memory
- Pod C: 0.5 CPU, 1 GB memory

Remaining Available:
- CPU: 0.5 cores
- Memory: 2 GB
```


