---
title: Arm TrustZone and OP-TEE for dummies
author: petman
date: 2026-08-25 14:00:00 +0200
categories: [Confidential computing, Trusted execution environments, Arm TrustZone]
tags: [tees, trustzone, op-tee, arm, stm32]
render_with_liquid: false
---

This is part of the [TEEs for dummies](/posts/tees-for-dummies/) series. Arm TrustZone is the main process-level TEE on Arm, and OP-TEE is the practical framework we will use to test it.

Arm TrustZone (TZ) is a hardware security extension in ARM-based processors that splits the processor into two protection domains: a **secure world**, where data is processed securely and isolated from the host OS (or hypervisor), and a **normal world** which has no access to secure-world resources. At any point in time, the processor operates exclusively in one of these worlds. A privileged software component called a *secure monitor* enables context switching between both worlds using *secure monitor calls* (SMC), analogous to SGX ecalls and ocalls.

A hardware component called the **TrustZone address space controller** (TZASC) enforces the separation between the secure world and the normal world by controlling access to physical memory. TZASC can be programmed so that some parts of physical memory (contiguous blocks) are only accessible in the secure world, or in both worlds. A special bit called the *non-secure* (NS) bit, stored in the *secure configuration register* (SCR), is used to determine which world the processor is currently operating in.

![Arm TrustZone architecture](assets/imgs/tees/trustzone/tz-arch.png)

A similar hardware component called the **TrustZone protection controller** (TZPC) arbitrates access to peripherals. TZPC can be configured so that a peripheral is accessible only from the secure world, or from both worlds.

Contrary to TEE technologies like SGX which encrypt data stored in memory, TrustZone only performs access-control checks to ensure confidentiality.

## Hardware and software setup

