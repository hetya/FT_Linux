For each package starting form `$LFS/sources` you will have to :

- extract the the tar.gz file unsing tar
- `cd` in to the created folder
- building the package
- go back to `$LFS/sources`

Ex : For `binutils`:

```Shell
tar -xf binutils-2.44.tar.xz
cd binutils-2.44
```

Command to build this package:

```Shell
mkdir -v build
cd       build
time { ../configure --prefix=$LFS/tools \
             --with-sysroot=$LFS \
             --target=$LFS_TGT   \
             --disable-nls       \
             --enable-gprofng=no \
             --disable-werror    \
             --enable-new-dtags  \
             --enable-default-hash-style=gnu && \
			 make && \
			 make install;}
```

Here I have wrap the build process in a time command to determine a SBU (Standard Build Unit),
it will be useful to determined the time other package take to be build.
This will not be needed for other packages.

Here we have :

```Shell
real	1m54.464s
user	5m55.358s
sys	0m50.916s
```

So for us a SBU is ~ 1m54

Other packag to build:
[GCC](https://www.linuxfromscratch.org/lfs/view/stable/chapter05/gcc-pass1.html)

LINUX Kernel :
Based on http://fr.linuxfromscratch.org/view/lfs-8.4-fr/chapter05/linux-headers.html and https://www.linuxfromscratch.org/lfs/view/stable/chapter05/linux-headers.html:

```Shell
make INSTALL_HDR_PATH=dest headers_install
make mrproper
MAY BE Use The Next line not sure:
find dest/include -type f ! -name '*.h' -delete
cp -rv dest/include $LFS/usr
```

[Glibc](https://www.linuxfromscratch.org/lfs/view/stable/chapter05/glibc.html)

Here may be sln is not mandatory even so the docs say it
Change the option `--enable-kernel=5.4` to `--enable-kernel=4.20` since we are using a 4.20 kernel

```Shell
../configure                             \
      --prefix=/usr                      \
      --host=$LFS_TGT                    \
      --build=$(../scripts/config.guess) \
      --enable-kernel=4.20                \
      --with-headers=$LFS/usr/include    \
      --disable-nscd                     \
      libc_cv_slibdir=/usr/lib
```
