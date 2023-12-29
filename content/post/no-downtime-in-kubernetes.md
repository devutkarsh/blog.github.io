---
title: "No downtime for microservices"
date: 2023-12-29
description: "How to prevent service downtime in kubernetes"
ogimage: assets/images/tech/kubernetes.jpeg
tags: 
- kubectl
- microservice
- kubernetes
categories:
- tech
---
![Kubernetes](assets/images/tech/kubernetes.jpeg)

# Achieving No Downtime in Kubernetes Deployments

## Introduction

Kubernetes is a powerful container orchestration platform that enables the deployment, scaling, and management of containerized applications. When deploying applications in a production environment, minimizing downtime is crucial to ensure a seamless user experience. In this article, we will explore strategies and best practices to achieve zero or minimal downtime during Kubernetes deployments.

## 1. Rolling Deployments

One of the fundamental features of Kubernetes is rolling deployments. Kubernetes supports updating a service by gradually replacing instances of the old application with the new one. This ensures that a portion of your application is always available, reducing downtime.

### Example Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: sample-app
        image: your-registry/sample-app:latest
```

Apply changes:

```bash
kubectl apply -f updated-deployment.yaml
```

Kubernetes will gradually replace pods, ensuring a smooth transition.

## 2. Readiness Probes

Use readiness probes to delay traffic until the application is ready to serve. This prevents routing requests to instances that are not yet fully operational.

### Example Readiness Probe:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-app
spec:
  containers:
  - name: sample-app
    image: your-registry/sample-app:latest
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

This configuration checks the `/healthz` endpoint and delays traffic until the probe succeeds.

## 3. Horizontal Pod Autoscaling

Automatically adjust the number of pod replicas based on CPU or memory usage. This helps maintain performance and availability under varying loads.

### Example Horizontal Pod Autoscaler:

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: sample-app-autoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: sample-app
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 80
```

This example scales the number of replicas based on CPU usage, ensuring your application can handle increased demand.

## 4. Blue-Green Deployments

Implementing a blue-green deployment strategy involves running two identical environments – blue (current) and green (new). Switching between them is a matter of updating the routing configuration.

### Example Service Switch:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sample-app
spec:
  selector:
    app: sample-app-blue  # or sample-app-green
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

Update the service to switch between blue and green deployments.

```bash
kubectl apply -f updated-service.yaml
```

## Conclusion

Minimizing downtime in Kubernetes deployments involves careful planning and leveraging the platform's features. Rolling deployments, readiness probes, horizontal pod autoscaling, and blue-green deployments are powerful tools to ensure a seamless user experience. Combining these strategies allows you to deploy updates with confidence, knowing that your application remains available and responsive. As you navigate the world of Kubernetes deployments, always test changes in staging environments before applying them to production to catch potential issues early in the development lifecycle.

