<h1 align="center">Why is 2 * (i * i) faster than 2 * i * i in Java?</h1>



## question

The following Java program takes on average between 0.50 secs and 0.55 secs to run:

```java
public static void main(String[] args) {
    long startTime = System.nanoTime();
    int n = 0;
    for (int i = 0; i < 1000000000; i++) {
        n += 2 * (i * i);
    }
    System.out.println((double) (System.nanoTime() - startTime) / 1000000000 + " s");
    System.out.println("n = " + n);
}
```

If I replace `2 * (i * i)` with `2 * i * i`, it takes between 0.60 and 0.65 secs to run. How come?

I ran each version of the program 15 times, alternating between the two. Here are the results:

```java
 2*(i*i)  |  2*i*i
----------+----------
0.5183738 | 0.6246434
0.5298337 | 0.6049722
0.5308647 | 0.6603363
0.5133458 | 0.6243328
0.5003011 | 0.6541802
0.5366181 | 0.6312638
0.515149  | 0.6241105
0.5237389 | 0.627815
0.5249942 | 0.6114252
0.5641624 | 0.6781033
0.538412  | 0.6393969
0.5466744 | 0.6608845
0.531159  | 0.6201077
0.5048032 | 0.6511559
0.5232789 | 0.6544526
```

The fastest run of `2 * i * i` took longer than the slowest run of `2 * (i * i)`. If they had the same efficiency, the probability of this happening would be less than `1/2^15 * 100% = 0.00305%`.

## answer

There is a slight difference in the ordering of the bytecode.

`2 * (i * i)`:

```java
     iconst_2
     iload0
     iload0
     imul
     imul
     iadd
```

vs `2 * i * i`:

```java
     iconst_2
     iload0
     imul
     iload0
     imul
     iadd
```

At first sight this should not make a difference; if anything the second version is more optimal since it uses one slot less.

So we need to dig deeper into the lower level (JIT)1.

Remember that JIT tends to unroll small loops very aggressively. Indeed we observe a 16x unrolling for the `2 * (i * i)` case:

```java
030   B2: # B2 B3 <- B1 B2  Loop: B2-B2 inner main of N18 Freq: 1e+006
030     addl    R11, RBP    # int
033     movl    RBP, R13    # spill
036     addl    RBP, #14    # int
039     imull   RBP, RBP    # int
03c     movl    R9, R13 # spill
03f     addl    R9, #13 # int
043     imull   R9, R9  # int
047     sall    RBP, #1
049     sall    R9, #1
04c     movl    R8, R13 # spill
04f     addl    R8, #15 # int
053     movl    R10, R8 # spill
056     movdl   XMM1, R8    # spill
05b     imull   R10, R8 # int
05f     movl    R8, R13 # spill
062     addl    R8, #12 # int
066     imull   R8, R8  # int
06a     sall    R10, #1
06d     movl    [rsp + #32], R10    # spill
072     sall    R8, #1
075     movl    RBX, R13    # spill
078     addl    RBX, #11    # int
07b     imull   RBX, RBX    # int
07e     movl    RCX, R13    # spill
081     addl    RCX, #10    # int
084     imull   RCX, RCX    # int
087     sall    RBX, #1
089     sall    RCX, #1
08b     movl    RDX, R13    # spill
08e     addl    RDX, #8 # int
091     imull   RDX, RDX    # int
094     movl    RDI, R13    # spill
097     addl    RDI, #7 # int
09a     imull   RDI, RDI    # int
09d     sall    RDX, #1
09f     sall    RDI, #1
0a1     movl    RAX, R13    # spill
0a4     addl    RAX, #6 # int
0a7     imull   RAX, RAX    # int
0aa     movl    RSI, R13    # spill
0ad     addl    RSI, #4 # int
0b0     imull   RSI, RSI    # int
0b3     sall    RAX, #1
0b5     sall    RSI, #1
0b7     movl    R10, R13    # spill
0ba     addl    R10, #2 # int
0be     imull   R10, R10    # int
0c2     movl    R14, R13    # spill
0c5     incl    R14 # int
0c8     imull   R14, R14    # int
0cc     sall    R10, #1
0cf     sall    R14, #1
0d2     addl    R14, R11    # int
0d5     addl    R14, R10    # int
0d8     movl    R10, R13    # spill
0db     addl    R10, #3 # int
0df     imull   R10, R10    # int
0e3     movl    R11, R13    # spill
0e6     addl    R11, #5 # int
0ea     imull   R11, R11    # int
0ee     sall    R10, #1
0f1     addl    R10, R14    # int
0f4     addl    R10, RSI    # int
0f7     sall    R11, #1
0fa     addl    R11, R10    # int
0fd     addl    R11, RAX    # int
100     addl    R11, RDI    # int
103     addl    R11, RDX    # int
106     movl    R10, R13    # spill
109     addl    R10, #9 # int
10d     imull   R10, R10    # int
111     sall    R10, #1
114     addl    R10, R11    # int
117     addl    R10, RCX    # int
11a     addl    R10, RBX    # int
11d     addl    R10, R8 # int
120     addl    R9, R10 # int
123     addl    RBP, R9 # int
126     addl    RBP, [RSP + #32 (32-bit)]   # int
12a     addl    R13, #16    # int
12e     movl    R11, R13    # spill
131     imull   R11, R13    # int
135     sall    R11, #1
138     cmpl    R13, #999999985
13f     jl     B2   # loop end  P=1.000000 C=6554623.000000
```

