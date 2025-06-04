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
keittth@INTB-A89940 ~ % docker run --rm debian ls
bin
boot
dev
etc
home
lib
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
keittth@INTB-A89940 ~ %
```

simply runs the `ls` command and exists. Notice in this case the output is a bit different because there was not post-processing. The text was simply spit out directly to the standard output. That's pretty nifty if you need to run a command that is not installed on your computer but is available in an existing docker image you can download.

Note that if you are using the docker desktop, you can also do these commands if you hunt around in the interface. In the dashboard, you can download an image and run it as a container. There are tabs for inspecting the container and issuing commands directly inside the container. Try it out.

## Images

In docker, an image is a static snapshot of the file system / namespace. Images are built from instructions and then stored locally or may be uploaded to a registry. Docker runs a registry, Docker Hub, that contains 10's of thousands of images covering many applications. There are other registries as well and you can run your own registry, although it is a pain to setup the certs as everything has to go over encrypted channels (https).

To build your own image, you create a dockerfile and run `docker build -f my_docker_file`. If you name your docker file `Dockerfile` and run the command in the same directory, you can simply do `docker build .`, where `.` specifies the current directory. Note that I am shoing command line commands. You can do this in the dashboard as well and `vscode` has plugins to manage all of this.

Lets try it out. I am doing this in Ubuntu Linux. The commands are the same in the OSX shell or Windows linux subsystem.

```
mkdir docker-test
cd docker-test
# This writes the lines after into a file
cat<<EOF > Dockerfile
FROM alpine:latest
CMD ["echo", "Hello, Docker!"]
EOF
docker build . -t minitest
docker run minitest
```

If I paste this into my terminal logged into the Ubuntu server, I see:

```
tkeitt@geo:~/projects/docker-tutorial$ mkdir docker-test
cd docker-test
# This writes the lines after into a file
cat<<EOF > Dockerfile
FROM alpine:latest
CMD ["echo", "Hello, Docker!"]
EOF
docker build . -t minitest
docker run minitest
[+] Building 1.2s (6/6) FINISHED                                                                                                                 docker:default
 => [internal] load build definition from Dockerfile                                                                                                       0.0s
 => => transferring dockerfile: 87B                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                                                           0.8s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                                          0.0s
 => => transferring context: 2B                                                                                                                            0.0s
 => [1/1] FROM docker.io/library/alpine:latest@sha256:8a1f59ffb675680d47db6337b49d22281a139e9d709335b492be023728e11715                                     0.3s
 => => resolve docker.io/library/alpine:latest@sha256:8a1f59ffb675680d47db6337b49d22281a139e9d709335b492be023728e11715                                     0.0s
 => => sha256:08001109a7d679fe33b04fa51d681bd40b975d8f5cea8c3ef6c0eccb6a7338ce 1.02kB / 1.02kB                                                             0.0s
 => => sha256:cea2ff433c610f5363017404ce989632e12b953114fefc6f597a58e813c15d61 581B / 581B                                                                 0.0s
 => => sha256:fe07684b16b82247c3539ed86a65ff37a76138ec25d380bd80c869a1a4c73236 3.80MB / 3.80MB                                                             0.2s
 => => sha256:8a1f59ffb675680d47db6337b49d22281a139e9d709335b492be023728e11715 9.22kB / 9.22kB                                                             0.0s
 => => extracting sha256:fe07684b16b82247c3539ed86a65ff37a76138ec25d380bd80c869a1a4c73236                                                                  0.1s
 => exporting to image                                                                                                                                     0.0s
 => => exporting layers                                                                                                                                    0.0s
 => => writing image sha256:05689c0132c18cefc6ff964b91874df0f18f9b6ab2244ff816b32023392e0245                                                               0.0s
 => => naming to docker.io/library/minitest                                                                                                                0.0s
Hello, Docker!
tkeitt@geo:~/projects/docker-tutorial/docker-test$ docker image ls
REPOSITORY                 TAG             IMAGE ID       CREATED         SIZE
minitest                   latest          05689c0132c1   5 days ago      8.31MB
# continues on because I have a bunch of orphan images on geo
```

I will add this docker file to the repo on github. You can clone the repo and run the command without creating the file. Note that I gave the image a _tag_ with the `-t` switch. It is helpful to name your images so you can find them later. Docker tracks the images by their cryptographic signatures and internal id's. If you want to list the images on your system, you can type `docker image ls`. You can remove images using `docker image rm <image id>` and you can get rid of zombie images with `docker image prune`.

Don't worry to much about keeping images around. I routinely delete them all for various reasons. Images are _meant to be disposable_. Never store anything important inside an image. Keep the docker file and you have all of the commands needed to fully reproduce your workflow. Images are not virtual machines. Do not think about them like installing a computer inside your computer. They simply isolate a file namespace from your main system. Build your images to do exactly one thing. It is just an ephemeral environbment in which to run an application.

