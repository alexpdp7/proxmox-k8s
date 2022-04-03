# Introduction

A simple script to simplify installing K8S on Proxmox

# Usage

```
./geniso <hostname> <fqdn> <iso_name>
```

* Downloads the latest Fedora CoreOS ISO
* Applies customizations
  * Uses `~/.ssh/id_rsa.pub` for the core user
  * Sets the hostname
  * Sets to autoinstall to `/dev/sda`
  * Installs k0s

Obtain the kubectl config:

```
$ ssh core@<fqdn> cat kubeconfig
```

# TODO

* Not really stable yet. The node can get stuck in NotReady, although a reboot can fix the issue.

# Non-goals

* Multiple nodes (I'm using this to run Kubernetes on a single Proxmox server, so besides updates, I don't have much use for multiple nodes)
