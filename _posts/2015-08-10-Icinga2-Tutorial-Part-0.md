---
layout: post
title: "Icinga2 Tutorial: Part 0 - Network Monitoring for the Masses"
date: 2015-08-10 01:00:00 -05:00
license: cc0
categories:
- Tutorials
- Networking
tags:
- Icinga2
- IcingaWeb2
---
* [Icinga2 Tutorial: Part 1 - Installation and Configuration][1]
* [Icinga2 Tutorial: Part 2 - Agent-Less Checks][2]
* [Icinga2 Tutorial: Part 3 - Agent-Based Checks][3]
* [Icinga2 Tutorial: Part 4 - Expanding Checks to SNMP][4]

__EDIT (2018/12/09):__ _These guides haven't been updated since 2015. It is
possible that there are dead links, or that the configuration syntax has changed
dramatically. These posts are also some of the most popular on my blog. I plan
to do a new guide eventually, but for right now please take the following
entries with a grain of salt._

## Introduction ## {: #icinga2-part-0-introduction }
This will be a bit more informal than most of my posts as this is more of a
hobby project than anything with really standardized applications. I’ve been
exploring the network protocols available to me on my EdgeRouter now for the
past year, and last night I sat down and taught myself SNMP. After three
hours and poking around the MIBs, I realized that I absolutely hate SNMP.
That being said, I very much like the idea of it. I have to use a CLI and
a GUI on the same device most of the time to watch all the stats I like to
watch, and I immensely dislike putting that kind of rendering work on my
EdgeRouter, because I can assure you, the poor thing is taxed enough.

So I started exploring multi-protocol management systems today. Clearly,
Spiceworks is one of the best programs you can get. However, it is Windows
only. Here on Linux, I really only have NAGIOS, or so I thought.

## Enter Icinga2 ##
Icinga2 is a very interesting program to me in the same way that Maxthon
was a very interesting browser. Both Maxthon 2 and Icinga 1 were built on top
of existing applications (Internet Explorer and NAGIOS, respectively) with the
intent of expanding their functionality. I know a good many people who would
insist that NAGIOS has all the functionality you could ever need, and I would
agree, except for one thing. If I want ease of configuration, I have to pay
[$2000USD][5] for my small home network. I don’t have that kind of money, and
if I did, you can bet your ass it would go to getting me embedded development
boards or external controllers for debugging and programming. Anyways, then
Maxthon 3 came out, and it stepped away from the Trident engine, and went with
Google’s WebKit. Just like Maxthon, Icinga2 has stepped away from NAGIOS and has
become its own solution, but still remains compatibility with NAGIOS monitoring
plugins.

## Purpose of These Posts ##
At first this was just going to be something fun for me to do. However, I have
noticed a somewhat disturbing lack of end to end documentation for Icinga.
Yes, all the information is there in the [manuals][6], but it is still nice to
see how someone sets it up from end to end. This series of posts is going to
show how to do just that, starting with how to install Icinga, which will go up
in the next post.

## Some Things to Consider ##
This is my first time using Icinga2. You will be learning right along with me.
Which means some posts may spend a lot of time going back and working on things
that should have already been done. Besides that, I am not using an Icinga
dedicated machine, which means I am running a desktop release, Linux Mint
Debian Edition 2 to be exact. I like 2. It is a good number. Also the
distribution is really nice, and DOESN’T have systemd, but
that’s a post for another day.

Additionally, my deployment is rather small. All told I will be monitoring
about 10 devices at most across four or five protocols, a fairly standard home
network. I will keep adding updates the more I learn about it, or if I add more
services. However, you may need to do some extra research to tweak things for
your deployment.

I do plan to branch off into Netflow at some point. So you can expect
that in the future.

[1]:  {% post_url 2015-08-11-Icinga2-Tutorial-Part-1 %}
[2]:  {% post_url 2015-08-12-Icinga2-Tutorial-Part-2 %}
[3]:  {% post_url 2015-08-17-Icinga2-Tutorial-Part-3 %}
[4]:  {% post_url 2015-09-07-Icinga2-Tutorial-Part-4 %}
[5]: https://assets.nagios.com/handouts/nagiosxi/Nagios-XI-2014-Pricing-Documentation.pdf
[6]: https://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc
