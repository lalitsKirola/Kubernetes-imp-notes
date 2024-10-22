
# Kubernetes-imp-notes

![image](https://github.com/user-attachments/assets/fc7dd686-6fd1-4dd4-b5c4-3824acd261c2)


![image](https://github.com/user-attachments/assets/f69fd1e5-d9bb-4a9b-ab99-73b2fcf75943)

---In the above example , we will create two deployment pods and two corresponding services and above these two service , there will be a ingress resource(kind:ingress) where we will
mention routing rules (path based routing or host based routing) .

---We will hit a url like ip/aksdemo1 , it will go to the ingress controller and then to ingress service , 
according to the rules in the ingress resource(kind:ingress) it will reach to that service and then that associated pods.



Let's break down this flow in a Kubernetes cluster using an example to make it clearer:

### Scenario:
You have a Kubernetes cluster where you're hosting two microservices: 
1. **Service A** accessible at `/serviceA`
2. **Service B** accessible at `/serviceB`

You want to manage external traffic efficiently so users can access these services through a common domain, let's say `example.com`, but route them to the appropriate services based on the path (`/serviceA` or `/serviceB`).

### Flow Breakdown:

1. **Load Balancer**:
   - The **Load Balancer** (usually provided by a cloud provider like Azure or AWS) sits outside your Kubernetes cluster. It exposes your services to the internet and distributes incoming traffic across multiple replicas of the Ingress controller (for high availability).
   - In our case, it's tied to a public IP and will direct any traffic coming to `example.com` to the Ingress Controller.

   **Example**: 
   - You have set up a Load Balancer with a public IP and domain (`example.com`).
   - A user tries to access `http://example.com/serviceA` or `http://example.com/serviceB`.

2. **Ingress Controller**:
   - The **Ingress Controller** is a Kubernetes component that watches for changes in the `Ingress` resources. It's responsible for processing incoming requests and forwarding them to the correct service based on the rules defined in the `Ingress` resource.
   - Popular Ingress Controllers include **Nginx**, **Traefik**, or cloud-specific ones like **Azure Application Gateway**.

   **Example**: 
   - The request from the Load Balancer (`http://example.com/serviceA`) reaches the Ingress Controller. 
   - The Ingress Controller checks the rules defined in the `Ingress` resource.

3. **Ingress Resource**:
   - The **Ingress Resource** contains the rules that define how traffic should be routed. These rules can be **host-based** (e.g., `serviceA.example.com`) or **path-based** (e.g., `/serviceA` or `/serviceB`).
   - It defines which services should handle which requests.

   **Example**:
   - You have an `Ingress` resource with the following path-based routing rules:
     - `http://example.com/serviceA` → **Service A**
     - `http://example.com/serviceB` → **Service B**

4. **Service**:
   - In Kubernetes, a **Service** is an abstraction that defines how to access a group of Pods. It usually has a ClusterIP (or other types like NodePort or LoadBalancer) to allow communication within the cluster.
   - Each service will route traffic to the Pods that are associated with it using labels.

   **Example**:
   - Once the Ingress Controller forwards the request to **Service A** (because the path is `/serviceA`), **Service A** takes care of routing this request to one of the Pods running the actual **Service A** microservice.

5. **Pod**:
   - The **Pod** is the smallest deployable unit in Kubernetes. It's a wrapper around one or more containers (e.g., Docker containers).
   - The service will forward the request to a specific Pod based on a round-robin or some other load-balancing strategy.

   **Example**:
   - **Service A** has 3 Pods running the microservice. It forwards the request to one of the Pods, which handles the request and sends back the response.

### Full Flow Example:

1. **Request Initiation**: 
   - A user goes to their browser and types `http://example.com/serviceA`.
   
2. **Load Balancer**: 
   - The request first hits the external Load Balancer, which has a public IP for `example.com`.

3. **Ingress Controller**: 
   - The Load Balancer forwards the request to the Ingress Controller running inside the Kubernetes cluster.

4. **Ingress Resource**: 
   - The Ingress Controller looks at the Ingress rules and determines that the request path `/serviceA` should go to **Service A**.

5. **Service**: 
   - The request is forwarded to **Service A**, which knows how to route traffic to the right set of Pods running **Service A**.

6. **Pod**: 
   - One of the Pods running **Service A** handles the request and returns a response.

7. **Response**: 
   - The response is sent back through the Ingress Controller, Load Balancer, and ultimately to the user's browser.

### Diagram Representation:

```
User Request → Load Balancer → Ingress Controller → Ingress Resource 
    → Service A (or B based on the path) → Pod (Service A/B) → Response
```

### Path-based Routing Example:
- `http://example.com/serviceA` → **Ingress rule** points to **Service A** → Pod running **Service A**.
- `http://example.com/serviceB` → **Ingress rule** points to **Service B** → Pod running **Service B**.

In this setup, you are managing both microservices under a single domain but routing traffic to different services based on paths.

**  what is ingress resource , is it ingress type service where we created rules  **

No, the **Ingress Resource** is not a type of Kubernetes service like `ClusterIP` or `NodePort`. It is a special Kubernetes object that defines the rules for how external traffic should be routed to internal services within a cluster. 

An **Ingress Resource** acts as a set of rules for controlling how requests coming into the cluster (via HTTP or HTTPS) should be directed to the services inside the cluster. These rules are based on factors like hostnames or URL paths.

Here’s how it works:

### 1. **Ingress Resource**:
   - The **Ingress Resource** is a YAML configuration file where you define the routing rules for incoming traffic.
   - It specifies:
     - **Host-based routing**: Routing based on the requested host (e.g., `example.com`, `api.example.com`).
     - **Path-based routing**: Routing based on the requested URL path (e.g., `/app1`, `/app2`).
     - TLS (optional): Terminate SSL (HTTPS) traffic.
   - The Ingress Resource tells the **Ingress Controller** what to do with the incoming requests.

   **Important**: An Ingress Resource by itself does nothing unless there's an **Ingress Controller** running in the cluster. The Ingress Controller interprets the rules from the Ingress Resource and routes the traffic accordingly.

### Example of Ingress Resource:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /serviceA
        pathType: Prefix
        backend:
          service:
            name: service-a
            port:
              number: 80
      - path: /serviceB
        pathType: Prefix
        backend:
          service:
            name: service-b
            port:
              number: 80
```

### Breakdown of the Example:

- **apiVersion**: The version of the Ingress resource (in this case, `networking.k8s.io/v1`).
- **kind**: Specifies the resource type, here it's `Ingress`.
- **metadata**: Contains the name (`example-ingress`) and optional annotations (e.g., rewrite rules specific to an Ingress Controller like NGINX).
- **spec**: The main section where the routing rules are defined.
  - **rules**: A list of rules for routing traffic.
  - **host**: Specifies the domain name (`example.com`) for which the Ingress rules will apply.
  - **http**: HTTP rules that define how paths should route traffic.
  - **paths**: Contains a list of paths with associated backend services.
    - **path**: The URL path (`/serviceA` and `/serviceB`) that should be routed to specific services.
    - **pathType**: `Prefix` indicates that any path starting with the given string (e.g., `/serviceA`) will match.
    - **backend**: Specifies the service (`service-a` and `service-b`) and the port (e.g., port `80`) that the traffic should be routed to.

### Ingress Resource vs Service:

- **Service**: A Kubernetes object that defines how to expose Pods internally or externally (e.g., `ClusterIP`, `NodePort`, `LoadBalancer`). It directs traffic to Pods based on label selectors.
  - **Example**: You have `service-a` and `service-b`, each managing traffic to a set of Pods.
  
- **Ingress Resource**: Defines how external HTTP/HTTPS traffic should be routed to services within the cluster.
  - **Example**: The Ingress resource defines rules like:
    - Traffic to `http://example.com/serviceA` → Service A
    - Traffic to `http://example.com/serviceB` → Service B

### Ingress Controller:

- **Ingress Controller**: A component responsible for implementing the rules defined in the Ingress Resource. It watches the Ingress Resource and configures the actual routing (using something like NGINX, Traefik, or cloud-native controllers).
  - Without an Ingress Controller, the Ingress Resource won’t function.
  - For example, the **NGINX Ingress Controller** reads the Ingress Resource and configures NGINX to direct traffic to the correct services based on the rules.

### In Summary:

- **Ingress Resource**: A set of rules that define how external traffic should be routed to services.
- **Ingress Controller**: The implementation of those rules (e.g., NGINX Ingress Controller).
- **Service**: An internal routing mechanism that directs traffic to Pods.

The **Ingress Resource** is like a blueprint of rules, while the **Ingress Controller** is the one responsible for executing those rules.

