---
title: "Build u-boot with trusted firmware for STM32"
image: images/tee.png
showonlyimage: false
weight: 2
draft: false
---
#### TL;DR

This is a dumbed-down summary of different documentation pieces on u-boot, trusted firmware and STM.
I use this to play around on a local STM board.

### Compile U-Boot

Docs [1] https://docs.u-boot.org/en/latest/board/st/index.html

* Chose a config from the supported boards under `configs/`, in our case `configs/stm32mp15_defconfig`
```shell
make stm32mp15_defconfig
```

* optional customize u-boot features
```shell
make menuconfig
```

* chose a device tree from `dts/upstream/src/<ARCH>/<VENDOR>/<BOARD>` in our case `stm32mp157a-dk1`
```shell
make DEVICE_TREE=stm32mp157a-dk1 -j $(nproc)
```

### Compiling TF-A
From TF-A documentation for STM32MP1 [1]

```
make ARM_ARCH_MAJOR=7 ARCH=aarch32 PLAT=stm32mp1 AARCH32_SP=sp_min \
DTB_FILE_NAME=stm32mp157a-dk1.dtb BL33=../u-boot/u-boot-nodtb.bin \
BL33_CFG=../u-boot/u-boot.dtb STM32MP_SDMMC=1 fip all
```

* Making sense of the parameters:

`ARM_ARCH_MAJOR` and `ARCH` depend on the target device. STM32157C devices thhave a Cortex-A7[3] wich is an aarc32 core compatible with ARMv7-A

`PLAT` A subfolder under `plat/` in our case  `stm32mp1` 

`BL33` This is the no-secure world bootloader/u-boot in our case

`BL33_CFG` Device tree for u-boot (since the device tree TF-A is not visible to u-boot running from RAM)

`DTB_FILE_NAME` device tree under `fdts/`

`AARCH32_SP` Choose the AArch32 Secure Payload component to be built as as the BL32 image when ARCH=aarch32. The value should be the path to the directory containing the SP source, relative to the bl32/; the directory is expected to contain a makefile called <aarch32_sp-value>.mk.

[1]https://trustedfirmware-a.readthedocs.io/en/latest/plat/st/index.html

[2] https://www.st.com/en/microcontrollers-microprocessors/stm32mp157.html

[3] https://en.wikipedia.org/wiki/ARM_Cortex-A7

### Flash SDCARD for STM32MP15XX


#### Partition SD card
We want to create 4 partitions as described in STM docs [1] and U-Boot[2]

[1] https://wiki.st.com/stm32mpu/wiki/STM32_MPU_Flash_mapping#SD_card_memory_mapping

[2] https://docs.u-boot.org/en/latest/board/st/stm32mp1.html#prepare-an-sd-card

* Clear existing partitions by zeroing the device
```shell
sudo dd if=/dev/zero of=/dev/your_device bs=1M count=128
#no need to clean the whole device,just the begining 
```
* Create gpt table and 4 partitions. Names and sizes follow convetions in docs [1] and [2]
```shell
sudo part /dev/your_device
(part) mklabel gpt
(part) mkpart fslb1 0% 1MiB
(part) mkpart fslb2 1MiB 2MiB
(part) mkpart fip 2MiB 4MiB
(part) mkpart bootfs 4MiB 68MiB

(parted) print                                                            
Model: Generic- Micro SD/M2 (scsi)
Disk /dev/sdb: 31.0GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name    Flags
 1      17.4kB  1049kB  1031kB               fsbl1
 2      1049kB  2097kB  1049kB               fsbl2
 3      2097kB  4194kB  2097kB               fip
 4      4194kB  71.3MB  67.1MB  ext4         bootfs
```


Note: Use sectors as units, since MiB and MB will generate alignment issues.

* Now, format the boot partition as an ext4 filesystem. This is where U-Boot saves its environment:
```shell
$ sudo mkfs.ext4 -L boot -O ^metadata_csum /dev/mmcblk0p4
```
The `-O ^metadata_csum` option allows to create the filesystem without enabling metadata checksums, which
U-Boot doesn’t seem to support yet


#### Write the relevant binaries

* fsbl (BL2) and fsbl2 (backup for BL2) include TF-A
```shell
sudo dd of=/dev/sdb1 if=bootloader/trusted-firmware-a/build/stm32mp1/release/tf-a-stm32mp157a-dk1.stm32 bs=1M conv=fdatasync
```

```shell
sudo dd of=/dev/sdb2 if=bootloader/trusted-firmware-a/build/stm32mp1/release/tf-a-stm32mp157a-dk1.stm32 bs=1M conv=fdatasync
```


* The `fip` is located in partition 3 or a partition named fip check relevant docs for each target
```shell
sudo dd of=/dev/sdb3 if=bootloader/trusted-firmware-a/build/stm32mp1/release/fip.bin  bs=1M conv=fdatasync
```

### Serve kenel binaries and source from Host over TFTP

#### install & configure TFTP
```shell
sudo apt install tftpd-hpa
sudo systemctl start tftpd-hpa
sudo cp -r <KERNEL_DIR> /srv/tftp/
sudo chmod -R 777 /srv/tftp/
```

#### ALWAYS stop and cleanup the tftp server
```shell
sudo systemctl stop tftpd-hpa
sudo rm -rf /srv/tftp/*
```