# MIPS32-Cross-Compile

*In the need for secure remote connection to my home network from abroad, I ended up using an old ISP router as an OpenVpn and SSL enabled device. The router I had laying around is a Speedport W 724V Typ Ci, running proprietary firmware from OTE. As expected it was completely lucking ssh and openvpn functionallity, although it had stunnel that could be used to securely tunnel ssh.Searching on the internet I found that the device is susceptible to shell injection attack. After gaining root shell access through telnet to the device, I found that the firmware is built around Linux 2.6.30(uname -a) and the C Standard library is uClibc 0.9.30(just checked the /lib path for clues for shared libraries ). The SoC on the router is a Broadcom BCM963268 that encloses a Broadcom4350 MIPS32 Big Endian ver.1 cpu.
So the main information needed for cross compiling for the target where found. The cross compiler if set to produce dynamicly linked binaries and libraries should be provided with Linux 2.6.30 Headers and uClibc 0.9.30(uClibc is not backward compatible).*

## Todo:
-Elaborate some more on the Buildroot procedure.

-Although it's really tempting to use the scp command to copy and test the executables on the fly on the target machine, it's actually pretty risky. Need to include qemu emulation procedure.

-Maybe add ssh and openvpn configuration steps.


## - Sourcery's CodeBench Lite Mips Cross-Compiler
   *First cross-compiler I tried was Sourcery's CodeBench Lite 2016.05-8, which is build for MIPS32r2 and by default makes use of Glibc and Linux Headers are for 2.6.32. Although you can pass options to build for MIPS32 rev.1(-march=mips32), build for Big Endian(-EB) and use uClibc 0.9.30 (-muclibc).*

#### (Note to myself) Step one should always be the "Hello World" program. To make my life easier I setup a working environment:

*Make a working directory for all the downloads and builds*
```bash
$ mkdir /home/$(whoami)/mips
$ export WORKDIR=/home/$(whoami)/mips
```

*Download and install cross-compiler*
```bash
$ cd $WORKDIR
$ wget https://sourcery.mentor.com/GNUToolchain/package14486/public/mips-linux-gnu/mips-2016.05-8-mips-linux-gnu-i686-pc-linux-gnu.tar.bz2
$ tar -xf mips-2016.05-8-mips-linux-gnu-i686-pc-linux-gnu.tar.bz2
```

*GNU Cross-Compiler Bin folder path*
```bash
$ export GCC_PATH=/${WORKDIR}/mips-2016.05/bin 
```

*Let the system know where the compiler excecutables are*
```bash
$ export PATH=${GCC_PATH}:$PATH
```

*Compiler excecutable*
```bash
$ export HOST=mips-linux-gnu
```

*Setup compiler options to produce big endian output, for mips32, using uClibc, soft-floating point arithmetics and produce staticly linked self-contained binaries. But actlually doesn't work for this compiler*
```bash
$ export CFLAGS="-EB -march=mips32 -muclibc -msoft-float"
$ export LDFLAGS="-static"
```

*Write Hello World program*
```c
$ nano hello.c
```
```code
#include <stdio.h>

void main(){
printf("Hello World\n");
}
```
Cntl + x and save

*Simple makefile*
```bash
$ nano Makefile
```
```code
CC=${HOST}-gcc
EXTRACFLAGS=-EB -march=mips32 -muclibc -msoft-float -static

hello: hello.c
	${HOST}-gcc hello.c -o hello $(EXTRACFLAGS)
clean:
	rm -f hello
```
Cntl + x and save

*Run file command on the binary*
```bash
$ file hello
hello:ELF 32-bit MSB executable, MIPS, MIPS32 rel2 version 1 (SYSV), statically linked, not stripped
```
IF one needs to include headers and link to libraries that have already been cross-compiled on other paths, the program should be compiled with these options
```bash
CFLAGS="-I${ROOTFS}/usr/include" LDFLAGS="-L${ROOTFS}/usr/lib"
```
Where $ROOTFS is the intermediate directory I used for the output objects

## Now lets build zlib and OpenSSL

*Path of the output folder for the intermediate and the final installs*
```bash
$ mkdir /home/$(whoami)/mips/rootfs
$ export ROOTFS=/home/$(whoami)/mips/rootfs
```
*Download and extract zlib, then configure and cross-compile*
```bash
$ cd $WORKDIR
$ wget https://www.zlib.net/zlib-1.2.11.tar.gz
$ tar -xvf zlib-1.2.11.tar.gz
$ cd zlib-1.2.11
 
$ CC=${HOST}-gcc AR=${HOST}-ar RANLIB=${HOST}-ranlib ./configure --prefix=$ROOTFS/usr  
$ make $(EXTRACFLAGS)
$ make install
```
*Download and extract OpenSSL, then configure and cross-compile*
```bash
$ git clone https://github.com/openssl/openssl
$ cd openssl
$ ./Configure linux-mips32 shared zlib-dynamic \
--cross-compile-prefix=mips-linux-gnu- \
--prefix=/usr \
--libdir=/usr/lib \
--openssldir=/usr/etc/ssl \
--with-zlib-include=$ROOTFS/usr/include \
--with-zlib-lib=$ROOTFS/usr/lib
```
After configuring we need to edit the Makefile and add DESTDIR=${ROOTFS}.Then simply run:
```bash
$ make CFLAGS="-I${ROOTFS}/usr/include" LDFLAGS="-L${ROOTFS}/usr/lib" $(EXTRACFLAGS)
$ make install
```

Inside the rootfs directory there is the user folder that contains all the necessary libs(zlib and libcrypto .so files in the /lib dir) and binaries(/usr/bin and /usr/sbin) that can be transfered to the target machine(remember to remount the targets rootfs or userfs as rw).  After transfering them to the target system, remember to chmod +x the binaries.
 


## Conclusion
-- **Sourcery's CodeBench Lite Mips Cross-Compiler** can produce working static binaries for my target machine. On the other hand, cross-compiling dynamic libraries and linking programs proved to be a real chore. I managed to cross-compile dynamic zlib and openssl, but other programs ended up partially working on the rootfs on my target router. 
Working with **Buildroot's toolchain cross-compiler** in contrast, made things really easy and simple. Make and configure process had way less hickups and stuff to fix were little to none. 

-- **If you want to grab some binaries and/or see how to use Buildroot to build staticly and dynamicly linked programs go to the Buildroot dir on the git.**
