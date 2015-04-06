---
layout: post
status: publish
published: true
title: JunOS config interfaces IP address parser
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 240
wordpress_url: https://remote-lab.net/?p=240
date: '2014-02-23 20:58:36 +0000'
date_gmt: '2014-02-23 18:58:36 +0000'
categories:
- Linux
tags: []
comments: []
---
<p>Hello guys,</p>
<p>In todays post I will show how you can obtain an interface IP address out of a JunOS curly brackets configuration file. You may find below the script and also the source configuration file. Please note that in order to run the script both files need to be placed in the same directory. </p>
<p>Please check it and let me know what you think, it's my first Perl script so it could definitely be improved.</p>
<p><code lang="perl[notools]">#!/usr/bin/perl<br />
open (CONFIG, 'config.txt');<br />
my $data = do { local $/; <CONFIG>;};<br />
close (CONFIG);</p>
<p>my $system = $data =~ m{(\bsystem\s*({(?:(?>[^{}]+)|(?-1))*}))}<br />
    ? $1<br />
    : die "system not found";</p>
<p>my $intconfig = $data =~ m{(\binterfaces\s*({(?:(?>[^{}]+)|(?-1))*}))}<br />
    ? $1<br />
    : die "interfaces not found";</p>
<p>if ($ARGV[0] eq 'sys') {<br />
    print $system;<br />
}</p>
<p>if ($ARGV[0] eq 'int') {<br />
    if (!defined $ARGV[1]) {<br />
        print $intconfig, "\n";<br />
    }<br />
    if (defined $ARGV[1]) {<br />
        my $int = $intconfig =~ m{(\b$ARGV[1]\s*({(?:(?>[^{}]+)|(?-1))*}))}<br />
            ? $1<br />
            : die "$ARGV[1] not found";</p>
<p>        if (!defined $ARGV[2]) {<br />
            print $int. "\n";<br />
        }</p>
<p>        if (defined $ARGV[2]) {<br />
            my $unit = $int =~m{(\bunit $ARGV[2]\s*({(?:(?>[^{}]+)|(?-1))*}))}<br />
                ? $1<br />
                : die "$ARGV[2] not found";</p>
<p>            my $inet = $unit =~ m{(\bfamily inet\s*({(?:(?>[^{}]+)|(?-1))*}))}<br />
                ? $1<br />
                : die "family inet not found in section";</p>
<p>            my $inetaddr = $inet =~ m{\baddress\s(\d{1,3}(?:\.\d{1,3}){3})}<br />
                ? $1<br />
                : die "no IP address";<br />
            print $inetaddr, "\n";<br />
        }<br />
    }<br />
}</p>
<p>if ($ARGV[0] eq '--help' or !defined $ARGV[0]) {<br />
    print "Usage : ./parser.pl sys                              # outputs system section config", "\n";<br />
    print "        ./parser.pl int                              # outputs interfaces section config", "\n";<br />
    print "        ./parser.pl int [int-name]                   # outputs specific interface section config", "\n";<br />
    print "        ./parser.pl int ge-1/1/7                     # outputs ge-1/1/7 interface section config", "\n";<br />
    print "        ./parser.pl int [int-name] [unit-id]         # outputs specific interface unit IP address", "\n";<br />
    print "        ./parser.pl int ge-1/1/7 1001                # outputs ge-1/1/7 interface unit 1001 IP address", "\n";<br />
}<br />
</code></p>
<p><code lang="perl[notools]"><br />
marius@remoteur:~>>> cat config.txt<br />
system {<br />
    host-name junos-device;<br />
    domain-name corporate.net<br />
    time-zone Europe/Bucharest;<br />
    default-address-selection;<br />
    no-redirects;<br />
    location country-code RO;<br />
}<br />
interfaces {<br />
    ge-1/0/0 {<br />
        description "Core: R:core1 RP:ge-0/1/4 (ptp, isis)";<br />
        mtu 9192;<br />
        unit 0 {<br />
            family inet {<br />
                address 192.168.140.29/31;<br />
            }<br />
        }<br />
    }<br />
    ge-1/0/1 {<br />
        description "Cust: R:cust-a RP:ge-1/0/0 (srx240H)";<br />
        unit 0 {<br />
            family inet {<br />
                address 172.16.166.196/30;<br />
            }<br />
        }<br />
    }<br />
    ge-1/0/2 {<br />
        flexible-vlan-tagging;<br />
        native-vlan-id 10;<br />
        mtu 9192;<br />
        unit 10 {<br />
            description "Cust: R:cust-b (data, feed A)";<br />
            vlan-id 10;<br />
            family inet {<br />
                address 192.168.136.184/31;<br />
            }<br />
        }<br />
        unit 1001 {<br />
            description "Core: R:cust-b (cpe management)";<br />
            vlan-id 1001;<br />
            family inet {<br />
                filter {<br />
                    output Protect-cpe;<br />
                }<br />
                address 10.15.4.6/30;<br />
            }<br />
        }<br />
    }<br />
    ae0 {<br />
        description "Core: R:colo-vc2 RI:ae5";<br />
        aggregated-ether-options {<br />
            minimum-links 1;<br />
            link-speed 1g;<br />
        }<br />
        unit 0 {<br />
            family inet {<br />
                address 192.168.140.126/31;<br />
            }<br />
        }<br />
    }<br />
    lo0 {<br />
        unit 0 {<br />
            description "Core: R:primary routing loopback";<br />
            family inet {<br />
                address 192.168.128.166/32;<br />
            }</p>
<p>            }<br />
        }<br />
    }<br />
}<br />
</code></p>