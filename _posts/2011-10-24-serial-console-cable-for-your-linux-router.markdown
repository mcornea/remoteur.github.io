---
layout: post
status: publish
published: true
title: Serial console cable for your Linux router
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 53
wordpress_url: http://www.remote-lab.net/?p=53
date: '2011-10-24 22:32:46 +0000'
date_gmt: '2011-10-24 19:32:46 +0000'
categories:
- Linux
tags: []
comments:
- id: 3
  author: k2 reviews
  author_email: usaarron@gmx.com
  author_url: http://www.besthomegymsreviews.com/bodycraft-k2/
  date: '2011-11-05 17:00:06 +0000'
  date_gmt: '2011-11-05 14:00:06 +0000'
  content: "Great! thanks for the share! \nArron"
---
<p>Hello guys,</p>
<p>Today I have been trying to get some working console cables for the Asus WL-500 routers I have. The first thing I tried was using an old Nokia CA-42 data cable which includes an Prolific USB to 3.3V TTL converter but unfortunately it ended in getting a messy input and output. So the other solution I thought of was building my own RS232-to-3.3V TTL converter.<br />
Below you may find a scheme of the simplest converter I found. </p>
<p><a href="http://www.remote-lab.net/wp-content/uploads/2011/10/ttltors2320kf.jpg"><img src="http://www.remote-lab.net/wp-content/uploads/2011/10/ttltors2320kf.jpg" alt="" title="converter" width="486" height="263" class="aligncenter size-full wp-image-54" /></a></p>
<p>You will need the following components to build it:<br />
2 x BC337 transistors<br />
1 x 1.5K resistor<br />
1 x 22K resistor<br />
1 x 4.7K resistor<br />
1 x 3.9K resistor<br />
1 x DB9 female connector</p>
<p>This converter did the job for me but only when using it with a computer which had a built-in serial port. When trying it with an USB-to-RS232 converter (Prolific chipset) I got a clean output from the UART port of the router but a messy input. I guess I'll have to try another type of USB-RS232 converter(other chipset than Prolific) or build an USB-to-3.3V TTL converter from scratch.<br />
I am also attaching a document containing a complete list of RS232-to-3.3V TTL converters:<br />
<a href='http://www.remote-lab.net/wp-content/uploads/2011/10/mt1389-serial-interface-gallery.pdf'>Converters list</a></p>
<p>Later Edit: I found a workaround for the messy input when using an USB-to-serial adapter. The default OpenWRT image is compiled with a default baud rate of 115200 bps for the console port. You need to recompile the kernel and use a baud rate of 9600 bps, I will later post a tutorial on how this should be done. </p>
<p><a href="http://www.remote-lab.net/wp-content/uploads/2011/10/IMG_0098.jpg"><img src="http://www.remote-lab.net/wp-content/uploads/2011/10/IMG_0098-1024x764.jpg" alt="" title="IMG_0098" width="550" height="410" class="aligncenter size-large wp-image-55" /></a></p>
<p><a href="http://www.remote-lab.net/wp-content/uploads/2011/10/IMG_0100.jpg"><img src="http://www.remote-lab.net/wp-content/uploads/2011/10/IMG_0100-1024x595.jpg" alt="" title="IMG_0100" width="550" height="319" class="aligncenter size-large wp-image-56" /></a> </p>