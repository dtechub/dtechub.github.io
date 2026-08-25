---
title: TEEs for Dummies
icon: fas fa-shield-alt
order: 3
---

Hands-on guides for setting up and testing trusted execution environments: Intel SGX, Intel TDX, AMD SEV-SNP, Arm TrustZone, Arm CCA, and RISC-V PMP.

This is a dummy-friendly starting point, not a production deployment guide. Source material is also on GitHub: [tees-for-dummies](https://github.com/Yuhala/tees-for-dummies).

## Articles

1. [TEEs for dummies](/posts/tees-for-dummies/) — what a TEE is, process-level vs VM-level isolation, and the TCB
2. [Intel SGX](/posts/intel-sgx/) — hardware check, SDK/PSW/driver install, first enclave
3. [SGX with Gramine](/posts/sgx-gramine/) — unmodified apps in an SGX enclave via a library OS
4. [SGX with Occlum](/posts/sgx-occlum/) — same idea, Docker-based Occlum flow
5. [WebAssembly in SGX](/posts/sgx-wasm/) — Wasm runtime inside the enclave
6. [Arm TrustZone and OP-TEE](/posts/arm-trustzone/) — STM32MP157D-DK1, build, flash, `xtest`
7. [Arm CCA](/posts/arm-cca/) — Realms, RME, and current simulator-based testing
8. [Intel TDX](/posts/intel-tdx/) — confidential VMs, Canonical setup, Nginx/`wrk` overhead
9. [AMD SEV-SNP](/posts/amd-sev-snp/) — confidential VMs on AMD
10. [RISC-V PMP](/posts/riscv-pmp/) — PMP basics and VisionFive 2 setup

All tutorials assume Linux (mostly Ubuntu) and TEE-capable hardware unless a simulator is mentioned.
