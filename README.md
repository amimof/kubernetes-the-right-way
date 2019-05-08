[![Build Status](https://travis-ci.org/amimof/kubernetes-the-right-way.svg?branch=master)](https://travis-ci.org/amimof/kubernetes-the-right-way)

# Kubernetes The Right Way
Install a [Kubernetes](https://kubernetes.io/) cluster with [Ansible](https://www.ansible.com/) on any infrastructure. Have a vanilla, almost-production-ready cluster in no time! 

<p float="left">
  <img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="100"> 
  <img src="https://github.com/bobthebutcher/vendor-icons-svg/raw/master/ansible.svg?sanitize=true" width="100">
</p>

This project aims to provide an automated way of deploying a Kubernetes cluster that isn't configured. This means that configuration and host preparations lies in the hands of the user executing the playbooks. After a successfull install you will have a cluster with:
* [containerd](https://github.com/containerd/containerd) as container runtime
* [runc](https://github.com/opencontainers/runc) for managing containers
* [cni](https://github.com/containernetworking/cni) plugins
* one or more [etcd](https://github.com/etcd-io/etcd) instances
* one or more masters (kube-api, kube-controller-manager, kube-scheduler) installed as services managed by systemd.
* one or more nodes (kubelet)
* a certificate authority for etcd
* a certificate authority for kube components

What you will **not** have:
* Host-level validation and pre-flight checks
* DNS plugin
* Pod networking
* Ingress
* Node roles

As always, it's highly recommended that you verify that your hosting environment meets any requirements before installing Kubernetes and its components.

# Requirements

On the control host
* [Ansible](https://github.com/ansible/ansible) >= 2.4
* Python 2.7
* OpenSSL

On each host in the cluster
* Python 2.7
* ca-certificates

# Variables
There are a few variables that you may set to further customize the deployment. 

| Name 	| Required 	| Default 	| Description 	|
|-------------------------	|----------	|--------------------------------------	|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| `config_path` 	| `False` 	| `~/.ktrw` 	| A path to a directory on the control host where cluster certificates and configuration is created. 	|
| `cluster_hostname` 	| `False` 	| `groups['masters'][0]` 	| The public hostname of the cluster. Defaults to the hostname of the first master in the [inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html). For multi-master installations, the value of cluster_hostname is usually a load balancer. 	|
| `cluster_port` 	| `False` 	| `6443` 	| The port number on which kube-apiserver listens on. 	|
| `cluster_name` 	| `False` 	| `cluster_hostname.split('.')[0]` 	| The name of the cluster, used for identification in `kubectl`. Defaults to the first segment of the `cluster_hostname`. 	|
| `cluster_cidr` 	| `False` 	| `10.19.0.0/16` 	| CIDR Range for Pods in cluster. This effectively sets the `--cluster-cidr` flag on `kube-controller-manager`.	|
| `regenerate_certificates` 	| `False` 	| `False` 	| Set to True to force create certificates. This will overwrite existing certificates. 	|
| `regenerate_keys` 	| `False` 	| `False` 	| Set to True to force create private certificates (keys). This will overwrite existing certificates. 	|
| `flags_apiserver` 	| `False` 	| 	| Additional options to kube-apiserver as an array, for example: `['--enable-admission-plugins=PodSecurityPolicy']`. 	|

# Deploying a cluster
Configure an Ansible [inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) file with the host groups `etcd`, `masters` and `nodes` and assign any host to their respective groups. Have a look at the [examples](https://github.com/amimof/kubernetes-the-right-way/tree/master/example). 

**Note!** If you plan on using `flannel` in your cluster, you must set `cluster_cidr=10.244.0.0/16` in the inventory.

- After you've defined an inventory, run the `install.yml` playbook

```shell
ansible-playbook -i inventory install.yml
```

- Use the kubeconfig in `~/.ktrw/<cluster_name>/kubeconfig` to manage the cluster
```shell
$ KUBECONFIG=~/.ktrw/<cluster_name>/kubeconfig kubectl version --short
Client Version: v1.14.1
Server Version: v1.14.1
```

## Installing additional plugins
After installation, you will have a bare minimum cluster. This means no cluster network or DNS. Refer to the [kubernetes docs](https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy) for more info. The choice is up to you. If you're not sure which ones to use, just stick with `flannel` and `CoreDNS` and you'll be fine.

Deploy `flannel` onto the cluster
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Deploy `CoreDNS` onto the cluster
```
kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml
```

# Cleanup
To remove a cluster run the `cleanup.yml` playbook.
```
ansible-playbook -i inventory cleanup.yml
```

# Generated certs and config
During installation private certificates, public certificates and configuration are generated on the control host, the host that executes the playbook. They are then copied to the hosts during installation but are kept on the control host in `~/.ktrw/`. This way, the cluster can be safely removed and re-installed without having to regenerate the cluster certificates. You may set which directory to store certs and config locally using the `config_path` variable.

# Adding nodes
To add a node to an existing cluster is as easy as adding it to the [inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) and running `install.yml` again.

# Version matrix

| Name                      | Version   | Role       |
| ------------------------- | --------- | ---------- |
| cni                       | 0.7.5     | node       |
| containerd                | 1.2.5     | node       |
| etcd                      | 3.3.12    | etcd       |
| kube-apiserver            | 1.14.1    | master     |
| kube-controller-manager   | 1.14.1    | master     |
| kube-scheduler            | 1.14.1    | master     |
| kube-proxy                | 1.14.1    | node       |
| kubelet                   | 1.14.1    | node       |
| runc                      | 1.0.0-rc6 | node       |

# How to contribute
This project is MIT licensed and accepts contributions via GitHub pull requests.

