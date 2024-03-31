# Implementation example of 16bit segment 8KB bank mega ROM controller using general-purpose logic IC
[日本語版 README はこちら](https://github.com/goriponsoft/16bitBank-MEGAROM-Controller-74670/blob/main/README.md)

This is a proposed circuit example (74x670 specification) that expands the segment register to 16 bits based on an ASCII-8 mega ROM controller.
Although operation has not been confirmed, an 8-bit segment mega ROM using a similar 74x670 (4x4 register file) is released as [ESE2-RAM](https://github.com/goriponsoft/ESE2RAM-Cartridge-74670), and its operation has been confirmed.

Released under the MIT license.

X(old Twitter): @goriponsoft

## Specifications
### Overview
Based on the ASCII mapper, by expanding the segment register to 16 bits, the number of segments is increased by 256 times to a maximum of 65,536 segments, making it possible to manage memory of up to 4 Gbit in the case of 8KB banks and up to 8 Gbits in the case of 16KB banks become.

### Differences with NEO-8 Mapper
Partially compatible with [NEO-8 mapper](https://aoineko.org/msxgl/index.php?title=NEO_mapper) with the following differences:
- Page 0 mapping is not supported and there is no segment register for page 0.
- The segment register and bank mirror addresses are different.
- The initial value of the segment register is different.
- The segment register is fully implemented with 16 bits, and the high 4 bits can also be used. The most significant bit is implemented as a backup RAM selection bit.

### Write access
In the conventional ASCII mapper, the low byte is located at even addresses (bit 0 of the address is 0) in the segment register address range, and the high byte is located at odd addresses (bit 0 of the address is 1). The idea of this part is completely the same as the NEO mapper.

In this implementation example (and NEO mapper), the low byte and high byte of the segment register are separated, and only the low or high byte can be rewritten, so as long as you do not access the high byte of an odd address, it is similar to a conventional ASCII mapper. The same code works perfectly.

Similar to the ASCII mapper, writes to the segment register range (6000h-7FFFh) are not transparent to the backup RAM segment (segment most significant bit is 1). Even if you map a backup RAM segment to banks 6000h-7FFFh, you cannot write to it. Whether writing to a ROM segment (the most significant bit of the segment is 0) or not using the backup RAM write signal (/MEM_WR) is transparent depends on the circuit configuration of the decoding section, which is not included in this implementation example. Therefore, please be careful when installing flash memory, etc.

Whether mirrors appear on page 0 and page 3 depends on the circuit configuration of the write signal (/WR).

### Read access
The contents of the ROM are mapped in the same way as the conventional ASCII mapper, except that the segment register is 16 bits. Since the segment register is write-only, the contents of the ROM can be read even within the segment register range.

Whether mirrors appear on page 0 and page 3 depends on the circuit configuration of the read signal (/RD). For example, if you use the /CS12 signal instead of the /RD signal, no mirror will appear for reading.

### Segment register initial value and initialization code
The 74HC670, the IC used to implement the segment register, does not have a reset function, and the register values are undefined immediately after power is turned on.
However, if the segment at startup of the MSX system is undefined, it cannot be used as a mapper, so as a countermeasure, immediately after power-on (and immediately after reset), segment output from the 74HC670 is stopped, and all bits of all segment registers are pulled down 0, the circuit is configured so that segment 0 is mapped to all banks.

The purpose of this feature is to ensure that segment 0 is mapped to all banks, including banks 4000h to 5FFFh, until the MSX system searches for the ROM header and executes the INIT entry for page 1. (This is similar to how banks 4000h to 5FFFh are fixed to segment 0 in the KONAMI4 mapper.)

However, since it does not function as a mapper as it is, segment fixing is now triggered by the "first write to the segment register" and stops (segment output by 74HC670 resumes).
At that time, all segment registers are switched at the same time, and the undefined value immediately after the power is turned on (in the case of a reset, the value written before the reset) is output to the segments other than the segment register where the write was performed. Therefore, from the program side, it appears that the mapping of all banks from 4000h to BFFFh has changed.

Due to the above circumstances, when using this mapper, it is necessary to map banks 4000h to 5FFFh to segment 0 and other banks to appropriate segments as early as possible during INIT entry.

### Mapper
|Bank|Segment register address|Initial Segment|
|:--|:--|--:|
|4000h-5FFFh(mirror at 0000h-1FFFh)|low byte 6000h/high byte 6001h (mirror at 6002h-67FFh.)|Undefined [^1]|
|6000h-7FFFh(mirror at 2000h-3FFFh)|low byte 6800h/high byte 6801h (mirror at 6802h-6FFFh.)|Undefined [^1]|
|8000h-9FFFh(mirror at C000h-DFFFh)|low byte 7000h/high byte 7001h (mirror at 7002h-77FFh.)|Undefined [^1]|
|A000h-BFFFh(mirror at E000h-FFFFh)|low byte 7800h/high byte 7801h (mirror at 7802h-7FFFh.)|Undefined [^1]|

[^1]: Until the first write is made to the segment register, it is fixed at 0, and all the segment registers other than the one to which the write is performed change to an undefined value all at once.
