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

# NetBSD

First steps

Install NetBSD/i386 or NetBSD/amd64.
(versions from 5.1.2 to 7.0 have been tested)
NetBSD 9.2

For NetBSD 9.2, go to the following link from a user on the updated packages and instructions needed. When the autoconf version of CDE (v2.4.0a+) get's closer to a release, the current instructions will be mostly replaced with the contents of this post. NOTE: the patches described in that post should not be needed in current master as of 12/17/21.

NOTE: for documentation to build properly, you should increase the size of the /tmp partition. By default in 9.2, it's 10MB which will cause NodeParser to fail due to running out of space. I increased this to 256MB on a 4GB system. This resolved the NodeParser failure.

https://sourceforge.net/p/cdesktopenv/wiki/NetBSD/#c4bd

The following instructions are dated and will be updated at a later date.
Install packages

The build expects the following packages under /usr/pkg:

    git
    ast-ksh
    freetype2
    font-adobe-75dpi
    font-adobe-100dpi
    fontconfig
    motif (for CDE version 2.3 or newer)
    pam-pwauth_suid
    tcl
    autoconf
    automake
    libtool
    gmake
    opensp
    sessreg
    lmdb

All other required packages are installed as dependencies.

Note
If you are using pkgsrc to install these packages, please note that ast-ksh does not build in NetBSD 7.0 and it doesn't look like it is going to be fixed anytime soon. Check your PKG_PATH environment variable and make sure it contains:

ftp://ftp.netbsd.org/pub/pkgsrc/packages/NetBSD/i386/6.1/All/ (for the i386 version) or
ftp://ftp.netbsd.org/pub/pkgsrc/packages/NetBSD/amd64/6.0/All/ (for the x64 version)

then install it by issuing pkg_add -v ast-ksh

As of February 2019 ast-ksh builds again with NetBSD 7.2, but fails for NetBSD 8.0. The 6.x binaries do however work fine in NetBSD 8.0.

As ast-ksh has been updated in 2020. It builds fine inthe current release NetBSD 9.
Build and Install Motif

NOTE: For versions of CDE 2.3 or later, this step can be skipped. Use the OS packaged version of Motif.

Build Motif from source! This has to be done as root.

Download pkgsrc from http://ftp.netbsd.org/pub/pkgsrc/stable/pkgsrc.tar.gz
and install it under /usr

Change into motif directory:
cd /usr/pkgsrc/x11/motif

Build and install motif:

  make
  make install

Note
add the two lines

.include "../../x11/libXp/buildlink3.mk"
.include "../../x11/printproto/buildlink3.mk"

at the end of /usr/pkgsrc/mk/motif.buildlink3.mk or other Motif dependent packages will not build properly.
Edit system files
For CDE version 2.3 or later

Add to /etc/rc.conf

  rpcbind=YES             rpcbind_flags="-l"

For CDE versions earlier than 2.3, you must enable insecure mode (-i):

Add to /etc/rc.conf

  rpcbind=YES             rpcbind_flags="-l -i"

In addition, for versions of CDE prior to 2.3, you must add your hostname (uname -n) to the localhost line in /etc/hosts or ttsession will fail to start.
Add fontpath to Xserver

Add to "Files"-section of /etc/X11/xorg.conf

    FontPath     "/usr/pkg/share/fonts/X11/100dpi/"
    FontPath     "/usr/pkg/share/fonts/X11/75dpi/"

Reboot
Clone the git repository or download the latest source code

Use the git clone command here:

On most platform you can use HTTPS:

git clone https://git.code.sf.net/p/cdesktopenv/code cdesktopenv-code

If that doesn't work (for instance some BSD distros) , use the native git protocol instead

git clone git://git.code.sf.net/p/cdesktopenv/code cdesktopenv-code

Or download the latest source release:

Note: The source archive will become out of date. When you want the latest code, clone the git repository.
Make symlinks

NOTE: For CDE version 2.3 or later, this step can be skipped. However, as of Nov. 28th 2018 it is still required if you use the modular x-org from the pkgsrc system

If you use the native X11 installation that came with NetBSD, create the following symlinks

  cd cdesktopenv-code/cde
  mkdir -p imports/x11/include
  ln -s /usr/X11R7/include/X11 imports/x11/include/
  ln -s /usr/pkg/include/Xm imports/x11/include/
  ln -s /usr/pkg/include/fontconfig imports/x11/include/
  ln -s /usr/pkg/include/freetype2/ft2build.h imports/x11/include/

If you installed modular-xorg from the pkgsrc tree, create the following symlinks instead:

  ln -s /usr/pkg /usr/X11R7
  cd cdesktopenv-code/cde
  mkdir -p imports/x11/include
  ln -s /usr/X11R7/include/X11 imports/x11/include/
  ln -s /usr/X11R7/include/Xm imports/x11/include/
  ln -s /usr/X11R7/include/fontconfig imports/x11/include/
  ln -s /usr/X11R7/include/freetype2/ft2build.h imports/x11/include/

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

You can now start the CDE login manager as root:

  /usr/dt/bin/dtlogin -nodaemon

Alternatively, you can start an X session as a normal user:

  startx /usr/dt/bin/Xsession
