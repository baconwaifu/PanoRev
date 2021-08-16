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
* Display Controller (uses a combination of tiles and run-length encoding)
  * Supports blitting?
* Keyboard/Mouse controller (part of USB controller? are these really separate?)
* Audio controller (PCM? perceptive compression?)
* Network Interface (stuff like IP)
  * "Reliable transport" state machine (handles the TCP-lite protocol for "reliable" messages)
* The LED?
* Initialization controller (Handles initial bringup of device once DHCP obtained)

## "Pano Direct" Protocol
The Pano direct "PDU" frame is a simple TLV-encoded buffer: it encodes the unit it's from/for, the length of the payload, and the actual command (unit specific).
There are also a few flags in there, but I still have to figure those out

Apparently at some point I dug up some documentation listing the protocol as "TNP" or "Thin(ium) Network Protocol".
A UDP "datagram"'s structure is as follows:
* 0: Protocol version
* 1: Flags
* 2-4: sequence and ack-sequence as packed 12-bit integers
  * 2: `seq << 4`
  * 3: `((ack >> 4) & 0xF0) | (seq & 0xf)`
  * 4: `ack & 0xff`
* 5-7: "security" (???)
* 8+: Unit PDUs

Unit PDU: 
* 0: 5-bit endpoint ID (0x1F), upper 3 bits of length field (0xE0)
* 1: lower 8 bits of length field.
* 2+: opcode list (contains 1+ "opcodes" or unit commands)

## Device Commands
A basic "opcode" is structured as such: (Opcode IDs are unit-specific)
* 0: 4-bit opcode ID (0x0F) and 4-bit dword-length (0xF0)
* 1-3: *word* address operand to opcode?
* 4-5: An unsigned short I called "trans" (???)
* 6+: `u32\[len\]`  
Fun fact: responses *from* the pano to the host are *also* encoded as opcodes. often different ones! (zero-endpoint has two for config read: request and response, 0 and 2 respectively)

## Initialization Sequence
When a pano first turns on, the bitstream is loaded from onboard flash. From there, it attempts to obtain an address from DHCP.
If this is successful, the pano will then do nothing. In order for the pano to do something, it must recieve a special vendor option in the DHCP response.
It is currently unknown whether this option is *truly* required, since you can connect to a pano "unsolicited" and simply redirect it's 'peer' address,
as the protocol is UDP with minimal, if any, security. For automatic operation, however, the option is needed to point the pano to a controller
(would probably also work with a pano-direct equipped VM, however I have not tested that)

Once the pano has something to talk to, the "full" initialization sequence starts. This involves a bunch of writes to registers
in the zero-endpoint, which control the operation of the other endpoints. Take the display controller for example: all the VGA config
is done in the zero-endpoint, with only tile data flowing to the actual "video" endpoint.

## Known Device Maps
"Units" in the pano protocol support what are known as "Capability Maps" which serve to identify the unit type, and several type-specific parameters.
"Zero-endpoint" is used to detect and "bringup" other endpoints, as well as manage "login" to the controller.
### Gen1
* 0: "Configuration/Bringup" along with some raw IO registers (SPI flash and i2c)
  * Contains "VDC debug path" which allows raw access to DDR (currently just a framebuffer AFAIK)
* 3: Wolfson audio codec (CHECK)
### Gen2
* ???

## Relevant Software
[x] == "Acquired a copy"
- [x] PanoDirect (VM Drivers)
- [ ] PanoController (what they connect to on "boot")
- [ ] Pano Maestro (Controller clustering)
- [x] PanoConnector-SCCM (??? something to do with on-demand VMs and hyper-v ???)
- [x] PanoGateway (???)
- [x] PanoRemote ("Virtual Pano" software client. Requires a USB-key that may contain an HSM? might also just be a flash drive)
- [x] PanoVirtualClient (???)
- [x] full-license (Unlocks all known features, 1000 of each device, and is valid until 2099. plenty of time to reverse in full)
- [x] Something called 'inferno.ovf' that holds a copy of PanoManager and 'tnptest' (a 'holy grail comprehensive test program' in my eyes. in source form, too)
