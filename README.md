# Kubernetes WorkShop Repository

Welcome to the **Kubernetes WorkShop Repository**!  This repository contains a collection of hands-on demos that cover essential Kubernetes concepts and practices. Each demo is designed to teach you how to work with specific Kubernetes resources and configurations.

## READMEs

**Note:** Each directory contains a unique README file that provides detailed information about the corresponding demo.

## Repository Overview

### 1. **Azure-PVCs**
In this demo, you'll learn how to work with **Persistent Volumes** and **Persistent Volume Claims (PVCs)** in Azure Kubernetes Service (AKS). By the end of this demo, you will know how to:

- Create and configure a Persistent Volume and PVC using Azure-managed disks.
- Mount the PVC into a Pod for persistent storage.
- Ensure that data persists even when Pods are deleted and recreated.

### 2. **Secrets and ConfigMaps**
This section introduces **Secrets** and **ConfigMaps**, key Kubernetes resources for managing configuration and sensitive data. The goal of this demo is to help you:

- Understand the difference between Secrets and ConfigMaps.
- Create and manage Secrets to securely store sensitive data (like passwords).
- Use ConfigMaps to store non-sensitive configuration data and inject them into Pods.
- Apply Secrets and ConfigMaps in your Pods to make your applications more secure and flexible.

### 3. **Services**
Here, you'll learn about **Kubernetes Services**, which expose your applications to other Pods or external traffic. This demo covers the following types of services:

- **ClusterIP**: Exposes your app within the cluster, making it accessible to other Pods.
- **NodePort**: Exposes your app on a specific port on every node in the cluster, allowing external access.
- **LoadBalancer**: Exposes your app externally via a cloud load balancer.

You'll understand when to use each service type and how they work in different scenarios.

### 4. **Deployments and ReplicaSets**
In this demo, you'll explore **Deployments** and **ReplicaSets**, which help manage the lifecycle and scaling of your applications. By the end of this section, you will learn how to:

- Use a **Deployment** to manage Pods, ensuring that the desired number of replicas are always running.
- Scale your application by adjusting the number of replicas in the Deployment.
- Understand the role of **ReplicaSets** in maintaining a stable set of Pods, even when Pods are recreated.
- Observe the automatic management of Pods and updates in Deployments.

---

## How to Get Started

1. **Clone the repository**:

    ```bash
    git clone https://github.com/your-repo/kubernetes-demo.git
    cd kubernetes-demo
    ```

2. **Apply the demo files**:
   
   For example, to start the **Azure-PVCs** demo, navigate to the corresponding directory and apply the YAML files:
   
    ```bash
    kubectl apply -f Azure-PVCs/azure-pvc-demo.yaml
    kubectl apply -f Azure-PVCs/nginx-pod-with-pvc.yaml
    ```

3. **Explore the resources** you’ve deployed:

    ```bash
    kubectl get pods
    kubectl get services
    kubectl get pvc
    ```