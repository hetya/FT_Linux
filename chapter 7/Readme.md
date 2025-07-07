Change the ownership of $LFS/\* folders to the root user (So the folder belongs to an existing user id on the new machine)
If the ownership is not change a later created user on the new machine could have the same id and so gaining ownership

```Shell
chown --from lfs -R root:root $LFS/{usr,lib,var,etc,bin,sbin,tools}
case $(uname -m) in
  x86_64) chown --from lfs -R root:root $LFS/lib64 ;;
esac
```

Create virtual filesystem that will be used by application running in the userspace:

```Shell
mkdir -pv $LFS/{dev,proc,sys,run}
```

Bind mount $LFS/dev

```Shell
mount -v --bind /dev $LFS/dev
```

Mount the other:

```Shell
mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run
```

/dev/shm is a temporary filesystem (typically tmpfs) used for POSIX shared memory. Programs can create shared memory segments there for fast IPC (inter-process communication).:

```Shell
if [ -h $LFS/dev/shm ]; then
  install -v -d -m 1777 $LFS$(realpath /dev/shm)
else
  mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
fi
```

Enter our temporary system with chroot

```Shell
chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin     \
    MAKEFLAGS="-j$(nproc)"      \
    TESTSUITEFLAGS="-j$(nproc)" \
    /bin/bash --login
```

Create the full directory structure:

```Shell
mkdir -pv /{boot,home,mnt,opt,srv}
mkdir -pv /etc/{opt,sysconfig}
mkdir -pv /lib/firmware
mkdir -pv /media/{floppy,cdrom}
mkdir -pv /usr/{,local/}{include,src}
mkdir -pv /usr/lib/locale
mkdir -pv /usr/local/{bin,lib,sbin}
mkdir -pv /usr/{,local/}share/{color,dict,doc,info,locale,man}
mkdir -pv /usr/{,local/}share/{misc,terminfo,zoneinfo}
mkdir -pv /usr/{,local/}share/man/man{1..8}
mkdir -pv /var/{cache,local,log,mail,opt,spool}
mkdir -pv /var/lib/{color,misc,locate}

ln -sfv /run /var/run
ln -sfv /run/lock /var/lock

install -dv -m 0750 /root
install -dv -m 1777 /tmp /var/tmp
```

Create needed symlink:
Historically, Linux maintained a list of the mounted file systems in the file /etc/mtab. Modern kernels maintain this list internally and expose it to the user via the /proc filesystem. To satisfy utilities that expect to find /etc/mtab, create the following symbolic link:

```Shell
ln -sv /proc/self/mounts /etc/mtab
```

Create basic/essential file :
/etc/hosts:

```Shell
cat > /etc/hosts << EOF
127.0.0.1  localhost $(hostname)
::1        localhost
EOF
```

/etc/passwd:

```Shell
cat > /etc/passwd << "EOF"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/dev/null:/usr/bin/false
daemon:x:6:6:Daemon User:/dev/null:/usr/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/run/dbus:/usr/bin/false
uuidd:x:80:80:UUID Generation Daemon User:/dev/null:/usr/bin/false
nobody:x:65534:65534:Unprivileged User:/dev/null:/usr/bin/false
EOF
```

/etc/group:

```Shell
cat > /etc/group << "EOF"
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tape:x:4:
tty:x:5:
daemon:x:6:
floppy:x:7:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
input:x:24:
mail:x:34:
kvm:x:61:
uuidd:x:80:
wheel:x:97:
users:x:999:
nogroup:x:65534:
EOF
```

Since some test need a user, we will add it now (it will be deleted after all test has been passed):

```Shell
echo "tester:x:101:101::/home/tester:/bin/bash" >> /etc/passwd
echo "tester:x:101:" >> /etc/group
install -o tester -d /home/tester
```

Starting a new shell since /etc/passwd and /etc/group are now created (this will remove : "I have no name!" from the prompt):

```Shell
exec /usr/bin/bash --login
```

Create log file, logs will not auto create the file if it doesn't exist:

```Shell
touch /var/log/{btmp,lastlog,faillog,wtmp}
chgrp -v utmp /var/log/lastlog
chmod -v 664  /var/log/lastlog
chmod -v 600  /var/log/btmp
```

Install packages ...
Util-linux:
Configure :
Maybe a problem because w haven't set runstatedir (not possible in this version)

Since unshare and setns are already defined we will add preprocessor to patch it
May be check with --disable-static instead

```Shell
sed -i '/# define UTIL_LINUX_NAMESPACE_H/a\
#define HAVE_UNSHARE 1\n#define HAVE_SETNS 1' include/namespace.h
sed -i '/#include "closestream.h"/a #define HAVE_PRLIMIT' sys-utils/prlimit.c
```

```Shell
./configure --libdir=/usr/lib     \
            --disable-chfn-chsh  \
            --disable-login      \
            --disable-nologin    \
            --disable-su         \
            --disable-sulogin \
            --disable-setpriv    \
            --disable-runuser    \
            --disable-pylibmount \
            --disable-static     \
            --without-python     \
			ADJTIME_PATH=/var/lib/hwclock/adjtime   \
            --docdir=/usr/share/doc/util-linux-2.33.1

            # --without-libcrypt \
```

```Shell
make runstatedir=/run
```

Install:

```Shell
make install runstatedir=/run
```

Remove the documentation file to save space and prevent them to end in the final system:

```Shell
rm -rf /usr/share/{info,man,doc}/*
```

On a modern Linux system, the libtool .la files are only useful for libltdl. No libraries in LFS are loaded by libltdl.
So we will remove them as well

```Shell
find /usr/{lib,libexec} -name \*.la -delete
```

The /tools folder (1GB) is not needed anymore, so we will remove it :

```Shell
rm -rf /tools
```

Since here we have a state where all essential program are installed, we will do a backup since if we have a problem later it may be simpler to just remove all files and rebuilding from here.

Exit the chroot environment:

```Shell
exit
```

Unmount the virtual file system:

```Shell
mountpoint -q $LFS/dev/shm && umount $LFS/dev/shm
umount $LFS/dev/pts
umount $LFS/{sys,proc,run,dev}
```

Backup all file this can take a while (10 min on relatively fast system)

```Shell
cd $LFS
tar -cJpf $HOME/lfs-temp-tools-12.3.tar.xz .
```
