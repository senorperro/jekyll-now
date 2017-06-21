---
layout: post
title: Kioptrix 1 Walkthrough
---

While going around online looking for something to keep my pentesting skils sharp, I came across the [Kioptrix challenges](http://www.kioptrix.com/blog/), which are a series of VMs with security misconfigurations so people can attempt to gain root access by whatever viable means they find (exploiting the virtualization software itself doesn't count!).

![_config.yml]({{ site.baseurl }}/images/newKioptrix_1.jpg)

Target Audience Level: [■ ■] Easy*

\* I say "easy" because, through personal experience, it seem like compilig the specific exploit used as part of this exercise might become  less straightforward as time passes and the libraries it uses are further updated.

The Kioptrix challenges are targeted towards folks trying to get an idea of what gaining administrative privileges to a single machine is like in a safe environment while also not losing their sanity in the process. You can find the VMs download link and other information on vulnhub's [Kioptrix Level 1 page](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/).

***SPOILERS AHEAD. IF YOU DON'T WANT THE ANSWER TO THIS CHALLENEGED SPOILED FOR YOU, DO NOT CONTINUE.

Once you've downloaded, configured and deployed the Kioptrix image on your prefered virtualization software, you can find its assigned IP address using Nmap's ping sweep flag: __nmap -sP 192.168.1.0/24__ <span style="color:red">(remember to use the IP address that belongs to your subnetwork)</span>