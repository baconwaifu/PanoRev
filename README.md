# PanoRev
Reverse engineering the "Pano Direct" protocol to de-brick the devices for their original purpose.

## Theory of Operation
The Pano "zero client" is not a standard software-based thin-client: It has no hard "CPU" that controls the system.
Instead, the pano is broken up into simple state-machine "units" arranged similar to the peripheral bus on a microcontroller,
and a simple "frame router" routes protocol frames from the ethernet interface to the apropriate unit.
A protocol frame is distinct from the UDP frame that carries them: a UDP frame may carry many protocol frames.
Some protocol frames are "realtime" meaning that if they're dropped, they're dropped, while others rely on a custom "TCP-lite" protocol
for reliable communication.

Every device on the pano has a counterpart attached to it's driver on the VM/host side, and the device can respond with frames *without command from the host*, similar to an "interrupt" in clasical computing.

Fundamentally, this is basically putting a network frontend on a peripheral bus, with a little bit of glue to encode "commands" instead of raw MMIO

## Known Existing Units
* USB controller (enumeration/hubs (possibly interrupt endpoints?) handled in pano hardware, only messages traverse the network)
* Display Controller (uses sparse-update compression + \[unknown\] compression)
  * I *think* I read something about hardware-mouse in here?
* Keyboard/Mouse controller (part of USB controller? are these really separate?)
* Audio controller (PCM? perceptive compression?)
* Network Interface (stuff like IP)
  * "Reliable transport" state machine (handles the TCP-lite protocol for "reliable" messages)
* The LED?
* Initialization controller (Handles initial bringup of device once DHCP obtained)

## "Pano Direct" Protocol
The Pano direct "PDU" frame is a simple TLV-encoded buffer: it encodes the unit it's from/for, the length of the payload, and the actual command (unit specific).
There are also a few flags in there, but I still have to figure those out

## Device Commands

## Known Device Maps
### Gen1
* ???
### Gen2
* ???
