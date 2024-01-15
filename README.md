# Kubernetes Lab Clusters with k3d  
CUSTOM CLUSTER for labs  
Using the docs: https://k3d.io/v5.0.1/usage/commands/k3d/  
We can create masters (control-plane,etcd,master)  
We can create agents(nodes)  

Always verify we are using correct context for kubectl commands  
> kubectl config set-context    
> verify current-context  
  
E.g  
> k3d cluster create --servers 3 --agents 6  
Will give us 3 masters and 3 workers nodes  
But we can also adding custom node with node name of our choice  

So we can do like: ( just to simulate a prod like env - for fun)  
CREATE CLUSTER AND NODES  
> k3d cluster create --servers 3 --agents 5 k3d-cluster --port '8080:80@loadbalancer'    
3 master, 3 workers, 1 infra, 1 monitor    
Using load-balancer in this example:    
disable Traefik, you can add the -p "8080:80@loadbalancer" flag when creating the cluster.   
This will expose the load balancer on port 8080 of the host machine.  
When we create a tunnel from host machine to load balancer with port-forward  

If we want to add a node  
> k3d node create nodeName -c k3d-cluster
-c for cluster string name and this will bind the nodes to that cluster
For deleting a node:
 
K3d and kubectl is operating on two diff layers  
K3d on ked discovery  
Kubectl on kubectl discovery  
So we need to any order for proper deletion  
1. > k3d node delete k3d-nodeName-0  
Kubectl will see this node as NotReady  
2. > kubectl delete nodes k3d-nodename-0  
K3d will see this node as running  


ROLE LABELS NODES:
> kubectl label nodes k3d-k3d-cluster-agent-1 k3d-k3d-cluster-agent-2 k3d-k3d-cluster-agent-3 node-role.kubernetes.io/worker=""
> kubectl label nodes k3d-k3d-cluster-agent-0 node-role.kubernetes.io/monitor=""
> kubectl label nodes k3d-k3d-cluster-agent-4 node-role.kubernetes.io/infra=""
Verify:
> kubectl 
Remove role label:
> kubectl label node k3d-k3d-cluster-agent-3  node-role.kubernetes.io/worker-
Adding role label:
> kubectl label nodes k3d-k3d-cluster-agent-3 node-role.kubernetes.io/monitor=""


DEPLOY APPLICATION:
Simple nginx
> kubectl create deployment --image=docker.io/nginx:latest --replicas 3 --port 80 nginx
With 3 replicas
Expose port 80
IP of Pod:
> kubectl get pod nginx --template '{{.status.podIP}}'
Access pod:
> kubectl exec --tty pods/nginx -- /bin/bash
Expose the pod:
> kubectl port-forward services/nginx 8001:80
Fastest but just for testing and labs
Or - more proper way, best is ifc LB
> kubectl create service clusterip nginx --tcp=80:80
And when create a ingress.networking.k8s.io by file +apply -f





CLEAN UP:  
> k3d cluster delete master  
> docker rm $(docker ps -aq) - this will rm all containers  
> docker rmi $(docker images -q) - this will remove all unreferenced images  






