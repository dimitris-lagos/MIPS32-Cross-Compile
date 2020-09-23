
## Dynamically linked binaries and libraries

Inside mips32_dynamic.tar.gz are included all the dynamic binaries and libraries I built with Buildroot 2010.05. All dependences should be included(libraries, configuration files, etc..).

For instance, for the nano binary to work , proper version of libncurses lib and linker files should be present in on your target's rootfs /lib or /usr/lib directory. Also the terminal database that your system is using should be located either at your system's specified directory path or inside \usr\share\terminfo\.
