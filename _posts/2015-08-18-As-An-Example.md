---
date: 2015-08-18 23:15:45 -05:00
layout: post
license: cc0
title: Just as an example..
---
of what I mean when I say I spend my time meddling in software or hardware
that I really don’t have any business meddling in, I’ve got this website
split into two branches now: Master, and Development. I do all my work in
development, then push to remote. Travis-CI then pulls the changes and builds
them, then runs HTML-Proofer over the output, and then sends me an e-mail
if something is broken. If nothing is broken, then I rebase my Development
branch on master, and then merge it into master to make sure my changes are
preserved.

I guess it should read “I meddle in software that I don’t need in the
slightest.”
