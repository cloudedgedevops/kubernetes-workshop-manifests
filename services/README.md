# Kubernetes Services Workshop

## Overview

In this workshop, we will explore various types of Kubernetes services:
- **ClusterIP**
- **LoadBalancer**
- **Headless Service**
- **NodePort**

These services enable different networking patterns and access methods to expose your application. The demo will use **NGINX** as the application, and we will showcase the different service types with a single **NGINX Pod**.

You will learn:
- How to expose Pods within the cluster using a **ClusterIP** service.
- How to expose Pods externally using **LoadBalancer** and **NodePort** services.
- How to configure a **headless service** to directly access Pods without a load balancer.
- How to create **internal LoadBalancer services** for restricted access.

## Prerequisites

Before starting this workshop, you need to have:
1. **A Kubernetes Cluster**: This example assumes you have a running Kubernetes cluster (e.g., AKS or any other cloud provider).
2. **kubectl installed and configured**: Ensure `kubectl` is installed and configured to communicate with your Kubernetes cluster.
3. **Basic understanding of Kubernetes resources**: Familiarity with Pods and Services is useful.

## Getting Started

### Step 1: Create the Namespace

We will first create a dedicated namespace called `workshop-services`. This helps organize and isolate the services.

Create the namespace:

```bash
kubectl apply -f namespace.yaml
```

The `namespace.yaml` should contain the following YAML:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: workshop-services
```

### Step 2: Create the Pod

We'll use a simple **NGINX** container for the demo. It will expose HTTP on port 80.

Create the NGINX Pod with:

```bash
kubectl apply -f nginx-pod.yaml
```

The `nginx-pod.yaml` should contain the following YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: workshop-services
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

#### Why create the Pod?

- The Pod is the basic unit of execution in Kubernetes. In this example, we deploy an NGINX Pod that will handle HTTP requests on port 80.
- By exposing this Pod with different services, we demonstrate different networking scenarios.

### Step 3: Create Services

Now we will create various types of services to expose the NGINX Pod.

#### 1. ClusterIP Service (Internal Access Only)

This service will expose the NGINX Pod internally within the Kubernetes cluster. **ClusterIP** is the default type of service in Kubernetes.

Create the ClusterIP service:

```bash
kubectl apply -f nginx-clusterip.yaml
```

The `nginx-clusterip.yaml` should contain the following YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
  namespace: workshop-services
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  clusterIP: None  # Headless service. Omit this line if you don't need a headless service.
```

#### 2. LoadBalancer Service (External Access)

This service will expose the NGINX Pod externally using a cloud provider's load balancer (e.g., Azure Load Balancer or AWS ELB).

Create the LoadBalancer service:

```bash
kubectl apply -f nginx-loadbalancer.yaml
```

The `nginx-loadbalancer.yaml` should contain the following YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
  namespace: workshop-services
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

#### 3. Internal LoadBalancer Service (Restricted External Access)

This service will expose the NGINX Pod via a **LoadBalancer** but restrict the access to internal use only (via Azure or AWS annotations).

Create the Internal LoadBalancer service:

```bash
kubectl apply -f nginx-internal-loadbalancer.yaml
```

The `nginx-internal-loadbalancer.yaml` should contain the following YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-internal-loadbalancer
  namespace: workshop-services
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

#### 4. NodePort Service (External Access via Node's IP)

This service will expose the NGINX Pod on a specific port on each node’s IP in the Kubernetes cluster.

Create the NodePort service:

```bash
kubectl apply -f nginx-nodeport.yaml
```

The `nginx-nodeport.yaml` should contain the following YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
  namespace: workshop-services
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80        # The port the service will expose internally in the cluster
      targetPort: 80   # The port the NGINX container is listening on
      nodePort: 30001  # The port on each node's IP that will expose the service externally
```

#### Why use each type of service?

- **ClusterIP**: Best for internal communication between Pods within the cluster. This service type doesn't expose the application externally.
- **LoadBalancer**: Best for exposing applications externally in cloud environments, where Kubernetes automatically provisions a load balancer.
- **Internal LoadBalancer**: Useful when you want to expose your application externally but restrict it to a private internal network.
- **NodePort**: Exposes the application on a static port on each node's IP address. It's often used for development or situations where you don't want to provision a full load balancer.

### Step 4: Verify the Services

After applying the services, verify that everything is working as expected:

#### Check the services

```bash
kubectl get services -n workshop-services
```

You should see something like:

```bash
NAME                             TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-clusterip                  ClusterIP      None           <none>        80/TCP         10m
nginx-internal-loadbalancer      LoadBalancer   10.0.0.1       <pending>     80:31952/TCP   10m
nginx-loadbalancer               LoadBalancer   10.0.0.2       <external-ip> 80:31953/TCP   10m
nginx-nodeport                   NodePort       10.0.0.3       <none>        80:30001/TCP   10m
```

- `nginx-clusterip`: Exposed internally with no external access.
- `nginx-loadbalancer`: Exposed externally with an external IP.
- `nginx-internal-loadbalancer`: Exposed internally but restricted to internal use.
- `nginx-nodeport`: Exposed on port `30001` on every node's external IP.

### Step 5: Access the Services

- **ClusterIP**: Accessible only from other Pods within the cluster.
- **LoadBalancer**: Accessible from external networks via the load balancer’s external IP.
- **Internal LoadBalancer**: Accessible only from within the internal network of the cloud provider.
- **NodePort**: Accessible from external networks by hitting any node's external IP on port `30001`.

#### Testing LoadBalancer Access

Once the LoadBalancer is provisioned, you can access it via the external IP:

```bash
kubectl get service nginx-loadbalancer -n workshop-services
```

Use the external IP provided to access the service:

```bash
curl <external-ip>
```

It should return the NGINX default page.

#### Testing NodePort Access

Access the service via any node's IP on port `30001`:

```bash
curl <node-ip>:30001
```

This will also return the NGINX default page.

### Step 6: Demo - Deleting the Pod and Service Recovery

1. **Delete the Pod**:

```bash
kubectl delete pod nginx-pod -n workshop-services
```

2. **Check Pod Recreation**:

Kubernetes will recreate the Pod automatically as it is managed by the **Deployment** or **ReplicaSet**. Verify the Pod is recreated:

```bash
kubectl get pods -n workshop-services
```

Kubernetes will recreate the Pod and ensure that the service continues to function.

### Troubleshooting

If any of the services aren’t working as expected, here are common troubleshooting steps:

- **Check service endpoints**:

```bash
kubectl describe service <service-name> -n workshop-services
```

Look at the "Endpoints" section to ensure the Pods are properly selected by the service.

- **Check Pod logs**:

```bash
kubectl logs <pod-name> -n workshop-services
```

Look for any issues in the application that might be affecting the service.

- **Check service status**:

```bash
kubectl describe service <service-name> -n workshop-services
```

This can provide insights into why the service isn't functioning, such as incorrect selectors or missing resources.

---

## Conclusion

In this workshop, we have explored how to expose Kubernetes Pods using different service types:
- **ClusterIP** for internal access.
- **LoadBalancer** for external access.
- **NodePort** for external access via a static port.
- **Internal LoadBalancer** for restricted access.

By using these services, we can flexibly expose our applications in Kubernetes based on different access requirements.