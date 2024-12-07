# orchestration
helps us in deploying and managing application dynamically.

# Kubernetes
kubernetes is container orchestration system and much more.
you can migrate one cloud provider to another, zero downtime, self healing container, load balancing, logs, can store secrete password

CNCF : https://www.cncf.io/

k8s

# What kubernetes takes care of the
- Automatic deployment of the containerized applications accross different servers
- Distribution of the load across multiple servers
- Auto-scaling of the deployed applications
- Monitoring and health check of the containers
- Replacement of the failed containers

# Supported container runtimes
1. Docker
2. CRI-0
3. Containerd


POD is the smallest unit is the k8s world
POD is like scheduling unit
Inside POD there could be one or multiple containers, and shared volumes, shared IP Address

worker node --> container runtime --> pod --> container

1. Create microservice
2. containerized it
3. put container in pods
4. Deploy these pods to controller

- One container per POD is the most common use case

# Kubernetes Cluster
Kubernetes cluster consistes of nodes(servers)
inside nodes(server) there is a pods

In k8s Cluster there is control plane(master node) and others are called worker nodes

# Kubernetes Service
Worker Node : this is where app is running
1. kubelet (communicate with API servers) (if worker node or cluster want to communicate with outside world it helps, this will provides IP address to nodes, for more learn k8s networking)
2. kube-proxy (communicate with API servers)
3. container runtime (communicate with kubelet)


