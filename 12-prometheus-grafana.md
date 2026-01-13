# Prometheus and Grafana on Amazon EKS with Persistent Storage and Secure Access

This guide walks through a complete, production-ready setup of Prometheus and Grafana on an EKS cluster with:

- Persistent EBS volumes
- Grafana exposed via LoadBalancer
- Secure admin access
- Required IAM and StorageClass configuration

---

## Prerequisites

- EKS Cluster with at least 2 x t3.large nodes
- Installed tools:
  - aws CLI
  - kubectl
  - eksctl
- kubeconfig set up for your EKS cluster

---

## Step 1: Install AWS EBS CSI Driver

Ensure EBS CSI driver is installed:

```bash
eksctl create addon --name aws-ebs-csi-driver --cluster <your-cluster-name> --force
```

---

## Step 2: Attach Required IAM Policy to Node Group Role

Identify the NodeInstanceRole from your node group and attach the AmazonEBSCSIDriverPolicy:

```bash
aws iam attach-role-policy   --role-name <NodeInstanceRoleName>   --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy
```

---

## Step 3: Create StorageClass for EBS CSI

Create a file with name storageclass.yaml and add below content then apply it with "kubectl apply -f storageclass.yaml"

```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp2-csi
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: ebs.csi.aws.com
parameters:
  type: gp2
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

---

## Step 4: Create Namespace

```bash
kubectl create namespace monitoring
```

---

## Step 5: Create values.yaml File

Save the following content in a file named `values.yaml`:

```yaml
prometheus:
  prometheusSpec:
    retention: 15d
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: gp2-csi
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 20Gi
    serviceMonitorSelectorNilUsesHelmValues: false
    podMonitorSelectorNilUsesHelmValues: false
    ruleSelectorNilUsesHelmValues: false

alertmanager:
  alertmanagerSpec:
    storage:
      volumeClaimTemplate:
        spec:
          storageClassName: gp2-csi
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi

grafana:
  adminPassword: "StrongPassword@123"
  service:
    type: LoadBalancer
  persistence:
    enabled: true
    storageClassName: gp2-csi
    accessModes: ["ReadWriteOnce"]
    size: 10Gi
```

---

## Step 6: Add Helm Repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## Step 7: Install Prometheus and Grafana Stack

```bash
helm install prometheus-stack prometheus-community/kube-prometheus-stack   --namespace monitoring   --values values.yaml
```

---

## Step 8: Verify Resources

```bash
kubectl get pods -n monitoring
kubectl get pvc -n monitoring
```

Ensure all pods are running and PVCs are bound.

---

## Step 9: Access Grafana

```bash
kubectl get svc -n monitoring prometheus-stack-grafana
```

Note the EXTERNAL-IP and access Grafana via:

```
http://<EXTERNAL-IP>
```

Login:
- Username: admin
- Password: StrongPassword@123

---

## Step 10: Import Dashboards

1. Go to Dashboards > Import
2. Use ID 6417 for Kubernetes Monitoring
3. Select Prometheus as data source

---

## Optional Cleanup

```bash
helm uninstall prometheus-stack -n monitoring
kubectl delete namespace monitoring
```

---

