Although you can set via the `USER` instruction of a `Dockerfile` what user a container image should run as, some container platforms will override the user the container image specifies. This is the case for multi tenant Kubernetes clusters, such as OpenShift.

In this case each distinct namespace in the Kubernetes cluster is allocated a different range of user IDs they can use, with the lowest in that range being what container images would usually be run as.

The forced use of different user IDs for applications deployed in each namespace is to provide an extra layer of security between applications of distinct users. By running with an assigned user ID different to other namespaces, if a way was found of escaping the container runtime, you would be neither the `root` user, nor a user ID the same as applications running in different namespaces. This limits even further the extent of damage that could be done in the case of a compromise.

To simulate this type of deployment environment, you can run the container image with an arbitrary user ID which doesn't have a matching user in the `/etc/passwd` file of the container image.

```execute
docker run --rm -u 1000000 greeting id
```

When using `docker run` the output will be:

```
uid=1000000 gid=0(root) groups=0(root)
```

Of note, the result here can be different on other container runtimes. If you had been using `podman` instead of `docker`, the output would be:

```
uid=1000000(1000000) gid=0(root) groups=0(root)
```

Subtle differences like this could cause problems if you developed your container image for one container runtime, but later then needed to deploy it on another.

To illustrate the difference, run:

```execute
docker run --rm -u 1000000 greeting whoami
```

For `docker run` the output will be the error:

```
whoami: cannot find name for user ID 1000000
```

If instead you were to use `podman run`, the output would be:

```
1000000
```

The reason the error occurs in the case of `docker` is because there is no entry in the `/etc/passwd` file of the container image, and so `whoami` cannot look up the user name for the user ID.

It does succeed for `podman run` though, as `podman` will detect that there is no entry in `/etc/passwd` and inject one for you automatically when the container is started. As a result, had you `podman` and run:

```
podman run --rm -u 1000000 greeting grep 1000000 /etc/passwd
```

it will yield:

```
1000000:x:1000000:0:container user:/opt/app-root/src:/bin/sh
```

When adding the entry, `podman` will use as the home directory for the user the last value of `WORKDIR` as set in the `Dockerfile`.

If using `docker run` no such entry would exist.

For a container image that needs to be portable to different container runtimes, you will need to accomodate for the lack of an entry in the `/etc/passwd` file. This is because a lack of an entry can cause some applications to fail.

If you had originally used `podman` when developing your container image and not taken this into consideration, your image could fail when run on `docker`.

Before we address that though, we are going to look at the issue of file system access when run as a random user ID.
