# Making Stuff
CHIP is more than a cool, small, inexpensive computer. It's a complete system for building projects that require remote control, network connectivity, and physical interfacing with people and the environment.
CHIP's pin headers have all the connections to make this happen. An annotated diagram of the pin headers can be found in the [hardware section](#pin-headers) of this manual.

## GPIO
GPIO provides basic digital connections to the physical world to create physical products with CHIP. These pins can act as 'reads' or 'writes', for example, to sense switch positions or turn an LED on or off. 

CHIP's most easily available IO pins are the "XIO" pins on header U14. This is the "GPIO eXpander" that uses an I2C bus to create eight (8) convenient pins for GPIO. These use address `0x38` on the TWI bus 2. Other pins are available for GPIO if more than eight are needed.

### Read and Write From Command Line
CHIP has several General Purpose Input/Output (GPIO) pins available for you to build around. If you want to access them in a very primitive way, just to confirm their existence, here's some things to try.

### Requirements
  * CHIP
  * Jumper Wire
  * LED
  * SSH or serial connection to CHIP or
  * Monitor and keyboard

### How You See GPIO
There are eight (8) GPIO pins always available for connecting CHIP to the sense-able world. If you orient CHIP with the USB connector pointed up, you'll find the GPIO pins in the middle of the right header, U14, Pins 13-20, labeled XIO-P0 to P7: 

![Pinout diagram for CHIP](images/chip_pinouts.jpg)

### Kernel 4.3 vs 4.4 GPIO - How To Tell The Difference

For various reasons related to the community nature of Linux development, the GPIO expander pin numbers are different between CHIP OS kernels 4.3 and 4.4. What follows is a very technical discussion of the GPIO access. If you just want to start making stuff and don't need low-level information, you might just want to skip this section and go straight to the [python library](#python-library).

If you are developing applications on CHIP that use GPIO pins and you would like consistent behavior between the two kernel versions, you need to know how to find out the base value for the GPIO values. It may be enough for you to know that the GPIO expander pins start at *408* on 4.3 and *1016* on 4.4, however, it would be ideal to calculate this in your application to truly future-proof for future kernels.

If you look in the directory `/sys/class/gpio`, you'll find two directories starting with `gpio`: `gpiochip0` and either `gpiochip408` (4.3) or `gpiochip1016` (4.4).

The `408` and `1016` are the bases for the expander pins. If you want to definitively find out what the base is using code, you should 

```shell
cat gpiochip*/label
cat gpiochip*/ngpio
cat gpiochip*/base
```

The `label` you are interested in is the value `pcf8574a` which is the device that provides GPIO expansion. This provides the number of GPIO as returned by `ngpio`. The first expander pin starts with the `base` value. If you parse all these values and apply to your code, you can setup your application to be kernel-agnostic for GPIO access. 

