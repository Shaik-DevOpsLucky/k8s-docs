Lets understand the things by examples.

---

### **Create a Basic Nginx Pod**
#### **`nginx-pod.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```

**annotated version of the nginx-pod.yaml**

```yaml
# Specifies the API version to use for this Kubernetes resource.
# 'v1' is the core API version that includes basic objects like Pods, Services, ConfigMaps, etc.
apiVersion: v1  

# Defines the type of Kubernetes resource being created.
# Here, we are creating a 'Pod'.
kind: Pod  

# Metadata section provides identifying information about the Pod.
metadata:  
  # Name of the Pod (should be unique within the namespace).
  name: nginx-pod  

  # Labels help categorize and select Kubernetes resources.
  # Here, we label the Pod with 'app: nginx' for easier identification.
  labels:  
    app: nginx  

# Specification section that defines the desired state of the Pod.
spec:  
  # The list of containers to run inside the Pod.
  containers:  
    - name: nginx-container  # Name of the container (useful for logging and debugging).
      image: nginx  # The Docker image to use for the container (nginx is a lightweight web server).
      
      # List of ports that the container exposes.
      ports:  
        - containerPort: 80  # Exposes port 80 inside the container (HTTP traffic).
```

#### **Apply the Pod**
```sh
kubectl apply -f nginx-pod.yaml
```
#### **Verify the Pod**
```sh
kubectl get pods
kubectl describe pod nginx-pod
```

---

### **Create a ReplicaSet for Nginx**
#### **`nginx-replicaset.yaml`**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx
          ports:
            - containerPort: 80
```
#### **Apply the ReplicaSet**
```sh
kubectl apply -f nginx-replicaset.yaml
```
#### **Verify the ReplicaSet**
```sh
kubectl get rs
kubectl get pods
```

---

### **Create a Deployment and a Service**
#### **`nginx-deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx
          ports:
            - containerPort: 80
```
#### **Apply the Deployment**
```sh
kubectl apply -f nginx-deployment.yaml
```
#### **Verify the Deployment**
```sh
kubectl get deployments
kubectl get pods
```

---

### **Create a Service to Expose the Deployment with NodePort Option**
#### **`nginx-service.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```
#### **Apply the Service**
```sh
kubectl apply -f nginx-service.yaml
```
#### **Verify the Service**
```sh
kubectl get services
kubectl describe service nginx-service
```

---

### **Access the Nginx Service**
- Get the **NodePort** assigned to the service:
  ```sh
  kubectl get service nginx-service
  ```
- Open a browser and go to:
  ```
  http://<your-cluster-node-ip>:<node-port>
  ```

  ---

### **Create a Service to Expose the Deployment with LoadBalancer Option**

#### **`nginx-service-elb.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

#### **Apply the Service**
```sh
kubectl apply -f nginx-service.yaml
```
#### **Verify the Service**
```sh
kubectl get services
kubectl describe service nginx-service
```