We see that there is 1 register that is "spilled" onto the stack.

And for the `2 * i * i` version:

```java
05a   B3: # B2 B4 <- B1 B2  Loop: B3-B2 inner main of N18 Freq: 1e+006
05a     addl    RBX, R11    # int
05d     movl    [rsp + #32], RBX    # spill
061     movl    R11, R8 # spill
064     addl    R11, #15    # int
068     movl    [rsp + #36], R11    # spill
06d     movl    R11, R8 # spill
070     addl    R11, #14    # int
074     movl    R10, R9 # spill
077     addl    R10, #16    # int
07b     movdl   XMM2, R10   # spill
080     movl    RCX, R9 # spill
083     addl    RCX, #14    # int
086     movdl   XMM1, RCX   # spill
08a     movl    R10, R9 # spill
08d     addl    R10, #12    # int
091     movdl   XMM4, R10   # spill
096     movl    RCX, R9 # spill
099     addl    RCX, #10    # int
09c     movdl   XMM6, RCX   # spill
0a0     movl    RBX, R9 # spill
0a3     addl    RBX, #8 # int
0a6     movl    RCX, R9 # spill
0a9     addl    RCX, #6 # int
0ac     movl    RDX, R9 # spill
0af     addl    RDX, #4 # int
0b2     addl    R9, #2  # int
0b6     movl    R10, R14    # spill
0b9     addl    R10, #22    # int
0bd     movdl   XMM3, R10   # spill
0c2     movl    RDI, R14    # spill
0c5     addl    RDI, #20    # int
0c8     movl    RAX, R14    # spill
0cb     addl    RAX, #32    # int
0ce     movl    RSI, R14    # spill
0d1     addl    RSI, #18    # int
0d4     movl    R13, R14    # spill
0d7     addl    R13, #24    # int
0db     movl    R10, R14    # spill
0de     addl    R10, #26    # int
0e2     movl    [rsp + #40], R10    # spill
0e7     movl    RBP, R14    # spill
0ea     addl    RBP, #28    # int
0ed     imull   RBP, R11    # int
0f1     addl    R14, #30    # int
0f5     imull   R14, [RSP + #36 (32-bit)]   # int
0fb     movl    R10, R8 # spill
0fe     addl    R10, #11    # int
102     movdl   R11, XMM3   # spill
107     imull   R11, R10    # int
10b     movl    [rsp + #44], R11    # spill
110     movl    R10, R8 # spill
113     addl    R10, #10    # int
117     imull   RDI, R10    # int
11b     movl    R11, R8 # spill
11e     addl    R11, #8 # int
122     movdl   R10, XMM2   # spill
127     imull   R10, R11    # int
12b     movl    [rsp + #48], R10    # spill
130     movl    R10, R8 # spill
133     addl    R10, #7 # int
137     movdl   R11, XMM1   # spill
13c     imull   R11, R10    # int
140     movl    [rsp + #52], R11    # spill
145     movl    R11, R8 # spill
148     addl    R11, #6 # int
14c     movdl   R10, XMM4   # spill
151     imull   R10, R11    # int
155     movl    [rsp + #56], R10    # spill
15a     movl    R10, R8 # spill
15d     addl    R10, #5 # int
161     movdl   R11, XMM6   # spill
166     imull   R11, R10    # int
16a     movl    [rsp + #60], R11    # spill
16f     movl    R11, R8 # spill
172     addl    R11, #4 # int
176     imull   RBX, R11    # int
17a     movl    R11, R8 # spill
17d     addl    R11, #3 # int
181     imull   RCX, R11    # int
185     movl    R10, R8 # spill
188     addl    R10, #2 # int
18c     imull   RDX, R10    # int
190     movl    R11, R8 # spill
193     incl    R11 # int
196     imull   R9, R11 # int
19a     addl    R9, [RSP + #32 (32-bit)]    # int
19f     addl    R9, RDX # int
1a2     addl    R9, RCX # int
1a5     addl    R9, RBX # int
1a8     addl    R9, [RSP + #60 (32-bit)]    # int
1ad     addl    R9, [RSP + #56 (32-bit)]    # int
1b2     addl    R9, [RSP + #52 (32-bit)]    # int
1b7     addl    R9, [RSP + #48 (32-bit)]    # int
1bc     movl    R10, R8 # spill
1bf     addl    R10, #9 # int
1c3     imull   R10, RSI    # int
1c7     addl    R10, R9 # int
1ca     addl    R10, RDI    # int
1cd     addl    R10, [RSP + #44 (32-bit)]   # int
1d2     movl    R11, R8 # spill
1d5     addl    R11, #12    # int
1d9     imull   R13, R11    # int
1dd     addl    R13, R10    # int
1e0     movl    R10, R8 # spill
1e3     addl    R10, #13    # int
1e7     imull   R10, [RSP + #40 (32-bit)]   # int
1ed     addl    R10, R13    # int
1f0     addl    RBP, R10    # int
1f3     addl    R14, RBP    # int
1f6     movl    R10, R8 # spill
1f9     addl    R10, #16    # int
1fd     cmpl    R10, #999999985
204     jl     B2   # loop end  P=1.000000 C=7419903.000000
```

