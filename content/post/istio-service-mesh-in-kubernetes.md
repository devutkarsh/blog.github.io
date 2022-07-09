---
title: "Istio service mesh in kubernetes"
date: 2022-06-24
description: "Routing, Gateway and basic telemetry with Istio Service Mesh in Kubernetes cluster"
ogimage: assets/images/tech/istio.jpeg
tags: 
- istioctl
- kubectl
- microservice
- kubernetes
- istio
categories:
- tech
---
Setting up Istio Service Mesh on Kubernetes 
---
Getting a kubernetes cluster up and running is fairly easy now. But I've seen challenges like -
- How to moinitor the network flow & health?
- How to ingress traffic is microservices ?
- How to do canary deployments & prevent downtimes?
- How to perform Auth on incoming requests?

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
![istio](assets/images/tech/istio-install.png)

> Note : Make sure the correct kubernetes cluster context is set before you install istio. 

> Also the more better way is to generate the manifests.yaml which can be used with kubectl so that you don't have to install istio shell when migrating to different environments. Manifests are easy to [configure and migrate using Kustomize.](../configuring-kubernetes-for-multiple-environments-with-kustomize)

4. Now once istioctl commands have executed successfully, you can setup a label to all your k8s deploymetns under default namespace. This would be used by istio to inject a side car proxy and monitor the running pods and services in the cluster.
```
kubectl label namespace default istio-injection=enabled
```

Now you will have a envoy side car proxy service running alongside all your pods.


To check if istio side car proxy is in place describe the pod and you will see 2 container are running.
![istio](assets/images/tech/istio-pods.png)

> As of June 2022, if you are on Apple Silicon M1 chip, and facing issues related to iptables in istio-init container, take a look at following way to install istio - [Istio on Apple MacOS M1 workaround.](https://stackoverflow.com/questions/72073613/istio-installation-failed-apple-silicon-m1/72837452#72837452).

Now when you have your istio side cars up and running. We will see the [benefits on istio](../uses-of-istio-service-mesh) in routing traffic and canary depolyments.

