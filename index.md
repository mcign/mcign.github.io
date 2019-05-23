## MCIGN - The Keyless Motorcycle Ignition

### What is MCIGN?

The MCIGN project seamlessly adds keyless start to stock motorcycle ignitions and makes it nearly impossible to hotwire or force the ignition. All modifications are contained in the stock ignition housing, giving a stock appearance. The modified ignition communicates with a smartphone app, and an optional proximity mode can automatically turn the bike on, either in a trusted location such as a garage or wherever the bike was last parked. The lock mechanism isn't altered, so the original key still works. An optional "digital killswitch" module makes hotwiring nearly impossible by using encrypted commands to signal that the ignition has been switched on, instead of just checking the resistance between 2 wires like most stock motorcycles sold in the US.

The ignition firmware can store several digital keys (currently 15), and keys can be set to expire at a certain date, or limited with a curfew (so the key only works at certain hours or on certain days). This is a safer way to loan or rent a vehicle than giving out physical keys, which are easy to copy (and physical locks are hard to change/rekey).

Most motorcycle engines are not started with the key, but instead by a button on the right handlebar (for safety, all controls can be reached while the rider's hands are on the handlebars). This project is able to switch the bike "on" in the same way turning the key would, but it can not start the engine. Most motorcycles also have a steering lock in the ignition that locks the handlebars to the side, making theft more dificult. The steering lock is not altered or disabled by this project, but cannot be controlled remotely.

### Current State

A working prototype has been built and tested on a motorcycle (Honda RC51). The smartphone app uses Ionic framework and cordova, and currently supports Android with iOS support nearly done. An automotive-grade version of the hardware has been designed, but firmware still needs to be ported to the new MCU (from BGM111 to nRF51824).

### Implementation

#### Hardware

![switch plate]()
Figure 1. A plastic plate removed from the original switch mechanism.

The hardware is made up of a main PCB that emulates the original switching mechanism shown in Figure 1, with the MCU and bluetooth radio on the opposite side. The main PCB can support either 2 10A relays, or a Molex SlimStack connector to attach a secondary PCB with up to 5 10A relays. To fit the 13.7mm tall relays in the 15.7mm vertical space of the ignition housing, a 1mm connector and 0.6mm and 0.4mm PCBs are needed. The 1mm connector leaves just enough space for the components on the main PCB (the tallest of which is 0.9mm). All relays are carefully arranged to avoid contact with the lock mechanism, which extends a few milimeters into the center of the space with a radius of 9mm and rotates about 150 degrees. The amount of usable space can be seen in figure 2, which shows a spacer from the original switch mechanism. PCB designs are shown in figures 3, and 4. The layout of the nRF51x and antenna are based on the reference design provided by Nordic Semiconductor.

![switch spacer]()
Figure 2. A spacer from the original switch mechanism that shows the amount of available space.

![main pcb](https://mcign.github.io/images/main_pcb.png)
Figure 3. The front and back layouts of the main PCB.

![relay pcb](https://mcign.github.io/images/relay_pcb.png)
Figure 4. The front and back layouts of the relay PCB.

#### Encryption

Bluetooth pairing is vulnerable to an MITM attack, allowing attackers to steal cryptographic keys and impersonate a legitimate device. Instead of relying on BLE encryption, this project encrypts all commands with [AES-128 and a SHA-256 HMAC](https://github.com/tozny/java-aes-crypto). A unique number is also included with each command to prevent replay attacks.

#### Bluetooth Proximity

The proximity feature is implemented in the smartphone app, which sends commands to switch the ignition on or off. To avoid excessive battery consumption, the smartphone app monitors for an iBeacon advertisement which is broadcast from the ignition before attempting to get a proximity measurement. There are also plans to use geolocation and activity recognition to further reduce battery consumption, by keeping track of where the motorcycle was parked and disabling beacon scanning while out of range, and disabling beacon scanning unless the device detects that the user is "walking" (with activity recognition).
