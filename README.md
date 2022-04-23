# What?

This is a simple and fast procedure to set up a single node Kubernetes cluster on "bare metal".
It should be possible to have a fully-functioning cluster in less than one hour.

The procedure has been developed and tested under Proxmox, but it could really work under any hypervisor or "true" bare metal.

# Why?

Single node clusters don't make a ton of sense- you are paying the complexity of Kubernetes, without reaping the biggest advantage.

However, some software (like [tobs](https://github.com/timescale/tobs)) is much more complex to deploy without Kubernetes.
Some stuff, like having a self-hosted PaaS or some CI setups, are easier on Kubernetes.
Also, you can learn a lot about Kubernetes on a single node cluster.

There are many interesting projects to run Kubernetes clusters, but using an immutable, minimal OS is very attractive.
Immutable OS typically provide easy ways for provisioning.
In this case, a simple script can create an ISO that sets everything up automatically on start.
I don't think there can be a simpler process with less manual steps.
For example, K3S, while offering interesting features (like replacing etcd with Sqlite) does not "solve" the underlying OS.
This project provisions both the underlying OS *and* Kubernetes in one step.

Projects like Minikube bring a base VM, taking care of the underlying OS.
However, Minikube is geared towards running that VM in an hypervisor in your workstation.
Using Minikube to deploy a VM to hypervisors such as Proxmox does not seem to be a first-class citizen (for now).

Another noteworthy feature is that this setup integrates with existing DHCP.
Most bare metal setups use MetalLB, which requires reserving an IP range.
This uses kube-vip as a load balancer that can request IPs via DHCP.

# Requirements

* Working DHCP/DNS on your network.
  I use dnsmasq because it's easy.
  Basically you need:

  * The Kubernetes host should get an IP address via DHCP, and a resolvable FQDN.
  * You should be able to delegate a DNS zone to the Kubernetes host, using port 30053, *not* 53.
  * You should be able to create a wildcard A record pointing to the Kubernetes host.

* Docker (Podman should also work) on the machine you will use to prepare the cluster.
* An empty cluster consumes less than 1gb of RAM.

# Overview

The `geniso` script creates an ISO image for Fedora CoreOS that will provision a Kubernetes cluster automatically.

The cluster has the following properties:

* Deployment using pure `kubeadm`.
* `~/.ssh/id_rsa.pub` is deployed to the host.
* Uses CRI-O as a container runtime.
* Uses `kube-router` to expose the Kubernetes API.
* Single-node, master and worker.
* Uses `kube-vip` as a DHCP-aware software load balancer.
* Exposes CoreDNS using a NodePort.
* Uses haproxy-ingress as an Ingress, using host networking.

Additionally, this README documents how to install OpenEBS for storage.

# Usage

Create the install ISO:

```
./geniso <hostname> <fqdn> <pod_network_cidr> <k8s_domain> <iso_name>
```

Boot the ISO in a VM.

Watch the installation process:

```
$ while true ; do date ; ssh -o ConnectTimeout=5 core@<fqdn> journalctl -xe -u install_k8s.service -f ; sleep 1 ; done
```

The system will reboot twice before being ready.
Until after the first reboot, SSH is not available.
The whole process should take less than 10 minutes.

Obtain the kubectl config:

```
$ ssh core@<fqdn> cat .kube/config >~/.kube/config
```

## Configure DNS

Apply the following dnsmasq configuration:

```
dhcp-host=<hostname>,<ip>,<hostname>
server=/<k8s_domain>/<ip>#30053
address=/<k8s_ingress_domain>/<ip>

```

* The `dhcp-host` entry fixes the IP of the K8S host, because dnsmasq can only delegate domains to static IPs.
* Delegates the K8S domain to CoreDNS exposed as a NodePort.
* Creates a "wildcard" DNS for Ingress.

Exposing a service using a LoadBalancer will make the service visible at `<service>.<namespace>.<k8s_domain>`.

Create ingresses like:

```
$ kubectl -n <ns> create ingress <ingress_name> --annotation kubernetes.io/ingress.class=haproxy --rule="<subdomain>.<k8s_ingress_domain>/*=<service_name>:<port>,tls"
```

They will be exposed at `<subdomain>.<k8s_ingress_domain>`.

## Add storage

```
$ ./install-openebs
```

# Priorities

Ranked from top to low.
We value all priorities, but when facing a tradeoff between two priorities, the one on top will win.

* Personal-production ready.
  Don't use this for anything where failure means monetary loss.
  But it should be fine for non-public, personal use.
* Simplicity. 
  Right now this is a 300-line script, plus this README.
  It should be possible to provision in a few minutes.
* Light.
  An empty cluster consumes less than 1gb of RAM.
* Security.
  This is meant to be run in a secure network.
  Only expose what you want to expose.
  Only trusted users should access SSH or the Kubernetes API.
* Multiple nodes.
  Although I'd like to run a worker node on an ARM SBC, right now I don't have much use for a fully-redundant clusters.
* Flexibility.
  This is an opinionated "just-works" setup.
  Between many options, we should choose the best consistent to these priorities.

# Known issues

Under certain conditions, some images (most notable BusyBox) do not resolve short names like `kube-dns.kube-system` correctly.
