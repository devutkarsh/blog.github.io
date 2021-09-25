---
title: "Getting started with Kubernetes Manifests"
date: 2021-09-23
description: "How to write a basic deployment and service."
ogimage: assets/images/tech/kubernetes.jpeg
tags: 
- kubectl
- microservice
- kubernetes
categories:
- tech
---
Deploy your first container on Kuberentes cluster
---
![Kubernetes](assets/images/tech/kubernetes.jpeg)

Well if you have followed us on the previous blog post on [how to create a kubernetes cluster on AWS EKS](../how-to-create-a-kubernetes-cluster-on-aws-eks) or have gone with local kubernetes clusters like [```minikube```](https://minikube.sigs.k8s.io/docs/start/) etc. Then you must be aware that to manage all the resources & objects on your cluster you will need a command-line tool [```kubectl```](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/) which interacts on your behalf with the ```kube-api-server``` to perform almost all operations on your cluster.

### What are deployments and pods?
A deployment can be understood as a template that defines how your actual running container will look based on the specifications provided. Each actual running container is built from the deployment specified and is termed as a pod.
A sample deployment will look like below -

```deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: s3-streamer
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: s3-streamer # used by deployment
  template:
    metadata:
      labels:
        app: s3-streamer # used for marking pods
    spec:
      containers:
        - name: s3-streamer
          image: devutkarsh/s3-streamer #image hosted on docker hub
          ports:
            - containerPort: 9999 
```

> This is a deployment manifest for a microservice named s3-streamer written on Java to stream AWS S3 objects into the cluster. More details about the service can be [found here](../streaming-aws-s3-objects-in-aws-eks-cluster/).

To apply this deployment you can go to the saved directory and run the following command in your OS terminal-
```zsh
kubectl apply -k deployment.yaml
```
And see a similar output -
```
deployment.apps/s3-streamer created
```
Then you can run commands to get deployment and pods to see them running status -
```zsh
devutkarsh@ud s3-streamer % kubectl get deployment
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
s3-streamer   1/1     1            1           19m
devutkarsh@ud s3-streamer % kubectl get pods      
NAME                           READY   STATUS    RESTARTS   AGE
s3-streamer-78779d7c85-j7clz   1/1     Running   0          19m
```
You can further run  **describe** commands on your resources to understand more on what's happening by running - ```kubectl describe pod s3-streamer-78779d7c85-j7clz```

> What's important to note here is that there can be multiple pods running for the same deployment based on the replica count set, which comes in handy in the case of load distribution and canary deployments. 
> Also, another important point to note down here is that this application running on port 9999 is just like a black box and has no cluster endpoint assigned to it. So it will not be accessible over the network, but can only be accessed by port-forwarding the pod or deployment directly as of now. Each pod will have its random IP which we can't guess because pods are volatile.

### What is a Service?
As stated in the note above that the deployed pods will not be accessible over the cluster network because they have no known IP assigned (but some random IPs based on your cluster CIDR block). This is exactly the problem that a service solves for you. It assigns a specific cluster DNS endpoint to your deployment and then all the pods that are running for that particular deployment are load-balanced under a single DNS name in a round-robin fashion. This all networking is taken care of by ```kube-proxy``` agent on your cluster.

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

To apply this service run the following command in your OS terminal -
```zsh
kubectl apply -k service.yaml
```
And see a similar output -
```
service/s3-streamer created
```
Then you can run commands to get or describe your service to see the status -
```zsh
devutkarsh@ud s3-streamer % kubectl get svc
NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
s3-streamer               ClusterIP   10.105.14.47    <none>        9999/TCP   29s
```
> Now this service is configured with a cluster IP. Also, the application would be available at http://service-name:port-number/ i.e. http://s3-streamer:9999/. We can also get an external IP attached, but we will see that some other time.

### The bridge in between
The deployment is an independent resource and so is the service. So we need to make sure that both are intertwined, which is taken care of by the ```label``` in deployment.yaml and ```selector``` in service.yaml. In this way, the provided selector ```app=s3-streamer``` in service yaml knows that it needs to take care of all the pods with the same labels while load balancing.

To test this we will create a throw-away pod using busy box docker image with curl utility installed and bash right into its shell -
```zsh
kubectl run curl-test --image=radial/busyboxplus:curl -i --tty --rm
```

The run curl command, keep the output handy and exit the pod. -
```zsh
[ root@curl-test:/ ]$ curl -I  http://s3-streamer:9999/abc/xyz
HTTP/1.1 400 
Content-Length: 0
Date: Fri, 24 Sep 2021 19:44:01 GMT
Connection: close

[ root@curl-test:/ ]$ exit
Session ended, resume using 'kubectl attach curl-test -c curl-test -i -t' command when the pod is running
pod "curl-test" deleted
```

This resulted in a 400 status error because it was looking for an AWS S3 bucket **abc** with object key **xyz** which is not present, but we know that we were able to trigger the service communication at least.

To verify that s3-streamer service was invoked using the DNS assigned on the local cluster network from inside of our throw-away pod, we will take a look at the logs generated by s3-streamer service as pasted below. To confirm we can see line number 27 in the output - 
```java
devutkarsh@ud s3-streamer % kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
s3-streamer-f8585b8cc-c8rvb   1/1     Running   0          12m
devutkarsh@ud s3-streamer % kubectl logs s3-streamer-f8585b8cc-c8rvb

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.4.3)

2021-09-24 19:37:02.515  INFO 1 --- [           main] com.s3streamer.Init                      : Starting Init using Java 1.8.0_212 on s3-streamer-f8585b8cc-c8rvb with PID 1 (/s3-streamer-0.0.1-SNAPSHOT/lib/s3-streamer-0.0.1-SNAPSHOT.jar started by root in /)
2021-09-24 19:37:02.521  INFO 1 --- [           main] com.s3streamer.Init                      : No active profile set, falling back to default profiles: default
2021-09-24 19:37:05.322  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 9999 (http)
2021-09-24 19:37:05.345  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-09-24 19:37:05.346  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.43]
2021-09-24 19:37:05.505  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-09-24 19:37:05.506  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 2837 ms
2021-09-24 19:37:06.432  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2021-09-24 19:37:06.879  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 9999 (http) with context path ''
2021-09-24 19:37:06.917  INFO 1 --- [           main] com.s3streamer.Init                      : Started Init in 5.631 seconds (JVM running for 6.555)
2021-09-24 19:40:24.830  INFO 1 --- [nio-9999-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2021-09-24 19:40:24.830  INFO 1 --- [nio-9999-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2021-09-24 19:40:24.842  INFO 1 --- [nio-9999-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 10 ms
Fetching /abc/xyz
Error The bucket is in this region: us-east-1. Please use this region to retry the request (Service: Amazon S3; Status Code: 301; Error Code: PermanentRedirect; Request ID: ....
```

You can refer to the yaml's on my GitHub repo as well --> [Kubernetes deployment and service example yaml](https://github.com/devutkarsh/kubernetes/tree/master/apps/service-bases/s3-streamer). 

I will be next writing on how to set up these manifest for better configuration management to support deployment on different environments and support [Multiple environments (DEV, Staging, QA, Prod) with Kubernetes and Kustomize](#) <!-- ../how-to-configure-kubernetes-manifests-for-multiple-environments-->