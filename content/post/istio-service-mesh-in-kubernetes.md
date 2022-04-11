---
title: "Istio service mesh in kubernetes"
date: 2022-10-16
description: "Routing, Gateway and basic telemetry with Istio Service Mesh in Kubernetes cluster"
ogimage: assets/images/tech/kustomize.jpeg
tags: 
- istioctl
- kubectl
- microservice
- kubernetes
- istio
categories:
- tech
---
Getting a kubernetes cluster up and running is fairly easy now. But I've seen challenges like -
- How to moinitor the network flow & health?
- How to ingress traffic is microservices ?
- How to do canary deployments ?

Then I came across this amazing service-mesh bundle for my infrastucture layer specially made for distributed systems running on Kubernetes. Kuberentes has made container orchestration a cake and then istio is the cherry on the cake that all DevOps need!

To get started you must have a kubernetes cluster running. You can follow [how to create a kubernetes cluster on AWS EKS](../how-to-create-a-kubernetes-cluster-on-aws-eks) or a [minikube](https://minikube.sigs.k8s.io/docs/start/) in your local docker setup.

## Setting up the Istio Service Mesh
1. Download and install istioctl command line

```bash
curl -L https://istio.io/downloadIstio | sh -
```
This will download the package into your local download location.

2. Go into the download istio package and set the bin path so that you can execute the istioctl commands
``` 
cd istio-{version}
```
```
export PATH=$PWD/bin:$PATH
```
3. Now your istioctl command line tool should be ready. Run the below command to setup istio to your kubernetes. Istio will be installed in it's seperate namespace.
```
istioctl install --set profile=demo -y
```
> Note : Make sure the correct kubernetes cluster context is set before you install istio.

4. Now once istioctl commands have executed successfully, you can setup a label to all your k8s deploymetns under default namespace. This would be used by istio to inject a side car proxy and monitor the running pods and services in the cluster.
```
kubectl label namespace default istio-injection=enabled
```

Now you will have a envoy side car proxy service running alongside all your pods.

