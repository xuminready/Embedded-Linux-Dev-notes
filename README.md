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
apt-get update
apt-get install crossbuild-essential-armhf
```
## install foreign architecture packages

This makes life much easier when working on cross compiling. 

The package will be install **/usr/lib/arm-linux-geueabihf**

`apt-get install package:architecture`

for example
`apt-get install libicu-dev:armhf`

source:

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

the initrd.img file will be generate in /boot/
