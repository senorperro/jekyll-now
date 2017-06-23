---
layout: post
title: Kioptrix 1 Walkthrough - Compilation Errors from Hell
---

While going around looking for something to keep my pentesting skils sharp, I came across the __[Kioptrix challenges](http://www.kioptrix.com/blog/)__, which are a series of VMs with security misconfigurations so people can attempt to gain root access by whatever viable means they find (exploiting the virtualization software itself doesn't count!).

![_config.yml]({{ site.baseurl }}/images/newKioptrix_1.jpg)

__Target Audience Level:__ [<span style="color:green">■ ■</span>] Easy*
<font size="-1">* Note: I say "easy" because, through personal experience, it seems like compilig the exploit used as part of this exercise might become less straightforward as time passes and the libraries it uses are further updated.</font>

The Kioptrix challenges are targeted towards folks trying to get an idea of what gaining administrative privileges to a single machine is like in a safe environment while also not losing their sanity in the process. You can find the VMs download link and other information on vulnhub's [Kioptrix Level 1 page](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/).

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

We can see that
