---
title: Running unmodified SGX apps with Gramine
author: petman
date: 2026-08-25 11:00:00 +0200
categories: [Confidential computing, Trusted execution environments, Intel SGX]
tags: [tees, sgx, gramine, libos]
render_with_liquid: false
---

This is part of the [TEEs for dummies](/posts/tees-for-dummies/) series. The [previous post](/posts/intel-sgx/) covered the Intel SGX SDK, which requires partitioning an application into trusted and untrusted parts. Gramine avoids that work.

Gramine (formerly Graphene-SGX) is a library OS which allows you to run unmodified applications inside an SGX enclave. It is useful when an application is too complex to partition as required by the SDK.

![Unmodified application in Gramine](assets/imgs/tees/sgx/gramine-sgx.png)

Gramine began as an academic project, based on [Graphene-SGX: A Practical Library OS for Unmodified Applications on SGX](https://www.usenix.org/system/files/conference/atc17/atc17-tsai.pdf). Using a library OS like Gramine increases the TCB of your SGX application, but makes deployment simpler.[^1]

You should already have [SGX software installed](/posts/intel-sgx/#sgx-software-installation) before continuing.

## Installing Gramine

On Ubuntu 22.04 or 24.04:

```bash
sudo curl -fsSLo /etc/apt/keyrings/gramine-keyring-$(lsb_release -sc).gpg https://packages.gramineproject.io/gramine-keyring-$(lsb_release -sc).gpg
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/gramine-keyring-$(lsb_release -sc).gpg] https://packages.gramineproject.io/ $(lsb_release -sc) main" \
| sudo tee /etc/apt/sources.list.d/gramine.list

sudo curl -fsSLo /etc/apt/keyrings/intel-sgx-deb.asc https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/intel-sgx-deb.asc] https://download.01.org/intel-sgx/sgx_repo/ubuntu $(lsb_release -sc) main" \
| sudo tee /etc/apt/sources.list.d/intel-sgx.list

sudo apt-get update
sudo apt-get install gramine
```

For other OSes, see the [official Gramine installation documentation](https://gramine.readthedocs.io/en/latest/installation.html).

Generate an enclave signing key. The generated key is stored in `$HOME/.config/gramine/enclave-key.pem`:

```bash
gramine-sgx-gen-private-key
```

## Deploying an SGX-protected program with Gramine

Gramine programs are configured with a manifest file. Clone the helloworld example from the Gramine repo to see how this is done:

```bash
git clone https://github.com/gramineproject/gramine.git && cd gramine/CI-Examples/helloworld
```

You can build with SGX support (if you have the hardware) or without SGX:

```bash
# build and run with SGX
make SGX=1
gramine-sgx helloworld

# build and run without SGX
make
gramine-direct helloworld
```

You can now replace the content of `helloworld.c` with your own program and test. Check the [Gramine GitHub repo](https://github.com/gramineproject/gramine/tree/master/CI-Examples) for more advanced examples.

Next: the same idea with [Occlum](/posts/sgx-occlum/), another library OS for SGX.

## Other resources

- [Gramine official documentation](https://gramine.readthedocs.io/en/latest/)

[^1]: One core challenge in systems security is the constant trade-off (or "tug of war") between three factors: security, usability, and performance. Improving one often comes at the expense of at least one of the other two.
