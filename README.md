# chips

A toolbox of chips and helper code to write 8-bit emulators in
dependency-free C99-headers.

*WORK IN PROGRESS*

Tests and example code is in a separate repo: https://github.com/floooh/chips-test

This is the meaty stuff from the YAKC emulator, rewritten from
C++ in C, moved into its own project.

The YAKC emulator is here (porting to the chips headers is underway in the 'chips' branch): 
- github repo: https://github.com/floooh/yakc
- asm.js/wasm demo: http://floooh.github.io/virtualkc/

## Chip Emulators

### Z80 CPU (chips/z80.h)

The Zilog Z80 CPU.

- tick-callback with CPU-pin bitmask, called with machine-cycle or single-tick granularity
- fast switch-case instruction decoder generated via python script
- wait-state injection from tick-callback via WAIT pin
- all undocumented instructions supported
- internal WZ register and undocumented XF and YF flags supported
- support for interrupt-priority handling (daisy chain) with help from the tick callback
- runs the ZEXDOC and ZEXALL tests (more to be added)
- NOT IMPLEMENTED/TODO:
    - NMI (non-maskable interrupts)
    - interrupt mode 0
    - refresh cycle in second half of opcode fetch machine cycle
    - bus request/acknowledge (BUSRQ/BUSAK pins)
    - sequences of DD/FD prefix bytes behave differently than a real CPU

### Z80 PIO (chips/z80pio.h)

The Zilog Z80 Parallel Input/Output controller.

- programmed via Z80-compatible chip-pin bitmask
- two callbacks for passive port A/B input/output
- write-function for active port A/B input (may trigger interrupt)
- can act as interrupt controller in a Z80 interrupt-daisy-chain
- NOT IMPLEMENTED:
    - bidirectional mode

### Z80 CTC (chips/z80ctc.h)

The Zilog Z80 Counter/Timer Channels.

- programmed via Z80-compatible chip-pin bitmask
- emulates the CLK/TRG and ZC/TO input/output pins
- can act as interrupt controller in a Z80 interrupt-daisy-chain

### MOS 6502 CPU (chips/m6502.h)

The MOS Technology 6502 CPU.

- single tick-callback with CPU-pin bitmask, called with tick-granularity
- fast switch-case instruction decoder generated via python script
- emulates all(?) quirks (like redundant and 'junk' read/write cycles, variable cycle counts in some addressing modes, page boundary wrap-around in indirect jump, etc...), mostly verified via visual6502.org
- emulates the known and useful 'documented-undocumented' opcodes (like LAX, SAX, DCP, ...)
- decimal mode implemented, can be disabled
- test coverage:
    - **NESTEST**: completely working (this runs through all documented, and most 'common'
      undocumented instructions but doesn't test decimal mode)
    - **Wolfgang Lorenz C64 Test Suite**: CPU instruction tests working completely 
      for documented instructions (including decimal mode), the following 
      undocumented/unintended instructions are failing: ANE, LXA, SBX, SHA, SHX, SHY, SHS, ANC (some of these are implemented, but fail the test, some are not implemented in the emulator)

### AY-3-8912 (chips/ay38912.h)

The General Instrument AY-3-8912 sound generator chip.

- programmable via chip-pin bitmask
- separate tick function called from CPU tick callback, returns true when a new audio sample is ready
- NOT IMPLEMENTED/TODO:
    - the envelope generator (hard to validate since it was hardly used)

## Helper Code

### Memory (chips/mem.h)

Helper code for 8-bit home computer memory systems.

- map 16-bit address ranges to host memory with 1-KByte page granularity
- memory pages can be mapped as RAM, ROM or RAM-behind-ROM via separate read/write pointers
- up to 4 'mapping layers' to simplify bank-switching or memory expansion systems

### Keyboard Matrix (chips/kbd.h)

Helper code for keyboard matrix typically found in 8-bit home computers.

- map host system key- or ASCII-codes to keyboard matrix locations
- up to 12x12 matrix size
- up to 4 configurable modifier keys (for shift keys)
- internal pressed-key-buffer for simultaneously pressed host system keys
- configurable 'sticky count' to lengthen short host-system key presses to give the emulated system enough time to scan the keyboard matrix state

### Sound Beeper (chips/beeper.h)

A square-wave beeper found in many simple home computers.

- toggle beeper state on/off from CPU tick callback
- tick function returns true when new audio-sample is ready

### Clock (chips/clk.h)

Clock/timer helper functions. This currently only has a single function to convert a duration in microseconds into a number of CPU ticks.

## NOT IMPLEMENTED YET:

- MOS 6522 VIA
- i8255 PPI
- MC6845
- MC6847
- ...