# Kubernetes Deployments and ReplicaSets Workshop

## Overview

In this workshop, we will walk through how to deploy applications using **Kubernetes Deployments** and **ReplicaSets**. These two resources are critical in ensuring that your application is highly available and scalable in a Kubernetes environment. The demo uses **NGINX** as a simple application to showcase how Pods are managed and scaled by these resources.

You will learn:
- How to use **Deployments** for managing and scaling Pods.
- How **ReplicaSets** ensure that a set of Pods are running at all times.
- How **Services** expose Pods to enable internal communication within the cluster.
- How Kubernetes automatically recreates Pods if they are deleted.

## Prerequisites

Before starting this workshop, you need to have:
1. **A Kubernetes Cluster**: For example, an AKS (Azure Kubernetes Service) or any other Kubernetes service.
2. **kubectl installed and configured**: Ensure `kubectl` is installed and connected to your Kubernetes cluster.
3. **Basic understanding of Kubernetes resources**: Familiarity with Deployments, Pods, ReplicaSets, and Services.

## Getting Started

### Step 1: Create the Namespace

We will first create a dedicated namespace for this workshop. Namespaces help in logically organizing Kubernetes resources and provide isolation within the cluster.

Create the namespace:

```bash
kubectl apply -f namespace.yaml
```

This will create a new namespace called `deployment-rs-namespace`.

### Step 2: Create the Deployment

A **Deployment** is a higher-level abstraction that manages the lifecycle of Pods. It automatically handles the creation, scaling, and updating of Pods.

The Deployment will:
- Use the `nginx:latest` image.
- Create 3 replicas of the NGINX Pods to ensure high availability.
- Ensure that the Pods are always running by monitoring their health.

Create the Deployment with the following command:

```bash
kubectl apply -f nginx-deployment.yaml
```

The `nginx-deployment.yaml` file should contain the following YAML configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: deployment-rs-namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      securityContext:
        runAsUser: 1000  # Ensures containers run as a non-root user
        runAsGroup: 1000  # Group ID for containers
        runAsNonRoot: true  # Ensures containers do not run as root
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

#### Why do we use a **Deployment**?

- **Scalability**: By specifying the number of replicas (`replicas: 3`), Kubernetes will ensure that there are always three Pods running. If any Pod crashes, Kubernetes will automatically recreate it to maintain the desired state.
- **Pod Management**: The Deployment manages the Pods by ensuring that they match the provided template. It also allows rolling updates and rollbacks to ensure smooth application upgrades.

### Step 3: Create the ReplicaSet

A **ReplicaSet** ensures that a specified number of identical Pods are running at all times. Although the **Deployment** resource already handles replication, we are adding a **ReplicaSet** to demonstrate how it can independently manage Pods.

Create the ReplicaSet:

```bash
kubectl apply -f nginx-replicaset.yaml
```

The `nginx-replicaset.yaml` should contain the following YAML configuration:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  namespace: deployment-rs-namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-rs
  template:
    metadata:
      labels:
        app: nginx-rs
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

#### Why do we use a **ReplicaSet**?

- **High Availability**: Similar to a Deployment, a ReplicaSet ensures that the specified number of Pods are always running. This is useful when you want to maintain a certain level of redundancy for your application.
- **Scaling**: ReplicaSets allow you to easily scale the number of Pods up or down.

### Step 4: Create the Services

To expose the Pods internally within the Kubernetes cluster, we will create **Services**. Services provide stable networking for Pods by exposing their IP addresses to other services within the cluster.

Create the Services for the Deployment and ReplicaSet:

#### Service for Deployment Pods

```bash
kubectl apply -f nginx-deployment-service.yaml
```

The `nginx-deployment-service.yaml` should contain the following YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: deployment-rs-namespace
spec:
  selector:
    app: nginx-deployment
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

#### Service for ReplicaSet Pods

```bash
kubectl apply -f nginx-replicaset-service.yaml
```

