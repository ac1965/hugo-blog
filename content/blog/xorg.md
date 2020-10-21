---
title: "Xorg"
date: 2010-06-01T07:39:03+09:00
draft: false
tags: ["linux", "xorg"]
---
GuruPlugがなかなか屆かないので、Funtoo と xorg-1.8 の整理をした。

* boot-update
grubの設定支援かな？
[boot-update](http://www.funtoo.org/en/funtoo/core/boot/)
で記述されている代物です。
grub-1.97+にしてみたが multiboot
にはまだ調整されていないみたい。
次のように記述して(/etc/boot.conf)、
`boot-update'
とタイプインするればよい。楽チンだけど、
"Backtrack 4"
は利用側の root-fs を無理やり埋め込まれているので、手直しが必要なのだ。
まぁ、/boot/grub/grub.cfg
を手修正すればよい。

```
boot {
	path /boot
	generate grub
	default "Funtoo Linux"
	timeout 3
}

"Funtoo Linux" {
	kernel /kernel-genkernel-x86[-v]
	initrd initramfs-genkernel-x86[-v]
	params += crypt_root=/dev/sdc2 root_keydev=/dev/sde1 root_key=/keyfile
	params += dolvm real_root=/dev/mapper/LVG-root
	params += i915.modeset=1 fbcon=map:1
	params += ramdisk=8192 quiet init=/linuxrc
}

"Backtrack 4" {
	kernel /bt4/vmlinuz
	initrd /bt4/initrd.gz
	params += BOOT=casper boot=casper persistent rw quiet
	params += real_root=auto
}
```

* xorg-1.8
けっこう放置していたのでトライしてみた。

1.  MASKを外す

``` bash
echo 'x11-base/xorg-server' >> /etc/portage/package.unmask
echo 'x11-base/xorg-server * ~* **' >> /etc/portage/package.keywords/x11-base
```

3.  emege xorg-server
USE="udev -hal" で emerge したけど、キーボードとマウスが認識していない。予想はついていたので、sshd
をあげて別端末から
pkill
した。

[ここを参考](http://body0r.wordpress.com/2010/04/16/xorg-udev-toggle/)する。

8.  emerge udev
MASKを外して、udev をアップデート。

``` bash
echo 'sys-fs/udev' >> /etc/portage/package.unmask
echo 'sys-fs/udev * ~* **' >> /etc/portage/package.keywords/sys-fs
emerge -u udev
```

11.  udevルールの追加
/usr/share/X11/xorg.conf.d
が system config なので、/etc/X11/xorg.conf.d を掘って
キーボードとマウスの設定を追加。


おけー。

``` bash
# cat /etc/X11/xorg.conf.d/10-keyboard.conf
Section "InputClass"
        Identifier "Keyboard"
        Driver "evdev"
        MatchIsKeyboard "on"
        Option "xkbmodel" "jp106"
        Option "xkblayout" "jp"
EndSection

# cat /etc/X11/xorg.conf.d/20-synaptics.conf
Section "InputClass"
	Identifier "Touchpad"
	Driver "synaptics"
	MatchIsTouchpad "on"
	Option "SHMConfig" "true"
	Option "MinSpeed" "0.20"
	Option "MaxSpeed" "0.60"
	Option "AccelFactor" "0.020"
	Option "HorizEdgeScroll" "true"
	Option "HorizScrollDelta" "100"
	Option "VertEdgeScroll" "true"
	Option "VertScrollDelta" "100"
	Option "TapButton1" "1"
EndSection
```