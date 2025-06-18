As `root` create a folder for the package source:

```Shell
mkdir -v $LFS/sources
```

Make this directory writable and sticky. “Sticky” means that even if multiple users have write permission on a directory, only the owner of a file can delete the file within a sticky directory. The following command will enable the write and sticky modes:

```Shell
chmod -v a+wt $LFS/sources
```

Download all package and patches using the wget-list-sysv file

```Shell
wget --input-file=wget-list-sysv --continue --directory-prefix=$LFS/sources
```

Since expat is not found, using the given url in the file we will download it ourselves,
Since it is also mandatory to have a 4.x version for the kernel we will download it to :

<!-- github : https://github.com/torvalds/linux/releases/tag/v4.20 -->

```Shell
wget --directory-prefix=$LFS/sources https://github.com/libexpat/libexpat/releases/download/R_2_6_4/expat-2.6.4.tar.xz
wget --directory-prefix=$LFS/sources https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.20.tar.xz
```

Check the checksum:

```Shell
pushd $LFS/sources
md5sum -c md5sums
popd
```

We will remove the 6.x kernel version that we will not use

```Shell
rm $LFS/sources/linux-6.13.4.tar.xz
```
