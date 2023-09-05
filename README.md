**Step 1: Set Up Kubernetes Cluster**

Install Kubernetes using kubeadm on your on-premises Linux machines. Follow the official Kubernetes documentation for your specific distribution: [Kubeadm Installation Guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

**Step 2: Install Containerd**

Install Containerd as the container runtime on all nodes. Below are example commands for installing Containerd on Ubuntu:

```bash
# Install Containerd dependencies
sudo apt-get update && sudo apt-get install -y containerd

# Configure Containerd
sudo mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml

# Restart Containerd
sudo systemctl restart containerd
```

**Step 3: Install Rook Ceph**

Deploy Rook Ceph on your Kubernetes cluster. You can use Helm to install Rook:

```bash
# Add the Rook Helm repository
helm repo add rook-release https://charts.rook.io/release

# Create a namespace for Rook
kubectl create namespace rook-ceph

# Install Rook Ceph
helm install --namespace rook-ceph rook-ceph rook-release/rook-ceph
```

Follow the Rook documentation for more detailed instructions: [Rook Documentation](https://rook.io/docs/rook/v1.7/ceph-quickstart.html)

**Step 4: Install Cilium**

Deploy Cilium on your Kubernetes cluster. You can use Helm to install Cilium:

```bash
# Add the Cilium Helm repository
helm repo add cilium https://helm.cilium.io/

# Install Cilium
helm install cilium cilium/cilium --version 1.9.4
```

Refer to the Cilium documentation for additional configuration and advanced features: [Cilium Documentation](https://cilium.io/docs/)

**Step 5: Install MetalLB**

Install MetalLB for load balancing on your cluster nodes. You can use Helm to install MetalLB:

```bash
# Add the MetalLB Helm repository
helm repo add metallb https://metallb.github.io/metallb

# Install MetalLB
helm install metallb metallb/metallb
```

Configure MetalLB to use a range of IP addresses from your on-premises network. Refer to the MetalLB documentation for configuration details: [MetalLB Documentation](https://metallb.universe.tf/)

**Step 6: Set Up NFS Provisioner**

Deploy an NFS provisioner on your Kubernetes cluster. You can use the NFS-Client Provisioner or another available option. Adjust the NFS server configuration to match your on-premises NFS server setup. Here's an example command for deploying the NFS-Client Provisioner:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/nfs-subdir-external-provisioner/master/deploy/deployment.yaml
```

**Step 7: Deploy Applications**

Deploy your applications in the Kubernetes cluster, specifying PVCs for storage. Use the PVCs created with the NFS provisioner for NFS-based storage and the storage classes created by Rook Ceph for Ceph-based storage.

**Step 8: Networking Configuration and Monitoring**

Configure your on-premises network to allow communication between the components. Implement monitoring and maintenance processes for the entire stack to keep it running smoothly.

Prerequisites:

1. set up a local cluster using Minikube or use a cloud-based Kubernetes service like Google Kubernetes Engine (GKE) or Amazon EKS.

2. kubectl command-line tool installed and configured to work with your Kubernetes cluster.

rook operator manifests for Ceph. You can download them from the Rook GitHub repository.

Installation Steps:

Deploy the Rook Operator:

First, create the Rook operator namespace and deploy the operator itself:

bash
kubectl create namespace rook-ceph
kubectl apply -f https://github.com/rook/rook/raw/master/cluster/examples/kubernetes/ceph/operator.yaml

========================================================================================================
Create a Ceph Cluster:
ceph-cluster.yaml define your Ceph cluster.
```
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    image: ceph/ceph:v15
  dataDirHostPath: /var/lib/rook
  mon:
    count: 3
  dashboard:
    enabled: true
 ```

bash
kubectl apply -f ceph-cluster.yaml

========================================================================================================
Create Ceph Pools and StorageClass: 

ceph-pool-storageclass.yaml 
```
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  replicated:
    size: 3
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-ceph-block
provisioner: ceph.rook.io/block
parameters:
  blockPool: replicapool
  clusterName: rook-ceph
  fstype: ext4
reclaimPolicy: Delete
Apply this configuration:
```
bash
kubectl apply -f ceph-pool-storageclass.yaml
Create Ceph Block PVCs (Persistent Volume Claims):
========================================================================================================
PVCs that will utilize the StorageClass you defined
 ceph-pvc.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-ceph-pvc
  namespace: default
spec:
  storageClassName: rook-ceph-block
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

kubectl apply -f ceph-pvc.yaml
Use the PVC in Your Pods:

========================================================================================================

Monitoring:

Rook provides a Ceph dashboard for monitoring. You can access it through a service created by Rook. To access the Ceph dashboard, you'll need the service's IP and port. You can find this information using:


kubectl -n rook-ceph get svc rook-ceph-mgr-dashboard
Then, access the dashboard using a web browser.
