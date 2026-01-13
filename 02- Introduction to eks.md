
## **What is EKS?**  
**Amazon Elastic Kubernetes Service (EKS)** is a fully managed Kubernetes service provided by AWS. It simplifies the deployment, management, and scaling of containerized applications using Kubernetes on AWS infrastructure. With EKS, you can run Kubernetes without needing to install, operate, or maintain your own control plane or nodes.

---

## **Key Features of EKS**  
1. **Managed Control Plane**:  
   - AWS manages the Kubernetes control plane (API server, etcd, scheduler, etc.), ensuring high availability and reliability.  
   - The control plane is automatically scaled and patched by AWS.  

2. **Integration with AWS Services**:  
   - Seamlessly integrates with AWS services like **IAM**, **VPC**, **CloudWatch**, **Load Balancers**, and **RDS**.  
   - Enables secure and scalable application architectures.  

3. **High Availability**:  
   - The control plane runs across multiple Availability Zones (AZs) to ensure fault tolerance.  
   - Worker nodes can also be distributed across AZs for resilience.  

4. **Security**:  
   - Integrates with AWS IAM for fine-grained access control.  
   - Supports encryption for data at rest and in transit.  
   - Provides network isolation using VPC and security groups.  

5. **Scalability**:  
   - Automatically scales worker nodes using **Cluster Autoscaler** or **AWS Fargate** for serverless workloads.  
   - Handles thousands of nodes and pods efficiently.  

6. **Cost-Effective**:  
   - Pay only for the resources you use (worker nodes, EBS volumes, etc.).  
   - No additional charge for the managed control plane (starting October 2020).  

7. **Hybrid and Multi-Cloud Support**:  
   - Supports **EKS Anywhere**, allowing you to run Kubernetes clusters on-premises or in other clouds.  
   - Provides consistent tooling and APIs across environments.  

---

## **EKS vs Self-Managed Kubernetes**  


| Feature | Self-Managed Kubernetes | Amazon EKS |
|---------|--------------------|------------|
| **Control Plane Management** | You must set up and maintain the Kubernetes control plane (API server, scheduler, controller manager, etc.). | AWS fully manages the control plane, including scaling, security, and updates. |
| **Worker Nodes** | You need to provision and manage worker nodes (on-premises or cloud VMs). | You only manage worker nodes; AWS handles the control plane. |
| **High Availability** | You must configure HA across multiple servers and regions. | AWS automatically runs the control plane across multiple Availability Zones for HA. |
| **Networking** | Requires setting up Kubernetes networking manually. | Integrates with AWS networking (VPC, ALB, NLB, etc.). |
| **Security & IAM** | You manage authentication, RBAC, and security policies. | Integrated with AWS IAM, security groups, and encryption for easy access control. |
| **Upgrades & Patching** | You need to manually upgrade and patch Kubernetes versions. | AWS provides automated upgrades and patches for the control plane. |
| **Monitoring & Logging** | You must configure tools like Prometheus and Grafana. | Integrated with AWS services like CloudWatch, CloudTrail, and X-Ray. |
| **Autoscaling** | Requires configuring Cluster Autoscaler manually. | Supports AWS Auto Scaling for nodes and pods with seamless integration. |
| **Integration with AWS Services** | Requires custom configurations for AWS integrations. | Built-in support for AWS services like ECR, IAM, CloudWatch, and RDS. |
| **Pricing** | Free, but you pay for the infrastructure you use (compute, storage, etc.). | You pay for worker nodes plus an additional charge of ~$0.10 per hour per EKS cluster. |

---

Under the hood, AWS **Elastic Kubernetes Service (EKS)** manages the **Kubernetes control plane** across multiple **Availability Zones (AZs)** to ensure **high availability and fault tolerance**.

**EKS runs three master node copies spanning three AZs**, providing **99.95% uptime SLA** for the Kubernetes API server.

