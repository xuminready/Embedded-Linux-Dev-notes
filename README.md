# Embedded-Linux-Dev-notes

# gcc/g++ --sysroot=dir

**dir** must use full path. **~/workspace** will not work(test on gcc 7.4.1)

>Use dir as the logical root directory for headers and libraries. For example, if the compiler normally searches for headers in /usr/include and libraries in /usr/lib, it instead searches dir/usr/include and dir/usr/lib.
>
>If you use both this option and the -isysroot option, then the --sysroot option applies to libraries, but the -isysroot option applies to header files.
>
>The GNU linker (beginning with version 2.16) has the necessary support for this option. If your linker does not support this option, the header file aspect of --sysroot still works, but the library aspect does not.
>
>[Options for Directory Search](https://gcc.gnu.org/onlinedocs/gcc/Directory-Options.html)


# Debian/Ubuntu cross toolchain packages

It's much easy to download a pre-built cross toolchain. Debian/Ubuntu based distribution has cross-tools package which make it easier. Using it with QEMU, ARM program can be execute on x86 machine.

require dpkg version is greater than 1.16.2

>dpkg --add-architecture **armhf**
>
>dpkg --print-foreign-architectures
>
>apt-get update
>
>apt-get install crossbuild-essential-**armhf**

## install foreign architecture packages

This makes life much easier when working on cross compiling. 

The package will be install **/usr/lib/arm-linux-geueabihf**

apt-get install **package:architecture** 

for example
>apt-get install libicu-dev:armhf

source:

[Exploring Raspberry Pi: Interfacing to the Real World with Embedded Linux](http://exploringrpi.com/)

[Multiarch](https://wiki.debian.org/Multiarch/HOWTO)

## QEMU emulating the foreign architecture
It looks like QEMU can do a lot of things. **chroot** to foreign filesystem and execute foreign binary program on x86 platform. It's very useful for embedded development.

>apt install qemu-user-static

[QEMU User Emulation](https://wiki.debian.org/QemuUserEmulation)

[Raspberry Pi Emulation Using qemu-user-static](https://wiki.debian.org/RaspberryPi/qemu-user-static)

