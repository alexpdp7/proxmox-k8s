The `ansible-proxmox-role.yml` file is a sample role you can use to install Kubernetes on Proxmox.

Create an inventory entry for your (new) Kubernetes host.

Define the following variables for the host.

```
---
pod_network: 10.42.43.0/24
lb_domain: <load balancer domain>
k8s_proxmox_host: <inventory name of the target Proxmox host>
k8s_iso_stage: <a writable path on your Proxmox host, for the user Ansible uses>
proxmox_vmid: 100
proxmox_cores: 4
proxmox_memory_mb: 2048
proxmox_onboot: <0 or 1>
ansible_user: core
```

You must disable facts gathering to run the role, because otherwise Ansible will try to connect to the host before it exists.
