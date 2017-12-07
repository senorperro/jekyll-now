---
layout: post
title: Kioptrix 1 Walkthrough
---

Kioptrix VMs are meant as entry level pentest challenges where the goal is to get root privileges.
Download [Kioptrix 1 here](http://www.kioptrix.com/blog/) or from [Vulnhub](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/#download).

Once you've downloaded, configured and deployed the Kioptrix image on your prefered virtualization software, you can find its assigned IP address using Nmap's ping sweep flag or the netdiscover tool

__nmap -sP 192.168.1.0/24__

Now that you know the Kioptrix host's IP address, you can launch a quick nmap scan to see what ports are open on it. Since we're performing these scans within the safety of our home LAN, we can be hasty with our scans.

```console
nmap -sV -p- -T4 192.168.201.132
Nmap scan report for 192.168.201.132
Host is up (0.000098s latency).
Not shown: 65529 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp   open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp  open  ssl/http    Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
1024/tcp open  status      1 (RPC #100024)
MAC Address: 00:0C:29:75:C4:41 (VMware)
```

Judging from the scan, there is an Apache web server running on the host

When searching online about the Apache version that is running on this host, we find that it is rather outdated, and an exploit called "[OpenFuckV2](https://www.exploit-db.com/exploits/764/)" comes up among the top search results, which exploits a buffer overflow in the mod_ssl module of apache, giving remote code execution (RCE) capabilities to anyone .Since it's written in C, you should be able to compile it using gcc if you're using any of the most common Linux distributions.


The exploit itself comes with a link to a blog post that contains instructions on how to modify the exploit in order for it to actually function, as well as gcc compilation syntax.

```c
/*
 * E-DB Note: Updating OpenFuck Exploit ~ http://paulsec.github.io/blog/2014/04/14/updating-openfuck-exploit/
 *
 * OF version r00t VERY PRIV8 spabam
 * Compile with: gcc -o OpenFuck OpenFuck.c -lcrypto
 * objdump -R /usr/sbin/httpd|grep free to get more targets
 * #hackarena irc.brasnet.org
 */ 
```

Although I modified the exploit apropriately, I still got compilation errors when using the gcc flags specified by the comments at the beginning of the code. GCC tells us that some variables used by the exploit, which are supposed to be defined in the libssl library, are missing or incomplete.

![_config.yml]({{ site.baseurl }}/images/openfuck-after-update-errors.png)

After a while of searching for an answer online, I finally found that the specific libssl version that the OpenFuck exploit makes use of must be installed separately from the one indicated in the tutorial.

__<span style="font-family:'consolas'">apt-get install libssl1.0-dev</span>__

With this missing library version installed, the OpenFuckV2 exploit should run without issues after compiling it with the indicated gcc flags.

__<span style="font-style: 'consolas'"gcc -o OpenFuckV2 764.c -lcrypto</span>

Once you've successfully complied the exploit, running it produces instructions on how to use it apropriately. It provides a list of possible OS version + Apache version combination that the exploit works for.

Goign from the nmap scan we already know 2 out of those things, the Apacher version running on the host and the OS that the server is running on, which is a RedHat Linux, although we do not know the exact version.

