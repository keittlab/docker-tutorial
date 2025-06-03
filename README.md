# Introduction to docker

This tutorial is to get you started using docker in your work. Docker containers are a highly valuable tool for reproducible workflows as well as installing software in an isolated environment. I more and more use docker containers rather than installing software directly on my computer, which can lead to mismatched versions and broken systems.

## The basic idea

Your computer is basically a CPU and some storage. It wakes up and starts executing code found in storage. (This is of course a simplification.) One piece of software the computer runs is called the kernel. It manages the hardware directly and provides an abstracted environment to applications. Your storage usually contains a file system that defines a namespace. The namespace is quite important. Typically, `/` or `C:\` refers to the root of the file system and all other names are relative to that root. So specifying `/mydir/myfile.txt` goes `/` -> `mydir` -> `myfile.txt`. These names are contantly being passed back and forth between your application and the kernel, for example, `fopen("/mydir/myfile.txt", "r")` which askes the kernel to interact with the disk and file system and open the file. This is all designed. You don't actually need a kernel. The old DOS operating system did not have one! It simply lanched the application and passed out.

## Containers

When you click on an application or type something on the command line, you are accessing the namespace, for example, `/bin/bash` a shell application in Linux. Most systems are setup to run shared libraries that are loaded on demand when you run an application. This is all setup in advance so that the system knows to find those shared libraries somewhere in the namespace, for example, `/usr/lib/libc.so`.

Now normally the kernel provides memory isolation between processes. If one application tries to reach over and poke memory locations allocated by another process, the kernel will terminate the application with a bus error or similar. But the kernel does not guarantee isolation of the file system namespace. That is all handled with permissions or access control lists. This means that application 1 and application 2 may both load `/usr/lib/libc.so` or worse overwrite the library with its own version during installation. This leads to chaos and that is what docker fixes, while adding some other nifty features while it is at it.

It turns out that there is nothing special about `/` being the root of the file system. You can use a command, `chroot`, in Linux to change it. So you can have `/root1/` and `/root2/` as their own unique namespaces. Now if you write into those directory trees, you will not impact the other. After changing the root, you alter `/root1/mydir/myfile.txt` to be accessed as `/mydir/myfile.txt` and the same for the second root directory. This is essentially what docker does. It provides an isolated storage environment while still using the same kernel. The kernel does not care that there are two separate operating systems installed. It just does it thing. The docker daemon ochestrates the setup and ensures that when file names are passed to the kernel, they are in the correct isolated path. This is different from a virtual machine, because there is no hardware emulation happening. You are still running ordinary applications on the same kernel, just with a different namespace for files. Actually, on non-linux systems, docker does do some emulation, but that is not the important point.

## Setting up docker

First you need to download, install, and run docker. Instructions are online. Docker is a daemon (a background application), plus other components, that manages the containers. Once installed, you should be able to type
```
docker run --rm -ti debian-slim bash
```
and end up with a command prompt running inside the container. This command downloads the debian-slim image and uses that image to bootstrap the container.

