# Control Plane and Its Components

---

## 🧠 What is the Control Plane?

The **Control Plane** is the brain of a Kubernetes cluster. It orchestrates and manages the overall state of the cluster, ensuring that applications run as intended and resources are allocated efficiently.

### Key Responsibilities:
- **Scheduling applications** to appropriate nodes.
- **Responding to cluster events**, such as a node failure.
- **Managing desired state** of applications (e.g., ensuring 3 replicas of a pod are always running).

### 🧩 Components of the Control Plane

![alt text](k8s_cluster_components.png)

The Control Plane consists of several critical components, each with a specific role:

| Component                | Description                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| **Kube-apiserver**       | Acts as the front-end for the Kubernetes control plane. It exposes the Kubernetes API and is the central point of communication for all components. |
| **Etcd**                 | A consistent and highly available key-value store used to store all cluster data, including configuration and state. |
| **Kube-scheduler**       | Assigns newly created pods to nodes based on resource availability and other scheduling policies. |
| **Kube-controller-manager** | Runs controller processes that regulate the state of the cluster (e.g., replication controller, node controller). |
| **Cloud-controller-manager** | Integrates with cloud provider APIs to manage cloud-specific resources like load balancers, storage, and networking. |

### ✈️ Visual Analogy

Imagine a **traffic/control tower at an airport**:
- The **Control Plane** is the tower.
- The **Nodes** (where your apps run) are the airplanes.
- The Control Plane tells the planes where to go, when to take off, and ensures everything runs smoothly.

---

## 📋 Cluster Prerequisites

A **Kubernetes cluster consists of a control plane and worker nodes**. Before deploying a Kubernetes cluster, ensure you have the minimum resources required for proper functioning.

### ⚙️ Minimum Resource Requirements
- **2 Virtual CPUs**
- **2 GB of Memory**

**Important Note**: You will not be able to deploy a Kubernetes cluster with only 1 virtual CPU and 1 GB of memory. These are the absolute minimum specifications to ensure the cluster operates efficiently.

---

## 🔐 Authentication in Kubernetes

Authentication in Kubernetes clusters doesn't rely on traditional usernames and passwords. Instead, Kubernetes uses a credential-based authentication mechanism.

### 📄 Kubeconfig File

When a cluster is created, the admin generates a **kubeconfig file**:
- This file contains the necessary **credentials** (such as tokens and certificates) to securely access the cluster.
- The kubeconfig file can be **shared with others** to grant them authenticated access.
- **Anyone who has this kubeconfig file can connect to and interact with the Kubernetes cluster**.

#### ⚠️ Security Considerations
- Treat the kubeconfig file as sensitive credentials—it acts like a password for your cluster.
- Control who has access to the kubeconfig file carefully.
- In production environments, consider using role-based access control (RBAC) in addition to kubeconfig to further restrict user permissions.

---

## 🛠️ kubectl: Connecting to the Cluster

To connect with a Kubernetes cluster, we use a binary known as **kubectl**, which is also a command-line tool.

### 📋 How kubectl Works

**kubectl** reads the kubeconfig file stored on your local system to authenticate and communicate with the cluster. It acts as the client-side interface for interacting with Kubernetes clusters.

### 👤 Setting Up kubectl for New Users

If a new user wants access to the same cluster, they need to follow these steps:

1. **Install the kubectl binary**: Download and install kubectl on their local machine.
2. **Create a .kube folder**: Create a `.kube` folder in their home directory (`~/.kube/`).
3. **Place the kubeconfig file**: The cluster admin should share the kubeconfig file, which the user places inside the `.kube` folder.
   - Typically, this file is named `config` and should be located at `~/.kube/config`.
4. **Run kubectl commands**: Once set up, they can run kubectl commands and connect to the cluster.

#### 💡 Example Commands
```bash
# After kubeconfig is in place, users can run:
kubectl get pods
kubectl get nodes
kubectl get services
```

---

## 🔒 Authentication vs Authorization in Kubernetes

While these terms are often used together, they serve distinct purposes in Kubernetes security:

### 🔑 Authentication

**Authentication** checks if the incoming request to the API server is from a **valid user**.

