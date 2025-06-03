# Introduction to docker

This tutorial is to get you started using docker in your work. Docker containers are a highly valuable tool for reproducible workflows as well as installing software in an isolated environment. I more and more use docker containers rather than installing software directly on my computer, which can lead to mismatched versions and broken systems.

## The basic idea

Your computer is basically a CPU and some storage. It wakes up and starts executing code found in storage. (This is of course a simplification.) One piece of software the computer runs is called the kernel. It manages the hardware directly and provides an abstracted environment to applications. Your storage usually contains a file system that defines a namespace. The namespace is quite important. Typically, `/` or `C:\` refers to the root of the file system and all other names are relative to that root. So specifying `/mydir/myfile.txt` goes `/` -> `mydir` -> `myfile.txt`. These names are contantly being passed back and forth between your application and the kernel, for example, `fopen("/mydir/myfile.txt", "r")` which askes the kernel to interact with the disk and file system and open the file. This is all designed. You don't actually need a kernel. The old DOS operating system did not have one! It simply lanched the application and passed out.

## Containers

When you click on an application or type something on the command line, you are accessing the namespace, for example, `/bin/bash` a shell application in Linux. Most systems are setup to run shared libraries that are loaded on demand when you run an application. This is all setup in advance so that the system knows to find those shared libraries somewhere in the namespace, for example, `/usr/lib/libc.so`.

Now normally the kernel provides memory isolation between processes. If one application tries to reach over and poke memory locations allocated by another process, the kernel will terminate the application with a bus error or similar. But the kernel does not guarantee isolation of the file system namespace. That is all handled with permissions or access control lists. This means that application 1 and application 2 may both load `/usr/lib/libc.so` or worse overwrite the library with its own version during installation. This leads to chaos and that is what docker fixes, while adding some other nifty features while it is at it.

It turns out that there is nothing special about `/` being the root of the file system. You can use a command, `chroot`, in Linux to change it. So you can have `/root1/` and `/root2/` as their own unique namespaces. Now if you write into those directory trees, you will not impact the other. After changing the root, you alter `/root1/mydir/myfile.txt` to be accessed as `/mydir/myfile.txt` and the same for the second root directory. This is essentially what docker does. It provides an isolated storage environment while still using the same kernel.

The kernel does not care that there are two separate operating systems installed. It just does its thing. The docker daemon ochestrates the setup and ensures that when file names are passed to the kernel, they are in the correct isolated path. This is different from a virtual machine, because there is no hardware emulation happening. You are still running ordinary applications on the same kernel, just with a different namespace for files. Actually, on non-linux systems, docker does do some emulation, but that is not the important point.

## Setting up docker

First you need to download, install, and run docker. Instructions are online. Docker is a daemon (a background application), plus other components, that manages the containers. Once installed, you should be able to type

```
docker run --rm -ti debian bash
```

and end up with a command prompt running inside the container. This command downloads the debian image and uses that image to bootstrap the container. Here is example output:

```
keittth@INTB-A89940 ~ % docker run --rm -ti debian bash
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
Digest: sha256:bd73076dc2cd9c88f48b5b358328f24f2a4289811bd73787c031e20db9f97123
Status: Downloaded newer image for debian:latest
root@e409e5f086f4:/# whoami
root
root@e409e5f086f4:/# ls
bin  boot  dev	etc  home  lib	media  mnt  opt  proc  root  run  sbin	srv  sys  tmp  usr  var
root@e409e5f086f4:/# exit
exit
keittth@INTB-A89940 ~ %
```

What this did is download a prebuilt image of the Debian Linux operating system and then launched the container and ran the `bash` shell. By default, you run as the Linux `root` superuser that has all privaleges _within the container_ but not outside the container. This is important. Deleting a file while inside the container _does not influence the host filesystem_, unless one explicitely mounts the host file system into the container with write permissions, a topic we will return to later.

The flag `--rm` tells docker to remove the container after running it. This is nice for reproducibility as each time the container is constructed from scratch. I also specified `-t` and `-i` together as `-ti`. The `-t` flag allocates a tty so that you can interact with the container on the command line. Tty's are related to teletypes, those giant machines that make loud printing noises in old movies. The `-i` flag makes the session interactive. If you do not need to type messages in the container, you can omit them and the program will just run, for example,
```
keittth@INTB-A89940 ~ % docker run --rm -ti debian ls  
bin  boot  dev	etc  home  lib	media  mnt  opt  proc  root  run  sbin	srv  sys  tmp  usr  var
keittth@INTB-A89940 ~ % 
```
simply runs the `ls` command and exists. That's pretty nifty if you need to run a command that is not installed on your computer but is available in an existing docker image you can download.
