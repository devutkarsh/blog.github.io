---
title: "Configuring Kuberenetes for Multiple Environments with Kustomize"
date: 2021-09-27
description: "Multiple environments (DEV, Staging, QA, Prod) with Kubernetes and Kustomize"
ogimage: assets/images/tech/kustomize.jpeg
tags: 
- kubectl
- microservice
- kubernetes
- kustomize
categories:
- tech
---
Kubernetes Configuration Management for DEV, Staging, QA, Production, etc - 
---

![kustomize](assets/images/tech/kustomize.jpeg)

So you may have learnt so far on how to setup a kuberentes cluster and how to write a manifest and then deploy a pod out of it. If not please refer to my the post on [getting started with kuberenetes manifests](../getting-started-with-kubernetes-manifests/) But you may still be puzzled on how to configure it for multiple environments. Yes, it is possible to use a single deployment and service template to be used for multiple environments. This is needed because all your application configs and environment variables will differ with different environment like DEV, Staging, QA and Production. To do this we will leverage  Kustomize, a native of ```kubectl``` CLI used for configuration management.

> **Note** : If you are going to apply the manifest **manually** for different environments, it is very much prone to manual error of not switching the cluster context while performing the deployment and accidently deploying in wrong cluster or environment.

To overcome the above problem, it is advisable to use a continous deployment pipeline which can look for your changes in manifest and deploy it for you in the cluster, instead of you doing it manually. The best in the market for implementing CD are ArgoCD and FluxCD as of 2021. I will write a blog on that too but some other time.








