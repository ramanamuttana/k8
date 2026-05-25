# AWS EKS Setup Guide

## Reference Documentation
Always refer to the official AWS documentation for the latest updates and detailed steps:
[Amazon EKS Setup Guide](https://docs.aws.amazon.com/eks/latest/userguide/setting-up.html)

---

## Installing kubectl

`kubectl` is the command-line tool used for interacting with Kubernetes clusters. It allows you to deploy applications, inspect and manage cluster resources, and view logs.

### Installing kubectl on Linux (amd64)

Download the `kubectl` binary:

```sh
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.35.3/2026-04-08/bin/linux/amd64/kubectl
```

Make the downloaded file executable:

```sh
chmod +x ./kubectl
```

#### Moving kubectl to a Directory in Your PATH
To ensure `kubectl` is accessible from anywhere in the terminal, move it to a directory included in your `PATH`. If you have a version of `kubectl` already installed, it is recommended to place the new binary in `$HOME/bin` and update your `PATH` accordingly.

```sh
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
```

Persist the changes by adding the updated `PATH` to your `.bashrc`:

```sh
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
```

To apply the changes, run:

```sh
source ~/.bashrc
```
To verify installation, run:

```sh
kubectl version
```
---

## Installing eksctl

`eksctl` is a command-line tool for creating and managing Amazon EKS clusters. It simplifies cluster creation and automates many manual steps.

Refer to the official installation guide: [eksctl Installation](https://eksctl.io/installation/)

### Download and Install the Latest Release

Set the architecture for your system (default is `amd64`). If using an ARM-based system, set `ARCH` to `arm64`, `armv6`, or `armv7`:

```sh
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
```

Download the latest `eksctl` binary:

```sh
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
```

(Optional) Verify the checksum to ensure file integrity:

```sh
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
```

Extract the binary and move it to `/usr/local/bin` for system-wide access:

```sh
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin
```

To verify the installation, check the version:

```sh
eksctl version
```

---

## **1. Creating an EKS Cluster**  
To create an EKS cluster with `eksctl`:  

## **Please note**

```bash
As of Jan 2026, version 1.31 is outdated. i have used version 1.34.. but you guys please check latest version in AWS and choose accordingly. you can also use 1.34 / 35.. remaining everything is same
 ```

```bash
eksctl create cluster --name=ekswithramana \
  --version 1.34 \
  --region=eu-central-1 \
  --zones=eu-central-1a,eu-central-1b \
  --nodegroup-name ng-default \
  --node-type t3.small \
  --nodes 2 \
  --managed
```

### **Alternative: Using a Config File**  
Instead of specifying all parameters in the command, you can create a YAML configuration file (`eksctl-create.yaml`) and use:  
```bash
eksctl create cluster --config-file=eksctl-create.yaml
```

### **If you want to create a node group manually**
Create a node group manually, we can use below command.

```bash
eksctl create nodegroup \
  --cluster ekswithramana \
  --name managed-ng \
  --node-type t3.small \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --node-ami-family AmazonLinux2 \
  --region eu-central-1
```
---

### **What is an IAM OIDC Provider in AWS EKS?**
The **IAM OpenID Connect (OIDC) Provider** allows **AWS Identity and Access Management (IAM)** to authenticate **Kubernetes service accounts** and assign **AWS IAM permissions** to them.  

In simple terms, it helps your EKS workloads securely access AWS services **without using static IAM credentials**.

---

## **How to Associate an IAM OIDC Provider in EKS?**

### **Step 1: Check if OIDC Provider Exists**
Run:

```bash
aws eks describe-cluster --name ekswithramana --region eu-central-1 --query "cluster.identity.oidc.issuer" --output text
```
If an **OIDC URL** is returned, it already exists. If **empty**, you need to create it.

---

### **Step 2: Create IAM OIDC Provider (If Not Exists)**

```bash
eksctl utils associate-iam-oidc-provider \
  --region eu-central-1 \
  --cluster ekswithramana \
  --approve
```

This registers the **OIDC provider** with IAM.

---

### **Step 3: Verify IAM OIDC Provider**
Run:

```bash
aws iam list-open-id-connect-providers | grep $(aws eks describe-cluster --name ekswitharamana --region eu-central-1 --query "cluster.identity.oidc.issuer" --output text | sed 's|https://||')
```

If it returns an `arn:aws:iam::xxxxx:oidc-provider/`, OIDC is successfully associated.

---

## **2. Managing Clusters**  

### **List all EKS Clusters**  
```bash
eksctl get cluster
```
  
### **View All Resources in the Cluster**  
```bash
kubectl get all
```

### **Delete an EKS Cluster**  
```bash
eksctl delete cluster --name=ekswithramana
```

---

## **3. Deploying an Application**  

### **Apply a Deployment Manifest**  
Deploy an application using a Kubernetes deployment YAML file:  
```bash
kubectl apply -f nginx-deploy.yaml
```

### **View All Deployments**  
```bash
kubectl get deployments
```

### **Get Detailed Information About a Deployment**  
```bash
kubectl describe deployment <deployment-name>
```

---

## **4. Working with Nodes**  

### **List All Nodes in the Cluster**  
```bash
kubectl get nodes
```

### **Describe a Specific Node**  
```bash
kubectl describe node <node-name>
```

---

## **5. Working with Pods**  

### **List All Pods**  
```bash
kubectl get pods
```

### **List All Pods in All Namespaces**  
```bash
kubectl get pods --all-namespaces
```

### **View Pod Logs**  
```bash
kubectl logs <pod-name>
```

### **Stream Logs from a Running Pod**  
```bash
kubectl logs -f <pod-name>
```

### **Delete a Pod**  
```bash
kubectl delete pod <pod-name>
```
---
### K8s Basic and Important commands

| **Category** | **Command** | **Description** |
|-------------|------------|----------------|
| **Cluster Information** | `kubectl cluster-info` | Displays cluster information. |
| | `kubectl version` | Shows Kubernetes client and server versions. |
| **Working with Nodes** | `kubectl get nodes` | Lists all nodes in the cluster. |
| | `kubectl describe node <node-name>` | Displays detailed information about a specific node. |
| **Working with Pods** | `kubectl get pods` | Lists all pods in the default namespace. |
| | `kubectl get pods --all-namespaces` | Lists all pods across all namespaces. |
| | `kubectl describe pod <pod-name>` | Displays detailed information about a pod. |
| | `kubectl logs <pod-name>` | Fetches logs from a pod. |
| | `kubectl logs -f <pod-name>` | Streams logs from a pod. |
| | `kubectl exec -it <pod-name> -- <command>` | Executes a command inside a running pod. |
| | `kubectl exec -it <pod-name> -- /bin/sh` | Opens a shell session inside a pod. |
| | `kubectl delete pod <pod-name>` | Deletes a pod. |
| **Working with Deployments** | `kubectl get deployments` | Lists all deployments. |
| | `kubectl apply -f <deployment-file.yaml>` | Creates or updates a deployment from a YAML file. |
| | `kubectl scale deployment <deployment-name> --replicas=<number>` | Scales a deployment. |
| | `kubectl delete deployment <deployment-name>` | Deletes a deployment. |
| **Managing Resources** | `kubectl apply -f <file.yaml>` | Applies a YAML configuration. |
| | `kubectl delete -f <file.yaml>` | Deletes resources defined in a YAML file. |
| | `kubectl edit <resource-type> <resource-name>` | Edits a resource directly. |
| **Working with Services** | `kubectl get services` | Lists all services. |
| | `kubectl apply -f <service-file.yaml>` | Creates a service from a YAML file. |
| | `kubectl delete service <service-name>` | Deletes a service. |
| **Debugging & Troubleshooting** | `kubectl describe <resource-type> <resource-name>` | Describes a Kubernetes resource. |
| | `kubectl get events` | Displays cluster events. |
| | `kubectl port-forward <pod-name> <local-port>:<pod-port>` | Forwards a local port to a pod. |
| **Miscellaneous** | `kubectl get all` | Lists all resources in the cluster. |
| | `kubectl top nodes` | Displays resource usage of nodes. |
| | `kubectl top pods` | Displays resource usage of pods. |
| **Cleanup** | `kubectl delete pods --all -n <namespace-name>` | Deletes all pods in a namespace. |
| | `kubectl delete all --all -n <namespace-name>` | Deletes all resources in a namespace. |

