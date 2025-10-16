Create udev rules :
Start by generating the rules :

```Shell
bash /usr/lib/udev/init-net-rules.sh
```

You can see them at `/etc/udev/rules.d/70-persistent-net.rules` :

```Shell
cat /etc/udev/rules.d/70-persistent-net.rules
```

All udev rules are made up of several keywords, separated by commas and optional whitespace. Here are the keywords, and an explanation of each one:

- SUBSYSTEM=="net" - This tells udev to ignore devices that are not network cards.

- ACTION=="add" - This tells udev to ignore this rule for a uevent that isn't an add ("remove" and "change" uevents also happen, but don't need to rename network interfaces).

- DRIVERS=="?\*" - This exists so that udev will ignore VLAN or bridge sub-interfaces (because these sub-interfaces do not have drivers). These sub-interfaces are skipped because the name that would be assigned would collide with the parent devices.

- ATTR{address} - The value of this keyword is the NIC's MAC address.

- ATTR{type}=="1" - This ensures the rule only matches the primary interface in the case of certain wireless drivers which create multiple virtual interfaces. The secondary interfaces are skipped for the same reason that VLAN and bridge sub-interfaces are skipped: there would be a name collision otherwise.

- NAME - The value of this keyword is the name that udev will assign to this interface.

Even if the custom udev rule file is created, udev may still assign one or more alternative names for a NIC. To prevent the issue where udev will rename a NIC with a name already assigned for another NIC. We will override the default config with an empty alternative assignment policy

```Shell
sed -e '/^AlternativeNamesPolicy/s/=.*$/=/'  \
       /usr/lib/udev/network/99-default.link \
     > /etc/udev/network/99-default.link
```

Network interface configuration file :
Here enp0s3 is the name of my interface. The name of the inteface must be at the end of the file and for the `IFACE` variable:

```Shell
cd /etc/sysconfig/
cat > ifconfig.enp0s3 << "EOF"
ONBOOT=yes
IFACE=enp0s3
SERVICE=ipv4-static
IP=192.168.1.2
GATEWAY=192.168.1.1
PREFIX=24
BROADCAST=192.168.1.255
EOF
```

creating the `/etc/resolv.conf` file, it contain a list of DNS server for DNS resolution:

```Shell
cat > /etc/resolv.conf << "EOF"
# Begin /etc/resolv.conf

# domain <Your Domain Name>
# search
nameserver 8.8.8.8
nameserver 8.8.4.4
# End /etc/resolv.conf
EOF
```

Set the hostname:

```Shell
echo "lfs-vm" > /etc/hostname
```

`/etc/hosts` :

```Shell
cat > /etc/hosts << "EOF"
# Begin /etc/hosts

127.0.0.1  localhost lfs-vm
::1       localhost ip6-localhost ip6-loopback
ff02::1   ip6-allnodes
ff02::2   ip6-allrouters

# End /etc/hosts
EOF
```

System V bootscript:

```Shell
cat > /etc/inittab << "EOF"
# Begin /etc/inittab

id:3:initdefault:

si::sysinit:/etc/rc.d/init.d/rc S

l0:0:wait:/etc/rc.d/init.d/rc 0
l1:S1:wait:/etc/rc.d/init.d/rc 1
l2:2:wait:/etc/rc.d/init.d/rc 2
l3:3:wait:/etc/rc.d/init.d/rc 3
l4:4:wait:/etc/rc.d/init.d/rc 4
l5:5:wait:/etc/rc.d/init.d/rc 5
l6:6:wait:/etc/rc.d/init.d/rc 6

ca:12345:ctrlaltdel:/sbin/shutdown -t1 -a -r now

su:S06:once:/sbin/sulogin
s1:1:respawn:/sbin/sulogin

1:2345:respawn:/sbin/agetty --noclear tty1 9600
2:2345:respawn:/sbin/agetty tty2 9600
3:2345:respawn:/sbin/agetty tty3 9600
4:2345:respawn:/sbin/agetty tty4 9600
5:2345:respawn:/sbin/agetty tty5 9600
6:2345:respawn:/sbin/agetty tty6 9600

# End /etc/inittab
EOF
```

Config system clock:

```Shell
cat > /etc/sysconfig/clock << "EOF"
# Begin /etc/sysconfig/clock

UTC=1

# Set this to any options you might need to give to hwclock,
# such as machine hardware clock type for Alphas.
CLOCKPARAMS=

# End /etc/sysconfig/clock
EOF
```

Configure Linux console :

```Shell
cat > /etc/sysconfig/console << "EOF"
# Begin /etc/sysconfig/console

UNICODE="1"
FONT="Lat2-Terminus16"

# End /etc/sysconfig/console
EOF
```

Print the list of locales supported :

```Shell
locale -a
```

```Shell
(lfs chroot) root:/# LC_ALL=en_US locale charmap
ISO-8859-1
```

Test the locale, print the language name, the character encoding used by the locale, the local currency, and the prefix to dial before the telephone number. If any of the commands fail your locale was either not installed or not supported by the default installation of Glibc.

```Shell
LC_ALL=en_US locale language
LC_ALL=en_US locale charmap
LC_ALL=en_US locale int_curr_symbol
LC_ALL=en_US locale int_prefix
```

Create the `/etc/profile` once the proper locale settings have been determined, but set the C.UTF-8 locale if running in the Linux console (to prevent programs from outputting characters that the Linux console is unable to render):

```Shell
cat > /etc/profile << "EOF"
# Begin /etc/profile

for i in $(locale); do
  unset ${i%=*}
done

if [[ "$TERM" = linux ]]; then
  export LANG=C.UTF-8
else
  export LANG=en_US.ISO-8859-1
fi

# End /etc/profile
EOF
```