Controle plane: which manages worker node
1. API Server (main service it's magage all k8s cluster, it communicate with etcd, controller manager, scheduler, and listen to every request on HTTPS and port 443)
2. Scheduler
3. Kube controller manager
4. Cloud controller manager
5. etcd (stores logs related to intire k8s cluster, like a database)
6. kubelet
7. kube-proxy
8. container runtime
9. DNS service


# Cluster management
API server manage all k8s cluster by kubectl (command line tool)
kubectl can be run in local computer
with kubectl we can manage remote cluster
and with that we can connect RESTapi with API server, it happens over https protocols

* k8s has it own internal DNS service called k8s DNS ( based on core dns), k8s DNS has ip address for every pod and how they communicate with each other

* if someone gives you kubeconfig file, you can access thier k8s cluster

# Local Setup

- Install kubectl
- Install minikube (To use k8s in local)


minikube status
minikube preview list

minikube start --driver=hyperv

minikube ssh

and after that we'll be inside k8s node
docker is default container runtime for k8s

docker ps (It'll list you all the container which were created inside k8s node)

if you enter kubectl command here you we'll get error, because kubectl is not available inside of k8s node
kubectl is external tool which used to manage k8s cluster

if exit your ssh connect and try to run kubectl it'll work there, because we installed it in our local
and if run
kubectl cluster-info

kubectl get nodes

kubectl get pods

kubectl get namespaces (you will see pods only available inside default namespace)
(list of namespaces which are available in this k8s cluster)
(namespaces provide a mechanism for isolating groups of resources within a single cluster)

check which pod is running inside specific namespaces
kubectl get pods --namespace=kube-system

* kubectl get all, give all the node pods 

## To create single pod
kubectl run pod_name_any_thing --image=docker_image_name
and new container will be created based on this image, and this container will be running inside of k8s pod

kubectl run nginx --image=nginx

- to get details about pod

kubectl describe pod pod_name

# To connect container
docker exec -it container_id sh (connecting using sh executable)

inside container, can run command
hostname
hostname -i (ip address)

now we can connect with tha server which is running in this container, because we are inside the node
curl ip_address

# this command will also give us container ip
If there are several containers in the Pod - they share the same IP address

kubectl get pods -o wide

If you try to connect with the container ip address from outside(you computer) you won't able to do that

# Delete pod
kubectl delete pod pod_name


# It's not convenient to create multiple pods in side k8s cluster because you're not able to scale and increase quantity of the pods, we can use deployment instead of k8s cluster

# Create deployment
kubectl create deployment name_of_the_deployment --image=image_name
kubectl create deployment nginx --image=nginx

kubectl get deployment
kubectl describe deployment deployment_name

kubectl delete deployment deployment_name

ReplicaSet - manages all the pods related to the deployment

# Scaling deployment
kubectl scale deployment deployment_name --replicas=number_of_replicas

# Creating and exploring ClusterIP service
kubectl expose deployment deployment_name --port=8080 --target-port=80

kubectl get serivces or kubectl get svc

after this command you will get info like : service_name, cluster_ip, port
the cluster_ip we'll get is different from pod ip
cluber_ip is a virtual ip address created by k8s, which cloud be used to connect any pod
this allows use to connect deployment inside k8s cluster

use case, database deployment which can be only accessasible inside k8s cluster, 

curl cluster_id:exposed_port - you can access inside node

the answer will be provided one of the your pod inside deployment, but you don't know which one


kubectl describe service

kubectl delete service service_name


# Creating node web application
set up node application

install docker, kubernetes extension

intall docker desktop and run it

create a Dockerfile

- creating build
docker build . -t kushwahaashish/k8s-node-holla

docker images | grep k8s-web

- push to docker hub
docker push image_name (kushwahaashish/k8s-node-holla)

- create deployment base on this image
kubectl create deployment deployment_name --image=image_name

- create service
kubectl expose deployment deployment_name --port=3000

- inside k8s node
curl container_id:exposed_port

while running that open new terminal and try increasing pods

and now if you hit, curl container_id:exposed_port many times, it get answere from different pods 

# Creating NodePort Service
kubectl expose deployment deployment_name --type=NodePort --port=8790

minikube ip (to get node ip address)

now you can use `nide_id:NodeType_port` to open in web
and can also use command
minikube service service_name


# Creating LoadBalancer Service
k expose deployment service_name --type=LoadBalancer --port=exposed_port
with loadblancer you'll get external_ip, it'll provided by the cloud service provider


# Rolling update of the deployment
make any code change in node app, and create new image with another tag, and push it on the docker hub

and after pushing, deploy new image of the deployment

kubectl set image deployment deployment_name pods=kushwahaashish/image_name:tag
after this command do this
kubectl rollout status deploy k8s-node-holla

# What happens when one of the pods is deleted
if we delete running pod, it get deleted but because we told that our replica should be 4, so it create new one again to maintain 4 pods

# Kubernetes Dashboard
minikube dashboard - because we are using minikube

# Creating YAML deployment specification file
so far we have used imperative aproche to create deployment and services, and we created it using different kubectl command
but this is not the way deployment and services are created, in most cases declerative aproche utilized, in declerativ aproche we have to create yaml file, which describe all details for deployment and services, and afterwords apply those configaration file using kubectl apply command

- command to delete everything inside default namespace
kubectl delete all --all

- In you vs code
outside of the node-project directory

create
deployment.yaml
service.yaml

docs
https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/

after creating deployment.yaml, run command
kubectl apply -f deployment.yaml

docs
https://kubernetes.io/docs/reference/kubernetes-api/service-resources/

after creating service.yaml, run command
kubectl apply -f service.yaml

now using minikube service service_name you can open app in browser

and we also delete our configaration file
kubectl delete -f deployment.yaml -f service.yaml



# Plan for the creation of the geplpymen
like connect front-end to back-end and back-end to database

you can combine deployment.yaml and service.yaml --> k8s-web-nginx.yaml
deployment.yaml
---
service.yaml

kubectl apply -f first_yaml_file -f second_taml_file






kubectl exec service_name -- nslookup nginx
kubectl exec service_name -- wget -q0- https://nginx


# Changing Container Runtime From Docker to CRI-O
first do minikube stop and delete

minikube start --driver=hyperv --container-runtime=cri-o



# Helpful tools
lens IDE
kubeshop
kubescape
datree
teleport

https://landscape.cncf.io/
