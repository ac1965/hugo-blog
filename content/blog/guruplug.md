---
title: "Guruplug"
date: 2010-04-14T23:31:13+09:00
tags: ["guruplug"]
draft: false
---
# 1日目

## guruplug が届く前に環境を作っておこう

クロスコンパイル環境とか、ルートファイルシステムなど。
[GuruPlug Wiki](http://www.plugcomputer.org/plugwiki/index.php/GuruPlug") に加筆したものも含めて、めもです。

## 参考にしたポイント

* [cross-development](http://www.gentoo.org/proj/en/base/embedded/cross-development.xml)
* [Building Kernel](http://computingplugs.com/index.php/Building_a_custom_kernel)
* [GuruPlug](http://www.plugcomputer.org/plugwiki/index.php/GuruPlug)

クロスコンパイルは [GNU Hurd](http://www.gnu.org/software/hurd/) で遊んで以来、やっていなかったけど、Gentoo は楽ちんですね。

## クロスコンパイル環境のセットアップ

``` bash
# emerge -av portage-utils crossdev
# crossdev armv5tel-softfloat-linux-gnueabi
```

## U-Boot とカーネルセットアップ

これは Wiki に書いてあるとおり。
最後にmkimageをPATHの通っているところにコピーしておく。カーネル(uImage)でこけるので。

``` bash
# git clone git://git.denx.de/u-boot-marvell.git u-boot-marvell.git
# cd u-boot-marvell.git
# git checkout -b testing origin/testing
# make mrproper
# make guruplug_config
# make u-boot.kwb CROSS_COMPILE=armv5tel-softfloat-linux-gnueabi-
# cp tools/mkimage /usr/bin
# wget http://www.plugcomputer.org/plugwiki/images/8/81/Guruplug-patchset.tar.bz2
# tar xfj Guruplug-patchset.tar.bz2
# git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-2.6.33.y.git
# for p in guruplug-patchset/*; do patch -p1 -E -d linux-2.6.33.y.git < $p; done
# cd linux-2.6.33.y.git
# make CROSS_COMPILE=armv5tel-softfloat-linux-gnueabi- ARCH=arm clean
# make CROSS_COMPILE=armv5tel-softfloat-linux-gnueabi- ARCH=arm guruplug_defconfig
# make CROSS_COMPILE=armv5tel-softfloat-linux-gnueabi- ARCH=arm uImage
# make CROSS_COMPILE=armv5tel-softfloat-linux-gnueabi- ARCH=arm modules
# make CROSS_COMPILE=armv5tel-softfloat-linux-gnueabi- ARCH=arm modules_install INSTALL_MOD_PATH=/usr/armv5tel-softfloat-linux-gnueabi
```

## mtd-toolsのセットアップ

git repository(git://git.infradead.org/mtd-utils.git)を使ってみる。
ubifs でルートファイルシステムを構築する準備です。

``` bash
# ebuild /usr/portage/sys-fs/mtd-utils/mtd-utils-99999999.ebuild compile
# ebuild /usr/portage/sys-fs/mtd-utils/mtd-utils-99999999.ebuild merge
# ebuild /usr/portage/sys-fs/mtd-utils/mtd-utils-99999999.ebuild clean
```

## ルートファイルシステムのセットアップ

crossdev で作ったクロスコンパイラで基本システム(stage1)を構築する。
ところが、`cpio' でコンパイル失敗している。

眠いので今日はここまで。

``` bash
# cd /usr/armv5tel-softfloat-linux-gnueabi
# mkdir etc
# cd etc
# ln -s /usr/portage/profiles/default/linux/arm/10.0 make.profile
# cp /tmp/make.conf-arm make.conf
# armv5tel-softfloat-linux-gnueabi-emerge -av @system
```

* 参考：/usr/armv5tel-softfloat-linux-gnueabi/etc/make.conf
僕の環境なので同じである必要はない。
ROOT="..." の default は "/" なので指定しておかないとoverwrite されるので注意
あと、クロスコンパイルの場合 CBUILD は必要。

```
# NEVER change this
CHOST="armv5tel-softfloat-linux-gnueabi"
CBUILD=i686-pc-linux-gnu
CHOST=${CHOST}
ARCH="arm"
ROOT=/usr/${CHOST}/
ACCEPT_KEYWORDS="arm ~arm"

# You can edit these.
USE="-X -cdr -kde -gnome -qt -gtk -fortran -opengl -quicktime -cups \
        -tcpd -apache -gpm -slang nls cjk \
        -alsa -xscreensaver \
        ssl userlocales apache2 sasl nptl nptlonly threads unicode utf8 \
        bzip2 crypt loop-aes mysql mysqli sqlite xmlrpc bash-completion \
        idn glibc-omitfp \
        dvd dvdr cdda cddb \
        mp3 vorbis wavepack wma mpeg ogg rtsp x264 win32codecs libv412 v4l2 v4l \
        wifi bluetooth curl git -dso subversion lzma"

APACHE2_MODULES=" \
        auth auth_basic authz_user authz_host authn_dbd authz_dbd dbd \
        alias filter deflate mime mime_magic expires headers unique_id \
        vhost_alias rewrite log_config logio env setenvif autoindex dir \
        proxy proxy_http dav dav_fs \
        "
APACHE2_MPMS="worker"

# Think twice about editing these.
MAKEOPTS="-j5"
CFLAGS="-Os -march=armv5te -pipe -fomit-frame-pointer"
CXXFLAGS="${CFLAGS}"

#FEATURES="distcc parallel-fetch userfetch userpriv"
FEATURES="parallel-fetch userfetch userpriv buildpkg"

PORTAGE_TMPDIR="/var/tmp/cross"
BUILD_PREFIX="${PORTAGE_TMPDIR}"

GENTOO_MIRRORS="http://gentoo.gg3.net/"

ACCEPT_LICENSE="*"
```

# 2日目

## cpioのコンパイルエラーは、stat(2) の引数が指定されていなかっただけだった。

野良ebuild を狭んで、おけー。

* src/filtypes.h の patch
```
diff -urN cpio-2.11-/src/filetypes.h cpio-2.11/src/filetypes.h
--- cpio-2.11-/src/filetypes.h	2010-04-15 18:03:16.639694128 +0900
+++ cpio-2.11/src/filetypes.h	2010-04-15 18:04:03.545691921 +0900
@@ -82,4 +82,4 @@
 #define lstat stat
 #endif
 int lstat ();
-int stat ();
+int stat (const char *path, struct stat *buf);
```

##  野良ebuild cpio-2.11-r1.ebuild
```
# Copyright 1999-2010 Gentoo Foundation
# Distributed under the terms of the GNU General Public License v2
# $Header: /var/cvsroot/gentoo-x86/app-arch/cpio/cpio-2.11.ebuild,v 1.1 2010/03/15 07:52:11 vapier Exp $

inherit eutils

EAPI="2"

DESCRIPTION="A file archival tool which can also read and write tar files"
HOMEPAGE="http://www.gnu.org/software/cpio/cpio.html"
SRC_URI="mirror://gnu/cpio/${P}.tar.bz2"

LICENSE="GPL-3"
SLOT="0"
KEYWORDS="~alpha ~amd64 ~arm ~hppa ~ia64 ~m68k ~mips ~ppc ~ppc64 ~s390 ~sh ~sparc ~sparc-fbsd ~x86 ~x86-fbsd"
IUSE="nls"

src_configure() {
	econf \
		$(use_enable nls) \
		--bindir=/bin \
		--with-rmt=/usr/sbin/rmt \
		|| die
}

src_install() {
	emake install DESTDIR="${D}" || die
	dodoc ChangeLog NEWS README
	rm "${D}"/usr/share/man/man1/mt.1 || die
	rmdir "${D}"/usr/libexec || die
}

src_unpack() {
	unpack ${A}
	cd "${S}"
	epatch "${FILESDIR}"/${P}-filetypes.patch
}
```

# 3日目

## クロス環境での stage1 がエラーで出来ていない状態

ものが届いていないのでまぁゆったりとな。
あとで、distcc でホストマシンでコンパイルできるように
クロスコンパイル環境を作り直そう。

クロス環境消して、ホスト環境とバージョンを合せておこうっと。

SYSROOT を検索する必要があるため binutils-2.19.51.0.12
より上にする。
[http://bugs.gentoo.org/275666](http://bugs.gentoo.org/275666)

``` bash
# crossdev --clean armv5tel-softfloat-linux-gnueabi
# emerge -p binutils linux-headers gcc glibc
....
[ebuild   R   ] sys-devel/binutils-2.20.1
[ebuild   R   ] sys-kernel/linux-headers-2.6.29
[ebuild   R   ] sys-devel/gcc-4.3.3-r2
[ebuild   R   ] sys-libs/glibc-2.9_p20081201-r2
...
# crossdev -t armv5tel-softfloat-linux-gnueabi \
#    --b 2.20.1 \
#    --k 2.6.29 \
#    --g 4.3.3-r2 \
#    --l 2.9_p20081201-r2 \
#    --ex-gdb
```

# 4日目

## クロスコンパイル環境で ARM(armv5tel)用の gentoo stage1 ができたので動作確認をした

正しくは perl と Linux-PAM はクロスコンパイルができなかったので、
own で作ることになるが。

## [http://www.gentoo.org/proj/en/base/embedded/handbook/?part=1&chap=5](http://www.gentoo.org/proj/en/base/embedded/handbook/?part=1&chap=5)
QEMU の user-mode + binfmt + chroot で動作確認をした。

``` bash
# export PS1="(`uname -m`:\W) "
(i684:/ ) USE="static" emerge --buildpkg --oneshot qemu-user
(i684:/ ) ROOT=/usr/armv5tel-softfloat-linux-gnueabi emerge --usepkgonly qemu-user
(i684:/ ) cat qemu-wrapper.c
#include <stdio.h>
#include <unistd.h>

int main(int argc, char **argv, char **envp) {
    char *newargv[argc + 3];

    newargv[0] = argv[0];
    newargv[1] = "-cpu";
    newargv[2] = "cortex-a8";

   memcpy(&newargv[3], &argv[1], sizeof(*argv) * (argc - 1));
   newargv[argc + 2] = NULL;
   return execve("/usr/bin/qemu-arm", newargv, envp);
}
(i684:/ ) gcc -static qemu-wrapper.c -o qemu-wrapper
(i684:/ ) cp qemu-wrapper /usr/armv5tel-softfloat-linux-gnueabi
(i684:/ ) [ -d /proc/sys/fs/binfmt_misc ] || modprobe binfmt_misc
(i684:/ ) [ -f /proc/sys/fs/binfmt_misc/register ] || \
mount binfmt_misc -t binfmt_misc /proc/sys/fs/binfmt_misc
(i684:/ ) echo ':arm:M::\x7fELF\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x28\x00:\xff\xff\xff\xff\xff\xff\xff\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/qemu-wrapper:' > /proc/sys/fs/binfmt_misc/register
(i684:/ ) cat /proc/sys/fs/binfmt_misc/arm
enabled
interpreter /qemu-wrapper
flags:
offset 0
magic 7f454c4601010100000000000000000002002800
mask ffffffffffffff00fffffffffffffffffeffffff
(i684:/ ) [ -d /usr/armv5tel-softfloat-linux-gnueabi/usr/portage ] || \
install -d /usr/armv5tel-softfloat-linux-gnueabi/usr/portage
(i684:/ ) [ -d /usr/armv5tel-softfloat-linux-gnueabi/proc ] || \
install -d /usr/armv5tel-softfloat-linux-gnueabi/proc
(i684:/ ) [ -d /usr/armv5tel-softfloat-linux-gnueabi/sys ] || \
install -d /usr/armv5tel-softfloat-linux-gnueabi/sys
(i684:/ ) mount --bind /usr/portage /usr/armv5tel-softfloat-linux-gnueabi/usr/portage
(i684:/ ) mount --bind /proc /usr/armv5tel-softfloat-linux-gnueabi/proc
(i684:/ ) mount --bind /sys /usr/armv5tel-softfloat-linux-gnueabi/sys
(i684:/ ) chroot /usr/armv5tel-softfloat-linux-gnueab /bin/busybox mdev -s
(i684:/ ) chroot /usr/armv5tel-softfloat-linux-gnueab /bin/bash --login
# export PS1="(`uname -m`:chroot:\W) "
(arm:chroot: /) env-update
(arm:chroot: /) exit
(i684:/ ) umount /usr/armv5tel-softfloat-linux-gnueabi/sys
(i684:/ ) umount /usr/armv5tel-softfloat-linux-gnueabi/proc
(i684:/ ) umount /usr/armv5tel-softfloat-linux-gnueabi/usr/portage
(i684:/ ) umount /proc/sys/fs/binfmt_misc
```
