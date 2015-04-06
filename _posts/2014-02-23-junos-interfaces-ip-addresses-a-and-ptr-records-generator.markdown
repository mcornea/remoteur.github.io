---
layout: post
status: publish
published: true
title: JunOS interfaces IP addresses DNS records generator
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 242
wordpress_url: https://remote-lab.net/?p=242
date: '2014-02-23 21:28:17 +0000'
date_gmt: '2014-02-23 19:28:17 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>This post is closely related to the previous one where I showed how you can parse the interfaces IP addresses from a curly bracket JunOS config file. The following script will be used to generate A and PTR records for a BIND zone file. Please note that the script needs to be run within the same directory as the Perl parser script and the config file.</p>
<p>The entries will have the following format:<br />
type-fpc-pic-port 300 IN A $value<br />
$value IN PTR type-fpc-pic-port.hostname.domain</p>
<p> <code lang="bash[notools]"><br />
#/bin/bash<br />
PERL='/usr/bin/perl'<br />
PARSER='./parser.pl'<br />
CONFIG_FILE='config.txt'<br />
CONFIG_SYS='sys'<br />
CONFIG_INT='int'<br />
hostname=`$PERL $PARSER $CONFIG_SYS | grep host-name | sed -e s/host-name// -e s/\;// | tr '\r' ' ' | sed -e s/\ //g`<br />
domain=`$PERL $PARSER $CONFIG_SYS | grep domain-name | sed -e s/domain-name// -e s/\;// | tr '\r' ' ' | sed -e s/\ //g`<br />
for i in `grep "ge-[0-9]\/[0-9]\/[0-9] {\|ae[0-9] {\|lo[0-9] {" $CONFIG_FILE | sed -e s/\ //g  -e s/\{// | tr '\r' ' '`<br />
    do<br />
        intname=`echo $i | sed s/\\\//-/g`<br />
        for j in `$PERL $PARSER $CONFIG_INT $i | grep unit | sed -e s/\ unit//g  -e s/\{// -e s/\ //g | tr '\r' ' '`<br />
            do<br />
                inetaddr=`$PERL $PARSER $CONFIG_INT $i $j | tr '\r' ' '`<br />
                lastoct=`echo $inetaddr | awk -F '.' {'print $4'}`<br />
                if [ $j -eq 0 ]<br />
                then<br />
                    echo "$intname 300 IN A  $inetaddr"<br />
                    echo "$lastoct  IN  PTR $intname.$hostname.$domain."<br />
                    echo</p>
<p>                else<br />
                    if [ $j -eq 10 ]<br />
                    then<br />
                        echo "$intname 300 IN A  $inetaddr"<br />
                        echo "$lastoct  IN  PTR $intname.$hostname.$domain."<br />
                        echo<br />
                    else<br />
                        echo "$intname-u$j 300 IN A  $inetaddr"<br />
                        echo "$lastoct  IN  PTR $intname-u$j.$hostname.$domain."<br />
                        echo<br />
                    fi<br />
                fi<br />
            done<br />
    done<br />
</code></p>