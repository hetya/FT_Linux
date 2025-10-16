Create the `fstab` for partition to be mounted :

Check needed partition with `fdisk -l`:

```Shell
Device Boot Start End Sectors Size Id Type
/dev/sdb1 2048 41945087 41943040 20G 83 Linux ===> / ext4
/dev/sdb2 41945088 44042239 2097152 1G 83 Linux ===> /boot ext2
/dev/sdb3 44042240 48234495 4192256 2G 83 Linux ===> swap
```

Here we have :
/dev/sdb1 ===> / , ext4
/dev/sdb2 ===> /boot, ext2
/dev/sdb3 ===> swap

So we can create the `fstab`:

```Shell
cat > /etc/fstab << "EOF"
# Begin /etc/fstab

# file system  mount-point    type     options             dump  fsck
#                                                                order

/dev/sdb1      /              ext4     defaults            1     1
/dev/sdb2      /boot          ext2     defaults            0     2
/dev/sdb3      swap           swap     pri=1               0     0
proc           /proc          proc     nosuid,noexec,nodev 0     0
sysfs          /sys           sysfs    nosuid,noexec,nodev 0     0
devpts         /dev/pts       devpts   gid=5,mode=620      0     0
tmpfs          /run           tmpfs    defaults            0     0
devtmpfs       /dev           devtmpfs mode=0755,nosuid    0     0
tmpfs          /dev/shm       tmpfs    nosuid,nodev        0     0
cgroup2        /sys/fs/cgroup cgroup2  nosuid,noexec,nodev 0     0

# End /etc/fstab
EOF
```

Setting up the kernel:

To setup the kernel check the documentation

Since we want to change the version name, in the `make menuconfig` go General Setup => config local version and add `-<custom_version>`

The path to the kernel image may vary depending on the platform being used. The filename below can be changed to suit your taste, but the stem of the filename should be vmlinuz to be compatible with the automatic setup of the boot process described in the next section. Here you can change `custom`

```Shell
cp -iv arch/x86/boot/bzImage /boot/vmlinuz-6.16.1-<custom>
```

Copy the symbol file for the kernel:

```Shell
cp -iv System.map /boot/System.map-6.16.1
```

Copy the .config file produced by the make `menuconfig` that contain the kernel config

```Shell
cp -iv .config /boot/config-6.16.1
```

Copy the Documentation:

```Shell
cp -r Documentation -T /usr/share/doc/linux-6.16.1
```

Since we will keep the kernel source file we will change the source folder permission to be sure it is owned by the root user.
And we will change the location of the folder to be in /usr/src

```Shell
chown -R 0:0 /sources/linux-6.16.1
cd ..
mv
```

Create boot process using grub:

Here we will create a boot loader on the sdb drive since it is the drive where LFS is installed :

```Shell
grub-install /dev/sdb
```

Create the grub config file :
Do not forget to change `<custom_version>` for the kernel version

```Shell
cat > /boot/grub/grub.cfg << "EOF"
# Begin /boot/grub/grub.cfg
set default=0
set timeout=5

insmod part_gpt
insmod ext2
set root=(hd1,2)
set gfxpayload=1024x768x32

menuentry "GNU/Linux, Linux 6.16.1-lfs-12.4" {
        linux   /vmlinuz-6.16.1-<custom_version> root=/dev/sdb1 ro
}
EOF
```

For future reference we will create a file `/etc/lfs-release` that contain the lfs version

```Shell
echo 12.4 > /etc/lfs-release
```

Now we will create two file that describe our system
The first one shows the status of your new system with respect to the Linux Standards Base (LSB):

```Shell
cat > /etc/lsb-release << "EOF"
DISTRIB_ID="Linux From Scratch"
DISTRIB_RELEASE="12.4"
DISTRIB_CODENAME="<your name here>"
DISTRIB_DESCRIPTION="Linux From Scratch"
EOF
```

The second one contains roughly the same information, and is used by systemd and some graphical desktop environments:

```Shell
cat > /etc/os-release << "EOF"
NAME="Linux From Scratch"
VERSION="12.4"
ID=lfs
PRETTY_NAME="Linux From Scratch 12.4"
VERSION_CODENAME="<your name here>"
HOME_URL="https://www.linuxfromscratch.org/lfs/"
RELEASE_TYPE="stable"
EOF
```
