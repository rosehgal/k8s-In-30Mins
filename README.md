<p align="center">
<img src="files/static/k8logo.png" height="30%" width="30%" />
<br>
<img src="files/static/in.png" height="30%" width="30%" />
<br>
<img alt="GitHub release (latest by date)" src="https://img.shields.io/github/v/release/r0hi7/k8s-In-30Mins">
<img alt="GitHub code size in bytes" src="https://img.shields.io/github/languages/code-size/r0hi7/k8s-In-30Mins">
<img alt="GitHub" src="https://img.shields.io/github/license/r0hi7/k8s-In-30Mins">
<br>
<img alt="GitHub issues" src="https://img.shields.io/github/issues-raw/r0hi7/k8s-In-30Mins">
<img alt="GitHub stars" src="https://img.shields.io/github/stars/r0hi7/k8s-In-30Mins">
<img alt="Twitter Follow" src="https://img.shields.io/twitter/follow/sehgal_rohit?style=social">
<img alt="GitHub followers" src="https://img.shields.io/github/followers/r0hi7?style=social">
<br>
<a target="_blank" href="https://twitter.com/intent/tweet?text=Learn Kubernetes in 30 mins, with single node cluster @sehgal_rohit. https://github.com/r0hi7/k8s-In-30Mins" title="Share on Twitter"><img src="https://img.shields.io/twitter/url/http/shields.io.svg?style=social&label=Share%20on%20Twitter"></a>
</p>

