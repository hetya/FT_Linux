For customization purpose and for practicality I will add some package:
All of the listed package are part of the [blfs](https://www.linuxfromscratch.org/blfs/view/12.4/index.html) page

1. We will download the `blfs-bootscripts` that are needed for multiple package

```Shell
wget --directory-prefix=$LFS/sources https://anduin.linuxfromscratch.org/BLFS/blfs-bootscripts/blfs-bootscripts-20250225.tar.xz
```

2. Install [`dhcpcd`](https://www.linuxfromscratch.org/blfs/view/svn/basicnet/dhcpcd.html):
   To launch the daemon at startup go to the `blfs-bootscripts` folder and exec :

   ```Shell
   make install-service-dhcpcd
   ```

   Since wan't to use a static ip any more we will change the `/etc/sysconfig/ifconfig.enp0s3` to use the ip provided by `dhcpcd`

   ```Shell
   cd /etc/sysconfig
   cat > ifconfig.enp0s3 << "EOF"
   ONBOOT=yes
   IFACE=enp0s3
   SERVICE=dhcpcd
   EOF
   ```

3. Openssh :
   In the base host environment download the needed file for the installation:

   ```Shell
   wget --directory-prefix=$LFS/sources  https://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-10.2p1.tar.gz
   ```

   Go to the `blfs-bootscripts` folder and exec :

   ```Shell
   make install-sshd
   ```

4. Installing [`fcron`](https://www.linuxfromscratch.org/blfs/view/svn/general/fcron.html)
   Here Same as sshd we will use `blfs-bootscripts` to launch the daemon at startup

   ```Shell
   make install-fcron
   ```

5. `Curl` and `wget` + dependency:

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

   1. [libunistring](https://www.linuxfromscratch.org/blfs/view/svn/general/libunistring.html)
   2. [libidn2](https://www.linuxfromscratch.org/blfs/view/svn/general/libidn2.html)
   3. [libpsl](https://www.linuxfromscratch.org/blfs/view/svn/basicnet/libpsl.html)

   4. [libtasn1](https://www.linuxfromscratch.org/blfs/view/svn/general/libtasn1.html)
   5. [p11-kit](https://www.linuxfromscratch.org/blfs/view/svn/postlfs/p11-kit.html)
   6. [make-ca](https://www.linuxfromscratch.org/blfs/view/svn/postlfs/make-ca.html)

   7. [Wget](https://www.linuxfromscratch.org/blfs/view/svn/basicnet/wget.html)
   8. [cURL](https://www.linuxfromscratch.org/blfs/view/svn/basicnet/curl.html)

6. Install [`git`](https://www.linuxfromscratch.org/blfs/view/svn/general/git.html)

7. Install [`zsh`](https://www.linuxfromscratch.org/blfs/view/svn/postlfs/zsh.html)
