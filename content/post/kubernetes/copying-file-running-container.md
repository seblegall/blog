---
title: "Copying files from local file system to a running container"
description: "Introducing Kopy"
date:   2019-07-05T09:00:00+02:00
author: Sébastien Le Gall
categories: [Kubernetes, Docker, cloud native, Go]
---

Container technology is great. It let us build immutable artifacts where processes are isolated and scalable when running from an orchestrator like Kubernetes.

But... Precisely, containers are, by design, immutable. Even if Docker volumes are very helpful for local development, it doesn't help for cloud ready applications. Editing a file on a running container inside a Kubernetes Pod is painful and ends most of the time with a dirty `kubctl exec -it vim /my/file`

Being able to copy a local files to a remote file system, running in a container, may be very helpful for debugging purpose. In many case, stopping, rebuilding and restarting the image is very painful.

For the past 2-3 years, people have started to create tools supposed to handle this issue. But most of them come with some requirements forcing you to change either your image, or your clusters configuration by adding stuff like a ssh server in the containers or a daemon set in your cluster which is in charge of syncing files using Kubernetes volumes or sidecar containers.

But the fact is : trying to sync files to a running container **is not a best practice**. It should not be done except in rare cases. And my point of view about *little tricks* we all use even knowing it's not a good practice is that it shouldn't impact your code or your architecture.

If you need to work with cloud native applications, [Skaffold](https://github.com/GoogleContainerTools/skaffold) is the tool you should use. Not `scp`, `nfs` or worst : `samba`.

Yet, `skaffold` is a dev tool. It handle dev use cases.

For quick debug, I have created `Kopy`.

Kopy is a very simple tool offering a very strong power : ability to copy a file or a directory from local file system to a running container file system (almost) natively.

* No need to install anything else than Kopy
* No need to run any daemon or daemonSet (Kubernetes)
* No need to change and rebuild your container image
* No need to restart your container
* No need to add anything in your Dockerfile

`kp` simply works like a regular `cp` but from local to a running container. The container may be run by Docker or by Kubernetes, inside a pod. Kopy support both.

Example with Docker :

```sh
kp -c {container_id} /path/to/my/file /tmp
```

Example with Kubernetes :

```sh
kp -p {pod_name} -n {namespace} -c {container} /path/to/my/file /tmp
```

If you want to know more about Kopy, have a look on the [Github Readme](https://github.com/seblegall/kp)
