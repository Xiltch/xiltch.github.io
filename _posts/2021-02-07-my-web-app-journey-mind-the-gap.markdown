---
layout: post
title: My Web App Journey - Mind the Gap
date: 2021-02-07 11:17:43.000000000 +00:00 
type: post
published: true
status: publish
categories:
- General
categories:
- Programming
- Technology
tags:
- Kubernetes
- My Web App Journey
- OpenFaaS
author: Jonathan Tweedle
permalink: "/2021/02/07/my-web-app-journey-mind-the-gap/"
---
Back in December [I began my exploration][prev_post] of creating my own OpenFaaS micro server. Because of natural tendency towards torture, the process was a slight deviation from the normal making for a much more complicated deployment as compared to the [simple guide from Alex][ellis_guide]. 

For some reason I focused on a setup that does not deploy the compiled images to the public github registry or using letEncrypt to act as my Certificate Authority. That decision turned into a snowball of challenges that I needed to find solutions for.

### The Circle of Trust

The linchpin that ensures everything will work together is my own Certificate Authority. The first step is generating your own certificates used for signing certificates which each of the layers will ultimately trust because your personal CA signed them as being trusted them and as long as each of your services trust the CA ... anything using a certificate signed by your CA will therefore also be trusted.

> Generate your own CA

```
openssl genrsa -des3 -out homeCA.key 2048
openssl req -x509 -new -nodes -key homeCA.key -sha256 -days 1825 -out homeCA.pem
```

You will need to answer a few questions, such as a pass phrase for the key but also some additional details but with it being a private CA ... such details wont really serve much purpose.

With your new CA, you are ready for generating your own signed certs. The basic flow is the following:

1. Generate a private RSA key for your service/domain
2. Create a CSR (Certificate Signing Request) using the key
3. Use the CA to take the CSR and generate a signed CRT (Certificate)

To help the process, I use the following script.

> generate_certificate.sh

```shell
#!/bin/sh

if [ "$#" -ne 1 ]
then
  echo "Usage: Must supply a domain"
  exit 1
fi

DOMAIN=$1

cd ~/certs

openssl genrsa -out $DOMAIN.key 2048
openssl req -new -key $DOMAIN.key -out $DOMAIN.csr

cat > $DOMAIN.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = $DOMAIN
EOF

openssl x509 -req -in $DOMAIN.csr -CA ~/certs/homeCA.pem -CAkey ~/certs/homeCA.key -CAcreateserial -out $DOMAIN.crt -days 825 -sha256 -extfile $DOMAIN.ext
```

### Docker Registry

Once you build your OpenFaaS functions, they get packaged and uploaded to a Docker Registry. You might want to simplify the setup and avoid a secure site using HTTPS but then your OpenFaaS server will fail to pull any deployed functions as it requires the registry be secured. So we need to not only create a local registry but also secure the traffic properly.

The easiest method for me was to compose a docker container that allows me to spin up my own registry that imports my CA certificates as well as the certificate files for the docker registry.

The steps to perform are:

* Configure hostname to IP lookup
* Create the directory structure to house the files
* Generate the htpasswd file containing the details to sign into the registry
* Generate signed certificates for the registry using the CA
* Create the docker-compose.yml file
* Initialize and start the docker container

### Configure Hostname

You might decide to host your docker registry directly on your local dev workstation or maybe you have a dedicated system you want to use. And you might end up moving your registry from one machine to another. For that reason I am using a different hostname for the docker registry. In this example it is `docker-registry.home.lan` but feel free to name this as you choose. 

You can then create a static hostname lookup on your router to specify the IP the hostname will resolve to or you can update the `hosts` file relevant to the system you are using to redirect requests. The latter approach is a bit of a pain as this requires more work to update your workstation and the server hosting OpenFaaS should your docker-registry IP change.

#### Directory Structure

I created the following directory that will look like this after I have completed all the steps.

```
Local Registry
├── docker-compose.yml
├── auth
│   └── registry.password
├── ca-certs
│   └── homeCA.pem
├── certs
│   ├── docker-registry.home.lan.crt
│   └── docker-registry.home.lan.key
└── data
```

#### Registry Password

The registry password uses the same format and encoding created using the [htpasswd][man_htpasswd] command. You can either install this on you local WSL (Windows Subsystem for Linux) or spin up a simple httpd container to gain access to the command.

To create the passwd is simple:

```shell
htpasswd -Bbn testuser testpassword
```

This results in the following output, which you can copy and paste into the `registry.password` file.

```
testuser:$2y$05$hMczGGIYEh6WrHs.Dbg79.pB57GRHeKlyBLNTenrnXbw4z2Z5u4ui
```

#### Certificates

Next use the script mentioned above to generate both the private and signed certificate files and place them into the `certs` folder. Then in the `ca-certs` folder you will place a copy of the `homeCA.pem` you created in the beginning.

#### Docker Compose file

