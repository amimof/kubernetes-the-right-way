[![Build Status](https://travis-ci.org/amimof/kubernetes-the-right-way.svg?branch=master)](https://travis-ci.org/amimof/kubernetes-the-right-way)

# Kubernetes The Right Way
Install a Kubernetes cluster with Ansible on any infrastructure. Have a production ready cluster in no time!

<p float="left">
  <img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="100"> 
  <img src="https://github.com/bobthebutcher/vendor-icons-svg/raw/master/ansible.svg?sanitize=true" width="100">
</p>


After a successfull install you will have a cluster with:
* [containerd](https://github.com/containerd/containerd) as container runtime
* [runc](https://github.com/opencontainers/runc) for managing containers
* [cni](https://github.com/containernetworking/cni) plugins
* one or more [etcd](https://github.com/etcd-io/etcd) instances
* one or more masters (kube-api, kube-controller-manager, kube-scheduler) installed as services managed by systemd.
* one or more nodes (kubelet)
* a certificate authority for etcd
* a certificate authority for kube components

# Requirements

On the control host
* [Ansible](https://github.com/ansible/ansible) >= 2.4
* Python 2.7
* OpenSSL

On each host in the cluster
* Python 2.7
* ca-certificates

# Generated certificates and configuration
Private certificates, public certificates and configuration are generated on the control host, the host that executes the playbook. They are copied to the hosts during installation but are kept on the control host in `~/.ktrw/`. This way, the cluster can be safely removed and re-installed without having to regenerate the cluster certificates. You may set which directory to store certs and config locally using the `config_path` variable.

# Variables
There are a few variabels that you may set to furher customize the deployment. 

| Name 	| Required 	| Default 	| Description 	|
|-------------------------	|----------	|--------------------------------------	|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| `config_path` 	| `False` 	| `~/.ktrw` 	| A path to a directory on the control host were cluster certificates and configuration is created. 	|
| `cluster_hostname` 	| `False` 	| `groups['masters'][0]` 	| The public hostname of the cluster. Defaults to the hostname of the first master in the [inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html). For multi-master installations, the value of cluster_hostname is usually a load balancer. 	|
| `cluster_port` 	| `False` 	| `6443` 	| The port number on which kube-apiserver listens on. 	|
| `cluster_name` 	| `False` 	| `cluster_hostname.split('.')[0]` 	| The name of the cluster, used for identification in `kubectl`. Defaults to the first segment of the `cluster_hostname`. 	|
| `regenerate_certificates` 	| `False` 	| `False` 	| Set to True to force create certificates. This will overwrite existing certificates. 	|
| `regenerate_keys` 	| `False` 	| `False` 	| Set to True to force create private certificates (keys). This will overwrite existing certificates. 	|

# Deploying a cluster
After you have defined an [inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) file, run the `install.yml` playbook. Have a look at the example inventories below.
```
ansible-playbook -i inventory install.yml
```

# Cleanup
To remove a cluster run the `cleanup.yml` playbook.
```
ansible-playbook -i inventory clanup.yml
```

# Examples
Configure an Ansible [inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) file with the host groups `etcd`, `masters` and `nodes` and assign any host to their respective groups. 

## Single host
```
[etcd]
master.mydomain.com

[masters]
master.mydomain.com

[nodes]
master.mydomain.com
```

## Multiple masters 
```
[all:vars]
cluster_hostname=masters.mydomain.com

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
``` 

## Multiple masters and multiple etcd
```
[all:vars]
cluster_hostname=masters.mydomain.com

[etcd]
etcd-1.mydomain.com
etcd-2.mydomain.com
etcd-3.mydomain.com

[masters]
master-1.mydomain.com
master-2.mydomain.com
master-3.mydomain.com

[nodes]
node-1.mydomain.com
node-2.mydomain.com
``` 

## Adding nodes
To add a node to an existing cluster is as easy as adding it to the [inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) and running `install.yml` again.

# Version matrix

| Name                      | Version   | Role       |
| ------------------------- | --------- | ---------- |
| cni                       | 0.6.0     | node       |
| containerd                | 1.2.1     | node       |
| etcd                      | 3.3.9     | etcd       |
| kube-apiserver            | 1.13.1    | master     |
| kube-controller-manager   | 1.13.1    | master     |
| kube-scheduler            | 1.13.1    | master     |
| kube-proxy                | 1.13.1    | node       |
| kubelet                   | 1.13.1    | node       |
| kubectl                   | 1.13.1    | controller |
| runc                      | 1.0.0-rc6 | node       |

# How to contribute
This project is MIT licensed and accepts contributions via GitHub pull requests.

