---
title: VirtualBox 管理虚拟磁盘 压缩磁盘镜像
date: 2024-01-20
updated: 2023-01-20
tags:
- 虚拟化
- VirtualBox
categories:
- 教程
- VirtualBox
---

下载[SDelete](https://learn.microsoft.com/zh-cn/sysinternals/downloads/sdelete)。
{% note info %}
接下来的命令默认你已经位于解压了 SDelete 的目录内，或者将其目录加入了环境变量，并且使用 cmd。  
如果你使用 Powershell 或者其它终端，请自行加上绝对路径/相对路径。
{% endnote %}

在虚拟机内打开 cmd，执行命令：
```cmd
sdelete -c -z C:
```
若你需要压缩其它盘符，更改 `C:` 即可。详细的操作说明请使用 `sdelete -?` 查看。

等待程序正常运行完成后，关闭虚拟机。

在宿主机上，使用 `VBoxManage` 工具[^1]，压缩已被全部设为`0x00`的空白空间：
```bash
VBoxManage modifymedium disk windows_11.vdi --compact
```
{% note warning %}
对于 Linux 用户，`vboxmanage` 与 `VBoxManage` 是两个不同的命令，请注意大小写。
若要处理旧的兼容格式，请考虑 `vboxmanage`。
{% endnote %}

<!-- more -->

另请参阅：
[^1]: [VirtualBox Mannual - Chapter 8. VBoxManage Section 8.31. VBoxManage modifymedium](https://www.virtualbox.org/manual/ch08.html#vboxmanage-modifymedium)
  **（更新）** 链接的锚点地址有变动(`#vboxmanage-modifyvdi` -> `#vboxmanage-modifymedium`)
- [VirtualBox Forum - Could not get the storage format of the medium](https://forums.virtualbox.org/viewtopic.php?t=75308)
- [StackExchange - VirtualBox - how to free up unused VDI disk place?](https://superuser.com/questions/388733/virtualbox-how-to-free-up-unused-vdi-disk-place)
- [StackExchange - How to compact VirtualBox's VDI file size?](https://superuser.com/questions/529149/how-to-compact-virtualboxs-vdi-file-size)
- [VirtualBox - ArchWiki](https://wiki.archlinux.org/title/VirtualBox)
- [Microsoft Learn / Sysinternals / SDelete](https://learn.microsoft.com/zh-cn/sysinternals/downloads/sdelete)