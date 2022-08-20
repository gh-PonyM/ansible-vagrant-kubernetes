# Kubernetes Starter with vagrant

The following project provides a playbook for setting up a minimal kubernetes cluster. 
It is written in a way that is makes use of `groups`, `group_vars` and thus can be used 
with a real inventory.

## Variables

Have a look at `group_vars/k8s_cluster.yml`.

- `kubernetes_version`: The kubernetes version to use
- `k8s_pod_subnet`: The subnet CIDR for the podcs
- `k8s_interface`: Interface that is used for setting up the cluster network so that all nodes can see each other
- `advertise_address`: IP derived from the `k8s_interface`
- `k8s_fqdn`: Definition of the domain name, used in combination with `advertise_address` added to every other node's `/etc/hosts`
- `kubectl_users`: Define a list of users on the vm to copy the kubectl config to interact with the cluster, defaults to the `ansible_user`

In real situations, you would overwrite `k8s_interface` in your inventory or with `host_vars` because the interface name changes from 
host to host. You can also define the variable `kubectl_users` including the names of all vm users that want to use kubectl 
to interact with the kubernetes api server.  

The ansible config `ansible_cfg` sets some basic config for ansible, e.g. you can inspect the fact cache in `.fact_cache`.

Start all machines:

    vagrant up --no-provision && vagrant provision

Starting the machines this way, we ensure all machines are booted before the get provisioned.

Since libvirt provider supports parallel start of the VM's, you can run this simpler command.

    vagrant up --provider libvirt

**Note**: A small benchmark I ran showed times of **6m25s for virtualbox** v.s. **4m57s with libvirt**.

(Re)Provision them:

    vagrant provision

Enter the controller machine, your kubectl config and bash completion is in place:

    vagrant ssh controller
    kubectl get nodes

You can also just provision the controller first, run the playbook multiple times and 
all should be fine.