# K3s Raspberry Pi Cluster Ansible Playbooks

This repository contains Ansible playbooks designed to install and configure a K3s cluster on a set of Raspberry Pi devices. The playbooks automate the process of setting up control plane and worker nodes for a fully functioning Kubernetes environment.

## Inventory

Before running the playbooks, ensure that your ./inventory file is updated with the correct IP addresses and hostnames for your Raspberry Pi devices. The inventory defines the control plane and worker nodes.

Example inventory format:

```ini
[k8s_controlplane]
controlplane1 ansible_host=192.168.1.91
controlplane2 ansible_host=192.168.1.86
controlplane3 ansible_host=192.168.1.90

[k8s_worker]
worker1 ansible_host=192.168.1.87
worker2 ansible_host=192.168.1.82

[k8s:children]
k8s_controlplane
k8s_worker

## Host key checking

To avoid host key issues running from a pi node host key checking can be disabled via:

Edit the config file shown in:

ansible-config dump

CONFIG_FILE() = /etc/ansible/ansible.cfg

ansible.cfg contents:

[defaults]
host_key_checking = False

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no

# k3s
# k3s
