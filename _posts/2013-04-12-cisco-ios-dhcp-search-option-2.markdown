---
layout: post
status: publish
published: true
title: Cisco IOS DHCP search option
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 193
wordpress_url: https://remote-lab.net/?p=193
date: '2013-04-12 23:50:25 +0000'
date_gmt: '2013-04-12 21:50:25 +0000'
categories:
- Linux
- IOS
- Python
tags: []
comments: []
---
<p>I was looking today for a way to set my home Cisco router to push multiple domains in the DHCP search list. I found this very useful post written by Jonathan Perkin: <a href="http://www.perkin.org.uk/posts/serving-multiple-dns-search-domains-in-ios-dhcp.html">http://www.perkin.org.uk/posts/serving-multiple-dns-search-domains-in-ios-dhcp.html</a> where he explains how we can achieve this by using Cisco’s hex sequence in the search option. He also provides a nice python script that converts the domain ASCII string to hex sequence.<br />
Thank you very much, Jonathan, very useful info.</p>
<p><code lang="python[notools]">marius@remoteur:~>>> cat ios-search.py<br />
#!/usr/bin/python<br />
import sys<br />
hexlist = []<br />
for domain in sys.argv[1:]:<br />
    for part in domain.split("."):<br />
        hexlist.append("%02x" % len(part))<br />
        for c in part:<br />
            hexlist.append(c.encode("hex"))<br />
    hexlist.append("00")<br />
print "".join([(".%s" % (x) if i and not i % 2 else x) \<br />
    for i, x in enumerate(hexlist)])<br />
</code></p>
<p><code lang="c[notools]">marius@remoteur:~>>> ./ios-search.py domain.net domain.org domain.com<br />
0664.6f6d.6169.6e03.6e65.7400.0664.6f6d.6169.6e03.6f72.6700.0664.6f6d.6169.6e03.636f.6d00<br />
</code></p>
<p><code lang="c[notools]">c881.remote-lab.net#show run | s ip dhcp pool<br />
ip dhcp pool HomeLeases<br />
   network 10.0.0.0 255.255.255.0<br />
   default-router 10.0.0.1<br />
   domain-name remote-lab.net<br />
   dns-server 8.8.8.8 8.8.4.4<br />
   option 119 hex 0664.6f6d.6169.6e03.6e65.7400.0664.6f6d.6169.6e03.6f72.6700.0664.6f6d.6169.6e03.636f.6d00<br />
</code></p>
<p><code lang="c[notools]">marius@remoteur:~>>> grep search /etc/resolv.conf<br />
search remote-lab.net domain.net domain.org domain.com<br />
</code></p>