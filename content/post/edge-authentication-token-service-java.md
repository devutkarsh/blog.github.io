---
title: "Edge Auth - Authentication - Part 1"
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
>Edge authentication or authorization is a way to validate all the requests before they reach the actual microservice or private networks i.e. at the edge. This is done at a global level for all the services, so that individual microservice do not have to handle the authentication or authorization on thier own.
This removes the security overhead from all microservices so that they can communicate insecurely in the mesh within a private network and focus on only concerned business logic.

 A JWT Token is generated at the edge using username & password after a Single Factor Authentication (SFA) in this demo, as in the diagram above. The JWT token is then used in all the subsequent API calls which are validated and authorized at a L7 reverse proxy service like nginx or netflix zuul or spring gateway. Once the token is validated and route access is authorized, the claims from the JWT are decoded and added to the header before passing to the downstream services.

> **For this session we will focus only on how to build a Edge Auth service in SFA. How to generate a JWT token and the JWKS WebKey Sets using Java. We will utilize auth0 for JWT and nibus-jose libraries for JWKS KeySet. This turns the edge-auth service to behave like a ID server.**

In the images added above we have a gateway, which exposes 2 public endpoints -
- `/login`  is exposed to get username password and on successful authentication responds with a JWT token and claims like username and user-type.
- `/.well-known/jwks.json` is a public JSON WebSet Keys endpoint which has public key to verify and validate the token which is signed with it's corresponding private key. This can be used if any external service wants to verify the authenticity of your token. Mostly the JWKS endpoints are public for external services to verify the token whenever required.

Then there is a private endpoint which is token hungry -
- `/validate` - This is to also to Authenticate token but most importantly Authorize and validate whether a requested endpoint or URL with a valid token is allowed access to the user based on it's user-type. This is controlled using the Open Policy Agent or say OPA.
So a user with user-type admin should be open to all admin API's but a normal user like a customer should not be allowed to access the admin API. This is controlled using Open Policy Agent and rego rules.

Let's see how to develop the authentication module first -

The Auth controller for login module and generates JWT token after validating username password. Also, we have the validation endpoints where we generate the JWKS endpoint. And a valiate endpoint for authorization, which we will cover later.
```java
@RestController
@RequestMapping("/")
@Slf4j
public class AuthController {

    @Autowired
    AuthOperations authOperations;

    @PostMapping("/login")
    public ResponseEntity<String> login(@RequestBody HashMap<String, String> credentials) {
        log.info("Login Controller..");
        // call some real validation API with credentials
        return new ResponseEntity<>(authOperations.generateToken(), HttpStatus.OK);
    }

    @GetMapping("/.well-known/jwks.json")
    public ResponseEntity<?> jwksEndpoint() {
        log.info("JWKS keyset Controller..");
        HttpHeaders responseHeaders = new HttpHeaders();
        responseHeaders.set("Content-Type", "application/json");
        return new ResponseEntity<>(authOperations.jwksKeySet(), responseHeaders, HttpStatus.OK);
    }

    @PostMapping("/validate")
    public ResponseEntity<Boolean> validate(@RequestBody HashMap<String, String> requestTokenAndUri) {
        log.info("Authorization token Controller..");
        return new ResponseEntity<Boolean>(authOperations.authorize(requestTokenAndUri), HttpStatus.OK);
    }
}
```

The underlying service operations would be the logic on how to generate a token. We are using the public and private RSA keys for this example. We can store the private key in `application.yml` also. But for this example we will generate them dynamically. 
```java
@Component
@Slf4j
public class AuthOperations {

    @Autowired
    KeysGenerator keys;

    @Value("${server.port}")
    private String serverPort;

    UUID kid = UUID.randomUUID();

    public String generateToken() {
        String token = null;
        try {
            PrivateKey privateKey = (PrivateKey) keys.getRSAKeys(KeysGeneratorStrategy.FILE).get(AuthConstants.PRIVATE_KEY);

            Date today = new Date();
            Calendar c = Calendar.getInstance();
            c.setTime(today);
            c.add(Calendar.DATE, 1);
            Date tomorrow = c.getTime();

            token = Jwts.builder()
            .setHeaderParam("kid", kid)
            .setId(UUID.randomUUID().toString())
            .setIssuedAt(today)
            .setSubject("tester")
            .setIssuer("localhost")
            .setExpiration(tomorrow)
            .signWith(privateKey)
            .compact();

            log.info("Token generated");
        } catch (Exception e) {
            log.error("Cannot generate token", e.getMessage());
        }
        return token;
    }

    @Cacheable
    public Map<String, List<JSONObject>> jwksKeySet() {
        try {
            PublicKey publicKey = (PublicKey) keys.getRSAKeys(KeysGeneratorStrategy.DYNAMIC).get(AuthConstants.PUBLIC_KEY);

            JWK jwk = new RSAKey.Builder((RSAPublicKey) publicKey)
                    .keyUse(KeyUse.SIGNATURE)
                    .keyID(kid.toString())
                    .build();

            Map<String, List<JSONObject>> keyset = new HashMap<String, List<JSONObject>>();
            List<JSONObject> keys = new ArrayList<JSONObject>();

            keys.add(jwk.toJSONObject());
            keyset.put("keys", keys);

            return keyset;
        } catch (Exception e) {
            log.error("Cannot generate JWKS keyset", e.getMessage());
        }
        return null;
    }

    public Boolean authorize(String request) {
        // logic for OPA. JWT token and URI validation
    }
}
```

