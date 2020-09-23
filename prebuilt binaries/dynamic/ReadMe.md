
## Dynamically linked binaries and libraries

This should be really helpfull for systems that are very limited on free space.
Inside mips32_dynamic.tar.gz are included all the dynamic executables and libraries I built with Buildroot 2010.05. All dependences should be included(libraries, configuration files, etc..). Again, the target has been compiled with 2.6.32 Linux Headers and uClibc 0.9.30. If your target systems kernel version and uClibc is close enough, you can might be able to get away by copying the executables and the rest of the missing dependencies.

Calling any executable with missing dependecies(libraries, configuration files or even other linked executables) from the target system, will return the relevant missing 'file' message.
For instance, for the nano executable to work , proper version of libncurses lib and linker files should be present in on your target's rootfs /lib or /usr/lib directory. Also the terminal database that your system is using should be located either at your system's specified directory path or inside /usr/share/terminfo.
