---
title: AMD SEV-SNP for dummies
author: petman
date: 2026-08-25 17:00:00 +0200
categories: [Confidential computing, Trusted execution environments, AMD SEV]
tags: [tees, sev, sev-snp, amd, confidential vms]
render_with_liquid: false
---

This is part of the [TEEs for dummies](/posts/tees-for-dummies/) series. AMD Secure Encrypted Virtualization (SEV) is AMD's counterpart to [Intel TDX](/posts/intel-tdx/): a TEE for confidential virtual machines (CVMs).

> SEV uses AES to encrypt a CVM's private memory. The latter can only be decrypted within the CPU when the CVM is being executed.

> This guide is still a first cut. I have not fully re-tested every step on current AMD hardware, so treat it as a map of the official flow rather than a battle-tested recipe.
{: .prompt-warning }

## Setup

Follow the [README from AMD](https://github.com/AMDESE/AMDSEV/blob/master/README.md). The example below uses the Ubuntu 18.04 setup from that repo.

### Prepare the host OS

Enable source repositories:

```bash
sudo sed -i '/deb-src/s/^# //' /etc/apt/sources.list && sudo apt update
```

Configure the default virtual network, then build and install components used for VM creation and management:

```bash
sudo virsh net-start default
git clone https://github.com/AMDESE/AMDSEV.git
cd distros/ubuntu-18.04
./build.sh  # or sudo ./build.sh
```

### Prepare the VM image

Create an empty virtual disk image:

```bash
qemu-img create -f qcow2 ubuntu-18.04.qcow2 30G
```

Create a private copy of `OVMF_VARS.fd`. This file is a template used to emulate persistent NVRAM storage. Each VM needs a writable copy:

```bash
sudo cp /usr/share/OVMF/OVMF_VARS.fd OVMF_VARS.fd
sudo ln -s /usr/share/OVMF/OVMF_CODE.fd /usr/local/share/qemu/OVMF_CODE.fd
```

Symlink the QEMU binary to `/usr/local/bin` if it is not already there:

```bash
sudo ln -s $(which qemu-system-x86_64) /usr/local/bin/qemu-system-x86_64
```

Install an Ubuntu 18.04 guest:

```bash
wget https://releases.ubuntu.com/bionic/ubuntu-18.04.6-live-server-amd64.iso
sudo ./launch-qemu.sh -hda ubuntu-18.04.qcow2 -cdrom ubuntu-18.04.6-live-server-amd64.iso
```

The `launch-qemu.sh` script is in the `distros` subfolder of the AMDSEV repo.

### Run the SEV guest

```bash
./launch-qemu.sh -hda ubuntu-18.04.qcow2
```

## AMD SEV-SNP

> SEV-SNP is SEV with Secure Nested Paging (SNP). It upgrades SEV by adding integrity and replay protection to guest-private memory, which was absent in original SEV.

Host setup for SNP is still a work in progress in this series. For now, start from the [SEV-SNP whitepaper](https://www.amd.com/content/dam/amd/en/documents/epyc-business-docs/white-papers/SEV-SNP-strengthening-vm-isolation-with-integrity-protection-and-more.pdf) and the AMDSEV repository.

## Resources

- [AMDSEV GitHub repository](https://github.com/AMDESE/AMDSEV)
- [SEV-SNP whitepaper](https://www.amd.com/content/dam/amd/en/documents/epyc-business-docs/white-papers/SEV-SNP-strengthening-vm-isolation-with-integrity-protection-and-more.pdf)
- [Confidential VMs explained: An Empirical Analysis of AMD SEV-SNP and Intel TDX](https://dl.acm.org/doi/pdf/10.1145/3700418)