Beside the service is the core logic for dynamic RSA keys generation. Here we have 2 strategies to load the private keys, which is defined using the enum. We can read the private key from the file (recommended in case of microservices) and dynamically (just for this example). 
```java
@Component
@Slf4j
public class KeysGenerator {

    public enum KeysGeneratorStrategy{
        FILE,
        DYNAMIC
    }

    @Value("${keys.private}")
    private String privateKeyString;

    Map<String, Object> keys;

    public Map<String, Object> getRSAKeys(KeysGeneratorStrategy strategy) {
        if (null == keys) {
            keys = genrateRSAKeys(strategy);
            return keys;
        }
        if (keys.size() > 0) {
            return keys;
        } else {
            keys = genrateRSAKeys(strategy);
            return keys;
        }
    }

    @Cacheable
    public Map<String, Object> genrateRSAKeys(KeysGeneratorStrategy strategy) {
        try {
            log.info("Generating New Keys {}", strategy);
            
            if(strategy.equals(KeysGeneratorStrategy.FILE)){
                PrivateKey privateKey = getPrivateKey();
                PublicKey publicKey  = getPublicKey(privateKey);
                Map<String, Object> keys = new HashMap<String, Object>();
                keys.put(AuthConstants.PRIVATE_KEY, privateKey);
                keys.put(AuthConstants.PUBLIC_KEY, publicKey);
                log.info("Keys Added Successfully");
                return keys;
            }

            if(strategy.equals(KeysGeneratorStrategy.DYNAMIC)){
                KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(AuthConstants.KEYPAIR_GENERATOR_ALGORITHM);
                keyPairGenerator.initialize(2048);
                KeyPair keyPair = keyPairGenerator.generateKeyPair();
                PrivateKey privateKey = keyPair.getPrivate();
                PublicKey publicKey = keyPair.getPublic();
                Map<String, Object> keys = new HashMap<String, Object>();
                keys.put(AuthConstants.PRIVATE_KEY, privateKey);
                keys.put(AuthConstants.PUBLIC_KEY, publicKey);
                log.info("Keys Generated Successfully");
                return keys;
            }

            return null;
        } catch (Exception e) {
            log.error("Cannot generate RSA keys {}", e);
            return null;
        }
    }

    public PublicKey getPublicKey(PrivateKey privateKey){
        try{
            RSAPrivateCrtKey privk = (RSAPrivateCrtKey) privateKey;
            RSAPublicKeySpec publicKeySpec = new java.security.spec.RSAPublicKeySpec(privk.getModulus(), privk.getPublicExponent());
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            PublicKey publicKey = keyFactory.generatePublic(publicKeySpec);
            return publicKey;
        }catch(Exception e){
            log.error("Cannot read public key {}", e);
            return null;
        }
    }

    public PrivateKey getPrivateKey(){
        try{
            log.info("Read key {}", privateKeyString);
            byte[] keyBytes = privateKeyString.getBytes();
            PKCS8EncodedKeySpec spec = new PKCS8EncodedKeySpec(keyBytes);
            KeyFactory kf = KeyFactory.getInstance("RSA");
            return kf.generatePrivate(spec);
        }catch(Exception e){
            log.error("Cannot read private key {}", e);
            return null;
        }

    }
}
```

The authentication source code is available on [github](https://github.com/plotkai-interactive/edge-auth) where you can explore the  The Authorization - Part 2 flow we will be covering in next tutorial along with basics about [Open Policy Agent](#). This whole edge auth module will also be a used in an upcoming tutorial for [istio and it's benefits](../istio-service-mesh-in-kubernetes) in [kubernetes](../getting-started-with-kubernetes-manifests).

Also, we are trying to build the edge-auth as a generic container image for secure system communications in a microservice architecture. If you like building stuff and want to work on FOSS projects feel free to reach out and contribute to the project.

