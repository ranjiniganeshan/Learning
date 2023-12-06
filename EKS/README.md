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
  version: "1.27"

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
Ranjinis-MacBook-Pro:eks ranjini$ kubectl create -f namespace.yaml
namespace/twitter created
namespace/facebook created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get namespaceRanjinis-MacBook-Pro:eks ranjini$ kubectl create -f replicaset.yaml 
deployment.apps/demo created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
demo-596dfbfb74   3         3         3       48s
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
demo   3/3     3            3           67s
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

Multistorage path 



```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl create -f multistorage.yaml 
pod/db created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl describe pod db -n facebook
Name:         db
Namespace:    facebook
Priority:     0
Node:         ip-192-168-69-241.us-west-2.compute.internal/192.168.69.241
Start Time:   Tue, 25 Apr 2023 22:26:51 +0530
Labels:       app=postgres
              tier=backend
              version=master
Annotations:  <none>
Status:       Running
IP:           192.168.65.187
IPs:
  IP:  192.168.65.187
Containers:
  db:
    Container ID:   containerd://cca11b9dd06fcb82ce6a074f0f67deada1f98e98d1892a564bac99e613e81083
    Image:          postgres
    Image ID:       docker.io/library/postgres@sha256:6cc97262444f1c45171081bc5a1d4c28b883ea46a6e0d1a45a8eac4a7f4767ab
    Port:           5432/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 25 Apr 2023 22:27:01 +0530
    Ready:          True
    Restart Count:  0
    Environment:
      POSTGRES_PASSWORD:  Password@123
    Mounts:
      /var/lib/postgresql/data from pgdata (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mkcv5 (ro)
  web:
    Container ID:   containerd://f3600fa9edfc59f4f65c29218088e66c97591988c5d61d1a3298effa3d0f6866
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:63b44e8ddb83d5dd8020327c1f40436e37a6fffd3ef2498a6204df23be6e7e94
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 25 Apr 2023 22:27:07 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html/ from webdata (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mkcv5 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  pgdata:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/postgres
    HostPathType:  DirectoryOrCreate
  webdata:
    Type:          HostPath (bare host directory volume)
    Path:          /var/www/html
    HostPathType:  DirectoryOrCreate
  kube-api-access-mkcv5:
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
  Normal  Scheduled  116s  default-scheduler  Successfully assigned facebook/db to ip-192-168-69-241.us-west-2.compute.internal
  Normal  Pulling    115s  kubelet            Pulling image "postgres"
  Normal  Pulled     106s  kubelet            Successfully pulled image "postgres" in 8.771079271s (8.771110347s including waiting)
  Normal  Created    106s  kubelet            Created container db
  Normal  Started    106s  kubelet            Started container db
  Normal  Pulling    106s  kubelet            Pulling image "nginx"
  Normal  Pulled     100s  kubelet            Successfully pulled image "nginx" in 6.106551926s (6.106574468s including waiting)
  Normal  Created    100s  kubelet            Created container web
  Normal  Started    100s  kubelet            Started container web
Ranjinis-MacBook-Pro:eks ranjini$ 

```

Complete yaml which create pod ns and storage

```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl create -f full.yaml 
namespace/twitter created
pod/webserver created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get all -n twitter
NAME            READY   STATUS    RESTARTS   AGE
pod/webserver   2/2     Running   0          19s

```

## Web and db pod creation in default namespace

