==================
PARIKSHAN - LIVE DEBUGGING
=================

Decription
--------

This is a description of our early prototype of Parikshan a live debugging framework. 
The following README shows how-to instructions for setup and network proxy/forwarding directions.

1. Instructions for setting up the container target
---------------

Install OpenVZ via the following script: https://github.com/nipunarora/sandbox-testing/blob/master/openvz-boot.sh

Install OpenVZ in Debian Wheezy: http://www.howtoforge.com/installing-and-using-openvz-on-debian-wheezy-amd64

Install Ubuntu 13.04 instructions: http://www.howtoforge.com/installing-and-using-openvz-on-ubuntu-13.04-amd64

Install OpenVZ in Centos 6.4: http://www.howtoforge.com/installing-and-using-openvz-on-centos-6.4

Install vzdump http://chrisschuld.com/2009/11/installing-vzdump-for-openvz-on-centos/

Installing OpenVZPanel http://owp.softunity.com.ru

Everything about vzctl: http://openvz.org/Man/vzctl

Ploop Articles: http://openvz.livejournal.com/40830.html http://openvz.org/Ploop/Backup

Setting up and fixing networking issue: VENET routed networking is probably the simplest to set up, simply add the IP address to the VE:

	[host-node]# vzctl set 101 --ipadd 192.168.2.1 --save

	After this the host should be able to ping the VE. To allow the VE to access the rest of the LAN we must enable forwarding and masquerading, as all activity on the LAN must look like it is coming directly from host (with its IP address).

	First stop and flush default iptables

	service iptables stop

	Set ip_forwarding for ipv4

	[host-node]# echo 1 > /proc/sys/net/ipv4/ip_forward

	Set iptables to MASQUERADE on eth0
	[host-node]# iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

	replace /etc/modprobe.d/openvz.conf from 1 to 0

	To allow for cloning to work - 
	http://wiki.hillockhosting.com/openvz/vzrst-module-is-not-loaded-on-the-destination-node/

	modprobe vzrst
	modprobe vzcpt


Starting a ploop container

	Download a CentOS 6 template:

	cd /vz/template/cache

	wget http://download.openvz.org/template/precreated/centos-6-x86_64.tar.gz

	Basic commands for using OpenVZ.

	To set up a VPS from the CentOS 6 template, run:

	vzctl create 101 --ostemplate centos-6-x86_64 --layout ploop --config basic

	The 101 must be a uniqe ID - each virtual machine must have its own unique ID. You can use the last part of the virtual machine's IP address for it. For example, if the virtual machine's IP address is 192.168.0.101, you use 101 as the ID.

	To set a hostname and IP address for the vm, run:

	vzctl set 101 --hostname test.example.com --save

	vzctl set 101 --ipadd 192.168.0.101 --save

	Next we set the number of sockets to 120 and assign a few nameservers to the vm:

	vzctl set 101 --numothersock 120 --save

	vzctl set 101 --nameserver 8.8.8.8 --nameserver 8.8.4.4 --nameserver 145.253.2.75 --save


Network Card Management in KVM
----------------

 The assignment of network cards to interfaces is done in /etc/udev/rules.d/70-persistent-net.rules you need to flush the lines, if you add a new NIC

Evaluation Install
------------------

	LAMP Install - https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-14-04
	Httperf - https://cs.uwaterloo.ca/~brecht/servers/openfiles.html - fixing file descriptors

Small Networking Issues
------------

Change ssh access to not use DNS and GGSAPI::

       useDNS no
       GGSAPIAuthentication no

Start mysqld server using ./bin/mysqld_safe --skip-name-resolve --user=mysql

Aggregator and Duplicator
------------

Duplicator can be found in the proxyClone folder. Execution instructions can be found by doing a ./proxyClone
Aggregator can be found in the proxyAgg folder. Execution instructions can be found by doing a ./proxyAgg

Execution instructions of both scripts are similar

Example - 
```
./proxy -l 9133 -h 127.0.0.1 -p 9132 -x 192.168.122.45 -d 9131 -m 127.0.0.1  -a

-l = local port
-h = local host
-p = production port
-x = production host
-d = duplicate port
-m = duplicate host
-a = asynchronous
-s = synchronous
-b <2048> = buffer size
```

Live Cloning Scripts
-----------------

vzclone_vz48 is the latest cloning script available.
Try vzclone -h for instructions to clone

./vzclone_vz48 --live --online <destination host ip> <container ID>

Instructions for Execution
-------

Please ensure that the duplicator is running before the vzclone is executed. 
Cloning will generate an active running clone with minimal disruption to the production container.
The debug container can then be instrumented by any instrumentation tool of your choice.

Step 1.

Launch a container with the target production application

Step 2.

Start a proxyClone

Step 3.

Clone the container using the live cloning script above

Step 4.

Debug debug container as required

Contact us
-----------------

We have verified that Parikshan runs on a variety of systems. 
The README provides instructions for the system setup. 
Please submit an issue or contact nipun@cs.columbia.edu for further queries.

License
-----------
This software is released under the MIT license.

Copyright (c) 2013, by The Trustees of Columbia University in the City of New York.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
