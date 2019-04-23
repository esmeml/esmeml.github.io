---
layout: post
title: Blog Updates
date: 2018-11-30 01:30:00 -06:00
license: cc0
published: True
categories:
- Blog
tags:
- Blog Updates
---
Yes, it's that time of year again. I have updated the blog! You can find more
information below.

#### Analytics ####
Google Analytics are back. I understand that some people may not like being
tracked, but at that point you should have an add or tracking blocker installed.
I recommend looking at the Brave Browser, which is what I personally use, or
installing uBlock Origin. The reason I have added this back to the blog is that
I have noticed links to this blog appearing in various places over the web, and
I would like to be able to detect how much traffic I am getting from these
links.

I wholeheartedly understand if you disapprove of the use of Google Analytics. If
anyone is able to suggest a better service, that collects less user data, please
open an issue in the GitHub repository for this site. I will gladly change
providers as long as the new one is also free.

#### Theme Updates ####
I forked the Lanyon repository and applied all the currently pending pull
requests. This should keep everything up to date with the latest version of
Jekyll. To make it easier for anyone else looking for that information, I
created a new PR in the lanyon repository to my mergers. You can also find them
under my GitHub site.

#### Fonts ####
Somehow the fonts on the site got nuked during the upgrade. They are back in
place now.

#### Page Speed ####
The new updates have not yet been optimized. Previously I used Google's page
load speed thingy to optimize the site. Maintaining that by hand is one of the
reasons the upgrade was so painful to implement. I'm looking for a way to
automate the process the same way that I have currently automated the link tests
that I run prior to a push. This will likely involve writing a new process in
the Rakefile, so it will take some time. In the meantime, the only thing that
is really being pulled is a few font files and the analytics script, so the
load impact _shouldn't_ be too bad.

#### Organization ####
There is a weird issue between rendering the site on my local machine and the
way GitHub pages renders it on their side. To solve this I created a new
template and appended it to all the pages that should not appear in the sidebar.
Hopefully this strikes a good balance between being able to use standardized
templates, as well as ease of use. The new template simply imports the existing
page template under a new name.

#### Images ####
While I am primarily focused on text on this blog, I have recently included a
few images. These are also hosted on GitHub, so I cleaned up the way the image
directory is laid out. This has impacted all of one image, but should make any
expansion in the future much easier.

#### Liquid Changes ####
There was some issue with the way the Liquid on the Tags page was written. This
has been corrected accordingly.

#### About Page ####
The about page has been correctly updated! My view on some of the listed issues
has evolved in recent years. You will now find those things struckthrough with
comments added underneath.

#### Future Improvements ####
There is still minifying, javascript inlining, and font work to be done to make
the page run faster. Additionally, the tags page is simply a disaster. Between
all of that and the project pages, there is still a lot left to be done.
Hopefully, all will be accomplished in time.

#### Conclusion #### {: #blog-updates-conclusion }
Hopefully all of these changes make the site a bit more enjoyable to use. I do
understand if the use of analytics bothers you. Please make an issue in the
tracker, or if someone already has, comment accordingly. If there are enough
people using this site that honestly care, I might consider removing the
analytics while I do further research.