```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl create -f web-db.yaml 
pod/first-pod created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl describe pod first-pod
Name:         first-pod
Namespace:    default
Priority:     0
Node:         ip-192-168-69-241.us-west-2.compute.internal/192.168.69.241
Start Time:   Tue, 25 Apr 2023 22:35:37 +0530
Labels:       app=cloudnloud
Annotations:  <none>
Status:       Running
IP:           192.168.84.234
IPs:
  IP:  192.168.84.234
Containers:
  web-server:
    Container ID:   containerd://8e2401ff486e022f9a2048003fcc84088350f5b3db17b36a8fe7cdc06b9de23f
    Image:          httpd
    Image ID:       docker.io/library/httpd@sha256:a182ef2350699f04b8f8e736747104eb273e255e818cd55b6d7aa50a1490ed0c
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 25 Apr 2023 22:35:41 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cjgvh (ro)
  redis:
    Container ID:   containerd://58469f556d748a85659ca9ea63703f3615b2fc2b76829ecb7a7174704d22b0ce
    Image:          redis
    Image ID:       docker.io/library/redis@sha256:f50031a49f41e493087fb95f96fdb3523bb25dcf6a3f0b07c588ad3cdbe1d0aa
    Port:           6379/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 25 Apr 2023 22:35:43 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cjgvh (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-cjgvh:
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
  Normal  Scheduled  61s   default-scheduler  Successfully assigned default/first-pod to ip-192-168-69-241.us-west-2.compute.internal
  Normal  Pulling    60s   kubelet            Pulling image "httpd"
  Normal  Pulled     58s   kubelet            Successfully pulled image "httpd" in 2.871219518s (2.87123069s including waiting)
  Normal  Created    58s   kubelet            Created container web-server
  Normal  Started    57s   kubelet            Started container web-server
  Normal  Pulling    57s   kubelet            Pulling image "redis"
  Normal  Pulled     55s   kubelet            Successfully pulled image "redis" in 2.189438491s (2.189448788s including waiting)
  Normal  Created    55s   kubelet            Created container redis
  Normal  Started    55s   kubelet            Started container redis
  ```
  # Replicaset 
  
  ```
  
Ranjinis-MacBook-Pro:eks ranjini$ kubectl create -f replicaset.yaml 
deployment.apps/demo created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
demo-596dfbfb74   3         3         3       48s
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
demo   3/3     3            3           67s

```
  
  #Services
  
  ClusterIP: is an Internal Load balancer is accessible only within the cluster
  
  kubctl apply -f clusterip.yaml. to access the service within the cluster use 10.100.110.87:32767
  ```

Ranjinis-MacBook-Pro:eks ranjini$ kubectl get svc
NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
kubernetes                       ClusterIP   10.100.0.1      <none>        443/TCP     40m
**nginx-deployment-clusterip-svc   ClusterIP   10.100.110.87   <none>        32767/TCP   5m18s**
```

  <img width="759" alt="Screen Shot 2023-04-27 at 10 12 28 PM" src="https://user-images.githubusercontent.com/32661402/234932006-50c33b2d-8539-484b-98ff-b728893ec2a2.png">
  
  
  Nodeport: Listens on port and forwards the request to pod. range 30000 to 32767
   <img width="929" alt="Screen Shot 2023-04-25 at 10 59 00 PM" src="https://user-images.githubusercontent.com/32661402/234356067-2327386c-0407-44a6-8094-98f33e177dee.png">
   
   ```
   Ranjinis-MacBook-Pro:eks ranjini$ kubectl apply -f nodeport.yaml 
deployment.apps/nginx-deployment unchanged
service/nginx-deployment-clusterip-svc configured
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get svc
NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
kubernetes                       ClusterIP   10.100.0.1      <none>        443/TCP           47m
nginx-deployment-clusterip-svc   NodePort    **10.100.110.87**   <none>        32767:**30000**/TCP   12m
```

http://35.90.205.82:30000/ after opening the port in security group for 30000. 

<img width="1280" alt="Screen Shot 2023-04-27 at 10 33 05 PM" src="https://user-images.githubusercontent.com/32661402/234936812-27ff56cd-081a-4f54-9230-453c4ff8d6a5.png">

#Loadbalncer

