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
Kubernetes Configuration Management for DEV, Staging, QA, Production, etc - 
---

![kustomize](assets/images/tech/kustomize.jpeg)

So you may have learned so far how to set up a kuberentes cluster and how to write a manifest and then deploy a pod out of it. If not please refer to my post on [getting started with kuberenetes manifests](../getting-started-with-kubernetes-manifests/). But you may still be puzzled about how to configure it for multiple environments. Yes, it is possible to use a single deployment and service template to be used for multiple environments. This is needed because all your application configs and environment variables will differ with the different environment like DEV, Staging, QA, and Production. To do this we will leverage Kustomize, a native of ```kubectl``` CLI used for configuration management.

So we have got a little project structure where we are going to put manifests for some apps as bases and refer them for building and deploying them in our **dev** and **staging** environment.

![dir-structure](assets/images/tech/dir-structure-kustomize.png)

Here we have a root **apps** folder that contains **service-bases** and **environment** sub-folder. **service-bases** folder is used to keep our template or base manifest yaml files that define what our deployments should look like irrespective of environment. So we have created an app folder named **s3-streamer** in services-bases and have put all your deployment.yaml and service.yaml files. Along with the yaml manifest, there is one extra file that you see here named as **```kustomization.yaml```** which tells what are resources are present at the current directory location.
```kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```
```deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: s3-streamer
  namespace: default
spec:
  replicas: 1 #this will be overridden based on environment
  selector:
    matchLabels:
      app: s3-streamer
  template:
    metadata:
      labels:
        app: s3-streamer
    spec:
      containers:
        - name: s3-streamer
          image: devutkarsh/s3-streamer
          ports:
            - containerPort: 9999
```
```service.yaml
apiVersion: v1
kind: Service
metadata:
  name: s3-streamer
spec:
  selector:
    app: s3-streamer
  type: ClusterIP
  ports:
    - name: http
      port: 9999
      targetPort: 9999
```
This defines our bases.

Now in our **environment** folder, we have our 2 deployment environments say **dev** and **staging** so we have 2 folders respectively for them. In each environment folder (i.e. dev and staging), we will create a similar app folder again and have a ```kustomization.yaml```. But this is an overlay that will be used to patch the templates that we defined in our bases.

To demonstrate this, we want different replica counts of our pods based on our environment. In **dev** we want just 3 pods while in **staging** we might need 6. So we will refer to the base template of the app and patch the required settings.

A setting to be patched would look like this for **dev** environment -
```replica-count.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: s3-streamer
spec:
  replicas: 3 #this will change in staging
```

While patching this, we will need our **kustomization.yaml** which is an overlay and defines where to look for base settings, how to patch the new settings. Which is defined as below -

```kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
   - ../../../service-bases/s3-streamer

patchesStrategicMerge:
  - replica-count.yaml
```

Similarly both files we have in our staging also, but with different replica counts.

To apply these settings for our **dev** environment we will run -
```
kubectl -k environment/dev/s3-streamer
```

To apply these settings for our **staging** environment we will run -
```
kubectl -k environment/staging/s3-streamer
```

So for both environments, bases are the same but we have patched the replica count based on the environment.

You can refer to the project structure on [my git here](https://github.com/devutkarsh/kubernetes/tree/master/apps).

> **Note** : If you are going to apply the manifest **manually** for different environments, it is very much prone to manual error of not switching the cluster context while performing the deployment and accidentally deploying in the wrong cluster or environment.

To overcome the above problem, it is advisable to use a continuous deployment pipeline that can look for your changes in manifest and deploy it for you in the cluster, instead of you doing it manually. The best in the market for implementing CD are ArgoCD and FluxCD as of 2021. I will write a blog on that too but some other time.