[![HitCount](http://hits.dwyl.com/r0hi7/k8s-In-30Mins.svg)](http://hits.dwyl.com/r0hi7/k8s-In-30Mins)


# K8s in 30 mins
This is not a comprehensive guide to learn Kubernetes from scratch, rather this is just a small guide/cheat sheet to quickly setup and run applications with Kubernetes and deploy a very simple application on single workload VM. This repo can be served as quick learning manual to understand kubernetes.

#### Prerequisite
- [Linux](https://files.fosswire.com/2007/08/fwunixref.pdf)
- [Dockers](https://docs.docker.com/develop/)
- [YAML](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)

## Table of Contents:
1. [**Setting up Kubernetes cluster in VM (NOT MINIKUBE)**](#Setting-up-Kubernetes-cluster-in-VM) : 1 VM cluster
    - Spining up a virtual machine with [**Vagrant**](https://www.vagrantup.com/docs/installation) : 2GB RAM + 2CPU cores (at least)
    - [Understanding](#What-are-kube):
        - **kubeadm**
        - **kubelet**
        - **kubectl**
1. [**Kuberenetes pods**](#Kubernetes-pods): How are they different than Docker containers.
    - [Moving from Docker container to Kubernetes pods.](#how-to-create-a-pod)
1. [**Kubernetes Resource**](#kubernetes-resources):
    - [Pods](#pods)
    - [Deployment](#deployments)
    - [Replicaset](#replicasets)
    - [Services](#services)
        - [LoadBalancer Service](#loadbalancer-service)
1. [**Kubernetes network manager**](#kubernetes-network-manager)
    - I will pick up the plugin called [Flannel](https://github.com/coreos/flannel#flannel).
1. [**Stateless Workload**](#stateless-workloads)
    - Replicasets & Deployments
1. [**Stateful Workloads**](#stateful-workloads)
    - [Persistent Volumes](#persistent-volumes)
    - [Persistent Volume Claims](#persistent-volume-claims)
1. [**Deploying End-to-End Service in Kubernetes cluster**](#sample-application-example)
    - [MySQL](#mysql-resource):
        - [PV](#step-1-create-pv-for-mysql-db)
        - [PVC](#step-2-create-pvc-for-pv)
        - [Deployment](#step-3-create-mysql-deployment-spec)
        - [Service](#step-4-expose-mysql-server-via-service)
    - [Sprinboot Application](#springboot-application):
        - Stateless Workload
            - [Custom Docker Image](files/Dockerfile)
        - [Replica based Deployment & link with DB Service](#step-1-build-and-deploy-appserver)
        - [Access Springboot Service outside Pods](#step-2-expose-appserver-service-via-service-type-lb-to-host)
    - [Infrastructure as a Code](#Infrastructure-as-a-code) 
        - [MySQL Full Spec](#mysql-full-spec)
        - [AppServer Full Spec](#appserver-full-spec)
1. [**Understanding** advance kubernetes resources](#advance-kubernetes-resources):
    - [Namespaces](#namespaces)
        - [Create Namespace and Add Resource](#creating-namespace-&-adding-resource) 
    - [Context](#context)
1. [**Cheat sheet**](#cheat-sheet)
1. [**Next steps**](#next-steps)



## Setting up Kubernetes cluster in VM
1. Download the Vagrant [File](Vagrantfile).
1. Download Virtual box and install from [here](https://www.virtualbox.org/).
1. Download and install [Vagrant](https://www.vagrantup.com/downloads).
1. In the terminal, run the two command to get the VM up and running, with out any configuration :smile:
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
    root@vagrant:/home/vagrant# kubectl taint nodes $(hostname) node-role.kubernetes.io/master:NoSchedule-

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

## Kubernetes pods

- Pods run multiple containers.
- Pods abstract out multilpe containers into single unit.
- If two service in pods are both exposing service on same port, the other one wont spin up and it will fail.
- The unit of Kubernetes work load is called Pod. 

### How to create a pod
You can create a simple nginx pod with following yaml spec. Save this in file name : [pod.yml](files/pod.yml)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```
|Key name| Key Description|
|---|---|
|`apiVersion`|Kubernetes server API|  
|`kind`|Kubernetes Resource type: `Pod`|  
|`metadata.name`|Name of Kubernetes Pod|
|`spec.container.name`|Name of Container which will run in a Pod|
|`spec.container.name`|Name of docker image to run| 

Run this Pod spec with. `kubectl apply -f pod.yml`
```bash
root@vagrant:/home/vagrant/kubedata# kubectl apply -f pod.yaml
pod/nginx created

# If everything goes OK, you will se something like this.

root@vagrant:/home/vagrant/kubedata# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          43s
root@vagrant:/home/vagrant/kubedata#
```
Use : ` kubectl get pods` to get the list of all Pods.

1. Running command into container, running inside Pod. `kubectl exec -it <pod_name> -c <container_name> -- <command>`
    ```
    root@vagrant:/home/vagrant/kubedata# kubectl exec -it nginx -c nginx -- whoami
    root

    root@vagrant:/home/vagrant/kubedata# kubectl exec -it nginx -c nginx -- /bin/sh
    # cat /etc/*-release
    PRETTY_NAME="Debian GNU/Linux 10 (buster)"
    NAME="Debian GNU/Linux"
    VERSION_ID="10"
    VERSION="10 (buster)"
    VERSION_CODENAME=buster
    ID=debian
    HOME_URL="https://www.debian.org/"
    SUPPORT_URL="https://www.debian.org/support"
    BUG_REPORT_URL="https://bugs.debian.org/"
    ```
1. Running multiple container in one pod.
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
      - name: curl
        image: appropriate/curl
        stdin: true
        tty: true
        command: ["/bin/sh"]
    ```
    Save this into [pod-with-two-containers.yml](files/pod-with-two-containers.yml).  
    Run this : `kubectl apply -f pod-with-two-containers.yml`
1. Delete a running pod. `kubectl delete -f pod-with-two-containers.yml`. This will remove the pod mentioned in spec file.
1. Container in a Pod can connect to another container in same pod with `spec.containers.name`.
    ```bash
    root@vagrant:/home/vagrant/kubedata# kubectl exec -it nginx -c curl -- /bin/sh
    # curl nginx
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    #
    ```

## Kubernetes Resources

### Pods
- Fundamental unit of k8s cluster.
- Abstraction for container/multiple-containers, running under single name.
- Discussed in detail : [here](#how-to-create-a-pod)

### Deployments
- A Deployment provides declarative updates for Pods.
- The configuration state in `yml` file, defines how the pods will run in cluster. They can specify:
    - Replicas
    - Resource allocation
    - Connection with Volumes etc.
    - We will see example once we see [replicasets](#replicasets)

### Replicasets
1. Run deployments in replicas.
2. Create [file](files/deployment-replica.yml) with following specification.
    ```yaml
    apiVersion: apps/v1
    
    kind: Deployment
    metadata:
      name: nginx

    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx-app
      template:
        metadata:
          labels:
            app: nginx-app
        spec:
          containers:
          - name: nginx
            image: nginx
    ```

    Notice the difference.

    ```diff
    -- kind: Pod
    ++ kind: Deployment

    ++ spec:
    ++  replicas: 3
    ++  selector:
    ++    matchLabels:
    ++      app: nginx-app
    ```
3. Remove existing pods(if any) `kubectl delete pods --all`, and create deployment.
    ```bash
    root@vagrant:/home/vagrant/kubedata# kubectl apply -f deployment-replica.yml
    deployment.apps/nginx created

    root@vagrant:/home/vagrant/kubedata# kubectl get deployments
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   0/3     3            0           7s

    root@vagrant:/home/vagrant/kubedata# kubectl get deployments -w
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   1/3     3            1           14s
    nginx   2/3     3            2           20s
    ```
4. Get the list of all `deployments`: `kubectl get deployments` or `kubectl get deploy`
4. Get the list of all `replicaset` : `kubectl get replicaset` or ` kubectl get rs`
    ```bash
    root@vagrant:/home/vagrant/kubedata# kubectl get pods
    NAME                    READY   STATUS    RESTARTS   AGE
    nginx-d6ff45774-f84l8   1/1     Running   0          4m59s
    nginx-d6ff45774-gzxfz   1/1     Running   0          4m59s
    nginx-d6ff45774-t69mw   1/1     Running   0          4m59s

    root@vagrant:/home/vagrant/kubedata# kubectl get deploy
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   3/3     3            3           162m

    root@vagrant:/home/vagrant/kubedata# kubectl get replicaset
    NAME              DESIRED   CURRENT   READY   AGE
    nginx-d6ff45774   3         3         3       162m

    root@vagrant:/home/vagrant/kubedata#
    ```
5. Print a detailed description of the selected resources, including related resources such as events or controllers: `kubectl describe <resource_type> <resouce_name>`
6. Get deployment configuration in `JSON` format: `kubectl get deployment nginx -o yaml`.

### Services
- Logical abstraction of Pods and policies to access them.
- They enable loose coupling between dependent Pods. e.g
    - Open Ports.
    - Security Policies between Pod interaction etc.
- Can be created independent of Pod declaration, but usually services linked to one Pod are present in same spec file.
- Lets create a simple service to expose nginx service port to host machine. [File](files/nginx-service.yml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```
- Service declaration starts by augmenting exiting deployment/pod spec with `---`.
- Service and Pod can share same names.
    - Each different resource must have unique amongst themselves.
- The above service, exposes port 80 on host specified by `spec.ports.port` to port 80 of target pod specified by `spec.ports.taregtPort`

```bash
root@vagrant:/home/vagrant/kubedata# kubectl apply -f nginx-service.yml
deployment.apps/nginx unchanged
service/nginx created

root@vagrant:/home/vagrant/kubedata#
```
- Once the service is created:
    - Run : `kubectl get services` to get the list of services.
        ```bash
        root@vagrant:/home/vagrant/kubedata# kubectl get services
        NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
        kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   2d5h
        nginx        ClusterIP   10.104.178.240   <none>        80/TCP    49s
        ```
        Cluster IP is the IP interface of Pod anstraction on host. `curl` cluster IP will connect us to the Pod.
        ```bash
        root@vagrant:/home/vagrant/kubedata# curl 10.104.178.240
        <!DOCTYPE html>
        <html>
        <head>
        <title>Welcome to nginx!</title>
        <style>
            body {
                width: 35em;
                margin: 0 auto;
                font-family: Tahoma, Verdana, Arial, sans-serif;
            }
        </style>
        </head>
        <body>
        <h1>Welcome to nginx!</h1>
        <p>If you see this page, the nginx web server is successfully installed and
        working. Further configuration is required.</p>

        <p>For online documentation and support please refer to
        <a href="http://nginx.org/">nginx.org</a>.<br/>
        Commercial support is available at
        <a href="http://nginx.com/">nginx.com</a>.</p>

        <p><em>Thank you for using nginx.</em></p>
        </body>
        </html>
        ```
    - Run : `kubectl get endpoints` or `kubectl get ep` to get list of exposed endpoints.
        ```bash
        root@vagrant:/home/vagrant/kubedata# kubectl get ep
        NAME         ENDPOINTS                                    AGE
        kubernetes   10.0.2.15:6443                               2d5h
        nginx        10.244.0.10:80,10.244.0.8:80,10.244.0.9:80   2m
        ```
        Since I am running 3 different replicas, we are seeing 3 different Pod IPs.

#### Loadbalancer Service
- Notice External IP in:
    ```bash
    root@vagrant:/home/vagrant/kubedata# kubectl get services
    NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
    kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   2d5h
    nginx        ClusterIP   10.104.178.240   <none>        80/TCP    49s
    ```
- Since we are running this in local setup, we dont have any **CCM**(Cloud Config manager), which can provision external IP for us to connect to the service running inside the Pod.
    - In case of Azure or AWS Cloud providers, the CCM provisions and links external IPs for us.
- So lets do a **hack** here.
    - Update nginx service to LoadBalancer. [File](files/nginx-service-lb.yml)
        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: nginx
        spec:
          type: LoadBalancer
          selector:
            app: nginx-app
          ports:
          - protocol: TCP
            port: 80
            targetPort: 80
        ```
        Notice:
        ```diff
        spec:
        ++ type: LoadBalancer
        ```
    - Apply the config:  `kubectl apply -f nginx-service-lb.yml`
        ```bash
        root@vagrant:/home/vagrant/kubedata# kubectl get svc
        NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
        kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP        2d5h
        nginx        LoadBalancer   10.104.178.240   <pending>     80:32643/TCP   17m
        ```
        Now the state is pending :) 
    - Run `netstat -nltp`, and notice the `kube-proxy`
        ```diff
        ++ tcp        0      0 0.0.0.0:32643           0.0.0.0:*               LISTEN      13095/kube-proxy
           tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      7024/kubelet
        ++ tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      13095/kube-proxy
        ```
        See the magic.
        ```bash
        root@vagrant:/home/vagrant/kubedata# curl 0.0.0.0:32643
        <!DOCTYPE html>
        <html>
        <head>
        <title>Welcome to nginx!</title>
        <style>
            body {
                width: 35em;
                margin: 0 auto;
                font-family: Tahoma, Verdana, Arial, sans-serif;
            }
        </style>
        </head>
        <body>
        <h1>Welcome to nginx!</h1>
        <p>If you see this page, the nginx web server is successfully installed and
        working. Further configuration is required.</p>

        <p>For online documentation and support please refer to
        <a href="http://nginx.org/">nginx.org</a>.<br/>
        Commercial support is available at
        <a href="http://nginx.com/">nginx.com</a>.</p>

        <p><em>Thank you for using nginx.</em></p>
        </body>
        </html>
        ```
        - The `LoadBalancer` exposed the service endpoints out of Kubernetes cluster IP interface and in our vagrant host we can access it now directly :) 
        - The next challenge to to expose this `kube-proxy` interface to host machine. And hack is done, then we can access the service running in Pod(replica set deployment) from our host interface directly.
        - This is how the network now looks like. The port `32643` is not exposed through kube-proxy over host/control-plane node.
            ```bash
                                                              Kubernetes Cluster
                                               +---------------------------------------------+
                                               |                               POD           |
                                               |                           +---------+       |
                                               |                    +------>  NGINX  |       |
                                               |                    |      +---------+       |
                                               |           LB       |                        |
                         +--------------+      |    +---------------+          POD           |
            0.0.0.0:32643|  Kube Proxy  |80    |    |               |      +---------+       |
                    <------------------>----------->+    SERVICE    +------>  NGINX  |       |
                         |              |      |  80|               |      +---------+       |
                         +--------------+      |    +---------------+                        |
                               HOST            |                    |          POD           |
                                               |                    |      +---------+       |
                                               |                    +------>  NGINX  |       |
                                               |                           +---------+       |
                                               +---------------------------------------------+
            ```

## Stateless workloads
- Deployments and Replicasets that we had deployed so far are stateless workloads.
- There is no state related information stored at Pods/Service, so request from kube-proxy via serivce resource can be routed to any of the Pod in the cluster.
- This constitutes stateless workload.
- Next section is to create a Stateful workload.

## Stateful workloads
- Preserve the state of data present on Pods.
- Two situations can be possible:
    - Multi pod stateful workload
        - If multiple pods are connecting to stateful workload, there should be worker based synchronization
        - Else, stateful data may go out of sync.
    - Single pod stateful workload
        - Create persitant volumes
        - Create persitant volume claims to access persitant volumes in a synchronized way, just to prevent ensure data atomicity.

### Persistent Volumes
- PV are like volumes in Docker, just that their lifecycle is independent of Pods.
- This is an API object. Captures details about storage implementation.
- Provised by Kubernetes administrator.
- Way to abstract storage resource.
- Create a persistent volume for MySQL server. [File](files/pv.yml)
    ```yaml
    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: pv
      labels:
        type: local
    spec:
      storageClassName: manual
      capacity:
        storage: 5Gi
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: "/data"
    ```
    This spec specifies the volume is at `/data` on cluster's node.
    Apply it : `kubectl apply -f pv.yml`
    ```bash
    root@vagrant:/home/vagrant/kubedata# kubectl get pv
    NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
    pv     5Gi        RWO            Retain           Available           manual                  62s
    ```


### Persistent Volume Claims
- Storage requeest by a user.
- PVCs consume PV resources.
- Way to access abstract storage.
- PVC can request specific size and access mode: `ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany`

|Access Mode| Meaning|
|---|---|
|ReadWriteOnce | volume can be mounted as read-write by a single node|
|ReadOnlyMany |  volume can be mounted read-only by many nodes|
|ReadWriteMany | volume can be mounted as read-write by many nodes|

- Create a PVC spec. [File](files/pvc.yml)
    ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: pv-claim
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
    ```
    Apply it.
    ```bash
    root@vagrant:/home/vagrant/kubedata# kubectl apply -f pv-claim.yml
    persistentvolumeclaim/pv-claim created

    root@vagrant:/home/vagrant/kubedata# kubectl get pvc
    NAME       STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    pv-claim   Bound    pv       5Gi        RWO            manual         8s

    root@vagrant:/home/vagrant/kubedata# kubectl describe pvc pv-claim
    Name:          pv-claim
    Namespace:     default
    StorageClass:  manual
    Status:        Bound
    Volume:        pv
    Labels:        <none>
    Annotations:   pv.kubernetes.io/bind-completed: yes
                   pv.kubernetes.io/bound-by-controller: yes
    Finalizers:    [kubernetes.io/pvc-protection]
    Capacity:      5Gi
    Access Modes:  RWO
    VolumeMode:    Filesystem
    Mounted By:    <none>
    Events:        <none>
    root@vagrant:/home/vagrant/kubedata#
    ```
    - Pods use PersistentVolumeClaims to request physical storage
    - After creating the PersistentVolumeClaim, the Kubernetes control plane looks for a PersistentVolume that satisfies the claim's requirements. If the control plane finds a suitable PersistentVolume with the same StorageClass, it binds the claim to the volume.

- Lets create a POD which will use PV as Volume using PVC. [File](files/nginx-pod-with-pv.yml)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-with-pvc
spec:
  volumes:
    - name: nginx-pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
  containers:
    - name: nginx-with-pv
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: nginx-pv-storage
```
```bash
root@vagrant:/home/vagrant/kubedata# kubectl get pods nginx-pod-with-pvc
NAME                 READY   STATUS    RESTARTS   AGE
nginx-pod-with-pvc   1/1     Running   0          16s

root@vagrant:/home/vagrant/kubedata# kubectl exec -it nginx-pod-with-pvc -c nginx-with-pv -- /bin/bash
root@nginx-pod-with-pvc:/# curl localhost
Hi PV

```
- The file we just created in storage is made accessible to Nginx POD.


### Summary
```
                                +--------------------------------------+
                                |     +------------+                   |
                                |     |    POD     |        +--------------->
                                |     +-----+------+        |          |    |
                                |           |               |          |    |
                                |           |         +-----+------+   |    v
                                |           |         |     PV     |   |   /data
                                |           |         +------+-----+   |
                                |     +-----v------+         ^         |
                                |     |    PVC     +---------+         |
                                |     +------------+                   |
                                |                                      |
                                +--------------------------------------+
```
- PV to PVC bind is automatic, based on storage class.
- Pod/Deployment/K8s-Resource link to PVC has to has to be done manually in spec file.

## Sample Application Example
1. This End to End setup will include:
  1. MySQL setup through PV and PVC.
  2. Building Custom Dockerfile for sprinboot application.
  3. Creating Deployment for SpringBoot application.
    1. Setup the environment for application to connect to DB.
    2. Setting up PVC setup in deployment.
    3. Creating Serivce for springboot application access outside pod.
      1. Service setup through LB

Once we create spec.yml in bits, we will create a big spec to show our Infrastructure as a Code and deploy that :smile:.

### MySQL Resource

#### Step 1: Create PV for MYSQL DB
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/mysql"   
```

#### Step 2: Create PVC for PV
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

#### Step 3: Create MySQL deployment Spec
```yaml
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: dbserver
  labels:
    app: dbserver
spec:
  selector:
    matchLabels:
      app: dbserver
  template:
    metadata:
      labels:
        app: dbserver
    spec:
      containers:
      - image: mysql
        name: mysql
        imagePullPolicy: Never
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: mysecretpassword
        ports:
        - containerPort: 3306
          name: dbserver
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
```
  - Once the DB server is up, please go adhead and login to MySQL and create `peopledb` for sprinboot application to access.
    - `mysql  -- mysql -u root -pmysecretpassword` & `create database peopledb`

#### Step 4: Expose MySQL server via Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: dbservice
spec:
  selector:
    app: dbserver
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
```
- This will expose this service over/inside cluster for other services to access.

### Springboot Application

#### Step 1: Build and Deploy AppServer
- Build the Docker image with name `appserver` from this [File](files/Dockerfile).
  ```bash
  docker build  -t appserver .
  ```
- Create Deployment spec for appserver.
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: appserver
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: appserver
    template:
      metadata:
        labels:
          app: appserver
      spec:
        containers:
        - name: appserver
          image: appserver
          imagePullPolicy: Never
          env:
          - name: DB_HOST
            value: dbservice
  ```

#### Step 2: Expose AppServer service via Service type LB to host.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: contacts
spec:
  type: LoadBalancer
  selector:
    app: appserver
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### Infrastructure as a Code
#### MySQL Full Spec
- You can find the full spec file here : [File](files/mysql-spec.yml)
  ```yaml
  kind: PersistentVolume
  apiVersion: v1
  metadata:
    name: mysql-pv
    labels:
      type: local
  spec:
    storageClassName: manual
    capacity:
        storage: 5Gi
    accessModes:
      - ReadWriteOnce
    hostPath:
      path: "/data/mysql"   

  ---
  kind: PersistentVolumeClaim
  apiVersion: v1
  metadata:
    name: mysql-pvc
  spec:
    storageClassName: manual
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi

  ---
  apiVersion: apps/v1 
  kind: Deployment
  metadata:
    name: dbserver
    labels:
      app: dbserver
  spec:
    selector:
      matchLabels:
        app: dbserver
    template:
      metadata:
        labels:
          app: dbserver
      spec:
        containers:
        - image: mysql
          name: mysql
          imagePullPolicy: Never
          env:
          - name: MYSQL_ROOT_PASSWORD
            value: mysecretpassword
          ports:
          - containerPort: 3306
            name: dbserver
          volumeMounts:
          - name: mysql-persistent-storage
            mountPath: /var/lib/mysql
        volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pvc
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: dbservice
  spec:
    selector:
      app: dbserver
    ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  ```
- `kuebctl apply -f mysql-spec.yml` :smile:

#### AppServer Full Spec
- You can find the full spec file here: [File](files/appserver-spec.yml)
  ```yaml

  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: appserver
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: appserver
    template:
      metadata:
        labels:
          app: appserver
      spec:
        containers:
        - name: appserver
          image: appserver
          imagePullPolicy: Never
          env:
          - name: DB_HOST
            value: dbservice

  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: contacts
  spec:
    type: LoadBalancer
    selector:
      app: appserver
    ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  ```
  Quickly apply it with `kubectl apply -f appserver-spec.yml`


## Understanding Advance Kubernetes Resources

### Namespace
Namespace are software level cluster virtualization over same physical k8s cluster.  
```bash
  root@vagrant:/home/vagrant# kubectl get ns
  NAME              STATUS   AGE
  default           Active   19d
  kube-node-lease   Active   19d
  kube-public       Active   19d
  kube-system       Active   19d
```

Kubernetes starts with 4 namespaces:
1. **default**: The default namespace for objects with no other namespace.
2. **kube-system**: The namespace for objects created by the Kubernetes system.
3. **kube-public**: This namespace is created automatically and is readable by all users (including those not **authenticated**). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.
4. **kube-node-lease**: This namespace for the lease objects associated with each node which improves the performance of the node heartbeats as the cluster scales.

Get Pods from specific namespace
``kubectl get pods --namespace=default`` OR `kubectl get pods -n default`
```bash
root@vagrant:/home/vagrant# kubectl get pods --namespace=kube-system
NAME                              READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-g9wxg           1/1     Running   5          19d
coredns-f9fd979d6-zrdvs           1/1     Running   5          19d
etcd-vagrant                      1/1     Running   5          19d
kube-apiserver-vagrant            1/1     Running   5          19d
kube-controller-manager-vagrant   1/1     Running   7          19d
kube-flannel-ds-64l2p              1/1     Running   6          19d
kube-proxy-4j4kw                  1/1     Running   5          19d
kube-scheduler-vagrant            1/1     Running   7          19d
```

#### Creating Namespace & Adding resource
- Create namespace : `kubectl create namespace qa`
- Once the namespace is created, just add the metadata field : `namespace: qa`, [File](files/pod-qa.yml)
  ```diff
  apiVersion: v1
  kind: Pod
  metadata:
     name: nginx
  ++ namespace: qa
  spec:
    containers:
    - name: nginx
      image: nginx
  ```
- Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are in some namespaces. However namespace resources are not themselves in a namespace. And low-level resources, such as nodes and persistentVolumes, are not in any namespace.
  - To see the list of resource not in namespace : `kubectl api-resources --namespaced=false`

### Context
- Is a tuple of **cluster**, **user**, **namespace**. This is useful when you connect to multiple clusters from one control plane.
  - Get the current context: `kubectl config get-contexts`
  ```bash
  root@vagrant:/home/vagrant/kubedata# kubectl config get-contexts
  CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
  *         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
  ```
- You can create kubernetes context using config file or using commands.
  - Create a qa-config: `kubectl config set-context dev-env --cluster=kubernetes --user=new-admin --namespace=dev-env`
    ```bash
    root@vagrant:/home/vagrant/kubedata# kubectl config set-context dev-env --cluster=kubernetes --user=new-admin --namespace=dev-env
    Context "dev-env" created.
    ```
    ```bash
    root@vagrant:/home/vagrant/kubedata# kubectl config get-contexts
    CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
             dev-env                       kubernetes   new-admin          dev-env
    *          kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
    ```
  - Now use the created context using : `kubectl config use-context dev-env`
  - All your k8s resource will now be in DEV name space under kubernetes cluster :smile:
    - But to create resource you will need user `new-admin` authentication. This is the user created during context creation.
    - Create username & password for user `new-admin` to use the resource in context and create a role binding: **Run this before switching context**
    `kubectl config set-credentials new-admin --username=adm --password=changeme`
    ```bash
    cat << EOF | kubectl apply -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: new-admin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - apiGroup: rbac.authorization.k8s.io
      kind: User
      name: marry@example.com

    EOF
    ```

## CheatSheet
- I plan to write a simple cheat sheet covering the commands in this repo. But for now Try : [k8s-official-cheat-sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Next Steps
- [In detail K8s Reference](https://kubernetes.io/docs/reference/)
- [API Guide](https://kubernetes.io/docs/reference/)
- [CLI Guide](https://kubernetes.io/docs/reference/)
- [K8s Design Docs](https://kubernetes.io/docs/reference/)
- Raising a PR makes me happy, take that as a next step.
- Issues are more than welcome.
- If you like it, share it.
