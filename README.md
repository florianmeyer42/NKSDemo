## Using Cloud Volumes ONTAP as persistant storage in a NKS Kubernetes Cluster

## Objectives
This exercise is intended to support a Live demonstration of NetApp Kubernetes Service.
The provided command snippets can be copied and executed in a terminal window during a demonstration.
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

### 3. Deploy CVO instance and connect with K8s cluster
Go to Cloud Central, create a new Cloudmanager and CVO instance in the same region where you created your K8s cluster. Connect your K8s cluster with your CVO instance in Cloud Manger.

```
$ kubectl get nodes

$ kubectl get pods

$ kubectl get pods --all-namespaces

$ kubectl get sc

$ kubectl get pv

$ kubectl get pvc
```

### 4. Create persistant Volume claim for a volume of Storage class netapp-file

```
$ kubectl create -f NKSDemoPVCforNAS.yaml

$ kubectl get pvc
```

### 5. Create K8s deployment

```
$ kubectl create -f NKSDemo_mynginx.yaml

$ kubectl get pods
```

### 6. Expose the deployment to the Internet
To be able to communicate with this webservice from the Internet, a service has to be create which can be exposed onto an external IP address. the deployment has to be exposed to a service
In AWS, the service type "LoadBalancer" creates an ELB (Elastic Load Balancer), which uses a long hostname, not an IP. The command ```kubectl describe service``` shows this long hostname.

```
$ kubectl expose deployment my-nginx --type=LoadBalancer --name=my-service
service "my-service” exposed

$ kubectl describe service my-service
Name:                     my-service
Namespace:                default
Labels:                   run=my-nginx
Annotations:              <none>
Selector:                 run=my-nginx
Type:                     LoadBalancer
IP:                       10.3.0.202
LoadBalancer Ingress:     ade604ef93a9911e9a96f0e163bff36e-758772448.us-east-1.elb.amazonaws.com
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30429/TCP
Endpoints:                10.2.1.5:80,10.2.2.3:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age   From                Message
  ----    ------                ----  ----                -------
  Normal  EnsuringLoadBalancer  27s   service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer   25s   service-controller  Ensured load balancer
```
It may take a couple of minutes for AWS to the create this LoadBalancer service IP.

### 7. verify that this container has mounted a NFS volume from CVO
The nginx webservice runs in a docker container. To verify if this container has a  NFS volume mounted from CVO, connect to that container and run the mount command.
NKS uses Trident to provison storage, all volumes provisioned by this service have "trident" in its name.


```
$ kubectl get pods

$ kubectl exec -it my-nginx-b7b56756c-qdrhr sh
# cd /usr/share/nginx/html

# mount | grep trident
172.23.1.254:/netyrsrzfh_trident_default_persistent_volume_claim_nas_8b126 on /usr/share/nginx/html type nfs4 (rw,relatime,vers=4.1,rsize=65536,wsize=65536,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=172.23.1.226,local_lock=none,addr=172.23.1.254)

# echo "<h1>the quick brown fox jumps over the lazy dog on host $HOSTNAME </h1>" > index.html

$ kubectl get pods
```
