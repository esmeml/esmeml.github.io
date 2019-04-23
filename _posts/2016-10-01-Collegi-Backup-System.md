---
layout: post
title: The Collegi Pixelmon Server Backup System
date: 2016-10-01 22:30:00 -05:00
license: cc0
categories:
- Work
tags:
- Collegi
---
Wow, time flies. It has been almost a year since I last updated this blog,
including fixing some of the issues that Jekyll 3.0 introduced in my formatting.
Luckily, that could be fixed by just adding a few spaces. In the past year,
quite a bit has happened, but nothing quite so exciting as becoming a co-owner
and the head developer of a new Minecraft community called [Collegi][1]. Collegi
is a [Pixelmon][2] server, which means we have Pokemon right inside Minecraft.
However, we strive to make the server Minecraft with Pokemon, instead of Pokemon
in Minecraft. It's a small difference, but one that we happen to find very
important. We want the survival aspect of the game to be front and centre.

The server has become absolutely massive, with each downloaded snapshot running
about 100GB in size. (Note, that throughout this article I will be using the
SI standard GB, which is 10<sup>9</sup>, versus the Gibibyte which is
2<sup>30</sup>, how hard drive manufacturers were allowed to change the value of
a gigabyte is something I will never understand.)

Now, with a 500GB flash drive on my MBP, I don't really have the room to save
all of those snapshots, especially considering we have snapshots going back six
months, across three different major versions of Minecraft. In fact, completely
expanded, the current backup amount at the time of writing is 1.11TB.

So, I began to search for a method of performing backups. I had some rather
strict requirements for these backups, that lead to the formulation of the
system I am going to discuss in this article.

__Requirements__

* Incremental FTP
* Deduplication
* Compression, and the ability to modify compression levels on the fly.
* Checksumming to silently detect corruption.
* Encryption
* Tools need to be actively maintained and ubiquitous.
* Able to sync repository with a remote source.
* Cheap
* Open source wherever possible.
* Easy to access archived versions.
* Must be able to be automated.
  * If not in setup, then in how it runs later.

## Step One - Getting the Data off the Server ##
We use a lovely company called [BisectHosting][3] to run our server. They
provide an extremely barebones budget package that gives us a large amount of
our most important specification: RAM. We can live without fancy support tickets
or SSD access if they offer us cheap RAM, which they do. Beyond that, however,
they also offer unlimited disk space, as long as that disk space goes towards
the server itself, so no keeping huge numbers of backups on the server.

Now, they did offer a built in backup solution, but it only keeps the past seven
days available in a rolling fashion, and I really really like to keep backups.

The only real gripe I have about BisectHosting is that they only allow the use
of FTP for accessing data on the Budget Server tier. Worse, they don't even use
FTP over TLS, so the authentication is in plain text. However, I just change my
password weekly and it seems to work alright.

The most important part of getting the data off the server is only getting the
new data, or the data that has changed. This requires using an FTP Client that
is able to sanely detect new data. Checksums aren't available, but modification
date and file size work just as well.

There were a large number of clients that I tried out over time. [Filezilla][4]
was the first of those. It seemed to work alright for a time, except that when
you have a large amount of identical files (We have 15,824 files at the time of
this writing) it hangs. Now, it does come back eventually, but it's still not
the best of features to have a client that hangs.

