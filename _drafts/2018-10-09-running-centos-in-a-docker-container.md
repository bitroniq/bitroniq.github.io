https://osric.com/chris/accidental-developer/2017/08/running-centos-in-a-docker-container/
```
Running CentOS in a Docker container
I’m just getting started with Docker. I’ve thought for years that containerization is a great idea, but I haven’t actually done anything with containers yet. Time to get started.

I ran through a couple tutorials on the Docker docs site and created a cloud.docker.com account to get some basic familiarity.

I found the CentOS container repository on Docker Hub: https://hub.docker.com/_/centos/

Let’s try running it!

$ docker pull centos
$ docker run centos

Did it do anything? It looks like it did something. At least, it didn’t give me an error. What did it do? How do I access it?

$ docker container ls
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

Nothing is actively running. That makes sense, because we’re not telling the containerized OS to do anything — it starts, it doesn’t have anything to do, and so it shuts down immediately. Instead we can tell it to run interactively and with a terminal by specifying a couple options:

-i, --interactive
-t, --tty (“allocate a pseudo-TTY”, i.e. a terminal)
(see docker run --help for details)

$ docker run -i -t centos
[root@4f0b435cdbd7 /]#

I’m in!

What if I want to modify the container? Right now it is pretty bare-bones. For example, this doesn’t even have man installed:

[root@4f0b435cdbd7 /]# man man
bash: man: command not found

[root@4f0b435cdbd7 /]# yum install man
...
[root@4f0b435cdbd7 /]# man man
No manual entry for man

Quite the improvement! Now we need to save our change:

[root@4f0b435cdbd7 /]# exit

$ docker commit 4f0b435cdbd7 man-centos
$ docker run -i -t man-centos

[root@953c512d6707 /]# man man
No manual entry for man

Progress! Now we have a CentOS container where man is already installed. Exciting.

I can’t (that I know of) inspect the container and know whether or not man is installed without running it. That’s fine for many cases, but next I will attempt to figure out how specify via a Dockerfile that man is installed.
```


# Now my listings:
```
➜  centos-systemd docker pull centos
Using default tag: latest
latest: Pulling from library/centos
Digest: sha256:6f6d986d425aeabdc3a02cb61c02abb2e78e57357e92417d6d58332856024faf
Status: Image is up to date for centos:latest
➜  centos-systemd docker run -i -t centos
[root@ad5356878673 /]# 
```

Yeah! I'm in :-)
```
[root@ad5356878673 /]# cat /etc/redhat-release
CentOS Linux release 7.5.1804 (Core)
```

Now let's test installing epel release:

https://fedoraproject.org/wiki/EPEL
RHEL/CentOS 7:
   # yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

on RHEL 7 it is recommended to also enable the optional and extras repositories since EPEL packages may depend on packages from these repositories:
`   # subscription-manager repos --enable "rhel-*-optional-rpms" --enable "rhel-*-extras-rpms"`



