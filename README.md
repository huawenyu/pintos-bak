pintos
======
[manual](http://web.stanford.edu/class/cs140/projects/pintos/pintos.html)  
  
The Pintos study edition by Huawen Yu.  

install on fedora 21 with bochs|qemu
====================================

[ref](https://pintosiiith.wordpress.com/2012/09/13/install-pintos-with-qemu/)  
Please change the $HOME '/home/wilson' to your own home directory.

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
$ sudo yum install libX11-devel   <<< Xlib.h
$ sudo yum install libXrandr-devel
$ sudo yum install xorg-x11-server-devel
$ make
$ sudo make install
```

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
you can use the -v option to disable X output:  
$ pintos -v -- run alarm-multiple.

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
