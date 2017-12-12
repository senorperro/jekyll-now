---
layout: post
title: Kioptrix 1 Walkthrough
---

Kioptrix VMs are meant as entry level pentest challenges where the goal is to get root privileges.
Download [Kioptrix 1 here](http://www.kioptrix.com/blog/test-page/) or from [Vulnhub](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/#download).

<h3>1.Finding the Host</h3>

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

A <code>nikto</code> scan yields some useful information on the host's webserver:

<pre class="console-output">
<span class="prompt">root@kali</span>:<span class="dir">~</span># nikto -h 192.168.201.132
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.201.132
+ Target Hostname:    192.168.201.132
+ Target Port:        80
+ Start Time:         2017-12-12 15:01:51 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
+ Server leaks inodes via ETags, header found with file /, inode: 34821, size: 2890, mtime: Wed Sep  5 23:12:46 2001
+ ... (other nikto output)...
+ OSVDB-2733: Apache/1.3.20 - Apache 1.3 below 1.3.29 are vulnerable to overflows in mod_rewrite and mod_cgi. CAN-2003-0542.
<span class="out-highlight">+ mod_ssl/2.8.4 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2002-0082, OSVDB-756.</span>
+ ///etc/hosts: The server install allows reading of any system file by adding an extra '/' to the URL.
+ ...(other nikto output)...
+ OSVDB-3233: /icons/README: Apache default file found.
+ OSVDB-3092: /test.php: This might be interesting...
+ 8345 requests: 0 error(s) and 21 item(s) reported on remote host
+ End Time:           2017-12-12 15:02:16 (GMT-5) (25 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
</pre>

Searching online for CVE-2002-0082 produces a corresponding exploit: [Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow](https://www.exploit-db.com/exploits/764/)

<h3>139/tcp netbios-ssn</h3>
