# NFS Setup for k3s with Rocky Linux

This guide provides step-by-step instructions for setting up an NFS server on a Rocky Linux host and integrating it with a k3s cluster.

---

## 1. Install and Configure NFS Server

### Install NFS Utilities
Install the required NFS utilities by running the following command:

```bash
sudo dnf install -y nfs-utils
```

### Create a Shared Directory
Create and configure the directory for NFS sharing:

```bash
sudo mkdir /data
sudo chown -R 1000:65534 /data
sudo chmod 2770 /data
```

### Configure the NFS Exports File
Edit the `/etc/exports` file and add the following line to share the directory:

```bash
/data 192.168.1.0/24(rw,sync,anonuid=1000,anongid=65534,root_squash)
```

### Enable and Start NFS Server
Run these commands to enable and start the NFS server:

```bash
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
sudo exportfs -arv
```

### Firewall Configuration
If your firewall is active, allow NFS traffic by running:

```bash
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --reload
```

## 2. Install NFS CSI Driver on k3s

### Add the Helm Repository
Add the Helm repository for the NFS CSI driver:

```bash
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm repo update
```

### Install the NFS CSI Driver
Install the driver into the kube-system namespace:

```bash
helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system
```

### Verify the Installation
Ensure the CSI driver is running and available:

```bash
kubectl wait pod --selector app.kubernetes.io/name=csi-driver-nfs --for condition=ready --namespace kube-system
kubectl get csidrivers
```

## 3. Create a StorageClass

### Define the StorageClass
Create a file named `nfs-storageclass.yaml` with the following content:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs
provisioner: nfs.csi.k8s.io
parameters:
  server: 192.168.1.xx
  share: /data
reclaimPolicy: Retain
volumeBindingMode: Immediate
mountOptions:
  - hard
  - nfsvers=4.1
```

### Apply the StorageClass
Apply the configuration:

```bash
kubectl apply -f nfs-storageclass.yaml
```

## 4. Configure k3s Nodes to Use NFS

### Install NFS Client on k3s Nodes
On each k3s node, install the NFS utilities:

```bash
sudo dnf install -y nfs-utils
```

### Mount the NFS Share
Create a mount point and mount the NFS share:

```bash
sudo mkdir -p /mnt/nfs/k3s
sudo mount -t nfs <NFS_SERVER_IP>:/data /mnt/nfs/k3s
```

To ensure the mount persists after a reboot, add the following entry to `/etc/fstab`:

```bash
<NFS_SERVER_IP>:/data /mnt/nfs/k3s nfs defaults 0 0
```

## 5. Deploy a Persistent Volume in k3s

Create a YAML manifest for the Persistent Volume (PV):

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /data
    server: <NFS_SERVER_IP>
```

Create a Persistent Volume Claim (PVC):

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```

Apply the manifests with:

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
```

## 6. Verify the Setup

- Confirm the NFS mount on each node:

```bash
mount | grep nfs
```

- Check the PVC status:

```bash
kubectl get pvc
```

If everything is correctly configured, the PVC should show as `Bound`.

---

This setup ensures you have a functional NFS server integrated with your k3s cluster for persistent storage management on Rocky Linux.

