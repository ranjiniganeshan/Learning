# Kubernetes Architecture

![image](https://user-images.githubusercontent.com/32661402/229692882-1cf180f6-b264-4b53-9614-1edae67a8a6c.png)

## Architecture of EKS
## Components in the Master nodes /control Plane
* Scheduler: is a control plane process that assigns pods to nodes. When you create a pod, kube-scheduler picks the most suitable node in your cluster and schedules the pod on the node
* Controller: Manages the lifecycle of the pod.
Node controller: Responsible for noticing and responding when nodes go down.
Job controller: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
EndpointSlice controller: Populates EndpointSlice objects (to provide a link between Services and Pods).
* ApiServer : The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.
The main implementation of a Kubernetes API server is kube-apiserver. kube-apiserver is designed to scale horizontally—that is, it scales by deploying more instances.
* etcd : Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.

## Worker Nodes
* kubelet: An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.
The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn't manage containers which were not created by Kubernetes.
* kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.
kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.
kube-proxy uses the operating system packet filtering layer if there is one and it's available. Otherwise, kube-proxy forwards the traffic itself.

* The container runtime is the software that is responsible for running containers.

```
Ranjinis-MacBook-Pro:~ ranjini$ cat cluster-trust-policy.json 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
Ranjinis-MacBook-Pro:~ ranjini$ aws iam create-role \
>   --role-name eksClusterRole \
>   --assume-role-policy-document file://"cluster-trust-policy.json"
{
    "Role": {
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17", 
            "Statement": [
                {
                    "Action": "sts:AssumeRole", 
                    "Effect": "Allow", 
                    "Principal": {
                        "Service": "eks.amazonaws.com"
                    }
                }
            ]
        }, 
        "RoleId": "AROA5HNQXE7VY4P6JPCKT", 
        "CreateDate": "2023-04-04T09:35:22Z", 
        "RoleName": "eksClusterRole", 
        "Path": "/", 
        "Arn": "arn:aws:iam::909293070315:role/eksClusterRole"
    }
}
Ranjinis-MacBook-Pro:~ ranjini$ aws iam attach-role-policy \
>   --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy \
>   --role-name eksClusterRole

```

##EKS Cluster creation

#Install eksctl on mac

# Install Homebrew on MacOs
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

Below commands are fine to install eksctl
# Install the Weaveworks Homebrew tap.
brew tap weaveworks/tap

# Install the Weaveworks Homebrew tap.
brew install weaveworks/tap/eksctl

# Verify eksctl version
eksctl version


# EKS cluster creation

1. Install eksctl using the above command
2. versify eksctl is installed
```
Ranjinis-MacBook-Pro:eks ranjini$ eksctl version
0.137.0
```
3. create demo-cluster.yaml file
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: demo-cluster
  region: us-west-2

nodeGroups:
  - name: ng-1
    instanceType: t2.medium
    desiredCapacity: 2
    volumeSize: 10
    ssh:
      allow: true # will use ~/.ssh/id_rsa.pub as the default ssh key
  - name: ng-2
    instanceType: t2.medium
    desiredCapacity: 2
    volumeSize: 10
    ssh:
      publicKeyPath: ~/.ssh/id_rsa.pub
  ```
  
 4. eksctl create cluster -f demo-cluster.yaml
 ```
Ranjinis-MacBook-Pro:eks ranjini$ eksctl create cluster -f demo-cluster.yaml 
2023-04-25 16:25:04 [ℹ]  eksctl version 0.137.0
2023-04-25 16:25:04 [ℹ]  using region us-west-2
2023-04-25 16:25:09 [ℹ]  skipping us-west-2d from selection because it doesn't support the following instance type(s): t2.medium
2023-04-25 16:25:09 [ℹ]  setting availability zones to [us-west-2c us-west-2a us-west-2b]
2023-04-25 16:25:09 [ℹ]  subnets for us-west-2c - public:192.168.0.0/19 private:192.168.96.0/19
2023-04-25 16:25:09 [ℹ]  subnets for us-west-2a - public:192.168.32.0/19 private:192.168.128.0/19
2023-04-25 16:25:09 [ℹ]  subnets for us-west-2b - public:192.168.64.0/19 private:192.168.160.0/19
2023-04-25 16:25:14 [ℹ]  nodegroup "ng-1" will use "ami-0e1597f2056965cab" [AmazonLinux2/1.25]
2023-04-25 16:25:14 [ℹ]  using SSH public key "/Users/ranjini/.ssh/id_rsa.pub" as "eksctl-demo-cluster-nodegroup-ng-1-43:09:bd:66:b0:1b:f9:60:96:5a:78:33:ba:08:61:d7" 
2023-04-25 16:25:15 [ℹ]  nodegroup "ng-2" will use "ami-0e1597f2056965cab" [AmazonLinux2/1.25]
2023-04-25 16:25:15 [ℹ]  using SSH public key "/Users/ranjini/.ssh/id_rsa.pub" as "eksctl-demo-cluster-nodegroup-ng-2-43:09:bd:66:b0:1b:f9:60:96:5a:78:33:ba:08:61:d7" 
2023-04-25 16:25:20 [ℹ]  using Kubernetes version 1.25
2023-04-25 16:25:20 [ℹ]  creating EKS cluster "demo-cluster" in "us-west-2" region with un-managed nodes
2023-04-25 16:25:20 [ℹ]  2 nodegroups (ng-1, ng-2) were included (based on the include/exclude rules)
2023-04-25 16:25:20 [ℹ]  will create a CloudFormation stack for cluster itself and 2 nodegroup stack(s)
2023-04-25 16:25:20 [ℹ]  will create a CloudFormation stack for cluster itself and 0 managed nodegroup stack(s)
2023-04-25 16:25:20 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-west-2 --cluster=demo-cluster'
2023-04-25 16:25:20 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "demo-cluster" in "us-west-2"
2023-04-25 16:25:20 [ℹ]  CloudWatch logging will not be enabled for cluster "demo-cluster" in "us-west-2"
2023-04-25 16:25:20 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-west-2 --cluster=demo-cluster'
2023-04-25 16:25:20 [ℹ]  
2 sequential tasks: { create cluster control plane "demo-cluster", 
    2 sequential sub-tasks: { 
        wait for control plane to become ready,
        2 parallel sub-tasks: { 
            create nodegroup "ng-1",
            create nodegroup "ng-2",
        },
    } 
}
2023-04-25 16:25:20 [ℹ]  building cluster stack "eksctl-demo-cluster-cluster"
2023-04-25 16:25:27 [ℹ]  deploying stack "eksctl-demo-cluster-cluster"
2023-04-25 16:25:57 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:26:32 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:27:37 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:28:42 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:29:47 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:30:52 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:31:58 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:33:02 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:34:07 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:35:12 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:36:17 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:37:22 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-cluster"
2023-04-25 16:39:45 [ℹ]  building nodegroup stack "eksctl-demo-cluster-nodegroup-ng-2"
2023-04-25 16:39:45 [ℹ]  building nodegroup stack "eksctl-demo-cluster-nodegroup-ng-1"
2023-04-25 16:39:45 [ℹ]  --nodes-min=2 was set automatically for nodegroup ng-1
2023-04-25 16:39:45 [ℹ]  --nodes-max=2 was set automatically for nodegroup ng-1
2023-04-25 16:39:45 [ℹ]  --nodes-min=2 was set automatically for nodegroup ng-2
2023-04-25 16:39:45 [ℹ]  --nodes-max=2 was set automatically for nodegroup ng-2
2023-04-25 16:39:51 [ℹ]  deploying stack "eksctl-demo-cluster-nodegroup-ng-1"
2023-04-25 16:39:51 [ℹ]  deploying stack "eksctl-demo-cluster-nodegroup-ng-2"
2023-04-25 16:39:51 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-1"
2023-04-25 16:39:51 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-2"
2023-04-25 16:40:25 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-2"
2023-04-25 16:40:25 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-1"
2023-04-25 16:41:01 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-2"
2023-04-25 16:41:01 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-1"
2023-04-25 16:41:37 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-1"
2023-04-25 16:43:01 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-2"
2023-04-25 16:43:16 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-1"
2023-04-25 16:44:03 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-1"
2023-04-25 16:44:08 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-2"
2023-04-25 16:45:24 [ℹ]  waiting for CloudFormation stack "eksctl-demo-cluster-nodegroup-ng-1"
2023-04-25 16:45:24 [ℹ]  waiting for the control plane to become ready
2023-04-25 16:45:24 [✔]  saved kubeconfig as "/Users/ranjini/.kube/config"
2023-04-25 16:45:24 [ℹ]  no tasks
2023-04-25 16:45:24 [✔]  all EKS cluster resources for "demo-cluster" have been created
2023-04-25 16:45:29 [ℹ]  adding identity "arn:aws:iam::909293070315:role/eksctl-demo-cluster-nodegroup-ng-NodeInstanceRole-1R297RQ40EF6C" to auth ConfigMap
2023-04-25 16:45:30 [ℹ]  nodegroup "ng-1" has 0 node(s)
2023-04-25 16:45:30 [ℹ]  waiting for at least 2 node(s) to become ready in "ng-1"
2023-04-25 16:46:15 [ℹ]  nodegroup "ng-1" has 2 node(s)
2023-04-25 16:46:15 [ℹ]  node "ip-192-168-60-197.us-west-2.compute.internal" is ready
2023-04-25 16:46:15 [ℹ]  node "ip-192-168-81-155.us-west-2.compute.internal" is ready
2023-04-25 16:46:15 [ℹ]  adding identity "arn:aws:iam::909293070315:role/eksctl-demo-cluster-nodegroup-ng-NodeInstanceRole-15XJVUS0ME1VD" to auth ConfigMap
2023-04-25 16:46:16 [ℹ]  nodegroup "ng-2" has 0 node(s)
2023-04-25 16:46:16 [ℹ]  waiting for at least 2 node(s) to become ready in "ng-2"
2023-04-25 16:47:21 [ℹ]  nodegroup "ng-2" has 2 node(s)
2023-04-25 16:47:21 [ℹ]  node "ip-192-168-11-193.us-west-2.compute.internal" is ready
2023-04-25 16:47:21 [ℹ]  node "ip-192-168-76-103.us-west-2.compute.internal" is ready
2023-04-25 16:47:31 [ℹ]  kubectl command should work with "/Users/ranjini/.kube/config", try 'kubectl get nodes'
2023-04-25 16:47:31 [✔]  EKS cluster "demo-cluster" in "us-west-2" region is ready

```
# creating namespace

refer to namespace yaml file https://github.com/ranjiniganeshan/Learning/tree/main/EKS/namespace
```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl create -f test-dev.yaml 
namespace/development created



Ranjinis-MacBook-Pro:eks ranjini$ kubectl create -f namespace.yaml
namespace/twitter created
namespace/facebook created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get namespace
NAME              STATUS   AGE
default           Active   39m
development       Active   4m21s
facebook          Active   2m53s
kube-node-lease   Active   39m
kube-public       Active   39m
kube-system       Active   39m
production        Active   5m3s
twitter           Active   2m57s
Ranjinis-MacBook-Pro:eks ranjini$ 

```
# Pod creation

```

Ranjinis-MacBook-Pro:eks ranjini$ kubectl create -f pod.yaml
pod/webserver created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get pods
No resources found in default namespace.
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get pods -A
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
kube-system   aws-node-bcdzn             1/1     Running   0          47m
kube-system   aws-node-sxfmz             1/1     Running   0          48m
kube-system   aws-node-whvkz             1/1     Running   0          47m
kube-system   aws-node-wpr9d             1/1     Running   0          48m
kube-system   coredns-67f8f59c6c-mv8gf   1/1     Running   0          61m
kube-system   coredns-67f8f59c6c-svg44   1/1     Running   0          61m
kube-system   kube-proxy-8wnzf           1/1     Running   0          47m
kube-system   kube-proxy-f6mjc           1/1     Running   0          47m
kube-system   kube-proxy-jqf5p           1/1     Running   0          48m
kube-system   kube-proxy-zgnkf           1/1     Running   0          48m
twitter       webserver                  1/1     Running   0          2m56s
Ranjinis-MacBook-Pro:eks ranjini$ 

```

Multipod creation

```

Ranjinis-MacBook-Pro:eks ranjini$ kubectl create -f multipod.yaml 
pod/multicontainer-pods created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get pods -n facebook
NAME                  READY   STATUS    RESTARTS   AGE
multicontainer-pods   2/2     Running   0          18s
Ranjinis-MacBook-Pro:eks ranjini$ 

```

To log into the container

Describe the pod 

```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl describe pod multicontainer-pods -n facebook
Name:         multicontainer-pods
Namespace:    facebook
Priority:     0
Node:         ip-192-168-60-197.us-west-2.compute.internal/192.168.60.197
Start Time:   Tue, 25 Apr 2023 17:40:20 +0530
Labels:       app=httpd
              tier=frontend-backend
              version=v1
Annotations:  <none>
Status:       Running
IP:           192.168.48.218
IPs:
  IP:  192.168.48.218
Containers:
  web:
    Container ID:   containerd://e148006664e89c7687a345d8e164865a9490cfcef7ebfd8a8095fe4716d68492
    Image:          httpd
    Image ID:       docker.io/library/httpd@sha256:a182ef2350699f04b8f8e736747104eb273e255e818cd55b6d7aa50a1490ed0c
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 25 Apr 2023 17:40:25 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-hphhc (ro)
  db:
    Container ID:   containerd://1937170e42413860068cdebbd4f49268ae28e3f4ff11414f164c2c5d8de03893
    Image:          mysql
    Image ID:       docker.io/library/mysql@sha256:a43f6e7e7f3a5e5b90f857fbed4e3103ece771b19f0f75880f767cf66bbb6577
    Port:           3306/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 25 Apr 2023 17:40:36 +0530
    Ready:          True
    Restart Count:  0
    Environment:
      MYSQL_ROOT_PASSWORD:  cloudnloud
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-hphhc (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-hphhc:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  5m41s  default-scheduler  Successfully assigned facebook/multicontainer-pods to ip-192-168-60-197.us-west-2.compute.internal
  Normal  Pulling    5m41s  kubelet            Pulling image "httpd"
  Normal  Pulled     5m36s  kubelet            Successfully pulled image "httpd" in 4.939456709s (4.939472527s including waiting)
  Normal  Created    5m36s  kubelet            Created container web
  Normal  Started    5m36s  kubelet            Started container web
  Normal  Pulling    5m36s  kubelet            Pulling image "mysql"
  Normal  Pulled     5m25s  kubelet            Successfully pulled image "mysql" in 10.923130775s (10.923178135s including waiting)
  Normal  Created    5m25s  kubelet            Created container db
  Normal  Started    5m25s  kubelet            Started container db
  
  
  ```
  
  #get the iinformation pod name and container name
  
  ```
  Ranjinis-MacBook-Pro:eks ranjini$ kubectl exec -it pod/multicontainer-pods -n facebook bash -c web
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@multicontainer-pods:/usr/local/apache2# 
```
# both the container using same port

https://github.com/ranjiniganeshan/Learning/blob/main/EKS/POD/duplicate-port.yaml

```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get all -n facebook
NAME                      READY   STATUS   RESTARTS     AGE
pod/multicontainer-pods   1/2     Error    1 (8s ago)   14s
```
```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl describe pod/multicontainer-pods -n facebook
Name:         multicontainer-pods
Namespace:    facebook
Priority:     0
Node:         ip-192-168-60-205.us-west-2.compute.internal/192.168.60.205
Start Time:   Tue, 25 Apr 2023 22:08:16 +0530
Labels:       app=httpd
              tier=frontend-backend
              version=v1
Annotations:  <none>
Status:       Running
IP:           192.168.60.216
IPs:
  IP:  192.168.60.216
Containers:
  httpd1:
    Container ID:   containerd://2406f3c9f61708bcb42571703c8635dcc35b587815b1baaf932e7f0935d92db4
    Image:          httpd
    Image ID:       docker.io/library/httpd@sha256:a182ef2350699f04b8f8e736747104eb273e255e818cd55b6d7aa50a1490ed0c
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 25 Apr 2023 22:08:22 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-j28ss (ro)
  httpd2:
    Container ID:   containerd://568b84698af3af0788306291b7c209d28fe96ed879b93d059611d425c3581096
    Image:          httpd
    Image ID:       docker.io/library/httpd@sha256:a182ef2350699f04b8f8e736747104eb273e255e818cd55b6d7aa50a1490ed0c
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Waiting
  **    Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error**
      Exit Code:    1
      Started:      Tue, 25 Apr 2023 22:09:06 +0530
      Finished:     Tue, 25 Apr 2023 22:09:06 +0530
    Ready:          False
    Restart Count:  3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-j28ss (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-j28ss:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  84s                default-scheduler  Successfully assigned facebook/multicontainer-pods to ip-192-168-60-205.us-west-2.compute.internal
  Normal   Pulling    83s                kubelet            Pulling image "httpd"
  Normal   Pulled     78s                kubelet            Successfully pulled image "httpd" in 4.817375917s (4.817392112s including waiting)
  Normal   Created    78s                kubelet            Created container httpd1
  Normal   Started    78s                kubelet            Started container httpd1
  Normal   Pulled     78s                kubelet            Successfully pulled image "httpd" in 561.392742ms (561.403591ms including waiting)
  Normal   Pulled     75s                kubelet            Successfully pulled image "httpd" in 519.814136ms (519.825682ms including waiting)
  Normal   Pulled     64s                kubelet            Successfully pulled image "httpd" in 538.520557ms (538.533185ms including waiting)
  Normal   Pulling    35s (x4 over 78s)  kubelet            Pulling image "httpd"
  Normal   Created    35s (x4 over 78s)  kubelet            Created container httpd2
  Normal   Pulled     35s                kubelet            Successfully pulled image "httpd" in 566.804757ms (566.81465ms including waiting)
  Normal   Started    34s (x4 over 78s)  kubelet            Started container httpd2
  Warning  BackOff    24s (x5 over 74s)  kubelet            Back-off restarting failed container
Ranjinis-MacBook-Pro:eks ranjini$ kubectl delete -f duplicate-pod.yaml 
pod "multicontainer-pods" deleted
Ranjinis-MacBook-Pro:eks ranjini$ kubectl delete ns facebook
namespace "facebook" deleted
```

# creating storage

```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl create -f storage.yml 
pod/database created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get all -n twitter
NAME           READY   STATUS    RESTARTS   AGE
pod/database   1/1     Running   0          11s
Ranjinis-MacBook-Pro:eks ranjini$ kubectl describe pod database -n twitter
Name:         database
Namespace:    twitter
Priority:     0
**Node:         ip-192-168-48-109.us-west-2.compute.internal/192.168.48.109**
Start Time:   Tue, 25 Apr 2023 22:15:33 +0530
Labels:       app=psql
              tier=backend
              version=master
Annotations:  <none>
Status:       Running
IP:           192.168.38.20
IPs:
  IP:  192.168.38.20
Containers:
  psql:
    Container ID:   containerd://0d7b34b8158bdd51ebe4342be49aa0230e3d777480a0c509a1c77f16d962a8ca
    Image:          postgres
    Image ID:       docker.io/library/postgres@sha256:6cc97262444f1c45171081bc5a1d4c28b883ea46a6e0d1a45a8eac4a7f4767ab
    Port:           5432/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 25 Apr 2023 22:15:43 +0530
    Ready:          True
    Restart Count:  0
    Environment:
      POSTGRES_PASSWORD:  Password@123
   ** Mounts:
      /var/lib/postgresql/data from pgdata (rw)**
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4z6d9 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  pgdata:
   ** Type:          HostPath (bare host directory volume)
    Path:          /var/lib/postgres**
    HostPathType:  DirectoryOrCreate
  kube-api-access-4z6d9:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  64s   default-scheduler  Successfully assigned twitter/database to ip-192-168-48-109.us-west-2.compute.internal
  Normal  Pulling    63s   kubelet            Pulling image "postgres"
  Normal  Pulled     54s   kubelet            Successfully pulled image "postgres" in 8.574403332s (8.574415988s including waiting)
  Normal  Created    54s   kubelet            Created container psql
  Normal  Started    54s   kubelet            Started container psql
  
```

Verified the hospath existed.

```
[root@ip-192-168-48-109 ~]# ls -ltra /var/lib/postgres
total 72
drwxr-xr-x 31 root root   4096 Apr 25 16:45 ..
drwx------  2 rngd input     6 Apr 25 16:45 pg_twophase
drwx------  2 rngd input     6 Apr 25 16:45 pg_snapshots
drwx------  2 rngd input     6 Apr 25 16:45 pg_serial
drwx------  2 rngd input     6 Apr 25 16:45 pg_notify
drwx------  2 rngd input     6 Apr 25 16:45 pg_dynshmem
drwx------  2 rngd input     6 Apr 25 16:45 pg_commit_ts
-rw-------  1 rngd input     3 Apr 25 16:45 PG_VERSION
drwx------  2 rngd input     6 Apr 25 16:45 pg_tblspc
drwx------  2 rngd input     6 Apr 25 16:45 pg_stat_tmp
drwx------  2 rngd input     6 Apr 25 16:45 pg_replslot
drwx------  4 rngd input    36 Apr 25 16:45 pg_multixact
-rw-------  1 rngd input 29525 Apr 25 16:45 postgresql.conf
-rw-------  1 rngd input    88 Apr 25 16:45 postgresql.auto.conf
-rw-------  1 rngd input  1636 Apr 25 16:45 pg_ident.conf
drwx------  2 rngd input    18 Apr 25 16:45 pg_xact
drwx------  3 rngd input    60 Apr 25 16:45 pg_wal
drwx------  2 rngd input    18 Apr 25 16:45 pg_subtrans
drwx------  5 rngd input    33 Apr 25 16:45 base
-rw-------  1 rngd input  4821 Apr 25 16:45 pg_hba.conf
drwx------ 19 rngd root   4096 Apr 25 16:45 .
-rw-------  1 rngd input    36 Apr 25 16:45 postmaster.opts
drwx------  2 rngd input     6 Apr 25 16:45 pg_stat
-rw-------  1 rngd input    94 Apr 25 16:45 postmaster.pid
drwx------  2 rngd input  4096 Apr 25 16:48 global
drwx------  4 rngd input    68 Apr 25 16:50 pg_logical
```



