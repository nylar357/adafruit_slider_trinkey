Slider Trinkey Brightness Controller for Linux
This repository contains the necessary scripts to use an Adafruit Slider Trinkey as a hardware monitor brightness controller on a Linux machine. It consists of two main parts:

CircuitPython Script: This code runs on the Slider Trinkey itself. It reads the position of the potentiometer slider and prints the value to the serial port.

Host Python Script: This script runs on your Linux PC. It listens to the serial output from the Slider Trinkey and uses the brightnessctl command-line utility to adjust your monitor's brightness.

💻 Setup and Usage
Step 1: Prepare the Adafruit Slider Trinkey
First, you need to ensure the Slider Trinkey is running the provided CircuitPython code.

Install CircuitPython: Follow the official Adafruit guide to get CircuitPython running on your device.

Upload trinkeycode.py: Copy the trinkeycode.py script onto your Slider Trinkey's CIRCUITPY drive. This script reads the potentiometer's value and prints it to the serial console.

Step 2: Prepare Your Linux Machine
Next, you'll set up your Linux machine to read from the Slider Trinkey and control the brightness.

Install Dependencies: You'll need python3, pip, and the brightnessctl utility. On Debian/Ubuntu-based systems, you can install them with the following command:

sudo apt install python3 python3-pip brightnessctl

Install Python Libraries: Install the necessary Python libraries for serial communication.

pip3 install pyserial

Download the Host Script: Save the ctlbright.py script to your computer.

Run the Script: Connect your Slider Trinkey to a USB port. Run the host script with sudo to ensure it has the necessary permissions to control brightness.

sudo python3 ctlbright.py

The script will print the current brightness and then automatically adjust the brightness as you move the slider on the trinkey.

🎬 Demonstration
A video demonstrating the functionality of the Slider Trinkey brightness controller can be added here.

📄 Code Reference
trinkeycode.py (CircuitPython for Slider Trinkey)
This script maps the analog input from the potentiometer to a 0-100 value and prints it to the serial port.

# SPDX-FileCopyrightText: 2021 Kattni Rembor for Adafruit Industries
#
# SPDX-License-Identifier: MIT

import time
import board
from analogio import AnalogIn
import adafruit_simplemath

analog_in = AnalogIn(board.POTENTIOMETER)

def read_pot(samples, min_val, max_val):
    sum_samples = 0
    for _ in range(samples):
        sum_samples += analog_in.value
    sum_samples /= samples

    return adafruit_simplemath.map_range(sum_samples, 100, 65535, min_val, max_val)

while True:
    print("Slider:", round(read_pot(10, 0, 100)))
    time.sleep(0.1)

ctlbright.py (Python for Linux Host)
This script listens for the serial output and uses the brightnessctl command to control the display brightness.

import sys
import serial
from serial.tools import list_ports
import subprocess
import time

def get_current_brightness():
    """
    Gets the current monitor brightness using brightnessctl.
    Returns the brightness value as an integer.
    """
    try:
        # Use subprocess to run the brightnessctl command to get the current brightness
        result = subprocess.run(
            ['brightnessctl', 'get'],
            capture_output=True,
            text=True,
            check=True
        )
        return int(result.stdout.strip())
    except (subprocess.CalledProcessError, ValueError) as e:
        print(f"Error getting brightness: {e}")
        return 0

def set_monitor_brightness(value):
    """
    Sets the monitor brightness using brightnessctl.
    The value is a percentage from 0 to 100.
    """
    try:
        # Use subprocess to run the brightnessctl command to set the brightness
        subprocess.run(
            ['sudo', 'brightnessctl', 'set', f'{value}%'],
            check=True
        )
    except subprocess.CalledProcessError as e:
        print(f"Error setting brightness: {e}")

# Find the Adafruit Slider Trinkey port
slider_trinkey_port = None
ports = list_ports.comports(include_links=False)
for p in ports:
    if p.pid is not None:
        print("Port:", p.device, "-", hex(p.pid), end="\t")
        # The PID for the Slider Trinkey is 0x8102
        if p.pid == 0x8102:
            slider_trinkey_port = p
            print("Found Slider Trinkey!")
            trinkey = serial.Serial(p.device)
            break
else:
    print("Did not find Slider Trinkey port :(")
    sys.exit()

# Get the initial brightness and a brightness value to start with
curr_brightness = get_current_brightness()
last_brightness = curr_brightness

print(f"Initial brightness: {curr_brightness}%")

# Main loop to read from the Slider Trinkey and adjust brightness
while True:
    try:
        x = trinkey.readline().decode('utf-8')
        if not x.startswith("Slider: "):
            continue

        # Extract the value from the serial output and convert it to a percentage
        val = int(float(x.split(": ")[1]))
        
        # Check if the value has changed significantly before updating
        # This prevents the script from constantly trying to update the brightness
        # which can cause issues or lag.
        if abs(val - last_brightness) > 1:
            print("Setting brightness to:", val)
            set_monitor_brightness(val)
            last_brightness = val
            time.sleep(0.1) # Small delay to avoid hammering the brightnessctl command

    except serial.SerialException as e:
        print(f"Serial communication error: {e}")
        break
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        break

print("Exiting script.")
