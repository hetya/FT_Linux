Enter the chroot env if not already in

```Shell
mount -v --bind /dev $LFS/dev
mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run
if [ -h $LFS/dev/shm ]; then
  install -v -d -m 1777 $LFS$(realpath /dev/shm)
else
  mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
fi
chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin     \
    MAKEFLAGS="-j$(nproc)"      \
    TESTSUITEFLAGS="-j$(nproc)" \
    /bin/bash --login
```

Follow the instruction for each package:

Glibc

Configure:

```Shell
../configure --prefix=/usr                            \
             --disable-werror                         \
             --enable-kernel=3.2                      \
             --enable-stack-protector=strong          \
             --disable-nscd                           \
             libc_cv_slibdir=/usr/lib
```

May be we can set --enable-kernel to 3.2 in chapter 5 too
Change from the current version () :
--enable-kernel=3.2
This option tells the build system that this Glibc may be used with kernels as old as 3.2 . This means generating workarounds in case a system call introduced in a later version cannot be used.

inetuils : libls.sh, may fail in the initial chroot environment

VIM : If an X Window System is going to be installed on the LFS system, it may be necessary to recompile Vim after installing X.

procps-ng use the version 3.3.15 may be we can just put --disable-pidwait on v4

```Shell
./configure --prefix=/usr \
 --docdir=/usr/share/doc/procps-ng-3.3.15 \
 --disable-static \
 --disable-kill \
 --enable-watch8bit
```

Util linux :

```Shell
./configure --bindir=/usr/bin     \
            --libdir=/usr/lib     \
            --sbindir=/usr/sbin   \
            --disable-chfn-chsh   \
            --disable-login       \
            --disable-nologin     \
            --disable-su          \
            --disable-setpriv     \
            --disable-runuser     \
            --disable-pylibmount  \
            --disable-liblastlog2 \
            --disable-static      \
            --without-python      \
            --without-systemd     \
            --without-systemdsystemunitdir        \
            ADJTIME_PATH=/var/lib/hwclock/adjtime \
            --docdir=/usr/share/doc/util-linux-2.33.1
```

```Shell
make runstatedir=/run
```

Install:

```Shell
make runstatedir=/run install
```

I have skip the stripping part of some libraries

Cleannup some files created for test purpose:

```Shell
rm -rf /tmp/{*,.*}
```

Delete .la file in /usr/lib and /usr/libexec not needed in our case

```Shell
find /usr/lib /usr/libexec -name \*.la -delete
```

Remove our compiler build in chapter 6 and 7 that is still partialy installed

```Shell
find /usr -depth -name $(uname -m)-lfs-linux-gnu\* | xargs rm -rf
```

Delete the test user:

```Shell
userdel -r tester
```