Access the services using externalIP( loadbalancer IP and port

```

Ranjinis-MacBook-Pro:eks ranjini$ kubectl apply -f loadbalancer.yaml 
deployment.apps/nginx-deployment unchanged
service/nginx-deployment-clusterip-svc configured
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get svc
NAME                             TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)           AGE
kubernetes                       ClusterIP      10.100.0.1      <none>                                                                    443/TCP           72m
nginx-deployment-clusterip-svc   LoadBalancer   10.100.110.87   a4532765251524ba2bc7955e1e6be67f-1825180946.us-west-2.elb.amazonaws.com   32767:30000/TCP   37m
```
<img width="980" alt="Screen Shot 2023-04-27 at 10 49 00 PM" src="https://user-images.githubusercontent.com/32661402/234940051-d15922eb-1614-4a87-857d-28c4c2b71879.png">

## Config map

How to store configuration information instead of hard coding in docker container. 

config map can be stored in file and folders and as well can be passed as literal or volume.

```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get configmap
NAME               DATA   AGE
kube-root-ca.crt   1      17m

Ranjinis-MacBook-Pro:eks ranjini$ kubectl create configmap testconfig --from-file="configmap/game.properties"
configmap/testconfig created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get configmap
NAME               DATA   AGE
kube-root-ca.crt   1      18m
testconfig         1      10s
Ranjinis-MacBook-Pro:eks ranjini$ kubectl describe configmap testconfig
Name:         testconfig
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemy.types=aliens,monsters
player.maximum-lives=5    

BinaryData
====

Events:  <none>
```


# Create config map using literal
```

Ranjinis-MacBook-Pro:eks ranjini$ kubectl create configmap litercal-config --from-literal=mykey=myval --from-literal=myname=vinodh
configmap/litercal-config created

Ranjinis-MacBook-Pro:eks ranjini$ kubectl describe configmap litercal-config
Name:         litercal-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
mykey:
----
myval
myname:
----
vinodh

BinaryData
====

Events:  <none>

```

config as environment variable

```
Ranjinis-MacBook-Pro:configmap ranjini$ kubectl create configmap special-config --from-literal=special.how=very
configmap/special-config created

```

How pod i s using env created by the above

Ranjinis-MacBook-Pro:configmap ranjini$ kubectl create -f config-env-pod-single.yaml 
pod/dapi-test-pod created
Ranjinis-MacBook-Pro:configmap ranjini$ kubectl get pods -A
NAMESPACE     NAME                       READY   STATUS      RESTARTS   AGE
default       dapi-test-pod              0/1     Completed   0          56s
kube-system   aws-node-6jslk             1/1     Running     0          86m
kube-system   aws-node-r56n6             1/1     Running     0          86m
kube-system   coredns-67f8f59c6c-7ccj2   1/1     Running     0          98m
kube-system   coredns-67f8f59c6c-xf2zr   1/1     Running     0          98m
kube-system   kube-proxy-l726j           1/1     Running     0          86m
kube-system   kube-proxy-thxfd           1/1     Running     0          86m
Pod using the above config as ENV

Ranjinis-MacBook-Pro:configmap ranjini$ kubectl logs dapi-test-pod
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.100.0.1:443
HOSTNAME=dapi-test-pod
SHLVL=1
HOME=/root
KUBERNETES_PORT_443_TCP_ADDR=10.100.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
SPECIAL_LEVEL_KEY=very
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.100.0.1:443
PWD=/
KUBERNETES_SERVICE_HOST=10.100.0.1

```
Config as volume

Ranjinis-MacBook-Pro:configmap ranjini$ kubectl create -f configmapvolume.yaml 
pod/configmap-demo-pod created
Ranjinis-MacBook-Pro:configmap ranjini$ kubectl exec -it configmap-demo-pod sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ # 
/ # df -h 
Filesystem                Size      Used Available Use% Mounted on
overlay                  10.0G      2.2G      7.8G  22% /
tmpfs                    64.0M         0     64.0M   0% /dev
tmpfs                     1.9G         0      1.9G   0% /sys/fs/cgroup
/dev/xvda1               10.0G      2.2G      7.8G  22% /config
/dev/xvda1               10.0G      2.2G      7.8G  22% /etc/hosts
/dev/xvda1               10.0G      2.2G      7.8G  22% /dev/termination-log
/dev/xvda1               10.0G      2.2G      7.8G  22% /etc/hostname
/dev/xvda1               10.0G      2.2G      7.8G  22% /etc/resolv.conf
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                     3.3G     12.0K      3.3G   0% /run/secrets/kubernetes.io/serviceaccount
tmpfs                     1.9G         0      1.9G   0% /proc/acpi
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/keys
tmpfs                    64.0M         0     64.0M   0% /proc/latency_stats
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                     1.9G         0      1.9G   0% /proc/scsi
tmpfs                     1.9G         0      1.9G   0% /sys/firmware
/ # cd /config
/config # ls
game.properties            user-interface.properties
```
## Volumes

* empty-dir

this is temporary storage. when the pod is getting created this occurs and when the pod is deleted the folder is also deleted on the worker node.

```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl apply -f empty-dir.yaml
pod/test-pd created
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get nodes -A
NAME                                           STATUS   ROLES    AGE     VERSION
ip-192-168-45-133.us-west-2.compute.internal   Ready    <none>   4m41s   v1.25.7-eks-a59e1f0
ip-192-168-76-225.us-west-2.compute.internal   Ready    <none>   4m42s   v1.25.7-eks-a59e1f0
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get pods -A
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
default       test-pd                    1/1     Running   0          23s
kube-system   aws-node-5pxxf             1/1     Running   0          4m50s
kube-system   aws-node-zsqng             1/1     Running   0          4m51s
kube-system   coredns-67f8f59c6c-lkfqk   1/1     Running   0          16m
kube-system   coredns-67f8f59c6c-z4bfl   1/1     Running   0          16m
kube-system   kube-proxy-6xtjt           1/1     Running   0          4m50s
kube-system   kube-proxy-v6ntc           1/1     Running   0          4m51s

```

* created a file within pod
```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl exec -it test-pd sh  -n default
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ # 
/ # df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  10.0G      2.2G      7.8G  22% /
tmpfs                    64.0M         0     64.0M   0% /dev
tmpfs                     1.9G         0      1.9G   0% /sys/fs/cgroup
/dev/xvda1               10.0G      2.2G      7.8G  22% /cache
/dev/xvda1               10.0G      2.2G      7.8G  22% /etc/hosts
/dev/xvda1               10.0G      2.2G      7.8G  22% /dev/termination-log
/dev/xvda1               10.0G      2.2G      7.8G  22% /etc/hostname
/dev/xvda1               10.0G      2.2G      7.8G  22% /etc/resolv.conf
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                     3.3G     12.0K      3.3G   0% /var/run/secrets/kubernetes.io/serviceaccount
tmpfs                     1.9G         0      1.9G   0% /proc/acpi
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/keys
tmpfs                    64.0M         0     64.0M   0% /proc/latency_stats
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                     1.9G         0      1.9G   0% /proc/scsi
tmpfs                     1.9G         0      1.9G   0% /sys/firmware
/ # cd /cache
/cache # echo "This is a test file"
This is a test file
/cache # echo "This is a test file" > test_file_empty_dir.txt
/cache # ls
test_file_empty_dir.txt
/cache # cat test_file_empty_dir.txt 
This is a test file
/cache # exit
```

* this file is available on worker node

get the uuid of the pod 
```
Ranjinis-MacBook-Pro:eks ranjini$ kubectl get pods -n default test-pd -o jsonpath='{.metadata.uid}'
00544750-d5ce-41d6-ba6d-6d68e1834190Ranjinis-MacBook-Pro:eks ranjini$ 
root@ip-192-168-45-133 cache-volume]# pwd
/var/lib/kubelet/pods/00544750-d5ce-41d6-ba6d-6d68e1834190/volumes/kubernetes.io~empty-dir/cache-volume
[root@ip-192-168-45-133 cache-volume]# cat test_file_empty_dir.txt
This is a test file
```
#Persistent volume : This is the permanent storage. Even if the pod is deleted the storage remains.

Steps 
1. create persistent volume (example admins say that they need 100G )
```
Ranjinis-MacBook-Pro:volumes ranjini$ kubectl apply -f pv.yaml 
persistentvolume/foo-pv created
```
2. clain the persistent volume ( developers take 10G from the above. they submit the claim)
```
Ranjinis-MacBook-Pro:volumes ranjini$ kubectl apply -f pvc.yaml
persistentvolumeclaim/foo-pvc created
Ranjinis-MacBook-Pro:volumes ranjini$ kubectl get pvc
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
foo-pvc   Bound    foo-pv   1Gi        RWO            local-storage   34m
Ranjinis-MacBook-Pro:volumes ranjini$ kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS    REASON   AGE
foo-pv   1Gi        RWO            Retain           Bound    default/foo-pvc   local-storage            40m
```
3. the submitted claims needs to be allocated to pod.
```
Ranjinis-MacBook-Pro:volumes ranjini$ kubectl apply -f pod-pvc.yaml 
pod/pvc-pod-01 created
Ranjinis-MacBook-Pro:volumes ranjini$ kubectl get pods -A
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
default       pvc-pod-01                 1/1     Running   0          14s
kube-system   aws-node-dfp8r             1/1     Running   0          52m
kube-system   aws-node-fcvf6             1/1     Running   0          52m
kube-system   coredns-67f8f59c6c-fpxpl   1/1     Running   0          66m
kube-system   coredns-67f8f59c6c-mw6tx   1/1     Running   0          66m
kube-system   kube-proxy-h4nrp           1/1     Running   0          52m
kube-system   kube-proxy-qvbcj           1/1     Running   0          52m
Ranjinis-MacBook-Pro:volumes ranjini$ kubectl exec -it pvc-pod-01 -- /bin/sh
/ # 
/ # df -h /mnt/storage
Filesystem                Size      Used Available Use% Mounted on
/dev/xvda1               10.0G      2.2G      7.8G  22% /mnt/storage
```



  
 





