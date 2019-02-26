---
title: "k3s - Lightweight Kubernetes"
date: 2019-02-05T09:52:46-07:00
name: "menu"
---

Lightweight Kubernetes.  Easy to install, half the memory, all in a binary less than 40mb.

Great for

  - Edge
  - IoT
  - CI
  - ARM
  - Situations where a PhD in k8s clusterology is infeasible

What is this?
---

k3s is intended to be a fully compliant Kubernetes distribution with the following changes:

1. Legacy, alpha, non-default features are removed. Hopefully you shouldn't notice the
   stuff that has been removed.
2. Removed most in-tree plugins (cloud providers and storage plugins) which can be replaced
   with out of tree addons.
3. Add sqlite3 as the default storage mechanism. etcd3 is still available, but not the default.
4. Wrapped in simple launcher that handles a lot of the complexity of TLS and options.
5. Minimal to no OS dependencies (just a sane kernel and cgroup mounts needed). k3s packages required
   dependencies
    * containerd
    * Flannel
    * CoreDNS
    * CNI
    * Host utilities (iptables, socat, etc)

Quick Start
-----------
1. Download `k3s` from latest [release](https://github.com/rancher/k3s/releases/latest), x86_64, armhf, and arm64 are
   supported
2. Run server 

```bash
sudo k3s server &
# Kubeconfig is written to /etc/rancher/k3s/k3s.yaml
sudo k3s kubectl get node

# On a different node run the below. NODE_TOKEN comes from /var/lib/rancher/k3s/server/node-token 
# on your server
sudo k3s agent --server https://myserver:6443 --token ${NODE_TOKEN}

```
