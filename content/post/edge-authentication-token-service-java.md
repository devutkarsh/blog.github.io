---
title: "Edge Authentication in Microservices"
date: 2022-07-01
description: "Single Factor Authentication in Java microservice using JWT token and JWKS KeySet validation."
ogimage: assets/images/tech/api-service.jpeg
tags: 
- microservice
- java
- security
categories:
- tech
---

![edge-auth](assets/images/tech/edge-auth.jpeg)
## What is edge authentication?
Edge authentication or authorization is a way to validate all the requests before they reach the actual microservice or private networks i.e. at the edge. This is done at a global level for all the services, so that individual microservice do not have to handle the authentication or authorization on thier own.

This removes the security overhead from all microservices so that they can communicate insecurely in the mesh within a private network and focus on only concerned business logic.

In the images added above we have a gateway, which exposes 2 public endpoints -
- `/login`  is exposed to get username password and on successful authentication responds with a JWT token and claims like username and user-type.
- `/.well-known/jwks.json` is a public JSON WebSet Keys endpoint which has public key to verify and validate the token which is signed with it's corresponding private key. This can be used if any external service wants to verify the authenticity of your token.

Then there is a private endpoint as well -
- `/validate` - This is to also to Authenticate but most importantly Authorize and validate whether a requested endpoint or URL with a valid token is allowed access to the user based on it's user-type. This is controlled using the Open Policy Agent or say OPA.

> So a user with user-type admin should be open to all admin API's but a normal user like a customer should not be allowed to access the admin API.