# Pano Internal Architecture
Deductions as to the actual FPGA layout of the pano.

## VGA Controller
Appears to be an off-the-shelf IP with a framebuffer in DDR memory.
All the registers are controlled via EP0.
The tiling and compression are likely handled by the actual endpoint logic.

## Display Controller
The tiling utilizes a form of run-length encoding for the pixel data.
Note that it copies entire runs of RGB pixels, not bytes.
The first byte is the tile 'opcode', with 1 being "24bpp tile" and 9 being "4bpp" (palletized?)
Next two bytes are X and Y coordinates of the tile on the screen. likely drawn from top-left?
The RLE is a bit more fun: 0x80 "tag" flag, ORed with the length - 1 (total 0x40 pixels, so 8x8)
followed by the tag's pixel value.

## EP0
Just a bog-standard MMIO peripheral bus using a 16-bit address and 32-bit data.
A bit of glue to attach it to a network stack, but still just an MMIO bus.
Registers are likely routed to off-the-shelf IP blocks, rather than anything pano-specific.
Also contains a "config space" which is read by different commands to traditional read/write.

### Config addresses
* 0x0: Board ID.
* 0x1: Bitstream Revision/"Chip ID"
* 0x10-11: Device MAC address (MSB first)
* 0x27: Watchdog Timer.
* 0x30: DHCP control (named `npc_ctrl` in the test suite)

### Peripheral Addresses
* 0xFF: The button on the pano. Reads current state (can also 'event' to the host)
* 0x1000: "Video Display Controller"
* 0x2000: DDR controller.
* 0x3000: VGA DAC controller.
* 0x4000: Audio Codec. 0 is ctrl, 1 is command, 2 is status.
* 0x5000: SPI Flash controller. register 1 is address, register 0 is data, register 2 is "erase"
* 0x6000: i2c controller.
* 0x7080: RNG. 0-3 are data registers, 4 is control. write 1 to enable, 0 to disable.
* 0x8000: LED controller. holds a few different patterns for different states.
