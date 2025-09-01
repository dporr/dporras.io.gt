---
title: "Cross-compiling libcapstone correctly"
image: images/capstone.png
showonlyimage: false
weight: 2
draft: false
---
#### TL;DR

x-compile libcapstone.so.4 in an x86 host and pass it to qemu-user aarch64 to run binaries that have a dependency on capstone

### X-Compile libcapstone for Aarch64

Seems that capstone/cmake won't respect your `CC`, `CXX` variables when cross-compiling. This is how I got libcapstone cross-compiled Host: x86, target: aarch64
```shell
sudo CROSS=/usr/bin/aarch64-linux-gnu- CAPSTONE_ARCHS="aarch64" ./make.sh install 
```
They have that `CROSS` variable  and I gave it a try, turns out it works:
```shell
$ file /usr/lib/libcapstone.so.4 
/usr/lib/libcapstone.so.4: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, BuildID[sha1]=e44f40ba64cbe01071992d33111a997d2a151d9e, not stripped
```
 

And this is how you run the bins inside qemu-user or "locally":
```shell
#You need these two libs cross-compiled
churro@pwn:~/Desktop/ARM$ ls /usr/aarch64-linux-gnu/lib/ld-linux-aarch64.so.1 /usr/aarch64-linux-gnu/lib/libcapstone.so.4 
/usr/aarch64-linux-gnu/lib/ld-linux-aarch64.so.1  /usr/aarch64-linux-gnu/lib/libcapstone.so.4
#Then just qemu-user the planet 
churro@pwn:~/Desktop/ARM$ qemu-aarch64 -L /usr/aarch64-linux-gnu/  level-1-0 
```

