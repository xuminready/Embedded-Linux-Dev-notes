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
