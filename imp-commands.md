Here are the commands to restart a **pod**, a **deployment**, and **reboot a node** when some configuration changes are made, along with explanations:

---

### **1. Restart a Pod**
Pods are ephemeral and managed by controllers like Deployments, so restarting a pod manually is rare. However, you can force a pod restart by deleting it (if controlled by a deployment, replica set, or daemon set).

#### Command:
```bash
kubectl delete pod <pod-name>
```

#### Explanation:
- **`kubectl delete pod`** deletes the specified pod.
- The associated controller (e.g., Deployment) will detect the missing pod and create a new one with the updated configuration.

---

### **2. Restart a Deployment**
If the configuration of a deployment (e.g., container image, environment variables, or resources) is updated, you can restart the deployment to apply the changes.

#### Command:
```bash
kubectl rollout restart deployment <deployment-name>
```

#### Explanation:
- **`kubectl rollout restart`** gracefully restarts the deployment by creating new pods and terminating old ones.
- Use this when no changes are made directly to the deployment manifest but you need to refresh the pods.

---

### **3. Reboot a Node**
If changes are made that require a node reboot (e.g., kernel updates or system configuration changes), you can reboot the node using `kubectl` or SSH.

#### Command (Drain the Node):
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

#### Explanation:
- **`kubectl drain`** safely evicts all pods from the node.
  - **`--ignore-daemonsets`** ensures daemon set pods remain unaffected.
  - **`--delete-emptydir-data`** deletes pods with emptyDir volumes.

#### Reboot the Node:
SSH into the node and reboot:
```bash
ssh <node-ip>
sudo reboot
```

Alternatively, you can reboot the node programmatically using `kubectl` if the node is part of a cloud-managed cluster:
```bash
kubectl cordon <node-name>    # Mark the node as unschedulable
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <node-name>  # Mark the node as schedulable after reboot
```

#### Post-Reboot:
After the node comes back online, make it schedulable again:
```bash
kubectl uncordon <node-name>
```

---

### Verifications After Changes:
1. **Check Pod Status:**
   ```bash
   kubectl get pods -o wide
   ```

2. **Check Deployment Rollout Status:**
   ```bash
   kubectl rollout status deployment <deployment-name>
   ```

3. **Check Node Status:**
   ```bash
   kubectl get nodes
   ```