- It verifies the identity of the user making the request.
- Authentication mechanisms include kubeconfig files, tokens, certificates, and other credential methods.
- The API server uses these credentials to confirm: "Are you who you claim to be?"

### 🚨 Authorization

**Authorization** checks if that **authenticated user has permission** to perform the requested action.

- It determines what resources and actions a user can access or modify.
- Authorization is typically enforced through role-based access control (RBAC), attribute-based access control (ABAC), or other policies.
- After confirming the user's identity, the API server asks: "Do you have permission to do this?"

### 📌 Example Scenario

1. We have restricted some users which only can see the k8s cluster resources.
2. Now such a user sends a request to **create a resource** (e.g., deploy a pod).
3. The API server **verifies the user is authenticated** (the credentials in the kubeconfig are valid).
4. The API server checks if the user has **authorization** to create resources.
5. If the user **lacks authorization** (no permission to create), the request is **denied**, even if they are a valid user.

### 🧠 Role of the API Server

The **kube-apiserver** is the central gatekeeper for both authentication and authorization:

- **All communication**—internal or external—goes through the API server.
- **Every request is validated** for both authentication (valid identity) and authorization (proper permissions).
- The API server enforces security policies before allowing any action on the cluster.
- This ensures that only authenticated and authorized users can modify cluster resources.

---

### 🔄 Master Node vs Control Plane

Kubernetes terminology has evolved to be more inclusive and precise. Here's a comparison:

| Terminology       | Description                                                                 |
|-------------------|-----------------------------------------------------------------------------|
| **Older: Master Node** | Referred to the node(s) running the control plane components like kube-apiserver, etcd, kube-scheduler, and controller-manager. |
| **Modern: Control Plane** | Preferred term in Kubernetes. Represents the set of components managing the cluster. Can run on one or more nodes, especially in high-availability setups. |

---
## Control Plan Component

### 🧭 Kube-apiserver

The **kube-apiserver** is the front-end of the control plane and exposes the Kubernetes API over HTTP. It is the central hub through which all operations on the cluster are performed. It means:
- All operations on the cluster go through the kube-apiserver.
  - Kubectl, all components, and client libraries (e.g., Java implementations) communicate with the kube-apiserver.
  - Even components communicate with each other exclusively through the API server.
  - If you want to connect k8s cluster from outside world then also you need to send request to API server

