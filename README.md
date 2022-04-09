# Introduction

A simple Kubernetes setup for Proxmox.

# Usage

Create the install ISO:

```
./geniso <hostname> <fqdn> <pod_network_cidr> <iso_name>
```

* Downloads the latest Fedora CoreOS ISO
* Applies customizations
  * Uses `~/.ssh/id_rsa.pub` for the core user
  * Sets the hostname
  * Sets to autoinstall to `/dev/sda`
  * Installs K8S

Obtain the kubectl config:

```
$ ssh core@<fqdn> cat .kube/config
```

# Non-goals

* Multiple nodes (I'm using this to run Kubernetes on a single Proxmox server, so besides updates, I don't have much use for multiple nodes)
