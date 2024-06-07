## kubespray (ubuntu server)

1. Clone the kubespray repo

        git clone https://github.com/kubernetes-sigs/kubespray
        cd kubespray

2. Install required components

        sudo pip install -r requirements.txt

In case of error `error: externally-managed-environment` use pip's argument `--break-system-packages`:

        sudo pip install --break-system-packages -r requirements.txt

3. Copy example and create new inventory for Ansible

        cp -rfp inventory/sample inventory/mycluster

3. Declare your nodes IP addresses

        declare -a IPS=(192.168.0.115 192.168.0.117 192.168.0.118)

4. Prepare inventory

        CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

5. (optional) if an error `No module named 'ruamel'` occured, install missing package and try again to create the
   inventory.

        pip3 install ruamel.yaml

   or add argument `--break-system-packages` or you can use directly `apt` to install the package:

        sudo apt install python3-ruamel.yaml

6. Edit file `inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml` where in line 70 (`kube_network_plugin`)
   change `calico` to `cilium` ([here](https://metallb.universe.tf/installation/network-addons/) description about
   network addons compatibility and why we change Calico to another plugin)
7. Edit file `inventory/mycluster/hosts.yaml` where line 19 should be commented, because we do not want to have 2 master
   nodes.

        # node2

8. Run the Ansible playbook to create the cluster

        ansible-playbook -K -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml

   Argument `-K` allows you to input the password


10. (optional) if something goes wrong you can reset your installation:

         ansible-playbook -K -i inventory/mycluster/hosts.yaml --become --become-user=root reset.yml

11. Copy kube config to local machine. File location is `<MasterNodeIpAddress>:/root/.kube/config`
11. Edit this file and in line 5 replace IP address with IP of your master node (node1).
12. Copy kube config to default location `~/.kube/config` (folder may not yet exist) or set env variable `KUBECONFIG`
    with file location:

         export KUBECONFIG=~/config

13. Now you should be able to run command on your k8s cluster. For example:

         kubectl get node

## k8s examples

```shell
kubectl run hostinfo --image=finconreply/hostinfo
kubectl create deployment hostinfo --image=finconreply/hostinfo --replicas 3

kubectl expose pod hostinfo --port 8080 --target-port 8080
kubectl expose deployment hostinfo --port 8080 --target-port 8080

kubectl port-forward svc/hostinfo 38000:8080

kubectl delete service hostinfo
kubectl delete deployment hostinfo
```
