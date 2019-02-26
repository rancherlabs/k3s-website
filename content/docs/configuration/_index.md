---
title: "Configuration"
weight: 3
---

This section contains information on using k3s with various configurations.

Open Ports/Network Security
---------------------------

The server needs port 6443 to be accessible by the nodes.  The nodes need to be able to reach
other nodes over UDP port 4789.  This is used for flannel VXLAN.  If you don't use flannel
and provide your own custom CNI, then 4789 is not needed by k3s. The node should not listen
on any other port.  k3s uses reverse tunneling such that the nodes make outbound connections
to the server and all kubelet traffic runs through that tunnel.

IMPORTANT. The VXLAN port on nodes should not be exposed to the world, it opens up your
cluster network to accessed by anyone.  Run your nodes behind a firewall/security group that
disables access to port 4789.


Server HA
---------
Just don't right now :)  It's currently broken.

    
Running in Docker (and docker-compose)
-----------------

I wouldn't be me if I couldn't run my cluster in Docker.  `rancher/k3s` images are available
to run k3s server and agent from Docker.  A `docker-compose.yml` is in the root of this repo that
serves as an example how to run k3s from Docker.  To run from `docker-compose` from this repo run

    docker-compose up --scale node=3
    # kubeconfig is written to current dir
    kubectl --kubeconfig kubeconfig.yaml get node
    
    NAME           STATUS   ROLES    AGE   VERSION
    497278a2d6a2   Ready    <none>   11s   v1.13.2-k3s2
    d54c8b17c055   Ready    <none>   11s   v1.13.2-k3s2
    db7a5a5a5bdd   Ready    <none>   12s   v1.13.2-k3s2

    
Hyperkube
--------

k3s is bundled in a nice wrapper to remove the majority of the headache of running k8s. If
you don't want that wrapper and just want a smaller k8s distro, the releases includes
the `hyperkube` binary you can use.  It's then up to you to know how to use `hyperkube`. If
you want individual binaries you will need to compile them yourself from source
    
containerd and Docker
----------

k3s includes and defaults to containerd. Why? Because it's just plain better. If you want to
run with Docker first stop and think, "Really? Do I really want more headache?" If still
yes then you just need to run the agent with the `--docker` flag

     k3s agent -u ${SERVER_URL} -t ${NODE_TOKEN} --docker &
     
systemd
-------

If you are bound by the shackles of systemd (as most of us are), there is a sample unit file
in the root of this repo `k3s.service` which is as follows

```ini
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
After=network.target

[Service]
ExecStartPre=-/sbin/modprobe br_netfilter
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/k3s server
KillMode=process
Delegate=yes
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity

[Install]
WantedBy=multi-user.target
```

Flannel
-------

Flannel is included by default, if you don't want flannel then run the agent with `--no-flannel` as follows

     k3s agent -u ${SERVER_URL} -t ${NODE_TOKEN} --no-flannel &
     
In this setup you will still be required to install your own CNI driver.  More info [here](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network)

CoreDNS
-------

CoreDNS is deployed on start of the agent, to disable add `--no-deploy coredns` to the server

     k3s server --no-deploy coredns
     
If you don't install CoreDNS you will need to install a cluster DNS provider yourself.

Service Load Balancer
---------------------

k3s includes a basic service load balancer that uses available host ports.  If you try to create
a load balancer that listens on port 80, for example, it will try to find a free host in the cluster
for port 80.  If no port is available the load balancer will stay in Pending.

To disable the embedded service load balancer (if you wish to use a different implementation like
MetalLB) just add `--no-deploy=servicelb` to the server on startup.
