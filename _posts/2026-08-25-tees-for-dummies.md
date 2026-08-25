---
title: TEEs for dummies
author: petman
date: 2026-08-25 09:00:00 +0200
categories: [Confidential computing, Trusted execution environments]
tags: [tees, confidential computing, sgx, tdx, sev-snp, trustzone, cca, riscv]
render_with_liquid: false
---

From my experience, it is often difficult to find practical and concise **hands-on** guides on how to use most TEE technologies. Official documentation is useful, but it usually contains far more information than a newbie needs to get the general idea and run a few tests. It is also rare to find articles that bring all of these technologies together.

This series is a relatively easy starting point for "dummies". It is **not** a production deployment guide. When you ship real applications, follow the official documentation. Source material also lives in the [tees-for-dummies](https://github.com/Yuhala/tees-for-dummies) GitHub repo.

The series is organized as follows:

1. [TEEs for dummies](/posts/tees-for-dummies/) (this post)
2. [Intel SGX](/posts/intel-sgx/)
3. [SGX with Gramine](/posts/sgx-gramine/)
4. [SGX with Occlum](/posts/sgx-occlum/)
5. [WebAssembly in SGX](/posts/sgx-wasm/)
6. [Arm TrustZone and OP-TEE](/posts/arm-trustzone/)
7. [Arm CCA](/posts/arm-cca/)
8. [Intel TDX](/posts/intel-tdx/)
9. [AMD SEV-SNP](/posts/amd-sev-snp/)
10. [RISC-V PMP](/posts/riscv-pmp/)

All tutorials are Linux-based (mostly Ubuntu). Some links redirect to the vendor site if you use a different OS, or if the official guide is already simple enough. The tutorials assume you have TEE-enabled hardware and do **not** require remote attestation for hardware verification (used in production). We still note what kind of hardware supports each technology.

## Background on trusted execution environments

> A Trusted Execution Environment (TEE) is a hardware-enforced secure execution context that enables code to be executed in isolation from the primary operating environment, such as the operating system or hypervisor.

TEEs typically provide some or all of the following security guarantees:

- **Confidentiality**: the data is not accessible to unauthorized entities.
- **Integrity**: the data cannot be modified or tampered with by an unauthorized entity; tampering can be detected.
- **Freshness**: we always have the most up-to-date version of the secured data.

TEEs use hardware-based mechanisms in the CPU for encrypting memory and enforcing strong access control.

They can be broadly classified into two categories:

1. **Process-level isolation**: a process creates a secure encrypted (and integrity-protected) region, usually called an *enclave*, in its address space at runtime. Memory pages in this region can only be decrypted in the CPU. Examples: Intel Software Guard Extensions (SGX) and Arm TrustZone (TrustZone does memory access-control checks, not encryption).
2. **Virtual machine (VM)-level isolation**: they protect entire VMs rather than single programs. Examples: Intel Trusted Domain Extensions (TDX), AMD Secure Encrypted Virtualization (SEV) with Secure Nested Paging (SNP), and Arm Confidential Compute Architecture (CCA).

The difference between both categories is the degree of isolation they provide, or the size of the *trusted computing base* (TCB): all software (and hardware) that needs to be trusted.

![Trusted computing base for TEEs](assets/imgs/tees/tee-tcb.png)

> This series is a starting point. It should not be used as a reference guide for deploying applications in production.
{: .prompt-warning }

## What you will need

- A Linux host (Ubuntu 22.04 or 24.04 in most guides)
- TEE-capable hardware for the technology you want to test, or a simulator where one is mentioned
- Basic comfort with the terminal, `git`, and installing packages

Contributions of any kind are welcome on the [tees-for-dummies](https://github.com/Yuhala/tees-for-dummies) repo. Next up: [setting up Intel SGX and running a first enclave](/posts/intel-sgx/).
