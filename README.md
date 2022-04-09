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

Boot the ISO in a VM.

Watch the installation process:

```
$ while true ; do date ; ssh -o ConnectTimeout=5 core@<fqdn> journalctl -xe -u install_k8s.service -f ; sleep 1 ; done
```

The system will reboot twice before being ready. Until after the first reboot, SSH is not available.

Obtain the kubectl config:

```
$ ssh core@<fqdn> cat .kube/config >~/.kube/config
```

## Add storage

```
$ kubectl apply -f https://openebs.github.io/charts/openebs-operator-lite.yaml
$ kubectl apply -f https://openebs.github.io/charts/openebs-lite-sc.yaml
```

# Non-goals

* Multiple nodes (I'm using this to run Kubernetes on a single Proxmox server, so besides updates, I don't have much use for multiple nodes)
