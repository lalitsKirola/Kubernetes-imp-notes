Upgrading a node in a Kubernetes cluster while ensuring zero downtime for the workloads involves **cordoning**, **draining**, and **scheduling the Pods elsewhere**. Kubernetes' built-in mechanisms like **Pod replication**, **Pod disruption budgets (PDBs)**, and **rolling updates** ensure minimal disruption. Here's how it can be managed in a 3-node architecture:

---

### **1. Preparation**
1. **Replication and High Availability**:
   - Ensure the Pods running on the node have replicas on other nodes. For critical workloads, set `replicas > 1` in their Deployment or StatefulSet configurations.
   - Example Deployment:
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
             - name: app-container
               image: my-app-image
     ```

2. **Pod Disruption Budgets (Optional)**:
   - Use Pod Disruption Budgets (PDBs) to limit how many Pods can be unavailable during node upgrades.
   - Example:
     ```yaml
     apiVersion: policy/v1
     kind: PodDisruptionBudget
     metadata:
       name: my-app-pdb
     spec:
       minAvailable: 2
       selector:
         matchLabels:
           app: my-app
     ```

---

### **2. Upgrade Process**

#### **Step 1: Cordon the Node**
Cordoning marks the node as unschedulable, preventing new Pods from being scheduled there.

```bash
kubectl cordon <node-name>
```

Output:
```
node/<node-name> cordoned
```

#### **Step 2: Drain the Node**
Draining evicts all the Pods from the node. Evicted Pods will be rescheduled on other nodes in the cluster if they are part of a Deployment, StatefulSet, or DaemonSet.

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

- `--ignore-daemonsets`: Ensures that DaemonSet Pods (e.g., monitoring or logging agents) remain running.
- `--delete-emptydir-data`: Deletes Pods using `emptyDir` volumes (ephemeral storage).

Output:
```
node/<node-name> drained
```

> Note: Standalone Pods (not managed by controllers) will not be rescheduled automatically. You must handle them manually.

#### **Step 3: Upgrade the Node**
- Perform the necessary upgrade steps (e.g., OS patching, Kubernetes version upgrade).
- Validate the upgrade by ensuring all services and configurations are restored correctly.

---

### **3. Post-Upgrade Steps**

#### **Step 4: Uncordon the Node**
After upgrading the node, mark it as schedulable again:

```bash
kubectl uncordon <node-name>
```

Output:
```
node/<node-name> uncordoned
```

#### **Step 5: Verify Pod Scheduling**
- Check the status of all Pods:
  ```bash
  kubectl get pods -o wide
  ```
  Ensure Pods are rescheduled and running correctly.

- Verify node statuses:
  ```bash
  kubectl get nodes
  ```
  Ensure the upgraded node is in a `Ready` state.

---

### **4. Automation with Cluster Management Tools**
Use tools like **kubeadm**, **kOps**, or **Rancher** to streamline node upgrades. For managed Kubernetes services like AKS, EKS, or GKE, node upgrades can often be automated with rolling updates managed by the provider.

---

### **Example Scenario in 3 Nodes**
1. **Start Upgrade on Node 1**:
   - Cordon and drain Node 1.
   - Pods are evicted and rescheduled onto Nodes 2 and 3.
   - Upgrade Node 1 and uncordon it.

2. **Proceed with Node 2 and Node 3**:
   - Repeat the same steps for Nodes 2 and 3, ensuring at least two nodes remain operational at all times.

---

### **5. Additional Tips**
- **Rolling Updates**: Use rolling deployments for your applications to update replicas incrementally.
- **Monitoring**: Use monitoring tools like **Prometheus**, **Grafana**, or **Kubernetes Dashboard** to track cluster health during the upgrade.
- **Backups**: Always take a backup of important configurations and data before starting the upgrade.

By following this approach, you can ensure that node upgrades occur seamlessly without downtime.