From my experience with Arm TrustZone, it is often trickier to get the right hardware with decent software support when compared to server-side TEEs like SGX. I believe this is partly due to the fragmented nature of the Arm ecosystem: Arm licenses IP to many SoC vendors (STMicro, NXP, HiSilicon, Broadcom, and others). For beginners, I recommend boards by [STMicroelectronics](https://www.st.com/). They are relatively cheap and provide excellent documentation. STM provides both Cortex-M processors for low-power microcontroller applications and Cortex-A processors for running full-fledged OSes like Linux.

For this guide, we use the [STM32MP157D-DK1](https://www.st.com/resource/en/data_brief/stm32mp157d-dk1.pdf), which features a Cortex-A7 core, TrustZone support, and good documentation.

## OP-TEE (Open Trusted Execution Environment)

Even with a board that supports TrustZone, you still need the right software tools to build programs that can leverage TrustZone's security features. This is where [OP-TEE](https://optee.readthedocs.io/) comes in. It is an open-source framework for building applications secured with TrustZone.[^1]

A simple analogy: OP-TEE is to TrustZone what the Intel SGX SDK is to SGX. Similar to SGX, a TrustZone-based application running in OP-TEE has two parts: a *client application* (CA) which is the untrusted part executing in the normal world, and a *trusted application* (TA) which is the trusted part executing in the secure world.

> The official OP-TEE documentation defines it as "a Trusted Execution Environment (TEE) designed as companion to a non-secure Linux kernel running on Arm Cortex-A cores using the TrustZone technology". Personally, I think this definition can confuse a beginner who already considers TrustZone to be the TEE. I refer to OP-TEE as a framework for building TZ applications.

As shown below, OP-TEE comprises two main components: `optee-os` on the trusted side (secure world) and `optee_client` on the untrusted side (normal world). The secure monitor bridges both components.

1. `optee-os`: a TEE OS executing at ARMv8 secure EL1. It provides generic OS-level functions such as interrupt handling, thread handling, crypto services, and shared memory. It implements the [GlobalPlatform TEE Internal Core API](https://globalplatform.org/wp-content/uploads/2021/03/GPD_TEE_Internal_Core_API_Specification_v1.3.1_PublicRelease_CC.pdf), used to implement TAs that run in the secure world at ARMv8 secure EL0.
2. `optee-client`: a normal-world user-space library and a normal-world user-space daemon. The library, `libteec.so`, implements the [GlobalPlatform TEE Client API](https://globalplatform.org/wp-content/uploads/2010/07/TEE_Client_API_Specification-V1.0.pdf), through which normal-world CAs interact with TAs. The daemon, TEE-supplicant, provides auxiliary functionality for the trusted OS, such as loading TAs from the normal-world file system into the secure world.

![OP-TEE architecture](assets/imgs/tees/trustzone/optee-arch.png)

### Chain of trust

In Arm TrustZone and OP-TEE, a [chain of trust](https://developer.arm.com/documentation/102418/0102/Software-architecture/Boot-and-the-chain-of-trust) ensures that each stage of boot is authenticated. It starts from the BootROM, which is the *root of trust* (RoT). This RoT authenticates the first-stage bootloader, which in turn authenticates the second-stage bootloader, and so on. Processes like secure boot define and implement this chain of trust.

## Building OP-TEE for the STM32MP157D-DK1

The following is based on the [official OP-TEE STM32MP1 guide](https://optee.readthedocs.io/en/latest/building/devices/stm32mp1.html). We will build OP-TEE for the STM32MP157D-DK1 and flash it on the board. These steps are done on your work/host PC, not on the board.

1. **Install prerequisites** (Ubuntu 22.04):

```bash
export DEBIAN_FRONTEND=noninteractive
export FORCE_UNSAFE_CONFIGURE=1
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y \
    adb acpica-tools autoconf automake bc bison build-essential ccache \
    cpio cscope curl device-tree-compiler e2tools expect fastboot flex \
    ftp-upload gdisk git libgnutls28-dev libattr1-dev libcap-ng-dev \
    libfdt-dev libftdi-dev libglib2.0-dev libgmp3-dev libhidapi-dev \
    libmpc-dev libncurses5-dev libpixman-1-dev libslirp-dev libssl-dev \
    libtool libusb-1.0-0-dev make mtools netcat ninja-build \
    python3-cryptography python3-pip python3-pyelftools python3-serial \
    python3-tomli python-is-python3 rsync swig unzip uuid-dev wget \
    xdg-utils xsltproc xterm xz-utils zlib1g-dev
```

See also the [official prerequisites](https://optee.readthedocs.io/en/latest/building/prerequisites.html#prerequisites).

2. **Install the Android `repo` tool**. `repo` is a Python wrapper from Google to manage multiple Git repositories at once. We need it for building OP-TEE:

```bash
sudo curl https://storage.googleapis.com/git-repo-downloads/repo -o /usr/local/bin/repo
sudo chmod a+x /usr/local/bin/repo
```

3. **Get OP-TEE for STM32MP1**. Check the [manifest XML](https://optee.readthedocs.io/en/latest/building/gits/build.html#current-version) corresponding to the board. Ours is `stm32mp1.xml`:

```bash
mkdir -p optee-stm32mp1 && cd optee-stm32mp1
repo init -u https://github.com/OP-TEE/manifest.git -m stm32mp1.xml
repo sync
```

4. **Build the toolchains**:

```bash
cd optee-stm32mp1/build
make -j2 toolchains
```

5. **Build the solution**. This compiles OP-TEE OS, the Linux kernel, Trusted Firmware-A, U-Boot, xtest, the root filesystem, and related components into a bootable stack.

> From the OP-TEE website, the `PLATFORM` option for `STM32MP1-57D-DK1` should be `stm32mp1-157A_DK1`. In practice, the OP-TEE driver was not configured correctly with that value and OP-TEE did not work after boot. `PLATFORM=stm32mp1-157C_DK2_SCMI` (or `stm32mp1-157C_DK2`) booted correctly and ran OP-TEE tests. It works, but you may hit issues later because the 157C-DK2 board configuration is not necessarily the same as the 157D-DK1. I opened an issue on the OP-TEE GitHub repo and hope this gets a cleaner fix.
{: .prompt-warning }

```bash
# make PLATFORM=stm32mp1-157A_DK1 all
make PLATFORM=stm32mp1-157C_DK2 all
# or multi-threaded: make -j`nproc` PLATFORM=stm32mp1-157C_DK2 all
```

This step takes some time. If you encounter build issues, pipe the build to a log file and check for errors. In that case, avoid the `-j` flag so the log is readable.

When the build completes, it generates `sdcard.img` in `../out/bin/` relative to the build root. The image is a GPT multi-partition image you can copy to the target SD card.

6. **Copy the image to an SD card**. Insert the SD card into your work PC, find its path with `lsblk`, then:

```bash
sudo dd if=../out/bin/sdcard.img of=/dev/sdX conv=fdatasync status=progress
sudo sgdisk -e /dev/sdX
```

`sgdisk -e /dev/sdX` converts the partition table from MBR to GPT while attempting to preserve existing partition data. Once the copy is complete, insert the SD card into the board.

7. **Access the serial console and boot the board**. On the STM32MP157D-DK1, the serial console is mapped to UART4 by default. UART4 is connected to the ST-LINK debugger chip, which handles serial-to-USB translation. Connect a USB micro cable from your PC to the `ST-LINK CN11` port and run:

```bash
dmesg | tail
```

You should see something like:

```text
usb 1-2: Product: STM32 STLink
cdc_acm 1-2:1.1: ttyACM0: USB ACM device
```

`ttyACM0` means the ST-LINK is at `/dev/ttyACM0`. Access the serial console with:

```bash
sudo apt install -y picocom
sudo usermod -a -G dialout $USER

# Enter the following in a new terminal
picocom -b 115200 /dev/ttyACM0
```

After that last command, power the board. You should see the boot process and a login prompt:

```text
OP-TEE embedded distrib for stm32mp1-157C_DK2
buildroot login:
```

This means you have successfully booted Linux. Enter `root` and press Enter. If you `cd /`, you can see the minimal Linux userland. Now we can test OP-TEE.

## Testing OP-TEE

In Buildroot, OP-TEE consists of: `TEE supplicant` running in the normal world, `libteec.so` (client library), and `xtest` (test TAs). Confirm these are present:

```bash
which tee-supplicant  # my result: /usr/sbin/tee-supplicant
which xtest           # my result: /usr/bin/xtest
```

> Some tutorials tell you to run `tee-supplicant &` at this step. In our case the daemon is already loaded at boot (check with `ps -elf`), so re-running it may error.
{: .prompt-info }

Run [OP-TEE xtest](https://optee.readthedocs.io/en/latest/building/gits/optee_test.html), a TEE sanity-test suite:

```bash
xtest
```

A successful `xtest` run means you are ready to build your own OP-TEE trusted applications.

> Building a first custom TA is still on my list. In the meantime, see [How to develop an OP-TEE Trusted Application](https://wiki.st.com/stm32mpu/wiki/How_to_develop_an_OP-TEE_Trusted_Application).
{: .prompt-info }

## Troubleshooting

- I opened a GitHub issue [here](https://github.com/OP-TEE/optee_os/issues/7521) on problems I hit while testing OP-TEE on this board.
- [STM32 MPU OP-TEE configuration switches](https://wiki.st.com/stm32mpu/wiki/OP-TEE_configuration_switches)
- [STM32 MPU How to build OP-TEE components](https://wiki.st.com/stm32mpu/wiki/How_to_build_OP-TEE_components)

## Other platforms with good TrustZone and OP-TEE support

1. [Nvidia Jetson boards](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/SD/Security/OpTee.html)
2. [Platforms listed by OP-TEE](https://optee.readthedocs.io/en/latest/general/platforms.html)
3. [OP-TEE on i.MX processors](https://timesys.com/webinars/Secure-by-Design-NXP-Webinar-Series-OP-TEE.pdf)

## More TrustZone documentation and publications

1. [Demystifying Arm TrustZone: A Comprehensive Survey](https://www.dpss.inesc-id.pt/~nsantos/papers/pinto_acsur19.pdf)
2. [TrustZone Explained: Architectural Features and Use Cases](https://www.researchgate.net/profile/Bernard-Ngabonziza-2/publication/312182612_TrustZone_Explained_Architectural_Features_and_Use_Cases/links/59f26a8a0f7e9beabfcc636b/TrustZone-Explained-Architectural-Features-and-Use-Cases.pdf)
3. [Arm TrustZone and OP-TEE](https://www.linkedin.com/pulse/arm-trustzone-unlocking-secure-world-embedded-systems-khaled-el-sayed-a3hpf/)
4. [OP-TEE on Nvidia Jetson](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/SD/Security/OpTee.html)
5. [STM32 MPU OP-TEE concepts overview](https://wiki.st.com/stm32mpu/wiki/OP-TEE_concepts_overview)
6. [STM32MP157D-DK1 databrief](https://www.st.com/resource/en/data_brief/stm32mp157d-dk1.pdf)
7. [STM32MP157x-DKx hardware description](https://wiki.st.com/stm32mpu/wiki/STM32MP157x-DKx_-_hardware_description)

[^1]: Though OP-TEE was initially created for Arm TrustZone, it has been structured to be compatible with other isolation technologies.