Here is a python script that demonstrates this [in a gist](https://gist.github.com/howientc/606545e0ff47e2cda61f14fca5c46eee)

Here is a bash script to compute the base (courtesy of [bbs user @fordsfords](https://bbs.nextthing.co/users?name=fordsfords))

```shell
LABEL_FILE=`grep -l pcf8574a /sys/class/gpio/*/*label` BASE_FILE=`dirname $LABEL_FILE`/base 
BASE=`cat $BASE_FILE`
```
 
### How The System Sees GPIO
There is a `sysfs` interface available for the GPIO. This just means you can access the GPIO states in a file-system-like manner. For example, you can reference XIO-P0 using this path:

```shell
  /sys/class/gpio/gpio408/
```

The number is somewhat unfortunate, since the `sysfs` names do not match the labels on our diagram! But is not too hard to translate. Pins XIO-P0 to P7 linearly map to `gpio408` to `gpio415` on kernel 4.3 and `gpio1016` to `gpio1023` on kernel 4.4. See [above](#kernel-4-3-vs-4-4-gpio-how-to-tell-the-difference) to learn more about that distinction.

### Some GPIO Switch Action
These lines of code will let us read values on pin XIO-P7. First, we tell the system we want to listen to this pin:

```shell
	#4.3
  sudo sh -c 'echo 415 > /sys/class/gpio/export'
  #4.4
  sudo sh -c 'echo 1023 > /sys/class/gpio/export'
```

View the mode of the pin. It should return “in”:

```shell
	#4.3
  cat /sys/class/gpio/gpio415/direction
  #4.4
  cat /sys/class/gpio/gpio1023/direction
```

Connect a jumper wire between Pin 20 (XIO-P7) and Pin 39 (GND). Now use this line of code to read the value:

```shell
	#4.3
  cat /sys/class/gpio/gpio415/value
	#4.4
  cat /sys/class/gpio/gpio1023/value
```

### Some GPIO Output
You could also change the mode of a pin from “in” to “out”

```shell
	#4.3
  sudo sh -c 'echo out > /sys/class/gpio/gpio415/direction'
  #4.4
  sudo sh -c 'echo out > /sys/class/gpio/gpio1023/direction'
```

Now that it's in output mode, you can write a value to the pin:

```shell
	#4.3
  sudo sh -c 'echo 1 > /sys/class/gpio/gpio415/value'
  #4.4
  sudo sh -c 'echo 1 > /sys/class/gpio/gpio1023/value'
```

If you attach an LED to the pin and ground, the LED will illuminate according to your control messages.

### Enough IO
When you are done experimenting, you can tell the system to stop listening to the gpio pin:

```shell
	#4.3
  sudo sh -c 'echo 415 > /sys/class/gpio/unexport'
	#4.4
  sudo sh -c 'echo 1023 > /sys/class/gpio/unexport'
```

### Learn More
You can learn more about GPIO and Linux [here:](https://www.kernel.org/doc/Documentation/gpio/sysfs.txt)

## Python Library
There is a well-maintained python library that works for 4.3 and 4.4 kernels available [here](https://github.com/xtacocorex/CHIP_IO). This is analogous to the RPi.GPIO library, but is designed for CHIP. It's an excellent place for quickly working with GPIO and PWM on CHIP.

## GPIO Types
There are many types of sensors that can be used with GPIO:

### Switches
Switches provide on/off state input from the physical world to your computer. You can [use the commandline interface](#some-gpio-switch-action) to listen to switch values. A python library was created for the [ChippyRuxpin project](https://github.com/NextThingCo/ChippyRuxpin) if you need a higher-level example in python. 

### LEDs
LEDs can be illuminated and turned off using the [commandline interface](#some-gpio-output). Refer to the [ChippyRuxpin project](https://github.com/NextThingCo/ChippyRuxpin) on a good example on how to manipulate the commandline using python.

### Relays
Relays are special hardware bridges used to switch higher voltage electronics, protecting CHIP from the high voltages that would destroy it.  Using a relay board is programmatically no different from using switches.

## Expanding GPIO
If you don't need to drive an LCD, you can use those pins for more, faster GPIO if you want to. 
These are the pins numbered 18-40 on U13 and 27-40 on U14 to act as GPIO to increase the number of available GPIO pins. 
Documentation on this process is forthcoming!

## Analog to Digital Conversion
Pin 9 on header U14 provides a link for low resolution analog to digital conversion (ADC). 
There is no driver for this link yet. ADC is used to read continuous sensors (temperature, pots, FSR, photoresistor, etc)

## 1 Wire
The 1 Wire serial protocol is not available in CHIP OS 4.3. In version 4.4 and above, you can access one wire data from the sysfs device. Find your one wire devices with

```shell
ls /sys/bus/w1/devices/2*/eeprom
```

The `*` is there because your eeprom device will register a unique UUID number with C.H.I.P., so the `ls` command will show you all available one wire devices.

## UART
UART connections can be made using the UART connections on header U14. 

## PWM
Pulse Width Modulation is used to control motors and other devices. 
It is possible to use GPIO pins to drive motors, but they generally are not fast enough for robust and smooth control.
PWM can be accessed through an `sysfs` protocol.

## I2C
I2C can be accessed through a `sysfs` protocol using the debian i2c-tools. In the terminal, use

```shell
sudo apt-get install i2c-tools
```

Note that the "XIO GPIO" pins are provided by an I2C expander at address 0x38 on the TWI bus 2, so that address cannot be used on bus 2.

## LCD Monitor Support
Using the numerous LCD header pins, a color touchscreen panel can be directly implemented on CHIP.

## Project Examples
Projects coming soon!
