---
layout: post
title: Kioptrix 1 Walkthrough
---

Kioptrix VMs are meant as entry level pentest challenges where the goal is to get root privileges.
Download [Kioptrix 1 here](http://www.kioptrix.com/blog/test-page/) or from [Vulnhub](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/#download).

<h3>Finding the Target</h3>

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
 <span class="out-line">192.168.201.132 00:0c:29:75:c4:41      1      60  VMware, Inc.</span>
 192.168.201.254 00:50:56:e2:0c:13      1      60  VMware, Inc.
</pre>

<h3>Enumeration</h3>

An <code>nmap</code> scan shows 6 open ports on the host:

<pre class="console-output">
<span class="prompt">root@kali</span>:<span class="dir">~</span># nmap -sV -p- -T4 192.168.201.132

Starting Nmap 7.60 ( https://nmap.org ) at 2017-12-07 17:02 EST
Nmap scan report for 192.168.201.132
Host is up (0.000076s latency).
Not shown: 65529 closed ports
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp    open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
32768/tcp open  status      1 (RPC #100024)
MAC Address: 00:0C:29:75:C4:41 (VMware)
</pre>

<h3>80/tcp http</h3>

<h3>139/tcp netbios-ssn</h3>
