### Checking What’s Running on Ports

If you're troubleshooting issues with nodes showing `NotReady` or connectivity problems, it's important to check which services are using critical ports. This can help identify port conflicts, such as the K3s API server failing to start because another process is using the port.

Here are the steps to check port usage and resolve potential conflicts:

### 1. **Check for Services Running on K3s Ports**

The default ports used by K3s are:

- **6443**: Kubernetes API server
- **6444**: Kubernetes aggregated API server
- **8472**: Flannel VXLAN
- **10250**: Kubelet API

You can check what's running on these ports using `lsof` or `netstat`.

#### Example for port `6443` (Kubernetes API Server):

```bash
sudo lsof -i :6443
```

This command will display a list of processes using the port.

Example output:

```
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
k3s     16193 root    7u  IPv4  56387      0t0  TCP controlplane1:6443 (LISTEN)
```

If another process is using the port, you can identify and stop it, or reconfigure K3s to use a different port.

#### Example for port `6444` (K3s aggregated API server):

```bash
sudo lsof -i :6444
```

If you find that another process (e.g., `k3s-agent`) is using this port, you can stop it:

```bash
sudo systemctl stop k3s-agent
sudo systemctl disable k3s-agent
```

### 2. **Check for Open Ports with netstat**

You can also use `netstat` to check open ports and services:

```bash
sudo netstat -tuln | grep LISTEN
```

This command shows all TCP and UDP ports in the `LISTEN` state, helping you identify which services are occupying important ports.

Example output:

```
tcp        0      0 0.0.0.0:6443            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:6444          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:10250           0.0.0.0:*               LISTEN     
```

### 3. **Kill Processes Blocking Ports**

If you find an unwanted process blocking a port (for example, a process occupying `6444`), you can kill the process using its PID:

```bash
sudo kill -9 <PID>
```

Replace `<PID>` with the process ID from the `lsof` or `netstat` output.

### 4. **Verify After Stopping Services**

Once the blocking process is stopped, restart the K3s service to ensure it's using the correct ports:

```bash
sudo systemctl restart k3s
```

You can then verify the services are running on the correct ports:

```bash
sudo lsof -i :6443
sudo lsof -i :6444
```

This ensures there are no port conflicts preventing K3s from starting correctly.
