# Kubernetes Architecture

![image](https://user-images.githubusercontent.com/32661402/229692882-1cf180f6-b264-4b53-9614-1edae67a8a6c.png)

## Architecture of EKS
## Components in the Master nodes
* Scheduler: is a control plane process that assigns pods to nodes. When you create a pod, kube-scheduler picks the most suitable node in your cluster and schedules the pod on the node
* Controller: Manages the lifecycle of the pod.
Node controller: Responsible for noticing and responding when nodes go down.
Job controller: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
EndpointSlice controller: Populates EndpointSlice objects (to provide a link between Services and Pods).
* ApiServer : The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.
The main implementation of a Kubernetes API server is kube-apiserver. kube-apiserver is designed to scale horizontally—that is, it scales by deploying more instances.
* etcd : Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.
