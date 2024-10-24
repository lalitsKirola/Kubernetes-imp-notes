systemassigned manaeged identity 

CSI drivers -to recognise and read encrypted keys



![image](https://github.com/user-attachments/assets/4f7b3b69-d99e-4a7e-a6af-97f76092c2fa)
To explain how **CSI drivers**, **AKS VMSS**, **system-assigned managed identity**, **pods**, and **Azure Key Vault** work together in accessing secrets from an **Azure Key Vault** from an **AKS pod**, I'll break it down step by step. Here's how these components come together:

### 1. **Azure Kubernetes Service (AKS) Setup**
AKS is a managed Kubernetes service in Azure that allows you to run Kubernetes clusters without handling the underlying infrastructure.

- **VMSS (Virtual Machine Scale Set)**: In AKS, the worker nodes where your pods run are managed as part of a VMSS. This ensures your cluster can scale up/down based on demand.

### 2. **Pod in AKS**
A **pod** is the smallest deployable unit in Kubernetes, which can contain one or more containers. The pod runs your application, and in this case, it might need access to secrets (like database credentials, API keys, etc.) stored in **Azure Key Vault**.

### 3. **System-Assigned Managed Identity**
**Managed identities** in Azure are a way for resources (such as VMs, or AKS nodes) to access other Azure resources securely, without needing to manage secrets like passwords or keys.

- **System-assigned managed identity**: When you enable a system-assigned managed identity on your AKS cluster or VMSS (the nodes of the cluster), Azure automatically creates an identity tied to the lifecycle of the cluster. This managed identity can be used by the AKS nodes to authenticate and access Azure services, like Key Vault.

### 4. **Azure Key Vault**
**Azure Key Vault** is a service that securely stores secrets, keys, and certificates. You’ll store sensitive data (like API keys, database connection strings, etc.) in the Key Vault and give permission to your AKS-managed identity to access it.

### 5. **CSI (Container Storage Interface) Driver for Secrets Store**
The **CSI Secrets Store Driver** is a Kubernetes plugin that allows pods to mount secrets from external providers, like Azure Key Vault, directly into the container filesystem. This avoids embedding sensitive information directly in the pod’s configuration.

### Steps to Access Secrets from Azure Key Vault in an AKS Pod:

1. **Create an Azure Key Vault**:
   - Store the secrets you need in the Key Vault (e.g., a database connection string).
   
2. **Assign Permissions to the Managed Identity**:
   - Grant the **system-assigned managed identity** of the AKS VMSS permissions (like **Get** and **List** secrets) to the Key Vault using Azure Role-Based Access Control (RBAC).

3. **Install the CSI Secrets Store Driver in AKS**:
   - The **Azure Key Vault Provider for Secrets Store CSI Driver** needs to be installed in your AKS cluster. This driver will allow you to mount Key Vault secrets as volumes in your pods.

4. **Create a Pod that Uses the CSI Driver**:
   - In the pod’s definition (YAML file), specify the volume to mount using the CSI driver and the secret from Key Vault.

### Example Walkthrough:

#### a. **Key Vault Setup**:
```bash
# Create a Key Vault
az keyvault create --name MyKeyVault --resource-group MyResourceGroup --location eastus

# Add a secret to Key Vault
az keyvault secret set --vault-name MyKeyVault --name MySecret --value "SuperSecretValue"
```

#### b. **Assign Managed Identity to VMSS and Grant Access**:
```bash
# Get the VMSS identity of your AKS cluster
AKS_IDENTITY=$(az aks show --resource-group MyResourceGroup --name MyAKSCluster --query "identityProfile.kubeletidentity.clientId" -o tsv)

# Assign Key Vault Reader Role to the Managed Identity
az role assignment create --role "Key Vault Secrets User" --assignee $AKS_IDENTITY --scope /subscriptions/<subscription-id>/resourceGroups/MyResourceGroup/providers/Microsoft.KeyVault/vaults/MyKeyVault
```

#### c. **Install CSI Driver in AKS**:
```bash
kubectl apply -f https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/deploy/secrets-store-csi-driver.yaml
kubectl apply -f https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/deploy/provider-azure-installer.yaml
```

#### d. **Create Pod Definition to Mount Secret**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: nginx
    volumeMounts:
    - name: secrets-store01-inline
      mountPath: "/mnt/secrets"
      readOnly: true
  volumes:
  - name: secrets-store01-inline
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: "azure-kvname"
---
apiVersion: secrets-store.csi.k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: azure-kvname
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true"
    userAssignedIdentityID: ""
    keyvaultName: "MyKeyVault"
    cloudName: ""
    objects: |
      array:
        - |
          objectName: MySecret
          objectType: secret        # object types: secret, key, or cert
    tenantId: "<azure-tenant-id>"
```

- In the above YAML file, the `SecretProviderClass` defines the Key Vault and the secret you want to mount.
- The pod will mount the secret as a file inside `/mnt/secrets/MySecret`.

#### e. **Run the Pod**:
When the pod starts, it will automatically use the system-assigned managed identity of the AKS node to authenticate and access the secret from Key Vault, which is then made available to the pod in the file system.

### Summary:
1. The AKS cluster nodes (running on VMSS) have a **system-assigned managed identity**.
2. You store secrets in **Azure Key Vault** and assign permissions to the managed identity.
3. The **CSI Secrets Store Driver** mounts secrets from Key Vault to the pod.
4. The pod accesses secrets securely without embedding them in the application code or environment variables.

This process ensures your application can access Azure Key Vault secrets securely within an AKS pod using the CSI driver, leveraging AKS VMSS's managed identity for authentication.

Let me know if you'd like further clarification or examples!


Link for detailed explanation : https://shailender-choudhary.medium.com/access-secrets-from-azure-key-vault-in-azure-kubernetes-service-e8efffe49427