Here we observe much more "spilling" and more accesses to the stack `[RSP + ...]`, due to more intermediate results that need to be preserved.

Thus the answer to the question is simple: `2 * (i * i)` is faster than `2 * i * i` because the JIT generates more optimal assembly code for the first case.

------

But of course it is obvious that neither the first nor the second version is any good; the loop could really benefit from vectorization, since any x86-64 CPU has at least SSE2 support.

So it's an issue of the optimizer; as is often the case, it unrolls too aggressively and shoots itself in the foot, all the while missing out on various other opportunities.

In fact, modern x86-64 CPUs break down the instructions further into micro-ops (µops) and with features like register renaming, µop caches and loop buffers, loop optimization takes a lot more finesse than a simple unrolling for optimal performance. [According to Agner Fog's optimization guide](https://www.agner.org/optimize/microarchitecture.pdf):

> The gain in performance due to the µop cache can be quite considerable if the average instruction length is more than 4 bytes. The following methods of optimizing the use of the µop cache may be considered:
>
> - Make sure that critical loops are small enough to fit into the µop cache.
> - Align the most critical loop entries and function entries by 32.
> - Avoid unnecessary loop unrolling.
> - Avoid instructions that have extra load time
>   . . .

Regarding those load times - [even the fastest L1D hit costs 4 cycles](https://stackoverflow.com/questions/4087280/approximate-cost-to-access-various-caches-and-main-memory), an extra register and µop, so yes, even a few accesses to memory will hurt performance in tight loops.

But back to the vectorization opportunity - to see how fast it can be, [we can compile a similar C application with GCC](https://gcc.godbolt.org/z/DdEDny), which outright vectorizes it (AVX2 is shown, SSE2 is similar)2:

```java
  vmovdqa ymm0, YMMWORD PTR .LC0[rip]
  vmovdqa ymm3, YMMWORD PTR .LC1[rip]
  xor eax, eax
  vpxor xmm2, xmm2, xmm2
.L2:
  vpmulld ymm1, ymm0, ymm0
  inc eax
  vpaddd ymm0, ymm0, ymm3
  vpslld ymm1, ymm1, 1
  vpaddd ymm2, ymm2, ymm1
  cmp eax, 125000000      ; 8 calculations per iteration
  jne .L2
  vmovdqa xmm0, xmm2
  vextracti128 xmm2, ymm2, 1
  vpaddd xmm2, xmm0, xmm2
  vpsrldq xmm0, xmm2, 8
  vpaddd xmm0, xmm2, xmm0
  vpsrldq xmm1, xmm0, 4
  vpaddd xmm0, xmm0, xmm1
  vmovd eax, xmm0
  vzeroupper
```

With run times:

- SSE: 0.24 s, or 2 times as fast.
- AVX: 0.15 s, or 3 times as fast.
- AVX2: 0.08 s, or 5 times as fast.

------

1 To get JIT generated assembly output, [get a debug JVM](https://github.com/ojdkbuild/ojdkbuild/releases) and run with `-XX:+PrintOptoAssembly`

2 The C version is compiled with the `-fwrapv` flag, which enables GCC to treat signed integer overflow as a two's-complement wrap-around.

## souce

[Why is 2 * (i * i) faster than 2 * i * i in Java?](https://stackoverflow.com/questions/53452713/why-is-2-i-i-faster-than-2-i-i-in-java)