The next one I tried was a Mac favourite known as [Cyberduck][5]. I really liked
the interface for Cyberduck, but the first nail in its coffin was the inability
to perform a modification time comparison and a file size comparison during the
same remote to host sync. That meant it took two syncs to grab everything up to
date, and even then it didn't always seem to take. During the time that I was
using Cyberduck, we had to restore from backup for some reason that is currently
eluding me, but when we did so we noticed that some recent changes on the map
hadn't synced properly. Combine all of the above with the fact that from time to
time it would hang on downloads (I'm assuming from the absurd number of files)
and that wasn't going to work.

The final GUI client that I tried was called [Transmit][6]. I really, really
enjoyed using Transmit. It is a very polished interface, but first off it
isn't free, or open source, so that invalidated two of the requirements.
However, if it worked well enough, I was willing to overlook the issues. Problem
was, it didn't work well. I forget what happened at the moment, but I know that
it experienced similar hanging to Filezilla.

Regardless, Transmit was the last GUI based client that I tried. It took me a
bit to realize, but if I used a GUI client there was a very minimal chance that
I would be able to automate the download.

That left command line tools, which after I found [LFTP][7] I kicked myself for
not looking into first. In addition to being an open source tool, LFTP has the
ability to perform multithreaded downloads, which isn't common in command line
clients. Furthermore, it was able to compare both modification time and file
size simultaneously, reducing the sync operations needed back to one. It is
actively maintained, available in [Homebrew][8] (though, at the time of writing
 it has been moved into the boneyard), written in C, and very easily scriptable.
You can call commands that would normally have to be ran from inside the FTP
client directly from the command line invokation of LFTP. It handled our data
quantity flawlessly, and easily worked through the large amount of files, though
it can take quite a while to parse our biggest directories. At the time of
writing, that directory is the map data repository for our main world, which has
12,567 items clocking in at 88.15GB. It takes between two and five minutes for
LFTP to parse the directory, which considering all the other benefits is fine
by me.

Our remote to local command utilizes the LFTP mirror function, and from within
the client, looks like this:

    mirror -nvpe -P 5 / ~/Development/Collegi/

## Step Two - Convert the Data to an Archive Repository ##
When you are talking about a server that a full backup runs 100GB, and you want
to perform daily backups at minimum, it becomes absurd to think that you could
run a full backup every day. However, the notion of completely incremental
backups is far too fragile. If a single incremental backup is corrupted, every
backup after it is invalid. More than that, to access the data that was on the
server at the time the incremental was taken would require replaying every
incremental up to that point.

The first solution I tried for this problem was to use [ZFS][9]. ZFS solves
almost every problem that we have by turning on deduplication and compression,
running it on top of Apple's [FileVault][10], and utilizing [snapshots][11]. The
snapshots are complete moments in time and can be mounted, and they only take
up as much space as the unique data for that snapshot. Using ZFS Snapshots, the
1.10TB of data we had at that time was reduced to 127GB on disk. Perfect. The
problem becomes, however, offsite replication.

Now, it is true that by having a copy of the data on the server, one on my
MacBook, and one on an external drive here at the house, the [3-2-1 Backup][12]
rule is satisfied. However, three backups of the data is not sufficient for a
server that contains over six months of work. It's reasonable that something
cataclysmic could happen and we'd be shit out of luck. We needed another offsite
location. The only such location that offers ZFS snapshot support is
[Rsync.net][13] which 100% violates the "Cheap" requirement mentioned above.
That's not a knock on their service, Rsync.net provides an incredible service,
but for our particular use case it just wasn't appropriate.

So the hunt began for a deduplicating, compression based, encrypted backup
solution that stored the repository in standard files on a standard filesystem.
The final contenders were:

  * Using plain old [Git][14]
  * [BUP][15] (As a side note, this client has the most adorable name for a
    backup utility that I have ever seen. I love it.)
  * [Attic Backup][16]
  * [BorgBackup][17]
  * [git-annex][18]

I was leaning very, very heavily toward BUP until I discovered BorgBackup. My
primary concerns with BUP was that it did not seem to be under active
development, and after over five years it still had not reached a stable 1.0.
Git would have been useful, but just like ZFS it would inevitably require a
"Smart Server" versus the presentation of just a dumb file-system.

BorgBackup sold me almost immediately. It allowed you to mount snapshots and
view the filesystem as it was at that time, it offers multiple levels of
compression ranging from fast and decent to slow and incredible, and it has
checksumming on top of HMAC encryption. It's worth noting at this time that
nothing on the server is really so urgent as to require encryption, as most of
the authentication is handled by Mojang, but I still prefer to encrypt things
wherever possible.

It was under active development, it's developers were active in the community
(I ended up speaking with the lead developer on twitter), and it was progressing
in a sane and stable fashion. As an added bonus, the release of 1.1 was to
provide the ability to repack already stored data, allowing us to potentially
add a heavier compression algorithm in the future and convert already stored
data over to it.

The only downside to Borg was that at first glance it seemed to require a Smart
server, just like git would.

Regardless, the system would work for now. If worst came to worst, I could
utilize something like [rclone][19] to handle uploading to an offsite location.

When everything was said and done, we had reduced the size of our 1.11TB backup
into a sane, usable 127GB.

The current command that is used looks like this:

    borg create --chunker-params=10,23,16,4095 --compression zlib,9 --stats \
        --progress /Volumes/Collegi/collegi.repo::1.10.2-09292016 .

## Step Three - Offsite Replication ##
I could easily spend a very long time here discussing how I chose the cloud
provider I would inevitably use for this setup, but it really comes down to
the fact that I quite like the company, and their cloud offering has a very
complete API specification, and is dirt cheap. We went with [BackBlaze B2][20].
I could, and probably will, easily write a whole separate post on how enthralled
I am with BackBlaze as a company, but more than that their $0.005/GB/Month price
is literally unbeatable. Even [Amazon Glacier][21] runs for $0.007/GB/Month and
they don't offer live restoration. It's cold storage as opposed to BackBlaze's
live storage.

The problem became this: How do I get the Borg repository to fully sync to B2,
but do so in such a way that if the local repository ever became damaged I could
pull back only the data that had been lost. This is what the
[documentation for Borg][22] means when it mentions you should really think
about if mirroring best meets your needs, and for us it didn't.

Again though, B2 is just a storage provider, not a smart server. So how do I set
things up in this way? The answer became to use another tool that was almost
used for backup in the first place, Git-Annex. The only reason git-annex wasn't
used for backup to begin with is that it doesn't allow us to retain versioning
information. It just manages large files through git, which wouldn't work.
What it would do, however, and do quite well, is to act as a layer between our
BorgBackup repository and the cloud.

So, I stored the entire borg repository into git annex. Once this was done, I
used a [plugin][23] for git-annex to add support for a B2 content backend. Then,
the metadata information for the git repository gets synced to [GitLab][24], and
the content is uploaded to B2.

## Conclusion ## {: #collegi-backup-system-conclusion }
The end result of this is that our 100GB server, as it stands at any day, is
mirrored in four separate locations. One on the host itself, one on the MBP
hard drive, one in the Borg Repository, and one on the BackBlaze B2 Cloud. More
than that though, we have a system that is easily automated via a simple shell
script, which after completing the initial setup (sending 20,000+ files to
Backblaze B2 can take a while), I will demonstrate here.

Thank you so much for reading, I look forward to sharing more about the inner
workings of the Collegi Infrastructure as time permits.

## Video ##
I just recently completed an asciinema of the process. See below. Also note
that you can copy and paste commands from inside the video itself. Go ahead, try
it!

<script type="text/javascript" src="https://asciinema.org/a/14pvurnnazr6res7f5u1yvg7u.js" id="asciicast-14pvurnnazr6res7f5u1yvg7u" async></script>

[1]:  https://collegi.enjin.com "Collegi Pixelmon Main Website"
[2]:  https://pixelmonmod.com/ "Pixelmon Mod Main Website"
[3]:  https://www.bisecthosting.com/ "BisectHosting"
[4]:  https://filezilla-project.org/ "Filezilla Project"
[5]:  https://cyberduck.io/ "Cyberduck Website"
[6]:  https://www.panic.com/transmit/ "Transmit Website"
[7]:  https://lftp.yar.ru/ "LFTP Homepage"
[8]:  https://brew.sh/ "Homebrew Website"
[9]:  https://en.wikipedia.org/wiki/ZFS "ZFS on Wikipedia"
[10]: https://en.wikipedia.org/wiki/FileVault "FileVault on Wikipedia"
[11]: https://docs.oracle.com/cd/E23824_01/html/821-1448/gbciq.html "Oracle's Documentation on ZFS Snapshots"
[13]: https://rsync.net/ "Rsync.Net Homepage"
[14]: https://git-scm.com/ "Git SCM"
[15]: https://bup.github.io/ "Bup: It Backs things Up!"
[16]: https://attic-backup.org/ "Attic Backup"
[17]: https://github.com/borgbackup/borg "BorgBackup"
[18]: https://git-annex.branchable.com/ "Git-Annex"
[19]: https://rclone.org/ "Rclone"
[20]: https://www.backblaze.com/b2/cloud-storage.html "BackBlaze Cloud Storage"
[21]: https://aws.amazon.com/glacier/ "Amazon Glacier"
[22]: https://borgbackup.readthedocs.io/en/stable/faq.html#can-i-copy-or-synchronize-my-repo-to-another-location "Borg Documentation: Can I sync my Repository to another Location?"
[23]: https://github.com/encryptio/git-annex-remote-b2 "Git-Annex B2 Backend"
[24]: https://gitlab.com/ "GitLab"
