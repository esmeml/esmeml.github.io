---
layout: post
title: "HughesNet Gen4 and IPv6 PMTUD: A Tragedy"
license: cc0
date: 2015-08-22 14:50:00 -05:00
categories:
- Networking
tags:
- IPv6
- HughesNet
---
So before I went to bed last night I started experiencing some very odd issues
with my connection. I could connect to Skype, but I couldn’t visit twitter.
I could talk to Google, but not GitHub. I was able to ping my HT1100 gateway,
but my Icinga2 monitoring system reported a socket timeout of longer than 10
seconds on HTTP.

I spent about three hours on it before I finally went to bed. I even went as
far as to spend five more dollars to purchase a 500MB token in order to see
if maybe I was being penalized for using too much data in my throttled state,
as I have been making use of aria2 to manage large downloads that would
otherwise suffer from a mysterious decryption failure when I was downloading
via HTTPS. Didn’t help.

This morning I bit the bullet and changed my LAN’s MTU from 1280 to 1500.
You may be wondering why I refer to this as having to bite the bullet, since
an MTU of 1500 is standard. Well, come to find out that PMTUD is broken on
HughesNet. Something about what is done on the HughesNet side causes the
packets to become too large. Now, IPv6 is supposed to handle this by sending
ICMPv6 Type 2 “Packet too Large” notices to the end point. While debugging
PMTUD the other day (about a week ago) I set up my firewall so that **ALL**
ICMPv6 is allowed, rather than having to itemize the different types. Still,
no good. I was getting silent failures that I had to use test-ipv6.com to
resolve, and they still indicated packets were becoming too large for my
connection. In a fit of irritation, I set my entire LAN to an MTU of 1280.
Eureka, I have IPv6 functionality.

Obviously after last night, I can’t exactly use that solution anymore, for
a reason I have yet to understand. So my MTU is set back to 1500, and without
using HughesNet’s squid proxy (what they call web acceleration), IPv6 fails.
Oh, I didn’t mention that did I? Yeah, their web acceleration lets IPv6 work.
However, I run my own squid proxy, locally, which is even faster than theirs.
It also saves me bandwidth. So I keep web acceleration off.

 * Web Acceleration On = PMTUD works, IPv6 BUT I have double caching. Not good.
 * Web Acceleration Off = IPv6 Fails. Not acceptable.

Just to see if I could get something reset in the modem, I even called tech
support. The solution of “It is a problem with your LAN” was obvious, but still
frustrating due to the issues described above. However, I am planning to reach
out to their support today via social media, so I am pushing this article live,
without links, just so I have something to refer to. I will come back to link
to the major terms in this a little later. Wish me luck.
