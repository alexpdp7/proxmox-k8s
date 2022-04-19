# Introduction

A simple Kubernetes setup for Proxmox.

# Usage

Create the install ISO:

```
./geniso <hostname> <fqdn> <pod_network_cidr> <k8s_domain> <iso_name>
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
$ kubectl apply -f https://openebs.github.io/charts/openebs-operator-lite.yaml
$ kubectl apply -f https://openebs.github.io/charts/openebs-lite-sc.yaml
$ kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

# Non-goals

* Multiple nodes (I'm using this to run Kubernetes on a single Proxmox server, so besides updates, I don't have much use for multiple nodes)

# Known issues

Under certain conditions, some images (most notable BusyBox) do not resolve short names like `kube-dns.kube-system` correctly.
