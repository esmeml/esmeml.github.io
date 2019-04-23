---
layout: post
title: Collegi Pixelmon - Developer Log
date: 2016-10-14 03:00:00 -05:00
license: cc0
categories:
- Work
tags:
- Collegi
- Git
- Git-Annex
---
The following post is an extreme rough draft. In fact, it isn't even actually a
post. These are my development notes from my refactoring of the collegi data
infrastructure. As such, they're arranged in no real sensible order besides
having been written chronologically. Additionally, these have not been
proofread, grammar checked, copyedited, or spell checked, as i write them in an
IDE and not an actual text editor. As such, please don't judge my writing
ability off of them. More importantly, however, these do not have the
standardized links that i provide to new concepts or commands in my blog posts,
as embedding links to things I already know or have access to in a developer log
that on average no one else sees just seems silly.

So, if you have questions, use google, and expect these to be updated over time.

The logs as of this posting run from 10/13/2016 to 10/16/2016, so over three
days of work. There is a -LOT- more to be done.

They are broken down into the following format. Each list is a set of specific
actions I took, and sometimes the list ends up with notes in it because, again,
no one generally sees these, but under the task list is the space reserved for
notes on the above list. Then a new task list is declared, then notes, then
tasks, and so on and so forth. Generally each new task heading would signify
a new blog post, talking about the tasks and the notes, so keep that in mind.

These were requested by Kan, a player on our server. Enjoy!

### Tasks ###

* Made a backup of the repository as it stood on 2016-10-13 in the event
anything breaks too badly during this.
* Removed all existing submodules from the `git` repository. Committed the
removal.
* Ran the previous backup script to make sure that 10/13 was backed up. This
included new additions to git annex.
* Forced git annex to drop the old SHA256E key-value backend files that were
made obsolete by the conversion to SHA512E key-value backend.

_**Notes 1:** During this time, and while watching the way the version 1.0
backup script ran, I noticed there is a significant performance penalty for
moving the location of the local mirror. Borg uses the entire path as the file
name, so any deviation in the path spec causes it to treat the files as brand
new. Note that this does not cause any issues with de-duplication, but the
process of adding these files causes a massive performance hit. This made me
start thinking about including the local mirror in the git annex so that as
long as the annex was kept in tact in regards to metadata, the paths would
remain the same as all additions to Borg would take place from the same root
directory._

_The problem with this would be the fact that annex keeps everything as
symlinks. As such, I am looking into the unlock feature of version six
repositories._

_**Notes 2:** Dropping unused from a local area goes -much- faster than
dropping from remote. Who knew, right?_ :tongue:

### Tasks ###

* `git-Annex` drop completed, but Finder isn't showing a reduction in used drive
space, but I think this is more an error on the side of finder than something
with git annex, as `du -h` showed the directory was down to the size it should
have been. Once I manage to get this finder thing figured out, I'll move on to
the next part.
* Finder is taking too bloody long to figure its shit out, so I moved on to the
next step in cleaning up the repository. I'm rewriting the commit history
to completely remove files I don't need from the actual `git` repo. In theory
this shouldn't touch `git-annex` at all, but that remains to be seen.
* Ran BFG Repo Cleaner on the following directories and files:
  * collegi.web
  * collegi.pack
  * collegi.git
  * .DS_Store
  * .gitmodules
  * collegi.logs (Just for a moment, and we made backups.)
  * collegi.configs
* Ran filter branch to purge any empty commits left after the above.
* Expired original ref-logs, repacked archive.

_**Notes 3:** At this point we had gone from 230 commits to 102 commits. We were
also left with the original envisioning of what this repo would be, which was
a simple git annex to push files to Backblaze b2 from the Borg repository. Now
to verify that all of our data is still 100% ok._

### Tasks ###

* Ran `git fsck`
* Ran `git annex fsck`

_**Notes 4:** Wow this is going to take a long fucking time. Who woulda thunk
it._

_**Notes 5:** So apparently the current version of `git-annex` is using the old
mixed hashing method, which is a format that "we would like to stop using"
according to the wiki. Might need to migrate. Need to figure out how._

_**Notes 6:** From the wiki: "Initial benchmarks suggest that going from xX/yY/KEY/OBJ to xX/yY/OBJ directories would improve speed 3x." It's worth
migrating._

### Tasks ###

* Run `git annex uninit`
* Reading through the `git-annex-init` man page to see what else we should
change now since we're already migrating. Post Uninit we're going to have to
run a full borg data consistancy check.

_**Notes 7:** Ugh. The document I found was actually an theoretical one, and
while it is true that `git-annex` does use the new hashing format in bare
repositories there is no actual way to move to the new one in a regular repo.
So I am running an `uninit` for basically no reason. The only good thing about
this that I can think of is that I will be able to reform the final `git-annex`
repo in a much saner fashion. The bad news is that I have lost the log files,
unless `git-annex` is going to bring those back for me. I am annoyed._

