# Introduction to docker

This tutorial is to get you started using docker in your work. Docker containers are a highly valuable tool for reproducible workflows as well as installing software in an isolated environment. I more and more use docker containers rather than installing software directly on my computer, which can lead to mismatched versions and broken systems.

## The basic idea

Your computer is basically a CPU and some storage. It wakes up and starts executing code found in storage. (This is of course a simplification.) One piece of software the computer runs is called the kernel. It manages the hardware directly and provides an abstracted environment to applications. Your storage usually contains a file system that defines a name-space. The name-space is quite important. Typically, `/` or `C:\` refers to the root of the file system and all other names are relative to that root. So specifying `/mydir/myfile.txt` goes `/` -> `mydir` -> `myfile.txt`. These names are contantly being passed back and forth between your application and the kernel, for example, `fopen("/mydir/myfile.txt", "r")` which askes the kernel to interact with the disk and file system and open the file. This is all designed. You don't actually need a kernel. The old DOS operating system did not have one! It simply lanched the application and passed out.

## Containers

When you click on an application or type something on the command line, you are accessing the name-space, for example, `/bin/bash` a shell application in Linux. Most systems are setup to run shared libraries that are loaded on demand when you run an application. This is all setup in advance so that the system knows to find those shared libraries somewhere in the name-space, for example, `/usr/lib/libc.so`.