#### 🔧 Responsibilities:
- **Handles RESTful API requests** from kubectl, other components, and client libraries (e.g., Java clients).
- **Validates and processes requests**: It checks if the request is valid (e.g. correct syntax, permisions) and then processes it.
- **Validates authentication and authorization**: Every request is validated for both authentication (verifying the user's identity) and authorization (checking if the user has permission to perform the requested action).
- **Stores state in etcd**: Interacts with etcd to persist and retrieve cluster state.
  - etcd: It is a key-value store that holds the entire cluster state.
- **Serves as a communication hub**: Enables control plane components like the scheduler, controller manager, and nodes to communicate with API server and to stay updated.

#### 📟 Example: What happens when you run `kubectl get pods`
1. `kubectl` sends a request to the kube-apiserver.
2. The API server authenticates and authorizes the request.
3. It queries etcd for the current state of pods.
4. The result is returned to `kubectl`.

---

### 📦 Scheduling Applications

In Kubernetes, **scheduling applications** means deciding which node in the clujster should run a new pod (the smallest deployable unit of an application).

#### 🔍 What Does Scheduling Involve?
When you deploy an application (e.g., a web server or database), Kubernetes must:
- **Find a suitable node** with enough resources (CPU, memory).
- **Match constraints** like node labels and affinity rules.
- **Place the pod** on the selected node.

This process is handled by the **kube-scheduler**, a key component of the Control Plane.

#### 🧪 Example Scenario
Imagine you have 3 nodes:
- **Node A**: 50% CPU used  
- **Node B**: 70% CPU used  
- **Node C**: 30% CPU used  

The scheduler evaluates these nodes and selects the most appropriate one based on multiple factors.

#### 📊 Factors the Scheduler Considers
- **Resource availability** (CPU, memory)
- **Node limits and tolerations**
- **Affinity/anti-affinity rules** (e.g., run pods together or apart)
- **Pod priority and preemption**
- **Custom scheduling policies**

---

### � Kubernetes Scheduler (Kube-Scheduler) Explained

The **kube-scheduler** is the component responsible for deciding which worker node should run a new application (pod). It's a critical decision-making component of the control plane. It works after the API server receives a deployment request and records it in ETCD.

#### 🔄 How Scheduling Works

The scheduling process follows these steps:

1. **You deploy an application** (e.g., a containerized app with a Docker image).
2. **The API server logs the request** in ETCD.
3. **The API server asks the kube-scheduler** to choose a suitable worker node.
4. **The scheduler evaluates available nodes** based on resource requirements and constraints.
5. **The pod is placed on the selected node**.

#### 🛠️ Scheduling Options

Kubernetes provides two main scheduling approaches:

**Manual Scheduling:**
- You **specify the target worker node** during deployment.
- The scheduler only checks if that node has enough resources.
- Useful when you have specific node requirements.

**Automatic Scheduling:**
- You **don't specify a node** in your deployment configuration.
- The scheduler **automatically picks a node** based on resource availability (CPU, memory, etc.).
- The scheduler evaluates all available nodes and selects the best fit.

#### 📌 Example:

Imagine you have 2 worker nodes with the following resource usage:
- **Worker Node 1**: CPU: 40%, Memory: 50%
- **Worker Node 2**: CPU: 80%, Memory: 85%

When deploying a new application:
- Both nodes have enough resources for the application.
- The scheduler would likely choose **Worker Node 1** (less resource utilization).
- This ensures better resource distribution across the cluster.

---

### �🗄️ Etcd

**etcd** is a distributed key-value store that acts as the single source of truth for the entire Kubernetes cluster.

#### 📘 What is etcd?
- A **highly available**, **consistent**, and **distributed** database.
- It stores all cluster data, including:
  - Node information
  - Pod states
  - ConfigMaps
  - Secrets
  - Service discovery data
- **Every request coming to the API server** and every change like: **new app deployment** gets recorded as **key-value pairs inside etcd**. In simple words, it stores all the yaml files.
  - This ensures a complete audit trail and persistent state of all cluster operations and resources.

#### 🚨 Why is etcd Important?
- It ensures **consistency across the cluster**.
- If etcd goes down or becomes corrupted, the **entire cluster state is lost** unless you have a backup.
- All Kubernetes components (like kube-apiserver) **read from and write to etcd**.

#### 🔄 How It Works in the Cluster
- **kube-apiserver** is the **only component** that directly communicates with etcd.
- When you **create or modify a resource** (like a pod), the API server writes the change to etcd.
- Other components (like the scheduler or controller manager) need to know the current state, they **read from the API server**, which in turn reads from etcd.

#### 📟 Example: What happens when you run  
`kubectl create deployment nginx --image=nginx`
1. `kubectl` sends the request to **kube-apiserver**  
2. **kube-apiserver** validates and stores the deployment object in **etcd**  
3. Other components (like the scheduler) schedule the pod based on the data stored in **etcd**

---

### 💾 ETCD Backup & Restore in Kubernetes

**ETCD holds the entire state of your Kubernetes cluster** — applications, configurations, deployments, and all other resources. Backing up ETCD means backing up your cluster's brain.

#### 📌 Why ETCD Backup Matters:

- If your cluster crashes, you don't need to redeploy everything manually.
- Just restore the ETCD backup and your apps and configs will come back automatically.
- ETCD backup is your safety net for disaster recovery scenarios.

#### 🛠️ Recovery Steps:

1. **Set up a new Kubernetes cluster** with the same control plane and worker node infrastructure.
2. **Restore the ETCD backup** to the new cluster.
3. **Your previous state will be redeployed** within minutes.
   - All pods, services, configurations, and deployments will be restored automatically.

#### 📦 Best Practice: External Storage

- **Always store your ETCD backup outside the cluster** — in external storage or a remote location.
- This ensures you don't lose the backup if the cluster itself goes down.
- Examples: AWS S3, Azure Blob Storage, NFS, or any external backup solution.

#### ⚠️ ETCD Security:

- **Only the API server is allowed to communicate with ETCD** for security reasons.
- Direct access to etcd from other components is not permitted.
- This strict access control protects the cluster's state from unauthorized modifications.

---

### 🧠 Kubernetes Controllers (Control Plane Components)

The **kube-controller-manager** is a key control plane component that includes several specialized controllers. Each controller runs in a control loop to monitor and maintain the desired state of the cluster (through API server). Means:
  - Ensuring that desired state (as defined in the cluster configuration) matched the actual state.
  - If it is not then it moves the current state towards desired state.

#### 🔁 Replication Controller

The **Replication Controller** ensures the desired number of application replicas are always running.

- **Ensures desired replicas**: If you specify 3 replicas, Kubernetes will maintain exactly 3 running instances.
- **Automatic replacement**: If any pod crashes, it automatically creates a replacement to maintain the desired state.
- **Scale management**: Handles scaling up or down based on your specifications.
- if all the applications goes down, then automatically k8s will create 3 new applications.

#### 🖥️ Node Controller

The **Node Controller** monitors the health of worker nodes in the cluster.
- If one of wroker node is not working properly. 
- It will show status "not ready" for that particular worker node.

- **Health monitoring**: Continuously checks if nodes are responsive and healthy.
- **Status updates**: If a node becomes unresponsive, it marks the status as **"Not Ready"**.
- **Pod eviction**: May evict pods from unhealthy nodes to reschedule them on healthy nodes.
- **Node cleanup**: Handles removal of pods when nodes are permanently deleted.

#### ☁️ Cloud Controller

The **Cloud Controller** integrates Kubernetes with your cloud provider (AWS, Azure, GCP, etc.).

#### 🌐 Exposing Applications with LoadBalancer Service

To expose your application to external users, Kubernetes provides the **LoadBalancer** service type. Here's how it works:

**When you create a LoadBalancer Service:**

1. You define it in a YAML file.
2. Kubernetes uses the **Cloud Controller Manager**.
3. The Cloud Controller will request cloud provider (AWS/Azure/GCP) to create a load balancer and assign a static IP.
4. The cloud provider creates:
   - A Load Balancer
   - A Public Static IP Address
5. The cloud provider returns the IP to Kubernetes.
6. Kubernetes updates the Service object with this **EXTERNAL-IP**.
7. You share this IP with your clients.
8. The Load Balancer forwards traffic to your app's pods inside the cluster.

So k8s using the cloud controller to get automatically connnected to the cloud.

---

**🔧 What Kubernetes Creates Internally**

When you define a Service of type LoadBalancer, Kubernetes creates a Service object inside the cluster. This object includes:

| Component      | Example Value                             | Purpose                                    |
|----------------|-------------------------------------------|--------------------------------------------|
| **Cluster IP** | 10.96.0.15                               | Internal routing for services within the cluster |
| **NodePort**   | 30080                                     | Port exposed on each worker node           |
| **Endpoints**  | 10.244.1.10:9376, 10.244.2.15:9376      | Pod IPs + container ports for actual traffic routing |

This internal service object is used by other microservices inside the cluster to communicate efficiently.

**📝 YAML Example:**

Let's say you create a service like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 9070
```

In this example:
- **Service name**: `my-loadbalancer`
- **Type**: LoadBalancer (which triggers the Cloud Controller to request a static IP from the cloud provider)
- **Selector**: Routes traffic to pods with label `app: my-app`
- **Port mapping**: External port 80 → Pod container port 9070

When this service is created:
1. Kubernetes creates a Cluster IP for internal communication
2. Kubernetes assigns a NodePort (e.g., 30080) on each worker node
3. Kubernetes creates Endpoints pointing to the selected pods
4. The Cloud Controller requests the cloud provider to provision a load balancer and static IP
5. The load balancer routes external traffic (on port 80) to the cluster, which then forwards it to pods on port 9070


**🌐 External Endpoint (for clients)**

After the Service is created and the Cloud Controller will help you to get **cloud-native load balancer** and **static IP**.

| Property          | Value                    | Details                                      |
|-------------------|--------------------------|----------------------------------------------|
| **External IP**   | 203.0.113.25             | Provisioned by the cloud provider (AWS ELB, Azure LB, GCP LB) |
| **Port**          | 80                       | The port clients use to access the service |
| **Full Endpoint** | 203.0.113.25:80          | The complete address clients use to reach your application |

This external IP is:
- **Assigned by the cloud provider** (not Kubernetes)
- **Associated with the LoadBalancer Service** that you created
- **Stored in the Service object** so you can retrieve it with `kubectl get svc`
- **Shared with clients** who need to access your application


**🔄 How Traffic Flows**

Here's the complete traffic flow from an external client to your application:

1. Client hits 203.0.113.25:80 (external IP from cloud LB).
2. Cloud LB forwards traffic to one of the cluster nodes on NodePort 30080.
3. Kubernetes routes the request to one of the pod endpoints like 10.244.1.10:9376.
4. Your application processes the request and sends the response back through the same path.

### 🚦 Internal Microservice Communication

If another microservice inside your cluster wants to talk to `my-loadbalancer`, it should not use the external IP.

Instead, it should use:
- `http://my-loadbalancer` (if in same namespace)
- `http://my-loadbalancer.default.svc.cluster.local` (fully qualified DNS)

This routes traffic via **ClusterIP → NodePort → Pod endpoints**, staying entirely within the cluster.

---


**✅ Clarifications About Load Balancers**

**Q1: Does Kubernetes create the Load Balancer?**

- ❌ **No.** Kubernetes does not create the load balancer itself.
- ✅ **It only requests the cloud provider to create one.**

Think of Kubernetes saying:
> "Hey AWS/Azure/GCP, I need a load balancer — please create one for me."

**Q2: Where is the Load Balancer created? Inside or outside the cluster?**

- 🟠 The Load Balancer is **always created outside the Kubernetes cluster**.
- It's a **cloud-native resource**, not a Kubernetes object.

Examples:
- On AWS → ELB/NLB is created in your AWS account
- On Azure → Azure Load Balancer is created
- On GCP → Google Cloud Load Balancer is created

These are provisioned in your cloud network, not inside Kubernetes.



**Q3: Why Does Kubernetes Need the External IP?**

- Kubernetes **doesn't use the external IP internally**.
- It stores the IP only to:
  - Display it in `kubectl get svc`
  - Map it to the correct Service
  - Help the Cloud Controller track the resource

✅ **Only clients use the external IP to reach your app.**
Kubernetes just shows it to you, the user.


**Q4: Does the Internal Service Object "Refer" to the External Load Balancer?**

✅ **Yes — indirectly.**

Kubernetes maps the external IP to the internal Service object.

It stores the IP so it can:
- Display it in `kubectl get svc`
- Help the cloud controller track the resource
- Let you (the user) know where clients can reach your app

❌ **Kubernetes does not use the external IP internally.**
✅ **Only clients use it to reach your app.**


---

#### ✅ Essential Control Plane Components

To set up a **functional Kubernetes cluster**, these **four components are critical**:

1. **API Server** — Handles all cluster communications and validates requests.
2. **ETCD** — Stores all cluster state and configuration data.
3. **Kube Scheduler** — Decides which node should run new pods.
4. **Kube Controller Manager** — Includes replication, node, and cloud controllers to maintain desired state.

**Additional Components (Optional):**
- **Kubernetes Dashboard** — Provides a web UI for cluster management (useful but not mandatory).
- **CoreDNS** — Provides service discovery via DNS (useful but not mandatory).
- **Ingress Controller** — Manages external HTTP/HTTPS routes (optional, depends on requirements).

---

## 📌 Additional Insights

Here are a few more important concepts to reinforce your understanding:

- **Declarative Configuration**: Kubernetes uses a declarative model where you define the desired state, and the control plane works continuously to maintain it.
- **High Availability**: In production environments, control plane components are often replicated across multiple nodes to ensure fault tolerance.
- **Security**: The kube-apiserver is the gatekeeper for all cluster operations. Authentication, authorization, and admission control are enforced here.
- **Cluster State Management**: Etcd is critical—if it fails or becomes inconsistent, the entire cluster's state can be compromised.

---
