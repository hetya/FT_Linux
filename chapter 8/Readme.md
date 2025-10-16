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

Create the `inputrc` file (options) for `readline` :

```Shell
cat > /etc/inputrc << "EOF"
# Begin /etc/inputrc
# Modified by Chris Lynn <roryo@roryo.dynup.net>

# Allow the command prompt to wrap to the next line
set horizontal-scroll-mode Off

# Enable 8-bit input
set meta-flag On
set input-meta On

# Turns off 8th bit stripping
set convert-meta Off

# Keep the 8th bit for display
set output-meta On

# none, visible or audible
set bell-style none

# All of the following map the escape sequence of the value
# contained in the 1st argument to the readline specific functions
"\eOd": backward-word
"\eOc": forward-word

# for linux console
"\e[1~": beginning-of-line
"\e[4~": end-of-line
"\e[5~": beginning-of-history
"\e[6~": end-of-history
"\e[3~": delete-char
"\e[2~": quoted-insert

# for xterm
"\eOH": beginning-of-line
"\eOF": end-of-line

# for Konsole
"\e[H": beginning-of-line
"\e[F": end-of-line

# End /etc/inputrc
EOF
```

Create the `etc/shells` file, applications use it to determine whether a shell is valid.

```Shell
cat > /etc/shells << "EOF"
# Begin /etc/shells

/bin/sh
/bin/bash

# End /etc/shells
EOF
```
