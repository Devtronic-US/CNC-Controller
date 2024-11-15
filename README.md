# Devtronic CNC Controller

Why one more controller needed? All controllers has connectors all over the PCB. As result, to make accurate wiring it require a lot of space. If we look on stepper motor drivers, all of them has connectors only on one side which allow to stack them as "sandwich". Why not design CNC controller like that? It exactly what Devtronic CNC Controller is:

![Image](Media/Devtronic_Controller_1.png "Devtronic CNC Controller V2")

This board is everything you need to build a high functioning CNC machine.

**Store:** https://devtronic.square.site/

## The key features of the Devtronic CNC controller

* Support [grblHAL](https://github.com/grblHAL)
* 5 Axis of step/direction motor control: 5V, external stepper driver required(closed-loop recommended)
* Optocouplers used on all inputs
* Onboard 32KB FRAM to store setting
* Input voltage up to 48V which allow to power CNC controller directly from power supply for NEMA23/24 stepper motors
* UART for control CNC machine with another controller(SmartPendant recommended)

**Note:** [Devtronic CNC Controller V1](CNC-Controller-V1/README.md) is obsolette. 

## Firmware

[grblHAL Web Builder](http://svn.io-engineering.com:8080/)

Configuration:

* Driver: **STM32F4xx**
* Board:  **Devtronic CNC Controller V2 (BlackPill F411)**
* Plugins -> MPG & DRO mode:  **Real time command switchover**
* Plugins -> Settings EEPROM: **24x256(32k)**
* Plugins -> EEPROM is FRAM:  **Should be set** 

To load new firmware [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) is used.
Connect CNC controller to PC using USB-C cable. Press and hold BOOT0 button, then short press NRST button, couple seconds later BOOT0 button can be released.
Open STM32CubeProgrammer. In top right corner choose "USB" from drop down list.
If field "Port" in "USB Configuration" show "No DFU detected" click update button near it.
Click "Connect" button - STM32CubeProgrammer should establish connection and show current device memory content.
Click "Open File" in left to corner, select firmware HEX file, then click "Download" button in top left corner.
When flashing is done, close STM32CubeProgrammer and short press NRST button on the Controller to restart it. 

**Note:** if flashing is successful, but firmware isn't work, check STM32CubeProgrammer log. If it says "sector 0000 does not exist" erasing operation wasn't successful and new firmware "merged"(write operation can only flip bits from 1 to 0) with old one. To fix this issue, press "eraser" button in bottom right corner and click "Ok" button in pop up window. After erasing is finished "Device memory" should show only FFFFFFFF. Flash new firmware after that. 

## Dimensions

**120** mm x **66** mm x **24** mm
Approx. 4-3/4" x 2-5/8" x 1"

## Pinout

![Image](Media/Devtronic_Controller_2.png "Devtronic CNC Controller")

### Connectors on bottom row

* Power connector: configurable between 7-12V and 12-48V by jumper (-, +)
* X motor (Direction-, Direction+, Step-, Step+)
* Y motor (Direction-, Direction+, Step-, Step+)
* Z motor (Direction-, Direction+, Step-, Step+)
* Motor Enable output (Ground, Signal, Voltage)
* Spindle control (Ground, Enable, Direction, Speed, Voltage)
* Flood Coolant output (Ground, Signal, Voltage)
* Encoder input (Ground, A, B, Z, Voltage)
* Probe input (Ground, Signal, Voltage)
* UART (Ground, RX, TX, +5V)
* Fault input (Ground, Signal, Voltage)

###  Connectors on top row

* B motor (Direction-, Direction+, Step-, Step+)
* A motor (Direction-, Direction+, Step-, Step+)
* Flood Mist output(Ground, Signal, Voltage)
* X Limit input (Ground, Signal, Voltage)
* Y Limit input (Ground, Signal, Voltage)
* Z Limit input (Ground, Signal, Voltage)
* A Limit input (Ground, Signal, Voltage)
* Door input (Ground, Signal, Voltage)
* USB

## Inputs

Optocouplers used on all inputs. Voltage used to light up LED inside optocouplers depends on output voltage configuration. If +12V is selected, optocouples will be connected to +12V and current trough LED will be about 20mA. In case of 5V current will be about 7 mA.

Because of schematic, only **NPN sensors is recommended**. Sensors can have internal pull-up resistor.
If mechanical switch used as endstop it should connect input terminal to GND, not to +V.

## Configuration

![Image](Media/Devtronic_Controller_3.png "Devtronic CNC Controller")

There three configuration blocks of jumpers.

### Power control

This controller can be powered by two ways:

1. +7-12V applied to PWR connector: Spindle analog control(0-10V) will work only if +12V applied.
2. +12-48V applied to PWR connector: Input voltage reduced to +12V via integrated DC-DC Step-Down converter.

**Note:** when USB cable connected to controller, it powers BlackPill only, it is possible to communicate with controller from PC, but any functions requaired more than +3.3V will be unavailable.

There two jumpers to control power:

* **VIN**: input voltage selector, two jumpers, should be switched as pair. PWR for input voltage +7-12V and REG for input voltage +12-48V. One jumper does actual selection, another jumper disconnects power from DC-DC converter in case if PWR used directly. Default is REG.
* **VOUT**: output voltage selector. Output voltage applied to all screw terminal marked as +V and optocouplers. It may be +12V voltage or +5V from linear regulator. UART connector always has +5V regardless of this jumper position. Default is +5V.

Note: default position when delivered to prevent damage in case of power up without checking settings. Those jumpers should be configured for particular build.

### Output control

Some outputs can be configured between Push-Pull(PP) or Open Collector(OC):

* **EN**: enable signal for steppers, default is PP.
* **COOLANT MIST**: default is OC.
* **COOLANT FLOOD**: default is OC.
* **SPINDLE EN**: spindle enable signal, default is PP.
* **SPINDLE DIR**: spindle direction signal, default is PP.
* **SPINDLE PWM**: spindle PWM signal, default is PP.
* **SPINDLE SPD**: spindle speed control method. Can be set between PWM or 0-10V. Default is 0-10V.
 
Push-Pull mean that output pin will have either GND(inactive) or +5V(active) generated by buffer IC. This option sutible for external controllers.
Open Collector mean that output pin will gave either Hi-Z state(inactive) or GND(active) via darlington array IC. This option suitable for relays.

### 5th Axis/Debug control

Most users don't need to touch this jumpers. If you plan to build and debug firmware yourself, then you need Serial Wire Debugger(SWD). In this case flip those jumpers into SWD position to pass SWCLK and SWDIO signals directly to BD+ and BS+. Then you can connect debug probe to those terminals. Connect debug probe GND signal to BD- or BS-.

## Schematic

![Image](Media/GRBL_Controller.png "Devtronic CNC Controller")
![Image](Media/GRBL_Contrller_Inputs.png "Devtronic CNC Controller")
![Image](Media/GRBL_Controller_Extension.png "Devtronic CNC Controller")
