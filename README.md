# Building-the-Common-Desktop-Environment

#Freebsd

Supported Versions

These instructions have been successfully tested on :

    FreeBSD 12.3-RELEASE aarch64 (Raspberry Pi3)
    FreeBSD 12.3-RELEASE x86_64
    FreeBSD 12.4-RELEASE aarch64 (Raspberry Pi3)
    FreeBSD 12.4-RELEASE x86_64
    FreeBSD 12-STABLE x86_64
    FreeBSD 13.1 x86_64
    FreeBSD 13-STABLE x86_64
    FreeBSD 14.0-CURRENT x86_64

First steps

Install FreeBSD 12-RELEASE or FreeBSD 13-RELEASE and install/update your ports tree.

freebsd-update fetch
freebsd-update install

reboot.

Complete install documentation can be found in the FreeBSD Handbook.

If you plan to install the dependencies from source, fetch the current ports tree and build the portmaster utility:

portsnap fetch extract update
cd /usr/ports/ports-mgmt/portmaster
make -DBATCH install clean

Install packages
Install packages from source

the source packages can be built and installed using the following sequence

portmaster -C -D --no-confirm -y \
  x11/xorg \
  devel/git \
  converters/iconv \
  shells/ksh93 \
  x11-toolkits/open-motif \
  lang/tcl86
  textproc/opensp

If you don't want to wade through the many configuration steps and build with the standard configuration instead, add the -G parameter to portmaster.

More information about ports and packages can be found in the FreeBSD Handbook Ports chapter. More information about X11 in FreeBSD can also be found in the X11 Chapter in the FreeBSD Handbook.
Install binary packages

To install precompiled binary packages, make the following call

pkg install cde xorg

If you wish to track the CDE development branch,

pkg install cde-devel xorg

The development branch is updated monthly though it may be updated more frequently.
Edit /etc/rc.conf

Add to /etc/rc.conf

rpcbind_enable="YES"
inetd_enable="YES"

Reboot
Clone or download the source code

Use the git clone command here:

On most platform you can use HTTPS:

git clone https://git.code.sf.net/p/cdesktopenv/code cdesktopenv-code

If that doesn't work (for instance some BSD distros) , use the native git protocol instead

git clone git://git.code.sf.net/p/cdesktopenv/code cdesktopenv-code

Or download the latest source release:

Note: The source archive will become out of date. When you want the latest code, clone the git repository.
Build CDE

Version 2.5.0 and newer (autoconf)

For the BSD's, you must use gmake, and you must specify the location of the TCL install directory (the below example assumes TCL v8.6).

$ ./autogen.sh
$ ./configure --with-tcl=/usr/local/lib/tcl8.6 MAKE="gmake"
$ gmake
$ sudo gmake install

Version 2.4.0 and earlier (imake) - deprecated

cd cdesktopenv-code/cde
make World
admin/IntegTools/dbTools/installCDE -s `pwd`

Start CDE

You can now start CDE login manager as root:

/usr/dt/bin/dtlogin -nodaemon

Alternatively, you can start an X session as a normal user:

startx /usr/dt/bin/Xsession

Install dtlogin as Login Manager

Switch to the CDE build directory and copy and enable the rc-file

cp contrib/rc/freebsd/dtlogin /usr/local/etc/rc.d/
echo 'dtlogin_enable="YES"' >> /etc/rc.conf
echo "allowed_users=anybody" > /usr/local/etc/X11/Xwrapper.config

reboot