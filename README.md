# Serialjoy

<img src="doc/serialjoy.svg" width="400">

## What is this?

Serialjoy is a tiny, generic and hobbyist-friendly adapter of joysticks/gamepads
on Linux using serial ports. That means you can program any sort of
microcontroller or similar devices, for example an Arduino, to interface with a
real game controller (even one that you made! That's why it's generic), then
send the data via a simple serial port and they will be able to emulate joystick
buttons, keyboard keys, analog axes & more on your computer. This way you can
cut a lot of time and complexity from using the USB protocol for such devices
(that's why it's hobbyist-friendly), besides being more flexible.

## How it works?

This project is composed of two main parts that interact: an *adapter* and a
*host*:

The *adapter* is the physical layer that interfaces with a game controller and
sends the data to a serial port, which can be a USB-Serial adapter or a real
RS232 port (with voltage levels corrected). It can be simple as an Arduino with
the on-board USB-Serial converter, although custom-designed boards will be
available.

The *host* is the software that your computer will be running when the *adapter*
is connected. It will create the required virtual joystick devices and quickly
translate the data received on the serial port to the device using `uinput`,
allowing you to generate real input events with your physical controller with
almost zero delay.

Although the project is composed of two parts, the name `serialjoy` will always
allude to the host software. Any adapter (board & firmware) that is made
available on a Github repo can be linked here, but currently (10/2019) there is
only support for SEGA Genesis controller.

### Some implementation details

The host talks to the adapter via a serial port (like `/dev/ttyUSB0` if you're
using an USB-Serial adapter) using only printable characters that represent the
state of a button (uppercase is pressed, lowercase is released) or some other
data (analog axes, create controller, etc). More details on the communication
protocol is described below and is subject to changes. No flow control is used,
only TX/RX pins.

## Contributing

Feel free to contribute in any way you can. This project is still young, so new
ideas are welcome (constructive criticism is also welcome). Writing
documentation is a must, as I haven't got the time to do that yet. If you want
to become an active developer or have some other question, please contact me at
my email:
[francosauvisky+serialjoy@gmail.com](mailto:francosauvisky+serialjoy@gmail.com).

Another way to contribute to this project is to donate/lend gamepads so I can
program/design the adapter for them. If you live nearby (Florianópolis, Santa
Catarina, Brazil), I can return them afterwards to you. Otherwise be welcome to
adapt them by yourself (and don't forget to share the code afterwards!).

## Build instructions

To build the host software, running the included `Makefile` on source folder
should be enough:

```
$ cd /src
$ make
```

To run the host software, your user must have read/write access to uinput device
(`/dev/uinput`).

## Communication Protocol

- Valid characters: Every printable character (from 0x20 up to 0x7E)

- Packets size: Strings of 1 to 5 chars/bytes.

### Available actions

- Simple actions "a":

a = [a..z]: Buttons press (up to 26 buttons per controller)

a = [A..Z]: Buttons release (up to 26 buttons per controller)

- Complex actions "axx":

a = [a..zA..Z]: Action ID (still to be defined), for example an analog axis

x = [0x40..0x5F]: Action data (less significant 5 bits of each char) = 10 bits

### Packet format

- From adapter to host:

1. "!n", where n = [0..9]: Create joystick device n

2. "^n", where n = [0..9]: Destroy joystick device n

3. "a", where a = [a..zA..Z]: Send simple action a to device 0 (LEGACY)

4. "na", where n = [0..9] and a = [a..zA..Z]: Send simple action a to device n

5. "n:axx" where n = [0..9], a = [a..zA..Z] and each x = [0x40..0x5F]: Send
complex action axx to device n

6. "s" where s is a null-terminated string: Preprogrammed string (only if host
asks)

- From host to adapter:

1. "d": Force adapter to (re)create connected devices

2. "v": Return preprogrammed string

- Handshake protocol:

1. "#": OK

2. "%": Not OK/cancel

3. "?": Are you there? (return OK)

When adapter sends "!n" (create device), host must answer OK when successful. If
adapter doesn't responds to "?", then go to legacy mode

## To do

- Create a wiki/add documentation (IMPORTANT!)

- Use a simpler received data <-> input action dictionary, not a switch
  statement within a function within a .c file (maybe with #define or an
  external configuration file)

- Automatic identification of the serial port

- Automatic service (daemon) which runs the program when an adapter is detected
  and starts at boot/user login.

- Change read-process-send loop to multiple simultaneous threads
