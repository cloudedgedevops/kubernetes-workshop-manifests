# Azure PVCs Workshop

## Overview

This repository demonstrates how to use Azure Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) within Kubernetes. The workshop includes working with Azure Managed Disks and Azure File Shares, showcasing their integration with Pods through PVCs.

In this workshop, you will:
- Create PVCs backed by Azure Managed Disks and Azure File Shares.
- Deploy Pods using these PVCs to provide persistent storage.
- Validate that PVCs retain data across Pod restarts by using simple file creation and deletion.

## Prerequisites

Before starting the workshop, ensure you have the following:

1. **A Kubernetes cluster**: You should have a Kubernetes cluster like Azure Kubernetes Service (AKS) ready.
2. **kubectl**: Make sure `kubectl` is installed and configured to interact with your Kubernetes cluster.
3. **An active Azure subscription**.

## Getting Started

### 1. Apply the Namespace

Create a dedicated namespace for the PVCs and Pods:

```bash
kubectl apply -f namespace.yaml
```

This ensures that your resources are logically grouped under the `azure-pvcs` namespace.

### 2. Create StorageClass

#### Azure Managed Disk PVC

This StorageClass will be backed by an Azure Managed Disk:

```bash
kubectl apply -f storageclass-disk.yaml
```

#### Azure File Share PVC

This StorageClass will be backed by an Azure File Share:

```bash
kubectl apply -f storageclass-fs.yaml
```
### 3. Create PVCs

#### Azure Managed Disk PVC

This PVC will be backed by an Azure Managed Disk:

```bash
kubectl apply -f pvc_disk.yaml
```

#### Azure File Share PVC

This PVC will be backed by an Azure File Share:

```bash
kubectl apply -f pvc_fs.yaml
```

### 4. Create Pods Using the PVCs

#### NGINX Pod with Azure Managed Disk PVC

```bash
kubectl apply -f pod_pvc_disk.yaml
```

This deploys an NGINX Pod with the `azure-managed-disk` PVC mounted at `/mnt/azure`.

#### NGINX Pod with Azure File Share PVC

```bash
kubectl apply -f pod_pvc_fs.yaml
```

This deploys an NGINX Pod with the `azure-file` PVC mounted at `/mnt/azure`.

### 5. Verify PVCs Are Bound to Pods

After applying the resources, check that the PVCs are correctly bound and attached to the Pods:

```bash
kubectl get pvc -n azure-pvcs
kubectl get pods -n azure-pvcs
```

- The `pvc` should show `Bound` under the `STATUS` column, indicating that the PVC is successfully bound to a Persistent Volume.
- The Pods should have their corresponding PVCs attached under `Volumes`.

### 6. Create a File and Verify PVC Persistence

#### Step 1: Create a File Inside the Pod

To verify that the PVC is working correctly, create a file inside the Pod that uses the Azure Managed Disk PVC:

```bash
kubectl exec -n azure-pvcs pod/nginx-with-disk-pvc -- touch /mnt/azure/test-file.txt
```

This command will create a file named `test-file.txt` in the directory `/mnt/azure` inside the Pod.

#### Step 2: Delete the Pod

Now, delete the Pod running with the PVC:

```bash
kubectl delete pod -n azure-pvcs nginx-with-disk-pvc
```

When you delete the Pod, Kubernetes will terminate it and eventually schedule a new one to replace it, but the Persistent Volume will remain.

#### Step 3: Verify the File in the New Pod

After the Pod is deleted, a new Pod will be created that uses the same PVC. To confirm that the file has persisted across Pod restarts, execute the following:

1. List the Pods in the `azure-pvcs` namespace to wait for the new Pod to start:

```bash
kubectl get pods -n azure-pvcs
```

2. Once the new Pod is up and running, check if the file is still present:

```bash
kubectl exec -n azure-pvcs pod/nginx-with-disk-pvc -- ls /mnt/azure
```

You should see `test-file.txt` listed, indicating that the data is persistent even after the Pod was deleted and recreated.

### 7. Troubleshooting PVC Attachment

If the PVC does not seem to be properly attached to the Pod, follow these steps for troubleshooting:

#### Step 1: Check Pod Details

First, describe the Pod to see if there are any issues with the volume mounting:

```bash
kubectl describe pod <pod-name> -n azure-pvcs
```

Look for the `Volumes` section and confirm that the PVC is listed there.

#### Step 2: Check PVC Status

Verify the PVC status to ensure it is `Bound`. If it’s not bound, there might be an issue with the storage backend.

```bash
kubectl get pvc -n azure-pvcs
```

The `STATUS` column should read `Bound`.

#### Step 3: Check Persistent Volume Status

Ensure the Persistent Volume (PV) is in the correct state:

```bash
kubectl get pv
```

If the PV is not available or not bound to the PVC, check your storage configuration or the availability of Azure resources.

#### Step 4: Review Events

Check for any events related to the PVC or Pod that could indicate issues:

```bash
kubectl get events -n azure-pvcs
```

Look for errors related to PVC binding or volume mounting.

### 8. Breakdown of Kubernetes Resources

This setup involves several key Kubernetes resources:

- **Namespace**: `azure-pvcs` — All resources are created under this namespace.
- **Persistent Volume Claims (PVCs)**: 
  - `azure-managed-disk` and `azure-file`, both requesting 5Gi of storage.
- **StorageClasses**: These define the storage provisioners and parameters for Azure Managed Disks and File Shares.
- **Pods**: Two NGINX Pods, each using one of the PVCs.

## Conclusion

In this workshop, you have successfully:
- Created and used Azure PVCs backed by Managed Disks and File Shares.
- Mounted these PVCs to NGINX Pods.
- Tested PVC persistence by creating a file inside a Pod, deleting the Pod, and confirming that the file was available in the newly created Pod.

This demonstrates the power of persistent storage in Kubernetes with Azure resources, ensuring your data remains available even when Pods are restarted or deleted.
