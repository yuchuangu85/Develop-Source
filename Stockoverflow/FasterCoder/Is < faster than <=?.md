<h1 align="center">Is < faster than <=?</h1>

## question

Is `if (a < 901)` faster than `if (a <= 900)`?

Not exactly as in this simple example, but there are slight performance changes on loop complex code. I suppose this has to do something with generated machine code in case it's even true.

## answer

No, it will not be faster on most architectures. You didn't specify, but on x86, all of the integral comparisons will be typically implemented in two machine instructions:

- A `test` or `cmp` instruction, which sets `EFLAGS`

- And a

   

  `Jcc` (jump) instruction

  , depending on the comparison type (and code layout):

  - `jne` - Jump if not equal --> `ZF = 0`
  - `jz` - Jump if zero (equal) --> `ZF = 1`
  - `jg` - Jump if greater --> `ZF = 0 and SF = OF`
  - (etc...)

------

**Example** (Edited for brevity) Compiled with `$ gcc -m32 -S -masm=intel test.c`

```c
    if (a < b) {
        // Do something 1
    }
```

Compiles to:

```none
    mov     eax, DWORD PTR [esp+24]      ; a
    cmp     eax, DWORD PTR [esp+28]      ; b
    jge     .L2                          ; jump if a is >= b
    ; Do something 1
.L2:
```

And

```c
    if (a <= b) {
        // Do something 2
    }
```

Compiles to:

```none
    mov     eax, DWORD PTR [esp+24]      ; a
    cmp     eax, DWORD PTR [esp+28]      ; b
    jg      .L5                          ; jump if a is > b
    ; Do something 2
.L5:
```

So the only difference between the two is a `jg` versus a `jge` instruction. The two will take the same amount of time.

------

I'd like to address the comment that nothing indicates that the different jump instructions take the same amount of time. This one is a little tricky to answer, but here's what I can give: In the [Intel Instruction Set Reference](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html), they are all grouped together under one common instruction, `Jcc` (Jump if condition is met). The same grouping is made together under the [Optimization Reference Manual](http://www.intel.com/content/www/us/en/architecture-and-technology/64-ia-32-architectures-optimization-manual.html), in Appendix C. Latency and Throughput.

> **Latency** — The number of clock cycles that are required for the execution core to complete the execution of all of the μops that form an instruction.
>
> **Throughput** — The number of clock cycles required to wait before the issue ports are free to accept the same instruction again. For many instructions, the throughput of an instruction can be significantly less than its latency

The values for `Jcc` are:

```none
      Latency   Throughput
Jcc     N/A        0.5
```

with the following footnote on `Jcc`:

> \7) Selection of conditional jump instructions should be based on the recommendation of section Section 3.4.1, “Branch Prediction Optimization,” to improve the predictability of branches. When branches are predicted successfully, the latency of `jcc` is effectively zero.

So, nothing in the Intel docs ever treats one `Jcc` instruction any differently from the others.

If one thinks about the actual circuitry used to implement the instructions, one can assume that there would be simple AND/OR gates on the different bits in `EFLAGS`, to determine whether the conditions are met. There is then, no reason that an instruction testing two bits should take any more or less time than one testing only one (Ignoring gate propagation delay, which is much less than the clock period.)

------

**Edit: Floating Point**

This holds true for x87 floating point as well: (Pretty much same code as above, but with `double` instead of `int`.)

```none
        fld     QWORD PTR [esp+32]
        fld     QWORD PTR [esp+40]
        fucomip st, st(1)              ; Compare ST(0) and ST(1), and set CF, PF, ZF in EFLAGS
        fstp    st(0)
        seta    al                     ; Set al if above (CF=0 and ZF=0).
        test    al, al
        je      .L2
        ; Do something 1
.L2:

        fld     QWORD PTR [esp+32]
        fld     QWORD PTR [esp+40]
        fucomip st, st(1)              ; (same thing as above)
        fstp    st(0)
        setae   al                     ; Set al if above or equal (CF=0).
        test    al, al
        je      .L5
        ; Do something 2
.L5:
        leave
        ret
```

## source

[Is < faster than <=?](https://stackoverflow.com/questions/12135518/is-faster-than)