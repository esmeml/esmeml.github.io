---
layout: post
title: ThothBackup - Part 2
date: 2018-12-02 20:30:00 -06:00
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
Hello! It's that time of the week again, where I update everyone on my latest
work. This episode is far less technical and focuses more on the concept of a
"One and Done" backup solution, aka the holy grail of data maintenance.

It fucking sucks.

### Introduction ### {: #thoth-2-introduction }
This entry is slightly unidirectional. The concept of a simple, easy to
implement, catch everything you might ever need solution is quite literally the
holy grail, yet it has never honestly been implemented. Sure, user data is
generally scooped out, but in the day and age of game mods, and with some
development projects taking place outside of the User directory, it seemed
prudent to at least _attempt_ the full backup. Well, I've been attempting it
for seven days. Here's what I've found.

### Focus ###
We will not be focusing on the space impact of a complete backup. This is
actually fairly negligible. With out-of-band deduplication, only one set of
operating system files would ever be stored, so server side storage would reach
a weird type of equilibrium fairly quickly. Instead, I'll talk about three
things:

  * Metadata Overhead
  * Metadata Processing
  * Initial Synchronization

There may be another post tonight talking about additional things, but this
deserves it's own little deal.

### Metadata Overhead ###
A fully updated Windows 10 partition of your average gamer, aka my fiancé, is
composed of __479,641 files__ and __70,005 directories__ which comprise a total
data size of __~216 GiB__. This is actually just the C drive and typical
programs. If you factor in the actual game drive in use by our test case, that
drive contains __354,315 files__ and __29,111 directories__ which comprise a
total of __~385 GiB__ of space.

In summation, an initial synchronization of what is typically considered a "full
system backup" comprises __833,956 files__ and __99116 directories__ comprising
__~601GiB__ which results in an average filesize of __~755KiB__ and an average
directory size of __~9 files__.

SyncThing creates a block store that is comprised of, by default, __128KiB__
blocks. This means that for our system, assuming the data is contiguous, we need
__4923392 Metadata Entries__. Assuming the files are NOT contiguous, this is
probably closer to about __5 Million__ metadata entries. As of right now, the
server side metadata storage for the testing pool is at __1.7 GiB__ and initial
syncronization is __not yet complete__. Extrapolating a bit, we can assume that
__2.0 GiB__ would not be an unreasonable size for a final server side data
store.

The client side store, at the time of writing, is approximately __1 GiB__ and
may grow slightly larger. However, I will use __1 GiB__. This means that there
is a plausible total of __3GiB of metadata overhead__ representing an overhead
percentage of __~0.5%__ across the pool. Scaling up, this means 10 clients
with 1TB of data each would require __51.2GB of Metadata__.

__Should anything happen to the metadata store, it would need to be rebuilt by
data reprocessing. This introduces a potentially massive liability, as scanning
frequency would need to be reduced to not impact the rebuild operation.__

### Metadata Processing ###
The server is capable of a hash rate of __107MB/s__. I am picking the server's
hash rate because it is both the slowest hash rate of the pool and would have
the most metadata that would need to be rebuilt.

For a complete rebuild of the data of our current cluster, it would take the
server __~96 Minutes__ during which no data synchronization could occur. This
equates to a minimum of __1 Missed Hourly Update__ and could potentially result
in up to 2 missed hourly updates if the timing was unfortunate enough.

For a complete rebuild of the data of our theoretical cluster, we will allow for
a hash rate of __300MB/s__. The total data needed to be rebuilt would be 10TB.
This would result in a database rebuilt time of __~10 Hours__ which could result
in up to 11 missed synchronization attempts.

### Initial Synchronization ###
The initial syncronization is composed of three primary parts. First, the
client and host must agree on what folders to syncronize. Second, the client
must build a database of the content hosted locally. Next, utilizing a rolling
hash algorithm, data is entered into the metadata cache and transmitted to the
server.

_Per the developer of SyncThing, millions of small files are the worst case
scenario for the backup system._ As of my independent, albeit anecdotal testing,
After 7 days the synchronization process is still in effect. This represents a
very __poor user experience__ and would not be ideal for a widespread rollout.

### Conclusion ### {: #thoth-2-conclusion }
The primary goal of a backup utility is to synchronize files and achieve cross
system consistency as quickly as possible. While it is true that eventually
consistent systems are utilized in large scale operations, this type of
consistency is allowable only, in my opinion, at data sizes over 10TB. The
current testing set is approximately 1TB at most, and thus this is unacceptable.

__Either the backup paradigm must change, or the utility used to implement it
must change.__ While I do not expect to find any faster utilities for performing
the backup process, I do plan to continue to experiment. At this time, however,
it seems that the most likely way to make the process as friendly as possible
would be the implementation of a default backup subset, with additional data
added upon user request, and after the high priority synchronization had been
completed.
