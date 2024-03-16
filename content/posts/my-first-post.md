---
title: "My First Post"
date: 2023-01-19T08:38:10+01:00
draft: false
---

Hello World!
This is just a test to see if everything works!

The source code of this blog is hosted on my personal [Gitea](https://gitea.io/) instance.
I am using [HUGO](https://gohugo.io/) as a static site generator and it is hopefully gonna work out well for a self-hosted, simple blog.
Also, the blog automatically built and deployed using [Woodpecker](https://woodpecker-ci.org/).
All of the components are self-hosted on a little Kubernetes cluster (running on [MicroK8s](https://microk8s.io/)) on an [ODROID-M1](https://www.hardkernel.com/shop/odroid-m1-with-8gbyte-ram/) and two [ROCK 4](https://rockpi.org/rockpi4)s.
I also use another ROCK 4 as a backup server and automatically sync the encrypted backups to a Hetzner storage box every week.

There used to also be a Raspberry Pi 4B as part of the cluster, however it was causing problems (for itself and for the other nodes) due to slow SD card speeds (likely the SD card's and not the device's fault).
Now I will probably use the RPi 4B for some services (maybe a CI build agent) that should run outside the cluster (e.g. to protect the cluster from load spikes).

More about my tech stack and hardware might said in a future post!

You can view the code, the content and the Woodpecker config here: https://git.jcm.re/jcm/blog
