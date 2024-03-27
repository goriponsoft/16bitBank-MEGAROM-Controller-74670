# An implementation example of a 16-bit bank mega ROM controller using a general-purpose logic IC.
[日本語版 README はこちら](https://github.com/goriponsoft/16bitBank-MEGAROM-Controller-74670/blob/main/README.md)

This is a proposed circuit example (74x670 specification) that expands the bank number register to 16 bits based on an ASCII mega ROM controller.
Although operation has not been confirmed, an 8-bit bank mega ROM using a similar 74x670 (4x4 register file) is released as ESE2-RAM, and its operation has been confirmed.

Partially compatible with [NEO-8 mapper](https://aoineko.org/msxgl/index.php?title=NEO_mapper) with the following differences:
- Page 0 mapping is not supported.
- Bank registers and bank mirror addresses are different.

Released under the MIT license.

X(old Twitter): @goriponsoft
