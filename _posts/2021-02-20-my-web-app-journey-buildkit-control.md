---
layout: post
type: post
author: Jonathan Tweedle
title: My Web App Journey - Buildkit Control
categories:
- General
categories:
- Programming
- Technology
tags:
- My Web App Journey
- Docker
- OpenFaaS
---

![banner][banner]

One of the steps when trying to [create my own air gapped solution][prev_post] for building and posting functions to my local OpenFaaS using my own private Docker repository. One of the challenges with that setup was my decision against using [Lets Encrypt][letsencrypt] and instead to use [my own CA][article_certificate_authority] that I created myself. This became a problem when using the OpenFaaS cli to perform the build step from my workstation. 

Basically it was unable to perform the final step of the process, which was to publish the now built image over to my private Docker Registry. The reason was simple, it did not trust my registry which was using a certificate signed by an 'Unknown Certificate Authority'.

The solution is to inject my CA certificate into the [buildkit][docker_buildkit] container which is used to build and publish the OpenFaaS images. By injecting my own CA cert, it would then trust the certificate of my local Docker Registry.

If you already tried to and failed to publish, you can skip to [Updating the Container](#updating-the-container).

### Starting Fresh

For my use case I required a buildkit that targets the ARM architecture. I create a build instance and set it as the current builder instance.

> docker buildx create --name multiarch --node multiarch --platform linux/arm64,linux/arm/v8,linux/arm/v7,linux/arm/v6 --use

After running the command, it simply prints out the name of the builder instance. You can verify this by listing out the build instances.

> docker buildx ls

```
NAME/NODE   DRIVER/ENDPOINT             STATUS  PLATFORMS
multiarch * docker-container
  multiarch unix:///var/run/docker.sock running linux/arm64*, linux/arm/v8*, linux/arm/v7*, linux/arm/v6*, linux/amd64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386
default     docker
  default   default                     running linux/amd64, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
```

Now we need to import our CA certificate into the builder instance but because we have not built anything yet, we do not have a destination for the certificate. The build instance works by using a docker container instance. The quickest way to create one is just try and use the build instance to build something. This is easiest done in an empty directory.

> docker buildx build .

```
WARN[0000] No output specified for docker-container driver. Build result will only remain in the build cache. To push result image into registry use --push or to load image into docker use --load
[+] Building 2.4s (3/3) FINISHED
 => [internal] booting buildkit                                                                                                                                                                             2.1s
 => => pulling image moby/buildkit:buildx-stable-1                                                                                                                                                          1.5s
 => => creating container buildx_buildkit_multiarch                                                                                                                                                         0.6s
 => [internal] load build definition from Dockerfile                                                                                                                                                        0.1s
 => => transferring dockerfile: 2B                                                                                                                                                                          0.0s
 => [internal] load .dockerignore                                                                                                                                                                           0.2s
 => => transferring context: 2B                                                                                                                                                                             0.0s
error: failed to solve: rpc error: code = Unknown desc = failed to solve with frontend dockerfile.v0: failed to read dockerfile: open /tmp/buildkit-mount138360677/Dockerfile: no such file or directory
```

### Updating the Container

Once the build container is created, list the docker containers in order to get the container ID.

> docker container ps

```
CONTAINER ID   IMAGE                                      COMMAND                  CREATED              STATUS                     PORTS                           NAMES
c28e2bcc8758   moby/buildkit:buildx-stable-1              "buildkitd"              About a minute ago   Up About a minute                                          buildx_buildkit_multiarch
```

With the Container ID, we can easily copy a local file from workstation into the container. This is great because we can take our CA certificate and copy it over. 

> docker cp homeCA.crt c28e2bcc8758:/usr/local/share/ca-certificates

This wont give you any output, and its not like your cert will automatically be used by the build instance. You need to manually trigger the process for updating the registered CA certificates. 

> docker exec c28e2bcc8758 update-ca-certificates

```
WARNING: ca-certificates.crt does not contain exactly one certificate or CRL: skipping
```

### All Done

You might get a warning after updating the ca certs command but this should be ok. You should be ready to start using the buildkit, for example, publish your OpenFaaS functions before deploying to a local OpenFaaS micro server.

<br>

---
### Page Resources

[Banner Image - pxhere.com - 1073943 ][banner_source]

[How to Create Your Own SSL Certificate Authority for Local HTTPS Development][article_certificate_authority] - Brad Touesnard (2020-06-23)

[Docker build cache sharing on multi-hosts with BuildKit and buildx][blog_buildkit] - Jiang Huan (2019-07-10)

[prev_post]: {% post_url 2021-02-07-my-web-app-journey-mind-the-gap %}
[banner]: /assets/img/structure-sky-building-construction-factory-industry-1073943-pxhere.com.jpg
[banner_source]: https://pxhere.com/en/photo/1073943
[letsencrypt]: https://letsencrypt.org/
[docker_buildkit]: https://docs.docker.com/develop/develop-images/build_enhancements/
[blog_buildkit]: https://medium.com/titansoft-engineering/docker-build-cache-sharing-on-multi-hosts-with-buildkit-and-buildx-eb8f7005918e
[article_certificate_authority]: https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/