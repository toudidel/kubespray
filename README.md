## kubespray (ubuntu server)

1. Clone the kubespray repo

        git clone https://github.com/kubernetes-sigs/kubespray
        cd kubespray

2. (optional) Install pip if not yet installed

        sudo apt install python3-pip

3. Install required components

        sudo pip install -r requirements.txt

In case of error `error: externally-managed-environment` use pip's argument `--break-system-packages`:

        sudo pip install --break-system-packages -r requirements.txt

4. Copy example and create new inventory for Ansible

        cp -rfp inventory/sample inventory/mycluster

5. Declare your nodes IP addresses

        declare -a IPS=(192.168.10.81 192.168.10.82 192.168.10.83)

6. Prepare inventory

        CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

7. (optional) if an error `No module named 'ruamel'` occured, install missing package and try again to create the
   inventory.

        pip3 install ruamel.yaml

   or add argument `--break-system-packages` or you can use directly `apt` to install the package:

        sudo apt install python3-ruamel.yaml

8. Edit file `inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml` where in line 70 (`kube_network_plugin`)
   change `calico` to `cilium` ([here](https://metallb.universe.tf/installation/network-addons/) description about
   network addons compatibility and why we change Calico to another plugin)
7. Edit file `inventory/mycluster/hosts.yaml` where line 19 should be commented, because we do not want to have 2 master
   nodes.

        # node2

8. Run the Ansible playbook to create the cluster

        ansible-playbook -K -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml

   Argument `-K` allows you to input the password


9. (optional) if something goes wrong you can reset your installation:

         ansible-playbook -K -i inventory/mycluster/hosts.yaml --become --become-user=root reset.yml

10. Copy kube config to local machine. File location is `<MasterNodeIpAddress>:/root/.kube/config`
11. Edit this file and in line 5 replace IP address with IP of your master node (node1).
12. Copy kube config to default location `~/.kube/config` (folder may not yet exist) or set env variable `KUBECONFIG`
    with file location:

         export KUBECONFIG=~/config

13. (optional) install kubectl

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        kubectl version --client

15. Now you should be able to run command on your k8s cluster. For example:

         kubectl get node

## k8s examples

```shell
kubectl run hostinfo --image=finconreply/hostinfo
kubectl expose pod hostinfo --port 8080 --target-port 8080

kubectl create deployment hostinfo --image=finconreply/hostinfo --replicas 3
kubectl expose deployment hostinfo --port 8080 --target-port 8080

kubectl port-forward svc/hostinfo 38000:8080

kubectl delete service hostinfo
kubectl delete deployment hostinfo
```

```shell
kubectl create -f deployment.yaml
kubectl create -f service.yaml

kubectl port-forward svc/hostinfo 38000:8080

kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```
