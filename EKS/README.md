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
 ````
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



