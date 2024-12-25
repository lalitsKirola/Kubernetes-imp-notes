### Kubernetes Architecture Overview

Kubernetes is an open-source container orchestration platform that manages containerized applications. Its architecture consists of **control plane components** and **worker nodes**, each playing a vital role in processing and handling requests.

---

### **Components of Kubernetes Architecture**

#### **Control Plane Components** (Master Node)
1. **API Server**: 
   - The entry point for all Kubernetes REST API requests.
   - Authenticates and validates requests before processing.

2. **Scheduler**:
   - Assigns unscheduled pods to suitable nodes based on resource requirements and constraints.

3. **Controller Manager**:
   - Handles controllers like ReplicationController, NodeController, and EndpointController to maintain the desired state.

4. **etcd**:
   - A distributed key-value store to persist the cluster's state and configuration.

5. **Cloud Controller Manager** (optional):
   - Integrates Kubernetes with underlying cloud provider APIs (e.g., AWS, Azure).

---

#### **Worker Node Components**
1. **Kubelet**:
   - Agent running on each node, ensuring containers are running as defined in pod specs.

2. **Kube Proxy**:
   - Manages network rules for exposing services and load balancing.

3. **Container Runtime**:
   - Executes containers (e.g., Docker, containerd).

4. **Pod**:
   - The smallest deployable unit in Kubernetes, containing one or more containers.

---

### **How a Request is Handled: An Example**

Imagine a user requests to deploy an application via a YAML file defining a **Deployment** and **Service**.

---

#### 1. **User Interaction with the API Server**
- The user submits a request to the **API Server** (e.g., `kubectl apply -f deployment.yaml`).
- The API Server:
  - Authenticates the user.
  - Validates the YAML file syntax.
  - Writes the deployment definition into **etcd**.

---

#### 2. **Scheduler Assigns Nodes**
- The **Scheduler** checks for unscheduled pods.
- Evaluates resource constraints (CPU, memory) and affinity rules.
- Assigns the pods to suitable worker nodes.

---

#### 3. **Controller Manager Ensures Desired State**
- The **ReplicaSet Controller** ensures the number of pods matches the desired state (e.g., 3 replicas).
- If pods fail or nodes go down, the controller recreates pods on other nodes.

---

#### 4. **Kubelet Executes Pods**
- Each **Kubelet** on the worker node communicates with the API Server.
- Pulls the pod specification and ensures containers are running.
- Communicates with the **Container Runtime** to start containers.

---

#### 5. **Kube Proxy Manages Networking**
- The **Kube Proxy** on worker nodes sets up networking rules.
- Exposes the application via ClusterIP, NodePort, or LoadBalancer service.
- Ensures traffic is routed to the appropriate pods.

---

#### **Example in Practice**
Let's deploy an application:

1. **Deployment YAML**:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-app
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: my-app-container
           image: nginx
           ports:
           - containerPort: 80
   ```

2. **Service YAML**:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-app-service
   spec:
     selector:
       app: my-app
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
     type: LoadBalancer
   ```

---

#### **Request Lifecycle**
1. **User Command**: `kubectl apply -f deployment.yaml`.
   - API Server validates and persists the deployment.

2. **Pods Created**:
   - Scheduler assigns pods to worker nodes.

3. **Containers Start**:
   - Kubelet pulls the image (nginx) and runs containers.

4. **Service Routes Traffic**:
   - Kube Proxy configures networking, exposing the application to the external world.

---

This modular and scalable architecture allows Kubernetes to handle deployments, scaling, and self-healing efficiently.
