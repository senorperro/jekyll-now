---
layout: post
title: Kioptrix 1 - Compilation Errors from Hell
---
__Introduction__

While going around looking for something to keep my pentesting skils sharp, I came across the __[Kioptrix challenges](http://www.kioptrix.com/blog/)__, which are a series of VMs with security misconfigurations so people can attempt to gain root access by whatever viable means they find (exploiting the virtualization software itself doesn't count!).

![_config.yml]({{ site.baseurl }}/images/newKioptrix_1.jpg)

__Target Audience Level:__ [<span style="color:green">■ ■</span>] Easy*

<sub>Note: I say "easy" because, through personal experience, it seems like compiling the exploit used as part of this exercise might become less straightforward as time passes and the libraries it uses are further updated.</sub>

What this tutorial assumes.

* Basic TCP/IP knowledge.
* Virtualization software usage knowledge.
* Basic Programming concepts.

The Kioptrix challenges are meant for folks trying to get an idea of what gaining administrative privileges to a single machine is like in a safe environment. You can find the VM's download link and other information on vulnhub's [Kioptrix Level 1 page](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/).

__<center><font style="color:red" size="+2">SPOILERS AHEAD. IF YOU DON'T WANT THE ANSWER TO THIS CHALLENEGE SPOILED FOR YOU, DO NOT CONTINUE.</font></center>__

Once you've downloaded, configured and deployed the Kioptrix image on your prefered virtualization software, you can find its assigned IP address using Nmap's ping sweep flag: 

__nmap -sP 192.168.1.0/24__

Now that you know the Kioptrix host's IP address, you can launch a quick nmap scan to see what ports are open on it. Since we're performing these scans within the safety of our home LAN, we can be hasty with our scans.


__nmap -sV -p- -T4 192.168.201.132__

<pre>
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
</pre>

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

__<span style="font-style: 'consolas'">apt-get install libssl1.0-dev</span>__

With this missing library version installed, the OpenFuckV2 exploit should run without issues after compiling it with the indicated gcc flags.

__<span style="font-style: 'consolas'"gcc -o OpenFuckV2 764.c -lcrypto</span>

Once you've successfully complied the exploit, running it produces instructions on how to use it apropriately. It provides a list of possible OS version + Apache version combination that the exploit works for.

Goign from the nmap scan we already know 2 out of those things, the Apacher version running on the host and the OS that the server is running on, which is a RedHat Linux, although we do not know the exact version.

