---
layout: post
status: publish
published: true
title: Virtualized application infrastructure
author:
  display_name: marius
  login: marius
  email: marius@remote-lab.net
  url: http://remote-lab.net/
author_login: marius
author_email: marius@remote-lab.net
author_url: http://remote-lab.net/
wordpress_id: 254
wordpress_url: https://remote-lab.net/?p=254
date: '2014-05-07 12:32:41 +0000'
date_gmt: '2014-05-07 10:32:41 +0000'
categories:
- Linux
- Virtualization
- Storage
tags: []
comments: []
---
<p>In this post I'll show how you can build a secure virtualized infrastructure for a basic webapp. We will break the setup into VMs that provide isolated services. You can find below the infrastructure diagram. The followings steps will show how you can set up a bare-metal server running Debian Wheezy to act as a KVM hypervisor and the process of deploying and configuring the VMs and the services they are running. </p>
<p><a href="https://remote-lab.net/wp-content/uploads/2014/05/rlug-New-Page.png"><img src="https://remote-lab.net/wp-content/uploads/2014/05/rlug-New-Page.png" alt="rlug - New Page" width="832" height="814" class="aligncenter size-full wp-image-255" /></a></p>
<p>Install kvm and tools:</p>
<p><code lang="bash[notools]">root@vmm:~>>> aptitude install qemu-kvm libvirt-bin virt-manager virt-viewer</code></p>
<p>Install openvswitch :</p>
<p><code lang="bash[notools]">root@vmm:~>>> aptitude install openvswitch-switch openvswitch-datapath-source</code></p>
<p>Build Open vSwitch datapath kernel module:</p>
<p><code lang="bash[notools]">root@vmm:~>>> module-assistant auto-install openvswitch-datapath</code></p>
<p>The management IP address of the hypervisor and the other public IP addresses are assigned on the same interface by the hosting provider. In order to provide Internet connectivity for the VMs we need to create a bridge containing the physical interface where the public IPs are routed and add the VMs ports to this bridge. The trouble is that since this is also the management link we'll lose connectivity after adding the physical interface to the bridge. After this operation we need to assign the management IP address to the bridge interface. For doing this we edit the /etc/network/interfaces file.</p>
<p>Add openvswitch bridges:</p>
<p><code lang="bash[notools]">root@vmm:~>>> ovs-vsctl add-br sw-net</code></p>
<p>Edit the /etc/network/interfaces file:</p>
<p><code lang="bash[notools]">root@vmm:~>>> cat /etc/network/interfaces<br />
auto sw-net<br />
iface sw-net inet static<br />
  address   46.4.71.66<br />
  broadcast 46.4.71.95<br />
  netmask   255.255.255.224<br />
  gateway   46.4.71.65<br />
pre-up ip link set dev eth0 up</code></p>
<p>At boot time the openvswitch daemon is started after the network init script so when the network init script is run it won't find the sw-net interface defined in /etc/network/interfaces file. A dirty workaround for this is to re-run the network init script after all the services are loaded. In order to do this we need to edit the /etc/rc.local file:  </p>
<p><code lang="bash[notools]">root@vmm:~>>> cat /etc/rc.local<br />
/etc/init.d/networking restart<br />
exit 0 </code></p>
<p>Now let's add the physical interface to the bridge. After this we should either restart the network service from the console or do a hard reset:</p>
<p><code lang="bash[notools]">root@vmm:~>>> ovs-vsctl add-port sw-net eth0</code></p>
<p>The next step is to add the second bridge, where the internal network ports will be connected </p>
<p><code lang="bash[notools]">root@vmm:~>>> ovs-vsctl add-br sw-lan</code></p>
<p>I prefer using virt-install for new VMs provisioning. The problem with it is that it currently doesn't support Open vSwitch bridges so we'll need to adjust it a little by adding the following line to the /usr/lib/pymodules/python2.7/virtinst/VirtualNetworkInterface.py file. This will add the virtualport tag to the VM xml definition:</p>
<p><code lang="bash[notools]">root@vmm:~>>> diff -u /usr/lib/pymodules/python2.7/virtinst/VirtualNetworkInterface.py /usr/lib/pymodules/python2.7/virtinst/VirtualNetworkInterface.py.orig<br />
--- /usr/lib/pymodules/python2.7/virtinst/VirtualNetworkInterface.py	2014-05-06 22:06:21.396072330 +0200<br />
+++ /usr/lib/pymodules/python2.7/virtinst/VirtualNetworkInterface.py.orig	2014-05-06 22:13:17.121958858 +0200<br />
@@ -384,7 +384,6 @@<br />
         xml += "      <mac address='%s'/>\n" % self.macaddr<br />
         xml += target_xml<br />
         xml += model_xml<br />
