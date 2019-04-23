---
layout: post
title: Collegi Pixelmon - Backup System Part 2
date: 2016-10-13 12:00:00 -05:00
license: cc0
categories:
- Work
tags:
- Collegi
- Git
- Git-Annex
---
What was originally intended to be a one off blog post may become my new source
of material for the coming weeks. After utilizing [BorgBackup][1] and
[git-annex][2] to backup what has now grown to almost [2.5 Terabytes][3] of
data, I began to wonder what other ways I could put git-annex to use for us here
at [Collegi][4]. We already use various [GitLab][5] repositories to manage
different facets of the project, and I began to wonder if there wouldn't be some
way to use git-annex to completely unify those repositories and distribute their
information as needed.

This started as a brief foray into [git submodules][6] which, while allowing me
to consolidate data **locally**, does nothing in helping me to properly
redistribute that data to various locations. The only way that it would be
possible to do such a thing would be to take all the various git repositories
that Collegi utilizes, which currently is sitting at [six total][7], including
the git-annex metadata repository (which isn't publicly visible), and merge them
into one master repository through the use of [git subtrees][8]. This would
allow me to still have multiple repositories for ease of project management, but
all those repositories would be pulled down, daily, to a local "master"
git-annex repository and merged into it.

Once this was done, the use of git annex's [preferred content][9] system would
allow me to decide what data needed to be sent to which remote. This would let
me back up some information to one remote, and other information to another.
As an added bonus, the use of git subtrees would even allow me to push changes
back upstream, and all of it would be centralized.

In the future, this would allow us to push very specific data to specific team
members, who would then modify the data, which would be pulled back down on the
next git-annex sync, we would see changes needing to be pushed upstream had been
made, unlock those files, then use git subtree to push them back to their
remotes. That's the theory at least. As far as I am aware, either no one has
done this before, no one who has done this before has lived to tell the tale, or
no one who has done this before has blogged about their experiences in doing so.

That's where this blog comes in. I'm currently in the process of making a
complete copy of the current root repository, which is still using git
submodules, and from there I can begin experimenting. Whether or not this works
remains to be seen, but it coincides neatly with a rewrite of the
[backup script][10] to update it to Google Shell Style Guidelines, which means
I can build the script around the new repository layout, and while doing so I
should be able to head off any unforeseen issues.

It's very likely that I am going to finish writing 2.0 of the script before
doing any of this crazy shit, but this post helps me to organize my thoughts.
Besides, it just means 3.0 will be that much more exciting when it drops.

Stay tuned for more of my antics and adventures with making this absurd system
take shape, and turn into the omnipresent repository of every single facet of
a Minecraft community.

[1]:  https://github.com/borgbackup/borg "BorgBackup"
[2]:  https://git-annex.branchable.com/ "Git-Annex"
[3]:  https://twitter.com/Zyradyl/status/786205897810780160 "Zyradyl's Twitter"
[4]:  https://collegi.enjin.com "Collegi Pixelmon Main Website"
[5]:  https://gitlab.com/ "GitLab"
[6]:  https://medium.com/@porteneuve/mastering-git-submodules-34c65e940407#.24xm3mdlt "Mastering Git Submodules"
[7]:  https://gitlab.com/groups/collegi "GitLab: Collegi Group"
[8]:  https://medium.com/@porteneuve/mastering-git-subtrees-943d29a798ec#.r5p4ozyfm "Mastering Git Subtrees"
[9]:  https://git-annex.branchable.com/git-annex-preferred-content/ "Git Annex: Preferred Content Manual Page"
[10]: https://gitlab.com/collegi/collegi-backup-automation "GitLab: Collegi Group - Backup Automation"
