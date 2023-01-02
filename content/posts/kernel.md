---
title: "Kernel Adventures"
date: 2023-01-01T22:15:52Z
image: images/kernel1.png
weigth: 4
showonlyimage: false
draft: false
---

Tile image source: https://static.lwn.net/images/pdf/LDD3/ch01.pdf
### How much about linux kernel can be learned in 30 days?

Let's find out.


#### Day 1: [CH1: An introduction to device drivers](https://static.lwn.net/images/pdf/LDD3/ch01.pdf)

_"The Linux kernel remains a
large and complex body of code, however, and would-be kernel hackers need an
entry point where they can approach the code without being overwhelmed by com-
plexity. Often, device drivers provide that gateway"_

Take aways:

* The kernel functionality can be roughly classified as:
   - Process management
   - Memory management
   - Filesystems 
   - Device control
   - Networking


- Device drivers could be of mainly 3 types (There are other more specialized types):
    - Char modules
    - Block modules
    - Network modules 

* In linux a module is a hot-pluggable piece of object code. Usualy resulting from 
the compilation of a device driver, ie. Char module  == Char device driver

##### Day 2, 3: Skipped.

#### Day 4: [open syscall](https://osg.tuhh.de/Advent/01-open/) 

_" Although the kernel has objects somewhat similar to FILE*s, it doesn’t give applications direct access to those objects. Instead, an array called the file descriptor table stores an array of such objects. The file descriptors that applications manipulate are indexes into this table. It’s very easy to check that an integer is in bounds.)"_

Take aways:
* The kernel receive integers for file descriptors and not direct pointers to files.
- FD are mapped on the [`fdtable`](https://cs61.seas.harvard.edu/site/ref/file-descriptors/#gsc.tab=0) and return a [`file struct`](https://elixir.bootlin.com/linux/v5.16/source/include/linux/fs.h#L962) The file struct is the actual manipulable file object.

* Zero a buffer with memset(BUFFER, 0, BUFF_SIZE)



### Easter egg - Outro


During testing of muy solution I found that my `cat` program was writting junk when invoked with `make run` or `./cat cat.c MakeFile`. This was caused by a previously innitialized buffer ~~the initial file being larger than the second one and thus I was writting to STDOUT  the contents of the second file (MakeFile) + junk from the previous buffer contents.~~  To solve this I did 2 things:


1. Debugged with GDB to see how the buffer was populated

```asm
(gdb) file cat 
(gdb) disassemble main 

[...]
0x0000555555555204 <+91>:	mov    esi,0x0
   0x0000555555555209 <+96>:	mov    rdi,rax
   0x000055555555520c <+99>:	mov    eax,0x0
   0x0000555555555211 <+104>:	call   0x5555555550b0 <open@plt>
   0x0000555555555216 <+109>:	mov    DWORD PTR [rip+0x3e24],eax        # 0x555555559040 <fd>
   0x000055555555521c <+115>:	jmp    0x555555555237 <main+142>
   0x000055555555521e <+117>:	mov    edx,0x1000
   0x0000555555555223 <+122>:	lea    rax,[rip+0x2e16]        # 0x555555558040 <buff>
   0x000055555555522a <+129>:	mov    rsi,rax
   0x000055555555522d <+132>:	mov    edi,0x1
   0x0000555555555232 <+137>:	call   0x555555555080 <write@plt>
   0x0000555555555237 <+142>:	mov    eax,DWORD PTR [rip+0x3e03]        # 0x555555559040 <fd>
   0x000055555555523d <+148>:	mov    edx,0x1000
   0x0000555555555242 <+153>:	lea    rcx,[rip+0x2df7]        # 0x555555558040 <buff>
   0x0000555555555249 <+160>:	mov    rsi,rcx
   0x000055555555524c <+163>:	mov    edi,eax
   0x000055555555524e <+165>:	call   0x5555555550a0 <read@plt>
=> 0x0000555555555253 <+170>:	test   rax,rax
   0x0000555555555256 <+173>:	jne    0x55555555521e <main+117>
   0x0000555555555258 <+175>:	add    DWORD PTR [rbp-0xc],0x1
   0x000055555555525c <+179>:	mov    eax,DWORD PTR [rbp-0xc]
   0x000055555555525f <+182>:	cmp    eax,DWORD PTR [rbp-0x14]
   0x0000555555555262 <+185>:	jl     0x5555555551c8 <main+31>
   0x0000555555555268 <+191>:	mov    eax,0x0
```

In <main+170> the return value of read its checked for 0. If 0 is found then we are done reading the file and can  continue to program exit routine on <main+175>, otherwise we go back to the start of the loop at <main+117>.

### Calling write
In <main+117> we are setting EDX to 0x1000. That's the  buffer size DEC 4096 in hex and loading the address of our buffer into RAX and passing that as argument to `write` by moving it to RSI. Finally we pass the file descriptro in the EDI register. Here 0x1 corresponds to the FD of STDOUT... Then Write is called.

### Calling read
This follows the same convention and basically we are passing the following arguments FD, BUFF_SIZE, BUFF
in inverse order. The value of FD is the return value of <main+104> in EAX and stored in memory at [rip+0x3e24] (As this is a relative offset to RIP that is changing during retrieval the offset may change), then this is retrieved from [rip+0x3e03] and passed into EAX again, the next intruction puts DEC 4096 in RDX. The next 2 instructions put the address to the buffer into RSI, and finally the FD ends in EDI as per convention

```asm
   0x0000555555555237 <+142>:	mov    eax,DWORD PTR [rip+0x3e03]        # 0x555555559040 <fd>
   0x000055555555523d <+148>:	mov    edx,0x1000
   0x0000555555555242 <+153>:	lea    rcx,[rip+0x2df7]        # 0x555555558040 <buff>
   0x0000555555555249 <+160>:	mov    rsi,rcx
   0x000055555555524c <+163>:	mov    edi,eax
   0x000055555555524e <+165>:	call   0x5555555550a0 <read@plt>
``` 

For more details on how syscalls parameters are passed see: https://x64.syscall.sh/

So basically printing the buffer contets every time 
```asm
=> 0x0000555555555253 <+170>:	test   rax,rax
```
was reached showed that the buffer was corrput from the previous iteration
</details>

2. Zeroed the buffer after each iteration: `memset(buff, 0, BUFF_SIZE)`