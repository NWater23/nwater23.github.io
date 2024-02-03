---
title: 在TTY或SSH上使用GPG，即没有GUI界面时
date: 2023-07-14 11:56:13
updated: 2023-07-14 11:56:13
tags:
- Linux
- TTY
- SSH
- 归档和压缩
- GnuPG
categories:
- 教程
- Linux
---

当通过ssh远程连接到服务器，或者在纯字符界面的tty上操作时，总之就是在无GUI情况下操作时。  
使用gpg签名（或者通过`git commit -S`等调用gpg），会报错：
```
gpg: signing failed: Inappropriate ioctl for device
gpg: [stdin]: clear-sign failed: Inappropriate ioctl for device
```

原因是无法调用默认的`/usr/bin/pinentry-gtk-2`，如果你没安装GTK的话也会出现这个报错。

根据Arch Wiki介绍[^1]，修改`.bashrc`即可（zsh同理）：
```bash .bashrc
export GPG_TTY=$(tty)
```

<!-- more -->

[^1]: (2023) [ArchWiki: GnuPG§9.12 Invalid IPC response and Inappropriate ioctl for device](https://wiki.archlinux.org/title/GnuPG#Invalid_IPC_response_and_Inappropriate_ioctl_for_device)
    ```wiki
        === Invalid IPC response and Inappropriate ioctl for device ===

        The default pinentry program is {{ic|/usr/bin/pinentry-gtk-2}}. If {{Pkg|gtk2}} is unavailable, pinentry falls back to {{ic|/usr/bin/pinentry-curses}} and causes signing to fail:

            gpg: signing failed: Inappropriate ioctl for device
            gpg: [stdin]: clear-sign failed: Inappropriate ioctl for device

        You need to set the {{ic|GPG_TTY}} environment variable for the pinentry programs {{ic|/usr/bin/pinentry-tty}} and {{ic|/usr/bin/pinentry-curses}}.

            $ export GPG_TTY=$(tty)
    ```