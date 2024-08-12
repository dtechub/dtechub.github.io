---
title: Home automation with Raspberry Pi - Part 2
author: petman
date: 2024-08-11 10:00:00 +0800
categories: [Raspberry Pi, Home automation, Electronics, DIY]
tags: [raspberry pi, home auto, diy, electronics]
render_with_liquid: false
---

## Raspberry Pi GPIO
- One very powerful feature of the Raspberry Pi is the set of General-Purpose Input/Output (GPIO) pins, which act as an interface between the Pi and the outside world. These allow sensors and actuators to be connected and controlled by the Pi.
- The Pi 3B has `40` GPI pins as shown in the following picture.
![Pi 3B pinout](assets/imgs/rpi/pi-3b-pinout.jpg)
- To identify the GPIO pins, run the `pinout` command inside your Pi's terminal.

> The GPIO pins operate on 3.3V logic. As such no sensors that operate at 5V logic should be used.
{: .prompt-warning }

## GPIO programming
- The Pi GPIO can be programmed with a high-level language like Python or low-level language like C or C++.  Though C provides much faster performance, Python has more support compared to C: ready-made data structures, algorithms, and modules/libraries, etc. which can easily be used as building blocks. For this reason we stick to Python for GPIO programming. In addition, using Python makes its easier to integrate Web frameworks like Flask when building a control dashboard.
- As an example, we will write a simple program to blink an LED. Create a folder `homeauto` and a Python script `piauto.py` in that directory.
```bash
mkdir homeauto
touch piauto.py
```
- Open this folder in VS Code SSH allowing to edit the Python script.
- Create a virtual environment and install useful Python modules. As seen in previous posts, creating a Python virtual environment isolates our project's dependencies from other system dependencies.
```bash
sudo apt install python3-virtualenv          # install virtualenv package systemwide
virtualenv homeautoenv                       # creates virtual environment
source homeautoenv/bin/activate              # activate virtual environment
pip3 install RPi.GPIO                        # install RPi.GPIO module in virtual environment
```
- In your Python script, import the `RPI.GPIO` module used to control GPIO pins.
```python
import RPi.GPIO as GPIO
from time import sleep 
```
- Declare the pin scheme: this defines how the GPIO pins are numbered. There are two modes: `BCM mode` which refers to the pins by the Broadcom SoC Channel number. In simple language, this means the pin numbering is exactly as shown in the output of the pinout command (`GPIOxx`). Thee second mode: `BOARD mode` means the pin numbering follows the numbering on the plug, i.e., the numbers printed on the board, e.g,. P1 etc. We use the BCM mode.

```python
GPIO.setmode(GPIO.BCM)
```
- Declaring the pin mode: this is similar to pin mode declarations when programming micro-controllers like the Arduino. Each GPIO pin can be either input or output. If you wish to control an LED for example, set the pin mode to `OUTPUT`. On the other hand, if the pin is to be used as an input pin (e.g., for a temperature sensor) set it as an `INPUT pin`. We will use pin GPIO14 (see pinout diagram) as an output pin to control our LED. NB: there is nothing special about GPIO pin 14, you can pick any other GPIO pin. We chose pin 14 because it is next to a ground pin (GND) which we need to complete the circuit. 
- Connect the LED to this pin using a bread-board. The anode (longer leg: +) of the LED should be connected to a resistor (e.g., 220 Ohm) to reduce the current flowing into the LED. The cathode (shorter leg: -) should be connected to a ground pin (GND) on the Pi board.

```python
ledPin = 14
GPIO.setup(ledPin,GPIO.OUT)
```
- GPIO states: you can set the state of a GPIO output pin high (3.3V) or low (0V). Use `GPIO.output(pin,GPIO.HIGH)` or `GPIO.output(pin,GPIO.LOW)` to set it high or low respectively.
To read the status of a GPIO pin, use `digitalRead(pin)`. You can use software pull-up or pull-down functions to ensure a well-defined state on a pin.
To add delays use `time.sleep(delay-in-seconds)`, e.g., `time.sleep(0.5)`. You can use PCM to dim LEDs.
For analog input, you will need an analog to digital converter (ADC). Some examples of ADC chips: MCP3008, ADS1x15
- To let the LED blink, we create a loop in which we set the LED pin to HIGH and LOW consecutively.

```python
while True:                          # Endless Loop
    GPIO.output(ledPin, GPIO.HIGH)   # Turn on
    print("LED on")                  # Prints state to console
    sleep(1)                         # Pause for 1 second
    GPIO.output(ledPin, GPIO.LOW)    # Turn off
    print("LED off")                 # Prints state to console
    sleep(1)                         # Pause for 1 second
```
- Execute the script with `python3 piauto.py`