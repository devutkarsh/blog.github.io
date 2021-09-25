---
title: "API Gateway vs Edge Proxy service vs API Microgateway"
date: 2020-06-01
description: "A breif introduction to Microservices."
ogimage: assets/images/tech/api-microgateway.jpeg
tags: 
- cloud
- microservice
categories:
- tech
---

## Routing request to different microservices from a single REST endpoint

![s3-and-lambda](assets/images/tech/api-microgateway.jpeg)

Having a single entry point for your all backend microservices in your application comes in handy for the frontend applications. The front end just needs to know one domain to reach for anything. Since in distributed architecture, we have multiple services running, it becomes easier for the frontend client to communicate only with one gateway only without knowing what happens in the background.

Before we proceed you need to understand the difference between the three -


> **API Gateway:** This is an application layer between the backend and the frontend. It is exposed to the public and provides an abstraction to the client and a seamless experience. Provides a single entry point and can perform operations like IP or MAC filtering, etc. This is commonly used for Monolithic architecture.


> **API MicroGateway:** This is similar to API-gateway but used in a microservice architecture, since a single api-gateway is not feasible where all services are running in a distributed fashion, so we end up having multiple micro gateways. We can have an api-microgateway at entry point routing to different subset of your application business, where internally that business module of application may have another api-microgateway to route to different other services. All api-gateway can perform their own authentication or routing, etc.


> **Edge-Proxy Service:** This is a service running on the API gateway resolving the proxying, routing, etc. This is just a logical layer. There can be multiple edge-services running on your api-gateway, but practically there is always one.

### Benefits of using API Microgateway

- You can have a single entry point, so your frontend client doesn't need to maintain multiple access points. CORS can be enabled for one service instead of multiple services.
- Dynamic routing for all requests. Can be very handy for load shedding.
- All the requests can be authenticated using JWT token before hitting the backend service. All security modules can be applied on the request beforehand, so each backend microservice does not need to worry about it.
- Fallback cache data or static data can be used if some backend service is unavailable momentarily. This ensures uptime for all services when you have CI/CD going on in the backend.
- Various request and response filters can be used to log or sanitize the requests or updating headers for the downstream.
- Global trace ID can be injected in the header that can be used across all services for logging and tracing.
- Can be connected with service discovery modules like Eureka.
- Circuit Breaker like Hystrix can be connected for fault tolerance.  

The usage of API micro gateway is immense and is one of the most common microservice used across container-based software development at the front door.

Now we have learned the usage, we will be writing one api-gateway using Netfix's Zuul which is the most popular and commonly used library -> [api-microgateway using Netflix zuul on Java Springboot](https://blog.devutkarsh.com/api-microgateway-service-using-netflix-zuul-on-java)
