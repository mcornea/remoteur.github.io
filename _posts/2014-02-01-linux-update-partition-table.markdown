---
layout: post
status: publish
published: true
title: Linux reload partition table
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 208
wordpress_url: https://remote-lab.net/?p=208
date: '2014-02-01 22:14:56 +0000'
date_gmt: '2014-02-01 20:14:56 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>Below are the steps you can use in order to be able to create a filesystem on a newly created partition (vda3 in our example) without reboot:</p>
<p><code lang="c[notools]">root@lfs:~# fdisk -l<br />
Disk /dev/vda: 16.1 GB, 16106127360 bytes<br />
16 heads, 63 sectors/track, 31207 cylinders, total 31457280 sectors<br />
Units = sectors of 1 * 512 = 512 bytes<br />
Sector size (logical/physical): 512 bytes / 512 bytes<br />
I/O size (minimum/optimal): 512 bytes / 512 bytes<br />
Disk identifier: 0x0003d859<br />
   Device Boot      Start         End      Blocks   Id  System<br />
/dev/vda1            2048     1953791      975872   82  Linux swap / Solaris<br />
/dev/vda2   *     1953792    11718655     4882432   83  Linux<br />
/dev/vda3        11718656    31457279     9869312   83  Linux<br />
root@lfs:~# ls /dev/vda<br />
vda   vda1  vda2<br />
root@lfs:~# aptitude install parted<br />
root@lfs:~# partprobe<br />
root@lfs:~# ls /dev/vda<br />
vda   vda1  vda2  vda3<br />
root@lfs:~# mkfs.ext4 /dev/vda3</code></p>