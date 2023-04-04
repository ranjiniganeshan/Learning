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

