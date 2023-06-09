---
title: zip 解压乱码：代码页 cp936
date: 2023-06-09 18:45:20
updated: 2023-06-09 18:45:20
tags:
- 归档和压缩
  - zip
    - unzip 解压
- Linux
  - 作为环境
- Windows
  - 作为来源
- 编码字符集
  - 东亚
    - 中文
        - cp936
        - GB2312
        - GBK
        - GB18030
- 问题与修复
  - 乱码
categories:
- 教程
  - Linux
    - 实用工具
---

Windows 使用的简体中文编码与 UTF-8 不兼容会导致乱码。[^debian-bug-report]

使用 `-O cp936` 选项[^1] [^2] [^3] [^4] [^5]
```bash
$ unzip -O cp936 <zip file>
```

也可以给 `-O` 选项使用参数：
- `GBK`
- `GB2312`
- `GB18030`

<!-- more -->

## 备选项 [^5]

安装 7zip 和 convmv 工具：
```bash
sudo apt-get install p7zip-full convmv
```

```bash
LANG=C
7z x -o{Directory} <zip file>
convmv -f gbk -t utf8 --notest -r <output dir>
# 或者先 cd 到输出文件夹
convmv -f gbk -t utf8 --notest  ./*
```

[^1]: (2014) [Linux 下 zip 文件解压乱码如何解决？](https://www.zhihu.com/question/20523036/answer/31746415)
[^2]: (2020) [Ubuntu压缩及解压文件简介](https://zhuanlan.zhihu.com/p/143846450)
[^3]: (2019) [【技巧】linux环境下，用unzip解压zip文件时，若解压文件中存在中文，会出现中文乱码问题](https://blog.csdn.net/qq_36441393/article/details/102639066)
[^4]: (2021) [linux下解压zip乱码问题的解决（unzip）](https://www.cnblogs.com/tesila/p/14982479.html)
[^5]: (2016) [Linux文件乱码](https://www.findhao.net/easycoding/1605)
[^debian-bug-report]: (2003) （英文）[#197427 - unzip: chinese filenames unwrapped on unix wrongly](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=197427)