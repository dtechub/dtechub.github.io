---
title: Home automation with Raspberry Pi - Part 1
author: petman
date: 2024-08-11 09:10:00 +0800
categories: [Raspberry Pi, Home automation, Electronics, DIY]
tags: [raspberry pi, home auto, diy, electronics]
render_with_liquid: false
---

This series dives into home automation with Raspberry Pi for DIY enthusiasts and electronics hobbyists. It is subdivided into 5 parts:
1. [Introduction to the Raspberry Pi](https://dtechub.github.io/posts/home-auto-rpi-1)
2. [Controlling Raspberry Pi GPIO](https://dtechub.github.io/posts/home-auto-rpi-2/)
3. Sensors: temperature and humidity, passive infrared, ultrasonic 
3. Automating lights
4. Controlling Tapo cameras
5. Building and integrating an AI voice assistant
6. Personalized web control interface with Flask
7. Building personalized PCB boards


## Introduction to the Raspberry Pi
- Raspberry Pi is a series of small single-board credit card-sized computers developed by the [Raspberry Pi Foundation](https://www.raspberrypi.org/about/). Raspberry Pi boards run on Linux OS, and have many interesting applications comprising simple learning projects to complex industrial automation and robotics, and is a common platform for electronics hobbyists and engineers. In this series, we will build a home automation platform based on Raspberry Pi.
- Raspberry Pi components: A Raspberry Pi board features a CPU, RAM, USB ports, WiFi and Bluetooth (depending on the model), as well as on-chip SPI, I2C, I2S, and UART peripherals, which are used to integrate different sensors and modules.
Raspberry Pis come in different models: model 3B, 4B, etc. In this series, we use the Raspberry Pi 3B. Nevertheless, things should work (with minor changes) on a more recent models like model 4B.
The following image highlights some of the hardware components that comprise a Raspberry Pi 3B board. ![Pi 3B](assets/imgs/rpi/rpi-3b.png)


## Setting up the Raspberry Pi
- All the development in this series is done principally on a Linux based OS, e.g., Ubuntu. However, tips will be given for Windows based users. In this series, the words "Raspberry Pi" and "Pi" will be used interchangeably to describe a single Raspberry Pi based board.
We recommend installing [Visual Studio Code](https://code.visualstudio.com/) as the main IDE.
- The first step involves flashing a Raspberry Pi operating system (OS) image. Tools like [BalenaEtcher](https://etcher.balena.io/) and [Raspberry Pi Imager](https://downloads.raspberrypi.org/imager) are popular for this purpose. 
We use the latter in this series. 
Download a compatible Raspberry Pi OS image for your board here: [Raspberry Pi OS Downloads](https://www.raspberrypi.com/software/operating-systems/). Pi imager also provides the possibility to download and flash a Pi OS directly.
The `lite` versions do not come with an embedded GUI and have less software included (`500MB` vs `2.7GB`).
We will use the lite version `Raspberry Pi OS Lite` (July 4th 2024 in my case), and connect to our Pi over the network via SSH (since no lite versions have no GUI by default).
- Download Pi imager: `sudo snap install rpi-imager`. For windows, download here: [Pi imager](https://downloads.raspberrypi.org/imager/imager_latest.exe)
- Launch Pi imager and follow the instructions to setup your Pi's: SSH username (e.g., `pi`), network hostname (e.g., `pi-homeauto`), password (use a strong but easy to remember password), and WiFi credentials.
- If need be, use `Ctrl + Shift + X` to open the SSH and Wifi configuration prompt in Pi imager.
- Flash the Pi OS on the SD card. After flashing, nmount and remove the microSD card from your PC, insert it into your Pi board and power up the board insert SD card into your PI and power up it up. Raspberry Pi OS will use the above configurations at boot time to automatically configure WiFi and SSH access to your Pi. A green LED flashing means the Pi has completed booting and is ready to be accessed over SSH.

### Getting your Pi IP address
- There are multiple ways to obtain your Pi's IP address: using tools like `nmap` or `arp-scan`. We use `nmap`.
Start by installing `nmap` and `net-tools`, and then run `ifconfig | grep netmask` in the terminal to obtain your subnet mask IP.
```bash
sudo apt install nmap net-tools
ifconfig | grep netmask
```
- 
Example ifconfig results: `inet 192.168.1.42  netmask 255.255.255.0` means the subnet mask IP is 192.168.1.0 and subnet mask /24. That is the first 24 bits (8x3) are masked.
For netmask 255.255.0.0, the subnet mask will be /16.
Run nmap with your subnet mask to scan your subnet and obtain your Pi's IP address.
```bash
sudo nmap -sP 192.168.1.0/24
```
- The results of an nmap scan on my local network are shown below.
![nmap scan](assets/imgs/rpi/nmap-scan.png)

The scan reveals my Raspberry Pi's IP address is `192.168.1.123`. Note this address down as it will be used to connect via SSH.
> The above image also reveals the MAC addresses of different devices on my home network. A MAC address uniquely identifies devices on a local network. While it's generally not risky to publicize your MAC address, I'd recommend you keep yours private if you are very cautious about security.
{: .prompt-info }

### Connecting to your Pi
Run the command ssh `pi@pi-IP-address` or `ssh pi@pi-homeauto`. For example:
```bash
ssh pi@192.168.1.123
```
`Are you sure you want to continue connecting (yes/no)?`: enter yes. `pi@130.125.11.172's password`. Enter the password configured during the setup. The default Pi password is `raspberry`.
You should now have a remote connection to your Pi.
- Update your source lists and install useful packages
```bash
sudo apt update && sudo apt install git -y
```

### Remote development with VS Code
- I'm a fan of VS Code and use it as my main IDE. You can configure VS Code to access folders and files remotely on your Pi. By doing this you can easily write scripts and test on your Pi board remotely.
- To connect to your Pi via VS Code, type `Ctrl + Shift + P`, choose `Remote-SSH: Connect Current Window to Host ...`, and type `ssh pi@192.168.1.123` and your password to connect to your Pi.
- For a more complete tutorial on remote development with SSH on VS Code, see [this tutorial](https://code.visualstudio.com/docs/remote/ssh).
- If you are more "geeky", you can use tools like `Emacs` or `Vim` to create and edit files directly via the terminal. 
- In [part 2](https://dtechub.github.io/posts/home-auto-rpi-2/) of this series, we will write simple Python programs to control GPIO pins.