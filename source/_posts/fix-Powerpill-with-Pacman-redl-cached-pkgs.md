---
title: fix Powerpill with Pacman re-download cached packages
date: 2024-02-07 04:37:06
updated: 2024-02-07 04:37:06
tags:
- Linux
- Arch Linux
- Pacman
- Powerpill
categories:
- 教程
- Linux
---

There for me, a strange thing happened:

1. Powerpill download package files(`*.pkg.tar.zst`) into CacheDir[^1]
2. Pacman download package files again

[^1]: Defined in `/etc/pacman.conf`

How it happen?

<!-- more -->

## Possible 1: Signature check failed

Pacman does check every pkg file according to default `pacman.conf`. For safety reason you should not change this options.

## Possible 2: Aira2c does downloading to `.part` file

Check the `/etc/pacman.conf`:
```conf /etc/pacman.conf
[options]
#XferCommand = /usr/bin/curl -L -C - -f -o %o %u
#XferCommand = /usr/bin/wget --passive-ftp -c -O %o %u
XferCommand = /usr/local/bin/aria2c --allow-overwrite=true --continue=true --file-allocation=none --log-level=error --max-tries=5 --max-connection-per-server=5 -s 1024 --lowest-speed-limit=200k --max-file-not-found=5 --min-split-size=4M --no-conf --remote-time=true --summary-interval=60 --timeout=5 --dir=/ --out %o %u
```

Look at the `XferCommand`'s value, when pacman downloading pkg file e.g. `http://mirrors.tuna.tsinghua.edu.cn/archlinux/core/os/x86_64/pacman-6.0.2-8-x86_64.pkg.tar.zst`, the command executed is:
```bash
/usr/local/bin/aria2c \
  --allow-overwrite=true --continue=true --file-allocation=none \
  --log-level=error --max-tries=5 --max-connection-per-server=5 \
  -s 1024 --lowest-speed-limit=200k --max-file-not-found=5 --min-split-size=4M \
  --no-conf --remote-time=true --summary-interval=60 --timeout=5 \
  --dir=/ \
  --out /var/cache/pacman/pkg/pacman-6.0.2-8-x86_64.pkg.tar.zst \
  http://mirrors.tuna.tsinghua.edu.cn/archlinux/core/os/x86_64/pacman-6.0.2-8-x86_64.pkg.tar.zst
```

You'll see it writes to file `//var/cache/pacman/pkg/pacman-6.0.2-8-x86_64.pkg.tar.zst.part`, the prefix `//` and suffix `.part` it not we expected.

This configure file will statisfiy we requirements:[^3]
```conf /etc/pacman.conf
[options]
XferCommand = echo Downloading %u ... && /usr/bin/aria2c --conf-path=/root/.aria2/pacman-aria2.conf %u
```

```conf /root/.aria2/pacman-aria2.conf
# error handling
timeout=60
connect-timeout=30
max-tries=5
retry-wait=10
max-file-not-found=1

# downloading
split=3
max-connection-per-server=3
min-split-size=1M
max-concurrent-downloads=1
file-allocation=none
remote-time=true
conditional-get=true
no-netrc=true

# resuming
continue=true
allow-overwrite=true
always-resume=false

# proxy
#http-proxy=127.0.0.1:8080
#https-proxy=127.0.0.1:8080
#ftp-proxy=127.0.0.1:8080

# console
#quiet=true
console-log-level=warn
summary-interval=0
#enable-color=false
#human-readable=false
#show-console-readout=false
#truncate-console-readout=false

# logging
log-level=warn
log=/var/log/pacman-aria2.log
```

[^3]: [pacman and aria2 - better configuration?](https://bbs.archlinux.org/post.php?tid=192072&qid=1491879)


## Possible 3: miss time stamp / local file is older than server

If server(Mirror server) did not pass the file update time to aria2c/wget/curl, then your client will download again. It behave as if as the local file is older than server.

If no time stamp provides, it will be default to `1 January 1970 00:00:00 UTC` (Unix Timestamp `0`).

To check if a mirror site provides file update time or not yet at all, run follow command:[^2]
```bash
curl --silent --head "$(pacman -Sp pacman --cachedir /dev/null)" | grep Last-Modified
```
Command result should be like `Last-Modified: Wed, 20 Sep 2023 23:37:12 GMT`.
- If `grep` exited with return value `1`, it means no match line found, this only happens when server response does not contain a time stamp.

[^2]: https://bbs.archlinux.org/post.php?tid=269644&qid=1992948

## End

This is the first time I write blog in English.
