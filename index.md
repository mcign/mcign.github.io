## MCIGN - The Keyless Motorcycle Ignition

### What is MCIGN?

The MCIGN project seamlessly adds keyless start to stock motorcycle ignitions and makes it nearly impossible to hotwire or force the ignition. All modifications are contained in the stock ignition housing, giving a stock appearance. The modified ignition communicates with a smartphone app, and an optional proximity mode can automatically turn the bike on, either in a trusted location such as a garage or wherever the bike was last parked. The lock mechanism isn't altered, so the original key still works. An optional "digital killswitch" module makes hotwiring nearly impossible by using encrypted commands to signal that the ignition has been switched on, instead of just checking the resistance between 2 wires like most stock motorcycles sold in the US.

The ignition firmware can store several digital keys (currently 15), and keys can be set to expire at a certain date, or limited with a curfew (so the key only works at certain hours or on certain days). This is a safer way to loan or rent a vehicle than giving out physical keys, which are easy to copy (and physical locks are hard to change/rekey).

Most motorcycle engines are not started with the key, but instead by a button on the right handlebar (for safety, all controls can be reached while the rider's hands are on the handlebars). This project is able to switch the bike "on" in the same way turning the key would, but it can not start the engine. Most motorcycles also have a steering lock in the ignition that locks the handlebars to the side, making theft more dificult. The steering lock is not altered or disabled by this project, but cannot be controlled remotely.

### Safety

Safety is a concern, as the device is capable of shutting off the motorcycle's engine. All components of the current version except for the Molex connector used in the dual-PCB configuration are AEC-qualified, and  A voltage sensor detects when the engine is running, and prevents the proximity feature from automatically shutting off the bike while the engine is running. Similarly, if the bike is unlocked with the proximity feature but loses connection with the smartphone for more than a preconfigured amount of time, it will automatically re-lock (assuming the engine is not running).

### Current State

A working prototype of the previous version, which used a different MCU and didn't work with the original key, has been built and tested on a motorcycle (Honda RC51). The smartphone app uses Ionic framework and Cordova, and currently supports Android with iOS support nearly done. Battery consumption is acceptable (uses ~15%/day on a Galaxy S9), and there are plans to further reduce battery consumption. An automotive-grade version of the hardware with support for the original key has been designed, but the firmware still needs to be ported to the new MCU (from BGM111 to nRF51824).

Development on the "digital killswitch" has not yet begun. It's planned to use a small MCU such as an ATTiny, and the same wire used by the ECU to sense if the ignition is switched on will be reused for a serial connection. A transistor will cause a short circuit between the ECU sense wire and ground until the device recieves an encrypted command from the ignition device.

### Open Source

This project is freely available, and design files (source code, schematic, layout) for the tested prototype (previous version) are available at the github repositories listed below. All files are released under the GPLv3, and contributions are appreciated. To contribute, or to make a comment or a feature request, please make a github account and [create an issue](https://help.github.com/en/articles/creating-an-issue) in the repective repository.

#### Project Repositories
 * [Smartphone App](https://github.com/mcign/app)
 * [Firmware](https://github.com/mcign/firmware)
 * [PCB schematic/layout](https://github.com/mcign/pcb)


### Implementation

#### Hardware

![switch plate](https://mcign.github.io/images/switch_plate.png)
Figure 1. A plastic plate removed from the original switch mechanism.

The hardware is made up of a main PCB that emulates the original switching mechanism shown in Figure 1, with the MCU and bluetooth radio on the opposite side. The main PCB can support either 2 10A relays, or a Molex SlimStack connector to attach a secondary PCB with up to 5 10A relays. To fit the 13.7mm tall relays in the 15.7mm vertical space of the ignition housing, a 1mm connector and 0.6mm and 0.4mm PCBs are needed. The 1mm connector leaves just enough space for the components on the main PCB (the tallest of which is 0.9mm). All relays are carefully arranged to avoid contact with the lock mechanism, which extends a few milimeters into the center of the space with a radius of 9mm and rotates about 150 degrees. The amount of usable space can be seen in figure 2, which shows a spacer from the original switch mechanism. PCB designs are shown in figures 3, and 4. The layout of the nRF51x and antenna are based on the reference design provided by Nordic Semiconductor.

The PCB was designed with Eagle.

![switch spacer](https://mcign.github.io/images/switch_spacer.png)
Figure 2. A spacer from the original switch mechanism that shows the amount of available space.

![main pcb](https://mcign.github.io/images/main_pcb.png)
Figure 3. The front and back layouts of the main PCB.

![relay pcb](https://mcign.github.io/images/relay_pcb.png)
Figure 4. The front and back layouts of the relay PCB.

#### Firmware

The firmware runs on a BGM111 (for the previous version) or a nRF51x (current version, not yet ported). The BGM111 firmware requires Simplicity Studio, a free but proprietary IDE. Code is written in C, and uses hardware encryption whenever possible.

#### Software

The smartphone app uses the Ionic Framework, which is built on Cordova. It is essentially a web app with plugins to access native functionality on the device such as bluetooth, and the app is written in Typescript, HTML, and SCSS. Angular framework is also used. Thanks to Cordova's cross-platform support, the only thing that needs to be done to support iOS is porting a [small cryptography library](https://github.com/tozny/java-aes-crypto).

##### Encryption

Bluetooth pairing is vulnerable to an MITM attack, allowing attackers to steal cryptographic keys and impersonate a legitimate device. Instead of relying on BLE encryption, this project encrypts all commands with [AES-128 and a SHA-256 HMAC](https://github.com/tozny/java-aes-crypto). A unique number is also included with each command to prevent replay attacks.

##### Bluetooth Proximity

The proximity feature is implemented in the smartphone app, which sends commands to switch the ignition on or off. To avoid excessive battery consumption, the smartphone app monitors for an iBeacon advertisement which is broadcast from the ignition before attempting to get a proximity measurement. There are also plans to use geolocation and activity recognition to further reduce battery consumption, by keeping track of where the motorcycle was parked and disabling beacon scanning while out of range, and disabling beacon scanning unless the device detects that the user is "walking" (with activity recognition).

Bluetooth is vulnerable to relay attacks, where two wireless devices are tricked into thinking they're much closer to eachother than they really are. The "keyless start" proximity mode is vulnerable to this attack, [which has also been used to steal luxury cars](https://electrek.co/2018/07/31/tesla-theft-tips-help-prevent-relay-attacks/). The only reliable method to detect a relay attack is to analyze the connection latency, but BLE latency is so much higher than other protocols (such as WiFi) that the extra latency would be nearly impossible to detect. However, a relay attack is much more dificult than hotwiring a motorcycle the traditional way, so this isn't considered a significant disadvantage. It is always recommended to use additional forms of physical security (lock & chain, alarm, etc) when parking a motorcycle in a dangerous area, whether using a stock ignition or a wireless ignition with Bluetooth proximity.

### Planned Features

 * Digital killswitch module
 * Start button override to start the engine with the app (eg. to warm up a bike in a cold climate)
 * Reduce BLE proximity mode with geolocation and activity recognition (scanning is disabled unless the smartphone detects you're walking and are relatively close to where the bike was last parked)
 * GPS & cellular chip for tracking and remote control
 * Motion-sensitive alarm (using stock horn or custom siren)
 * RPM sensor
 * Temperature sensors (oil, coolant, ambient)
 * Detect phone mount (with RFID tag), start navigation app
 * Smartphone app integration with bluetooth headsets (Sena, Domio, etc)
 * auto-start music player and/or Waze
