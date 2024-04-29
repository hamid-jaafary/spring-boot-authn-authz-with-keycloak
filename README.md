# Spring Boot Authentication and Authorization with Keycloak Identity and Access Management (IAM)

**A comprehensive guide to leverage the power of Keycloak IAM for authentication (authn) and authorization (authz).**

This repo contains configuration files for setting up authn and authz in `Spring Boot` applications and learn how to seamlessly get `Keycloak IAM` running in **development** & **production** mode. Using `Docker Compose`, streamlined the entire setup process.

# Table of Contents:
- **Introduction to Keycloak IAM**

- **Running Keycloak in Development Mode**

- **Running Keycloak in Production Mode**

- **Enabling Authn & Authz in Spring Boot**

You can skip introduction section, if you're already familiar with Keycloak. Due to high quality of the original documents, many quotes have been used from the original doc.

<details>
<summary><h2>Introduction to Keycloak IAM</h2></summary>

`Keycloak` is an open source Identity and Access Management. Following paragraph has been quoted from main site[^1]:

> Add authentication to applications and secure services with minimum effort.
No need to deal with storing users or authenticating users.
>
> Keycloak provides user federation, strong authentication, user management, fine-grained authorization, and more.

**Features** contains:

* Single-Sign On

* Identity Brokering and Social Login

* User Federation

* Admin Console

* Account Management Console

* Standard Protocols

* Authorization Services

</details>

<details>
<summary><h2>Running Keycloak in Development Mode</h2></summary>

**1. Starting Keycloak using the Original Image:**

Run the following command to start the Keycloak in development mode[^2]:
```shell
docker-compose -f "/path/to/docker-compose-dev.yml" up
```

Environments explanation[^2]:

* *Database*
> If you have db ready for dev environment, specify db connection settings in `KC_DB*` environments[^3].

* *Initial Admin Credentials*
> You have to provide the following environment variables to create the initial admin user:
> `KEYCLOAK_ADMIN`, `KEYCLOAK_ADMIN_PASSWORD`.

* *Exposing the Container to a Different Port*
> The server is listening for http using port 8080. If you want to expose the container using a different port, you need to set the `KC_HOSTNAME_PORT` equal to new port, and update exposed port mapping,
> for example: KC_HOSTNAME_PORT=3000 and change `8080:8080` to `3000:8080`

after receiving following message in docker logs, you can access the admin console page using the provided admin credentials:

`Keycloak 24.0.3 on JVM (powered by Quarkus 3.8.3) started in .... Listening on: http://0.0.0.0:8080`

</details>

<details>
<summary><h2>Running Keycloak in Production Mode</h2></summary>

**2. Building Optimized Keycloak Image with Self-Signed Certificate:**

In order to run keycloak in production, you should first build optimized image. Here is a quote about the importance of the build step:
> For the best start up of your Keycloak container, build an image by running the `build` step during the container build. This step will save time in every subsequent start phase of the container image.

Run the following command to build the optimized Keycloak:
```shell
docker build -f Dockerfile-self-signed -t mykeycloak:latest-self .
```

Run the following command to start the Keycloak with built optimized image which contains self-signed certificate:
```shell
docker-compose -f "/path/to/docker-compose-prod.yml" up -d
```

> Keycloak starts in production mode, using only secured `HTTPS` communication, and is available on `https://localhost:8443`.
>
> * **Health Endpoints:** Health check endpoints are available at `https://localhost:8443/health`, `https://localhost:8443/health/ready` and `https://localhost:8443/health/live`.
>
> * **Metrics Endpoints:** Opening up `https://localhost:8443/metrics` leads to a page containing operational metrics that could be used by your monitoring solution.

Environments explanation:

* *KC_HTTP_ENABLED*:
> When set to false, it disables HTTP access to Keycloak, ensuring only HTTPS connections are permitted.

* *KC_HOSTNAME_STRICT*:
> Enforces strict hostname checking for incoming requests to Keycloak, enhancing security by ensuring requests match the configured hostname exactly.

* *KC_HOSTNAME_STRICT_BACKCHANNEL*:
> Similar to `KC_HOSTNAME_STRICT`, but specifically applies to backchannel requests, further securing communication channels within Keycloak.

**3. Building Optimized Keycloak Image with CA Certificate:**

If you have CA certificate and corresponding RSA private key, you have to create a store of PKCS12 type which contains RSA private key and CA certificate using following command:
```shell
openssl pkcs12 -export -in PUBLIC_CERT.crt -inkey PRIVATE_KEY.key -certfile PUBLIC_CERT.crt -out server.p12 -name server
```

Copy created server.p12 file to certs folder. `COPY` statement for making a copy of cert into docker image is provided in `Dockerfile-with-certs` file. Run the following command to build the optimized Keycloak:
```shell
docker build -f Dockerfile-with-certs -t mykeycloak:latest-certs .
```

change the image name in docker-compose-prod.yml file, and start the Keycloak with built optimized image which contains your ca certificate:
```shell
docker-compose -f "/path/to/docker-compose-prod.yml" up -d
```
</details>

<details>
<summary><h2>Enabling Authn & Authz in Spring Boot</h2></summary>

**4. add corresponding dependencies to pom.xml:**

add two following dependencies to pom.xml of the spring boot project:

```xml
    <dependencies>
  ....
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
  </dependency>
  ....
</dependencies>
```
* **spring-boot-starter-oauth2-client**:

This Maven dependency provides a convenient way to integrate OAuth2 client functionality into a Spring Boot application. If you want to implement client-credentials flow in your application you need this dependency.

* **spring-boot-starter-oauth2-resource-server**:

This Maven dependency enables you to set up a resource server that can authenticate and authorize incoming requests using OAuth2 tokens. In other words, it allows your Spring Boot application to act as an OAuth2-protected resource server.

**5. add required resource files:**

Supposing you've created realm called `myrealm` in Keycloak. Update its address and name in provided `application-client.yml` and `application-resource.yml` files.
Then add them to resources standard maven folder of corresponding project.

**6. enable corresponding profile in application.yml file**

```yaml
spring:
  application:
    name: APPLICATION_NAME
  profiles:
    active: OTHER_PROFILES,client,resource
```
If you want to implement client-credentials flow in your application you need `application-client.yml`.
</details>

<hr/>

Several days were required for me to ensure everything was up and running flawlessly. I must acknowledge the exceptional quality and efficiency of the original documents. I recommend consulting them during your setup process. I hope this material has been informative and helpful in your journey towards implementing authentication and authorization in your spring boot applications using powerful `Keycloak IAM`.

Thank you for your attention and Good Luck!


[^1]: https://www.keycloak.org/
[^2]: https://www.keycloak.org/server/containers
[^3]: https://www.keycloak.org/server/db