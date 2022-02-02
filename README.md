# Embedded-Linux-Dev-notes

# gcc/g++ --sysroot=dir

**dir** must use full path. **~/workspace** will not work(test on gcc 7.4.1)

The rootfs extract from an image may contain absolute path symbolic link. For instance **lib\*.so** may link to **/lib/\*.so**. It may caused problem for cross compling. My solution is add `${sysroot}/lib/` to library search path.

>Use dir as the logical root directory for headers and libraries. For example, if the compiler normally searches for headers in /usr/include and libraries in /usr/lib, it instead searches dir/usr/include and dir/usr/lib.
>
>If you use both this option and the -isysroot option, then the --sysroot option applies to libraries, but the -isysroot option applies to header files.
>
>The GNU linker (beginning with version 2.16) has the necessary support for this option. If your linker does not support this option, the header file aspect of --sysroot still works, but the library aspect does not.
>
>[Options for Directory Search](https://gcc.gnu.org/onlinedocs/gcc/Directory-Options.html)

# Linux shared libraries secondary dependencies
When building C/C++ program using sysroot mentioned above, one can run into the shared libraries secondary dependencies problem. The linker actually gives good hints

>ld: warning: *.so, needed by *.so, not found (**try using -rpath or -rpath-link**)
>
>*.so: undefined reference to '**'

As linker suggests, the solution is using -rpath or -rpath-link.
for instance, add the flag below to **ld** or **g++**. 

`-Wl,-rpath-link=${sysroot}/usr/lib/aarch64-linux-gnu:${sysroot}/lib/aarch64-linux-gnu`

>Note that I used -rpath key instead of -rpath-link. The difference is that -rpath-link is used at linking time only for checking that all symbols in the final executable can be resolved, whereas -rpath actually embeds the path you specify as parameter into the ELF:

[Why does ld need -rpath-link when linking an executable against a so that needs another so?](https://stackoverflow.com/a/35748179)

[Better understanding Linux secondary dependencies solving with examples](http://www.kaizou.org/2015/01/linux-libraries.html)

Why need **-Wl,**???

### for cmake 
```
 -DCMAKE_EXE_LINKER_FLAGS="--sysroot=${sysroot} -Wl,-rpath-link=${sysroot}/usr/lib/aarch64-linux-gnu:${sysroot}/lib/aarch64-linux-gnu" \
```


# Debian/Ubuntu cross toolchain packages

It's much easy to download a pre-built cross toolchain. Debian/Ubuntu based distribution has cross-tools package which make it easier. Using it with QEMU, ARM program can be execute on x86 machine.

require dpkg version is greater than 1.16.2
```
dpkg --add-architecture armhf
dpkg --print-foreign-architectures


apt update
apt install crossbuild-essential-armhf
apt install crossbuild-essential-arm64
```
## install foreign architecture packages

This makes life much easier when working on cross compiling. 

The package will be install **/usr/lib/arm-linux-geueabihf**

`apt-get install package:architecture`

for example
`apt-get install libicu-dev:armhf`

source:
[Raspberry Pi Cross-Compiling](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#cross-compiling-the-kernel)

[Exploring Raspberry Pi: Interfacing to the Real World with Embedded Linux](http://exploringrpi.com/)

[Multiarch](https://wiki.debian.org/Multiarch/HOWTO)

## QEMU emulating the foreign architecture
It looks like QEMU can do a lot of things. **chroot** to foreign filesystem and execute foreign binary program on x86 platform. It's very useful for embedded development.

`apt install qemu-user-static`

[QEMU User Emulation](https://wiki.debian.org/QemuUserEmulation)

[Raspberry Pi Emulation Using qemu-user-static](https://wiki.debian.org/RaspberryPi/qemu-user-static)


# use debootstrap to create a minimal Ubuntu/Debian rootfs
Besides the Linux Krenel, rootfs is needed for a Linux system. Rootfs contains all the user program and more. There're several distribution. If you want a minimal version of Ubuntu, you can use debootstrap to create one.  

`debootstrap --arch $ARCH $RELEASE  $DIR $MIRROR`

example below is Ubuntu20.04 focal for arm64.

`sudo debootstrap --arch=arm64 focal rootfs_path`

once it's done, use chroot to add user and install software. Root password is not set by default, add a user, or set rootpassword. Linux kernel modules is not include. 
[chroot_rootfs](https://github.com/xuminready/chroot_rootfs)

```
adduser mx // or use useradd
adduser mx sudo // add mx to sudo group
```

# create initrd.img using initramfs-tools
initrd.img is a small temporary rootfs used when Linux kernel is bootup. It's only requires when the Linux kernel doesn't have storage driver build-in, and rootfs is on that storage driver. 
1. copy Linux modules to **/lib/modules/**
2. install initramfs-tools `apt install initramfs-tools`
3. `update-initramfs -cv -k all`

[update-initramfs](http://manpages.ubuntu.com/manpages/focal/man8/live-update-initramfs.8.html)
the initrd.img file will be generate in /boot/

# PXE Network Boot
It's very convenient to use Network boot if you need to change to different rootfs, or in my case, no storage devices on board. It use tftp to serve pxelinux.cfg, Linux kernel, device tree, and initramfs. NFS is used to serve rootfs.  You can mount tftp or nfs on a Linux machine to debug the server before you try to network boot your board. I even tried to set TFTP and NFS server in Google cloud. TFTP is timeout, I guess it's because it's using UDP. NFS works OK, but very slow.

Please notice set the right permission for NFS export folder and TFTP file.

## Ethernet Controller driver for network boot
Linux kernel need to load Ethernet Controller driver to access rootfs on NFS. There are two way to add the driver for netowrk boot.
1. compile Ethernet driver into the kernel. It depends on the SOC(Ethernet controller), there might be several moudle's need to enable.
2. use an initrd.img(initramfs), the small rootfs contains some basic drivers.

### pxelinux.cfg/default

```
LABEL LinuxBoot
KERNEL Image
FDT meson-gxl-s805x-libretech-ac.dtb
APPEND initrd=initrd.img-5.6.7 boot=nfs root=/dev/nfs rw rootwait nfsroot=192.168.0.21:/home/mx/nfs/rootfs,nolock,vers=3,tcp ip=dhcp
```

*initrd=initrd.img* is optional.

`mount -t nfs -o vers=3 <Server>:</SHARE> /<MOUNTPOINT>` is usful to test NFS server.Make sure *vers* is set to the right NFS version. 

[how to use PXE boot on ROCK Pi 4](https://wiki.radxa.com/Rockpi4/dev/u-boot/pxe)

[How to boot the kernel via TFTP from U-Boot](https://wiki.st.com/stm32mpu/wiki/How_to_boot_the_kernel_via_TFTP_from_U-Boot)

[Configuring PXE Network Boot Server on Ubuntu 18.04 LTS](https://linuxhint.com/pxe_boot_ubuntu_server/)

[U-Boot Environment Variables](https://www.denx.de/wiki/view/DULG/UBootEnvVariables)

[Booting with an NFS Root Filesystem](https://wiki.emacinc.com/wiki/Booting_with_an_NFS_Root_Filesystem)


# pipe in systemD service, and run a script before shutdown
Need to turn off modem before shutdown Linux. 
require: systemD, microcom
```
[Unit]
Description=Turn off modem at ttyUSB3

[Service]
Type=oneshot
RemainAfterExit=true
ExecStop=/bin/sh -c "/bin/echo -ne 'AT+QPOWD\r' | /bin/busybox microcom -s 115200 -t 8000 /dev/ttyUSB3"

[Install]
WantedBy=multi-user.target
```
[How to run a script with systemd right before shutdown?](https://unix.stackexchange.com/questions/39226/how-to-run-a-script-with-systemd-right-before-shutdown)

[Pipe output of script through Exec in systemd service?](https://unix.stackexchange.com/questions/496368/pipe-output-of-script-through-exec-in-systemd-service)

# Compile Just One Kernel Module
There's no need to rebuild the whole Linux kernel if only need some loadable modules. In my case, I want to use an USB DVD driver for Jetson Nano 2G. The drivers are not available in released Image. Lucky, all the drivers can be enabled as loadable modules.

```
make ARCH=arm64 menuconfig

# enable the modules list below
#BLK_DEV_SR (SCSI CDROM support)
#UDF_FS (UDF file system support)
#CDROM_PKTCDVD (Packet writing on CD/DVD media)

make scripts prepare modules_prepare
make -C . M=drivers/scsi
make -C . M=drivers/cdrom/
make -C . M=fs/udf/

sudo cp drivers/scsi/*.ko /lib/modules/$(uname -r)/kernel/drivers/scsi/

sudo mkdir /lib/modules/$(uname -r)/kernel/drivers/cdrom/
sudo cp drivers/cdrom/*ko /lib/modules/$(uname -r)/kernel/drivers/cdrom/

sudo mkdir /lib/modules/$(uname -r)/kernel/fs/udf/
sudo cp fs/udf/udf.ko /lib/modules/$(uname -r)/kernel/fs/udf/

sudo depmod
```
[How to Compile Just One Kernel Module](https://yoursunny.com/t/2018/one-kernel-module/)
