
# Horizontal Pod Autoscaling (HPA)

**HPA (Horizontal Pod Autoscaler) in Kubernetes** automatically scales the number of pods based on CPU, memory, or custom metrics to handle variable workloads efficiently. 

 **Example:**  
 - If CPU usage exceeds **50%**, HPA increases pod count.  
 - If load decreases, HPA reduces pods to save resources.  

 **Command to enable HPA:**  
 ```sh
 kubectl autoscale deployment my-app --cpu-percent=50 --min=2 --max=10
 ```
 This keeps **at least 2 pods**, scales up to **10 pods**, and maintains **50% CPU utilization**.  


# Understand Requests and Limits and HPA

---
### **CPU Requests and Limits in Kubernetes**  
---

### **What does `200m` and `500m` mean in Kubernetes CPU requests/limits?**  
- **`m` stands for "millicores"`**  
- **`1 CPU = 1000m` (1000 millicores)**  
- Kubernetes uses millicores to allow fine-grained CPU allocation.  

### **How is it calculated?**  
- `200m` = **200 millicores** = **0.2 CPU**  
- `500m` = **500 millicores** = **0.5 CPU**  
- `1000m` = **1 CPU core**  
- `2000m` = **2 CPU cores**  

So, when you set:  
```yaml
resources:
  requests:
    cpu: 200m  # 0.2 CPU (Minimum required)
  limits:
    cpu: 500m  # 0.5 CPU (Maximum allowed)
```
This means:
- The container **gets at least 0.2 of a CPU core**.  
- The container **can use up to 0.5 of a CPU core**, but not more.  

---

### **Why is it in `m` (millicores) instead of full cores?**  
Kubernetes runs multiple containers on a single node. Instead of assigning **whole CPU cores**, it allows **fractions of a core**, so small applications don’t waste resources.

Example:
- A **small app** might only need `100m` (0.1 CPU).
- A **database** might need `2 CPU` (`2000m`).
- If you only had full cores (1, 2, etc.), small apps would waste resources.

---

### **Example**  
Imagine your node has **4 CPU cores** (**4000m CPU**).  
- You deploy **10 small containers**, each with `200m` CPU.  
- Each container gets **0.2 CPU**, so Kubernetes can run **20 such containers** on a **4-core machine**.

---

- **1 CPU = 1000m**
- **200m = 0.2 CPU**
- **500m = 0.5 CPU**
- Millicores allow fine-grained CPU allocation in Kubernetes.

---


---
### **Memory Requests and Limits in Kubernetes**  
---
 
 Memory is measured in **bytes** and commonly specified in:  
 - **Mi (Mebibytes) → 1 Mi = 1024 KiB = 1024 × 1024 bytes**  
 - **Gi (Gibibytes) → 1 Gi = 1024 Mi = 1024 × 1024 × 1024 bytes**  
 - You can also use `M` (Megabytes) or `G` (Gigabytes), but `Mi` and `Gi` are recommended for accuracy.  

 ---

 ### **Example**
 ```yaml
 resources:
   requests:
     memory: "256Mi"  # Minimum required (256 Mebibytes)
   limits:
     memory: "512Mi"  # Maximum allowed (512 Mebibytes)
 ```
 This means:  
 - The container **is guaranteed at least `256Mi`** (256 MB).  
 - The container **can use up to `512Mi`**, but not more.  

 ---

 ### **Example**  
 - A **small Node.js app** might need:  
   ```yaml
   requests:
     cpu: "100m"    # 0.1 CPU
     memory: "128Mi" # 128 MB
   limits:
     cpu: "500m"    # 0.5 CPU
     memory: "256Mi" # 256 MB
   ```
 - A **MySQL database** might need:  
   ```yaml
   requests:
     cpu: "500m"    # 0.5 CPU
     memory: "1Gi"  # 1 GB
   limits:
     cpu: "2"       # 2 CPUs
     memory: "4Gi"  # 4 GB
   ```

 ---


# Horizontal Pod Autoscaling (HPA) Testing:

## Prerequisites
Before testing HPA, ensure you have the following:
- A running Kubernetes cluster (EKS in this case)
- `kubectl` installed and configured
- `eksctl` installed if using Amazon EKS
- `metrics-server` deployed for resource monitoring

## Step 1: Verify Cluster and Environment
```sh
kubectl get all
eksctl get cluster
```
Ensure your cluster is up and running.

## Step 2: Deploy the PHP-Apache Application
Create a file `php-apache.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
```

Apply the deployment:
```sh
kubectl apply -f php-apache.yaml
```

Verify deployment and service:
```sh
kubectl describe svc php-apache
kubectl describe deployment php-apache
```

## Step 3: Enable Horizontal Pod Autoscaler (HPA)
Create HPA with CPU threshold of 50% and scaling between 1 and 10 pods:
```sh
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

Check HPA status:
```sh
kubectl get hpa
```

## Step 4: Generate Load
Run a load generator to simulate traffic. This command continuously sends HTTP requests to php-apache, simulating high traffic load. The HPA monitors the CPU usage and, if it exceeds the threshold (50% in this case), Kubernetes scales up the number of pods automatically.

```sh
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

Monitor HPA scaling:
```sh
kubectl get hpa php-apache --watch
kubectl get deployment php-apache
kubectl get pods
```

**In the terminal where you created the Pod that runs a busybox image, terminate the load generation by typing <Ctrl> + C.**


Deploy Metrics Server (If Not Already Installed)
Check if the metrics server is running:
```sh
kubectl get deployment metrics-server -n kube-system
```

If not present, install it:
```sh
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verify HPA and resource usage:
```sh
kubectl get hpa
kubectl top pods
```
