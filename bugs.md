# Bug: `lib/unicorn-arm.min.js` aborts on memory access

Attempting to use memory access instructions causes Unicorn to `abort()` on
execution. The provided ARM assembler with the last compiled Unicorn engine
in the last commit possibly uses optimizations, making the abort either
occur at the start of program execution OR when a memory access instruction
is executed.

## Expected Behaviour

Memory access instructions in the assembly editor element write register
values to the respective memory address specified in the instruction.

## Actual Behaviour

The Unicorn Engine encounters an error on executing a memory access instruction,
aborting the program execution prematurely.

Optimizations may be taken to minimize the number of memory accesses during
execution, resulting in errors occurring well after the apparent execution of a
memory access instruction. These optimizations are not consistent between
"resets" of the register-bank and memory, and `abort()` may occur earlier than
a previous execution `abort()`.

This *might be an assembler parsing error* since `abort()` sometimes occurs
when the pseudo-instruction ldr instruction is encountered.

## Replication

The engine recognizes memory access instructions and attempts to execute them
eagerly. The following code thus fails to run on reading the first instruction,
as OakSim attempts to write to memory address `0x40400`.

```arm
    ldr     r0, =0x40400    ; pseudo-instruction ldr <rX>, =<expr> 
                            ; OakSim's valid memory space is 0x40000 to 0x60000,
                            ; and 0x40400 is not a valid immediate value.
                            ; This pseudo-instruction allows for larger
                            ; immediate values.
    mov     r1, #0x2a
    str     r1, [r0]
```

Output:

```
abort() at jsStackTrace@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:5:18821 stackTrace@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:5:18992 abort@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:29:7211 _abort@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:5:200315 YB@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:13:134507 Sz@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:17:127559 GRa@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:9:184719 invoke_iiiiii@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:5:284951 WC@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:19:111734 SC@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:19:99989 RC@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:19:99479 RH@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:11:132730 ccallFunc@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:5:9906 Unicorn/this.emu_start@https://wunkolo.github.io/OakSim/lib/unicorn-arm.min.js:313:32 CurContext
```

## Fix

The error traceback indicates unicorn.js is the offending component. 
The last build of unicorn.js in this repository dates back to the 
unicorn.js commit [8a0e6bf](https://github.com/AlexAltea/unicorn.js/tree/8a0e6bfc03539d45ee3f7e3673fb4b9992a1fa45).

Assuming commit 8a0e6bf contained an undetected bug, `unicorn-arm.min.js` has
been rebuilt from the latest commit [7a19ad6](https://github.com/AlexAltea/unicorn.js/tree/7a19ad6c00729acdd944e9a75b6d95fea1f0ee87). After the rebuild, 
`unicorn-arm.min.js` no longer aborts on memory access.