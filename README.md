# malloc
 
 This project is about implementing a dynamic memory allocation mechanism.

---



<details> 
<summary>References</summary>

https://levelup.gitconnected.com/malloc-from-scratch-dbc1bc23dfde

system calls

mmap(2)

(pt) [playlist youtube : sistemas operacionais Tiago Tavares](https://youtu.be/Acy9ra2L9pY?si=CBHO9gIefcEDReiW)

munmap(2)

</details>

<details>
<summary>8 bits block </summary>

That final line size = (size + sizeof(void*) — 1) & ~(sizeof(void*) — 1) is a common trick to align size up to the next multiple of sizeof(void*). We do this because some CPU architectures either require or strongly prefer that certain data types are aligned to specific memory boundaries.

size = (size + sizeof(void*) - 1) & ~(sizeof(void*) - 1);

sizeof(void*) == 8

size = (size + 8 - 1) & ~(8 - 1);

size must be multiple of 8
```
8
16
24
32
40
48
...
```

exp:
```

CPUs and Address Alignment
That final line size = (size + sizeof(void*) — 1) & ~(sizeof(void*) — 1) is a common trick to align size up to the next multiple of sizeof(void*). We do this because some CPU architectures either require or strongly prefer that certain data types are aligned to specific memory boundaries. There’s a physical hardware reason for why some CPUs can’t access misaligned data. RAM is fundamentally a giant array of bytes, where each byte has a unique address. However, CPUs don’t fetch data one byte at a time. They read and write memory in fixed-size chunks called words which are typically 4 bytes on 32-bit systems or 8 bytes on 64-bit systems.


```
Address:    0    1    2    3  |  4    5    6    7  |  8    9   10   11
           ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
RAM:       │    │    │    │    │    │    │    │    │    │    │    │    │
           └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
           └──── Word 0 ─────┘ └──── Word 1 ─────┘ └──── Word 2 ─────┘
              (one read)            (one read)            (one read)
```

Now take a look at the above visualization and suppose we’re on a 32-bit system where the memory bus is 4 bytes wide, so the CPU can only read address 0–3 together, addresses 4–7 together, and addresses 8–11 together in one operation. With aligned access, the CPU only has to perform one memory operation since an entire integer fits perfectly in one word because the int datatype is 4 bytes. With misaligned access, what ends up happening varies by architecture. ARM and SPARC enforce strict alignment, so the hardware will raise an alignment fault and the program will crash. On x86, unaligned access doesn’t crash, but it’s slower because when data spans across word boundaries, the CPU has to perform multiple memory operations to reconstruct the value.

```
On x86:
Aligned at address 0:

[12][34][56][78] | [??][??][??][??]
└─── Word 0 ────┘   └─── Word 1 ────┘
CPU reads Word 0 and gets entire int

Misaligned at address 1:

[??][12][34][56] | [78][??][??][??]
└─── Word 0 ────┘   └─── Word 1 ────┘
CPU must read BOTH words and piece together bytes [12][34][56][78]
```

Hopefully that covered the why well enough, so I’m going to take a step back to the actual line itself: size = (size + sizeof(void*) — 1) & ~(sizeof(void*) — 1). The line rounds up to the nearest multiple of the pointer size (typically 8 bytes on 64-bit systems) by first adding 7 to make sure any non-aligned size gets bumped into the next alignment boundary. So for example, say size = 13, then 13 + 7 = 20 bytes = 0000…0001 0100. Then, we use the bitwise mask ~7 = 1111…1111 1000 with the & operator to clear the lower 3 bits, effectively rounding down to a multiple of 8. 0000…0001 0100 & 1111…1111 1000 = 0000…0001 0000 = 16 bytes. This combination of "round up then round down" gives us the proper rounding to the nearest pointer size multiple. With that line out of the way, let’s continue with the rest of the implementation.


exp: 

size = 13

13 + 7 = 20

binary:

13 = 00001101
 7 = 00000111
----------------
20 = 00010100

then:

```& ~7```

7 = 00000111 (negation) ~7 = 11111000

```
00010100   (20)
11111000   (~7)
--------
00010000   (16)
```

Result: ```16```

ref: https://levelup.gitconnected.com/malloc-from-scratch-dbc1bc23dfde
</details>
