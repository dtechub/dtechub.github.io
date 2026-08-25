---
title: Running unmodified SGX apps with Occlum
author: petman
date: 2026-08-25 12:00:00 +0200
categories: [Confidential computing, Trusted execution environments, Intel SGX]
tags: [tees, sgx, occlum, libos, docker]
render_with_liquid: false
---

This is part of the [TEEs for dummies](/posts/tees-for-dummies/) series. Similar to [Gramine](/posts/sgx-gramine/), [Occlum](https://github.com/occlum/occlum) is a library OS which allows you to run unmodified applications inside an SGX enclave. It is based on an [academic paper from ASPLOS'20](https://madsys.cs.tsinghua.edu.cn/publication/occlum-secure-and-efficient-multitasking-inside-a-single-enclave-of-intel-sgx/ASPLOS20-shen.pdf).

## Setup

To use Occlum, you can either download its repo from GitHub and [build from source](https://occlum.readthedocs.io/en/latest/build_and_install.html), or use a Docker image with the Occlum runtime already set up. We will go for the latter.

- First, [install Docker](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository).
- Configure Docker:

```bash
sudo groupadd docker
sudo gpasswd -a $USER docker
```

- If not yet installed, install the SGX driver as explained in the [SGX post](/posts/intel-sgx/#sgx-driver-installation). Create softlinks for SGX devices used by Occlum containers:

```bash
mkdir -p /dev/sgx
ln -sf ../sgx_enclave /dev/sgx/enclave
ln -sf ../sgx_provision /dev/sgx/provision
```

## Build and run a program with Occlum

The [Occlum quickstart](https://occlum.readthedocs.io/en/latest/quickstart.html) walks through a similar flow. We do something equivalent below.

Create a `Dockerfile`:

```dockerfile
FROM occlum/occlum:latest-ubuntu20.04

RUN apt-get update && apt-get install -y build-essential make vim libnuma-dev

WORKDIR /root/occlum-tests

COPY helloworld.c Makefile ./
```

Create `helloworld.c`:

```c
#include <stdio.h>

int main()
{
    printf("Helloworld from an Occlum container!\n");
    return 0;
}
```

Create a `Makefile`:

```make
CXX = g++
CC = gcc
OCCLUM_GCC = occlum-gcc

.PHONY = all clean

all: occlum-hello

occlum-hello: helloworld.c
	$(OCCLUM_GCC) -Wall -o $@ $^

clean:
	rm -f occlum-hello helloworld.o
```

Build and run the Docker container:

```bash
docker build -t occlum-hello .  # builds the container from the Dockerfile in the same directory
docker run -it --device /dev/sgx/enclave --device /dev/sgx/provision occlum-hello
```

Once you have access to the container's terminal, build and run an SGX-protected application:

```bash
make occlum-hello
mkdir occlum_instance && cd occlum_instance
occlum init
cp ../occlum-hello image/bin/
occlum build
occlum run /bin/occlum-hello
```

You can adapt the helloworld program to compile and run something more complex.

Stop and remove all Docker containers when you are done:

```bash
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

## Other documentation

- [Occlum official documentation](https://occlum.readthedocs.io/en/latest/)