The `nginx-replicaset-service.yaml` should contain the following YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: deployment-rs-namespace
spec:
  selector:
    app: nginx-rs
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

### Step 5: Verify the Resources

Once the resources are applied, verify that everything is set up correctly.

#### Verify Namespace

```bash
kubectl get namespaces
```

Ensure that `deployment-rs-namespace` is listed.

#### Verify Deployment

```bash
kubectl get deployment -n deployment-rs-namespace
```

This will list the Deployment, showing 3 replicas for the `nginx-deployment`.

#### Verify ReplicaSet

```bash
kubectl get replicaset -n deployment-rs-namespace
```

This should list the ReplicaSet, ensuring 3 replicas of the `nginx-rs` Pods are running.

#### Verify Services

```bash
kubectl get service -n deployment-rs-namespace
```

Both `nginx-service` for the Deployment and ReplicaSet should be listed.

#### Verify Pods

```bash
kubectl get pods -n deployment-rs-namespace
```

You should see 6 Pods in total: 3 from the Deployment and 3 from the ReplicaSet.

### Step 6: Demo - Deleting a Pod and Pod Recreation

Kubernetes ensures high availability by managing Pods. Let's demonstrate how Kubernetes automatically recreates a Pod when it is deleted.

1. **Delete a Pod**:

```bash
kubectl delete pod <pod-name> -n deployment-rs-namespace
```

Replace `<pod-name>` with the name of one of the Pods from the `nginx-deployment` or `nginx-replicaset`.

2. **Observe Pod Recreation**:

Kubernetes will automatically recreate the deleted Pod to maintain the desired state specified in the Deployment or ReplicaSet. You can monitor the new Pod creation by checking the Pod status:

```bash
kubectl get pods -n deployment-rs-namespace
```

You should see a new Pod being created with a different name (but the same labels), ensuring that there are still 3 Pods running.

#### Why does Kubernetes recreate the Pod?

Kubernetes' primary goal is to maintain the desired state. When you specify that there should be 3 replicas, Kubernetes ensures that 3 Pods are always running. If one Pod fails or is deleted, Kubernetes automatically recreates it to match the desired replica count.

### Step 7: Troubleshooting

If any of the resources are not working as expected, here are some common troubleshooting steps:

#### Step 1: Verify Pod Status

If the Pod fails to start, describe it to get detailed information:

```bash
kubectl describe pod <pod-name> -n deployment-rs-namespace
```

Look for events or conditions that might indicate issues such as image pull errors, resource issues, or unfulfilled readiness probes.

#### Step 2: Verify Deployment Status

```bash
kubectl describe deployment nginx-deployment -n deployment-rs-namespace
```

Ensure the Deployment is correctly managing the Pods, and check for any errors related to scaling or replica issues.

#### Step 3: Verify ReplicaSet Status

```bash
kubectl describe replicaset nginx-replicaset -n deployment-rs-namespace
```

Check if the ReplicaSet is correctly managing the Pods. Look for any discrepancies in the number of replicas.

#### Step 4: Check Service Endpoints

If the Service isn't working, verify that it has endpoints:

```bash
kubectl describe service nginx-service -n deployment-rs-namespace
```

Check the "Endpoints" section to ensure that the Service is selecting the correct Pods.

#### Step 5: View Logs

If the Pods are running but the application is not behaving as expected, view the logs:

```bash
kubectl logs <pod-name> -n deployment-rs-namespace
```

Look for application errors or issues with the NGINX server configuration.

---

## Conclusion

In this workshop, you have:
- Learned how to use Kubernetes **Deployments** and **ReplicaSets** to manage Pods.
- Exposed the application using **Services**.
- Demonstrated how Kubernetes automatically manages Pods by deleting and recreating them when necessary.
- Troubleshot common issues with Pods, Deployments, and Services.

This setup ensures that your application remains highly available, and Kubernetes will automatically scale and manage the Pods to meet your desired state.