Now all that is left is to create the yaml file for composing the container using the default `docker-registry` image. The local volume mounting points will vary based on where your project is located and if your running on a mac/linux or not using WSL2 integration on a windows machine.

```yaml
---
version: '3'

services:
  docker-registry:
    image: registry:2
    container_name: docker-registry
    ports:
      - 5000:5000
    restart: always    
    volumes:
      - /mnt/e/Workspace/Docker/Local Registry/data:/var/lib/registry
      - /mnt/e/Workspace/Docker/Local Registry/ca-certs:/usr/local/share/ca-certificates
      - /mnt/e/Workspace/Docker/Local Registry/certs:/certs
      - /mnt/e/Workspace/Docker/Local Registry/auth:/auth
    environment: 
      ServerName: docker-registry.home.lan
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/docker-registry.home.lan.crt
      REGISTRY_HTTP_TLS_KEY: /certs/docker-registry.home.lan.key
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password

  docker-registry-ui:
    image: konradkleine/docker-registry-frontend:v2
    container_name: docker-registry-ui
    ports:
      - 8080:80
    environment: 
      ServerName: docker-registry.home.lan
      ENV_DOCKER_REGISTRY_HOST: docker-registry
      ENV_DOCKER_REGISTRY_PORT: 5000
```

#### Run it

Now all that is left is to start up your container and hopefully everything works without an issue. Open a command prompt/shell and navigate to the directory where your `docker-compose.yml` is located.

```
docker compose up
```

You can now login to your local repository using the credentials you specified for the htpasswd file.

```
docker login docker-registry.home.lan:5000
```

### OpenFaaS BuildKit

So you have your OpenFaaS server running and while you were able to deploy the sample function that was already pre-compiled to target the Arm 64 architecture, chances are that you are using a x86 platform to write and build your functions on. For this reason you will end up relying on a BuildKit that will help build images that target the ARM architecture. The downside is the buildkit does not trust your CA so will fail to deploy to your docker registry.

The solution I found was to attempt the build process and fail. Once the buildkit container is provisioned, start it and copy my CA over to it. Restart the buildkit and it should now trust our private CA and thus will not fail during the build process.

As mentioned above I navigated to the folder where I had created my OpenFaas functions (on my workstation) and triggered the publish.

```
faas-cli publish -f node-functions.yml --platforms linux/arm64,linux/arm/7 --no-cache
```

This goes through the build motions which first requires creating a docker container for the build kit that targets multiple architectures. Once it has finished building the images, it will try to deploy to your 

You will first need to get the Container ID for the build kit in order to copy your CA from a folder on your host machine (in this case my workstation).

```
docker container ls -a
```

This will give you a list of containers (example below) where you will find the Container ID that has the names of `buildx_buildkit_multiarch`. 

```
CONTAINER ID   IMAGE                                      COMMAND                  CREATED       STATUS                      PORTS                           NAMES
11740c0f2585   moby/buildkit:buildx-stable-1              "buildkitd"              6 weeks ago   Exited (1) 51 seconds ago                                   buildx_buildkit_multiarch
ed3c9250c2dd   registry:2                                 "/entrypoint.sh /etc…"   7 weeks ago   Up 6 minutes                0.0.0.0:5000->5000/tcp          docker-registry
fc8e664eb511   konradkleine/docker-registry-frontend:v2   "/bin/sh -c $START_S…"   7 weeks ago   Exited (255) 5 hours ago    443/tcp, 0.0.0.0:8080->80/tcp   docker-registry-ui
```

Now copy your `homeCA.crt` certificate file directly into the volume of the build kit like so and force the container to update the certificates. You might also need to restart the build kit container if it fails.

```
docker cp homeCA.crt 11740c0f2585:/usr/local/share/ca-certificates
docker exec 11740c0f2585 update-ca-certificates
docker restart 11740c0f2585
```

Now you should be able to publish your functions which should 

```
faas-cli publish -f node-functions.yml --platforms linux/arm64,linux/arm/7 --no-cache
```

### Contributing Research Material

[Alex Ellis OpenFaaS in 15 minutes][ellis_guide]

[How to Create Your Own SSL Certificate Authority for Local HTTPS Development][article_certificate_authority]

[How to create your own private Docker registry and secure it][article_docker_registry]

[htpasswd - Manage user files for basic authentication][man_htpasswd]


[article_certificate_authority]: https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/
[article_docker_registry]: https://gabrieltanner.org/blog/docker-registry
[guide_docker_registry]: https://docs.docker.com/registry/deploying/
[man_htpasswd]: https://httpd.apache.org/docs/2.4/programs/htpasswd.html
[ellis_guide]: https://alexellisuk.medium.com/walk-through-install-kubernetes-to-your-raspberry-pi-in-15-minutes-84a8492dc95a
[prev_post]: {% post_url 2020-12-06-my-web-app-journey-openfaas %}