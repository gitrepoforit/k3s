Sure! Below is a document with all the troubleshooting steps for your K3s cluster, including both previous steps and recent steps for resolving the "NotReady" status. It also includes steps to configure `kubectl` for accessing your K3s cluster.

---

# K3s Cluster Troubleshooting Guide

This document outlines steps to troubleshoot and verify the correct functioning of a K3s cluster. It includes general verification steps as well as steps to resolve specific issues like nodes showing a `NotReady` status.

---

## Table of Contents

- [Verify Cluster Setup](#verify-cluster-setup)
  - [Check Cluster Health](#check-cluster-health)
  - [Check K3s Service Status](#check-k3s-service-status)
  - [Verify kubectl Configuration](#verify-kubectl-configuration)
  - [Check Connectivity Between Nodes](#check-connectivity-between-nodes)
  - [Check Network Pods](#check-network-pods)
  - [Check Resource Utilization](#check-resource-utilization)
- [Troubleshooting NotReady Nodes](#troubleshooting-notready-nodes)
  - [K3s Service Status](#k3s-service-status)
  - [Node Logs](#node-logs)
  - [Node Labeling](#node-labeling)
  - [Cluster Networking](#cluster-networking)
  - [K3s Agent on Control Plane](#k3s-agent-on-control-plane)
  - [DNS and CoreDNS Check](#dns-and-coredns-check)

---

## Verify Cluster Setup

### 1. **Check Cluster Health**

To check the overall health of your K3s cluster:

```bash
kubectl get nodes
```

Ensure all nodes are in the `Ready` state. If any node shows `NotReady`, check the troubleshooting steps below.

### 2. **Check K3s Service Status**

Ensure that the K3s service is running properly on all nodes:

```bash
sudo systemctl status k3s
```

If the service is not running, restart it:

```bash
sudo systemctl restart k3s
```

Check the service logs for any errors:

```bash
sudo journalctl -xeu k3s.service
```

### 3. **Verify kubectl Configuration**

To ensure `kubectl` is properly configured for your K3s cluster:

1. The K3s installation automatically sets up the `kubectl` config in `/etc/rancher/k3s/k3s.yaml`. To configure your local machine:
   - Copy the `k3s.yaml` from the control plane node to your local machine:
   
     ```bash
     scp root@<control-plane-ip>:/etc/rancher/k3s/k3s.yaml ~/.kube/config
     ```
   
   - Modify the `server` field in the `~/.kube/config` file to point to the control plane's IP:

     ```yaml
     server: https://<control-plane-ip>:6443
     ```

2. Verify `kubectl` access:

   ```bash
   kubectl get nodes
   ```

### 4. **Check Connectivity Between Nodes**

Verify that all nodes can connect to the API server on the control plane node (`6443`):

```bash
curl -k https://<control-plane-ip>:6443/healthz
```

Ensure the API is reachable and responding with `ok`.

### 5. **Check Network Pods**

Verify that all network-related pods (e.g., Flannel, CoreDNS) are running and healthy:

```bash
kubectl get pods -n kube-system
```

Ensure no pods are stuck in `Pending` or `CrashLoopBackOff` states.

### 6. **Check Resource Utilization**

Make sure that all nodes have sufficient resources available:

- Disk space:
  
  ```bash
  df -h
  ```

- Memory and CPU:
  
  ```bash
  free -m
  ```

---

## Troubleshooting NotReady Nodes

If nodes are showing a `NotReady` status, follow these steps:

### 1. **K3s Service Status**

Ensure that K3s is running on the affected nodes. Check the service status:

```bash
sudo systemctl status k3s
```

If the service is not running or failing, check the logs for errors:

```bash
sudo journalctl -xeu k3s.service
```

### 2. **Node Logs**

Check the logs on the affected nodes to identify specific errors:

```bash
sudo journalctl -u k3s
```

Look for errors related to network communication, certificates, or kubelet issues.

### 3. **Node Labeling**

Check if the nodes are labeled correctly. For control plane nodes, they should have the `control-plane` role.

If not, manually label the nodes:

```bash
kubectl label node controlplane2 node-role.kubernetes.io/control-plane=true
kubectl label node controlplane3 node-role.kubernetes.io/control-plane=true
```

### 4. **Cluster Networking**

Verify that the nodes are able to communicate with the API server and other nodes. You can check network components like `flannel` or `calico`:

```bash
kubectl get pods -n kube-system
```

If network-related pods are failing, investigate those pods for more details.

### 5. **K3s Agent on Control Plane**

Ensure that `k3s-agent` is **not** running on control plane nodes. If it is, stop and disable the `k3s-agent` service:

```bash
sudo systemctl stop k3s-agent
sudo systemctl disable k3s-agent
```

### 6. **DNS and CoreDNS Check**

DNS issues can also cause nodes to become `NotReady`. Ensure that `CoreDNS` is running properly:

```bash
kubectl get pods -n kube-system | grep coredns
```

If CoreDNS is not running or is failing, restart the pods and check the logs.

---

## Additional Steps for a Healthy Cluster

1. **Ensure the cluster is using SQLite**:
   On the control plane nodes, verify that the datastore is correctly using SQLite:

   ```bash
   if grep -q 'etcd' /etc/rancher/k3s/k3s.yaml; then
       echo "K3s is using etcd, which is incorrect."
       exit 1
   fi
   ```

2. **Ensure sufficient resources**:
   Ensure that no node is running out of memory, CPU, or disk space.

3. **Restart K3s if necessary**:
   If any changes are made, such as configuration updates, restart K3s on the affected nodes:

   ```bash
   sudo systemctl restart k3s
   ```

---

## Conclusion

This document provides comprehensive steps for both verifying a healthy K3s cluster and resolving issues such as nodes showing a `NotReady` status. Follow the steps outlined here to maintain the operational status of your cluster.
