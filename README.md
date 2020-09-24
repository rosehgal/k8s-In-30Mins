<p align="center">
</p>

# K8s in 30 mins
This is not exclusive guide to learn Kubernetes from scratch, rahter this is just a small guide to quickly start with Kubernetes and deploy a very simple application on single workload VM.

## This post talks about:
1. [**Setting up Kubernetes cluster in VM**](#Setting-up-Kubernetes-cluster-in-VM) : 1 VM cluster
    - Spining up a virtual machine with [**Vagrant**](https://www.vagrantup.com/docs/installation) : 2GB RAM + 2CPU cores(Atleat)
    - [Understanding](#What-are-kube*):
        - **kubeadm**
        - **kubelet**
        - **kubectl**
1. **Kuberenetes pods**: How are they different than Docker containers.
    - Moving from Docker container to Kubernetes pods.
1. **Kubernetes Resource**:
    - Pods
    - Deployment
    - Services
    - Replicaset
    - Namespaces
    - Context
    - Config
1. **Kubernetes network manager**
    - I will pick up the plugin called [Flannel](https://github.com/coreos/flannel#flannel).
1. **Stateless Workload**
    - Replicasets & Deployments
1. **Stateful Workloads**
    - Persistent Volumes
    - Persistent Volume Claims
1. **Deploying a simple Java sprinboot app in kubernetes cluster**
    - Java app deployment with Mysql PV & PVC.
    - Setting up LB service to connect to Springboot application.
        - A short discussion about Cloud Config Manager(CCM)
1. **Understanding**:
    - Namespaces
    - Context
    - Config
1. **Next steps**


## Setting up Kubernetes cluster in VM
1. Download the Vagrant [File](Vagrantfile).
1. Download Virtual box and install from [here](https://www.virtualbox.org/).
1. Download and install [Vagrant](https://www.vagrantup.com/downloads).
1. In the terminal run, the two command to get the VM up and running, with out any config :smile:
    ```bash
    # In the same directory where you have downloaded Vagrantfile, run
    vagrant up
    vagrant ssh
    ```
    This will download the Ubuntu box image and do the entire setup for you with the help of virtual box. It just need virtual box installed.
1. The Vagrantfile comes preconfigured with **kubeadm**, **kubelet**, **kubectl**
1. Check if kubernetes cluster is perfectly installed.
    ```bash
    root@vagrant:/home/vagrant# kubectl version -o json
    {
      "clientVersion": {
        "major": "1",
        "minor": "19",
        "gitVersion": "v1.19.2",
        "gitCommit": "f5743093fd1c663cb0cbc89748f730662345d44d",
        "gitTreeState": "clean",
        "buildDate": "2020-09-16T13:41:02Z",
        "goVersion": "go1.15",
        "compiler": "gc",
        "platform": "linux/amd64"
      },
      "serverVersion": {
        "major": "1",
        "minor": "19",
        "gitVersion": "v1.19.2",
        "gitCommit": "f5743093fd1c663cb0cbc89748f730662345d44d",
        "gitTreeState": "clean",
        "buildDate": "2020-09-16T13:32:58Z",
        "goVersion": "go1.15",
        "compiler": "gc",
        "platform": "linux/amd64"
      }
    }
    ```
1. Start the Kubernetes cluster master node.
    ```bash
    # This will spin up Kubernetes cluster with CIDR: 10.244.0.0/16
    root@vagrant:/home/vagrant# kubeadm init --pod-network-cidr=10.244.0.0/16
    kubeadm join 10.0.2.15:6443 --token 3m5dsc.toup1iv7670ya7wc --discovery-token-ca-cert-hash sha256:73f4983d43f9618522eaccf014205f969e3bacd76c98dd0c

    root@vagrant:/home/vagrant# mkdir -p $HOME/.kube
    root@vagrant:/home/vagrant# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    root@vagrant:/home/vagrant# sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
1. Conenct other VM to this cluster: Not required in case of single VM cluster. For this run perfectly, make sure:
    - VM to VM connectivity is there.
    - All there kube-* are installed in VM.
    ```bash
    kubeadm join 10.0.2.15:6443 --token 3m5dsc.toup1iv7670ya7wc --discovery-token-ca-cert-hash sha256:73f4983d43f9618522eaccf014205f969e3bacd76c98dd0c
    ```
1. At this point, Kubernetes is installed and cluster master is up, but still we need a agent to provision and manager network for new nodes for us, This is where [Flannel](https://github.com/coreos/flannel#flannel) comes to rescue. Install Flannel to manager docker network for pods.
    ```bash
    kubectl apply -f \
        https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    ```
1. This step applies, if we wish to use, **our master node as worker as well**. Which is yes in our case:
    ```
    kubectl taint nodes $(hostname) node-role.kubernetes.io/master:NoSchedule-

    # If everything goes well, you will see something like this.
    root@vagrant:/home/vagrant# kubectl get node
    NAME      STATUS   ROLES    AGE     VERSION
    vagrant   Ready    master   3m40s   v1.19.2
    ```


> Run all the commands from root shell.


### What are kube*
Kubernetes runs in client server model, similar to the way the docker runs. Kubernetes server exposes kubernetes-api, and each of kubeadm, kubelet and kubectl connect with this kubernetes server api to get the task done. In the master slave model, there are two entities: 
- Control Plane
- Worker Nodes

**Control Plane** : Connects with Worker nodes for resource allocation.  
**Worker nodes**  : Cluster entitiy that actually allocates tasks and run Pods. 

1. **kubeadm**:
    - Sets-up the cluster
    - Connect various worker nodes togather.
2. **kubectl**:
    - It is a client cli.
    - Connects with control plane kubernetes api server and send execution requests to control plane.
3. **kubelet**:
    - Receives request from control planes.
    - Runs in Worker nodes.
    - Runs task over worker nodes.
    - Maintain Pod lifecycle. Not just for pods, but all Kubernetes resources lifecycle.
