
Welcome to this project!

This repo contains all files you need to build and program an 8-bit CPU that I have designed.

You would need to have the two PCBs fabricated: Firstly, there's the CPU board itself.
It is actually a micro-controller, in that this board implements RAM and an IO interface.
Then secondly, there is an IO expansion board, that fits on top of the CPU board via Arduino type stackable headers.

The CPU board looks like this:

![CPU board](https://github.com/Dosflange/Sonne/blob/main/board_and_screws.jpg)

These plastic spacers go between the CPU board and the IO board. The distance/length of the spacer part (without the protruding thread) is 2 cm.

![CPU board](https://github.com/Dosflange/Sonne/blob/main/board_screws.jpg)

Both modules go together like so:

![CPU board](https://github.com/Dosflange/Sonne/blob/main/board_sandwich.jpg)

The assembled development kit i.e. hardware portion of this project:

![CPU board](https://github.com/Dosflange/Sonne/blob/main/board_ready.jpg)

### Address space

The address space of this CPU is unusual. Eight bits give you 256 address places. The first 128 places form a region, which can be switched to point to another memory segment ("bank switching"). The next 64 places after that are fixed (for global variables). The remaining 64 places form a region again, which can be switched out to point to another memory segment (for local variables).

### Instruction format

There are four types of instructions. (1) The high order four bits of the instruction word are all zero: These are sixteen "signal" instructions that take no operands, where the signal is encoded in bits 0-3. (2) The high order bit is zero, but bits 4-6 are not all zero: These are register-to-register transfers, where bits 4-6 encode the source register, bits 0-3 encode the target register. (3) The high order bit is one, bit 6 is 0: These are micro-calls, where bits 0-5 encode the call-target. (4) Bits 6 and 7 or both one: These are register-to-memory transfers, from the two accumulator registers to the first 8 bytes of the global or local memory section, where bit 5 encodes which register it is, bit 4 encodes global or local, bit 3 is whether it's load or store, and bits 0-2 encode the address offset. (See the opcode_matrix file in the repo for details.)

## Groups of registers

### L

L stands for literal. It's a pseudo-register that can only be a source in register-register transfers. The value transfered is the next byte in the instruction stream, which is then skipped.

### A, G and M

M is a pseudo-register. Reading or writing it transfers between memory and a register operand.
By default, the G register holds the byte offset to which reading from or writing to M will go in memory. The G register can be forced to take on this role by executing the SETG signal. The SETA signal switches to register A to provide the byte offset.

### R

The R register holds the bank prefix of the 128 byte "row" region from 0-127 of the address space. Changing R switches all 128 bytes simultaneously by address mapping.

### B, E, T

These are pseudo-registers, that can only occur as register-targets in transfers. Transfering a value to B does an unconditional jump to that "branch". You can't jump to a byte offset, only to the first byte offset (0) of a branch. E means Else - the branch is taken when the ALU result is zero. T means Then - the branch is taken when the ALU result is not zero.

### C, X, Y

The C register is a pseudo register, that can only occur as a register-target in transfers. Transfering a value to C executes a subroutine call. The "branch" get's stored in T, the byte offset for instruction fetches within that "branch" gets stored in X. By writing values into X and Y, the return address can be redirected. The RET signal returns from the subroutine, by restoring the "branch" and byte offset from X and Y. The PULL signal transfers the current X into A, and the current Y into R.

### Q, A, F

Q and A are two accumulator registers. These are the two input operands connected to the ALU (Arithmetic-Logic Unit). F stands for Function. Writing an ALU opcode to F computes the respective function. Reading from F yields the result. Reading from F has the side effect of restoring the default function ("Value of Q"). So in the default state, reading from F is the same as reading Q.

### U, V, S, P

Each bit in the U and V registers selects an implementation specific IO device. Writing to P transfers a value onto the tri-state IO bus. Reading from P transfers a value from the IO-bus. Use the OFF signal to tri-state the bus. Writing to P removes the tri-state.

Writing to the S register prepares the transfered byte for serialization. The CSO signal ("clock serial out") shifts out one bit on the MOSI line. Reading from S yields the currently deserialized byte in the shift-register. The CSI signal ("clock serial in") shift in one bit from the MISO line. The SCH and SCL signals respectively toggle the serial master clock high or low.