_**Notes 8:** Good news! I just remembered that I had made a `rsyn`ced backup
of the repository before I started fucking with it. So I didn't actually lose
the log files, I just went ahead and pulled them out of the `git-annex`
backup._

### Tasks ###
* After the git annex had uninitialized, I decided that if I was going to do
this whole damn thing over again I was going to do it right.
* Started a new `borg` repository in new-collegi. Pulled out contents from the
original `borg` repository, using backups to restore any files that got hit in
the above clusterfuck, then recompressed with maximum LZMA compression.
* During this period I also standardized how the `borg create` paths would work.
The server would exist within a collegi.mirror directory, and the entire
directory would be added to `borg` upon each run of the backup script. This
effectively means we never have to worry about the LZMA penalty discussed below
again after the first re-add, unless we do major server restructuring, because
paths will remain stable between commits.

_**Notes 9:** The initial speed penalty for using LZMA is absolutely jaw
dropping. One `borg create` took eight hours to complete. Eight. However, I
quickly noticed that due to Borg's de-duplication mechanism, the add times got
faster the more data I added, and gzip-9 to lzma-9 did actually yield some
improvement. It also reduces the incentive for me to do this fucking disaster
again, because of how much it absolutely fucking sucks._

_**Notes 10:** As an example of what I mean by the above, the initial adding of
1.8.9 took six hours with LZMA-9. When the map was changed from NewSeed over to
Collegi, it took another four hours just to update the paths and what not, even
though the data hadn't updated, just the paths have changed. (This is indicated
by the fact that the total repository size barely increased, all the size that
changed could be explained by new metadata.) However, when the paths are kept
the same, adding 100GB of data takes 13 to 15 minutes. So, the benefit of
LZMA-9 is worth the initial startup, imho._

_**Notes 11:** `borg extract`ing from the GZIP-9 archives takes about 40
minutes, and that's from highly de-duplicated and GZIP-9 archives. What this
means is that pulling from an lzma-9 is probably going to take about an hour,
depending on just how de-duplicated the archive is (as in, how many different
chunk files contain parts needed to reassemble the original content)._

_**Notes 12:** Have hit the series of backups where things have moved into the
Users path, and I'm restructuring them. It made me think about how I will handle
the mirror directory in the future. I think I am going to do a few new things
with respect to the new setup. The mirror directory will be a part of the
`git-annex` repository, so there will be a new folder inside it called
`collegi.mirror` or something similar, and then I can move the new backup
script to be ran from the root directory, which will be beneficial. That way
everything is neatly packaged. the issue becomes mirroring this, because
uploading that much constantly changing data to backblaze would be literally
stupid, and not at all within our budget. What I will likely do is initialize
a "bare repository" on my time machine drive, and mirror the entirity of the
`git-annex` repository to that._

### Mandatory Break Notes ###

* You need to run borg info to make sure the latest creation thingy is the
proper size, and a borg check might not be a bad idea either as you fell asleep
and closed the mac during work on the repo.
* Cleaned the time machine volume of the repeated backups of the new repository
because it doesn't make any sense to have 20 versions of it.
* Moved the repo to the time machine drive as temporary storage using rsync.

### Tasks ###
* Restarted the transfer process starting on the 8th of October

_**Notes 13:** Not a huge shock but running some of these commands across USB
2.0 can add anywhere from 10 to 30 minutes. Doing them cross device gets even
worse, with some transactions taking almost an hour._

_**Notes 14:** I've been going back and forth on what filesystem I would like
to deploy since I am redoing the collegi drive as a whole. Now the interesting
thing to note here is that by the time I get this thing fully ready to deploy,
the drive I have here may not be the drive it ends up on, but this is as good
of a testbed as any. I'm really thinking I will go with apfs. Most of the
gripes I have with it are easily resolved through borg and git annex._

_**Notes 15:** In a highly amusing turn of events, it is bigger in lzma 9 than
it was with gzip 9. weird._

_**Notes 16:** While it would likely be prudent to go back to the previous
compression method, the benefits that I have made to the directory structure
while redoing the borg repository are worth the few extra gigabytes of overhead
especially concerning with Backblaze B2 it barely costs a penny._

### Tasks ###

* Use JHFSX for the new drive. I would have really liked to use APFS but I am
still worried about the data loss considering there is almost a year till it
will ship. JHFSX is reasonable enough for right now, while still being safe to
unplug.
* I went round and round on using encryption on the new drive. did it.
* using rsync to bring the data to its final resting location.
* OK started setting things up
* Defined gitlab as the metadata backup again
* created a bare repository on skaia
* set up prefered content so skaia requires everything in the main repo
* set the main repo to require a --force to drop content via preferred content
* Set the backend to SHA512E
* began the long process of adding the data to the git-annex
* Set up bin directory to not be tracked by git-annex but instead by git
* added backblaze remote, not encrypted, with a proper prefix
* started to sync to backblaze
* noticed an issue with how the sync was going to gitlab, will correct.
  * corrected the issue
