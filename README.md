## Using Cloud Volumes ONTAP as persistant storage in a NKS Kubernetes Cluster

## Objectives
This exercise focuses on enabling you to do the following:
- Create a simple service in a NKS deployed Kubernetes Cluster. As an example, this K8s deployment will create a simple Webservice, using nginx.

## Requirements
This exercise focuses on the deployment of a simple K8s service using NKS (NetApp Kubernetes Service) and CVO (Cloud Volumes ONTAP). It is required that you are a registered user of
- NetApp Cloud Services: CVO and NKS
- AWS (Amazon Web Services)

### 1. export KUBECONFIG
After you downloaded the kubeconfig.txt for your K8s cluster, use the file to export KUBECONFIG for your local kubectl client.

```
$ export KUBECONFIG=kubeconfig.txt
```

### 2. Verify your K8s Cluster
Some things to check before you connect a CVO instance with your Kubernetes Cluster.
- Running nodes in your K8s Cluster
- Running pods in your namespace and in all namespaces
- Available Storage classes
- Available persistant Volumes (PVs) and persistant Volume claims (PVCs)

```
$ kubectl get nodes

$ kubectl get pods

$ kubectl get pods --all-namespaces

$ kubectl get sc

$ kubectl get pv

$ kubectl get pvc

```

### Deploy CVO instance and connect with K8s cluster
Go o Cloud Central, create a new Cloudmanager and CVO instance in the same rewgion where you created your K8s cluster. Connect your K8s cluster with your CVO instance in Cloud Manger. This is not covered here. Now after sucessful creation 

```
$ kubectl get sc

$ kubectl get pv

$ kubectl create -f NKSDemoPVCforNAS.yaml

$ kubectl get pvc

$ kubectl create -f NKSDemo_mynginx.yaml

$ kubectl get pods

$ kubectl expose deployment my-nginx --type=LoadBalancer --name=my-service
service "my-service” exposed

$ kubectl describe service my-service

$ kubectl get pods

$ kubectl exec -it my-nginx-b7b56756c-qdrhr sh
# cd /usr/share/nginx/html

# mount | grep trident
172.23.1.254:/netyrsrzfh_trident_default_persistent_volume_claim_nas_8b126 on /usr/share/nginx/html type nfs4 (rw,relatime,vers=4.1,rsize=65536,wsize=65536,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=172.23.1.226,local_lock=none,addr=172.23.1.254)

# echo "<h1>the quick brown fox jumps over the lazy dog on host $HOSTNAME </h1>" > index.html

$ kubectl get pods
```
