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

# OpenBSD

Install the following packages:

pkg_add git
pkg_add ksh93         # OpenBSD ksh is still _not_ sufficient
pkg_add motif         # OpenBSD 6.0+
pkg_add tcl-8.6.6p0   # or whatever tcl v8.6 version is available
pkg_add gmake autoconf automake libtool bison opensp

Download and build CDE

OpenBSD support is broken in release 2.2.4, but fixed in git 2.2.4a and later versions.

Use the git clone command here:

On most platform you can use HTTPS:

git clone https://git.code.sf.net/p/cdesktopenv/code cdesktopenv-code

If that doesn't work (for instance some BSD distros) , use the native git protocol instead

git clone git://git.code.sf.net/p/cdesktopenv/code cdesktopenv-code

Or download the latest source release:

Note: The source archive will become out of date. When you want the latest code, clone the git repository.
Build

Version 2.4.0a and newer (autoconf)

For the BSD's, you must use gmake, and you must specify the location of the TCL install directory (the below example assumes TCL v8.6). This should be the directory that contains the tclConfig.sh file. On OpenBSD specifically, you must set some environment variables and add some C/CXXFLAGS to the configure step:

$ export AUTOMAKE_VERSION=x.xx # whatever versions you have installed
$ export AUTOCONF_VERSION=x.xx
$ export LIBRARY_PATH="/usr/local/lib"
$ ./autogen.sh
$ ./configure --with-tcl=/usr/local/lib/tcl/tcl8.6 MAKE="gmake" \
        CFLAGS="-I/usr/local/include" CXXFLAGS="-I/usr/local/include"
$ gmake
$ doas gmake install

Note: Documentation does not build as of version 2.4.0a. This is an issue with the new build system and is currently being worked on.

Version 2.4.0 and earlier (imake)

cd cdesktopenv-code/cde
make World
admin/IntegTools/dbTools/installCDE -s `pwd`

Configure system

add to /etc/rc.conf.local

shlib_dirs="/usr/dt/lib"

You will need to add entries for your hostname (NOT just localhost) in /etc/hosts

127.0.0.1    myname    myname.my.domain
::1          myname    myname.my.domain

and create an init script for rpc.cmsd, for example:

daemon="/usr/dt/bin/rpc.cmsd &"

. /etc/rc.d/rc.subr

pexp="rpc.cmsd: ${daemon}${daemon_flags:+ ${daemon_flags}} \[listener\].*"

rc_reload() {
        ${daemon} ${daemon_flags} -t && pkill -HUP -xf "${pexp}"
}

rc_cmd $1

Save as /etc/rc.d/cmsd

doas rcctl enable cmsd
doas rcctl enable inetd # not sure this does anything but might be needed
doas rcctl enable portmap # needed to get past dthello

Reboot
Start and test CDE

You can now start dtlogin manager as root (not recommended):

/usr/dt/bin/dtlogin -nodaemon

Alternatively, you can start an X session as a normal user:

startx /usr/dt/bin/Xsession

Cleanup and post-install

To automatically source your .profile when opening a terminal in CDE uncomment the last line in your user's .dtprofile file:

DTSOURCEPROFILE=true

Starting dtlogin at boot (not recommended for security reasons)

To automatically go into the CDE graphical login at system startup add the following line to /etc/rc.conf.local

xenodm_flags=NO
pkg_scripts=dtlogin

And create a new file /etc/rc.d/dtlogin with the following contents:

#!/bin/sh

daemon="/usr/dt/bin/dtlogin"

. /etc/rc.d/rc.subr

rc_reload=NO

if [ -n "${INRC}" ]; then
# on boot: make sure we don't hang in _rc_wait
_rc_wait() {
    return 0
}
# on boot: wait for ttys to be initialized
rc_start() {
    ( local i=0
    while ! pgrep -qf "^/usr/libexec/getty "; do
        sleep 1
        [ $((i++)) -ge 10 ] && return 1
    done
    ${rcexec} "${daemon} ${daemon_flags}" ) &
}
fi

rc_cmd $1

Then make it executable:

chmod +x /etc/rc.d/dtlogin

Using xenodm (recommended)

if you don't like dtlogin, you can start CDE via xenodm:

ln -s /usr/dt/bin/Xsession ~/.xsession

Or to make it default for all newly created users:

ln -s /usr/dt/bin/Xsession /etc/skel/.xsession

Man pages

To make man work:

cp /etc/examples/man.conf /etc/man.conf

Then add the following to /etc/man.conf

manpath /usr/dt/share/man

You will need to logout/login for the change to take effect.
Screenshots


An older screenshot.


A newer screenshot with some recommended customizations:

    Lines in dtwm's en_US.utf-8 app defaults have been uncommented so the front panel uses the correct color
    The mailer is old and broken, so its icon has been replaced with the Internet application group
    Dtinfo does not currently build, so its icon has been replaced with the Help Manager (this was the default long ago)
