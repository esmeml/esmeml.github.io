---
layout: post
title: More Blog Updates
date: 2018-12-10 01:00:00 -06:00
license: cc0
published: True
categories:
- Blog
tags:
- Blog Updates
---
There are yet more updates to the blog. The first of which is that we now have
an actual domain name, which is `zyradyl.moe`. In keeping with my tradition of
complete transparency, the domain was acquired through Gandi.net, after I found
out that IWantMyName was unable to accept Discover cards. While I am still
supportive of IWMN as a company, if they don't accept my card it leaves me
unable to use them.

Next, the DNS for this site is now handled through Cloudflare, which means that
this site is now fully available via HTTPS with a valid SSL certificate. So,
small victories.

While running through the process of updating the blog, I noticed several things
were broken and went ahead and fixed those:

  * The "Site Version" link in the sidebar now properly links to the GitHub
    source repository.
  * ~~A long standing issue with pagination has been corrected by updating to
    the jekyll-paginate-v2 gem, and rewriting the appropriate liquid blocks.~~
      * Github-Pages does not support the v2 gem. Therefore, the site has been
        downgraded back to the v1 gem, and the liquid blocks were cleaned up
        based on trial and error.
  * Related posts are now actually related! This is accomplished by iterating
    through tags at compile time and creating a list of related posts. While
    this may not always be accurate, it is far more accurate than the time
    based system jekyll uses by default.
  * A small issue has been corrected with the header file used across pages.
    There was a typo that was generating invalid HTML. It didn't cause any
    visible issues, but it was a problem all the same.
  * The archive page now uses a new Liquid code block. This is to resolve the
    long standing `</ul>` problem, where the code would generate trailing
    closing tags.
  * HTTPS links have been enforced across the board. I cannot promise the site
    that you visit will have a valid SSL certificate, but we will certainly try
    to redirect the connection over SSL now.

~~HTML proofer is still throwing a few errors related to my consistent use of
the Introduction and Conclusion headers, but these are not actual errors.~~
  * Even these errors have been fixed. HTMLProofer now returns a completely safe
    site.

I'm also in the process of going back through previous posts and cleaning up
the YAML front matter. While this front-matter previously had very little
impact on the site, it now can matter quite a lot with the way the related
posts system works.
