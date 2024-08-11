---
title: Home automation with Raspberry Pi - Part 1
author: petman
date: 2024-08-11 09:10:00 +0800
categories: [Raspberry Pi, Home automation, Electronics, DIY]
tags: [pi, home auto, diy]
render_with_liquid: false
---

This series dives into home automation with Raspberry Pi for DIY enthusiasts and electronics hobbyists. It is sudivided into 5 parts:
1. [Introduction to the Raspberry Pi](#introduction-to-the-raspberry-pi)
2. [Controlling Raspberry Pi GPIO](https://dtechub.github.io/posts/home-auto-rpi-2/)
3. Sensors: temperature and humidity, passive infrared, ultrasonic 
3. Automating lights
4. Controlling Tapo cameras
5. Building and integrating an AI voice assistant
6. Personalized web control interface


## Introduction to the Raspberry Pi

- We will use the Raspberry Pi 3B+.



## Setting up the Raspberry Pi
- All the development in this series is done principally on a Linux based OS, e.g., Ubuntu. However, tips will be given for Windows based users. Also, the words "Raspberry Pi" and "Pi" will be used interchangeably. 
We recommend installing [Visual Studio Code](https://code.visualstudio.com/) as the main IDE.
- The first step involves flashing a Raspberry Pi operating system (OS) image. Tools like [BalenaEtcher](https://etcher.balena.io/) and [Raspberry Pi Image]() are popular for this purpose. 
We use the latter in this series. 
Download a compatible Raspberry Pi OS image for your board here: [Raspberry Pi OS Downloads](https://www.raspberrypi.com/software/operating-systems/).
The `lite` versions do not come with an embedded GUI and have less software included (`364MB` vs `2.7GB`).
We will use the lite version: `Raspberry Pi OS Lite`; more specificallya: `2023-05-03-raspios-bullseye-armhf-lite.img.xz` for this series, and connect to our Pi over the network via SSH.
- Download Pi image: `sudo snap install rpi-imager`. For windows, download here: [Pi imager](https://downloads.raspberrypi.org/imager/imager_latest.exe)
- Launch Pi imager and follow the instructions to setup your Pi's: SSH username (e.g., `pi`), network hostname (e.g., `pi-homeauto`), password (use a strong but easy to remember password), and WiFi credentials.
- If need be, use `Ctrl + Shift + X` to open the SSH and Wifi configuration prompt in Pi imager.
- Flash the Pi OS on the SD card. After flashing, nmount and remove the microSD card from your PC, insert it into your Pi board and power up the board insert SD card into your PI and power up it up. Raspberry Pi OS will use the above configurations at boot time to automatically configure WiFi and SSH access to your Pi. A green LED flashing means the Pi has completed booting and is ready to be accessed over SSH.

### Getting your Pi IP address
There are multiple ways to obtain your Pi's IP address.
1. Using `arp-scan`: this is the quickest solution (IMO)
```bash
sudo apt install arp-scan
sudo arp-scan --localnet
```
A sample result of the scan. THe Pi IP here is `130.125.11.172`

2. Using `nmap`
TODO

### Connecting to your Pi
` Run the command ssh `pi@pi-IP-address` or `ssh pi@pi-homeauto`. For example:
```bash
ssh pi@130.125.11.172
```
`Are you sure you want to continue connecting (yes/no)?`: enter yes. `pi@130.125.11.172's password`: the default Pi password is `raspberry`.
You should now have a remote connection to your Pi.