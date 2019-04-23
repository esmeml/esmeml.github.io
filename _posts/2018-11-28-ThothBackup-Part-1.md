---
layout: post
title: ThothBackup - Part 1
date: 2018-11-28 16:30:00 -06:00
license: cc0
published: True
categories:
- ThothBackup
tags:
- BorgBackup
- Git
- Git-Annex
- Wasabi
- SyncThing
---
As many people may have guessed, this backup system very quickly got much larger
than I initially expected. Because of the size of the backup project, the number
of people interested, and how quickly things are changing along the way, I've
decided to approach this project in a new way.

In the sidebar to the left you will notice there is a new link to a "Projects"
directory. Here you will be able to find all my larger works. The project is
now called ThothBackup, and what follows is a list of things I have learned
along the way. All of this data will be consolidated and entered in a more
coherent fashion into the project pages, so keep an eye out for those to update.

But for now, we have a lot of ground to cover, so let's get to work.

#### Part 1 - Cross Platform is hard ####

One of the biggest parts of the backup project was its ability to be
cross-platform. I want the system to be easy enough to use that anyone and
everyone could grab a client, get it configured, and get going. To facilitate
this, the initial idea was to use tools that were built into the operating
system. On Linux and macOS this is easy enough, as `rsync` is installed on most
distributions by default, and if it isn't installed it is just a quick package
manager installation away.

Then however, entered Windows. Initially I assumed that it would be easy to use
with windows as well. After all, [rsync.net][1] has a nice little guide
explaining how to set it up. However, their client can detect when you're not
using rsync.net servers (which is totally fair, there's no hate from me on that)
and limits using the program to 30 days. The other alternative is cwRsync, which
was initially freeware, but has since changed to being a paid product. Obviously
asking someone to play for a program to even be able to start to use the backup
isn't a great selling point.

The first idea that I had was to write something on my own. Maybe have shell
scripts on all platforms check for required code and fetch anything that is
needed. However, shell scripts are hard for many people to debug, and the sight
of a command prompt can strike fear into the hearts of many windows users.

The second iteration of the idea was to write something in Python. However at
that point the client is becoming a software project in its own right, and I
didn't start this to develop software, I started it because I wanted to set up
a neat little backup service for my friends and family.

Thankfully, there are many other software suites that are both cross platform
and useful for this task. We ended up going with [SyncThing][2]. SyncThing is
an open-source (MPL2, which is a permissive form of copyleft) synchronization
library that is cross-platform and written in Go. I'm a huge fan of Go even
though I don't actually write it myself, as it is a fantastic language for
exactly this type of thing. Even better, SyncThing comes with easy to use and
easy to understand GUIs, and is capable of NAT and Firewall punching via relays,
and makes device configuration dependent on acceptance from both the server
and the client. The protocol it uses is open source, and based on the usage
reports at least one person is using it on 30 million files with 2,000 peers.
Last, but most certainly not least, traffic is encrypted with 128 bit AES,
and the protocol maintains perfect forward secrecy.

All of this (and a whole lot more, it really is an awesome bit of software)
makes SyncThing perfect for our use case. This may not always remain the case,
but it gives me somewhere to start. Even if we end up moving beyond SyncThing in
the future, you really should give it a look. It is a phenomenal piece of
software.

#### Part 2 - Changes in Sync Methods ####

As hinted above, the original plan to synchronize systems wasn't going to work
without more work than I was willing to put in to a single component of the
system. Once we threw out the initial way the system was supposed to work, we
had to retool the way things worked on the operating system too.

The original way the sync process was meant to work was that every user's
operating system would get its own Server Account, and rsync or some other
synchronization system would be tunneled through SSH. I wasn't sure if
authentication would be handled by system accounts or LDAP, because I never got
that far. But I did specifically pick the operating system (OpenSUSE) because
of that distribution's system configuration manager (YaST).

Now with the use of SyncThing, a daemon process would run under a single user,
and all clients would then connect to that daemon process which would then
write to disk using that daemon's permission set. Thus, no need to worry about
ACLs or anything of the like. It was interesting to work with ACLs though. You
can see some of my old code if you browse through the commits history of the
ThothBackup GitHub repository.

#### Part 3 - Filesystem Considerations ####

When I was testing the original synchronization strategy, I had everything being
deposited onto BTRFS subvolumes that were mounted with the `compress` option. To
be entirely honest, I wasn't that impressed with the way the compression was
working.

In the new system, BTRFS subvolumes are still being used (User, System Name,
Operating System, Drive Name, Backup Client, Archive Client, etc) except now
the subvolumes are mounted with `compress-force` option. Additionally, I have
learned about out-of-band BTRFS deduplication and plan to play around with that
at this stage in the project as well.

#### Part 4 - Operating Systems ####

I really, really like OpenSUSE. Like, a whole whole lot. It may very well be my
favorite binary distribution, and I've used quite a few. I think the whole way
it works is simply phenomenal, I like the company behind it, and it honestly
boils down to just that: I like it.

But, after all the changes above I began to consider if I shouldn't change
distributions. Originally I thought of changing to BSD, but I was concerned
about software availability. I know FreeBSD tends to have a very well maintained
ports collection, but I was still.. concerned. Most tools in this arena seem to
cater toward Linux, and if I was already changing multiple systems to avoid
having to write new software, did I really want to run the risk of needing to
write server side software?

After much deliberation, I ended up settling on Debian Stable with backports.
The initial installation is extremely lean, and there is a truly massive amount
of documentation available for Debian. It paid off well too. The initial install
of Debian stable clocked in at 60MB of ram used, where as OpenSUSE was running
around 200MB after reboot.

#### Conclusion #### {: #thoth-1-conclusion }

There is honestly still quite a bit more that needs to be discussed. One of the
most amusing things that the past week or so has taught me is that Sydney's
computer is as good of a backup test as a normal single family household. His
System has 4 drives, over one million files, a quarter of a million directories,
and about a terabyte of _used_ storage on it. Combining his single computer
with my mac and a windows virtual machine, and we have as much testing as we
could need.

#### Notes #### {: #thoth-1-notes }
I'm going to start including a little section at the bottom of each post to
remind me what I need to work on. Hopefully having this publicly viewable will
encourage me to actually follow through on writing more than one blog post every
18 days.

  * stage 1 project page
  * stage 2 project page
  * talk about security improvements that can be done
  * rewrite the server side new client script
  * talk about specific SyncThing configuration options used
  * write utility script to keep server config files up to date in git

[1]: https://www.rsync.net/resources/howto/windows_rsync.html
[2]: https://syncthing.net/
