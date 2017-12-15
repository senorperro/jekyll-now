---
layout: post
title: Kioptrix 1 Walkthrough
---

Kioptrix VMs are meant as entry level pentest challenges where the goal is to get root privileges.
Download [Kioptrix 1 here](http://www.kioptrix.com/blog/test-page/) or from [Vulnhub](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/#download).

<h3>Finding the Host</h3>

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

<h3>Enumeration:80/tcp http</h3>

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
+ ... (other nikto output) ...
+ OSVDB-2733: Apache/1.3.20 - Apache 1.3 below 1.3.29 are vulnerable to overflows in mod_rewrite and mod_cgi. CAN-2003-0542.
<span class="out-highlight">+ mod_ssl/2.8.4 - mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2002-0082, OSVDB-756.</span>
+ ///etc/hosts: The server install allows reading of any system file by adding an extra '/' to the URL.
+ ... (other nikto output) ...
+ OSVDB-3233: /icons/README: Apache default file found.
+ OSVDB-3092: /test.php: This might be interesting...
+ 8345 requests: 0 error(s) and 21 item(s) reported on remote host
+ End Time:           2017-12-12 15:02:16 (GMT-5) (25 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
</pre>

Searching online for CVE-2002-0082 produces a corresponding PoC exploit: [Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow](https://www.exploit-db.com/exploits/764/)

<h3>Enumeration:139/tcp netbios-ssn</h3>

The samba version running on the host is identified using MSF's <code>smb_version</code> module:

<pre class="console-output">
<u>msf</u> auxiliary(<span class="prompt"><b>smb_version</b></span>) > run

<span class="dir"><b>[*]</b></span> 192.168.201.132:139   - Host could not be identified: Unix (<span class="out-highlight">Samba 2.2.1a</span>)
<span class="dir"><b>[*]</b></span> Scanned 1 of 1 hosts (100% complete)
<span class="dir"><b>[*]</b></span> Auxiliary module execution completed
</pre>

Searching online for this samba version also yields an exploit:<br> [Samba < 2.2.8 (Linux/BSD) - Remote Code Execution](https://www.exploit-db.com/exploits/10/)<br>
MSF: [Samba trans2open Overflow (Linux x86)](https://www.rapid7.com/db/modules/exploit/linux/samba/trans2open)

<h3>Exploitation:80/tcp http</h3>

A copy of the OpenFuckV2.c is copied into the current dir, which contains [instructions on how to update](http://paulsec.github.io/blog/2014/04/14/updating-openfuck-exploit/) and compile it. Once updated, trying to compile in Kali Linux produces some errors using <code>gcc</code>:

<pre class="console-output">
<span class="prompt">root@kali</span>:<span class="dir">~</span># searchsploit -m 764
Exploit: Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Exploit
    URL: https://www.exploit-db.com/exploits/764/
   Path: /usr/share/exploitdb/platforms/unix/remote/764.c

Copied to: /root/764.c

<span class="prompt">root@kali</span>:<span class="dir">~</span># ... (Steps Update OpenFuck Exploit) ...

<span class="prompt">root@kali</span>:<span class="dir">~</span># gcc -o OpenFuck 764.c -lcrypto
764.c:645:24: error: ‘SSL2_MAX_CONNECTION_ID_LENGTH’ undeclared here (not in a function); did you mean ‘SSL_MAX_SSL_SESSION_ID_LENGTH’?
  unsigned char conn_id[SSL2_MAX_CONNECTION_ID_LENGTH];
                        ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                        SSL_MAX_SSL_SESSION_ID_LENGTH
764.c: In function ‘read_ssl_packet’:
764.c:847:19: error: ‘SSL2_MT_ERROR’ undeclared (first use in this function); did you mean ‘SSL_METHOD’?
    if ((buf[0] == SSL2_MT_ERROR) && (rec_len == 3)) {
                   ^~~~~~~~~~~~~
                   SSL_METHOD
764.c:847:19: note: each undeclared identifier is reported only once for each function it appears in
764.c: In function ‘get_server_hello’:
764.c:979:16: error: ‘SSL2_MT_SERVER_HELLO’ undeclared (first use in this function); did you mean ‘SSL3_MT_SERVER_HELLO’?
  if (*(p++) != SSL2_MT_SERVER_HELLO) {
                ^~~~~~~~~~~~~~~~~~~~
                SSL3_MT_SERVER_HELLO
764.c: In function ‘send_client_master_key’:
764.c:1071:10: error: dereferencing pointer to incomplete type ‘EVP_PKEY {aka struct evp_pkey_st}’
  if (pkey->type != EVP_PKEY_RSA) {
          ^~
764.c: In function ‘get_server_verify’:
764.c:1148:16: error: ‘SSL2_MT_SERVER_VERIFY’ undeclared (first use in this function); did you mean ‘SSL3_MT_SERVER_HELLO’?
  if (buf[0] != SSL2_MT_SERVER_VERIFY) {
                ^~~~~~~~~~~~~~~~~~~~~
                SSL3_MT_SERVER_HELLO
764.c: In function ‘send_client_finished’:
764.c:1160:11: error: ‘SSL2_MT_CLIENT_FINISHED’ undeclared (first use in this function); did you mean ‘SSL3_MT_FINISHED’?
  buf[0] = SSL2_MT_CLIENT_FINISHED;
           ^~~~~~~~~~~~~~~~~~~~~~~
           SSL3_MT_FINISHED
764.c: In function ‘get_server_finished’:
764.c:1173:16: error: ‘SSL2_MT_SERVER_FINISHED’ undeclared (first use in this function); did you mean ‘SSL3_MT_SERVER_DONE’?
  if (buf[0] != SSL2_MT_SERVER_FINISHED) {
                ^~~~~~~~~~~~~~~~~~~~~~~
                SSL3_MT_SERVER_DONE
</pre>

This can be solved by running <code>apt-get install libssl1.0-dev</code>.
