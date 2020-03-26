---
layout: post
title: How to upgrade portainer running as a docker container
categories:
  - geeking out
tags:
  - linux
  - lxc
  - lxd
  - portainer
  - docker
published: true
---
>*I thought it would be kind of cool if `portainer` could upgrade itself using its gui. Alas it falls into a chicken and egg situation and you need to step in and use the console. Here's how to upgrade `portainer` safely without losing configuration data.*

I'm going to assume you used `docker` without `swarm` when you installed your current version of `portainer` since I haven't tried swarms yet.

We have to make note of the original docker command we've used while initially setting up the container `portainer` lives in. Currently the default way to do that based on the official documentation is:

{: .box-note}
   docker run -d -p 9000:9000 -p 8000:8000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

{: .box-warning}
You'll notice that I am running the `docker` commands as `root`. My environment is within an `lxc` (lxd managed) container and does not expose itself to the Internet. Safe enough for my tinkering purposes.

Here's our roadmap. We're going to stop the container. Remove it. Remove the stale image we have for it.

`portainer` is still running. Find it's name if you don't know and stop it.

{: .box-note}
   root@portainer:~# docker ps
   CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                            NAMES
   f8299de001cc        dnknth/ldap-ui:latest   "/usr/bin/hypercorn …"   11 minutes ago      Up 5 minutes        0.0.0.0:32773->5000/tcp                          ldap-ui
   cb5affb59e4d        portainer/portainer     "/portainer"             7 days ago          Up 2 days           0.0.0.0:8000->8000/tcp, 0.0.0.0:9000->9000/tcp   portainer
   root@portainer:~# docker stop portainer
   portainer
   root@portainer:~# docker ps
   CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                     NAMES
   f8299de001cc        dnknth/ldap-ui:latest   "/usr/bin/hypercorn …"   11 minutes ago      Up 6 minutes        0.0.0.0:32773->5000/tcp   ldap-ui


Now that it's stopped it's time to remove the instance and it's current old image.

{% highlight code %}
`root@portainer:~# docker rm portainer
portainer
root@portainer:~# docker rmi portainer/portainer
Untagged: portainer/portainer:latest
Untagged: portainer/portainer@sha256:026381c60682b82a863f0c3737a9b4a414beaddd4cf050477a7749ff5ac61189
Deleted: sha256:10383f5b5720d7e1f5f824137034c69b7f6d82cc8aa33afcc4e9d508b561af77
Deleted: sha256:01b8db7b5a6e256e37d9a57a6cdcdd07c33fe5051b5d21117ad4842723f68083
Deleted: sha256:dd4969f97241b9aefe2a70f560ce399ee9fa0354301c9aef841082ad52161ec5
root@portainer:~#
``
{% endhighlight %}

Good riddance. Now it's time to get the new fresh goodies.

`root@portainer:~# docker pull portainer/portainer:latest
latest: Pulling from portainer/portainer
d1e017099d17: Pull complete
a7dca5b5a9e8: Pull complete
Digest: sha256:4ae7f14330b56ffc8728e63d355bc4bc7381417fa45ba0597e5dd32682901080
Status: Downloaded newer image for portainer/portainer:latest
docker.io/portainer/portainer:latest
`

Now we have to start the new portainer with the same storage settings as before so it will keep the configuration details and data. Use the same command you used for the old version to launch it. In my case we'll use the command I've listed at the top which is:

{% highlight code %}
root@portainer:~# docker run -d -p 9000:9000 -p 8000:8000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
67c671cea3b3408e76c4abe7cda0fb34ad5f733feb52283e4b22b6fb6f3f5f65
root@portainer:~#
{% endhighlight %}

Looks good enough. Let's see if it's actually running.

{% highlight code %}
root@portainer:~# docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                            NAMES
67c671cea3b3        portainer/portainer     "/portainer"             15 minutes ago      Up 15 minutes       0.0.0.0:8000->8000/tcp, 0.0.0.0:9000->9000/tcp   portainer
f8299de001cc        dnknth/ldap-ui:latest   "/usr/bin/hypercorn …"   28 minutes ago      Up 22 minutes       0.0.0.0:32773->5000/tcp                          ldap-ui
{% endhighlight %}

It is indeed. In my case when I went to the web inteface everything was up and running with the same config as before using the latest version of portainer. But I'm generally a lucky person. I wish you the same kind of luck.
