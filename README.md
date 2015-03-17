pintos
======
[manual](http://web.stanford.edu/class/cs140/projects/pintos/pintos.html)  
  
The Pintos study edition by Huawen Yu.  

install on fedora 21 with bochs|qemu
====================================

[ref](https://pintosiiith.wordpress.com/2012/09/13/install-pintos-with-qemu/)  
Please change the $HOME '/home/wilson' to your own home directory.  
Now the qemu hang on 'Powering Off ...', so use bochs as our default simulator.

source
------
$ cd ~; mkdir proj; cd ~/proj  
$ git clone https://github.com/huawenyu/pintos.git  

simulator-qemu
--------------
Fedora, qemu is called qemu-system-i386  
  
$ sudo yum install qemu  
$ sudo ln -s /bin/qemu-system-i386 /bin/qemu  

simulator-bochs
---------------
If install from fedora using 'sudo yum install bochs', there have serval error when start bochs.  
So we can install it from source. If download the latest tar from bochs, even have compile error.  
So we use svn checkout the developing code and import it here.  

###build from newest version

```bash
### get the source
$ svn co http://svn.code.sf.net/p/bochs/code/trunk/bochs bochs
$ cd bochs
<or>
$ cd /home/wilson/proj/pintos/tools
$ tar xjf bochs.tar.bz2
$ cd bochs

### $ ./configure LDFLAGS='-pthread'  <<< fedora gui/libgui.a(x.o): undefined reference to symbol 'XSetForeground'
$ ./configure LDFLAGS='-pthread' --enable-gdb-stub --with-x --with-x11 --with-term --with-nogui

### install the devel lib for fedora
$ sudo yum install libX11-devel libXrandr-devel xorg-x11-server-devel
$ make
$ sudo make install
```

###build from version 2.2.6

[Using Bochs as a simulator](http://courses.mpi-sws.org/os-ss13/assignments/pintos/pintos_13.html)  
[Install Pintos](http://web.stanford.edu/class/cs140/projects/pintos/pintos_12.html#SEC167)  
[Solve compile error](http://blog.csdn.net/geeker_12/article/details/11409009)

```diff
index- bochs-2.2.6/bx_debug/symbol.cc:97
using namespace std;  
  
+ #ifdef __GNUC__
+ using namespace __gnu_cxx;
+ #endif

struct symbol_entry_t
{
```

$ cd proj/pintos/src/misc
$ sudo yum install SDL-devel byacc dev86 docbook-utils gtk2-devel iasl libXpm-devel libXt-devel readline-devel svgalib-devel
$ sudo yum groupinstall "Development Tools" "Development Libraries"
$ ./boshs-2.2.6-build.sh
env SRCDIR=<srcdir> PINTOSDIR=<srcdir> DSTDIR=<dstdir> sh ./bochs-2.2.6-build.sh
$ env SRCDIR=/home/wilson/proj/pintos/tools PINTOSDIR=/home/wilson/proj/pintos DSTDIR=/home/wilson/bin sh ./bochs-2.2.6-build.sh

patch scripts
-------------
$ patch src/utils/pintos-gdb  
```diff
-GDBMACROS=/usr/class/cs140/pintos/pintos/src/misc/gdb-macros
+GDBMACROS=/home/wilson/proj/pintos/src/misc/gdb-macros
```
  
$ patch src/utils/pintos  
```diff
-    $sim = "bochs" if !defined $sim;   <<< if using simulator-bochs
+    $sim = "qemu" if !defined $sim;    <<< if using simulator-qemu
  
-	my $name = find_file ('kernel.bin');
+	my $name = find_file ('/home/wilson/proj/pintos/src/threads/build/kernel.b<
```
  
$ patch src/utils/Pintos.pm  
```diff
-    $name = find_file ("loader.bin") if !defined $name;
+    $name = find_file ("/home/wilson/proj/pintos/src/threads/build/loader.bin") if !defined $name;
```

patch tools
-----------
$ cd src/utils  
$ make  
squish-pty.c:10:21: fatal error: stropts.h: No such file or directory  
 #include <stropts.h>  
  
$ patch src/utils/squish-pty.c  
```diff
+/*
 #include <stropts.h>
+*/

   /* System V implementations need STREAMS configuration for the
      slave. */
+  /*
   if (isastream (slave))
     {
       if (ioctl (slave, I_PUSH, "ptem") < 0
           || ioctl (slave, I_PUSH, "ldterm") < 0)
         fail_io ("ioctl");
     }
+  */
```
  
$ patch src/utils/squish-unix.c  
```diff
+/*
 #include <stropts.h>
+*/
```
  
make tools
----------
$ cd src/utils  
$ make  
$ PATH=$PATH:~/proj/pintos/src/utils  


make pintos kernel
------------------
Please sure the dir 'utils' have setted in $PATH  

$ patch src/threads/Make.vars  
```diff
-SIMULATOR = --bochs   <<< if using simulator-bochs
+SIMULATOR = --qemu    <<< if using simulator-qemu
```
  
$ cd ~/proj/pinto/src/threads  
$ make  

run
---
Please sure the dir 'utils' have setted in $PATH  

$ pintos -h  
pintos run alarm-multiple, which passes the arguments 'run alarm-multiple' to the Pintos kernel.  
In these arguments, run instructs the kernel to run a test and alarm-multiple is the test to run.  

$ pintos run alarm-multiple  

you can use the -v option to disable X output, -q let simulator auto-quit:  
$ pintos -v -- run alarm-multiple.  
$ pintos -v -- -q run alarm-single   <<< -v no-gui, -q after run and quit  
$ pintos -vk -T 60 --bochs -- -q run alarm-single    <<< -T 60 timeout 60s and kill simulator if it's not quit.  

TESTS
-----
Please sure the dir 'utils' have setted in $PATH  

```bash
$ cd src/threads/build

$ make check
$ make check VERBOSE=1    <<< output more detail info

### select different simulator
$ make check SIMULATOR=--qemu
$ make check SIMULATOR=--bochs

### run specific test
$ touch ../../lib/debug.c
$ make tests/threads/alarm-multiple.result
$ cat tests/threads/alarm-multiple.result
PASS
```

REF
---
[code ref1](https://code.google.com/p/jawpintos/source/checkout)  
[code ref project-1](https://github.com/ryantimwilson/Pintos-Project-1)  
[code ref project-2](https://github.com/ryantimwilson/Pintos-Project-2)  
[code ref project-3](https://github.com/ryantimwilson/Pintos-Project-3)  
[code ref project-4](https://github.com/ryantimwilson/Pintos-Project-4)  