### **Master Node Architecture in AWS EKS:**

   AWS EKS runs **three copies** of the Kubernetes control plane across **three different AZs** in the selected AWS Region.
   - The control plane nodes are **distributed** across **three AZs** to prevent a single point of failure.  
   - If one AZ goes down, the other two can continue handling requests.  
   - AWS handles **scaling**, **patching**, and **failure recovery** of the master nodes.  
   - Users **do not have access** to these control plane nodes.
   - The control plane communicates with worker nodes using an **Amazon VPC endpoint**, ensuring **low latency** and **secure communication**.

---

<img width="983" alt="image" src="https://github.com/user-attachments/assets/908b7b50-5084-49ba-86d0-5334969b738d" />


---

### **EKS Data Plane Options**  

---

In Amazon EKS (Elastic Kubernetes Service), the **Control Plane** is managed by AWS, but you need to choose how your **worker nodes (Data Plane)** run. There are three main options:  

1️⃣ **Amazon EC2 Self-Managed Node Groups**  
2️⃣ **Amazon EC2 Managed Node Groups**  
3️⃣ **AWS Fargate**  

---

## **1️⃣ Amazon EC2 Self-Managed Node Groups** (Full Control, More Management)  
- You manually **provision, configure, and manage** EC2 instances as worker nodes.  
- You must handle **scaling, updates, and security patches**.  
- Best for **custom configurations, special instance types, or Spot instances**.  
- Requires a **custom AMI (Amazon Machine Image)** or an optimized AMI provided by AWS.  

**Use Case**: When you need **full control** over node management.  

---

## **2️⃣ Amazon EC2 Managed Node Groups** (AWS Manages Nodes)  
- AWS **automatically provisions and manages** EC2 worker nodes.  
- Supports **automatic updates, scaling, and lifecycle management**.  
- Uses **Amazon Linux 2 or Bottlerocket** as default AMI.  
- You only need to **select the instance type and size**.  

**Use Case**: When you **want EC2 flexibility** but without managing updates manually. **Easier autoscaling** compared to self-managed EC2 nodes.  

---

## **3️⃣ AWS Fargate** (Serverless, No Nodes to Manage)  
- **Fully serverless**—no need to manage EC2 instances.  
- AWS automatically provisions and scales **pods**, not nodes.  
- **Pay-per-use** pricing—**only pay for running pods**.  
- No need to worry about OS patching, scaling, or security updates.  

**Use Case**:  
- When you want a **fully managed, cost-effective** approach.  
- For **short-lived workloads, bursty applications, and batch jobs**.  
- **Best for microservices** that don’t need direct node management.  

---

## **Comparison Chart: EKS Data Plane Options**  

| Feature               | EC2 Self-Managed Nodes | EC2 Managed Nodes | AWS Fargate |
|-----------------------|----------------------|-------------------|-------------|
| **Who Manages Nodes?** | **You** | **AWS** | **AWS (No Nodes, Only Pods)** |
| **Scaling** | Manual or Custom | Auto-managed | Auto-managed |
| **OS Updates** | Manual | AWS-managed | AWS-managed |
| **Custom AMI Support** | Yes |  No (AWS AMI only) |  No (AWS manages runtime) |
| **Pod Density Control** | Yes | Yes |  No (Pods run individually) |
| **Best For** | Custom setups, Spot Instances | Standard workloads, auto-scaling | Serverless workloads, cost-efficient apps |
| **Cost** |  High (EC2 cost + management effort) |  Medium (AWS manages updates) |  Low (Pay per pod runtime) |
| **Ideal Workloads** | High-performance, GPUs, ML workloads | Web apps, backend services | Serverless, microservices |

---

### **Most Commonly Used AWS Services with EKS**  

