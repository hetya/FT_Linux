For customization purpose and for practicality I will add some package:

Openssh :
In the base host environment download the needed file for the installation:

```Shell
wget --directory-prefix=$LFS/sources  https://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-10.2p1.tar.gz
```

I use the blfs bootscript to launch `sshd` at startup

```Shell
wget --directory-prefix=$LFS/sources https://anduin.linuxfromscratch.org/BLFS/blfs-bootscripts/blfs-bootscripts-20250225.tar.xz
```

Installing `fcron`
[FCRON](https://www.linuxfromscratch.org/blfs/view/svn/general/fcron.html)
Use the blfs bootscript configuration download before

`Curl` and `wget` + dependency:

```Shell
wget --directory-prefix=$LFS/sources https://ftp.gnu.org/gnu/libunistring/libunistring-1.4.1.tar.xz
wget --directory-prefix=$LFS/sources https://ftp.gnu.org/gnu/libidn/libidn2-2.3.8.tar.gz
wget --directory-prefix=$LFS/sources https://github.com/rockdaboot/libpsl/releases/download/0.21.5/libpsl-0.21.5.tar.gz
wget --directory-prefix=$LFS/sources https://ftp.gnu.org/gnu/libtasn1/libtasn1-4.20.0.tar.gz
wget --directory-prefix=$LFS/sources https://github.com/p11-glue/p11-kit/releases/download/0.25.10/p11-kit-0.25.10.tar.xz
wget --directory-prefix=$LFS/sources https://github.com/lfs-book/make-ca/archive/v1.16.1/make-ca-1.16.1.tar.gz
wget --directory-prefix=$LFS/sources https://ftp.gnu.org/gnu/wget/wget-1.25.0.tar.gz
wget --directory-prefix=$LFS/sources https://curl.se/download/curl-8.16.0.tar.xz
```

We will install package in this order:
[libunistring](https://www.linuxfromscratch.org/blfs/view/svn/general/libunistring.html)
[libidn2](https://www.linuxfromscratch.org/blfs/view/svn/general/libidn2.html)
[libpsl](https://www.linuxfromscratch.org/blfs/view/svn/basicnet/libpsl.html)

[libtasn1](https://www.linuxfromscratch.org/blfs/view/svn/general/libtasn1.html)
[p11-kit](https://www.linuxfromscratch.org/blfs/view/svn/postlfs/p11-kit.html)
[make-ca](https://www.linuxfromscratch.org/blfs/view/svn/postlfs/make-ca.html)

[Wget](https://www.linuxfromscratch.org/blfs/view/svn/basicnet/wget.html)
[cURL](https://www.linuxfromscratch.org/blfs/view/svn/basicnet/curl.html)

Install `git`: [git](https://www.linuxfromscratch.org/blfs/view/svn/general/git.html)

Install `dhcpcd`:

Install `zsh`:
