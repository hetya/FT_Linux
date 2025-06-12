docs from : http://www.linuxfromscratch.org/lfs/view/stable/index.html

On a host machine (machine that will build the new OS)

Install required package to build the OS:

```Shell
sudo apt install bash-3.2 binutils-2.13.1 bison-2.7 coreutils-8.1 diffutils-2.8.1 findutils-4.2.31 gawk-4.0.1 gcc-5.2 grep-2.5.1a gzip-1.3.12 m4-1.4.10 make-4.0 patch-2.5.4 perl-5.8.8 python-3.4 sed-4.1.5 tar-1.22 texinfo-5.0 xz-5.0.0
OR :
sudo apt install bash binutils bison coreutils diffutils findutils gawk gcc grep gzip m4 make patch perl python3 sed tar texinfo xz-utils
```

Run the version_check.sh script

If sh is not a alias of bash :

```Shell
sudo ln -s bash /bin/sh.bash
sudo mv /bin/sh.bash /bin/sh

IF YOU WANT TO RESTORE IT BACK:

sudo ln -s dash /bin/sh.dash
sudo mv /bin/sh.dash /bin/sh
```

Since we are on a OS that already have a swap we will reuse our current swap. So we don't have to recreate it

Let's list our currents partition:

```Shell
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0aba473a

Device     Boot    Start      End  Sectors  Size Id Type
/dev/sda1  *        2048 39942143 39940096   19G 83 Linux
/dev/sda2       39944190 41940991  1996802  975M  5 Extended
/dev/sda5       39944192 41940991  1996800  975M 82 Linux swap / Solaris


Disk /dev/sdb: 23 GiB, 24696061952 bytes, 48234496 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

We see we have two disk /dev/sda for the host and /dev/sdb for the new OS.
/dev/sdb is 23 G we will use 20G for the /, 1G for the /boot and the rest for the swap

So first we will create the / partition:

```Shell
sudo fdisk /dev/sdb

...
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-48234495, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-48234495, default 48234495): +20G

Created a new partition 1 of type 'Linux' and of size 20 GiB.
```

Now we can write the modification and check that the disk has been partitioned:

```Shell
sudo fdisk -l
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0aba473a

Device     Boot    Start      End  Sectors  Size Id Type
/dev/sda1  *        2048 39942143 39940096   19G 83 Linux
/dev/sda2       39944190 41940991  1996802  975M  5 Extended
/dev/sda5       39944192 41940991  1996800  975M 82 Linux swap / Solaris


Disk /dev/sdb: 23 GiB, 24696061952 bytes, 48234496 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xbf3f1cfe

Device     Boot    Start      End  Sectors Size Id Type
/dev/sdb1           2048 41945087 41943040  20G 83 Linux
/dev/sdb2       41945088 44042239  2097152   1G 83 Linux
/dev/sdb3       44042240 48234495  4192256   2G 83 Linux
```

Now we will create the filesystem for the /boot and /:

For the / we will use ext4

```Shell
mkfs -v -t ext4 /dev/
```

And for the /boot ext2:

```Shell
sudo mkfs -v -t ext2 /dev/sdb2
```