| **AWS Service** | **Usage with EKS** |
|---------------|----------------|
| **Amazon VPC** | Provides **networking and security** for EKS clusters. Pods communicate via VPC CNI. |
| **Elastic Load Balancer (ELB)** | Exposes EKS applications to the internet using **ALB, NLB, or CLB**. |
| **Amazon EBS** | Used for **persistent storage** with EKS worker nodes. Ideal for databases and stateful apps. |
| **Amazon EFS** | Provides **shared file storage** for multiple EKS pods, useful for distributed workloads. |
| **AWS IAM** | Manages **authentication and authorization** for users, roles, and Kubernetes service accounts. |
| **Amazon RDS & Aurora** | Managed **databases** that EKS applications can connect to for persistent storage. |
| **Amazon S3** | Stores **logs, backups, and artifacts** for applications running in EKS. |
| **AWS CloudWatch** | **Monitors logs, metrics, and alarms** for EKS workloads. Integrated with Fluent Bit for logging. |
| **AWS X-Ray** | Helps with **distributed tracing and debugging** of microservices in EKS. |
| **AWS Secrets Manager** | Stores **secrets (API keys, credentials, DB passwords)** securely for use in EKS. |
| **AWS App Mesh** | Implements a **service mesh** for observability, traffic control, and security in EKS. |
| **AWS KMS** | Encrypts **secrets, data, and Kubernetes secrets** stored in EKS. |

---
### **Comparison of EKS Cluster Creation Methods**  

| **Method** | **Infrastructure Control** | **Best For** |
|------------|----------------|-------------|
| **eksctl (Preferred)** | Limited (Auto-configured VPC, IAM, Networking) | Quick & production-ready clusters |
| **AWS Console** | Limited | Beginners, testing |
| **AWS CLI** | Full Control | Advanced users, scripting |
| **CloudFormation** |Full Control | Enterprises, repeatable infra |
| **AWS CDK / Terraform** |Full Control | Large-scale production workloads |

---

### **Why is `eksctl` the Preferred Option?**  

**Simplifies Cluster Creation** – `eksctl` automates the entire EKS cluster setup, including VPC, node groups, and IAM roles, reducing manual configurations.  
**One command setup** → `eksctl create cluster --name my-cluster --region us-east-1`  
**YAML support** → Easy to define cluster config in a file for repeatability.  
**Officially recommended by AWS** → Designed for EKS, built by Weaveworks.  
**Simplifies Node Group Management** → Easily set up managed/self-managed nodes.  
**Faster and More Efficient** – It creates an EKS cluster with a single command, saving time compared to using AWS Management Console or manual `kubectl` and CloudFormation configurations.  

---

| **Command** | **Brief Description** |
|------------|----------------------|
| `eksctl create cluster` | Creates an EKS cluster with a default configuration, including one node group with two `m5.large` nodes. |
| `eksctl create cluster --name <name> --version 1.31 --node-type t3.micro --nodes 2` | Creates an EKS cluster with Kubernetes version `1.31`, using a node group with two `t3.micro` nodes. |
| `eksctl create cluster --name <name> --fargate` | Creates an EKS cluster with **Fargate**, enabling serverless compute for running pods without managing worker nodes. |


## eksctl & kubectl

| **Feature**  | **eksctl** | **kubectl** |
|-------------|-----------|------------|
| **Purpose**  | Used for creating and managing **EKS** clusters and node groups. | Used for managing Kubernetes workloads and resources (pods, deployments, services, etc.) across **any** Kubernetes cluster. |
| **Scope**    | **Only for Amazon EKS**. | Works with **all** Kubernetes clusters, including EKS, self-managed, and other cloud K8s providers. |
| **Functionality** | Automates EKS cluster setup, including VPC, IAM, and node groups. | Interacts with the Kubernetes API to deploy, manage, and inspect resources. |


### **Prerequisites to Install `eksctl`**  

1. **AWS CLI** – Must be installed and configured with credentials (`aws configure`).  
2. **kubectl** – Required for managing Kubernetes resources after cluster creation.  
3. **AWS IAM Permissions** – User must have IAM permissions to create and manage EKS clusters.  

---
