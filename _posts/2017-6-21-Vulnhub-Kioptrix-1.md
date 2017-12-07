---
layout: post
title: Kioptrix 1 Walkthrough
---

Kioptrix VMs are meant as entry level pentest challenges where the goal is to get root privileges.
Download [Kioptrix 1 here](http://www.kioptrix.com/blog/test-page/) or from [Vulnhub](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/#download).

<h3>Find the Target</h3>

Using <code>netdiscover</code> the IP address of the target is found.

<pre class="console-output">
<span class="prompt">root@kali</span>:<span class="dir">~</span># netdiscover -r 192.168.201.0/24 -i eth0

 Currently scanning: Finished!   |   Screen View: Unique Hosts        
 19 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 1140
 _________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname  
 -------------------------------------------------------------------------
 192.168.201.1   00:50:56:c0:00:08     14     840  VMware, Inc.
 192.168.201.2   00:50:56:fa:93:9d      3     180  VMware, Inc.
 192.168.201.132 00:0c:29:75:c4:41      1      60  VMware, Inc.
 192.168.201.254 00:50:56:e2:0c:13      1      60  VMware, Inc.
</pre>

<h3>Enumerate</h3>

Placeholder text.
