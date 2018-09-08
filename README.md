# Kubernetes The Right Way

Install a Kubernetes cluster with Ansible on any infrastructure. Have a production ready provisioned cluster in no time!

After a successfull install you will have a cluster with:
* [containerd](https://github.com/containerd/containerd) as container runtime
* [runc](https://github.com/opencontainers/runc) for managing containers
* [cni](https://github.com/containernetworking/cni) plugins
* one or more etcd instances
* one or more masters (kube-api, kube-controller-manager, kube-scheduler) installed as services managed by systemd.
* one or more nodes (kubelet)
* a certificate authority for etcd
* a certificate authority for kube components

# Requirements

On the control host
* Ansible
* Python 2.7
* OpenSSL

On each host in the cluster
* Python 2.7
* ca-certificates

# Example
Configure an Ansible inventory file with the host groups `etcd`, `masters` and `nodes` and assign any host to their groups. Following inventory will install all components on a single host:
```
[etcd]
master.mydomain.com

[masters]
master.mydomain.com

[nodes]
master.mydomain.com
```

You may set the variables `cluster_ip`, `cluster_hostname` and `cluster_port` for advanced deployments. For example:
```
[all:vars]
cluster_hostname=masters.mydomain.com
cluster_port=8443
cluster_ip=172.41.1.10

[etcd]
master-1.mydomain.com
master-2.mydomain.com
master-3.mydomain.com

[masters]
master-1.mydomain.com
master-2.mydomain.com
master-3.mydomain.com

[nodes]
node-1.mydomain.com
node-2.mydomain.com
node-3.mydomain.com
node-4.mydomain.com
node-5.mydomain.com
node-6.mydomain.com
``` 

You may also install etcd and master components on separate hosts
```
[etcd]
etcd.mydomain.com

[masters]
master.mydomain.com

[nodes]
node-1.mydomain.com
node-2.mydomain.com
``` 

Run playbook `main.yml`
```
ansible-playbook -i inventory main.yml
```

# Cleanup
To remove a cluster run the `cleanup.yml` playbook
```
ansible-playbook -i inventory clanup.yml
```