-        xml += "      <virtualport type='openvswitch'/>\n"<br />
         xml += "    </interface>"<br />
         return xml</code></p>
<p>Now that we have the networking ready the last thing that we need are the storage files that the VMs will use. For creating the files we use the qemu-img utility. I prefer qcow2 files as they provide thin provision and snapshot capabilities. /var/lib/libvirt/images is the default directory used by libvirt-bin so let's create the storage files here:</p>
<p><code lang="bash[notools]">root@vmm:/var/lib/libvirt/images>>> qemu-img create -f qcow2 rtr01.qcow2 10G<br />
Formatting 'rtr01.qcow2', fmt=qcow2 size=10737418240 encryption=off cluster_size=65536 </code></p>
<p>We can now create the VM and start the OS installation. We'll first install the rtr01 VM as it will provide Internet connectivity for the rest of the VMs in the internal network. The following command will generate a VM called rtr01 with 4 vCPUs, 4GB of ram, storage file located at /var/lib/libvirt/images/rtr01.qcow2 and 2 network interfaces - one in the bridge connected to the Internet and another connected to the internal network, the console is presented over VNC and it will first boot from the cdrom device loaded from the /var/lib/libvirt/images/vyatta-livecd_VC6.6R1_amd64.iso file. The disk and network interface will use paravirtualized drivers to obtain increased I/O performance.</p>
<p><code lang="bash[notools]">root@vmm:~>>> virt-install --name rtr01 --vcpus=4 --ram=4096 --disk path=/var/lib/libvirt/images/rtr01.qcow2,bus=virtio --network bridge=sw-net,model=virtio --network bridge=sw-lan,model=virtio --graphics vnc --cdrom /var/lib/libvirt/images/vyatta-livecd_VC6.6R1_amd64.iso --boot cdrom</code></p>
<p>After this command is issued a console windows will pop up and it will prompt the cdrom installation. After finishing the installation we can proceed to configuring the device:</p>
<p><code lang="bash[notools]"><br />
# set interfaces IP addresses<br />
set interfaces ethernet eth0 mac 00:50:56:00:5e:97<br />
set interfaces ethernet eth0 address 46.4.71.77/27<br />
set protocols static route 0.0.0.0/0 next-hop 46.4.71.65<br />
set system host-name rtr01<br />
set system domain-name nullzero.me<br />
set interfaces ethernet eth1 10.0.1.1/24<br />
set interfaces ethernet eth1 address 10.0.1.1/24</p>
<p># set SNAT for internal network<br />
set nat source rule 10 source address 10.0.1.0/24<br />
set nat source rule 10 outbound-interface eth0<br />
set nat source rule 10 translation address masquerade</p>
<p># set DNAT for the request coming on port tcp 80 on the public IP<br />
set nat destination rule 10 destination address 46.4.71.77<br />
set nat destination rule 10 inbound-interface eth0<br />
set nat destination rule 10 destination port 80<br />
set nat destination rule 10 translation address 10.0.1.2<br />
set nat destination rule 10 translation port 80<br />
set nat destination rule 10 protocol tcp</p>
<p># generate server and client certificates and keys<br />
vyatta@rtr01:~$ sudo -s<br />
vbash-4.1# cp -R /usr/share/doc/openvpn/examples/easy-rsa/2.0/* /etc/openvpn/<br />
edit KEY_COUNTRY, KEY_PROVINCE, KEY_CITY, KEY_ORG, KEY_EMAIL variables<br />
vbash-4.1# vi /etc/openvpn/vars<br />
vbash-4.1# cd /etc/openvpn<br />
vbash-4.1# source vars<br />
vbash-4.1# ./clean-all<br />
vbash-4.1# ./build-ca<br />
vbash-4.1# ./build-dh<br />
vbash-4.1# ./build-key-server rtr01<br />
vbash-4.1# ./build-key client<br />
vbash-4.1# mkdir /config/auth<br />
vbash-4.1# cp -R /etc/openvpn/keys/* /config/auth</p>
<p># configure the server certificates and key location<br />
set interfaces openvpn vtun0 tls ca-cert-file /config/auth/ca.crt<br />
set interfaces openvpn vtun0 tls cert-file /config/auth/rtr01.crt<br />
set interfaces openvpn vtun0 tls dh-file /config/auth/dh1024.pem<br />
set interfaces openvpn vtun0 tls key-file /config/auth/rtr01.key</p>
<p># configure the openvpn server<br />
set interfaces openvpn vtun0 mode server<br />
set interfaces openvpn vtun0 server subnet 172.16.17.0/24<br />
set interfaces openvpn vtun0 server push-route 10.0.1.0/24<br />
set interfaces openvpn vtun0 openvpn-option "--comp-lzo --mssfix --tun-mtu 1488"</p>
<p># openvpn client config file</p>
<p>marius@remoteur:~>>> cat /etc/openvpn/nullzero.conf<br />
client<br />
dev tun<br />
proto udp<br />
remote 46.4.71.77 1194<br />
resolv-retry infinite<br />
nobind<br />
persist-key<br />
persist-tun<br />
ca /etc/openvpn/nullzero/ca.crt<br />
cert /etc/openvpn/nullzero/client.crt<br />
key /etc/openvpn/nullzero/client.key<br />
ns-cert-type server<br />
comp-lzo<br />
verb 3</p>
<p>#configure firewall<br />
set firewall state-policy established action 'accept'<br />
set firewall state-policy related action 'accept'<br />
set firewall all-ping 'enable'<br />
edit firewall name rtr01<br />
set default-action 'drop'<br />
set rule 10 action accept<br />
set rule 10 destination port 22<br />
set rule 10 protocol tcp<br />
set rule 11 action accept<br />
set rule 11 destination port 80<br />
set rule 11 protocol tcp<br />
set rule 12 action accept<br />
set rule 12 destination port 1194<br />
set rule 12 protocol udp<br />
exit<br />
set interfaces ethernet eth0 firewall in name rtr01</code></p>
<p>After completing these steps we should have a working router, firewall and VPN server.</p>
<p>Now let's continue with creating the second VM. We'll do a network install from minimal CD. First create the storage file:</p>
<p><code lang="bash[notools]">root@vmm:~>>> qemu-img create -f qcow2 /var/lib/libvirt/images/lb01.qcow2 10G<br />
Formatting '/var/lib/libvirt/images/lb01.qcow2', fmt=qcow2 size=10737418240 encryption=off cluster_size=65536</code></p>
<p>Next we can start the installation process by using the cdrom file located at /var/lib/libvirt/images/debian-7.5.0-amd64-netinst.iso </p>
<p><code lang="bash[notools]">root@vmm:~>>> virt-install --name lb01 --vcpus=2 --ram=4096 --disk path=/var/lib/libvirt/images/lb01.qcow2,bus=virtio --network bridge=sw-lan,model=virtio  --graphics vnc --cdrom /var/lib/libvirt/images/debian-7.5.0-amd64-netinst.iso --boot cdrom</code></p>
<p>After completing the OS installation we have a fresh running Debian Wheezy system. We don't want to repeat the install process for the other files so we'll just copy the existing image of the Debian system and modify the IP settings and hostnames. We first copy the base image, then attach it by using qemu-nbd, mount the partition where the file system resides and then edit the files that we need.</p>
<p><code lang="bash[notools]">root@vmm:/var/lib/libvirt/images>>> cp lb01.qcow2 db01.qcow2; cp lb01.qcow2 web01.qcow2<br />
root@vmm:/var/lib/libvirt/images>>> modprobe nbd max_part=8<br />
root@vmm:/var/lib/libvirt/images>>> qemu-nbd -c /dev/nbd0 web01.qcow2<br />
root@vmm:/var/lib/libvirt/images>>> kpartx -a /dev/nbd0<br />
root@vmm:/var/lib/libvirt/images>>> mount /dev/mapper/nbd0p1 /mnt<br />
root@vmm:/var/lib/libvirt/images>>> vim /mnt/etc/network/interfaces<br />
root@vmm:/var/lib/libvirt/images>>> vim /mnt/etc/hosts<br />
root@vmm:/var/lib/libvirt/images>>> vim /mnt/etc/hostname<br />
root@vmm:/var/lib/libvirt/images>>> umount /mnt<br />
root@vmm:/var/lib/libvirt/images>>> kpartx -d /dev/nbd0<br />
root@vmm:/var/lib/libvirt/images>>> qemu-nbd -d /dev/nbd0</code></p>
<p>We repeat the steps above for the db01.qcow2 file.</p>
<p>Let's now create the web01 and db01 VMs. Since we already have the base storage files we don't need to run the OS installation:</p>
<p><code lang="bash[notools]">root@vmm:~>>> virt-install --name web01 --vcpus=4 --ram=4096 --disk path=/var/lib/libvirt/images/web01.qcow2,bus=virtio --network bridge=sw-lan,model=virtio  --graphics vnc --import<br />
root@vmm:~>>> virt-install --name db01 --vcpus=4 --ram=4096 --disk path=/var/lib/libvirt/images/db01.qcow2,bus=virtio --network bridge=sw-lan,model=virtio  --graphics vnc --import</code></p>
<p>Once we have booted al the VMs let's start configuring the services.</p>
<p>On the http load balancer we'll install varnish and configure the web server as backend: </p>
<p><code lang="bash[notools]">root@lb01:~>>> aptitude install varnish<br />
root@lb01:~>>> sed -i 's/6081/80/' /etc/default/varnish<br />
root@lb01:~>>> sed -i 's/127.0.0.1/10.0.1.3/' /etc/varnish/default.vcl<br />
root@lb01:~>>> sed -i 's/8080/80/' /etc/varnish/default.vcl<br />
root@lb01:~>>> /etc/init.d/varnish restart</code></p>
<p>On the web server we'll install nginx and php-fpm and configure the default vhost:</p>
<p><code lang="bash[notools]">root@web01:~>>> aptitude install nginx php5-fpm php5-mysql</code></p>
<p>Add the following location block to the first server block:</p>
<p><code lang="bash[notools]">location ~ \.php$ {<br />
        fastcgi_pass   unix:/var/run/php5-fpm.sock;<br />
        fastcgi_index  index.php;<br />
        include        fastcgi_params;<br />
}</code></p>
<p>Create an index file in the document root that will query the database server:</p>
<p><code lang="bash[notools]">root@web01:/srv/www>>> cat index.php<br />
<?php<br />
$con=mysqli_connect("10.0.1.4","user","parola","test");</p>
<p>$result = mysqli_query($con,"SELECT * FROM testable");</p>
<p>$row = mysqli_fetch_array($result);<br />
echo $row['hello'];</p>
<p>mysqli_close($con);<br />
?></code></p>
<p>On the database server we'll install mysql server and create a dummy database and table;</p>
<p><code lang="bash[notools]">root@db01:~>>> aptitude install mysql-server<br />
root@db01:~>>> mysql<br />
Welcome to the MySQL monitor.  Commands end with ; or \g.<br />
Your MySQL connection id is 43<br />
Server version: 5.5.37-0+wheezy1 (Debian)</p>
<p>Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.</p>
<p>Oracle is a registered trademark of Oracle Corporation and/or its<br />
affiliates. Other names may be trademarks of their respective<br />
owners.</p>
<p>Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.</p>
<p>mysql> create database test;<br />
mysql> use test;<br />
mysql> CREATE TABLE testable (hello VARCHAR(20));<br />
mysql> INSERT INTO testable (hello) VALUES("Hello World!");<br />
mysql> CREATE USER 'user'@'10.0.1.3' IDENTIFIED BY 'parola';<br />
mysql> GRANT ALL PRIVILEGES ON * . * TO 'user'@'10.0.1.3';<br />
mysql> FLUSH PRIVILEGES;<br />
mysql> exit</code></p>
<p>After this final step we have our setup ready and http://app.nullzero.me/ should show the Hello World!</p>