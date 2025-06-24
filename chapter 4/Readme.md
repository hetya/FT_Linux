Create the file architecture (as root):

```Shell
mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}

for i in bin lib sbin; do
  ln -sv usr/$i $LFS/$i
done

case $(uname -m) in
  x86_64) mkdir -pv $LFS/lib64 ;;
esac
```

Create a directory for the cross compiler:

```Shell
mkdir -pv $LFS/tools
```

Create a lfs group and a lfs user:
-k /dev/null tell the process to not copy file like .bashrc, .profile it will create an empty home for the user

```Shell
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
```

Set the password of the user:

```Shell
passwd lfs
```

Grant the lfs user ownership of $LFS folders

```Shell
chown -v lfs $LFS/{usr{,/*},var,etc,tools}
case $(uname -m) in
  x86_64) chown -v lfs $LFS/lib64 ;;
esac
```

Log as lfs

```Shell
su - lfs
```

Create .bash_profile

```Shell
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

Create the .bashrc

```Shell
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```

Since /etc/bash.bashrc can modify the environment we will rename the file so it is not loaded during the creation:

```Shell
[ ! -e /etc/bash.bashrc ] || mv -v /etc/bash.bashrc /etc/bash.bashrc.NOUSE
```

We can speedup the makefile build by using all the cores we have, to build in parallel:
You can use `make -j<NB_OF_CORE>` OR `export MAKEFLAGS=-j<NB_OF_CORE>`

```Shell
cat >> ~/.bashrc << "EOF"
export MAKEFLAGS=-j$(nproc)
EOF
```

To be sure that all modification are take, we will source `.bashrc`

```Shell
source ~/.bash_profile
```
